# Mira System API — Requirements & Design Brainstorm

**Status:** Draft for review
**Date:** 2026-11-03
**Scope:** The client ↔ backend API for a durable, personal AI agent product (in the spirit of Meta's Muse), consumed by cross-platform clients (iOS, Android, web, desktop, wearables).

---

## 1. The core insight

Every mainstream LLM API — including Gemini's new Interactions API — is **turn-based and client-driven**: the client sends a request, the server responds (possibly slowly, possibly in the background), and the "conversation" is a chain of request/response pairs.

Chatting with a *person* is not like that. A person:

- receives your messages even when they're busy doing something else;
- reacts instantly ("👍", "on it", typing indicator) while the real work takes minutes or hours;
- handles you sending **three messages in a row** without treating each as a separate "turn" — they read all three and respond once, or re-plan mid-work;
- keeps working after you put your phone in your pocket, and pings you when done;
- is the *same person* on every device you reach them from.

So the fundamental design decision is: **the unit of the API is not the request/response turn — it is the durable conversation event log.** The client and the agent are both *participants* appending events to a shared, server-owned log; delivery to any device is a sync problem, not a response to an HTTP call. This is closer to how Slack/WhatsApp/Matrix work than how OpenAI/Gemini chat completions work. Gemini's Interactions API gets partway there (server-side state via `previous_interaction_id`, `background=true` tasks that survive disconnects, reconnectable streams) — we adopt those ideas, but wrap them in a messaging-grade event layer instead of exposing turn-taking to clients.

---

## 2. Requirements

### Functional

| # | Requirement |
|---|-------------|
| F1 | **Mono-thread main conversation.** One durable, unbounded conversation per user with the agent — the user's "relationship thread." (Side threads/sub-tasks exist but hang off the main thread.) |
| F2 | **Fire-and-forget sends.** The user can send N messages in quick succession without waiting for a response to any of them. Sending never blocks on agent inference. |
| F3 | **Immediate lightweight acks.** Within ~1s of a message, the agent can react with an emoji, a read receipt, a typing indicator, or a one-line ack ("looking into it 👀") — independent of the full response. |
| F4 | **Durable long-running work.** Agent tasks (research, booking, purchases) run server-side for minutes–hours, survive client disconnects and app kills, and are steerable/cancellable mid-flight. |
| F5 | **Message coalescing & interruption.** New user messages arriving while the agent is thinking are injected into the in-flight turn (re-plan) or queued — not answered as isolated turns out of order. |
| F6 | **Agent-initiated messages.** The agent can proactively append to the thread (task done, reminder fired, "your flight changed") and trigger push notifications. |
| F7 | **Cross-device continuity.** Any device can attach, backfill history from a cursor, and receive live events; state lives entirely server-side. |
| F8 | **Rich event types.** Text, media, reactions, typing, tool-call progress ("browsing united.com…"), approval requests ("OK to pay $412?"), receipts, edits/deletes. |
| F9 | **Human-in-the-loop approvals.** Consequential actions (payments, sending email) block on an explicit approval event from the user. |

### Non-functional

- **Exactly-once message intake** via client-generated idempotency keys (retries on flaky mobile networks must not duplicate messages or purchases).
- **Ordered delivery** per conversation via monotonic sequence numbers; clients detect gaps and backfill.
- **p50 < 300ms for ack events**; full responses can take arbitrarily long.
- **Offline-tolerant clients**: optimistic local echo, queued outbox, reconcile on reconnect.
- **Auditability**: the event log is the source of truth for everything the agent did (tool calls, purchases) — critical for an agent that spends money.

---

## 3. Approaches considered

**A. Turn-based API with background mode (Gemini Interactions style).**
`POST /interactions {previous_interaction_id, background:true}` → poll/stream result.
✅ Simple, matches existing SDK idioms. ❌ Turn-taking is client-driven: no clean story for rapid multi-send, agent-initiated messages, emoji acks, or multi-device fan-out — you end up bolting a sync layer on anyway.

**B. Event-log + sync protocol (messaging architecture). ← Recommended**
Append-only per-conversation event log; one write endpoint, one sync/stream endpoint; the agent runtime is just another producer/consumer of the log.
✅ Everything in the requirements falls out naturally (acks are just small early events; multi-device = cursor sync; durability = the log *is* the state). ❌ More backend machinery (event store, fan-out, push pipeline); clients must handle eventual ordering.

**C. Full bidirectional realtime session (WebRTC/Live-API style).**
✅ Lowest latency, good for voice. ❌ Session-scoped by design — the opposite of durable; reconnect/resume becomes the hard problem. Keep as a *future overlay* for voice mode, not the foundation.

**Recommendation: B**, with A's `background`/reconnect semantics reused inside the Tasks subsystem, and C reserved as a later voice transport that still writes its transcript into the same event log.

---

## 4. API design

### 4.1 Resource model

```
User
 └─ Conversation            (one "main" per user; kind: main | side)
     ├─ Event               (append-only, seq-numbered; the source of truth)
     ├─ Task                (durable agent run; emits events into the log)
     └─ Participant         (user devices + the agent; presence, cursors)
Attachment                   (content-addressed blobs, referenced by events)
```

**Event** is the universal envelope:

```jsonc
{
  "id": "evt_9f3k",            // server-assigned
  "seq": 48211,                 // monotonic per conversation — ordering & sync cursor
  "conversation_id": "conv_main_u123",
  "sender": "user:u123" | "agent:mira" | "system",
  "client_id": "cl_7b2e",      // client-generated idempotency key (user events)
  "reply_to": "evt_...",       // optional threading within the mono-thread
  "created_at": "2026-11-03T18:02:11Z",
  "type": "message | reaction | typing | receipt | task.created |
           task.progress | task.completed | approval.requested |
           approval.resolved | message.edited | message.deleted | presence",
  "ephemeral": false,           // typing/presence: fan out live, never persisted
  "payload": { ... }            // type-specific
}
```

The **emoji ack** is nothing special — it's just an early, cheap event:

```jsonc
{ "type": "reaction", "sender": "agent:mira",
  "payload": { "target": "evt_user_msg", "emoji": "👀" } }
```

### 4.2 gRPC service definition

Transport is **gRPC** (protobuf over HTTP/2). Native clients (iOS/Android/desktop/wearables) use standard gRPC stubs; browsers use **gRPC-Web or the Connect protocol** via an Envoy/Connect gateway — see the cross-platform notes below. Package is versioned: `mira.v1`.

```protobuf
syntax = "proto3";
package mira.v1;

service ConversationService {
  // Write path. Acks INTAKE (<100ms), not agent processing.
  // Idempotent on client_id (24h window): a retry returns the original event.
  rpc AppendEvent(AppendEventRequest) returns (AppendEventResponse);

  // Backfill / gap-fill: finite server stream of persisted events after a cursor.
  rpc SyncEvents(SyncEventsRequest) returns (stream Event);

  // Live fan-out: one long-lived bidi stream per device, all conversations.
  //   server → client: Event frames (incl. ephemeral typing/presence)
  //   client → server: cheap ephemeral signals (typing, receipts, presence)
  // Resume by opening a new stream with the last seen cursor.
  rpc StreamEvents(stream ClientSignal) returns (stream Event);
}

service TaskService {
  rpc ListTasks(ListTasksRequest) returns (ListTasksResponse);
  rpc GetTask(GetTaskRequest) returns (Task);          // status, plan, step log, artifacts
  rpc WatchTask(GetTaskRequest) returns (stream Task); // live progress for a task drawer UI
  rpc CancelTask(CancelTaskRequest) returns (Task);
  rpc SteerTask(SteerTaskRequest) returns (Task);      // mid-flight instruction, no cancel
  rpc ResolveApproval(ResolveApprovalRequest) returns (Event);
}

message AppendEventRequest {
  string conversation_id = 1;
  string client_id = 2;          // client-generated idempotency key (required)
  EventBody body = 3;            // type + payload (oneof below)
}
message AppendEventResponse {
  Event event = 1;               // server-assigned id, seq, timestamps
}

message SyncEventsRequest {
  string conversation_id = 1;
  int64 after_seq = 2;
  int32 limit = 3;               // server caps; stream ends when caught up
}

message Event {
  string id = 1;
  int64 seq = 2;                 // monotonic per conversation — ordering & sync cursor
  string conversation_id = 3;
  Sender sender = 4;             // USER / AGENT / SYSTEM + principal id
  string client_id = 5;
  string reply_to = 6;
  google.protobuf.Timestamp created_at = 7;
  bool ephemeral = 8;            // typing/presence: fanned out live, never persisted
  oneof payload {                // closed set per proto version; see compat note
    Message message = 10;
    Reaction reaction = 11;      // { target_event_id, emoji }
    Typing typing = 12;
    Receipt receipt = 13;
    TaskUpdate task_update = 14; // created / progress / completed / failed
    ApprovalRequested approval_requested = 15;
    ApprovalResolved approval_resolved = 16;
    MessageEdit message_edit = 17;
    Presence presence = 18;
  }
}

message ClientSignal {           // client→server frames on StreamEvents
  oneof signal {
    Resume resume = 1;           // { cursor } — first frame on (re)connect
    Typing typing = 2;
    Receipt receipt = 3;
    Presence presence = 4;
  }
}
```

Sending three quick messages = three `AppendEvent` calls; the server, not the client, decides how the agent consumes them. Tasks are *created by the agent*, not the client. Every task state change also emits a `TaskUpdate` event into the conversation, so a client that only implements `StreamEvents` still renders progress cards; `TaskService` exists for rich UI (task drawer, retry, artifacts).

**gRPC-specific mechanics:**

- **Streams are resumable, not durable.** A dropped `StreamEvents` is fine: reconnect, send `Resume{cursor}`, and the server replays from the log (or tells you to `SyncEvents` first if too far behind). Durability lives in the log, never in the connection.
- **Keepalive tuned for mobile:** client keepalive ~30s with server-enforced minimum; expect and handle `UNAVAILABLE`/`GOAWAY` on network switches — reconnect with jittered backoff is the normal path, not an error path.
- **Deadlines:** `AppendEvent` 5s, `SyncEvents` 30s; `StreamEvents`/`WatchTask` run long but the server cycles streams (~30min max age) for load-balancing — clients must treat stream end as "reconnect," not failure.
- **Push:** offline devices get APNs/FCM pushes generated from the same log (server-side rules decide which events are push-worthy); the push payload carries `conversation_id` + `seq` so the app cold-starts straight into `SyncEvents`.

**Cross-platform reality check:** browsers cannot speak native gRPC, and gRPC-Web does not support bidi streaming. The web client therefore uses the degraded-but-equivalent pair: unary `AppendEvent` + server-streaming `SyncEvents`/`StreamEvents` (bidi collapses to a server-stream; the client→server ephemeral signals go over small unary calls, e.g. `rpc PublishSignal(ClientSignal) returns (Empty)` — worth adding for this reason). The Connect protocol (connectrpc.com) is the pragmatic choice for the gateway: one proto, native gRPC for apps, fetch-friendly JSON/HTTP for web, no Envoy translation layer required.

### 4.3 The agent runtime behind the API (two-lane model)

This is what makes F2/F3/F5 work:

```
user events ─► Conversation Inbox (per-conv ordered queue)
                 │
                 ├─► FAST LANE (small/cheap model, <1s SLA)
                 │     classify + ack: emoji reaction, read receipt,
                 │     "on it", or full answer if trivial
                 │
                 └─► MAIN LANE (frontier model, single logical agent turn
                       per conversation at a time)
                       • debounce ~2–3s: coalesce burst messages into one turn
                       • if a turn is in flight: inject new message at next
                         checkpoint (re-plan) or queue (agent's choice)
                       • may spawn Tasks (durable, checkpointed workers that
                         outlive the turn and emit events as they go)
```

Key invariants: **one main-lane turn per conversation at a time** (prevents the agent arguing with itself); the fast lane may never take actions, only acknowledge; every lane's output is just events appended to the log. Tasks checkpoint their step state so a worker crash resumes rather than restarts — and any step marked consequential (payment, send-email) requires an `approval.resolved` event before executing (F9).

### 4.4 Worked example

```
t+0.0  user POSTs "find me a flight to Tokyo next month"   → seq 101
t+0.3  fast lane: reaction 👀 on 101                        → seq 102
t+1.1  user POSTs "oh and under $900"                       → seq 103
t+1.2  user POSTs "window seat if possible"                 → seq 104
t+3.5  main lane wakes (debounce), reads 101+103+104 as ONE brief
t+4.0  agent: message "On it — searching flights now ✈️"    → seq 105
t+4.1  agent: task.created {task_t9, title:"Tokyo flight"}  → seq 106
       ... user closes the app; task keeps running ...
t+6m   task.progress "found 3 options under $900"           → seq 107  → push
t+6m   approval.requested "Book ANA 212, $848, window?"     → seq 108  → push
t+9m   user (from laptop this time) resolves approval       → seq 109
t+11m  task.completed + message with itinerary card          → seq 110 → push
```

### 4.5 Versioning, auth, errors

- **Versioning:** proto package (`mira.v1` → `mira.v2`); within v1, evolve by adding fields and new `oneof payload` arms. Protobuf makes unknown `oneof` arms land in the "not set" case — clients must render an event whose payload they don't recognize as a generic "activity" row (using a parallel `string type_hint` field) rather than drop it. Never renumber or reuse field tags.
- **Auth:** OAuth2/OIDC; access token in per-RPC `authorization` metadata over TLS. Per-device tokens so a lost device can be revoked without ending the relationship thread; scopes per connected integration (email, payments…).
- **Errors:** canonical gRPC status codes (`INVALID_ARGUMENT`, `NOT_FOUND`, `RESOURCE_EXHAUSTED`, `FAILED_PRECONDITION` for e.g. resolving an already-resolved approval) with `google.rpc.Status` error details for machine-readable causes. Irrecoverable cursor gaps (log truncation) surface as `OUT_OF_RANGE` with a `ResyncRequired` detail → client refetches the tail via `SyncEvents` from the earliest retained seq.
- **Retries:** only `AppendEvent` is safely retryable (idempotent on `client_id`); use gRPC retry policy on `UNAVAILABLE` for it, manual reconnect logic for streams.

---

## 5. What we deliberately did NOT include (YAGNI)

- Multi-user / group conversations (mono-thread product; model leaves room via `Participant`).
- Client-side plugin/tool protocol (tools are server-side integrations for v1).
- Realtime voice transport (future overlay per Approach C).
- Federation, E2E encryption of the log (server must read messages to be the agent; document this honestly instead).

## 6. Open questions for review

1. **Retention:** is the main thread truly forever, or windowed with server-side memory summarization? (Impacts cursor/truncation design — `resync_required` assumes truncation is possible.)
2. **Side threads:** expose `kind:"side"` conversations in v1, or keep strictly one thread and render tasks as inline cards only?
3. Should the **fast lane** ever produce text replies, or emoji/receipts only? (Text risks contradicting the main lane.)
4. Public developer API from day one, or first-party clients only? (Affects how much of the Task model we freeze now.)
