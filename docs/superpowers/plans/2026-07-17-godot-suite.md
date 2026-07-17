# godot-suite Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build `godot-suite`, an installable Claude Code plugin that curates the Godot 4.x AI dev ecosystem (composes GodotPrompter + a Godot MCP server + pinned addons) and adds four gap skills (Steam publishing, game design/GDD, AI asset pipeline, console porting) plus orchestration for the user's 25 roles.

**Architecture:** A Claude Code plugin marketplace repo. The plugin ships skills (`SKILL.md`), subagents (`agents/*.md`), a slash command, and an MCP server declaration. It **composes** existing projects (does not vendor their code). A bootstrap skill (`godot-suite-setup`) wires up GodotPrompter + MCP + addons idempotently; an orchestrator skill + `godot-producer` subagent route the 25 roles to the right skill/agent.

**Tech Stack:** Markdown (SKILL.md, agents, commands, docs), JSON (marketplace.json, plugin.json, .mcp.json), `jq` for validation, `node`/`npx` (MCP runtime, already on machine via nvm), Godot 4.x (target engine, user-provided). No TypeScript/compiled code.

## Global Constraints

- Target **Godot 4.x only**; if Godot 3.x is detected, abort with an actionable message (spec §2).
- **Compose, do not vendor**: never copy GodotPrompter/Randroids/fenixnix skill files into this repo (spec §1, §2).
- Default MCP server: **Coding-Solo/godot-mcp** (`npx @coding-solo/godot-mcp`); mkdevkit documented only as upgrade (spec §3).
- Every `SKILL.md` uses the **core + references** pattern and stays **under 16 KB** (spec §6).
- Every `SKILL.md` and agent file has valid YAML frontmatter with `name` + `description` (triggers explicit) (spec §6).
- **No secrets** in the repo: no Steam App IDs, API keys, or console SDKs; skills instruct the user to supply them locally (spec §8).
- GodotPrompter install commands must be **verified against its current README at implement time**, not hardcoded (spec §5.1).
- `docs/roles.md` must **enumerate all 25 roles explicitly**, including partials (Inventory, Quest, Documentation Generator) (spec §5.2).
- Commit frequently; conventional commit messages.

---

### Task 1: Verify plugin manifest schema and scaffold repo skeleton

**Files:**
- Create: `.claude-plugin/marketplace.json`
- Create: `plugins/godot-suite/.claude-plugin/plugin.json`
- Create: `plugins/godot-suite/.mcp.json`
- Create: `README.md`
- Create: `.gitignore`

**Interfaces:**
- Consumes: nothing (first task).
- Produces: a valid, installable plugin skeleton. Later tasks add `skills/`, `agents/`, `commands/` directories under `plugins/godot-suite/`, and `docs/` at repo root.

- [ ] **Step 1: Confirm the current Claude Code plugin + marketplace manifest schema**

Dispatch the `claude-code-guide` agent (or fetch docs) with this question:
"What are the exact required fields and file layout for a Claude Code plugin marketplace repo in 2026 — specifically `.claude-plugin/marketplace.json`, a plugin's `.claude-plugin/plugin.json`, how skills/agents/commands directories are discovered, and how a plugin declares an MCP server (plugin.json `mcpServers` vs a `.mcp.json` file, and the `${CLAUDE_PLUGIN_ROOT}` variable)."

Record the confirmed schema. If it differs from the JSON below, use the confirmed schema and adjust all later tasks accordingly.

- [ ] **Step 2: Write `.claude-plugin/marketplace.json`**

```json
{
  "name": "godot-suite",
  "owner": {
    "name": "freyrecorp",
    "url": "https://gitlab.com/jfreyre/godot-suite"
  },
  "metadata": {
    "description": "Curated Godot 4.x indie game dev suite for Claude Code",
    "version": "0.1.0"
  },
  "plugins": [
    {
      "name": "godot-suite",
      "source": "./plugins/godot-suite",
      "description": "Curated Godot 4.x dev environment: composes GodotPrompter + Godot MCP + pinned addons, adds Steam/GDD/AI-asset/console skills and 25-role orchestration."
    }
  ]
}
```

