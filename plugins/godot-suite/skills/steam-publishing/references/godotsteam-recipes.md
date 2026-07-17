# GodotSteam recipes

Short, copy-pasteable stubs for the common GodotSteam integration points. These
assume the GodotSteam GDExtension is already added to the project (autoload name
`Steam` in the examples below — adjust to your project's setup).

Method names below match the GodotSteam 4.x GDExtension API as of this writing.
GodotSteam's API shifts between releases — **verify each call against the
GodotSteam docs for the version you're pinned to** (flagged inline where it
matters most).

---

## SDK init with fallback

Put this in an autoload (e.g. `SteamManager.gd`) so it runs once at boot. Never
hardcode the App ID — read it from a local, gitignored file or an env var, and
always allow the game to run without Steam (dev machines, CI, non-Steam builds).

```gdscript
# res://autoload/SteamManager.gd
extends Node

var steam_enabled: bool = false

func _ready() -> void:
	# Steam requires steam_appid.txt next to the executable in dev builds,
	# or the app to be launched via Steam in production. Never commit the
	# real App ID to the repo — steam_appid.txt should be gitignored, or
	# read the ID from an env var for CI/dev builds.
	var app_id := _load_app_id()
	if app_id <= 0:
		push_warning("SteamManager: no App ID configured, running without Steam")
		return

	# NOTE: verify against GodotSteam docs for your pinned version — some
	# releases expose steamInitEx(app_id, embed_callbacks), others steamInit().
	var init_result: Dictionary = Steam.steamInitEx(app_id, true)
	if init_result.get("status", 1) != 0:
		push_warning("SteamManager: Steam init failed (%s) — continuing without Steam" % init_result.get("verbal", "unknown error"))
		steam_enabled = false
		return

	steam_enabled = true
	print("SteamManager: Steam initialized for user %s" % Steam.getPersonaName())

func _process(_delta: float) -> void:
	if steam_enabled:
		Steam.run_callbacks()

func _load_app_id() -> int:
	# Local, gitignored override for dev machines.
	var override_path := "res://.local/steam_app_id.txt"
	if FileAccess.file_exists(override_path):
		var f := FileAccess.open(override_path, FileAccess.READ)
		return int(f.get_as_text().strip_edges())
	# CI / packaged-build override.
	var env_val := OS.get_environment("STEAM_APP_ID")
	if env_val != "":
		return int(env_val)
	return 0 # no App ID configured — run without Steam
```

Usage elsewhere: guard every Steam call with `if SteamManager.steam_enabled:`.

---

## Unlock achievement

Achievement API IDs must first be defined in the Steamworks dashboard (App Admin
> Stats and Achievements). Use the same string ID here as configured there —
never invent IDs client-side.

```gdscript
# Somewhere in game logic, e.g. after a boss kill:
func unlock_achievement(achievement_api_id: String) -> void:
	if not SteamManager.steam_enabled:
		return
	Steam.setAchievement(achievement_api_id)
	Steam.storeStats() # flushes the achievement + any pending stats to Steam

# Example call:
# unlock_achievement("ACH_DEFEATED_FIRST_BOSS")
```

To read achievement state (e.g. for an in-game achievements screen):

```gdscript
func is_achievement_unlocked(achievement_api_id: String) -> bool:
	if not SteamManager.steam_enabled:
		return false
	# NOTE: verify return shape against docs — getAchievement typically
	# returns a Dictionary like {"ret": true, "achieved": true}.
	var result: Dictionary = Steam.getAchievement(achievement_api_id)
	return result.get("achieved", false)
```

Stats (integers/floats backing leaderboards or progress) follow the same
pattern: `Steam.setStatInt("STAT_ID", value)` / `Steam.setStatFloat(...)`, then
`Steam.storeStats()`. Call `Steam.requestCurrentStats()` once after init and
wait for the `current_stats_received` signal before reading/writing stats.

---

