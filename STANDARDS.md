# Codechu Project Standards

Discipline rules applied across **all** Codechu products. Independent of
programming language, target platform, license model, or distribution
channel.

## 0. Philosophy

> "A user familiar with one Codechu product should feel at home in the next."

Identity, naming, repository structure, branding, documentation
conventions — all predictable across products. Technology decisions
(language, runtime, platform) are per-product.

## 1. What this document constrains

| Constrains | Does **not** constrain |
|---|---|
| Vendor identity (name, namespace, app ID) | Product feature set |
| Repository conventions (branch, hooks, .gitattributes) | Programming language |
| Brand assets (palette, typography, logo grammar) | Operating system / platform |
| Documentation structure | Distribution channel |
| Public vs private repo discipline | License model (open / source-available / proprietary) |
| Commit / versioning conventions | Build tooling |

A Codechu product can be a CLI tool, a GUI app, a library, a service, a
mobile app, a web app, or a hardware-adjacent firmware — same identity
rules apply.

## 2. Identity (vendor + product / library)

### Products (end-user applications)

| Field | Pattern | Notes |
|---|---|---|
| Vendor | `codechu` | constant |
| Product slug | `kebab-case`, 1–2 words | e.g. `<product>` |
| Reverse-DNS App ID | `com.codechu.<ProductName>` (CamelCase) | for any platform that uses bundle/app IDs |
| GitHub repo | `codechu/<product>` | one repo per product |
| Package registry name | `codechu-<product>` | language-specific (PyPI, npm, crates.io...) |
| Source module | `<product_module>` (lang convention) | snake_case for Python, camelCase for JS, etc. |
| Binary / executable | `<product>` (short) **or** `codechu-<product>` (vendor-prefixed) | UX trade-off |

### Libraries (reusable components)

Libraries can be language-specific (a Python wrapper around an algorithm)
or polyglot (the same idea implemented in multiple languages). The repo
name carries the **language suffix** so multiple implementations coexist.

| Field | Pattern | Examples |
|---|---|---|
| GitHub repo | `codechu/<purpose>-<lang>` | `codechu/events-py`, `codechu/treeviz-rs`, `codechu/xdg-go` |
| Package registry name | `codechu-<purpose>` | `codechu-events` (PyPI), `@codechu/events` (npm), `codechu_events` (crates.io) — registry is already language-scoped |
| Source module / import | `codechu_<purpose>` (lang convention) | `import codechu_events` (Py), `use codechu_events::*` (Rust) |

**Why language suffix in repo name?** A multi-language publisher will at
some point need the same idea in Rust + Python + Go (event bus, paths
abstraction, telemetry). The suffix lets `codechu/events-py` and
`codechu/events-rs` coexist as sibling repos, each its own SemVer track.

**Rule:** Vendor namespace is **required** in system-level identifiers
(reverse-DNS app ID, repo URL, package registry names). Short form is OK
for end-user identifiers (binary, file name) unless there's namespace
collision.

**Plugin families**: when a library has a clean core but variant data
or adapters that don't all ship together (fonts, locale packs, codecs),
split into a core package + sibling `codechu-<thing>-<variant>` plugin
packages that depend on the core. Each variant carries its own data,
license, and release cadence.

**Repo granularity — one library per repo, or a monorepo?** The
deciding question is **release coupling**, not package count:

| The packages… | Repo shape |
|---|---|
| ship independently, à la carte, each its own SemVer cadence (most Codechu libs — `cli-py`, `log-py`; a core + optional locale packs) | **one repo per library** (default) |
| version-lock and release in lockstep — a core plus adapters that **cannot be used without it** and bump together (e.g. a UI framework: agnostic core + DOM/native renderers) | **one monorepo, many packages** (workspaces) |

The monorepo is the React/Vue/Babel shape: one repo, many published
packages, internal deps resolved by the package-manager's workspace
mechanism (no `file:` links, no publish step for local dev). It is named
for the **framework** with the language suffix — `codechu/ui-ts`, packages
`core` / `web` / `native` publishing `@codechu/ui` / `@codechu/ui-web` /
`@codechu/ui-native`. Do **not** split a version-locked family into
sibling repos: that recreates the publish-bump dance across repos (the
polyrepo-sprawl anti-pattern) for no isolation benefit. External consumers
still depend on a **published** package (the registry is the boundary —
git-dependencies cannot install a sub-directory of a monorepo).

## 3. Filesystem layout (when applicable)

Products that store user data on disk should use the vendor namespace.

### Unix / Linux / macOS (XDG-style)
```
~/.config/codechu/<product>/         # config
~/.cache/codechu/<product>/          # cache (regeneratable)
~/.local/share/codechu/<product>/    # persistent data
$XDG_RUNTIME_DIR/codechu/<product>/  # runtime (sockets, pids, locks)
```

