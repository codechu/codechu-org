# Research-Line Conventions (language-agnostic)

Rules that apply to any Codechu **research line**: a method, the
pipeline that runs it, and an honest verdict about whether it works.
Language-specific extensions live under
[`lang/<name>/`](../lang/).

Read first: [`STANDARDS.md`](../STANDARDS.md).

## 1. What makes a repository this type

A research line is a repository whose product is a **finding**. The
code exists to make the finding reproducible, and the honest status of
the finding is a first-class deliverable rather than a caveat at the
bottom.

Three questions place a repo here — all three must be yes:

| Question | Research line |
|---|---|
| Is there something to install and call from other software? | No |
| Does a human open it as a product? | No |
| Does it publish a verdict that could come out negative? | **Yes** |

Anything that answers yes to the first is a [library](LIBRARY.md);
yes to the second, an [application](APPLICATION.md).

## 2. Two §7.1 rules do not apply here

[STANDARDS §7.1](../STANDARDS.md#71-readme-is-the-vitrine) is written
for things that ship as packages. Two of its bullets have no referent
in a research line, and forcing them produces a dishonest README:

- **"One quick example, 5–15 lines, that a reader can run right after
  install."** There is nothing to install, and a line whose product is
  a verdict cannot honestly promise that five lines produce a result.
  Replace it with **§4's `## Run it`**: one command that runs today,
  offline, followed by a real captured trace.
- **"Family table (sibling Codechu packages)."** What ties siblings
  here is a shared method and identity, not a registry namespace. A
  package table would list nothing. Say the relationship in a sentence
  and link the sibling repository.

Every other §7.1 rule and every §7.1 prohibition still holds.

## 3. The three obligations, in order

1. **Classify** — what kind of artefact, in what runtime, for whom.
   The reader gets five seconds.
2. **Enable** — one command that runs today, with nothing bought and
   nothing installed beyond the repository.
3. **Let them judge** — the real output and the honest status, without
   digging.

Doctrine is the fourth thing. In this repository type doctrine is the
substance and is never cut, but it is *justification*, and
justification is only legible once the reader knows what is being
justified. **Every doctrinal passage follows the concrete thing it
defends.**

## 4. README is the vitrine — research-line skeleton

```markdown
[hero image — ONE, <p align="center">]

# Repo-Name

[badge row — ALL badges on ONE source line, EVERY badge linked]

*[tagline — one italic sentence]*

[lede — 3–5 sentences: what it is, what goes in, what comes out, who
 scores it. MUST contain the noun that classifies it: "Python",
 "CLI", "stdlib-only"]

[status stamp — ONE line carrying the strongest single number,
 linking down to ## Status]

## Run it              fenced + language-tagged, <= 6 lines, offline,
                       followed immediately by a REAL captured trace
## What it refuses     the verdict / refusal table — strongest asset, early
## <the mechanism>     how a run proceeds; doctrine attached here
## <the making>        the part a reader has to do themselves
## Failure modes       how to misuse it, including what no tool can catch
## Status              state + numbers + WHAT WOULD CHANGE IT
## Documentation       TABLE with a "when you need it" column
<details>Layout</details>
---
[footer: seal image, Contributing · Changelog · License]
```

**Maximum nine `##` sections.** Above nine, GitHub's outline menu
becomes a wall. *Incident: a research-line README carried thirteen,
six of them beginning with the word "What" — unnavigable in a sidebar
that truncates.*

**No two headings begin with the same word.** Headings are noun
phrases, six words maximum. The test: read only the outline menu — can
you navigate?

## 5. The fold is not where you think

**On a GitHub repository page the README is not the fold.**
*Measured on a 1090 px-wide capture: the README began ~1080 px down.
The first screen is the file listing and the About box.*

Three consequences, all cheap and all easy to miss:

- **The About box is the most-read text on the page.** A repo shipped
  with *"No description, website, or topics provided"* while its README
  was being polished. `description` and topics are **repo settings, not
  files** — no amount of README work reaches them.
- **Topics must not contradict the description.** *Incident: a repo
  carried the topic `evaluation` under a description saying it does not
  score anything — the one thing it is built not to do, offered as a
  keyword.*
- **Directory names and commit subjects are landing copy.** They are on
  the first screen; the README is not.

## 6. Rules earned by measurement

Each rule below names the incident that produced it. A rule with no
incident behind it is a guess.

**Every typable thing is fenced with a language tag.** *Verified
through `gh api /markdown`: an indented four-space block renders as a
bare `<pre>`; a fenced block renders as
`<div class="highlight highlight-source-shell">`.* Indented blocks get
no syntax highlighting — reserve them for non-typable ASCII.

**Commands must fit the column.** *Incident: `# 80 tests, no network`
rendered as `# 80 tests, no networ` — the first command a reader sees,
cut off.* Keep command lines under ~60 characters.

**Two blockquote lines in one paragraph collapse into one.**
*Incident: the best sentence in a README had been rendering as a single
running line since the day it was written.* End the first line with a
trailing `<br>`.

**Every badge is a link.** *Incident: three of four badges were dead
images.* A badge that points nowhere is decoration wearing the costume
of a signal.

**A captured trace beats prose about the tool.** For a line whose
product is a verdict, the verdict printing is the most persuasive thing
the page can carry. Show a run that **held**, not a clean one —
stopping is the feature.

**The trace must be literal, and bound by a test.** *Incident: a trace
was trimmed by one line to fit the column, inside a block claiming to
be real output. No reader would have caught it; the test asserting that
every printed line appears verbatim in the README did.* Prose drifts
silently — bind it.

**Images: at most three, each with a job.** Hero (position 1, alone);
evidence (after the first command block); seal (footer, ≤96 px). An
image that illustrates a *concept* rather than a *result* belongs in
`docs/`. *Incident: a plate at `width="380"` took 475 vertical pixels —
more than its section's entire text — and at that size its subject was
invisible.* Before shrinking an image, ask whether it can show its
subject at the size it is seen at; if not, the height is spent for
nothing.

**Render it and look at it.** Every rule in this section was found by
rendering through `gh api /markdown` and looking at the page. None of
them are visible while reading the markdown source.

## 7. The honest-status rule (two positions)

This is the type's distinguishing quality and the easiest thing to get
wrong in both directions. Burying an unproven status is dishonest;
putting the whole disclaimer above the fold buries the tool under it,
and a reader who does not yet know what the tool is cannot evaluate a
limitation.

- **One line above the fold** — a fact plus a pointer, no hedging.
- **The full section in its own `##` in the lower third.** It must
  (a) carry "Status" or "Limitations" in the heading so it is findable
  in the outline, (b) state numbers or say explicitly "unmeasured",
  (c) say **what would change the verdict**, and (d) not be the last
  thing before the footer — put `## Documentation` after it, so the
  page ends on a door and not a wall.

## 8. Every number carries its record

A research line is believed on its numbers, so a number without an
openable record is worse than no number.

- A number printed in the README, the changelog, or any durable
  document is either traceable to a stored run record or labelled
  **UNMEASURED**. Never estimate into a permanent file.
- A threshold quoted as a result names what it was measured against.
  A threshold chosen after seeing the results is not a threshold.
- Negative results are published, with the reason distinguished:
  *"no benefit was demonstrated"* and *"the cost was too high"* are
  different verdicts and must not be swapped.

## 9. Nothing may be reachable only by browsing

*Incident: one repository shipped a runnable example with zero
references anywhere in it, a second document referenced only from
inside a JSON snippet, and a starter directory named in prose but never
linked.* A reader asking the two most natural questions had runnable
answers sitting in the repo with no path to them.

Check with a link walker, and check for orphans by grepping each path.
After any rename, grep the old name repository-wide — nothing fails
when a cross-reference dies.

## 10. Checklist

- [ ] Hero alone on the first screen; no second image competing
- [ ] Badge row on one source line, every badge linked
- [ ] Lede names the runtime ("Python", "stdlib-only") — grep for it
- [ ] Status stamp above the fold, full `## Status` in the lower third
- [ ] `## Run it` present, fenced, language-tagged, offline, ≤60-char lines
- [ ] A real captured trace, literal, with a test binding it
- [ ] ≤9 `##` sections, no two starting with the same word
- [ ] Documentation as a table with a "when you need it" column
- [ ] Every README number traceable to a record, or labelled UNMEASURED
- [ ] No orphan files: every doc and template linked from somewhere
- [ ] No dead cross-references: grep old names after any rename
- [ ] Repo description + topics set, and topics do not contradict it
- [ ] Rendered and looked at, not just read as markdown
