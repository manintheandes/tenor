# Phase 6: Analyze

Pull all responses from Typeform, anonymize the data, run statistical tests, validate distractor products, and produce output files for the report phase.

## Prerequisites

- Phase 5 complete: responses have reached the target sample size
- `typeform_urls.json` exists with form IDs per arm (created in Phase 3)
- Typeform API token available
- Python with scipy, numpy, pandas, and requests installed

## Step 1: Retrieve Responses from Typeform

Typeform's Responses API returns a maximum of 200 responses per request. For studies with more than 200 responses per form, you must paginate using the `before` token.

The `before` parameter takes the token of the oldest response in the previous page. Typeform returns responses newest-first, so each page moves backward in time.

### Complete retrieval pattern

```python
import requests
import json

TYPEFORM_TOKEN = "YOUR_TOKEN"
HEADERS = {
    "Authorization": f"Bearer {TYPEFORM_TOKEN}",
}

def fetch_all_responses(form_id):
    """Fetch every response from a Typeform, handling pagination."""
    all_items = []
    url = f"https://api.typeform.com/forms/{form_id}/responses"
    params = {"page_size": 200}

    while True:
        resp = requests.get(url, headers=HEADERS, params=params)
        resp.raise_for_status()
        data = resp.json()
        items = data.get("items", [])
        if not items:
            break
        all_items.extend(items)
        # Stop if we got fewer than a full page
        if len(items) < 200:
            break
        # Use the oldest response's token as the 'before' cursor
        params["before"] = items[-1]["token"]

    return all_items
```

Load form IDs from the file created in Phase 3:

```python
with open("typeform_urls.json") as f:
    forms = json.load(f)

raw_responses = {}
for arm_name, info in forms.items():
    form_id = info["form_id"]
    responses = fetch_all_responses(form_id)
    raw_responses[arm_name] = responses
    print(f"{arm_name}: {len(responses)} responses")
```

## Step 2: Parse Responses

Typeform answers arrive as a list of objects, each with a `type` field and a `field.ref` that maps back to your question refs. Parse each answer type into clean Python values.

### Answer type mapping

| Typeform `type` | Raw structure | Clean value |
|---|---|---|
| `choice` | `{"label": "Some option"}` | `str` - the label |
| `choices` | `{"labels": ["A", "B"]}` | `list[str]` - all selected labels |
| `number` | `3` | `int` or `float` |
| `text` | `"free text here"` | `str` |
| `boolean` | `true` | `bool` |

### Parsing function

```python
def parse_answer(answer):
    """Convert a single Typeform answer object to a clean Python value."""
    t = answer["type"]
    if t == "choice":
        return answer["choice"]["label"]
    elif t == "choices":
        return answer["choices"]["labels"]
    elif t == "number":
        return answer["number"]
    elif t == "text":
        return answer["text"]
    elif t == "boolean":
        return answer["boolean"]
    else:
        return None


def parse_response(response):
    """Convert one Typeform response into a flat dict keyed by field ref."""
    parsed = {}
    for answer in response.get("answers", []):
        ref = answer["field"]["ref"]
        parsed[ref] = parse_answer(answer)
    return parsed
```

### Build the clean dataset

```python
clean_data = {}
for arm_name, responses in raw_responses.items():
    arm_key = arm_name.lower().replace(" ", "_")
    parsed_list = []
    for r in responses:
        row = parse_response(r)
        row["_arm"] = arm_key
        row["_submitted"] = r.get("submitted_at", "")
        row["_prolific_pid"] = r.get("hidden", {}).get("prolific_pid", "")
        parsed_list.append(row)
    clean_data[arm_key] = parsed_list
```

## Step 3: Anonymize

Strip Prolific IDs, timestamps, and any other identifying fields. Assign sequential IDs per arm.

```python
def anonymize(clean_data):
    """Remove PII, assign sequential participant IDs per arm."""
    anonymized = {}
    for arm_key, rows in clean_data.items():
        anon_rows = []
        for i, row in enumerate(rows):
            anon = dict(row)
            # Remove identifying fields
            anon.pop("_prolific_pid", None)
            anon.pop("_submitted", None)
            # Assign sequential ID
            anon["participant_id"] = f"{arm_key}_{i+1:04d}"
            anon_rows.append(anon)
        anonymized[arm_key] = anon_rows
    return anonymized

anonymized = anonymize(clean_data)
```

## Step 4: Demographic Balance Check

Before running hypothesis tests, confirm that the arms are balanced on demographics. If any demographic variable differs significantly across conditions, it is a confound.

Run chi-square tests on gender, age, and diet distributions across arms.

