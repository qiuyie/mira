# Mira, the map companion: product spec

Status: draft for review, 2026-11-22. v2: navigation frame, lens, one thread with strands. UI mocks and the character sheets: https://claude.ai/artifact/K31Jt8FNidngqVZU2gUf3Z
Backend references: `docs/api-outline.md`, `proto/mira/v1/*.proto`.

## 1. Summary

Mira is an AI agent that lives on a mobile map as a small glowing figure in a round glass button, the *lens*, at the top-centre of the screen. The map is full-bleed with no search bar: the composer at the bottom edge is how the user asks for anything. She reads the user's location, the map view and what the user types or says, and answers in the smallest form that works, from one line under the lens to a full itinerary. Her glow and the lens rim are the status display, and the user's proof of when she is reading location or the viewport. A fixed frame surrounds the map (section 4). The sheet above the composer holds one conversation, the *thread*, which tracks side chats inline as *strands*. The backend has the durable parts (event log, Tasks, approvals, Cards, memory). It has no notion yet of a viewport, a place, a strand, or how loudly an agent-initiated event may speak (section 12).

## 2. Principles

| Principle | Consequence |
|---|---|
| The map is the product; Mira is a guest on it. | Nothing she draws blocks panning. She moves the camera only after a user action, and a whisper never moves it. |
| Every query goes to Mira. | The composer is the only input, so a plain place-name search must still feel instant. |
| The frame never moves. | Each edge has one job at every sheet height. |
| One surface, two views. | Map and sheet share one selection and one subject. |
| The smallest form that answers. | She starts small and the user opens it up. |
| The glow never lies. | If she reads location or the viewport, she is visibly Noticing or Working. |
| Attention is a budget. | Viewport noticing stops at a whisper. Only approvals persist until answered. |
| Consequential actions need the user's yes. | Booking, paying and sending go through an approval whose text is the exact consent. |

## 3. The figure

### 3.1 Who she is

**Mira is a map pin that came alive.** A pin is a teardrop pointing down, stuck to one place. Turn it over and let it float, and it becomes a small flame of light that travels with you. A pin marks where you are going; Mira goes with you.

The name is the star Mira, the first star people saw brighten and fade, and the Spanish "¡mira!", "look!". Her character is warm, curious and calm. If nothing is worth saying she says nothing, and the figure shows that by resting.

### 3.2 Anatomy

| Part | What it is | What it expresses |
|---|---|---|
| Belly | A round body of honey light (`#FFB627` to ember `#FF7A3D`) | Her presence. It is always visible |
| Tip | The curled point on top, the old pin point | Mood and attention. It sways, perks and flops |
| Face | Two ink dot eyes set low, a blush | Where she is looking. The eyes follow the pan |
| Halo | A soft radial glow behind her | How much she is doing |
| Motes | Up to three small lights in orbit | One per running Task. The same motes mark each strand in the thread (section 8) |
| Ring | A steady blue circle | It is the user's turn. Blue is the user's colour everywhere |
| Lens | The 76 px round glass button that frames her | That she can be tapped. Its rim and glow repeat her state |

She ships inside the lens: a 76 px glass button, top-centre, 54 px from the top edge, with a body of about 44 px. The lens never moves, at any sheet height. Tapping it opens Mira's drawer (section 4). Outside the lens she scales from 24 px (notification row) to 240 px (brand).

We frame her because the bare v1 figure did not read as a button, and because her halo lost contrast over parks and water. Glass gives her the same backdrop on every map.

### 3.3 Signature motion per state

Each state shows twice: in her body, and on the lens rim and glow. The rim makes a state readable in the corner of the eye, without resolving a 44 px face.

| State | Motion | Lens rim and glow |
|---|---|---|
| Resting | Slow float and halo breath on a 4.8 s cycle, an occasional blink | Soft glow, no rim |
| Noticing | Body and face lean toward the pan; a sheen sweeps the belly as the scan | Thin honey rim |
| Working | Halo pulses on a 1.8 s beat, one mote orbits per running Task | Rim pulses on the same beat |
| Has-something | A ring of short rays blooms once, and the whisper unfurls under the lens | Bright rim |
| Listening | Body widens; two rings track voice amplitude | Rings swell to the rim |
| Composing | **The flip.** She spins tip-down inside the lens, a pin again for a moment, and pours drops of light onto the map | Bright rim |
| Needs-you | The blue ring appears and holds. She is almost still. Nothing pulses | The ring sits on the rim |
| Delivered | One hop with smiling eyes and three sparkles | Bright, then soft |
| Dimmed | Grey, sitting lower, eyes closed, tip flopped over | No glow |

