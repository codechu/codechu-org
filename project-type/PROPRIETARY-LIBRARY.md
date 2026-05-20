# Proprietary Library Conventions (language-agnostic)

Rules for **closed-source / source-available libraries** distributed
to paying customers. Extends — never weakens — the base
[`LIBRARY.md`](LIBRARY.md) and [`STANDARDS.md`](../STANDARDS.md).

If you are starting an **open-source MIT** library, you want
[`LIBRARY.md`](LIBRARY.md). This file applies when the artifact is
sold or distributed under a license that restricts redistribution.

> Legal templates (EULA, PolyForm, BUSL boilerplate) live in
> `codechu-internal/legal/`. They are templates, not legal advice.
> Run them past a lawyer before signing a customer.

---

## 1. Repository placement

| Placement | When |
|---|---|
| `codechu-internal/products/<name>/` | Pre-release: design notes, prototype code, customer drafts. Always private. |
| Separate private repo `codechu/<name>` (visibility = private) | Once the library has a customer-facing release surface, give it its own private repo. Keeps history clean and lets per-repo branch protection apply. |
| **Never** in `codechu-org/`, `codechu-fmt`, or any other public namespace | Public namespace is reserved for MIT artifacts. |

A library may **start** in `codechu-internal/products/<name>/` and
graduate to its own private repo when it has paying customers — same
two-tier discipline as STANDARDS.md §10, with the public/private
boundary moved one notch closer to the maintainer.

## 2. License model selection

Choose the most permissive license that still protects the business
goal. Trade-offs:

| License | Customer can read source | Customer can self-host | Customer can fork & ship | Best for |
|---|---|---|---|---|
| **PolyForm Strict 1.0.0** | yes (eval only) | no | no | Pure commercial libs; defends against verbatim reuse |
| **PolyForm Shield 1.0.0** | yes | yes (non-compete) | no (no competing product) | SaaS-adjacent libs; blocks competitors only |
| **PolyForm Perimeter 1.0.0** | yes | yes (non-compete + scope) | no | Libs intended for embedding in customer's own product |
| **BUSL 1.1** | yes | yes (after change date: open) | yes (after change date) | Eventually-open infra (e.g. databases). Pick a Change Date and Change License up front. |
| **Proprietary (EULA only)** | no (binary-only) | depends on EULA | no | High-IP libs, model weights, anything where source-read is itself the leak |

Default for a **new commercial Codechu library**: **PolyForm Shield
1.0.0** — protects the business without blocking embedding. Move to
PolyForm Strict only if a customer needs source for audit but must
not self-host. Move to fully proprietary only when source itself
contains protected IP (training data preprocessing, novel
algorithms).

Document the chosen model and the rationale in
`docs/LICENSE-RATIONALE.md` (internal-only — strategic context).

## 3. License header in source files

Every source file carries an SPDX line plus a short copyright. Use
the matching template under
`codechu-internal/legal/source-headers/`.

PolyForm Shield example:

```
# SPDX-License-Identifier: LicenseRef-PolyForm-Shield-1.0.0
# Copyright (c) 2026 Codechu. All rights reserved.
# Distributed under PolyForm Shield 1.0.0 — see LICENSE.
```

Proprietary example:

```
# Copyright (c) 2026 Codechu. All rights reserved.
# Confidential and proprietary. Distribution prohibited without a
# signed license from Codechu. See LICENSE.
```

The header is mandatory in **every** source file, including tests
and tooling scripts that ship to the customer. Auto-enforce in CI
(`ruff check --select=...` plus a custom header-grep step, or a
pre-commit hook).

## 4. Build & distribution policy

- **Binary-only releases** are the default. Customers receive
  compiled artifacts (wheels, JARs, DLLs, native binaries) — never a
  source tarball — unless the EULA explicitly grants source access.
- **Sealed builds.** Releases are built from a CI runner with no
  developer SSH; the build identity is the workflow itself
  (OIDC-based signing). No "I built it on my laptop" releases.
- **Reproducibility.** Pin all build inputs (lockfile, base image
  digest, compiler version). A release built today must be
  bit-reproducible six months from now during a vulnerability triage.
- **Signed artifacts.** Sign every release with the Codechu release
  key (cosign / sigstore for OCI; minisign / GPG for tarballs;
  Authenticode for Windows). Verification instructions ship in
  `docs/VERIFY.md`.
- **Private distribution channel.** Wheels / packages go to a
  private index (devpi, pypiserver behind auth, GitHub Releases with
  a per-customer token) — never to public PyPI/crates.io/npm.

## 5. Customer-facing API contract

Stricter than the OSS rules because customers paid for stability.

