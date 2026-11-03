# mira.v1 API Outline

One-page reference. Authoritative definitions: `proto/mira/v1/*.proto` (buf module in `proto/`).
Design rationale: `docs/superpowers/specs/2026-11-03-agent-system-api-design.md`.
Demo handoff: `docs/superpowers/specs/2026-11-03-conversation-service-e2e-demo-spec.md`.

## Principles

- The append-only, seq-ordered **Event log is the conversation**; clients and the agent are both participants.
- **Split protocol:** unary idempotent write (`AppendEvent`) + stream delivery (`SyncEvents`/`StreamEvents`). No co-owned session state; reconnect = Resume with a cursor.
- The wire protocol is **turn-blind**: steering, elicitation answers, and reattach are ordinary RPCs; the agent's turn machine lives server-side only.

## ConversationService (`conversation_service.proto`) — core, demo scope

| RPC | Shape | Purpose |
|---|---|---|
| `ListConversations` | unary | conversations + `latest_seq` / `earliest_retained_seq` for cold-start sync decisions |
| `AppendEvent` | unary | the write path; idempotent on `client_id`; acks intake (<100ms), never agent processing |
| `SyncEvents` | server-stream | finite backfill, `seq > after_seq`, ordered; `OUT_OF_RANGE` + `ResyncRequired` past retention |
| `StreamEvents` | bidi-stream | live fan-out, all conversations per device; first frame `Resume{cursors}`; ephemeral typing/presence ride it; `FAILED_PRECONDITION` + `SyncFirst` when too far behind |
| `PublishSignal` | unary | ephemeral signals for transports without bidi (gRPC-Web) |

`AppendEventRequest.body` is `EventBody` — the client-writable payload subset (message, reaction, receipt, message_edit, message_delete). Task/approval payloads are agent-only and rejected.
`ClientSignal` frames: `Resume`, `TypingPayload`, `ReceiptPayload`, `PresencePayload`, `DeliveryAck` (cumulative acked seqs; replay-buffer trim hint, correctness still rests on Resume).

## Event envelope (`event.proto`)

`Event{event_id, seq, conversation_id, Sender, client_id, reply_to_event_id, created_at, ephemeral, segment_id, type_hint, oneof payload}` with payloads:
`MessagePayload` (markdown + `AttachmentRef[]` + `Card{kind, fallback_markdown, payload_json}`), `ReactionPayload` (the instant emoji ack), `TypingPayload{active, activity_hint}`, `ReceiptPayload{read_up_to_seq}`, `PresencePayload`, `MessageEditPayload`, `MessageDeletePayload`, `TaskUpdatePayload{task_id, TaskStatus, note}`, `ApprovalRequestedPayload{summary_markdown = exact consent text, expires_at}`, `ApprovalResolvedPayload{APPROVED|DENIED|EXPIRED, note}`.
Unknown payload arms render as a generic activity row via `type_hint`. Ephemeral events: `seq=0`, never persisted, never in `SyncEvents`.

## TaskService (`task_service.proto`, `task.proto`) — post-demo

`ListTasks` (status filter, paged) · `GetTask` · `WatchTask` (server-stream snapshots) · `CancelTask` (idempotent on terminal) · `SteerTask` (mid-flight instruction, re-plan at next checkpoint) · `ResolveApproval` (first resolution wins across devices; returns the appended `approval_resolved` Event as proof).
`Task{status: RUNNING|WAITING_APPROVAL|WAITING_EXTERNAL|COMPLETED|FAILED|CANCELLED, plan_markdown, TaskStep[], artifacts}`; a `TaskStep{consequential:true}` must reference the `approval_id` that authorized it. Tasks are created by the agent, never by clients.

## MemoryService (`memory_service.proto`) — post-demo

`ListMemories` (PROFILE + STATE tiers) · `UpdateMemory` (empty id = create user-authored) · `DeleteMemory` · `ForgetSegment` (removes a segment's episodic memory; returns tombstoned derived profile facts).
`Memory{tier: PROFILE|STATE, content, source_segment_ids, timestamps}`. Episodic tier is internal.

## Reserved seams (no proto yet, by design)

- **WorkspaceService** — reactive shared state (Get/Watch/Mutate, epoch-fenced single writer) for live interactive surfaces; v1 ships static Cards only.
- **Live media** — multiplexed audio/video session envelope, added alongside the event log if full-duplex voice/video becomes core.
