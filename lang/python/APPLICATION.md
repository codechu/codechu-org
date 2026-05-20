# Python Application Conventions

Conventions for **Python end-user applications** under the `codechu`
namespace. The canonical example is
[`codechu/disk-cleaner`](https://github.com/codechu/disk-cleaner) — a
GTK 3 + PyGObject + Cairo desktop app.

This file is intentionally lighter than
[`LIBRARY.md`](LIBRARY.md). Most discipline (pyproject, ruff, pytest,
coverage floor) is shared. Only the bits that **differ** for apps are
listed here.

Read first:

- [`STANDARDS.md`](../../STANDARDS.md)
- [`project-type/APPLICATION.md`](../../project-type/APPLICATION.md)
- [`lang/python/LIBRARY.md`](LIBRARY.md) — most rules carry over

## 1. Apps are not published to PyPI

Distribution is platform-native: Snap, Flatpak, AppImage, `.deb`,
`.rpm`. PyPI is for libraries.

A Python app **may** still ship a `pyproject.toml` for development
ergonomics (`pip install -e .` in a venv), but the release artifact is
never `sdist` / `wheel`.

## 2. Dependency declaration

Apps depend on Codechu libraries with **declared version ranges**:

```toml
dependencies = [
    "codechu-events>=0.2,<0.3",
    "codechu-xdg>=0.1,<0.2",
    "codechu-treeviz>=0.1,<0.2",
    "PyGObject>=3.42",
]
```

Use `>=X.Y,<X.(Y+1)` for pre-1.0 libraries (minor bumps may be
breaking under SemVer 0.y conventions), `>=X,<X+1` afterwards.

## 3. Repo structure

Apps do **not** use the `src/` layer:

```
disk_cleaner/             # app package at repo root
    __init__.py
    main.py
    controllers/
    views/
    scanners/
tests/
po/                       # gettext translations
assets/                   # icons, screenshots
packaging/                # snap, flatpak, deb manifests
docs/
```

Rationale: there's no PyPI publish, no namespace collision risk, and a
flatter tree is easier for new contributors to navigate.

## 4. Singletons

App-level singletons are **fine** — there's exactly one process, one
config, one event bus.

```python
# OK in an application
from codechu_events import Bus
bus = Bus()  # module-level singleton
```

Library-level singletons are **forbidden** — see
[`project-type/LIBRARY.md`](../../project-type/LIBRARY.md) §"Public
API contract".

## 5. App-specific concerns

| Concern | Notes |
|---|---|
| i18n | `gettext` + `po/` directory; canonical source is English; Turkish ships when practical |
| Settings UI | Backed by `~/.config/codechu/<product>/settings.toml` (or platform equivalent) |
| GUI testing | Controllers/scanners must be **headless-safe** (no GTK imports at module top level) — only the view layer touches GTK |
| Packaging | `packaging/snap/`, `packaging/flatpak/`, etc.; each platform's manifest lives in its own subdir |
| App ID | Reverse-DNS `com.codechu.<ProductName>` — used by `.desktop`, `.appdata.xml`, DBus, GSettings |
| Control socket | `$XDG_RUNTIME_DIR/codechu/<product>/control.sock` if the app exposes one |

## 6. Test layout

Tests follow the library convention but with an extra split:

```
tests/
    test_*.py             # headless unit tests (CI default)
    gui/test_*.py         # GTK-required integration tests (manual / X11 CI)
```

`pytest` defaults to the headless set. The `gui/` set runs only when
explicitly invoked (`pytest tests/gui`) so `pytest -q` stays green on
plain runners.

## 7. Release artifacts

The first-class release artifact is whatever the user installs:

- **Snap:** `snapcraft.yaml` + Snap Store upload
- **Flatpak:** `com.codechu.<Product>.yml` + Flathub PR
- **AppImage:** `appimage-builder.yml` + GitHub Release attachment
- **.deb:** `debian/` directory + Launchpad PPA

GitHub Release notes link to all platform artifacts. Tag → workflow →
artifact, same as the library flow, but the workflow uploads to the
platform store instead of PyPI.
