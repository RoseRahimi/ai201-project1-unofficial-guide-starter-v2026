# The Unofficial Guide

Fatima Rahimi, advice_threads corpus

> **This file is your submission.** Fill it in as you go — most sections get
> written during the milestone that produces them, not at the end.
>
> How the starter works, and every command you'll need, is in `RUNNING.md`.
> Leave that file alone.
>
> **Paste everything as text.** No screenshots, no video. A typed table gets
> full credit; a picture of the same table gets none.
>
> Delete these instruction blocks as you replace them. The `<!-- -->` comments
> are notes to you and don't show up when the page renders — you can leave them
> or remove them.

---

# Unit 1

## What This Does

This is a question answering system built on the advice_threads corpus, 23 forum threads where students answer each other's practical questions about campus life: laundry timing, laptop specs, meal plans, bike commuting, that kind of thing. You ask it a question, it finds the thread closest in meaning to your question, and a model writes a short answer from that thread only, naming the file it came from. If your question isn't something the threads cover, a relevance gate refuses to answer instead of letting the model make something up.

## Chunking Strategy

**Chunk size: one whole thread (317–793 characters)**
**Overlap: none**

Each file in my corpus is one thread: a `THREAD:` title that states the question, then the replies that answer it. The replies only make sense under the title. "Counterpoint, I sold mine" is useless without knowing the question was about bikes, so cutting anywhere inside a thread separates answers from their question. I measured the threads and the longest is 793 characters, under the embedding model's roughly 1000 character limit, so nothing needs splitting and one thread = one chunk.

I changed my mind to get here. I first experimented with small fixed-size chunks through the fallback splitter, and when I looked at what retrieval brought back, some chunks were cut mid reply and one literally started mid word ("t...."). That's what convinced me the thread boundary is the only place a cut makes sense in this corpus. If a longer thread ever shows up, my plan is one chunk per reply with the `THREAD:` title line prepended to each.

## Sample Chunks

<!-- Five chunks, pasted as text. Label each one and name the file it came from
     AND the function that produced it — the grader checks your code against
     what you claim here.

     `python app.py chunks -n 5` prints all three for you. Copy them straight
     across.

     Milestone 3. -->

**Chunk 1** — source: `thread_bike_commute.txt#0` — produced by: `chunker.py::split_documents`

```
THREAD: Is a bike worth it for a 20 minute walk commute?

--- reply 1 (14 votes) ---
Yeah. Cuts an 18 minute walk to about 6. The thing nobody mentions is storage — covered bike parking exists at three buildings and is full by 9am at all three.

--- reply 2 (9 votes) ---
Counterpoint, I sold mine. Between November and March the paths are either icy or salted and salt destroys a drivetrain in one season.

--- reply 3 (22 votes) ---
Both true. I keep a cheap bike for September to November and walk the rest of the year. Total cost was about $120 for the bike and I don't care what happens to it.

--- reply 4 (5 votes) ---
If you do get one, the campus does free registration and it's the only reason I got mine back after it was taken.
```

**Chunk 2** — source: `thread_first_year_regret.txt#0` — produced by: `chunker.py::split_documents`

```
THREAD: What do you wish you'd known in first year?

--- reply 1 (41 votes) ---
That the add/drop deadline and the withdrawal deadline are different dates and only one of them is on the calendar everyone reads.

--- reply 2 (28 votes) ---
That you can take a course pass/fail and declare it late — up to week eight. I carried a grade I didn't need to.

--- reply 3 (35 votes) ---
That the writing centre will read a draft for any course, not just writing courses. Free, and the appointments go unbooked.

--- reply 4 (52 votes) ---
Honestly: that nobody is watching as closely as you think. I spent a year worried about looking like I knew what I was doing.

--- reply 5 (17 votes) ---
That your adviser's job is partly to know the exceptions to rules. Ask before assuming a deadline is fixed.
```

**Chunk 3** — source: `thread_laundry_timing.txt#0` — produced by: `chunker.py::split_documents`

```
THREAD: When is laundry actually free in the dorms?

--- reply 1 (27 votes) ---
Tuesday and Wednesday mornings, every building. Sunday evening is the worst and it isn't close.

--- reply 2 (8 votes) ---
Depends heavily on your building. Morrow has more machines per person than Fenwick so it's less of a problem there.

--- reply 3 (16 votes) ---
The app that shows machine availability is wrong about half the time. It reports a machine as free for a few minutes after it finishes but before someone unloads it.
```

**Chunk 4** — source: `thread_printing.txt#0` — produced by: `chunker.py::split_documents`

