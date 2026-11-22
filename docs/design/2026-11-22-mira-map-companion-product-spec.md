# Mira, the map companion: product spec

Status: draft for review, 2026-11-22. v2: navigation frame, lens, one thread with strands. UI mocks and the character sheets: https://claude.ai/artifact/K31Jt8FNidngqVZU2gUf3Z
Backend references: `docs/api-outline.md`, `proto/mira/v1/*.proto`.

## 1. Summary

Mira is an AI agent that lives on a mobile map as a small glowing figure in a round glass button, the *lens*, at the top-centre of the screen. The map is full-bleed with no search bar: the input box at the bottom edge, the *bar*, is how the user asks for anything. She reads where the user is, what the map shows, and what the user types or says, and she answers in the smallest form that works, from one line under the lens to a full itinerary. Her glow and the lens rim are the status display, and they are also the user's proof of when she is reading location or the viewport. The screen has a fixed frame of four edges: menu, lens and app settings along the top, map controls on the right, the bar at the bottom. The sheet above the bar holds one continuous conversation, the *thread*, in which side chats are tracked inline as *strands*. The backend already has the durable parts (event log, Tasks, approvals, Cards, memory); it has no notion yet of a viewport, a place, a strand, or how loudly an agent-initiated event may speak (section 12).

## 2. Principles

| Principle | Consequence |
|---|---|
| The map is the product; Mira is a guest on it. | Nothing she draws blocks panning, and she moves the camera only when the user asked. |
| Every query goes to Mira. | The bar is the only input, so a plain place-name search must still feel instant. |
| The frame never moves. | Each edge has one job at every sheet height, so the user learns it once. |
| One surface, two views. | Map and sheet share one selection and one subject. |
| The smallest form that answers. | She starts small and the user opens it up. She never raises the sheet past Peek herself. |
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
| Lens | The 76 px round glass button that frames her | That she can be tapped. Its rim and outer glow repeat her state (section 3.3) |
| Motes | Up to three small lights in orbit | One per running Task. The same motes mark each strand's capsule in the thread (section 8) |
| Ring | A steady blue circle | It is the user's turn. Blue is the user's colour everywhere in the app |

She ships inside the *lens*: a 76 px glass button, top-centre, 54 px from the top edge. Her body is about 44 px inside it. The lens never moves, at any sheet height. Tapping it opens Mira's drawer (section 4). Outside the lens she scales to 24 px in a notification row, 120 px for voice and 240 px for brand use.

We frame her because the bare figure in v1 did not read as a button, and because her halo lost contrast over parks and water. The glass gives her the same backdrop on every map, light or dusk.

### 3.3 Signature motion per state

Each state shows twice: in her body, and on the lens rim and glow. The rim is the second channel, so a state is readable in the corner of the eye without resolving a 44 px face.

| State | Motion | Lens rim and glow |
|---|---|---|
| Resting | Slow float and halo breath on a 4.8 s cycle, a lazy tip sway, a blink about every 5 s | Soft glow, no rim |
| Noticing | Body leans and the face slides toward the pan, the tip trails behind, a sheen sweeps across the belly as the scan | Thin honey rim |
| Working | Halo pulses on a 1.8 s beat, the tip ticks in time, one mote orbits per running Task | Rim pulses on the same beat |
| Has-something | Brightest halo, a ring of short rays blooms once, a quick squash and perk, and the whisper unfurls in the slot under the lens | Bright rim |
| Listening | Body widens, the tip cocks to one side, and two rings track voice amplitude | Rings swell to the rim |
| Composing | **The flip.** She spins tip-down inside the lens and is a pin again for a moment, pouring drops of light below it onto the map, then pops back upright | Bright rim |
| Needs-you | The blue ring appears and holds. She is almost still and looks straight out. Nothing pulses | The ring sits on the rim, blue |
| Delivered | Squash, hop, land, with smiling eyes and three small sparkles, played once | Bright rim, then back to soft |
| Dimmed | Grey, sitting lower, eyes closed, tip flopped over | No glow |

The flip is the gesture that ties the figure to the map. Every mark she places arrives along a thread of light from her tip, so the user sees where an answer came from and that it belongs to her.

### 3.4 Motion principles

