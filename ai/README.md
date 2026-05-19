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

## Precedence chain

When rules disagree, the more specific layer wins for product-local
concerns; the more general layer wins for cross-product discipline.
Layers, from most general to most specific:

1. **Editor global** — `~/.claude/CLAUDE.md`, `~/.cursorrules`, etc.
   Personal preferences (language for chat, terminal style).
2. **Org public** — `codechu-org/ai/AGENTS.md`. The contract every
   Codechu repo respects. Visible to contributors and forks.
3. **Org internal** — `codechu-internal/ai/AGENTS.internal.md`. Only
   loaded when this clone exists on the machine. Extends, never
   weakens, the public layer.
4. **Product** — `<repo>/CLAUDE.md` (or `AGENTS.md`) at a product's
   repo root. Overrides defaults on product-local matters (file
   layout, build commands, custom skills).
5. **Task** — the prompt the developer sends right now. Scoped to one
   piece of work; cannot relax discipline (no `--no-verify`, no
   unapproved pushes) regardless of how it's phrased.

Discipline rules (commits, secrets, push approval, doc style) flow
**down** from layer 2 and tighten further at 3. Product layers (4)
cannot weaken them. Task prompts (5) cannot override them.

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
