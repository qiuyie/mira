# Mira, the map companion: product spec

Status: draft for review, 2026-11-22. v4: journeys, layers and the shelf, after one critique round. Final figure: the brand pin, set free, with eyes. UI mocks and the character sheets: https://claude.ai/artifact/K31Jt8FNidngqVZU2gUf3Z
Backend references: `docs/api-outline.md`, `proto/mira/v1/*.proto`.

## 1. Summary

Mira is an AI agent that lives on a mobile map as a small figure in a round glass button, the *lens*, at the top-centre of the screen. The map is full-bleed with no search bar: the bar at the bottom edge is how the user asks for anything. She reads the user's location, the map view and what the user types or says, and answers by changing the screen: marks, photos and blocks, with one line of text. Her figure and the lens aura are the status display, and the user's proof of when she is reading location or the viewport. The sheet above the bar holds one append-only log, the *thread*. Each goal in it is a *journey*, and each answer is a *leaf* (section 8). The backend has the durable parts (event log, Tasks, approvals, Cards, memory). It has no notion yet of a viewport, a place, a journey, or how loudly an agent-initiated event may speak (section 12).

## 2. Principles

| Principle | Consequence |
|---|---|
| The map is the product; Mira is a guest on it. | Nothing she draws blocks panning. She moves the camera only after a user action, and a whisper never moves it. |
| Every query goes to Mira. | The bar is the only input, so a plain place-name search must still feel instant. |
| The frame never moves. | Each edge has one job at every sheet height. |
| The response is a change to the screen. | She leads with the map or the photo, and the user raises the sheet for more. |
| The figure never lies. | If she reads location or the viewport, she is visibly Noticing or Working. |
| Attention is a budget. | Viewport noticing stops at a whisper. Only approvals hold the lens until answered. |
| Consequential actions need the user's yes. | Booking, paying and calling go through an approval whose text is the exact consent. |

## 3. The figure

### 3.1 Who she is

**Mira is the brand pin, set free.** She is the product's own four-colour map pin, flipped tip-up, with the pin's round opening kept as her white face window. A pin marks a place; flipped and set free, it goes with you.

The name is the star Mira, the first star people saw brighten and fade, and the Spanish "¡mira!", "look!". She is warm, curious and calm. If nothing is worth saying she says nothing, and the figure rests.

The figure took two rounds. "The pin that came alive" won the first on silhouette and the flip, over a star-sprite and a lantern creature. The second rooted it in the brand. Its other option, the icon upright with a face in its opening, read at rest as a logo with a face.

Eyes were a decision. Of three built versions (eyes, no eyes with a window that blinks and squints, eyes on demand) we chose eyes, for warmth and because gaze, sleep and joy read without learning.

The figure is built on the product's brand mark, which assumes brand approval: an open dependency (question 1).

### 3.2 Anatomy

| Part | What it is | What it expresses |
|---|---|---|
| Tip | The curled point on top, the old pin point, green | Mood and attention. It sways, perks and flops |
| Window | The pin's round opening, white: two dot eyes, a blush, a tiny smile | Her face. The eyes follow the pan |
| Body | The four brand colours, rotated with the flip: green tip and shoulders, yellow right, red lower right, blue lower left | Her presence. It is always visible |
| Aura | A soft glow in her own colours | How busy she is |
| Motes | Up to three small lights in orbit: yellow, green, red in that order, never blue | One per running Task. The same motes mark each journey capsule (section 8) |
| Ring | A steady blue circle on a white gap ring | It is the user's turn |
| Lens | The 76 px round glass button that frames her: white rim, four-colour aura | That she can be tapped |

She ships inside the lens: a 76 px glass button, top-centre, 54 px from the top edge (assumption: this clears a camera cutout), with a body of about 44 px. Outside the lens she scales from 24 px (notification row) to 240 px (brand).

We frame her because the bare v1 figure did not read as a button and its aura lost contrast over parks and water.

### 3.3 Signature motion per state

Each state shows in her body and in the strength of the lens aura, which reads in the corner of the eye without resolving a 44 px face.

