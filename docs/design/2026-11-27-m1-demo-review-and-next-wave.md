# Mira M1 demo: review and spec for the next wave

Date: 27 November 2026.
Reviewed: the iOS recording "Mira M1 Cycle5B, iPhone 16 Pro, Live" (2 min 53 s, no audio).
Method: watched at one frame per second, plus ten moments at full size. Transition smoothness and anything that happens between frames was not judged.
Design reference: `2026-11-22-mira-map-companion-product-spec.md` (called "the spec" below).

The build's own debug page says it runs in "Text first (M1)" mode and lists "Rich companion (M2)" as the other mode. Several findings below are therefore expected for M1. They are still listed, because they are what M2 has to change.

## 1. What the recording shows

| Time | What happens |
|---|---|
| 0:02 | Home. Top row: "Your map" pill, Mira's lens, a history button, an account avatar. Bottom bar: home, "Ask Maps" input with the brand pin, microphone, bookmark. |
| 0:06 to 0:21 | Tapping the input opens an "Ask Maps" sheet with one line of empty-state text. The sheet changes height five times while the user types "Best bakeries near Ferry Building". |
| 0:22 to 0:24 | A thread appears: stamp, user bubble, "Thinking", then "Searching bakeries near Ferry Building". Mira flips into the pin. The answer streams in as paragraphs. |
| 0:25 to 0:36 | The sheet collapses itself to a one-line bar. The camera flies to open water north of the piers. A whisper says "5 places found · Tap to view". The result pins sit under the bottom bar. A second whisper, cut off, names a pier gallery. |
| 0:37 to 0:55 | The user opens and closes the sheet three times and pans. The map shows water the whole time. |
| 0:56 to 1:11 | The user types a second question, a coffee meetup between three New York locations. Only now does the camera show the bakery pins. Their labels overlap. |
| 1:12 to 1:25 | The map flashes blank, then shows the East Village with no result marks. The answer is again paragraphs. At its end is a card: "3-way coffee meetup · 4 map views · 3 places". |
| 1:26 to 2:01 | The card opens one layer down: a back arrow, four view chips (Overlap, Wider area, Areas, Routes), two place chips, and the same paragraphs again. The Routes view draws three coloured paths to one cafe. The input reads "Ask about these". |
| 2:02 to 2:25 | Back in the thread, the map marks are gone and the list is mid-scroll. |
| 2:26 to 2:53 | "Your map" opens a popover with the account, a Light or Dusk switch and a server-connection row that leads to a debug page. |

## 2. What works

1. **The figure is alive on a real map.** Resting, the Composing flip, Delivered and Has-something all read at lens size on light and dusk maps.
2. **The Routes view is the best moment in the demo.** Three origins, three coloured paths, one destination. It answers the question on the map. This is the product.
3. **One layer down exists.** The meetup card opens its own layer with a back arrow, and the input scopes itself ("Ask about these").
4. **The collapsed bar keeps the view chips** (1:54), so the map can be explored without losing the controls.
5. **Status is said in words** ("Searching bakeries near Ferry Building"), and a stop button replaces the microphone while she works.

## 3. Findings

P0 breaks the task. P1 costs time or trust. P2 is polish.

