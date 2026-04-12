# Phase 7: Publish

Generate the final report, deploy it, deposit open data on OSF, and mint a DOI.

## 1. Report Generation

Invoke `/frontend-design` to build a single-page HTML report. Pass `templates/report-css.md` as the design system reference. The report must be self-contained (inline CSS, embedded images via relative paths or base64).

### Report Structure (enforced, in order)

1. **Cover page** - Study title, date, authors, recruitment platform, sample size
2. **Abstract** - Four labeled subsections, each a single paragraph:
   - *Background:* Why the study exists, one sentence on the gap in literature
   - *Methods:* Design type, sample size, recruitment platform, statistical tests used
   - *Results:* Primary outcome with inline statistics (chi-square, p-value, effect size)
   - *Conclusions:* What the findings mean, stated conservatively
3. **Key Findings** - Numbered findings (3-5), each with a headline statistic and one-sentence explanation
4. **Introduction** - Context for the research question. State each hypothesis explicitly, numbered (H1, H2, ...). Reference relevant prior work if available.
5. **Methods**
   - Study design (between-subjects, number of conditions)
   - Participants (N, recruitment platform, compensation, demographics summary)
   - Stimuli (description of each condition with embedded images showing the actual stimulus)
   - Distractor products (what they are, why they were chosen)
   - Measures (list every measured variable with its scale type)
   - Procedure (participant flow from recruitment through completion)
   - Statistical analysis plan (which tests, why, alpha level)
