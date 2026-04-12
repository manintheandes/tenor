# Phase 5: Launch and Monitor

Create a Prolific study, publish it, monitor response collection, and handle mid-study adjustments. This phase covers everything from first publish through final response.

## Prerequisites

Before starting this phase:
- Phase 4 (pre-test) is complete and the user has approved launch
- Typeform surveys are created and tested end-to-end
- Typeform thank-you screens redirect to Prolific completion URL
- Survey randomization router is deployed and verified
- User has a funded Prolific account with an API token

Confirm the user has their Prolific API token set:

```bash
export PROLIFIC_API_TOKEN='their-token-here'
```

## Prolific API Reference

**Base URL:** `https://api.prolific.com/api/v1`

**Authentication:** All requests require the header:
```
Authorization: Token {PROLIFIC_API_TOKEN}
```

**Key endpoints:**

| Action | Method | Endpoint | Body |
|--------|--------|----------|------|
| Check balance | GET | `/users/me/` | (none) |
| Create study | POST | `/studies/` | Study JSON (see below) |
| Publish study | POST | `/studies/{id}/transition/` | `{"action": "PUBLISH"}` |
| Pause study | POST | `/studies/{id}/transition/` | `{"action": "PAUSE"}` |
| Resume paused study | POST | `/studies/{id}/transition/` | `{"action": "PUBLISH"}` |
| Stop study | POST | `/studies/{id}/transition/` | `{"action": "STOP"}` |
| Expand study | PATCH | `/studies/{id}/` | `{"total_available_places": N}` |
| Get study status | GET | `/studies/{id}/` | (none) |
| List submissions | GET | `/studies/{id}/submissions/` | (none) |

## Step 1: Verify Account and Balance

Before creating anything, verify the token works and check the wallet balance.

```python
import requests, os, json

API_BASE = "https://api.prolific.com/api/v1"
TOKEN = os.environ["PROLIFIC_API_TOKEN"]
HEADERS = {
    "Authorization": f"Token {TOKEN}",
    "Content-Type": "application/json",
}

# Verify auth and check balance
resp = requests.get(f"{API_BASE}/users/me/", headers=HEADERS)
resp.raise_for_status()
user = resp.json()
print(f"Authenticated as: {user.get('name', user.get('email'))}")
print(f"Balance: {user.get('balance', 'check dashboard')}")
```

Calculate the required budget before proceeding:

```
total_cost = n_participants * reward_cents * 1.43  (reward + ~43% Prolific fees)
```

If the wallet balance is insufficient, tell the user exactly how much to add. Do not proceed until funded.

## Step 2: Create the Study

### Complete Study Creation Payload

Every field below is required for a working study. Adapt values to match the current study design.

```json
{
  "name": "Product Survey",
  "internal_name": "study-internal-name-YYYYMMDD",
  "description": "We're studying how consumers think about food products. You'll look at several products and answer questions about your impressions. Takes about 12 minutes.",
  "external_study_url": "https://your-router.vercel.app?PROLIFIC_PID={{%PROLIFIC_PID%}}&STUDY_ID={{%STUDY_ID%}}&SESSION_ID={{%SESSION_ID%}}",
  "prolific_id_option": "url_parameters",
  "completion_codes": [
    {
      "code": "YOUR_COMPLETION_CODE",
      "code_type": "COMPLETED",
      "actions": [{"action": "AUTOMATICALLY_APPROVE"}]
    }
  ],
  "total_available_places": 200,
  "estimated_completion_time": 12,
  "maximum_allowed_time": 25,
  "reward": 400,
  "device_compatibility": ["desktop", "tablet", "mobile"],
  "peripheral_requirements": [],
  "filters": [],
  "study_labels": ["survey"]
}
```

### Field-by-field guidance

**name** - The public title participants see. Keep it generic. "Product Survey" or "Food Choices" works. Do NOT reveal the research focus (no mention of seals, labels, sustainability, or the IV).

**internal_name** - Your private label. Include a version number or date suffix so you can distinguish waves later.

**description** - Neutral framing only. Mention the general topic (food products, shopping preferences), approximate duration, and eligibility. Do not mention the independent variable.

**external_study_url** - The survey router URL with three Prolific URL parameters:
- `PROLIFIC_PID` - Participant's unique ID (use double curly braces: `{{%PROLIFIC_PID%}}`)
- `STUDY_ID` - The Prolific study ID
- `SESSION_ID` - The session ID for this submission