| Item | Rule |
|---|---|
| Public API surface | Documented in `docs/API.md`. What's not in `docs/API.md` is not public. |
| Deprecation window | **Minimum 2 minor versions** (vs 1 for OSS). |
| Breaking change SLA | Customer notification ≥90 days before a major release containing breaks. |
| Migration guide | Required for every major. Sample code, not just prose. |
| Support contracts | Define which versions remain supported (e.g. last 2 majors + current). |

## 6. Telemetry

Closed-source libraries follow the org telemetry policy
(`codechu-internal/telemetry/POLICY.md`). Summary:

- **Opt-in default.** A library that calls home on import is
  forbidden.
- **Transparent disclosure.** README + `docs/TELEMETRY.md` describe
  what is collected, when, how to disable.
- **No PII** in default events: no usernames, file paths, URLs,
  email addresses, IP addresses (hashed only).
- **Kill switch:** environment variable + library-level API
  (`set_telemetry(False)`) + config file.
- **Third-party processors** (Sentry, PostHog, etc.) declared in the
  privacy notice with the DPA reference.

## 7. Source-code redistribution to customers

When a customer needs source (audit, allowlisting, regulatory
compliance):

1. The base EULA does **not** grant source — even PolyForm Strict
   only grants source for *evaluation*.
2. A **separate source-access addendum** is signed. Template in
   `codechu-internal/legal/PROPRIETARY-TEMPLATE.md` §addendum.
3. Source is delivered as a **time-bounded snapshot** (one tag, no
   ongoing git access) unless the addendum says otherwise.
4. The snapshot is sanitized: no internal `# TODO(customer-name)`
   markers, no credentials, no references to other customers.
5. Snapshot delivery is logged in
   `codechu-internal/customers/<customer>/source-access.md`.

## 8. Internal → customer code promotion workflow

A library matures internally before reaching a customer. The
promotion path:

```
codechu-internal/products/<name>/   →   private repo codechu/<name>
  (prototype + design notes)            (customer-facing releases)
```

Promotion checklist (run before the first customer-tagged release):

- [ ] Secrets sweep: `gitleaks detect --source . --redact` clean
- [ ] No customer names in commit messages, file names, comments
- [ ] No internal hostnames, dashboard URLs, vendor account IDs
- [ ] No pricing or financial commentary in code or docs
- [ ] LICENSE present and matches the chosen model
- [ ] `docs/LICENSE-RATIONALE.md` exists (internal copy)
- [ ] EULA / customer-facing license reviewed by lawyer
- [ ] CHANGELOG starts fresh at the customer-facing v1.0.0 (or
      carries a "pre-customer" note for earlier history)
- [ ] CI signs release artifacts
- [ ] Private distribution channel set up + smoke-tested
- [ ] `SECURITY.md` declares the disclosure path (security@codechu.com)

## 9. Versioning + changelog

Same SemVer scheme as OSS libraries, with two changelog views:

| View | Audience | Location |
|---|---|---|
| Customer-facing | The customer | `CHANGELOG.md` in the private repo. Shipped with the release. |
| Internal | Codechu maintainers | `codechu-internal/products/<name>/CHANGELOG-internal.md`. Captures pricing changes, contract notes, customer-specific patches. |

The two never merge. A customer never sees the internal changelog.

## 10. Support model

Each commercial library declares a support tier. Template:

| Tier | Response SLA | Channels | Versions supported |
|---|---|---|---|
| Bronze | 5 business days | Email | Current major only |
| Silver | 2 business days | Email + shared Slack channel | Current major + previous |
| Gold | 1 business day (24/7 for sev-1) | Dedicated engineer | Current + 2 previous majors |

Define the tier mix in `codechu-internal/products/<name>/support.md`
along with the on-call rotation. Customer-visible support terms live
in the EULA / order form, not in the public repo.

## 11. Vulnerability disclosure

Override of STANDARDS.md §10 for closed-source products:

1. **Internal triage first** — `codechu-internal/sec-advisories/`.
2. **Customer notification before public disclosure.** Customers
   receive a private advisory + patched release **at least 7 days
   before** any public CVE filing.
3. **Patch backports.** Vulnerabilities are patched on every
   supported version per the support tier table — not only the
   latest.
4. **No public detail until patches are available.** CVE entry,
   blog post, or release notes describing the vulnerability wait
   until patched releases are out and customers have had time to
   apply.
5. Coordinated disclosure window: 90 days from internal report to
   public announcement, extendable by mutual agreement with the
   reporter.

## 12. References

- [`STANDARDS.md`](../STANDARDS.md) — org-wide rules (license policy
  §9, two-tier model §10)
- [`LIBRARY.md`](LIBRARY.md) — base library conventions (SemVer,
  API, testing, etc. — still apply)
- [`PROPRIETARY-APPLICATION.md`](PROPRIETARY-APPLICATION.md) —
  closed-source applications
- `codechu-internal/legal/` — license templates + EULA + source headers
- `codechu-internal/telemetry/POLICY.md` — telemetry policy
