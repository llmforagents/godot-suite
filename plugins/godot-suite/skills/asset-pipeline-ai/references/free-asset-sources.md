# Free game-asset sources (sprites, tilesets, audio)

Curated open-license providers for getting a prototype playable fast. Orientation:
2D / pixel-art / 8-bit, but most apply broadly. **Always verify the per-asset
license** — "free" ≠ "CC0". See License hygiene at the bottom.

## Quick chooser

- Need a whole cohesive pack, zero attribution, fastest path → **Kenney** (CC0).
- Need a specific character/tileset with animations → **itch.io** (filter CC0) or
  **OpenGameArt**.
- Need standardized animated humanoid characters → **LPC generator** (CC-BY-SA/GPL,
  attribution required).
- Need retro SFX with no files to ship → generate at runtime with **jsfxr/bfxr**.

## Sprite / tileset providers

| Source | License | Best for | How to get it |
|--------|---------|----------|---------------|
| **Kenney.nl** (kenney.nl/assets) | **CC0** (public domain, no attribution) | Cohesive packs: Pixel Platformer (18×18 tiles), Micro Roguelike / Tiny Dungeon (8×8, 16×16), UI, audio | Download the `.zip` per pack from the asset page; extract PNGs/tilesheets. Agents already know Kenney's folder/animation naming — lean on it. |
| **itch.io** (itch.io/game-assets/free) | **Mixed** — filter `Tag: Pixel Art` + `License: Creative Commons` / free; verify each | Full player+enemy+tileset packs: e.g. **Sunny Land**, **Treasure Hunters** | Free/name-your-price download on the asset page (may need itch account). Read the asset's license text on the page. |
| **OpenGameArt.org** | **Mixed** — CC0, CC-BY, CC-BY-SA, GPL. **Filter by license** | Individual sprites, tilesets, and chiptune music | Filter search by license; download the file; copy required attribution. |
| **Universal LPC Spritesheet Generator** (github.com/LiberatedPixelCup / sanderfrenken.github.io/Universal-LPC-Spritesheet-Character-Generator) | **CC-BY-SA 3.0 / GPL 3.0** (dual) — **attribution + share-alike required** | Standardized animated humanoids: walk/run/jump/attack, aligned frames, PNG export | Configure a character in the web generator → export spritesheet PNG. Save the generated credits list — LPC layers have per-author attribution. |

## Curated GitHub collections

| Repo | What it is | License note |
|------|-----------|--------------|
| `madjin/awesome-cc0` | Curated **CC0** resources; the 2D section has 8-bit spritesheets safe for commercial use, no attribution | CC0 (still spot-check individual links) |
| `teamgravitydev/gamedev-free-resources` | Curated 2D assets, sprites, tilesets, SFX across many repos | Licenses **vary per linked source** — check each |

## Audio / SFX (8-bit / chiptune)

| Source | Approach | Notes |
|--------|----------|-------|
| **jsfxr** (github.com/chr15m/jsfxr) / **bfxr** | **Generate SFX in code, no files shipped.** Ask the agent to produce a script that synthesizes retro SFX at runtime via the jsfxr library, or export tiny `.wav`s from the web tool | Generated sounds are yours — no license restriction. Ideal for jump/coin/hit/laser blips. For Godot, export `.wav` from the tool and import, or reproduce params in an `AudioStreamGenerator`. |
| **OpenGameArt.org** → Music / Chiptune | Full tracks and loops; **filter by license** | Attribution often required (CC-BY) — record it. |

## Getting assets into Godot 4.x

1. Download into `res://assets/<type>/<name>/` (keep the pack's own subfolders).
2. For a **spritesheet** PNG: import as texture, then either
   - create an `AtlasTexture` per frame, or
   - build a `SpriteFrames` resource (for `AnimatedSprite2D`) slicing by the grid, or
   - use a `TileSet` (for tilesets) with the tile size matching the grid.
   Pixel-art import: set texture filter to **Nearest** and disable mipmaps/compression.
3. If the source is `.aseprite`, prefer the **Aseprite Wizard** addon (tags → animations).
4. Record provenance (see below).

## The spritesheet-grid rule (why agents get this wrong)

LLMs slice spritesheets wrong when the grid is unstated. **Always pin the grid
before writing loader code.** Reusable prompt shape:

> "Spritesheet is `<W>×<H>` px per frame. Row 0 = run (frames 0–5), row 1 = jump
> (frames 0–2). Slice into `<W>×<H>` tiles and build the animation states."

For Godot: translate that into a `SpriteFrames` (frame size `<W>×<H>`, one animation
per row, the stated frame ranges) or an `AtlasTexture` region per frame.

## License hygiene (do this every time)

- **Confirm the license on the asset's own page**, not the site's homepage. "Free
  download" is not a license.
- **CC0 / public domain** (Kenney, `awesome-cc0`) → no attribution, commercial OK.
- **CC-BY** → attribution required. **CC-BY-SA / GPL** (LPC) → attribution **and**
  derivative must stay under the same license — a real obligation for a shipped game.
- Keep a project `CREDITS.md`: for every attribution-required asset record
  **author, source URL, license, and any changes made**.
- When in doubt, prefer CC0 for anything you might sell.
