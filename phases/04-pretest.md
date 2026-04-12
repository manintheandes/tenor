# Phase 4: Pre-test with LLM Judges

Pre-test the survey instrument by running simulated respondents through each arm. This catches design problems before real money is spent on recruitment.

**Input:** Completed survey (Typeform surveys created in Phase 3), stimulus images for each arm.

**Output:** Interactive results dashboard, list of flagged issues, go/no-go recommendation.

---

## Mandatory Caveat

Before showing any pre-test results to the user, display this notice prominently:

> **LLM judges provide directional signal only. They cannot predict real human responses. This pre-test catches survey design issues, not consumer behavior. Real results may differ substantially.**

Do not bury this. It goes at the top of every dashboard and at the top of any summary you present.

---

## Step 1: Generate Personas

Create simulated respondents with realistic demographic distributions. Do not use uniform random sampling. Real populations are not uniformly distributed, and uniform sampling produces unrealistic panels.

### Demographic Weights

Use `random.choices` with the `weights` parameter for each dimension:

**Age:**
| Range | Weight |
|-------|--------|
| 18-24 | 12% |
| 25-34 | 25% |
| 35-44 | 22% |
| 45-54 | 18% |
| 55-64 | 13% |
| 65+   | 10% |

**Gender:**
| Identity | Weight |
|----------|--------|
| Female | 50% |
| Male | 47% |
| Non-binary | 3% |

**Diet:**
| Type | Weight |
|------|--------|
| Omnivore | 55% |
| Flexitarian | 20% |
| Pescatarian | 5% |
| Vegetarian | 12% |
| Vegan | 8% |

### Additional Persona Dimensions

Add study-relevant dimensions with reasonable weights. For a grocery/food product study, include:

- **Income bracket** (6 tiers, skewed toward middle)
- **Primary grocery store type** (conventional, natural/organic, mass retailer, online, farmers market)
- **Purchase frequency for the product category** (daily through rarely)

### Persona Construction

```python
import random

AGES = ["18-24", "25-34", "35-44", "45-54", "55-64", "65+"]
AGE_WEIGHTS = [0.12, 0.25, 0.22, 0.18, 0.13, 0.10]

GENDERS = ["Female", "Male", "Non-binary"]
GENDER_WEIGHTS = [0.50, 0.47, 0.03]

DIETS = ["Omnivore", "Flexitarian", "Pescatarian", "Vegetarian", "Vegan"]
DIET_WEIGHTS = [0.55, 0.20, 0.05, 0.12, 0.08]

def make_persona():
    return {
        "age": random.choices(AGES, weights=AGE_WEIGHTS, k=1)[0],
        "gender": random.choices(GENDERS, weights=GENDER_WEIGHTS, k=1)[0],
        "diet": random.choices(DIETS, weights=DIET_WEIGHTS, k=1)[0],
        # Add other dimensions with their own weighted pools
    }
```

### Sample Size

Run 50 to 100 simulated respondents per arm. More arms means you can use the lower end. Fewer arms (2 to 3) means use 75 to 100 per arm for stable percentages.

---

## Step 2: Execute Judge Calls

Each simulated respondent is a single API call to a vision-capable model. The model receives the stimulus image and a prompt containing the persona, survey questions, and response format.

### Model Selection

Use **Claude Sonnet** (e.g., `claude-sonnet-4-20250514`). It balances cost, speed, and instruction-following for this task. Do not use Opus for pre-testing; the cost scales with judge count and the marginal quality gain does not justify it.

### Image Encoding

Load each stimulus image as base64:

```python
import base64

def encode_image(path):
    with open(path, "rb") as f:
        return base64.standard_b64encode(f.read()).decode("utf-8")
```

### Prompt Structure

The prompt has four parts in this order:

1. **Role and persona:** "You are simulating a real consumer taking a product survey." Include all persona fields.
2. **Image instruction:** "You are looking at [product description] (shown in the image). Look at it carefully as a real shopper would."
3. **JSON response schema:** Every survey question with its key, response type, and valid options. Use the exact question wording from the Typeform survey.
4. **Realism instruction:** This is critical. Without it, LLMs over-index on "correct" or "thoughtful" answers that real consumers would never give.

