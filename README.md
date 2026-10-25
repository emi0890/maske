Türkçe · [English](README.en.md)

# maske 🔒

Türkçe metinlerde kişisel veri (PII) tespiti — açık kaynak, KVKK odaklı, iki katmanlı.

[![Lisans: MIT](https://img.shields.io/badge/license-MIT-green)](LICENSE) [![Model: maske-berturk-baseline](https://img.shields.io/badge/model-maske--berturk--baseline-blue)](https://huggingface.co/emi08/maske-berturk-baseline)

## Neden var?

Türkçe için PII tespiti yapan mevcut araçların çoğu yalnızca sentetik veriyle değerlendirilmiş durumda ve hiçbiri deterministik, sıfır-yanlış-pozitif bir kural katmanı sunmuyor. KVKK kapsamına giren verilerle çalışırken "muhtemelen yakalar" yeterli değil. maske'nin cevabı iki katman:

1. **Kural katmanı** — TCKN, IBAN, telefon, plaka, e-posta. Kontrol toplamı (checksum) ve biçim doğrulaması: bu alanlarda model yok, halüsinasyon yok, hedef sıfır yanlış pozitif.
2. **NER katmanı** — ek almış isimler (Ayşe'ye, Ahmet'ten) ve adresler gibi kuralların yakalayamayacağı esnek kısımlar için fine-tune edilmiş BERTurk.

Ve ayırt edici taahhüt: **her model, gerçek Türkçe metninden oluşturulmuş insan doğrulamalı altın test setiyle raporlanacak** — sentetik skorlar değil.

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
### Karşılaştırma

Aynı değerlendirme betiği, WikiANN-tr test bölümünün tamamı (10.000 cümle) üzerinde; etiket düzeyinde seqeval precision/recall/F1:

| Model | Precision | Recall | F1 |
|---|---|---|---|
| **emi08/maske-berturk-baseline** | 0.923 | 0.934 | **0.929** |
| savasy/bert-base-turkish-ner-cased | 0.919 | 0.930 | 0.925 |
| Werea-co/Werea-TR-NER | 0.907 | 0.917 | 0.912 |
| akdeniz27/bert-base-turkish-cased-ner | 0.699 | 0.649 | 0.673 |

Notlar:

- Dört model de aynı betik ve aynı test bölümüyle ölçülmüştür; farklı veri kümeleri üzerinde yapılan değerlendirmelerle karşılaştırılamaz.
- savasy ile arasındaki fark (0.004 F1) eğitim rastgeleliği sınırındadır; kesin bir üstünlük iddiası değildir.
- Werea ile arasındaki fark (0.017 F1) bu sınırın üzerindedir ve Werea'nın kendi kartında bildirdiği 0.917 F1 ile uyumludur. Başlıca fark taban modeldir: Werea 32 binlik vocab'lı dbmdz/bert-base-turkish-cased tabanını, maske ise 128 binlik vocab'lı dbmdz/bert-base-turkish-128k-cased tabanını kullanır.
- akdeniz27/bert-base-turkish-cased-ner WikiANN ile değil Küçük (2016) Türkçe NER veri kümesiyle eğitildiğinden, WikiANN-tr üzerindeki düşük skoru model kalitesinden çok eğitim/test verisi uyuşmazlığını yansıtır.
- WikiANN otomatik etiketli, haber tarzı bir veri kümesi olduğundan bu skorlar gerçek metinlerde üst sınırdır; gerçek metin performansı aşama 2'deki altın test setiyle ölçülecektir.

## Katkı

Hatalı tespit örnekleri bulursanız issue açın. Tek kuralımız: **gerçek kişisel veri asla bu depoya commit edilmez** — örneklerde uydurma isim/numara kullanın.

## Lisans

[MIT](LICENSE)
