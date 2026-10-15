# Acceptance criteria — TakeMeter

Five criteria that say what "working" means for this classifier, written in
unit 5 **before** anything was trained.

**All five are yours this time.** None are given. You've had two projects of
practice.

An acceptance criterion names a number. *"The model is accurate"* is an
opinion. *"Every label has an F1 of at least 0.60 on the held-out set"* is a
criterion.

Under each, write a sentence or two on **why that number**. A reason that says
something about your data or your taxonomy earns credit — *"I picked 0.60 F1
for `reaction` because it's my smallest label and I only have about 50
examples of it"*. A reason that could be attached to any project does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

---

## Pick numbers you can defend

Not numbers that sound impressive. Three labels means a coin-flip guesser gets
about 33%, so a target of 0.40 is barely a target. Your number should sit
somewhere you'd honestly call useful.

**Cover at least three of these five areas.** They're here as prompts, not as a
form to fill in — a criterion that fits none of them is fine if it names a
number.

| Area | A question it could answer |
|---|---|
| Overall accuracy | How often does it need to be right to be worth using? |
| Per-label performance | Is one label allowed to be much worse than the others? |
| Balance | How lopsided can your label counts get before it's a problem? |
| Consistency | If someone else labelled the same posts, how often should you agree? |
| Confidence | Should a confident prediction be right more often than an unsure one? |

Two things worth knowing before you pick numbers, because both will affect
whether you hit them:

- **Your smallest label will have the jumpiest score.** If a label has 50
  examples, about 8 land in the test split. An F1 computed on 8 examples moves
  a lot between seeds. A target for that label should be looser than one for
  your biggest label, and saying so is a good reason.
- **Unit 6 tests across three seeds, and the target has to hold across all
  three.** A target of 0.65 against results of 0.71, 0.62, 0.68 is a **miss**.
  Pick with that in mind — it is stricter than it first sounds.

---

## 1. Overall accuracy

On the held-out test split, the model's overall accuracy is at least 0.50 in each of three seeds.

**Why this target:** I have three labels, so random guessing gets about 0.33. My test split will be about 30 posts, so one wrong post moves accuracy by about 0.03, and a very high target would mostly measure noise.

---

## 2. Per-label F1

On the held-out test split, every label (question, rant and tip) has an F1 of at least 0.60 in each of three seeds.

**Why this target:** My smallest label will have only about 10 posts in the test split, so its F1 will jump around between seeds. I chose 0.60 because a label scoring lower than that is one I would not trust the model to sort, and I expect this to be my hardest criterion to hold in all three seeds.

---

## 3. Balance

In labels.csv, no single label makes up more than 70% of the rows.

**Why this target:** This is the course limit. Rants look common in r/PharmacyTechnician, so one label could take over my data, and a model trained on a lopsided file learns to guess that label.

---

## 4. Agreement with staff labels

On the posts that staff also labeled, my labels match the staff label on at least 80% of posts.

**Why this target:** I wrote the label definitions before labeling, and I hold the CPhT, so I know this community's topics. I expect my disagreements to cluster on rant versus question, where I wrote a decision rule, so 80% allows about 1 mismatch in 5.

---

## 5. Confidence

Among test posts where the model's confidence is above 0.80, at least 90% are predicted correctly, in each of three seeds. I will also report how many test posts fell above 0.80.

**Why this target:** A confident answer should be more reliable than the model's answers overall. With only about 30 test posts, few predictions may be above 0.80, so I will report that count next to the result.

---
<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 6 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath, like this:

         ## 2. Every label performs acceptably

         The model performs well on all labels.

         **Why this target:** ...

         > **Revised in unit 6:** Every label has an F1 of at least 0.60 on
         > the held-out set.
         >
         > **Why revised:** "performs well" gave me nothing to check. I
         > couldn't produce a verdict from it at all.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "Overall accuracy of at least 0.65" → "at least 0.55", because
            0.65 turned out to be optimistic for 200 examples.

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.
     ───────────────────────────────────────────────────────────────────────── -->
