---
tags: [economy, model]
date: 2026-10-04
sources: [~/fruit-empire/sim/economy.py, ~/fruit-empire/sim/calibrate.py, ~/fruit-empire/sim/final.py]
---
# Stage model: the rebirth-gated economy

This is the backbone every other system is priced against. It turns Samarth's rule from the [[Brain Dump 2026-10-04]] into numbers:

> "the stages of the game is the rarity of fruit the player is on starting from the common category all the way to transcendent"

**Words used here**
- **Stage**: the newest rarity a player has unlocked. Stage 1 is Common (0 rebirths) and stage 9 is Transcendent (8 rebirths).
- **Merge cap**: the best fruit you can make in a stage, the top fruit of your newest rarity. Two cap fruits never merge.
- **Spawns up to**: the best fruit your plot can grow. With Spawn Rarity maxed, the plot still never grows the top 2 fruits of your newest rarity. You must merge to them.
- **Full plot $/s**: 35 fruit, all the cap fruit, times the rebirth multiplier. It is the most a plot can earn in that stage (before the Index bonus).
- **Plateau**: the point in a stage where the plot is nearly full of cap fruit and income stops rising.

![[economy-stage-income.png]]

## The stage table (proposal, from the model)
| Stage | Rarity unlocked | Rebirths | Fruits | Merge cap (best fruit) | Spawns up to | Cap fruit $/s | Full plot $/s* | Rebirth needs: cash | Rebirth needs: shards | Shards per merge into this rarity | Time in stage | Time so far |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 1 | Common | 0 | 5 | Apple | Carrot | 20 | 707 | 56.5K | 50 | 3 | 15 min | 0.2 h |
| 2 | Uncommon | 1 | 8 | Blue Lollipop | Candy Sunflower | 9.8K | 430K | 104M | 250 | 7 | 34 min | 0.8 h |
| 3 | Rare | 2 | 11 | Pear | Eggplant | 71.2M | 3.7B | 8.1T | 1.5K | 15 | 61 min | 1.8 h |
| 4 | Epic | 3 | 8 | Watermelon | Pumpkin | 59.4B | 3.6T | 10.9Qa | 3.5K | 32 | 75 min | 3.1 h |
| 5 | Legendary | 4 | 7 | Tropical Melon | Pineapple | 25.5T | 1.8Qa | 7.8Qi | 7.5K | 70 | 90 min | 4.6 h |
| 6 | Mythic | 5 | 8 | Dragon Pepper | Cactus | 31.6Qa | 2.5Qi | 13Sx | 20K | 155 | 105 min | 6.3 h |
| 7 | Astral | 6 | 5 | Paradise Petal | Serenity | 3Qi | 265Qi | 1.9Sp | 30K | 340 | 120 min | 8.3 h |
| 8 | Eternal | 7 | 8 | Solar Bloom | Lunar Cherry | 5.3Sx | 506Sx | 5.7Oc | 100K | 748 | 150 min | 10.8 h |
| 9 | Transcendent | 8 | 4 | Soul Fruit | Zenflare | 237Sx | 24.9Sp | (final stage) | - | 1646 | endgame | - |

*Before the Index bonus. Rebirth multiplier: +25% per rebirth, added (×1.25 at 1 rebirth, ×3.0 at 8).

**Read the "Rebirth needs" columns like this:** to rebirth out of stage 3 you need every Common, Uncommon and Rare fruit discovered in the Index, **8.1T cash** and **1.5K shards**. Cash goes to 0 on rebirth anyway. The shards are spent.

## How long it takes (median of 7 simulated careers)
| Stage | Uses pets + ores (3 s merges) | Ignores pets + ores | Slow merger (6 s) |
|---|---|---|---|
| 1 Common | 15 min | 15 min | 15 min |
| 2 Uncommon | 34 min | 34 min | 38 min |
| 3 Rare | 61 min | 70 min | 70 min |
| 4 Epic | 75 min | 88 min | 93 min |
| 5 Legendary | 90 min | 108 min | 109 min |
| 6 Mythic | 105 min | 126 min | 127 min |
| 7 Astral | 120 min | 148 min | 140 min |
| 8 Eternal | 150 min | 189 min | 172 min |
| **Total** | **10.8 h** | **13.0 h** | **12.7 h** |

- **Ignoring pets and ores costs about 2 hours**, which makes them worth doing without being mandatory.
- **A slow merger** (6 s per merge, for example a mobile player without Merge All) takes about 2 hours longer.
- The total is about **11 hours of active play** to unlock Transcendent. One knob scales all of it: the target minutes per stage in `calibrate.py`. Doubling them roughly doubles the total.

