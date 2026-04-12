---
name: consumer-study
description: Run a complete consumer research study from research question to published report with DOI. Handles survey design, Prolific recruitment, LLM pre-testing, statistical analysis, and academic reporting. Use when a user wants to test anything with real consumers.
---

# Consumer Study

Run a complete consumer research study. From "I want to test X" to a published, DOI-minted study with statistical rigor and open data.

## When to Use

When a user wants to test something with real consumers. Examples:
- "Does this packaging change how people perceive our product?"
- "Which logo makes people trust us more?"
- "Does this label shift attitudes toward sustainability?"
- "How do consumers react to different price points?"

## Required API Keys

Before starting, confirm the user has these keys available:
- **Typeform API token** (survey creation): Settings > Personal tokens at typeform.com
- **Prolific API token** (recruitment): Settings > API Tokens at prolific.com
- **Anthropic API key** (LLM pre-testing): Set as ANTHROPIC_API_KEY env var

Optional:
- **Vercel** (report deployment): Logged in via `vercel` CLI
- **OSF API token** (data deposit + DOI): Settings > Personal access tokens at osf.io

## The Pipeline

Execute phases in order. Each phase has its own guide file with complete instructions.

### Phase 1: Structure the Study
**Read:** `phases/01-structure.md`

Ask clarifying questions to define the research question, independent variable, dependent variables, conditions, hypotheses, and distractor stimuli. Output a study design document.

### Phase 2: Design Stimuli
**Read:** `phases/02-stimuli.md`

Create stimulus variants using /frontend-design and headless Chrome. Render hi-res composites. Build showcase page for user review. Iterate.

### Phase 3: Build the Survey
**Read:** `phases/03-survey.md`

Construct the survey instrument with enforced methodology. Create Typeform surveys via API. Deploy randomization router.

### Phase 4: Pre-test with LLM Judges
**Read:** `phases/04-pretest.md`

Run 50-100 simulated respondents per arm using vision-capable models. Generate results dashboard. Surface issues before real money is spent.

### Phase 5: Launch and Monitor
**Read:** `phases/05-launch.md`

Create Prolific study. Publish. Monitor responses. Support multi-wave recruitment and live survey editing.

### Phase 6: Analyze
**Read:** `phases/06-analyze.md`

Pull responses. Run chi-square, Mann-Whitney U, Cramer's V. Validate with distractor product check. Compute margins of error.

### Phase 7: Publish
**Read:** `phases/07-publish.md`

Generate academic-style HTML report via /frontend-design. Deploy. Create OSF project with open data. Mint DOI.

## Methodology Rules (Non-negotiable)

These rules produce credible research. Do not skip or override any of them.

1. **Between-subjects only.** Each participant sees exactly one condition.
2. **Distractor products required.** Minimum 2 unrelated products with identical questions.
3. **Unprompted before prompted.** The key association question comes before any questions mentioning the treatment.
4. **Balanced response options.** All options in a multi-select must be similar in length and specificity.
5. **No leading language.** Flag questions containing words that prime the target response. Suggest neutral alternatives.
6. **Statistical tests required.** Every comparison includes test statistic, p-value, and effect size.
7. **Distractor validation.** The report must show distractor products had no cross-condition differences.
8. **Limitations section required.** Minimum 4 honest limitations.
9. **Open data by default.** Data deposit is part of the pipeline, not an afterthought.

## Style Rules

- Do not use emdashes anywhere in generated reports or survey questions. Use commas, periods, or semicolons instead.

## Phase Transitions

Do not advance to the next phase until the current phase is complete and the user has approved the output:
- Phase 1 -> 2: User approves study design document
- Phase 2 -> 3: User approves all stimulus variants
- Phase 3 -> 4: User approves survey instrument (walk through every question)
- Phase 4 -> 5: User reviews LLM pre-test results and approves launch
- Phase 5 -> 6: Responses reach target sample size
- Phase 6 -> 7: User reviews analysis and approves for publication
