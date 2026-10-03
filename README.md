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

**3.** In this unit, I used an AI model to check whether my verdicts were being read too generously and whether a proposed hybrid-search improvement would really help a retrieval failure rather than just look more advanced. Because the system already met all five targets before the fix, the model helped me keep the before/after comparison honest instead of writing a change that only sounded plausible.

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

I ran the full evaluation with `python run_eval.py --label before` and logged the result in `results/run_2026-10-03_1307_before.md`. The file was produced by `run_eval.py::main` and the gate evidence was produced by `run_eval.py::check_out_of_scope`.

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 2. Every answer names a source | 5 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 3. Gate stops out-of-corpus questions | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 4. Sampled chunks read as complete thoughts | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 5. Answer contains the expected fact and no invented alternative | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |

### Real output from the before run

This is the actual text the system produced in `results/run_2026-10-03_1307_before.md`.

#### Question 1 — Is the housing lottery random?

```
The housing lottery is not entirely random; rising sophomores get a number drawn at random, but juniors and seniors are ordered by accumulated credit hours first, with random tie-breaks used only for ties (admin_housing_lottery.txt).
```

#### Question 2 — What are the actual wait times at Kestrel Commons around lunch?

```
The wait times at Kestrel Commons are 20 to 25 minutes between 12:15 and 1:00, and under 5 minutes before 11:45. 

Source: dining_kestrel_commons.txt (and also mentioned in dining_kestrel_commons_followup.txt)
```

#### Question 3 — Which dorm is closest to the science quad?

```
Aldridge Hall is the closest building to the science quad (four minutes away). 

Source: housing_aldridge_hall.txt (also mentioned in transit_walking.txt)
```

#### Question 4 — How much does laundry cost at Aldridge Hall?

```
Laundry at Aldridge Hall costs $1.75 for a wash and $1.50 for a dryer. 

Sources: `housing_aldridge_hall_laundry.txt` and `housing_aldridge_hall.txt`
```

#### Question 5 — What do students say about the dining hall salad bar after 1:30?

```
Students say that the salad bar wilts after 1:30. 

Source: `dining_kestrel_commons.txt` and `dining_kestrel_commons_followup.txt`
```

#### Gate result for out-of-scope questions

```
What is the capital of Mongolia? | 0.825 | refused
How do I change the oil in a diesel engine? | 0.934 | refused
Who won the 1994 World Cup? | 0.886 | refused
What is the recommended dosage of ibuprofen for a headache? | 0.844 | refused
How do I write a for loop in Rust? | 0.896 | refused
```

## Verdicts

| # | Criterion | Verdict | How I decided |
|---|---|---|---|
| 1 | Retrieved chunks contain the answer | MET | All five in-scope questions had at least one retrieved chunk that included the factual answer, and each answer included the key fact from the corpus. |
| 2 | Every answer names a source | MET | Every answer included a document name or source label, and the pattern stayed consistent across all three runs. |
| 3 | The relevance gate stops out-of-corpus questions | MET | The gate refused all five off-topic questions in the deterministic out-of-scope pass, which is 5/5 and above the 4/5 target. |
| 4 | Sampled chunks read as complete thoughts | MET | The paragraph-based chunker kept the practical fact intact in the sample chunks, and the reviewed chunks read as complete statements rather than cut-off thoughts. |
| 5 | The answer contains the expected fact and not an invented alternative | MET | Each answer matched the documented fact closely and did not introduce a conflicting claim; the benchmark examples all stayed grounded in the retrieved text. |

## Diagnoses

I did not miss any criteria, so there is no failed pipeline diagnosis to assign. The reason the system looked strong was not that the criteria were weak; it was that the corpus and the retrieval setup were already aligned with the questions I chose. The criterion I would tighten in a later unit is criterion 5: instead of a broad "close paraphrase" standard, I would require the answer to include the exact quantity or phrase that the document used for the most fact-sensitive questions, such as the exact laundry prices and wait times, so the response has a stricter factual anchor.

## The Improvement

**What I changed:** I added a hybrid retrieval option in `store.py` and a `HYBRID_SEARCH` toggle in `config.py` that blends the existing semantic cosine search with a lightweight BM25 keyword pass using `rank_bm25`.

**Why I picked it:** I picked this because it targets the retrieval stage directly: if a question contains exact names, numbers, or uncommon terms, a keyword pass can surface the strongest chunk even when semantic similarity is close but not decisive. The improvement was chosen as a retrieval-stage change, not a generation prompt change, because the raw answers were already grounded and the gate was already refusing the off-scope questions.

### Run Log — After

I ran the full evaluation again with the hybrid flag enabled in the shell: `AI201_HYBRID=1 python run_eval.py --label after`. The log was written to `results/run_2026-10-03_1322_after.md`.

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 2. Every answer names a source | 5 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 3. Gate stops out-of-corpus questions | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 4. Sampled chunks read as complete thoughts | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 5. Answer contains the expected fact and no invented alternative | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |

**Did it help?** No. The before and after numbers were the same: 5/5 on all five criteria and 5/5 refusal on the out-of-scope gate. Because the system was already meeting all targets before the change, the hybrid search did not move the measured outcomes, so I would not keep it as a permanent improvement unless I had a harder set of questions that relied on exact names or numbers to break semantic matching.

## What's Still Broken

Nothing is still broken against the criteria I set in unit 1. The system met the targets it was designed for, and the main remaining issue is that the criteria themselves are a little easy: they are specific, but my test set is small and carefully selected. If I were iterating again, I would add a few more edge cases involving exact prices, nested policy language, or multi-part answers to make the system harder to pass accidentally.

## What I'd Do Differently

I would tighten criterion 5 and add one more question that requires the model to choose between similar-sounding alternatives, such as a dorm or a dining hall with multiple numbers in the corpus. That would better test whether the model is really grounding on the right document instead of picking the most plausible answer. I would also keep one or two false-positive questions that are near-miss or partially overlapping with the corpus, because those are more realistic for a relevance gate than totally unrelated trivia.
