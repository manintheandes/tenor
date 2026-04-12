# Topline Questionnaire Format

Pew Research Center-style topline. Every survey question appears with verbatim wording and per-condition percentage breakdowns.

## Structure

Each question block contains:

1. **Question header row** - Verbatim question text spanning all columns, bold
2. **Column headers** - One column per condition (e.g., Control, Treatment A, Treatment B)
3. **Response rows** - One row per response option with percentage in each condition column
4. **n row** - Sample size per condition at the bottom in lighter text

## Formatting Rules

- Percentages are whole numbers with `%` sign (e.g., `23%`, not `23.4%` or `23`)
- Round to the nearest whole number using standard rounding (0.5 rounds up)
- Percentages within a single-select question should sum to 100% per column (rounding may cause +/- 1% variance, this is acceptable)
- Multi-select questions: add a note below the table reading "Respondents could select multiple options. Percentages may sum to more than 100%."
- The `n` row uses lighter text color (`--text-light`) to visually separate it from data rows
- Question numbers match the survey instrument order (Q1, Q2, Q3, ...)

## Treatment-Only Questions

Questions asked only of treatment groups omit the control column entirely. Show only the columns for conditions that received the question. Add a note above the table: "Asked of [Treatment A / Treatment B] respondents only."

## HTML Pattern

```html
<!-- Standard question with all conditions -->
<div class="topline-block">
  <table class="topline">
    <thead>
      <tr>
        <th colspan="4" class="topline-question">
          Q3. Which of the following do you associate with this product?
          Select all that apply.
        </th>
      </tr>
      <tr>
        <th class="topline-option">Response</th>
        <th class="topline-data">Control</th>
        <th class="topline-data">Treatment A</th>
        <th class="topline-data">Treatment B</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>Premium quality</td>
        <td class="topline-data">45%</td>
        <td class="topline-data">42%</td>
        <td class="topline-data">48%</td>
      </tr>
      <tr>
        <td>Good value</td>
        <td class="topline-data">31%</td>
        <td class="topline-data">29%</td>
        <td class="topline-data">33%</td>
      </tr>
      <tr>
        <td>Impact on animals</td>
        <td class="topline-data">12%</td>
        <td class="topline-data">34%</td>
        <td class="topline-data">41%</td>
      </tr>
      <tr class="topline-n">
        <td>n</td>
        <td class="topline-data">340</td>
        <td class="topline-data">342</td>
        <td class="topline-data">345</td>
      </tr>
    </tbody>
  </table>
  <p class="topline-note">Respondents could select multiple options. Percentages may sum to more than 100%.</p>
</div>

<!-- Treatment-only question -->
<div class="topline-block">
  <p class="topline-scope">Asked of Treatment A and Treatment B respondents only.</p>
  <table class="topline">
    <thead>
      <tr>
        <th colspan="3" class="topline-question">
          Q8. What does the seal on the package mean to you?
          Select all that apply.
        </th>
      </tr>
      <tr>
        <th class="topline-option">Response</th>
        <th class="topline-data">Treatment A</th>
        <th class="topline-data">Treatment B</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>Product is organic</td>
        <td class="topline-data">18%</td>
        <td class="topline-data">22%</td>
      </tr>
      <tr>
        <td>No pesticides used</td>
        <td class="topline-data">52%</td>
        <td class="topline-data">47%</td>
      </tr>
      <tr>
        <td>Nothing / I did not notice a seal</td>
        <td class="topline-data">14%</td>
        <td class="topline-data">11%</td>
      </tr>
      <tr class="topline-n">
        <td>n</td>
        <td class="topline-data">342</td>
        <td class="topline-data">345</td>
      </tr>
    </tbody>
  </table>
  <p class="topline-note">Respondents could select multiple options. Percentages may sum to more than 100%.</p>
</div>
```

## CSS

```css
.topline-block {
  margin: 32px 0;
}

.topline {
  width: 100%;
  border-collapse: collapse;
  font-family: 'IBM Plex Sans', sans-serif;
  font-size: 14px;
}

.topline-question {
  text-align: left;
  font-weight: 600;
  font-size: 14px;
  padding: 12px;
  background: var(--surface);
  border-top: 2px solid var(--text);
  border-bottom: 1px solid var(--border);
  line-height: 1.5;
}

.topline thead tr:last-child th {
  font-size: 12px;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 1px;
  color: var(--text-mid);
  padding: 8px 12px;
  border-bottom: 2px solid var(--text);
}

.topline-option {
  text-align: left;
}

.topline-data {
  text-align: center;
  font-family: 'IBM Plex Mono', monospace;
  font-size: 13px;
}

.topline tbody td {
  padding: 6px 12px;
  border-bottom: 1px solid var(--border-light);
}

.topline tbody tr:last-child td {
  border-bottom: 2px solid var(--text);
}

.topline-n td {
  color: var(--text-light);
  font-style: italic;
  font-size: 12px;
}

.topline-note {
  font-size: 12px;
  color: var(--text-light);
  font-style: italic;
  margin-top: 4px;
}

.topline-scope {
  font-size: 12px;
  color: var(--gold);
  font-weight: 500;
  margin-bottom: 4px;
}
```

## Ordering

Present questions in the same order they appear in the survey instrument:
1. Screener questions
2. Product impression and purchase intent (target product)
3. Association questions (target product)
4. Treatment-only questions (seal meaning, open-ended)
5. Distractor product questions (in order presented)
6. Attitude and food choice scales
7. Treatment-only impact and gateway questions
8. Demographics
