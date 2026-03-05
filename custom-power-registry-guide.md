# Custom Power Registry — Quickstart Guide

Host your own Kiro Powers registry so your team sees internal powers in the "Recommended" tab, right alongside (or instead of) the official ones.

## 1. Create your registry JSON

Host a JSON file on any HTTPS endpoint your developers can reach (S3, GitHub Pages, internal CDN, etc.):

```json
{
  "schemaVersion": "1.0.0",
  "powers": [
    {
      "name": "my-power",
      "displayName": "My Internal Power",
      "author": "Platform Team",
      "description": "Short description shown on the tile",
      "iconUrl": "https://internal.example.com/icons/my-power.png",
      "repositoryUrl": "https://github.com/my-company/my-power",
      "license": "MIT",
      "repositoryCloneUrl": "https://github.com/my-company/my-power.git",
      "pathInRepo": "",
      "repositoryBranch": "main"
    }
  ]
}
```

The official registry is public at `https://prod.download.desktop.kiro.dev/powers/default_registry.json` — merge entries from it into yours if you want to keep official powers visible.

### Registry schema

| Field | Required | Notes |
|-------|----------|-------|
| `schemaVersion` | ✅ | `"1.0.0"` |
| `powers[].name` | ✅ | Unique ID, lowercase alphanumeric + hyphens |
| `powers[].author` | ✅ | |
| `powers[].description` | ✅ | Short, shown on tile |
| `powers[].iconUrl` | ✅ | 512×512 PNG recommended |
| `powers[].repositoryUrl` | ✅ | Public URL for "View Source" |
| `powers[].license` | ✅ | SPDX identifier, can be `""` |
| `powers[].repositoryCloneUrl` | ✅ | Git clone URL (HTTPS or SSH) |
| `powers[].pathInRepo` | ✅ | Path to power dir within repo, `""` for root |
| `powers[].displayName` | | Human-readable name |
| `powers[].repositoryBranch` | | Defaults to `main` |
| `powers[].launchUrl` | | `kiro://kiro.powers/add?name=...` deep link |

## 2. Point Kiro at your registry

Set `kiroAgent.powersRecommendationUrl` to your hosted JSON URL. Kiro fetches it on launch and re-caches every 5 minutes.

**Per-project** (committed to git, applies to everyone who opens the repo):

`.vscode/settings.json`:
```json
{
  "kiroAgent.powersRecommendationUrl": "https://internal.example.com/kiro/registry.json"
}
```

**Per-machine** (global, applies to all projects):

`Cmd+Shift+P` → "Preferences: Open User Settings (JSON)":
```json
{
  "kiroAgent.powersRecommendationUrl": "https://internal.example.com/kiro/registry.json"
}
```

File location: `~/Library/Application Support/Kiro/User/settings.json` (macOS).
For org-wide rollout, push this file via config management.

## 3. What happens when a developer installs

1. Developer opens Kiro → your URL is fetched → powers appear in the "Recommended" tab
2. Developer clicks "Install" on a power
3. Kiro clones the repo from `repositoryCloneUrl`, checks out `repositoryBranch`, reads `pathInRepo`
4. Files are copied to `~/.kiro/powers/<name>/`
5. MCP servers from the power's `mcp.json` are registered as `power.<name>.<server-name>`

Push a new entry to your registry JSON and developers see it within 5 minutes.

## 4. Build a power

Each power lives in a git repo. Minimum structure:

```
my-power/
├── POWER.md              # Required — docs + frontmatter
├── mcp.json              # Optional — MCP server definitions
└── steering/             # Optional — steering files
    └── guide.md
```

**POWER.md** (frontmatter is required):

```markdown
---
displayName: My Power
description: What this power does
author: Platform Team
keywords:
  - internal
  - tooling
---

# My Power

Documentation content here...
```

**mcp.json** (if your power provides MCP tools):

```json
{
  "mcpServers": {
    "my-server": {
      "command": "uvx",
      "args": ["my-mcp-server@latest"],
      "env": {
        "FASTMCP_LOG_LEVEL": "ERROR"
      }
    }
  }
}
```

## 5. Share install links

Once a power is in your registry, share deep links:

```
kiro://kiro.powers/add?name=my-power
```

Opens the power details panel — one click to install.

Note: GitHub strips `kiro://` links from rendered markdown (it only allows `http`, `https`, and `mailto`). On internal wikis or docs platforms that support custom protocols, you can make clickable badges with [Shields.io](https://shields.io):

```markdown
[![Add to Kiro](https://img.shields.io/badge/Add%20to-Kiro-blue)](kiro://kiro.powers/add?name=my-power)
```

On GitHub, show the link as a code block so users can copy-paste it:

```
kiro://kiro.powers/add?name=python-scaffolder
```

---

## Auto-install via local registry (optional)

The recommended feed requires users to click "Install." If you need powers to install automatically, use a local registry file instead.

Drop a JSON file into `~/.kiro/powers/registries/`:

```json
{
  "name": "My Company Powers",
  "type": "local",
  "powers": [
    {
      "name": "my-power",
      "description": "Internal tooling power",
      "source": {
        "type": "repo",
        "repositoryCloneUrl": "https://github.com/my-company/my-power.git",
        "pathInRepo": "",
        "repositoryBranch": "main"
      },
      "displayName": "My Power",
      "author": "Platform Team",
      "keywords": ["internal"],
      "autoInstall": true
    }
  ]
}
```

Kiro watches that directory with `fs.watch` — no restart needed. Powers with `autoInstall: true` install immediately.

Deliver the file via onboarding script, dotfiles, config management, or a cron job. Note the schema differs from the recommended feed (uses `source` objects instead of flat fields).

If a user manually uninstalls an auto-installed power, it's recorded in `installed.json` under `dismissedAutoInstalls` and won't be re-installed.

### Recommended feed vs. local registry

| | Recommended Feed | Local Registry |
|---|---|---|
| Setup | One setting in `settings.json` | File on disk + delivery mechanism |
| Auto-update | Built-in (5-min cache) | External sync needed |
| Auto-install | No | Yes |
| UI | Shows in "Recommended" tab | Not in Recommended tab |
| Official powers | Replaced (merge manually) | Additive |
| Git-distributable | Yes (`.vscode/settings.json`) | No |

Start with the recommended feed. Add a local registry only if you need auto-install.

---

## Reference

### Registry resolution order

1. `kiro-recommended` — remote feed (configurable URL, 5-min cache)
2. `user-added` — `~/.kiro/powers/registries/user-added.json` (managed by UI)
3. Custom registries — other `*.json` files in `~/.kiro/powers/registries/`

### Key paths

| Path | Purpose |
|------|---------|
| `~/.kiro/powers/registries/*.json` | Registry files |
| `~/.kiro/powers/<name>/` | Installed power files |
| `~/.kiro/powers/installed.json` | Installation tracking |
| `KIRO_POWERS_HOME` env var | Override base path |
