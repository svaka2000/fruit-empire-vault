---
tags: [fruit-empire, project, upgrades, ui]
status: built
updated: 2026-11-08
code: ~/fruit-empire-upgrade-tree
handoffs: [21]
---
# Upgrade skill tree

Back to [[Project Status Board]].

## What players get

The Upgrade button opens a skill tree of **15 glossy hexagon tiles**. No new upgrades and no price changes: the three panel upgrades are split into tiles. Max Capacity (green): Capacity I, II, III. Spawn Rate (blue): Spawn Rate I, II, III. Spawn Rarity (purple): one tile per rarity, each opened by its rebirth. Plot Size stays on the pen sign. A tile opens when the one before it is maxed; links glow gold between maxed tiles. Holding Upgrade buys up to 5 a second. On phones the tree pans and pinches; on computers the wheel zooms. (STATE.md "22 Oct: the Upgrade skill tree")

## Samarth's words

He showed a picture of Tower Defense Simulator's skill tree: "something like this for upgrades... extremely high quality", then "its called a skills tree menu", and "you draw the hex art". (memory `project_fruit_empire_upgrade_tree.md`)

## Where it lives

- **In the place:** `StarterPlayerScripts.UpgradeMenu` (rewritten) and `ReplicatedStorage.GardenConfig.UpgradeTree` (every number and word, plus `PanelFit`; required directly, GardenConfig does not load it). `UpgradeShow` and `ZZTools.GuiFitCheck` also changed. The server is unchanged. Backup and the old menu: `ServerStorage.Upgrades_backup_2026-10-22` (the handoff has an undo snippet).
- **Local:** `~/fruit-empire-upgrade-tree` (`before/`, `src/`, `tools/hexart.py` and `linkart.py`, `RESEARCH.md` with cited skill-tree research). No Claude Design canvas: the hex art was drawn in code.
- **Handoff:** [PR #21](https://github.com/svaka2000/fruit-empire-handoffs/pull/21).

> [!note] Larger text
> The panel takes most of the "Larger" text setting's extra off, so tile names fit their hexagons. See [[UI Kit and Text Rules]].

## Open items

- Weilin's approval of the restyled panel.
- Not tested with two players, on a real phone (two-finger pinch, holding with a finger) or with a gamepad.
- Nobody has listened to its sounds (the base-upgrade show's, pitched).
- Screen sizes were checked by calculation (`PanelFit`) and `GuiFitCheck`; `UIOverlapCheck` only at Studio's 1529 x 671.
- Delete the backup only once both partners accept.

Sources: [STATE.md](https://github.com/svaka2000/fruit-empire-handoffs/blob/main/STATE.md); memory `project_fruit_empire_upgrade_tree.md`.
