# Phase 1: Structure the Study

You are guiding a user from a vague idea to a rigorous between-subjects consumer study design. Ask questions one at a time. Do not dump a list of questions. Wait for each answer before moving on.

Throughout this guide, the **Gateway Study** is used as a concrete example: it tested whether insecticide-free certification seals on blueberry packaging shift consumer associations toward "Impact on animals," using Oreo cookies and Huel as distractor products, across 1,027 real participants on Prolific.

---

## Step 1: Open with one question

Ask:

> What do you want to test? Describe it in plain language. For example: "I want to know if putting an insecticide-free seal on our blueberry packaging makes people associate the product with animal welfare."

Accept whatever they give you. The rest of this phase turns their answer into a proper study design.

---

## Step 2: Clarifying questions (one at a time, in order)

Ask each question individually. Wait for the user's response before asking the next. Adapt your phrasing based on their previous answers.

### 2a. Independent variable

> What is the one thing you are changing between groups? This is your independent variable. It should be a single, concrete manipulation.

Help them isolate one variable. If they describe multiple changes, push back. A clean experiment changes one thing.

**Gateway example:** The independent variable was the certification seal design on the blueberry package. One group saw no seal (control). Two treatment groups each saw a different seal design (a bee-themed seal and a pollinator-protector seal).

### 2b. Control condition

> What does the control group see? The control must be identical to the treatment in every way except the one thing you are testing.

If they are testing a label, the control sees the product without the label. If they are testing a name, the control sees the original name. The control is never "nothing" in the abstract. It is a specific, concrete stimulus.

**Gateway example:** The control group saw the same blueberry package with no certification seal.

### 2c. Number of conditions

> How many treatment conditions do you want? I recommend 2 to 3 treatment conditions plus 1 control. More conditions means more participants and more cost, but lets you compare variants.

Explain the tradeoff: each additional condition requires a full sample (300+ participants for statistical power at that level, minimum 100 for a pilot). Two treatments plus one control is a good default. More than four treatment conditions usually introduces noise without adding insight.

**Gateway example:** 3 total conditions: 1 control (no seal) + 2 treatments (bee seal, pollinator seal). This allowed direct comparison of two seal concepts while keeping costs reasonable.

### 2d. Dependent variables

> What outcomes do you want to measure? I will help you choose measures that capture real attitudes and behaviors, not just self-reported preferences.

Guide them toward two types of outcomes:

1. **Behavioral outcomes** (strongest): purchase intent, willingness to pay, choice between products
2. **Attitudinal outcomes** (important): associations, perceptions, feelings about the product

Push for at least one **unprompted measure**. This is critical. An unprompted measure asks participants to generate their own response before you give them any hint about what the study is really about. It is the most credible evidence that your manipulation actually shifted how people think.

**Gateway example:**
- **Primary outcome (unprompted):** "When you think about buying blueberries, which of these considerations come to mind?" with options including "Impact on animals" alongside non-target options like taste, price, health, food safety, and environmental impact. The key metric was whether treatment groups selected "Impact on animals" at higher rates than control.
- **Secondary outcomes (prompted):** Likert-scale ratings of environmental concern, wildlife protection importance, purchase intent, and willingness to pay more.

### 2e. Hypotheses

> What do you expect to happen? State it as a prediction that could be proven wrong. If your prediction cannot possibly be false, it is not a hypothesis.

Each hypothesis must be falsifiable. Help them write 2 to 3 hypotheses using this structure:

- **H1** should address the primary outcome (the unprompted measure).
- **H2** should address a secondary attitudinal or behavioral outcome.
- **H3** (optional) should address a comparison between treatment variants, or a downstream behavioral outcome.

**Gateway example:**
- **H1:** Participants who see an insecticide-free seal will select "Impact on animals" as a purchase consideration at a higher rate than participants who see no seal.
- **H2:** Participants who see an insecticide-free seal will report higher concern for wildlife protection than participants who see no seal.
- **H3:** Participants who see an insecticide-free seal will report higher purchase intent for the blueberries than participants who see no seal.

Each of these can be tested with a statistical comparison and could come back showing no difference. That is what makes them falsifiable.

### 2f. Distractor products

> Now pick 2 distractor products. These are unrelated products that participants also evaluate using the same questions. They serve two purposes:
>
> 1. **Masking:** They hide the study's real focus. If participants only answer questions about blueberries, they will guess the study is about blueberries and adjust their answers. Adding Oreos and Huel makes the study look like a general consumer preferences survey.
> 2. **Validation:** If your treatment somehow shifts attitudes toward Oreos too, something is wrong. Distractors should show no difference between conditions. When they do not differ, it confirms that your effect is specific to the product you manipulated.
>
> Good distractors are real, recognizable products from different categories than your focal product.

