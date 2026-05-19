# Codechu

Yazılım ürünleri yayımlayan bağımsız bir publisher.

> İngilizce kanonik: [README.md](README.md). Bu Türkçe paralel.

## Ürünler

| Ürün | Açıklama | Repo / Kanal | Durum |
|---|---|---|:---:|
| **Disk Cleaner** | Linux için süreç-farkındalıklı güvenli disk temizleyici | [codechu/disk-cleaner](https://github.com/codechu/disk-cleaner) | 0.1 beta |

Lisans modeli, hedef platform ve dağıtım kanalı ürüne göre değişir.

## Yeniden kullanılabilir library'ler

Codechu ürünlerinden bağımsız library olarak çıkarılmış bileşenler — Codechu
dışı projeler de dahil her projede kullanılabilir.

| Library | Dil | PyPI / dist | Repo |
|---|---|---|---|
| **codechu-events** — Çok-kanallı, thread-safe event bus | Python | `codechu-events` | [events-py](https://github.com/codechu/events-py) |
| **codechu-treeviz** — Squarified treemap + sunburst layout | Python | `codechu-treeviz` | [treeviz-py](https://github.com/codechu/treeviz-py) |
| **codechu-xdg** — Vendor-namespace'li XDG path'leri (Linux) | Python | `codechu-xdg` | [xdg-py](https://github.com/codechu/xdg-py) |

Repo adları dil soneki taşır (`-py`, `-rs`, `-go`, vb.) — böylece aynı
fikrin polyglot implementasyonları yan yana yaşayabilir.

## Bu repo nedir

**Tüm Codechu ürünleri için ortak içerik** — programlama dili,
işletim sistemi, lisans modeli veya dağıtım kanalından bağımsız.

Bu repo şunları içerir:

- Vendor kimlik kuralları (adlandırma, namespace, marka)
- Repository disiplini (public vs private, git config, hooks)
- Yeni ürün başlatma prosedürü
- Marka rehberi (palet, tipografi, logo grameri)

Bu repo şunları içermez:

- Ürün-spesifik kod veya doküman (her ürün kendi repo'sunda)
- Teknoloji stack kararları (her ürün kendi seçer)
- Platform kararları (her ürün kendi hedefini seçer)
- Lisans kararları (her ürün kendi LICENSE'ını ilan eder)

## İçerik

| Dosya | Amaç |
|---|---|
| [STANDARDS.md](STANDARDS.md) | Codechu ürünleri arasında uygulanan disiplin kuralları (İngilizce) |
| [STANDARDS.tr.md](STANDARDS.tr.md) | Türkçe paralel |
| [TEMPLATE-CHECKLIST.md](TEMPLATE-CHECKLIST.md) | Yeni ürün başlatma prosedürü (İngilizce) |
| [TEMPLATE-CHECKLIST.tr.md](TEMPLATE-CHECKLIST.tr.md) | Türkçe paralel |
| [BRAND-GUIDELINES.md](BRAND-GUIDELINES.md) | Paylaşılan palet / tipografi / logo grameri |
| [BRAND-GUIDELINES.tr.md](BRAND-GUIDELINES.tr.md) | Türkçe paralel |
| [hooks/](hooks/) | Opsiyonel paylaşılan git hook'ları |

## Dil politikası

Kanonik dokümanlar **İngilizce** (uluslararası erişim, sektör normu).
Türkçe versiyonlar `*.tr.md` dosyalarında paralel olarak yaşar ve
İngilizce orijinalleriyle senkron tutulur.

## İletişim

- Web: https://codechu.com _(yapım aşamasında)_
- E-posta: info@codechu.com
- GitHub: [@codechu](https://github.com/codechu)

## Lisans

Bu repodaki dokümantasyon: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).

Her ürün kendi LICENSE'ını taşır — açık kaynak, kaynak-erişimli veya
proprietary olabilir (ürün stratejisine göre).
