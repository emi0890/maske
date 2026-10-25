Türkçe · [English](README.en.md)

# maske 🔒

Türkçe metinlerde kişisel veri (PII) tespiti — açık kaynak, KVKK odaklı, iki katmanlı.

[![Lisans: MIT](https://img.shields.io/badge/license-MIT-green)](LICENSE) [![Model: maske-berturk-baseline](https://img.shields.io/badge/model-maske--berturk--baseline-blue)](https://huggingface.co/emi08/maske-berturk-baseline)

## Neden var?

Türkçe için PII tespiti yapan mevcut araçların çoğu yalnızca sentetik veriyle değerlendirilmiş durumda ve hiçbiri deterministik, sıfır-yanlış-pozitif bir kural katmanı sunmuyor. KVKK kapsamına giren verilerle çalışırken "muhtemelen yakalar" yeterli değil. maske'nin cevabı iki katman:

1. **Kural katmanı** — TCKN, IBAN, telefon, plaka, e-posta. Kontrol toplamı (checksum) ve biçim doğrulaması: bu alanlarda model yok, halüsinasyon yok, hedef sıfır yanlış pozitif.
2. **NER katmanı** — ek almış isimler (Ayşe'ye, Ahmet'ten) ve adresler gibi kuralların yakalayamayacağı esnek kısımlar için fine-tune edilmiş BERTurk.

Ve ayırt edici taahhüt: **her model, gerçek Türkçe metninden oluşturulmuş insan doğrulamalı altın test setiyle raporlanacak** — sentetik skorlar değil.

## Değerlendirme

WikiANN-tr test split'inin tamamı üzerinde (10.000 cümle) seqeval varlık düzeyi F1.
Tüm satırlar birebir aynı ince ayar tarifini kullanır (lr 2e-5, 3 epoch, batch 16,
max_length 128, aynı tohum) — satırlar arasında yalnızca taban checkpoint değişir.

| Model | Taban | Param | Precision | Recall | F1 |
|---|---|---|---|---|---|
| [maske-berturk-baseline](https://huggingface.co/emi08/maske-berturk-baseline) | dbmdz/bert-base-turkish-128k-cased | 110M | 0.923 | 0.934 | **0.929** |
| [maske-modernbert-tr-bakeoff](https://huggingface.co/emi08/maske-modernbert-tr-bakeoff) | ytu-ce-cosmos/modernbert-tr-base-1k | 150M | 0.923 | 0.935 | **0.929** |

**Bulgu:** 2020 tarihli bir BERT ile 144 milyar Türkçe token üzerinde sıfırdan
eğitilmiş 2026 tarihli bir ModernBERT, aynı tarif altında tamamen aynı varlık
düzeyi F1'e ulaşıyor; aynı ModernBERT tabanının farklı bir Türkçe NER veri
kümesindeki ([Küçük & Küçük 2016](https://huggingface.co/akdeniz27/modernbert-tr-base-ner))
üçüncü taraf ince ayarı da aynı tabloyu gösteriyor. WikiANN-tr taban ölçekte
doygun görünüyor; maske'nin kalite yol haritası bu yüzden genel liderlik tablosu
farkları yerine KVKK-PII dikeyini hedefliyor (alan altın seti + sağlama toplamı
kural katmanı).

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
