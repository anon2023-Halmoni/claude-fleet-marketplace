---
description: Author a KFP-style practice stem on a named topic, with marking key at the AU specificity bar. Pre-loads distractor-design, examiner-feedback-mining, and kfp-failure-bucket. Routes the work through the exam-stem-author subagent.
---

# /kfp-stem $ARGUMENTS

Author a fully-specified KFP practice stem for the topic in `$ARGUMENTS`.

## Workflow

1. Pull source material via medical-acquisition: eTG topic, AMH drug monograph, Murtagh chapter, plus the most recent RACGP KFP examiner report that mentions the topic.
2. Author the stem in RACGP KFP format: clinical vignette plus 3 to 5 numbered prompts.
3. Author the marking key at AU specificity bar. Drug answers must include drug-dose-route-frequency-duration. Investigation answers must name the test (FBE, UEC, CRP, LFTs, blood cultures x 2). Vague answers like "antibiotics" or "blood tests" are forbidden in the key.
4. Classify each anticipated wrong-answer pattern using kfp-failure-bucket (8 buckets) and distractor-design (72 categories).
5. Cross-reference against examiner-feedback-mining (16 AU classes). Confirm the stem exercises at least one named class.

## Output

```
TOPIC: $ARGUMENTS
SOURCES: <medical-acquisition pulls used>

STEM:
<vignette>
1. <prompt>
2. <prompt>
...

MARKING KEY:
1. <answer at AU specificity bar>
2. <answer>
...

ANTICIPATED FAILURE BUCKETS:
- <bucket label>: <candidate perspective>
- <bucket label>: <candidate perspective>

EXAMINER CLASS EXERCISED:
<one of the 16 AU classes>
```

## Invariants

- Pull from eTG / AMH / Murtagh / RACGP via medical-acquisition. Never invent guideline content.
- Marking key answers MUST clear the specificity bar.
- At least one prompt should require a safety-net or red-flag screen if the topic has any sentinel diagnosis.

If the topic is outside RACGP primary care scope, abort and return "out_of_scope" with a one-line suggestion of the right exam (AMC clinical, ACRRM, specialist college).