- [ ] **Step 3: Write `plugins/godot-suite/.claude-plugin/plugin.json`**

```json
{
  "name": "godot-suite",
  "version": "0.1.0",
  "description": "Curated Godot 4.x indie game dev suite: setup bootstrap, 25-role orchestration, and gap skills (Steam publishing, game design/GDD, AI asset pipeline, console porting).",
  "author": { "name": "freyrecorp" },
  "keywords": ["godot", "gamedev", "gdscript", "indie", "steam", "mcp"]
}
```

- [ ] **Step 4: Write `plugins/godot-suite/.mcp.json`**

```json
{
  "mcpServers": {
    "godot": {
      "command": "npx",
      "args": ["-y", "@coding-solo/godot-mcp"],
      "env": {}
    }
  }
}
```

- [ ] **Step 5: Write `README.md` and `.gitignore`**

`README.md`:
```markdown
# godot-suite

Curated Godot 4.x indie game development suite for Claude Code.

**What it is:** a meta-plugin that composes the existing Godot AI ecosystem
(GodotPrompter skills + a Godot MCP server + pinned community addons) and adds
four gap skills (Steam publishing, game design/GDD, AI asset pipeline, console
porting) plus orchestration for 25 game-dev roles.

**Requires:** Godot 4.x, Node.js (for the MCP server via `npx`).

## Install

```
claude plugins marketplace add <this-repo-url>
claude plugins install godot-suite@godot-suite
```

Then run the `godot-suite-setup` skill in a Godot 4.x project to bootstrap the
environment (GodotPrompter, MCP server, recommended addons).

## Design & plan

See `docs/superpowers/specs/` and `docs/superpowers/plans/`.
```

`.gitignore`:
```
node_modules/
*.log
.DS_Store
```

- [ ] **Step 6: Validate JSON manifests**

Run:
```bash
jq empty .claude-plugin/marketplace.json && \
jq empty plugins/godot-suite/.claude-plugin/plugin.json && \
jq empty plugins/godot-suite/.mcp.json && \
jq -e '.plugins[0].source == "./plugins/godot-suite"' .claude-plugin/marketplace.json && \
jq -e '.mcpServers.godot.command == "npx"' plugins/godot-suite/.mcp.json && \
echo "MANIFESTS OK"
```
Expected: prints `MANIFESTS OK` with no `jq` errors.

- [ ] **Step 7: Commit**

```bash
git add -A
git commit -m "feat: scaffold godot-suite plugin skeleton and manifests"
```

---

### Task 2: Author `docs/roles.md` and `docs/addons.md` (sources of truth)

**Files:**
- Create: `docs/roles.md`
- Create: `docs/addons.md`

**Interfaces:**
- Consumes: nothing.
- Produces: `docs/roles.md` (the 25-role → destination routing table consumed by the orchestrator in Task 4) and `docs/addons.md` (pinned addon table consumed by `godot-suite-setup` in Task 3).

- [ ] **Step 1: Write `docs/roles.md` with all 25 roles enumerated**

Create a Markdown table mapping every one of the user's 25 roles to a destination. Use exactly these 25 rows (role → destination → note):

```markdown
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
| 24 | Documentation Generator | SP/generic docs skill | Document destination + limits |
| 25 | Game Designer | GS `game-design-gdd` | NEW gap skill |
```

- [ ] **Step 2: Write `docs/addons.md` with pinned versions**

```markdown
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
```

- [ ] **Step 3: Validate the roles table has 25 rows**

Run:
```bash
grep -cE '^\| [0-9]+ \|' docs/roles.md
```
Expected: prints `25`.

- [ ] **Step 4: Commit**

```bash
git add -A
git commit -m "docs: add roles routing table and pinned addons registry"
```

---

### Task 3: Author `godot-suite-setup` skill (bootstrap)

**Files:**
- Create: `plugins/godot-suite/skills/godot-suite-setup/SKILL.md`
- Create: `plugins/godot-suite/skills/godot-suite-setup/references/addons.md` (copy pointer, see below)

**Interfaces:**
- Consumes: `docs/addons.md` (addon list/versions).
- Produces: an idempotent bootstrap procedure invoked by name `godot-suite-setup` and by the `godot-new-game` command (Task 7).

