# Language-specific conventions

Per-language extensions of the org-wide
[`STANDARDS.md`](../STANDARDS.md). Each subdirectory targets one
language and contains conventions for the project types Codechu ships
in that language.

| Language | Status | Path |
|---|---|---|
| Python | Active — 15+ libraries, 1 app | [`python/`](python/) |
| Rust | Planned (`*-rs` repo suffix reserved) | _coming_ |
| Go | Planned (`*-go` repo suffix reserved) | _coming_ |
| TypeScript / JS | Planned (`@codechu/*` npm scope reserved) | _coming_ |

When a second language reaches its first published library, copy
`python/{LIBRARY,APPLICATION}.md` as a starting skeleton and adapt
toolchain-specific bits (build system, lint, package registry,
OIDC publisher equivalent).

## When to add a new language

Add a `lang/<name>/` directory only after the **first** library in
that language is published. Pre-emptive conventions tend to age badly
— write them when at least one concrete project has stress-tested the
assumptions.
