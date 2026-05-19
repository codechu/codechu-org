# Codechu — AI harness conventions (public)

Editor-agnostic conventions for AI coding assistants (Claude Code,
Cursor, Aider, Continue, …) working in any Codechu repository.

This directory is **public** and shared with contributors / forks. For
internal-only conventions (finance, customer comms, private workflows),
see the private `codechu-internal/ai/` tree — never mirror those here.

## Layout

```
ai/
├── AGENTS.md       # global rules every agent should respect
├── skills/         # reusable playbooks (release, new-product, i18n-sync…)
└── agents/         # reusable sub-agent definitions (brand-reviewer, i18n-guard…)
```

## How contributors enable it

Drop a one-line include in your editor's config:

- **Claude Code**: clone this repo and reference `ai/AGENTS.md` from
  `~/.claude/CLAUDE.md`, or copy `ai/skills/*` into
  `~/.claude/skills/`.
- **Cursor / Continue**: point at `ai/AGENTS.md` from the project's
  rule file.
- **Plain reader**: `ai/AGENTS.md` is regular markdown — read it
  before sending an LLM into a Codechu repo.

## Scope

What lives here:

- Commit / PR discipline (Conventional Commits, no AI signature)
- Language / tone for docs (English canonical, Turkish parallel)
- Test / lint / type-check gates that agents must run before declaring
  done
- Doc-style guards ("no speculative language", "no version-specific
  promises") — see [../STANDARDS.md](../STANDARDS.md)
- Reusable skills that any product can adopt

What does **not** live here:

- Product-specific runbooks → that product's repo (`docs/`)
- Internal coordination (releases, customers, strategy) → private repo
- API keys, secrets, vendor URLs → never anywhere in a public repo
