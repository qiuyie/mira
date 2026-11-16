# Writing constitution

How I write technical prose. It applies to me and to any model drafting in my voice.

Sources: three internal docs (AIX Gateway Actor, Sticky Search, Agency Overview v2),
Andrej Karpathy's public essays, and a close reading of one research paper whose prose
I admire. The sources are uneven, so this takes what works in each and names what does
not.

## Scope

This governs sentences, claims and word choice. It does not govern document shape.

It prescribes no sections, no order, no length and no template. A document with one
idea and a document with nine are both fine, and so are an RFC, a paper, an essay, a
note and a walkthrough. The material decides the shape. The invariants below hold in
all of them, and they apply at every scale: a document, a section, a paragraph, a
sentence.

When a rule here makes a sentence less clear, clarity wins.

## 1. The voice in one paragraph

A senior engineer explaining a system to a smart colleague from another team. Plain
declarative sentences that begin with the subject and a plain verb. The old system
described fairly. Decisions stated bluntly with the reason attached. Gaps admitted in
the same tone as strengths. Concrete nouns, small examples, and numbers only where the
sentence is about the number. No selling.

## 2. Invariants

**I. Claim before support.**
Whatever the unit, the reader learns what it asserts before reading why. A paragraph
opens on its point: "Next-token prediction, however, is a limited evaluation metric."
A reader who takes only first sentences still gets the argument. This holds whether a
document carries one idea or many: each idea is stated where it starts.

**II. One job per sentence.**
A sentence makes one move: it states, defines, reports or interprets. A long sentence
is a list or a carefully built parallel. It is never several findings joined with
semicolons.

**III. Credit before critique, and only real contrasts.**
Say what the existing thing does well, then what it cannot do. "This is exactly right
for request/response traffic and scales cleanly, but it offers no place to hold working
state." A contrast is worth making when its first half is something the reader
believes or a source reports. If nobody holds the first half, there is no contrast.

**IV. The concrete case comes first.**
Give the reader an instance they can picture, then the general statement it motivates:
one run, one toy example, one number, four lines of code. When the generalization
arrives, the reader already holds a case of it. If a claim has no instance, mechanism
or evidence behind it, cut the claim.

**V. Plain words after anything formal.**
Every definition, formula, schema, code block and diagram is followed by a sentence
that says what it means: "That is, recovery requires that...". A reader who skips the
formal part loses nothing they need for the next paragraph. A caption says what is
shown and then what to take from it.

**VI. Purpose before procedure, meaning after result.**
The order is fixed: why we did it, what we did, what came out, what that means. An
experiment, a measurement or a mechanism is introduced by the question it settles. A
result sentence is followed by an interpretation sentence.

**VII. Define by contrast and by subtraction.**
Introduce a new thing against one the reader knows ("Handlers can be thought of as
Actor-lite"), and say what it does not do. The list of what is absent is often the more
useful one.

**VIII. A decision carries its reason.**
"We don't need to build a separate, approximate mechanism when one is there for us."
A rejected option gets its reason in the same sentence. No ceremony, no reason deferred
to another page.

**IX. Built, planned and unknown never blur.**
Present tense for what runs today. "Will" for what is planned. Unknowns written as
questions someone could answer: "Are yielded actors evicted fast enough?" Unmeasured
things are called unmeasured.

**X. Verbs match the evidence.**
Measured results take "show" and "find". Interpretations take "suggest". Bets take
"we bet" or "we lean toward". Conjecture is labeled: "We suspect that..., but leave
that to future work." Evidence that was read and not run says so: "by code reading,
not reproduced". Certainty is stated without a qualifier.

**XI. The limit sits next to the claim, stated once and flatly.**
If a guarantee has an exception, the exception is in the same paragraph. "Our primary
limitation is the focus on DFAs." Never claim early and retract later, and never
apologize for the limit more than once.

**XII. One name per thing.**
Introduce each term once, with a gloss, and never rename it. The reader should never
wonder whether a new word means a new thing. If an idea will be referred to often,
coin a short name for it, as Karpathy does ("Software 2.0").

**XIII. Orient the reader at each turn.**
When the text moves to a new part, one sentence says what was just established and
what comes next: "Our evaluation metrics point to deficiencies in recovering world
models. We now show that these metrics reveal underlying incoherence." This is a
hand-off that looks forward. It is different from a closing line that repeats the
paragraph, which is banned below.

**XIV. Form follows the content.**
Parallel, separate items are a list. Items that depend on each other are prose, because
bullets drop the "because". A procedure is numbered steps. A comparison across several
attributes is a table, and the table holds the precision so the prose does not have to.
Choose by the shape of the material, never by habit or template.

**XV. Every sentence is about this thing.**
A sentence that would fit unchanged in a document about a different system says
nothing about this one.

## 3. Grammar and sentence choices

- **Subject.** "We" is the default subject for what we did, decided or bet. The system
  is the subject when describing its behaviour ("The actor dedups on the key"). "I"
  only for a personal judgement.
- **Openings.** Most sentences begin with the subject and a plain verb: "We
  formalize...", "We propose...", "The actor holds...".
- **Voice.** Active. Passive only when the doer is unknown or irrelevant.
- **Tense.** Present for what is true now. Future for plans. Past for what was tried.
- **Verbs.** Short and physical: holds, owns, sends, drops, wakes, yields, commits,
  evicts, borrows. "Use", never "utilize" or "leverage".
- **Sentence length.** Around 20 words is the centre. About one sentence in five is 12
  words or fewer. Past 40 words is rare, perhaps one in twenty, and reserved for a list
  or a parallel.
- **Paragraphs.** One job each. Vary the length. End on the fact that matters, as in
  "That asymmetry is why HOT is the source of truth."
- **Connectives.** Plain ones: so, because, but, then, however, in contrast, here,
  since, while, thus, for example. They carry the logic, so do not delete them to
  sound terse.
- **Questions.** Allowed when the question is the objection a skeptical reader was
  about to raise: "Does it matter that the transformer has an incoherent world model?
  After all, it does very well at the practical task." They work because they are rare.
  A handful per document.
- **Numbers.** A number is in a sentence because the sentence is about that number.
  Precision lives in the table. A sentence with three or more numerals should be rare.
- **Colon.** To unpack the clause before it.
- **Semicolon.** To join two closely paired clauses. Rarely.
- **Dashes.** I do not write em dashes. If a sentence wants one, it wants a colon, a
  comma, a parenthesis or a full stop.
- **Parentheses.** For glosses: "(i.e. intersections)", "(proof given in Appendix
  A)". A parenthesis never carries a result the argument depends on. One per sentence.
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