1. **Light before motion.** Brightness is the first signal and movement the second. A state must be readable from the lens rim and glow alone, which is also what the reduced-motion version shows.
2. **One signal at a time.** She shows one state, chosen by the precedence below. Only the ring and the motes persist under other states.
3. **She never bounces for attention.** No state loops a hop, a shake or a badge count. Delivered and Has-something play once and settle.
4. **Slower than the map.** Her resting rhythms are slower than a pan, so she reads as calm next to the thing the user is moving.
5. **Honest glow.** If she is reading the viewport or the location, she is visibly Noticing or Working. There is no silent mode in which she looks idle and is not.

### 3.5 Colour

Honey is Mira: her body, her marks on the map, her messages, and place references she makes. Blue (`#2F6BFF`) is the user: the location dot, the user's messages, and anything waiting on the user's decision. No other accent colours are used, and neither colour is the only signal for anything (see section 11).

### 3.6 Directions considered

Three figures were built and compared as animated studies at 240, 120 and 52 px on light and dusk maps. The studies predate the lens, so 52 px was the unframed ship size then.

| Direction | What it gave | Why it was or was not chosen |
|---|---|---|
| A. Star-sprite, a plump star with rays | Warm, and the rays can form a compass | Reads as a familiar game star and sits close to the generic AI sparkle |
| **B. The pin that came alive (chosen)** | One silhouette that is clear at 52 px, a tip that acts, and the flip | The only direction with a reason to live on a map |
| C. Lantern creature, a round sprite with fins and a glowing belly | The most character and charm at large sizes | Too many parts at 52 px, and the glow is only a third of her body |

### 3.7 States and signals

| State | Trigger | What the user can do | Backing API signal |
|---|---|---|---|
| Resting | Nothing running, nothing unread | Ask in the bar; tap the lens for her drawer | No agent `TypingPayload`; no `RUNNING` Task |
| Noticing | Camera settles and the viewport is sent to her | Keep panning (cancels); tap to ask about this view | Client-local, tied to sending the viewport. **Gap:** no viewport signal |
| Working | One or more Tasks are `RUNNING` | Tap the lens to see the running strands and why | `TaskUpdatePayload`, `WatchTask`; one mote per running Task |
| Has-something | She has a line worth saying | Tap the whisper to open it up, swipe it away, or ignore it | Agent `MessagePayload`. **Gap:** no attention level |
| Listening | User holds the mic in the composer | Speak; release to send; slide away to cancel | Client-local. **Gap:** live media seam has no proto |
| Composing | She is answering or building UI | Wait, or interrupt with a new message | Agent `TypingPayload{active, activity_hint}` until the message lands |
| Needs-you | An approval is open | Tap the blue strand chip, read the consent text, approve or deny | Unresolved `ApprovalRequestedPayload`; Task `WAITING_APPROVAL` |
| Delivered | A Task completes | Tap the strand's capsule to open the result | `TaskUpdatePayload{COMPLETED}` |
| Dimmed | Offline, paused, or location off | Tap the lens to resume or see what is off | Client-local. **Gap:** no server-side pause |

The table forces two decisions. A Task in `WAITING_EXTERNAL` (a table watch that waits three days) does not pulse: Working means compute is happening now, so the watch is a parked mote on a Resting figure. A `FAILED` Task has no state of its own: she shows Has-something with a plain whisper, without the bloom or the hop, and the tip drooped.

When several states apply, the proposed precedence is Dimmed, Listening, Needs-you, Composing, Has-something, Noticing, Working, Resting. The Needs-you ring stays visible under any state shown above it.

## 4. The frame: four edges

The screen is the base map, the *map marks* Mira draws on it (lit areas, labelled pins, routes, annotations), and a frame of four edges. Each edge has one job, and the frame does not move when the sheet does. v1 put everything behind the figure and a floating pill, which kept the map clear but hid where saved places and settings lived.

| Edge | Element | Job |
|---|---|---|
| Top-left | Menu, a 48 px glass button | The app's content: saved places, lists, trips, map layers, offline maps |
| Top-centre | The lens | Opens *Mira's drawer*, a panel that descends from under the frame |
| Top-right | App settings, a 48 px glass button | Units, navigation preferences, notifications, privacy and location, account |
| Under the lens | The *whisper slot* | Mira's one-line whisper or a short status label. Nothing else goes there |
| Right edge, above the sheet | Map controls | Recentre; in navigation, mute and overview |
| Bottom | The bar | The composer, always the last thing in the sheet |

