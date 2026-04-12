# Report Design System

Design system for `/frontend-design` when generating the study report. Follow these rules exactly. Do not improvise colors, fonts, or spacing.

## Typography

Load from Google Fonts:

```html
<link href="https://fonts.googleapis.com/css2?family=Cormorant+Garamond:ital,wght@0,400;0,600;0,700;1,400&family=IBM+Plex+Sans:wght@400;500;600&family=IBM+Plex+Mono:wght@400;500&display=swap" rel="stylesheet">
```

| Role | Font | Weight | Size |
|------|------|--------|------|
| H1 (cover title) | Cormorant Garamond | 700 | 48px |
| H2 (section) | Cormorant Garamond | 600 | 32px |
| H3 (subsection) | IBM Plex Sans | 600 | 14px, uppercase, letter-spacing: 2px |
| Body | IBM Plex Sans | 400 | 15px, line-height: 1.75 |
| Table data | IBM Plex Mono | 400 | 13px |
| Table header | IBM Plex Sans | 600 | 12px, uppercase, letter-spacing: 1px |
| Captions | IBM Plex Sans | 400 | 13px, italic |
| Stat annotations | IBM Plex Mono | 400 | 12px |

## Colors

```css
:root {
  --text:        #141414;
  --text-mid:    #555555;
  --text-light:  #888888;
  --gold:        #8B6914;
  --gold-bg:     #F5F0E4;
  --teal:        #1B6B5F;
  --teal-bg:     #E8F0EE;
  --bg:          #FEFDFB;
  --surface:     #FFFFFF;
  --border:      #E0DCD4;
  --border-light:#ECEAE4;
  --bar-track:   #F0EDE6;
}
```

