# Codechu Proje Standartları

Tüm Codechu ürünleri için **ortak** disiplin kuralları. Programlama
dili, hedef platform, lisans modeli veya dağıtım kanalından bağımsız.

> İngilizce kanonik: [STANDARDS.md](STANDARDS.md). Bu Türkçe paralel —
> değişiklikler İngilizce kaynağa yapılır, Türkçe güncellenir.

## 0. Felsefe

> "Bir Codechu ürününü bilen kullanıcı, ikincisini de bildiğini hissetsin."

Kimlik, adlandırma, repo yapısı, marka, dokümantasyon konvansiyonları —
ürünler arası tahmin edilebilir. Teknoloji kararları (dil, runtime,
platform) ürün başına alınır.

## 1. Bu doküman neyi kısıtlar

| Kısıtlar | Kısıtlamaz |
|---|---|
| Vendor kimliği (ad, namespace, app ID) | Ürün feature seti |
| Repo konvansiyonları (branch, hook, .gitattributes) | Programlama dili |
| Marka assetleri (palet, tipografi, logo grameri) | İşletim sistemi / platform |
| Dokümantasyon yapısı | Dağıtım kanalı |
| Public/private repo disiplini | Lisans modeli (açık / kaynak-erişimli / kapalı) |
| Commit / versioning konvansiyonları | Build tooling |

Bir Codechu ürünü CLI, GUI app, library, servis, mobil app, web app
veya firmware olabilir — aynı kimlik kuralları uygulanır.

## 2. Kimlik (vendor + product)

| Alan | Kalıp | Not |
|---|---|---|
| Vendor | `codechu` | sabit |
| Product slug | `kebab-case`, 1–2 sözcük | örn. `<product>` |
| Reverse-DNS App ID | `com.codechu.<ProductName>` (CamelCase) | bundle/app ID kullanan tüm platformlar |
| GitHub repo | `codechu/<product>` | ürün başına tek repo |
| Paket registry adı | `codechu-<product>` | npm, PyPI, crates.io, Maven artifact-id |
| Kaynak modül | dil konvansiyonu (snake_case, kebab-case, camelCase) | dil kuralı önceliklidir |
| Binary | `<product>` (kısa) **veya** `codechu-<product>` (vendor-prefix) | kısa default |

**Kural:** Vendor namespace **zorunlu** sistem-düzeyi tanımlayıcılarda
(reverse-DNS, repo URL, paket registry). Kısa form son-kullanıcı
tanımlayıcılarında OK — namespace çakışması yoksa.

## 3. Dosya sistemi yerleşimi (uygulanabildiğinde)

### Unix / Linux / macOS (XDG)
```
~/.config/codechu/<product>/         # config
~/.cache/codechu/<product>/          # cache (regenerate)
~/.local/share/codechu/<product>/    # kalıcı data
$XDG_RUNTIME_DIR/codechu/<product>/  # runtime
```

### Windows
```
%APPDATA%\Codechu\<Product>\         # config + kalıcı
%LOCALAPPDATA%\Codechu\<Product>\    # cache
```

### macOS native
```
~/Library/Application Support/Codechu/<Product>/
~/Library/Caches/Codechu/<Product>/
~/Library/Logs/Codechu/<Product>/
```

### Mobil / web / sandboxed
Platform-native lokasyon + `codechu.<Product>` bundle ID.

**Kural:** Tek bir dizin açıldığında tüm Codechu ürünleri görünür.
Multi-product görünürlüğü tasarım hedefi.

## 4. Branding (paylaşılan)

| Element | Kural |
|---|---|
| Renk paleti | Navy `#1d2939` + Gold `#e0a020` + Paper `#fafaf7` |
| Display font | Ubuntu Sans (veya benzer karakterli alternatif) |
| Mono font | JetBrains Mono (veya platform-native mono) |
| Logo grameri | Tek-glyph mark + wordmark + lockup |