The flip ties the figure to the map: every mark she places arrives as light from her tip, so the user sees where an answer came from.

### 3.4 Motion principles

1. **Light before motion.** A state must be readable from the lens rim and glow alone, which is also what the reduced-motion version shows.
2. **One signal at a time.** She shows one state, chosen by the precedence in 3.6. Only the ring and the motes persist under other states.
3. **She never bounces for attention.** Nothing loops a hop or shows a badge count.

### 3.5 Colour

Honey is Mira: her body, her marks on the map, her messages. Blue (`#2F6BFF`) is the user: the location dot, the user's messages, and anything waiting on the user's decision. There are no other accents, and neither colour is the only signal for anything (section 11).

### 3.6 States and signals

| State | Trigger | What the user can do | Backing API signal |
|---|---|---|---|
| Resting | Nothing running, nothing unread | Ask in the composer; tap the lens for her drawer | No agent `TypingPayload`; no `RUNNING` Task |
| Noticing | Camera settles and the viewport is sent to her | Keep panning (cancels); ask about this view | Client-local, tied to sending the viewport. **Gap:** no viewport signal |
| Working | One or more Tasks are `RUNNING` | Tap the lens to see what is running and why | `TaskUpdatePayload`, `WatchTask`; one mote per running Task |
| Has-something | She has a line worth saying | Tap the whisper, swipe it away, or ignore it | Agent `MessagePayload`. **Gap:** no attention level |
| Listening | User holds the mic in the composer | Release to send; slide away to cancel | Client-local. **Gap:** live media seam has no proto |
| Composing | She is answering or building UI | Wait, or interrupt with a new message | Agent `TypingPayload{active, activity_hint}` until the message lands |
| Needs-you | An approval is open | Tap the blue strand chip, read the consent text, approve or deny | Unresolved `ApprovalRequestedPayload`; Task `WAITING_APPROVAL` |
| Delivered | A Task completes | Tap the strand's capsule to open the result | `TaskUpdatePayload{COMPLETED}` |
| Dimmed | Offline, paused, or location off | Tap the lens to resume or see what is off | Client-local. **Gap:** no server-side pause |

Two decisions follow. A Task in `WAITING_EXTERNAL` (a table watch that waits three days) does not pulse: Working means compute is happening now, so the watch is a waiting mote on a Resting figure. A `FAILED` Task has no state of its own: she shows Has-something with a plain whisper, no bloom, and a drooped tip.

When several states apply, the precedence is Dimmed, Listening, Needs-you, Composing, Has-something, Noticing, Working, Resting. The Needs-you ring stays visible under any state shown above it.

## 4. The frame: four edges

The screen is the base map, the *map marks* Mira draws on it (lit areas, labelled pins, routes), and a frame whose four edges each have one job. The frame does not move when the sheet does. v1 put everything behind the figure, which kept the map clear but gave saved places and settings no visible home.

| Edge | Element | Job |
|---|---|---|
| Top-left | Menu | The app's content: saved places, lists, trips, layers, offline maps |
| Top-centre | The lens | Opens *Mira's drawer*, a panel that descends from under the frame |
| Top-right | App settings | Units, navigation preferences, notifications, privacy and location, account |
| Under the lens | The *whisper slot* | Her one-line whisper or a short status label, and nothing else |
| Right, above the sheet | Map controls | Recentre; in navigation, mute and overview |
| Bottom | The composer | Always the last thing in the sheet |

Mira's drawer holds three groups: *Now* (what she is doing and which inputs she is reading, with reasons), *Ongoing* (live strands) and *From Mira* (unread things she started). Below them sit a searchable History and the agent settings: proactivity level (Quiet, Balanced or Chatty), what she can see (location, this map view), memory, voice, and Pause Mira.

**App settings and agent settings are separate on purpose.** App settings change how the map behaves with or without Mira: kilometres or miles, avoiding stairs. Agent settings change what Mira may see, what she remembers and how often she speaks. They sit behind the lens because a user who wants her quieter looks at her first. Location is on both sides: app settings hold the operating system permission, and the drawer holds whether Mira reads it.

