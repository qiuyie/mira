# Mira, the map companion: product spec

Status: draft for review, 2026-11-22. UI mocks and the character sheets: https://claude.ai/artifact/K31Jt8FNidngqVZU2gUf3Z
Backend references: `docs/api-outline.md`, `proto/mira/v1/*.proto`.

## 1. Summary

Mira is an AI agent that lives on a mobile map as a small glowing figure at the top-centre of the screen. The map is full-bleed with no search bar: tapping Mira, or the composer handle at the bottom edge, is how the user asks for anything. She reads where the user is, what the map shows, and what the user types or says, and she answers in the smallest form that works, from one line under her to a full itinerary. Her glow is the status display, and it is also the user's proof of when she is reading location or the viewport. A bottom sheet holds the conversation and sorts her work into three shelves: Now, Ongoing and From Mira. The backend already has the durable parts (event log, Tasks, approvals, Cards, memory); it has no notion yet of a viewport, a place, or how loudly an agent-initiated event may speak (section 12).

## 2. Principles

| Principle | Consequence |
|---|---|
| The map is the product; Mira is a guest on it. | Nothing she draws blocks panning, and she moves the camera only when the user asked. |
| Mira is the entry point. | Every query goes to her, so a plain place-name search must still feel instant. |
| One surface, two views. | Map and sheet share one selection and one subject. |
| The smallest form that answers. | She starts small and the user opens it up. She never opens the sheet herself. |
| The glow never lies. | If she reads location or the viewport, she is visibly Noticing or Working. |
| Attention is a budget. | Viewport noticing stops at a whisper. Only approvals persist until answered. |
| Consequential actions need the user's yes. | Booking, paying and sending go through an approval whose text is the exact consent. |

## 3. The figure

### 3.1 Who she is

**Mira is a map pin that came alive.** A pin is a teardrop pointing down, stuck to one place. Turn it over and let it float, and it becomes a small flame of light that travels with you. That inversion is the whole product idea in one shape: a pin marks where you are going, Mira goes with you.

The name carries two meanings. Mira is the star omicron Ceti, "the wonderful", the first star people saw brighten and fade, which is what the figure does. In Spanish, "¡mira!" means "look!", which is what a companion on a walk says.

Her character is warm, curious and calm. She is a companion, and she does not perform for attention. If nothing is worth saying she says nothing, and the figure shows that by resting.

### 3.2 Anatomy

| Part | What it is | What it expresses |
|---|---|---|
| Belly | A round body of honey light (`#FFB627` to ember `#FF7A3D`) | Her presence. It is the only part that is always visible |
| Tip | The curled point on top, the old pin point | Her mood and attention. It sways, perks, cocks like an ear, wags, and flops when she sleeps |
| Face | Two ink dot eyes set low, a blush, a mouth that disappears at small sizes | Where she is looking. The eyes follow the direction of the pan |
| Halo | A soft radial glow behind her | How much she is doing. Brightness and rhythm change with work; nothing else does |
| Motes | Up to three small lights in orbit | One per running Task. The same motes appear beside each row in Ongoing |
| Ring | A steady blue circle | It is the user's turn. Blue is the user's colour everywhere in the app |

She ships at a 52 px body, top-centre, 58 px from the top edge, and never moves from there except at the Full detent, where she rides the sheet header. She scales to 24 px in a notification row, 120 px for voice and 240 px for brand use.

### 3.3 Signature motion per state

| State | Motion |
|---|---|
| Resting | Slow float and halo breath on a 4.8 s cycle, a lazy tip sway, a blink about every 5 s |
| Noticing | Body leans and the face slides toward the pan, the tip trails behind, a sheen sweeps across the belly as the scan |
| Working | Halo pulses on a 1.8 s beat, the tip ticks in time, one mote orbits per running Task |
| Has-something | Brightest halo, a ring of short rays blooms once, a quick squash and perk, and the whisper unfurls beneath her |
| Listening | Body widens, the tip cocks to one side, and two rings track voice amplitude |
| Composing | **The flip.** She spins tip-down and is a pin again for a moment, pouring light from the tip onto the map, then pops back upright |
| Needs-you | The blue ring appears and holds. She is almost still and looks straight out. Nothing pulses |
| Delivered | Squash, hop, land, with smiling eyes and three small sparkles, played once |
| Dimmed | Grey, almost no halo, sitting lower, eyes closed, tip flopped over |

