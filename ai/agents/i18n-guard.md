---
name: i18n-guard
description: Scan a Python codebase for hard-coded user-facing strings missing _() wrapping
---

# i18n-guard

Subagent that audits a Codechu Python product for translation coverage.

## Use when

- After a feature lands that touched the UI
- Before cutting a release
- After a translator reports a missing string

## What it checks

- Every string literal passed to `set_label`, `set_title`,
  `set_tooltip_text`, `set_placeholder_text`, `set_markup`,
  `Gtk.MessageDialog`, or assigned to a `label=` keyword arg must be
  inside a `_()` call.
- AST-based, not regex — skips docstrings, type annotations, dunder
  attributes, log-format strings, and exception messages (those are
  developer-facing).
- Whitelist exemptions: literal language picker entries ("English",
  "Türkçe"), brand names, single-character punctuation.

## What it does NOT check

- Translation completeness (msgmerge + msgfmt is the build's job)
- Quality of translations (human review)
- Plural forms (gettext ngettext) — separate skill if needed

## Output

- List of `file:line — string` violations, one per line
- Exit code 1 if any violation found, 0 if clean
- No auto-fixes — wraps require human judgment for context (some
  strings should not be translated, e.g., env-var names)

## Implementation hint

Use Python's `ast` module. The existing
`codechu-org/hooks/pre-commit` is a working reference — re-use its
visitor pattern.
