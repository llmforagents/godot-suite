# godot-suite

Curated Godot 4.x indie game development suite for Claude Code.

**What it is:** a meta-plugin that composes the existing Godot AI ecosystem
(GodotPrompter skills + a Godot MCP server + pinned community addons) and adds
four gap skills (Steam publishing, game design/GDD, AI asset pipeline, console
porting) plus orchestration for 25 game-dev roles.

**Requires:** Godot 4.x, Node.js (for the MCP server via `npx`).

## Install

```
claude plugins marketplace add <this-repo-url>
claude plugins install godot-suite@godot-suite
```

Then run the `godot-suite-setup` skill in a Godot 4.x project to bootstrap the
environment (GodotPrompter, MCP server, recommended addons).

## Design & plan

See `docs/superpowers/specs/` and `docs/superpowers/plans/`.