The flip is the gesture that ties the figure to the map. Every mark she places arrives along a thread of light from her tip, so the user sees where an answer came from and that it belongs to her.

### 3.4 Motion principles

1. **Light before motion.** Brightness is the first signal and movement the second. A state must be readable from the halo alone, which is also what the reduced-motion version shows.
2. **One signal at a time.** She shows one state, chosen by the precedence below. Only the ring and the motes persist under other states.
3. **She never bounces for attention.** No state loops a hop, a shake or a badge count. Delivered and Has-something play once and settle.
4. **Slower than the map.** Her resting rhythms are slower than a pan, so she reads as calm next to the thing the user is moving.
5. **Honest glow.** If she is reading the viewport or the location, she is visibly Noticing or Working. There is no silent mode in which she looks idle and is not.

### 3.5 Colour

Honey is Mira: her body, her marks on the map, her messages, and place references she makes. Blue (`#2F6BFF`) is the user: the location dot, the user's messages, and anything waiting on the user's decision. No other accent colours are used, and neither colour is the only signal for anything (see section 11).

### 3.6 Directions considered

Three figures were built and compared as animated studies at 240, 120 and 52 px on light and dusk maps.

| Direction | What it gave | Why it was or was not chosen |
|---|---|---|
| A. Star-sprite, a plump star with rays | Warm, and the rays can form a compass | Reads as a familiar game star and sits close to the generic AI sparkle |
| **B. The pin that came alive (chosen)** | One silhouette that is clear at 52 px, a tip that acts, and the flip | The only direction with a reason to live on a map |
| C. Lantern creature, a round sprite with fins and a glowing belly | The most character and charm at large sizes | Too many parts at 52 px, and the glow is only a third of her body |

### 3.7 States and signals

| State | Trigger | What the user can do | Backing API signal |
|---|---|---|---|
| Resting | Nothing running, nothing unread | Tap to ask; long-press for status | No agent `TypingPayload`; no `RUNNING` Task |
| Noticing | Camera settles and the viewport is sent to her | Keep panning (cancels); tap to ask about this view | Client-local, tied to sending the viewport. **Gap:** no viewport signal |
| Working | One or more Tasks are `RUNNING` | Tap to open Ongoing; long-press to see why | `TaskUpdatePayload`, `WatchTask`; one mote per running Task |
| Has-something | She has a line worth saying | Tap the whisper to open it up, swipe it away, or ignore it | Agent `MessagePayload`. **Gap:** no attention level |
| Listening | User holds the composer handle | Speak; release to send; slide away to cancel | Client-local. **Gap:** live media seam has no proto |
| Composing | She is answering or building UI | Wait, or interrupt with a new message | Agent `TypingPayload{active, activity_hint}` until the message lands |
| Needs-you | An approval is open | Read the consent text; approve or deny | Unresolved `ApprovalRequestedPayload`; Task `WAITING_APPROVAL` |
| Delivered | A Task completes | Tap to open the result | `TaskUpdatePayload{COMPLETED}` |
| Dimmed | Offline, paused, or location off | Long-press to resume or see what is off | Client-local. **Gap:** no server-side pause |

The table forces two decisions. A Task in `WAITING_EXTERNAL` (a table watch that waits three days) does not pulse: Working means compute is happening now, so the watch is a parked mote on a Resting figure. A `FAILED` Task has no state of its own: she shows Has-something with a plain whisper, without the bloom or the hop, and the tip drooped.