The sheet has four heights. Content always sits above the composer, which never leaves the bottom edge.

| Height | Size on a 390 x 844 screen | Above the composer |
|---|---|---|
| Rest | 112 px | Nothing |
| Peek | 200 to 230 px | One answer, a card rail, or a route strip |
| Half | About 430 px | The thread, a result list, or a place brief |
| Full | Sheet top at y = 150 | The thread, a focused strand, or a full canvas. The frame stays visible above |

```
 Rest                                   Peek, with a card rail
+----------------------------+        +----------------------------+
| (menu)    ((Mira))   (app) |        | (menu)    ((Mira))   (app) |
|                            |        |   [ 4 lunch spots here ]   |
|      o                     |        |   o        (o) <- lifted   |
|               (you)        |        |        o          o    (>) |
|                        (>) |        +============================+
+============================+        | [card ][card ][card ][ca   |
| [+  Ask Mira        (mic)] |        | [+  Ask Mira        (mic)] |
+----------------------------+        +----------------------------+
```

The composer is the thumb's entry point: tap to type, hold the mic to talk. The lens is a status display first and is tapped rarely, so its position costs little reach. Assumption: 54 px clears a camera cutout.

## 5. Inputs

| Input | What it means | What Mira does | Privacy note |
|---|---|---|---|
| Passive location | "I am here, moving this way" | Checks saved places and live strands against it. Stops at a whisper unless a strand asked for alerts. | Needs the OS permission and the drawer's switch. Not kept as a trail (assumption). |
| Viewport | The deictic "here" | On camera settle (assumption: about 600 ms idle) the client sends bounds and zoom. She shows Noticing and may whisper. | Sent only while she shows Noticing, and only if "this map view" is on. |
| Tap on the lens | "What are you doing?" | Opens Mira's drawer. | This is the privacy control surface. |
| Tap on a mark | Select it | Mark lifts with a callout; the matching list row scrolls into view. | None |
| Tap on a place on the map | "What is this?" | Composes a *place brief* at Half. | The place is sent only on the tap. |
| Drag a mark or map point into the composer | "About this" | A context chip appears; the next message is about that place. | Sent only with the message. |
| Text | Explicit request | Composing, then the smallest form that answers. | Persisted with a snapshot of viewport and selection. |
| Voice | Text, with hands busy | Listening while the mic is held. | v1 assumption: transcribed on the device, so audio stays there. |

Scope resolves by rule: "here" and unqualified requests mean the viewport, "near me" means the device location, and a context chip overrides both. Every answer names its scope ("in this view").

## 6. Outputs and generative UI

| Form | When | Example | Wire mapping |
|---|---|---|---|
| Whisper | One fact or one offer | "4 lunch spots in this view" | One-line `MessagePayload.markdown`. **Gap:** attention level |
| Map marks | The answer is where things are | A lit block, six pins | `Card{kind: "map_marks"}`, geometry in `payload_json`. **Gap:** placement hint |
| Card rail | 3 to 8 comparable options, at Peek | Lunch cards above the composer | `Card{kind: "options_list"}`, placement rail |
| Route strip | Directions in progress, at Peek | "Left on Kiln Street, 120 m" | **Gap:** no route Card kind |
| Sheet list | More than a rail holds, at Half | Twelve restaurants, sortable | `Card{kind: "options_list"}`, placement sheet |
| Place brief | The user tapped a place, at Half | What The Dye Works is, its hours | **Gap:** no place Card kind |
| Full canvas | A structured plan, at Full | Weekend itinerary by day | `Card{kind: "itinerary"}`. Static in v1; live editing needs the reserved WorkspaceService |
| Off-app | Time-critical, or navigating | Push, lock-screen live activity | Push from server-side rules over the log. **Gap:** no priority field |

Cards are a closed, versioned vocabulary, and we keep that: Mira composes the content and picks the kind and size, not the layout. A map-placed Card's `fallback_markdown` must read as a list, because a screen reader and an older client get exactly that.

## 7. Map and sheet choreography

**Camera padding.** The visible region is the screen minus a top inset of 150 px (the frame) and a bottom inset equal to the sheet height. Padding follows the sheet drag, so the subject stays centred in what is visible. At Full the camera freezes.

