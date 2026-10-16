---
tags: [system, design]
date: 2026-10-04
status: proposed
depends_on: [Rarity Gating and Rebirth, Pets, Plot Overhaul and Environment, Explosives and Fruit Health]
---
# Underground and Ores

Source of truth: [[Brain Dump 2026-10-04]]. Economy backbone: [[Stage Model]]. Decisions: [[Decisions Log]]. UI parts: [[UI Kit Inventory]].

Code citations use short file names. `GardenService.lua` is `ServerScriptService.GardenService.lua`. `Carrying.lua`, `Merging.lua`, `Storage.lua`, `Tools.lua`, `RebirthLock.lua`, `Persistence.lua`, `Progression.lua`, `FruitFactory.lua`, `Upgrades.lua`, `PlotRegistry.lua`, `Ambient.lua`, `Crown.lua`, `CrownArena.lua` and `DevPanel.lua` are the `ServerScriptService.GardenService.*.lua` modules. `GardenStore.lua` is `ServerScriptService.GardenStore.lua`. `GardenConfig.lua` is `ReplicatedStorage.GardenConfig.lua`. Client scripts are `StarterPlayer.StarterPlayerScripts.<Name>.lua`, cited as `<Name>.lua`. All are in `/Users/samarthvaka/fruit-empire/src-export-2026-10-04`.

## 2. Why

Samarth's words (verbatim):

> also add a base upgrade you can buy, but for 100k shards. you get access to an underground portion of your base. the underground should look super good. and you can extend it further. we are going to make base upgrades scale vertically, we will start below, and then we can later add vertical scaling that towers upwards as well, but we will start with underground scaling. this will give you acccess to ores. you can mine and merge ore to get better ore, before we expand start with basic ores like coal which merge to iron and we can expand later, make this a category in the index too.

> i sent a reference for how the ores should look, higher tier ores should be glow.

> the ores merge and the animals merge should both be reasonable in the amount of moeny they give meaning they should match what stage of the game a player is in, this is because especcially since this is a merge game earnings scale exponentially, this means that everything needs to correspond with the stage of the game a player is in. you can use rebirths to control and organize this.

> make this super planned and tell me exactly what assets you need dont make slop unless it looks clean and studded.

> https://create.roblox.com/store/asset/105342635739603/Studed-Mountain - you can figure out the uses for all the stuff im sending.

> what i noticed with the king feature in the sky island is that it was a bunch of dopamaine slop. instead everything should be structured. like for players its going to be confusing. all the UI should never overlap and also everything should be super clean.

What the system is for. The underground is the first "vertical" base upgrade: a private cave under your plot that you buy with shards and extend downward one floor at a time. In it you mine studded rock nodes for ore, carry ore the same way you carry fruit, and merge two of the same ore into a better one (Coal + Coal = Iron). Ores lying in your mine raise your fruit income by a percentage, so they always pay at the size of your current stage, and they fill a new Ores page in the Index. It is a long-term shard sink and a showpiece, not a second main game.

Words used in this note:
- **Mine**: one player's underground. It has 1 to 3 **floors**.
- **Mine Lift**: the lift inside the stone underground-entrance hut at the back-right corner of your lot ([[Plot Overhaul and Environment]] builds the hut; this system puts the lift cage, lamps, sign and prompts inside it). It takes you down to Floor 1 and back up.
- **Node**: a studded rock with ore gems in it, sitting in a fixed spot on a floor. Mining it empties its HP (hit points). It then breaks, drops one ore, and grows back.
- **Ore**: the item a node drops. You carry it above your head like fruit. Two of the same ore merge into the next one.
- **Ore slot**: one place for an ore in your mine. The number of slots is set by how many floors you own.
- **Ore Bonus**: the sum of the boosts of every ore lying in your mine, as a percentage. It multiplies your fruit income only (not pet income), so pets and ores never compound (decision D-018).
- **plotBase**: the income of the fruit on your plot before any bonus, the existing function at `FruitFactory.lua:39-45`.
- **Full-plot $/s** and **plateau**: as defined in [[Stage Model]] (35 cap fruit times the rebirth multiplier; the point where income stops rising in a stage).
- **Visitor**: another player walking around your mine. Visitors can look but not touch.
- **Yield-free**: a block of server code with no `task.wait` or web call in it, so nothing else can run in the middle (the term the codebase already uses, `RebirthLock.lua:63-67`).

## 3. Player view