When several states apply, the proposed precedence is Dimmed, Listening, Needs-you, Composing, Has-something, Noticing, Working, Resting. The Needs-you ring stays visible under any state shown above it.

## 4. Anatomy of the screen

The screen is four layers: (0) the base map, full-bleed; (1) *map marks*, the things Mira draws on the map (lit areas, labelled pins, routes, annotations); (2) Mira at about 52 px with one optional *whisper* chip under her; (3) the sheet.

| Detent | Height | Contents | Where Mira is |
|---|---|---|---|
| Closed | Composer handle only | Map, marks, card rail if present | Top-centre of map |
| Peek | About 15% | Composer and one line of current context | Top-centre of map |
| Half | About 50% | Conversation or result list; map stays live | Top-centre of map |
| Full | Full height | Threads home | Rides the sheet's top edge as its header |

```
 Peek, with a card rail                 Half, with a result list
+----------------------------+        +----------------------------+
|         status bar         |        |         status bar         |
|           (Mira)           |        |           (Mira)           |
|   [ 4 lunch spots here ]   |        |                            |
|                            |        |    o          (o) <- lifted|
|   o        (o)             |        |         o                  |
|        o          o        |        +============================+
|                            |        |  Lunch, in this view       |
| [card ][card ][card ][ca   |        |  > Tomo Ramen      4 min   |
+============================+        |    Verde           6 min   |
|  Lunch, in this view       |        |    Pilar's         7 min   |
|  [ Ask Mira...   (hold) ]  |        |  [ Ask Mira...   (hold) ]  |
+----------------------------+        +----------------------------+
```

The composer handle is the thumb's entry point: tap to type, hold to talk. Tapping Mira does the same thing as tapping the handle, so her top-centre position costs no reach. Assumption: she sits far enough below the status area to clear a camera cutout; the offset is per-device layout work.

## 5. Inputs

| Input | What it means | What Mira does | Privacy note |
|---|---|---|---|
| Passive location | "I am here, moving this way" | Checks saved places and Ongoing threads against it. Stops at a whisper unless a thread asked for alerts. | Needs the OS permission. Used for the check, not kept as a trail (assumption). |
| Viewport | The deictic "here" | On camera settle (assumption: about 600 ms idle) the client sends bounds and zoom. She shows Noticing and may whisper. | Sent only while she shows Noticing. |
| Tap on Mira | "I want to ask" | Sheet to Peek, keyboard up. If a whisper is showing, the tap opens it instead. | None |
| Long-press on Mira | "What are you doing?" | Opens the status panel: what she is doing and why, inputs in use, pause switch. | This is the privacy control surface. |
| Tap on a mark | Select it | Mark lifts, callout opens, the matching list row scrolls into view. | None |
| Drag or long-press a mark or map point into the composer | "About this" | A context chip appears; the next message is about that place. | Sent only with the message. |
| Text | Explicit request | Composing, then the smallest form that answers. | Persisted in the event log with a snapshot of viewport and selection. |
| Voice | Text, with hands busy | Listening while held. | v1 assumption: transcribed on the device, so audio stays there. |

Scope resolves by rule: "here" and unqualified requests mean the viewport, "near me" means the device location, and a context chip overrides both. Every answer names its scope ("in this view"), so a wrong guess is one tap to fix.

## 6. Outputs and generative UI

Each form has an "open it up" gesture that moves to the next row.

| Form | When | Example | Wire mapping |
|---|---|---|---|
| Whisper | One fact or one offer | "4 lunch spots in this view" | One-line `MessagePayload.markdown`. **Gap:** attention level |
| Map marks | The answer is where things are | A lit block, six pins | `Card{kind: "map_marks"}`, geometry in `payload_json`. **Gap:** placement hint |
| Mark callout | Detail on the selected mark | Name, closing time, walk time | Same Card, one item expanded by the client |
| Card rail | 3 to 8 comparable options at Closed or Peek | Lunch cards over the map | `Card{kind: "options_list"}`, placement rail |
| Sheet list or comparison | More than a rail holds | Twelve restaurants, sortable | `Card{kind: "options_list"}`, placement sheet |
| Full canvas | A structured plan | Weekend itinerary by day | `Card{kind: "itinerary"}`. Static in v1; live editing needs the reserved WorkspaceService |
| Off-app | Time-critical, or navigating | Push, lock-screen live activity | Push from server-side rules over the log. **Gap:** no priority field |