**Shared selection.** There is one selected item. Tapping a mark, a rail card or a list row sets it: the mark lifts, the row highlights, and the camera moves the minimum distance needed to show the mark. Scrolling a list never moves the camera.

**Place references.** Every place named in the thread is tappable and flies the camera there. At Full, the sheet drops to Half first.

**Browse mode.** While a card rail or result list is open, the query stands and follows the map. On a pan the list re-sorts at once by distance to the new centre, using results it already has. On settle, Mira shows Noticing and fetches for the new viewport. New results arrive under a "3 new in this view" row and never replace rows under the user's finger. The selected item stays pinned. Browse mode ends when the user dismisses the rail or asks something else.

## 8. One thread, many strands

The sheet holds one continuous conversation with Mira, the thread, newest at the bottom. v1 sorted work into Now, Ongoing and From Mira on a separate screen. That kept long work findable, but the user had to leave the conversation to reach it. v2 tracks work where it started.

A *strand* is a side chat inside the thread: one piece of work the user can return to.

- **Capsule.** A strand appears in the thread as one compact row at the point it started: a mote (running, waiting or done), a title, its latest line, and a count or state. It updates in place, so a three-day table watch adds no rows to the thread. Tapping it focuses the strand.
- **Focused strand.** The sheet shows only that strand's messages under a back control and its title. The composer turns honey and carries a *scope chip* naming the strand, so the user sees where a message will go. Leaving returns to the same scroll position.
- **Strand rail.** Chips directly above the composer list the two or three live strands. One that needs a decision is blue.
- **Fold.** Quick answers that have faded collapse to one centred line ("3 quick answers · Alder Quay · 12:40"). Tapping unfolds it.
- **Stamp.** A place and time separator ("Alder Quay · today 12:38"), so the thread is organised by where the user was as well as when.

| Journey | In the thread | Sheet height |
|---|---|---|
| Ephemeral question and answer | Two bubbles, later part of a fold | Peek |
| Directions | A capsule ("Walking to Kiln & Rye") | Peek, showing the route strip |
| Trip planning | A capsule that opens the full canvas | Full |
| Long-horizon task | A capsule with steps; it can wait for days | Rest while it waits; Half when focused |
| Proactive | A whisper; once opened, a message tagged "From Mira" | Rest, then Peek |
| Discovery browsing | One entry for the standing query, not one per pan | Peek, in browse mode |
| Getting to know a place | A place brief | Half |

**Lifecycle.** An ephemeral exchange fades when the user moves on (assumption: the viewport no longer overlaps and 30 minutes pass), and faded neighbours merge into a fold. A fold hides rows and deletes nothing: the messages stay in history and search until the user forgets them (`ForgetSegment`). A strand starts when Mira creates a Task or the user taps Keep. It ends when its Tasks are terminal and the result is seen, or the user ends it. Its capsule stays, with a done mote.

**Keep.** Every answer and card has a Keep action, which turns the exchange into a strand so it does not fold. If the intent is unclear Mira asks once ("Keep as a list, or watch for a table?").

Now, Ongoing and From Mira survive only as groups in Mira's drawer. They are views over the thread, so nothing lives only in the drawer. An item leaves From Mira when the user replies to it, keeps it or dismisses it, or when it expires.

The API ships exactly one `KIND_MAIN` conversation today, which fits the thread. Whether a strand is a `KIND_SIDE` conversation or a projection over the main one is question 1.

## 9. Proactivity and attention budget

| Rung | Cost to the user | Allowed when |
|---|---|---|
| Glow only | None | Always |
| Whisper | A glance. Dismisses itself (assumption: about 6 seconds) | She has something specific and current |
| Message in the thread | A "From Mira" tag in the thread and an unread row in her drawer | The item belongs to a strand, or the user opened the whisper |
| Push notification | An interruption | Time-critical, or the user asked for alerts on that strand |

The proactivity level in Mira's drawer sets how far up the ladder she climbs unasked. Quiet stops at glow, plus approvals. Balanced allows whispers. Chatty lowers her bar for a whisper. The thresholds are unmeasured.

Hard rules, at every level:

1. Viewport noticing never escalates past a whisper on its own.
2. Only Needs-you persists until answered. Every other rung expires.
3. One whisper shows at a time. A new one replaces the old one.
4. A dismissed whisper is not repeated on the same topic in the same session.
5. No whisper appears while the user is typing, speaking or dragging the map.
6. A push states its reason ("You asked me to watch this").
7. She never raises the sheet unasked. A whisper stays in its slot until tapped.