Detay: [BRAND-GUIDELINES.tr.md](BRAND-GUIDELINES.tr.md).

## 5. Versiyonlama + commit

| Kural | Değer |
|---|---|
| Version şeması | [SemVer 2.0](https://semver.org/) |
| Commit formatı | [Conventional Commits](https://www.conventionalcommits.org/) |
| Release tag | `v<MAJOR>.<MINOR>.<PATCH>` |
| Pre-release | `-alpha.N`, `-beta.N`, `-rc.N` |
| Changelog | [Keep a Changelog](https://keepachangelog.com/) |

## 6. Repo disiplini

| Madde | Kural |
|---|---|
| Default branch | `main` |
| Branch protection (olgun ürün) | CI green + 1 review |
| Pre-commit hook | önerilir — [hooks/pre-commit](hooks/pre-commit) |
| .gitattributes | LF normalize |
| Tag imzalama | release'ler için önerilir |
| İki-katmanlı repo | public + private — bkz. §10 |

## 7. Dokümantasyon yapısı

Bu dosyalar beklenir; içerikleri ürüne göre değişir.

```
README.md, README.tr.md (opsiyonel)
LICENSE, CHANGELOG.md
CONTRIBUTING.md, SECURITY.md, CODE_OF_CONDUCT.md
docs/                # detay
.github/             # GitHub-spesifik
```

Branding / packaging / i18n olan ürünler için ek:
```
docs/BRAND.md, docs/I18N.md, docs/VERSIONING.md, docs/PUBLISHING.md
packaging/           # platform-spesifik
po/                  # gettext (kullanılıyorsa)
assets/              # logo, icon, screenshot
```

## 8. CI/CD baseline (dil-agnostik)

| Kontrol | Uygulanır |
|---|---|
| Lint / format | dile özgü tool |
| Unit test + coverage | ≥%60 önerilir |
| Static security scan | yıkıcı işlem yapan ürünler |
| Translation sync | i18n yapan ürünler |
| Build artifact | binary dağıtım yapan ürünler |
| Branch protection | olgun ürünler |

## 9. Lisans politikası (ürün-başına)

Lisans ürün-düzeyi karar:
- **Açık kaynak permissive** (MIT, Apache 2.0, BSD)
- **Açık kaynak copyleft** (GPL, AGPL)
- **Source-available** (PolyForm, BUSL)
- **Proprietary** — kapalı kaynak, ticari

Codechu default lisansı yoktur — seçim ürün stratejisine ait.

## 10. Public vs private disiplin

İki-katmanlı:

```
PUBLIC                                  PRIVATE
─────────────────────────────────       ─────────────────────────────────
codechu/<product>          ✓            codechu/internal           🔒
codechu/.github            ✓            codechu/sec-advisories     🔒
codechu/codechu-org        ✓
codechu/codechu-template   ✓ (planlı)
```

### İçerik

| İçerik | Katman |
|---|---|
| Yayınlanmış kod / doküman | PUBLIC |
| Brand asset, marketing | PUBLIC |
| Coarse roadmap | PUBLIC |
| Unstable design draft | PRIVATE |
| Strateji, finans, fiyat | PRIVATE |
| NDA görüşmeleri | PRIVATE |
| Yamasız güvenlik açığı | PRIVATE |

**Senkron:** Private → Public, feature olgunlaşınca tek temiz commit.
Pre-push hook ile credential / sensitive-keyword guard.

## 11. Uluslararasılaştırma (opsiyonel)

Kanonik kaynak dili **İngilizce**. Çeviriler ürün repo'sunda. Türkçe
çeviri önerilir, zorunlu değil. Diğer diller community-driven.

## 12. Canlı doküman

Yeni ürün edge case ortaya çıkarırsa PR ile güncelle. STANDARDS değişimi
**breaking change** etiketi — diğer ürünlere ripple olur.

## 13. Aktif repolar

Liste için [README.tr.md](README.tr.md).
