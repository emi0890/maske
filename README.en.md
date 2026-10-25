[Türkçe](README.md) · English

# maske 🔒

Turkish PII (personally identifiable data) detection — open source, KVKK-focused, two-layer.

[![License: MIT](https://img.shields.io/badge/license-MIT-green)](LICENSE) [![Model: maske-berturk-baseline](https://img.shields.io/badge/model-maske--berturk--baseline-blue)](https://huggingface.co/emi08/maske-berturk-baseline)

## Why maske?

Existing Turkish PII tools are mostly evaluated on synthetic data only, and none offers a deterministic, zero-false-positive rule layer. When you work with data covered by KVKK (Turkey's GDPR equivalent), "it probably catches it" isn't good enough. maske's answer is two layers:

1. **Rule layer** — TCKN (national ID), IBAN, phone, license plate, email. Checksum and format validation: no model involved in these fields, no hallucinations, zero-false-positive target.
2. **NER layer** — a fine-tuned BERTurk for the flexible parts rules can't catch: suffixed names (Ayşe'ye, Ahmet'ten) and addresses.

And the distinguishing commitment: **every model will be reported on a human-verified golden test set built from real Turkish text** — not synthetic scores.

## Results

The phase 1 baseline (`emi08/maske-berturk-baseline`) compared against four Turkish NER models with the same evaluation script on the full WikiANN-tr test split (10,000 sentences):

| Model | Precision | Recall | F1 |
|---|---|---|---|
| **maske (this project)** | 0.923 | 0.934 | **0.929** |
| savasy/bert-base-turkish-ner-cased | 0.919 | 0.930 | 0.925 |
| Werea-co/Werea-TR-NER | 0.907 | 0.917 | 0.912 |
| akdeniz27/bert-base-turkish-cased-ner | 0.699 | 0.649 | 0.673 |

- All four models were measured on the same test split with the same script; results are not comparable with scores from other datasets.
- The gap to savasy (0.004 F1) is within training-randomness noise; the gap to Werea (0.017 F1) matches the 0.917 F1 reported on Werea's own card. The main difference is the base model (32k vs 128k vocabulary).
- akdeniz27 was trained on the Küçük (2016) dataset, not WikiANN; its low score reflects a train/test data mismatch rather than model quality.
- WikiANN is auto-labeled and news-style, so these scores are a ceiling for real-world text. Real performance will be measured with the phase 2 gold test set.

Last measured: October 2026 · Full method: [model card](https://huggingface.co/emi08/maske-berturk-baseline)

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
