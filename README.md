Türkçe · [English](README.en.md)

# maske 🔒

Türkçe metinlerde kişisel veri (PII) tespiti — açık kaynak, KVKK odaklı, iki katmanlı.

[![Lisans: MIT](https://img.shields.io/badge/license-MIT-green)](LICENSE) [![Model: maske-berturk-baseline](https://img.shields.io/badge/model-maske--berturk--baseline-blue)](https://huggingface.co/emi08/maske-berturk-baseline)

## Neden var?

Türkçe için PII tespiti yapan mevcut araçların çoğu yalnızca sentetik veriyle değerlendirilmiş durumda ve hiçbiri deterministik, sıfır-yanlış-pozitif bir kural katmanı sunmuyor. KVKK kapsamına giren verilerle çalışırken "muhtemelen yakalar" yeterli değil. maske'nin cevabı iki katman:

1. **Kural katmanı** — TCKN, IBAN, telefon, plaka, e-posta. Kontrol toplamı (checksum) ve biçim doğrulaması: bu alanlarda model yok, halüsinasyon yok, hedef sıfır yanlış pozitif.
2. **NER katmanı** — ek almış isimler (Ayşe'ye, Ahmet'ten) ve adresler gibi kuralların yakalayamayacağı esnek kısımlar için fine-tune edilmiş BERTurk.

Ve ayırt edici taahhüt: **her model, gerçek Türkçe metninden oluşturulmuş insan doğrulamalı altın test setiyle raporlanacak** — sentetik skorlar değil.

## Sonuçlar

maske modelleri ile üç Türkçe NER modeli, aynı değerlendirme betiği ile
WikiANN-tr test split'inin tamamında (10.000 cümle) ölçüldü:

| Model | Precision | Recall | F1 |
|---|---|---|---|
| **maske-modernbert-tr-bakeoff** — [kart](https://huggingface.co/emi08/maske-modernbert-tr-bakeoff) | 0.923 | 0.935 | **0.929** |
| **maske-berturk-baseline** — [kart](https://huggingface.co/emi08/maske-berturk-baseline) | 0.923 | 0.934 | **0.929** |
| savasy/bert-base-turkish-ner-cased | 0.919 | 0.930 | 0.925 |
| Werea-co/Werea-TR-NER | 0.907 | 0.917 | 0.912 |
| akdeniz27/bert-base-turkish-cased-ner | 0.699 | 0.649 | 0.673 |

- Beş model de aynı test split'i ve aynı betikle ölçüldü; bu sonuçlar başka veri kümelerinden gelen skorlarla karşılaştırılamaz.
- İki maske satırı birebir aynı ince ayar tarifini kullanır ve yalnızca tabanda farklıdır: 2020 tarihli BERTurk (128k) ile 2026 tarihli ModernBERT-TR (150M, 144 milyar Türkçe token). İkisi 0.929'da berabere — WikiANN-tr taban ölçekte doygun; aynı tabanın üçüncü taraf ince ayarı ([akdeniz27, Küçük 2016](https://huggingface.co/akdeniz27/modernbert-tr-base-ner)) da bunu doğruluyor. Gelecekteki eğitim bu yüzden yeni tabanlara değil, alan verisine yönelecek.
- savasy ile fark (0.004 F1) eğitim rastgeleliği gürültüsü içinde; Werea ile fark (0.017 F1) Werea'nın kendi kartında bildirdiği 0.917 ile uyumlu. Temel fark taban modelde (32k ve 128k kelime dağarcığı).
- akdeniz27, WikiANN değil Küçük (2016) veri kümesiyle eğitildi; düşük skoru model kalitesinden değil, eğitim/test verisi uyuşmazlığından kaynaklanır.
- WikiANN otomatik etiketli ve haber tarzıdır; bu skorlar gerçek dünya metni için tavan değerdir. Gerçek performans 2. aşama altın test seti ile ölçülecek.

Son ölçüm: Ekim 2026 · Yöntem: [baseline kartı](https://huggingface.co/emi08/maske-berturk-baseline) · [bake-off kartı](https://huggingface.co/emi08/maske-modernbert-tr-bakeoff)

## Durum

| Aşama | Çıktı | Durum |
|---|---|---|
| 1 — Baseline NER | [emi08/maske-berturk-baseline](https://huggingface.co/emi08/maske-berturk-baseline) (PER/ORG/LOC) | ✅ |
| 2 — Altın test seti | gerçek metinden insan doğrulamalı değerlendirme seti | 🔜 |
| 3 — Kural katmanı | TCKN/IBAN/telefon doğrulayıcıları | 🔜 |
| 4 — maske paketi | `pip install maske` — birleşik tespit + maskeleme | 🔜 |

## Hızlı başlangıç (aşama 1)

~~~python
from transformers import pipeline

ner = pipeline(
    "token-classification",
    model="emi08/maske-berturk-baseline",
    aggregation_strategy="simple",
)
print(ner("Ayşe'ye 500 TL gönderdim, makbuzu Ahmet Bey'in e-postasına yazdım."))
~~~

⚠️ Deneysel baseline — üretim kullanımı için hazır değildir. Ayrıntılar: [model kartı](https://huggingface.co/emi08/maske-berturk-baseline).

## Depo yapısı (plan)

~~~text
maske/
├── README.md                      # buradasınız (Türkçe)
├── README.en.md                   # English
├── LICENSE                        # MIT
├── data/                          # aşama 2: altın test seti buraya gelecek
└── maske/                         # aşama 3-4: kural katmanı ve paket kodu
~~~

## Katkı

Hatalı tespit örnekleri bulursanız issue açın. Tek kuralımız: **gerçek kişisel veri asla bu depoya commit edilmez** — örneklerde uydurma isim/numara kullanın.

## Lisans

[MIT](LICENSE)
