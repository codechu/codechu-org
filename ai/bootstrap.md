# Codechu bootstrap — public entry point

Single self-contained instruction set for an AI agent invoked by a
developer on a fresh machine. **Designed to be applied verbatim**:
a developer's only prompt is

> Apply `codechu/codechu-org` repo's `ai/bootstrap.md`.

The agent fetches this file (public — no auth needed) and follows it.

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

- **Success** → Employee path. Continue to "Employee bootstrap" below.
- **404 / forbidden** → External contributor path. Continue to
  "External contributor bootstrap" below.

## Employee bootstrap

The developer is a Codechu org member. The internal layer applies.

### Workspace discovery

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

## External contributor bootstrap

The developer is **not** a Codechu org member. Only the public layer
applies.

### Workspace discovery (public-only)

Same discovery order as the employee path, but looking only for
`codechu-org/`:

1. `$CODECHU_ORG_HOME` env var
2. `~/.config/codechu/config.toml`'s `org_home`
3. Walk up from `$PWD` looking for sibling `codechu-org/`
4. Ask, defaulting to `$PWD/..`

### Clone `codechu-org` only

```bash
cd "$WORKSPACE"
[ -d codechu-org ] || gh repo clone codechu/codechu-org
```

Do **not** attempt to clone `codechu-internal` — the developer
already failed the org probe and shouldn't see 404 errors.

### Wire public skills + agents

```bash
mkdir -p ~/.claude/skills ~/.claude/agents
for src in "$WORKSPACE"/codechu-org/ai/skills/*.md; do
  [ -e "$src" ] && ln -sf "$src" ~/.claude/skills/$(basename "$src")
done
for src in "$WORKSPACE"/codechu-org/ai/agents/*.md; do
  [ -e "$src" ] && ln -sf "$src" ~/.claude/agents/$(basename "$src")
done
```

### Append the public-only bootstrap clause to `~/.claude/CLAUDE.md`

Guarded by a marker; idempotent on re-runs.

```bash
mkdir -p ~/.claude
MARKER="<!-- codechu-bootstrap -->"
if ! grep -qF "$MARKER" ~/.claude/CLAUDE.md 2>/dev/null; then
  cat >> ~/.claude/CLAUDE.md <<EOF

<!-- codechu-bootstrap -->
## Codechu repos (external contributor)

\`org_home\` for this machine: \`$WORKSPACE\`. When the cwd is inside
any clone of a \`codechu/*\` repo, bootstrap per
\`codechu-org/ai/AGENTS.md\` §0 at session start without being asked:

1. Read \`\$org_home/codechu-org/ai/AGENTS.md\`.
2. Read the current repo's \`CLAUDE.md\` if present.
3. State the loaded layers in one short line at the start of the
   first response.

For local git operations (commit, branch, edits) proceed without
pausing. Before remote push, tag push, opening PRs, or external
messages, always show a summary and wait for explicit approval.
<!-- /codechu-bootstrap -->
EOF
fi
```

### Route the current folder

Same routing rules as the employee path (NEW PROJECT / EXISTING
PROJECT / ASK), but without the internal overrides layer.

## Final report (both paths)

End with a short status block:

```
✓ gh: <version> (auth: <username>)
✓ Workspace: <absolute path>
✓ Mode: <employee | external contributor>
✓ Skills installed: <count>
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