The realism instruction must include language like:

> Be realistic. Most consumers do NOT think deeply about [topic]. They care about price, taste, and convenience. Only deviate from that baseline if something in the image genuinely catches your attention. Do not over-index on "correct" answers. Real survey data is messy.

Replace `[topic]` with whatever the study is actually about (e.g., "animals," "sustainability," "environmental impact").

### API Call Pattern

```python
import anthropic
import json
import time

client = anthropic.Anthropic()

def run_judge(persona, arm_key, image_path, prompt_text):
    img_b64 = encode_image(image_path)

    for attempt in range(3):  # 3 retries on parse failure
        try:
            response = client.messages.create(
                model="claude-sonnet-4-20250514",
                max_tokens=1500,
                messages=[{
                    "role": "user",
                    "content": [
                        {
                            "type": "image",
                            "source": {
                                "type": "base64",
                                "media_type": "image/png",
                                "data": img_b64,
                            },
                        },
                        {"type": "text", "text": prompt_text},
                    ],
                }],
            )

            text = response.content[0].text.strip()

            # Strip markdown code fences if present
            if text.startswith("```"):
                text = text.split("```")[1]
                if text.startswith("json"):
                    text = text[4:]

            result = json.loads(text)
            result["_arm"] = arm_key
            result["_persona"] = persona
            return result

        except json.JSONDecodeError:
            print(f"  JSON parse failed, retry {attempt + 1}/3")
            time.sleep(1)
        except Exception as e:
            print(f"  API error: {e}, retry {attempt + 1}/3")
            time.sleep(2)

    return None  # All retries exhausted
```

### Rate Limiting

Wait **0.3 seconds** between calls (`time.sleep(0.3)`). This prevents hitting API rate limits while keeping total runtime reasonable. For 4 arms at 50 judges each, expect roughly 10 to 15 minutes of runtime.

### Execution Loop

Iterate over arms, then judges within each arm. Print progress so the user sees it moving:

```python
results = []
for arm_key, arm_info in arms.items():
    for i in range(judges_per_arm):
        persona = make_persona()
        print(f"  [{i+1}/{judges_per_arm}] {arm_key} | "
              f"{persona['age']} {persona['gender']} {persona['diet']}")

        result = run_judge(persona, arm_key, arm_info["image"], ...)
        if result:
            results.append(result)
        time.sleep(0.3)
