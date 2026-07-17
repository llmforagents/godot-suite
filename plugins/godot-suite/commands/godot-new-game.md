---
description: Scaffold a new Godot 4.x project with recommended structure and run godot-suite-setup.
---

Scaffold a new Godot 4.x game project:

1. Create the recommended folder layout under the current project:
   `res://scenes/`, `res://scripts/`, `res://assets/{sprites,audio,fonts}/`,
   `res://ui/`, `res://autoload/`, `res://addons/`.
2. Create a minimal `project.godot` if none exists (Godot 4.x config version).
3. Invoke the `godot-suite-setup` skill to bootstrap GodotPrompter, the MCP server,
   and offer recommended addons.
4. Optionally invoke `game-design-gdd` to start a `docs/GDD.md`.

Report the created structure and next steps.
