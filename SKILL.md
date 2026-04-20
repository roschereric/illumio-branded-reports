---
name: illumio-branded-reports
description: >
  Generate professional branded PDF and HTML reports using Illumio's visual identity
  (or adaptable to other brands). Produces print-ready A4 documents with running page
  headers, architecture diagrams, code blocks, callouts, and phase-based deployment guides.
  Uses WeasyPrint for HTML-to-PDF conversion with CSS Paged Media features.
  MANDATORY: Use this skill whenever the user asks to create a report, guide, runbook,
  deployment document, technical brief, or any professional document for Illumio customers.
  Also trigger when the user mentions "branded PDF", "print-ready document",
  "customer-facing guide", "deployment guide", "technical document with diagrams",
  or asks to format content in Illumio style.
---

# Illumio Branded Report Generator

Create professional, print-ready PDF and HTML documents using Illumio's brand identity.
The system produces A4 documents with consistent running headers, architecture diagrams,
syntax-highlighted code blocks, callout boxes, and structured deployment phases.

## Workflow

1. Read `references/branding-tokens.md` for color palette and typography
2. Copy `template.html` **together with the `assets/` and `styles/` directories** from this skill's directory as the starting point. The template is intentionally small (~8 KB of markup + a single `<link>` tag) — the heavy styling lives in `styles/report.css` and the logos in `assets/*.png` so the template stays token-light when Claude reads it.
3. Read `references/css-print-architecture.md` for the print CSS system (or open `styles/report.css` directly if you need to tweak tokens or print rules)
4. Read `references/component-catalog.md` for available UI components
5. Customize the template content for the specific document. Keep the `<link rel="stylesheet" href="styles/report.css">` intact unless you intentionally want to fork the CSS.
6. Generate PDF using WeasyPrint (Python) — ensure both `assets/` AND `styles/` are present next to the HTML (or pass `base_url` pointing at a directory that contains them)
7. **Visual review loop (MANDATORY).** Render every page of the PDF to PNG and inspect each one before delivery. Compare against the Visual Review Checklist below. If any item fails — logo distortion, text overflow, diagram label collision, split element, half-empty page — fix the HTML/CSS and re-render. Loop until all checks pass. Do NOT deliver a PDF that has not been visually inspected page-by-page.
8. **Deliver BOTH the HTML and the PDF.** Always save the final `.html` AND the final `.pdf` side-by-side in the user's output folder and present both. The HTML lets the user hand the file to a web designer, re-edit, or host on an intranet; the PDF is the print-ready deliverable. Never ship only one format.

### Visual Review Checklist

After rendering, inspect every page against these checks. Fix and re-render if any item fails.

| # | Check | Likely fix |
|---|-------|-----------|
| 1 | Cover is full-bleed, no running header, gradient + geometric shapes visible | Confirm `.cover { page: cover-page; }` and `@page cover-page { margin: 0 }` |
| 2 | Running header on every content page: orange top border + logo + SECTION LABEL | Confirm `position: running(running-header)` and `@page { @top-left { content: element(running-header); } }` |
| 3 | Logo orange mark is a **perfect circle**, not a vertical ellipse | Use an embedded PNG from `assets/` or an inline `<svg><circle></svg>`. Do NOT use a CSS `::before` pseudo-element with `border-radius: 50%` inside an `inline-flex` container — WeasyPrint stretches it into an ellipse |
| 4 | Section subtitles (uppercase tracking text) fit on one line | Reduce `letter-spacing` to ~1.2px and `font-size` to ~11.5px; keep subtitles short |
| 5 | Diagram labels and arrows do not overlap the boxes on either side | Widen the SVG `viewBox`, shrink label text, or reposition |
| 6 | Code blocks, tables, callouts, and diagrams stay on a single page | Apply `break-inside: avoid` |
| 7 | No empty or half-empty pages where the next section could have flowed in | Override with `.section:nth-child(N) { break-before: auto; }` for short sections |
| 8 | Headings are not orphaned at the bottom of a page | Apply `break-after: avoid` to `h2, h3, h4` |
| 9 | In non-English versions, no Spanish/Portuguese text overflows boxes in diagrams | Widen SVG boxes or shorten translated labels (Spanish runs ~15% longer than English) |

### Visual review — implementation pattern

```python
# After doc.write_pdf('report.pdf'):
from pdf2image import convert_from_path
imgs = convert_from_path('report.pdf', dpi=110)
for i, img in enumerate(imgs, 1):
    img.save(f'page_{i}.png')
# Then, via the Read tool, read each page_*.png and compare
# against the checklist. Iterate on HTML/CSS until all checks pass.
```

