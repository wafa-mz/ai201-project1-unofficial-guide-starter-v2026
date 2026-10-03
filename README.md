# The Unofficial Guide

This project uses the campus_life corpus and answers practical student questions such as whether the housing lottery is random, how long the Kestrel Commons line is at lunch, and how much laundry costs in a dorm. The system retrieves text from the corpus, enforces a relevance cutoff, and then answers using only the matched chunks and their source files.

---

# Unit 1

## What This Does

The system indexes the campus_life corpus, splits each document into complete thought chunks, and retrieves the closest chunks for a new question. It then uses a relevance gate to refuse off-topic questions before asking the model to answer, keeping the generated response grounded in the actual student posts. This makes it useful for questions like course advice, housing tradeoffs, and dining hall logistics instead of vague general-purpose advice.

## Chunking Strategy

**Chunk size:** 420 characters
**Overlap:** 80 characters

I picked a paragraph-aware chunker for this corpus because the documents are short student posts where a useful fact usually sits inside one paragraph or one clear sentence. A hard 800-character window would cut through student advice and bury the real claim in unrelated text, while a smaller window would split a rule or description into fragments. The overlap keeps neighboring chunks from losing continuity when a paragraph spans a sentence break, but it stays short enough that each chunk still reads as one coherent answer.

## Sample Chunks

**Chunk 1** — source: `admin_add_drop_deadline.txt` — produced by: `chunker.py::split_documents`

```
On the add/drop deadline You can add a course through the end of the second week. Dropping is a longer window — through the end of week six — but a drop after week two shows as a W on your transcript. Nothing anywhere on the registrar's site says this plainly, and students find out from each other.
```

**Chunk 2** — source: `course_biol_160_exams.txt` — produced by: `chunker.py::split_documents`

```
BIOL 160 Cell Biology — assessment Four unit tests and a cumulative final. Not curved. The unit tests come fast, roughly every three weeks; falling behind once is very hard to recover from.
```

**Chunk 3** — source: `course_math_220.txt` — produced by: `chunker.py::split_documents`

```
MATH 220 Linear Algebra I lived here my sophomore year. Format is chalk-and-talk lecture, weekly problem sets marked for correctness. Assessment: two midterms and a cumulative final. Curved to a b- median. Expect 6 to 8 hours a week, almost all of it on problem sets. The one piece of advice: the problem sets are the course; the lectures make sense afterwards rather than during.
```

**Chunk 4** — source: `dining_the_atrium_followup.txt` — produced by: `chunker.py::split_documents`

```
Re: The Atrium Adding to what people have said about The Atrium. The wait figure of no queue matches what I've seen. If you're trying to eat between classes, go before 11:45 and it's a different building entirely. Also worth saying: pickup is clean by 1:15 and not restocked again until the next morning. Nobody tells you this at orientation.
```

**Chunk 5** — source: `housing_innisfree_hall.txt` — produced by: `chunker.py::split_documents`

```
Laundry costs $1.75 wash, $1.75 dry, app-based. On noise: moderate; the building is L-shaped and the short wing is much quieter.
```

## Sample Answer

**Question:** Is the housing lottery random?

**Answer:** No. According to `admin_housing_lottery.txt`, the lottery is only random for rising sophomores; juniors and seniors are ordered by accumulated credit hours first, and only tie-breaks are random.

**My relevance cutoff:** 0.68

I set the cutoff at 0.68 after recording the best distance for five in-corpus questions and five clearly out-of-scope ones. The in-corpus questions clustered between 0.17 and 0.49, while the off-topic questions stayed between 0.82 and 0.93. The gap between 0.49 and 0.82 was wide enough that it refused off-topic questions without rejecting the questions the corpus actually covered.

| Question | In corpus? | Best distance |
|---|---|---|
| Is the housing lottery random? | Yes | 0.2541 |
| What are the actual wait times at Kestrel Commons around lunch? | Yes | 0.1688 |
| Which dorm is closest to the science quad? | Yes | 0.4880 |
| How much does laundry cost at Aldridge Hall? | Yes | 0.2376 |
| What do students say about the dining hall salad bar after 1:30? | Yes | 0.4034 |
| What is the capital of Mongolia? | No | 0.8246 |
| How do I change the oil in a diesel engine? | No | 0.9340 |
| Who won the 1994 World Cup? | No | 0.8859 |
| What is the recommended dosage of ibuprofen for a headache? | No | 0.8442 |
| How do I write a for loop in Rust? | No | 0.8960 |

