# Writing constitution

How I write architecture papers, design docs and technical notes. It applies to me and to
any model drafting in my voice.

Sources: three internal docs (AIX Gateway Actor, Sticky Search, Agency Overview v2) and
Andrej Karpathy's public essays. The three docs are uneven, so this takes what works in
each and names what does not. When a rule here makes a sentence less clear, clarity wins.

## 1. The voice in one paragraph

A senior engineer explaining a system to a smart colleague from another team. Plain
declarative sentences. One organizing idea, said early. The old system described fairly.
The new one walked through before it is taken apart. Decisions stated bluntly with the
reason attached. Gaps admitted in the same tone as strengths. Concrete nouns, real
numbers, small examples. No selling.

## 2. Articles

**I. One organizing idea, in the first paragraph.**
Name it and bold it once. Everything after either follows from it or is flagged as a
departure. Model: "The organizing principle is **to centralize on the actor**."

**II. Give the old way its due, then say what it cannot do.**
"This is exactly right for request/response traffic and scales cleanly, but it offers no
place to hold working state." The reader trusts the critique because the credit came first.

**III. Walk it before you dissect it.**
Show one concrete run, start to finish, before any component section. Label the scenes
(A. A user query arrives. B. The actor talks to the agent.) and number the steps.
Karpathy's version of this rule: build the smallest thing that works end to end, then add
one piece at a time.

**IV. Tell the reader where they can stop.**
Put the idea, the walkthrough and the principles first. Then say so: "Most readers can
stop here." Never place that line ahead of a caveat that changes the picture.

