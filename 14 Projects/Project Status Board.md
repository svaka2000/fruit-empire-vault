---
tags: [fruit-empire, projects, status, moc]
status: current
updated: 2026-11-08
---
# Project status board

Every Fruit Empire project at a glance, as of 8 Nov 2026. Back to [[Knowledge Base - Start Here]].

> [!warning] Nothing since 4 Oct is live
> The last publish on record is about 04:00 PDT on 4 Oct 2026. Everything below is **in Studio only** until a person publishes. (STATE.md "Live game and Studio")

**Status words:** *built* = in Studio and Play-tested solo; *waiting* = built but blocked on someone; *in progress* = being built; *on hold* = Samarth said wait.

| Project | Status | Handoff PRs | Local code | Biggest open item |
|---|---|---|---|---|
| [[Pets, Eggs and Fuse Machine]] | in progress (install not confirmed) | #6, #7, #8; pets turn open since 30 Oct | `~/fruit-empire-pets` | Confirm the install; publish the pets handoff |
| [[Mine]] (pickaxes, ores, cave) | built | #11 to #16 | `~/fruit-empire-mine` | Tougher ores (phase 2); phone frame rate |
| [[Mine Digging and Cave-in]] (digging, egg races, safe pockets) | built; safe pockets in progress 8 Nov | #20, #22, #23, #27 | `~/fruit-empire-mine/dig`, `~/fruit-empire-mine/pockets` | Safe pockets; 2-player race test |
| [[Crystal Fruit]] | built | #17 | `~/fruit-empire-mine/crystal` | Real save and load before publish |
| [[Nine Bats]] | built | #18 | `~/fruit-empire-bats` | 2-player hit test |
| [[Troll Potion]] | built (shard version) | #19 | `~/fruit-empire-troll` | 2-player test; Robux version parked |
| [[Mega Rebirth]] | built | #31 | `~/fruit-empire-mega-rebirth` | Rejoin test before publish |
| Aether cloud plot | on hold | none | none | Wait for Samarth ([[Mega Rebirth]]) |
| [[Rebirth Panel]] (and harder rebirths) | built; Mega look next | #28, #30 | `~/fruit-empire/rebirth-panel` | Mega look + Floor style button |
| [[Robux Shop]] | waiting | #29 | `~/fruit-empire/robux-shop` | Passes and products made on Weilin's account |
| [[Plot Castle]] | built | #9, #24 | `~/fruit-empire-plot-upgrades` | Weilin's OK; Lock dome height |
| [[Upgrade Skill Tree]] | built | #21 | `~/fruit-empire-upgrade-tree` | Phone pinch test |
| [[Map Panel]] | built | #10, #25, #26 | `~/fruit-empire-minimap` | Redraw after world changes |
| [[Forest Animals]] | built (models) | #7, #8 | `~/fruit-empire-pets/forest-animals` | Rigged import by Samarth |
| [[Sky Stars]] | built (9 Oct) | none (before the repo) | `~/fruit-empire` | Samarth's go before publish |

PR links are on [[Handoffs Repo Links]].

## Waiting on Weilin

Weilin's OK is needed on every new or restyled screen and on changes to the scripts Weilin wrote (`Mining`, `Mine`, `PickaxeClient`, `MineBuilder`). That covers nearly every row above. No handoff from Weilin's Claude exists yet. ([ui/README.md](https://github.com/svaka2000/fruit-empire-handoffs/blob/main/ui/README.md); STATE.md "Open items")

## Waiting on Samarth

- Import the rigged animals (`~/Downloads/forest-animals-rigged`) and upload the Pets button art.
- Make the Robux passes and products (on Weilin's account) and say when.
- Home button art in the old glossy style.
- The design calls in [DECISIONS.md](https://github.com/svaka2000/fruit-empire-handoffs/blob/main/DECISIONS.md) "Waiting for a person".
- Placeholder art: REBIRTH and MAX FRUIT titles, padlock and tick icons, Feather Falling and Luck potion art, the Aqua Affinity picture and badge, bat swing art, shared bar art, the Troll Potion badge. (STATE.md "Open items")
- Before the next publish: delete `Workspace._AssetViewer`; do the Mega rejoin test and the Crystal fruit save test. (STATE.md)

## Not started (from the old roadmap)

M10 Boats and docks is next on the October roadmap; M14-M15 sea weapons and M18 final art come later. M16-M17 (sword animations, blade skins) were for the removed arena; whether to drop them is open. (STATE.md "Next up"; [[Roadmap]])