## How I Used AI

**1.** I asked an AI model to help me reason about my corpus shape before coding the chunker. It suggested a paragraph-oriented strategy, but the actual documents are short and highly fact-dense, so I tightened the plan to paragraph-plus-sentence splitting and added a target size and overlap that fit the campus_life posts specifically.

**2.** I asked an AI model to test whether my acceptance criteria were measurable enough to defend. It pointed out that "good retrieval" was too vague, so I replaced it with specific test questions and explicit fact-check expectations tied to the corpus rather than broad qualitative language.

<!-- ── Stretch features ─────────────────────────────────────────────────────
     Doing one? Say so here BEFORE you start. A feature this README never
     claims earns nothing.
     ───────────────────────────────────────────────────────────────────────── -->

---

# Unit 2

<!-- These sections get ADDED to what's already above. Don't delete or rewrite
     unit 1 — the point is that someone can see what you said before you knew
     how it went. -->

## Run Log — Before

<!-- Your five criteria, three runs each. `python run_eval.py --label before`
     runs the questions, puts the OUT_OF_SCOPE ones through the gate, and
     writes it all into results/ for you. Targets come from criteria.md; the
     verdict column is your call.

     Criterion 3 is measured in one deterministic pass rather than three, so
     the same number goes in all three run columns. That's correct, not lazy.

     Milestone 1. 

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 |  |  |  |  |
| 2. Every answer names a source | 5 of 5 |  |  |  |  |
| 3. Gate stops out-of-corpus questions | 4 of 5 |  |  |  |  |
| 4. | | | | | |
| 5. | | | | | |

<!-- Underneath, paste the REAL output for each criterion from one of your
     runs — the actual text your system produced, not a description of it.
     Name the file and function that produced it. -->

## Verdicts

<!-- MET or MISSED for each of the five, against the target you wrote last
     unit — not a new one. Plus a sentence on how you decided. That sentence
     matters most where it was close.

     If your target said 4 of 5 and your runs came out 4, 3, 4, that's a MISS.
     The target has to hold, not show up occasionally.

     Milestone 2. -->

| # | Criterion | Verdict | How I decided |
|---|---|---|---|
| 1 |  |  |  |
| 2 |  |  |  |
| 3 |  |  |  |
| 4 |  |  |  |
| 5 |  |  |  |

## Diagnoses

<!-- For each miss: which stage caused it, and how. The stage alone isn't
     enough — you need the mechanism.

     Not a diagnosis: "Question 3 didn't work."
     A diagnosis:     "Question 3 asks about laundry costs. The answer is in
                       one sentence that got split across two chunks, so
                       neither chunk on its own contains it."

     The five stages: loading → chunking → embedding → retrieval → generation.

     Look for a pattern. If three misses all ask about numbers, that's one
     problem, not three.

     Missed nothing? Say so, then say honestly whether your targets were set
     low, and which one you'd tighten and to what.

     Milestone 3. -->

## The Improvement

**What I changed:**

**Why I picked it:**

<!-- Connect it to a specific diagnosis above in one sentence. If you can't,
     you picked a fix because it sounded impressive. -->

### Run Log — After

<!-- Same format, same five criteria, three runs each.
     `python run_eval.py --label after` -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 |  |  |  |  |
| 2. Every answer names a source | 5 of 5 |  |  |  |  |
| 3. Gate stops out-of-corpus questions | 4 of 5 |  |  |  |  |
| 4. | | | | | |
| 5. | | | | | |

**Did it help?**

<!-- Say plainly whether it did, and how you know. If it made things worse,
     say that — a change that backfired, honestly reported, earns full credit
     and is more interesting than one that worked. What matters is that you can
     tell.

     Milestone 4. -->

## What's Still Broken

<!-- For each criterion still missed after your fix: what you'd do about it,
     and why you stopped where you did.

     "I ran out of time" is fine if it's true. Pretending nothing is left is
     not.

     Milestone 5. -->

## What I'd Do Differently

<!-- Knowing what you know now — which of your five criteria would you write
     differently, and why?

     Milestone 5. -->
