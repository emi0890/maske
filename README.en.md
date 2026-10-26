# maske 🇹🇷 — Turkish KVKK PII-Masking NER

[🇬🇧 English](README_EN.md) · [🇹🇷 Türkçe](README.md)

[![Hugging Face · maske](https://img.shields.io/badge/%F0%9F%A4%97%20maske-emi08%2Fmaske-yellow)](https://huggingface.co/emi08/maske)
[![Hugging Face · bakeoff](https://img.shields.io/badge/%F0%9F%A4%97%20bakeoff-emi08%2Fmaske--modernbert--tr--bakeoff-yellow)](https://huggingface.co/emi08/maske-modernbert-tr-bakeoff)
[![Hugging Face · baseline](https://img.shields.io/badge/%F0%9F%A4%97%20baseline-emi08%2Fmaske--berturk--baseline-yellow)](https://huggingface.co/emi08/maske-berturk-baseline)

## Overview

`maske` is a family of token-classification (NER) models built to detect and automatically mask personal data in Turkish text under **Law No. 6698 on the Protection of Personal Data (KVKK, the Turkish GDPR)**. The family consists of the flagship model `emi08/maske`, trained entirely on synthetic data, plus two reference models trained to measure the architecture choice: `emi08/maske-modernbert-tr-bakeoff` and `emi08/maske-berturk-baseline`. All training data is synthetically generated, so no real personal data is ever used during training. The models tag nine entity types with the BIO scheme and are evaluated with a strict span-exact criterion.

## Models

| Model | Notes | WikiANN-tr F1 |
|---|---|---|
| [emi08/maske](https://huggingface.co/emi08/maske) | Flagship — trained on synthetic KVKK data; a training update improving e-mail detection is in progress | **0.9317** |
| [emi08/maske-modernbert-tr-bakeoff](https://huggingface.co/emi08/maske-modernbert-tr-bakeoff) | Architecture bake-off — modernbert-tr-base | 0.929 |
| [emi08/maske-berturk-baseline](https://huggingface.co/emi08/maske-berturk-baseline) | Stage-1 reference — BERTurk base | 0.929 |

## Entity Types

| Type | Description | Example |
|---|---|---|
| PER | Person name | Ayşe Yılmaz |
| ORG | Organization / company | Kolaykargo A.Ş. |
| LOC | City / country | Kadıköy, Ankara |
| ADDR | Street address | Bağdat Cad. No:12 D:3, Kadıköy/İstanbul |
| TC_NO | Turkish national ID (11 digits, with checksum) | 34628194872 |
| IBAN | Turkish IBAN (mod-97 valid) | TR33 0006 1005 1978 6457 8413 26 |
| PHONE | Phone number | 0532 123 45 67, +90 532 123 45 67 |
| EMAIL | E-mail address | ad.soyad@firma.com.tr |
| PLATE | Vehicle license plate | 34 ABC 123 |

## Label Set (19)

~~~text
O
B-PER   I-PER
B-ORG   I-ORG
B-LOC   I-LOC
B-ADDR  I-ADDR
B-TC_NO I-TC_NO
B-IBAN  I-IBAN
B-PHONE I-PHONE
B-EMAIL I-EMAIL
B-PLATE I-PLATE
~~~

## Quick Start

~~~python
from transformers import pipeline

pipe = pipeline(
    "token-classification",
    model="emi08/maske",
    aggregation_strategy="simple",
)

text = (
    "Sayın Ayşe Yılmaz, TC kimlik no 12345678901. "
    "Ödemeyi TR33 0006 1005 1978 6457 8413 26 IBAN hesabına yapın. "
    "Sorularınız için ayse.yilmaz@ornek.com.tr adresine yazın."
)

for entity in pipe(text):
    print(f"{entity['entity_group']:8s} {entity['word']}  ({entity['start']}–{entity['end']})")
~~~

To automatically mask text:

~~~python
def mask_text(text: str, pipe) -> str:
    """Replaces detected PII entities with [THEIR TYPE]."""
    entities = sorted(pipe(text), key=lambda e: e["start"], reverse=True)
    for e in entities:
        text = text[: e["start"]] + f"[{e['entity_group']}]" + text[e["end"] :]
    return text

print(mask_text("Ayşe Yılmaz adına fatura, 34 ABC 123 plakalı araca kesildi.", pipe))
# [PER] adına fatura, [PLATE] plakalı araca kesildi.
~~~

## Evaluation

All numbers are produced by the same span-exact evaluation harness: an entity only counts as correct if both its type and its exact character span match the gold annotation.

### WikiANN-tr comparison

| Model | P | R | F1 |
|---|---|---|---|
| emi08/maske | 0.9260 | 0.9374 | **0.9317** |
| emi08/maske-berturk-baseline | 0.923 | 0.934 | 0.929 |
| emi08/maske-modernbert-tr-bakeoff | 0.923 | 0.935 | 0.929 |
| savasy/bert-base-turkish-ner-cased | 0.919 | 0.930 | 0.925 |
| Werea-co/Werea-TR-NER | 0.907 | 0.917 | 0.912 |
| akdeniz27/bert-base-turkish-cased-ner | 0.699 | 0.649 | 0.673 |

### emi08/maske — per evaluation set

| Set | P | R | F1 |
|---|---|---|---|
| Synthetic test | — | — | 1.0000 |
| WikiANN-tr | 0.9260 | 0.9374 | 0.9317 |
| Werea-KVKK-Bench (240 rows) | 0.1789 | 0.4875 | 0.2617 |
| Real text (29 docs) | 0.6809 | 0.9143 | 0.7805 |

### Real-text evaluation set

The hand-curated set contains **29 documents / 35 gold entities**; 8 of the documents are trap documents (TC ID numbers with a broken checksum digit, invalid IBANs, 11-digit order numbers, card numbers, VKNs, URLs, etc.). Trap values are not part of the gold set and count as false positives if the model reports them; the current release produced 7 FPs on these documents. Per-type gold distribution:

~~~text
PER 7 · ORG 5 · LOC 5 · TC_NO 5 · PHONE 4 · ADDR 3 · IBAN 2 · EMAIL 2 · PLATE 2 — total 35
~~~

**Known issue (being fixed in the current retrain):** e-mail addresses are occasionally caught as fragments on real text (EMAIL F1 = 0.0). The rescue post-processing documented on the model card — dropping EMAIL predictions without '@' and normalizing predictions that overlap a valid e-mail regex to the full address span — addresses this in production.

## Status & Roadmap

- [x] Flagship `emi08/maske` + baseline / bake-off comparison and external-model benchmarks
- [ ] **Training update (in progress):** e-mail booster data (full-span positives + hard negatives for observed FPs) and the rescue post-processing layer
- [ ] To be added once the update finishes: real-text overall F1, real-text EMAIL F1, Werea-KVKK-Bench F1, and the per-type real-text table
- [ ] Final refresh of the model cards

## License

This project is released under the **Apache 2.0** license.

## Citation

~~~bibtex
@misc{maske_2026,
  author       = {emi08},
  title        = {maske: Turkish Personal Data (PII) Detection for KVKK Workflows},
  year         = {2026},
  url          = {https://huggingface.co/emi08/maske}
}
~~~