Mira's drawer holds four groups. *Now* lists what she is doing at this moment and which inputs she is reading, with the reason for each. *Ongoing* lists live strands with their motes. *From Mira* lists unread things she started. Below them sit History (earlier conversations, searchable) and the agent settings: proactivity level (Quiet, Balanced or Chatty), what she can see (location, this map view), memory, voice, and Pause Mira.

**App settings and agent settings are separate on purpose.** App settings change how the map behaves whether or not Mira is involved: kilometres or miles, avoiding stairs. Agent settings change what Mira may see, what she remembers and how often she speaks. They sit behind her face because a user who wants her quieter looks at her, not through a general settings list. Location is the one item on both sides: app settings hold the operating system permission, and the drawer holds whether Mira reads it.

The sheet is one object with four heights. Content always sits above the composer, and the composer never leaves the bottom edge.

| Height | Size on a 390 x 844 screen | Above the composer |
|---|---|---|
| Rest | 112 px | Nothing: the grabber and the composer |
| Peek | 200 to 230 px | One answer, a card rail, or a route strip |
| Half | About 430 px | The thread, a result list, or a place brief. The map stays live |
| Full | Sheet top at y = 150 | The thread, a focused strand, or a full canvas. The frame shows on the strip of map above |

```
 Rest                                   Peek, with a card rail
+----------------------------+        +----------------------------+
| (=)       ((Mira))     (%) |        | (=)       ((Mira))     (%) |
|                            |        |   [ 4 lunch spots here ]   |
|      o                     |        |                            |
|               (you)        |        |   o        (o) <- lifted   |
|                            |        |        o          o    (>) |
|                        (>) |        +============================+
+============================+        | [card ][card ][card ][ca   |
| [+  Ask Mira        (mic)] |        | [+  Ask Mira        (mic)] |
+----------------------------+        +----------------------------+
 (=) menu   (%) app settings   (>) recentre   ((Mira)) the lens
```

The composer is the thumb's entry point: tap to type, hold the mic to talk, plus to attach a place or photo. The lens is at the top because it is a status display first and is tapped rarely; asking never needs the reach. Assumption: 54 px clears a camera cutout; the offset is per-device layout work.

## 5. Inputs

| Input | What it means | What Mira does | Privacy note |
|---|---|---|---|
| Passive location | "I am here, moving this way" | Checks saved places and live strands against it. Stops at a whisper unless a strand asked for alerts. | Needs the OS permission and the drawer's location switch. Used for the check, not kept as a trail (assumption). |
| Viewport | The deictic "here" | On camera settle (assumption: about 600 ms idle) the client sends bounds and zoom. She shows Noticing and may whisper. | Sent only while she shows Noticing, and only if "this map view" is on in the drawer. |
| Tap on the composer | "I want to ask" | Keyboard up; the sheet stays at its height. | None |
| Tap on the lens | "What are you doing?" | Opens Mira's drawer. | This is the privacy control surface. |
| Tap on a whisper | "Tell me more" | Opens the whisper up: marks on the map and one answer at Peek. | None |
| Tap on a mark | Select it | Mark lifts, callout opens, the matching list row scrolls into view. | None |
| Tap on a place on the map | "What is this?" | Composes a *place brief* at Half (scenario f). | The place is sent only on the tap. |
| Drag or long-press a mark or map point into the composer | "About this" | A context chip appears; the next message is about that place. | Sent only with the message. |
| Text | Explicit request | Composing, then the smallest form that answers. | Persisted in the event log with a snapshot of viewport and selection. |
| Voice | Text, with hands busy | Listening while the mic is held. | v1 assumption: transcribed on the device, so audio stays there. |

Scope resolves by rule: "here" and unqualified requests mean the viewport, "near me" means the device location, and a context chip overrides both. Every answer names its scope ("in this view"), so a wrong guess is one tap to fix. Which conversation a message joins is a separate matter, shown by the scope chip (section 8).

## 6. Outputs and generative UI

Each form has an "open it up" gesture that moves to the next row.

