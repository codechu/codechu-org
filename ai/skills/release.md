---
name: release
description: Cut a release for a Codechu product following its VERSIONING playbook
---

# /release

Cut a release. Works in any Codechu product repo that has
`docs/VERSIONING.md`.

## When to use

User says "let's release vX.Y.Z" or "bump to next version".

## Inputs

- `version` — the target version (SemVer)
- If omitted, suggest the next version based on `CHANGELOG.md`'s
  `[Unreleased]` entries (bug fixes → PATCH, new features → MINOR,
  breaking → MAJOR; 0.x rules: MINOR can break, mark it explicitly).

## Steps

1. Read `docs/VERSIONING.md` — every product owns its own release
   playbook; do not assume.
2. Verify working tree is clean and on `main` (or whichever release
   branch the playbook names).
3. Run the full test + lint suite locally. If anything fails, stop and
   report — do not auto-fix during a release.
4. Apply version-number updates listed in the playbook (typically
   `pyproject.toml` / `Cargo.toml`, `CHANGELOG.md`, `debian/changelog`,
   `*.appdata.xml`).
5. Move `[Unreleased]` entries to `[X.Y.Z] — YYYY-MM-DD` in
   `CHANGELOG.md`. Add a fresh `[Unreleased]` heading.
6. Commit with `release: vX.Y.Z`.
7. Tag with `vX.Y.Z` and an annotated message.
8. **Do not push** unless the user explicitly says so — releases are
   the most visible action.

## Done criteria

- All version-number files agree
- CHANGELOG has the dated entry
- Tag is annotated, not lightweight
- Working tree clean after commit
- User has been shown the diff and given the explicit go-ahead before
  any `git push`
