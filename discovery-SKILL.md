---
name: discovery
description: >
  Activate when the user uses the [DISCOVERY] tag. Generates structured discovery
  documents in HTML with dense, editorial presentation oriented around information
  blocks: cover, metadata, numbered sections, card grids, comparison tables, and
  gap/opportunity blocks. Output always via present_files.
trigger: "[DISCOVERY]"
version: "1.1"
author: joao
depends_on: figma-html
---
# SKILL: [DISCOVERY] — Discovery Document

## When to activate
Triggered by the `[DISCOVERY]` tag in any message. Produces a structured discovery document in HTML with dense, editorial presentation oriented around information blocks.

---

## Required output
Always generate a complete HTML file. Use the `figma-html` skill as the technical base to ensure compatibility with Figma import via html.to.figma. Final output must be presented via `present_files`.

---

## Document structure

Every [DISCOVERY] document follows this section sequence. Not all sections need to appear, but order must be respected when present.

### 1. COVER / HERO
- Large title with typographic weight variation: part in light/regular, part in bold (e.g., "Shum in **Atma**")
- Short subtitle paragraph (2-4 lines) describing the document scope
- Light background (#F5F5F3 or similar), no images

### 2. METADATA BAR (optional)
- Horizontal strip with 3-5 short metrics: number of apps analyzed, gaps, recommendations, coverage percentage
- Layout: large numbers + small label below
- Visually separated by vertical line or spacing

### 3. NUMBERED SECTIONS
Each section follows this pattern:
```
[number] — [UPPERCASE SMALL LABEL]
Section title in larger typography
Optional lead paragraph
```
Thin horizontal separator before each section.

### 4. CARD GRID
2-column layout (or 1 column for featured cards). Card variants:

**Standard card:**
- White background, border `#E0E0DE`, border-radius 12px
- Title in bold (14-16px)
- Body in regular (13-14px), color #444
- Badge(s) in footer

**Pull quote card (conceptual highlight):**
- White or lightly tinted background
- Uppercase small label (10-11px, wide tracking), accent color
- Large text (18-22px), regular or light
- No badges

**Product insight card:**
- Tinted background (#F0F0EE)
- 3px left border, color #555553
- Mixed bold + regular text
- Explicit prefix: "Why this matters for the product:"

### 5. COMPARISON TABLES
For benchmarks and app/product comparisons:
- Header: columns with labels (APP, DIFFERENTIATOR, LEARNING FOR X)
- Lightly zebra-striped rows (#FAFAFA / white)
- App name in bold, rest in regular
- Category badges to the left of the name (e.g., color label for tier/type)
- Maximum 4-5 columns to preserve readability

### 6. GAP / OPPORTUNITY BLOCKS
Numbered cards with prefix G01, G02... or O01, O02...:
- Layout: main block left + side annotation right (2 columns, ~70/30)
- Main block: numbered label, gap title, explanatory body
- Side annotation: soft colored background, implication or direct recommendation text
- Separated by thin horizontal line

### 7. "WHAT X DOES / DOESN'T DO" SECTION
Table or grid with columns:
- Pattern / Feature / Behavior
- Who has it / evidence
- How to adapt for the product

---

## Badge / tag system

Small chips (padding 4px 8px, border-radius 4px, font-size 11px, font-weight 600, uppercase, letter-spacing 0.05em).

Badges are the only exception to the document's monochromatic palette. Saturated colors are intentional: each color carries semantic meaning (green = verified, amber = estimated, purple = inference, red = problem). Do not replace with neutral tones.

| Tag | Background | Text color | Usage |
|---|---|---|---|
| DATA | #D1FAE5 | #065F46 | Verified information, primary source |
| ESTIMATED | #FEF3C7 | #92400E | Calculated or interpolated data |
| INFERENCE | #EDE9FE | #5B21B6 | Analytical conclusion without direct source |
| PRIMARY SOURCE | #D1FAE5 | #065F46 | + source text in uppercase |
| OPPORTUNITY | #DBEAFE | #1E40AF | Product implication |
| GAP | #FEE2E2 | #991B1B | Identified problem |

---

## Typography

```css
--font: 'Inter', system-ui, sans-serif;

--title-hero: 48px / light 300 + bold 700 (mixed in same element via <strong>)
--title-section: 28-32px / regular 400
--label-section: 11px / semibold 600 / uppercase / letter-spacing 0.1em / color #888
--card-title: 14-15px / bold 700
--card-body: 13-14px / regular 400 / color #444 / line-height 1.6
--meta-number: 32px / bold 700
--meta-label: 12px / regular 400 / color #888
--pull-quote: 18-22px / regular 400 / line-height 1.5
```

---

## Base palette

```css
--bg-page: #F5F5F3
--bg-card: #FFFFFF
--bg-tinted-light: #F0F0EE      /* highlight card background */
--bg-tinted-mid: #E8E8E6        /* insight card background */
--border: #E0E0DE
--border-strong: #C8C8C6        /* accent border, e.g. insight left border */
--text-primary: #1A1A1A
--text-secondary: #444444
--text-muted: #888888
--accent-border: #555553        /* dark neutral for accent borders */
```

---

## Writing rules within the document

- No em dashes where comma, period, or colon resolves the sentence
- No artificial line breaks within paragraphs
- Section titles: noun phrases, not questions
- Card labels: nouns or short noun phrases
- Badges always uppercase

---

## Integration with other modes

- If content comes from `[DEEP RESEARCH]`, use DATA/ESTIMATED/INFERENCE notations mapped to corresponding badges.
- For competitive benchmark sections, reuse the `[COMPETITOR]` template table structure.
- Generated HTML must be compatible with the `figma-html` skill (no external dependencies, fonts via Google Fonts CDN, layout in px, not viewport units).
- **This skill's palette fully overrides figma-html color tokens.** Do not use figma-html dark tokens (`--bg`, `--s1`, `--s2`, `--t1`, `--t2`, `--ac` etc.). Valid tokens are those defined in the "Base palette" section above.

---

## Example call

```
[DISCOVERY] Competitive analysis of onboarding flows in Brazilian meditation apps, focus on D1-D7 retention.
```

Claude must:
1. Read this SKILL.md
2. Read `/mnt/skills/user/figma-html/SKILL.md` for HTML technical rules
3. Structure content into the blocks above as relevant
4. Generate complete HTML and present via `present_files`