6. **Results**
   - Numbered tables (Table 1, Table 2, ...) with descriptive caption above each
   - Horizontal bar charts (CSS-only, per `templates/report-css.md`) for key comparisons
   - Chi-square test statistics and p-values annotated directly below each table
   - Effect sizes (Cramer's V) reported inline
   - Organize by dependent variable, not by condition
7. **Distractor Validation**
   - Table showing distractor product results across all conditions
   - Chi-square test for each distractor measure
   - One paragraph explaining what the table shows and why it matters (confirms specificity of treatment effect)
8. **Discussion** - Interpret each finding. Connect back to hypotheses. Note unexpected results.
9. **Detailed Methodology**
   - Recruitment waves (dates, n per wave, any blocklist changes)
   - Randomization method (URL-based router, equal probability)
   - Questionnaire development (iterations, LLM pre-test results, changes made)
   - Incentive structure
   - Data quality checks (completion time, attention checks if used)
   - Margin of error table: one row per condition showing n, MOE at 95% CI
10. **Limitations** - Minimum 4 limitations, honestly stated. Common ones to consider:
    - Online sample may not represent general population
    - Single exposure (participants saw stimulus once, not repeatedly)
    - Hypothetical purchase intent vs. actual purchasing behavior
    - Demographic skew relative to census
    - Order effects within survey
    - Self-report bias
11. **Conclusion** - Summary of what was found and practical implications
12. **References** - Any cited work, formatted consistently
13. **Appendix A: Survey Instrument** - Render every question as a visual mockup (styled to look like the survey platform). Show question text, response options, and any embedded images. Mark treatment-only questions.
14. **Appendix B: Distractor Product Data** - Full data tables for distractor products across all conditions and measures.
15. **Appendix C: Open-Ended Responses** - Selected quotes from open-ended questions, styled per `templates/report-css.md` quote format. Group by theme. Include condition label for each quote.
16. **Appendix D: Topline Questionnaire** - Full topline per `templates/topline.md` format.

### Responsive Requirements

- **Desktop (>820px):** Full-width tables, side-by-side charts where appropriate
- **Tablet (480-820px):** Single-column layout, tables scroll horizontally if needed
- **Phone (<480px):** Stacked layout, larger touch targets for interactive elements
- **Print:** `@media print` styles. No shadows, no background colors on text, clean borders. Page breaks before major sections (`break-before: page`).

## 2. Deployment

Deploy the report as a static site.

### Vercel (recommended)

```bash
# From the directory containing the report HTML and images
npx vercel --prod --yes
```

### Image Optimization

If PNG stimulus images exceed 500KB, compress to JPEG before deploying:

```bash
# Convert PNG to JPEG at 85% quality via sips (macOS built-in)
sips -s format jpeg -s formatOptions 85 input.png --out output.jpg
```

Update image references in the HTML after conversion.

### Alternative Hosts

Any static hosting works: Netlify, GitHub Pages, S3 + CloudFront. The report is a single HTML file with relative image paths.

## 3. OSF Data Deposit

Deposit anonymized data, stimuli, and the report on the Open Science Framework. All API calls use `Authorization: Bearer {OSF_TOKEN}` header.

### Step 1: Create Project

```bash
curl -X POST https://api.osf.io/v2/nodes/ \
  -H "Authorization: Bearer ${OSF_TOKEN}" \
  -H "Content-Type: application/vnd.api+json" \
  -d '{
    "data": {
      "type": "nodes",
      "attributes": {
        "title": "Study Title Here",
        "description": "Brief study description",
        "category": "project",
        "tags": ["consumer research", "between-subjects", "survey experiment"]
      }
    }
  }'
```

Save the returned `id` as `PROJECT_ID`.

### Step 2: Upload Files

Upload each file individually. The OSF files API uses PUT, not POST.

```bash
# Upload a file
curl -X PUT "https://files.osf.io/v1/resources/${PROJECT_ID}/providers/osfstorage/?kind=file&name=anonymized_responses.csv" \
  -H "Authorization: Bearer ${OSF_TOKEN}" \
  -H "Content-Type: application/octet-stream" \
  --data-binary @anonymized_responses.csv
```

Files to upload:
- `anonymized_responses.csv` - Flat response data
- `anonymized_responses.json` - Structured response data by arm
- `statistical_tests.json` - All test results
- `report.html` - The full report
- Stimulus images (all conditions)
- `README.md` - File descriptions and data dictionary

### Step 3: Set Metadata

```bash
curl -X PATCH "https://api.osf.io/v2/nodes/${PROJECT_ID}/" \
  -H "Authorization: Bearer ${OSF_TOKEN}" \
  -H "Content-Type: application/vnd.api+json" \
  -d '{
    "data": {
      "type": "nodes",
      "id": "'${PROJECT_ID}'",
      "attributes": {
        "title": "Full Study Title with Descriptive Subtitle",
        "description": "A between-subjects experiment with N participants examining...",
        "tags": ["consumer research", "between-subjects", "survey experiment", "open data"]
      }
    }
  }'
```

### Step 4: Set License (CC-BY 4.0)

```bash
curl -X PATCH "https://api.osf.io/v2/nodes/${PROJECT_ID}/" \
  -H "Authorization: Bearer ${OSF_TOKEN}" \
  -H "Content-Type: application/vnd.api+json" \
  -d '{
    "data": {
      "type": "nodes",
      "id": "'${PROJECT_ID}'",
      "relationships": {
        "license": {
          "data": {
            "type": "licenses",
            "id": "563c1cf88c5e4a3877f9e96a"
          }
        }
      }
    }
  }'
```

### Step 5: Create Wiki

```bash
curl -X POST "https://api.osf.io/v2/nodes/${PROJECT_ID}/wikis/" \
  -H "Authorization: Bearer ${OSF_TOKEN}" \
  -H "Content-Type: application/vnd.api+json" \
  -d '{
    "data": {
      "type": "wiki-pages",
      "attributes": {
        "name": "home",
        "content": "# Study Title\n\n## Overview\nBrief description of the study.\n\n## Files\n- anonymized_responses.csv: Response data\n- statistical_tests.json: Test results\n- report.html: Full report\n\n## Citation\nAuthors (Year). Title. OSF. https://doi.org/..."
      }
    }
  }'
```

### Step 6: Make Public

```bash
curl -X PATCH "https://api.osf.io/v2/nodes/${PROJECT_ID}/" \
  -H "Authorization: Bearer ${OSF_TOKEN}" \
  -H "Content-Type: application/vnd.api+json" \
  -d '{
    "data": {
      "type": "nodes",
      "id": "'${PROJECT_ID}'",
      "attributes": {
        "public": true
      }
    }
  }'
```

### Step 7: Mint DOI

```bash
curl -X POST "https://api.osf.io/v2/nodes/${PROJECT_ID}/identifiers/" \
  -H "Authorization: Bearer ${OSF_TOKEN}" \
  -H "Content-Type: application/vnd.api+json" \
  -d '{
    "data": {
      "type": "identifiers",
      "attributes": {
        "category": "doi"
      }
    }
  }'
```

The response contains the DOI in `data.attributes.value`.

## 4. Final Output

Present three links to the user:

1. **Report URL** - The deployed report (e.g., `https://study-name.vercel.app`)
2. **OSF URL** - The data repository (e.g., `https://osf.io/PROJECT_ID/`)
3. **DOI URL** - The permanent identifier (e.g., `https://doi.org/10.17605/OSF.IO/PROJECT_ID`)

Format as:

```
Study published.

Report:  https://study-name.vercel.app
Data:    https://osf.io/XXXXX/
DOI:     https://doi.org/10.17605/OSF.IO/XXXXX
```
