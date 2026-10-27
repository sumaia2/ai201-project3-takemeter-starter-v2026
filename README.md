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

I fine-tuned istilbert-base-uncased on my 156 labeled posts using the starter notebook (takemeter.ipynb), with the default settings: 3 epochs, learning rate 2e-5, batch size 16, 128 tokens per post, and a 70/15/15 train/validation/test split. I did not change any hyperparameters. The run was on a CPU (PyTorch 2.14.1+cpu), and the notebook wrote its metrics to results.json and its held-out posts to test_split.csv.

## How I Used AI

**Moment 1:** I copied pages from old.reddit.com and asked Claude to turn them into CSV rows. It removed usernames and links, cut very long posts down to their opening, and left out posts with mental health details. I accepted the shortening because the model only reads the first 128 tokens.

**Moment 2:** I asked Claude to suggest labels for 136 of my posts, using my label definitions. I labeled the first 20 posts myself with no AI help. I read Claude's suggestions and corrected the ones I disagreed with. I also had Claude format my acceptance criteria, but I chose the numbers and the targets myself.

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained
The baseline is facebook/bart-large-mnli, a general model that sorts posts using only my three label definitions (label_definitions.txt), with no training. The trained model is distilbert-base-uncased, fine-tuned on my 108 training posts. Both were scored on the same 24 held-out posts (test_split.csv). Device: CPU.

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy | 0.625 | 0.583 | -0.042 |
| Macro F1 | 0.396 | 0.246 | -0.150 |
| F1: `question` | 0.743 | 0.737 | -0.006 |
| F1: `rant` | 0.444 | 0.000 | -0.444 |
| F1: `tip` | 0.000 | 0.000 | +0.000 |

**What I predicted before I looked:** I did not write down a prediction before seeing the numbers, so I don't have one. Looking back, I would have predicted my trained model would beat the baseline on `rant`, because it was trained on my own labels. It did not.

**What the gap actually means:** the trained model did not beat the baseline. It scored 0.583 accuracy against the baseline's 0.625, and it predicted `question` for almost every post, scoring 0.0got `rant` partly right.



---

## Run Log — Before

<!-- Five criteria across three seeds. The notebook's section 6 prin00 on `rant` and `tip`. That is an imbalance problem: only 25 rants and 20 tips were in the 108 training posts, so the model learned to guess the majority label. The baseline only has to follow the definitions I wrote, so it needs no training examples and ts the
     spread table; the Target and Verdict columns are yours. -->

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |


## Confusion Matrix (before the improvement)

Device: CPU. Seeds: 42, 7, 2024. All three seeds gave the same result: accuracy 0.583, macro F1 0.246. Rows are the true labels and columns are the model's guesses (test split of 24 posts).

| true \ predicted | question | rant | tip |
|---|---|---|---|
| **question** | 14 | 0 | 0 |
| **rant** | 6 | 0 | 0 |
| **tip** | 4 | 0 | 0 |

The model called every post `question`. The biggest confusion is 6 real `rant` posts called `question`, followed by 4 real `tip` posts called `question`. Nothing was ever predicted as `rant` or `tip`.

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

My labels matched the staff labels on 24 of 30 posts (80%). I labeled all 30 posts under the staff definitions in data/staff_taxonomy.md before I opened the staff labels. Staff marked sp_014, sp_020 and sp_029 as borderline.

**sp_008**: "Went back through the club results going back to 2012. Every single year the number of people who enter the spring marathon and don't start it sits between 18 and 23 percent..."
- Me: hot_take. Staff: analysis.
- My call: staff is right. The post names a span (2012 onward) and a range (18 to 23 percent), and it uses them to argue against a one-off explanation ("it's the base rate"). Under rule 1 the fact carries the argument. I treated it as an assertion because the data isn't shown, but the taxonomy only requires a checkable fact that does the work, so I applied the rule too strictly.
- Pile: (a), the rule covered it and I applied it too strictly.

**sp_014**: "Times are down about 3% since the shoe rules changed, so obviously the shoes are the whole story and the sport has a problem."
- Me: analysis. Staff: hot_take.
- My call: staff is right. The writer reaches the conclusion with "so obviously", and the 3% is decoration on a conclusion already made. This is almost the same as the taxonomy's own rule 1 example, where "times are down 3%" is a hot_take. I saw a number and missed that it wasn't load-bearing.
- Pile: (a), the taxonomy's rule 1 example is almost identical.

**sp_020**: "Every injury thread on this forum is the same injury and the same three replies."
- Me: analysis. Staff: hot_take.
- My call: staff is right. "Three replies" sounds countable, but no count or example is given, so the writer asserts and doesn't show. The taxonomy says the fact has to be checkable and do work in an argument, and here nothing is argued. It is a general claim with no support.
- Pile: (a), no count was given, so rule 1 covers it.

**sp_024**: "Watching the last 400 of that and I had to stand up. Still standing up actually."
- Me: hot_take. Staff: reaction.
- My call: staff is right. Rule 2 separates reaction from hot_take by whether the post is about the writer or about the world. This post makes no general claim; it reports what the writer is doing and feeling right now. My hot_take label doesn't fit.'
- Pile: (a), rule 2 covers it.

**sp_025**: "Blew up at 20 with the pace band still in my hand like it was going to help. Rough one."
- Me: analysis. Staff: reaction.
- My call: staff is right. The "20" is a detail in the writer's own race story, and nothing is argued from it. Rule 3 says recency doesn't outrank evidence, but that only applies when there is an argument backed by a fact, and here there isn't one.
- Pile: (a), rule 3 covers it.

**sp_029**: "Just realised the shoes I set every PB in this year are the ones I bought by accident in the wrong size."
- Me: analysis. Staff: reaction.
- My call: this is the closest one, and staff marked it borderline. I still think my reading is defensible, because the post states a checkable fact (every PB, one pair of shoes) and implies a point about shoes. But the taxonomy's analysis definition needs an argument that the fact supports, and this post draws no conclusion. "Just realised" signals a personal, immediate observation, so I accept staff's reaction label, while noting that rule 4 (which label gets most of the words) could support mine.
- Pile: (c), the rule is ambiguous here and my reading is defensible.

**Pattern:** five of the six disagreements are pile (a) and one is pile (c). In most of them I labeled on a surface feature, like a number or a firm tone, without checking whether it did the work the rule requires. That is a consistency problem in how I applied the rules, and it may also have happened in my own labeling of `rant` and `question`. The model's result, though, looks more like too few examples: it predicted `question` for every post, and only 25 rants and 20 tips were in the training set.

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
