# Phase 3: Build the Survey

Construct the survey instrument, create Typeform forms via API, and deploy the randomization router.

## Prerequisites

- Approved study design from Phase 1 (conditions, IVs, DVs, hypotheses)
- Approved stimulus images from Phase 2 (one per condition)
- Typeform API token
- Distractor product images (minimum 2)

## Enforced Question Order

Every survey form follows this exact sequence. Do not reorder.

### 1. Introduction Statement

A `statement` field. Explain the study topic neutrally. Do not mention the treatment variable or hypothesis. Include: "There are no right or wrong answers."

### 2. Screener Questions (2 items)

Two `multiple_choice` questions that confirm the participant belongs to the target population. Examples: purchase frequency, primary shopping channel. These also serve as warm-up questions.

### 3. Target Product Viewing

A `statement` field with the stimulus image attached. Include explicit text: "Please look at the following product carefully for at least 10 seconds before continuing."

Typeform statement fields have no input, so the participant must actively click "Continue." This creates a natural minimum dwell time.

### 4. Target Product: Impression (5-point scale)

`multiple_choice` with 5 options from "Very positive" to "Very negative." Include the product name in the question text so participants know what they are rating.

### 5. Target Product: Purchase Intent (5-point scale)

`multiple_choice` with 5 options from "Definitely would" to "Definitely would not."

### 6. Target Product: Unprompted Association (multi-select, 6 balanced options)

`multiple_choice` with `allow_multiple_selection: true`. Exactly 6 options. Options must be balanced:
- Similar in word count (within 20% of each other)
- Similar in specificity
- Mix of target-relevant and target-irrelevant associations
- No option should prime the treatment variable

Example: `["Health", "Quality", "Environment", "How insects, birds, and other animals are affected", "Food safety", "Taste"]`

### 7. [Treatment Only] Seal/Treatment Meaning

`long_text` asking what the participant thinks the treatment element means. Show the stimulus image again. This is the first time the participant is prompted about the treatment.

Only include this field when `is_treatment` is `True`. Omit entirely from control forms.

### 8. Distractor Product 1

Repeat the same question sequence as the target product (steps 3 through 6):
1. Product viewing statement with image
2. Impression (5-point, identical options)
3. Purchase intent (5-point, identical options)
4. Unprompted association (identical option list)

The distractor must use the exact same response options and question structure. Any difference invalidates the distractor as a methodological control.

### 9. Distractor Product 2

Same sequence again with a second unrelated product. Same rules apply.

### 10. [Treatment Only] Open-ended: Seal/Treatment Thoughts

`long_text` asking for thoughts or feelings about the treatment element. Show the stimulus image. Only in treatment forms.

### 11. [Treatment Only] Open-ended: Farming/Living Things

`long_text` asking whether the treatment element makes them think about broader impacts (animals, environment, farming practices). Show the stimulus image. Only in treatment forms.

### 12. General Food Attitude Scales (5 items, 1-5 Likert)

`opinion_scale` fields with `start_at_one: true` and `steps: 5`. Labels: `left: "Not at all"`, `right: "A lot"`.

Five items covering:
1. Nutritional content
2. Environmental impact of food production
3. How food is grown or produced
4. Impact on animal life
5. Price of groceries

These appear in all forms (control and treatment).

### 13. Food Choice Importance (6 items, 1-5 Likert)

`opinion_scale` fields with `start_at_one: true` and `steps: 5`. Labels: `left: "Not at all important"`, `right: "Extremely important"`.

Six items covering:
1. Price
2. Taste
3. Nutritional value
4. Environmental impact
5. Impact on animal life
6. Brand reputation

These appear in all forms.

### 14. [Treatment Only] Impact Questions (5 items, Much less to Much more)

`multiple_choice` fields. Five items asking whether the treatment element changes how much the participant thinks about:
1. Personal health
2. Environment
3. How food is grown
4. Impact on animal life
5. What companies to support

Response options: `["Much less", "Somewhat less", "About the same", "Somewhat more", "Much more"]`

Show the stimulus image on each item. Only in treatment forms.

### 15. [Treatment Only] Gateway Question (3 options)

`multiple_choice` with exactly 3 options. Asks whether the treatment element made the participant think about broader topics they would not have considered otherwise. Options should capture: yes it expanded thinking, no it did not, unsure/neutral.

Only in treatment forms.

### 16. Demographics

Preceded by a `statement` field: "Finally, a few questions about you."

Standard demographic questions:
- Age (categorical brackets)
- Gender (include "Prefer not to say")
- Household income (categorical brackets, include "Prefer not to say")
- Diet type

## Typeform API Patterns

### Authentication

All requests use a Bearer token in the Authorization header:

```
Authorization: Bearer {TYPEFORM_API_TOKEN}
Content-Type: application/json
```

### Base URL

```
https://api.typeform.com
```

### Image Upload

