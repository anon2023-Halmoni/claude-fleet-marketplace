# smp-kat-tools

Medical-education exam-prep toolkit for the SMP KAT study project plus any RACGP / ACRRM / AMC exam-adjacent work across the fleet.

## What is bundled

### Skills (4)

| Skill | When it activates |
| --- | --- |
| `medical-acquisition` | DOI, PMID, PMC ID, RACGP exam cycle, AU clinical reference fetch |
| `examiner-feedback-mining` | "what do RACGP examiners flag", per-college pitfall analysis |
| `distractor-design` | author or evaluate MCQ / KFP distractors against the 72-class taxonomy |
| `kfp-failure-bucket` | route a wrong KFP answer into one of 8 named failure buckets |

Each skill is model-invoked. Claude picks the right one based on the task description, no manual invocation required.

### Subagents (2)

| Subagent | Purpose |
| --- | --- |
| `acquisition-fetcher` | Pre-loads `medical-acquisition`, runs fetch specs, returns `AcquisitionResult` JSON |
| `exam-stem-author` | Pre-loads all 4 skills, authors KFP / MCQ stems with marking key plus failure analysis |

Use these when a parent agent needs the work isolated in its own context window (long fetches, long stem authoring runs).

### Slash commands (3)

| Command | What it does |
| --- | --- |
| `/smp-kat-tools:acquire <spec>` | one-shot fetch via `~/tools/bin/acquisition` |
| `/smp-kat-tools:kfp-stem <topic>` | author a full KFP stem with marking key plus failure-bucket map |
| `/smp-kat-tools:refresh-capabilities` | run `~/claude-library/scripts/capability-check.py --fix` and report |

Note the namespace prefix `smp-kat-tools:`. This is automatic for plugin commands per Anthropic spec.

### Hooks

None at plugin level. Fleet hooks live at `~/claude-library/hooks/` and apply across all projects via user-scope `~/.claude/settings.json`. Honouring GitHub issue #15897, this plugin does not stack matchers.

## Install

From this marketplace:

```
/plugin install smp-kat-tools@fleet
/reload-plugins
```

Or test locally:

```
claude --plugin-dir <path-to-this-plugin>
```

## Dependencies on the host machine

The medical-acquisition skill expects:

- `~/tools/bin/acquisition` (CLI shim) installed
- `~/claude-library/acquisition-client/` Python package available
- USyd OpenAthens credentials prompted per run (the framework handles the prompt)

The slash command `/smp-kat-tools:refresh-capabilities` expects:

- `~/claude-library/scripts/capability-check.py`
- `~/.claude/rules/capability-manifest.json`

If a dependency is missing, the skill or command will surface a clear error and stop. It will not silently fall back to web scraping.

## Version

0.1.0 - initial release.
