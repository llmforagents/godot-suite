# Recommended Godot addons (pinned)

`godot-suite-setup` offers to install these into a project's `res://addons/`.
Versions are pinned; record installed version + date in the project's
`addons.lock.md`. Install only with user confirmation; never overwrite silently.

| Addon | Pinned version | Purpose | Source |
|-------|----------------|---------|--------|
| LimboAI | 1.8.0 | Behavior trees + HSM | github.com/limbonaut/limboai |
| Dialogic 2 | 2.x (latest 2.x) | Dialogue system | github.com/dialogic-godot/dialogic |
| GdUnit4 | latest 4.x | Unit testing | github.com/MikeSchulze/gdUnit4 |
| netfox | latest | Multiplayer netcode | github.com/foxssake/netfox |
| Terrain3D | latest | Large 3D terrain | github.com/TokisanGames/Terrain3D |
| Aseprite Wizard | latest | Aseprite→Godot import | github.com/viniciusgerevini/godot-aseprite-wizard |
| Debug Draw 3D | latest | Runtime debug drawing | github.com/DmitriySalnikov/godot_debug_draw_3d |

Note: Godot 4.4+ absorbed Jolt physics into core (Project Settings toggle) —
no addon needed. Verify each version against the addon's releases at install time.