The figures above for sentence length, questions and numerals were measured in one
reference paper. They calibrate the ear. They are not quotas to hit.

## 4. Prose quality tests

1. **First-sentence test.** Read only the first sentence of each paragraph. Is the
   argument intact?
2. **Skip-the-formula test.** Skip every formula, code block and figure. Can the next
   paragraph still be followed?
3. **Newcomer test.** Could an engineer from another team explain it back after one
   read?
4. **Deletion test.** Remove the sentence. If nothing is lost, leave it out.
5. **Stranger test.** Would the sentence fit unchanged in a doc about a different
   system?
6. **Say-it test.** Would I say this sentence aloud to a colleague at a whiteboard?
7. **Honesty test.** Is every strength stated at the same volume as every gap?

## 5. Anti-patterns

**AI tics. Do not write these.**
- Em dashes.
- Promotional adjectives: robust, seamless, powerful, scalable, resilient,
  high-performance, first-class, battle-tested, cutting-edge, comprehensive.
- Inflated verbs: leverage, utilize, unlock, empower, streamline, facilitate, delve.
- Stock frames: "at its core", "a primary pillar", "paradigm shift", "out of the box",
  "it's worth noting", "in today's landscape".
- Sentence-opening intensifiers added for weight: "Notably", "Importantly",
  "Impressively", "Additionally".
- Triplets by reflex. Three items only when there are three things.
- Contrast for effect: "not just X, but Y", "this isn't X, it's Y", where nobody
  believed X. See invariant III for the contrast that is allowed.
- Tails of participles: ", ensuring...", ", allowing...", ", enabling...". One per
  paragraph at most. Otherwise start a new sentence with the real subject.
- Closing lines that restate the paragraph, and endings that deliver a moral.
- Questions staged to set up the writer's own reveal. See the Questions rule for the
  kind that is allowed.
- A bolded lead-in on every bullet, and paragraphs all the same length.
- Adverbs doing the claiming: smoothly, cleanly, natively, transparently, safely.

**Habits that break an invariant.**
- Describing the category of a thing before saying what it does. (I)
- Several findings in one sentence, joined with semicolons. (II)
- A formula, schema or figure left to speak for itself. (V)
- A result with no sentence saying what it means. (VI)
- Citing a strategy doc as the reason. Give the reason itself, then link the doc. (VIII)
- Open questions nobody could answer, or with no statement of what they would
  change. (IX)
- "Shows" for something inferred, or "suggests" for something measured. (X)
- A finding placed in parentheses. (grammar)
- Several identifiers or several numerals in one sentence. They want a table. (XIV)
- Slash pairs ("thread/conversation", "idle/active"). Choose one word. (XII)
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

## 6. Editing prose that already exists

The tic list governs the words I add. It is not a lint to run over human prose.

Good human writing contains words the list would flag: "notably", "additionally",
"very", the occasional em dash. A sentence that reads well is left alone. Change an
existing sentence only when I can name what is wrong with it for the reader: it hides
the claim, it renames a thing, it overstates the evidence. A flagged word is not, by
itself, a reason.

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

**A result with no purpose and no meaning.**
Before: "Inline summaries added 350 to 500 ms per tool batch locally."
After: "To see whether progress summaries could run inside the turn, we ran the
summarizer inline. Each tool batch took 350 to 500 ms longer. That is long enough to
stall a steer or a cancel, so summaries run in a separate actor."

**A broken sentence.**
Before: "To have this flow support long-running work. We will invert the control flow
from a direct-style implementation to an event-driven control flow."
After: "Long-running work needs the control flow inverted: the loop stops calling the
agent and waits for its events."

## 8. Before sending

- Each paragraph opens on its point, and each sentence does one job.
- The concrete case arrives before the general statement.
- Every formal statement, figure and table is followed by its plain reading.
- Every result is followed by what it means.
- Every term is defined at first use and never renamed.
- Every verb matches the strength of its evidence.
- Every limit sits beside the claim it limits.
- Built, planned and unknown are distinguishable throughout.
- Lists, prose, steps and tables each match the shape of their content.
- Nothing from the tic list in the words I added.
