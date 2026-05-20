# Python Proprietary Conventions

Python-specific rules for **closed-source / source-available**
libraries and applications. Companion to [`LIBRARY.md`](LIBRARY.md)
(OSS-focused) and [`APPLICATION.md`](APPLICATION.md).

Extends:

- [`STANDARDS.md`](../../STANDARDS.md)
- [`project-type/PROPRIETARY-LIBRARY.md`](../../project-type/PROPRIETARY-LIBRARY.md)
- [`project-type/PROPRIETARY-APPLICATION.md`](../../project-type/PROPRIETARY-APPLICATION.md)

If a rule here conflicts with a language-agnostic rule, the
language-agnostic one wins unless the conflict is a documented
Python-only deviation.

---

## 1. Distribution format

| Format | When | Pros | Cons |
|---|---|---|---|
| **Wheel, no sdist** | Default for libraries. | Ships compiled-ish form; pip just works. | Pure-Python wheels are still trivially decompiled (`unzip`, `dis`). |
| **Cython-compiled binary wheels (`cython --embed`)** | Libraries with IP worth protecting. | Source replaced by `.so`/`.pyd`; harder to read. | Per-platform builds (manylinux × macOS × Windows × arch). Slower CI. |
| **Nuitka standalone** | End-user applications. | Single-file or self-contained dir, harder to inspect, no Python runtime requirement on the host. | Larger artifact; longer build; some C-extension compat work. |
| **PyInstaller** | Quick app packaging when Nuitka is overkill. | Mature, easy. | Inspectable: `pyinstxtractor` reverses bundles trivially. Use only with Cython-protected modules inside. |
| **Bytecode-only `.pyc`** | **Never.** | — | Effectively unprotected (`uncompyle6`). Listed only to mark as forbidden. |

Default recipe for a commercial Python library:

```toml
# pyproject.toml additions for cython-compiled binary wheel
[build-system]
requires = ["setuptools>=68", "wheel", "Cython>=3.0"]
build-backend = "setuptools.build_meta"
```

`setup.py` (yes, here we need it for Cython):

```python
from setuptools import setup
from Cython.Build import cythonize

setup(
    ext_modules=cythonize(
        ["src/codechu_<name>/**/*.py"],
        compiler_directives={"language_level": "3"},
    ),
)
```

Build per-platform wheels with `cibuildwheel`. Do **not** publish
an sdist (`python -m build --wheel` only) — shipping sdist alongside
defeats the purpose.

## 2. Source protection — choosing a method

```
Risk: customer reads code     Method
────────────────────────────  ──────────────────────────────
Low (trust customer)          Pure-Python wheel, no obfuscation
Medium                        Cython --embed compile of hot/IP modules
High (algorithm secret)       Nuitka standalone for apps;
                              Cython + only-binary index for libs
Very high (model weights)     Hardware-bound license + encrypted assets
                              decrypted with per-customer keys
```

Obfuscate **only the IP-bearing modules**, not the whole package.
A 90% pure-Python package with 10% Cython-compiled core ships
faster, debugs better, and protects what matters.

## 3. License key library

Recommended stack:

```python
# pip install cryptography
from cryptography.hazmat.primitives.asymmetric.ed25519 import (
    Ed25519PublicKey,
)
import base64, json

PUBLIC_KEY_BYTES = b"<32-byte-ed25519-public-key>"  # embedded literal

def verify_license(token: str) -> dict:
    """Verify an Ed25519-signed license token. Returns the payload
    dict on success, raises on failure."""
    payload_b64, sig_b64 = token.rsplit(".", 1)
    payload = base64.urlsafe_b64decode(payload_b64 + "==")
    signature = base64.urlsafe_b64decode(sig_b64 + "==")
    key = Ed25519PublicKey.from_public_bytes(PUBLIC_KEY_BYTES)
    key.verify(signature, payload)              # raises on bad sig
    data = json.loads(payload)
    # expiry, feature, tier checks here
    return data
```