- Primary text: `--text` (#141414)
- Secondary text: `--text-mid` (#555555)
- Tertiary/caption text: `--text-light` (#888888)
- Accent (findings, highlights, borders): `--gold` (#8B6914)
- Secondary accent (positive results, comparison): `--teal` (#1B6B5F)
- Page background: `--bg` (#FEFDFB)
- Card/surface background: `--surface` (#FFFFFF)

## Layout

```css
body {
  background: var(--bg);
  color: var(--text);
  font-family: 'IBM Plex Sans', sans-serif;
  line-height: 1.75;
  -webkit-font-smoothing: antialiased;
}

.container {
  max-width: 860px;
  margin: 0 auto;
  padding: 0 40px;
}
```

## Tables

Minimal borders. No vertical rules. Horizontal rules only.

```css
table {
  width: 100%;
  border-collapse: collapse;
  margin: 24px 0 8px;
  font-family: 'IBM Plex Mono', monospace;
  font-size: 13px;
}

thead th {
  font-family: 'IBM Plex Sans', sans-serif;
  font-size: 12px;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 1px;
  color: var(--text-mid);
  padding: 8px 12px;
  text-align: left;
  border-top: 2px solid var(--text);
  border-bottom: 2px solid var(--text);
}

tbody td {
  padding: 8px 12px;
  border-bottom: 1px solid var(--border-light);
}

tbody tr:last-child td {
  border-bottom: 2px solid var(--text);
}

/* Highlight row for significant results */
tr.significant td {
  background: var(--gold-bg);
}

/* Caption above table */
.table-caption {
  font-family: 'IBM Plex Sans', sans-serif;
  font-size: 13px;
  font-style: italic;
  color: var(--text-mid);
  margin-bottom: 4px;
}

/* Stat annotation below table */
.table-note {
  font-family: 'IBM Plex Mono', monospace;
  font-size: 12px;
  color: var(--text-light);
  margin-top: 4px;
}
```

## Bar Charts (CSS-only)

Horizontal bars. No JavaScript. No SVG. Use CSS grid for alignment.

```css
.bar-chart {
  margin: 24px 0;
}

.bar-row {
  display: grid;
  grid-template-columns: 140px 1fr 60px;
  align-items: center;
  gap: 12px;
  margin-bottom: 8px;
}

.bar-label {
  font-family: 'IBM Plex Sans', sans-serif;
  font-size: 13px;
  color: var(--text-mid);
  text-align: right;
}

.bar-track {
  height: 28px;
  background: var(--bar-track);
  border-radius: 4px;
  overflow: hidden;
}

.bar-fill {
  height: 100%;
  border-radius: 4px;
  transition: width 0.3s ease;
}

/* Condition colors */
.bar-fill.control   { background: var(--text-light); }
.bar-fill.treatment  { background: var(--gold); }
.bar-fill.treatment2 { background: var(--teal); }

.bar-value {
  font-family: 'IBM Plex Mono', monospace;
  font-size: 13px;
  font-weight: 500;
  color: var(--text);
  text-align: right;
}
```

### Bar Chart HTML Pattern

```html
<div class="bar-chart">
  <div class="bar-row">
    <span class="bar-label">Control</span>
    <div class="bar-track">
      <div class="bar-fill control" style="width: 23%"></div>
    </div>
    <span class="bar-value">23%</span>
  </div>
  <div class="bar-row">
    <span class="bar-label">Treatment A</span>
    <div class="bar-track">
      <div class="bar-fill treatment" style="width: 41%"></div>
    </div>
    <span class="bar-value">41%</span>
  </div>
</div>
```

## Quote Blocks

For open-ended response quotes in Appendix C.

```css
.quote {
  border-left: 3px solid var(--gold);
  background: var(--gold-bg);
  padding: 16px 20px;
  margin: 16px 0;
  border-radius: 0 6px 6px 0;
}

.quote p {
  font-family: 'Cormorant Garamond', serif;
  font-style: italic;
  font-size: 16px;
  line-height: 1.6;
  color: var(--text);
  margin: 0;
}

.quote .attribution {
  font-family: 'IBM Plex Sans', sans-serif;
  font-style: normal;
  font-size: 12px;
  color: var(--text-light);
  margin-top: 8px;
}
```

### Quote HTML Pattern

```html
<div class="quote">
  <p>"The bee on the label made me think about where my food comes from."</p>
  <div class="attribution">Treatment A participant</div>
</div>
```

## Key Findings Cards

```css
.finding {
  background: var(--gold-bg);
  border-left: 4px solid var(--gold);
  padding: 20px 24px;
  margin: 24px 0;
  border-radius: 0 8px 8px 0;
}

.finding .stat {
  font-family: 'Cormorant Garamond', serif;
  font-size: 36px;
  font-weight: 700;
  color: var(--gold);
  margin-bottom: 4px;
}

.finding .desc {
  font-family: 'IBM Plex Sans', sans-serif;
  font-size: 14px;
  color: var(--text-mid);
}

/* Teal variant for secondary findings */
.finding.teal {
  background: var(--teal-bg);
  border-left-color: var(--teal);
}
.finding.teal .stat { color: var(--teal); }
```

## Cover Page

```css
.cover {
  padding: 100px 0 60px;
  border-bottom: 2px solid var(--border);
  margin-bottom: 60px;
}

.cover .tag {
  font-family: 'IBM Plex Mono', monospace;
  font-size: 11px;
  letter-spacing: 4px;
  text-transform: uppercase;
  color: var(--gold);
  margin-bottom: 20px;
}

.cover h1 {
  font-family: 'Cormorant Garamond', serif;
  font-size: 48px;
  font-weight: 700;
  line-height: 1.15;
  color: var(--text);
  margin-bottom: 16px;
}

.cover .meta {
  font-size: 14px;
  color: var(--text-light);
  line-height: 1.8;
}
```

## Abstract

```css
.abstract {
  background: var(--surface);
  border: 1px solid var(--border);
  padding: 32px;
  margin: 32px 0;
  border-radius: 4px;
}

.abstract h3 {
  font-family: 'IBM Plex Sans', sans-serif;
  font-size: 12px;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 2px;
  color: var(--gold);
  margin-bottom: 8px;
}

.abstract p {
  font-size: 14px;
  line-height: 1.7;
  margin-bottom: 16px;
}
```

## Section Headers

```css
section {
  margin-bottom: 56px;
}

section h2 {
  font-family: 'Cormorant Garamond', serif;
  font-size: 32px;
  font-weight: 600;
  color: var(--text);
  padding-bottom: 12px;
  border-bottom: 1px solid var(--border);
  margin-bottom: 24px;
}

section h3 {
  font-family: 'IBM Plex Sans', sans-serif;
  font-size: 14px;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 2px;
  color: var(--text-mid);
  margin: 32px 0 12px;
}
```

## Stimulus Images

```css
.stimulus-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: 16px;
  margin: 24px 0;
}

.stimulus-card {
  border: 1px solid var(--border);
  border-radius: 6px;
  overflow: hidden;
  background: var(--surface);
}

.stimulus-card img {
  width: 100%;
  display: block;
}

.stimulus-card .label {
  padding: 10px 14px;
  font-family: 'IBM Plex Sans', sans-serif;
  font-size: 12px;
  font-weight: 600;
  letter-spacing: 1px;
  text-transform: uppercase;
  color: var(--text-mid);
  text-align: center;
  border-top: 1px solid var(--border-light);
}
```

## Responsive Breakpoints

### Tablet (max-width: 820px)

```css
@media (max-width: 820px) {
  .container { padding: 0 24px; }
  .cover h1 { font-size: 36px; }
  section h2 { font-size: 26px; }
  .bar-row { grid-template-columns: 100px 1fr 50px; }
  .stimulus-grid { grid-template-columns: 1fr 1fr; }

  table {
    display: block;
    overflow-x: auto;
    -webkit-overflow-scrolling: touch;
  }
}
```

### Phone (max-width: 480px)

```css
@media (max-width: 480px) {
  .container { padding: 0 16px; }
  .cover { padding: 60px 0 40px; }
  .cover h1 { font-size: 28px; }
  section h2 { font-size: 22px; }
  .finding .stat { font-size: 28px; }
  .bar-row { grid-template-columns: 80px 1fr 44px; gap: 8px; }
  .bar-label { font-size: 11px; }
  .stimulus-grid { grid-template-columns: 1fr; }
  .abstract { padding: 20px; }
}
```

## Print

```css
@media print {
  body { background: white; color: black; font-size: 11pt; }
  .container { max-width: 100%; padding: 0; }
  .cover { padding: 40px 0 20px; }

  section { break-inside: avoid; }
  table { break-inside: avoid; }
  .finding { break-inside: avoid; }

  h2 { break-after: avoid; }

  section h2,
  .cover { break-before: page; }

  .bar-fill { -webkit-print-color-adjust: exact; print-color-adjust: exact; }
  .finding { box-shadow: none; border: 1px solid #ccc; }
  .quote { box-shadow: none; }

  /* Force background colors in print */
  .finding, .quote, tr.significant td {
    -webkit-print-color-adjust: exact;
    print-color-adjust: exact;
  }
}
```

## Rules

- No emdashes. Use commas, periods, or semicolons.
- No rounded corners larger than 8px.
- No drop shadows on any element.
- No gradients.
- No animations except bar width transitions.
- All statistical values in monospace (`IBM Plex Mono`).
- Table numbers right-aligned, text left-aligned.
- Every table has a numbered caption above it (e.g., "Table 1. Association selections by condition").
- Every chart has a label or caption.
- Significant results (p < 0.05) highlighted with `--gold-bg`.
