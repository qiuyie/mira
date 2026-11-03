# ConversationService E2E Demo — Implementation Spec

**Status:** Ready for implementation handoff
**Date:** 2026-11-03
**Audience:** an implementing agent. This document is self-contained: everything needed to build and verify the demo is here or in the referenced proto files. The broader product/design rationale lives in `2026-11-03-agent-system-api-design.md` (§4.1–4.3, §4.8) and is background reading, not a dependency.

---

## 1. Goal

Build a working end-to-end demo of `mira.v1.ConversationService` (proto already in this repo at `proto/mira/v1/`) that proves the core protocol properties of the split design:

1. **Fire-and-forget sends** — a client can send a burst of messages without waiting for agent output.
2. **Immediate lightweight acks** — an emoji reaction event arrives within 1s of each user message, long before the full reply.
3. **Message coalescing** — a burst of N quick messages produces ONE agent reply that addresses all of them.
4. **Durable log + resumable streams** — kill the stream mid-conversation, reconnect with a cursor, and receive exactly the missed events, in order.
5. **Exactly-once writes** — retrying `AppendEvent` with the same `client_id` after a lost ACK returns the original event; no duplicate messages.

The demo is **deterministic and offline**: the agent is scripted (no LLM calls, no API keys). The agent sits behind an interface so an LLM-backed implementation can be swapped in later; that swap is out of scope.

## 2. Scope

**In scope:** `ConversationService` only — all 5 RPCs (`ListConversations`, `AppendEvent`, `SyncEvents`, `StreamEvents`, `PublishSignal`) — plus the minimal in-process agent runtime (fast lane + main lane) needed to demonstrate the properties above, an in-memory event store, a demo CLI client, and the acceptance tests in §7.

**Out of scope:** `TaskService`, `MemoryService`, approvals, segments/`segment_id` (leave empty), attachments/blob upload (`AttachmentRef` may appear in protos but no upload path), auth (single hardcoded user `u-demo`, conversation `conv-main`), persistence across server restarts, push notifications, gRPC-Web gateway, retention/TTL, `DeliveryAck`-based buffer trimming (accept the frame, log it, no-op).

## 3. Stack and layout

- **Language:** Go ≥ 1.22. gRPC via `google.golang.org/grpc` + `google.golang.org/protobuf`.
- **Codegen:** `buf generate` against the existing module in `proto/` (add `proto/buf.gen.yaml` with `protoc-gen-go` and `protoc-gen-go-grpc` plugins; generated code under `gen/go/`). Do not modify the `.proto` files except: if a demo gap is found, propose the change in the PR description rather than silently editing.
- **Repo layout:**

```
cmd/mirad/main.go          server binary (flag: --addr, default :50051)
cmd/mira-cli/main.go       demo client binary
internal/log/              event log store (in-memory) + types
internal/hub/              per-conversation fan-out to live streams
internal/agent/            inbox, fast lane, main lane, Agent interface
internal/server/           grpc service implementation, validation
gen/go/                    generated protobuf/grpc code (committed)
e2e/                       acceptance tests (go test, real server over bufconn or localhost)
Makefile                   gen / build / test / demo targets
```

## 4. Component specifications

### 4.1 Event log (`internal/log`)

The single source of truth. In-memory, one instance per conversation.

- `Append(body, sender, clientID) (Event, dup bool)` — assigns `event_id` (UUIDv7 or ULID), `seq` (monotonic, starts at 1, no gaps), `created_at`; appends under a mutex.
- **Idempotency (REQ-W1):** keep a map `client_id → event_id` for user-originated events. A repeated `client_id` returns the ORIGINAL event with `dup=true` and appends nothing. (Demo keeps the map forever; production would window 24h.)
- `Read(afterSeq, limit) []Event` — ordered slice of persisted events with `seq > afterSeq`.
- Ephemeral events (typing/presence) never enter the log and never consume a seq (`seq=0`, `ephemeral=true`).

### 4.2 Hub (`internal/hub`)

Fan-out of events to live device streams.

- `Subscribe(convID) (ch <-chan Event, cancel func())` — registered streams receive every event appended after subscription, plus ephemeral events, in append order.
- Appends publish to the hub AFTER the log commit (REQ-D1: a client can never see an event on the stream that a crash would un-persist).
- Slow consumer policy: buffered channel (size 256); on overflow, drop the subscriber and close its channel — the client's normal reconnect-with-cursor path recovers it (this is deliberate; do not build per-subscriber replay buffers).