| State | Motion |
|---|---|
| Resting | Slow float on a 4.8 s cycle, the tip sways, an occasional blink, low aura |
| Noticing | She leans toward the pan, the face window slides and peers, and a sheen sweeps her body as the scan |
| Working | The aura pulses on a 1.8 s beat, the tip ticks, and one mote orbits per running Task |
| Has-something | She perks up, amber rays bloom once, a white idea dot sits above the tip, and a whisper, where one is allowed, unfurls under the lens |
| Listening | Her body opens and the tip cocks like an ear. Thin green rings and the aura follow the voice |
| Composing | **The flip.** She spins tip-down, the curled tip straightens and the window shrinks to the icon's proportion, so she lands exactly on the brand pin (red upper-left, blue upper-right, yellow left, green tip). She pours drops from the tip and spins back |
| Needs-you | The blue ring holds and the aura turns down. She is almost still |
| Delivered | One happy hop with smiling eyes and three sparkles, played once |
| Dimmed | Greyscale, eyes closed, tip flopped over, no aura |

The flip ties the figure to the map: every mark she places arrives as a drop from her tip, so the user sees where an answer came from.

### 3.4 Motion principles

1. **A still frame before motion.** A state must be readable from a still frame (aura, ring, motes, eyes, tip). That is also what the reduced-motion version shows.
2. **One signal at a time.** She shows one state, chosen by the precedence in 3.6. Only the ring and the motes persist under other states.
3. **She never bounces for attention.** Nothing loops a hop or shows a badge count.

### 3.5 Colour

**Four colours are Mira. Yellow is her light. Solid blue is you.** Her body is the four brand colours: red `#EA4335`, blue `#4285F4`, yellow `#FBBC04` and green `#34A853`. Yellow is her light: her marks on the map, lit areas, the dotted threads that run from her to each mark she places, the tint of her messages, and her chips. The user is solid blue `#1A73E8`: the location dot, the user's messages, approvals and the Needs-you ring.

One tension is known: her body contains a blue segment. So blue as "you" is always a solid ring or a solid fill and never a blend, and no prop outside her body is blue. Colour is never the only signal (section 11).

### 3.6 States and signals

| State | Trigger | What the user can do | Backing API signal |
|---|---|---|---|
| Resting | Nothing running, nothing unread | Ask in the bar; tap the lens for her drawer | No agent `TypingPayload`; no `RUNNING` Task |
| Noticing | Camera settles and the viewport is sent to her | Keep panning (cancels); ask about this view | Client-local, tied to sending the viewport. **Gap:** no viewport signal |
| Working | One or more Tasks are `RUNNING` | Tap the lens to see what is running and why | `TaskUpdatePayload`, `WatchTask`; one mote per running Task |
| Has-something | She has a line worth saying | Tap the whisper, swipe it away, or ignore it | Agent `MessagePayload`. **Gap:** no attention level |
| Listening | User holds the microphone in the bar | Release to send; slide away to cancel | Client-local. **Gap:** live media seam has no proto |
| Composing | She is answering or building UI | Wait, or interrupt with a new message | Agent `TypingPayload{active, activity_hint}` until the message lands |
| Needs-you | An approval is open | Tap the shelf row, read the consent text, approve or decline | Unresolved `ApprovalRequestedPayload`; Task `WAITING_APPROVAL` |
| Delivered | A Task completes | Open the journey from its post or capsule | `TaskUpdatePayload{COMPLETED}` |
| Dimmed | Offline, paused, or location off | Tap the lens to resume or see what is off | Client-local. **Gap:** no server-side pause |

Two decisions follow. A Task in `WAITING_EXTERNAL` does not pulse: Working means compute is happening now, so a three-day table watch is a waiting mote on a Resting figure. A `FAILED` Task has no state of its own: she shows Has-something with no bloom and a drooped tip.

When several states apply, the precedence is Dimmed, Listening, Needs-you, Composing, Has-something, Noticing, Working, Resting.

## 4. The frame

The screen is the base map, the *map marks* Mira draws on it (lit areas, labelled pins, routes), and a frame whose edges each have one job. Map controls (recentre; in navigation, mute and overview) sit on the right above the sheet.

- **Top-left, the menu:** saved places, lists, trips, layers, offline maps.
- **Top-centre, the lens:** opens *Mira's drawer*, a panel that descends from under the frame.
- **Top-right, app settings:** units, navigation, notifications, privacy and location, account.
- **Under the lens, the *label slot*:** the breadcrumb label on Layers 1 and 2 (section 8.3). A whisper uses the slot only at Layer 0 with the sheet at Rest, and never repeats text visible in the sheet.
- **Bottom, the *bar*:** a full-width composer with the microphone pinned right, the same on every screen.