```python
import numpy as np
from scipy.stats import chi2_contingency
import pandas as pd

# Build a flat dataframe
all_rows = []
for arm_key, rows in anonymized.items():
    for row in rows:
        all_rows.append(row)
df = pd.DataFrame(all_rows)

def check_balance(df, column, arm_col="_arm"):
    """Run chi-square on a demographic variable across arms."""
    ct = pd.crosstab(df[arm_col], df[column])
    chi2, p, dof, expected = chi2_contingency(ct.values)
    balanced = "PASS" if p >= 0.05 else "FAIL"
    print(f"  {column}: chi2={chi2:.2f}, p={p:.4f} [{balanced}]")
    return {"chi2": round(chi2, 2), "p": round(p, 4), "balanced": p >= 0.05}

print("Demographic balance across arms:")
balance_results = {}
for col in ["q13_age", "q14_gender", "q16_diet"]:
    if col in df.columns:
        balance_results[col] = check_balance(df, col)
```

If any variable shows p < 0.05, note it as a limitation. Consider whether the imbalance could explain treatment effects.

## Step 5: Statistical Tests

Run the primary hypothesis tests. Every comparison must include the test statistic, p-value, and effect size.

### Chi-square test of independence

Use for categorical outcomes (e.g., "Did participants select 'Impact on animals' from the association list?").

```python
from scipy.stats import chi2_contingency

# Example: comparing "Impact on animals" selection between control and treatment
# c_yes = number in control who selected it
# c_no = number in control who did not
# t_yes = number in treatment who selected it
# t_no = number in treatment who did not
table = [[c_yes, c_no], [t_yes, t_no]]
chi2, p, dof, expected = chi2_contingency(table)
```

### Cramer's V (effect size for chi-square)

```python
import numpy as np

# n = total number of observations in the table
n = c_yes + c_no + t_yes + t_no
v = np.sqrt(chi2 / n)
```

Interpretation: V < 0.1 negligible, 0.1-0.3 small, 0.3-0.5 medium, > 0.5 large.

### Mann-Whitney U test

Use for ordinal or non-normal continuous outcomes (e.g., Likert scale ratings).

```python
from scipy.stats import mannwhitneyu

# control = list of numeric ratings from control arm
# treatment = list of numeric ratings from treatment arm
u, p = mannwhitneyu(control, treatment, alternative='two-sided')
```

### Margin of error

Calculate for each arm's sample size. Uses 95% confidence (z = 1.96) and maximum variance assumption (p = 0.5).

```python
import math

# n = sample size for the arm
moe = 1.96 * math.sqrt(0.5 * 0.5 / n) * 100
```

### Running all primary tests

```python
def run_chi2(label, c_yes, c_no, t_yes, t_no):
    """Run chi-square with Cramer's V, return results dict."""
    table = [[c_yes, c_no], [t_yes, t_no]]
    chi2, p, dof, expected = chi2_contingency(table)
    n = c_yes + c_no + t_yes + t_no
    v = np.sqrt(chi2 / n)
    result = {
        "test": "chi-square",
        "label": label,
        "chi2": round(chi2, 2),
        "p": round(p, 4),
        "dof": dof,
        "cramers_v": round(v, 3),
        "n": n,
    }
    print(f"  {label}: chi2={chi2:.2f}, p={p:.4f}, V={v:.3f}")
    return result


def run_mann_whitney(label, control_values, treatment_values):
    """Run Mann-Whitney U, return results dict."""
    u, p = mannwhitneyu(control_values, treatment_values, alternative='two-sided')
    result = {
        "test": "mann-whitney-u",
        "label": label,
        "U": float(u),
        "p": round(p, 4),
        "control_median": float(np.median(control_values)),
        "treatment_median": float(np.median(treatment_values)),
        "control_mean": round(float(np.mean(control_values)), 2),
        "treatment_mean": round(float(np.mean(treatment_values)), 2),
    }
    print(f"  {label}: U={u:.0f}, p={p:.4f}, "
          f"ctrl_mean={result['control_mean']}, tx_mean={result['treatment_mean']}")
    return result


# Collect results
test_results = []

# Get arm data
control_rows = anonymized["control"]
# Run against each treatment arm
treatment_arms = [k for k in anonymized.keys() if k != "control"]

for tx_arm in treatment_arms:
    tx_rows = anonymized[tx_arm]
    c_n = len(control_rows)
    t_n = len(tx_rows)

    print(f"\n--- Control (n={c_n}) vs {tx_arm} (n={t_n}) ---")

    # Chi-square on association selections (multi-select questions)
    # Example: "Impact on animals" from q5_bb_associations
    association = "How insects, birds, and other animals are affected"
    c_yes = sum(1 for r in control_rows
                if association in r.get("q5_bb_associations", []))
    c_no = c_n - c_yes
    t_yes = sum(1 for r in tx_rows
                if association in r.get("q5_bb_associations", []))
    t_no = t_n - t_yes
    test_results.append(
        run_chi2(f"animal_association_{tx_arm}", c_yes, c_no, t_yes, t_no))

    # Mann-Whitney on Likert scales
    for q_ref, q_label in [
        ("q8d_animals", "thought_about_animals"),
        ("q10e_animals", "importance_animals"),
    ]:
        c_vals = [r[q_ref] for r in control_rows if q_ref in r]
        t_vals = [r[q_ref] for r in tx_rows if q_ref in r]
        if c_vals and t_vals:
            test_results.append(
                run_mann_whitney(f"{q_label}_{tx_arm}", c_vals, t_vals))

    # Margins of error
    c_moe = 1.96 * math.sqrt(0.5 * 0.5 / c_n) * 100
    t_moe = 1.96 * math.sqrt(0.5 * 0.5 / t_n) * 100
    print(f"  Margin of error: control={c_moe:.1f}%, {tx_arm}={t_moe:.1f}%")
```

