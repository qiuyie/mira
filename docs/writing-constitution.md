# Writing constitution

How I write technical prose. It applies to me and to any model drafting in my voice.

Sources: three internal docs (AIX Gateway Actor, Sticky Search, Agency Overview v2) and
Andrej Karpathy's public essays. The three docs are uneven, so this takes what works in
each and names what does not.

## Scope

This governs sentences, claims and word choice. It does not govern document shape.

It prescribes no sections, no order, no length and no template. A document with one
idea and a document with nine are both fine, and so are an RFC, an essay, a note and a
walkthrough. The material decides the shape. The invariants below hold in all of them,
and they apply at every scale: a document, a section, a paragraph, a sentence.

When a rule here makes a sentence less clear, clarity wins.

## 1. The voice in one paragraph

A senior engineer explaining a system to a smart colleague from another team. Plain
declarative sentences. The old system described fairly. Decisions stated bluntly with
the reason attached. Gaps admitted in the same tone as strengths. Concrete nouns, real
numbers, small examples. No selling.

## 2. Invariants

**I. Claim before support.**
Whatever the unit, the reader learns what it asserts before reading why. A paragraph
opens on its point. A section's first sentence says what the section is for. This holds
whether a document carries one idea or many: each idea is stated where it starts.

**II. Credit before critique.**
Say what the existing thing does well, then what it cannot do. "This is exactly right
for request/response traffic and scales cleanly, but it offers no place to hold working
state." The reader trusts the critique because the credit came first.

**III. An instance next to every abstraction.**
A general statement arrives with a concrete case: one run, one example, one number,
four lines of code. `"the" -> "the quick" -> "the quick brown"`. Karpathy's version:
show the smallest thing that works, then generalize. If a claim has no instance,
number or mechanism behind it, cut the claim.

**IV. Define by contrast and by subtraction.**
Introduce a new thing against one the reader knows ("Handlers can be thought of as
Actor-lite"), and say what it does not do. The list of what is absent is often the more
useful one.

**V. A decision carries its reason.**
"We don't need to build a separate, approximate mechanism when one is there for us."
"We don't have (or really want) a distributed commit." A rejected option gets its
reason in the same sentence. No ceremony, no reason deferred to another page.

**VI. Built, planned and unknown never blur.**
Present tense for what runs today. "Will" for what is planned. Unknowns written as
questions someone could answer: "Are yielded actors evicted fast enough?" Unmeasured
things are called unmeasured.

**VII. The limit sits next to the claim.**
If a guarantee has an exception, the exception is in the same paragraph. Never claim
early and retract later.

**VIII. One name per thing.**
Pick the name at first use, define it in a clause, and never vary it. If an idea will
be referred to often, coin a short name for it, as Karpathy does ("Software 2.0",
"vibe coding"). A good coined name saves a paragraph every time it recurs.

**IX. Belief is marked as belief.**
"We bet", "we lean toward", "in my experience", "by code reading, not reproduced". One
honest qualifier in the place it applies. Certainty is stated without one.

**X. Form follows the content.**
Parallel, separate items are a list. Items that depend on each other are prose, because
bullets drop the "because". A procedure is numbered steps. A comparison across several
attributes is a table. Choose by the shape of the material, never by habit or template.

**XI. Every sentence is about this thing.**
A sentence that would fit unchanged in a document about a different system says
nothing about this one.

## 3. Grammar and sentence choices

- **Subject.** The system is the subject when describing behaviour ("The actor dedups
  on the key"). "We" is the subject for decisions and bets. "I" only for a personal
  judgement.
- **Voice.** Active. Passive only when the doer is unknown or irrelevant.
- **Tense.** Present for what is true now. Future for plans. Past for what was tried.
- **Verbs.** Short and physical: holds, owns, sends, drops, wakes, yields, commits,
  evicts, borrows. "Use", never "utilize" or "leverage".
- **Sentence length.** Mostly 10 to 25 words. A long sentence is fine when it traces
  one causal chain. Follow it with a short one.
- **Paragraphs.** One job each. Vary the length. End on the fact that matters, as in
  "That asymmetry is why HOT is the source of truth."
- **Connectors.** "So", "because", "but", "then". They carry the logic, so do not
  delete them to sound terse.
- **Colon.** To unpack the clause before it. This is the main structural mark.
- **Semicolon.** To join two tight, parallel clauses. Rarely.
- **Dashes.** No em dashes. If a sentence wants one, it wants a colon, a comma, a
  parenthesis or a full stop.
- **Parentheses.** For a definition, an example or a short honest aside. One per
  sentence.
- **Emphasis.** Bold for the few phrases a skimming reader must not miss. Italics for a
  term at first use and for the rare stressed word. Nothing else.
- **Code font.** Only for things a reader could type or grep. Never for a concept.
- **Abbreviations.** Spelled out at first use, however common they feel inside the
  team. "e.g." and "i.e." are fine.
- **Modals.** "Must", "should" and "may" mean different things; choose on purpose.
- **Contractions.** Fine where they sound like a person deciding something.
- **"Just" and "simply".** Only when reducing something to a thing the reader already
  knows: "Restore is just a stale catch-up request."
- **Parallel items.** Every item in a list has the same grammatical shape.
- **Complete sentences.** No fragments made by a stray full stop.

## 4. Prose quality tests

1. **First-sentence test.** Read only the first sentence of each unit. Do you know what
   each one claims?
2. **Newcomer test.** Could an engineer from another team explain it back after one
   read?
3. **Deletion test.** Remove the sentence. If nothing is lost, leave it out.
4. **Stranger test.** Would the sentence fit unchanged in a doc about a different
   system?
5. **Say-it test.** Would I say this sentence aloud to a colleague at a whiteboard?
6. **Honesty test.** Is every strength stated at the same volume as every gap?

## 5. Anti-patterns

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
- Closing lines that restate the paragraph, and endings that deliver a moral.
- Questions asked only to be answered in the next sentence.
- A bolded lead-in on every bullet, and paragraphs all the same length.
- Adverbs doing the claiming: smoothly, cleanly, natively, transparently, safely.

**Habits that break an invariant.**
- Describing the category of a thing before saying what it does. (I)
- Citing a strategy doc as the reason. Give the reason itself, then link the doc. (V)
- Open questions nobody could answer, or with no statement of what they would
  change. (VI)
- Several identifiers in one sentence. More than two wants a table. (X)
- Slash pairs ("thread/conversation", "idle/active"). Choose one word. (VIII)
- The same point made in more than one place.
- Trailing "etc." Finish the list or say "among others".
- Drafting notes left in the published text ("what tools?", "TBD").
- A sentence only an insider can parse, where an outsider is the reader.

**From Karpathy, take and leave.**
- Take: intuition before formalism; the smallest working example; naming the ways a
  thing fails silently; first-person honesty about what is belief and what is
  measured; one dry remark where it is earned.
- Leave: exclamation marks, emoji, internet slang and enthusiasm as a register. They
  suit an essay and cost credibility in a design doc.

## 6. Before and after

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

## 7. Before sending

- Each unit states its point before its support.
- Every term is defined at first use and never renamed.
- Every claim has a number, an example or a mechanism.
- Every limit sits beside the claim it limits.
- Built, planned and unknown are distinguishable throughout.
- Lists, prose, steps and tables each match the shape of their content.
- No em dashes, and nothing from the tic list.