Issuer side (Codechu-controlled, runs against the HSM / vault):
`cryptography.hazmat.primitives.asymmetric.ed25519.Ed25519PrivateKey`
signs the same `payload`. Never embed the private key anywhere — not
in CI logs, not in test fixtures, not in `internal-tools/` repos.

Recommended internal tooling location:
`codechu-internal/tools/license-issuer/`. Air-gapped machine or HSM
preferred.

## 4. Telemetry

Use `codechu-events` (the OSS in-process event bus) for emission, and
a thin first-party shipper module for HTTP transport. Do **not**
bundle a third-party SDK (Sentry, PostHog) into the library itself —
those go in the host application only.

Pattern:

```python
from codechu_events import bus

bus.publish("license.verified", {"tier": tier, "ts_minute": ts})
# A separate opt-in shipper subscribes and POSTs to Codechu telemetry endpoint.
```

The shipper module:

- Reads the kill-switch (env, settings, `--no-telemetry`).
- Batches events; drops on offline.
- Strips PII per the policy in
  `codechu-internal/telemetry/POLICY.md`.

## 5. Build pipeline

### 5.1 Private repo

`codechu/<name>` with visibility **private**. Branch protection on
`main`: required CI + 1 review. The repo lives under the
`codechu/` GitHub org alongside the public repos — visibility is the
boundary, not org membership.

For purely-internal libraries that never reach customers, keep them
under `codechu-internal/products/<name>/` instead of a separate repo.

### 5.2 GitHub Actions matrix

```yaml
# .github/workflows/release.yml — sketch
on:
  push:
    tags: ["v*"]

jobs:
  build:
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest, windows-latest]
        python: ["3.10", "3.11", "3.12", "3.13"]
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with: { python-version: ${{ matrix.python }} }
      - run: pip install cibuildwheel
      - run: cibuildwheel --output-dir dist
      - uses: actions/upload-artifact@v4
        with: { name: wheels-${{ matrix.os }}-${{ matrix.python }}, path: dist }

  publish:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/download-artifact@v4
      - name: Upload to private devpi
        env:
          DEVPI_TOKEN: ${{ secrets.DEVPI_TOKEN }}
        run: devpi upload --index codechu/prod dist/*
      - name: Sign wheels
        run: |
          for f in dist/*.whl; do
            cosign sign-blob --yes --output-signature "$f.sig" "$f"
          done
```

### 5.3 Private index

Options ranked:

1. **devpi** behind nginx + basic auth — preferred. Mirrors PyPI for
   transitive deps; serves Codechu wheels at the same URL.
2. **pypiserver** — simpler; no mirror. Fine for small customer
   counts.
3. **GitHub Releases with per-customer tokens** — works, but no
   `pip install -U` UX; customer must download then `pip install
   ./codechu_x-1.2.3-cp312-...whl`. Acceptable for low-volume
   distribution.

## 6. Customer install flow

### 6.1 Index-based (devpi / pypiserver)

```bash
pip install --index-url https://<customer-token>@pkg.codechu.com/simple/ \
            --extra-index-url https://pypi.org/simple/ \
            codechu-<name>
```

Customer credentials go in `~/.netrc` or `pip.conf` (instructions in
`docs/INSTALL.md` shipped with the order).

### 6.2 Bootstrap installer

For apps that want a more curated experience: ship a single
bootstrap script (`codechu-install.py`) that:

1. Verifies the customer license key.
2. Fetches signed wheels for the host platform.
3. Verifies signatures against the embedded trust root.
4. Installs into a project-local venv.

The bootstrap itself is unsigned-but-public; trust derives from the
embedded keys + signed payloads, not from the script.

## 7. Mixed OSS + proprietary (open-core)

The canonical Codechu open-core pattern:

```
codechu-glyph         (public, MIT)         — core engine, registries,
                                              entry-point discovery
codechu-glyph-pro     (private, EULA)       — premium features as a plugin
codechu-glyph-figlet  (public, MIT)         — community plugin example
```

