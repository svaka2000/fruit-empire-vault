---
tags: [economy, check]
date: 2026-10-04
sources: [~/fruit-empire/sim/economy.py, ~/fruit-empire/sim/integration_check.py, ~/fruit-empire/sim/integration_prices.py, ~/fruit-empire/sim/integration_results.json]
---
# Integration check: the economy across all 11 systems

This note checks the 11 system specs in [[02 Systems]] against the [[Stage Model]] together, not one at a time. The fixes it led to are in [[Integration Review]] and the [[Decisions Log]] (D-013 to D-033).

**Words used here**
- **Stage**: the newest rarity a player has unlocked. Stage = rebirths + 1, from 1 (Common) to 9 (Transcendent).
- **Full-plot $/s**: 35 cap fruit times the rebirth multiplier, the most a plot earns in a stage ([[Stage Model]]).
- **Plateau**: late in a stage, when the plot is nearly full of cap fruit.
- **D (shards per merge)**: the shards one merge into your newest rarity pays (D-010): 3, 7, 15, 32, 70, 155, 340, 748, 1,646 for stages 1 to 9.
- **R_s**: shards a player earns per minute in stage s (gross: Index claims plus merge drops).
- **Side income**: pets plus ores.

## How it was checked

- `~/fruit-empire/sim/integration_check.py` imports `economy.py` read-only and runs a copy of `simulate_stage` with three additions: a pet layer, forest trips, and mine floors with an Ore Bonus. Same cash needs (D-011), shard needs and shard drops (D-010) as `final.py`. 7 careers per row, attentive player (3 s merges), median.
- **Pet layer.** A "regular" pet player from [[Pets]] table 8: pets reach 4, 16, 18, 15, 21, 25, 25, 25% of full-plot $/s at the gates of stages 1 to 8. Units ramp inside a stage from the last gate (counted one size smaller) to the next. Pet income = units x income of the run-best fruit (Pets P27), so it is small early in a stage.
- **Forest trips.** From minute 4, the player leaves for 3 minutes every 15 minutes. No merging during a trip; income keeps running.
- **Mine floors.** A floor is bought the first moment shards minus its price still cover the current stage's rebirth shard need. The mine fills 3 minutes after purchase. The model's planned spending (`SPEND` in `final.py`, minus its 100K underground) is kept.
- `~/fruit-empire/sim/integration_prices.py` prints every shard price and reward as minutes of R_s.
- Re-run: `cd ~/fruit-empire/sim && python3 integration_check.py && python3 integration_prices.py` (about 5 seconds).

## Check A: pets plus ores stay near 35% of full-plot $/s at the plateau

The [[Stage Model]] rule 6 target is 35% for pets and ores together.

| Design | Pets at most | Ores at most (3 floors) | How they combine | Side income at the plateau | Verdict |
|---|---|---|---|---|---|
| As first written ([[Pets]] + [[Underground and Ores]]) | 25% | 17.5% | Ore Bonus in `cashMultiplier`, pets in `plotBase`, so the Ore Bonus also multiplies pets | (1.25 x 1.175) - 1 = **46.9%** | Over by 11.9 points |
| Underground's suggested split | 17.5% | 17.5% | one factor | 35.0% | On target, but early stages only get 17.5% (no mine before stage 8) |
| **Resolved (D-018)** | **25%** | **10.5%** | Ore Bonus on fruit income only: `(plotBase x (1 + OreBonus) + petBase) x cashMultiplier` | **35.5%** | On target |

Measured at each rebirth gate in the re-run (median of 7):

| Stage | Pets share | Ores share (floors) | Resolved total | As written total |
|---|---|---|---|---|
| 1 | 4.0% | 0 | 4.0% | 4.0% |
| 3 | 17.2% | 0 | 17.2% | 17.2% |
| 6 | 24.6% | 0 | 24.6% | 24.6% |
| 8 | 24.7% | 7.5% (1 floor) | 32.2% | 39.6% |
| 9 (first hour) | 25.0% | 10.5% (3 floors) | 35.5% | 46.9% |

Why the ore caps went down instead of the pet limit: pets exist from stage 1, ores only from about 8.5 h (Check C). The early stages need the larger share to stay near the Stage Model's 10.8 h. Floor 1 still adds 7.5% of fruit income in one purchase, the largest single income boost a player can buy with shards.