| # | Finding | Evidence | Fix |
|---|---|---|---|
| 1 | **P0. The camera leaves the answer.** After the answer the map shows open water and the pins are under the bar. The user spends about 30 seconds looking for them. | 0:25 to 0:55 | Fit all result marks inside the visible map rectangle: below the lens, above the sheet at its current height. Do not move the camera before results exist. |
| 2 | **P0. The answer hides itself.** The sheet collapses on its own and a whisper says "Tap to view". The user asked; the answer should be on screen. | 0:25 | A response to a question keeps the sheet at Peek with the answer in it. Whispers are only for things Mira raises herself (spec 9). |
| 3 | **P0. The answer is a wall of text.** Five places arrive as paragraphs with rating, price and hours inside sentences. Nothing in the list is tied to a pin. | 0:24, 0:38, 1:13 | The M2 response: photo callouts on the map, three blocks for the selected place, one line of text (spec 6). |
| 4 | **P1. An unprompted whisper interrupts the task, and is cut off.** "Pier 1 Fisher Bay Observatory Gallery a…" appears ten seconds after the user's own question. | 0:35 | No viewport whisper while an answer is on screen or within a set quiet time after a question. A whisper must fit one line; write it to fit, never truncate. |
| 5 | **P1. The sheet has no stable heights.** It moves between at least five positions with no rule the user can learn. The empty "Ask Maps" sheet takes most of the screen to show one sentence. | 0:06 to 0:21 | Four heights only: Rest, Peek, Half, Full. Tapping the input focuses it and shows suggestion chips; it does not open an empty sheet. |
| 6 | **P1. Top controls change identity.** The avatar becomes a close button when the sheet is open. The history button disappears at some heights. "Your map" becomes a close button when its popover is open. | 0:06, 1:21, 2:26 | The top row never changes. The sheet closes by dragging down or by the back pill. |
| 7 | **P1. "Your map" is not the user's map.** It opens account, appearance and a server row. The avatar on the right is a second account entry. Saved places sit behind an unlabelled bookmark. | 2:26 | Left: the user's map (saved, lists, layers). Right: app settings and account. Debug rows only in debug builds, under settings. |
| 8 | **P1. Two names and two pins.** The sheet and input say "Ask Maps" with the brand pin. The figure above is Mira, who is that pin. | every frame | One name in the interface. Remove the pin from the input. The decision is the product owner's (section 6). |
| 9 | **P1. Map marks collide.** Bakery pins and labels stack on each other. "#2 Tipsy Village" and "#3 GNIHTON" overlap. Overlap and Areas views are zoomed out to the whole city, so the reach shapes are a few millimetres wide. | 1:07, 1:27, 1:41 | Collision handling for labels; rank decides which label shows. Each view sets its own camera: fit the shapes it draws, with padding. |
| 10 | **P1. The same text appears twice.** The meetup layer repeats the thread's paragraphs under its chips. | 1:26 | The layer shows its canvas (views, places, blocks). Its history is a short log, not the answer again. |
| 11 | **P1. The input is one line.** A long question scrolls out of view while typing. | 0:59 | The input grows to three lines, then scrolls. |
| 12 | **P2. Thread controls overlap content.** The jump-to-latest arrow sits on top of text and on the card title. The card is half under the bar. Reply and a menu show under "Thinking" before there is anything to reply to. | 1:13, 2:02, 0:22 | Arrow in a clear zone above the bar. Bottom inset equal to the bar. Message actions appear when the message is complete. |
| 13 | **P2. Leaving a layer loses the place.** Back from the meetup clears the map and lands mid-scroll. The collapsed bar shows the question, not the thing made from it. | 2:02 | Back returns to the same scroll position with the card in view. The collapsed bar names the current thing. |
| 14 | **P2. Jumping between cities is abrupt.** A blank map shows for about a second between San Francisco and New York. Bakery pins from the first answer appear only when the second question is sent. | 1:12, 1:06 | Hold the old tiles until the new ones load. Marks belong to an answer and show when it is in view. |
| 15 | **P2. Map labels run under the clock and the lens.** | 1:49, 2:09 | A soft fade at the top of the map; camera padding keeps subject labels out from under the lens. |

Not shown in the recording, so not judged: tapping Mira, voice, the bookmark and home buttons, any long-running work, approvals, the shelf, and whether anything is remembered between sessions. The debug page records one dropped stream during the session; whether the user saw an effect is unknown.

## 4. Spec for the next wave

Goal: one question, asked once, is answered on the screen without the user hunting for it. Everything in this wave serves that sentence. Long-running work, approvals and the shelf stay out of this wave.

### 4.1 Camera contract (fixes 1, 9, 14, 15)