The core declares the plugin contract via Python entry points:

```toml
# In the host's pyproject.toml — plugins register here.
[project.entry-points."codechu_glyph.plugins"]
pro = "codechu_glyph_pro:register"
```

At plugin load time, the closed-source plugin verifies its license:

```python
# codechu_glyph_pro/__init__.py
from codechu_glyph_pro._license import require_valid_license

def register(host):
    require_valid_license(feature="glyph.pro")   # raises if invalid
    host.add_renderer(...)
```

Caveats:

- The core stays MIT and has no awareness of which plugins are paid.
- The plugin enforces its own license — the core never asks.
- Customers can use the core without the plugin; the open-core
  promise must be real (the free core must be genuinely useful).

## 8. Update mechanism for Python apps

### 8.1 Libraries

Customers run `pip install -U codechu-<name>` against their
configured private index. No bundled updater needed — pip is the
updater.

### 8.2 Applications

Bundle a small updater module:

```python
# codechu_<app>/_updater.py — sketch
def check_for_updates(channel="stable"):
    manifest = fetch_signed_manifest(channel)
    verify_manifest_signature(manifest)
    if manifest["version"] > current_version():
        return manifest
    return None
```

The signed manifest contains: version, URLs of platform wheels, SHA-
256 of each wheel, signature over the whole manifest. The updater
verifies signature, downloads, verifies SHA, and uses `pip install`
into the app's venv (or the standalone Nuitka bundle replaces
itself).

See PROPRIETARY-APPLICATION.md §3 for the channel / rollback rules.

## 9. Secrets management

- **Never commit secrets** — to either repo. `codechu-internal/`
  being private is **not** an excuse.
- Use environment variables for runtime config. For CI/CD: GitHub
  Actions secrets (org-level for shared keys, repo-level for
  per-product).
- For developer machines: a vault (`pass`, `1Password CLI`,
  HashiCorp Vault). Bootstrap script populates env from vault on
  shell start.
- **Pre-commit hook** scans for accidental commits:

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.18.0
    hooks:
      - id: gitleaks
```

The hook lives in `codechu-org/hooks/pre-commit` alongside the other
canonical hooks once we cut a hooks release; until then copy from
this file.

## 10. Checklist (proprietary Python release)

```markdown
- [ ] Repo visibility = private
- [ ] LICENSE matches chosen model (PolyForm / BUSL / Proprietary)
- [ ] License header on every source file
- [ ] docs/LICENSE-RATIONALE.md (internal-only) explains the choice
- [ ] Cython compile configured for IP-bearing modules (if any)
- [ ] cibuildwheel matrix green on Linux + macOS + Windows
- [ ] Wheels signed with cosign (or platform-native signer)
- [ ] Private index credentials documented in docs/INSTALL.md
- [ ] License-key infra: Ed25519 keypair generated, private key in vault
- [ ] Telemetry kill switch tested (env + flag + setting)
- [ ] Zero-network smoke test passes
- [ ] EULA reviewed by lawyer
- [ ] CHANGELOG-internal.md created
- [ ] Support tier defined in codechu-internal/products/<name>/support.md
- [ ] SECURITY.md declares security@codechu.com
- [ ] gitleaks pre-commit hook installed
```

## 11. References

- [`STANDARDS.md`](../../STANDARDS.md)
- [`LIBRARY.md`](LIBRARY.md) — base Python library rules (still apply)
- [`APPLICATION.md`](APPLICATION.md) — base Python application rules
- [`project-type/PROPRIETARY-LIBRARY.md`](../../project-type/PROPRIETARY-LIBRARY.md)
- [`project-type/PROPRIETARY-APPLICATION.md`](../../project-type/PROPRIETARY-APPLICATION.md)
- `codechu-internal/legal/` — license templates + EULA
- `codechu-internal/telemetry/POLICY.md`