## Shards
| Stage | Shards held at the rebirth gate (median) | Planned spending in that stage (model) |
|---|---|---|
| 1 Common | 232 | - |
| 2 Uncommon | 1.5K | 1K |
| 3 Rare | 6.6K | 10K |
| 4 Epic | 12.8K | 10K |
| 5 Legendary | 28.3K | 15K |
| 6 Mythic | 86.2K | 120K |
| 7 Astral | 111K | 30K |
| 8 Eternal | 439K | 30K |

- Shards come from three places: the Index (one time per fruit, the existing 5 → 2,500 curve), **every merge that makes a fruit of your newest rarity** (the "Shards per merge" column, new), and royal events.
- Rebirth shard costs stay well under what a stage pays. For the modelled player, who never spends below the next rebirth's need, shards are never the thing that blocks a rebirth. Cash is. (A player who spends shards after their plot fills can be blocked: from that point no merge pays shards until the rebirth. Found by the player-path pass, [[Underground Unlock - Stage 6-7]] P3-5.) The shard cost makes the choice real: spend on potions and the boat now, or save for the rebirth and the 100K underground.
- With these rates a player who saves can afford the **100K underground during stage 6–7 (Mythic–Astral), about 6–8 hours in**. To bring it earlier, raise the per-merge drop, not the price Samarth set.
- Late shards inflate (about 440K held at the stage 8 gate). The sinks that soak this up are underground extensions, bigger rebirth costs, and cosmetics (bat skins, weapon skins). Each extension should cost more than the last.

## Rules the model uses (each is a proposal; see [[Decisions Log]])
1. **One rebirth = one rarity.** 0 rebirths: Common only, merge cap Apple. Rebirth n unlocks rarity n+1.
2. **Top 2 fruits of your newest rarity never spawn.** You merge to them. Without this, a maxed plot grows its own cap fruit, and the stage has no merging left once Spawn Rarity is maxed. It adds about 5 minutes of real merging per stage and about 20% more shard merges.
3. **Head start after rebirth 2+.** Spawn Rarity starts at the first fruit of the rarity below your newest one. Rebirth 1 starts from Blueberry ("start from the beginning", as asked). Without the head start, every rebirth replays all old rarities, and stage length grows from about 27 to 108 minutes of pure replay (measured).
4. **What rebirth resets:** cash, the cash upgrades capacity, spawn rate and spawn rarity, plot fruit, stored fruit. **What it keeps:** the Index, shards, tools (except explosives, which are refunded in shards at their pre-rebirth price), plot size, pets, the underground, the boat, the bat, royal reputation. (Plot size and the explosive refund were added by the [[Integration Review]], decision D-020; [[Plot Overhaul and Environment]] measured plot size as 10.89 h vs 10.84 h for a career, so the model is unchanged.) Today's rebirth **wipes the Index** (`RebirthLock.lua`) and repays its shards. Under gating the Index *is* the rebirth requirement, so it must be kept.
5. **Rebirth gives** the next rarity, +1 storage, +25% cash (added), and the next bat colour.
6. **Pets and ores add income once the plot is full.** The model assumes they add 35% of full-plot income. That is the target for their combined share at the plateau ([[Pets]], [[Underground and Ores]]). The specs split it as pets up to 25% and ores up to 10.5% (fruit income only, so they never compound), 35.5% in total (decision D-018). The [[Integration Check]] re-runs this model with a pet layer, forest trips and ore floors: 10.59 h to Transcendent.
7. The modelled player buys whatever upgrade is affordable (Spawn Rarity first), merges the most valuable pair every 3 s, and uses Clear when the plot is full of singles that cannot pair.

## How much of a stage is spent on a full plot (measured by the player-path pass)
From stage 3 on, the plot fills with cap fruit 33 to 40 minutes into each stage, and after that nothing can merge until the rebirth. That wait is 24 min at stage 3, 36.5 at stage 4, 54 at stage 5, 62 at stage 6, 83 at stage 7 and 109 at stage 8: 59% of stages 1 to 8 for the attentive player (45% for a 6-second merger). Re-run with `python3 plateau_wait.py`. The ranked fixes are in [[00 Paths Summary]] (P2-1).

## What the model does not include
- Offline earnings (3% of live, 5 h cap): small, and the same for everyone.
- The Friend bonus (+10% per friend), Merge All (gamepass), potions (Luck adds roughly one Spawn Rarity level for 5 minutes).
- TNT damage taken from other players. If TNT can destroy fruit, stages get longer for players who are attacked. See [[Explosives and Fruit Health]].
- Royal shard income (decrees 350–750, contests 100–3,000).

## Re-running it
```
cd ~/fruit-empire/sim
python3 probe2.py          # stage phases: when the cap fruit, spawn ceiling and plateau arrive
python3 calibrate.py 0.35  # solve cash requirements for the target minutes (side income 35%)
python3 final.py           # 7 careers x 4 player types, with shard costs and spending
python3 chart.py           # the chart above
```