## 10. Scenarios

**(a) Lunch nearby**

1. Resting, Rest. The user taps the composer and types "anything good for lunch?".
2. Composing. Light falls from the lens to six map marks. The sheet rises to Peek with a card rail, and the whisper slot reads "Lunch, in this view".
3. The user drags to Half. The rail becomes a list and the camera re-pads to keep the marks in view.
4. The user pans east. Browse mode re-sorts the list; on settle, Noticing, then "2 new in this view".
5. The user taps a row and its mark lifts. Unless kept, the exchange later joins a fold.

**(b) Wandering**

1. The user pans across an unfamiliar neighbourhood. Noticing on each settle; most yield nothing.
2. Has-something. Whisper: "This street has three old bookshops." The user ignores it, it fades, and nothing lands in the thread.
3. Two pans later she whispers about a covered market, and the user taps it.
4. Composing, Peek. Three marks appear, with one answer above the composer tagged "From Mira".
5. The user taps Keep. The market becomes a saved place under the menu, and the exchange becomes a strand.

**(c) Weekend trip**

1. The user holds the mic and says "plan a weekend in Porto next month". Listening, then Composing.
2. Mira creates a Task. Working, one mote. A capsule, "Weekend in Porto", appears in the thread. The Task keeps running after the user closes the app.
3. Next day: Delivered on open. Tapping the capsule focuses the strand at Full, showing the itinerary canvas.
4. With the composer scoped to the strand, the user asks her to book the guesthouse. Working, then Needs-you. A push goes out because the hold has an `expires_at`.
5. The strand's chip is blue. The user reads the exact text ("Book Casa do Rio, 2 nights, EUR 214, free cancellation to 4 Dec") and approves. The client calls `ResolveApproval`.
6. Delivered. A receipt Card joins the strand and the capsule's latest line updates.
7. The restaurant has no table, so that Task is `WAITING_EXTERNAL` and its capsule shows a waiting mote.

**(d) Proactive nudge while walking**

1. The user is walking. A location update puts a saved bakery two minutes off the route, closing in an hour.
2. Has-something. Whisper: "Padaria Lume is 2 min off your route, closes at 6."
3. The user taps the whisper. Composing: the detour draws as a route mark with the added time.
4. The user taps "Add stop". The route updates, and the message joins the walk's strand and leaves From Mira.
5. With the phone locked, the nudge is a push only if the user asked for alerts on saved places. Otherwise it waits in From Mira.

**(e) Walking directions, with a question on the way**

1. The user asks for a walking route to Kiln & Rye. The route draws, a capsule "Walking to Kiln & Rye" appears, and the sheet sits at Peek with the route strip.
2. Mid-walk the user holds the mic: "is there a cash machine on the way?". The composer is not scoped, so the question joins the thread, not the strand.
3. Composing. One mark lands beside the route and a one-line answer sits above the route strip. The route does not change.
4. On arrival the capsule's mote turns done.

**(f) Getting to know a place**

1. The user taps The Dye Works on the map.
2. Composing, Half. Mira composes a place brief: what it is, when it is open, and why it suits this user.
3. The composer carries a context chip for The Dye Works, so "is it busy on Sundays?" needs no place name.
4. Unkept, the brief later joins a fold under the Alder Quay stamp.

## 11. Trust, privacy, accessibility

**Honesty of the glow.** The client sends a viewport or location signal only while it renders Noticing or Working. The Now group in her drawer lists each input in use with its reason ("Reading this view because you asked about lunch"). This is a client contract; nothing on the server enforces it.

**Pause.** Pause Mira, in her drawer, sets Dimmed. The client then sends no viewport or location, and proactive work stops. Tasks the user started keep running, and the drawer says so.

**Memory.** The Memory entry in Mira's drawer lists PROFILE memories (editable, deletable) and STATE memories (open commitments, read-only, each linked to its strand). "Forget this" on a strand or a fold calls `ForgetSegment`. Assumption to confirm: Mira stores places the user saved and facts the user stated, and no movement trail.

**Accessibility.**

