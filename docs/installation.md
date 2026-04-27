# Installation

Three ways to install this marketplace plus its plugins.

## Option 1: GitHub shorthand (recommended)

Inside any Claude Code session:

```
/plugin marketplace add anon2023-Halmoni/claude-fleet-marketplace
/plugin install smp-kat-tools@fleet
```

Or from the CLI without an interactive session:

```bash
claude plugin marketplace add anon2023-Halmoni/claude-fleet-marketplace
```

## Option 2: User-scope settings entry

Edit `~/.claude/settings.json` to register the marketplace at user scope so every Claude Code session picks it up automatically:

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

Merge this block into the existing `settings.json` rather than replacing it.

After registering, install plugins from the marketplace:

```
/plugin install smp-kat-tools@fleet
/reload-plugins
```

## Option 3: Project-scope distribution

To prompt teammates (or your future self on a fresh machine) to install the marketplace when they trust a specific project folder, add the same `extraKnownMarketplaces` block to the project's `.claude/settings.json`. Claude Code will prompt on first trust.

## Pin to a tag or branch

To pin to a specific release ref:

```
/plugin marketplace add anon2023-Halmoni/claude-fleet-marketplace@v0.1.0
```

Or in settings:

```json
{
  "extraKnownMarketplaces": {
    "fleet": {
      "source": {
        "source": "github",
        "repo": "anon2023-Halmoni/claude-fleet-marketplace",
        "ref": "v0.1.0"
      }
    }
  }
}
```

## Update

```
/plugin marketplace update fleet
```

Or refresh all marketplaces:

```
/plugin marketplace update
```

## Validate locally before pushing changes

```
claude plugin validate plugins/smp-kat-tools
claude plugin validate .
```

The validator checks `plugin.json`, `marketplace.json`, skill / agent / command frontmatter, and `hooks/hooks.json` (when present).

## Test locally with --plugin-dir

```
claude --plugin-dir ./plugins/smp-kat-tools
```

Inside the session: try `/smp-kat-tools:acquire 10.31128/AJGP-04-23-6803`. The `--plugin-dir` flag overrides any installed copy of the same plugin name for that session.

## Private repository auth (future)

If this marketplace ever moves to private:

```bash
export GITHUB_TOKEN=ghp_xxxxxxxxxxxxxxxxxxxx
```

The token is required for background auto-updates. Manual installs use existing git credential helpers.