The backend defines Cards as a closed, versioned vocabulary, and we keep that. "Conjured on the fly" means Mira composes the content and picks the kind and size; it does not mean arbitrary layout. A map-placed Card's `fallback_markdown` must read correctly as a list, because a screen reader and an older client get exactly that.

## 7. Map and sheet choreography

**Camera padding.** The visible region is the screen minus a top inset (Mira plus whisper, about 120 px with the safe area) and a bottom inset equal to the sheet height. Padding follows the sheet drag frame by frame, so the subject stays centred in what is visible. At Full the camera freezes.

**Camera ownership.** The user's gestures always win. Mira moves the camera only after a user action (tapping a place reference, a list row, or "show on map"). A proactive whisper never moves it.

**Shared selection.** There is one selected item. Tapping a mark, a rail card or a list row sets it: the mark lifts, the row highlights, and the camera moves the minimum distance needed to show the mark.

**Scroll to lift.** As the list or rail scrolls, the mark for the topmost row lifts. Scrolling does not move the camera, because a list that drags the map around is hard to read.

**Place references.** Every place named in the conversation is tappable and flies the camera there. At Full, the sheet drops to Half first.

**Pan while a result list is open.** The list re-sorts at once by distance to the new centre, using results it already has. On settle, Mira shows Noticing and fetches for the new viewport. New results arrive under a "3 new in this view" row and never replace rows under the user's finger. The selected item stays pinned even if it leaves the viewport, and a hand-sorted list keeps its order.

**Pan with no list open.** Mira shows Noticing and may whisper. Nothing else changes.

## 8. Threads: Now, Ongoing, From Mira

A *thread* is one piece of work the user can return to. Threads home shows three shelves, with search over past conversations below them.

| Shelf | Holds | Starts when | Ends when | Backing |
|---|---|---|---|---|
| Now | Things tied to this viewport or moment | The user asks about "here" or opens a viewport whisper | The user moves on (assumption: the viewport no longer overlaps and 30 minutes pass), unless kept | A topic segment (`segment_id`) of the main conversation |
| Ongoing | Long-horizon work | Mira creates a Task, or the user keeps a Now thread | Its Tasks are terminal and the result is seen, or the user ends it | A side Conversation (`KIND_SIDE`) with zero or more Tasks |
| From Mira | Things she started | She appends an agent-initiated message | The user replies or keeps it (to Ongoing), dismisses it, or it expires | Agent events. **Gap:** no initiator or expiry field |

The shelves mix two axes, lifetime and initiator. So From Mira is an inbox: an item leaves it the moment the user acts, and every thread sits on exactly one shelf.

"Evaporates" means the thread leaves the Now shelf. The log is append-only, so its messages stay in history and search until the user deletes them or calls forget (`ForgetSegment`).

Every Now thread and every card has a Keep action, which moves the thread to Ongoing. If the intent is unclear Mira asks once ("Keep as a list, or watch for a table?"). A kept list with no Task is still Ongoing and has no mote.

One durable main conversation sits under all of it. The API ships exactly one `KIND_MAIN` conversation today; `KIND_SIDE` is reserved, and the backend spec leans against shipping it in v1. Question 1 asks how to resolve that.

## 9. Proactivity and attention budget

| Rung | Cost to the user | Allowed when |
|---|---|---|
| Glow only | None | Always |
| Whisper | A glance. Dismisses itself (assumption: about 6 seconds), never blocks the map | She has something specific and current |
| Message in the sheet | A dot on the composer handle | The item belongs to a thread |
| Push notification | An interruption | Time-critical, or the user asked for alerts on that thread |

