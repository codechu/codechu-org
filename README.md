# Codechu

Independent publisher of software products.

## Products

| Product | Description | Repo / Channel | Status |
|---|---|---|:---:|
| **Disk Cleaner** | Process-aware safe disk cleanup utility for Linux | [codechu/disk-cleaner](https://github.com/codechu/disk-cleaner) | 0.1 beta |

License model, target platform, and distribution channel vary per product.

## Reusable libraries

Components extracted from Codechu products as standalone libraries — usable
in any project, including non-Codechu ones.

| Library | Language | PyPI / dist | Repo |
|---|---|---|---|
| **codechu-events** — Multi-channel thread-safe event bus | Python | `codechu-events` | [events-py](https://github.com/codechu/events-py) |
| **codechu-treeviz** — Squarified treemap + sunburst layout | Python | `codechu-treeviz` | [treeviz-py](https://github.com/codechu/treeviz-py) |
| **codechu-xdg** — Vendor-namespaced XDG paths (Linux) | Python | `codechu-xdg` | [xdg-py](https://github.com/codechu/xdg-py) |

Repo names carry a language suffix (`-py`, `-rs`, `-go`, etc.) so polyglot
implementations of the same idea can coexist.

## What this repo is

**Organization-wide content shared across all Codechu products** — independent
of programming language, operating system, license model, or distribution
channel.

This repo holds:

- Vendor identity rules (naming, namespaces, brand)
- Repository discipline (public vs private, git config, hooks)
- New-product bootstrap procedure
- Brand guidelines (palette, typography, logo grammar)

This repo does **not** hold:

- Product-specific code or docs (those live in each product's own repo)
- Technology stack decisions (each product chooses its own)
- Platform decisions (each product targets its own platforms)
- Licensing decisions (each product declares its own LICENSE)

## Contents

| File | Purpose |
|---|---|
| [STANDARDS.md](STANDARDS.md) | Discipline rules applied across Codechu products |
| [STANDARDS.tr.md](STANDARDS.tr.md) | Türkçe paralel |
| [TEMPLATE-CHECKLIST.md](TEMPLATE-CHECKLIST.md) | New-product bootstrap procedure |
| [TEMPLATE-CHECKLIST.tr.md](TEMPLATE-CHECKLIST.tr.md) | Türkçe paralel |
| [BRAND-GUIDELINES.md](BRAND-GUIDELINES.md) | Shared palette / typography / logo grammar |
| [BRAND-GUIDELINES.tr.md](BRAND-GUIDELINES.tr.md) | Türkçe paralel |
| [hooks/](hooks/) | Optional shared git hooks |

## Language policy

Canonical documentation is **English** (international reach, sector norm).
Türkçe versions exist as parallel translations in `*.tr.md` files and stay
synchronized with the English originals.

## Contact

- Web: https://codechu.com _(under construction)_
- Email: info@codechu.com
- GitHub: [@codechu](https://github.com/codechu)

## License

Documentation in this repository: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).

Each product carries its own LICENSE — may be open-source, source-available,
or proprietary depending on product strategy.
