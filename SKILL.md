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
2. Copy `template.html` and the `assets/` directory from this skill's directory as the starting point
3. Read `references/css-print-architecture.md` for the print CSS system
4. Read `references/component-catalog.md` for available UI components
5. Customize the template content for the specific document
6. Generate PDF using WeasyPrint (Python) — ensure `assets/` is in the same directory as the HTML

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

1. Replace CSS custom properties in `:root` with the target brand's colors
2. Replace the logo files in `assets/` (`logo-white.png` and `logo-dark.png`) with the target brand's logos
3. Adjust the cover gradient in `.cover { background: linear-gradient(...) }`
4. Adjust geometric shapes (`.geo-1`, `.geo-2`, `.geo-3`) or remove them
5. Update the footer copyright text

The structural CSS (running headers, page breaks, components) stays unchanged.

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
| `template.html` | Always — this is your starting point | Complete HTML template with all CSS (logos in `assets/`) |
| `assets/logo-white.png` | Copied with template | White logo for cover page and dark backgrounds |
| `assets/logo-dark.png` | Copied with template | Dark logo for section running headers |

## Common Pitfalls

- **Do NOT use Chrome print for PDF** — it ignores `position: running()`. WeasyPrint is required.
- **Do NOT create separate sections for sub-phases** — phases belong inside their parent section.
- **Do NOT embed images as base64 in the HTML** — use external files in `assets/` and reference via `src="assets/..."`. This keeps the template lightweight and avoids burning AI context tokens. For diagrams, use inline SVG.
- **Do NOT forget to copy `assets/` alongside the HTML** — WeasyPrint needs the logo files at the relative path. When writing the final HTML to the output directory, copy `assets/` there too.
- **Do NOT set `@page { margin: 0 }` globally** — the running header needs `margin-top: 60px`.
- **Do NOT use `leverages`, `utilize`, `in order to`** — these are AI speech patterns. Use plain language.
- **Do NOT hardcode API versions or tenant-specific values** — use `<PLACEHOLDER>` format.
