# Python Library Conventions

Conventions for **publishing a Python library** under the `codechu`
namespace to PyPI. Distilled from the May 2026 audit of the first 15
Codechu Python libraries (`codechu-events`, `codechu-treeviz`,
`codechu-xdg`, `codechu-cli`, `codechu-fmt`, `codechu-meter`,
`codechu-spark`, and friends).

This file extends:

- [`STANDARDS.md`](../../STANDARDS.md) — org-wide rules
- [`project-type/LIBRARY.md`](../../project-type/LIBRARY.md) —
  language-agnostic library conventions

If a rule here conflicts with one of those, that's a bug — open a PR.

---

## 1. Repo creation

```bash
gh repo create codechu/<name>-py \
    --public \
    --description="<one-liner — what it gives the caller, ≤80 chars>" \
    --homepage="https://github.com/codechu/<name>-py"
```

**Both `--description` and topics are mandatory before the first push.**
Description shows up in PyPI sidebar, GitHub list view, and search
results; topics drive discovery. Skipping them and "fixing later" is
the most common reason a release goes out looking abandoned.

```bash
gh repo edit codechu/<name>-py --add-topic python,library,codechu,<domain>
```

Topic taxonomy: always include `python`, `library`, `codechu`. Add one
or two domain topics (`treemap`, `events`, `xdg`, `formatter`, ...).

## 2. Naming

| Identifier | Pattern | Example |
|---|---|---|
| GitHub repo | `<name>-py` | `events-py` |
| PyPI distribution | `codechu-<name>` | `codechu-events` |
| Python import module | `codechu_<name>` | `import codechu_events` |
| Source directory | `src/codechu_<name>/` | `src/codechu_events/` |
| Tests directory | `tests/` | `tests/test_bus.py` |

`<name>` is `kebab-case` in distribution names, `snake_case` in import
names. Never `Codechu`/`CodeChu`.

## 3. Branch

Default branch is `main`. Never `master`.

Some Python tools (older `git init` defaults, some IDEs) still create
`master`. Rename **before the first push**:

```bash
git branch -m master main
```

If you already pushed `master`, follow the GitHub rename flow; do not
force-push over `main`.

## 4. `pyproject.toml` skeleton

Setuptools-backed, PEP 621 metadata, no `setup.py`. Copy and adapt:

```toml
[build-system]
requires = ["setuptools>=68", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "codechu-<name>"
version = "0.1.0"
description = "<one-liner from gh repo create>"
readme = "README.md"
requires-python = ">=3.10"
# Legacy file-based license. PyPI Warehouse rejected PEP 639 SPDX
# expressions in early 2026 (HTTP 400 on upload). Stay on the file form
# until Warehouse re-enables PEP 639.
license = { file = "LICENSE" }
authors = [{ name = "Codechu", email = "info@codechu.com" }]
keywords = ["<domain-keyword-1>", "<domain-keyword-2>"]
classifiers = [
    "Development Status :: 4 - Beta",
    "Intended Audience :: Developers",
    "License :: OSI Approved :: MIT License",
    "Operating System :: OS Independent",
    "Programming Language :: Python :: 3",
    "Programming Language :: Python :: 3.10",
    "Programming Language :: Python :: 3.11",
    "Programming Language :: Python :: 3.12",
    "Programming Language :: Python :: 3.13",
    "Topic :: Software Development :: Libraries :: Python Modules",
    "Typing :: Typed",
]
dependencies = []  # zero-dep stdlib-only is the Codechu default

[project.optional-dependencies]
dev = [
    "pytest>=7.0",
    "pytest-cov>=4.1",
    "ruff>=0.4",
]

[project.urls]
Homepage = "https://github.com/codechu/<name>-py"
Source   = "https://github.com/codechu/<name>-py"
Issues   = "https://github.com/codechu/<name>-py/issues"
# Use `Source`, NOT `Repository`. PyPI sidebar renders both, but
# `Source` is the label our existing libraries use — keep it consistent.

[tool.setuptools.packages.find]
where = ["src"]

[tool.ruff]
line-length = 100
target-version = "py310"

[tool.ruff.lint]
select = ["E", "F", "I", "B", "UP", "SIM"]

[tool.pytest.ini_options]
testpaths = ["tests"]
pythonpath = ["src"]
addopts = "-ra --strict-markers"

[tool.coverage.run]
branch = true
source = ["src/codechu_<name>"]

[tool.coverage.report]
fail_under = 85
show_missing = true
```

## 5. Required files

Must exist before the first `git push --tags`:

| File | Purpose |
|---|---|
| `LICENSE` | MIT default, `Copyright (c) 2026 Codechu` |
| `README.md` | ASCII banner + tagline blockquote + sections |
| `CHANGELOG.md` | [Keep a Changelog](https://keepachangelog.com) format |
| `CLAUDE.md` | Per-repo agent rules (links back to org AGENTS.md) |
| `CONTRIBUTING.md` | Canonical from `codechu-fmt` |
| `SECURITY.md` | Canonical from `codechu-fmt` |
| `CODE_OF_CONDUCT.md` | Contributor Covenant v2.1 |
| `.gitignore` | Python stdlib + `.coverage`, `*.egg-info`, `dist/`, `build/` |
| `.gitattributes` | LF normalization (from `codechu-org/.gitattributes`) |
| `pyproject.toml` | Per §4 |
| `docs/API.md` | Full public-symbol reference |
| `docs/RECIPES.md` | ≥5 idiomatic usage patterns |
| `docs/MIGRATION.md` | Only after the first breaking version |

### README structure

Every Codechu Python library README follows the same skeleton — readers
should feel at home moving between them.

```
ASCII banner (figlet-ish, project name)
> Single-sentence tagline.

## What it gives you
- bullets ...

## Install
pip install codechu-<name>

## Quick examples
```python
import codechu_<name>
...
```

## Documentation
- API reference: [docs/API.md](docs/API.md)
- Recipes:        [docs/RECIPES.md](docs/RECIPES.md)

## License
MIT — see [LICENSE](LICENSE).
```

## 6. Source layout

```
src/codechu_<name>/
    __init__.py        # re-exports the public surface
    <module1>.py
    <module2>.py
    _internal.py       # leading underscore = private, no semver promise
tests/
    test_<module1>.py
    test_<module2>.py
```

- Public API is what `__init__.py` re-exports. Anything else is
  considered private regardless of `_`-prefix.
- `__all__` is mandatory in `__init__.py` — both for docs tooling and
  for `from codechu_<name> import *` consumers.
- Type hints on every public function. `py.typed` marker file in the
  package root.

## 7. GitHub workflows

Paste the canonical pair from `codechu-fmt`. They are short enough to
inline here so a new repo can be wired up without cross-repo lookup.

### `.github/workflows/ci.yml`

```yaml
name: CI
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ["3.10", "3.11", "3.12", "3.13"]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}
      - run: pip install -e ".[dev]"
      - run: ruff check src tests
      - run: pytest -q --cov --cov-report=term-missing
```

### `.github/workflows/release.yml`

```yaml
name: Release
on:
  push:
    tags: ["v*"]
  workflow_dispatch:

permissions:
  id-token: write   # OIDC for Trusted Publisher
  contents: read

jobs:
  publish:
    runs-on: ubuntu-latest
    environment: pypi
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - run: pip install build
      - run: python -m build
      - uses: pypa/gh-action-pypi-publish@release/v1
```

## 8. PyPI publishing

- **Use OIDC Trusted Publisher.** Never long-lived API tokens for new
  projects — PyPI applies stricter abuse heuristics on token uploads
  for fresh accounts, and we hit "Too many new projects created"
  rate-limits during the May 2026 batch.
- Pending Publisher must be **registered on PyPI before the first
  push**. The form is at
  <https://pypi.org/manage/account/publishing/>. Up to 3 pending
  publishers per registration session.
- Tag push (`vX.Y.Z`) triggers `release.yml` automatically.
- If a tag predates the release workflow, dispatch manually:
  `gh workflow run release.yml`.

## 9. Coverage

- Floor: **85%** branch coverage.
- Exceptional libraries may drop to 80% only with a justification
  paragraph in `CHANGELOG.md` for the release.
- Coverage below 80% blocks publish.

## 10. Lessons learned (May 2026 batch)

Real failures from extracting 15 libraries — codified so the next
maintainer doesn't relearn them.

### PEP 639 license rejection

PyPI Warehouse returned HTTP 400 on uploads using PEP 639 SPDX
expressions (`license = "MIT"`). Use the legacy file form
(`license = { file = "LICENSE" }`) until Warehouse re-enables it.

### Token-based publish gets throttled

For brand-new PyPI accounts, token-based uploads of multiple new
projects in a short window trigger "Too many new projects created."
OIDC Trusted Publisher upload paths don't hit this heuristic as hard.

### `gh repo create --source` skips metadata

`gh repo create --source=. ...` creates the repo but does **not**
attach description or topics from local state. Pass `--description`
and follow up with `gh repo edit --add-topic` explicitly.

### `gh repo create` remote weirdness

`gh repo create` sometimes adds an `origin` remote that points to the
HTTPS URL even when SSH is the team default, and occasionally fails to
set it at all. After creation:

```bash
git remote -v   # verify
git remote set-url origin git@github.com:codechu/<name>-py.git
```

### `master` vs `main`

`git init` on older systems and some IDE templates still produce
`master`. Always check `git branch --show-current` before the first
push.

## 11. Pre-flight checklist

Paste into the first release PR description:

```markdown
- [ ] PyPI Pending Publisher registered
- [ ] GitHub repo created with description + topics
- [ ] pytest -q && ruff check src tests passing
- [ ] CHANGELOG entry for the version
- [ ] README banner + tagline
- [ ] docs/API.md complete
- [ ] docs/RECIPES.md ≥3 patterns
- [ ] LICENSE year correct
- [ ] All required files present (see §5)
- [ ] git tag vX.Y.Z + git push origin main vX.Y.Z
```