### Windows
```
%APPDATA%\Codechu\<Product>\         # config + persistent
%LOCALAPPDATA%\Codechu\<Product>\    # cache + machine-specific
```

### macOS (native conventions for non-XDG apps)
```
~/Library/Application Support/Codechu/<Product>/   # config + data
~/Library/Caches/Codechu/<Product>/                # cache
~/Library/Logs/Codechu/<Product>/                  # logs
```

### Mobile / web / sandboxed
Each platform's native location, namespaced by `codechu.<Product>`
bundle identifier.

**Rule:** One open directory (e.g. `~/.config/codechu/` on Linux) reveals
all Codechu products. Multi-product visibility is the design goal.

## 4. Branding (shared across products)

| Element | Rule |
|---|---|
| Color palette | Navy `#1d2939` + Gold `#e0a020` + Paper `#fafaf7` (product accent allowed) |
| Display font | Ubuntu Sans (or product-appropriate alternative with similar character) |
| Mono / code font | JetBrains Mono (or platform-native mono fallback) |
| Logo grammar | Single-glyph mark + wordmark + lockup |
| Logo variants | `mark.svg`, `mark-knockout.svg`, `mark-mono.svg`, `wordmark.svg`, `lockup-horizontal.svg`, `lockup-stacked.svg` |

Details: [BRAND-GUIDELINES.md](BRAND-GUIDELINES.md).

## 5. Versioning + commits (language-agnostic)

