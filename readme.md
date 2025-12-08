# **SIGS v1 — Semantic Intent Grounding Symbols (Minimal Release)**

SIGS is an **AI-facing wire language** that encodes common intents as short, deterministic tokens. Humans type normal text; a translator layer converts text ⇄ SIGS. Models operate on SIGS tokens.

This README documents the **wire format** and the **dataset schema** for the current lexicon (≈1040 entries). Categories in this release: **AGR, NEG, MOT, OWN, QTY, TIM, LOC, SRT, FIL, ATT, AGG, OUT, ENT, ASP, CON, EPI, COG, MET, EMO, SOC, ETH, INT, CAU, MOD, LOG, CMP, MAN, PER, PAR**.

---

# \#\# Architecture Overview

# SIGS is an \*\*AI-facing wire protocol\*\* that operates as an invisible translation layer between humans and AI models.

# \#\#\# The Flow

# 1\. \*\*Human input\*\* (natural language with all its messiness)

#    ↓

# 2\. \*\*SIGS Translator\*\* (this model) converts speech to SIGS tokens

#    ↓

# 3\. \*\*SIGS-native AI model\*\* processes the compact, deterministic tokens

#    ↓

# 4\. \*\*SIGS Translator\*\* converts SIGS response back to natural language

#    ↓

# 5\. \*\*Human output\*\* (natural language response)

# 

# \*\*Key point:\*\* Users never see or learn SIGS codes. They speak normally. 

# The translator handles encoding/decoding transparently.

# 

# \#\#\# Why This Matters

# \- \*\*Token efficiency\*\*: 1,040 deterministic codes vs. infinite natural language variations

# \- \*\*Reduced ambiguity\*\*: Precise intent representation for AI models

# \- \*\*Smaller models\*\*: Training on SIGS can produce more efficient, focused models

# \- \*\*Bidirectional\*\*: Same model handles both encoding (human→SIGS) and decoding (SIGS→human)

---

# **1\) Wire Transport (v1)**

**Message form.** A SIGS message is one line of **space-separated tokens**, terminated by newline (LF).

* **Token alphabet:** `[a-z0-9]+` (e.g., `agr001`, `neg02t`, `tim030`)

* **Separator:** single space only (no tabs / no double spaces)

* **Boundary:** newline ends a message

**Out-of-Vocabulary (OOV) fallback — required.**  
 If a concept is not covered by the lexicon, emit a literal block; **never drop text**:

`{txt:"<original words with escapes>"}`

Escapes inside quotes: `\"` for a quote, `\\` for a backslash. No nested `{}` in v1.

**Examples**

* Truth-stance disagreement  
   English: “I disagree with your statement.”  
   SIGS: `neg02t`

* Refusal with temporal constraint  
   English: “I can’t do that now; maybe later.”  
   SIGS: `neg02v tim030`  
   *(Use IDs that exist in your sheet if these differ.)*

---

# **2\) Categories (v1)**

* **AGR	Agreement / affirmation**  
* **NEG	Negation / refusal**  
* **MOT	Motion / Movement**  
* **OWN	Possession / ownership**  
* **QTY	Quantity / Degree**  
* **TIM	Temporal constraints**  
* **LOC	Spatial / constraints**  
* **SRT	Sort / order**  
* **FIL	Filter / predicate**  
* **ATT	Attribute / field select**  
* **AGG	Aggregation**  
* **OUT	Output / format**  
* **ENT	Entity / type hints**  
* **ASP	Aspect / tense**  
* **CON	Condition**  
* **EPI	Epistemic**  
* **COG	Cognition**  
* **MET	Meta-communication**  
* **EMO	Emotion / Affective State**  
* **SOC	Social / Relational Mode**  
* **ETH	Ethical / Moral Stance**  
* **INT	Intent / Goal**  
* **CAU	Causality / Reason**  
* **MOD	Modality / Constraint**  
* **LOG	Logical / Conjunction**  
* **CMP	Comparison / Relation**  
* **MAN	Manner**  
* **PER	Perception / Sensation**  
* **PAR	Paralinguistic / Non-verbal**

---

# **3\) Dataset & Schema**

