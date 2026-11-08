---
tags: [fruit-empire, project, mine, crystal-fruit]
status: built
updated: 2026-11-08
code: ~/fruit-empire-mine/crystal
handoffs: [17]
---
# Crystal fruit (the mine's purpose)

Back to [[Project Status Board]].

## What players get

Ores in the mine drop **geodes** (from the Amethyst Pick up). Crack a geode on a fruit on your plot and it becomes a **Crystal fruit**: it pays shards over time (about 0.5 units a minute at your newest rarity), at most 5 on a plot. The crystal rides through merges. A rebirth clears them. (memory `project_fruit_empire_mine.md`, 22 Oct; `~/fruit-empire-mine/MINE-PURPOSE.md`)

## Why

Samarth wanted the mine to feed the merge and rebirth loop, the way pets will, and rejected ore merging and ores with no use. On 22 Oct, ~02:35 PDT: "Go with Geodes and Crystal fruit as the mine's purpose. Use the defaults in section 6 of MINE-PURPOSE.md for now." (memory `project_fruit_empire_mine.md`)

## Where it lives

- **In the place:** `GardenConfig.MineGeodes`, `GardenService.Geodes`, `CrystalFruitClient`, `ReplicatedStorage.Tools.Geode`. Changed: GardenConfig loader, GardenService, Tools, Weilin's `Mining` (3 lines), `Merging`, `Storage` (refuses a Crystal fruit), `Carrying` (Drop lets a Crystal fruit onto a full plot, one over), `Persistence` + `GardenStore` (save field `crystal`), `GardenClient`, `ShopMenu`. Backup `ServerStorage.Backup_2026-10-22_before_crystal_fruit`.
- **Local:** `~/fruit-empire-mine/crystal` (git, commit 160a8be), with `tests/run.sh` and the rebase tools.
- **Handoff:** [PR #17](https://github.com/svaka2000/fruit-empire-handoffs/pull/17).

## Open items

- Samarth's five waiting items in DECISIONS.md: the Drop rule, the Geode cavern bonus, idle shards (about 112k an hour at stage 8), stage 9, and the look (crystals hard to see on purple fruit; the CRYSTAL! popup sits on the name tag).
- **Before any publish: a real save and load** of the `crystal` field.
- Weilin's OK on the `Mining` change; both people's OK on the new UI.
- Not tested on a phone or with two players. Numbers are first guesses.

Sources: [STATE.md](https://github.com/svaka2000/fruit-empire-handoffs/blob/main/STATE.md) "22 Oct: Crystal fruit" and "Open items"; memory `project_fruit_empire_mine.md`.

Related: [[Mine]], [[Mega Rebirth]] (the Mega patch touched `Geodes`).
