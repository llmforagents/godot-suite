# godot-suite: End-to-End Install Verification

This document records verification results for the `godot-suite` Claude Code plugin
(marketplace repo at `/home/soho/gitlab-repos/jfreyre/godot-suite`, branch
`feat/godot-suite`). It corresponds to Task 8 of the implementation plan.

Two categories of checks are covered:

1. **Structural / static validation** — executed directly in this session (safe,
   read-only, no mutation of the user's Claude Code environment).
2. **Live install + discovery** — deliberately **not executed** here, because it
   requires `claude plugins marketplace add` / `claude plugins install` against the
   user's real Claude Code configuration. Manual steps are provided below for the
   user to run themselves.

## Step 1: Tree and manifest validation (executed)

Commands run from the repo root:

```bash
jq empty .claude-plugin/marketplace.json
jq empty plugins/godot-suite/.claude-plugin/plugin.json
jq empty plugins/godot-suite/.mcp.json
ls plugins/godot-suite/skills/*/SKILL.md | wc -l   # expect 6
ls plugins/godot-suite/agents/*.md | wc -l          # expect 2
ls plugins/godot-suite/commands/*.md | wc -l        # expect 1
echo "TREE OK"
```

Results:

| Check | Result |
|---|---|
| `jq empty .claude-plugin/marketplace.json` | valid JSON (exit 0) |
| `jq empty plugins/godot-suite/.claude-plugin/plugin.json` | valid JSON (exit 0) |
| `jq empty plugins/godot-suite/.mcp.json` | valid JSON (exit 0) |
| SKILL.md count | **6** (expected 6) |
| agents/*.md count | **2** (expected 2) |
| commands/*.md count | **1** (expected 1) |

`TREE OK` — all checks passed.

Skills found: `asset-pipeline-ai`, `console-porting`, `game-design-gdd`,
`godot-suite-orchestrator`, `godot-suite-setup`, `steam-publishing`.

Agents found: `godot-producer`, `godot-release-manager`.

Command found: `godot-new-game`.

## Step 2: Read-only plugin validator (executed)

The Claude Code CLI (`claude` v2.1.205) exposes a read-only
`claude plugin validate <path>` command that checks marketplace/plugin manifest
correctness **without installing anything**. This was run in place of an actual
install:

```console
$ claude plugin validate .
Validating marketplace manifest: /home/soho/gitlab-repos/jfreyre/godot-suite/.claude-plugin/marketplace.json

✔ Validation passed

$ claude plugin validate . --strict
Validating marketplace manifest: /home/soho/gitlab-repos/jfreyre/godot-suite/.claude-plugin/marketplace.json

✔ Validation passed

$ claude plugin validate plugins/godot-suite
Validating plugin manifest: /home/soho/gitlab-repos/jfreyre/godot-suite/plugins/godot-suite/.claude-plugin/plugin.json

✔ Validation passed

$ claude plugin validate plugins/godot-suite --strict
Validating plugin manifest: /home/soho/gitlab-repos/jfreyre/godot-suite/plugins/godot-suite/.claude-plugin/plugin.json

✔ Validation passed
```

All four invocations (marketplace manifest, plugin manifest, both with and without
`--strict`) passed with no warnings or errors.

## Step 3: Per-file frontmatter and size checks (executed)

Every `SKILL.md`, agent `.md`, and command `.md` was checked for:
- starting with a `---` YAML frontmatter fence,
- containing `name:` (skills and agents) and/or `description:` fields in that
  frontmatter,
- being well under the 16384-byte size ceiling.

| File | Starts with `---` | `name:` present | `description:` present | Size (bytes) |
|---|---|---|---|---|
| `skills/asset-pipeline-ai/SKILL.md` | yes | yes | yes | 1174 |
| `skills/console-porting/SKILL.md` | yes | yes | yes | 1211 |
| `skills/game-design-gdd/SKILL.md` | yes | yes | yes | 1009 |
| `skills/godot-suite-orchestrator/SKILL.md` | yes | yes | yes | 1318 |
| `skills/godot-suite-setup/SKILL.md` | yes | yes | yes | 2059 |
| `skills/steam-publishing/SKILL.md` | yes | yes | yes | 1444 |
| `agents/godot-producer.md` | yes | yes | yes | 777 |
| `agents/godot-release-manager.md` | yes | yes | yes | 837 |
| `commands/godot-new-game.md` | yes | n/a* | yes | 652 |

\* Slash-command frontmatter only requires `description:` — the command's invocation
name is derived from the filename (`godot-new-game.md` → `/godot-new-game`), not from
a `name:` frontmatter field. This matches standard Claude Code command conventions
and is not a defect.

**Summary: all 9 files have valid frontmatter and are well within size limits**
(largest is 2059 bytes, roughly 12.5% of the 16384-byte ceiling).

## What was executed vs. not executed

**Executed in this session** (read-only, no mutation of the user's Claude Code
config):
- `jq empty` on all three manifests.
- File-count checks (`ls | wc -l`) for skills/agents/commands.
- `claude plugin validate .` and `claude plugin validate plugins/godot-suite`
  (with and without `--strict`).
- Per-file frontmatter presence and byte-size checks on all 9 component files.

**Not executed** (would mutate the user's real Claude Code environment — must be
run interactively by the user):
- `claude plugins marketplace add ...`
- `claude plugins install ...`
- Live discovery confirmation (skills list, `/godot-new-game` availability, `/mcp`
  server status).

## User-run steps (not executed here)

Run these yourself in a Claude Code session to complete live install verification:

```bash
claude plugins marketplace add /home/soho/gitlab-repos/jfreyre/godot-suite
claude plugins install godot-suite@godot-suite
```

Then, inside that Claude Code session, confirm:

1. All six skills appear in the skills list: `asset-pipeline-ai`, `console-porting`,
   `game-design-gdd`, `godot-suite-orchestrator`, `godot-suite-setup`,
   `steam-publishing`.
2. `/godot-new-game` is available as a slash command.
3. Run `/mcp` and confirm the `godot` server shows a green dot with `tools > 0`
   once `npx` fetches `@coding-solo/godot-mcp`.

If the live install surfaces any manifest/schema mismatch not caught by the
read-only checks above, fix `plugins/godot-suite/.claude-plugin/plugin.json`,
`.claude-plugin/marketplace.json`, or `plugins/godot-suite/.mcp.json` accordingly
and re-run both the structural checks and the live install.

## Conclusion

All structural/static validation available without installing passed cleanly:
tree counts match exactly (6 skills, 2 agents, 1 command), all three JSON manifests
are valid, the CLI's own read-only validator reports success in strict mode, and
every component file has correct, appropriately-sized frontmatter. No structural or
schema issues were found. Live install/discovery verification is deferred to the
user per the manual steps above.
