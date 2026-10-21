# TakeMeter

## What This Does
TakeMeter sorts posts from r/PharmacyTechnician, a public subreddit for pharmacy technicians, into three types: `question`, `rant` and `tip`. It is a fine-tuned text classifier trained on posts I collected and labeled.

## Label Taxonomy

| Label | Definition | Example 1 | Example 2 |
|---|---|---|---|
| `question` | The post asks the community for an answer or advice. | "Why Is It So Hard to Get a Pharmacy Tech Job?" | "Desiccant dispensing?" |
| `rant` | The post vents frustration and needs no answer. | "Medi-Cal does it again!" | "anyone else tired of the glp 1s?" |
| `tip` | The post shares information that helps others and asks for nothing. | "Let's share extensions!!" (phone shortcuts for chain pharmacies) | "Cool tip for CPhTs" (the digital badge) |

**Decision rule for the hardest boundary (`rant` vs `question`):** many vents end with a question. Mentally delete the question sentence. If the post still reads as a complete post, it is a `rant`. If it no longer makes sense without the question, it is a `question`.

## The Dataset

- **Source:** public posts from r/PharmacyTechnician, copied from the top-of-the-year listing on old.reddit.com. I left out usernames, links, memes and posts that were only a picture.
- **Count:** 156 posts. I stopped short of the target of 200 because I was short on time as a parent of a young child. 156 is above the course's 150 floor.
- **Count per label:** `question` 91 (58%), `rant` 36 (23%), `tip` 29 (19%).
- **Labeling process:** I labeled the first 20 rows myself, with no help, and marked them `cold` in the note column. Claude suggested labels for the other 136 rows, which are marked `pre-labeled` in the note column, and I read them and corrected the ones I disagreed with.
- **How posts were chosen:** I did not copy the subreddit's own flair as my label, but I used the Question and Tips & Tricks flair to find more posts of those types, since most of the top posts were rants and memes. Very long posts were shortened to their opening, because the model only reads the first 128 tokens of each post. I left out posts that included someone's mental health history.
- **What I noticed:** many posts that sound like vents end with a request for advice, so `question` became the biggest label and `rant` was smaller than I expected. `tip` is the smallest label, so its score will be unstable, since only a handful of tip posts will land in the test split. Four tips come from one author, and they read like long study guides, so the model may learn "long study-guide post" instead of "helpful post".

**Hard cases:**
1. **"end of the year":** it was tagged Question and asks "can I have this filled before the end of the year?", but the post is really a vent about a question the poster hears all the time. I labeled it `rant`.
2. **"After three tries finally passed my PTCB exam":** a celebration that also recommends a study resource. It asks for nothing and names one thing that helped, so I labeled it `tip`.
3. **"Considering Retail":** it describes a hostile hospital job (a vent) but ends by asking whether this has happened to others. Under my rule it does not make sense without that question, so I labeled it `question`.

## The Training Run

I fine-tuned a small pretrained language model on my 156 labeled posts using the starter notebook (takemeter.ipynb), with the default settings: 3 epochs, learning rate 2e-5, batch size 16, 128 tokens per post, and a 70/15/15 train/validation/test split. I did not change any hyperparameters. The run was on a CPU (PyTorch 2.14.1+cpu), and the notebook wrote its metrics to results.json and its held-out posts to test_split.csv.

## How I Used AI

**Moment 1:** I copied pages from old.reddit.com and asked Claude to turn them into CSV rows. It removed usernames and links, cut very long posts down to their opening, and left out posts with mental health details. I accepted the shortening because the model only reads the first 128 tokens.

**Moment 2:** I asked Claude to suggest labels for 136 of my posts, using my label definitions. I labeled the first 20 posts myself with no AI help. I read Claude's suggestions and corrected the ones I disagreed with. I also had Claude format my acceptance criteria, but I chose the numbers and the targets myself.

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy |  |  |  |
| Macro F1 |  |  |  |
| F1 — `label_one` |  |  |  |
| F1 — `label_two` |  |  |  |

**What I predicted before I looked:**
<!-- Milestone 1 asks you to write this BEFORE seeing the trained numbers. A
     prediction made afterwards isn't one. -->

