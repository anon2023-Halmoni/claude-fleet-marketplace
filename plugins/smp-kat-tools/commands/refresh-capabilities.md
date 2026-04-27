---
description: Run the fleet capability-check script and report what is current, missing, or out of date. Drives ~/claude-library/scripts/capability-check.py with --fix.
---

# /refresh-capabilities

Audit and refresh the local capability manifest at `~/.claude/rules/capability-manifest.json` against what is actually installed on this fw16 plus what is reachable on gx10 via Tailscale.

## Execution

```bash
python3 ~/claude-library/scripts/capability-check.py --fix
```

If the script does not exist, fall back to:

```bash
python3 ~/claude-library/scripts/capability-check.py --report
```

If neither exists, print the missing-script error and stop.

## Output

Report three sections:

1. **Current and healthy**: capabilities verified present and reachable
2. **Drifted or stale**: manifest entries whose live state differs from what is recorded (version bump, port change, service down)
3. **Missing**: manifest entries with no live evidence on this machine or gx10

For each drifted or missing entry, print:

- the manifest path of the entry
- what the manifest says
- what the live check found
- the suggested fix command (if `--fix` was applied, what was applied)

End with a one-line summary: `<n> healthy, <n> drifted, <n> missing`.

## Pointers

- Manifest: `~/.claude/rules/capability-manifest.json`
- Capability inventory rule: `~/.claude/rules/capabilities.md`
- gx10 health probe: `~/claude-library/hooks/gx10-health-probe.sh` (runs at SessionStart)
