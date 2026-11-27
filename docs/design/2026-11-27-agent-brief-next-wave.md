# Mira: brief for the build agent, next wave

Copy the block below to the agent. It is the same text as the published "Mira agent brief" page. Background and evidence: `2026-11-27-m1-demo-review-and-next-wave.md`.

```text
You are continuing the Mira iOS prototype (build "M1 Cycle5B"). A design review of the latest recording found that the main task fails: the user asks a question and then has to hunt for the answer.

GOAL FOR THIS WAVE
A question asked once is answered on the screen, without the user hunting for it.

GROUND RULES
- Work the packages below in order. Finish and verify one before starting the next.
- Do not add features outside these packages. Out of scope this wave: voice, proactive finds beyond the rules in package 4, long-running work and its reports, approvals, reply-to-continue.
- Keep what already works: the figure and its states, the Routes view of the meetup, the back arrow on the meetup layer, the scoped placeholder "Ask about these".
- ASSISTANT_NAME is a single constant. Default it to "Mira". Every place the interface names the assistant reads from it. Do not hard-code "Ask Maps" anywhere.
- After each package, record the two test questions on the smallest supported iPhone and report: what changed, what you could not do, and anything you decided on your own.

TEST QUESTIONS (use these for every acceptance check)
Q1: "Best bakeries near Ferry Building" asked while the map shows San Francisco.
Q2: "Coffee meetup between Union Sq, Christopher St and Bedford Ave" asked straight after Q1.


PACKAGE 1: NAVIGATION

THE FRAME (five fixed parts; none of them ever moves, hides or turns into another control)
1. Top left, "Your map": opens the user's own things. Saved places, lists, map layers, offline maps.
2. Top centre, the lens (Mira): opens Mira's panel. Earlier conversations, what she is doing now, what she may see, pause.
3. Top right, the avatar: opens account and app settings. Appearance (Light / Dusk) lives here. Server and debug rows appear here in debug builds only.
4. Under the lens, the label slot: shows where you are when you are one layer down, as "< parent | here  x". Empty at the top layer.
5. Bottom, the bar: plus, text input, microphone. Nothing else.

REMOVE OR MOVE
- Remove the history button from the top row. History is "Earlier" inside Mira's panel.
- Remove the home and bookmark buttons from the bar. Saved places are in "Your map". Going home is the x in the label slot.
- Remove the brand pin from inside the input.
- The avatar must never turn into a close button. "Your map" must never turn into a close button. Panels close by tapping outside them or tapping their own button again.
- Move account, appearance and the server row out of "Your map" into the avatar's panel.

THREE LEVELS (the only places a user can be)
- Level 0, the map. The sheet is at Rest or shows the thread.
- Level 1, an answer or a thing (a set of results, a place, a plan such as "3-way coffee meetup").
- Level 2, something inside it (one place from the results, one view of the plan).
Do not create a fourth level. A tap that would go deeper replaces Level 2.

BACK, CLOSE, HOME (three different actions, never mixed)
- Back = up one level. A back pill is the first item in the chip row above the bar, on Levels 1 and 2. It shows a chevron and the parent's exact title. The iOS edge swipe does the same thing.
- Close a panel = tap outside it. Panels are "Your map", Mira's panel, the avatar panel. Closing a panel never changes the level.
- Home = the x in the label slot. Returns to Level 0 with the sheet at Rest. Whatever was open keeps running and stays in the thread.
- Dragging the sheet changes its height only. It never changes the level.

THE SHEET (one sheet, four heights, no others)
- Rest: the bar only.
- Peek: about 230 pt. One answer.
- Half: about 430 pt. A list, a place, a plan.
- Full: up to 150 pt from the top. The whole thread.
Who may change the height:
- The user, by dragging. Always.
- The app, in exactly two cases: sending a question raises the sheet to Peek; tapping the back pill at Level 1 returns to the height the user had before.
- The app never lowers the sheet on its own.

WHAT EVERY TAP DOES
- Tap the input: focus it, keyboard up, one row of suggestion chips above the bar. No sheet opens.
- Send: sheet goes to Peek, level becomes 1, label slot shows "< Thread | <the question>  x".
- Tap a result on the map or in the sheet: select it in place. Tap it again, or its "More" button: Level 2.
- Tap a card in the thread (such as "3-way coffee meetup"): Level 1 for that thing.
- Tap the lens: Mira's panel drops from the top. Tap the lens again or outside: it closes.
- Pull the sheet to Full at Level 0: the thread.

ACCEPT WHEN
- Recording Q1 then Q2: the top row and the bar look identical in every frame, apart from the label slot text and Mira's state.
- From any screen, one tap on the x reaches the map at Rest, and one tap on the back pill reaches the parent.
- No control changes its icon to a close button anywhere.


PACKAGE 2: CAMERA

PROBLEM SEEN
After Q1 the camera moved to open water. The result pins sat under the bottom bar. The user spent about 30 seconds looking for them (0:25 to 0:55 in the recording).

RULES
1. The visible map rectangle = the screen, minus the top 150 pt, minus the sheet at its current height.
2. When an answer has marks, fit all of them inside that rectangle with 24 pt padding. One mark: centre it at street zoom.
3. Move the camera once per answer, when the first result is known. Never before. Never while a finger is on the map.
4. When the sheet height changes, re-fit only if a mark would be covered.
5. While an answer is active, the recentre button has two steps: first tap fits the answer, second tap goes to the user's location.
6. Each map view of a plan (Overlap, Wider area, Areas, Routes) declares its own bounds and fits them. Do not show a whole city to draw a shape a few blocks wide.
7. When jumping to another city, keep the old tiles on screen until the new ones have loaded. No blank frame.
8. Marks belong to an answer. Show them when that answer is on screen and hide them when it is not.
9. Labels must not overlap. When two collide, the higher-ranked result keeps its label and the other shows a dot until selected.

ACCEPT WHEN
- For Q1 and Q2, every result mark is fully visible within one second of the answer finishing, at Peek and at Half.
- No frame of the recording shows only water or a blank map.
- No two labels overlap in any of the four meetup views.


PACKAGE 3: THE ANSWER

PROBLEM SEEN
Both answers arrived as paragraphs, with rating, price and hours inside sentences, and nothing in the text was tied to a pin. The sheet then collapsed and a banner said "5 places found - Tap to view". The meetup layer repeated the same paragraphs.

FOR A PLACE QUESTION (Q1)
1. On the map: up to five callouts, ranked, each with a photo, the name and one short line. The top one is selected.
2. In the sheet at Peek, in this order:
   - one line from the assistant, never more than one line
   - the selected place's name
   - three blocks side by side: walk time, open state, one reason to go
   - one primary button: "Walk there"
3. Tapping another callout swaps the name and blocks in place. Nothing scrolls.
4. Chips above the bar hold filters and assumptions only ("Open now", "Under 10 min"). Actions are buttons, never chips.
5. A row "More about these" opens the long text at Full. The paragraphs appear nowhere else.

FOR A PLAN QUESTION (Q2)
1. Open the plan itself at Peek, on the Routes view. Do not show paragraphs first.
2. The thread keeps one card for the plan. Do not repeat the plan's text in the thread.
3. Inside the plan: the four view chips, the place chips, three blocks for the selected place. No repeated paragraphs.

WHILE SHE WORKS
- Keep the status line in words ("Searching bakeries near Ferry Building").
- Show no Reply, Copy or menu controls until the answer is complete.

IN THE THREAD
- The jump-to-latest arrow sits in a clear strip above the bar and never covers text or a card.
- The last item is fully visible above the bar.
- Returning from a plan restores the scroll position with its card in view.
- The input grows to three lines, then scrolls.

ACCEPT WHEN
- For Q1, a user can pick a place and start walking without scrolling and without reading a paragraph.
- For Q2, the first thing on screen after the answer is three routes to one cafe.
- The words of the answer appear once, not twice.


PACKAGE 4: ATTENTION

PROBLEM SEEN
The answer to Q1 was delivered as a banner under the lens. Ten seconds later a second banner about a nearby gallery appeared, cut off mid-word.

RULES
1. A banner under the lens is never how an asked-for answer is delivered. Asked-for answers are in the sheet.
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
4. Anything in these instructions that conflicts with the existing code or with each other. Stop and ask before guessing on: the assistant's name, what the lens panel contains beyond "Earlier" and "Pause", and whether paragraphs survive behind "More about these".
```