## Check B: career length

Hours to unlock Transcendent (stages 1 to 8), median of 7 careers.

| Scenario | Total | S1 | S2 | S3 | S4 | S5 | S6 | S7 | S8 |
|---|---|---|---|---|---|---|---|---|---|
| No pets, no ores | 12.95 h | 14.8 | 34.3 | 69.7 | 87.8 | 108.0 | 126.4 | 147.5 | 189.0 |
| Stage Model assumption (35% after the plateau) | 10.84 h | 14.8 | 34.3 | 61.2 | 74.7 | 89.7 | 104.9 | 120.1 | 149.6 |
| As written (46.9% at the plateau) | 10.47 h | 15.1 | 34.8 | 56.8 | 73.0 | 90.9 | 102.1 | 117.8 | 139.2 |
| **Resolved (35.5%)** | **10.59 h** | 15.1 | 34.8 | 56.8 | 73.0 | 90.9 | 102.1 | 117.8 | 145.4 |
| Even split, 17.5% each | 10.64 h | 15.1 | 34.8 | 56.8 | 73.0 | 91.2 | 103.2 | 120.1 | 143.1 |

Minutes per stage. Reading it:
- The resolved design lands within 15 minutes of the Stage Model's 10.84 h, so D-011's rebirth cash needs still hold.
- Stages 1 and 2 are 0.3 to 0.5 minutes longer with pets, because forest trips cost merging time in a short stage (the [[Pets]] note found 2 to 4 minutes with longer trips). From stage 3 on every stage is faster.
- No stage drops below 77% of its no-pets length (stage 8: 145.4 / 189.0). The Stage Model's own assumption gives 79% there.
- The [[Pets]] note's own simulation (with feeding cost and capped tames) gives 11.30 h for a regular player without ores. This layer is simpler and a little more generous; both agree that pets are worth doing without being mandatory.

## Check C: shard flows and every shard price

Gross shard income per stage in the resolved run (median):

| Stage | Minutes | Shards earned | R_s (per min) | R_s / D | Rebirth shard need (D-011) |
|---|---|---|---|---|---|
| 1 | 15.1 | 232 | 15.2 | 5.1 | 50 |
| 2 | 34.8 | 1,340 | 36.8 | 5.3 | 250 |
| 3 | 56.8 | 5,098 | 89.6 | 6.0 | 1,500 |
| 4 | 73.0 | 10,498 | 142.5 | 4.5 | 3,500 |
| 5 | 90.9 | 22,405 | 246 | 3.5 | 7,500 |
| 6 | 102.1 | 58,965 | 577 | 3.7 | 20,000 |
| 7 | 117.8 | 89,220 | 758 | 2.2 | 30,000 |
| 8 | 145.4 | 296,234 | 2,023 | 2.7 | 100,000 |
| 9 (first hour) | 60 | about 330,000 | about 5,500 | 3.3 | - |

Stage 7 is the tightest shard stage (Astral has only 5 fruits, so fewer frontier merges). Every D-priced item costs about twice as many minutes there.

**Stage-scaled prices** (shared `Config.ShardPrice`, D-017; minutes of R_s in brackets):

| Item | Units of D | Stage 1 | Stage 3 | Stage 6 | Stage 9 |
|---|---|---|---|---|---|
| Standard potion | 25 | 75 (4.9) | 380 (4.2) | 3,900 (6.8) | 41,000 (7.5) |
| Luck | 37.5 | 110 (7.2) | 560 (6.2) | 5,800 (10.1) | 62,000 (11.3) |
| Aether | 50 | 150 (9.9) | 750 (8.4) | 7,800 (13.5) | 82,000 (14.9) |
| Aqua Affinity | 25, cap 500 | 75 (4.9) | 380 (4.2) | 500 (0.9) | 500 (0.1) |
| TNT / Big / Mega / Nuke | 15 / 40 / 100 / 250 | locked | TNT 230 (2.6) | 2,300 / 6,200 / 16,000 | 25,000 / 66,000 / 160,000 / 410,000 |
| Cherry Bomb / Mortar / Cannon / Barrage | 12 / 20 / 24 / 40 | locked | 180 / 300 | 1,900 / 3,100 / 3,700 | 20,000 / 33,000 / 40,000 / 66,000 |
| Statue Greatsword / Celestial Saber | 150 / 200 | locked | 2,300 (26) | 23,000 / 31,000 | 250,000 / 330,000 |

