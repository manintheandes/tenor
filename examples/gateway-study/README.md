# Gateway Study -- Worked Example

A complete worked example of the consumer-study skill. This study tested whether an insecticide-free certification seal on a blueberry container shifts consumer attention toward the impact of food production on animal life.

## Research Question

Does an insecticide-free certification seal on a blueberry container shift consumer attention toward the impact of food production on animal life?

## Hypotheses

- **H1:** Consumers who view a blueberry container with an insecticide-free seal will be more likely to spontaneously associate the product with "Impact on animals" compared to control.
- **H2:** This shift will occur without reducing purchase intent.
- **H3:** Exposure to the seal will increase self-reported interest in plant-based foods.

## Conditions

| Condition             | n   | Description                                                                  |
| --------------------- | --- | ---------------------------------------------------------------------------- |
| Control               | 391 | California Giant blueberry container, no seal                                |
| Soft Bee              | 289 | Same container with illustrated bee seal ("Insecticide Free, Safe for bees & butterflies") |
| Pollinator Protector  | 347 | Same container with honeycomb seal ("No insecticides used")                  |

## Stimuli

The Soft Bee seal featured a cute illustrated bee character with closed eyes and rosy cheeks. The Pollinator Protector used an abstract honeycomb icon. Both were gold circular seals digitally composited onto the product image. Multiple bee icon styles were tested during design (geometric, cartoon, honeycomb pattern, soft illustrated) before the soft illustrated bee was selected.

## Distractor Products

Two distractor products were included in the survey to validate that any observed effects were specific to the blueberry container and not artifacts of condition assignment:

- **Oreo cookies:** Universally recognized, no environmental or animal associations expected.
- **Huel (meal replacement):** Health-focused product, no animal associations expected.

These were chosen because they are unrelated to insecticide use and would not prime animal thinking.

## Sample

1,027 U.S. adults recruited via Prolific across 5 waves, April 11-12, 2026. $4.00 compensation. Median completion: 10 minutes.

## Key Results

- **Animal association:** 3.6% (control) to 18.0% (Soft Bee) and 17.3% (Pollinator), both p<0.001, Cramer's V = 0.236 and 0.223
- **Environmental association doubled:** 17.6% to 37.0% and 39.2%
- **Purchase intent improved:** 83.0% and 81.3% vs 74.7% (p<0.05)
- **Plant-based interest:** 44.3% (Soft Bee) and 39.1% (Pollinator) said increased after viewing seal
- **Distractor products showed no cross-condition differences** (all p>0.05)

## Published Links

- Report: https://gateway-report-deploy.vercel.app
- Data and Materials: https://osf.io/73njw/
- DOI: https://doi.org/10.17605/OSF.IO/73NJW

## Lessons Learned

1. **LLM judges disagreed with real humans on which seal would win.** Sonnet picked "Every Creature Counts" (text-based). Opus picked "Wildlife Friendly." Real humans responded most to the soft bee with a face. Pre-test for design issues, not predictions.

2. **Demographic balance matters.** Early rounds had too many vegetarians in some arms due to small sample sizes. At 1,027 the imbalance resolved naturally.

3. **Iterative survey refinement is normal.** We modified question wording, response options, and images across multiple rounds based on reviewing the live survey. The Typeform API makes this possible without recreating forms.

4. **The "gateway question" (plant-based interest) should reference the seal explicitly and only appear for treatment groups.** Including it in control creates an apples-to-oranges comparison.

5. **Hi-res images matter.** Low-resolution seal composites were unreadable on Typeform. Rendering at 2x via headless Chrome solved this.

6. **The distractor product validation (Table 3/4 in the report) is the single most important credibility element.** It proves the effect is real, not an artifact.
