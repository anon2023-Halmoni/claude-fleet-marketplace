---
name: medical-acquisition
description: Fetch medical references (PubMed, PMC, EuropePMC, RACGP/ACRRM/AMC exam reports, AJGP, AIHW, ClinicalKey, UpToDate, AMH, eTG, Murtagh, plus 8 generic publishers via USyd OpenAthens) using a unified Python framework. Use when the user provides a DOI, PMID, PMC ID, RACGP exam cycle, AU clinical reference, or asks to fetch any academic / examiner-feedback / clinical-guideline content. Returns standardised AcquisitionResult with tier-tagged content. Single auth gateway (USyd OpenAthens) prompts fresh per run; graceful OA fallback on no-subscription.
disable-model-invocation: false
---

# Medical reference acquisition

Fleet-wide unified scraper for medical references. One contract, 21 backends, single import surface. The framework lives at `~/claude-library/acquisition-client/`. Compatibility shim at `~/projects/personal/scrape-med-references/src/acquisition/` keeps existing absolute-path imports working.

## When to use this skill

Activate when the request involves any of:

- DOI, PMID, PMC ID, EuropePMC ID
- RACGP / ACRRM / AMC examiner reports or feedback documents
- AJGP (Australian Journal of General Practice) articles or Clinical Challenge cases
- AIHW reports, Choosing Wisely Australia recommendations
- ClinicalKey topics, UpToDate cards, AMH drug monographs, eTG topics, Murtagh chapters
- Generic publisher articles (Wiley, Elsevier, Springer, Nature, Taylor and Francis, BMJ, Cambridge, Oxford) where the owner has USyd OpenAthens access
- Anything asking for "fetch the paper", "get the full text", "scrape this DOI"

Do NOT activate for: general SERP search (use Tavily / Brightdata search), screenshot-driven extraction, or any non-medical content.

## CLI entry point

```
~/tools/bin/acquisition list-backends
~/tools/bin/acquisition fetch "10.31128/AJGP-04-23-6803"
~/tools/bin/acquisition fetch "PMID:38825755"
~/tools/bin/acquisition fetch '{"cycle":"2024.2","exam_type":"akt"}' --backend racgp_exam
~/tools/bin/acquisition fetch-batch specs.json --max-concurrent 5
```

The shim is invocable from any working directory and any Python environment.

## Library entry point

```python
from acquisition import fetch, fetch_batch, list_backends

res = fetch("https://pubmed.ncbi.nlm.nih.gov/12345678/")
print(res.tier, res.source_type, res.success)

res = fetch({"pmid": "12345678"}, backend_name="pubmed")

results = fetch_batch(
    [
        "https://pubmed.ncbi.nlm.nih.gov/12345678/",
        {"spec": "https://racgp.org.au/ajgp/...", "backend": "ajgp"},
    ],
    max_concurrent=5,
)
```

If installed via `pip install -e ~/claude-library/acquisition-client`, no path manipulation is needed. Otherwise inject `~/claude-library/acquisition-client/src` into `sys.path` first.

## Backend catalogue (21 total)

Open access, no auth:

- `pubmed` (P2, abstract via E-utilities)
- `pmc` (P2, full text from PMC OA subset)
- `europepmc` (P2, full text mirror with broader coverage)
- `unpaywall` (P2, OA PDF locator by DOI)
- `doi` (P2, dispatcher that walks PMC then Unpaywall then EuropePMC then Wayback)
- `wayback` (P3, Internet Archive fallback)

Australian clinical, no auth:

- `racgp_exam` (P1, examiner reports KFP / AKT / OSCE / RCE by cycle)
- `acrrm_exam` (P1, ACRRM Rural Generalist examiner feedback)
- `ajgp` (P1, Australian Journal of General Practice articles)
- `aihw` (P1, Institute of Health and Welfare reports)
- `choosing_wisely` (P1, Choosing Wisely Australia recommendations)

Authed clinical, USyd-brokered, cache-first:

- `clinicalkey` (P0, Talley Examination Med 10e and other ClinicalKey content)
- `uptodate` (P0, topic cards via Camoufox crawler)
- `amh` (P0, Australian Medicines Handbook drug monographs)
- `etg` (P0, Therapeutic Guidelines topics)
- `murtagh` (P0, Murtagh's General Practice 9e chapters)

USyd-brokered generic publishers (paywall fallback to OA chain):

- `wiley`, `elsevier`, `springer`, `nature`, `taylor_francis`, `bmj`, `cambridge_oxford`, `institutional_proxy`

Use `list_backends()` for the live JSON catalogue including `requires_auth`, `default_tier`, `rate_limit_per_minute`, and `description` per backend.

## Standard return contract

Every backend returns an `AcquisitionResult`:

| field | type | notes |
| --- | --- | --- |
| `url` | `str` | canonical URL the content came from |
| `retrieval_date` | `str` | YYYY-MM-DD UTC |
| `source_type` | `str` | e.g. `pubmed_abstract`, `ajgp_article`, `etg_topic` |
| `tier` | `str` | `P0` / `P1` / `P2` / `P3` |
| `content_md` | `str` | Markdown rendering |
| `content_html` | `Optional[str]` | raw HTML when available |
| `metadata` | `dict` | backend-specific (authors, year, DOI, ...) |
| `success` | `bool` | False on any failure |
| `error` | `Optional[str]` | error message when success is False |
| `fallback_used` | `Optional[str]` | name of fallback path taken, if any |

Failures inside a backend are caught at the framework level and folded into a result with `success=False`. Batch callers never need to wrap individual calls.

## USyd OpenAthens single auth gateway

The owner's only institutional auth route is USyd OpenAthens. No direct subscriptions to Wiley, Elsevier, Springer, Nature, Taylor and Francis, BMJ, Cambridge, Oxford, ClinicalKey, UpToDate, AMH, eTG, or Murtagh exist. Every paywalled backend routes through `*.usyd.idm.oclc.org` via `acquisition.auth.usyd_openathens`.

Behaviour:

- **Fresh prompt per run.** First call to `get_session()` in a Python process invokes `prompt_oauth("usyd_openathens", ...)`. The user pastes cookies, a verbatim Cookie header, and an optional User-Agent.
- **In-memory session reused within the run.** Subsequent calls return the cached `requests.Session` without re-prompting. Never persisted to disk across runs.
- **Graceful fallthrough on no-subscription.** If a publisher 401/402/403s or returns a paywall interstitial under the proxy, the publisher backend returns `success=False, error="usyd_no_subscription_or_paywall"`. The DOI dispatcher catches this and falls through to Unpaywall, then Europe PMC, then Wayback.
- **Proxy URL rewrite pattern.** `proxy_url("https://onlinelibrary.wiley.com/doi/10.1002/x")` returns `"https://onlinelibrary-wiley-com.usyd.idm.oclc.org/doi/10.1002/x"`. Dots in the publisher hostname become hyphens; `usyd.idm.oclc.org` is appended.
- **Reset mid-run.** `acquisition.auth.usyd_openathens.reset_session()` clears the cached session and forces a fresh prompt on the next `get_session()` call. Use when an OpenAthens session expires mid-batch.

```python
from acquisition.auth.usyd_openathens import (
    get_session, proxy_url, is_paywall_interstitial, reset_session,
)

session = get_session()
proxied = proxy_url(direct_url)
resp = session.get(proxied, timeout=60)
if resp.status_code in (401, 402, 403) or is_paywall_interstitial(resp.text):
    pass  # fall through to OA chain
```

## Cache-first authed crawlers

ClinicalKey, UpToDate, AMH, eTG, and Murtagh keep their cache-first behaviour against local SQLite databases (`talley-exam-med-10e.db`, `uptodate.db`, `amh-online.db`, `etg-complete.db`, `murtagh-gp-9e.db`). On cache miss the backend calls `get_session()` to confirm the USyd session is live, then returns a structured fail-loud message pointing the caller at the dedicated CDP / Camoufox crawler script. The framework never silently replays cookies and never invents content.

## When to use this vs MCP scrape tools

Use this skill when:

- The source is a known medical / academic backend (any in the catalogue above)
- A DOI / PMID / PMC ID is the input
- The result needs the standardised `AcquisitionResult` shape for downstream pipelines
- USyd OpenAthens auth is required

Use Brightdata / Firecrawl / Tavily / Exa MCP tools instead when:

- The source is a public web page outside the medical / academic catalogue
- You need SERP search rather than direct retrieval
- You need anti-bot bypass for a non-medical site
- Semantic search across the wider web is the goal

## Concurrency model

`fetch_batch` uses `asyncio` plus a `Semaphore` to cap simultaneous fetches. Each backend's synchronous `fetch()` is wrapped in `asyncio.to_thread` so a single blocking source does not stall the loop. Per-source rate limits are the backend's own responsibility; the framework caps wall-clock concurrency only.

## Pointers

- Canonical source: `~/claude-library/acquisition-client/`
- CLI shim: `~/tools/bin/acquisition`
- Compat shim (legacy import path): `~/projects/personal/scrape-med-references/src/acquisition/`
- Auto-loaded rule: `~/.claude/rules/medical-acquisition.md`
- Per-project fragment for new projects: `~/claude-library/claude-md-fragments/medical-acquisition.md`
