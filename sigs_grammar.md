# SIGS Grammar & Syntax Specification (v1)


SIGS (Semantic Intent Grounding Symbols) is a strict wire format. Unlike natural language, it does not have "flowery" syntax, but it does have rules.


## 1. Message Structure
A SIGS message is a linear sequence of tokens separated by single spaces.


**Format:**
`[ACTION/STATE] [MODIFIERS] [CONTEXT]`


* **ACTION:** The core intent (e.g., `agr001`, `neg02v`, `mot033`).
* **MODIFIERS:** Time, Quantity, or Manner (e.g., `tim02u`, `qty02j`, `man001`).
* **CONTEXT:** Entities or Objects (if strictly necessary and not handled by OOV).


**Example:**
* *English:* "I strongly refuse to do this right now."
* *SIGS:* `neg004 tim02t`
    * `neg004`: Complete dismissal/refusal
    * `tim02t`: Now / Present moment


## 2. Token Ordering (Canonical Order)
To ensure models can predict tokens efficiently, we recommend (but do not enforce) the following order:


1.  **Primary Intent** (AGR, NEG, INT, MOT, REQ)
2.  **Epistemic/Modal** (MOD, EPI, ASP)
3.  **Details/Constraints** (TIM, LOC, QTY, MAN)


**Good:** `int001 tim02v` ("I intend to do this tomorrow")
**Bad:** `tim02v int001` (Technically readable, but breaks convention)


## 3. Combinatorics (Stacking)
You can stack tokens to create complex meaning.


* **Intensifiers:** Use `qty02j` (High Degree) to boost an emotion.
    * `emo001` (Joy) + `qty02j` (High Degree) = "Overjoyed"
* **Negation:** Use `neg` tokens to inverse preceding logic if a specific antonym doesn't exist, though specific `neg` codes are preferred.


## 4. The "Entity" Problem (Pronouns)
SIGS is agent-centric.
* **First Person ("I"):** Implied by the transmission. If Agent A sends `agr001`, it means "I agree."
* **Second Person ("You"):** Implied by the receipt.
* **Third Person:** Explicitly tagged using `ent` codes or OOV literals if critical.


## 5. Handling Unknowns (OOV)
If a concept does not exist in the lexicon, use the JSON fallback:
`{txt:"original text"}`


* *Example:* "Deploying to Kubernetes."
* *SIGS:* `mot030 {txt:"Kubernetes"}`