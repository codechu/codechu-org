# Proprietary Application Conventions (language-agnostic)

Rules for **closed-source / source-available end-user applications**
sold to customers. Extends [`APPLICATION.md`](APPLICATION.md) and
[`STANDARDS.md`](../STANDARDS.md).

Open-source applications (e.g. `disk-cleaner`) follow
[`APPLICATION.md`](APPLICATION.md) only. This file applies when the
user pays for the binary.

> Legal templates live in `codechu-internal/legal/`. EULA + activation
> contracts must be reviewed by a lawyer before customer rollout.

---

## 1. License model

End-user applications almost always ship under an **EULA** rather
than an OSS or source-available license — the user is not a
developer reading the source; they need a contract that says what
they may and may not do with the installed binary.

| Choice | When |
|---|---|
| **EULA + proprietary** | Default. Standard commercial app. |
| **EULA + source-available (PolyForm)** | Customer wants audit rights; you still want to block competing forks. |
| **Open-core (MIT core + EULA addons)** | Free tier exists and is genuinely useful; paid features live in a closed plugin. See PROPRIETARY-LIBRARY.md §plugin pattern. |

**EULA requirement:** the customer must affirmatively accept on
first launch (click-through or signed order form). Implicit
acceptance via continued use is weaker and varies by jurisdiction.

EULA template:
`codechu-internal/legal/LICENSES/PROPRIETARY-TEMPLATE.md`. **Review
with a lawyer before any customer sees it.**

## 2. License key / activation

### 2.1 Token format

Default: **Ed25519-signed offline-verifiable license tokens.** A
license token is a JSON payload + Ed25519 signature, base64-encoded.
The app ships with the public verification key embedded; the private
signing key lives in a Codechu-controlled HSM or secrets vault.

Payload fields (recommended minimum):

| Field | Purpose |
|---|---|
| `customer_id` | Internal customer identifier (UUID — not the company name). |
| `product` | Product slug (e.g. `glyph-pro`). |
| `tier` | `trial` / `seat` / `org` / `site`. |
| `seats` | Integer; how many simultaneous activations allowed. |
| `issued_at` | ISO-8601. |
| `expires_at` | ISO-8601 or `null` for perpetual. |
| `features` | Array of feature flags this token unlocks. |
| `nonce` | Random; prevents identical tokens. |

The app validates signature, checks `expires_at`, and unlocks
features. **No network call required** for the happy path —
customers in air-gapped environments must work.

### 2.2 Online check (optional)

For SaaS-tied products, supplement the offline token with a
periodic online check (e.g. weekly) to detect revoked tokens. Grace
period **must** be at least 14 days of offline operation so a
revoked customer still gets a working app while they dispute.

### 2.3 Trial / free tier

- Trials are time-limited tokens (`expires_at` set). No special
  trial-mode code path — the trial is just a license with limited
  duration / features.
- Free tier (if any) ships with a built-in `tier=free` token whose
  signature the app verifies the same way. This keeps one code path
  for licensing.

### 2.4 Revocation

Revocation list (CRL-style): a signed JSON file of revoked
`customer_id` values, served from a Codechu endpoint. The app
fetches it on the online-check cadence. Without internet, the
last-cached CRL applies. Grace period above.

### 2.5 Seat model

| Model | Validation |
|---|---|
| **Per-seat** | The token's `seats` field caps simultaneous activations. Activation tracked via hardware fingerprint + Codechu activation server. |
| **Per-org** | `seats=∞`; the token is bound to an org domain. Suitable for site licenses. |
| **Per-machine** | `seats=1`, hardware-bound. Simplest model; least flexible for the customer. |

Default: **per-seat** with a soft activation server (customer can
deactivate from one machine to activate on another self-service).

## 3. Auto-update / delta patches

### 3.1 Signed releases mandatory

Every release is signed (cosign / minisign / Authenticode /
notarization on macOS). The updater verifies the signature against
a pinned trust root before applying any update.

### 3.2 Update channels

| Channel | Audience | Cadence |
|---|---|---|
| `stable` | Default for all customers. | Monthly + security patches. |
| `beta` | Opt-in. | Weekly. |
| `internal` | Codechu employees + design partners. | Per-commit / nightly. |