Two things sit above the bar, and neither ever moves it.

- The *chip row* is one row directly above the bar. It holds assumptions and filters only ("Leave now", "Under 10 min"), ordered by importance. Actions are buttons in the sheet, never chips.
- The *shelf* sits above the chip row, outside the scrolling log, at every sheet height. It is the one place for decisions: two rows at most, then a "+1 more" line, and one row at Rest. A row shows a title, a state with its deadline, and one pill ("Review"). With no decision open it may show one running journey.

Mira's drawer holds five groups, all views over the thread:

- **Needs you**: the shelf's open decisions.
- **Running**: what she is doing now, and each input she is reading and why.
- **Coming up**: confirmed things with a date, until the date passes.
- **Watching**: Standing journeys, each with "last checked today · last find 14 Oct" and a Stop button.
- **Earlier**: everything else, searchable.

Her settings follow: proactivity level (Quiet, Balanced or Chatty), what she can see (location, this map view), memory, voice, and Pause Mira. Agent settings sit behind the lens, apart from app settings, because a user who wants her quieter looks at her first. Location is on both sides: the operating system permission in app settings, whether Mira reads it in the drawer.

The sheet has four heights, and the thread opens at Half.

| Height | Size on a 390 x 844 screen | Above the bar |
|---|---|---|
| Rest | 112 px, plus the shelf row if there is one | Nothing else |
| Peek | 200 to 230 px | One answer, a card rail, or a route strip |
| Half | About 430 px | The thread's newest entries, a journey, or a leaf |
| Full | Sheet top at y = 150 | The whole log with search in its header, or a full canvas. The frame stays visible above |

## 5. Inputs

| Input | What Mira does | Privacy note |
|---|---|---|
| Passive location | Checks saved places and running journeys against it. | Needs the OS permission and the drawer's switch. Not kept as a trail (assumption). |
| Viewport, the deictic "here" | On camera settle (assumption: about 600 ms idle) the client sends bounds and zoom. She shows Noticing and may whisper. | Sent only while she shows Noticing, and only if "this map view" is on. |
| Tap on a place | Opens a place leaf at Half. | The place is sent only on the tap. |
| Drag a mark or map point into the bar | An "About" chip joins the chip row. | Sent only with the next message. |
| Text or voice | Composing, then a leaf. Listening while the microphone is held. | Persisted with a snapshot of viewport and selection. v1 assumption: voice is transcribed on the device. |

Scope resolves by rule: "here" and unqualified requests mean the viewport, "near me" means the device location, and an "About" chip overrides both. Every answer names its scope as a chip ("In this view").

## 6. Outputs: the response layer

We adopt the response layer from the separate "Beyond the Chat Box" work, which designed how an answer uses a whole map screen where a transcript would use a bubble. Its three principles are now ours:

1. The response is a change to the screen, not a message about it.
2. Lead with the medium the evidence lives in.
3. Act first, but name the assumption where it can be changed.

### 6.1 Intents, canvases and blocks

A request has one of three intents. The glosses are our reading of that work.

| Intent | Mira responds by | Example |
|---|---|---|
| Frame | Putting the right thing in view: camera, lit area, chips | "Show me Alder Quay" |
| Show and Tell | Composing a canvas with one line of text | "Dinner near the viewpoint" |
| Do | Acting, then leaving a receipt or asking approval | "Add a coffee stop" |

A response uses one of four canvases, chosen by where the evidence lives: *map-heavy* (two coffee stops beside a route), *imagery-heavy* (what a terrace looks like), *both* (dinner places as photo callouts on the map with cards below), or *text* ("Saltmarsh Books: Sunday 11 to 4").

A canvas is composed of *blocks* on a grid one, two or three across. A block has a lead-in, a key highlight and related information. A *map photo callout* is a small photo that sits above its point on the map. Mira picks the canvas and blocks, not the layout: Cards stay a closed, versioned vocabulary. A map-placed Card's `fallback_markdown` must read as a list, because a screen reader gets exactly that.

**The canvas chooses the medium; the sheet height chooses how much.** The dinner response at Peek is the map with four callouts and one line. Half adds a row of cards, and Full adds the blocks for the selected place.

### 6.2 Where we differ from that work

That work removes the transcript, and for a single answer it is right: nobody needs a bubble saying what the map shows. We differ in three places:

- **Conversation stays.** Messages and posts exist and the thread is the record, because work that runs for days needs somewhere to report.
- **The frame stays.** Menu, lens and settings hold their edges on every layer, and the lens stays 76 px.
- **Consequential actions wait for approval.** "Act first" covers reversible actions only.

### 6.3 Acting

A reversible action happens at once and leaves a receipt with Undo ("Stop added · +2 min · Undo"). A consequential action (booking, paying, calling on the user's behalf) opens the *approval sheet*, the one approval component, whose text is the exact consent (`ApprovalRequestedPayload.summary_markdown`). In a sheet it is one full-width button that shows the deadline: "Review booking · held until 9:00 pm". Posts, shelf rows and that button open the approval sheet and never commit.

In the approval sheet, "Don't book" declines. "Not now" leaves the approval pending and clears the lens ring; its shelf row and post remain until the deadline.

A post with an approval renders from the approval's live state. While pending it shows the deadline ("Held until 6:00 · 52 min left") and buttons. Once resolved, one line replaces them: "Confirmed 5:12 pm · on your watch", "Expired 6:00 pm · Ask again" or "Declined".

## 7. Map and sheet choreography

**Camera padding.** The visible region is the screen minus a top inset of 150 px (the frame) and a bottom inset equal to the sheet height. Padding follows the sheet drag, so the subject stays centred. At Full the camera freezes.

**Shared selection.** There is one selected item. Tapping a mark, a callout, a card or a list row sets it: the mark lifts, the row highlights, and the camera moves the least distance that shows the mark. Scrolling a list never moves the camera.

**The thread drives the map.** The entry in view sets what the map shows: a find shows its callout, a call shows its pin. Every place named in the thread is tappable and flies the camera there.

**Browse mode.** While a card rail or result list is open, the query follows the map. A pan re-sorts the list by distance to the new centre. On settle, Mira shows Noticing and fetches for the new viewport. New results arrive under a "3 new in this view" row and never replace rows under the user's finger.

## 8. Journeys: one log, three layers

The sheet shows one log at three depths: the thread, a journey, and a leaf. v2's capsules updated in place, which kept a three-day table watch to one row, but Thursday's result sat in Monday's row. v4 keeps the capsule and adds posts for news.

### 8.1 Layer 0: the thread

The thread is one append-only log per person, with stamps ("Alder Quay · today 12:38") that mark where the user was as well as when. The product owner decided the append-only rule: **a capsule stays where the journey started, and news arrives as posts at the bottom.** Chat stays: Mira can always message the user, who can always type.

The log holds four kinds of entry.

| Entry | What it is | Example |
|---|---|---|
| Message | The user or Mira speaking | "Is it open Sunday?" "Yes, 11 to 4." |
| Journey capsule | A live view of one goal: title, mote, latest line, canvas thumbnail, state word | "Weekend in Porto · 3 of 5 sorted · Needs you" |
| Post | A journey's news: a result, a step, an approval request or a find. It names its journey | "From Table at Osteria Lume: 7:30 for two. Held until 6:00." |
| Fold | Quick answers or a past day in one line. It deletes nothing | "Tue 3 Nov · Alder Quay · 2 topics, 5 answers" |

The shelf is not an entry. It is a pinned view of the open decisions, and each decision's post stays in the log. A capsule's position never changes and its content is always current. It is never redrawn at the bottom for being active: that is the shelf's job.

A post's "From Weekend in Porto" link opens the journey and never scrolls the log. A "Since you were away" line marks where the user left off.

### 8.2 Layers 1 and 2

**Layer 1 is a journey.** Tapping a capsule takes the sheet one layer deeper. It shows the journey's canvas (a plan, a route, a shortlist), its own log, and a list of its side questions. Input is scoped to the journey, and a chip says so: "In: Weekend in Porto".

**Layer 2 is a leaf**: one response on the whole canvas, with no bubbles and one line of text at most. It updates in place. Tapping a card or a callout opens a deeper leaf (a place, a menu). Every leaf question and action is an event in its journey, so a leaf leaves a trace.

### 8.3 Navigation

Each control means one thing. Sheet height is how much. Back is one layer up. Scroll is time. × returns to the map at Rest, and the journey keeps running.

Back sits by the thumb. On Layers 1 and 2 the first item in the chip row is the *back pill*: a chevron and the parent's exact title ("‹ Weekend in Porto", "‹ Thread"). The operating system's back gesture does the same; there is no other swipe for depth, because one would collide with chips, photo strips and map panning. The *breadcrumb label* in the label slot names where the user is, with the same parent title, and carries the ×.

### 8.4 Who organises

Every input goes to the same bar, and Mira files it: continue this journey, start a new one, or just answer. She shows the filing as one chip on the answer, "In: Weekend in Porto", first after the back pill. Tapping it offers "Move to: New journey · Just answer".

Later re-filing is a long-press on any entry, then Move. The move leaves a consequence receipt that says plainly what changed and what could not be undone: "Moved out of Weekend in Porto · plan change undone · Undo".

### 8.5 Lifetimes, states and promotion

A journey has one of four *lifetimes*. They are internal vocabulary that sets attention (section 9) and expiry. People see a state word, never the lifetime.

| Lifetime | Lasts | Journey types | Leaf canvas |
|---|---|---|---|
| Moment | Minutes | A quick question; getting to know a place; discovery browsing | Text; both for a place or a browse |
| Errand | Hours | Directions; tonight's table | Map-heavy with the route strip; text with blocks |
| Project | Days | Trip planning | Both, under a plan canvas at Layer 1 |
| Standing | Until stopped | A watch ("New places near home") | Both, one leaf per find |

A Moment shows in the thread as its messages and later folds; browsing is one entry, not one per pan. The other lifetimes show a capsule and posts. A proactive find takes the lifetime of its journey, and is a Moment if it has none.

Quick answers stay quick. Nothing is promoted silently. A Moment becomes an Errand in three cases: Mira keeps working on it (she creates a Task), the user replies to it later, or the user taps Keep. Reply is the quote-reply of any messaging app: a long-press on an answer offers Reply, Keep and Move, and a reply carries `reply_to_event_id`, which the API already has. From the reply on, the answer has a title and a capsule where its first message sits. An Errand becomes a Project when it spans a night or gains a second goal. Nothing becomes Standing silently: Mira asks first. A promoted Moment gains a capsule where its first message sits.

A journey is in one of eleven states, and its state word says which in the journey's own terms ("Calling at 5").

- **Active:** running; waiting on someone else (a `WAITING_EXTERNAL` Task); needs you (an approval is open); retrying ("Retrying 5:30"); watching (a Standing journey between finds).
- **Held:** paused, by the user or by Pause Mira.
- **Over:** done; failed (every attempt spent); lapsed (time ran out with no decision: "Hold expired"); ended (by the user, before the goal was met); stopped (a Standing journey, by the user).

The mote beside the title shows state and nothing else. A capsule stays in the log in its last state.

**Words people never see.** Journey, leaf, post, shelf, capsule and the four lifetime names are working vocabulary for this spec and the team. The interface shows titles ("Weekend in Porto"), state words ("Needs you") and familiar verbs (Reply, Keep, Move). The top layer is called Thread; a new one is a "New topic".

### 8.6 How the old terms map

A v2 strand is a journey. The strand rail is the shelf, which now lists decisions, not every live journey. The capsule is kept, as a live view fixed where the journey started. The focused strand is Layer 1, and the bar no longer changes colour.

## 9. Proactivity and attention

Mira may speak unasked on four rungs: the canvas and figure, a whisper (assumption: gone after about 6 seconds), a post in the thread, and a push. A journey's lifetime sets how high she climbs.

| Lifetime | Highest rung unasked | Push |
|---|---|---|
| Moment | Canvas only | Never |
| Errand | Whisper | Approvals, final failure, and results awaited while the app is closed |
| Project | Post in the thread | Approvals with a deadline, and time-critical changes |
| Standing | Post in the thread | Only if the user opted in for that journey |

The proactivity level in Mira's drawer can lower these ceilings and never raises them: Quiet stops at the figure, plus approvals, and Chatty lowers her bar for a whisper. The thresholds are unmeasured.

Hard rules, at every level:

1. Viewport noticing never escalates past a whisper on its own. An ignored whisper leaves nothing in the thread; an opened one becomes a Moment.
2. Only an open approval holds the lens ring, until it is answered, deferred or expired.
3. One whisper shows at a time, and only where section 4 allows. Elsewhere the news is only a post.
4. A dismissed whisper is not repeated on the same topic in the same session, and none appears while the user is typing, speaking or dragging the map.
5. Every push states its reason ("You asked me to watch this") and is withdrawn when its cause resolves.
6. A post never displaces the current task, and she never raises the sheet unasked. While the user navigates, a post is one row above the route strip, and the next turn stays.

### 9.1 Unhappy paths

| Case | Post | Capsule | Lens | Push |
|---|---|---|---|---|
| A call fails or gets no answer | "No answer at Osteria Lume. Trying again at 5:30." | "Retrying 5:30", then "Couldn't book" | Nothing while retrying; then Has-something, tip drooped | Only on final failure |
| A hold expires | "Expired 6:00 pm · Ask again" | "Hold expired" | The ring clears | Withdrawn |
| Confirmed on another device (the first resolution wins) | "Confirmed 5:12 pm · on your watch" | The new state | The ring clears on every device | Withdrawn |
| Two decisions at once | One each | Each reads "Needs you" | One ring, no count; the shelf orders them by deadline | One per approval |
| Filed in the wrong journey after Mira acted | A receipt: what was undone and what was not | Both show their new state | No change | None |
| A Standing journey silent for six weeks | "Nothing new near home in six weeks. Keep watching?" | "Watching". Unanswered, it lapses (assumption: after two more weeks) | Nothing | None |
| Trip dates pass | Pending approvals read "Expired" | "Done" if anything was confirmed, otherwise "Dates passed" | The ring clears | Withdrawn |

A move cannot undo a committed consequential action. The receipt says so and offers to cancel, through its own approval.

## 10. Scenarios

**(a) Lunch nearby.** With the sheet at Rest, the user types "anything good for lunch?". Mira flips and drops six yellow marks on the map. A leaf opens at Peek with photo callouts, a card rail and one line; the chip row reads "In this view", "Open now". At Half the rail becomes a list, and a pan east re-sorts it. The user taps ×, and the Moment later joins a fold.

**(b) Wandering.** The user pans an unfamiliar neighbourhood with the thread at Rest. A whisper takes the label slot: "This street has three old bookshops." The user ignores it, and nothing lands in the thread. A later whisper about a covered market gets a tap, and a leaf opens at Peek. The user taps Keep: the Moment becomes an Errand with a capsule, and a receipt offers Undo.

**(c) Weekend trip.** The user says "plan a weekend in Porto next month". Mira files a new journey and creates a Task, and a capsule, "Weekend in Porto", appears in the thread. Next day a post waits under "Since you were away" and opens the journey at Full, with "‹ Thread" as the back pill. Lantern Room holds a table for Saturday at 8:00 pm until 9:00 pm: a post lands with the deadline, the shelf shows a "Review" row, the lens shows the ring, and a push goes out. The user approves in the approval sheet, and the client calls `ResolveApproval`. The post reads "Confirmed", the push is withdrawn, and the booking joins Coming up.

**(d) Proactive nudge while walking.** The user is walking to Kiln & Rye with the route strip at Peek. A saved bakery, two minutes off the route, closes in an hour. The sheet is not at Rest, so there is no whisper: the post is one row above the route strip. Tapping it opens a leaf with the detour and an "Add stop" button, which leaves "Stop added · +2 min · Undo". With the phone locked, it is a push only if the user asked for alerts on saved places.

**(e) Walking directions, with a question on the way.** The user asks for a walking route to Kiln & Rye. Mira files a new journey and shows its route strip at Peek, with "Leave now" and "Avoid stairs" in the chip row. Mid-walk the user asks by voice for a cash machine on the way. Mira files it in this journey ("In: Walking to Kiln & Rye") and answers with a leaf: one mark beside the route and one line. The back pill returns to the route.

**(f) Getting to know a place.** The user taps The Dye Works on the map. A place leaf opens at Half: photos, a block for its hours, and one line on why it suits the user. The chip row carries "About: The Dye Works", so "is it busy on Sundays?" needs no place name. When the user leaves, the Moment joins a fold.

## 11. Trust, privacy, accessibility

**Honesty of the figure.** The client sends a viewport or location signal only while it renders Noticing or Working. The Running group in her drawer lists each input in use with its reason ("Reading this view because you asked about lunch"). Nothing on the server enforces this.

**Pause.** Pause Mira sets Dimmed. The client sends no viewport or location, and proactive work stops. Tasks the user started keep running, and the drawer says so.

**Memory.** The drawer lists PROFILE memories (editable, deletable) and STATE memories (open commitments, read-only, each linked to its journey). "Forget this" on a journey or a fold calls `ForgetSegment`. Assumption to confirm: Mira stores places the user saved and facts the user stated, and no movement trail.

**Accessibility.**

- Each state has a spoken name, announced on change ("Mira, working on 2 tasks"), except Noticing, which fires on every pan. A layer change announces the new title.
- Colour never carries state alone: figure states differ in shape or motion, and a capsule has its state word.
- With reduced motion on, pulses and hops become static changes and camera flights become cuts.
- Every map-placed Card is also a list in the sheet, and with a screen reader on, whispers stay until dismissed.
- Hold-to-talk has a tap-to-toggle alternative. The lens is a 76 px target, the other frame buttons are 48 px, and chips and shelf pills extend their hit areas to 44 pt.

## 12. API mapping and gaps

**We propose that a journey is a new resource of its own, inside the one main conversation.** It is not a Task: a journey can hold several Tasks or none (a kept list), and clients cannot create Tasks. It is not a side Conversation: posts must land in the one append-only log, and a history per journey would split it. The log and the single `KIND_MAIN` conversation already fit the thread. The first table lists the other gaps; everything in the second is proposed, not built.

| UX need | Existing `mira.v1` primitive | Gap |
|---|---|---|
| Canvases, blocks, callouts | `Card{kind, fallback_markdown, payload_json}` | No canvas or grid hint; route, place and callout kinds undefined |
| Receipts with Undo | An agent message | No client call that undoes an agent action |
| Viewport and location | `ClientSignal`, `PublishSignal` | No context signal (bounds, zoom, location, selection); no context snapshot on `AppendEvent` |
| Browse mode | None | A standing viewport query; today each pan is a new message |
| Traceless whispers | Ephemeral events | No ephemeral agent message; no persist-on-tap |
| Tappable places, "About" chips | `AttachmentRef` (blobs only) | No place reference type |
| Agent settings, Pause, voice | `MemoryService`, `PresencePayload`, the live media seam | No server-side proactivity level or pause; no voice proto |
| Background nudges | Push | Background location cannot reach the agent |

| Journey need | Proposed addition |
|---|---|
| The resource | `Journey{id, title, lifetime, state, latest_line, canvas_ref, expires_at, task_ids[]}` |
| Capsules as live views | Journey started and journey updated events, plus calls to list and watch journeys |
| Posts and attention rules | `journey_id` on agent events, and an attention level with an expiry |
| Scoped input | A target journey on `AppendEvent`. It is a hint, because Mira files |
| Filing and re-filing | An agent-appended filing event, and a client re-file call that appends and triggers compensation for what Mira did under the old filing |
| Pause, end, stop, keep | A client journey update call |
| Standing journeys | A recurring Task kind |
| Pushes | Withdrawal on resolution |
| "Not now" | An approval defer, so the ring clears on every device while the approval stays pending |
| Leaf state | The reserved WorkspaceService seam. Until it has a proto, a leaf is a static Card |
| Folds | A client projection over the log. No server change |

## 13. Open questions

1. **Does brand approve a character built on its mark?** This spec assumes yes. Without approval the figure cannot ship as drawn.
2. **Is reply-to-continue enough?** Section 8.5 decides that an answer lasts only if Mira keeps working, the user replies, or the user taps Keep. If testing shows people lose answers they wanted back, Keep needs to be more visible than a long-press.
3. **How loud may a Standing journey be by default?** We propose thread only, with push opt-in per journey. If finds go unseen for days, the default needs one push.
4. **Is the thread log ever pruned for the user?** Folds delete nothing, so the log grows for years. The server has a retention edge (`earliest_retained_seq`); a user-visible rule would decide what Earlier can search.
5. **How much conversation shows by default?** This is our remaining disagreement with the response-canvas team. They would open on the canvas with no transcript; we open the thread at Half. A test of which opening lets people find long-running work would settle it.
6. **Which proactivity level is the default, and is "this map view" on at first run?** Balanced with the view on makes wandering work from day one; Quiet makes the first Noticing a choice.
7. **Where are background location triggers evaluated?** On the device (private, capped by OS quotas) or on the server (uploads location while backgrounded). This decides whether scenario (d) works with the phone locked.