| Form | When | Example | Wire mapping |
|---|---|---|---|
| Whisper | One fact or one offer, in the whisper slot | "4 lunch spots in this view" | One-line `MessagePayload.markdown`. **Gap:** attention level |
| Map marks | The answer is where things are | A lit block, six pins | `Card{kind: "map_marks"}`, geometry in `payload_json`. **Gap:** placement hint |
| Mark callout | Detail on the selected mark | Name, closing time, walk time | Same Card, one item expanded by the client |
| Card rail | 3 to 8 comparable options at Peek | Lunch cards above the composer | `Card{kind: "options_list"}`, placement rail |
| Route strip | Directions in progress, at Peek | "Left on Kiln Street, 120 m" | **Gap:** no route Card kind |
| Sheet list or comparison | More than a rail holds, at Half | Twelve restaurants, sortable | `Card{kind: "options_list"}`, placement sheet |
| Place brief | The user tapped a place, at Half | What The Dye Works is, hours, why it suits this user | **Gap:** no place Card kind |
| Full canvas | A structured plan, at Full | Weekend itinerary by day | `Card{kind: "itinerary"}`. Static in v1; live editing needs the reserved WorkspaceService |
| Off-app | Time-critical, or navigating | Push, lock-screen live activity | Push from server-side rules over the log. **Gap:** no priority field |

The backend defines Cards as a closed, versioned vocabulary, and we keep that. "Conjured on the fly" means Mira composes the content and picks the kind and size; it does not mean arbitrary layout. A map-placed Card's `fallback_markdown` must read correctly as a list, because a screen reader and an older client get exactly that.

## 7. Map and sheet choreography

**Camera padding.** The visible region is the screen minus a top inset of 150 px (the frame, and the line the Full sheet stops at) and a bottom inset equal to the sheet height. Padding follows the sheet drag frame by frame, so the subject stays centred in what is visible. At Full the camera freezes. Map controls ride the sheet's top edge up to Half.

**Camera ownership.** The user's gestures always win. Mira moves the camera only after a user action (tapping a place reference, a list row, or "show on map"). A proactive whisper never moves it.

**Shared selection.** There is one selected item. Tapping a mark, a rail card or a list row sets it: the mark lifts, the row highlights, and the camera moves the minimum distance needed to show the mark.

**Scroll to lift.** As the list or rail scrolls, the mark for the topmost row lifts. Scrolling does not move the camera, because a list that drags the map around is hard to read.

**Place references.** Every place named in the thread is tappable and flies the camera there. At Full, the sheet drops to Half first.

**Browse mode.** While a card rail or result list is open, it follows the map: the query stands and the viewport changes under it. On a pan the list re-sorts at once by distance to the new centre, using results it already has. On settle, Mira shows Noticing and fetches for the new viewport. New results arrive under a "3 new in this view" row and never replace rows under the user's finger. The selected item stays pinned even if it leaves the viewport, and a hand-sorted list keeps its order. Browse mode ends when the user dismisses the rail or asks something else.

**Pan with nothing open.** Mira shows Noticing and may whisper. Nothing else changes.

## 8. One thread, many strands

The sheet holds one continuous conversation with Mira, the thread, newest at the bottom. v1 sorted work onto three shelves in a "threads home" screen at Full. That kept long work findable, but it made the user file each piece of work and leave the conversation to find it. v2 removes the screen: work is tracked where it started.

A *strand* is a side chat inside the thread: one piece of work the user can return to. Five elements make it work.

- **Capsule.** A strand appears in the thread as one compact row at the point it started: a mote (running, waiting or done), a title, its latest line, and a count or state. The capsule updates in place, so a three-day table watch adds no rows to the thread. Tapping it focuses the strand.
- **Focused strand.** The sheet shows only that strand's messages under a header with a back control ("Thread") and the strand title. The composer turns honey and carries a *scope chip* naming the strand, so the user sees where a message will go before sending it. Leaving returns to the same scroll position in the thread.
- **Strand rail.** A row of chips directly above the composer at Peek, Half and Full lists the two or three live strands for one-tap switching. A strand that needs a decision is blue.
- **Fold.** A run of quick answers that has faded collapses to one centred line ("3 quick answers · Alder Quay · 12:40"). Tapping unfolds it.
- **Stamp.** A place and time separator ("Alder Quay · today 12:38"). The thread is organised by where the user was as well as when.