- [ ] **Step 1: Write `SKILL.md`**

````markdown
---
name: godot-suite-setup
description: Use to bootstrap or verify a Godot 4.x project's Claude Code environment — installs/checks GodotPrompter, the Godot MCP server, and recommended addons. Trigger on "set up godot", "godot suite setup", "bootstrap godot environment", "install godot skills/addons".
---

# godot-suite-setup

Idempotent bootstrap for a professional Godot 4.x dev environment. Safe to run
repeatedly — each step is a no-op if already satisfied. Verify every step before
reporting success (evidence before assertions).

## Procedure

1. **Detect Godot.** Run `godot --version` (or `godot4 --version`). If missing,
   stop and tell the user to install Godot 4.x (do not auto-install). If the
   version is 3.x, STOP with: "godot-suite targets Godot 4.x; 3.x APIs differ."
2. **Install GodotPrompter (composition).** Verify the exact install commands
   against its current README first — see `references/godotprompter.md`. Run them;
   if already installed, no-op and report the version. If install fails or the
   marketplace/plugin name changed, report the repo URL instead of failing silently.
3. **Verify MCP.** Confirm `npx` is available (`npx --version`). The plugin ships
   `.mcp.json` pointing at `@coding-solo/godot-mcp`. Tell the user to run `/mcp`
   and confirm `godot` shows a green dot with tools > 0. For runtime/screenshots,
   point to `docs/mcp-upgrade.md`.
4. **Install addons (opt-in).** For each addon in `references/addons.md`, ask the
   user before installing into `res://addons/`. Never overwrite an existing addon
   without asking. Record installed addon + version + date in the project's
   `addons.lock.md`.
5. **Report.** Summarize what is installed, versions, and what is still missing.

## Rules
- Idempotent; never overwrite without asking; never write outside the project or
  `~/.claude` without telling the user; never commit secrets.
- If no Godot project is active (`project.godot` absent in cwd), skip addon steps
  and say so.

See `references/` for install command details and the addon table.
````

- [ ] **Step 2: Write `references/addons.md`**

Copy the addon table from `docs/addons.md` into this reference (the skill loads it
on demand). Also create `references/godotprompter.md` containing:

```markdown
# GodotPrompter install (verify before running)

GodotPrompter migrated repos; confirm current commands at
https://github.com/jame581/GodotPrompter before running. As of research:

    claude plugins marketplace add jame581/skillsmith
    claude plugins install godot-prompter@skillsmith

If these fail, the marketplace/plugin name likely changed — check the README and
report the correct command to the user rather than guessing.
```

- [ ] **Step 3: Validate frontmatter and size**

Run:
```bash
f=plugins/godot-suite/skills/godot-suite-setup/SKILL.md
head -1 "$f" | grep -qx '---' && \
grep -q '^name: godot-suite-setup$' "$f" && \
grep -q '^description: ' "$f" && \
[ "$(wc -c < "$f")" -lt 16384 ] && echo "SKILL OK ($(wc -c < "$f") bytes)"
```
Expected: prints `SKILL OK (<N> bytes)` with N < 16384.

- [ ] **Step 4: Commit**

```bash
git add -A
git commit -m "feat: add godot-suite-setup bootstrap skill"
```

---

### Task 4: Author `godot-suite-orchestrator` skill + `godot-producer` subagent

**Files:**
- Create: `plugins/godot-suite/skills/godot-suite-orchestrator/SKILL.md`
- Create: `plugins/godot-suite/agents/godot-producer.md`

**Interfaces:**
- Consumes: `docs/roles.md` (routing table).
- Produces: role-based routing available by invoking `godot-suite-orchestrator` or dispatching the `godot-producer` agent.

- [ ] **Step 1: Write `SKILL.md`**

````markdown
---
name: godot-suite-orchestrator
description: Use when the user asks for Godot work by "role" (e.g. "act as the Physics Specialist", "I need the Steam Publishing role", "as the Game Designer") or for multi-role game features. Routes each of the 25 roles to the correct skill/agent. Trigger on role names or "which godot skill for X".
---

