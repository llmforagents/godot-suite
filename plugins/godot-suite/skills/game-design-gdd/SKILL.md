---
name: game-design-gdd
description: Use for game design work (not code) — writing/maintaining a Game Design Document, defining core loops, economy, progression curves, balancing, and meta-progression. Trigger on "game design", "GDD", "core loop", "balance", "progression", "game economy".
---

# game-design-gdd

Produce and maintain design docs for a Godot game. Output/updates a `docs/GDD.md`
in the game project.

## What to cover
1. **Pillars & core loop:** the 15-second and session loops; what the player does
   and why it's fun.
2. **Systems:** mechanics, controls, camera, feedback.
3. **Economy & progression:** resources, sinks/sources, curves, unlock pacing.
4. **Balancing:** define target numbers and how they'll be tuned/tested.
5. **Scope:** MVP vs stretch; cut ruthlessly (YAGNI).

## Rules
- This is design, not implementation. Hand off code work to the orchestrator's
  gameplay/architecture roles.
- Keep the GDD living: update it as decisions change; note open questions explicitly.
