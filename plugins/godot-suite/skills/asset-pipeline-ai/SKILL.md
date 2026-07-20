---
name: asset-pipeline-ai
description: Use for the game art/audio asset pipeline — sourcing FREE assets (sprites, tilesets, SFX) from open-license providers (Kenney, itch.io, OpenGameArt, LPC), generating sprites/textures with AI, automating Aseprite→Godot import, building atlases, and setting naming/compression conventions. Trigger on "asset pipeline", "free assets", "sprites", "tilesets", "kenney", "opengameart", "download game assets", "generate sprite/texture", "aseprite import", "texture atlas", "asset naming", "chiptune/SFX".
---

# asset-pipeline-ai

Build and run a repeatable asset pipeline for a Godot 4.x game: source free assets,
or generate them, then import them cleanly.

## Capabilities
1. **Free asset sourcing:** find and download open-license sprites, tilesets, and
   SFX from known providers (Kenney = CC0, itch.io, OpenGameArt, LPC generator).
   Best for getting a prototype playable fast. See
   `${CLAUDE_PLUGIN_ROOT}/skills/asset-pipeline-ai/references/free-asset-sources.md`.
2. **AI generation:** if the `mcp__image-gen__generate_image` tool is available,
   generate placeholder/production sprites and textures from prompts; otherwise
   guide the user to an external tool.
3. **Aseprite → Godot:** use the Aseprite Wizard addon (see
   `${CLAUDE_PLUGIN_ROOT}/skills/godot-suite-setup/references/addons.md`) to
   import `.aseprite` with tags/slices as animations.
4. **Atlases & import:** build texture atlases; set import presets (filter, mipmaps,
   compression) appropriate to 2D pixel-art vs 3D.
5. **Conventions:** enforce folder layout (`res://assets/<type>/<name>`) and naming.

## Rules
- **Verify each asset's license before use.** Not everything free is CC0 — Kenney
  is CC0 (no attribution), but LPC is CC-BY-SA / GPL (attribution + share-alike),
  and itch.io / OpenGameArt are mixed per-asset. Record license + author + source
  URL for every downloaded asset; keep a project `CREDITS.md` for attribution-required
  assets. See the license-hygiene section in the references file.
- Document licensing limits of AI-generated assets; do not claim rights the user
  doesn't have.
- Keep generation reproducible: record prompts/seeds alongside outputs.
- When importing a spritesheet, always establish the **grid** first (frame WxH, rows,
  columns, which row = which animation) — the references file has a reusable prompt.