# godot-suite-orchestrator

Maps the user's 25 game-dev roles to the concrete skill or agent that does the
work. The authoritative table lives in `docs/roles.md`.

## How to route
1. Identify the role the user is asking for (match against the 25 in the table).
2. Route to its destination: a GodotPrompter skill (GP), a superpowers skill (SP),
   or a godot-suite skill (GS). Invoke that skill / dispatch that agent.
3. For work spanning multiple roles, dispatch the `godot-producer` agent to plan
   and delegate role-by-role.
4. If the role is ambiguous or unmapped, ask the user to clarify — do not guess.

## Routing table
(Load `docs/roles.md` — it enumerates all 25 roles and destinations, including the
partials: Inventory, Quest, Documentation Generator.)

Gap roles handled by godot-suite's own skills:
- Steam Publishing → `steam-publishing`
- Game Designer → `game-design-gdd`
- Asset Pipeline → `asset-pipeline-ai`
- Console Porting → `console-porting`
````

- [ ] **Step 2: Write `agents/godot-producer.md`**

````markdown
---
name: godot-producer
description: Plans multi-role Godot features and delegates each part to the right skill/agent. Use for game features that span several of the 25 roles (e.g. "build an inventory with UI, save support, and controller input").
---

You are the Godot Producer. Given a game feature request:

1. Decompose it into the 25-role vocabulary (see the orchestrator's `docs/roles.md`).
2. Order the roles into a sensible build sequence.
3. For each role, invoke its mapped skill or delegate to its agent, one at a time.
4. Keep the work honest: verify each part before moving on; surface blockers.
5. Never fabricate Godot APIs — defer to GodotPrompter's GDScript/API skills.

Report a short plan first, then execute role-by-role, then summarize what was built.
````

- [ ] **Step 3: Validate frontmatter/size for both files**

Run:
```bash
for f in plugins/godot-suite/skills/godot-suite-orchestrator/SKILL.md \
         plugins/godot-suite/agents/godot-producer.md; do
  head -1 "$f" | grep -qx '---' && grep -q '^name: ' "$f" && \
  grep -q '^description: ' "$f" && [ "$(wc -c < "$f")" -lt 16384 ] && \
  echo "OK $f ($(wc -c < "$f") bytes)" || echo "FAIL $f"
done
```
Expected: two `OK` lines, no `FAIL`.

- [ ] **Step 4: Commit**

```bash
git add -A
git commit -m "feat: add orchestrator skill and godot-producer agent for 25-role routing"
```

---

### Task 5: Author gap skill `steam-publishing`

**Files:**
- Create: `plugins/godot-suite/skills/steam-publishing/SKILL.md`
- Create: `plugins/godot-suite/skills/steam-publishing/references/godotsteam-recipes.md`

**Interfaces:**
- Consumes: nothing.
- Produces: skill `steam-publishing` (destination of role #20).

- [ ] **Step 1: Write `SKILL.md`**

````markdown
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
4. **Workshop / rich presence:** optional; see recipes.
5. **Upload builds:** configure depots and run `steamcmd` with a build script.

## Rules
- No App IDs, keys, or credentials in the repo — instruct the user to provide them
  via local config/env and document where to get each.
- Provide a checklist; defer exhaustive API detail to references.

See `references/godotsteam-recipes.md`.
````

- [ ] **Step 2: Write `references/godotsteam-recipes.md`**

Include concrete recipe stubs (headed sections, no secrets): "SDK init with fallback",
"Unlock achievement", "Write/read cloud save", "steamcmd build script skeleton
(placeholders for App ID / depot ID that the user fills in)". Each is a short GDScript
or shell snippet using `<YOUR_APP_ID>`-style placeholders explicitly, never real IDs.

- [ ] **Step 3: Validate frontmatter/size**

Run:
```bash
f=plugins/godot-suite/skills/steam-publishing/SKILL.md
head -1 "$f" | grep -qx '---' && grep -q '^name: steam-publishing$' "$f" && \
grep -q '^description: ' "$f" && [ "$(wc -c < "$f")" -lt 16384 ] && \
echo "OK ($(wc -c < "$f") bytes)"
```
Expected: prints `OK (<N> bytes)`, N < 16384.

- [ ] **Step 4: Verify no secrets leaked**

Run:
```bash
grep -rniE 'app_?id[[:space:]]*[:=][[:space:]]*[0-9]{5,}|api[_-]?key|sk-[A-Za-z0-9]' \
  plugins/godot-suite/skills/steam-publishing/ && echo "SECRET FOUND" || echo "NO SECRETS"
```
Expected: prints `NO SECRETS`.

- [ ] **Step 5: Commit**

```bash
git add -A
git commit -m "feat: add steam-publishing gap skill"
```

---

### Task 6: Author gap skills `game-design-gdd`, `asset-pipeline-ai`, `console-porting`

**Files:**
- Create: `plugins/godot-suite/skills/game-design-gdd/SKILL.md`
- Create: `plugins/godot-suite/skills/asset-pipeline-ai/SKILL.md`
- Create: `plugins/godot-suite/skills/console-porting/SKILL.md`

**Interfaces:**
- Consumes: `asset-pipeline-ai` uses the `mcp__image-gen__generate_image` tool if available.
- Produces: skills that are destinations of roles #25, #23, #22.

- [ ] **Step 1: Write `game-design-gdd/SKILL.md`**

````markdown
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
````

- [ ] **Step 2: Write `asset-pipeline-ai/SKILL.md`**

````markdown
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
2. **Aseprite → Godot:** use the Aseprite Wizard addon (see `docs/addons.md`) to
   import `.aseprite` with tags/slices as animations.
3. **Atlases & import:** build texture atlases; set import presets (filter, mipmaps,
   compression) appropriate to 2D pixel-art vs 3D.
4. **Conventions:** enforce folder layout (`res://assets/<type>/<name>`) and naming.

## Rules
- Document licensing limits of AI-generated assets; do not claim rights the user
  doesn't have.
- Keep generation reproducible: record prompts/seeds alongside outputs.
````

- [ ] **Step 3: Write `console-porting/SKILL.md`**

````markdown
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
````

- [ ] **Step 4: Validate all three skills (frontmatter, size, no secrets)**

Run:
```bash
for s in game-design-gdd asset-pipeline-ai console-porting; do
  f=plugins/godot-suite/skills/$s/SKILL.md
  head -1 "$f" | grep -qx '---' && grep -q "^name: $s\$" "$f" && \
  grep -q '^description: ' "$f" && [ "$(wc -c < "$f")" -lt 16384 ] && \
  echo "OK $s ($(wc -c < "$f") bytes)" || echo "FAIL $s"
done
grep -rniE 'api[_-]?key|sk-[A-Za-z0-9]{10}' plugins/godot-suite/skills/ \
  && echo "SECRET FOUND" || echo "NO SECRETS"
```
Expected: three `OK` lines and `NO SECRETS`.

- [ ] **Step 5: Commit**

```bash
git add -A
git commit -m "feat: add game-design-gdd, asset-pipeline-ai, console-porting gap skills"
```

---

### Task 7: Author `godot-release-manager` agent, `godot-new-game` command, and MCP upgrade doc

**Files:**
- Create: `plugins/godot-suite/agents/godot-release-manager.md`
- Create: `plugins/godot-suite/commands/godot-new-game.md`
- Create: `docs/mcp-upgrade.md`

**Interfaces:**
- Consumes: `steam-publishing`, `console-porting`, GodotPrompter export skill, `godot-suite-setup`.
- Produces: the release agent, the `/godot-new-game` command, and MCP upgrade docs.

- [ ] **Step 1: Write `agents/godot-release-manager.md`**

````markdown
---
name: godot-release-manager
description: Orchestrates a Godot game release — runs export (GodotPrompter export skill), then platform publishing (steam-publishing, console-porting) in the right order. Use for "release", "ship the game", "cut a build".
---

You are the Godot Release Manager. For a release request:

1. Confirm the game builds/exports cleanly (delegate to GodotPrompter's export
   skill). Do not proceed if export fails.
2. For each target platform, delegate in order:
   - Desktop/itch → export presets.
   - Steam → `steam-publishing` skill.
   - Console → `console-porting` skill (port-readiness + partner process).
3. Verify each step's output before the next. Never claim a build shipped without
   evidence (an artifact path, an upload confirmation).
4. Summarize artifacts, versions, and any blockers.
````

- [ ] **Step 2: Write `commands/godot-new-game.md`**

````markdown
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
````

- [ ] **Step 3: Write `docs/mcp-upgrade.md`**

```markdown
# MCP upgrade: runtime capabilities

The suite defaults to `@coding-solo/godot-mcp` (stable). To get runtime
inspection, in-game screenshots, and input simulation, switch to a
runtime-capable server:

- mkdevkit/godot-mcp — https://github.com/mkdevkit/godot-mcp
- DaRealDaHoodie/Claude-GoDot-MCP — https://github.com/DaRealDaHoodie/Claude-GoDot-MCP

To switch: edit `plugins/godot-suite/.mcp.json` (or your project `.mcp.json`) to
point `mcpServers.godot` at the chosen server per its README, then restart Claude
Code and verify with `/mcp` (green dot, tools > 0).
```

- [ ] **Step 4: Validate the two agent/command files**

Run:
```bash
for f in plugins/godot-suite/agents/godot-release-manager.md \
         plugins/godot-suite/commands/godot-new-game.md; do
  head -1 "$f" | grep -qx '---' && grep -q '^description: ' "$f" && \
  echo "OK $f" || echo "FAIL $f"
done
jq empty plugins/godot-suite/.mcp.json && echo "MCP JSON OK"
```
Expected: two `OK` lines and `MCP JSON OK`.

- [ ] **Step 5: Commit**

```bash
git add -A
git commit -m "feat: add release-manager agent, godot-new-game command, MCP upgrade doc"
```

---

### Task 8: End-to-end install verification

**Files:**
- Modify: none (verification only); create `docs/verification.md` recording results.

**Interfaces:**
- Consumes: the whole plugin.
- Produces: evidence that the plugin installs and its skills/command/agents are discoverable.

- [ ] **Step 1: Validate the full tree against the confirmed schema**

Run:
```bash
jq empty .claude-plugin/marketplace.json
jq empty plugins/godot-suite/.claude-plugin/plugin.json
jq empty plugins/godot-suite/.mcp.json
ls plugins/godot-suite/skills/*/SKILL.md | wc -l   # expect 6
ls plugins/godot-suite/agents/*.md | wc -l          # expect 2
ls plugins/godot-suite/commands/*.md | wc -l         # expect 1
echo "TREE OK"
```
Expected: `6`, `2`, `1`, then `TREE OK`.

- [ ] **Step 2: Install the plugin locally and verify discovery**

In a Claude Code session:
```
claude plugins marketplace add /home/soho/gitlab-repos/jfreyre/godot-suite
claude plugins install godot-suite@godot-suite
```
Then confirm: the six skills appear in the skills list, `/godot-new-game` is
available, and `/mcp` shows `godot` (green dot, tools > 0) once `npx` fetches the
server. If the plugin manifest schema from Task 1 differed, fix manifests now and
re-run.

- [ ] **Step 3: Record results and commit**

Write `docs/verification.md` with the command outputs (tree counts, install result,
`/mcp` status). Then:
```bash
git add -A
git commit -m "test: record end-to-end install verification results"
```

---

### Final Task: Production Readiness Audit

- [ ] **Step 1: Run `/audit`**

Execute the full `/audit` protocol against everything in this plan. Re-read the spec
(`docs/superpowers/specs/2026-07-17-godot-suite-design.md`) first — Category 4
(Completeness) and Category 5 (Logic Correctness) are scored against its
requirements. For this doc/config repo, score the applicable categories
(Completeness, Logic Correctness, Edge Cases & Validation, Security) and treat the
`jq`/frontmatter/no-secrets checks as the build/test evidence.

- [ ] **Step 2: Fix all findings and re-run**

Fix every issue in the report — not just some — and re-run until PASS.

- [ ] **Step 3: Commit audit fixes**

```bash
git add -A
git commit -m "fix: address production readiness audit findings"
```