```
THREAD: Is the printing quota enough?

--- reply 1 (17 votes) ---
For most people yes. $30 is about 600 pages black and white. It's the colour printing that eats it — eight times the cost per page.

--- reply 2 (11 votes) ---
Doesn't roll over between semesters. Print your readings in December rather than losing it.
```

**Chunk 5** — source: `thread_roommate_conflict.txt#0` — produced by: `chunker.py::split_documents`

```
THREAD: Roommate situation isn't working. What now?

--- reply 1 (28 votes) ---
Talk to your RA early, and frame it as 'we need help sorting this out' rather than 'move me'. Room changes are possible but the process starts with mediation and skipping that step slows it down.

--- reply 2 (14 votes) ---
Room changes happen at the semester boundary almost always, and mid-semester only in fairly serious cases.

--- reply 3 (33 votes) ---
Write down specifics before the meeting. 'It's not working' is hard to act on; 'guests four nights a week past 2am' is not.
```

## Sample Answer

**Question:** When are the dorm laundry machines actually free?

**Answer:**

```
  (best distance 0.270, cutoff 0.65)

Laundry is actually free in all buildings on Tuesday and Wednesday mornings.

Source: thread_laundry_timing.txt
```

**My relevance cutoff:** 0.65

I ran my five test questions and the five OUT_OF_SCOPE questions through `python app.py retrieve` and wrote down the best distance for each. The two groups came out with a clean gap: everything my corpus covers scored 0.23–0.40, and everything it doesn't scored 0.83–0.95, with nothing in between. I put the cutoff at 0.65 because it sits in the middle of that gap with margin on both sides: low enough that no junk question gets through, high enough that a slightly awkwardly worded real question still passes. I originally had it at 0.85, which turned out to be too loose: two of the five out-of-scope questions would have gotten past the gate and reached the model.

| Question | In corpus? | Best distance |
| -------- | ---------- | ------------- |
| Is it worth getting a bike to shorten my walk to campus? | yes | 0.233 |
| How much RAM do students say you actually need for CS courses? | yes | 0.259 |
| How many courses can I take pass fail in a year? | yes | 0.274 |
| What time of week is it easiest to find an open washing machine in the dorms? | yes | 0.366 |
| How many black and white pages does the printing quota cover? | yes | 0.399 |
| What is the recommended dosage of ibuprofen for a headache? | no | 0.828 |
| How do I write a for loop in Rust? | no | 0.871 |
| How do I change the oil in a diesel engine? | no | 0.930 |
| What is the capital of Mongolia? | no | 0.948 |
| Who won the 1994 World Cup? | no | 0.952 |

## How I Used AI

<!-- Two specific moments. For each: what you asked for, what came back, and
     what you changed about it.

     "I asked Claude to write the chunking function from my notes. It ignored
     the overlap, so I added that myself" is the level of detail we're after.
     "I used AI to help me code" is not.

     Milestone 5. -->

**1.** I had set my relevance cutoff to 0.85 and asked Claude whether it was right. It ran my five out-of-scope questions through retrieval and showed me that two of them (ibuprofen at 0.79, the World Cup at 0.83 on the old index) came in under 0.85, meaning the gate would have passed them through to the model, so my own criterion 3 would have failed at 3 of 5. I moved the cutoff to 0.65, inside the gap between my two groups of distances, and re-verified that all five junk questions now get refused.