Upload images before creating forms. Typeform needs hosted image URLs.

```
POST /images
Body: { "image": "<base64-encoded-image>", "file_name": "product.png" }
Response: { "src": "https://images.typeform.com/..." }
```

Read the file, base64-encode it, send it. Save the returned `src` URL for use in form fields.

### Form Creation

```
POST /forms
Body: Full form JSON (see field structure below)
Response: { "id": "abc123", "_links": { "display": "https://form.typeform.com/to/abc123" } }
```

### Form Update

```
PUT /forms/{form_id}
Body: Full form JSON (replaces the entire form)
```

Use this to fix questions after creation or to set redirect URLs.

### Response Retrieval

```
GET /forms/{form_id}/responses?page_size=200
```

Returns up to 200 responses per page. To paginate, use the `before` token from the response:

```
GET /forms/{form_id}/responses?page_size=200&before={token}
```

Continue until the response contains fewer items than `page_size`.

### Field Types

**statement** - Display-only. No response collected. Use for instructions and product viewing.
```json
{
  "ref": "intro", "title": "Welcome text here", "type": "statement",
  "properties": { "hide_marks": true, "button_text": "Continue" },
  "attachment": { "type": "image", "href": "https://..." }
}
```

**multiple_choice** - Single or multi-select.
```json
{
  "ref": "q1", "title": "Question text", "type": "multiple_choice",
  "properties": {
    "choices": [{ "label": "Option A" }, { "label": "Option B" }],
    "randomize": false, "allow_multiple_selection": false, "allow_other_choice": false
  },
  "validations": { "required": true },
  "attachment": { "type": "image", "href": "https://..." }
}
```

**opinion_scale** - Likert-type scale.
```json
{
  "ref": "q8a", "title": "How much have you thought about X?", "type": "opinion_scale",
  "properties": { "start_at_one": true, "steps": 5, "labels": { "left": "Not at all", "right": "A lot" } },
  "validations": { "required": true }
}
```

**long_text** - Open-ended response.
```json
{
  "ref": "q6", "title": "What are your thoughts?", "type": "long_text",
  "validations": { "required": true },
  "attachment": { "type": "image", "href": "https://..." }
}
```

### Completion Redirect

Set a thank-you screen that redirects to the Prolific completion URL:

```json
{
  "thankyou_screens": [{
    "ref": "prolific_redirect",
    "title": "Thank you for completing this survey!",
    "properties": {
      "show_button": true,
      "button_mode": "redirect",
      "button_text": "Complete Study",
      "redirect_url": "https://app.prolific.com/submissions/complete?cc=YOUR_COMPLETION_CODE",
      "share_icons": false
    }
  }]
}
```

The completion code must match what you configure in Prolific. Participants who do not reach this screen will not be marked as complete.

## Methodology Enforcement

Run these checks before finalizing any form. Violations must be fixed, not noted and skipped.

### 1. Option Length Balance

For every `multiple_choice` field, compute the character count of each option label. The longest option must be no more than 120% of the shortest option's length. If the check fails, rewrite the options to be more uniform.

Exception: Demographic questions (age brackets, income brackets) are exempt.

### 2. Treatment Word Contamination

Define a list of treatment-related words (e.g., "seal", "insecticide-free", "pollinator", "organic"). Scan every question title and option label in the survey. If any treatment word appears in a question that is not in the treatment-only block, flag it and rewrite the question.

The unprompted association question (step 6) is especially sensitive. Its options must not contain any word that directly names the treatment.

### 3. Distractor Identity Check

Export the question titles and option lists for the target product and both distractor products. The structure must be identical:
- Same question wording pattern (only the product name differs)
- Same response options in the same order
- Same field types

If any structural difference exists between target and distractor question sets, fix it.

### 4. Treatment-Only Question Isolation

List all fields with `is_treatment`-gated logic. Verify that none of these fields appear in the control form. The simplest check: create the control form JSON and the treatment form JSON, then diff the field refs. Treatment-only refs must be absent from control.

## Implementation Checklist

1. Upload all images (target product variants + distractors)
2. Build form JSON for each condition using the enforced question order
3. Run methodology checks (option length, treatment contamination, distractor identity, treatment isolation)
4. Create forms via POST /forms
5. Set completion redirect URL with Prolific completion code
6. Deploy the survey router (see `templates/survey-router.html`)
7. Test each form end-to-end: verify all images load, all questions appear, redirect works
8. Save form IDs and URLs to a JSON file for use in Phase 5

## Output

- N Typeform forms (one per condition), each with a unique URL
- A `typeform_urls.json` file mapping condition names to form IDs and URLs
- A deployed survey router that randomly assigns participants to conditions
- Evidence that methodology checks passed

## Gate: User Approval

Before advancing to Phase 4, walk the user through every question in the survey instrument. Present each question with its wording, response options, and any attached images. The user must explicitly approve the complete survey before proceeding.
