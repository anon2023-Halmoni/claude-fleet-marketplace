---
description: Fetch a medical reference via the unified acquisition framework. Argument is a DOI, PMID, PMC ID, URL, or JSON spec. Routes through ~/tools/bin/acquisition.
---

# /acquire $ARGUMENTS

Fetch the medical reference described by the argument.

Use the medical-acquisition skill (preloaded by this command) to dispatch the spec. Do not fall back to MCP scrape tools for any source the framework covers.

## Argument forms

- Bare DOI: `10.31128/AJGP-04-23-6803`
- PubMed: `PMID:38825755`
- PMC: `PMC10765432`
- URL: `https://www1.racgp.org.au/ajgp/2024/may/...`
- JSON spec with backend: `{"cycle":"2024.2","exam_type":"akt"} --backend racgp_exam`

## Execution

Run the CLI shim:

```bash
~/tools/bin/acquisition fetch "$ARGUMENTS"
```

For JSON specs that need an explicit backend, parse the trailing `--backend NAME` from the argument and pass it through.

## Output

Print a one-paragraph summary including:

- tier (P0 / P1 / P2 / P3)
- source_type
- canonical URL
- success / error
- fallback_used (if any)
- first 1000 chars of `content_md`

If `success=False`, surface the error verbatim. Do not silently retry.

## Hook-aware

This command never calls WebSearch or WebFetch (hook-blocked). It never calls MCP scrape tools for sources the framework covers.
