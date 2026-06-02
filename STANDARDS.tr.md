# Codechu Proje Standartları

**Tüm** Codechu ürünleri için ortak disiplin kuralları. Programlama
dili, hedef platform, lisans modeli veya dağıtım kanalından bağımsız.

> İngilizce kanonik: [STANDARDS.md](STANDARDS.md). Bu Türkçe paralel —
> değişiklikler İngilizce kaynağa yapılır, Türkçe güncellenir.

## 0. Felsefe

> "Bir Codechu ürününü bilen kullanıcı, ikincisini de bildiğini hissetsin."

Kimlik, adlandırma, repo yapısı, marka, dokümantasyon konvansiyonları —
ürünler arası tahmin edilebilir. Teknoloji kararları (dil, runtime,
platform) ürün başına alınır.

## 1. Bu doküman neyi kısıtlar

| Kısıtlar | Kısıtla**maz** |
|---|---|
| Vendor kimliği (ad, namespace, app ID) | Ürün feature seti |
| Repo konvansiyonları (branch, hook, .gitattributes) | Programlama dili |
| Marka assetleri (palet, tipografi, logo grameri) | İşletim sistemi / platform |
| Dokümantasyon yapısı | Dağıtım kanalı |
| Public vs private repo disiplini | Lisans modeli (açık / kaynak-erişimli / kapalı) |
| Commit / versioning konvansiyonları | Build tooling |

Bir Codechu ürünü CLI, GUI app, library, servis, mobil app, web app
veya donanıma yakın firmware olabilir — aynı kimlik kuralları
uygulanır.

## 2. Kimlik (vendor + ürün / library)

### Ürünler (son-kullanıcı uygulamaları)

| Alan | Kalıp | Not |
|---|---|---|
| Vendor | `codechu` | sabit |
| Product slug | `kebab-case`, 1–2 sözcük | örn. `<product>` |
| Reverse-DNS App ID | `com.codechu.<ProductName>` (CamelCase) | bundle/app ID kullanan tüm platformlar |
| GitHub repo | `codechu/<product>` | ürün başına tek repo |
| Paket registry adı | `codechu-<product>` | dile özgü (PyPI, npm, crates.io...) |
| Kaynak modül | `<product_module>` (dil konvansiyonu) | Python için snake_case, JS için camelCase vb. |
| Binary / executable | `<product>` (kısa) **veya** `codechu-<product>` (vendor-prefix) | UX trade-off |

### Library'ler (yeniden kullanılabilir bileşenler)

Library'ler dile-özgü olabilir (bir algoritmanın Python wrapper'ı) veya
polyglot olabilir (aynı fikir birden çok dilde). Repo adı **dil
sonek**ini taşır, böylece birden çok implementasyon yan yana yaşar.

| Alan | Kalıp | Örnek |
|---|---|---|
| GitHub repo | `codechu/<purpose>-<lang>` | `codechu/events-py`, `codechu/treeviz-rs`, `codechu/xdg-go` |
| Paket registry adı | `codechu-<purpose>` | `codechu-events` (PyPI), `@codechu/events` (npm), `codechu_events` (crates.io) — registry zaten dile göre kapsamlanmış |
| Kaynak modül / import | `codechu_<purpose>` (dil konvansiyonu) | `import codechu_events` (Py), `use codechu_events::*` (Rust) |

**Neden repo adında dil soneki?** Çok dilli bir yayımcı bir noktada aynı
fikre Rust + Python + Go'da ihtiyaç duyacak (event bus, path soyutlaması,
telemetri). Sonek sayesinde `codechu/events-py` ve `codechu/events-rs`
kardeş repo olarak yaşayabilir, her biri kendi SemVer hattıyla.

**Kural:** Vendor namespace sistem-düzeyi tanımlayıcılarda **zorunlu**
(reverse-DNS app ID, repo URL, paket registry adları). Son-kullanıcı
tanımlayıcılarında (binary, dosya adı) kısa form OK — namespace
çakışması yoksa.

