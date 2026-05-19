# New Product Checklist

Step-by-step procedure for starting a new Codechu product. Independent of
language, platform, license — only identity, repo, branding, and process
items are mandated. Technology choices belong to the product team.

## 0. Decision gate (block if undone)

- [ ] Product name decided (`kebab-case`, max 2 words)
- [ ] Reverse-DNS App ID: `com.codechu.<ProductName>` (CamelCase)
- [ ] One-line pitch (≤80 chars) written
- [ ] **Programming language** chosen
- [ ] **Target platform(s)** chosen (Linux / Windows / macOS / mobile / web / cross-platform)
- [ ] **License model** chosen (open-source, source-available, proprietary)
- [ ] **Public vs private** initial release strategy decided
- [ ] No identity overlap with other Codechu products

## 1. Repository bootstrap

```bash
# Public product repo (adjust visibility if private)
gh repo create codechu/<product> --public --description="<one-liner>"
cd <product> && git init
git config user.name "<your-name>"
git config user.email "<your-email>"
```

- [ ] Default branch is `main`
- [ ] `.gitattributes` added (LF normalize) — copy from this repo
- [ ] `.gitignore` covers chosen toolchain (build artifacts, IDE, OS)
- [ ] Pre-commit hook installed (adapt from `codechu-org/hooks/pre-commit`)

## 2. Scaffold (files that apply to every product)

```
README.md            # canonical, English
README.tr.md         # optional, Turkish parallel
LICENSE              # whichever license the product carries
CHANGELOG.md         # Keep a Changelog format
CODE_OF_CONDUCT.md   # Contributor Covenant ref (or internal version)
CONTRIBUTING.md      # contributor onboarding or "closed contributions" notice
SECURITY.md          # vuln disclosure (still relevant for proprietary)
.github/
├─ ISSUE_TEMPLATE/
└─ PULL_REQUEST_TEMPLATE.md
```

- [ ] All `<product>` placeholders replaced
- [ ] LICENSE filled (year + Codechu + chosen license terms)
- [ ] README hero references brand lockup
- [ ] Documentation skeleton in `docs/`

## 3. Identity / Branding (mandatory)

- [ ] Read [BRAND-GUIDELINES.md](BRAND-GUIDELINES.md)
- [ ] Palette + typography applied per shared rules
- [ ] Product-specific logo glyph designed (`mark.svg`)
- [ ] Lockup variants (horizontal, stacked) produced
- [ ] Knockout + monochrome mark variants ready
- [ ] Platform icons: native size set for each target platform
- [ ] Social preview / Open Graph image (1280×640 PNG)
- [ ] `docs/BRAND.md` documents product-specific usage rules

## 4. Filesystem layout (if product stores user data)

Use the vendor namespace appropriate for the target platform:

- **Linux / Unix**: XDG (`~/.config/codechu/<product>/`, etc.)
- **macOS**: `~/Library/Application Support/Codechu/<Product>/`
- **Windows**: `%APPDATA%\Codechu\<Product>\`
- **Mobile / web**: native sandbox + `com.codechu.<Product>` bundle ID

- [ ] Path constants centralized (single config / paths module)
- [ ] Migration helper from older path layouts (if any)
- [ ] No hardcoded `/tmp/`, `%TEMP%`, etc. — all under the platform's
      proper runtime directory

## 5. Internationalization (if applicable)

- [ ] i18n wrapper module exists
- [ ] All user-facing strings flagged for translation
- [ ] Extraction tool configured (gettext / equivalent)
- [ ] Turkish translation ship-included if practical
- [ ] User-visible language selector (settings UI) if multilingual
- [ ] CI guard against untranslated strings

## 6. CI / CD

- [ ] Lint / format step (language-specific tool)
- [ ] Unit tests + coverage threshold (≥60% for most products)
- [ ] Static security scan (for products handling user data or with destructive operations)
- [ ] Build artifact step (binary, installer, package)
- [ ] Release workflow (tag push → artifact upload)
- [ ] Branch protection on main (mature products)

## 7. Packaging (depends on target platform)

Items below as applicable to the platform — not all apply to every product.

- [ ] Linux: `.desktop` + `.appdata.xml` + Debian / AppImage / Flatpak / Snap
- [ ] macOS: `.app` bundle + DMG / notarization / Mac App Store
- [ ] Windows: `.exe` / MSI / MSIX / Windows Store
- [ ] Mobile: TestFlight / App Store / Google Play / F-Droid
- [ ] Web: static hosting / Docker / Kubernetes manifest
- [ ] Library: language-registry publish (npm, PyPI, crates.io, Maven)

## 8. Pre-release smoke test

- [ ] All tests pass
- [ ] Lint clean
- [ ] App / library starts and exercises core flow on target platform
- [ ] Branding assets validated (icon renders at all required sizes)
- [ ] Translations compile / load
- [ ] Release workflow rehearsed with `v0.0.1-rc1`

## 9. First release (v0.1.0)

- [ ] CHANGELOG `[0.1.0]` entry
- [ ] Version bumped in build metadata
- [ ] `git tag -a v0.1.0` + push tag
- [ ] Release notes published
- [ ] Update [codechu-org/README.md](README.md) product roster

## 10. Optional — distribution channels

Per target platform; pick what fits the product:

- [ ] GitHub Releases (binary downloads)
- [ ] Platform-native stores (Mac App Store, Microsoft Store, Snap Store, Flathub, F-Droid, Play, App Store)
- [ ] Package registries (PyPI, npm, crates.io, Homebrew, Chocolatey, Scoop, AUR)
- [ ] Distribution-specific repos (Launchpad PPA, OBS, Copr)
- [ ] Launch announcement (Reddit, Hacker News, Mastodon, blog)

---

**Note:** This checklist is living. When a new product reveals a missing
step, PR against this file. Items already handled by the platform's
defaults can be skipped — only the **mandatory** sections (1, 2, 3) are
non-negotiable.