`pdf2image` requires `poppler-utils`. On Debian/Ubuntu: `apt-get install poppler-utils`. On macOS: `brew install poppler`.

### Output delivery — both formats, always

```bash
cp report.html "$OUTPUT_DIR/Document_Name.html"
cp report.pdf  "$OUTPUT_DIR/Document_Name.pdf"
```

Then present BOTH files to the user (via `present_files` in Cowork, or direct links if Claude Code). The HTML enables further editing, intranet hosting, and re-use; the PDF is what goes to the customer.

## Branding Overview

Illumio's visual identity uses three anchor colors with a warm, professional tone:

| Token              | Value     | Usage                                    |
|--------------------|-----------|------------------------------------------|
| `--ill-orange`     | `#E8611A` | Primary accent, buttons, badges, borders |
| `--ill-cream`      | `#F5F0EA` | Page background, section fills           |
| `--ill-charcoal`   | `#2D2D2D` | Body text, table headers, phase banners  |

Typography: **Inter** (Google Fonts) at weights 300-900. Code: JetBrains Mono / Fira Code / Consolas.

Logos are stored as PNG files in `assets/`:
- `assets/logo-white.png` — used on the cover (dark backgrounds)
- `assets/logo-dark.png` — used in section running headers (light backgrounds)

The template references them via relative `src="assets/..."` paths. When generating
a document, ensure the `assets/` directory is copied alongside the HTML file so
WeasyPrint can resolve the image paths.

## Repository Structure

The skill is deliberately split into three kinds of files so that the HTML template stays lightweight and cheap to read into context:

```
illumio-branded-reports/
├── template.html           ← ~8 KB — markup only, links to CSS + images
├── styles/
│   └── report.css          ← all screen + print CSS lives here
├── assets/
│   ├── logo-white.png      ← cover (dark backgrounds)
│   └── logo-dark.png       ← section running headers (light backgrounds)
├── references/             ← deep-dive docs (read only when needed)
├── SKILL.md
└── README.md
```

**Why the split?** When Claude reads `template.html` it sees structure, not 100 KB of base64 logos or 11 KB of inline CSS. That keeps token usage low across every session that touches this skill. Read `styles/report.css` only when you need to tweak tokens (colors, fonts, page margins) or print rules — otherwise leave it alone and it never enters context.

### Rendering pattern

WeasyPrint resolves `styles/report.css` and `assets/logo-*.png` relative to the HTML's base URL. When the working document sits inside the skill folder (or `assets/` and `styles/` are copied alongside it), this just works:

```python
from weasyprint import HTML
HTML(filename='document.html').render().write_pdf('Document.pdf')
```

When rendering from a different location, either (a) copy `assets/` and `styles/` next to `document.html`, or (b) pass `base_url` pointing at the skill root:

```python
HTML(filename='/path/to/document.html',
     base_url='/path/to/illumio-branded-reports/').write_pdf('Document.pdf')
```

## Document Architecture

Every document follows this structure:

```
Cover Page          → Full-bleed dark gradient, geometric shapes, white logo
  ↓
Section 1           → Running header (logo + label), content
  ↓
Section 2           → New page (or flows from previous)
  ↓
...                 → Sections continue, overflow pages keep the running header
  ↓
Final Section       → Closing callout, footer with copyright
```

### Sections and Page Breaks

Each `<div class="section">` represents a logical document section. In print:
- Default: each section forces a new page via `break-before: page`
- Override short sections to flow after the previous one: `.section:nth-child(N) { break-before: auto; }`
- The first section always flows after the cover (`:first-child`)

### Running Headers (Critical — Read This)

The running header system uses WeasyPrint's CSS Paged Media support:

```css
/* Each section-header is removed from flow and placed in the page margin */
.section-header { position: running(running-header); }

/* The @page rule picks it up */
@page {
  margin: 60px 0 30px 0;
  @top-left {
    content: element(running-header);
    width: 100%;
  }
}

/* Cover page overrides: no margin, no header */
@page cover-page {
  margin: 0;
  @top-left { content: none; }
}
.cover { page: cover-page; }
```

This ensures **every page** — including overflow pages — gets the correct header
with the Illumio logo and the current section label. The header auto-updates
each time a new section begins.

### Section Hierarchy Rules

- A "section" is a top-level document division (Overview, Prerequisites, Procedure, etc.)
- Phases (Phase 1, Phase 2...) live **inside** sections, never as separate sections
- If content like "Validate the Deployment" is Phase 4 of Deployment Procedure,
  it stays inside the Deployment Procedure section — do NOT create a separate section for it