**2.** I asked Claude to explain `chunker.py` line by line and suggest improvements. It suggested one thread = one chunk and backed it up by measuring my corpus (23 threads, 317–793 characters each, all under the embedding model's input limit). I also learned from the retrieval output that my old index had chunks cut mid-word. The index was stale from an earlier chunk-size experiment, and I hadn't realized the index only changes when you rebuild it. After changing the chunker I re-indexed and re-ran all ten retrieve commands to confirm the distances myself; the out-of-scope group actually moved further away (0.79 → 0.83 at the closest).

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

     Milestone 1. -->

| Criterion                              | Target | Run 1 | Run 2 | Run 3 | Verdict |
| -------------------------------------- | ------ | ----- | ----- | ----- | ------- |
| 1. Retrieved chunk contains the answer | 4 of 5 | 5/5   | 5/5   | 5/5   | MET     |
| 2. Every answer names a source         | 5 of 5 | 5/5   | 5/5   | 5/5   | MET     |
| 3. Gate stops out-of-corpus questions  | 4 of 5 | 5/5   | 5/5   | 5/5   | MET     |
| 4. Every chunk is one complete thread  | 5 of 5 | 5/5   | 5/5   | 5/5   | MET     |
| 5. Conflicting replies shown both ways | 2 of 3 | 3/3   | 3/3   | 2/3   | MET     |

The evidence files are `results/run_2026-10-11_1411_before.md` (criteria 1 to 3,
scored by `scorer.py::judge`) and `results/run_2026-10-11_1422_criterion5_before.md`
(criterion 5). Criteria 3 and 4 are measured in one deterministic pass, so the
same number fills all three run columns. Criterion 1 counts whether the sources
retrieved for each question include the thread that holds the answer, and
retrieval is deterministic too, so its runs match; criteria 2 and 5 depend on
the generated answer and genuinely could have varied, they just didn't.

**Criterion 1** (retrieval, from `run_eval.py::main` via `store.py::search`): for
every question the retrieved sources include the right thread, for example:

```
### What time of week is it easiest to find an open washing machine in the dorms? — run 1
- Best distance: 0.3658 (passed the gate)
- Sources retrieved: thread_commuting.txt, thread_late_work.txt, thread_laundry_timing.txt, thread_sleep_schedule.txt, thread_study_spots.txt
```

**Criterion 2** (source naming, answer produced by `generate.py::answer_from_chunks`):
all 15 answers name their source file. One of them:

```
The printing quota of $30 covers about 600 black and white pages. (Source: thread_printing.txt)
```

**Criterion 3** (the gate, from `run_eval.py::check_out_of_scope`, cutoff 0.65):

```
| Out-of-scope question | Best distance | Gate |
|---|---|---|
| What is the capital of Mongolia? | 0.948 | refused |
| How do I change the oil in a diesel engine? | 0.930 | refused |
| Who won the 1994 World Cup? | 0.952 | refused |
| What is the recommended dosage of ibuprofen for a headache? | 0.828 | refused |
| How do I write a for loop in Rust? | 0.871 | refused |
```

**Criterion 4** (chunks, from `python app.py chunks -n 5`, chunks produced by
`chunker.py::split_documents`): all five sampled chunks are complete threads
starting with their title line:

```
Chunk 1  |  source: thread_bike_commute.txt#0  |  produced by: chunker.py::split_documents
THREAD: Is a bike worth it for a 20 minute walk commute?
```

**Criterion 5** (conflicting replies, questions from `questions.py::CONFLICT_QUESTIONS`,
answers by `generate.py::answer_from_chunks` with cache off): parking and textbook
kept both sides in every run; the bike answer did too in runs 1 and 2 but led with
"yes" in run 3, which I scored as a fail, leaving run 3 at 2 of 3. One of the
clean ones:

```
According to thread_parking.txt, a west permit is worth getting if you are
commuting daily; otherwise, it is not worth bothering.
```

## Verdicts

<!-- MET or MISSED for each of the five, against the target you wrote last
     unit — not a new one. Plus a sentence on how you decided. That sentence
     matters most where it was close.

     If your target said 4 of 5 and your runs came out 4, 3, 4, that's a MISS.
     The target has to hold, not show up occasionally.

     Milestone 2. -->

| # | Criterion | Verdict | How I decided |
| - | --------- | ------- | ------------- |
| 1 | Retrieved chunk contains the answer | MET | the sources retrieved for every question include the thread that holds the answer, in all three runs (5/5, target 4/5). retrieval is deterministic, so one check per question settles it. |
| 2 | Every answer names a source | MET | I read all 15 answers in the before log rather than trusting the scorer for this one; every single answer names a thread file (15/15, target all). |
| 3 | Gate stops out-of-corpus questions | MET | the gate refused all 5 OUT_OF_SCOPE questions, closest at 0.828 against the 0.65 cutoff. but this verdict is softer than it looks: the question set made it unmissable, see the revision under criterion 3 in criteria.md. |
| 4 | Every chunk is one complete thread | MET | all 5 sampled chunks start with their THREAD: line and end at the thread's end. the chunker emits whole files, so a broken chunk is impossible by construction, which is why one sample pass is enough. |
| 5 | Conflicting replies shown both ways | MET | parking and textbook answers kept the disagreement in all runs. bike run 3 was borderline, it led with "yes" and kept the downsides as caveats, so I scored it strictly as a fail, and the run still comes out 2 of 3, which meets the target. |

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

| Criterion                              | Target | Run 1 | Run 2 | Run 3 | Verdict |
| -------------------------------------- | ------ | ----- | ----- | ----- | ------- |
| 1. Retrieved chunk contains the answer | 4 of 5 |       |       |       |         |
| 2. Every answer names a source         | 5 of 5 |       |       |       |         |
| 3. Gate stops out-of-corpus questions  | 4 of 5 |       |       |       |         |
| 4.                                     |        |       |       |       |         |
| 5.                                     |        |       |       |       |         |

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