Each journey type has one home in the thread and one usual sheet height.

| Journey | In the thread | Sheet height |
|---|---|---|
| Ephemeral question and answer | Two bubbles, later part of a fold | Peek for the answer; Half to read back |
| Directions | A capsule ("Walking to Kiln & Rye") | Peek, showing the route strip |
| Trip planning | A capsule that opens the full canvas | Full |
| Long-horizon task | A capsule with steps; it can wait for days | Rest while it waits; Half when focused |
| Proactive | A whisper under the lens; once opened, a message tagged "From Mira" | Rest, then Peek |
| Discovery browsing | One entry for the query; the rail is not logged per pan | Peek, in browse mode |
| Getting to know a place | A place brief | Half |

**Lifecycle.** An ephemeral exchange fades when the user moves on (assumption: the viewport no longer overlaps and 30 minutes pass), and faded neighbours merge into a fold. A fold hides rows and deletes nothing: the log is append-only, so the messages stay in history and search until the user deletes them or calls forget (`ForgetSegment`). A strand starts when Mira creates a Task or the user taps Keep, and ends when its Tasks are terminal and the result is seen, or the user ends it. A finished capsule keeps its place with a done mote. An item leaves From Mira the moment the user replies to it, keeps it or dismisses it, or when it expires.

**Keep.** Every answer and every card has a Keep action, which turns the exchange into a strand so it does not fold. If the intent is unclear Mira asks once ("Keep as a list, or watch for a table?"). A kept list with no Task is still a strand and has a done mote.

Now, Ongoing and From Mira survive only as groupings in Mira's drawer (section 4). They are views over the same strands and messages, so nothing lives in the drawer that is not also in the thread.

One durable main conversation sits under all of it, which matches the API: it ships exactly one `KIND_MAIN` conversation today. Whether a strand is a `KIND_SIDE` conversation or a projection over the main one is question 1.

## 9. Proactivity and attention budget

| Rung | Cost to the user | Allowed when |
|---|---|---|
| Glow only | None | Always |
| Whisper | A glance. Dismisses itself (assumption: about 6 seconds), never blocks the map | She has something specific and current |
| Message in the thread | A "From Mira" tag in the thread and an unread row in her drawer | The item belongs to a strand, or the user opened the whisper |
| Push notification | An interruption | Time-critical, or the user asked for alerts on that strand |

The proactivity level in Mira's drawer sets how far up the ladder she climbs unasked. Quiet stops at glow, plus approvals. Balanced allows whispers. Chatty lowers her bar for what is worth a whisper. The thresholds behind Balanced and Chatty are unmeasured.

Hard rules, at every level:

1. Viewport noticing never escalates past a whisper on its own.
2. Only Needs-you persists until answered. Every other rung expires.
3. One whisper shows at a time. A new one replaces the old one.
4. A dismissed whisper is not repeated on the same topic in the same session.
5. No whisper appears while the user is typing, speaking or dragging the map.
6. A push states its reason ("You asked me to watch this").
7. No level raises the sheet. A whisper stays in its slot until the user taps it.

## 10. Scenarios

**(a) Lunch nearby**

1. Resting, Rest. The user taps the composer and types "anything good for lunch?".
2. Composing, Peek. The message carries the viewport snapshot; the hint reads "looking around this view".
3. Composing. Drops of light fall from the lens to six map marks, then a card rail appears above the composer.
4. Resting, Peek. The label in the whisper slot reads "Lunch, in this view". Swiping the rail lifts each card's mark.
5. The user drags to Half. The rail becomes a sheet list and the camera re-pads to keep all six marks above the sheet.
6. The user pans east. Browse mode re-sorts the list; on settle, Noticing, then "2 new in this view".
7. The user taps a row: the mark lifts with a callout. Unless kept, the exchange later joins a fold.

**(b) Wandering**

1. Resting, Rest. The user pans across an unfamiliar neighbourhood.
2. Noticing on each settle. Most settles yield nothing and she returns to Resting.
3. Has-something. Whisper in the slot: "This street has three old bookshops."
4. The user ignores it. The whisper fades and nothing lands in the thread.
5. Two pans later, Has-something on a covered market. The user taps the whisper.
6. Composing, Peek. A lit area and three marks appear, and one answer about the market sits above the composer, tagged "From Mira".
7. The user taps Keep. The market becomes a saved place (it now appears under the menu) and the exchange becomes a strand with a capsule.