Every repeat purchase stays between about 2 and 15 minutes of that stage's shard income; one-off showpieces (skins, the Nuke at stage 8) are 25 to 94 minutes. Nothing is a rounding error at stage 9 and nothing is out of reach at its unlock stage.

**Fixed prices** (Samarth's numbers or one-time unlocks):

| Item | Price | First affordable while keeping the next rebirth's shard need | Minutes of R_s then | Verdict |
|---|---|---|---|---|
| Boat | 1,000 | late stage 2 (about 0:44 into a career) | 27 at stage 2 | Fits: the Stage Model already plans it as the stage 2 spend |
| Underground Floor 1 | 100,000 | early stage 8, about 8.5 h (median) | 49 at stage 8 | Fits as the big late goal; later than the Stage Model's "stage 6 to 7" ([[Underground and Ores]] Q1) |
| Underground Floors 2 and 3 | 150,000 / 300,000 | about 10.8 h and 11.1 h (early stage 9) | 27 / 55 at stage 9 | Fits as the stage 9 shard sink |
| Pet slots | 10,000 / 150,000 | stage 4 to 5 / stage 8 to 9 | 70 at stage 4 / 74 at stage 8 | Fits as optional sinks |
| Ores Index claims | +500 / +1,000 (pays) | with Floor 1 | - | Small one-off |

**Late inflation is absorbed.** Through stage 8 a regular player earns about 484K shards. Rebirth needs take 213K, the model's planned spending 116K and Floor 1 100K. Stage 9 then pays about 330K an hour, which Floors 2 and 3 (450K), the second pet slot (150K), the two shard skins (580K at stage 9 prices) and stage-priced consumables soak up.

## Check D: flat amounts that break at stage 1 or stage 9

| Amount | Where | Stage 1 value | Stage 9 value | Verdict and fix |
|---|---|---|---|---|
| Royal rewards 100 to 3,000 shards | `CrownConfig.lua:261-268` | Victory = 333 frontier merges (20 times the stage 1 rebirth shard need) | Victory = 0.6 merges | **Broke.** Now `ShardReward(units)`: Defense 10, Victory 25, Contest win 40 (D-017, [[Royal Combat]]) |
| Decree rewards 350 to 900 shards | `CrownConfig.lua:209-254` | Royal Race 1st = 300 merges | 0.5 merges | **Broke.** Now units: Merge Rush 10, Royal Race 25 / 15 / 8, Crown Hunt 20, Empire Build 10 |
| Potions 5,000 / 7,500 / 10,000 | `GardenConfig.lua:1631-1704` | 322 min of shard income (more than the whole stage pays) | 0.75 min | **Broke.** Now D-priced ([[Potions]]) |
| Old TNT 2,500 | `GardenConfig.lua:1616-1626` | unaffordable | trivial | **Broke.** Replaced by the D-priced ladder; old stock refunded |
| Aqua Affinity 250 | [[Boats and Swimming]] first draft | 16 min | 0.05 min | Replaced by the [[Potions]] price (D-029) |
| Sea Crate cash: 30 s of live income per 180 s | [[Boats and Swimming]] section J | +16.7% income while sailing | same | Proportional, so it scales, but it was cash outside the side-income budget. Off in v1; shards only if shipped (D-016) |
| Boat 1,000 | Samarth | stage 2 buy | 0.2 min | Kept: one-time unlock, his number |
| Underground 100K / 150K / 300K | Samarth (100K) and [[Underground and Ores]] | out of reach | 18 to 55 min | Kept: late goals by design |
| Pet slots 10K / 150K | [[Pets]] | out of reach | 2 / 27 min | Kept: optional sinks |
| Plot Size 2.5K to 1.28M cash | live | L2 = 35% of the stage 1 rebirth cash | under 0.001 s of income | Kept, and now bought once per career (D-020) |
| Ores Index 500 / 1,000 shards | [[Underground and Ores]] | only with a mine | small | Kept: one-off thank-you |
| Weapon combat numbers, bat stats, HP, timers | several | not money | not money | Fine: flat on purpose (fairness, readability) |

## Check E: nothing lets a stage 1 player skip ahead

Cash is what blocks a rebirth ([[Stage Model]]), so the question is whether any system pays a stage 1 player cash at a higher stage's size, or lets them spend into a higher stage.

| Source | Can it pay above the player's stage? | Why not |
|---|---|---|
| Pets | No | Income uses the run-best fruit, capped at `CapRankFor(R)` ([[Pets]] P25). Wild animals roll tiers up to the stage at most; higher ones are locked. |
| Pets Index claims | No | 60 s of the cap fruit of the pet's tier, which is never above the stage. At stage 1 that is 1.2K cash per card against a 56.5K rebirth. |
| Ores | No | A share of current fruit income; a stage 1 player cannot buy a mine (100K shards against 232 held). |
| Golden Crate | No | Shards only, `ShardReward(20)` (60 at stage 1). |
| Royal and decree rewards | No | Shards only, now stage-scaled. Cash is never paid. |
| Luck | No | Capped at the spawn ceiling and refused there (D-019). |
| Explosives and weapons | No | Locked at stage 1; damage uses the lower of the two stages; weapons never move currency. |
| Migration (Placement) | Intentional | Existing players are raised to the stage their fruit already prove ([[Rarity Gating and Rebirth]] rule 22). |
| Plot Size kept through rebirth | No | Area only; capacity and spawn rate still reset. Measured +0.05 h, not faster. |

Result: no system lets a stage 1 player earn or buy their way past the stage. The fastest a player can rebirth is still set by D-011's cash need.

## Check F: one simulated player path (resolved design, regular pet player)

| Time into the career | Stage | What happens |
|---|---|---|
| 0:00 | 1 | Joins with a grey Common bat. Merges Blueberries upward. |
| 0:04 | 1 | First forest trip by swimming (about 65 s each way with rests). Feeds a Strawberry, tames a Baby Sheep. |
| 0:04 to 0:11 | 1 | Buys Plot Size 1 and 2 (kept forever, D-020). Makes the first Apple; the Max Fruit popup explains the cap. |
| 0:15 | 1 to 2 | Rebirth 1 (56.5K cash, 50 shards, all 5 Commons). Bat turns green. TNT and Cherry Bomb cards unlock. |
| about 0:44 | 2 | Has 1,250 shards: buys the boat. Trips to the forest now take about 14 s each way. |
| 0:50 | 2 to 3 | Rebirth 2. The plot starts at Dandelion (head start). Melon Mortar unlocks. |
| 1:47 | 3 to 4 | Rebirth 3. Pets are near 17% of a full plot. |
| 3:00 to 4:30 | 4 to 5 | Buys the first pet slot (10K) when potion spending allows. |
| 6:12 | 6 to 7 | Pets at the 25% limit from here. |
| 8:10 | 7 to 8 | Rebirth 7 into Eternal. Nuke unlocks (190,000 shards). |
| about 8:30 | 8 | Buys Underground Floor 1 (100K) while keeping the 100K rebirth need. Fills it with Iron in about 3 minutes: +7.5% of fruit income. |
| 10:35 | 8 to 9 | Rebirth 8: Transcendent unlocked. |
| about 10:50 and 11:05 | 9 | Buys Floors 2 and 3. Side income reaches 35.5% at the plateau. |

Times are medians from the re-run, rounded; purchases of optional items depend on how much a player spends on potions and weapons.

## What this check does not model

- Bat fights, Golden Crate pickups (about 6% extra shards at stage 3, [[Bats]]), royal rewards, the Rotten Fort and decrees. All pay shards, never cash, so they cannot shorten a stage; they only bring optional purchases earlier.
- Fruit stuns from explosives. A full nuke costs the victim 20 s of income ([[Explosives and Fruit Health]] N5), under 0.6% of any stage.
- Offline earnings and the friend bonus (the same for every design).
- Pet feeding costs and tame limits (the [[Pets]] note's own simulation models those and gives 11.30 h without ores).
