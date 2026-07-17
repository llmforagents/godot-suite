# 25 Roles → Skill / Agent routing

Source of truth for `godot-suite-orchestrator`. Every role routes to a
GodotPrompter skill (GP), a superpowers skill (SP), a godot-suite skill (GS),
or a combination.

| # | Role | Destination | Note |
|---|------|-------------|------|
| 1 | Godot Architect | GP `godot-game-architect` agent | System/scene architecture |
| 2 | GDScript Senior Engineer | GP GDScript skill | Idioms, typing |
| 3 | Scene Composer | GP scene/composition skill | Node trees |
| 4 | UI Designer | GP UI skill | Control nodes, themes |
| 5 | Gameplay Programmer | GP gameplay skill | Core mechanics |
| 6 | AI Programmer | GP LimboAI/Beehave skill | Behavior trees / FSM |
| 7 | Physics Specialist | GP physics skill | Bodies, Jolt |
| 8 | Multiplayer Engineer | GP multiplayer skill (netfox) | Networking |
| 9 | Save System Engineer | GP save/persistence skill | Serialization |
| 10 | Inventory Engineer | GP gameplay skill | PARTIAL — no dedicated skill; document coverage |
| 11 | Quest System Engineer | GP gameplay skill | PARTIAL — no dedicated skill; document coverage |
| 12 | Dialogue Writer | GP Dialogue Manager/Dialogic skill | Branching dialogue |
| 13 | Animation Controller | GP animation skill | AnimationTree/Player |
| 14 | Shader Expert | GP shader skill | Godot shading language |
| 15 | Audio Integration | GP audio skill | Buses, playback |
| 16 | Performance Profiler | GP optimization skill | Profiling, budgets |
| 17 | Bug Hunter | SP `systematic-debugging` + GP `godot-debugging` | Combined |
| 18 | Test Generator | GP/Randroids GdUnit4 testing skill | Unit/integration |
| 19 | Export & CI | GP export-pipeline skill | Builds, CI/CD |
| 20 | Steam Publishing | GS `steam-publishing` | NEW gap skill |
| 21 | Mobile Optimization | GP mobile shipping skill | Android/iOS |
| 22 | Console Porting | GS `console-porting` | NEW gap skill |
| 23 | Asset Pipeline | GS `asset-pipeline-ai` | NEW gap skill |
| 24 | Documentation Generator | SP/generic docs skill | PARTIAL — document destination + limits |
| 25 | Game Designer | GS `game-design-gdd` | NEW gap skill |