- Each state has a spoken name, announced on change ("Mira, working on 2 tasks"). Noticing is not announced, because it fires on every pan.
- Colour never carries state alone. Each state differs in shape or motion (ring for Needs-you, motes for Working, closed eyes for Dimmed). The scoped composer changes colour and also shows the scope chip.
- With reduced motion on, pulses and hops become static changes (rim, ring, mote count) and camera flights become cuts.
- Every map-placed Card is also a list in the sheet, so marks are reachable without the map.
- With a screen reader on, whispers stay until dismissed.
- Hold-to-talk has a tap-to-toggle alternative. The lens is a 76 px target and the other frame buttons are 48 px. Strand chips are 34 px tall, so their hit area extends to 44 pt.

## 12. API mapping and gaps

| UX need | Existing `mira.v1` primitive | Gap |
|---|---|---|
| Durable thread across devices | Event log, `SyncEvents`, `StreamEvents` | None |
| "She's answering", with a reason | `TypingPayload.activity_hint` | None |
| Background work, motes | `Task`, `TaskStep`, `ListTasks`, `WatchTask` | No recurring or watch Task kind; no geo anchor |
| Approvals | `ApprovalRequestedPayload`, `ResolveApproval` | None |
| Structured results | `Card{kind, fallback_markdown, payload_json}` | No placement hint; map, route and place kinds undefined |
| Viewport and location as input | `ClientSignal` on `StreamEvents`, `PublishSignal` | No context signal (bounds, zoom, location, heading, selection); no durable context snapshot on `AppendEvent` |
| Browse mode | None | A standing viewport query: one query that re-runs as bounds change. Today each pan would be a new message |
| Whisper-only and push-worthy events | Push from server-side rules | No attention level, expiry or initiator on agent events |
| Viewport whispers that leave no trace | Ephemeral events (typing, presence) | No ephemeral agent message; no persist-on-tap |
| Tappable places, "about this" | `MessagePayload.markdown`, `AttachmentRef` (blobs only) | No place reference type |
| Strands and capsules | `Conversation.Kind.KIND_SIDE` (reserved), `segment_id` | Side conversation or projection is open (question 1). Either way a capsule needs a stable strand id, a title, a state and the latest line. Keep has no write path, since clients cannot create Tasks |
| Scoped composer | `AppendEvent` | No field naming the conversation or strand an event targets |
| Folds | `segment_id` on ephemeral segments | None if folds are a client-side projection, which we propose |
| Agent settings | `MemoryService` for memory | No server-side preference for proactivity level |
| Voice | Reserved live media seam | No proto |
| Live itinerary editing | Reserved WorkspaceService seam | No proto; selection is client-local in v1 |
| Editable memory | `MemoryService` PROFILE and STATE, `ForgetSegment` | Memories are free text; no saved-place structure |
| Pause | `PresencePayload` | No server-side pause of proactive work |
| Nudges while backgrounded | Push | No path for background location to reach the agent |

## 13. Open questions

1. **Are strands side Conversations or a client projection?** Side Conversations give each strand its own history and a target for the scoped composer, but reverse the backend's v1 lean. A projection over `segment_id` and Tasks ships sooner but has nowhere to store the id and title of a kept list with no Task.
2. **Which proactivity level is the default, and is "this map view" on at first run?** Balanced with the view on makes wandering work from day one. Quiet makes the first Noticing an explicit choice.
3. **Are unopened whispers persisted?** We propose that an ignored viewport whisper leaves nothing and an opened one lands in the thread. That needs an ephemeral agent message and a persist-on-tap path.
4. **What does a fold promise?** Hiding rows (proposed) or deletion from history. Deletion contradicts the append-only log and needs a retention rule.
5. **Where are background location triggers evaluated?** On the device (private, capped by OS quotas) or on the server (uploads location while backgrounded). This decides whether scenario (d) works with the phone locked.
6. **Who enforces the attention budget?** The server holds the proactivity level and all pending items; the client knows whether the user is mid-gesture. A split needs a rule for which side may downgrade.
7. **Is a drooped Has-something enough for `FAILED`?** If testing shows people miss failures, the vocabulary needs a tenth state.
8. **Does a plain place-name search go through the agent?** If her first response to "Lisbon airport" is slower than a geocoder, the client needs a local fast path that still writes to the log.
9. **May Mira re-route a message the scope chip sent to the wrong place?** A user focused on the Porto strand may ask about lunch. If the server may move it to the thread, the target on `AppendEvent` is a hint; if not, it is binding.