Hard rules:

1. Viewport noticing never escalates past a whisper on its own.
2. Only Needs-you persists until answered. Every other rung expires.
3. One whisper shows at a time. A new one replaces the old one.
4. A dismissed whisper is not repeated on the same topic in the same session.
5. No whisper appears while the user is typing, speaking or dragging the map.
6. A push states its reason ("You asked me to watch this").
7. Assumption: a user-adjustable daily cap on whispers and pushes exists. Its numbers are unmeasured.

## 10. Scenarios

**(a) Lunch nearby**

1. Resting, Closed. The user taps the composer handle and types "anything good for lunch?".
2. Composing, Peek. The message carries the viewport snapshot; the hint reads "looking around this view".
3. Composing. Light streams to six map marks, then a card rail appears.
4. Resting, Peek. The context line reads "Lunch, in this view". Swiping the rail lifts each card's mark.
5. The user drags to Half. The rail becomes a sheet list and the camera re-pads to keep all six marks above the sheet.
6. The user pans east. The list re-sorts; on settle, Noticing, then "2 new in this view".
7. The user taps a row: the mark lifts with a callout. The thread sits in Now.

**(b) Wandering**

1. Resting, Closed. The user pans across an unfamiliar neighbourhood.
2. Noticing on each settle. Most settles yield nothing and she returns to Resting.
3. Has-something. Whisper: "This street has three old bookshops."
4. The user ignores it. The whisper fades and nothing is saved.
5. Two pans later, Has-something on a covered market. The user taps the whisper.
6. Composing, Closed. A lit area and three marks appear, with a callout on the market.
7. The user taps Keep. The market becomes a saved place and the thread moves from Now to Ongoing.

**(c) Weekend trip**

1. Half. The user holds the handle and says "plan a weekend in Porto next month". Listening, then Composing.
2. Mira creates a Task. Working, one mote. A draft itinerary Card opens a new Ongoing thread.
3. The user closes the app. The Task keeps running server-side.
4. Next day: Delivered on open. The thread shows the full-canvas itinerary, and at Half the map draws the route between stops.
5. The user asks her to book the guesthouse. Working, then Needs-you. A push goes out because the hold has an `expires_at`.
6. The user taps Mira and reads the exact text ("Book Casa do Rio, 2 nights, EUR 214, free cancellation to 4 Dec"), then approves. The client calls `ResolveApproval`.
7. Working, then Delivered. A receipt Card joins the thread.
8. The restaurant has no table, so that Task is `WAITING_EXTERNAL`: a parked mote until a table opens.

**(d) Proactive nudge while walking**

1. The user is walking with the phone in hand. Resting, Closed.
2. A location update puts a saved bakery two minutes off the route, closing in an hour.
3. Has-something. Whisper: "Padaria Lume is 2 min off your route, closes at 6."
4. The user taps the whisper. Composing: the detour draws as a route mark, with the added time in a callout.
5. The user taps "Add stop". The route updates and the item leaves From Mira for the walk's thread.
6. With the phone locked, the same nudge is a push only if the user asked for alerts on saved places. Otherwise it waits in From Mira.

## 11. Trust, privacy, accessibility

**Honesty of the glow.** The client sends a viewport or location signal only while it renders Noticing or Working. The status panel lists each input in use with its reason ("Reading this view because you asked about lunch"). This is a client contract today; nothing on the server enforces it.

**Pause.** The pause switch sets Dimmed. The client then sends no viewport or location, and proactive work stops. Tasks the user started keep running, and the panel says so. An open approval stays answerable.

**Memory.** A "What Mira knows" screen lists PROFILE memories (editable, deletable) and STATE memories (open commitments, read-only, each linked to its thread). "Forget this" on a thread calls `ForgetSegment`. Assumption for the lead to confirm: Mira stores places the user saved and facts the user stated, and no movement trail.

**Accessibility.**