```

Save raw results to a JSON file after all judges complete. Include persona data and arm assignment in each record.

---

## Step 3: Analyze Results

After all judges have run, compute these analyses. Each one catches a specific class of survey design problem.

### 3a: Primary Outcome Comparison

For the study's primary dependent variable, compute the percentage for each arm and compare treatment arms to control.

```
Control:     22%  (11/50)
Treatment A: 48%  (24/50)  +26pp vs control
Treatment B: 44%  (22/50)  +22pp vs control
```

**What to look for:** Directional signal that treatment arms differ from control. If they do not differ at all, the stimulus may not be visible or the question may not be sensitive enough to detect it.

### 3b: Flag Control > 5% on Target (Question Leading)

If the control arm scores above 5% on the target response for the primary unprompted question, the question wording may be leading respondents toward that answer even without seeing the treatment.

**Example:** If the study tests whether a "bee-friendly" seal increases selection of "How insects and animals are affected," but 20% of control judges also select it, the option wording itself is priming the response. Consider rewording or rebalancing options.

**Threshold:** Control > 5% on the target response for the primary outcome. Flag it. Do not automatically fail the pre-test, but surface it clearly.

### 3c: Flag Any Option > 80% Across ALL Arms (Biased Option)

If any single response option is selected by more than 80% of judges in every arm (including control), that option is too attractive regardless of treatment. It is a ceiling effect.

**Example:** If "Health" is selected by 90% of judges in control AND 88% in treatment, the option dominates the question and reduces the sensitivity of the measure. Consider splitting it into sub-options or removing it and measuring health separately.

**Check this across all questions, not just the primary outcome.**

### 3d: Flag Treatment Effect = 0% (Stimulus Not Visible)

If no treatment arm shows any difference from control on the primary outcome, the stimulus is likely not noticeable in the image. Possible causes:

- Image too small or low resolution
- Stimulus blends into background
- Stimulus is cropped or partially hidden
- The survey question does not map to what the stimulus communicates

Surface this with a clear recommendation: check the stimulus images, zoom level, and question wording.

### 3e: Secondary Metrics

For all other survey questions (scales, multi-select, open-ended), compute arm-level summaries:

- **Likert scales:** Mean and standard deviation per arm
- **Multi-select:** Percentage selecting each option per arm
- **Open-ended:** Collect representative quotes per arm (useful for the dashboard)

---

## Step 4: Build the Dashboard

Use `/frontend-design` to create an interactive HTML dashboard that presents the pre-test results visually.

### Dashboard Structure

The dashboard is a single HTML file with embedded CSS and JavaScript. No external dependencies. It must work when opened directly in a browser.

### Required Sections

**1. Caveat banner (top of page, always visible):**

Display the mandatory caveat text (from the top of this document) in a prominent banner. Use a yellow/amber background. It should not be dismissable.

**2. Bar charts for primary outcome:**

One grouped bar chart showing each arm's performance on the primary dependent variable. Use distinct colors per arm. Label bars with percentages. Highlight the control arm in a neutral color (gray) and treatment arms in the study's color scheme.

**3. Heatmap for multi-select questions:**

A grid where rows are response options and columns are arms. Cell color intensity represents selection percentage. This makes it easy to spot options that are uniformly high (biased) or options that only spike in treatment arms (signal).

**4. Scale comparison table:**

For Likert-scale questions, show mean values per arm in a table with conditional formatting (darker background for higher values).

**5. Open-ended quotes:**

A section showing selected open-ended responses grouped by arm. Pick 3 to 5 representative quotes per arm. These give qualitative texture to the quantitative results.

**6. Flags and warnings:**

A dedicated section listing every flagged issue from Step 3 (leading questions, biased options, invisible stimuli). Use red/orange severity indicators. Each flag should include:
- What was detected
- Which question/option triggered it
- Suggested fix

### Chart Implementation

Use inline SVG or HTML/CSS bar charts. Do not require Chart.js or D3 or any CDN. The dashboard must be fully self-contained.

For bar charts, simple `<div>` elements with percentage-based widths work well:

```html
<div class="bar" style="width: 48%; background: #4A90D9;">
  Treatment A: 48%
</div>
```

For heatmaps, use a `<table>` with background-color interpolation based on the percentage value. Map 0% to white and 100% to a saturated color.

---

## Step 5: Present Results to User

After the dashboard is built, present findings to the user with this structure:

1. **Repeat the caveat.** Every time.
2. **Primary outcome summary.** One sentence: "Treatment arms showed [X]pp higher selection of [target] compared to control."
3. **Flagged issues.** List each flag with severity and suggested fix.
4. **Go/no-go recommendation.** Based on flags:
   - **No flags:** "Pre-test looks clean. Ready for Phase 5 (launch)."
   - **Minor flags (1 to 2, non-critical):** "Pre-test surfaced [N] issues. Consider addressing them before launch. Details in the dashboard."
   - **Major flags (leading question, invisible stimulus):** "Pre-test found significant design issues. Recommend revising before spending on real participants."
5. **Link to dashboard HTML file.**

Wait for user approval before advancing to Phase 5.

---

## Common Pitfalls

**LLMs are too thoughtful.** Real consumers spend 15 seconds on a grocery purchase decision. LLMs analyze every pixel. The realism instruction mitigates this but does not eliminate it. Expect LLM judges to over-select "Environment" and "Animals" compared to real humans.

**JSON parse failures.** Some responses come back with commentary before or after the JSON. The markdown fence stripping handles most cases. Three retries is enough; if a judge fails all three, skip it and move on.

**Treating pre-test as prediction.** The pre-test tells you whether your survey instrument works, not what real consumers will do. If LLM judges show 0% treatment effect, your stimulus is probably broken. If they show a 30% effect, real humans might show 5% or 25%. The magnitude is not predictive. The direction is sometimes predictive. The presence or absence of any signal is the most useful output.

**Running too few judges.** Below 30 per arm, percentage differences are noisy enough to be meaningless. 50 is the practical minimum.
