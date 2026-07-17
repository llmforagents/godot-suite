---
name: asset-pipeline-ai
description: Use for the game art/audio asset pipeline — generating sprites/textures with AI, automating Aseprite→Godot import, building atlases, and setting naming/compression conventions. Trigger on "asset pipeline", "generate sprite/texture", "aseprite import", "texture atlas", "asset naming".
---

# asset-pipeline-ai

Build and run a repeatable asset pipeline for a Godot 4.x game.

## Capabilities
1. **AI generation:** if the `mcp__image-gen__generate_image` tool is available,
   generate placeholder/production sprites and textures from prompts; otherwise
   guide the user to an external tool.
2. **Aseprite → Godot:** use the Aseprite Wizard addon (see
   `${CLAUDE_PLUGIN_ROOT}/skills/godot-suite-setup/references/addons.md`) to
   import `.aseprite` with tags/slices as animations.
3. **Atlases & import:** build texture atlases; set import presets (filter, mipmaps,
   compression) appropriate to 2D pixel-art vs 3D.
4. **Conventions:** enforce folder layout (`res://assets/<type>/<name>`) and naming.

## Rules
- Document licensing limits of AI-generated assets; do not claim rights the user
  doesn't have.
- Keep generation reproducible: record prompts/seeds alongside outputs.
