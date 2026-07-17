---
name: godot-suite-orchestrator
description: Use when the user asks for Godot work by "role" (e.g. "act as the Physics Specialist", "I need the Steam Publishing role", "as the Game Designer") or for multi-role game features. Routes each of the 25 roles to the correct skill/agent. Trigger on role names or "which godot skill for X".
---

# godot-suite-orchestrator

Maps the user's 25 game-dev roles to the concrete skill or agent that does the
work. The authoritative table lives in
`${CLAUDE_PLUGIN_ROOT}/skills/godot-suite-orchestrator/references/roles.md`
(also available at the skill-local `references/roles.md`).

## How to route
1. Identify the role the user is asking for (match against the 25 in the table).
2. Route to its destination: a GodotPrompter skill (GP), a superpowers skill (SP),
   or a godot-suite skill (GS). Invoke that skill / dispatch that agent.
3. For work spanning multiple roles, dispatch the `godot-producer` agent to plan
   and delegate role-by-role.
4. If the role is ambiguous or unmapped, ask the user to clarify — do not guess.

## Routing table
(Load `${CLAUDE_PLUGIN_ROOT}/skills/godot-suite-orchestrator/references/roles.md`
— it enumerates all 25 roles and destinations, including the partials: Inventory,
Quest, Documentation Generator.)

Gap roles handled by godot-suite's own skills:
- Steam Publishing → `steam-publishing`
- Game Designer → `game-design-gdd`
- Asset Pipeline → `asset-pipeline-ai`
- Console Porting → `console-porting`
