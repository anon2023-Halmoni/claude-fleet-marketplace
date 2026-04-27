# Claude Fleet Marketplace

Personal Claude Code plugin marketplace for the fw16 plus gx10 fleet. Hosts plugins shared across 35-plus projects under `~/projects/personal/`.

## Layout

```
claude-fleet-marketplace/
  .claude-plugin/
    marketplace.json          # marketplace manifest (Anthropic spec)
  plugins/
    smp-kat-tools/            # the first published plugin
      .claude-plugin/
        plugin.json
      skills/                 # 4 skills
      agents/                 # 2 specialised subagents
      commands/               # 3 slash commands
  docs/
    installation.md
    skills-reference.md
    changelog.md
```

## Active plugin: smp-kat-tools

Version 0.1.0. Bundles:

- 4 skills: `medical-acquisition`, `examiner-feedback-mining`, `distractor-design`, `kfp-failure-bucket`
- 2 subagents: `acquisition-fetcher`, `exam-stem-author`
- 3 slash commands: `/smp-kat-tools:acquire`, `/smp-kat-tools:kfp-stem`, `/smp-kat-tools:refresh-capabilities`
- No bundled hooks. Fleet-wide hooks live at `~/claude-library/hooks/` and stay at user level so they apply to every project, not just plugin installs.

See `plugins/smp-kat-tools/README.md` for the per-plugin docs.

## Add this marketplace

In any Claude Code session:

```
/plugin marketplace add anon2023-Halmoni/claude-fleet-marketplace
/plugin install smp-kat-tools@fleet
```

Or declare it in `~/.claude/settings.json` (user scope) so every session loads it:

```json
{
  "extraKnownMarketplaces": {
    "fleet": {
      "source": {
        "source": "github",
        "repo": "anon2023-Halmoni/claude-fleet-marketplace"
      }
    }
  }
}
```

For project-scope distribution (auto-prompts teammates to install when they trust the project folder), use `.claude/settings.json` at the project root with the same shape.

## Version policy

- `version` in `plugin.json` bumps on any change to public API: skill description, command surface, agent contract.
- Patch (0.1.x) for new content within an existing skill, slash-command body changes, doc fixes.
- Minor (0.x.0) for new skills, new commands, new agents, new bundled hooks.
- Major (x.0.0) for breaking changes to the contract: removed skills, renamed commands, changed return shapes.

The git commit SHA is the fallback identifier when `version` is absent. With `version` set, users only get an update when the field bumps.

## Testing locally before push

```
claude --plugin-dir ./plugins/smp-kat-tools
```

Inside the session: `/smp-kat-tools:acquire 10.31128/AJGP-04-23-6803` to smoke-test.

Validate before pushing:

```
claude plugin validate plugins/smp-kat-tools
claude plugin validate .
```

## Hook safety

GitHub issue #15897 documents that PreToolUse `updatedInput` is broken when multiple hooks match the same tool. This marketplace deliberately does not bundle plugin-level hooks. Fleet hooks live at the user level and are scoped per-project as needed. If a future plugin needs a hook, it will be a single matcher per tool, no stacking.

## Crosswalk to claude-library

| In this marketplace | Mirrors in claude-library |
| --- | --- |
| `plugins/smp-kat-tools/skills/medical-acquisition/` | `~/.claude/skills/medical-acquisition/` (standalone copy) |
| `plugins/smp-kat-tools/skills/examiner-feedback-mining/` | new, plugin-only |
| `plugins/smp-kat-tools/skills/distractor-design/` | new, plugin-only |
| `plugins/smp-kat-tools/skills/kfp-failure-bucket/` | new, plugin-only |
| `plugins/smp-kat-tools/agents/` | new, plugin-only |
| `plugins/smp-kat-tools/commands/` | new, plugin-only |

The standalone `~/.claude/skills/medical-acquisition/` copy is kept so projects without the plugin still get the skill from user scope.

## Owner

Personal fleet under `anon2023-Halmoni`. HTTPS only. No external contributors expected.
