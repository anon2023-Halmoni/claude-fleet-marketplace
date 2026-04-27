# Changelog

## 0.1.0

Initial marketplace plus first plugin.

### Added

- Marketplace manifest at `.claude-plugin/marketplace.json` (Anthropic spec compliant)
- Plugin `smp-kat-tools` at `plugins/smp-kat-tools/`
- 4 skills: `medical-acquisition`, `examiner-feedback-mining`, `distractor-design`, `kfp-failure-bucket`
- 2 subagents: `acquisition-fetcher`, `exam-stem-author`
- 3 slash commands: `/smp-kat-tools:acquire`, `/smp-kat-tools:kfp-stem`, `/smp-kat-tools:refresh-capabilities`

### Notes

- No bundled plugin hooks. Fleet-wide hooks remain at `~/claude-library/hooks/` and apply via user-scope `settings.json`. Per GitHub issue #15897 (PreToolUse `updatedInput` broken when multiple hooks match), this plugin does not stack matchers.
- `medical-acquisition` skill is a copy of the standalone version at `~/.claude/skills/medical-acquisition/`. The standalone copy is retained so projects that do not install this plugin still get the skill at user scope.