- Each state has a spoken name, announced politely on change: "Mira, working on 2 tasks", "Mira needs your approval". Noticing is not announced, because it fires on every pan.
- Colour never carries state alone. Each state differs in shape or motion (ring for Needs-you, motes for Working, closed eyes for Dimmed), and the ring differs from the honey glow in lightness as well as hue.
- With reduced motion on, pulses and hops become static changes (ring, mote count, badge), camera flights become cuts, and light streams become fades.
- Every map-placed Card is also a list in the sheet, so marks are reachable without the map.
- With a screen reader on, whispers stay until dismissed.
- Hold-to-talk has a tap-to-toggle alternative. Mira, the whisper and the composer handle each have a target of at least 44 pt.

## 12. API mapping and gaps

| UX need | Existing `mira.v1` primitive | Gap |
|---|---|---|
| Durable conversation across devices | Event log, `SyncEvents`, `StreamEvents` | None |
| "She's answering", with a reason | `TypingPayload.activity_hint` | None |
| Background work, motes, Ongoing | `Task`, `TaskStep`, `ListTasks`, `WatchTask` | No recurring or watch Task kind; no geo anchor |
| Approvals | `ApprovalRequestedPayload`, `ResolveApproval` | None |
| Structured results | `Card{kind, fallback_markdown, payload_json}` | No placement hint; map kinds and geometry schema undefined |
| Viewport and location as input | `ClientSignal` on `StreamEvents`, `PublishSignal` | No context signal (bounds, zoom, location, heading, selection); no durable context snapshot on `AppendEvent` |
| Whisper-only and push-worthy events | Push from server-side rules | No attention level, expiry or initiator on agent events |
| Viewport whispers that leave no trace | Ephemeral events (typing, presence) | No ephemeral agent message; no persist-on-tap |
| Tappable places, "about this" | `MessagePayload.markdown`, `AttachmentRef` (blobs only) | No place reference type |
| Threads and shelves | `Conversation.Kind.KIND_SIDE`, `segment_id` | One main conversation in v1; no title, shelf or kept flag; Keep has no write path, since clients cannot create Tasks |
| Voice | Reserved live media seam | No proto |
| Live itinerary editing, selection shared across devices | Reserved WorkspaceService seam | No proto; selection is client-local in v1 |
| Editable memory | `MemoryService` PROFILE and STATE, `ForgetSegment` | Memories are free text; no saved-place structure |
| Pause | `PresencePayload` | No server-side pause of proactive work |
| Nudges while backgrounded | Push | No path for background location to reach the agent |

## 13. Open questions

1. **Are threads side Conversations or a client projection?** Side Conversations give each thread its own history but reverse the backend's v1 lean. A projection over `segment_id` and Tasks ships sooner but cannot hold a kept list that has no Task.
2. **Is ambient viewport noticing on by default?** On makes wandering work from day one. Off makes the first Noticing an explicit choice. The answer sets the onboarding flow and the default cost per session.
3. **Are viewport whispers persisted?** Ephemeral whispers leave no trail, but need a new ephemeral agent message and a persist-on-tap path.
4. **What does "evaporate" promise?** Leaving the shelf (proposed) or deletion from history. Deletion contradicts the append-only log and needs a retention rule.
5. **Where are background location triggers evaluated?** On the device (geofences on saved places, private, capped by OS quotas) or on the server (flexible, uploads location while backgrounded). This decides whether scenario (d) works with the phone locked.
6. **Who enforces the attention budget?** The server knows all pending items; the client knows whether the user is mid-gesture. A split needs a rule for which side may downgrade.
7. **Is a drooped Has-something enough for `FAILED`?** Section 3 decides that failure reuses Has-something with a drooped tip and no bloom. If testing shows people miss failures, the vocabulary needs a tenth state.
8. **Does a plain place-name search go through the agent?** With no search bar, "Lisbon airport" lands on Mira. If her first response is slower than a geocoder, the client needs a local fast path that still writes to the log.
