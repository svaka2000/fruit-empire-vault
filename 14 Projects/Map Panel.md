---
tags: [fruit-empire, project, map, ui]
status: built
updated: 2026-11-08
code: ~/fruit-empire-minimap
handoffs: [10, 25, 26]
---
# Map panel (minimap)

Back to [[Project Status Board]].

## What players get

A **Map button** at the top right (left of the Settings gear), or the M key, opens a map with three tabs: **Ground** (main island, sea, forest, mountains), **Aether** (the Cloud Garden) and, since 24 Oct, **Mines** (a floor plan of the mine). It opens on your layer, centred on you. Every player shows live with their headshot and name; yours is gold with an arrow. A player under an Invisibility potion is hidden. Players on the beanstalk move from Ground to Aether at 220 studs up. Since 24 Oct the Mines tab also draws the tunnels players dig, as they dig them, and a race egg as a sign in its rarity colour until someone takes it. (STATE.md "21 to 24 Oct: the map")

## Samarth's words

- 20 Oct: "a minimap feature that you can open where you can switch between the aether and the ground. also showing other players and where they are ... make it loook super good."
- 23 Oct: "add a minimap for the mines", "make it an option along witht eh aether and the ground".
- 24 Oct, on showing dug tunnels and the race egg: "yes", then "yes to both".

Source: memory `project_fruit_empire_minimap.md`; DECISIONS.md.

## Where it lives

- **In the place:** `ReplicatedStorage.MapConfig` (numbers, words, image ids), `ServerScriptService.MapService` (`GardenRemotes.MapWhere`, for players too far away to be on your computer), `StarterPlayerScripts.MapMenu`, `StarterGui.GardenUI.MapButton`; one line in `Kit.Modals`, three in `Config.HudClusters`. Backups `ServerStorage.MinesMap_backup_2026-10-24`, `ServerStorage.MapDigs_backup_2026-10-24`.
- **Local:** `~/fruit-empire-minimap` (`src/`, `tools/` with `export_map.lua`, `render_map.py`, `floor_map.lua`, `patch_mapmenu.py`, `make_delta.py`; `art/`; `out/` full-size renders; `data/asset_ids.json`). The source of truth for `MapMenu` is `tools/patch_mapmenu.py` applied to `src/MapMenu.studio.lua`.
- **Canvas:** [Fruit Empire Map UI](https://claude.ai/artifact/1i6LMz5UaM2VFFQzgpiC18).
- **Handoffs:** [PR #10](https://github.com/svaka2000/fruit-empire-handoffs/pull/10), [PR #25](https://github.com/svaka2000/fruit-empire-handoffs/pull/25), [PR #26](https://github.com/svaka2000/fruit-empire-handoffs/pull/26).

> [!warning] The pictures are drawings of the world
> The map images were rendered from the place's geometry on 21 Oct (the mine on 24 Oct). Any world change (new docks, the cave, safe pockets) means re-running the export and render tools and uploading new ids into `MapConfig`. (memory `project_fruit_empire_minimap.md`)

## Links to other scripts

- With the Mines tab open, `MapMenu` calls `GardenRemotes.MineDigSync` every 3 s. MineDig allows one call a second per player, so a map call can turn away one `MineDigClient` sync; MineDigClient retries. Anyone who changes MineDigSync must update the map's reader. (queue note "Known links between chats' scripts")
- `MapMenu` was moved onto `Kit.panelFrame` by a change with no handoff (22 Oct); the Mines tab builds on it. (STATE.md "Open items")
- The pets design adds nest, Legend and Fuse Machine pins. ([[Pets, Eggs and Fuse Machine]])

## Open items

- Both people's approval as new UI (with the Mines tab, dug tunnels, egg sign).
- Not checked on a phone or tablet, at the five screen sizes with the map open, or with two real players (only stand-ins).
- Samarth: does the egg sign make the race too easy?

Sources: [STATE.md](https://github.com/svaka2000/fruit-empire-handoffs/blob/main/STATE.md); memory `project_fruit_empire_minimap.md`.