1. You save shards. The [[Integration Check]] re-run says a typical player has 100K spare shards (after keeping the next rebirth's shard cost) early in the Eternal stage, about 8.5 hours in; a player who skips potions and weapons gets there in late Astral.
2. At the back-right corner of your lot, stepping stones lead to a small stone hut with locked wooden doors and a gold padlock ([[Plot Overhaul and Environment]]). Inside the doorway you can see the **Mine Lift**: an iron cage under a pulley, a sign that reads MINE, and three dark lamps. You press E **Open Mine**.
3. The **Mine** panel opens in the middle of the screen. It looks like the Index panel: blue title bar, dark body. It shows three cards: Floor 1 (100K shards, Buy), Floor 2 (150K, locked) and Floor 3 (300K, locked). You press Buy, then Yes on the "Buy Floor 1?" check.
4. The hut doors swing open, the first lamp lights amber, and a green notice says "Floor 1 unlocked!".
5. You step into the cage and press E **Go Down**. The screen fades to black for about half a second. You arrive in a warm, studded cave: grey stepped stone walls, timber beams, hanging lanterns, amber crystals glowing in the corners, a campfire by the lift, and six grey rocks dotted with black gems.
6. You walk up to a rock and press F **Mine** (tap the prompt on a phone). Your character takes out a pickaxe and chops every half second. Stone chips fly, and a small bar above the rock drains. After 10 chops the rock bursts and a lump of **Coal** hops out. The rock leaves a low pile of rubble and grows back in 25 seconds.
7. You press E on the Coal to pick it up. The first time, it sparkles and Coal enters your Ores Index. You carry it to another Coal and press E **Merge**. The two flash white and become **Iron**, which sparkles as a new discovery.
8. Every ore lying in the cave adds to your **Ore Bonus**: Coal +0.3%, Iron +0.75%. The board next to the lift shows "Ores 6 / 10, Ore bonus +3.6%, +$X/s" (4 Iron and 2 Coal). Small "+$" numbers float over your ores each income tick.
9. When the cave holds 10 ores and no odd Coal is left to pair, Mine is refused. While two Coal can still be merged it says "Your ore floor is full. Merge ores to make room." Once all 10 are Iron it says "Your floor is full of Iron, the best ore for now. More slots come with the next floor." Ten Iron is +7.5%, the most Floor 1 can give.
10. You open the Index. Three tabs sit on the panel's left edge: Fruits, Pets, Ores. The **Ores** tab shows Coal and Iron cards. You claim each one for shards and money.
11. Later (Eternal stage, about 9 hours in) you buy Floor 2 from the board by the lift. The stone slab over the spiral stair in the back corner crumbles. You walk down into a cooler, blue-lit cave with two more rocks and two more ore slots. Floor 3 (early Transcendent) is a violet crystal grotto.
12. You walk back to the lift on Floor 1 and press E **Go Up**. Friends can press **Visit Mine** on your lift to walk around your cave.
13. You rebirth. The mine and its ores stay. Your fruit plot starts over, and because ores pay a share of your fruit income, your ore income starts small again and grows back with your plot.

## 4. Rules

Every rule is a proposal. Numbers come from section 5.

**Buying and floors**
1. The mine is bought with shards only, one floor at a time, in order: Floor 1 costs 100,000, Floor 2 costs 150,000, Floor 3 costs 300,000. No cash and no Robux are involved.
2. A purchase succeeds only if all of these are true on the server: the profile is loaded, the player has a plot, the player is not mid-rebirth, the floor number the client sent equals owned floors + 1, and shards are at least the price. Shards are taken and the floor count goes up in one yield-free step. Then the profile is force-saved.
3. A floor is never sold, refunded or lost.

**Access**
4. Every player who has a plot has a Mine Lift. It stands inside the underground-entrance hut that [[Plot Overhaul and Environment]] builds in the back-right corner of every lot (x +40 to +50, a 229 to 243 in that note's plot frame, outside the reserved pen zone at every plot size). The hut's `LiftAnchor` attachment marks where the cage goes. Plots with no owner have no Mine Lift (the hut stays closed). Changed by the [[Integration Review]] (conflict C14, decision D-023): one entrance, not two.
5. Before Floor 1 is bought, the hut doors are shut and padlocked, the three depth lamps are off, and the only prompt is Open Mine (owner only). After, the doors stand open (Plot Overhaul rule D3, read from `UndergroundFloors`) and one depth lamp is lit per owned floor.
6. Floor 1 is reached only through the Mine Lift: the server fades the screen and teleports the player. Floors 2 and 3 are reached by walking down a spiral stair from the floor above. The stair to an unbought floor is sealed with a stone slab.
7. The lift refuses, with a red warning, if the player is carrying anything (fruit, tool or ore).
8. If the owner has Floor 1, any other player can press Visit Mine on that lift. At most 6 visitors can be in one mine. Visitors cannot mine, pick up, merge, buy or claim anything there. Nothing in a mine can be taken by anyone but its owner, and nothing in a mine can leave it.

**Mining**
9. Node sockets per floor: Floor 1 has 6, Floors 2 and 3 have 2 each. In v1 every node is a Coal Node with 200 HP that drops 1 Coal when broken.
10. Pressing Mine (keyboard F, gamepad Y, tap on touch) on your own intact node starts auto-mining. The server swings once every 0.5 s for 20 damage. Each swing needs: you are within 10 studs of the node, you carry nothing, the node is intact and you are still mining that node.
11. Mining stops when you press Mine again, move out of reach, pick something up, die, leave the mine, or the node breaks.
12. Mining can start only if (ores in your mine, counting one you carry) is below your ore slots, OR the number of Coal in your mine (counting one you carry) is odd. The same test runs at the moment the node would break. If it fails then, the node stays at 1 HP and mining stops with a warning. The warning depends on why: if two ores of the same kind below the top ore can still merge, "Your ore floor is full. Merge ores to make room."; if every ore is the top ore (in v1, all Iron), "Your floor is full of Iron, the best ore for now. More slots come with the next floor." (with 3 floors owned: "Your mine is full of Iron, the best ore for now."). Two Iron cannot merge in v1, so the first line would send the player to look for a merge that does not exist. (Split by the player-path pass, [[Underground Unlock - Stage 6-7]].)
13. A broken node grows back to full HP 25 s after it broke. Node HP is never saved: after a rejoin every node is full.

**Ores**
14. A mined ore lands within 4 studs of its node, on the side facing the player, on the cave floor.
15. Ores are carried like fruit. E on your own ore picks it up. E on a matching ore of yours while carrying one merges them. Two of the same ore make one of the next ore. In v1 that is Coal + Coal = Iron. Iron is the top ore in v1, so two Iron do not merge.
16. A merge is one yield-free step: both inputs are removed and the result is placed in the same server step, where the target ore stood.
17. Drop puts the carried ore on the floor of the mine you are standing in, near your feet. Drop is always allowed inside your own mine; it never changes the ore count. Ores cannot be dropped anywhere else, cannot go into Storage, and cannot ride the lift.
18. If a carried ore ends up outside its mine for any reason (death, leaving the server, a teleport), the server puts it back on a free spot of Floor 1 of that mine.
19. An ore enters the Ores Index the first time you pick one up (Coal) or merge into one (Iron). Claiming an Ores Index card pays its shards (Coal 500, Iron 1,000) plus 60 seconds of that ore's income at the moment of the claim. Ores are not part of the rebirth Index requirement.

**Income**
20. Each ore lying in your mine adds its boost to your Ore Bonus: Coal +0.3%, Iron +0.75%. A carried ore adds nothing. The Ore Bonus is capped at 7.5% with 1 floor, 9% with 2 and 10.5% with 3 (slots x the Iron boost). With the pet limit of 25% ([[Pets]] P27) the two reach 35.5% of full-plot $/s at the plateau, the [[Stage Model]]'s 35% target (decision D-018; the earlier 12.5 / 15 / 17.5% plus pets would have reached 46.9%, [[Integration Check]]).
21. The Ore Bonus applies to fruit income only: income per second = (plotBase × (1 + Ore Bonus) + petBase) × (1 + friends bonus) × (1 + Index bonus) × rebirth multiplier. `petBase` is [[Pets]] P27. The Ore Bonus is not a factor of the cash multiplier, so it never multiplies pet income.
22. Offline earnings include the Ore Bonus, on fruit income only (a mine is a permanent property of the account, like rebirths). The friends bonus stays out of offline pay, as today.
23. So ore income is always that percentage of your current fruit income. At the plateau of any stage it equals Ore Bonus × that stage's full-plot $/s (times the Index and friends bonuses).

**Rebirth, saving, leaving**
24. Rebirth keeps every floor, every ore, the node layout and the Ores Index (as [[Decisions Log]] D-009 keeps "underground"). A carried ore is put back on the floor before the reset.
25. Saved: floors owned, ore counts by name, Ores Index states. Not saved: ore positions, node HP, who is in the mine.
26. When the owner leaves, the mine is saved first, then removed from the world. Visitors inside are sent to their own plot's spawn with a notice.

**Safety and look**
27. Inside any mine, explosives cannot be used (decision D-031; [[Explosives and Fruit Health]] rule 34 agrees) and bats or weapons cause no knockback or damage. Each floor's interior box is tagged `BatSafeZone` ([[Bats]] B22b), and the bat safe band reaches 240 studs below the soil (Bats B22a).
28. Entering a mine eases the cave lighting in over 0.6 s. Leaving eases it out over 0.6 s.

## 5. Numbers

### 5.1 Ore ladder (v1)

| Level | Ore | Rarity band | Frame and colour | How you get it | Boost per ore | Two of the level below give |
|---|---|---|---|---|---|---|
| 1 | Coal | Common | COMMON frame, grey-blue | Mined from any node | +0.3% | (none) |
| 2 | Iron | Common | COMMON frame, grey-blue | Merge 2 Coal | +0.75% | 2 Coal = +0.6%, so the merge pays +25% and frees a slot |

- Ore levels count up (1 is the worst). This is the opposite of fruit rank, where 1 is the best (`FruitIndex.lua:5`). The OreIndex module (section 9) uses `level` to avoid that trap.
- The growth per level is 2.5. A merge-2 game needs more than 2 or merging loses money ([[Rebirth Economies in Merge and Incremental Games]], section 5). Iron is worth more than two Coal, so a merge never feels like a loss ([[Mining and Ore Merging - Reference Games]], "What to avoid").
- Measured in fruit: at the plateau, one Iron earns 0.75% × 35 = 0.26 of one cap fruit, and one Coal earns 0.105 of one cap fruit. That ratio is the same at every stage. (Boosts were 0.5% and 1.25% before the [[Integration Review]] set the pets and ores split, D-018.)

Later ladder, sketch only, not built (proposal): one band of two ores per rarity colour, each band living on its own deeper floor. Common Coal, Iron. Uncommon Copper, Emerald. Rare Sapphire, Silver. Epic Amethyst, Obsidian. Legendary Gold, Topaz. Mythic Ruby, Garnet. Astral, Eternal and Transcendent: made-up glowing ores, names to be picked by Samarth. The order follows the common pattern in [[Mining and Ore Merging - Reference Games]] (section 1). When a band ships, the boost table in 5.1 and the caps in 5.2 must be re-solved so pets (25%) plus ores still stay near the 35% budget, for example by giving deeper bands a higher cap on new floors and checking the sum with `integ.py` ([[Integration Check]]).

Glow ramp for the later ladder (proposal, from [[Mining and Ore Merging - Reference Games]] proposal 5):

| Band | Gems | Light | Particles |
|---|---|---|---|
| Common (v1) | SmoothPlastic, no glow. Iron gets a faint glint (one Shiny-01 sparkle, rate 0.4/s). | none | Iron glint only |
| Uncommon | SmoothPlastic, brighter colour | none | none |
| Rare, Epic | Neon gems in the band colour | one PointLight on an Attachment, Brightness 0.8, Range 8, no shadows | none |
| Legendary, Mythic | Neon gems | as above, Range 10 | Shiny-01 at 2/s |
| Astral, Eternal | Neon gems plus Neon veins in the rock | Range 12 | Shiny-01 at 3/s |
| Transcendent | colour cycling on the gems, reusing `Config.TranscendentShimmer` | Range 12 | Stars-01 at 2/s |

### 5.2 Floors and prices

| Floor | Name | Price (shards) | Ore slots added | Total slots | Nodes added | Total nodes | Ore Bonus cap (all Iron) | Look |
|---|---|---|---|---|---|---|---|---|
| 1 | Coal Seam | 100,000 | 10 | 10 | 6 | 6 | 7.5% | warm grey stone, amber crystals |
| 2 | Iron Hollow | 150,000 | 2 | 12 | 2 | 8 | 9.0% | cool blue slate, ice-blue crystals |
| 3 | Crystal Grotto | 300,000 | 2 | 14 | 2 | 10 | 10.5% | indigo slate, violet crystals |
| All | | 550,000 | | 14 | | 10 | 10.5% | |

How the prices sit against the [[Stage Model]] (shard figures from section 5.6):
- Floor 1 = 100K, Samarth's number. With the rebirth shard costs (D-011) and the model's other spending paid, this note's own run put it in the middle of stage 7 (Astral), about 7.2 h in; the [[Integration Check]] re-run, which adds forest trips for pets, puts it early in stage 8 (Eternal), about 8.5 h in. The Stage Model note says "stage 6-7".
- Floor 2 = 1.5 × the stage 8 rebirth shard cost (100K). It is about a third of the 439K shards a player holds at the stage 8 gate. Bought in stage 8 (Eternal), about 9.1 h in.
- Floor 3 = 3 × the stage 8 rebirth shard cost. Bought about 0.5 h into stage 9 (Transcendent), about 11.6 h in. Stage 9 earns about 190K shards per hour, so it is the right sink for late shard inflation, as the Stage Model asks ("Each extension should cost more than the last").

The ore caps are front-loaded on purpose: Floor 1 gives 7.5 of the 10.5 points, so the 100K purchase is the one that matters. Floors 2 and 3 add 1.5 points each, plus two nodes, a new look and (later) a new ore band.

### 5.3 Ore income formula and worked values

Formulas (decision D-018, set in the [[Integration Review]]):
- `OreBonus = min( sum of boost(ore) over ores lying in the mine , BonusCap[floors] )`
- `income per second = (plotBase × (1 + OreBonus) + petBase) × cashMultiplier`, where `cashMultiplier = (1 + FriendBonus × friends) × (1 + indexBonus) × rebirthMultiplier` is unchanged (`Progression.lua:332-336`) and `petBase` is [[Pets]] P27. The Ore Bonus touches fruit income only, so it never compounds with pets.
- One ore's share, for display: `ore $/s = boost(ore) × plotBase × cashMultiplier`. The server publishes the total as the `OreIncome` attribute.
- At the plateau: `ore $/s = boost(ore) × full-plot $/s × (1 + indexBonus) × (1 + friends)`.

Worked values at the plateau, before the Index and friends bonuses (full-plot $/s from the [[Stage Model]] table):

| Stage | Rarity | Full-plot $/s | One Coal (0.3%) | One Iron (0.75%) | Floor 1 full: 10 Iron (7.5%) | Floors 1-3 full: 14 Iron (10.5%) |
|---|---|---|---|---|---|---|
| **1** | **Common** | **707** | **2.12** | **5.3** | **53** | **74** |
| 2 | Uncommon | 430K | 1.29K | 3.23K | 32.3K | 45.2K |
| **3** | **Rare** | **3.74B** | **11.2M** | **28M** | **280M** | **392M** |
| 4 | Epic | 3.64T | 10.9B | 27.3B | 273B | 382B |
| 5 | Legendary | 1.78Qa | 5.36T | 13.4T | 134T | 187T |
| **6** | **Mythic** | **2.49Qi** | **7.47Qa** | **18.7Qa** | **187Qa** | **261Qa** |
| 7 | Astral | 265Qi | 795Qa | 1.99Qi | 19.9Qi | 27.8Qi |
| 8 | Eternal | 506Sx | 1.52Sx | 3.8Sx | 38Sx | 53.2Sx |
| **9** | **Transcendent** | **24.9Sp** | **74.7Sx** | **187Sx** | **1.87Sp** | **2.61Sp** |

Worked examples:
- Stage 1: 707 × 0.0075 = 5.3 $/s per Iron. Ten Iron = 53 $/s, which is 7.5% of 707. (A player only has a mine at stage 1 through dev tools; the formula still holds.)
- Stage 3: 3.74B × 0.0075 = 28M $/s per Iron. Floors 1-3 full = 0.105 × 3.74B = 392M $/s.
- Stage 6: 2.49Qi × 0.0075 = 18.7Qa $/s per Iron. Floor 1 full = 187Qa $/s.
- Stage 9: 24.9Sp × 0.0075 = 187Sx $/s per Iron. Floors 1-3 full = 2.61Sp $/s.
- With pets at their 25% limit, a stage 9 plateau earns 24.9Sp (fruit) + 2.61Sp (ores) + 6.22Sp (pets) = 33.7Sp, which is 35.5% above the plot alone.

Why a percentage of current fruit income and not a fixed share of full-plot $/s. Ores are kept through rebirth (rule 24). If an ore paid a fixed share of the new stage's full-plot $/s, it would pay that the moment you rebirth, while your fresh plot still earns almost nothing. Example: right after the 6th rebirth the plot holds about 3 Passionfruit (61.4T $/s each, the first Mythic fruit, per D-006's head start). That is 184T $/s base, 460T $/s after the ×2.5 rebirth multiplier. With the percentage rule, Floor 1 full adds 7.5% of that, 34.5T $/s. With a fixed share of stage 7's full-plot $/s it would add 19.9Qi $/s, about 575,000 times more, and the whole stage would be skipped. The percentage rule keeps ore income matched to where the player is inside the stage, not just to which stage.

### 5.4 Mining timings

| What | Value | Config key (section 9) |
|---|---|---|
| Swing interval | 0.5 s | `PickaxeSwingSeconds` |
| Damage per swing | 20 | `PickaxeDamage` |
| Coal Node HP | 200 (10 swings, 5.0 s) | `NodeMaxHP` |
| Drop per node | 1 Coal | `OreIndex.NodeDrop` |
| Regrow time | 25 s | `NodeRegrowSeconds` |
| Mine prompt range / server reach | 8 studs / 10 studs | `MinePromptDistance`, `MineReach` |
| Ore prompt range / server reach | 10 studs / 16 studs (fruit use 12 / 16, `GardenConfig.lua:304-305`) | `OrePromptDistance`, `Config.MaxReach` |
| Ore landing spot | within 4 studs of the node | `OreDropRadius` |
| Fill Floor 1 (10 Iron = 20 Coal, 10 merges) | about 3 min (estimate: 20 × (5 s mining + 2 s walking) + 10 × 3 s merging) | |
| Fill Floors 1-3 (14 Iron = 28 Coal) | about 4 min (estimate) | |

With 6 nodes, mining all six takes about 42 s, longer than the 25 s regrow, so a player never waits for rocks on Floor 1. The v1 mine is short to fill on purpose: it has only two ores. The later ladder is what gives the mine long-term play.

### 5.5 Ores Index rewards

| Ore | Shards on claim | Money on claim | Coin bonus share |
|---|---|---|---|
| Coal | 500 | 60 s of one Coal's live $/s | none |
| Iron | 1,000 | 60 s of one Iron's live $/s | none |

- 60 s mirrors `Config.IndexMoneySeconds` for fruit (`GardenConfig.lua:394`). At the plateau: Coal pays 127 (stage 1), 673M (stage 3), 448Qa (stage 6), 4.48Sp (stage 9). Iron pays 2.5× that.
- The 1,500 shards in total are 1.5% of the Floor 1 price: a thank-you, not a faucet. Shards stay flat on purpose, like fruit Index shards (`IndexRewards.lua:73-75`).
- Ores do not add to the Index coin bonus. That bonus is normalised over the fruit ladder only (`IndexRewards.lua:38-56`), and ores already have their own bonus.

### 5.6 What the simulation says

Method. A scratch copy of `~/fruit-empire/sim/economy.py` `simulate_stage`, with three changes: (1) side income is split into a pet share and an ore share; (2) side income is a percentage of the CURRENT fruit income every tick (what the Ore Bonus does), instead of a fixed 35% of full-plot $/s after the plateau; (3) floors are bought the first second shards minus the price still cover the current stage's rebirth shard cost, and the model's other spending is kept (`SPEND` in `final.py` minus its 100K underground). Cash requirements, shard drops (D-010) and shard costs (D-011) are the Stage Model's. 7 seeds, attentive player (3 s merges). The pet share of 17.5% is an assumption (see section 13, Pets).

| Scenario | Stage 7 | Stage 8 | Hours to unlock Transcendent |
|---|---|---|---|
| Stage Model as written: pets + ores 35% from stage 1, after the plateau | 120.1 min | 149.6 min | 10.84 h |
| No pets, no ores | 147.5 min | 189.0 min | 12.95 h |
| Pets 17.5% only | 130.2 min | 164.4 min | 11.49 h |
| **Pets 17.5% + ores by floor, this spec** | **121.5 min** | **148.7 min** | **11.10 h** |
| Ores only, no pets | 135.6 min | 167.7 min | 12.40 h |

| Purchase (this spec) | Bought in | Median time |
|---|---|---|
| Floor 1, 100K | stage 7, Astral | 7.2 h |
| Floor 2, 150K | stage 8, Eternal | 9.1 h |
| Floor 3, 300K | stage 9, Transcendent | 11.6 h |
| Shards left after 2 h of stage 9 | | 167K |

**Superseded split.** The rows above used ores at 12.5 / 15 / 17.5% and pets at 17.5%. The [[Integration Review]] kept the [[Pets]] limit at 25% and cut ores to 7.5 / 9 / 10.5%, fruit only (D-018). The [[Integration Check]] re-run with forest trips gives: no pets and no ores 12.95 h; pets plus ores as first written (compounding, 46.9% at the plateau) 10.47 h; **the resolved split 10.59 h**, with Floor 1 at about 8.5 h (stage 8), Floor 2 at about 10.8 h and Floor 3 at about 11.1 h (early Transcendent).

Reading it:
- Ores save about 23 minutes over a whole career when pets exist, and about 33 minutes when they do not (this note's own run, old split). With the smaller resolved caps and the later Floor 1, ores mostly pay in stages 8 and 9. That still fits "worth doing without being mandatory" from the [[Stage Model]].
- The Stage Model's 10.8 h assumes 35% side income from stage 1. Ores cannot exist before about 7 h at a 100K price, so the realistic total is about 11.1 h. This is flagged in section 14.
- Earlier stages are unchanged by ores, because no one owns a mine before stage 7.

## 6. UI

All text is FredokaOne (`Font.new("rbxasset://fonts/families/FredokaOne.json")`, as `ShopMenu.lua:47` does), white, black UIStroke scaled to panel width like `ClearMenu.lua:172-193`. Panel titles are images. No emoji or non-ASCII characters anywhere, including toast strings. No new HUD buttons and no new always-on HUD elements. Image ids below are from [[UI Kit Inventory]].

Sizes are given against the Index panel's mockup canvas of 1831 × 969 (`IndexMenu.lua:9-10`), because the Mine panel copies that shell.

### U1. Mine panel (`GardenUI.MineMenu`)
- **Built from:** TitleFrame 110138832714236, TitleBlue 95532041481396, new title image "MINE" (request A1), Body 82827190351667, Inline 110403403749726, square red close 107111219044453. Same structure as IndexMenu, cloned from it in Studio so the proportions match.
- **Zone and size:** screen centre, the exact Size, Position and AnchorPoint of IndexMenu, aspect 1831:969. On touch-only devices it widens to 92% of the viewport width (aspect kept) so buttons stay at least 44 px tall on an 844 × 390 phone.
- **Open from:** the Open Mine prompt on the lift (owner, before Floor 1) and the Mine Upgrades prompt on the Mine Board (owner, in the mine). The client opens it on its own `PromptTriggered`; the server ignores these two prompts.
- **Behaviour:** same open/close pattern as every panel: an `isOpen` flag, UIScale 0.85 to 1 with Back easing in 0.22 s, 0.14 s Quad out (`IndexMenu.lua:456-457`). Opening it closes the other five panels, and the other five close it (section 9, client changes).
- **Name ends in "Menu"** so the loading screen pre-draws it (`LoadingScreen.lua:411-416`).
- **Overlap:** occupies the same rectangle as the Index, so it overlaps nothing the Index does not already cover. It is never open at the same time as another panel.

### U2. Floor cards (inside U1)
- **Built from:** Upgrade card art 114196041045860 (native 240 × 374), three in a row. Each card is 366 × 570 on the 1831 canvas (20% of the panel width), with 4% gaps, centred, top edge at 13% of the panel height.
- **Inside each card, top to bottom (fractions of card height):** "Floor 1" title 0.05-0.15; a ViewportFrame 0.18-0.52 showing that floor's node model with its crystal cluster, static (no spin; spinning viewports each cost a RenderStepped connection, `ShopMenu.lua:289-297`); "+10 ore slots" 0.55-0.62; "6 ore nodes" 0.63-0.70; "Ore bonus up to 7.5%" 0.71-0.78; action button 0.82-0.95, 80% of the card width.
- **States:**
  - Next to buy: Shop Buy button 97067256017679 (the shard crystal is baked into it, `ShopMenu.lua:315`), price text "100K" drawn over it. Price text turns red (`Config.UpgradeUnaffordable`) when shards are short, as UpgradeMenu does for cash (`UpgradeMenu.lua:57-85`).
  - Owned: new OWNED button (request A3).
  - Locked: card ImageColor3 (0.6, 0.6, 0.6); Buy art tinted `Config.IndexClaimedTint`; text "Buy Floor 1 first".
- **Status strip** below the cards, at 76-88% of panel height, inside the body: three groups, each a kit chip plus a text line: bonus chip 89282702522748 "Ore bonus +7.5%"; money chip 82056408182031 "+$187Qa/s"; shards chip 78947574768646 "Shards 412K". The $/s is the `OreIncome` attribute the income tick publishes (ore share of income, fruit only, D-018), so no new remote is needed.

### U3. Purchase check
- **Built from:** the Clear dialog: ClearConfirm.Panel 112127226387884 (565 × 373) with Yes 71592791133553 and No 105863640938118, reused the way the rebirth check reuses it (`RebirthMenu.lua:190-193`, `:211-213`): the baked "ARE YOU SURE?" header stays, the title line is set to "Buy Floor 1?".
- **Body text:** "100K shards. You will have 12K shards left. Your next rebirth needs 30K shards." The second sentence uses the D-011 shard cost of the player's current stage, so nobody spends shards they need for a rebirth by accident.
- **Zone:** screen centre, above U1, with the full-screen transparent Blocker one ZIndex below (pattern at `RebirthMenu.lua:174-188`, `Animated` set before parenting).

### U4. Index category tabs (Fruits, Pets, Ores)
- **Built from:** the Shop's vertical tab art 110917091182913 (native 100 × 132), three copies, each with a child ImageLabel holding a new icon with a baked label (request A2: "Fruits", "Pets", "Ores"). One tab column serves both new categories: [[Pets]] U5 uses these tabs instead of title-bar pills (decision D-026, [[Integration Review]] conflict C17). The Pets tab shows from launch; the Ores tab shows once the player owns Floor 1 or has an Ores Index entry.
- **Zone:** siblings of the Index panel art, tucked behind its left edge exactly as the Shop tabs are behind the Shop panel. The column is re-centred in code the way `ShopMenu.lua:66-85` does it.
- **States:** selected tab white, other tabs `Config.ShopTabDim`, same 0.12 s tween as `ShopMenu.lua:89-102`. The Index opens on Fruits. "Found n/m" in the title bar ([[Rarity Gating and Rebirth]] U3) counts the open tab's cards.
- **Overlap check:** at 1600 × 900 the tabs end about 120 px right of the left HUD column. Test at 844 × 390 and 667 × 375 (section 12). Fallback if they touch the Inventory/Index/Shop column on a phone: move both tabs inside the panel as a row at the top of the grid area, same art scaled to 60%.

### U5. Ore cards, info box and rewards box
- **Grid:** a second ScrollingFrame `OreList`, a copy of `List` with the same size, position, 5 columns and pixel-height maths (`IndexMenu.lua:520-575`). Only one of List and OreList is visible. It must be a separate frame because `fitCanvas` counts every child of its list (`IndexMenu.lua:571-575`).
- **Cell:** `Templates.Cell` with the COMMON rarity frame 131104257655900. Name label "Coal" / "Iron". The rarity line shows "Floor 1" instead of "1/x". The ViewportFrame shows the ore template. Locked ores use the existing "???" look (`IndexMenu.lua:95-188`).
- **InfoBox** (101725900357985): name, "Floor 1" in place of the rarity line, tier "Common" in the COMMON colour, rate "+0.75% income".
- **RewardsBox** (126819257706583): shards chip "x1K", money chip "$2.8B" (live), bonus chip and its label hidden (Visible = false, nothing else moves). Claim (132804991293392) fires `OreIndexClaim`.
- **Badge:** the red "!" on the Index HUD button also lights for an unclaimed ore.

### U6. Carry row while holding an ore
- **Built from:** the existing CarryButtons (Drop 104911876480159) and CarryInfo (NameBar 123470450245311, RateBar 120297790302429), bottom centre, no position change.
- **States:** Drop visible, Use and Store hidden. NameBar "Coal". RateBar "+0.3% income".
- **Highlights:** matching ores in your mine get the green merge Highlight (`Config.HighlightColor`), nearest first, at most 12. They share the 24-Highlight budget (`GardenConfig.lua:1889`); fruit highlights are always 0 in a mine, because fruit cannot be carried down.

### U7. Ore name tag (world)
- A BillboardGui named `NameTag` with a `Label`, built like the fruit tag (`FruitFactory.lua:256-320`) with `Config.NameTagFont`, so if the UI overhaul switches world text to Fredoka the ores follow. Text: ore name in the COMMON colour, second line "+0.75%". Shown only while its prompt is shown.

### U8. Node HP bar (world)
- BillboardGui on the node, 5 × 0.6 studs, 3.4 studs above its top, MaxDistance 40. Hidden at full HP. Shown while HP is below max. Fill colour amber (240, 196, 92) for Coal nodes. Width of the fill = HP / MaxHP.
- **Built from:** the one shared bar pair, `UIKit.Images.BarTrack` and `UIKit.Images.BarFill` ([[King UI Overhaul]] A8 and A9, 336 x 54; decision D-027). Until that art exists: the CarryInfo RateBar image 120297790302429 as the track, and the same image tinted as the fill inside a ClipsDescendants frame.

### U9. Ore earn popups (world)
- Only for the owner, only while in their own mine. On each `Income` event (`EarnPopups.lua:87-108`), a "+$X" billboard over each ore, using the EarnPopups look (`Config.EarnPopup*`). X = `boost × plotBase × multiplier × dt` (the Income payload's multiplier is `CashMultiplier`, which no longer contains the Ore Bonus). At most 14 per tick, staggered by `Config.EarnPopupStagger`.

### U10. Mine Board (world, in the cave)
- A 6 × 7 stud board on two timber posts beside the cave lift. SurfaceGui (Face Front, canvas 600 × 700) with the Index InfoBox art 101725900357985 as its background. Four Fredoka lines: "FLOOR 1 OF 3", "Ores 8 / 10", "Ore bonus +10.0%", "+$31.1Qa/s". Filled on each client from the owner's replicated attributes, so visitors see the owner's numbers.
- Prompt "Mine Upgrades" (owner only) opens U1.

### U11. Mine Lift sign and lamps (world, surface)
- A 6 × 1.6 stud board above the cage door with a SurfaceGui holding the MINE title image (request A1). Three depth lamps are 3D parts, not UI (section 7).

### U12. Travel fade
- A full-screen black Frame on its own ScreenGui with DisplayOrder above GardenUI. 0.25 s fade in, held until the character arrives (or 1.5 s), 0.35 s fade out. Shown only during lift travel, so it never sits over the HUD for longer than about one second.

### U13. Prompts (default ProximityPrompt look, as fruit use today)
| Where | ActionText | Key (keyboard / gamepad) | Who sees it | Hold | Range |
|---|---|---|---|---|---|
| Lift, before Floor 1 | Open Mine | E / X | owner | 0 | 8 |
| Lift, after Floor 1 | Go Down | E / X | owner | 0 | 8 |
| Lift, after Floor 1 | Visit Mine | E / X | others | 0 | 8 |
| Cave lift | Go Up | E / X | owner and visitors | 0 | 8 |
| Mine Board | Mine Upgrades | E / X | owner | 0 | 8 |
| Node | Mine / Stop | F / Y | owner | 0 | 8 |
| Ore | Pick Up / Merge | E / X | owner | 0 | 10 |

- F is used by royal combat only inside the arena (`RoyalCombat.lua:152`), never in a mine. Nodes use a different key from ores so both can show at once (research advice, [[Roblox Platform Constraints for the Expansion]], "Prompts").
- `RequiresLineOfSight = false` on node and ore prompts, so a visitor standing in the way cannot block them. `ClickablePrompt = true` on touch devices, as fruit do (`GardenClient.lua:94`).

### U14. Toasts
Plain ASCII strings in Config, sent through the existing Notify (red, top) and Notice (green, bottom) remotes (`GardenService.lua:66-70`):
- Notice: "Floor 1 unlocked!", "New ore: Coal".
- Warnings: "Put down what you are carrying first.", "Ores stay in your mine.", "Your ore floor is full. Merge ores to make room.", "Your floor is full of Iron, the best ore for now. More slots come with the next floor.", "Your mine is full of Iron, the best ore for now.", "You need 100K shards.", "This mine is full of visitors.", "The mine's owner left. Back to the surface.", "No fruit from storage in the mine.", "No explosives in the mine."

### U15. HUD
- No new buttons or panels on the HUD. The Ore Bonus is not part of `CashMultiplier` any more (it applies to fruit income only, D-018), so the CornerHud multiplier line does not show it; the Mine Board and the Mine panel do.
- The Roblox Backpack hotbar is already off for the whole game ([[Bats]] BatHud turns it off once at start). This system never turns it back on (an on/off toggle when leaving a mine would bring the bar back for everyone, [[Integration Review]] conflict C35).

## 7. World and assets

### 7.1 Where the mine is
- **Mine Lift (surface):** inside the underground-entrance hut that [[Plot Overhaul and Environment]] builds in the back-right corner of every lot (x +40 to +50, a 229 to 243 in that note's plot frame; door on the hut's front face). The hut is outside the reserved pen zone at every plot size, so the lift never ends up inside a grown pen. Plot Overhaul's stepping stones lead to the door. The hut carries a `LiftAnchor` attachment (Plot Overhaul's `ShaftTop` marker, renamed) where this system places the cage. Decision D-023 ([[Integration Review]] conflict C14): one entrance, built by Plot Overhaul, used by this system.
- The lift cage, lamps and sign live in `workspace.Underground.UG_<PlotName>`, never inside the plot model. That keeps them out of soil detection (`PlotRegistry.lua:21-30`) and out of the pen rebuild. The hut shell itself is static `Lot` decor owned by Plot Overhaul.
- **Cave:** directly under the plot. Floor 1's floor top is 140 studs below `plot.topY`. Each deeper floor is 36 studs lower. Interior per floor: 56 wide × 72 long × 22 tall, centred under the soil centre, turned to the soil's yaw, with the lift landing at the back-right end, under the hut. Neighbouring plots are about 105 studs apart along the ring ([[World Tour]]: 12 plots about 200 studs from the centre), so caves have about 40 studs of rock between them.
- **Why teleport instead of a hole:** a walk-down shaft would need a hole cut through the plaza and the island under it. The island's underside is not in the exported code, so the builder cannot know what is there. A lift teleport needs no hole and also avoids open drops (the 220-stud fall kill, `Ambient.lua:271-301`) and the "no floor under me reads as water" rule (`PlayerVision.lua:141-153`, `Ambient.lua:359-375`). The cave is still physically under your plot, so "vertical scaling" stays literal.
- **Depth safety:** 140 to 230 studs down is above the SaturationPass cut-off of y -500 and far above the asset staging area near y -790 ([[World Builders and Dev Tools]]; [[Existing VFX and Props]]).

### 7.2 Mine Lift (server-built inside the Plot Overhaul hut, seen by everyone) - about 30 parts
| Part | Size (studs) | Material and colour | Source |
|---|---|---|---|
| Pulley wheel and axle | 3 dia | Metal (60, 62, 66) | built |
| Cage bars, 5 per side on 3 sides | 0.3 × 10 × 0.3 | Metal (52, 54, 58) | built |
| Cage floor | 8 × 0.4 × 8 | Metal (70, 72, 78) | built |
| Glow strip on the cage lip | 8 × 0.15 × 0.3 | Neon, colour of the best ore owned (Coal (60, 60, 64), Iron (232, 214, 182)), Transparency 0.3 | built |
| Hanging lantern | see 7.4 | PointLight (255, 170, 90), Brightness 1.2, Range 14, no shadows | built |
| 3 depth lamps | 0.8 sphere | off: SmoothPlastic (60, 60, 64); on: Neon (255, 176, 92) | built |
| Sign board | 6 × 1.6 × 0.3, on the hut's front wall above the door | wood (105, 76, 52) + SurfaceGui MINE image | built + A1 |

The hut's walls, roof, doors and padlock are Plot Overhaul's (its "Underground entrance" table, 29 parts); this table lists only what this system adds inside it.

How others see it: the lift, its lit lamps (one per floor owned) and the colour of the glow strip tell everyone how deep your mine goes and your best ore. Nothing of the cave is visible from the surface.

### 7.3 Cave shell (server-built, collision only) - about 40 parts per floor
Floor slab, ceiling slab, four walls (all 4 studs thick, Slate + RidgeStuds in the floor's wall colour), the cave lift cage (Floor 1 only, about 25 parts, same design as 7.2 without the roof), the Mine Board, and a spiral stair to the next floor: 30 steps of 1.2-stud rise and 5-stud treads wound around a 4 × 4 studded central pillar in a 14 × 14 shaft in the back-left corner. The pillar leaves no open drop. A 14 × 1 × 14 slab seals the shaft until the next floor is bought.

### 7.4 Cave decor (client-built, only for the floor you are on)
Cloned on each client from `ReplicatedStorage.UndergroundKit.Decor_F<n>` when the local player is in that floor's volume, and destroyed when they leave it. Visitors see the same decor because it is built from the same kit.

**Floor 1, Coal Seam (warm):**
| Element | Count | Spec | Source |
|---|---|---|---|
| Wall terraces | about 150 parts | 3 stepped inset layers along all walls, blocks 4-12 studs, Slate + RidgeStuds, colour (96, 92, 96) ± 6 per channel, seeded | built, look copied from Studed Mountain's stepped faces |
| Ceiling slabs | about 30 | studded slabs hanging 0-3 studs lower, (82, 78, 82) | built |
| Stalactites | 3 clusters, about 60 parts each | Studed Mountain shell (its 60 largest parts), flipped, scale 0.15 (about 15 × 12 × 15) | Studed Mountain 105342635739603 |
| Floor rubble | 24 | studded blocks 1-2 studs, (88, 84, 88) | built |
| Worn path | 6 | flat Plastic + Studs, (110, 82, 58), lift to node area | built |
| Timber supports | 6 posts + 4 beams | wood (105, 76, 52) | Studded Pack `Logs` |
| Lanterns | 10 | body: 4 thin Metal posts (40, 40, 44) + caps; core Neon (255, 176, 92), Transparency 0.15; PointLight on an Attachment, (255, 170, 90), Brightness 1.6, Range 18, Shadows off except 3 "hero" lanterns (landing, middle, back) | built |
| Crystal clusters | 4 corners × 6 wedges | Glass (255, 184, 104), Transparency 0.2, Reflectance 0.1; one PointLight per cluster (255, 170, 90), Brightness 1.0, Range 14, no shadows | built |
| Glow mushrooms | 6 | Studded Pack `Mushroom` at scale 0.3, cap recoloured Neon (90, 230, 200), Transparency 0.3; 2 of them with a PointLight (90, 230, 200), Brightness 0.6, Range 8 | Studded Pack |
| Campfire | 1, left of the lift | Studded Pack `CampFire`; PointLight (255, 140, 60), Range 16, Brightness flickering 1.6-2.0 on the client | Studded Pack |
| Rails and cart | 2 rails, 10 sleepers, 1 cart (8 parts) | Metal rails (70, 70, 76), wood sleepers | built |
| Dust motes | 1 emitter | invisible 50 × 16 × 64 box; Rate 6, Lifetime 8-12, Size 0.12-0.2, Speed 0.2-0.5, LightEmission 0.6, colour (255, 220, 170), Transparency 0.6 to 1 | built |
| **Budget** | **about 650 parts, 18 PointLights (3 with shadows), 2 emitters** | | |

**Floor 2, Iron Hollow (cool):** walls (70, 76, 88), ceiling (58, 64, 74); 5 crystal clusters ice blue (130, 210, 255), lights (120, 200, 255) Brightness 1.1 Range 14; 8 lanterns; violet mushrooms (170, 120, 255); 4 still pools (Glass (90, 140, 170), Transparency 0.4, flat). 15 PointLights.

**Floor 3, Crystal Grotto (violet):** walls (58, 54, 82), ceiling (46, 42, 66); 7 clusters violet (186, 130, 255), lights (180, 120, 255) Brightness 1.2 Range 16, two of them 10-stud landmark clusters; 6 lanterns; pearl mushrooms (240, 236, 220). 15 PointLights.

The stone gets darker and colder with each floor, as Azure Mines and Ultimate Mining Tycoon do per layer ([[Mining and Ore Merging - Reference Games]], section 5).

### 7.5 Cave lighting (client, folded into PlayerVision)
A `cave` value eases from 0 to 1 over 0.6 s on entry. Each property is lerped from its day/night value toward the floor's cave value by `cave`:

| Property | Floor 1 | Floor 2 | Floor 3 |
|---|---|---|---|
| Lighting.Ambient and OutdoorAmbient | (46, 40, 44) | (34, 38, 50) | (40, 32, 56) |
| Lighting.Brightness | 0.6 | 0.5 | 0.5 |
| VisionTint TintColor | (0.96, 0.92, 0.90) | (0.90, 0.94, 1.00) | (0.94, 0.90, 1.00) |
| VisionTint Contrast / Saturation | +0.06 / +0.08 | +0.08 / +0.05 | +0.08 / +0.12 |
| Atmosphere Density (if one exists) | 0 | 0 | 0 |
| CaveBloom (new BloomEffect, enabled only in a mine) | Intensity 0.5, Size 28, Threshold 1.4 | same | Intensity 0.65 |

Why: dark Ambient and no Atmosphere fog in caves are the standard advice ([[Mining and Ore Merging - Reference Games]], section 7). Brightness is lowered too, because the place's lighting style is not known from the code and, without shadows, the sun would still light the cave. Bloom makes the Neon lantern cores and crystals glow without making them brighter.

### 7.6 Nodes and ore items (server-built from kit templates)
- **Coal Node:** 10 parts. Base 3.4 × 2.2 × 3.0 and three smaller shoulder blocks (2.0 × 1.6 × 2.2, 1.6 × 1.2 × 1.6, 1.4 × 1.0 × 1.2) in Slate + RidgeStuds (100, 98, 102); six 0.6-stud gem cubes, SmoothPlastic (28, 28, 32), Reflectance 0.2, half sunk into the faces. This is the look of Samarth's reference image (`Images/ore-reference.webp`: studded grey clumps with small coloured gem cubes). Three shape variants, picked per socket from a seed of the plot name. Rock parts CanCollide on, gems off. Broken state: shoulders and gems hidden, base lowered to a 0.6-stud rubble slab.
- **Coal (item):** 6 parts. Rock 1.8 × 1.3 × 1.6 plus two small blocks, three 0.45-stud black gems. All Massless, CanCollide/CanTouch/CanQuery off, welded to the rock, which is the part that is carried (the same single-part carry as fruit).
- **Iron (item):** 7 parts. Same rock, colour (104, 100, 104); four cream gems (232, 214, 182), Reflectance 0.3; a Shiny-01 emitter at rate 0.4 for a faint glint.
- **Pickaxe:** a `Tool` named `Pickaxe`, 4 parts: wooden handle 0.3 × 3.4 × 0.3 (124, 92, 62), Metal head 2.6 × 0.45 × 0.45 (150, 150, 158) with two wedge tips. `CanBeDropped = false`, `ManualActivationOnly = true`. The server equips it when mining starts and destroys it when mining stops.
- **Swing animation:** each server swing adds a `toolanim` StringValue "Slash" to the pickaxe, the same way the arena animates swords for players (`CrownArena.lua:229-238`). The default character Animate script then plays its tool slash, and because it is the player's own character, everyone sees it. Optional upgrade: a hand-made swing (request A5).

### 7.7 Effects (all from packs already in the place)
| Moment | Effect | Source ([[Existing VFX and Props]]) |
|---|---|---|
| Each swing hit | 6 rock chips + 1 small impact flash at the hit point, scale 0.35, grey | `vfx pack` → `f` (rocks, impact) |
| Node shake | the node's parts nudge 0.15 studs for 0.08 s, on the client only | code |
| Node breaks | 14 rock chips + 4 smoke puffs | `vfx pack` → `f` rocks; `Anime` → Smoke-01 |
| Ore lands | a 0.4 s hop played on the client | code |
| First discovery | the existing discovery sparkle | `sparkleOn` (`FruitFactory.lua:148-195`) |
| Merge | white wash Highlight like `flashOn` (`Merging.lua:34-45`), 8 stars | `Anime` → Stars-01 |
| Regrow | parts fade in over 0.6 s, 3 sparkles | `Anime` → Shiny-01 |
| Iron glint | 0.4 sparkles per second | `Anime` → Shiny-01 |
| Planks off the lift | two planks drop and fade, client only | code |

### 7.8 Sounds
Picked from Roblox's own audio in the Creator Store (Roblox-uploaded audio plays in any experience, the route D-002 used for music). Ids go into Config; none are chosen yet.
- Pickaxe on stone, 3 variants, pitch 0.95-1.10, Volume 0.5, RollOffMaxDistance 60.
- Rock crumble on break, Volume 0.7.
- Small stone clink when an ore lands, Volume 0.5.
- Merge: the fruit merge sound `Config.MergeSoundId`, for consistency.
- Lift: metal gate plus a short rumble, Volume 0.6.
- Cave ambience loop (drips, low wind), Volume 0.35, as a new biome track in MusicPlayer; the playlist is ducked to 50% while in a mine.

### 7.9 What must be built
`ServerStorage.ZZTools.UndergroundBuilder`, a hand-run builder like the other ZZTools (seeded, run from the Studio command bar). It builds `ReplicatedStorage.UndergroundKit` (Shell_F1-F3, Decor_F1-F3, Lift, CaveLift, node variants, Pickaxe) and `ReplicatedStorage.Ores.Coal` / `.Iron`. It uses the Studed Mountain and Studded Pack copies already loaded in `ServerStorage.AssetQuarantine` ([[Creator Store Packs]]). It also runs a check: for each of the 12 plots it queries the cave volumes and the lift footprint (`workspace:GetPartBoundsInBox`, and a Terrain read) and prints any plot where something is already there.

## 8. Asset requests for Samarth

**A1. Title image "MINE"** (panel title and lift sign). ChatGPT / Gemini prompt:
> Create a transparent PNG, 404 x 148 pixels, of the single word "MINE" as a game panel title. Use the attached image as the exact style reference: it is the "INDEX" title from my Roblox game's Index panel (asset 103221050515090). Match it exactly: the same heavy italic rounded letters, the same white fill with its subtle top-to-bottom shading, the same thick dark outline, the same slant and the same drop shadow. Same letter height as INDEX. No other decoration, no icons, no background.

Upload it and send the asset id. It goes in `Config.MineTitleImage`.

**A2. Three Index tab icons, "Fruits", "Pets" and "Ores".** (The Pets icon was added by the [[Integration Review]], because [[Pets]] now uses the same tab column.) Prompt:
> Create three transparent PNGs, 200 x 264 pixels each, to sit on top of my game's vertical shop tab (attached, asset 110917091182913). Style reference: my Index HUD button (attached, asset 118941334639047): a glossy cartoon object with a thick black outline, and the label baked underneath in white rounded bold text with a black stroke. Icon 1: one shiny red apple, label "Fruits". Icon 2: the head of a blocky white studded sheep, label "Pets". Icon 3: two chunky blocky grey stone ore lumps with square studs on top, one dotted with small black gem cubes and one with small cream gem cubes, label "Ores". Keep all three icons the same size and position so they line up.

**A3. "OWNED" button.** Prompt:
> Create a transparent PNG, 346 x 122 pixels. Copy the exact shape, gloss, border and drop shadow of my shop Buy button (attached, asset 97067256017679), but remove the shard crystal, change the fill to the calm green of my Claim button (attached, asset 132804991293392), and bake the word "OWNED" in the same white rounded bold text with a black stroke, centred.

**A4. Health bar frame and fill:** no separate request. The node bar uses the one shared bar pair, [[King UI Overhaul]] A8 (`kit_bar_track.png`) and A9 (`kit_bar_fill.png`), 336 x 54, made from the carry info bar 120297790302429 (decision D-027). Make it once there.

**A5. Optional: pickaxe swing animation.** Only if the default tool slash looks wrong in the playtest (section 12, rule 10 test). Animate an R15 overhead chop, 0.5 s, priority Action, in the Animation Editor, then File, Publish to Roblox under the account that owns the game (`hikeALT12212` per [[Roblox Platform Constraints for the Expansion]]), and send the id. Animations only play live when published by the game's owner (D-005).

Robux products: none. Ore drops are earned-only randomness and nothing here is sold for Robux ([[Roblox Platform Constraints for the Expansion]], section 10).

## 9. Code plan

All server code follows the existing pattern: a ModuleScript under GardenService that is `return function(G)`, takes what it needs from G when required, and puts its own functions back into G (`GardenService.lua:88-98`). Functions from modules that load later are looked up through G at call time (`Carrying.lua:352-353`).

### 9.1 New files
1. **`ReplicatedStorage.OreIndex`** (ModuleScript). Same shape as FruitIndex (`FruitIndex.lua:89-99`): `List`, `ByName`, `Count`, `Get(level)`, `MergeInto(name)` (returns the next entry, or nil for Iron in v1), `Template(name)` (`ReplicatedStorage.Ores.<name>`), `NodeDrop = "Coal"`. Each entry: `level, name, display, tierName, boost, floor, shards, gem`. Kept separate from FruitIndex on purpose: FruitIndex's Count drives spawn tiers, income, Index rewards and the rebirth gate (`GardenConfig.lua:87`, `FruitEconomy.lua:40-46`, `IndexRewards.lua:42-50`, `Progression.lua:311-320`).
2. **`ServerScriptService.GardenService.Underground`** (ModuleScript, `return function(G)`). Required right after `require(script.CrownDecree)(G)` at `GardenService.lua:518`. Contents:
   - State: `mines[plot] = { model, floors, nodes = {id = node}, ores = Folder, visitors = {}, owner }`, `mining[player] = { node, token }`.
   - `buildMine(plot, floors)`, `addFloor(plot, n)`, `destroyMine(plot)` (sends visitors home first).
   - `travel(player, plot, direction)`: checks, sets `TravelFadeAt`, waits 0.25 s, `player:RequestStreamAroundAsync(target)` in a pcall (retry once, else refuse with a toast), clears the character's `FallPeak` attribute (`Ambient.lua:271-301`), `PivotTo`, sets `InUnderground`.
   - `startMining(player, node)`, `stopMining(player)`, `swingLoop(player, token)`, `breakNode(mine, node)`, regrow sweep every 0.5 s.
   - `pickUpOre(player, ore)`: same refusals as `pickUp` (`Carrying.lua:160-203`), then `G.holdFruit(player, character, head, ore)` (`Carrying.lua:95-143`), then `held[player].ore = ore.Name` and `CarryingOre`. Reusing holdFruit keeps its load-bearing weld order.
   - `dropOre(player, silent)`: `G.releaseHold(player, false)` (`Carrying.lua:73-89`), then a free-spot search on the floor rectangle (modelled on `freeSpotNearIn`, `FruitFactory.lua:109-121`), parent into the mine's Ores folder, `SetNetworkOwner(nil)`, anchor, set `Busy` for 0.4 s. The ore is placed at its final spot at once; the client plays the hop. Silent (death, leave, outside the mine): Floor 1 free spot.
   - `mergeOre(player, target)`: one yield-free step: checks (owner, same name, not Busy, in reach, `MergeInto` not nil, `Store.IsReady`), destroy carried and target, place the result at the target's spot, stamp `MergedAt` for the client flash, discover. No pending-merge ledger is needed, because nothing is ever missing between steps (compare `GardenService.lua:134-169`).
   - `returnCarriedOre(player)`: yield-free; used by rebirth, reclaim and the watchdog.
   - `oreBoost(player)`: cached sum, recomputed on any ChildAdded/ChildRemoved of the Ores folder; writes the `OreBoost` attribute. The next income tick picks it up (no `publishMultiplier` call is needed, because the Ore Bonus is not part of `CashMultiplier`, D-018).
   - `buyFloor(player, n)`, `claimOreIndex(player, name)`, `discoverOre(player, name)`.
   - `capture(player, profile, loaded)` and `apply(player, profile)` (section 9.4).
   - Watchdog every 0.5 s: a player whose `InUnderground` is set but whose root is outside that mine's box (plus 4 studs) has it cleared, has mining stopped and has any carried ore returned; a character inside a mine's box with no `InUnderground` (an exploit teleport) is sent to their own plot spawn.
   - Its own `ProximityPromptService.PromptTriggered` handler for prompts whose parent carries the `UndergroundPrompt` attribute. The fruit handler returns early for anything not in the fruit folder (`GardenService.lua:522-526`), so the two never collide.
   - Exports: `G.oreBoost, G.dropOre, G.returnCarriedOre, G.undergroundCapture, G.undergroundApply, G.undergroundRelease, G.inUnderground`.
3. **`StarterPlayer.StarterPlayerScripts.UndergroundClient`** (LocalScript): decor load and unload by floor; Backpack hotbar off in a mine; travel fade; prompt enabling (owner only, Busy off, `ClickablePrompt` on touch); its own focus outline on the whole node model; ore name tags; node HP bars; hit, break, regrow and merge effects driven by attribute changes; ore earn popups; Mine Board text; the MineMenu panel and its confirm dialog.
4. **`ServerStorage.ZZTools.UndergroundBuilder`** (section 7.9).

### 9.2 Changes to existing scripts
| File:line | Change |
|---|---|
| `GardenService.lua:36-41` | Next to the fruit folder: destroy any stale `workspace.Underground` and create a fresh one (anything saved in it in Studio must not survive a start, the same reason as the fruit folder). |
| `GardenService.lua:59-83` | `local undergroundBuyRemote = remote("UndergroundBuy")` and `local oreClaimRemote = remote("OreIndexClaim")`; add both to the G fill at `:101-115`. |
| `GardenService.lua:518` | `require(script.Underground)(G)` after CrownDecree. |
| `GardenService.lua:450-480` (`releasePlot`) | After `clearPlot(plot)` at `:453`: `if G.undergroundRelease then G.undergroundRelease(plot) end`. teardownPlayer captures the profile before releasePlot (`:703-715`), so the mine is saved first. |
| `GardenService.lua:653` | Studio-only dev door `DevSetUndergroundFloors`, built with the existing `devSet` helper (`:632-647`). |
| `Carrying.lua:291-297` (`dropFromPlayer`) | First line after reading `state`: `if state.ore then return G.dropOre(player, silent) end`, before the tool branch at `:297`. This also covers the silent drops on death (`GardenService.lua:601-602`) and character removal (`:692`). |
| `Merging.lua:158` (`merge`) | `if state.tool or state.ore then return end`. |
| `Storage.lua:24-40` (`storeCarried`) | Before the FruitIndex name check at `:40`: if `state.ore`, warn "Ores stay in your mine." and return. Today the name check would return silently. |
| `Storage.lua:56` (`equipStored`) | Refuse with a warning while `InUnderground` is set (a fruit taken out underground would be dropped on the surface plot by `dropAt`, because `isOnPlot` is X/Z only, `FruitFactory.lua:63-68`). |
| `Tools.lua:244-263` (`useTool`) | For any non-consume tool: refuse with "No explosives in the mine." while `InUnderground` is set, before `plotUnder` at `:259`. Today it would already refuse at `:259-263` with the off-plot text, because the cave is far below the 12-stud stand band (`FruitFactory.lua:79-85`). [[Explosives and Fruit Health]] will rewrite this function; the check moves with it. |
| `RebirthLock.lua:71-73` | Before `releaseHold(player, true)`: `if G.returnCarriedOre then G.returnCarriedOre(player) end`. Yield-free, so the rebirth block's guarantee (`:63-67`) holds. Nothing else in doRebirth touches the mine. The mine is not in `Config.UpgradeKeys`, so `publishUpgrades(player, nil)` at `:95` cannot reset it. |
| `Persistence.lua:81` (`captureProfile`) | After the royal line: `if G.undergroundCapture then G.undergroundCapture(player, profile, loaded) end`. |
| `Persistence.lua:278` (`applyProfile`) | After the plot-size block (`:274-278`) and before `if not plot then return end` (`:293`): `if G.undergroundApply then G.undergroundApply(player, profile) end`. It runs before offline pay (`:324-331`) so the restored ores count offline. |
| `Persistence.lua:191` (`offlineEarnings`) | `local rate = (plotBase(plot) * (1 + (G.oreBoost and G.oreBoost(player) or 0)) + (G.petBase and G.petBase(player) or 0)) * (1 + indexBonus(player)) * rebirthMultiplier(player) * Config.OfflineRate` (one line shared with [[Pets]]; fruit-only Ore Bonus, D-018). |
| `Persistence.lua:337-359` (`reclaimCarried`) | If the held item is an ore: `G.returnCarriedOre(player)` and return no orphan. |
| `Progression.lua:332-336` (`cashMultiplier`) | **No change** (D-018). The Ore Bonus is applied to fruit income in `Ambient.lua:465` instead: `local base = plotBase(plot) * (1 + (G.oreBoost and G.oreBoost(player) or 0)) + (G.petBase and G.petBase(player) or 0)` (one line, shared with [[Pets]]), and the tick also publishes `OreIncome = plotBase(plot) * oreBoost * cashMultiplier(player)`. `G.oreBoost` is looked up at call time like `G.crownRefresh` (`Progression.lua:54`). |
| `GardenStore.lua:220-283` (`Default`) | Add `ugFloors = 0`, `ores = {}`, `oreIndex = {}`. |
| `GardenStore.lua:309-505` (`sanitise`) | Copy and check all three (section 9.4). Required: sanitise copies only the keys it names (comment at `:447-450`). |
| `GardenStore.lua:1197-1209` (self-test) | Add every ore at 999 and every Ores Index entry to the worst payload; it must stay under 8,192 bytes. |
| `GardenConfig.lua` | New "underground" section (section 9.6). `UpgradeKeys` (`:37`) and `HudClusters` (`:1863-1867`) do not change. |
| `GardenClient.lua:138-140` (`setTags`) | Return at once if the prompt's parent has `UndergroundPrompt`; UndergroundClient owns those. |
| `GardenClient.lua:210-262` (`refresh`) | If `CarryingOre` is set: name and "+x% income" from OreIndex; Drop on, Use and Store off. Without this the rate bar would read "$0/s" (`Economy.ForName` returns 0 for a non-fruit). |
| `IndexMenu.lua` | Tabs (U4), `OreList` (U5), an ore branch in `stateOf` (`:236-241`) and `paintInfo` (`:243-273`), cells built in `build` (`:301-361`), Claim firing `OreIndexClaim` for ores (`:444-448`), and a watcher on `OreIndexState`. |
| `AlertBadges.lua:71-78` (`indexWaiting`) | Also true for any 0 in `OreIndexState`. |
| `ShopMenu.lua:617`, `IndexMenu.lua:47`, `StoredFruitsMenu.lua:541`, `UpgradeMenu.lua:421`, `RebirthMenu.lua:51` | Add `"MineMenu"` to each hard-coded list of panels to hide. |
| `PlayerVision.lua:190-194`, `:402-418`, `:449-469` | Read `InUnderground` and the floor; ease a `cave` value; fold the cave look (7.5) into the single Ambient/Brightness write and the single tint write; own the new CaveBloom and the Atmosphere density. Publish `VisionCave` like `VisionSky` (`:386`). PlayerVision must stay the only writer of these (its own rule, `PlayerVision.lua:5-6`). |
| `MusicPlayer.lua:101-111`, `:120-194`, `:243-244` | A `CaveAmbience` biome track driven by `VisionCave`, and the playlist ducked to 50% by it, inside `applyVolume`, the only volume writer. |
| `DevPanel.lua:162-175` (`clearPlayerResidentState`), `:224-237` (`devSeed`) | Clear `OreIndexState`, `UndergroundFloors`, `OreBoost` and destroy the mine; seed 1 floor and 4 Coal + 3 Iron. |

### 9.3 Remotes
| Name | Direction | Payload | Server validation |
|---|---|---|---|
| `UndergroundBuy` | client to server | floor number (integer 1-3) | integer type; equals `ugFloors + 1`; `Store.IsReady`; has a plot; not `rebirthing[player]`; shards at least the price; then yield-free: `setShards(shards - price)` (`Progression.lua:216-218`), floors + 1, build the floor, Notice, `task.spawn(Store.ForceSave, player)`. |
| `OreIndexClaim` | client to server | ore name (string) | string type; `OreIndex.ByName[name]`; entry exists in `OreIndexState` with value 0; pay shards and 60 s of the ore's live $/s; value set to 1; `publishCash`; ForceSave (as `claimIndex`, `Progression.lua:345-372`). |

Everything else uses ProximityPrompts handled on the server (lift, mine, pick up, merge), as fruit do today, and replicated attributes for display. The existing `Income` payload is unchanged: its multiplier already includes the Ore Bonus through `cashMultiplier` (`Ambient.lua:473-478`).

### 9.4 Save document fields
| Field | Type | Default | Sanitise rule |
|---|---|---|---|
| `ugFloors` | integer | 0 | number, not NaN, floored, clamped 0 to 3 (`#Config.UndergroundFloors`) |
| `ores` | table, ore name to count | {} | names must be in `OreIndex.ByName`; counts 1 to `MAX_COUNT` (`GardenStore.lua:65`); same shape as `sanitiseCounts` (`:285-304`) but checked against OreIndex |
| `oreIndex` | table, ore name to 0 or 1 | {} | names in `OreIndex.ByName`; value 1 stays 1, anything else becomes 0 |

- **Old saves:** they have none of these fields, so they load as 0, {} and {} with no migration. This is exactly how GardenStore is designed (`GardenStore.lua:8-12`). No schema bump, so nobody gets kicked (`GardenStore.lua:764-772`).
- **Capture:** counts by name over the mine's Ores folder, plus a carried ore (already returned by `reclaimCarried` on leave). No plot this session (full server): `profile.ores = loaded.ores`, the same carry-forward fruit use (`Persistence.lua:106-110`). `ugFloors` always comes from the `UndergroundFloors` attribute.
- **Apply:** publish `UndergroundFloors`; build the `OreIndexState` folder; if there is a plot, build the mine and place the ores rarest first on random free spots (Floor 1 first). A save holding more ores than slots (only possible if slots are lowered in a later update) loads every ore; mining stays blocked until the count drops. Nothing is deleted.

### 9.5 Attributes
| On | Attribute | Type | Written by | Read by |
|---|---|---|---|---|
| Player | `UndergroundFloors` | int 0-3 | Underground | MineMenu, Mine Board, lift lamps |
| Player | `OreBoost` | number (0.075 = 7.5%) | Underground | Mine Board, MineMenu, earn popups |
| Player | `OreIncome` | number, $/s from ores (fruit-only bonus) | Ambient income tick | Mine Board, MineMenu |
| Player | `InUnderground` | string plot name or nil | Underground (travel, watchdog) | PlayerVision, MusicPlayer, UndergroundClient, other systems (bats, weapons) |
| Player | `MiningNode` | string node id or nil | Underground | UndergroundClient (hit effects) |
| Player | `CarryingOre` | string ore name or nil | Underground | GardenClient carry row |
| Player | `TravelFadeAt` | server time | Underground | UndergroundClient fade |
| Player folder | `OreIndexState` | IntValues 0/1 | Underground | IndexMenu, AlertBadges |
| Mine model | `PlotName`, `OwnerUserId`, `Floors` | | Underground | UndergroundClient |
| Node part | `NodeId`, `HP`, `MaxHP`, `Broken`, `UndergroundPrompt` | | Underground | UndergroundClient |
| Ore part | `PlotName`, `HeldBy`, `Busy`, `MergedAt`, `UndergroundPrompt` | | Underground | UndergroundClient |

`OreIndexState` is a separate folder from `IndexState` on purpose: the King's Throne counts every child of `IndexState` for eligibility (`Crown.lua:289-292`), and the save drops non-fruit names from `index` (`Persistence.lua:69-71`).

### 9.6 Config keys (new section in GardenConfig)
`UndergroundFloors` (the table in 5.2: name, price, slots, nodes, bonusCap), `UndergroundDepth = 140`, `UndergroundFloorPitch = 36`, `UndergroundInterior = Vector3.new(56, 22, 72)`, `MineLiftAnchor = "LiftAnchor"` (the attachment in Plot Overhaul's hut), `NodeMaxHP = 200`, `PickaxeDamage = 20`, `PickaxeSwingSeconds = 0.5`, `NodeRegrowSeconds = 25`, `MineReach = 10`, `MinePromptDistance = 8`, `OrePromptDistance = 10`, `OreDropRadius = 4`, `MineVisitorCap = 6`, `OreIndexMoneySeconds = 60`, `CaveLook` (table 7.5), `CaveFadeSeconds = 0.6`, `MineTitleImage`, `IndexTabFruitsIcon`, `IndexTabPetsIcon`, `IndexTabOresIcon`, `OwnedButtonImage` (all `"rbxassetid://..."` strings so the loading screen preloads them, `LoadingScreen.lua:333-338`), sound ids, and every toast string in U14.

## 10. Edge cases and abuse

1. **Carrying fruit or a tool to the lift:** refused (rule 7). This also stops a fruit being carried down and then dropped on the surface plot from below.
2. **Taking fruit out of storage in the mine:** refused (`equipStored` change).
3. **Explosives in a mine:** refused (rule 27). Plot-targeted explosives cannot reach a cave 140 studs down anyway (`standingOn` band, `FruitFactory.lua:79-85`).
4. **Bats and weapons in a mine:** no knockback, no damage. [[Bats]] and [[Weapons and Artillery]] check `InUnderground` (proposal).
5. **Visitors trying to interact:** their client shows no prompts; the server checks owner on every action anyway. Visitors cannot take or change anything. Stealing stays removed.
6. **Body-blocking:** prompts ignore line of sight; nodes have no back side; at most 6 visitors. There is no kick button in v1 (section 14).
7. **Exploit teleport into a mine:** the watchdog sends them to their own plot spawn within 0.5 s.
8. **Exploit teleport out with an ore** (the carried ore is welded to the character the client owns, `Carrying.lua:122-139`): the watchdog returns the ore to Floor 1. Capture counts it either way.
9. **Double-click on Buy:** the payload is the floor number, so a second request for the same floor fails the `ugFloors + 1` test. No accidental second floor.
10. **Remote spam:** `OreIndexClaim` pays only on a 0 entry. Mine prompt spam: each start replaces the loop token, and a start within 0.3 s of the last is ignored.
11. **Speed hacks while mining:** the server times every swing; the client never reports hits.
12. **Leaving mid-mining:** the loop ends when `player.Parent` is nil. The node keeps partial HP until the mine is removed.
13. **Leaving while carrying an ore:** `reclaimCarried` puts it back before `SaveAndRelease` (ordering at `GardenService.lua:703-715`).
14. **Leaving mid-merge:** impossible to split: the merge is yield-free.
15. **Leaving mid-travel:** the travel coroutine checks `player.Parent` after each wait and does nothing for someone who left.
16. **Server shutdown:** BindToClose runs the normal leave path for everyone (`GardenService.lua:770-781`).
17. **Rebirth while carrying an ore:** the ore goes back first (rule 24).
18. **Rebirth while mining:** mining continues; rebirth does not touch the mine.
19. **Owner leaves with visitors inside:** visitors are sent to their own plot spawn with a notice.
20. **Death in a mine:** respawn on the surface pad (`GardenService.lua:575-578`); carried ore returned by the silent drop; the watchdog clears `InUnderground`; the client restores the Backpack hotbar and normal lighting.
21. **Pulled into a royal duel while in a mine:** the arena moves the character; the watchdog clears `InUnderground` within 0.5 s.
22. **Full server, no plot:** no lift and no mine this session; ores and floors carry forward untouched.
23. **Corrupt or edited save:** sanitise clamps floors and drops unknown names; extra ores load but block mining until merged.
24. **Spending shards needed for a rebirth:** the confirm dialog states the shards left and the next rebirth's shard cost (U3).
25. **Streaming:** the server asks the destination to stream in before moving the player. If that fails twice the trip is refused with a warning and the player stays put. Mine models and ores use `ModelStreamingMode` Atomic. Nothing is Persistent ([[Roblox Platform Constraints for the Expansion]], section 2).
26. **Server authority:** every rule is checked on the server. Clients only draw. Ores and nodes are anchored and server-owned except while an ore is carried, the same exception fruit have.
27. **Mobile:** mining is one tap, then automatic; prompts are tappable; the Mine panel widens on touch; the carry row is the existing one.
28. **Fall and "water" checks:** no open drops (lift teleport, stair around a pillar), and `FallPeak` is cleared before each teleport. The 12-stud ground ray always finds a cave floor (`Ambient.lua:359-375`).

## 11. Performance

| Item | Budget | How it is kept |
|---|---|---|
| Server parts per player, 3 floors, full | about 330: shells 120, nodes 100, ores up to 15 × 7, lift 45 | small parts, no decor on the server |
| Server parts, 12 plots, all full | about 4,000 | same |
| Client decor | about 650 parts per floor; loaded only for the floor you are on, plus the one above or below while on the stair: at most about 1,300 | cloned from ReplicatedStorage on entry, destroyed on exit |
| PointLights | 18 per floor (at most 36 loaded), 3 with shadows | lights live in client decor only; shadows are what cost ([[Mining and Ore Merging - Reference Games]], section 7) |
| Particle emitters | 2 always-on per floor (dust, campfire); short bursts on hits | low rates; mobile caps each emitter at 100/s anyway ([[Roblox Platform Constraints for the Expansion]], section 6) |
| Textures | at most 60 per floor (Studded Pack Logs) | if the `Studs` MaterialVariant matches the log look side by side, use it instead, because textures do not batch |
| Kit in ReplicatedStorage | about 2,700 parts held in memory on every client, not rendered until cloned | one copy, reused |
| Per-frame client work | one attribute read; a box test at 4 Hz; PlayerVision lerps only while `cave` is moving, writes only on change (its `lightApplied` pattern, `PlayerVision.lua:413-418`) | |
| Per-frame server work | none. Timers: one mining loop per active miner at 2 Hz; one regrow sweep at 2 Hz over at most 120 nodes; one watchdog at 2 Hz | |
| Network | per swing: one HP attribute (about 20 bytes) and one `toolanim` value (about 60 bytes) at 2 Hz, about 160 B/s per miner; a node break sends one 7-part ore once; teleport is one move | far under the 50 KB/s per client budget ([[Roblox Platform Constraints for the Expansion]], section 3) |
| Prompts | per floor at most 6 node prompts (F) and 14 ore prompts (E), plus lift and board | one per key shows at a time; default limit 16 |
| Highlights | at most 12 ore highlights | inside the 24 cap (`GardenConfig.lua:1889`) |
| DataStore | +3 fields, worst case about 120 bytes | self-test keeps the 8,192-byte check (`GardenStore.lua:1209`) |
| Remote calls | 2 new client-to-server remotes, both one-shot | far under 500/s |

## 12. Test plan

Setup for every test: in Studio, run `workspace:SetAttribute("StudioNoSave", true)` in the command bar before pressing Play, so the playtest never writes the real save (`GardenStore.lua:572-575`). Give shards with the existing Studio door: in the server command bar, `game.Players:GetPlayers()[1]:SetAttribute("DevSetShards", 600000)` (`GardenService.lua:653`). For save tests, use the Dev Panel's Round trip and Reload buttons ([[World Builders and Dev Tools]]); with Studio API access off they use the in-memory store.

| Rule | Test |
|---|---|
| 1, 2 | With 99,999 shards press Buy on Floor 1: red "You need 100K shards", no change. Set 100,000, Buy, Yes: shards 0, `UndergroundFloors` = 1, Notice shown. From the client command bar fire `UndergroundBuy` with 3: nothing happens. Fire it with 2 while holding 140K: nothing (price 150K). |
| 2 | Start a rebirth and fire `UndergroundBuy` in the same frame from the command bar: no purchase while `rebirthing` is set. |
| 3, 24 | Buy Floor 1, place 4 Iron, rebirth (dev cash and Index): floors still 1, 4 Iron still on the floor, Ores Index unchanged. |
| 4 | With plot size 0 and then 4 (`DevSetPlotSize`), the lift stays inside the back-right hut, outside the pen, and the hut doors open only when `UndergroundFloors` is 1 or more. Builder check output lists no blocked plots; if it does, see section 14. |
| 5 | Others (second Studio client) see a boarded lift with dark lamps before purchase, and the right number of lit lamps after each floor. |
| 6 | Floor 1 only by lift. Floor 2 slab is solid and blocks walking until Floor 2 is bought; after purchase the slab is gone and the stair works both ways. |
| 7 | Carry a fruit, a TNT and an ore to each lift: each refused with the warning. |
| 8 | Second client: Visit Mine works; no Mine, Pick Up or Merge prompts show. On the visitor's client command bar, force a node prompt and an ore prompt on (`prompt.Enabled = true`) and press them: the server changes nothing (HP, ore count and `HeldBy` stay the same). Add a 7th visitor: refused. |
| 9, 10, 11 | Mine a node: 10 swings, 5.0 s (time it), Coal drops. Walk 11 studs away mid-mining: it stops. Press Mine again: it stops. Pick up an ore mid-mining: it stops and the pickaxe disappears. |
| 10 (look) | Watch the swing from a second client. If the default slash looks wrong, raise request A5. |
| 12 | Fill to 10 ores with 1 Coal among them: mining allowed (odd Coal). Make it 10 Iron: Mine refused with the "ore floor full" warning. With 9 Iron + 1 Coal, mine: 11 ores; merge the two Coal: 10 Iron. |
| 13 | Break a node and time the regrow: 25 s. Round trip: all nodes full. |
| 14, 15, 16 | Coal lands within 4 studs. Merge two Coal: one Iron appears at once where the target was. Carry an Iron to another Iron: no Merge prompt. Server log shows no ore missing if Round trip is pressed during a merge. |
| 17 | Drop in your mine: lands near your feet even at 11 of 10 ores. Store button hidden while carrying an ore; firing the `Store` remote returns "Ores stay in your mine." |
| 18 | Carry an ore and reset the character: ore reappears on Floor 1. Carry an ore and teleport the character to the surface from the command bar: ore back on Floor 1 within 0.5 s. |
| 19 | First Coal pick-up sparkles and adds Coal (value 0) to `OreIndexState`; Claim pays 500 shards and the right money; second Claim pays nothing. The rebirth gate ignores ores (rebirth allowed with ores unclaimed). |
| 20, 21 | With 10 Iron, `OreBoost` = 0.075; `CashMultiplier` is unchanged and the fruit part of `CashPerSecond` is 1.075 times its value with no ores, while pet income is unchanged. Pick up one Iron: `OreBoost` = 0.0675. Buy Floors 2-3 and fill 14 Iron: 0.105. Add a 15th ore with the dev seed: still capped at 0.105. |
| 22 | Note `CashPerSecond` with 10 Iron; Round trip with a back-dated `lastSeen` (dev seed): offline pay includes the 7.5% on the fruit part only. |
| 23 | Check the table in 5.3 for stage 6: give stage-6 fruit with the dev panel, fill the plot with Dragon Pepper, no pets, 10 Iron: `OreIncome` = 7.5% of the fruit income, so 0.075 / 1.075 of `CashPerSecond`. |
| 25 | Round trip: floors, ore counts and Ores Index identical; positions may differ; node HP full. |
| 26 | Second client visiting; first client leaves: visitor lands on their own plot spawn with the notice; the mine is gone from `workspace.Underground`. |
| 27 | Equip TNT in a mine: Use refused. A bat swing in a mine: no knockback (once [[Bats]] exists). |
| 28 | Enter and leave: lighting eases over 0.6 s both ways, no snap; day/night still correct outside. |
| UI | At 1600 × 900, 1920 × 1080, 844 × 390 (touch) and 667 × 375 (touch): the Mine panel, the Index tabs and the confirm dialog do not overlap the left or right HUD columns, the CornerHud, the top pills or each other. Buttons at least 44 px tall on touch. |
| Performance | In a mine with all 3 floors and 14 ores, check the Studio Stats: part count, draw calls and frame time before and after entering; record them in the vault. |

## 13. Dependencies and conflicts

- **[[Rarity Gating and Rebirth]]:** keeps the mine through rebirth (D-009 lists "underground"; this spec adds that ores and the Ores Index are kept too). Ores are not part of the "fill out your index" rebirth gate; that gate must count only `IndexState`. The stage decides how much ores pay only through plotBase, so the rarity gates need nothing from this system. Shards spent on floors compete with D-011 rebirth shard costs; the confirm dialog shows both.
- **[[Bats]]:** no knockback inside a mine: each floor's box is tagged `BatSafeZone`, and the bat safe band now reaches 240 studs below the soil (Bats B22). The pickaxe and the bat share one hand: starting to mine unequips the bat, equipping the bat stops mining (D-025).
- **[[Explosives and Fruit Health]]:** explosives are refused in a mine (D-031; that note's rule 34 now agrees). The node HP bar and fruit health bars use the one shared bar pair (King UI A8 and A9). Fruit losses from TNT lower plotBase, which lowers ore income by the same share; nothing extra is needed. If that system wants TNT as a mining tool later, nodes already have HP.
- **[[Weapons and Artillery]]:** artillery cannot hit a cave (it is 140 studs under the plot and has no server damage path); weapons do nothing inside a mine.
- **[[Pets]]:** shares the 35% side-income budget of the [[Stage Model]]. Resolved in the [[Integration Review]] (C16, D-018): pets keep their 25% limit (8.75 best-fruit units) and ores take 7.5 / 9 / 10.5% by floor. The Ore Bonus applies to fruit income only and pets are added to the base after it, so the two never compound: at the plateau they reach 25% + 10.5% = 35.5%. Pets stay on the surface in v1 and do not follow you into the mine. The Index has one tab column for Fruits, Pets and Ores (U4).
- **[[Boats and Swimming]]:** no direct link. Removing the sea currents also removes PlayerVision's airborne push (`PlayerVision.lua:266-297`), which the mine avoids anyway. If swimming ever allows diving, check that caves under the island cannot be seen from underwater.
- **[[Potions]]:** Speed and Jump work in a mine. Luck does not touch nodes in v1. Potions plans a Haste potion (pickaxe x1.5); the [[Integration Review]] moved it to ship with the second ore band, because the v1 mine fills in about 3 minutes and Haste would be bought for nothing (D-032). The ownership attribute it reads is `UndergroundFloors`.
- **[[Plot Overhaul and Environment]]:** builds the entrance hut in the back-right lot corner, opens its doors from `UndergroundFloors`, and provides the `LiftAnchor` attachment; this system adds the lift cage, lamps, sign and prompts inside it (D-023). The lift's materials match the new plot style. New plot decor must not be placed in the plot model with the name `Part` (`Upgrades.lua:168-173`) and must not be bigger than the soil (`PlotRegistry.lua:21-30`); the lift already follows both rules by living outside the plot model. The later "towers upward" upgrade will stack above the plot, so it should reuse this system's floor table shape (price, slots, bonusCap) for consistency.
- **[[King UI Overhaul]]:** uses the same kit; no screen-zone conflict (the royal top-left stack and top-centre card are at `RoyalUI.lua:303` and `:355-356`, and the Mine panel is centred like the Index). Empire Power becomes the `EmpirePower` attribute ([[Explosives and Fruit Health]] code plan), which includes the Ore Bonus and pets and ignores fruit stuns; that is fair, because it is real income.
- **[[Royal Combat]]:** no interaction. If a duel moves a player out of a mine, the watchdog restores their state.

Decisions touched: D-009 (extended to ores and the Ores Index, not contradicted). D-003 (the studded Logs are reused for timber). D-005 (any custom swing must be published by Samarth). D-010 and D-011 (used, not changed).

## 14. Open questions for Samarth

Each has a default so building can start now.

1. **Stage Model mismatch.** The model assumes pets and ores pay 35% from stage 1, but at 100K shards nobody owns a mine before about 7.2 h (stage 7, not stage 6 as the model note says). The realistic total is about 11.1 h instead of 10.8 h. Options: accept it, raise the per-merge shard drop (D-010) so the mine comes earlier, or lower the price. **Default: accept; keep 100K as you set it.**
2. **Ore income as a share of fruit income.** This spec pays ores as a percentage of your current fruit income (Coal 0.3%, Iron 0.75%, cap 7.5% / 9% / 10.5%, fruit income only). The alternative is a fixed amount per stage, which section 5.3 shows would skip whole stages after a rebirth. **Default: percentage.**
3. **Pet and ore split of the 35%.** **Decided in the [[Integration Review]] (D-018): pets 25%, ores 7.5 / 9 / 10.5% by floor, fruit only, no compounding.** The alternative, 17.5% each, is listed in the review's open questions.
4. **Ores through rebirth.** Kept (your D-009 "underground" read as including the ores). The alternative, ores reset each rebirth like fruit, would make mining something you redo every stage. **Default: kept.**
5. **Floor prices after Floor 1.** **Default: 150K and 300K** (section 5.2).
6. **Visitors.** **Default: anyone may visit, at most 6, look only, no kick button in v1.** Alternative: friends only.
7. **Lift instead of a walk-down shaft.** **Default: lift with a fade**, because the island's underside is unknown and a shaft needs a hole in the plaza. If you want to walk down, the builder's volume check (7.9) tells us if a shaft is possible.
8. **Lift location.** **Default: inside Plot Overhaul's stone hut in the back-right corner of every lot** (D-023). The earlier plaza-side spot would have stood in the new front yard and walkway.
9. **Pickaxe animation.** **Default: the default tool slash (no work for you).** Request A5 only if it looks wrong.
10. **Explosives underground.** **Default: not allowed in v1.**
11. **Iron glint.** Iron is Common, but a faint sparkle makes merging feel like an upgrade. **Default: yes, very faint (0.4/s).** Real glow starts at Rare ores in the later ladder.
12. **Later ore ladder.** **Default: one band of two ores per rarity colour, each band on a new deeper floor (depth-gated), names from 5.1 as placeholders.** Alternative: gate bands by rebirth like fruit. Decide before the next ore ships.
13. **Ores Index rewards.** **Default: 500 and 1,000 shards plus 60 s of income, no coin-bonus share.**