Channel switching is in the settings UI. Switching to `internal` is
guarded behind a token to prevent customers wandering into it
accidentally.

### 3.3 Forced vs optional updates

- **Forced updates** are reserved for security-critical patches.
  The UI tells the user *why* the update is mandatory and what was
  fixed.
- **Optional updates** are the default. The user can defer.
- **Skipped versions never become silently mandatory.** If a user
  skips three optional updates, the fourth doesn't suddenly force —
  unless one of them was security-critical, in which case that one
  forces on its own.

### 3.4 Rollback

The updater retains the previous version's binary for at least one
update cycle. A failed launch within 60 seconds of an update auto-
rolls-back. Manual rollback is documented in `docs/UPDATES.md`.

## 4. Telemetry & customer data

Strict superset of the OSS telemetry rules
(`codechu-internal/telemetry/POLICY.md`):

- **Opt-in default.** New installs see a first-launch screen that
  explains telemetry in plain language and asks for consent. The
  default radio button is "No, I'd rather not."
- **GDPR / KVKK compliance.** EU users get the GDPR notice (Art. 13
  + 14 disclosure). Turkish users get the KVKK aydınlatma metni.
  Both link to the full privacy policy.
- **No PII without explicit consent.** Names, emails, addresses,
  file paths, file contents — never collected by default. If a
  specific feature (e.g. crash reports) needs them, the user
  consents per-feature, not once globally.
- **Data residency.** EU users' telemetry goes to EU servers. The
  privacy policy says where. Other regions follow as customer
  contracts require.
- **Kill switch — three layers**:
  - CLI flag: `--no-telemetry`
  - Settings UI toggle (prominent — first item in the privacy tab)
  - Environment variable: `CODECHU_TELEMETRY=0`
- **Zero-network mode.** The app must function fully without any
  outbound network call beyond licensing checks. Verify in CI with
  a network-namespace-isolated smoke test.

## 5. Customer support SLA

Same tier table as PROPRIETARY-LIBRARY.md §10 — apps and libraries
share the support model. Defined per product in
`codechu-internal/products/<name>/support.md`. Customer-visible
terms live in the EULA / order form.

## 6. Code obfuscation / IP protection

Default: **none.** Source protection is friction with limited
return — a determined attacker reverses anything. Apply only to
genuinely high-IP components:

| Component | Protection |
|---|---|
| Bulk product code (UI, glue, business logic) | None. Ship as ordinary compiled artifacts. |
| Licensing / activation code | Compile with Cython / native extension. Makes "bypass the check" harder. Don't claim it's unbreakable. |
| Core algorithm IP (model preprocessing, novel solver) | Cython or Nuitka standalone. Tradeoff: harder to debug crash reports. |
| Cryptographic keys | Hardware-backed (HSM, OS keystore) **or** derived from user input. Never embedded as literal bytes — that's not protection, that's a delay. |

**Tradeoff calendar:** obfuscation costs ~2× debug time on customer
crash reports. Budget for the support cost before turning it on.

## 7. Update mechanism security

| Concern | Rule |
|---|---|
| Signature verification | Mandatory before apply. Failure = no update, log the event. |
| Trust root | Pinned in the binary. Rotating it requires a normal release that bundles the new root next to the old (overlap period). |
| Channel switching | UI-driven; no environment-variable override that could be flipped silently by malware. |
| Download integrity | HTTPS + SHA-256 manifest. The manifest itself is signed. |
| Update server identity | Pinned hostname + certificate pinning when feasible. |
| Replay protection | Updates have monotonically increasing version numbers. Updater refuses to "update" to an older version unless the user explicitly rolls back from the UI. |

## 8. References

- [`STANDARDS.md`](../STANDARDS.md) — org-wide rules
- [`APPLICATION.md`](APPLICATION.md) — base application conventions
- [`PROPRIETARY-LIBRARY.md`](PROPRIETARY-LIBRARY.md) — closed-source libraries
- `codechu-internal/legal/LICENSES/PROPRIETARY-TEMPLATE.md` — EULA
- `codechu-internal/telemetry/POLICY.md` — telemetry policy
- GDPR Articles 13, 14, 15, 17 — disclosure + access + deletion rights
- KVKK (Türkiye) — Kişisel Verilerin Korunması Kanunu