**Plugin family'ler**: bir library'nin temiz bir core'u var ama hepsi
birlikte gönderilmesi gerekmeyen variant data ya da adapter'ları varsa
(font'lar, locale paketleri, codec'ler), core paket + kardeş
`codechu-<thing>-<variant>` plugin paketleri olarak ayır. Her variant
kendi data'sını, lisansını, release döngüsünü taşır.

**Repo granülerliği — library başına repo mu, monorepo mu?** Belirleyici
soru paket sayısı değil, **release bağıdır (coupling)**:

| Paketler… | Repo şekli |
|---|---|
| bağımsız, à la carte gönderilir, her biri kendi SemVer döngüsü (çoğu Codechu lib'i — `cli-py`, `log-py`; core + opsiyonel locale paketleri) | **library başına bir repo** (varsayılan) |
| version-lock olur ve lockstep release olur — core artı onsuz **kullanılamayan** ve birlikte bump olan adapter'lar (ör. UI framework: agnostik core + DOM/native renderer'lar) | **tek monorepo, çok paket** (workspaces) |

Monorepo, React/Vue/Babel şeklidir: tek repo, çok yayınlanan paket, iç
bağımlılıklar paket-yöneticisinin workspace mekanizmasıyla çözülür
(`file:` link yok, lokal geliştirmede publish adımı yok). **Framework**
adıyla + dil suffix'iyle isimlendirilir — `codechu/ui-ts`, paketler
`core` / `web` / `native`, yayınlananlar `@codechu/ui` / `@codechu/ui-web`
/ `@codechu/ui-native`. Version-lock'lu bir aileyi **kardeş repolara
bölme**: bu, repolar arası publish-bump dansını yeniden yaratır
(polyrepo-sprawl anti-pattern'i) ve hiçbir izolasyon faydası vermez. Dış
tüketiciler yine **yayınlanmış** pakete bağımlı olur (sınır registry'dir
— git-dependency bir monorepo'nun alt-dizinini kuramaz).

## 3. Dosya sistemi yerleşimi (uygulanabildiğinde)

Diskte kullanıcı verisi tutan ürünler vendor namespace kullanmalı.

### Unix / Linux / macOS (XDG tarzı)
```
~/.config/codechu/<product>/         # config
~/.cache/codechu/<product>/          # cache (yeniden üretilebilir)
~/.local/share/codechu/<product>/    # kalıcı data
$XDG_RUNTIME_DIR/codechu/<product>/  # runtime (socket, pid, lock)
```

### Windows
```
%APPDATA%\Codechu\<Product>\         # config + kalıcı
%LOCALAPPDATA%\Codechu\<Product>\    # cache + makineye özgü
```

### macOS (XDG kullanmayan native uygulamalar için)
```
~/Library/Application Support/Codechu/<Product>/   # config + data
~/Library/Caches/Codechu/<Product>/                # cache
~/Library/Logs/Codechu/<Product>/                  # log
```

### Mobil / web / sandboxed
Her platformun native lokasyonu, `codechu.<Product>` bundle
identifier'ı ile namespace'lenir.

**Kural:** Tek bir dizin açıldığında (örn. Linux'ta
`~/.config/codechu/`) tüm Codechu ürünleri görünür. Multi-product
görünürlüğü tasarım hedefidir.

## 4. Branding (ürünler arası paylaşılan)

| Element | Kural |
|---|---|
| Renk paleti | Navy `#1d2939` + Gold `#e0a020` + Paper `#fafaf7` (ürün aksenti eklenebilir) |
| Display font | Ubuntu Sans (veya benzer karakterli, ürüne uygun alternatif) |
| Mono / kod font | JetBrains Mono (veya platform-native mono fallback) |
| Logo grameri | Tek-glyph mark + wordmark + lockup |
| Logo varyantları | `mark.svg`, `mark-knockout.svg`, `mark-mono.svg`, `wordmark.svg`, `lockup-horizontal.svg`, `lockup-stacked.svg` |

Detay: [BRAND-GUIDELINES.md](BRAND-GUIDELINES.md).

## 5. Versiyonlama + commit (dil-agnostik)

| Kural | Değer |
|---|---|
| Version şeması | [SemVer 2.0](https://semver.org/) |
| Commit formatı | [Conventional Commits](https://www.conventionalcommits.org/) |
| Release tag | `v<MAJOR>.<MINOR>.<PATCH>` |
| Pre-release | `-alpha.N`, `-beta.N`, `-rc.N` |
| Changelog | [Keep a Changelog](https://keepachangelog.com/) formatı |

## 6. Repo disiplini (her ürün)

| Madde | Kural |
|---|---|
| Default branch | `main` (`master` değil) |
| Branch protection (olgun ürün) | CI green + 1 review zorunlu |
| Pre-commit hook | önerilir — bkz. [hooks/pre-commit](hooks/pre-commit) |
| .gitattributes | text dosyalar için LF normalize |
| Commit signing | opsiyonel ama release tag'leri için önerilir |
| İki-katmanlı model | public ürün repoları + private internal / sec-advisories — bkz. §10 |

## 7. Dokümantasyon yapısı (her ürün repo'su)

Bu dosyalar beklenir; **içerik ürüne göre değişir**.

```
README.md                  # kanonik, İngilizce — **vitrin** (bkz. §7.1)
README.tr.md               # opsiyonel, Türkçe paralel
LICENSE                    # ürünün taşıdığı lisans
CHANGELOG.md               # Keep a Changelog formatı
CONTRIBUTING.md            # katkıcı onboarding (veya "kapalı katkı" notu)
SECURITY.md                # güvenlik açığı bildirimi (kapalı kaynak için de)
CODE_OF_CONDUCT.md         # Contributor Covenant v2.1 referansı (OSS) veya iç sürüm
docs/                      # detay dokümantasyon — manual, referans, recipe
assets/                    # banner, demo cast, diagram, ekran görüntüsü
.github/                   # GitHub-spesifik (template, workflow)
```

Branding, packaging veya i18n olan ürünler için ek:

```
docs/BRAND.md, docs/I18N.md, docs/VERSIONING.md, docs/PUBLISHING.md
packaging/                 # platform-spesifik kurulum artifact'leri
po/                        # gettext çevirileri (kullanılıyorsa)
```

### 7.1 README vitrin

Kök `README.md` bir **vitrin**dir, kullanım kılavuzu değil. Bir
ziyaretçinin PyPI'da, GitHub aramasında veya registry sayfasında
gördüğü ilk şey — son söz değil, ilk izlenim. Buna göre davran.

**İlkeler:**

- **Tek çarpıcı görsel** üstte — demo cast (asciinema / SVG / GIF),
  ekran görüntüsü veya temiz bir diyagram. Projenin ekran çıktısı
  varsa *göster*. Yoksa, veri akışı / mimari için bir Mermaid veya
  ASCII diyagram doyurucu bir alternatiftir. Düz metin duvarı kabul
  edilemez.
- **Tek hızlı örnek**, 5–15 satır, kurulumdan hemen sonra
  çalıştırılabilir olsun. Her varyantı sıralama.
- **Yetenek bullet'ları**, API tabloları değil. README *yapabildiklerini*
  listeler; `docs/API.md` *her public sembolü* listeler.
- **`docs/`'a yönlendir** — API referansı, recipe'ler, migration,
  mimari için. Her link tek satır açıklama ile gelsin.
- **Family tablosu** (kardeş Codechu paketleri) — ziyaretçi
  ekosistemi bulabilsin.
- **Lisans + credits** en altta.

**README'de açık yasaklar:**

- Tam API referans tabloları yok (`docs/API.md`'de).
- Çok-bölümlü örnek mutfağı yok (`docs/RECIPES.md`'de).
- Tasarım gerekçesi / mimari derin dalış yok (`docs/ARCHITECTURE.md`
  veya `DESIGN_PRINCIPLES.md`'de).
- Extension noktası / plug-in kataloğu yok.
- Migration rehberi yok (`docs/MIGRATION.md`'de).

**Yumuşak uzunluk hedefi:** badge'ler ve görsel sonrası gövde
yaklaşık **150 satır**'a sığsın. Daha uzunsa içerik `docs/`'a aittir.

**Görsel asset'ler** `assets/` altında:

```
assets/
  preview.svg          # README hero statik önizleme (veya banner.svg)
  preview.cast         # asciinema kaydı (opsiyonel)
  preview.gif          # .cast'ten türetilmiş animasyon (opsiyonel)
  diagram-<isim>.svg   # Mermaid render veya el-çizimi mimari
  screenshots/         # GUI ürünleri için
```

v0.1.0 itibarıyla README sadece-metin olamaz. Ya inline bir Mermaid
bloğu ya da `assets/` referansı şart. Proje-tipi rehberleri
(`project-type/LIBRARY.md`, `project-type/APPLICATION.md`) bu kuralı
tip-spesifik iskeletlerle yeniden ifade eder.

## 8. CI/CD baseline

Dil- ve platform-agnostik minimum:

| Kontrol | Uygulanır |
|---|---|
| Lint / format | her ürün, dile özgü tool (ruff/eslint/clippy/gofmt/vb.) |
| Unit test + coverage threshold | test'i olan her ürün; ≥%60 önerilir |
| Static security scan | kullanıcı verisi, secret veya yıkıcı işlem yapan ürünler |
| Translation sync (gettext / eşdeğer) | i18n yapan ürünler |
| Build artifact (binary, AppImage, MSI, IPA, vb.) | binary dağıtım yapan ürünler |
| Branch protection | olgun ürünler (post-v1.0 veya geniş kullanıcı tabanı) |

Her ürünün `.github/workflows/`'u bunları barındırır — tooling
değişebilir.

## 9. Lisans politikası (ürün-başına)

Lisans **ürün-düzeyi** karardır, organizasyon kuralı değil.
Bir Codechu ürünü şunlardan biri olabilir:

- **Açık kaynak permissive** (MIT, Apache 2.0, BSD) — community-built
- **Açık kaynak copyleft** (GPL, AGPL) — share-alike
- **Source-available** (örn. PolyForm, BUSL) — ticari kullanım kısıtlı
- **Proprietary** — kapalı kaynak, ticari ürün

Her ürünün LICENSE'ı şartlarını ilan eder. "Codechu default lisansı"
yoktur — seçim ürün stratejisine aittir.

## 10. Public vs private repo disiplini

Codechu iki-katmanlı repo modeli kullanır. Public repolar yayımlanmış,
düzenlenmiş içeriği tutar; private repolar strateji, taslak ve
yayımlanmamış işleri tutar.

### Katmanlar

```
PUBLIC                                  PRIVATE
─────────────────────────────────       ─────────────────────────────────
codechu/<product>          ✓            codechu/internal           🔒
  yayımlanmış kod + doküman               roadmap (Q1/Q2/yıllık)
  public CHANGELOG                        strateji, fiyatlandırma
  public roadmap (kaba)                   iç metrik, finans
  marketing assetleri                     yayımlanmamış tasarım / deney
                                          müşteri görüşmeleri (NDA)
codechu/codechu-org        ✓            codechu/sec-advisories     🔒
  bu repo (standartlar)                   yamasız güvenlik açıkları + disclosure
codechu/.github            ✓
  organizasyon-geneli profil + default'lar
```

### İçerik yerleşimi

| İçerik | Katman | Gerekçe |
|---|---|---|
| Yayımlanmış kod | PUBLIC | Zaten ship'lendi |
| Yayımlanmış dokümantasyon | PUBLIC | Kullanıcıya dönük |
| Brand assetleri, marketing | PUBLIC | Dışarıdan görünür |
| Public roadmap (kaba) | PUBLIC | Yön sinyali |
| Olgunlaşmamış tasarım taslakları | PRIVATE | Yayımdan önce olgunlaştır |
| Fiyatlandırma, finans, iş kararları | PRIVATE | Yalnız iç kullanım |
| Müşteri görüşmeleri (NDA) | PRIVATE | Sözleşmesel |
| Yamasız güvenlik açıkları | PRIVATE | Koordineli disclosure |
| Rakip analizi, iç strateji | PRIVATE | Gizli |
| Devam eden marketing A/B testleri | PRIVATE | Pre-launch parlatma izlenimi vermemek için |

### Senkron disiplini

- **Private → Public**: feature'lar private'da olgunlaşır, public'e tek
  temiz commit olarak iner. İterasyon gürültüsü private'da kalır.
- **Public → Private read**: maintainer'lar private notlardan public
  commit'e cross-link verebilir (issue link).
- **Risk: sızıntı.** Credential ve hassas anahtar kelime kontrolü için
  pre-push hook veya `git-secrets`.

### Maliyet / fayda

GitHub free plan sınırsız private repo sunar — bu disiplin bedavadır,
sadece zihinsel disiplin gerektirir.

Sektör pratiği: bağımsız stüdyoların çoğu benzer modeli kullanır.

## 11. Uluslararasılaştırma (opsiyonel)

### Ürünler

Ürünler (son-kullanıcı uygulamaları) i18n yapabilir veya yapmayabilir.
Yaparsa:

- Kanonik kaynak dili **İngilizce**'dir
- Çeviriler ürün repo'sunun çeviri dizininde yaşar (örn. `po/`,
  `locales/`, `i18n/`)
- Türkçe çeviri önerilir ama zorunlu değildir (çoğu Codechu projesi
  taşır çünkü katkıcılar Türkçe konuşur)
- Diğer diller community-driven

Her ürünün i18n workflow'u `docs/I18N.md`'de.

### Library'ler

`codechu/` altında yayımlanan library'ler **çıktılarını lokalize
etmez**.

- Exception mesajları, log mesajları ve developer-facing diagnostic'ler
  English, hard-coded yazılır. Bunlar UI değil, diagnostic'tir;
  grep'lenebilir, aratılabilir ve locale'lar arası taşınabilir
  olmalıdır.
- Library'ler kullanıcıya çıkan metin üretmez. Semantic data döner
  (marker'lar, enum'lar, count field'ları, type field'ları); render
  + çeviri tüketen uygulamanın işidir.
- Library'ler `gettext`, `.po` dosyası veya herhangi bir çeviri
  altyapısı içermez.

**İstisna:** bir library tüketen uygulamanın araya giremeyeceği bir
UI surface'i doğrudan render ediyorsa (CLI help generator, form
widget, terminal scaffold), iki seçeneği vardır: (a) belgelenmiş
locale-resolution politikasıyla içeride çevirir, veya (b) dependency
injection ile translator callback kabul eder. **(b) tercih edilir**
— library loose coupling'i korur, host uygulama kendi i18n stack'i
üzerinde kontrolü elinde tutar.

Bu pratik sektörle uyumlu: `requests`, `pydantic`, `SQLAlchemy`,
`pytest`, `structlog`, `tokio`, `serde`, `lodash` — hiçbiri library
seviyesinde i18n yapmaz. Python stdlib'in `argparse`/`optparse`'i ve
framework'ler (Django, Flask, Click) istisnanın altına girer — son
rendering surface'ini onlar sahiplenir.

Referanslar: Microsoft .NET "Best practices for exceptions",
jbe2277/waf wiki'sindeki "Should exception messages be localized?"
tartışması, API-craft mailing list uzlaşısı.

## 12. Canlı doküman

Yeni bir ürün edge case ortaya çıkardığında bu dosya PR ile güncellenir.
`STANDARDS.md`'ye dokunan bir PR başlığında **breaking change**
etiketlenir — tüm ürün repolarına yansıyabilir.

## 13. Aktif repolar

### Organizasyon geneli
- [codechu/codechu-org](https://github.com/codechu/codechu-org) — bu repo (PUBLIC, standartlar)
- `codechu/internal` 🔒 — strateji, taslak, finans (PRIVATE)

### Ürünler
Güncel ürün listesi için [README.md](README.md).

### Yeniden kullanılabilir library'ler (Python)
- [codechu/events-py](https://github.com/codechu/events-py) — `codechu-events` — çok-kanallı, thread-safe event bus
- [codechu/treeviz-py](https://github.com/codechu/treeviz-py) — `codechu-treeviz` — squarified treemap + sunburst layout
- [codechu/xdg-py](https://github.com/codechu/xdg-py) — `codechu-xdg` — vendor-namespace'li XDG path'leri
