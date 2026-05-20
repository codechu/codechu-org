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

## 8. README is the vitrine — application skeleton

The vitrine principle from [STANDARDS §7.1](../STANDARDS.md#71-readme-is-the-vitrine)
applies to applications with an app-flavoured skeleton:

```markdown
[banner / logo]

[![Release](...)] [![CI](...)] [![License](...)] [package-channel badges]

> *one-line tagline*

# app-name

[1–3 sentence value proposition — what user problem this solves]

[ONE hero visual: prominent screenshot or screen recording. GUI apps
 require a real screenshot from the running app; CLI apps require a
 terminal cast (asciinema SVG / GIF) of a typical session.]

## Install

| Channel | Command |
|---|---|
| Flatpak | `flatpak install flathub <id>` |
| Snap | `snap install <name>` |
| AppImage | download from [Releases](...) |
| Distro package | per-distro instructions linked |
| From source | `pip install -e .` *(or equivalent)* |

(Only list the channels actually shipping.)

## Features

- feature bullet 1
- feature bullet 2
   (5–10 bullets of *what the user gets*, not *what the code does*)

## Screenshots

(Optional second-tier visual section — 2–4 additional screenshots
showing different views or states. Skip if the hero already says
enough.)

## Read more

- [User guide](docs/GUIDE.md) — task-oriented walkthrough *(when applicable)*
- [Settings reference](docs/SETTINGS.md) — every preference
- [Telemetry policy](docs/TELEMETRY.md) — what the app collects, if any
- [Architecture](docs/ARCHITECTURE.md) — for contributors
- [Changelog](CHANGELOG.md)

## Privacy

(One paragraph if telemetry exists; "no telemetry" sentence otherwise.
Linked to `docs/TELEMETRY.md` for detail.)

## License

[License name] — see [LICENSE](LICENSE).

Part of [Codechu](https://github.com/codechu).
```

**Application-specific hero requirements:**

| Application kind | Hero requirement |
|---|---|
| GUI desktop | Real screenshot of running app on Linux (primary platform), preferred in light + dark if both supported |
| CLI tool | Terminal cast (asciinema SVG / GIF) of a representative invocation |
| TUI / curses | Asciinema or screenshot of a typical view |
| Web | Annotated screenshot of the canonical view; for SPAs include a short cast |
| Daemon / background service | Architectural diagram (Mermaid) showing inputs and outputs |

The prohibition list from STANDARDS §7.1 still holds: no full options
reference, no manual content, no extension catalogue — those live in
`docs/`.
