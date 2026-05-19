# AGENTS.md — Codechu global agent conventions

Editor-agnostic rules for AI coding assistants operating in any Codechu
repository. Read alongside [`../STANDARDS.md`](../STANDARDS.md).

## 0. Bootstrap — read this first

If you are an AI agent invoked inside a Codechu repository, do this
**before any other action**:

1. Read this file (`codechu-org/ai/AGENTS.md`) — global discipline.
2. Read the repo-root `CLAUDE.md` (or `AGENTS.md`) of the product
   you're working in — it overrides org defaults on product-local
   concerns (file layout, build commands), but **never on discipline**
   (commits, secrets, push approval).
3. If `codechu-internal/ai/AGENTS.internal.md` exists on this machine,
   read it too — internal overrides extend, never weaken, these public
   rules. If the path doesn't exist, you are operating as an external
   contributor; do **not** invent its contents or speculate about
   private context.
4. Read the product's `docs/ARCHITECTURE.md` and `docs/CONTRIBUTING.md`
   if present.
5. Summarize back to the user in 3 bullets: (a) which repo and product
   you're in, (b) any rule conflicts you spotted between the layers,
   (c) what you understand the task to be. Wait for confirmation
   before editing.

**Never** push, tag, open PRs, comment on remotes, send messages, or
upload to third-party services without an explicit per-action
go-ahead — even if a prior action was approved.

## 1. Commits & PRs

- **Conventional Commits** — `type(scope): subject` in lowercase.
- **No AI signature** — never append `Co-Authored-By: Claude / Copilot /
  …` or "Generated with …" footers. Commit author is the human running
  the tool.
- **No `--no-verify`** — fix the hook, don't bypass it.
- **Never amend a published commit** — create a new commit instead.
- Subject line ≤ 72 chars. Body explains *why*, not *what*.

## 2. Language

- **Code, comments, identifiers, log messages, exception text**:
  English.
- **User-facing UI strings** (in products): gettext (`_()`), source
  in English, translations under `po/`.
- **Libraries do not do i18n.** No `_()`, no `gettext`, no `.po`
  files. Expose semantic data (markers, enums, counts) and let the
  consuming application render and translate. See
  [`STANDARDS.md` §11 "Libraries"](../STANDARDS.md#libraries) for
  the rationale and the rare carve-out.
- **Repo docs**: English canonical (`FOO.md`), Turkish parallel
  (`FOO.tr.md`) — both kept in sync.
- Conversation with the maintainer can be in any language; written
  artifacts follow the rules above.

## 3. Doc style — what to avoid

These come from real review feedback and apply to every public doc:

- **No speculative / forward-looking promises** — no "we will", "by
  v2.0 we plan to", "expected Q3". Document what *is*, not what
  *might be*.
- **No SLAs in public docs** — "best-effort" instead of "within 24h".
- **No version-specific commitments** — "supported Python versions"
  yes; "supports Python 3.12 indefinitely" no.
- **No internal context leaks** — customer names, incident IDs,
  pricing, headcount, contracts.

## 4. Test / lint / type-check before "done"

An agent must run and pass these before declaring a task complete:

- `ruff check .` (or the project's configured linter)
- `pytest -q` (or the project's test runner)
- Any project-specific `pre-commit` hooks

If a check fails: fix the underlying cause. Do not silence, skip, or
mark-as-xfail without writing why in the commit body.

## 5. Scope discipline

- Don't add features or refactors beyond what was asked.
- Don't introduce abstractions for hypothetical future needs.
- Don't add error handling for impossible cases — trust internal
  invariants, validate only at system boundaries.
- Don't write comments that restate the code. Only document *why* when
  it's non-obvious.

## 6. Risky actions — confirm first

These always require explicit user confirmation, even mid-task:

- Destructive git: `reset --hard`, `push --force`, branch deletion,
  amending shared history
- Removing or downgrading dependencies
- Pushing to a remote, opening PRs, posting comments, sending messages
- Touching CI / release pipelines
- Uploading content to third-party services (pastebins, diagram
  renderers, gist) — anything that leaves the machine

Approval for one action is not approval for the next one.

## 7. Vendor namespace

All Codechu products live under `codechu/<product>/` for paths and
package names. When generating new code:

- XDG config: `~/.config/codechu/<product>/`
- Cache: `~/.cache/codechu/<product>/`
- Data: `~/.local/share/codechu/<product>/`
- Runtime: `$XDG_RUNTIME_DIR/codechu/<product>/`
- Python packages: `codechu-<thing>` on PyPI, `codechu_<thing>` import
- Repo names: `codechu/<thing>-py` / `-rs` / `-go` (language suffix)

See [`../STANDARDS.md`](../STANDARDS.md) for the full rule set.
