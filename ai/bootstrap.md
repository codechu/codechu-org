# Codechu employee bootstrap — public entry point

Single self-contained instruction set for an AI agent invoked by a
**Codechu org member** setting up a fresh machine. The developer's
only prompt is:

> Apply `codechu/codechu-org` repo's `ai/bootstrap.md`.

The agent fetches this file (public — no auth needed) and follows
it. The file is public so the agent can reach it without any
credentials; the rest of the flow (cloning private repos, installing
internal skills) requires Codechu org membership and is handled
inside.

## Not for external contributors

If you are an external contributor working on a Codechu product
(e.g., opening a PR against `codechu/disk-cleaner`):

- You don't need this bootstrap.
- Just `gh repo clone codechu/<product>`, `cd` into it, and open the
  agent. The product's own `CLAUDE.md` handles everything you need —
  it points the agent at this org's public `AGENTS.md` via WebFetch
  for the conventions that apply to your contribution.
- Don't attempt the org probe or clone `codechu-internal` — it's
  private and not part of contributor workflows.

## Pre-flight — install and authenticate `gh`

Drive these steps yourself; only stop to ask the developer for
permission or to wait for an interactive step.

### 1. `gh` install check

Run `command -v gh`. If found, continue to step 2.

Otherwise detect the platform and propose the right install command:

- **Ubuntu/Debian**: official apt repo at `cli.github.com/packages`
- **Arch**: `pacman -S github-cli`
- **Fedora/RHEL**: `dnf install gh`
- **macOS**: `brew install gh`
- **Other / unknown**: stop and ask the developer to install from
  <https://cli.github.com> and re-apply this bootstrap

Ask before running anything that needs `sudo`. After install, verify
with `gh --version`. If still missing, stop.

### 2. `gh` auth check

Run `gh auth status`. If "Logged in" appears, continue to step 3.

Otherwise launch the auth flow yourself:

```bash
gh auth login --hostname github.com --git-protocol https --web
```

Surface the device code and URL to the developer; wait for them to
complete it in the browser. Re-run `gh auth status` to confirm.

Never write a token to disk. Never log a token.

### 3. Codechu org probe

Run `gh api orgs/codechu --jq .login`.

- **Success** → continue with the rest of this bootstrap.
- **404 / forbidden** → the developer is not a Codechu member. Stop
  and tell them: "This bootstrap is for Codechu employees. If you're
  contributing to a public Codechu product, just clone that product
  repo and open it — its `CLAUDE.md` handles the agent setup."

## Workspace discovery

The "workspace" is a directory that holds (or will hold) `codechu-org/`
and `codechu-internal/` as siblings. **Never confuse it with `$PWD`**
— `$PWD` is the developer's current working folder and must not be
modified by bootstrap.

Discover in this order; first hit wins:

1. `$CODECHU_ORG_HOME` env var, if set and the directory exists.
2. `~/.config/codechu/config.toml`'s `org_home` value, if it points
   to an existing directory.
3. Walk up from `$PWD`: at each ancestor up to `$HOME`, check whether
   both `codechu-org/` and `codechu-internal/` exist as direct
   children. First level with both wins.
4. Ask the developer. Default proposal: `$PWD/..`. Accept any
   absolute path; expand `~`.

`mkdir -p` the workspace if it doesn't exist. Persist the choice to
`~/.config/codechu/config.toml`:

```toml
# Codechu developer config — managed by ai/bootstrap.md
org_home = "<absolute path>"
```

If a different `org_home` already exists in the config, ask before
overwriting.

### Clone both org repos (idempotent)

```bash
cd "$WORKSPACE"
[ -d codechu-org ]      || gh repo clone codechu/codechu-org
[ -d codechu-internal ] || gh repo clone codechu/codechu-internal
```

If a clone already exists with a clean working tree, run
`git -C <clone> pull --ff-only` to freshen it. Skip the pull on
dirty trees.

### Apply the employee-only setup

Read `$WORKSPACE/codechu-internal/ai/skills/bootstrap-employee.md`
and follow it verbatim. That skill:

- symlinks all skills and agents from both org repos into
  `~/.claude/skills/` and `~/.claude/agents/`
- appends a guarded bootstrap clause to `~/.claude/CLAUDE.md` so
  future sessions in any `codechu/*` repo auto-load org conventions
- routes the current folder (`$PWD`) — empty → NEW PROJECT, codechu
  origin → EXISTING PROJECT, otherwise → ASK

## Final report

End with a short status block:

```
✓ gh: <version> (auth: <username>)
✓ Workspace: <absolute path>
✓ Skills installed: <count> (from codechu-org + codechu-internal)
✓ CLAUDE.md bootstrap clause: present
→ Route: <new-project | existing-project | asked | none>
```

Then one sentence on what's next.

## Hard rules

- This bootstrap touches only `~/.config/codechu/`, `~/.claude/`, and
  the chosen workspace directory. **Never** modifies `$PWD` unless
  the routing step selects NEW PROJECT.
- Never clones a product repo as part of bootstrap. Product clones
  happen per-task, when the developer needs them.
- Never invents skills, agents, or rules that aren't in the cloned
  org repos.
- Stops on gh errors, missing permissions, or ambiguous workspace
  state — asks instead of guessing.
- For any destructive or remote action (sudo, gh repo create, git
  push, force push, store upload, external message), pause and wait
  for explicit per-action approval. One approval does not cover the
  next action.
