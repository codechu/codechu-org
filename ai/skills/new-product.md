---
name: new-product
description: Bootstrap a new Codechu product repo from the TEMPLATE-CHECKLIST
---

# /new-product

Bootstrap a new product repository following Codechu org standards.

## When to use

User wants to start a new product under the Codechu publisher (e.g.,
"let's start a new tool called X").

## Steps

1. Confirm the product slug (kebab-case, used in repo name + XDG paths):
   `codechu/<slug>`.
2. Confirm the primary language (Python, Rust, Go, …) — repo name gets
   the language suffix only for libraries (`-py`, `-rs`); end-user
   products stay unsuffixed.
3. Read [`../../TEMPLATE-CHECKLIST.md`](../../TEMPLATE-CHECKLIST.md)
   and apply every item, in order. Do not skip — the checklist exists
   because earlier products were missing pieces.
4. Set vendor namespace everywhere:
   - XDG: `codechu/<slug>/`
   - Python package: `codechu_<slug>` (or library variant)
   - License: MIT default, ask if user wants otherwise (CC BY for docs,
     proprietary for internal)
5. Wire the i18n skeleton even if Turkish ships later — pot file +
   empty `po/` directory + Makefile target.
6. Generate `README.md` + `README.tr.md` from the template with the
   product's tagline filled in.
7. Set up `.github/workflows/ci.yml` with: ruff/clippy, tests,
   coverage gate (≥35 % to start), bandit (Python) or equivalent.
8. **Do not push** — leave the repo local until the user reviews.

## Done criteria

- All checklist items applied
- Tests pass on a fresh clone
- README renders correctly on both light and dark GitHub themes
- No personal info in any file (run a final grep for the maintainer's
  email, real name, machine hostname before reporting done)
