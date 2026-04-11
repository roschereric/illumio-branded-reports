# CSS Print Architecture for WeasyPrint

## Overview

The print system uses two CSS layers:
1. **Screen CSS** — the default styles (visible in browser)
2. **Print CSS** (`@media print`) — overrides for PDF generation via WeasyPrint

WeasyPrint supports CSS Paged Media Level 3 features that browsers don't:
`position: running()`, `content: element()`, named `@page` rules, and margin boxes.

## Page Setup

### Named Pages

```css
@page {
  size: A4 portrait;          /* 210mm x 297mm */
  margin: 60px 0 30px 0;      /* Top margin houses the running header */

  @top-left {
    content: element(running-header);
    width: 100%;
  }
}

/* Cover: full bleed, no header */
@page cover-page {
  margin: 0;
  @top-left { content: none; }
}
```

### Why 60px Top Margin?

The running header contains:
- 4px orange top border
- 12px padding-top
- 22px logo height (flex-aligned with section label text)
- 8px padding-bottom
- 1px bottom border

Total = ~47px. The 60px margin gives 13px breathing room so content doesn't touch the header.

## Running Headers — The Key Mechanism

### How It Works

```css
/* In @media print: */
.section-header {
  position: running(running-header);
  /* ... styling ... */
}
```

1. `position: running(name)` removes the element from the document flow
2. It places the element into a "running element slot" identified by `name`
3. `content: element(name)` in a `@page` margin box renders that element
4. Each time a new `.section-header` is encountered, it **replaces** the previous one in the slot
5. On overflow pages (where no new section starts), the **last assigned** header persists

This means: if "Deployment Procedure" spans 3 pages, all 3 pages show "DEPLOYMENT PROCEDURE" in the header. No manual duplication needed.

### Running Header Styling (in print)

```css
.section-header {
  position: running(running-header);
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 50px 8px 50px;
  border-top: 4px solid var(--ill-orange);
  border-bottom: 1px solid var(--ill-cream-dark);
  background: var(--ill-cream);
  margin: 0;
}
```

The header contains:
- Left: `<img>` with the dark Illumio logo (22px height)
- Right: `<span class="section-label">` with the section name (uppercase, gray-400)

## Page Break Strategy

### Section-Level Breaks

```css
.section {
  break-before: page;           /* Default: each section starts on a new page */
  padding: 20px 50px 30px 50px; /* Top padding reduced since header is in margin */
}
.section:first-child {
  break-before: auto;           /* First section flows after cover */
}
.section:nth-child(3) {
  break-before: auto;           /* Example: short section flows after previous */
}
```

Adjust `:nth-child()` overrides based on content length. Short sections (under half a page)
should flow into the previous section's page to avoid wasted whitespace.

### Element-Level Break Control

```css
/* Never break inside these */
.step, .callout, .diagram-container, table, .checklist li {
  break-inside: avoid;
}

/* Keep headings with their following content */
h2, h3, h4 { break-after: avoid; }

/* Keep phase banners with their first step */
.phase-banner {
  break-after: avoid;
  break-inside: avoid;
}

/* Orphan/widow control for text */
p, li { orphans: 3; widows: 3; }
```

### Code Blocks — Allow Splitting

Large code blocks (`<pre>`) are deliberately NOT given `break-inside: avoid`.
Preventing splits on long code blocks creates massive blank gaps. Better to let
WeasyPrint split them naturally. Reduce font size in print for better fit:

```css
pre { font-size: 10px; padding: 12px 16px; }
```

## Cover Page

```css
.cover {
  page: cover-page;        /* Uses the named page with margin: 0 */
  width: 210mm;
  height: 297mm;           /* Explicit A4 dimensions */
  page-break-after: always;
  padding: 80px 50px 60px 50px;
  position: relative;
  overflow: hidden;
}
```

The cover uses `page: cover-page` which maps to `@page cover-page { margin: 0; }`.
This gives full-bleed rendering without the running header.

## Content Wrapper

```css
.content-wrap {
  max-width: 100%;  /* Full width in print (860px max in screen) */
  padding: 0;
  background: var(--ill-cream);
}
.content-wrap::before { display: none; }  /* Hide the orange top bar (causes blank page) */
```

## Print-Specific Spacing

WeasyPrint gives 0 extra padding at automatic page breaks. Add breathing room
to elements that might land at the top of a page:

```css
.step { padding-top: 14px; margin-top: 0; }
.callout { margin-top: 16px; }
.checklist li { padding-top: 10px; }
h3 { padding-top: 18px; }
.phase-banner { padding-top: 10px; }
table { margin-top: 16px; }
.diagram-container { margin-top: 16px; }
```

## Troubleshooting

### Header missing on some pages
- Verify `position: running(running-header)` is in the `@media print` block
- Verify `@page { @top-left { content: element(running-header); } }` exists
- Check that `margin-top` on `@page` is large enough (60px minimum)

### Cover shows a running header
- Ensure `.cover { page: cover-page; }` is set
- Ensure `@page cover-page { @top-left { content: none; } }` overrides the default

### Blank pages appearing
- Remove `content-wrap::before` in print (the orange bar forces a blank page)
- Check that no element has both `break-before: page` AND `break-after: always`

### Elements splitting awkwardly
- Add `break-inside: avoid` to the element
- For code blocks, let them split — the alternative (huge blank gaps) is worse

### Font not rendering
- WeasyPrint fetches Google Fonts at build time. Ensure network access or
  install fonts locally: `apt-get install fonts-inter`