- The running header always shows the parent section name, not the phase name

## PDF Generation

### WeasyPrint (Required)

```bash
pip install weasyprint --break-system-packages
```

```python
from weasyprint import HTML

src = 'document.html'
dst = 'Document.pdf'

# Render and inspect page count
doc = HTML(filename=src).render()
print(f"Pages: {len(doc.pages)}")

# Write PDF
doc.write_pdf(dst)
```

### Verification Checklist

After generating a PDF, verify:

1. Cover page has no header, full-bleed gradient, geometric shapes visible
2. Every content page has the running header (orange top border + logo + section label)
3. Overflow pages show the SAME section label as the section they belong to
4. Phase banners never appear at the bottom of a page without content following
5. Code blocks have dark background with syntax highlighting preserved
6. Tables don't split across pages (small tables) or split cleanly (large tables)
7. Callout boxes don't split across pages
8. No orphaned headings (heading at bottom of page with content on next page)

## Adapting for Other Brands

The template is designed around Illumio but is parameterizable:

1. In `styles/report.css`, replace the CSS custom properties in `:root` with the target brand's colors
2. Replace the logo files in `assets/` (`logo-white.png` and `logo-dark.png`) with the target brand's logos (keep the same filenames so `template.html` still resolves)
3. In `styles/report.css`, adjust the cover gradient in `.cover { background: linear-gradient(...) }`
4. Adjust geometric shapes (`.geo-1`, `.geo-2`, `.geo-3`) in `styles/report.css` or remove them
5. Update the footer copyright text in `template.html`

The structural CSS (running headers, page breaks, components) in `styles/report.css` stays unchanged — that's why the template markup and the stylesheet live in separate files.

## Document Structure is Flexible

The template shows one example of each component type. The actual document you
produce should have as many (or few) sections, diagrams, phases, tables, and code
blocks as the content requires. The print CSS is structural — it works regardless
of how many components are used.

- Sections: 2 to 10+ (each gets a running header automatically)
- Diagrams: 0 to many (each wrapped in `.diagram-container`)
- Phases: 0 to many (all inside their parent section)
- Code blocks, callouts, tables, steps: unlimited, mix freely

For page break tuning on short sections, override with `.section:nth-child(N) { break-before: auto; }`
in the `@media print` block so short sections flow after the previous one instead of wasting a page.

## Reference Files

Read these for implementation details:

| File | When to Read | Content |
|------|-------------|---------|
| `references/branding-tokens.md` | Always | Full color palette, typography, spacing |
| `references/css-print-architecture.md` | When modifying print CSS | Page margins, running headers, break rules |
| `references/component-catalog.md` | When adding content | Callouts, code blocks, tables, steps, inline SVGs |
| `references/diagrams-guide.md` | When creating diagrams | Excalidraw workflow, hand-coded SVG, Mermaid fallback |
| `template.html` | Always — this is your starting point | Markup-only HTML (links to `styles/report.css` and `assets/*`) |
| `styles/report.css` | When tweaking colors, fonts, page margins, or print rules | All screen + print CSS, extracted from the template so the HTML stays small |
| `assets/logo-white.png` | Copied with template | White logo for cover page and dark backgrounds |
| `assets/logo-dark.png` | Copied with template | Dark logo for section running headers |

## Common Pitfalls

- **Do NOT use Chrome print for PDF** — it ignores `position: running()`. WeasyPrint is required.
- **Do NOT create separate sections for sub-phases** — phases belong inside their parent section.
- **Do NOT embed images as base64 in the HTML** — use external files in `assets/` and reference via `src="assets/..."`. This keeps the template lightweight and avoids burning AI context tokens. For diagrams, use inline SVG.
- **Do NOT paste the full CSS back into `<style>` inside `template.html`** — the stylesheet lives in `styles/report.css` precisely so the HTML stays under ~8 KB. Leave the `<link rel="stylesheet" href="styles/report.css">` in place.
- **Do NOT forget to copy `assets/` AND `styles/` alongside the HTML** — WeasyPrint needs both at their relative paths. When writing the final HTML to the output directory, copy both folders there too (or pass `base_url` to WeasyPrint pointing at a directory that has them).
- **Do NOT set `@page { margin: 0 }` globally** — the running header needs `margin-top: 60px`.
- **Do NOT use `leverages`, `utilize`, `in order to`** — these are AI speech patterns. Use plain language.
- **Do NOT hardcode API versions or tenant-specific values** — use `<PLACEHOLDER>` format.
