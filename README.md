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

Faz 1 taban çizgisi (`emi08/maske-berturk-baseline`), WikiANN-tr test bölümünün tamamında (10.000 cümle) aynı betikle dört Türkçe NER modeliyle karşılaştırıldı:

| Model | Precision | Recall | F1 |
|---|---|---|---|
| **maske (bu proje)** | 0.923 | 0.934 | **0.929** |
| savasy/bert-base-turkish-ner-cased | 0.919 | 0.930 | 0.925 |
| Werea-co/Werea-TR-NER | 0.907 | 0.917 | 0.912 |
| akdeniz27/bert-base-turkish-cased-ner | 0.699 | 0.649 | 0.673 |

- Dört model de aynı test bölümü ve aynı değerlendirme betiğiyle ölçüldü; farklı veri kümeleriyle yapılan skorlarla karşılaştırılamaz.
- savasy ile fark (0.004 F1) eğitim rastgeleliği sınırındadır; Werea ile fark (0.017 F1) Werea'nın kendi bildirdiği 0.917 F1 ile uyumludur. Başlıca fark taban modeldir (32 binlik vs 128 binlik vocab).
- akdeniz27 WikiANN ile değil Küçük (2016) veri kümesiyle eğitildi; düşük skoru model kalitesinden çok eğitim/test uyuşmazlığını yansıtır.
- WikiANN otomatik etiketli, haber tarzı bir kümedir; bu skorlar gerçek metinlerde üst sınırdır. Gerçek performans Faz 2'deki altın test setiyle ölçülecek.

Son ölçüm: Ekim 2026 · Yöntem ayrıntıları: [model kartı](https://huggingface.co/emi08/maske-berturk-baseline)

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