**What the gap actually means:**
<!-- If the baseline matched your trained model, your fine-tuning added
     nothing — and that is a real finding, not a failure. Say it plainly. -->



---

## Run Log — Before

<!-- Five criteria across three seeds. The notebook's section 6 prints the
     spread table; the Target and Verdict columns are yours. -->

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

### Confusion matrix

<!-- ⚠️ TYPED AS A MARKDOWN TABLE. The notebook prints one ready to paste.
     An image of a matrix earns nothing. -->

| true \ predicted |  |  |  |
|---|---|---|---|
| **** |  |  |  |
| **** |  |  |  |
| **** |  |  |  |

**My biggest off-diagonal number, and what it means:**
<!-- Not "the model made mistakes" — WHICH boundary it didn't learn, and which
     direction. "7 real analysis posts were called hot_take and only 3 went the
     other way" is a direction, not just an error rate. -->



---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 |  |  |  |  |
| 2 |  |  |  |  |
| 3 |  |  |  |  |
| 4 |  |  |  |  |
| 5 |  |  |  |  |

**Diagnoses**

<!-- For each miss: the cause, and how you know. The four common causes are:
     too few examples for a label, a boundary you applied inconsistently, a
     genuinely hard label pair, and a task the model can't reach from this
     much data.

     ⚠️ Use your agreement report as evidence. It is the only instrument you
     have that can tell a LABELLING problem from a MODEL problem, and this
     section is graded on whether you used it that way. -->



---

## Agreement Report

<!-- Your rate against the staff set, and every disagreement adjudicated.

     Remember you labelled these 30 under the STAFF taxonomy in
     data/staff_taxonomy.md, not your own — so every argument below is made
     from those definitions and those decision rules. -->

**Agreement rate:** ___ / 30 = ___%

<!-- Nobody grades this number. A 60% who argues every disagreement from the
     stated rules beats a 95% who wrote "staff was right" nine times. Several
     of the 30 were chosen because they're genuinely ambiguous — you should be
     winning some of these. -->

**Disagreements**

<!-- Three lines each: the post, both labels, and who you think is right and
     why — grounded in the staff definitions you were both applying.

     Then sort each into one of three piles:
       (a) the rule covered it and I applied it loosely → a consistency problem
       (b) the rule genuinely doesn't say               → a gap in the definitions
       (c) the rule is ambiguous here and my reading is defensible → argue it.
           This is a legitimate win.

     Pile (a) is the one that matters most for your diagnosis: if you applied a
     written rule two different ways on 30 posts, that is direct evidence about
     what you did across your own 200. -->

**1.**
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**2.**
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**What the pattern in my disagreements tells me:**



---

## The Improvement

**What I changed:**

**Which diagnosis pointed at it:**

### Run Log — After

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

**Did it help, and how do I know:**

<!-- If it didn't, say so. Relabelling that didn't help is a genuinely
     interesting result and earns full credit. -->



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped. -->



**The gap between what I meant my labels to capture and what the model
learned:**
<!-- Two sentences. Your confusion matrix is the evidence. -->



<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 5

       [ ] criteria.md has five numbered criteria, each naming a NUMBER
       [ ] Each has a reason underneath tied to your data or taxonomy
       [ ] labels.csv: at least 150 rows, text/label/note, ONE file not split
       [ ] No label above 70%
       [ ] All five unit 5 sections have real content
       [ ] Label Taxonomy includes the decision rule for your hardest boundary
       [ ] The Dataset includes three hard cases
       [ ] results.json and test_split.csv committed (the notebook does this)
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN

     SUBMISSION CHECKLIST — unit 6

       [ ] Baseline vs. Trained table, with your prediction written beforehand
       [ ] Run Log — Before, five criteria across three seeds
       [ ] Confusion matrix TYPED AS A MARKDOWN TABLE
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, using the agreement report as evidence
       [ ] Agreement Report with every disagreement adjudicated
       [ ] One improvement, with Run Log — After
       [ ] What's Still Broken
       [ ] results_three_seeds_before.json, results_three_seeds_after.json,
           baseline_results.json and
           agreement_results.json committed
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
