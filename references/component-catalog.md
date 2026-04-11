# Component Catalog

All reusable UI components for Illumio branded reports.
Copy these HTML snippets into your document sections.

## Table of Contents
1. [Section with Header](#section-with-header)
2. [Callout Boxes](#callout-boxes)
3. [Code Blocks](#code-blocks)
4. [Phase Banners](#phase-banners)
5. [Numbered Steps](#numbered-steps)
6. [Data Tables](#data-tables)
7. [Checklists](#checklists)
8. [Architecture Diagrams (SVG)](#architecture-diagrams)
9. [Cover Page](#cover-page)
10. [Footer](#footer)

---

## Section with Header

Every content section follows this pattern. The `.section-header` feeds the running header in print.

```html
<div class="section">
  <div class="section-header">
    <img src="DATA_URI_DARK_LOGO" alt="Illumio">
    <span class="section-label">Section Name Here</span>
  </div>
  <h2>Section Title</h2>
  <p>Section content goes here.</p>
</div>
```

Rules:
- One `section-header` per section (exactly one — it feeds the running header)
- The `section-label` text appears in the page header on every page of this section
- Keep labels short (1-3 words): "OVERVIEW", "PREREQUISITES", "DEPLOYMENT PROCEDURE"

---

## Callout Boxes

Four semantic variants. Use sparingly — one or two per page maximum.

### Warning (Orange) — Constraints, Timelines, Important Notes
```html
<div class="callout callout-warning">
  <div class="callout-label">Callout Title</div>
  Content text here. Supports <strong>bold</strong> and <code>inline code</code>.
</div>
```

### Info (Blue) — Background Information, FYI
```html
<div class="callout callout-info">
  <div class="callout-label">Information Title</div>
  Content text here.
</div>
```

### Success (Green) — Tips, Best Practices, Confirmations
```html
<div class="callout callout-success">
  <div class="callout-label">Tip or Best Practice</div>
  Content text here.
</div>
```

### Critical (Red) — Blockers, Destructive Actions, Hard Requirements
```html
<div class="callout callout-critical">
  <div class="callout-label">Critical Warning</div>
  Content text here. Use only for genuine blockers or risks.
</div>
```

---

## Code Blocks

Dark-themed code blocks with syntax highlighting via CSS classes.

```html
<pre><span class="label">FILENAME OR CONTEXT</span>
<span class="comment"># Comment text</span>
<span class="cmdlet">Command-Name</span> <span class="param">-Parameter</span> <span class="string">"value"</span>
<span class="variable">$variable</span> = something
</pre>
```

### Syntax Highlight Classes
| Class       | Color     | Usage                          |
|-------------|-----------|--------------------------------|
| `.comment`  | `#6A9955` | Comments (green)               |
| `.string`   | `#CE9178` | String literals (salmon)       |
| `.param`    | `#9CDCFE` | Parameters/flags (light blue)  |
| `.cmdlet`   | `#DCDCAA` | Commands/functions (yellow)    |
| `.variable` | `#C586C0` | Variables (purple)             |
| `.label`    | `#666`    | Top-right label (small, gray)  |

### Inline Code
```html
<code>inline-code-here</code>
```
Renders with cream-dark background, charcoal text, small monospace font.

---

## Phase Banners

Mark major deployment phases. Always followed by steps or content.

```html
<div class="phase-banner">
  <span class="phase-num">PHASE 1</span>
  <span class="phase-title">Phase Title Here</span>
</div>
```

Rules:
- Phase banners belong INSIDE their parent section (never as separate sections)
- Use `break-after: avoid` to keep the banner with its first content element
- Add a `<div class="phase-spacer"></div>` between phases if needed for screen layout
  (hidden in print: `display: none`)

---

## Numbered Steps

Step-by-step instructions with orange number badges and white cards.

```html
<div class="step">
  <div class="step-number">1</div>
  <div class="step-content">
    <h4>Step Title</h4>
    <p>Step description. Use <code>inline code</code> for paths and commands.</p>
  </div>
</div>
```

Rules:
- Steps have `break-inside: avoid` — they won't split across pages
- Keep step content concise (2-3 lines). For longer content, use a callout after the step.
- Number sequentially within each phase (restart numbering per phase)

---

## Data Tables

Tables with dark charcoal headers and alternating row colors.

```html
<table>
  <tr>
    <th>Column 1</th>
    <th>Column 2</th>
    <th>Column 3</th>
  </tr>
  <tr>
    <td>Data</td>
    <td>Data</td>
    <td>Data</td>
  </tr>
  <tr>
    <td>Data</td>
    <td>Data</td>
    <td>Data</td>
  </tr>
</table>
```

For checklist-style tables with checkboxes:
```html
<table>
  <tr><th style="width:25px">#</th><th>Task</th><th>Owner</th><th style="width:45px">Done</th></tr>
  <tr><td>1</td><td>Task description</td><td>Role</td><td style="text-align:center">&#9744;</td></tr>
</table>
```

Use `&#9744;` (☐) for empty checkbox, `&#9745;` (☑) for checked.

---

## Checklists

CSS-only checkboxes (no JavaScript needed) with orange borders.

```html
<ul class="checklist">
  <li><strong>Item title</strong> — Description of the checklist item.</li>
  <li><strong>Another item</strong> — Another description.</li>
</ul>
```

The checkbox square is rendered via CSS `::before` pseudo-element with an orange border.

---

## Architecture Diagrams

Inline SVGs styled with the brand palette. No external images needed.

### Basic Pattern
```html
<div class="diagram-container">
  <div class="diagram-title">Diagram Title Here</div>
  <svg viewBox="0 0 700 300" xmlns="http://www.w3.org/2000/svg" font-family="Inter, sans-serif">
    <!-- Boxes -->
    <rect x="10" y="60" width="180" height="100" rx="8" fill="#F5F0EA" stroke="#2D2D2D" stroke-width="2"/>
    <text x="100" y="90" text-anchor="middle" font-weight="700" font-size="13" fill="#2D2D2D">Box Title</text>
    <text x="100" y="110" text-anchor="middle" font-size="11" fill="#555">Subtitle</text>

    <!-- Arrows (dashed) -->
    <line x1="190" y1="110" x2="350" y2="110" stroke="#2D2D2D" stroke-width="1.5" stroke-dasharray="6,4"/>
    <polygon points="348,106 358,110 348,114" fill="#2D2D2D"/>

    <!-- Accent boxes (orange border) -->
    <rect x="400" y="40" width="200" height="130" rx="8" fill="none" stroke="#E8611A" stroke-width="2" stroke-dasharray="6,3"/>
    <text x="500" y="70" text-anchor="middle" font-weight="700" font-size="13" fill="#2D2D2D">Accent Box</text>

    <!-- Flow labels (orange background) -->
    <rect x="220" y="95" width="120" height="24" rx="4" fill="#E8611A"/>
    <text x="280" y="112" text-anchor="middle" font-size="10" fill="white" font-weight="600">1 Step Label</text>
  </svg>
</div>
```

### SVG Color Palette
| Usage            | Fill/Stroke | Value     |
|------------------|-------------|-----------|
| Box fill         | fill        | `#F5F0EA` |
| Box border       | stroke      | `#2D2D2D` |
| Accent border    | stroke      | `#E8611A` |
| Arrow lines      | stroke      | `#2D2D2D` |
| Flow labels      | fill (bg)   | `#E8611A` |
| Title text       | fill        | `#2D2D2D` |
| Subtitle text    | fill        | `#555`    |
| Accent text      | fill        | `#E8611A` |
| White on orange  | fill        | `white`   |

### Diagram Tips
- Use `rx="8"` on all rectangles for rounded corners
- Use `stroke-dasharray="6,3"` for dashed borders (emphasis/future items)
- Keep SVG `viewBox` proportional to A4 width (~700px wide works well)
- Always wrap in `<div class="diagram-container">` for consistent padding and `break-inside: avoid`

---

## Cover Page

The cover is a full-bleed section with a dark gradient, geometric shapes, and the white logo.

```html
<div class="cover">
  <div class="geo-1"></div><div class="geo-2"></div><div class="geo-3"></div>
  <div class="dots"></div><div class="dots-2"></div>
  <div class="cover-logo">
    <img src="DATA_URI_WHITE_LOGO" alt="Illumio" class="cover-logo-img" style="height:40px;">
  </div>
  <h1>Document Title Here</h1>
  <p class="subtitle">One-sentence description of the document purpose and audience.</p>
  <div class="cover-meta">
    <div class="cover-meta-item">Version<strong>1.0</strong></div>
    <div class="cover-meta-item">Date<strong>Month Year</strong></div>
  </div>
  <div class="cover-footer">
    <span>&copy; 2026 Illumio, Inc. All Rights Reserved.</span>
    <span>CONFIDENTIAL</span>
  </div>
</div>
```

---

## Footer

Closing section with CTA and copyright.

```html
<div style="margin-top:28px;padding:20px;background:var(--ill-charcoal);border-radius:10px;text-align:center;position:relative;overflow:hidden;">
  <div style="position:absolute;top:-10px;right:30px;width:50px;height:100px;background:var(--ill-orange);transform:skewX(-8deg);opacity:0.12;"></div>
  <p style="color:rgba(255,255,255,0.6);font-size:12px;margin-bottom:4px;">Questions? Contact your Illumio pre-sales engineer</p>
  <p style="color:#fff;font-weight:700;font-size:14px;">We're here to support every phase of your deployment.</p>
</div>
<div class="content-footer">
  <span>&copy; 2026 Illumio, Inc. All Rights Reserved.</span>
</div>
```
