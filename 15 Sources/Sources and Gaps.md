---
tags: [fruit-empire, sources, reference]
status: current
updated: 2026-11-08
---
# Sources and gaps

Where every fact in the 8 Nov notes came from, and what could not be sourced. Back to [[Knowledge Base - Start Here]].

## How notes cite

- **Handoffs repo pages** are linked on GitHub (main branch), for example [STATE.md](https://github.com/svaka2000/fruit-empire-handoffs/blob/main/STATE.md). The local clone is `~/fruit-empire-handoffs`. See [[Handoffs Repo Links]].
- **"memory `name.md`"** means a file in Samarth's Claude memory folder: `~/.claude/projects/-Users-samarthvaka/memory/`. Those files are notes Claude chats wrote for later chats; dates inside them are when the fact was saved.
- **Local paths** (`~/fruit-empire-...`) are the project repos on Samarth's computer.
- **The queue note** is `~/Library/Application Support/Claude/side-session-notes/local_13ce427d-f956-4fc4-9afc-874a9359fd95/studio-turn-rule.md`.

## Sources read on 8 Nov 2026

**Handoffs repo:** CLAUDE.md, README.md, STATE.md, PLACE.md, DECISIONS.md (first rows and outline), ui/README.md, the PR list (`gh pr list`, 31 merged), the blank `handoffs/2026-10-30-samarth-pets/HANDOFF.md`.

**Memory files:** `index_fruit_empire.md`, `project_fruit_empire.md` (selected parts: it is 220 KB), `project_fruit_empire_pets.md`, `_mine.md`, `_minimap.md`, `_forest_animals.md`, `_bats.md`, `_troll_potion.md`, `_mega_rebirth.md`, `_rebirth_panel.md`, `_robux_shop.md`, `_plot_castle.md`, `_upgrade_tree.md`, `feedback_fruit_empire_chats_self_coordinate.md`, `feedback_fruit_empire_no_turn_permissions.md`, `feedback_game_models_go_overboard.md`, `feedback_study_game_before_designing.md`, `feedback_roblox_studio_no_killing.md`, `feedback_no_ai_attribution_footers.md`, `reference_roblox_studio_mcp.md`, `reference_roblox_text_size_setting.md`, `reference_roblox_ui_measuring.md`, `reference_glb_to_roblox_parts.md`, `reference_roblox_game_skill.md`, `reference_studio_overnight_survival.md`.

**Project repos and files:**
- `~/fruit-empire-mine`: `CAVE-OVERHAUL.md`, `DIG-AND-RACE.md` (parts), `cave/v3/ServerStorage.ZZTools.MineBuilder.lua` (palette), `pockets/MinePocketBuilder.lua` (header and palette), git logs of `cave/`, `crystal/`, `dig/`, `code/`.
- `~/fruit-empire-pets`: `NEEDS-tracker.md`, `INSTALL-PLAN.md` (start), `DESIGN.md` (lines on always day and sounds), `NOTES-after-build.md` (sound rule), `handoff-draft/DECISIONS-rows.md`, `tools/make_deploy.py`, `tools/check_deploy.py`, `tools/status_compact.lua`, git log; `ref/scripts/ReplicatedStorage.UIKit.lua` and `ReplicatedStorage.GardenConfig.lua` (the 29 Oct export).
- `~/fruit-empire-mega-rebirth`: `SPEC2.md` (styles), `src/ServerStorage.PlotFloorDecor.*.lua` headers, `src/ServerScriptService.GardenService.FloorLift.lua` header, the `deploy/` and `tools/` listings.
- `~/fruit-empire-plot-upgrades`: `src/PlotPenDecor.luau` (header, colours, levels), git log.
- `~/fruit-empire/docs/mk/` listing.
- This vault's own [[MK Verification]], [[World Tour]], [[Decisions Log]], [[UI Kit Inventory]].

## What could not be sourced or is uncertain

> [!warning] Check these before relying on them
> - **Whether the pets system is installed in Studio.** The turn began 30 Oct and deploy files exist, but no handoff or note says the install finished. ([[Pets, Eggs and Fuse Machine]])
> - **The Studio turn on 8 Nov.** STATE.md still shows the 30 Oct turn open; the safe pockets work happened on 8 Nov. Under which turn is not written down. ([[Studio Turn Rules]])
> - **Safe pockets** are described from the builder file only; nothing about them is confirmed in Studio. ([[Mine Digging and Cave-in]])
> - **The licensed-sounds rule** is the chats' practice; no quote of Samarth saying it was found. ([[Light, Day and Sound]])
> - **"Warm, kid-friendly cartoon"** is the vault brief's phrase; the sources support it (young audience, cartoon bonks) but do not quote it. ([[Samarth's Taste]])
> - **Mega floor style colour tables** were not copied; only each style's described look. ([[Colour Palettes]])
> - **Pickaxe prices:** `PICKAXE-ECONOMY.md` exists, but whether its numbers replaced the first guesses is not recorded. ([[Mine]])
> - **MCP clicks on GUI buttons** worked in the pickaxe test and failed in the Robux shop test; why is not recorded. ([[Testing in Studio]])
> - **No canvas link found** for the Troll Potion, Crystal fruit, Sky Stars, or the mine's pickaxe and ore models. ([[Claude Design Canvases]])
> - **The map's dug-tunnel drawing uses a `UIGradient`** (memory `project_fruit_empire_minimap.md`), while ui/README.md forbids `UIGradient` on boxes. Whether that counts as a break of the rule was never discussed. `[unclear]`

## Left out on purpose

- **No secrets, keys or passwords.** None are in these notes.
- **No Roblox user ids or account names.** This vault folder is a public git repo (`svaka2000/fruit-empire-vault`). The account details are in the private [PLACE.md](https://github.com/svaka2000/fruit-empire-handoffs/blob/main/PLACE.md) "Which place, and who is who".
- **The marketing side** (the TikTok batch, memory `project_fruit_empire_tiktok.md`, repo `~/fruit-empire-tiktok`) is not covered; it is not about building the game.
- **The July 2026 history** in `project_fruit_empire.md` (the Grow a Garden era, auras, the Prism Codex) is not covered; most of it was replaced by the October design.