### 4.3 Agent runtime (`internal/agent`)

**Inbox:** every persisted user `message` event is enqueued per conversation.

**Fast lane (REQ-A1):** on each user message, within 1s, append a `reaction` event from `sender{AGENT,"mira"}` targeting that message. Scripted rule: text containing `?` → "🤔"; containing "book"/"buy"/"find" → "👀"; otherwise "👍".

**Main lane (REQ-A2, A3):** a per-conversation worker that:
- **Debounces:** on a new user message, wait `debounce` (default 2s, configurable via flag for tests, e.g. 200ms) and drain any further user messages that arrive meanwhile into one burst.
- **Single-flight:** at most one main-lane turn runs per conversation. A user message arriving mid-turn joins the NEXT burst (demo does not implement mid-turn injection).
- **Emits, in order:** an ephemeral `typing{active:true, activity_hint}` event; then (after a simulated think of 1s) one `message` event whose markdown enumerates every message in the burst, e.g. `Got it — acting on 3 messages: (1) "...", (2) "...", (3) "..."`; then `typing{active:false}`.

**Agent interface (extension seam):**
```go
type Agent interface {
    // Reply is called once per coalesced burst with full burst texts.
    Reply(ctx context.Context, burst []string) (markdown string, err error)
}
```
`ScriptedAgent` implements it as above. The runtime must depend only on this interface.

### 4.4 gRPC service (`internal/server`)

Implements `mira.v1.ConversationServiceServer` against the log + hub + agent runtime.

- `ListConversations` → the one demo conversation with live `latest_seq`, `earliest_retained_seq = 1`.
- `AppendEvent` (REQ-W1..W3):
  - validate: known `conversation_id`, non-empty `client_id`, payload is one of the `EventBody` arms → else `INVALID_ARGUMENT`.
  - append (idempotent), publish to hub, enqueue to agent inbox (only on `dup=false`), return the persisted event. Target: returns without waiting on any agent work.
