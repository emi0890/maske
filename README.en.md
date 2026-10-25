[Türkçe](README.md) · English

# maske 🔒

Turkish PII (personally identifiable data) detection — open source, KVKK-focused, two-layer.

![License](https://img.shields.io/badge/license-MIT-green) ![Model](https://img.shields.io/badge/model-maske--berturk--baseline-blue)

## Why maske?

Existing Turkish PII tools are mostly evaluated on synthetic data only, and none offers a deterministic, zero-false-positive rule layer. When you work with data covered by KVKK (Turkey's GDPR equivalent), "it probably catches it" isn't good enough. maske's answer is two layers:

1. **Rule layer** — TCKN (national ID), IBAN, phone, license plate, email. Checksum and format validation: no model involved in these fields, no hallucinations, zero-false-positive target.
2. **NER layer** — a fine-tuned BERTurk for the flexible parts rules can't catch: suffixed names (Ayşe'ye, Ahmet'ten) and addresses.

And the distinguishing commitment: **every model will be reported on a human-verified golden test set built from real Turkish text** — not synthetic scores.

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
├── .github/
│   └── FUNDING.yml                # "Sponsor this project" button
├── notebooks/
│   └── maske-01-baseline.ipynb    # stage-1 training notebook (Colab/Kaggle)
├── data/                          # stage 2: golden test set will live here
└── maske/                         # stages 3-4: rule layer and package code
~~~

## Contributing

Found a wrong detection? Open an issue. One hard rule: **real personal data is never committed to this repo** — use made-up names/numbers in examples.

## License

[MIT](LICENSE)
