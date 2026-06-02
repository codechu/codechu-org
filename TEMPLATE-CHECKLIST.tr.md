# Yeni Ürün Checklist'i

Yeni bir Codechu ürünü başlatma prosedürü. Dil, platform, lisanstan
bağımsız — sadece kimlik, repo, branding ve süreç maddeleri zorunlu.

> İngilizce kanonik: [TEMPLATE-CHECKLIST.md](TEMPLATE-CHECKLIST.md).
> Bu Türkçe paralel.

## 0. Karar kapısı (yapılmadıysa devam etme)

- [ ] Ürün adı belirlendi (`kebab-case`, ≤2 kelime)
- [ ] Reverse-DNS App ID: `com.codechu.<ProductName>` (CamelCase)
- [ ] One-liner pitch (≤80 ch) yazıldı
- [ ] **Programlama dili** seçildi
- [ ] **Hedef platform(lar)** seçildi (Linux/Windows/macOS/mobil/web/cross-platform)
- [ ] **Lisans modeli** seçildi (açık kaynak / kaynak-erişimli / proprietary)
- [ ] **Public vs private** başlangıç stratejisi belirlendi
- [ ] Diğer Codechu ürünleriyle kimlik çakışması yok

## 1. Repository bootstrap

```bash
gh repo create codechu/<product> --public --description="<one-liner>"
cd <product> && git init
git config user.name "<your-name>"
git config user.email "<your-email>"
```

- [ ] **Repo şekli kararlaştırıldı** (STANDARDS §2 "Repo granülerliği"):
      bağımsız library → library başına bir repo; version-lock'lu framework
      (core + lockstep bump olan adapter'lar) → tek monorepo, çok paket
- [ ] Default branch `main`
- [ ] `.gitattributes` eklendi (LF normalize) — codechu-org'dan kopyala
- [ ] `.gitignore` toolchain'i kapsıyor (build artifacts, IDE, OS)
- [ ] Pre-commit hook kuruldu (codechu-org/hooks/pre-commit'ten adapt edilir)

## 2. Scaffold (her ürün için)

```
README.md            # kanonik, İngilizce
README.tr.md         # opsiyonel, Türkçe paralel
LICENSE
CHANGELOG.md
CODE_OF_CONDUCT.md
CONTRIBUTING.md
SECURITY.md
.github/
├─ ISSUE_TEMPLATE/
└─ PULL_REQUEST_TEMPLATE.md
```

- [ ] `<product>` placeholder'ları gerçek adla değiştirildi
- [ ] LICENSE doldurulmuş (yıl + Codechu + lisans şartları)
- [ ] README hero brand lockup'ı referans veriyor
- [ ] Doküman iskeleti `docs/`'ta hazır

## 3. Identity / Branding (zorunlu)

- [ ] [BRAND-GUIDELINES.tr.md](BRAND-GUIDELINES.tr.md) okundu
- [ ] Palette + tipografi uygulandı
- [ ] Ürün-spesifik logo glyph tasarlandı (`mark.svg`)
- [ ] Lockup varyantları üretildi (horizontal, stacked)
- [ ] Knockout + monokrom mark variant'ları hazır
- [ ] Platform-native icon boyut seti hazır
- [ ] Social preview / OG image (1280×640 PNG)
- [ ] `docs/BRAND.md` ürün-spesifik kullanım kurallarını içerir

## 4. Filesystem layout (kullanıcı verisi varsa)

Hedef platforma uygun vendor namespace:
- **Linux/Unix**: XDG (`~/.config/codechu/<product>/`)
- **macOS**: `~/Library/Application Support/Codechu/<Product>/`
- **Windows**: `%APPDATA%\Codechu\<Product>\`
- **Mobil/web**: native sandbox + `com.codechu.<Product>` bundle ID

- [ ] Path sabitleri merkezi (tek config/paths modülü)
- [ ] Eski path layout varsa migration helper
- [ ] Hardcoded `/tmp/`, `%TEMP%` yok — hepsi platform-native runtime dizini altında

## 5. i18n (uygulanabildiğinde)

- [ ] i18n wrapper modülü
- [ ] Tüm user-facing string çeviri için flag'lendi
- [ ] Çıkarma tool yapılandırıldı (gettext / equivalent)
- [ ] Türkçe çeviri ship-included (mümkünse)
- [ ] User-visible language selector
- [ ] CI guard çevrilmemiş string için

## 6. CI / CD

- [ ] Lint / format
- [ ] Unit test + coverage threshold (≥%60)
- [ ] Static security scan (yıkıcı işlem veya user data varsa)
- [ ] Build artifact step
- [ ] Release workflow (tag push → artifact upload)
- [ ] Branch protection on main (olgun ürün)

## 7. Packaging (platform-bağımlı)

Hedef platforma uygun:

- [ ] **Linux**: `.desktop` + `.appdata.xml` + Debian/AppImage/Flatpak/Snap
- [ ] **macOS**: `.app` bundle + DMG / notarization / Mac App Store
- [ ] **Windows**: `.exe` / MSI / MSIX / Windows Store
- [ ] **Mobil**: TestFlight / App Store / Google Play / F-Droid
- [ ] **Web**: static host / Docker / Kubernetes manifest
- [ ] **Library**: language-registry publish (npm, PyPI, crates.io, Maven)

## 8. Pre-release smoke test

- [ ] Tüm testler geçiyor
- [ ] Lint temiz
- [ ] App / library hedef platformda başlıyor ve core flow çalışıyor
- [ ] Branding assetleri tüm gerekli boyutlarda render ediliyor
- [ ] Çeviriler compile / load oluyor
- [ ] Release workflow `v0.0.1-rc1` ile rehearsal edildi

## 9. İlk release (v0.1.0)

- [ ] CHANGELOG `[0.1.0]` entry
- [ ] Build metadata'da version bump
- [ ] `git tag -a v0.1.0` + push
- [ ] Release notes yayınlandı
- [ ] [codechu-org/README.tr.md](README.tr.md) ürün listesi güncellendi

## 10. Opsiyonel — dağıtım kanalları

Hedef platforma göre seç:

- [ ] GitHub Releases
- [ ] Platform-native store (Mac App Store, Microsoft Store, Snap Store, Flathub, F-Droid, Play, App Store)
- [ ] Package registry (PyPI, npm, crates.io, Homebrew, Chocolatey, Scoop, AUR)
- [ ] Distribution repo (Launchpad PPA, OBS, Copr)
- [ ] Launch announcement (Reddit, HN, Mastodon, blog)

---

**Not:** Bu checklist canlıdır. Yeni ürün başlatırken eksik/karışan adım
fark edersen PR ile güncelle. Platform default'larıyla otomatik gelen
adımlar atlanabilir — sadece **zorunlu** bölümler (1, 2, 3) müzakeresiz.
