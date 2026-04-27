# Skills reference

Catalogue of skills in this marketplace, with the trigger patterns each one is tuned for.

## medical-acquisition

**Trigger phrases**

- "fetch this DOI", "get the full text", "scrape this paper"
- a bare DOI, PMID, or PMC ID in the user message
- "RACGP examiner report for cycle X"
- "ClinicalKey topic Y", "UpToDate card", "AMH monograph", "eTG topic", "Murtagh chapter"
- "publisher article via USyd OpenAthens"

**Returns**: standardised `AcquisitionResult` (tier, source_type, content_md, metadata, success, fallback_used).

**Backends**: 21 total. OA: pubmed, pmc, europepmc, unpaywall, doi, wayback. AU clinical: racgp_exam, acrrm_exam, ajgp, aihw, choosing_wisely. Cache-first authed: clinicalkey, uptodate, amh, etg, murtagh. USyd-brokered publishers: wiley, elsevier, springer, nature, taylor_francis, bmj, cambridge_oxford, institutional_proxy.

**Auth**: USyd OpenAthens single gateway. Fresh prompt per Python process. Graceful fall-through to OA chain on no-subscription.

**Do not use for**: SERP search (use Tavily / Brightdata), screenshot extraction, non-medical content.

## examiner-feedback-mining

**Trigger phrases**

- "what do RACGP examiners flag"
- "common KFP errors", "common AKT pitfalls", "AMC clinical exam feedback"
- "which examiner class does this answer fall into"
- "PESCI candidate report patterns"

**Returns**: 16-class AU misconception taxonomy plus per-college style notes (RACGP KFP, AKT, RCE / OSCE; ACRRM Rural Generalist; AMC clinical).

**Companion skills**: `distractor-design` for option-level analysis, `kfp-failure-bucket` for stem-level routing.

## distractor-design

**Trigger phrases**

- "write me an MCQ", "author a stem", "write distractors for X"
- "evaluate this MCQ", "are these distractors discriminating"
- "classify why a candidate picked option B"
- "stress-test this question bank"

**Returns**: 72-entry taxonomy (12 categories x 6 sub-classes) plus 5 KFP failure buckets, plus author and reviewer workflows.

**Source basis**: Haladyna and Rodriguez 2016, NBME Item Writing Manual, RACGP examiner reports 2018 to 2024.

## kfp-failure-bucket

**Trigger phrases**

- "why is this answer wrong"
- "which bucket does this fail into"
- "triage these wrong answers"
- "label this candidate response"

**Returns**: one of 8 buckets (specificity miss, ranking error, drug or class error, dose component, investigation tier or threshold, escalation mismatch, cultural-safety / consent / notification miss, premature closure plus safety-netting miss) plus mapped distractor-design taxonomy classes plus corrective study action.

**Operational use**: feed in stem, marking key, candidate answer; get back a structured failure label for the study log.

## When to chain skills

For a full per-question study-log entry:

1. `medical-acquisition` to pull the source guideline / examiner report
2. `kfp-failure-bucket` to label the wrong answer
3. `examiner-feedback-mining` to map onto the 16 AU classes
4. `distractor-design` only if the question is MCQ and you want option-level analysis

For stem authoring:

1. `medical-acquisition` for source pulls (eTG, AMH, Murtagh, RACGP exam reports)
2. `examiner-feedback-mining` to pick the target examiner class
3. `distractor-design` for option-level taxonomy mapping
4. `kfp-failure-bucket` to pre-empt candidate failure modes

The `exam-stem-author` subagent automates this chain.
