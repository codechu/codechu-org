# Overlays

An **overlay** is a discipline a repository adopts *in addition to* its
project type. Project types narrow (a library is a kind of repository);
overlays add (a library may or may not publish verdicts).

| Overlay | Adopt it when | Rules |
|---|---|---|
| **Published verdict** | The repo's product is a claim — a finding, a score, a verdict — that could come out negative | [PUBLISHED-VERDICT.md](PUBLISHED-VERDICT.md) |

Read in order: [`STANDARDS.md`](../STANDARDS.md) →
[`project-type/`](../project-type/) → [`lang/`](../lang/) → overlays.
An overlay may suspend a specific rule from an earlier layer, but it
must say which rule and why; silence means the earlier layer stands.