The authoritative source is the **Master Lexicon** sheet. Exports (CSV) should mirror it column-for-column.

**Columns (order matters):**

1. `token_uid` — `SIGS{CATEGORY_CODE}{INDEX}` (e.g., `SIGSMOT02Z`). Canonical, stable.

2. `category_code` — One of `AGR, NEG, MOT, OWN, QTY, TIM` (uppercase).

3. `category_name` — Human label (e.g., “Temporal constraints”).

4. `index` — **Base-36**, **uppercase**, **min 3 chars** with left-zero padding (e.g., `02Z`, `030`).

5. `regkey_raw` — `SP:{CATEGORY_CODE}:{INDEX}` (e.g., `SP:QTY:030`). Debug/registry namespace.

6. `title` — Short canonical gloss/lemma (ASCII).

7. `definition` — Single, precise sense for this entry.

8. `examples` — Short usage phrases (semicolon-separated).

9. `wire_token` — `lower(category_code + index)` (e.g., `mot02z`, `tim030`). **Globally unique**.

10. `definition_id` — **Global**, strictly increasing integer assigned in the Master sheet; never re-used.

**Identifier & formatting rules**

* **Per-category indexing:** Within a category, `index` advances in Base-36  
   e.g., `… 02X, 02Y, 02Z, 030, 031 …`.

* **Uniqueness:**

  * `(category_code, index)` **must be unique**.

  * `wire_token` **must be unique** across the entire lexicon.

* **Casing:** `category_code` & `index` are uppercase; `wire_token` is lowercase.

* **Allowed chars:** `index` is alphanumeric only; tokens have no spaces/tabs.

---

# **4\) Global ID (`definition_id`)**

Each lexeme receives a single **global integer ID** in the **Master Lexicon**.

* The ID is **monotonic** (strictly increasing) and **never re-used**.

* Category tabs may **mirror** this value for convenience, but **must not generate** or alter it.

* New rows take the **next available** global ID at append time.

---

# **5\) Appending New Entries (maintainers/contributors)**

Submit new entries as a JSON array of objects with these required fields:

`[`  
  `{`  
    `"category_code": "MOT",`  
    `"category_name": "Motion / Action-like",`  
    `"title": "open",`  
    `"definition": "Make a resource visible or active for interaction.",`  
    `"examples": "open the file; open settings"`  
  `}`  
`]`

Upon import, the maintainer’s tool will:

* assign the next **per-category** `index` (Base-36, 3+ chars),

* build `wire_token`, `token_uid`, `regkey_raw`,

* assign the next global `definition_id`,

* enforce uniqueness and formatting rules.

**Quality gates**

* No duplicates of `wire_token` or `(category_code, index)`.

* `index` matches `^[0-9A-Z]{3,}$` and follows Base-36 sequencing per category.

* `definition` is single-sense and unambiguous; `examples` are short and idiomatic.

---

**6\) Dataset Files**

The complete SIGS lexicon is available in this repository:

\- \*\*\`The\_SIGS\_Lexicon\_v1 \- STATIC COPY \- Master Lexicon.csv\`\*\* \- Machine-readable CSV format containing all 1,040+ intent codes with their definitions, examples, and mappings. Use this for programmatic access and data processing.

\- \*\*\`The\_SIGS\_Lexicon\_v1 \- STATIC COPY.xlsx\`\*\* \- Complete workbook with all category tabs. Use this for human-readable browsing and reference.

These files represent the v1 lexicon snapshot that this translator model was trained on. They are static copies to ensure reproducibility and version consistency.

**7\) License**

Copyright 2025 Jaccob Barrett

Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with the License.

You may obtain a copy of the License at  
    [http://www.apache.org/licenses/LICENSE-2.0](http://www.apache.org/licenses/LICENSE-2.0)

Unless required by applicable law or agreed to in writing, software  
distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.

See the License for the specific language governing permissions and limitations under the License.

### **Notes on removed legacy fields**

Older drafts referenced uid\_canonical, scaffold, and namespace. These are not used in v1 and are intentionally absent from the Master schema. Keep all identifiers aligned to the columns listed above.

