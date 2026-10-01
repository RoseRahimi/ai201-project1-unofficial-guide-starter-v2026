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

I ran my five test questions and the five OUT_OF_SCOPE questions through `python app.py retrieve` and wrote down the best distance for each. The two groups came out with a clean gap: everything my corpus covers scored 0.26–0.40, and everything it doesn't scored 0.83–0.95, with nothing in between. I put the cutoff at 0.65 because it sits in the middle of that gap with margin on both sides: low enough that no junk question gets through, high enough that a slightly awkwardly worded real question still passes. I originally had it at 0.85, which turned out to be too loose: two of the five out-of-scope questions would have gotten past the gate and reached the model.

| Question | In corpus? | Best distance |
| -------- | ---------- | ------------- |
| How much RAM do students say you actually need for CS courses? | yes | 0.259 |
| When are the dorm laundry machines actually free? | yes | 0.270 |
| How long do I have to change my meal plan tier after the semester starts? | yes | 0.285 |
| Which cafe do students recommend as a study spot before 10am? | yes | 0.365 |
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
| 1. Retrieved chunk contains the answer | 4 of 5 |       |       |       |         |
| 2. Every answer names a source         | 5 of 5 |       |       |       |         |
| 3. Gate stops out-of-corpus questions  | 4 of 5 |       |       |       |         |
| 4.                                     |        |       |       |       |         |
| 5.                                     |        |       |       |       |         |

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
| - | --------- | ------- | ------------- |
| 1 |           |         |               |
| 2 |           |         |               |
| 3 |           |         |               |
| 4 |           |         |               |
| 5 |           |         |               |

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
