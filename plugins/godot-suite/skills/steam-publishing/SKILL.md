---
name: steam-publishing
description: Use for shipping a Godot game on Steam — GodotSteam SDK init, achievements, stats, cloud saves, Steam Workshop, rich presence, and uploading builds with steamcmd. Trigger on "steam", "godotsteam", "achievements", "cloud saves", "steam workshop", "publish to steam".
---

# steam-publishing

Ship a Godot 4.x game on Steam via GodotSteam. Core steps here; recipes in
`references/godotsteam-recipes.md`.

## Prerequisites (user-supplied — never hardcoded here)
- A Steamworks account, an **App ID**, and depot config from the Steamworks site.
- The GodotSteam GDExtension (or a GodotSteam-enabled build) installed in the project.
- `steamcmd` installed for uploads.

## Workflow
1. **Init the SDK** on game start with the App ID (read from local config/env, not
   committed). Guard against init failure and run without Steam in dev.
2. **Achievements & stats:** define them on Steamworks, then set/clear from GDScript.
3. **Cloud saves:** use Steam Remote Storage or Auto-Cloud; keep local saves as fallback.
4. **Workshop / rich presence:** optional; see the GodotSteam docs for these APIs.
5. **Upload builds:** configure depots and run `steamcmd` with a build script.

## Rules
- No App IDs, keys, or credentials in the repo — instruct the user to provide them
  via local config/env and document where to get each.
- Provide a checklist; defer exhaustive API detail to references.

See `references/godotsteam-recipes.md`.