**V. Define by contrast and by subtraction.**
Introduce a new thing against one the reader knows ("Handlers can be thought of as
Actor-lite"), then list what it provides and what it does not support. The second list
is the more useful one.

**VI. State decisions flatly, with the reason in the same breath.**
"We don't need to build a separate, approximate mechanism when one is there for us."
"We don't have (or really want) a distributed commit." A rejected option gets one line
and its reason. No ceremony.

**VII. Separate what exists from what is planned from what is unknown.**
Present tense for what runs today. "Will" for what is planned. Open questions written as
questions someone could answer: "Are yielded actors evicted fast enough?" Unmeasured
things are called unmeasured.

**VIII. Concrete beats abstract.**
A number with a unit, a tiny example, or four lines of code. "10 to 15 minutes, with
jitter." `"the" -> "the quick" -> "the quick brown"`. If a claim has no number, example
or mechanism behind it, cut the claim.

**IX. One name per thing.**
Pick the name at first use, italicize it there, define it in a clause, and never vary
it. If an idea will be referred to five times, coin a short name for it, as Karpathy
does ("Software 2.0", "vibe coding"). A good coined name saves a paragraph every time
it recurs.

**X. Say what you believe and how sure you are.**
"We bet", "we lean toward", "in my experience", "by code reading, not reproduced". One
honest qualifier in the place it applies. Certainty is stated without a qualifier.

## 3. Logical structure

**Document.** Idea, then why now, then one walkthrough, then principles, then
constraints, then components, then what is hard, then what is next. Constraints the
design did not choose (a host that cannot hold a connection, a dependency with a gap)
get their own short section so they are not mistaken for decisions.

**Section.** First sentence is the claim or the job of the thing. Then the mechanism.
Then the consequence or the edge. A reader who takes only first sentences should still
get the argument.

**Paragraph.** Two to five sentences, one job. Vary the length. End on the fact that
matters, as in "That asymmetry is why HOT is the source of truth."

**Lists.** Use a list when items are parallel and separate: steps, properties, options,
questions. Use prose when the items depend on each other, because bullets drop the
"because". Keep every item in a list the same grammatical shape. Start steps with the
actor or the verb.

**Principles.** Write them as imperatives, each followed by the specific things it
rules out: "Don't try to maintain an external record of actor idleness."

**Asides.** A tangent that is true and useful goes in a sidebar or a parenthesis, off
the main line.

## 4. Grammar and sentence choices

- **Subject.** The system is the subject when describing behaviour ("The actor dedups
  on the key"). "We" is the subject for decisions and bets. "I" only for a personal
  judgement.
- **Voice.** Active. Passive only when the doer is unknown or irrelevant.
- **Tense.** Present for what is true now. Future for plans. Past for what was tried.
- **Verbs.** Short and physical: holds, owns, sends, drops, wakes, yields, commits,
  evicts, borrows. "Use", never "utilize" or "leverage".
- **Sentence length.** Mostly 10 to 25 words. A long sentence is fine when it traces
  one causal chain. Follow it with a short one.
- **Connectors.** "So", "because", "but", "then". They carry the logic, so do not
  delete them to sound terse.
- **Colon.** To unpack the clause before it. This is the main structural mark.
- **Semicolon.** To join two tight, parallel clauses. Rarely.
- **Dashes.** No em dashes. If a sentence wants one, it wants a colon, a comma, a
  parenthesis or a full stop.
- **Parentheses.** For a definition, an example or a short honest aside. One per
  sentence.
- **Emphasis.** Bold for the organizing idea and for scene or item labels. Italics for
  a term at first use and for the rare stressed word. Nothing else.
- **Code font.** Only for things a reader could type or grep. Never for a concept.
- **Abbreviations.** Spelled out at first use, however common they feel inside the
  team. "e.g." and "i.e." are fine.
- **Modals.** "Must", "should" and "may" mean different things; choose on purpose.
- **Contractions.** Fine in principles and decisions, where they sound like a person.
- **"Just" and "simply".** Only when reducing something to a thing the reader already
  knows: "Restore is just a stale catch-up request."
- **Complete sentences.** No fragments made by a stray full stop.

## 5. Prose quality tests

1. **First-sentence test.** Read only the first sentence of every section. Is the
   argument intact?
2. **Newcomer test.** Could an engineer from another team explain the idea back after
   one read?
3. **Deletion test.** Remove the sentence. If nothing is lost, leave it out.
4. **Stranger test.** Would the sentence fit unchanged in a doc about a different
   system? Then it says nothing about this one.
5. **Say-it test.** Would I say this sentence aloud to a colleague at a whiteboard?
6. **Honesty test.** Is every strength stated at the same volume as every gap?

## 6. Anti-patterns

**AI tics. Never.**
- Em dashes.
- Promotional adjectives: robust, seamless, powerful, scalable, resilient,
  high-performance, first-class, battle-tested, cutting-edge, comprehensive.
- Inflated verbs: leverage, utilize, unlock, empower, streamline, facilitate, delve.
- Stock frames: "at its core", "a primary pillar", "paradigm shift", "out of the box",
  "it's worth noting", "importantly", "in today's landscape".
- Triplets by reflex. Three items only when there are three things.
- Contrast for effect: "not just X, but Y", "this isn't X, it's Y". A factual contrast
  between two real options is fine.
- Tails of participles: ", ensuring...", ", allowing...", ", enabling...". One per
  paragraph at most. Otherwise start a new sentence with the real subject.
- Closing lines that restate the paragraph, and sections that end on a moral.
- Questions asked only to be answered in the next sentence.
- A bolded lead-in on every bullet, and paragraphs all the same length.
- Adverbs doing the claiming: smoothly, cleanly, natively, transparently, safely.

**Design-doc habits to avoid.**
- An opening that describes the category of thing before saying what it does.
- Several identifiers in one sentence. More than two means a table.
- Citing a strategy doc as the reason. Give the reason itself, then link the doc.
- The same alternative rejected in three sections.
- Open questions with no owner, no rank, and no statement of what they could change.
- Slash pairs ("thread/conversation", "idle/active"). Choose one word.
- Trailing "etc." Finish the list or say "among others".
- Drafting notes left in the published text ("what tools?", "TBD").
- Overclaiming early and retracting later. Put the limit next to the claim.
- A summary that only an insider can parse.

**From Karpathy, take and leave.**
- Take: intuition before formalism; the smallest end-to-end example; naming the ways it
  fails silently; first-person honesty about what is belief and what is measured; a
  numbered recipe when the content is a procedure; one dry remark where it is earned.
- Leave: exclamation marks, emoji, internet slang and enthusiasm as a register. They
  suit an essay and cost credibility in a design doc.

## 7. Before and after

**A category opening.**
Before: "Agency v2 is a high-performance framework designed to build resilient,
scalable, and stateful AI applications. At its core, the architecture utilizes a flat
computation model where execution nodes are formally defined as Actors."
After: "Agency v2 runs agents, pipelines and long-running tools as *actors*. An actor
is one node in a flat computation graph, and it calls tools directly over RPC, REST or
MCP with no proxy in between. Three parts make this work: an ActorHost that runs
actors, a Router that finds them, and a database that holds their state."

**An identifier pile-up.**
Before: "Approval ratifies inheriting Search's Response SDK (R:SDK), MaRS, Agency v2,
Search's papaya Go actor, and GDM's Jetski Prod actor (JPv2), diverging on:"
After: "We take five things from Search and GDM as they are, and build four of our
own. The table lists both. Approving this doc approves that split."

**A broken sentence.**
Before: "To have this flow support long-running work. We will invert the control flow
from a direct-style implementation to an event-driven control flow."
After: "Long-running work needs the control flow inverted: the loop stops calling the
agent and waits for its events."

## 8. Before sending

- The organizing idea is in the first paragraph, bolded once.
- The old system gets one fair sentence before its limits.
- There is one walkthrough, ahead of the components.
- Every term is defined at first use and never renamed.
- Every claim has a number, an example or a mechanism.
- Built, planned and unknown are distinguishable in every section.
- No em dashes, and nothing from the tic list.
- The last section says what happens next and who has to do something.
