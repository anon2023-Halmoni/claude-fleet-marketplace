---
name: distractor-design
description: Author or evaluate MCQ and KFP distractors against a 72-entry taxonomy plus the 5 KFP failure buckets. Use when the user asks to write a stem with distractors, evaluate whether existing distractors discriminate, classify a wrong-answer pattern, or stress-test an option set for tautology, give-aways, or examiner-flagged sloppiness. Returns taxonomy class, discrimination notes, and a rewrite suggestion when the option fails.
disable-model-invocation: false
---

# Distractor design

Operational taxonomy for authoring MCQ / KFP distractors that discriminate. Built from psychometric literature (Haladyna 2016, Case and Swanson NBME guide) plus AU exam style cues from RACGP and AMC reports.

## When to use this skill

Activate when the request involves:

- writing a new MCQ or KFP stem with distractors
- reviewing an existing stem for distractor quality
- classifying why a candidate picked the wrong option (option-level analysis)
- stress-testing a question bank for predictable patterns
- producing a rewrite of a weak distractor

Do NOT activate for: stem-level failure analysis (use kfp-failure-bucket), examiner-style review of an entire answer (use examiner-feedback-mining), retrieval of source material (use medical-acquisition).

## The 72-entry taxonomy (12 categories x 6 sub-classes)

Each distractor falls into exactly one category. The author's job is to spread distractors across categories so no two options share a category in the same stem. A stem with two distractors in category 3 is psychometrically wasteful.

### Category 1: Wrong drug class

1.1 Right indication, wrong class (NSAID for migraine prophylaxis instead of beta-blocker)
1.2 Same class, weaker evidence (atenolol for prophylaxis instead of propranolol)
1.3 Same class, contraindicated subgroup (non-cardioselective beta-blocker in asthma)
1.4 Adjacent class with overlapping receptor (alpha-blocker for migraine instead of beta-blocker)
1.5 Outdated standard (ergotamine where triptan is now first-line)
1.6 Veterinary or off-label adjacent

### Category 2: Wrong dose

2.1 Adult dose in paediatric patient
2.2 Loading dose used as maintenance
2.3 Frequency error (BD when TDS required)
2.4 Duration error (5 days when 10 days mandated)
2.5 Route error (oral when IV indicated)
2.6 Renal / hepatic dose-adjustment ignored

### Category 3: Wrong investigation

3.1 Right modality, wrong tier (CT before X-ray)
3.2 Right test, wrong context (D-dimer in low-pretest probability)
3.3 Outdated marker (CK-MB when troponin is standard)
3.4 Screening test misused as diagnostic
3.5 Specialist test in primary care first-line slot
3.6 Test that does not exclude the sentinel diagnosis

### Category 4: Wrong target / threshold

4.1 BP target wrong for population (140/90 when CKD wants 130/80)
4.2 HbA1c target wrong for age / frailty
4.3 Cholesterol target wrong for primary vs secondary prevention
4.4 Anticoagulation target INR wrong for indication
4.5 Threshold for treatment vs threshold for screening conflated
4.6 Population threshold applied to individual

### Category 5: Wrong timing

5.1 Right action, wrong window (thrombolysis past 4.5 h)
5.2 Premature intervention (statin before lifestyle trial)
5.3 Delayed intervention (deferring biopsy past red-flag window)
5.4 Wrong follow-up interval (6 months when 6 weeks)
5.5 Pre-op / peri-op timing error
5.6 Pregnancy trimester mismatch

### Category 6: Wrong patient population

6.1 Adult guideline applied to child
6.2 Non-pregnant guideline applied in pregnancy
6.3 Non-Indigenous guideline applied without ATSI adjustment
6.4 Urban-tertiary plan applied in rural-remote
6.5 Non-immunocompromised plan in immunocompromised
6.6 Adult plan in older frail patient

### Category 7: Wrong site / route

7.1 Topical when systemic indicated
7.2 Systemic when topical sufficient
7.3 Wrong injection site / depth
7.4 Wrong inhaler device for age
7.5 Oral when patient NBM
7.6 Rectal when oral tolerated

### Category 8: Wrong escalation

8.1 ED when GP scope sufficient
8.2 GP when ED indicated
8.3 Specialist referral when GP follow-up sufficient
8.4 GP follow-up when specialist mandated
8.5 Inpatient admission when ambulatory care fits
8.6 Discharge when admission criteria met

### Category 9: Wrong consent / capacity / legal

9.1 Capacity not assessed
9.2 Gillick competence not applied
9.3 Mandatory reporting missed
9.4 Notifiable disease not flagged
9.5 Fitness to drive not addressed
9.6 Advance care directive not respected

### Category 10: Wrong communication / cultural safety

10.1 Interpreter not offered
10.2 ATSI cultural framing missing
10.3 Trauma-informed approach missing
10.4 Plain-language explanation missing
10.5 Shared decision making missing
10.6 Safety-netting missing

### Category 11: Wrong documentation / Medicare

11.1 GPMP / TCA item number error
11.2 MHCP item number error
11.3 Telehealth item number error
11.4 Care plan content gap
11.5 Recall and reminder system gap
11.6 Privacy or My Health Record handling

### Category 12: Plausible but tautological

12.1 Restates the stem in different words
12.2 Catch-all "all of the above" pattern
12.3 Negative-knowledge probe ("which is NOT")
12.4 Convergent absurdity (clearly wrong, fails as distractor)
12.5 Give-away with grammatical mismatch
12.6 Length give-away (correct option visibly longer)

## The 5 KFP failure buckets (option-level)

For KFP short-answer banks, distractors are written-in candidate responses rather than fixed options. The 5 buckets identify the dominant wrong-answer pattern at the option layer.

A. Vague generic ("antibiotics", "blood tests", "scan") that fails the specificity bar
B. Defensible but second-line (correct class, wrong first-line choice)
C. Out-of-scope escalation or non-escalation
D. Drug-dose-frequency-duration partial (drug correct, one component wrong)
E. Tautology or restated stem

## Author workflow

1. Pick the target answer and verify it is the ONLY option that satisfies all stem cues
2. Pick 4 distractor categories from the 72-entry taxonomy, spread across at least 3 different parent categories
3. Write each distractor to be defensible enough that a confident candidate with one knowledge gap would pick it
4. Stress-test for category 12 (tautology / give-away) on every option
5. Run the kfp-failure-bucket skill if the stem is a KFP short-answer

## Reviewer workflow

1. Map each option to its taxonomy slot
2. Flag two options sharing a parent category as wasted distractor space
3. Flag any option in category 12 as a rewrite candidate
4. Confirm the correct option does not telegraph itself by length, grammar, or specificity asymmetry
5. Confirm at least one distractor from category 9, 10, or 11 if the stem context warrants

## Pointers

- Companion skills: kfp-failure-bucket (KFP-specific bucket router), examiner-feedback-mining (per-college style)
- Source: Haladyna T M, Rodriguez M C 2016 "Developing and Validating Test Items"; NBME Item Writing Manual; RACGP examiner reports 2018 to 2024
- Project home: `~/projects/personal/smp-kat-study/`
