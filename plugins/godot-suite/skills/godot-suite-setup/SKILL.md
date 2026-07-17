---
name: godot-suite-setup
description: Use to bootstrap or verify a Godot 4.x project's Claude Code environment — installs/checks GodotPrompter, the Godot MCP server, and recommended addons. Trigger on "set up godot", "godot suite setup", "bootstrap godot environment", "install godot skills/addons".
---

# godot-suite-setup

Idempotent bootstrap for a professional Godot 4.x dev environment. Safe to run
repeatedly — each step is a no-op if already satisfied. Verify every step before
reporting success (evidence before assertions).

## Procedure

1. **Detect Godot.** Run `godot --version` (or `godot4 --version`). If missing,
   stop and tell the user to install Godot 4.x (do not auto-install). If the
   version is 3.x, STOP with: "godot-suite targets Godot 4.x; 3.x APIs differ."
2. **Install GodotPrompter (composition).** Verify the exact install commands
   against its current README first — see `references/godotprompter.md`. Run them;
   if already installed, no-op and report the version. If install fails or the
   marketplace/plugin name changed, report the repo URL instead of failing silently.
3. **Verify MCP.** Confirm `npx` is available (`npx --version`). The plugin ships
   `.mcp.json` pointing at `@coding-solo/godot-mcp`. Tell the user to run `/mcp`
   and confirm `godot` shows a green dot with tools > 0. For runtime/screenshots,
   point to `${CLAUDE_PLUGIN_ROOT}/skills/godot-suite-setup/references/mcp-upgrade.md`.
4. **Install addons (opt-in).** For each addon in `references/addons.md`, ask the
   user before installing into `res://addons/`. Never overwrite an existing addon
   without asking. Record installed addon + version + date in the project's
   `addons.lock.md`.
5. **Report.** Summarize what is installed, versions, and what is still missing.

## Rules
- Idempotent; never overwrite without asking; never write outside the project or
  `~/.claude` without telling the user; never commit secrets.
- If no Godot project is active (`project.godot` absent in cwd), skip addon steps
  and say so.

See `references/` for install command details and the addon table.
