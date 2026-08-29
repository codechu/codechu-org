# Tool Conventions (language-agnostic)

Rules that apply to any Codechu **tool**: software a developer installs
and runs by hand to produce an artefact — a report, a conversion, a
measurement — rather than software another program imports, or a
product a consumer opens. Language-specific extensions live under
[`lang/<name>/`](../lang/).

Read first: [`STANDARDS.md`](../STANDARDS.md).

## 1. What makes a repository this type

| Question | Tool |
|---|---|
| Does other software import it and call its API? | No — that is a [library](LIBRARY.md) |
| Does a non-technical user open it as a product? | No — that is an [application](APPLICATION.md) |
| Does a developer install it and invoke it to get an artefact? | **Yes** |

The boundary matters because `APPLICATION.md` assumes a consumer
product: distribution channels, an update mechanism, a telemetry
policy, a first-run experience, settings state, localisation. A tool
has none of those and should not be made to pretend it does. What it
has instead is a command-line contract and an obligation to be
provable offline.

## 2. The CLI surface is the public API

Flags, subcommand names, exit codes and the shape of what is printed
are the contract. SemVer applies to all four.

- **Renaming or removing a flag is a breaking change.** So is changing
  what an exit code means, or the schema of a machine-readable output.
- **Exit codes must separate three outcomes**: it ran and the answer is
  yes; it ran and the answer is no; it did not run. A tool that returns
  0 for "could not measure" turns an absence into a result, and the
  caller cannot tell. *Incident: a gate returned `COMPLETE` with exit 0
  on a run that had never been measured.*
- **Machine-readable output carries a schema version**, and a reader of
  an older version is told to upgrade rather than handed a silent
  misparse.

## 3. It must be provable with nothing

Ship one command that exercises the whole pipeline with **no network,
no credentials, no model, no configuration** — and make it the first
command in the README. It is the only claim a reader can check before
deciding to trust the tool with anything real.

A tool that can only be demonstrated against a paid endpoint cannot be
evaluated by the person deciding whether to adopt it.

## 4. It refuses rather than guesses

When an input is missing or malformed, stop and name what is missing.
Do not infer, default, or produce a plausible artefact — a wrong
artefact that looks right costs more than no artefact.

- A stop names *what* is missing and *what would unblock it*.
- A stop is not an exception traceback. *Incident: four module entry
  points answered a wrong path with a raw traceback — the opposite of
  what the tool claimed about itself everywhere else.*

## 5. Every artefact carries its provenance

Anything the tool writes that could later be quoted — a report, a
score, a converted file — carries the tool version, the inputs, and
whatever seeds or settings would be needed to produce it again.

**The version stamp must come from one place.** *Incident: a release
bumped the version in the packaging metadata and left three other
records behind, so reports produced by that release were stamped with a
version that did not produce them.* Where a version is recorded more
than once, a test asserts the records agree.

## 6. Long runs are watchable

A run that takes minutes prints progress as it goes — unbuffered, to a
stream the operator can follow, with a measured estimate rather than a
guessed one. Output that appears only at the end is indistinguishable
from a hang, and the operator's only recourse is to kill it.

Do not swallow output in a buffer for tidiness. If filtering is needed,
filter line-by-line.

## 7. Distribution

- Publish to the language's package index; recommend the isolated
  installer (`pipx` and equivalents) first, because a tool goes on the
  PATH rather than into a project's dependency tree.
- The import/command name and the published package name may differ —
  say so in the README, once, where the install command is.
- No update mechanism, no telemetry, no first-run wizard. If a tool
  wants any of those, ask first whether it is really an application.

## 8. README is the vitrine — tool skeleton

The vitrine principle from
[STANDARDS §7.1](../STANDARDS.md#71-readme-is-the-vitrine) applies with
a tool-flavoured skeleton:

```markdown
[hero: a terminal cast or a screenshot of real output]

# tool-name

[badge row — one source line, every badge linked]

*[tagline — one italic sentence]*

[lede: what goes in, what comes out, what runtime it needs]

## Install & run     the offline proof first, then the real invocation
## What you get      the artefact itself, captured, not described
## Commands          a table; the heading must not carry a count
## How it works      the mechanism, briefly
## Limitations       what it cannot do, and what would change that
## Documentation     table with a "when you need it" column
---
[footer: Contributing · Changelog · License]
```

A tool whose product is a claim about the world — a benchmark, a
grader, a measurement line — additionally adopts
[`overlay/PUBLISHED-VERDICT.md`](../overlay/PUBLISHED-VERDICT.md).

The prohibition list from §7.1 still holds: no full options reference
(that is `docs/CLI.md`), no recipes cookbook, no architecture deep
dive.
