# Mira: brief for the build agent, next wave

Copy the block below to the agent. It is the same text as the published "Mira agent brief" page. Background and evidence: `2026-11-27-m1-demo-review-and-next-wave.md`.

```text
You are continuing the Mira iOS prototype (build "M1 Cycle5B"). A design review of the latest recording found that the main task fails: the user asks a question and then has to hunt for the answer.

GOAL FOR THIS WAVE
A question asked once is answered on the screen, without the user hunting for it.

GROUND RULES
- Work the packages below in order. Finish and verify one before starting the next.
- Do not add features outside these packages. Out of scope this wave: voice, proactive finds beyond the rules in package 4, long-running work and its reports, approvals, reply-to-continue.
- Keep what already works: the figure and its states, the Routes view of the meetup, the back arrow on the meetup page, the placeholder "Ask about these".
- The model is one thread. Do not build levels, breadcrumbs, a home button or a close button on the thread. Package 1 is the whole navigation.
- ASSISTANT_NAME is a single constant. Default it to "Mira". Every place the interface names the assistant reads from it. Do not hard-code "Ask Maps" anywhere.
- After each package, record the two test questions on the smallest supported iPhone and report: what changed, what you could not do, and anything you decided on your own.

TEST QUESTIONS (use these for every acceptance check)
Q1: "Best bakeries near Ferry Building" asked while the map shows San Francisco.
Q2: "Coffee meetup between Union Sq, Christopher St and Bedford Ave" asked straight after Q1.


PACKAGE 1: NAVIGATION

THE MODEL IN ONE LINE
There is one thread. Everything Mira says, shows or makes is an item in it. A complex thing folds into a card; tapping the card opens a page. That is the only place you can go.

THE FRAME (five fixed parts; none of them ever moves, hides or turns into another control)
1. Top left, "Your map": the user's own things. Saved places, lists, map layers.
2. Top centre, the lens (Mira): her panel. Earlier topics, what she may see, pause.
3. Top right, the avatar: account and app settings. Appearance lives here. Server and debug rows in debug builds only.
4. Under the lens, the whisper slot: one line from Mira when she speaks first. Nothing else ever appears there. No breadcrumb, no close button, no parent link.
5. Bottom, the bar: plus, text input, microphone. Nothing else.

REMOVE OR MOVE
- Remove the history button from the top row. History is "Earlier" in Mira's panel.
- Remove the home and bookmark buttons from the bar.
- Remove the brand pin from inside the input.
- The avatar and "Your map" must never turn into a close button.
- Move account, appearance and the server row out of "Your map" into the avatar's panel.

TWO PLACES ONLY
- The thread. It lives in the sheet over the map. This is where the user always is.
- A page. Opened by tapping a card in the thread. A page is a mini app: a plan, a comparison, a place in depth.
There is no third place. A page never opens another page. A link inside a page that would go deeper replaces the page's content and the same back control returns to the thread.

ONE WAY BACK
- A page has one back control, top left of the page, and a downward swipe does the same. Both return to the thread at the exact scroll position, with the card in view.
- Panels ("Your map", Mira's panel, the avatar panel) close by tapping outside them.
- There is no home button and no close button on the thread. Dragging the sheet down to Rest shows the map. That is all "home" means.

THE SHEET IS THE THREAD (one continuous scroll, four resting heights)
- Rest: the bar only.
- Peek: about 230 pt. Shows the newest item.
- Half: about 430 pt. The newest item and what came before.
- Full: up to 150 pt from the top.
It is the same list at every height. Raising the sheet only reveals more of it. There is no separate "answer view" and "thread view".
Who may change the height:
- The user, by dragging. Always.
- The app, in one case: sending a question raises the sheet to Peek so the answer is visible.
- The app never lowers the sheet.

SMOOTHNESS (this is the quality bar for the wave)
- The sheet follows the finger with no jump, and settles on the nearest height with one spring. No intermediate heights.
- New items grow in at the bottom; the list does not jump. If the user has scrolled up, do not pull them down; show a small "New" marker instead.
- The map never moves while a finger is on the map or on the sheet.
- Opening a page: the card expands into the page. Closing: the page shrinks back into its card. Same element, no cut.
- Nothing on screen changes position except the thing the user is touching or the thing that is arriving.

TOPICS (grouping without navigation)
- Mira gives every item a topic. Items on the same subject share one, even hours apart.
- In the thread, a topic shows as a small heading above its first item ("Bakeries near Ferry Building").
- An older topic folds to one line: its title and a count ("Coffee meetup · 4 items"). Tap to unfold in place. Nothing navigates.
- The user never creates, names or files topics. They can long-press an item and choose "Move to another topic" or "Start a new topic" if Mira grouped it wrongly.
- "Earlier" in Mira's panel lists topics. Tapping one scrolls the thread to it.

WHAT EVERY TAP DOES
- Tap the input: focus it, keyboard up, one row of suggestion chips above the bar. No sheet opens.
- Send: the question appears in the thread, the sheet rises to Peek.
- Tap a place on the map or in an item: select it in place. The item updates. Nothing opens.
- Tap a card: its page opens.
- Tap a whisper: the thread scrolls to that item at Peek.
- Tap the lens: Mira's panel.

ACCEPT WHEN
- Recording Q1 then Q2: the top row and the bar are identical in every frame apart from Mira's state. Nothing but a whisper ever appears under the lens.
- The user is never more than one back away from the thread.
- Dragging the sheet from Rest to Full and back is one continuous list with no reload, jump or change of content.


PACKAGE 2: CAMERA

PROBLEM SEEN
After Q1 the camera moved to open water. The result pins sat under the bottom bar. The user spent about 30 seconds looking for them (0:25 to 0:55 in the recording).

RULES
1. The visible map rectangle = the screen, minus the top 150 pt, minus the sheet at its current height.
2. When an answer has marks, fit all of them inside that rectangle with 24 pt padding. One mark: centre it at street zoom.
3. Move the camera once per answer, when the first result is known. Never before. Never while a finger is on the map.
4. When the sheet height changes, re-fit only if a mark would be covered.
5. While an answer is active, the recentre button has two steps: first tap fits the answer, second tap goes to the user's location.
6. Each map view on a plan page (Overlap, Wider area, Areas, Routes) declares its own bounds and fits them. Do not show a whole city to draw a shape a few blocks wide.
7. When jumping to another city, keep the old tiles on screen until the new ones have loaded. No blank frame.
8. Marks belong to an item. Show them when that item is in view and hide them when it is not.
9. Labels must not overlap. When two collide, the higher-ranked result keeps its label and the other shows a dot until selected.

ACCEPT WHEN
- For Q1 and Q2, every result mark is fully visible within one second of the answer finishing, at Peek and at Half.
- No frame of the recording shows only water or a blank map.
- No two labels overlap in any of the four meetup views.


PACKAGE 3: WHAT MIRA CAN EMIT

PROBLEM SEEN
Both answers arrived as paragraphs with facts buried in sentences. The sheet collapsed and a banner said "Tap to view". The meetup page repeated the same paragraphs.

THE IDEA
Mira does not write a reply. She uses tools, and each tool call becomes one item in the thread. The client renders items; it does not decide what kind of answer to give.

THE TOOLS (the agent's whole vocabulary for reaching the user)
1. say(text): a short message. One or two sentences. Plain text.
2. show(kind, data): a small piece of UI that sits in the thread, complete without tapping. Kinds for this wave:
   - places: up to five places. Renders callouts on the map and, in the thread, the selected place's name with three blocks (walk time, open state, one reason) and one primary button.
   - fact: one large value with a label ("Sunday · 11 to 4").
   - choice: two to four options as buttons.
3. open(kind, title, summary, data): something too big to sit in the thread. Renders in the thread as one card (title, one-line summary, a small preview). Tapping the card opens its page. Kinds for this wave:
   - plan: the meetup. Its page has the map with view chips (Overlap, Wider area, Areas, Routes), the places, and three blocks for the selected one.
   - place: one place in depth.
   - text: the long written version, for anyone who wants the paragraphs.
4. mark(places, fit): what to draw on the map for an item, and what the camera should fit. Attached to a show or an open.
5. status(text): a transient line while she works ("Searching bakeries near Ferry Building"). Not an item; it disappears.
6. whisper(text, item): one line under the lens, pointing at an item. Package 4 says when it is allowed.
Every tool call carries a topic (package 1).

RULES
- An answer is usually one say plus one show. Q1 = say("Five open now, all within six minutes.") + show(places) + mark.
- Use open only when the thing has its own controls or more than fits at Peek. Q2 = say(one line) + open(plan), with the card's preview showing the Routes view.
- The same words never appear twice. If the detail is in a show or a page, say does not repeat it.
- Paragraphs exist only inside open(text). Never in say.
- Tapping a place inside a show changes the selection in place. It does not open anything.
- The map shows the marks of whichever item is in view. Scroll to an older item and its marks return.
- On a page, the bar stays. Its placeholder becomes "Ask about this". What the user sends there appears in the thread, quoting the card.
- No Reply, Copy or menu controls on an item until it is complete.
- The input grows to three lines, then scrolls.
- The jump-to-latest control sits in a clear strip above the bar and never covers an item.

ACCEPT WHEN
- For Q1, a user can pick a place and start walking without scrolling, without opening anything and without reading a paragraph.
- For Q2, the thread shows one line and one card; one tap shows three routes to one cafe; one back returns to the same spot.
- Every item on screen can be traced to exactly one tool call.


PACKAGE 4: ATTENTION

PROBLEM SEEN
The answer to Q1 was delivered as a banner under the lens. Ten seconds later a second banner about a nearby gallery appeared, cut off mid-word.

RULES
1. A whisper is never how an asked-for answer is delivered. Asked-for answers are items in the thread, visible at Peek.
   A whisper always points at an item that is already in the thread; tapping it scrolls there.
2. No unprompted banner while an answer is on screen, while the sheet is above Rest, or within 60 seconds of the user's last question. Treat 60 as a tunable constant.
3. A banner is one line at the narrowest phone width. If the text does not fit, do not send it. Never truncate.
4. At most one banner at a time. It dismisses itself.

ACCEPT WHEN
- Recording Q1 then Q2 shows no banner at all.
- Leaving the map idle at Rest for two minutes shows at most one banner, complete, on one line.


REPORT FORMAT (after each package)
1. One recording of Q1 then Q2 on the smallest supported iPhone.
2. A list: rule by rule, done / not done / done differently, with one line of why.
3. Anything you decided that these instructions did not cover.
4. Anything in these instructions that conflicts with the existing code or with each other. Stop and ask before guessing on: the assistant's name, what the lens panel contains beyond "Earlier" and "Pause", and any answer that seems to need a tool or a kind not listed in package 3.
```
