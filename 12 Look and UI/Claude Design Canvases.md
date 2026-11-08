---
tags: [fruit-empire, design, canvas, links]
status: current
updated: 2026-11-08
---
# Claude Design canvases

A Claude Design canvas is a private page on claude.ai where a chat draws design ideas before anything is built in Studio. Only Samarth's account can open these links. Back to [[Knowledge Base - Start Here]].

> [!tip] How they are used
> Concepts go on the canvas first, drawn from the game's real kit art and real in-game screenshots. Samarth picks one. Then a Studio turn builds it. See [[Samarth's Taste]].

| Project | Canvas | What is on it | Source |
|---|---|---|---|
| Pets, eggs, Fuse Machine | [Fruit Empire Pets UI](https://claude.ai/artifact/GApyrQArNP1fabtFW5sPeh) | 15 screens: Fuse Machine (4, asleep and waking), PETS panel (TEAM, BAG, EGGS, GUIDE), hatch reveal, forest scout and chase, the Mist, the HUD Pets button | memory `project_fruit_empire_pets.md`; pets commit f1f7da1 |
| Mega Rebirth | [floor styles](https://claude.ai/artifact/Uu3rtKkKe4gCPbqHf9PQ2K) | Sky Keep, Hanging Gardens, Sky Platform concepts | memory `project_fruit_empire_mega_rebirth.md` |
| Rebirth panel | [layouts](https://claude.ai/artifact/T4w3TH8QntJtAcmViUnKuP) | Now, A (picked), B, C, each with a Mega Rebirth state | memory `project_fruit_empire_rebirth_panel.md` |
| Robux shop | [shop icons](https://claude.ai/artifact/LiXjHcL2S5qAukYwypaPnn) | Claude's original pass and pack icons (Samarth's ChatGPT versions are in `~/Downloads/fruit-empire-shop-icons/chatgpt/`) | memory `project_fruit_empire_robux_shop.md` |
| Plot Castle | [castle concepts](https://claude.ai/artifact/57uK9K1QhRiS2z7h85AHP7) | Castle, Five Worlds, Fruit Factory concepts, plus in-game shots of the built castle | memory `project_fruit_empire_plot_castle.md` |
| Base-upgrade show | [LEVEL UP! titles](https://claude.ai/artifact/JksqFPNqBgW27AVdL3KJcU) | Title art (LEVEL UP!, CAPACITY UP!, MAX LEVEL!) and rings; 9 images uploaded to Roblox | memory `project_fruit_empire.md` (20-21 Oct entry) |
| Bats | [bat board](https://claude.ai/artifact/AKMm89kKmG6ZrgheXhoHyh) | The nine fruit bats | memory `project_fruit_empire_bats.md` |
| Cave overhaul | [cave boards](https://claude.ai/artifact/LfLzAVMycG4NvNpiRVK3VA) | Timber, caverns and light concepts; files also in `~/fruit-empire-mine/cave/design/` | `~/fruit-empire-mine/CAVE-OVERHAUL.md` |
| Digging and egg races | [dig boards](https://claude.ai/artifact/3coTyLchg2MKeqXr3Nx8ZG) | Dig, cave-in and egg race screens | memory `project_fruit_empire_mine.md`; `~/fruit-empire-mine/DIG-AND-RACE.md` |
| Map panel | [Fruit Empire Map UI](https://claude.ai/artifact/1i6LMz5UaM2VFFQzgpiC18) | Map frames, pins, tabs; a "Mines" board | memory `project_fruit_empire_minimap.md` |

## Other claude.ai pages (not canvases)

| Page | What it is | Source |
|---|---|---|
| [Sky islands handoff prompt](https://claude.ai/artifact/NCeQwgY6rnbgBRBxvRSnNZ) | The prompt Samarth was to use to plan the cloud island's purpose with ChatGPT (3 Oct) | memory `project_fruit_empire.md` (3 Oct entry) |
| [Steal An Egg teardown](https://claude.ai/artifact/MB8KQY5r4xVgmxiJ7DhC5W) | Research page on the reference game and a growth plan (26 Sep) | memory `project_fruit_empire.md` (26 Sep entry) |

## Projects with no canvas found

- **Upgrade skill tree:** Samarth said "you draw the hex art"; the hex art was drawn in code (`~/fruit-empire-upgrade-tree/tools/hexart.py`). (memory `project_fruit_empire_upgrade_tree.md`)
- **Troll Potion, Crystal fruit, Sky Stars, mine pickaxes and ores:** no canvas link was found in the sources. Local art lives in each project's repo (for example `~/fruit-empire-troll/art`). `[not found]`

## Pulling kit art for a canvas

Kit art is pulled read-only from Studio with `AssetService:CreateEditableImageAsync`, sent back in slices from `execute_luau`, then rebuilt as PNGs (`~/fruit-empire/rebirth-panel/tools/unkit.py`, `~/fruit-empire-pets/tools/pull_kit.py`). HttpService stays off. Most kit frames are exact 9-slices, so they shrink to a few KB. (memory `project_fruit_empire_rebirth_panel.md`, `project_fruit_empire_pets.md`)

Related: [[UI Kit and Text Rules]], [[Project Status Board]].