**Gateway example:** Oreo cookies and Huel (a meal replacement shake). Both are well-known, from completely different categories than fresh blueberries. Neither has any connection to insecticide-free farming. In the final analysis, neither product showed statistically significant differences between conditions on any measure, confirming the seal's effect was specific to blueberries.

### 2g. Target sample size

> How many participants per condition? I recommend:
>
> - **300+ per arm** for a well-powered study that can detect small-to-medium effects
> - **100 per arm minimum** for a pilot or resource-constrained study
>
> Fewer than 100 per arm risks missing real effects (underpowered) or producing unreliable estimates.

Explain: total participants = (number of conditions) x (participants per arm). With 3 conditions and 300 per arm, that is 900 total. With 100 per arm, that is 300 total.

**Gateway example:** Targeted 300+ per arm. Final sample: 318 (control), 242 (bee seal), 293 (pollinator seal) = 853 total for the three arms included in the primary analysis, 1,027 total across all waves.

### 2h. Budget calculation

> Here is your estimated cost. Prolific charges participant reward plus fees (service fee, VAT, and transaction fees typically total around 43%).

Calculate: `total_cost = n_conditions x n_per_arm x reward_per_participant x 1.43`

Present a table:

| Parameter | Value |
|---|---|
| Conditions | [number] |
| Participants per arm | [number] |
| Total participants | [number] |
| Reward per participant | $[amount] (recommend $3-5 for a 10-15 min survey) |
| Prolific fees (service + VAT) | ~43% |
| **Estimated total cost** | **$[total]** |

**Gateway example:**

| Parameter | Value |
|---|---|
| Conditions | 5 (including additional seal variants) |
| Participants per arm | 100 initial, scaled to 300+ |
| Total participants | 1,027 |
| Reward per participant | $4.00 |
| Prolific fees (service + VAT) | ~43% |
| **Estimated total cost** | **~$5,874** |

---

## Step 3: Generate the study design document

After all questions are answered, generate a study design document in this format. Save it to a file the user can review and edit (e.g., `study-design.md` in the project root).

```markdown
# [Study Title]

## Research Question

[One sentence. What are you trying to find out?]

## Hypotheses

- **H1:** [Primary hypothesis, targeting unprompted measure]
- **H2:** [Secondary hypothesis, targeting attitudinal/behavioral outcome]
- **H3:** [Tertiary hypothesis, optional]

## Conditions

| Condition | Label | Description |
|---|---|---|
| Control | [label] | [what participants see] |
| Treatment 1 | [label] | [what participants see] |
| Treatment 2 | [label] | [what participants see] |

## Primary Outcome

**Measure:** [description of the unprompted measure]
**Type:** Unprompted multi-select / free response / forced choice
**Analysis:** Chi-square test of independence (categorical) or Mann-Whitney U (ordinal)

## Secondary Outcomes

1. [Outcome name]: [description, scale type, planned analysis]
2. [Outcome name]: [description, scale type, planned analysis]
3. [Outcome name]: [description, scale type, planned analysis]

## Distractor Products

1. **[Product 1]:** [why it works as a distractor]
2. **[Product 2]:** [why it works as a distractor]

## Sample

| Parameter | Value |
|---|---|
| Design | Between-subjects |
| Participants per arm | [n] |
| Total participants | [n] |
| Platform | Prolific |
| Estimated reward | $[amount] per participant |
| Estimated total cost | $[amount] |

## Estimated Cost

`[n_conditions] x [n_per_arm] x $[reward] x 1.33 = $[total]`
```

---

## Step 4: Methodology gate

Before advancing to Phase 2 (Stimuli Design), verify all of the following. Do not proceed until every box is checked. If one fails, go back and fix it with the user.

- [ ] **Between-subjects confirmed.** Each participant sees exactly one condition. No within-subjects, no crossover.
- [ ] **2+ distractors chosen.** At least two distractor products from different categories than the focal product. Both will receive the same questions as the focal product.
- [ ] **All hypotheses are falsifiable.** Each hypothesis makes a specific, testable prediction. None are tautologies ("people will have opinions") or unfalsifiable ("the seal will resonate").
- [ ] **Unprompted measure identified.** At least one dependent variable captures participant associations before any hint about the study's true focus. This measure appears early in the survey, before prompted questions.

Present this checklist to the user and confirm each item explicitly before moving on.

---

## What comes next

Once the user approves the study design document and the methodology gate is passed, proceed to **Phase 2: Design Stimuli** (`phases/02-stimuli.md`).