**(c) Weekend trip**

1. Half. The user holds the mic and says "plan a weekend in Porto next month". Listening, then Composing.
2. Mira creates a Task. Working, one mote. A capsule, "Weekend in Porto", appears in the thread with a running mote.
3. The user closes the app. The Task keeps running server-side.
4. Next day: Delivered on open. Tapping the capsule focuses the strand at Full, showing the full-canvas itinerary; at Half the map draws the route between stops.
5. In the strand, with the composer honey and scoped to "Weekend in Porto", the user asks her to book the guesthouse. Working, then Needs-you. A push goes out because the hold has an `expires_at`.
6. The strand's chip in the rail is blue. The user reads the exact text ("Book Casa do Rio, 2 nights, EUR 214, free cancellation to 4 Dec"), then approves. The client calls `ResolveApproval`.
7. Working, then Delivered. A receipt Card joins the strand and the capsule's latest line updates.
8. The restaurant has no table, so that Task is `WAITING_EXTERNAL`: the capsule shows a waiting mote until a table opens.

**(d) Proactive nudge while walking**

1. The user is walking with the phone in hand. Resting, Rest.
2. A location update puts a saved bakery two minutes off the route, closing in an hour.
3. Has-something. Whisper: "Padaria Lume is 2 min off your route, closes at 6."
4. The user taps the whisper. Composing: the detour draws as a route mark, with the added time in a callout.
5. The user taps "Add stop". The route updates, and the message joins the walk's strand and leaves From Mira.
6. With the phone locked, the same nudge is a push only if the user asked for alerts on saved places. Otherwise it waits in From Mira, in her drawer.

**(e) Walking directions, with a question on the way**

1. The user asks for a walking route to Kiln & Rye. Composing; the route draws, and a capsule "Walking to Kiln & Rye" appears.
2. Resting, Peek. The route strip shows the next turn. Map controls add mute and overview.
3. Mid-walk the user holds the mic: "is there a cash machine on the way?". The composer is not scoped, so the question joins the thread, not the strand.
4. Composing. One mark lands beside the route and a one-line answer sits above the route strip. The route does not change.
5. The answer fades into a fold. On arrival the capsule's mote turns done.

**(f) Getting to know a place**

1. Resting, Rest. The user taps The Dye Works on the map.
2. Composing, Half. The place lifts, and Mira composes a place brief: what it is, when it is open, and one line on why it suits this user.
3. The composer carries a context chip for The Dye Works, so "is it busy on Sundays?" needs no place name.
4. The user taps Keep, or drags the sheet down. Unkept, the brief later joins a fold under the Alder Quay stamp.

## 11. Trust, privacy, accessibility

**Honesty of the glow.** The client sends a viewport or location signal only while it renders Noticing or Working. The Now group in Mira's drawer lists each input in use with its reason ("Reading this view because you asked about lunch"). This is a client contract today; nothing on the server enforces it.

**Pause.** Pause Mira, in her drawer, sets Dimmed. The client then sends no viewport or location, and proactive work stops. Tasks the user started keep running, and the drawer says so. An open approval stays answerable.

**Memory.** The Memory entry in Mira's drawer lists PROFILE memories (editable, deletable) and STATE memories (open commitments, read-only, each linked to its strand). "Forget this" on a strand or a fold calls `ForgetSegment`. Assumption for the lead to confirm: Mira stores places the user saved and facts the user stated, and no movement trail.

**Accessibility.**

- Each state has a spoken name, announced politely on change: "Mira, working on 2 tasks", "Mira needs your approval". Noticing is not announced, because it fires on every pan.
- Colour never carries state alone. Each state differs in shape or motion (ring for Needs-you, motes for Working, closed eyes for Dimmed), and the ring differs from the honey glow in lightness as well as hue. The scoped composer changes colour and also shows the scope chip.
- With reduced motion on, pulses and hops become static changes (rim, ring, mote count), camera flights become cuts, and drops of light become fades.
- Every map-placed Card is also a list in the sheet, so marks are reachable without the map.
- With a screen reader on, whispers stay until dismissed.
- Hold-to-talk has a tap-to-toggle alternative. The lens is a 76 px target and the menu, app settings and map controls are 48 px. Strand chips are 34 px tall, so their hit area extends to 44 pt.

