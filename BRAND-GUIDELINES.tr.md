# Codechu Marka Rehberi

Tüm Codechu ürünleri için ortak görsel kimlik kuralları. Dil, platform,
lisanstan bağımsız. Her ürün kendi glyph'ini tasarlar ama **paylaşılan
palet + tipografi** kullanır → ürün ailesi tutarlı durur.

> İngilizce kanonik: [BRAND-GUIDELINES.md](BRAND-GUIDELINES.md).
> Bu Türkçe paralel.

## 1. Renk paleti (paylaşılan)

### Primary — Navy

| Token | Hex | Kullanım |
|---|---|---|
| `--brand-navy-light` | `#34507a` | Gradient highlight (üst-sol) |
| `--brand-navy`       | `#1d2939` | Birincil marka rengi |
| `--brand-navy-dark`  | `#0f1928` | Gradient gölgesi (alt-sağ) |

### Accent — Gold

| Token | Hex | Kullanım |
|---|---|---|
| `--brand-gold`       | `#e0a020` | Tek-element aksanı, link highlight, "marka kıvılcımı" |

### Neutrals

| Token | Hex | Kullanım |
|---|---|---|
| `--brand-paper`      | `#fafaf7` | Off-white background |
| `--brand-ink`        | `#1d2939` | Body text |

### Ürün-spesifik aksent (opsiyonel, 1 ekstra renk)

Her ürün **bir** ekstra aksent ekleyebilir (palet küçük kalır). Ekleme
ürünün domain'ini sinyaller.

### Oran kuralı

- Navy ailesi: kompozisyonun **%80'i**
- Paper (off-white): **%15**
- Gold (+ ürün aksenti varsa): **%5** — küçük, kasıtlı anlar

Gold geniş alan dolgusu değil — sadece dikkat kıvılcımı.

## 2. Tipografi (paylaşılan)

### Primary — Ubuntu Sans (veya benzer karakterli)

Humanist sans-serif. Hedef platformda uygun değilse benzer karakterli
(açık, dostça, nötr) bir typeface seç. Örnek fallback chain:

```css
font-family: "Ubuntu Sans", "Inter", system-ui, -apple-system, sans-serif;
```

| Element | Size | Weight | Letter-spacing |
|---|---|---|---|
| Hero / display | 92px | 700 | -2 |
| Section heading | 48px | 600 | -1.5 |
| Body | 16–18px | 400 | 0 |
| Caption | 14px | 400 | 0.5 |

### Secondary — JetBrains Mono (kod, terminal, CLI banner)

```css
font-family: "JetBrains Mono", "DejaVu Sans Mono", "Cascadia Code", monospace;
```

## 3. Logo sistemi (ürün başına)

### Glyph felsefesi

Her ürün **tek-glyph mark** tasarlar. Glyph **anlam taşımak zorunda
değil** — sadece marka imzası. Anlamı wordmark söyler.
Notion / Linear / Vercel paterni.

### Zorunlu varyantlar

| Dosya | Kullanım |
|---|---|
| `assets/logo/mark.svg` | Tam renkli mark |
| `assets/logo/mark-knockout.svg` | Koyu zeminde için ters (inverted) |
| `assets/logo/mark-mono.svg` | Tek renk, `currentColor` (sistem tray, B&W print) |
| `assets/logo/wordmark.svg` | Sadece tipografi |
| `assets/logo/lockup-horizontal.svg` | Mark + wordmark yan yana |
| `assets/logo/lockup-stacked.svg` | Mark üstte, wordmark altta |

### Platform-spesifik app icon (uygulanabildiğinde)

| Dosya | Format |
|---|---|
| Master | Vector (SVG), ürünün native viewBox'ı |
| Mono / symbolic | 16px monokrom sistem tray için |
| Small-size simplified | Gradient/detay 16-32px'te kaybolursa ayrı sade dosya |
| Platform-native sizes | Her hedefin gerekli set'i master'dan rasterize |

## 4. Logo kullanım kuralları (tüm ürünler)

### Safe zone
Mark etrafında, mark'ın kendi yarıçapı kadar boş alan.

### Min size
- **Mark only:** 16px
- **Horizontal lockup:** 120px geniş min
- **Stacked lockup:** 100px geniş min

### Yapma
- ❌ Rotate / skew / mirror
- ❌ Palet değiştirme (altın yerine yeşil hub vs.)
- ❌ Master gradient'i flat'leme (mono varyantı bunun için)
- ❌ Wordmark'ı başka fontla yazma
- ❌ Mark içine ekstra element (tagline, version, badge)
- ❌ Mark'a ekstra çerçeve ekleme

## 5. Tone of voice (paylaşılan)

| Karakter | Açıklama |
|---|---|
| **Calm + precise** | Factual, no marketing fluff. "Revolutionary" / "best-in-class" yasak. |
| **Process-aware** | Ürünün ne yaptığını **ve** ne yapmadığını söyle. |
| **Platform-respecting** | Platform-native konvansiyonları kullan, evrensel olmayan tool'u öyle pazarlama. |
| **Bilingual-ready** | İngilizce kanonik, Türkçe paralel mümkünse. Diğer diller welcome. |

## 6. Asset format kuralları

- **SVG kaynak**: tüm logo, renkler `<stop>` içinde literal hex (CSS variable risk: theme override)
- **PNG raster** (release prep): 16/24/32/48/64/128/256/512/1024 standart set
- **Open Graph image**: 1280×640 PNG
- **Favicon**: ICO (16+32+48 multi-res), opsiyonel

## 7. Telif

Brand assetleri © Codechu. Ürün kod katkıları MIT (veya ürün lisansı).

İzin verilen:
- ✅ Codechu ürünleri içinde
- ✅ Ürünü tanıtan makale / video / blog
- ❌ Türev marka oluşturma (sahte "Codechu")
- ❌ Ürünü fork edip Codechu brand assetlerini tutma (fork rebrand etmeli)

## 8. Canlı doküman

Yeni ürün edge case ortaya çıkarırsa PR. Major değişiklik (palet kayması,
typography swap) breaking — ürün repoları arası koordinasyon gerek.
