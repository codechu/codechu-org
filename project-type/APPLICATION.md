# Application Conventions (language-agnostic)

Rules that apply to **any** Codechu end-user application, regardless
of language or platform. Language-specific extensions live under
[`lang/<name>/APPLICATION.md`](../lang/).

Read first: [`STANDARDS.md`](../STANDARDS.md).

## 1. Changelog discipline

Apps maintain **two** views of changes:

| View | Audience | Location |
|---|---|---|
| User-facing | End users — what changed in the product | `CHANGELOG.md` (Keep a Changelog) + release notes |
| Internal | Maintainers — refactors, build, infra | Commit history + `docs/` notes |

Don't mix the two. A user doesn't care that the test runner was
swapped; the user cares that startup got 200 ms faster.

## 2. Release artifacts

The release artifact is whatever a user installs. Per platform:

| Platform | Artifact |
|---|---|
| Linux | `.deb`, `.rpm`, Snap, Flatpak, AppImage |
| macOS | `.app` bundle, DMG, Mac App Store |
| Windows | `.exe` installer, MSI, MSIX |
| Mobile | App Store, Play Store, F-Droid, TestFlight |
| Web | Container image or static bundle |

Each platform's manifest lives under `packaging/<platform>/` in the
product repo. GitHub Releases collect links to all per-platform
artifacts.

## 3. Update mechanism

Apps declare how they update:

- **Distro-managed**: rely on the platform store (Snap, Flatpak,
  Microsoft Store, Mac App Store). No in-app updater needed.
- **Self-managed**: the app checks for new versions and prompts the
  user. The check is **opt-in or off-by-default** for offline-first
  apps; opt-out for connected services.
- **No silent updates** without a user-visible record of what changed
  (the user-facing changelog).

## 4. Telemetry policy

- **Default: off.** Telemetry is opt-in unless the app is a network
  service where it's intrinsic to the function.
- The settings UI exposes the toggle prominently — not buried.
- The telemetry payload schema is documented in `docs/TELEMETRY.md`.
- Personally identifiable data is never collected by default.
- A user with telemetry off must get the full product experience.

## 5. Settings & state

- Settings live in the platform's standard config location (see
  [`STANDARDS.md` §3](../STANDARDS.md#3-filesystem-layout-when-applicable)).
- Settings are documented — every key has a description in the
  settings UI or `docs/SETTINGS.md`.
- Migration logic for older settings layouts is mandatory; users
  should never lose their config across upgrades.

## 6. First-run experience

- App opens to a usable state without forcing the user through a
  setup wizard, unless the app cannot function without one (e.g.
  account-based service).
- Any required permission prompts (filesystem, network, notifications)
  are explained in the same screen they're requested on.

## 7. Internationalization (when applicable)

See [`STANDARDS.md` §11](../STANDARDS.md#11-internationalization-optional).

Apps own the user-facing rendering; localization lives in the app, not
in its libraries.