## 12. API mapping and gaps

| UX need | Existing `mira.v1` primitive | Gap |
|---|---|---|
| Durable thread across devices | Event log, `SyncEvents`, `StreamEvents` | None |
| "She's answering", with a reason | `TypingPayload.activity_hint` | None |
| Background work, motes | `Task`, `TaskStep`, `ListTasks`, `WatchTask` | No recurring or watch Task kind; no geo anchor |
| Approvals | `ApprovalRequestedPayload`, `ResolveApproval` | None |
| Structured results | `Card{kind, fallback_markdown, payload_json}` | No placement hint; map, route and place kinds and their geometry schema undefined |
| Viewport and location as input | `ClientSignal` on `StreamEvents`, `PublishSignal` | No context signal (bounds, zoom, location, heading, selection); no durable context snapshot on `AppendEvent` |
| Browse mode | None | A standing viewport query: one query that re-runs as bounds change. Today each pan would be a new message |
| Whisper-only and push-worthy events | Push from server-side rules | No attention level, expiry or initiator on agent events |
| Viewport whispers that leave no trace | Ephemeral events (typing, presence) | No ephemeral agent message; no persist-on-tap |
| Tappable places, "about this" | `MessagePayload.markdown`, `AttachmentRef` (blobs only) | No place reference type |
| Strands and capsules | `Conversation.Kind.KIND_SIDE` (reserved), `segment_id` | Side conversation or projection is open (question 1). Either way a capsule needs a stable strand id, a title, a state and the latest line. Keep has no write path, since clients cannot create Tasks |
| Scoped composer | `AppendEvent` | No field naming the conversation or strand an event targets |
| Folds | `segment_id` on ephemeral segments | None on the server if folds are a client-side projection; the client needs a segment's place and end time to label one |
| Agent settings | `MemoryService` for memory | No server-side preference for proactivity level or for what she can see |
| Voice | Reserved live media seam | No proto |
| Live itinerary editing, selection shared across devices | Reserved WorkspaceService seam | No proto; selection is client-local in v1 |
| Editable memory | `MemoryService` PROFILE and STATE, `ForgetSegment` | Memories are free text; no saved-place structure |
| Pause | `PresencePayload` | No server-side pause of proactive work |
| Nudges while backgrounded | Push | No path for background location to reach the agent |

## 13. Open questions

1. **Are strands side Conversations or a client projection?** Side Conversations give each strand its own history and a natural target for the scoped composer, but reverse the backend's v1 lean. A projection over `segment_id` and Tasks ships sooner but cannot hold a kept list that has no Task, and it has nowhere to store a strand's id and title.
2. **Which proactivity level is the default, and is "this map view" on at first run?** Balanced with the view on makes wandering work from day one. Quiet makes the first Noticing an explicit choice. The answer sets the onboarding flow and the default cost per session.
3. **Are unopened whispers persisted?** We propose that an ignored viewport whisper leaves nothing and an opened one lands in the thread. That needs a new ephemeral agent message and a persist-on-tap path.
4. **What does a fold promise?** Hiding rows (proposed) or deletion from history. Deletion contradicts the append-only log and needs a retention rule.
5. **Where are background location triggers evaluated?** On the device (geofences on saved places, private, capped by OS quotas) or on the server (flexible, uploads location while backgrounded). This decides whether scenario (d) works with the phone locked.
6. **Who enforces the attention budget?** The server holds the proactivity level and knows all pending items; the client knows whether the user is mid-gesture. A split needs a rule for which side may downgrade.
7. **Is a drooped Has-something enough for `FAILED`?** Section 3 decides that failure reuses Has-something with a drooped tip and no bloom. If testing shows people miss failures, the vocabulary needs a tenth state.
8. **Does a plain place-name search go through the agent?** With no search bar, "Lisbon airport" lands on Mira. If her first response is slower than a geocoder, the client needs a local fast path that still writes to the log.
9. **May Mira re-route a message the scope chip sent to the wrong place?** A user focused on the Porto strand may ask about lunch. If the server may move it to the thread, the target on `AppendEvent` is a hint; if not, it is binding and the client must make leaving a strand obvious.
