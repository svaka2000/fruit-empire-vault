---
tags: [fruit-empire, studio, mcp, gotchas]
status: current
updated: 2026-11-08
---
# Studio MCP traps

The MCP is the bridge that lets a Claude chat drive Roblox Studio (its tools include `execute_luau`, `screen_capture`, `start_stop_play`, `user_mouse_input`). These are the traps that cost the most time. Back to [[Knowledge Base - Start Here]].

> [!note] Full list
> The complete, dated list lives in memory: `~/.claude/projects/-Users-samarthvaka/memory/reference_roblox_studio_mcp.md`. The shared gotchas are in [PLACE.md](https://github.com/svaka2000/fruit-empire-handoffs/blob/main/PLACE.md) "Gotchas". How to connect Studio is in PLACE.md's first section.

## Connecting

- The Studio MCP is a host that Studio connects *to* (`ws://127.0.0.1:13469`). A one-shot call dies before Studio attaches and says "no studios". Keep one process alive and turn on **Assistant > Manage MCP Servers > Enable Studio as MCP server**.
- Call `list_roblox_studios` first: the studio id changes whenever Studio reconnects or reopens.
- Never open a second Studio with `open "roblox-studio:..."`; it starts a second process.

## Code and modules

- **`require` is cached** for the whole session in the command bar and `execute_luau`. After changing a module, require a clone. A clone of a module that reads `script.Parent` (GardenConfig) must be parented under ReplicatedStorage while it loads.
- **`ChangeHistoryService` recording does not work** from `execute_luau`, so deletions are not one Cmd+Z. Scope them tightly.
- **GardenConfig is near the 200,000-character script cap** (about 193k on 22 Oct). New config goes in child modules (`GardenConfig.MineDig`, `.BatTiers`, `.Pets`...) loaded by a guarded block. (memory `project_fruit_empire.md`, `project_fruit_empire_mine.md`)
- Some sessions refuse to parent a ModuleScript into ServerStorage; keep builder code in a local file then.
- `VirtualInputManager` is blocked from `execute_luau`. Use the MCP's `user_keyboard_input` and `user_mouse_input` (with `instance_path`).

## Screenshots

- `screen_capture` never draws BillboardGuis with `AlwaysOnTop` on, and Edit mode does not draw SurfaceGuis. Check those in Play, or measure them.
- In Play, `screen_capture` with `camera_position` and `look_at_position` gives a fixed view (since 22 Oct). Roblox lights well only near the character, so move the character there first and wait about 3 s.
- `execute_luau` puts the camera back to Custom after each call. A Scriptable camera set inside it does not last.
- When Samarth's Mac screen is locked, captures time out and the client's `RenderStepped` stops (Heartbeat still runs).
- **Never take a full-screen macOS screenshot.** It captures whatever Samarth is doing, not the game. Delete it if one is made.

## Play tests

- **A Play test can write real saves** when Studio's API access is on (it was on in practice on 18 Oct). Set StudioNoSave first. See [[Testing in Studio]].
- **Studio is shared:** another chat's `execute_luau` can run inside your Play test. See [[Studio Turn Rules]].
- A gamepass purchase prompt (pressing Merge All) blocks every MCP click until Play stops. Do those last.
- The join popup (`GardenUI.OfflinePopup`) has a full-screen blocker that eats clicks and the M key; close it first. Ask "what is on top?" with `PlayerGui:GetGuiObjectsAtPosition(x, y)`.
- `user_mouse_input` y is 58 px above the viewport y (the top bar inset). After a move, read `UserInputService:GetMouseLocation()` to be sure.
- Short moments (a 3-second banner): trigger and capture in the **same** message; the next message comes 10+ seconds later.

## World editing

- `insert_asset` stacks every part of a model on one point. `game:GetObjects("rbxassetid://ID")` inside `execute_luau` keeps positions (strip scripts).
- After big terrain edits, the Edit view keeps the old far-away mesh until the camera has been near; trust raycasts and `ReadVoxels` over a picture.
- A part's colour is stored 8 bits per channel and rounded down; compare within 1/255.
- Unsaved MCP work is lost on a Studio restart. Scripts saved separately survived once; the world did not. See [[Builders and ZZTools]].

## UI

- The "Larger" text setting and `TextService` disagree between the console and game scripts. See [[UI Kit and Text Rules]].
- The mouse wheel over an Active GUI never reaches `ContextActionService`; read the GUI's own `InputChanged`. (memory `reference_roblox_text_size_setting.md`)
- Read image sizes with `EditableImage`, never from screenshots. (memory `reference_roblox_ui_measuring.md`)

Every point above is from `reference_roblox_studio_mcp.md` unless another source is named.

Related: [[Deploy Method]], [[Testing in Studio]], [[Builders and ZZTools]].
