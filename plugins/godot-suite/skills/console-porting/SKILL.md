---
name: console-porting
description: Use for porting a Godot game to consoles (Switch, PlayStation, Xbox) via W4 Games — certification requirements, per-platform input/guidelines, and build management. Trigger on "console port", "switch/ps/xbox port", "w4 games", "console certification".
---

# console-porting

Guide porting a Godot 4.x game to console via W4 Games' commercial console ports.

## Reality check first
- Console ports require **manufacturer licenses and NDAs**. This skill contains NO
  proprietary SDKs and NO NDA-covered information.
- The practical path for Godot is W4 Games (or an approved porting partner).

## What to cover
1. **Readiness:** input abstraction (already using `InputMap`? controller-first UI?),
   resolution/safe-area handling, save API abstraction, performance budgets.
2. **Per-platform guidelines:** TRC/lotcheck-style requirements at a checklist level
   (button prompts, suspend/resume, storage, age ratings).
3. **Process:** engage W4/partner, obtain platform SDK under license, build/cert loop.

## Rules
- Never include or request NDA/proprietary material.
- Focus on making the game *port-ready*; the licensed SDK work happens under the
  partner's tooling.
