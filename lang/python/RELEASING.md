# Releasing a Python package

Extends [`STANDARDS.md` §5.1](../../STANDARDS.md#51-cutting-a-release),
which carries the language-agnostic rules. This is what §5.1 means when
the artefact is a wheel on PyPI.

Read first: [`STANDARDS.md`](../../STANDARDS.md), then your project type.

## 1. Where the version lives

A Python project writes its version in more places than it looks:

| File | Why it matters |
|---|---|
| `pyproject.toml` | what the index sees |
| `<package>/__init__.py` | what the running tool reports, and what any artefact it produces is stamped with |
| `CITATION.cff` | what a citation resolves to |
| `.zenodo.json` | what the DOI archives |

**One command moves all of them**, refuses to run when they disagree
beforehand, and does not tag, commit or push — the tag stays a deliberate
act. A test asserts the records agree, so a mismatch fails CI rather than
shipping. `__init__.py` is the quiet one: it is what a produced artefact
records, and an artefact stamped with a version that did not produce it
cannot be reproduced.

## 2. The long description is generated, never maintained

PyPI renders the description outside the repository, so every relative
link in the README 404s there. If the project ships a rewritten copy,
**generate it** and check it in CI — a copy maintained by remembering to
is a copy that ships one revision behind, and only a reader notices.

## 3. Publish through a trusted publisher

No API token in the repository, and no token in a secret either where the
index supports OIDC. The workflow runs on the tag, builds, runs the suite
and the offline self-check *again* inside the release, then publishes.

Grant the job `contents: write` if any step after publication creates a
release or touches the repository. Publication is irreversible; a job
that can 403 after the upload leaves a state no re-run repairs.

## 4. The order that survives

```
changelog entry            # the release note, written first
bump                       # one command, every record
suite + self-check + any generated-file check
commit, push, let CI pass
tag vX.Y.Z, push the tag   # the only trigger
verify: index, release page, anything downstream
```

## 5. Installation the README recommends

A **library** is installed into a project's dependency tree: `pip install`.
A **tool** goes on the PATH and should not join anyone's dependency tree:
recommend `pipx install` first, and say what the import/command name is
when it differs from the distribution name.

A tool that shells out to another Codechu tool declares it as a
dependency, checks for it before the stage that needs it, and records the
version it found beside the result — see
[`project-type/TOOL.md`](../../project-type/TOOL.md) §5.
