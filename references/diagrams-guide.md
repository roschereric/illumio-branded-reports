# Diagrams Guide

This document covers three approaches to creating architecture diagrams and visual
explainers for branded reports. Choose based on complexity and available tools.

## Table of Contents
1. [Approach Selection](#approach-selection)
2. [Excalidraw Diagrams](#excalidraw-diagrams)
3. [Hand-Coded SVG Diagrams](#hand-coded-svg-diagrams)
4. [Mermaid Diagrams (Fallback)](#mermaid-diagrams)
5. [Embedding in the Report](#embedding-in-the-report)
6. [Brand Palette for Diagrams](#brand-palette-for-diagrams)

---

## Approach Selection

| Approach     | Best For                                | Tools Required         | Brand Control |
|--------------|----------------------------------------|------------------------|---------------|
| Excalidraw   | Complex architecture, flow diagrams    | excalidraw.com or CLI  | High (manual) |
| Hand SVG     | Simple 2-5 box flows, inline in HTML   | None (code only)       | Perfect       |
| Mermaid      | Quick sequence/flow diagrams           | Mermaid CLI or online  | Low           |

**Default recommendation:** Use Excalidraw for anything with more than 5 boxes or
crossing arrows. Use hand-coded SVG for simple linear flows (A → B → C).

---

## Excalidraw Diagrams

Excalidraw (https://excalidraw.com) produces hand-drawn style diagrams that can be
exported as SVG. The SVGs embed cleanly in the HTML report.

### Workflow

1. **Create the diagram** in Excalidraw (web app or desktop)
   - Use the brand color palette below for fills and strokes
   - Use Inter font (select "Normal" in Excalidraw — it uses a hand-drawn font by default)
   - For a cleaner look: set stroke style to "Architect" or "Artist" mode

2. **Export as SVG**
   - File → Export image → SVG
   - Check "Embed scene" if you want to re-edit later
   - Check "Background" to include the white/transparent background

3. **Clean up the SVG for embedding**
   ```bash
   # Option A: Use Excalidraw CLI (if available in Claude Code)
   npx @excalidraw/excalidraw-cli export input.excalidraw --type svg --output diagram.svg

   # Option B: Manual — open the .svg file and:
   # 1. Remove the XML declaration (<?xml ...?>)
   # 2. Remove width/height attributes from the root <svg>, keep only viewBox
   # 3. Optionally replace Excalidraw's default colors with brand colors
   ```

4. **Embed in the HTML report**
   ```html
   <div class="diagram-container">
     <div class="diagram-title">Architecture Overview</div>
     <!-- Paste the SVG directly here -->
     <svg viewBox="..." xmlns="http://www.w3.org/2000/svg">
       ...
     </svg>
   </div>
   ```

### Excalidraw Brand Color Mapping

When drawing in Excalidraw, use these hex values:

| Excalidraw Element | Color to Use | Hex       | Illumio Token      |
|--------------------|--------------|-----------|--------------------|
| Box fill           | Light cream  | `#F5F0EA` | `--ill-cream`      |
| Box stroke         | Dark gray    | `#2D2D2D` | `--ill-charcoal`   |
| Accent box stroke  | Orange       | `#E8611A` | `--ill-orange`     |
| Arrow lines        | Dark gray    | `#2D2D2D` | `--ill-charcoal`   |
| Label backgrounds  | Orange       | `#E8611A` | `--ill-orange`     |
| Label text on orange | White      | `#FFFFFF` | —                  |
| Title text         | Dark gray    | `#2D2D2D` | `--ill-charcoal`   |
| Subtitle text      | Medium gray  | `#555555` | `--ill-gray-600`   |
| Status/accent text | Orange       | `#E8611A` | `--ill-orange`     |
| Dashed borders     | Orange       | `#E8611A` | (for TBD/pending)  |

### Excalidraw Tips for Print

- Set canvas background to transparent (not white) — the report has a cream background
- Keep diagrams under 700px logical width for good A4 fit
- Avoid very thin strokes (< 1.5px) — they may not print well
- Text should be at least 11px for readability in print
- Excalidraw's hand-drawn style works well at screen sizes but can look rough when
  printed small — test at A4 scale before finalizing

### Saving Excalidraw Files

Save the `.excalidraw` source file alongside the report for future editing:
```
project/
├── report.html
├── diagrams/
│   ├── architecture.excalidraw    ← source (editable)
│   └── architecture.svg           ← export (embedded in HTML)
└── Report.pdf
```

---

## Hand-Coded SVG Diagrams

For simple flows (2-5 boxes with arrows), coding the SVG directly is faster and
gives perfect brand alignment. See `component-catalog.md` for the full pattern.

### Quick Reference

```svg
<svg viewBox="0 0 700 200" xmlns="http://www.w3.org/2000/svg" font-family="Inter, sans-serif">
  <!-- Solid box -->
  <rect x="10" y="40" width="160" height="80" rx="8"
        fill="#F5F0EA" stroke="#2D2D2D" stroke-width="2"/>
  <text x="90" y="75" text-anchor="middle"
        font-weight="700" font-size="13" fill="#2D2D2D">Title</text>

  <!-- Dashed accent box (for pending/external components) -->
  <rect x="400" y="30" width="200" height="100" rx="8"
        fill="none" stroke="#E8611A" stroke-width="2" stroke-dasharray="6,3"/>

  <!-- Arrow with label -->
  <line x1="170" y1="80" x2="390" y2="80"
        stroke="#2D2D2D" stroke-width="1.5" stroke-dasharray="6,4"/>
  <polygon points="388,76 398,80 388,84" fill="#2D2D2D"/>
  <rect x="230" y="68" width="100" height="24" rx="4" fill="#E8611A"/>
  <text x="280" y="85" text-anchor="middle"
        font-size="10" fill="white" font-weight="600">1 Action</text>
</svg>
```

### When to Use Hand-Coded SVG vs Excalidraw

- **Hand SVG**: Linear flows (A → B → C), simple request/response diagrams, 
  component relationship diagrams with < 6 elements
- **Excalidraw**: Network topologies, complex architectures with crossing connections,
  anything requiring spatial layout decisions, diagrams the customer might want to edit

---

## Mermaid Diagrams

Mermaid is useful for quick sequence diagrams or flowcharts but offers limited
brand control. Use as a fallback when Excalidraw is unavailable.

### Workflow

```bash
# Install Mermaid CLI
npm install -g @mermaid-js/mermaid-cli

# Create diagram definition
cat > diagram.mmd << 'EOF'
graph LR
    A[Domain Controller] -->|GPO delivers script| B[Windows Server]
    B -->|Downloads VEN| C[Illumio PCE]
    C -->|Policy sync| B
EOF

# Render as SVG
mmdc -i diagram.mmd -o diagram.svg -t neutral --backgroundColor transparent

# Then embed the SVG in the report (same as Excalidraw export)
```

### Mermaid Limitations
- Limited color control (theme-level only, not per-element)
- Hand-drawn look not available
- Complex layouts can produce cluttered output
- Always review the output — Mermaid's auto-layout may not match your mental model

---

## Embedding in the Report

All diagram types use the same embedding pattern:

```html
<div class="diagram-container">
  <div class="diagram-title">Diagram Title in Uppercase</div>
  <svg viewBox="..." xmlns="http://www.w3.org/2000/svg">
    <!-- SVG content (from Excalidraw export, hand-coded, or Mermaid output) -->
  </svg>
</div>
```

The `.diagram-container` provides:
- White background card with cream-dark border
- 10px rounded corners
- 22px padding
- `break-inside: avoid` in print (diagram never splits across pages)
- Centered SVG with `max-width: 100%`

### Multiple Diagrams

There's no limit on the number of diagrams per document. Place them where they add
context — after the paragraph that describes the concept. Avoid consecutive diagrams
without explanatory text between them.

---

## Brand Palette for Diagrams

Quick-copy hex values for use in any diagramming tool:

```
Primary:    #E8611A  (orange — accents, labels, active connections)
Background: #F5F0EA  (cream — box fills)
Dark:       #2D2D2D  (charcoal — borders, text, arrows)
Light text: #555555  (gray — subtitles, descriptions)
Accent:     #F28C50  (light orange — secondary elements)
Success:    #2E8B57  (green — completed/active states)
Info:       #3B7DD8  (blue — informational elements)
Danger:     #D63031  (red — errors/blockers)
White:      #FFFFFF  (text on dark/orange backgrounds)
```