These parameters must use Prolific's template syntax exactly: `{{%PROLIFIC_PID%}}`, `{{%STUDY_ID%}}`, `{{%SESSION_ID%}}`. The router passes them through to Typeform as hidden fields.

**prolific_id_option** - Always `"url_parameters"`. This tells Prolific to append participant IDs via URL params rather than asking participants to paste them.

**completion_codes** - The code must exactly match what the Typeform thank-you screen redirects to. If Typeform redirects to `https://app.prolific.com/submissions/complete?cc=MYCODE`, the code here must be `MYCODE`. Use `AUTOMATICALLY_APPROVE` for auto-approval or `MANUALLY_REVIEW` if you want to inspect submissions first.

**total_available_places** - Number of participants. For between-subjects designs with a randomization router, this is the total across all arms (the router handles assignment). Set to the target sample size for this wave.

**estimated_completion_time** - Minutes. Must be honest. Prolific flags studies where actual times deviate significantly. Use the median time from LLM pre-testing as a baseline, but expect real humans to take longer.

**maximum_allowed_time** - Minutes. Set to roughly 2x the estimated time. Participants who exceed this are timed out.

**reward** - In cents. $4.00 = 400. Prolific enforces a minimum hourly rate (currently ~$8/hour). For a 12-minute study, the minimum reward is roughly $1.60, but paying above minimum ($3-5 for 10-15 minutes) improves recruitment speed and data quality.

### Create the study

```python
payload = {
    # ... the JSON above with your values ...
}

resp = requests.post(
    f"{API_BASE}/studies/",
    headers=HEADERS,
    json=payload,
)

if not resp.ok:
    print(f"ERROR: {resp.status_code}")
    print(resp.json())
    # Common errors:
    # 400 - Invalid payload (check field names and types)
    # 402 - Insufficient funds
    # 422 - Validation error (e.g., reward below minimum)
else:
    study = resp.json()
    study_id = study["id"]
    print(f"Created study: {study_id}")
    print(f"Status: {study['status']}")  # Should be UNPUBLISHED
    print(f"Dashboard: https://app.prolific.com/researcher/studies/{study_id}")
```

Save the study ID. You will need it for publishing, monitoring, and multi-wave blocking.

## Step 3: Publish

Publishing makes the study visible to participants and begins spending money. Double-check everything first.

**Pre-publish checklist:**
- [ ] Survey router URL is live and returns correct Typeform for each arm
- [ ] Typeform thank-you screens redirect to Prolific with the correct completion code
- [ ] Completion code in Prolific matches the redirect URL
- [ ] Wallet has sufficient balance
- [ ] `blocked_previous_studies` is set if this is a follow-up wave (see Multi-Wave section)

```python
resp = requests.post(
    f"{API_BASE}/studies/{study_id}/transition/",
    headers=HEADERS,
    json={"action": "PUBLISH"},
)

if resp.ok:
    result = resp.json()
    print(f"Published. Status: {result.get('status')}")
else:
    print(f"Publish failed: {resp.status_code}")
    print(resp.json())
    # Common failure: insufficient balance
```

## Step 4: Monitor Responses

Once published, monitor response counts across all Typeform surveys. The goal is to track per-arm recruitment so you can catch imbalances early.

### Poll Typeform response counts

```python
TYPEFORM_TOKEN = os.environ["TYPEFORM_API_TOKEN"]
TYPEFORM_HEADERS = {
    "Authorization": f"Bearer {TYPEFORM_TOKEN}",
}

# form_ids is a dict: {"Control": "abc123", "Treatment A": "def456", ...}
for arm_name, form_id in form_ids.items():
    resp = requests.get(
        f"https://api.typeform.com/forms/{form_id}/responses",
        headers=TYPEFORM_HEADERS,
        params={"page_size": 1},  # We only need the total_items count
    )
    resp.raise_for_status()
    data = resp.json()
    count = data["total_items"]
    print(f"  {arm_name}: {count} responses")
```

### Monitor Prolific study status

```python
resp = requests.get(f"{API_BASE}/studies/{study_id}/", headers=HEADERS)
resp.raise_for_status()
study = resp.json()
print(f"Places taken: {study['places_taken']} / {study['total_available_places']}")
print(f"Status: {study['status']}")
```

