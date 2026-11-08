---
tags: [fruit-empire, deploy, tools, studio]
status: current
updated: 2026-11-08
---
# Deploy method

How Samarth's Claude chats put scripts into Studio without breaking someone else's work. Words used here (fingerprint, before/after copy) are defined below or in [[Fruit Empire Glossary]]. Back to [[Knowledge Base - Start Here]].

## The idea

The place is the source of truth: there is no Rojo sync. But each project keeps an exact local copy of every script it touches, in its own git repo on Samarth's computer. A deploy is a set of Luau **snippets** run through `execute_luau`. Every snippet first checks that the script in Studio is exactly the version it expects. If someone changed it, the snippet refuses. (PLACE.md "Scripts: tell the other Claude what you changed")

**Fingerprint (h32):** a number from a script's text. Same length + same fingerprint = the exact same script. The snippet from PLACE.md:

```lua
local function h32(s) local h = 0 for i = 1, #s do h = (h * 31 + string.byte(s, i)) % 4294967296 end return h end
local src = game.ServerScriptService.GardenService.Swimming.Source
print(#src, h32(src))
```

## A project repo's layout

| Folder | What it holds |
|---|---|
| `before/` | Exact copies of the live scripts, pulled from Studio right before the turn |
| `src/` | The new and changed scripts |
| `after/` | What each patched script must look like once installed |
| `tools/patches/*.py` | The edits that turn `before/` into `after/` |
| `deploy/` | The numbered snippets to run, in order |
| `tests/run.sh` | Offline tests (Lune, with fake Roblox objects) |

Seen in `~/fruit-empire-pets` and `~/fruit-empire-mega-rebirth`; the mine and plot repos use `before/` and `src/` the same way.

## The snippets `tools/make_deploy.py` writes

From the docstring of `~/fruit-empire-pets/tools/make_deploy.py` (30 Oct 2026); the Mega Rebirth repo has its own version in `~/fruit-empire-mega-rebirth/tools/`.

1. **`00_backup.lua`:** copies every live script it will change into a dated ServerStorage backup (disabled), and refuses if one is not the copy in `before/`.
2. **`NN_new_<Name>_<k>.lua`:** a new script, sent in pieces. Each piece is stored in a StringValue; the last piece joins them, checks the fingerprint and writes the script.
3. **`NN_patch_<Name>.lua`:** checks the live script's length and fingerprint match `before/`, applies the edits, checks the result matches `after/`, writes it.
4. **`99_status.lua`:** read-only: each script is DONE, BEFORE, CHANGED or MISSING.
5. **`undo.lua`:** puts the backups back and removes the new scripts.

Every snippet checks `game.PlaceId` first. `tools/check_deploy.py` checks the whole `deploy/` folder offline.

## Lessons from the 30 Oct pets install

> [!warning] zstd frames must store their content size
> New scripts travelled zstd-compressed and base64-encoded, decoded in Studio by `EncodingService:DecompressBuffer`. Roblox refuses a zstd frame that does not store its content size ("Error (generic)"). Compressing from stdin leaves it out. Fix: `zstd --stream-size=<bytes>`; the checker now enforces it.
> Source: `~/fruit-empire-pets/tools/make_deploy.py` lines 229-231; commit ec56f65.

> [!tip] Plain text is cheaper for the agent than base64
> Base64 costs about one token per character for the installing agent to copy; plain Lua costs about a third of that. So `DEPLOY_PLAIN=1` sends plain Lua pieces (whole lines, so no piece cuts a character), and an agent can carry about three times as much per message. Compression only pays when bytes, not tokens, are the limit.
> Source: `~/fruit-empire-pets/tools/make_deploy.py` lines 180-182; commit 57d36d5.

## Smaller updates

- **One already-installed script:** `python3 tools/make_update.py <commit> <path> <out> "why" [after|src]` (Mega Rebirth repo). (memory `project_fruit_empire_mega_rebirth.md`)
- **Only the changed parts of a big script:** `~/fruit-empire-mine/dig/tools/make_patch.py` turns the git diff into `swap()` steps (each anchor must be unique; length and fingerprint checked before and after). It sent 11 KB instead of 48 KB. Trap: a Lua long string `[==[` drops one leading newline. (memory `reference_roblox_studio_mcp.md`, 22 Oct)
- **The map's scripts:** `~/fruit-empire-minimap/tools/make_delta.py OLD NEW path OUT`. (memory `project_fruit_empire_minimap.md`)

## When Studio changed under you

If a script changed since your `before/` copy, pull it again and rebase: `recheck.luau` (does Studio still match?), export, pull to `fresh/`, then a three-way merge that regenerates the patches (`~/fruit-empire-mine/crystal/tools/rebase.sh`). (memory `project_fruit_empire_mine.md`, 22 Oct)

## Getting exact copies out of Studio

HttpService is off in the place, so scripts come out another way:

- **Through the log:** print each line with a marker as JSON, then read the markers out of `~/Library/Logs/Roblox/*Studio*_last.log`. Log lines are cut at about 1,022 characters, so print in chunks. (memory `reference_roblox_studio_mcp.md`, 3 Oct and 6 Oct)
- **Through a big reply:** pad an `execute_luau` reply past about 95,000 characters; the harness saves it to a file instead of showing it (`~/fruit-empire-upgrade-tree/tools/stream.luau` and `unstream.py`, with a length + fingerprint check). (memory `reference_roblox_studio_mcp.md`, 22 Oct)
- `execute_luau` replies are cut at 100,000 characters. `script_read` returns the whole script and ignores start and end lines. (memory `reference_roblox_studio_mcp.md`)

## After the install

List every script changed or added in the handoff, with its full path, new length and fingerprint, plus every non-script change (GUI, models, folders, attributes, terrain). Fingerprints only cover script text. (PLACE.md; [[Studio Turn Rules]])

Related: [[Builders and ZZTools]], [[Studio MCP Traps]], [[Testing in Studio]].
