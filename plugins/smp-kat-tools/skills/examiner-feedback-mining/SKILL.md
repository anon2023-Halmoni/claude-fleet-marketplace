---
name: examiner-feedback-mining
description: Surface examiner-flagged failure patterns from RACGP, ACRRM, AMC, and PESCI exam reports. Use when the user asks "what do RACGP examiners flag", "common KFP errors", "AMC clinical exam pitfalls", or wants to map a candidate's wrong answer onto a recognised examiner-criticism class. Returns the wave-1 findings catalogue (16-class AU misconception taxonomy) plus per-college style notes.
disable-model-invocation: false
---

# Examiner feedback mining

Pattern library distilled from Australian college examiner reports. Use this skill before authoring any practice stem, before reviewing a candidate answer, or when the user asks why a model answer differs from the marking key.

## When to use this skill

Activate when the request involves:

- "what do examiners flag" / "what errors does the report mention" / "common pitfalls"
- RACGP KFP / AKT / RCE / OSCE feedback documents
- ACRRM Rural Generalist examination reports
- AMC clinical exam (MCQ + clinical) feedback
- PESCI candidate reports
- mapping a failed answer onto a named examiner-criticism class

Do NOT activate for: general medical content lookup (use medical-acquisition), single-MCQ distractor authoring (use distractor-design), KFP option-by-option failure routing (use kfp-failure-bucket).

## Core taxonomy: 16 examiner-flagged classes (AU)

Distilled from RACGP KFP cycles 2018 to 2024 plus AMC clinical exam reports 2019 to 2023.

1. Premature closure: anchoring on the first plausible diagnosis without ruling in / out alternates
2. Failure to safety-net: no follow-up window, no red-flag list, no return advice
3. Inappropriate escalation: ED referral when ambulatory work-up fits Australian primary care scope
4. Inappropriate non-escalation: managing in primary care when guidelines mandate ED, paeds review, or oncology MDT
5. Wrong investigation tier: ordering MRI when ultrasound or X-ray is first-line per RACGP guideline
6. Missing red-flag screen: no question or check that excludes a sentinel diagnosis
7. Drug dose / interval error: correct drug, wrong dose, wrong frequency, or wrong duration per AMH
8. Drug class error: wrong class for the indication per eTG
9. Missed contraindication: drug or procedure given in face of named contraindication
10. Cultural safety blind spot: missing Aboriginal and Torres Strait Islander health context, missing interpreter, missing trauma-informed framing
11. Consent / capacity error: no capacity check, no Gillick assessment, missing advance care directive review
12. Notification / mandatory reporting miss: failing to flag notifiable disease, child protection, elder abuse, fitness to drive
13. Documentation gap: care plan, MHCP, GPMP, TCA missing or incomplete
14. Medicare item number error: wrong item number for the consultation type
15. Practice-context mismatch: solution assumes urban tertiary access in a rural / remote scenario
16. Empathy and communication: no acknowledgement of patient distress, no shared decision making

## Per-college style notes

### RACGP KFP

Examiners reward specificity. A KFP answer that says "antibiotics" when the marking key wants "amoxicillin 500 mg orally three times daily for five days" scores zero. Generic responses fail. Aim for drug name plus dose plus route plus frequency plus duration. For investigations, name the exact test (FBE, UEC, CRP, LFTs) rather than "blood tests".

Examiners punish over-investigation. KFP cycles 2022 and 2023 explicitly flagged candidates who ordered CT before clinical exam, MRI before plain film, and tumour markers in screening contexts.

### RACGP AKT

Multiple-best-answer. Examiners flag candidates who pick a defensible but second-line option when a first-line option is present. Read the stem for context cues (rural, paediatric, pregnancy) that shift the first-line choice.

### RACGP RCE / OSCE

Communication, structure, safety-netting. Examiners explicitly score "explained the plan in plain language", "offered a follow-up window", "screened for red flags" as separate marking points. Failing one of these is a common path to a marginal pass that becomes a fail.

### ACRRM Rural Generalist

Resource-constrained context is mandatory. Solutions assuming tertiary access fail. Examiners reward retrieval planning, telehealth referral pathways, and procedural backup arrangements.

### AMC clinical

Junior-doctor context. Examiners reward clear escalation lines, explicit handover, and recognition of when a case is beyond scope. Empathy and cultural safety are scored separately from the medical decision.

## Operational use

When a user asks "why is my answer wrong", run this sequence:

1. Identify which examiner class the wrong answer falls into (1 to 16 above)
2. Quote the relevant report excerpt if available via medical-acquisition `racgp_exam` or `acrrm_exam` backend
3. Suggest the corrected answer in the form the marking key expects (specificity bar from per-college notes)
4. Flag any second-order classes the candidate is at risk of (e.g. premature closure plus missing red-flag screen often co-occur)

When authoring a stem, run this sequence:

1. Pick the target examiner class from the 16
2. Build the distractors so each distractor exemplifies a different class (see distractor-design skill)
3. Confirm the marking key answer requires the specificity that examiners flag (drug-dose-route-frequency-duration, named investigation)

## Pointers

- Wave-1 findings: `~/projects/personal/smp-kat-study/knowledge/examiner-feedback-wave1.md` (when present)
- RACGP exam acquisition: `~/tools/bin/acquisition fetch '{"cycle":"2024.2","exam_type":"akt"}' --backend racgp_exam`
- AMC exam reports: via medical-acquisition `pubmed` or direct AMC URL through `institutional_proxy`
- Companion skills: distractor-design (option-level taxonomy), kfp-failure-bucket (per-stem bucket router)