### What to watch for

- **Arm imbalance:** If the router uses simple random assignment, small samples may produce uneven arms. With 200 participants across 3 arms, expect roughly 65-68 per arm, but 55-80 is normal. Only intervene if the imbalance is extreme.
- **Completion rate:** If `places_taken` on Prolific is much higher than total Typeform responses, participants are starting but not finishing. This may indicate a survey that is too long, confusing, or broken.
- **Speed:** Most Prolific studies with 200+ eligible participants fill within 1-4 hours. If recruitment stalls, the reward may be too low or filters too narrow.

## Step 5: Pause, Resume, or Stop

**Pause** - Temporarily stops recruitment. Participants who already started can finish. Use this if you discover a survey issue and need time to fix it.

```python
requests.post(
    f"{API_BASE}/studies/{study_id}/transition/",
    headers=HEADERS,
    json={"action": "PAUSE"},
)
```

**Resume** - Unpause a paused study. Uses the same PUBLISH transition.

```python
requests.post(
    f"{API_BASE}/studies/{study_id}/transition/",
    headers=HEADERS,
    json={"action": "PUBLISH"},
)
```

**Stop** - Permanently ends recruitment. Cannot be undone. Only use when you have enough responses or are abandoning the study.

```python
requests.post(
    f"{API_BASE}/studies/{study_id}/transition/",
    headers=HEADERS,
    json={"action": "STOP"},
)
```

## Step 6: Expand an Active Study

If the study is still active and you want more participants, increase `total_available_places`:

```python
new_total = current_places + additional_needed

resp = requests.patch(
    f"{API_BASE}/studies/{study_id}/",
    headers=HEADERS,
    json={"total_available_places": new_total},
)
```

This only works on active (published) studies. The new total must be greater than the current `places_taken`. Expanding requires sufficient wallet balance for the additional places.

## Multi-Wave Recruitment

When you need more participants than a single wave can deliver (wallet constraints, iterative sample sizing, or follow-up waves), create new studies that block previous participants.

### Critical rule: blocked_previous_studies must be set BEFORE publishing

The `blocked_previous_studies` field cannot be modified after a study is published. If you forget to set it, you must create a new study. There is no fix for a published study missing its blocklist.

### Wave 1

Create and publish normally. Save the study ID.

```python
wave1_id = "abc123..."  # from the creation response
```

### Wave 2

Create a new study with all previous study IDs in the blocklist:

```python
wave2_payload = {
    # ... same fields as wave 1 ...
    "blocked_previous_studies": [wave1_id],
}

resp = requests.post(f"{API_BASE}/studies/", headers=HEADERS, json=wave2_payload)
wave2_id = resp.json()["id"]

# Verify the blocklist is set BEFORE publishing
resp = requests.get(f"{API_BASE}/studies/{wave2_id}/", headers=HEADERS)
study = resp.json()
assert wave1_id in str(study)  # Confirm blocklist is present

# Now publish
requests.post(
    f"{API_BASE}/studies/{wave2_id}/transition/",
    headers=HEADERS,
    json={"action": "PUBLISH"},
)
```

### Wave 3+

Each new wave blocks ALL previous waves:

```python
wave3_payload = {
    # ...
    "blocked_previous_studies": [wave1_id, wave2_id],
}
```

### Handling insufficient wallet balance

If the wallet cannot cover the full target n, try progressively smaller batches:

```python
target_n = 500
reward_cents = 400
fee_multiplier = 1.43  # reward + Prolific fees

# Get current balance
resp = requests.get(f"{API_BASE}/users/me/", headers=HEADERS)
balance = resp.json().get("balance", 0)  # in cents

# Calculate max affordable n
max_n = int(balance / (reward_cents * fee_multiplier))

if max_n >= target_n:
    n = target_n
elif max_n >= 100:
    n = max_n  # Take what we can afford
    print(f"Wallet only covers {n} of {target_n}. Will need another wave.")
else:
    print(f"Wallet too low for meaningful wave ({max_n} participants). Fund the account.")
    # Do not create a study with fewer than ~50 participants per wave
```

Create the study with the affordable n, then create follow-up waves as the wallet is refunded.

### Tracking waves

Maintain a list of all study IDs across waves. Every new wave must block every previous ID.

