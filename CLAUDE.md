# godot-suite — Claude Code instructions

Curated Claude Code **plugin** (a marketplace repo) for professional Godot 4.x indie
game development. It **composes** the existing Godot AI ecosystem and adds the gaps —
it does not reinvent it.

## What this repo is (and is not)

- It is a **plugin distributed via a marketplace**, not a game and not application code.
- Everything here is **Markdown + JSON**. There is no compiled code and no unit-test
  runner. "Tests" = `jq` on manifests, frontmatter/size checks on skills, a no-secrets
  scan, and `claude plugin validate .`.
- **Compose, do not vendor.** Never copy third-party skills (GodotPrompter, Randroids,
  fenixnix) into this repo. Reference them by URL / install command only.

## Layout

```
.claude-plugin/marketplace.json      # marketplace manifest (lists the plugin)
plugins/godot-suite/
  .claude-plugin/plugin.json         # plugin manifest
  .mcp.json                          # default MCP server: Coding-Solo/godot-mcp
  skills/<name>/SKILL.md             # 6 skills; heavy detail in references/
  agents/<name>.md                   # godot-producer, godot-release-manager
  commands/godot-new-game.md         # /godot-new-game
docs/                                # roles.md, addons.md, mcp-upgrade.md, verification.md
docs/superpowers/specs|plans/        # design spec + implementation plan
```

The 6 skills: `godot-suite-setup` (idempotent bootstrap), `godot-suite-orchestrator`
(routes the user's 25 roles), and the 4 gap skills `steam-publishing`,
`game-design-gdd`, `asset-pipeline-ai`, `console-porting`.

## Hard rules (from the design spec — do not weaken)

- **Target Godot 4.x only.** `godot-suite-setup` aborts on 3.x.
- **No secrets, ever.** No real Steam App IDs, API keys, or console SDKs. Recipes use
  `<YOUR_APP_ID>` / `<DEPOT_ID>` placeholders; the user supplies real values locally.
- **Default MCP = Coding-Solo** (`npx @coding-solo/godot-mcp`); mkdevkit / DaRealDaHoodie
  only in `docs/mcp-upgrade.md` as an opt-in upgrade.
- **Every `SKILL.md`**: valid YAML frontmatter (`name` + `description` with explicit
  triggers), core+references pattern, **under 16 KB** (16384 bytes).
- `docs/roles.md` enumerates **all 25 roles** incl. the partials (Inventory #10,
  Quest #11, Documentation Generator #24). The 4 gap roles route to the exact skill
  names above.
- GodotPrompter install commands are marked "verify against current README", never
  asserted as guaranteed (its marketplace/plugin names have changed before).

## Packaging gotcha (important)

Files inside `plugins/godot-suite/` must reference **plugin-local** paths, not repo-root
`docs/*`. The installed unit is `plugins/godot-suite/` — bare `docs/roles.md` resolves
against the user's project cwd at runtime and is unreachable. Bundle needed docs into a
skill's `references/` and point at them with `${CLAUDE_PLUGIN_ROOT}/skills/.../references/…`.
`roles.md`, `addons.md`, and `mcp-upgrade.md` already have bundled copies — keep the
bundled copy **byte-identical** to the repo-root source when either changes.

## Validate before committing

```bash
jq empty .claude-plugin/marketplace.json plugins/godot-suite/.claude-plugin/plugin.json plugins/godot-suite/.mcp.json
claude plugin validate .                                   # read-only; must print "Validation passed"
# no in-plugin bare docs/ refs:
grep -rnE 'docs/(roles|addons|mcp-upgrade)\.md' plugins/godot-suite/ && echo BAD || echo OK
# no secrets:
grep -rniE 'api[_-]?key|sk-[A-Za-z0-9]{10}|app_?id[[:space:]]*[:=][[:space:]]*[0-9]{5,}' plugins/godot-suite/ && echo SECRET || echo "NO SECRETS"
# each SKILL.md < 16384 bytes and starts with frontmatter (head -1 == ---, has name:/description:)
```

Note: the local `grep` is aliased to `ugrep`; use `grep -- 'pattern'` when matching a
leading `---`, or use `head -1` / `wc -c`.

Do **not** run `claude plugins marketplace add` / `install` against the user's real
config as a "test" — that mutates their environment. Structural validation +
`claude plugin validate` is the check. Live install steps are user-run (see
`docs/verification.md`).

## Install / use (user-run)

```
claude plugins marketplace add /home/soho/gitlab-repos/jfreyre/godot-suite
claude plugins install godot-suite@godot-suite
```
Then invoke the `godot-suite-setup` skill inside a Godot 4.x project.

The repo is local with **no remote**. Ask before configuring one or pushing.