| Rule | Value |
|---|---|
| Version scheme | [SemVer 2.0](https://semver.org/) |
| Commit format | [Conventional Commits](https://www.conventionalcommits.org/) |
| Release tags | `v<MAJOR>.<MINOR>.<PATCH>` |
| Pre-release | `-alpha.N`, `-beta.N`, `-rc.N` |
| Changelog | [Keep a Changelog](https://keepachangelog.com/) format |

## 6. Repository discipline (every product)

| Item | Rule |
|---|---|
| Default branch | `main` (not `master`) |
| Branch protection (mature products) | required CI green + 1 review |
| Pre-commit hooks | recommended — see [hooks/pre-commit](hooks/pre-commit) |
| .gitattributes | LF normalization for text files |
| Commit signing | optional but recommended for release tags |
| Two-tier model | public product repos + private internal / sec-advisories — see §10 |

## 7. Documentation structure (every product repo)

These docs are expected; **exact contents vary by product**.

```
README.md                  # canonical, English — the **vitrine** (see §7.1)
README.tr.md               # optional, Turkish parallel
LICENSE                    # whichever license the product carries
CHANGELOG.md               # Keep a Changelog format
CONTRIBUTING.md            # contributor onboarding (or "closed contributions" notice)
SECURITY.md                # vuln disclosure (even for proprietary products)
CODE_OF_CONDUCT.md         # Contributor Covenant v2.1 reference (for OSS) or internal version
docs/                      # deeper documentation — manual, reference, recipes
assets/                    # banner, demo cast, diagrams, screenshots
.github/                   # GitHub-specific (templates, workflows)
```

For products with branding, packaging, or i18n, see also:

```
docs/BRAND.md, docs/I18N.md, docs/VERSIONING.md, docs/PUBLISHING.md
packaging/                 # platform-specific install artifacts
po/                        # gettext translations (if using gettext)
```

### 7.1 README is the vitrine

The root `README.md` is a **showcase**, not a manual. It is the first
thing a visitor sees on PyPI, GitHub search, and registry pages —
first impression, not last word. Treat it accordingly.

**Principles:**

- **One striking visual** above the fold — a demo cast (asciinema /
  SVG / GIF), a screenshot, or a clean diagram. If the project has
  any screen output at all, *show it*. If it has none, a Mermaid or
  ASCII diagram of the data flow / architecture is a satisfying
  substitute. A wall of text is not.
- **One quick example**, 5–15 lines, that a reader can run right
  after install. Don't catalogue every variant.
- **Capability bullets**, not API tables. README lists *what you can
  do*; `docs/API.md` lists *every public symbol*.
- **Deflect to `docs/`** for depth — API reference, recipes,
  migration, architecture. Each link gets a one-line description.
- **Family table** (sibling Codechu packages) so visitors find the
  ecosystem.
- **License + credits** at the bottom.

**Explicit prohibitions in README:**

- No full API reference tables (those live in `docs/API.md`).
- No multi-section examples cookbook (that is `docs/RECIPES.md`).
- No design rationale / architecture deep dive (that is
  `docs/ARCHITECTURE.md` or `DESIGN_PRINCIPLES.md`).
- No extension-point / plug-in catalogue.
- No migration guides (that is `docs/MIGRATION.md`).

**Soft length target:** the body after badges and the hero visual
fits in roughly **150 lines**. Anything longer belongs in `docs/`.

**Visual assets** live under `assets/`:

```
assets/
  preview.svg          # static hero preview for README (or banner.svg)
  preview.cast         # asciinema recording (optional)
  preview.gif          # animated preview derived from .cast (optional)
  diagram-<name>.svg   # Mermaid renders or hand-drawn architecture
  screenshots/         # for products with GUI
```

By v0.1.0 the README must not be text-only. Either inline a Mermaid
block or reference an `assets/` file. The per-project-type docs
(`project-type/LIBRARY.md`, `project-type/APPLICATION.md`) restate
this rule with type-specific skeletons.

### 7.2 Research & reference subtree (`docs/research/`)

When a decision rests on external literature or on knowledge that may
post-date an agent's training, keep the research in the repo — current,
durable, offline. Two docs, **research pure, synthesis separate**:

- **`docs/research/INDEX.md`** — the *pure* catalogue of external
  sources. Per entry: an id line (venue / id / date / licence / cache
  file) + a neutral "what it is" + a link. **No Codechu verdict here.**
- **`docs/<TOPIC>_LANDSCAPE.md`** (at `docs/` root) — *our* synthesis:
  the decision (header `Decision date · Decider · Status`), the verdicts
  (in / out under the project's stated constraints), the reasoning, and
  our own measurements. It **references** `INDEX.md`. Keeping the source
  record un-coloured lets the synthesis be revised without rewriting the
  research, and audited against neutral facts. `docs/ARCHITECTURE.md`
  links to the landscape doc for discoverability.
- **`docs/research/fetch.sh`** — rebuilds a **gitignored**
  `docs/research/papers/` cache of the full sources (download once,
  offline thereafter).
- **`docs/research/figures/`** — committed, downsized, attributed figures,
  embedded **only** where the licence permits redistribution.

Copyright rules (see also `ai/AGENTS.md` §8):

- **R1 — no full work committed.** A copyrighted work in full (PDF /
  dataset / full text) is never committed to any tier — only the
  gitignored cache + rebuild script + a link to the original.
- **R2 — figure redistribution gate.** An embedded figure carries its
  licence + source + author beside it. If the licence does not grant
  third-party redistribution (e.g. the arXiv non-exclusive distribution
  licence, "all rights reserved"), the figure is **link-only**; a public
  repo embeds only figures whose licence permits it or that stand on a
  sound quotation basis (TR FSEK m.35 / EU 2001/29 Art.5(3)(d)).
- **Tier matters.** Distribution rights trigger on public release;
  redistribution-unlicensed visuals do not enter a public-tier repo.

This is where §7.1's rationale belongs: the vitrine README keeps design
rationale OUT, the research subtree is where it lands. These files render
only on GitHub (never shipped to PyPI), so the §7.1 cross-surface
constraints do not apply. Reference implementation: `codechu-trace-py`.

## 8. CI/CD baseline

Language- and platform-agnostic minimum:

| Check | Apply when |
|---|---|
| Lint / format | every product, language-specific tool (ruff/eslint/clippy/gofmt/etc.) |
| Unit tests + coverage threshold | every product with tests; threshold ≥60% recommended |
| Static security scan | products handling user data, secrets, or destructive operations |
| Translation sync (gettext / equiv.) | products with i18n |
| Build artifact (binary, AppImage, MSI, IPA, etc.) | products with binary distribution |
| Branch protection | mature products (post-v1.0 or large user base) |

Each product's `.github/workflows/` houses these — exact tooling varies.

## 9. License policy (per product)

License is a **product-level** decision, not an organization rule.
A Codechu product may be:

- **Open-source permissive** (MIT, Apache 2.0, BSD) — community-built
- **Open-source copyleft** (GPL, AGPL) — share-alike
- **Source-available** (e.g. PolyForm, BUSL) — restricted commercial use
- **Proprietary** — closed source, commercial product

Each product's LICENSE declares its terms. There is no "Codechu default
license" — the choice belongs to product strategy.

For closed-source / source-available products see the dedicated
project-type guides:

- [`project-type/PROPRIETARY-LIBRARY.md`](project-type/PROPRIETARY-LIBRARY.md)
  — closed-source libraries, license model selection
  (PolyForm / BUSL / proprietary), customer-facing API contract,
  promotion workflow from `codechu-internal/`.
- [`project-type/PROPRIETARY-APPLICATION.md`](project-type/PROPRIETARY-APPLICATION.md)
  — closed-source applications, EULA, license-key / activation,
  signed auto-update, code obfuscation trade-offs.
- Python-specific extras:
  [`lang/python/PROPRIETARY.md`](lang/python/PROPRIETARY.md).

Legal templates (MIT, PolyForm, BUSL, EULA, source-file headers)
live in `codechu-internal/legal/`. The Codechu-wide telemetry policy
lives in `codechu-internal/telemetry/POLICY.md` and applies to both
open and closed products.

## 10. Public vs private repository discipline

Codechu uses a two-tier repo model. Public repos hold released, curated
content; private repos hold strategy, drafts, and unreleased work.

### Tiers

```
PUBLIC                                  PRIVATE
─────────────────────────────────       ─────────────────────────────────
codechu/<product>          ✓            codechu/internal           🔒
  released code + docs                    roadmap (Q1/Q2/year)
  public CHANGELOG                        strategy, pricing
  public roadmap (coarse)                 internal metrics, finance
  marketing assets                        unreleased designs / experiments
                                          customer interviews (NDA)
codechu/codechu-org        ✓            codechu/sec-advisories     🔒
  this repository (standards)            unpatched vulns + disclosure
```

### Content placement

| Content | Tier | Rationale |
|---|---|---|
| Released code | PUBLIC | Already shipped |
| Published documentation | PUBLIC | User-facing |
| Brand assets, marketing | PUBLIC | Externally visible |
| Public roadmap (coarse) | PUBLIC | Direction signal |
| Unstable design drafts | PRIVATE | Mature before publishing |
| Pricing, financial, business decisions | PRIVATE | Internal-only |
| Customer interviews (NDA) | PRIVATE | Contractual |
| Unpatched security vulnerabilities | PRIVATE | Coordinated disclosure |
| Competitor analysis, internal strategy | PRIVATE | Confidential |
| Marketing A/B tests in progress | PRIVATE | Avoid appearance of pre-launch polish |

### Sync discipline

- **Private → Public**: features mature in private, land in public as one
  clean commit. Iteration noise stays private.
- **Public → Private read**: maintainers may cross-link a public commit
  from a private note (issue link).
- **Risk: leakage.** Pre-push hooks or `git-secrets` for credential and
  sensitive-keyword guards.

### Cost / benefit

GitHub free plan offers unlimited private repos — this discipline is
free, requires only mental discipline.

Industry practice: most independent studios use a similar model.

## 11. Internationalization (optional)

### Products

Products (end-user applications) may or may not internationalize. If
they do:

- Canonical source language is **English**
- Translations live in the product-repo's translation directory
  (e.g. `po/`, `locales/`, `i18n/`)
- A Turkish translation is encouraged but not mandatory (most Codechu
  projects ship one because contributors are Turkish-speaking)
- Other languages are community-driven

Each product's i18n workflow lives in `docs/I18N.md`.

### Libraries

Libraries published under `codechu/` **do not localize their output**.

- Exception messages, log messages, and developer-facing diagnostics
  are written in English, hard-coded. They are diagnostic, not UI;
  they must stay greppable, searchable, and stable across locales.
- Libraries do not produce user-facing strings. They expose semantic
  data (markers, enums, counts, type fields) and let the consuming
  application render and translate.
- Libraries do not ship `gettext`, `.po` files, or any translation
  infrastructure.

**Carve-out:** if a library directly renders a UI surface the
consuming application cannot intercept (CLI help generators, form
widgets, terminal scaffolds), it may either: (a) translate internally
with a documented locale-resolution policy, or (b) accept a translator
callback via dependency injection. **(b) is preferred** — the library
stays loosely coupled and the host application keeps control of its
i18n stack.

This matches the de-facto practice of `requests`, `pydantic`,
`SQLAlchemy`, `pytest`, `structlog`, `tokio`, `serde`, and `lodash`.
The Python stdlib's `argparse`/`optparse` and frameworks (Django,
Flask, Click) fall under the carve-out — they own the final
rendering surface to the user.

Authoritative writeups: Microsoft .NET "Best practices for exceptions",
the "Should exception messages be localized?" discussion in the jbe2277/waf
wiki, and the API-craft mailing list consensus.

## 12. Living document

When a new product surfaces an edge case, update this file via PR. A PR
touching `STANDARDS.md` should be tagged **breaking change** in the
title — it may ripple to all product repos.

## 13. Layered conventions

This file is the language-agnostic, org-wide layer. Two narrower
layers extend it:

| Layer | Path | Scope |
|---|---|---|
| Project type | [`project-type/`](project-type/) | Library vs application, any language |
| Language | [`lang/<name>/`](lang/) | Per-language extensions (Python today) |

Read in order: STANDARDS → project-type → lang.

## 14. Active repositories

### Organization-wide
- [codechu/codechu-org](https://github.com/codechu/codechu-org) — this repo (PUBLIC, standards)
- `codechu/internal` 🔒 — strategy, drafts, finance (PRIVATE)

### Products + libraries
See [README.md](README.md) for the current roster. Per-language
library conventions live under [`lang/python/`](lang/python/) (and
future sibling directories as other languages come online).
