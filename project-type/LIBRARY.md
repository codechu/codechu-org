# Library Conventions (language-agnostic)

Rules that apply to **any** Codechu library, regardless of
implementation language. Language-specific extensions live under
[`lang/<name>/LIBRARY.md`](../lang/).

Read first: [`STANDARDS.md`](../STANDARDS.md).

## 1. Semver discipline

- Strict [SemVer 2.0](https://semver.org/).
- Pre-1.0: a minor bump (`0.1 → 0.2`) may carry breaking changes;
  signal it loudly in `CHANGELOG.md`. Consumers should pin
  `>=0.X,<0.(X+1)`.
- Post-1.0: breaking changes require a major bump and a migration
  document.

## 2. Public API contract

A library has exactly two surfaces:

| Surface | Promise |
|---|---|
| **Public** — what's re-exported from the top-level module / what the language considers exported | SemVer applies; deprecation cycle before removal |
| **Private** — everything else, underscore-prefixed by convention | No promise; may change in a patch release |

- Anything callable from outside without crossing a clearly-private
  marker is public. Don't rely on "but nobody imports that" — they will.
- Library-level **singletons are forbidden.** Libraries instantiate
  state through constructors the caller controls. (Apps may keep
  module-level singletons; libraries may not.)

## 3. Breaking changes

A breaking change requires:

1. A major version bump (or 0.x minor bump pre-1.0).
2. `docs/MIGRATION.md` entry covering: what changed, why, the old →
   new migration snippet, the deprecation window.
3. `CHANGELOG.md` `### Breaking` subsection under the release.

## 4. Deprecation policy

- One full minor version cycle of deprecation warning before removal
  (post-1.0).
- Deprecation message includes the replacement and the version the
  removal is scheduled for.
- Pre-1.0 libraries may skip the cycle but must still document the
  removal in `CHANGELOG.md`.

## 5. Dependencies

The default for a Codechu library is **zero dependencies, stdlib
only.** Add a dependency only when:

- The functionality is non-trivial to reimplement correctly
  (cryptography, parsing, complex protocols), or
- The ecosystem already standardized on a single canonical package
  for this concern.

When you add a dependency:

- Pin a permissive range, not a single version.
- Justify it in the CHANGELOG entry that introduces it.
- Audit the dependency's own dep tree — transitive bloat counts.

### Intra-family dependencies

Codechu libraries are siblings, not a layered stack. Avoid one
Codechu library depending on another unless:

- The relationship is unambiguous (`codechu-treeviz` reasonably uses
  `codechu-events` for layout-progress signals), and
- Both libraries are pinned to compatible ranges, and
- The dependency is **declared**, not inlined.

The opposite anti-pattern — inlining a sibling library's code as a
private module to avoid the declared dep — is forbidden in published
libraries. (Apps may inline during transition; libraries must depend
cleanly or not at all.)

**Framework monorepos are the exception.** "Siblings, not a layered
stack" governs *independent libraries in separate repos*. A cohesive
framework shipped as one monorepo (STANDARDS §2, "Repo granularity")
intentionally **is** a layered stack: its adapter packages depend on the
core and version-lock to it. Inside that monorepo a declared core→adapter
dependency at the shared workspace version is correct, not an anti-pattern
— the "avoid" rule still applies *across* repos, between independently
released libraries.

## 6. Documentation expectations

Every library ships:

| File | Required when |
|---|---|
| `README.md` | Always — **vitrine** (see [STANDARDS §7.1](../STANDARDS.md#71-readme-is-the-vitrine)) |
| `docs/API.md` | Always — full public-symbol reference |
| `docs/RECIPES.md` | Always — ≥5 idiomatic usage patterns |
| `docs/MIGRATION.md` | After the first breaking version |
| `docs/ARCHITECTURE.md` | When internal layering, threading model, or non-obvious data flow benefits from a diagram |
| `CHANGELOG.md` | Always — Keep a Changelog format |
| `assets/preview.svg` (or Mermaid in README) | By v0.1.0 — no text-only READMEs |

`API.md` is a reference (every public symbol, signature, brief
description). `RECIPES.md` is a cookbook (how to do common things end
to end). `ARCHITECTURE.md` is the design rationale and diagrams.

### 6.1 Library README skeleton

The vitrine principle in [STANDARDS §7.1](../STANDARDS.md#71-readme-is-the-vitrine)
applies to libraries with this concrete skeleton:

```markdown
[banner — optional ASCII or SVG image]

[![PyPI](...)] [![Python](...)] [![CI](...)] [![License](...)]

> *one-line tagline in italic quote*

# package-name

[1–3 sentence value proposition — what + who + why]

[ONE hero visual: <img src="assets/preview.svg"> for visual libs,
 or a fenced ```mermaid block for non-visual libs]

```bash
pip install package-name
```

```python
# 5–15 line "wow" example
```

## What you get

- capability bullet 1
- capability bullet 2
- capability bullet 3
   (3–8 bullets total; capabilities, NOT API signatures)

## Read more

- [API reference](docs/API.md) — every public symbol
- [Recipes](docs/RECIPES.md) — idiomatic patterns
- [Architecture](docs/ARCHITECTURE.md) — internal design *(when applicable)*
- [Migration guide](docs/MIGRATION.md) — between major versions *(when applicable)*
- [Changelog](CHANGELOG.md)

## Family

| Package | What it does |
|---|---|
| [codechu-sibling-1](https://pypi.org/project/codechu-sibling-1/) | one-line |
| [codechu-sibling-2](https://pypi.org/project/codechu-sibling-2/) | one-line |
   (3–6 entries — most-relevant siblings)

## Credits

- 1–3 inspirations or acknowledgements

## License

[License name] — see [LICENSE](LICENSE).

Part of [Codechu](https://github.com/codechu).
```

**Library-specific hero requirements:**

| Library category | Hero requirement |
|---|---|
| Visual output (CLI rendering, charts, glyphs) | asciinema cast as SVG, or PNG terminal screenshot |
| Layout / geometry | Mermaid or hand-drawn diagram of input → output |
| Data / structure | Mermaid class or ER-style diagram, or small input → output table |
| System / IO (fs, ipc, xdg) | ASCII tree (filesystem) or Mermaid sequence (protocol) |
| Behavior / orchestration (events, log, config) | Mermaid sequence or state diagram |
| Formatters / measurement (fmt, meter) | Compact input → output table |

The exact tool is the author's choice; the requirement is that **the
hero communicates the value before the user reads any prose**.

## 7. Testing & coverage

- Coverage floor: **85%** branch coverage, recommended across all
  Codechu libraries regardless of language.
- Headless test suite must pass without external services (no
  network, no GUI, no real filesystem outside `tmp_path`-equivalents).
- Public API is exercised by tests. Private helpers may be tested
  through the public surface or directly — either is fine.

## 8. Release artifacts

Libraries publish to the canonical package registry for their language
(PyPI, crates.io, npm, etc.). The GitHub release is a companion, not a
replacement — package registries are the source of truth for
consumers.

Tags follow `vMAJOR.MINOR.PATCH`. Tag push triggers the release
workflow. No manual `twine upload` / `cargo publish` from a laptop.
