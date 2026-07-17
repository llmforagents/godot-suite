# godot-suite

Curated Godot 4.x indie game development suite for Claude Code.

**What it is:** a meta-plugin that composes the existing Godot AI ecosystem
(GodotPrompter skills + a Godot MCP server + pinned community addons) and adds
four gap skills (Steam publishing, game design/GDD, AI asset pipeline, console
porting) plus orchestration for 25 game-dev roles.

**Requires:** Godot 4.x, Node.js (for the MCP server via `npx`).

## Install

**Option A — hosted catalog (Cloudflare Pages):**

```
claude plugin marketplace add https://godot-suite.llm4agents.com/marketplace.json
claude plugin install godot-suite@godot-suite
```

**Option B — straight from GitHub (clones the whole repo):**

```
claude plugin marketplace add llmforagents/godot-suite
claude plugin install godot-suite@godot-suite
```

### Inside Claude Code (slash commands)

You can also install without leaving a Claude Code session. Type these as slash
commands, then reload — no restart needed:

```
/plugin marketplace add https://godot-suite.llm4agents.com/marketplace.json
/plugin install godot-suite@godot-suite
/reload-plugins
```

Or from GitHub: `/plugin marketplace add llmforagents/godot-suite`.
(`/plugin` with no arguments opens the interactive plugin manager instead.)

### On Windows (GUI surfaces)

**Claude Code desktop app:** click the **`+`** button next to the prompt box →
**Plugins** → **Add plugin** to open the plugin browser; use **Manage plugins**
to enable/disable/uninstall. (Plugins are not available in WSL sessions of the
desktop app — use the terminal CLI there.)

**VS Code extension:** run `/plugins` (plural) to open the Manage plugins dialog.
In the **Marketplaces** tab add `https://godot-suite.llm4agents.com/marketplace.json`
(or `llmforagents/godot-suite`), then install `godot-suite` from the **Plugins** tab.

**JetBrains IDEs:** no GUI plugin manager — use the integrated terminal with the
`claude plugin marketplace add …` / `claude plugin install …` commands above.

**Windows prerequisites:** Godot 4.x and Node.js on `PATH` (the MCP server runs
via `npx`; the plain `"command": "npx"` form in `.mcp.json` works natively on
Windows once Node is installed — no `cmd /c` wrapper needed).

### After installing

Run the `godot-suite-setup` skill in a Godot 4.x project to bootstrap the
environment (GodotPrompter, MCP server, recommended addons).

## Publishing

The plugin lives in a public GitHub repo; the marketplace catalog
(`public/marketplace.json`) is served as a static file from Cloudflare Pages at
`godot-suite.llm4agents.com`. Because a URL-based marketplace only downloads the
catalog JSON (not plugin files), the catalog references the plugin via a
`git-subdir` source pointing back at GitHub. See `docs/publishing.md` for the
full deploy flow.

## Design & plan

See `docs/superpowers/specs/` and `docs/superpowers/plans/`.
