# Codechu Brand Guidelines

Shared visual identity rules for **all** Codechu products. Independent
of language, platform, or license. Each product designs its own glyph
but uses the shared palette + typography → product family looks
coherent.

## 1. Color palette (shared)

### Primary — Navy

| Token | Hex | Use |
|---|---|---|
| `--brand-navy-light` | `#34507a` | Gradient highlight (top-left) |
| `--brand-navy`       | `#1d2939` | Primary brand color |
| `--brand-navy-dark`  | `#0f1928` | Gradient shadow (bottom-right) |

### Accent — Gold

| Token | Hex | Use |
|---|---|---|
| `--brand-gold`       | `#e0a020` | Single-element accent, link highlight, brand spark |

### Neutrals

| Token | Hex | Use |
|---|---|---|
| `--brand-paper`      | `#fafaf7` | Off-white background |
| `--brand-ink`        | `#1d2939` | Body text |

### Product-specific accent (optional, one extra color)

Each product may add **one** extra accent (palette stays small). The
addition signals product domain.

### Mixing rule

- Navy family: **80%** of any composition
- Paper (off-white): **15%**
- Gold (+ product accent if any): **5%** — small, deliberate moments

Gold is never a fill for large areas — only an attention spark.

## 2. Typography (shared)

### Primary — Ubuntu Sans (or close alternative)

A humanist sans-serif. If Ubuntu Sans isn't appropriate for a target
platform, pick a typeface with similar character (open, friendly,
neutral). Example fallbacks:

```css
font-family: "Ubuntu Sans", "Inter", system-ui, -apple-system, sans-serif;
```

| Element | Size | Weight | Letter-spacing |
|---|---|---|---|
| Hero / display | 92px | 700 | -2 |
| Section heading | 48px | 600 | -1.5 |
| Body | 16–18px | 400 | 0 |
| Caption | 14px | 400 | 0.5 |

### Secondary — JetBrains Mono (code, terminal, CLI banners)

```css
font-family: "JetBrains Mono", "DejaVu Sans Mono", "Cascadia Code", monospace;
```

## 3. Logo system (per product)

### Glyph philosophy

Each product designs a **single-glyph mark**. The glyph does **not
need to carry semantic meaning** — it is a brand signature, not a
metaphor. The wordmark carries the meaning (e.g. "Disk Cleaner",
"File Explorer"). Notion / Linear / Vercel pattern.

### Required variants (every product)

| File | Use |
|---|---|
| `assets/logo/mark.svg` | Full-color mark |
| `assets/logo/mark-knockout.svg` | Inverted for dark backgrounds |
| `assets/logo/mark-mono.svg` | Single-color, uses `currentColor` (system tray, B&W print) |
| `assets/logo/wordmark.svg` | Typography only (no mark) |
| `assets/logo/lockup-horizontal.svg` | Mark + wordmark side-by-side |
| `assets/logo/lockup-stacked.svg` | Mark above wordmark |

### Platform-specific app icons (when applicable)

| File | Format |
|---|---|
| Master | Vector (SVG) at the product's native viewBox |
| Mono / symbolic | 16px monochrome for system tray contexts |
| Small-size simplified | If gradients/details don't survive at 16–32px, ship a separate simplified file |
| Native sizes | Each target platform's required size set rasterized from master |

## 4. Logo usage rules (all products)

### Safe zone
Around the mark, leave clear space equal to the mark's own radius
(or half-height for non-circular marks).

### Minimum size
- **Mark only:** 16px (favicon threshold)
- **Horizontal lockup:** 120px wide minimum
- **Stacked lockup:** 100px wide minimum

### Do not
- ❌ Rotate, skew, or mirror the mark
- ❌ Change the palette (e.g. green hub instead of gold)
- ❌ Flatten the gradient on master (use the mono variant instead)
- ❌ Re-typeset the wordmark in a different font
- ❌ Add elements inside the mark (tagline, version, badge)
- ❌ Frame the mark with an extra border (the mark stands on its own)

## 5. Tone of voice (shared across products)

| Trait | Description |
|---|---|
| **Calm + precise** | Factual, no marketing fluff. "Revolutionary" / "best-in-class" / "game-changing" forbidden. |
| **Process-aware** | Say what the product does **and** what it doesn't. |
| **Platform-respecting** | Use platform-native conventions and language; don't pretend a tool is universal when it isn't. |
| **Bilingual-ready** | English canonical, Turkish parallel when applicable. Other languages welcome. |

## 6. Asset format rules

- **SVG source**: all logo files; colors literal (hex) inside `<stop>`s — avoid CSS variables (theme override risk on Linux)
- **PNG raster** (release prep): standard set 16 / 24 / 32 / 48 / 64 / 128 / 256 / 512 / 1024 px
- **Open Graph image**: 1280×640 PNG
- **Favicon**: ICO (16+32+48 multi-resolution), optional

## 7. Copyright

Brand assets © Codechu. Community contributions to product code are MIT
(or per-product license — see product LICENSE).

Permitted use:
- ✅ Inside Codechu products
- ✅ Articles / videos / blog posts that **describe** the product
- ❌ Creating a derivative brand (fake "Codechu")
- ❌ Modifying a product fork while retaining Codechu brand assets
  (forks should rebrand)

## 8. Living document

When a new product reveals an edge case, PR against this file. Major
changes (palette shift, typography swap) are breaking — coordinate
across product repos before merging.
