---
name: brand-reviewer
description: Audit a Codechu product for brand-guideline compliance (logos, palette, voice)
---

# brand-reviewer

Subagent that reviews a product against
[`../../BRAND-GUIDELINES.md`](../../BRAND-GUIDELINES.md).

## Use when

- Before a public release
- After a logo / palette / typography change
- When onboarding a new product into the publisher's gallery

## What it checks

**Assets**

- `assets/logo/` has all required variants (icon, lockup-horizontal,
  lockup-horizontal-dark)
- SVG sources present alongside any rasterized exports
- Social preview image exists and matches the spec dimensions

**Palette**

- Code-defined palette matches the brand guideline hex values
- Dark / light theme variants both defined
- No hard-coded "drive-by" colors in UI code that bypass the palette
  module

**Voice**

- README tone: no marketing fluff, no superlatives ("revolutionary",
  "best-in-class"), no speculative promises
- Tagline ≤ 80 chars
- Both English and Turkish READMEs in sync (same sections, same claims)

## What it does NOT check

- Subjective aesthetics — that's a human call
- Typography rendering on specific platforms
- Accessibility contrast ratios (separate skill)

## Output

- Markdown report grouped by category (Assets / Palette / Voice)
- Each finding: severity (blocker / warning / note) + file:line + the
  guideline clause it references
- No auto-fixes — brand decisions are deliberate
