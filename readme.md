This was the initial research phase for the intent classification protocol. For the production system, see BridgeLang.

# SIGS v1 — Semantic Intent Grounding Symbols (Atomic Router Release)

[![Hugging Face](https://img.shields.io/badge/%F0%9F%A4%97%20Hugging%20Face-Model-yellow)](https://huggingface.co/J-Barrert/SIGS-v1-Atomic-Encoder)
### Note: The V1 model is an Atomic Intent Classifier optimized for routing and state detection. It maps inputs to the single most relevant token. V2 (Compositional/Multi-Token) is currently in training.

SIGS is an **AI-facing wire language** that encodes common user intents as short, deterministic tokens. Humans write normal text; a translator layer converts:

**Natural Language ⇄ SIGS Tokens**

Models operate directly on SIGS tokens for clarity and efficiency.

This repository contains:
- The **Master Lexicon dataset** (≈1040 intent symbols)
- Documentation of the **wire format**
- Rules and schema for extending the lexicon

Categories defined in this release:  
AGR, NEG, MOT, OWN, QTY, TIM, LOC, SRT, FIL, ATT, AGG, OUT, ENT, ASP, CON, EPI, COG, MET, EMO, SOC, ETH, INT, CAU, MOD, LOG, CMP, MAN, PER, PAR

---

## Architecture Overview

SIGS functions as a hidden translation layer between humans and AI models.

### Processing Flow

1. **Human input** — plain language
2. **SIGS Translator** — converts to SIGS tokens
3. **SIGS-native AI model** — processes deterministic intents
4. **SIGS Translator** — converts tokens back to human language
5. **Human output**

Users never see or learn SIGS directly — the translator handles everything.

### Why SIGS?

- **Token efficiency**  
  1,040 canonical tokens vs infinitely variable language
- **Precise semantics**  
  Removes ambiguity in intent representation
- **Smaller & specialized models**  
- **Bidirectional**  
  Same system handles encoding and decoding

---

## 1) Wire Transport (v1)

**Message structure**
- Tokens: lowercase `[a-z0-9]+`  
  Example: `tim030`, `mot02z`
- Separator: single space
- Message ends with newline (`\n`)

**OOV fallback**

If a concept is not in the lexicon:
{txt:"original text here"}


Escapes:
- `\"` for quotes
- `\\` for backslash

**Examples**

| English | SIGS |
|--------|------|
| “I disagree.” | `neg02t` |
| “Not now, maybe later.” | `neg02v tim030` |

*(Token IDs shown are examples — refer to dataset for canon values.)*

---

## 2) Category Codes

| Code | Domain |
|------|--------|
| AGR | Agreement / Affirmation |
| NEG | Negation / Refusal |
| MOT | Motion / Movement |
| OWN | Possession |
| QTY | Quantity / Degree |
| TIM | Temporal |
| LOC | Spatial |
| SRT | Sorting |
| FIL | Filtering |
| ATT | Attribute selection |
| AGG | Aggregation |
| OUT | Output format |
| ENT | Entity type hints |
| ASP | Aspect / Tense |
| CON | Conditionals |
| EPI | Epistemic stance |
| COG | Cognition |
| MET | Meta-communication |
| EMO | Emotion |
| SOC | Social mode |
| ETH | Ethical stance |
| INT | Intent / Goal |
| CAU | Causality |
| MOD | Modality |
| LOG | Logic |
| CMP | Comparison |
| MAN | Manner |
| PER | Perception |
| PAR | Paralinguistic |

---

## 3) Dataset Schema

Column order:

1. `token_uid` — canonical ID (`SIGSMOT02Z`)
2. `category_code` — uppercase (`MOT`)
3. `category_name` — human label
4. `index` — base-36, uppercase, ≥3 digits (`02Z`, `030`)
5. `regkey_raw` — registry namespace (`SP:QTY:030`)
6. `title` — short lemma
7. `definition` — precise single sense
8. `examples` — semicolon-separated list
9. `wire_token` — lowercase concat (`mot02z`)
10. `definition_id` — monotonically increasing global integer

---

## 4) Global ID Rules

- Strictly increasing values
- Never re-used
- Governs uniqueness across entire lexicon

---

## 5) Adding New Entries

Submit new lexicon entries as JSON array:

```json
[
  {
    "category_code": "MOT",
    "title": "open",
    "definition": "Make a resource visible or active.",
    "examples": "open the file; open settings"
  }
]
```

### Maintainer tools automatically apply:

- Base-36 index assignment

- Token generation

- Global ID

- Validation (uniqueness + formatting)

## 6) Dataset Files

Included:

- the_sigs_lexicon_v1-static_copy-master_lexicon.csv
Machine-readable dataset for training and inference

Planned additions:

- Processed training artifacts

- SIGS Translator checkpoints (via Hugging Face links)

## 7) License — Apache 2.0

 Copyright 2025 Jaccob Barrett
 
 Licensed under the Apache License, Version 2.0 (the "License");
 http://www.apache.org/licenses/LICENSE-2.0

 Distributed on an “AS IS” basis with no warranties or conditions.

### Status

This is the v1 SIGS lexicon snapshot.
Future updates will extend categories and refine definitions.

Contributors welcome.