- `SyncEvents` (REQ-R1): stream persisted events `seq > after_seq` in order, honoring `limit` (default/cap 500), then end the stream. `after_seq` beyond `latest_seq` → empty stream, OK. (Retention never truncates in the demo, so `OUT_OF_RANGE` is not reachable; still return it if `after_seq < earliest_retained_seq - 1` to keep the client contract honest.)
- `StreamEvents` (REQ-R2..R4):
  - first client frame MUST be `Resume` → else `FAILED_PRECONDITION`.
  - on Resume: subscribe to hub FIRST, then replay persisted events after the per-conversation cursor from the log, then deliver live events, deduplicating by seq (replay/live overlap is resolved by "skip seq ≤ last sent"). This ordering guarantees no gap between replay and live (REQ-R3).
  - subsequent frames: `typing`/`presence` → wrap as ephemeral agent-visible events and publish to hub (typing from the user is fanned out to the user's other devices, not to the agent); `receipt` → append as a persisted receipt event; `delivery_ack` → accept and ignore (log it).
  - wrap every outbound event in `StreamEventsResponse`.
- `PublishSignal`: same handling as in-stream signals, unary.
- Errors use canonical codes throughout; no panics across RPC boundaries.

### 4.5 Demo CLI (`cmd/mira-cli`)

Two subcommands, enough for a human to run the demo in two terminals:

- `mira-cli chat` — interactive: opens `StreamEvents` (Resume from a cursor persisted in `~/.mira-demo-cursor`), renders incoming events (messages, reactions as `[mira reacted 👀 to #seq]`, typing as a transient status line), and sends each typed line via `AppendEvent` with a fresh UUID `client_id`. Sending never blocks on the stream.
- `mira-cli burst "msg1" "msg2" "msg3"` — fires N `AppendEvent`s back-to-back (no delay) and exits; used to demonstrate coalescing while `chat` watches in another terminal.

## 5. The demo script (what "done" looks like)

Terminal A: `mirad`. Terminal B: `mira-cli chat`. Terminal C: `mira-cli burst "find me a flight to tokyo" "under $900" "window seat if possible"`.

Expected in Terminal B, in this order:
1. three user messages appear (seq N, N+1, N+2);
2. within 1s of each: a 👀/👍 reaction from mira;
3. one typing indicator with an activity hint;
4. ONE mira reply addressing all three messages;
5. Ctrl-C Terminal B, send another burst from C, restart B → it resumes from its cursor and shows exactly the missed events, no duplicates, no gaps.

## 6. Numbered requirements (normative)

| ID | Requirement |
|----|-------------|
| REQ-W1 | `AppendEvent` with a previously seen `client_id` returns the original event; the log length does not change; the agent inbox is not re-enqueued. |
| REQ-W2 | `AppendEvent` returns without blocking on fast-lane or main-lane work (enqueue only). |
| REQ-W3 | `EventBody` arms other than message/reaction/receipt/message_edit/message_delete, unknown conversation, or empty `client_id` → `INVALID_ARGUMENT`. |
| REQ-D1 | No event is observable on any stream before it is committed to the log (ephemeral events exempt). |
| REQ-R1 | `SyncEvents(after_seq=k)` returns exactly the persisted events with `seq>k`, in ascending seq order. |
| REQ-R2 | `StreamEvents` whose first frame is not `Resume` fails with `FAILED_PRECONDITION`. |
| REQ-R3 | After `Resume{cursor:k}`, the client receives every persisted event with `seq>k` exactly once and in order, regardless of concurrent appends during the replay/live handover. |
| REQ-R4 | Ephemeral events are delivered to live streams, carry `seq=0`, `ephemeral=true`, and never appear in `SyncEvents`. |
| REQ-A1 | Each non-duplicate user message gets exactly one agent `reaction` event within 1s. |
| REQ-A2 | A burst of messages arriving within the debounce window yields exactly one main-lane `message` reply referencing every burst message. |
| REQ-A3 | At most one main-lane turn is in flight per conversation; messages arriving mid-turn are answered by a subsequent single reply, never dropped. |

## 7. Acceptance tests (`e2e/`)

Go tests against a real server (bufconn is fine; one test may use a TCP listener). Use a short debounce (200ms) via config. Each test maps to requirements:

1. **TestIdempotentAppend** (REQ-W1): append, append again with same `client_id`; assert same `event_id`/`seq`, log length 1 message, exactly one reaction ever arrives.
2. **TestBurstCoalescing** (REQ-A1, A2): send 3 messages < debounce apart; assert 3 reactions, then exactly 1 reply mentioning all 3 texts; assert reply seq > all reaction seqs.
3. **TestFireAndForget** (REQ-W2): with the agent runtime artificially slowed (inject a blocking ScriptedAgent), assert `AppendEvent` p99 returns < 100ms.
4. **TestResumeNoGapNoDup** (REQ-R3): open stream, receive through seq k, hard-cancel the stream, append 5 events while disconnected, reconnect `Resume{k}`; assert received seqs are exactly k+1..k+5. Run with `-race`.
5. **TestReplayLiveHandover** (REQ-R3): resume with cursor 0 while a goroutine appends concurrently; assert the merged delivery is strictly ordered and duplicate-free up to the final seq.
6. **TestSubwayScenario** (REQ-W1, R3 — the §4.8 trace): mid-turn, send a message, drop the stream, "lose" the AppendEvent response (client ignores it), then in parallel retry AppendEvent (same `client_id`) and Resume; assert: retry returns the original seq; the stream delivers that seq exactly once; total user messages in log = 1.
7. **TestEphemeral** (REQ-R4): typing signals reach a second live stream but never `SyncEvents`.
8. **TestStreamHandshake** (REQ-R2) and **TestValidation** (REQ-W3): error-code assertions.

CI gate: `buf lint` + `buf build` on `proto/`, `go vet`, `go test ./... -race`. All green = spec satisfied.

## 8. Implementation order (suggested milestones)

1. `buf.gen.yaml` + codegen committed; server skeleton serving `ListConversations`.
2. Event log + `AppendEvent` + `SyncEvents` (+ tests 1, 8-validation).
3. Hub + `StreamEvents` resume/handover (+ tests 4, 5, 7, 8-handshake).
4. Agent runtime: fast lane, then main lane with debounce (+ tests 2, 3, 6).
5. CLI + demo script in §5 verified manually; README snippet with the three-terminal walkthrough.

Keep commits per milestone. Do not start a milestone with failing tests from the previous one.

## 9. Resolved-by-fiat decisions (do not re-open during implementation)

- Go, in-memory store, scripted agent, single user/conversation — demo simplicity beats realism.
- Slow stream consumers are dropped, not buffered (reconnect path is the recovery mechanism).
- Mid-turn steering/injection is NOT implemented; coalescing into the next turn is sufficient for the demo (the design doc's §4.3 injection behavior is a production concern).
- `DeliveryAck` is accepted but inert.
- Generated code is committed so the repo builds without buf installed.