## Write/read cloud save

Steam Cloud (Remote Storage) mirrors files by name inside Steam's per-user
sandbox. Always keep a local save as a fallback for offline play or when
Steam Cloud is disabled by the user.

```gdscript
# res://autoload/SaveManager.gd (excerpt)
const CLOUD_SAVE_NAME := "savegame.dat"
const LOCAL_SAVE_PATH := "user://savegame.dat"

func save_game(payload: PackedByteArray) -> bool:
	# Always write locally first — this is the fallback path.
	var f := FileAccess.open(LOCAL_SAVE_PATH, FileAccess.WRITE)
	if f == null:
		push_error("SaveManager: failed to open local save for write")
		return false
	f.store_buffer(payload)
	f.close()

	if not SteamManager.steam_enabled:
		return true
	# NOTE: verify signature against docs — fileWrite(file: String, data: PackedByteArray) -> bool.
	var ok: bool = Steam.fileWrite(CLOUD_SAVE_NAME, payload)
	if not ok:
		push_warning("SaveManager: Steam Cloud write failed, local save still intact")
	return true

func load_game() -> PackedByteArray:
	if SteamManager.steam_enabled and Steam.fileExists(CLOUD_SAVE_NAME):
		var size: int = Steam.getFileSize(CLOUD_SAVE_NAME)
		# NOTE: verify signature against docs — fileRead(file: String, size: int) -> PackedByteArray (or Dictionary wrapping it).
		var data: PackedByteArray = Steam.fileRead(CLOUD_SAVE_NAME, size)
		if data.size() > 0:
			return data
		push_warning("SaveManager: Steam Cloud read returned empty, falling back to local save")

	if FileAccess.file_exists(LOCAL_SAVE_PATH):
		var f := FileAccess.open(LOCAL_SAVE_PATH, FileAccess.READ)
		var data := f.get_buffer(f.get_length())
		f.close()
		return data

	return PackedByteArray()
```

Confirm Steam Cloud is enabled for the app (Steamworks dashboard > Steam Cloud)
and, if you rely on Auto-Cloud instead of the Remote Storage API, configure the
save file path pattern there rather than calling `fileWrite`/`fileRead`.

---

## steamcmd build script skeleton

Two VDF files plus one shell invocation. Fill in `<YOUR_APP_ID>`, `<DEPOT_ID>`,
and paths locally — **never commit real IDs**; keep these files outside the
repo or in a gitignored `build/steam/` directory, and template them at build
time (e.g. from CI secrets) if you need them versioned.

`build/steam/app_build.vdf`:

```vdf
"AppBuild"
{
	"AppID" "<YOUR_APP_ID>"
	"Desc" "Automated build via steamcmd"

	"ContentRoot" "../../dist/"
	"BuildOutput" "./output/"

	"Depots"
	{
		"<DEPOT_ID>" "depot_build_<DEPOT_ID>.vdf"
	}
}
```

`build/steam/depot_build_<DEPOT_ID>.vdf`:

```vdf
"DepotBuild"
{
	"DepotID" "<DEPOT_ID>"
	"ContentRoot" "../../dist/"
	"FileMapping"
	{
		"LocalPath" "*"
		"DepotPath" "."
		"recursive" "1"
	}
}
```

Upload invocation (run after your Godot export writes to `dist/`):

```bash
#!/usr/bin/env bash
set -euo pipefail

# Credentials come from steamcmd's own login flow / a local config —
# never hardcode a username or password in this script or in CI logs.
STEAM_USERNAME="${STEAM_USERNAME:?set STEAM_USERNAME in your local env}"

steamcmd \
	+login "$STEAM_USERNAME" \
	+run_app_build "$(pwd)/build/steam/app_build.vdf" \
	+quit
```

First run requires an interactive Steam Guard confirmation; subsequent runs can
use `steamcmd`'s cached login or a CI secrets-backed config per Valve's docs.
