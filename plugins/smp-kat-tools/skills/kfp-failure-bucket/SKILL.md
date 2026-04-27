---
name: kfp-failure-bucket
description: Route a candidate's wrong KFP answer into one of 8 named failure buckets and map it onto the distractor-design 12-category taxonomy. Use when the user asks "why is this answer wrong", "which examiner class does this fall into", or wants a structured failure-mode label for a model-vs-candidate diff. Returns bucket label, candidate-perspective explanation, and the corrective specificity bar examiners want.
disable-model-invocation: false
---

# KFP failure bucket router

Maps any wrong KFP short-answer onto one of 8 buckets. Bucket choice drives the corrective study action: more reading, more practice stems, more specificity drilling, or more cultural-safety framing.

## When to use this skill

Activate when the request involves:

- "why is this answer wrong" on a KFP-style short-answer
- comparing a candidate response against a marking key
- triaging a batch of wrong answers into study-action buckets
- producing a per-question failure label for a study log

Do NOT activate for: stem authoring (use distractor-design), full per-college style review (use examiner-feedback-mining), retrieval of source material (use medical-acquisition).

## The 8 buckets

### Bucket 1: Specificity bar miss

The candidate named the right thing but at the wrong granularity. "Antibiotics" when the key wants "amoxicillin 500 mg orally three times daily for five days". "Blood tests" when the key wants "FBE, UEC, CRP, LFTs, blood cultures x 2".

Maps to distractor-design KFP bucket A (vague generic).

Corrective action: drill drug-dose-route-frequency-duration cards. Drill named-investigation cards.

### Bucket 2: Knowledge correct, ranking wrong

The candidate picked a defensible second-line option when a first-line option is clearly indicated. Knows the topic; missed the ranking.

Maps to distractor-design KFP bucket B (defensible but second-line).

Corrective action: pull the eTG / RACGP red book first-line list for that condition. Drill "what is first-line for X" cards.

### Bucket 3: Wrong drug or class

The candidate picked an entirely wrong drug or wrong class for the indication.

Maps to distractor-design taxonomy 1.1 to 1.6.

Corrective action: re-read the relevant eTG topic via medical-acquisition `etg` backend. Drill class-to-indication mapping cards.

### Bucket 4: Wrong dose component

The candidate named the right drug but got dose, route, frequency, or duration wrong.

Maps to distractor-design taxonomy 2.1 to 2.6 plus KFP bucket D.

Corrective action: AMH drug monograph re-read via medical-acquisition `amh` backend. Confirm renal / hepatic / paeds adjustment table.

### Bucket 5: Investigation tier or threshold error

The candidate ordered the wrong-tier test or applied the wrong threshold.

Maps to distractor-design taxonomy 3.1 to 3.6 plus 4.1 to 4.6.

Corrective action: pull RACGP red book or Choosing Wisely Australia recommendation for that screening / diagnostic question.

### Bucket 6: Escalation mismatch

The candidate sent the patient to ED when GP scope was sufficient, or kept the patient in GP scope when ED was mandated.

Maps to distractor-design taxonomy 8.1 to 8.6.

Corrective action: drill the named criteria for ED referral in that condition. Confirm the rural-remote variant if applicable.

### Bucket 7: Cultural safety, consent, or notification miss

The candidate produced a clinically defensible answer but missed the ATSI framing, missed interpreter offer, missed mandatory reporting, missed notifiable disease, missed fitness-to-drive, missed capacity check.

Maps to distractor-design taxonomy 9.1 to 9.6 plus 10.1 to 10.6.

Corrective action: re-read the cultural safety, consent, capacity, and notification chapters in Murtagh GP 9e via medical-acquisition `murtagh`.

### Bucket 8: Premature closure plus safety-netting miss

The candidate locked onto one diagnosis and did not safety-net for the missed-sentinel scenario. Often co-occurs with bucket 5.

Maps to distractor-design taxonomy 12.1 to 12.6 (because it is the option-set tell rather than a class error).

Corrective action: drill differential-diagnosis-and-safety-net cards. For every red-flag presentation, write down 3 differentials and the safety-net advice for each.

## Routing algorithm

```
input: stem, marking_key_answer, candidate_answer
output: bucket_label, taxonomy_classes, corrective_action

1. Tokenise candidate_answer and marking_key_answer
2. If candidate_answer is a vague generic (single noun without dose / interval / duration): bucket 1
3. If candidate_answer names a different drug or class: bucket 3
4. If candidate_answer names the same drug but a different dose / route / frequency / duration: bucket 4
5. If candidate_answer names a defensible second-line treatment: bucket 2
6. If candidate_answer names a different investigation or threshold: bucket 5
7. If candidate_answer escalates differently from the key: bucket 6
8. If marking_key_answer requires cultural safety / consent / notification framing not present in candidate_answer: bucket 7
9. If candidate_answer locks onto one diagnosis without differential or safety-net when the stem flagged a red flag: bucket 8
10. If multiple buckets fire, prioritise 7, then 8, then the first matching above
```

## Output shape

For any one wrong answer, produce:

```
bucket: <label>
taxonomy_classes: [<list from distractor-design>]
candidate_perspective: <one paragraph on what the candidate likely thought>
specificity_bar: <what the marking key wants>
corrective_action: <study action from the per-bucket section above>
```

## Pointers

- Companion skills: distractor-design (option-level 72-class taxonomy), examiner-feedback-mining (per-college style)
- Project: `~/projects/personal/smp-kat-study/`
- KFP examiner reports via medical-acquisition `racgp_exam`
- AMH for dose drilling via medical-acquisition `amh`
- eTG for first-line drilling via medical-acquisition `etg`
- Murtagh GP 9e for cultural safety / consent via medical-acquisition `murtagh`