```python
all_study_ids = []

# After each wave:
all_study_ids.append(new_study_id)

# For the next wave:
next_payload["blocked_previous_studies"] = all_study_ids.copy()
```

## Live Survey Editing

Typeform surveys can be modified while responses are being collected. This is useful for fixing typos, swapping images, adjusting question wording, or adding/removing questions mid-study.

**Warning:** Editing a live survey changes it for all future respondents. Responses already collected used the old version. If the change is significant (different question wording, different images), note when the change was made and how many responses were collected before vs. after. This may need to be reported in the limitations section.

### Read the current form

```python
TYPEFORM_BASE = "https://api.typeform.com"
TYPEFORM_HEADERS = {
    "Authorization": f"Bearer {TYPEFORM_TOKEN}",
    "Content-Type": "application/json",
}

resp = requests.get(
    f"{TYPEFORM_BASE}/forms/{form_id}",
    headers=TYPEFORM_HEADERS,
)
resp.raise_for_status()
form = resp.json()
```

### Update the entire form

Typeform's update endpoint replaces the entire form definition. The pattern is: GET the current form, modify the fields you want to change, PUT the full form back.

```python
# Modify the form object
# Example: change a question title
for field in form["fields"]:
    if field["ref"] == "q5_bb_associations":
        field["title"] = "Updated question text here"

# PUT the full form back
resp = requests.put(
    f"{TYPEFORM_BASE}/forms/{form_id}",
    headers=TYPEFORM_HEADERS,
    json=form,
)
if resp.ok:
    print(f"Form {form_id} updated successfully")
else:
    print(f"Update failed: {resp.status_code} {resp.text[:200]}")
```

### Common live edits

**Fix a typo in a question:**

```python
for field in form["fields"]:
    if field["ref"] == "target_ref":
        field["title"] = "Corrected question text"
```

**Swap an image on a question:**

```python
# Upload the new image first
import base64

with open("new_image.png", "rb") as f:
    img_data = base64.b64encode(f.read()).decode()

resp = requests.post(
    f"{TYPEFORM_BASE}/images",
    headers=TYPEFORM_HEADERS,
    json={"image": img_data, "file_name": "new_image.png"},
)
new_image_url = resp.json()["src"]

# Update the field's attachment
for field in form["fields"]:
    if field["ref"] == "product_bb_intro":
        field["attachment"] = {"type": "image", "href": new_image_url}
```

**Update choices on a multiple-choice question:**

```python
for field in form["fields"]:
    if field["ref"] == "q5_bb_associations":
        field["properties"]["choices"] = [
            {"label": "Health"},
            {"label": "Quality"},
            {"label": "Environment"},
            {"label": "Animal welfare"},
            {"label": "Food safety"},
            {"label": "Taste"},
        ]
```

**Add a new question:**

```python
new_field = {
    "ref": "q_new_question",
    "title": "New question text",
    "type": "multiple_choice",
    "properties": {
        "choices": [{"label": "Option A"}, {"label": "Option B"}],
        "randomize": False,
        "allow_multiple_selection": False,
        "allow_other_choice": False,
    },
    "validations": {"required": True},
}

# Insert at the desired position
form["fields"].insert(desired_index, new_field)
```

**Remove a question:**

```python
form["fields"] = [f for f in form["fields"] if f["ref"] != "ref_to_remove"]
```

### Applying edits across all arms

When the study has multiple Typeform forms (one per arm, or separate control/treatment forms), apply the same edit to every form. Loop over all form IDs:

```python
for arm_name, form_id in form_ids.items():
    resp = requests.get(f"{TYPEFORM_BASE}/forms/{form_id}", headers=TYPEFORM_HEADERS)
    form = resp.json()

    # Apply the edit (same logic for each form)
    for field in form["fields"]:
        if field["ref"] == "target_ref":
            field["title"] = "Updated text"

    resp = requests.put(
        f"{TYPEFORM_BASE}/forms/{form_id}",
        headers=TYPEFORM_HEADERS,
        json=form,
    )
    print(f"  {arm_name}: {'OK' if resp.ok else 'FAILED'}")
```

## Phase Completion

This phase is complete when:
- All target responses have been collected (check Typeform counts, not just Prolific places_taken)
- The Prolific study is stopped or fully filled
- All study IDs and wave information are saved for reference in the analysis phase

Proceed to Phase 6 (Analyze) once the target sample size is reached.