1. The *visible map rectangle* is the screen minus the top 150 pt (status bar, lens, label slot) and minus the sheet at its current height.
2. When an answer has marks, the camera fits all of them inside that rectangle with 24 pt padding. With one mark it centres on it at street zoom.
3. The camera moves once per answer, when the first result is known. It does not move while the user is touching the map.
4. When the sheet changes height, the camera re-fits only if a mark would otherwise be covered.
5. While an answer is active, the recentre button has two steps: first tap fits the answer, second tap goes to the user's location.
6. Each map view in a layer (Overlap, Areas, Routes) declares its own bounds and fits them.

Accepted when: for the two questions in the recording, every result mark is fully visible within one second of the answer completing, at Peek and at Half, on the smallest supported phone.

### 4.2 Sheet contract (fixes 2, 5, 6)

1. Heights: Rest (bar only), Peek (about 230 pt), Half (about 430 pt), Full (to 150 pt from the top). No others.
2. Tapping the input focuses it and shows one row of suggestion chips above the bar. No sheet opens.
3. Sending a question raises the sheet to Peek and keeps it there through thinking and the answer. Mira never lowers the sheet herself.
4. Dragging is the only way the height changes after that, apart from rule 3.
5. The top row is fixed. No control in it is replaced by a close button.

Accepted when: a first-time user can say, after one question, where the answer will appear next time.

### 4.3 The answer (fixes 3, 10, 12)

For a place question ("best bakeries near…"):
1. Up to five photo callouts on the map, ranked. The top one is selected.
2. In the sheet at Peek: one line from Mira (at most one line), the selected place's name, three blocks (walk time, open state, one reason), and one primary button.
3. Tapping a callout swaps the blocks. Nothing scrolls.
4. Refinements are chips above the bar. The paragraphs are gone. A "More about these" row opens the long form at Full for those who want it.
5. No message actions until the answer is complete.

For a plan question (the three-way meetup): the answer is the meetup layer directly, opened at Peek on the Routes view, since that is the view that answers "where". The thread keeps a one-line capsule for it.

Accepted when: the bakery answer needs no scrolling and no reading of a paragraph to pick a place and start walking.

### 4.4 Attention (fixes 2, 4)

1. A whisper is never the delivery of something the user asked for.
2. No unprompted whisper while an answer is on screen, while the sheet is above Rest, or within 60 seconds of the user's last question. The 60 seconds is an assumption to tune.
3. Whisper copy is written to one line at the narrowest width. If it does not fit, it is not sent.

### 4.5 Frame and naming (fixes 6, 7, 8)

1. Top left: the user's map (saved places, lists, layers). Top centre: Mira. Top right: app settings and account. History moves into Mira's drawer.
2. One name for the assistant across the sheet title, the input placeholder and the figure.
3. No brand pin inside the input.
4. Debug and server rows appear only in debug builds, under app settings.

### 4.6 Smaller items

- Input grows to three lines (11).
- Back from a layer restores scroll position and map marks of the thread entry in view (13).
- Hold old map tiles until new ones load (14).
- Top fade on the map under the status bar (15).

### 4.7 Out of this wave

Voice, proactive finds beyond the rules in 4.4, long-running work and its reports, approvals, the shelf, reply-to-continue, and Mira's drawer contents. These are specified in the spec and wait until one question is answered well.

## 5. Order of work

1. Camera contract. It is the largest loss in the recording and is independent of everything else.
2. Sheet contract and the rule that an asked-for answer is never a whisper.
3. The place answer with callouts and blocks.
4. The plan answer opening on Routes.
5. Frame and naming.
6. Smaller items.

## 6. Decisions needed

1. **The name.** "Ask Maps" or Mira in the interface. This review recommends one name everywhere and takes no side; it depends on brand.
2. **Paragraphs.** Whether the long text survives as an optional "More about these" at Full, or goes entirely. This review recommends keeping it one tap away for M2 and measuring how often it is opened.
3. **What tapping Mira does in M2.** The recording never shows it. The spec says it opens her drawer; if the drawer is not in this wave, the tap needs an interim answer.