## Step 6: Distractor Validation

The study includes distractor products (e.g., Oreo, Huel) to detect survey priming. If distractor associations differ across conditions, the survey itself may be biasing responses rather than the treatment stimulus.

Run chi-square on every distractor association across conditions. Flag any result with p < 0.05.

```python
distractor_results = []
distractor_products = ["oreo", "huel"]
associations = [
    "Health", "Quality", "Environment",
    "How insects, birds, and other animals are affected",
    "Food safety", "Taste",
]

for product in distractor_products:
    assoc_key = f"q5_{product}_associations"
    for assoc in associations:
        c_yes = sum(1 for r in control_rows
                    if assoc in r.get(assoc_key, []))
        c_no = len(control_rows) - c_yes

        for tx_arm in treatment_arms:
            tx_rows = anonymized[tx_arm]
            t_yes = sum(1 for r in tx_rows
                        if assoc in r.get(assoc_key, []))
            t_no = len(tx_rows) - t_yes

            table = [[c_yes, c_no], [t_yes, t_no]]
            chi2, p, dof, expected = chi2_contingency(table)
            n = c_yes + c_no + t_yes + t_no
            v = np.sqrt(chi2 / n)

            entry = {
                "product": product,
                "association": assoc,
                "comparison": f"control_vs_{tx_arm}",
                "chi2": round(chi2, 2),
                "p": round(p, 4),
                "cramers_v": round(v, 3),
                "flagged": p < 0.05,
            }
            distractor_results.append(entry)
            if p < 0.05:
                print(f"  WARNING: {product}/{assoc} differs between "
                      f"control and {tx_arm} (p={p:.4f}). "
                      f"Possible survey priming.")

flagged_count = sum(1 for d in distractor_results if d["flagged"])
print(f"\nDistractor validation: {flagged_count} of "
      f"{len(distractor_results)} tests flagged (p < 0.05)")
```

If any distractor associations are flagged, include this in the limitations section of the report. A small number of flags at p < 0.05 may be expected by chance (roughly 5% of tests). A pattern of flags on the same association across multiple distractor products is a stronger signal of priming.

## Step 7: Output Files

Produce three output files.

### anonymized_responses.json

Responses grouped by arm. This is the primary data file for the report phase.

```python
with open("anonymized_responses.json", "w") as f:
    json.dump(anonymized, f, indent=2)
print(f"Wrote anonymized_responses.json")
```

### anonymized_responses.csv

Flat file with one row per participant. Includes `participant_id` and `condition` columns. Multi-select fields are pipe-delimited.

```python
import csv

flat_rows = []
for arm_key, rows in anonymized.items():
    for row in rows:
        flat = {"participant_id": row["participant_id"], "condition": arm_key}
        for k, v in row.items():
            if k in ("participant_id", "_arm"):
                continue
            if isinstance(v, list):
                flat[k] = "|".join(v)
            else:
                flat[k] = v
        flat_rows.append(flat)

# Collect all column names
all_cols = []
seen = set()
for row in flat_rows:
    for k in row.keys():
        if k not in seen:
            all_cols.append(k)
            seen.add(k)

with open("anonymized_responses.csv", "w", newline="") as f:
    writer = csv.DictWriter(f, fieldnames=all_cols)
    writer.writeheader()
    writer.writerows(flat_rows)
print(f"Wrote anonymized_responses.csv ({len(flat_rows)} rows)")
```

### statistical_tests.json

All test results in one file: primary tests, distractor validation, demographic balance, and margins of error.

```python
output = {
    "primary_tests": test_results,
    "distractor_validation": distractor_results,
    "demographic_balance": balance_results,
    "sample_sizes": {arm: len(rows) for arm, rows in anonymized.items()},
    "margins_of_error": {
        arm: round(1.96 * math.sqrt(0.5 * 0.5 / len(rows)) * 100, 1)
        for arm, rows in anonymized.items()
    },
}

with open("statistical_tests.json", "w") as f:
    json.dump(output, f, indent=2)
print("Wrote statistical_tests.json")
```

## Checklist Before Advancing to Phase 7

- [ ] All responses retrieved (verify count matches Prolific completions)
- [ ] Anonymization complete: no Prolific IDs or timestamps in output files
- [ ] Participant IDs are sequential per arm (control_0001, control_0002, ...)
- [ ] Demographic balance checked across all arms
- [ ] All primary hypothesis tests include chi2/U statistic, p-value, and effect size
- [ ] Distractor validation complete with flags noted
- [ ] Margins of error computed for every arm
- [ ] Three output files written: anonymized_responses.json, anonymized_responses.csv, statistical_tests.json
- [ ] User has reviewed the results and approved for publication
