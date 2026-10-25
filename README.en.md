[Türkçe](README.md) · English

# maske 🔒

Turkish PII (personally identifiable data) detection — open source, KVKK-focused, two-layer.

[![License: MIT](https://img.shields.io/badge/license-MIT-green)](LICENSE) [![Model: maske-berturk-baseline](https://img.shields.io/badge/model-maske--berturk--baseline-blue)](https://huggingface.co/emi08/maske-berturk-baseline)

## Why maske?

Existing Turkish PII tools are mostly evaluated on synthetic data only, and none offers a deterministic, zero-false-positive rule layer. When you work with data covered by KVKK (Turkey's GDPR equivalent), "it probably catches it" isn't good enough. maske's answer is two layers:

1. **Rule layer** — TCKN (national ID), IBAN, phone, license plate, email. Checksum and format validation: no model involved in these fields, no hallucinations, zero-false-positive target.
2. **NER layer** — a fine-tuned BERTurk for the flexible parts rules can't catch: suffixed names (Ayşe'ye, Ahmet'ten) and addresses.

And the distinguishing commitment: **every model will be reported on a human-verified golden test set built from real Turkish text** — not synthetic scores.

### Comparison

All four models measured with the same evaluation script on the full WikiANN-tr test split (10,000 sentences) — entity-level seqeval precision/recall/F1:

| Model | Precision | Recall | F1 |
|---|---|---|---|
| **emi08/maske-berturk-baseline** | 0.923 | 0.934 | **0.929** |
| savasy/bert-base-turkish-ner-cased | 0.919 | 0.930 | 0.925 |
| Werea-co/Werea-TR-NER | 0.907 | 0.917 | 0.912 |
| akdeniz27/bert-base-turkish-cased-ner | 0.699 | 0.649 | 0.673 |

Notes:

- All four models were measured with the same script on the same test split; results are not comparable with evaluations on other datasets.
- The gap to savasy (0.004 F1) is within training-randomness noise; not a claim of superiority.
- The gap to Werea (0.017 F1) is above that noise level and is consistent with the 0.917 F1 reported on Werea's own model card. The main difference is the base model: Werea fine-tunes the 32k-vocabulary dbmdz/bert-base-turkish-cased, while maske fine-tunes the 128k-vocabulary dbmdz/bert-base-turkish-128k-cased.
- akdeniz27/bert-base-turkish-cased-ner was trained on the Küçük (2016) Turkish NER dataset rather than WikiANN, so its low score on WikiANN-tr reflects a train/test data mismatch more than model quality.
- WikiANN is auto-labeled and news-style, so these scores are a ceiling for real-world text; real-text performance will be measured with the gold test set in phase 2.

## Status

| Stage | Output | Status |
|---|---|---|
| 1 — Baseline NER | [emi08/maske-berturk-baseline](https://huggingface.co/emi08/maske-berturk-baseline) (PER/ORG/LOC) | ✅ |
| 2 — Golden test set | human-verified evaluation set from real text | 🔜 |
| 3 — Rule layer | TCKN/IBAN/phone validators | 🔜 |
| 4 — maske package | `pip install maske` — unified detection + masking | 🔜 |

## Quick start (stage 1)

~~~python
from transformers import pipeline

ner = pipeline(
    "token-classification",
    model="emi08/maske-berturk-baseline",
    aggregation_strategy="simple",
)
print(ner("Ayşe'ye 500 TL gönderdim, makbuzu Ahmet Bey'in e-postasına yazdım."))
~~~

⚠️ Experimental baseline — not production-ready. Details: [model card](https://huggingface.co/emi08/maske-berturk-baseline).

## Repository layout (planned)

~~~text
maske/
├── README.md                      # Türkçe
├── README.en.md                   # you are here
├── LICENSE                        # MIT
├── data/                          # stage 2: golden test set will live here
└── maske/                         # stages 3-4: rule layer and package code
~~~

## Contributing

Found a wrong detection? Open an issue. One hard rule: **real personal data is never committed to this repo** — use made-up names/numbers in examples.

## License

[MIT](LICENSE)
