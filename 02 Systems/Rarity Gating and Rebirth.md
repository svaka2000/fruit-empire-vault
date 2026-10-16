---
tags: [system, design]
date: 2026-10-04
status: proposed
depends_on: []
used_by: [Bats, Pets, Underground and Ores, Explosives and Fruit Health, Weapons and Artillery, Boats and Swimming, Potions, Plot Overhaul and Environment, King UI Overhaul, Royal Combat]
---
# Rarity Gating and Rebirth

This is the base system. Every other system prices itself against the stage it sets. It turns [[Stage Model]] and decisions D-006 to D-011 in [[Decisions Log]] into rules, screens and code changes for the live game.

**Words used in this note**
- **Rank**: a fruit's place on the 64-fruit ladder. Rank 64 is Blueberry (worst), rank 1 is Soul Fruit (best). Two fruit of rank R merge into one of rank R-1 (`ReplicatedStorage.Fruits.FruitIndex.lua:5`).
- **Tier / rarity**: the 9 colour groups, tier 1 COMMON to tier 9 TRANSCENDENT (`ReplicatedStorage.Fruits.FruitIndex.lua:10-73`).
- **R**: the player's rebirth count (the `Rebirths` attribute). Counted 0 to 8.
- **Stage S**: `S = R + 1`. The stage is the newest rarity the player has unlocked. Stage 1 is Common, stage 9 is Transcendent.
- **Unlocked fruit**: every fruit whose tier is S or lower.
- **Cap fruit / merge cap**: the best fruit of tier S. Two cap fruits never merge until the next rebirth.
- **Spawn ceiling**: the best fruit the plot can grow in this stage. It sits 2 ranks below the cap fruit (D-007).
- **Luck at the ceiling**: the Luck potion adds one Spawn Rarity level but never goes past the spawn ceiling. When Spawn Rarity is already at the ceiling, drinking Luck is refused ([[Potions]] rule 26, decision D-019). There is no separate "Luck ceiling" (changed by [[Integration Review]], conflict C6).
- **Head start**: the Spawn Rarity level a player starts a stage on (D-006).
- **Sealed**: a new Index state (proposal). The fruit's rarity is not unlocked yet, so it cannot be found until a rebirth.
- **Full-plot $/s**: 35 cap fruit times the rebirth multiplier. The most a plot earns in that stage, before the Index bonus ([[Stage Model]]).
- **Newest-rarity merge**: a merge whose result fruit is in tier S. It pays shards (D-010).
- **Placement**: the one-time save migration that lifts an existing player's rebirth count so their cap covers the fruit they already own (proposal).
- **Toast**: the one-line message that slides on screen. Red WARN from the top, green NOTICE from the bottom (`ReplicatedStorage.GardenToast.lua:49-67`).

## Why

Samarth's words, from [[Brain Dump 2026-10-04]]:

> also rarity in the index should be somethng that you have to unlock by rebirthing. this will match with the color of the bat that you unlock. when you have zero rebirths the max fruit you can get is apple. you cant merge past that. as you rebirth you can also get a bigger storage. you get +1 storage everytime you rebirth. to rebirth you need to fill out your index + you need a certain amount of shards + a certain amount of money. dont make it easy to rebirth. then once they rebirth they get a slight boost and they start from the beginning and they have access to both common and uncommon. it should be obvious to them that to unlock more fruits to merge and more fruits in the index they need to rebirth.

> you can use rebirths to control and organize this. the stages are of the game is the rarity of fruit the player is on starting from the common category all the way to transcendent.

> also all Uis you make should follow the UI that we created

> what i noticed with the king feature in the sky island is that it was a bunch of dopamaine slop. instead everything should be structured. like for players its going to be confusing. all the UI should never overlap and also everything should be super clean.

> I dont like the emoji-slop that you pushed on the sky island kings feature.

What the system is for. Today any player can merge all the way to Soul Fruit, and rebirth only wipes the run for a cash multiplier (`ServerScriptService.GardenService.RebirthLock.lua:35-146`). This system makes each rebirth open exactly one new rarity, so the rarity a player is on becomes the "stage" that every price and reward in the game is pinned to. It also has to teach itself: the Index, the moment you hit the cap, and the Rebirth menu must all say plainly that rebirthing unlocks more fruit.

## Player view

1. **A new player (0 rebirths).** The plot grows Blueberries. They buy Spawn Rarity up to Carrot. After that the Spawn Rarity card reads "Up to Carrot" with a grey LOCKED plate and the line "Rebirth to raise".
2. **First Apple.** They merge two Tomatoes. The Apple gets the normal discovery sparkle. A small panel titled MAX FRUIT opens in the middle of the screen. It shows the Apple card and three lines: "Apple is your best fruit", "Two Apples cannot merge until you rebirth", "Rebirth 1 unlocks 8 Uncommon fruits". It has a red X and a green REBIRTH button.
3. **Trying anyway.** Later they carry an Apple to another Apple. The prompt reads "Max Fruit" with "Rebirth to merge" under it. The Apple has no green match glow. If they press E, a red toast says "Apple is your best fruit. Rebirth to unlock Uncommon fruit." The Apple stays in their hands.
4. **The Index.** The five Commons look as they do today. Every card from Dandelion on is dimmed, shows a padlock, and reads "Rebirth 1" (Uncommon) up to "Rebirth 8" (Transcendent). The title bar reads "Found 4/5" on the left.
5. **The Rebirth menu.** It looks like the Index. The big left box says "Rebirth 1 unlocks 8 Uncommon fruits" and shows those 8 cards. The right top box, NEEDS, lists Index 4/5, Cash $41.2K / $56.5K, Shards 40 / 50, with a green tick on each one that is met and red text on each one that is not. The right bottom box, REWARDS, shows +25% cash, +1 storage and the next bat. The REBIRTH button stays grey until all three needs are met.
6. **Ready.** When all three are met, the red "!" badge appears on the Rebirth button. They open the menu and press REBIRTH. The confirm dialog says what resets and what stays. They press Yes.
7. **After the rebirth.** The plot is cleared and seeded with 3 starter fruit. Cash is 0. 50 shards are gone. The Index is kept, and its Uncommon cards are now ordinary "???" cards waiting to be found. Storage reads 0/6. Cash earns +25%. Every merge that makes an Uncommon fruit pays 7 shards, shown as a small purple "+7" over the new fruit.
8. **Later rebirths.** From rebirth 2 on, the plot starts from the first fruit of the rarity just below the new one, so old rarities are not replayed (D-006).
9. **The end.** At 8 rebirths the Rebirth menu reads "Every rarity is unlocked" and has no button.
10. **Existing players.** On their first join after the update, a green toast says "Rarities now unlock with rebirths. You are on Rebirth 8: Transcendent fruit unlocked." They keep every fruit they own.

## Rules

All rules below are proposals that implement the brain dump and D-006 to D-011. Numbers for each stage are in the next section.

**Stage and merge cap**
1. Stage `S = min(R, 8) + 1`. Unlocked fruit are all fruit with tier at most S.
2. A merge is legal only if the result fruit's tier is at most S. The cap fruit is the lowest rank in tier S: rank 60, 52, 41, 33, 26, 18, 13, 5, 1 for stages 1 to 9.
3. **Hand merge at the cap.** The server refuses before the carried fruit is consumed. The fruit stays in the player's hands. The server sends one red toast (`Config.MergeCapText`) and one `RarityGate` event of kind `"attempt"`. The toast is sent at most once per 1.5 s per player.
4. **Soul Fruit.** Carrying Soul Fruit (rank 1) onto Soul Fruit gets the red toast `Config.MergeBestText` instead of the silent return it gets today (`ServerScriptService.GardenService.Merging.lua:179-180`).
5. **Prompt at the cap.** When the carried fruit has rank at most the cap rank and the target matches, the prompt's ActionText is "Max Fruit" and ObjectText is "Rebirth to merge" ("Best fruit in the game" at stage 9). That pair gets no green match highlight. The prompt stays enabled so a press always gets an answer.
6. **Merge All at the cap.** The cascade never promotes a pair whose result is above the cap. Cap pairs stay on the plot. If a press merged nothing and at least one cap pair exists, the player gets the red toast `Config.MergeAllCapNothing` and no cooldown starts (the existing rule at `ServerScriptService.GardenService.Merging.lua:566-568`). If it merged something and cap pairs remain, the green toast also gives the number of cap pairs waiting.
7. **The cap moment.** The first time an account makes the cap fruit of its newest rarity, by hand or by Merge All, the server sends `RarityGate` kind `"reached"`. The client opens the Max Fruit popup. The Index is kept (D-009), so this fires once per stage per account. After an `"attempt"`, the client opens the popup only if it has not opened in this session for this stage.

**Spawning**
8. **Spawn ceiling (D-007).** The plot never spawns above Spawn Rarity level `62 - capRank(S)`: Carrot, Candy Sunflower, Eggplant, Pumpkin, Pineapple, Cactus, Serenity, Lunar Cherry, Zenflare.
9. **Spawn Rarity at the ceiling.** `Config.UpgradePrice` returns nil at the ceiling. A purchase press there gets the red toast `Config.SpawnCapText`. The alert badge on the Upgrade button does not count it.
10. **Luck potion.** While Luck is active the plot picks from `min(level + 1, spawnCeiling(S))`. At the ceiling, drinking Luck is refused with "Spawn Rarity is maxed. Save it for after your next rebirth!" ([[Potions]] rule 26). At stage 9 there is no next rebirth: the line is "Spawn Rarity is maxed for good." and the Shop stops selling Luck once the stage 9 ceiling is reached ([[Potions]] UI 1; player-path pass, [[Endgame - Transcendent]]). So the top 2 fruits of tier S never spawn, with or without Luck (D-007 kept strict, decision D-019). A Lucky fruit still enters the Index when picked up (`ServerScriptService.GardenService.Carrying.lua:185-196`).
11. **Head start (D-006).** On a rebirth to R = 1, Spawn Rarity goes to level 0 (Blueberry). On a rebirth to R of 2 or more, it goes to `64 - firstRank(R)`, the first fruit of tier R (the rarity below the new one). Capacity, spawn rate and plot size go to level 0.

**Rebirth**
12. **Requirements to leave stage S (S = 1 to 8)**, checked by the server on every press: (a) every unlocked fruit is discovered in the Index (an IntValue exists, claimed or not); (b) cash at least `cashNeed(S)`; (c) shards at least `shardNeed(S)`.
13. At stage 9 (8 rebirths) the server refuses with `Config.RebirthAllUnlocked`.
14. The server refuses while any of the player's merges is mid-animation (`pendingFor(plot) > 0`) with `Config.RebirthMerging`. This also fixes a live race: a merge that finishes during a rebirth places its fruit on the fresh plot (`ServerScriptService.GardenService.Merging.lua:99-106`). It also refuses, with "Finish your royal fight first", while the player has `InArena` or `G.royalBusy(player)` is `duel`, `contest` or `spar` ([[Royal Combat]] rule 54, looked up through G at call time).
15. **What a rebirth does.** Shards go down by `shardNeed(S)`. Cash goes to 0 (so the cash need is a threshold, not a second charge). R goes up by 1. Plot fruit, stored fruit, Best Merge and every cash upgrade except Plot Size reset, with Spawn Rarity set to the head start. **Plot Size is kept** (decision D-020, from [[Plot Overhaul and Environment]] rule E1). The Index is kept and pays no shards again (D-009). Tools are kept, and a tool in the player's hands goes back to the bag first (today it is destroyed: `releaseHold(player, true)` at `ServerScriptService.GardenService.RebirthLock.lua:71-73` does not return it, unlike `stowTool` at `ServerScriptService.GardenService.Tools.lua:172-181`). **One exception:** explosives (TNT, Big TNT, Mega TNT, Nuke) are turned back into shards at their pre-rebirth price before R goes up ([[Explosives and Fruit Health]] rule 4, D-020). Potions, weapon charges, pets, the mine, the boat, the bat and blade skins are kept. Running potions are kept. Three starter fruit are seeded at the head start level.
16. **Cash multiplier (D-008).** `1 + 0.25 x min(R, 8)`, added, live and offline. It still multiplies with the friend and Index bonuses (`ServerScriptService.GardenService.Progression.lua:332-336`).
17. **Storage.** The storage cap is `5 + min(R, 8)` fruit, all kinds together. As today it is enforced only when storing (`ServerScriptService.GardenService.Storage.lua:43-47`).
18. **Shard drop (D-010).** Every newest-rarity merge pays `shardDrop(S)` shards. A hand merge pays once. Merge All pays once per pair it promotes into tier S. Merges into lower tiers pay nothing.
19. **Rebirth badge.** The red "!" on the Rebirth button shows only when rule 12 (a), (b) and (c) all hold and S is below 9.

**Index**
20. **Sealed state.** A fruit with tier above S and no Index entry is Sealed. Its card is dimmed and shows a padlock and the text "Rebirth N", where N = tier - 1. Its Claim button is grey. A fruit that has an Index entry is never shown as Sealed.
21. The Index title bar shows "Found n/m": m is the number of unlocked fruit, n is how many of them are discovered.

**Migration (runs once per account)**
22. When a loaded profile has `gateVersion` below 1, before anything is applied: let T be the highest tier among plot fruit, stored fruit, `best` and Index entries. Set `rebirths = max(saved rebirths, min(T - 1, 8))`. Clamp the Spawn Rarity level to the new spawn ceiling and pay back the cash price of each removed level. Fruit, Index, cash, shards and tools are not touched. Set `gateVersion = 1`. At `Config.RefundNoticeDelay` + 4 s after load, send the notice `Config.RarityMigratedNotice`; when Spawn Rarity levels were refunded, the notice also names the cash paid back. The 4 s gap keeps it out of the frame of the retired-item refund notice: the green toast has one label and a second line sent in the same frame replaces the first (`ReplicatedStorage.GardenToast.lua`). (Timing and refund text added by the player-path pass, [[Existing Player Migration]].)
23. Running the migration twice gives the same result.

**Look**
24. Every new screen uses kit images from [[UI Kit Inventory]], FredokaOne white text with a black stroke, and no emoji. New text over fruit uses Highway, the game's rule for in-world fruit text (`ReplicatedStorage.GardenConfig.lua:20-25`).

## Numbers

### Formulas
- `capRank(S)` = the lowest rank with tier S. `firstRank(t)` = the highest rank with tier t. Both come from `FruitIndex` at load.
- `spawnCeiling(S) = 64 - (capRank(S) + 2)` (a Spawn Rarity level).
- Luck: `min(level + 1, spawnCeiling(S))`; refused when `level >= spawnCeiling(S)` (rule 10).
- `headStart(R) = 0` if R is below 2, else `64 - firstRank(R)`.
- `storage(R) = 5 + R`.
- `mult(R) = 1 + 0.25 R`.
- `full(S) = 35 x income(capRank(S)) x mult(S - 1)`, where income is `FruitEconomy.ForRank`.
- `cashNeed(S) = roundPrice(k_S x full(S))`, with `k_S` = 79.92, 241.8, 2170, 2996, 4375, 5224, 7091, 11300 seconds for S = 1 to 8. These are D-011's cash values, rewritten as seconds of that stage's full-plot income (they reproduce D-011 exactly after the existing 3-figure rounding, `ReplicatedStorage.GardenConfig.lua:161-165`). If the income ladder is ever retuned, the cash needs follow it.
- `shardDrop(S) = round(3 x 2.2^(S - 1))` (D-010).
- **Shared shard price and reward (decision D-017, one copy for every system):** `ShardPrice(units, R) = NicePrice(units x shardDrop(min(R, 8) + 1))`, where `NicePrice(x)` rounds to the nearest 5 below 100 and to 2 significant figures above (halves up). Every shard price in the expansion (potions, explosives, weapons, blade skins) goes through it. `ShardReward(units, R) = units x shardDrop(S)`, exact, for every stage-scaled payout (Golden Crate, royal and decree rewards, nuke survivor bonus, Rotten Fort). See [[Integration Review]] conflict C7.
- `shardNeed(S)` = D-011's shard column. It is 16% to 27% of the shards the median simulated player holds at the gate.

### Table 1: the gate per stage

| S | Rarity | R | Merge cap (rank) | Spawn ceiling (level) | Luck stops at (refused once Spawn Rarity reaches it) | Head start on entering S (level) | Storage | Cash mult | Unlocked fruit (Index to fill) |
|---|---|---|---|---|---|---|---|---|---|
| 1 | Common | 0 | Apple (60) | Carrot (2) | Carrot | Blueberry (0), new player | 5 | x1.00 | 5 |
| 2 | Uncommon | 1 | Blue Lollipop (52) | Candy Sunflower (10) | Candy Sunflower | Blueberry (0) | 6 | x1.25 | 13 |
| 3 | Rare | 2 | Pear (41) | Eggplant (21) | Eggplant | Dandelion (5) | 7 | x1.50 | 24 |
| 4 | Epic | 3 | Watermelon (33) | Pumpkin (29) | Pumpkin | Bee Balm (13) | 8 | x1.75 | 32 |
| 5 | Legendary | 4 | Tropical Melon (26) | Pineapple (36) | Pineapple | Lemon (24) | 9 | x2.00 | 39 |
| 6 | Mythic | 5 | Dragon Pepper (18) | Cactus (44) | Cactus | Cacao (32) | 10 | x2.25 | 47 |
| 7 | Astral | 6 | Paradise Petal (13) | Serenity (49) | Serenity | Passionfruit (39) | 11 | x2.50 | 52 |
| 8 | Eternal | 7 | Solar Bloom (5) | Lunar Cherry (57) | Lunar Cherry | Glowshroom (47) | 12 | x2.75 | 60 |
| 9 | Transcendent | 8 | Soul Fruit (1) | Zenflare (61) | Zenflare | Sarracenia (52) | 13 | x3.00 | 64 |

### Table 2: cash, pinned to full-plot $/s

| S | Cap fruit $/s | full(S) | cashNeed(S) | k_S (seconds of full plot) | k_S in minutes | Median time in stage ([[Stage Model]]) |
|---|---|---|---|---|---|---|
| 1 | 20.2 | 707 | 56.5K | 79.92 | 1.3 | 15 min |
| 2 | 9.83K | 430K | 104M | 241.8 | 4.0 | 34 min |
| 3 | 71.2M | 3.74B | 8.11T | 2170 | 36.2 | 61 min |
| 4 | 59.4B | 3.64T | 10.9Qa | 2996 | 49.9 | 75 min |
| 5 | 25.5T | 1.78Qa | 7.81Qi | 4375 | 72.9 | 90 min |
| 6 | 31.6Qa | 2.49Qi | 13.0Sx | 5224 | 87.1 | 105 min |
| 7 | 3.03Qi | 265Qi | 1.88Sp | 7091 | 118.2 | 120 min |
| 8 | 5.26Sx | 506Sx | 5.72Oc | 11300 | 188.3 | 150 min |
| 9 | 237Sx | 24.9Sp | none (final) | - | - | endgame |

`k_S` grows because a player only has a full plot near the end of a stage. The real "hard to rebirth" cost is the stage time, which the simulation solved for (D-011).

### Table 3: shards

| S | shardNeed(S) | shardDrop(S) per newest-rarity merge | Merges to cover the need | Median shards held at the gate (sim) | Need as % of held | Index shards for tier S (one time) |
|---|---|---|---|---|---|---|
| 1 | 50 | 3 | 17 | 232 | 22% | 31 |
| 2 | 250 | 7 | 36 | 1.5K | 16% | 94 |
| 3 | 1.5K | 15 | 100 | 6.6K | 23% | 343 |
| 4 | 3.5K | 32 | 110 | 12.8K | 27% | 610 |
| 5 | 7.5K | 70 | 108 | 28.3K | 27% | 1,125 |
| 6 | 20K | 155 | 130 | 86.2K | 23% | 2,700 |
| 7 | 30K | 340 | 89 | 111K | 27% | 3,200 |
| 8 | 100K | 748 | 134 | 439K | 23% | 9,750 |
| 9 | - | 1,646 | - | - | - | 8,675 |

Shards held come from `~/fruit-empire/sim/final_results.json` (attentive player, 7 careers). Index shards per tier are from [[Config and Fruit Data]].

### Table 4: what the spawn ceiling is worth, relative to the cap fruit

| S | Spawn ceiling fruit $/s as % of cap fruit $/s |
|---|---|
| 1 | 22.1% |
| 3 | 19.2% |
| 6 | 16.5% |
| 9 | 14.8% |

The last two fruits of the newest rarity are always merged, never grown, with or without Luck (D-007 strict, D-019). The earlier "Luck grows the second-best fruit" proposal was dropped in the [[Integration Review]] (conflict C6): [[Potions]] measured that it saves only 0.2 to 1.0 minute late in a stage, while Luck drunk early in a stage saves 2.5 to 3.4 minutes.

### Worked values
- **Stage 1 (0 rebirths).** Cap Apple, $20.2/s. full(1) = 35 x 20.2 x 1.00 = $707/s. To rebirth: 5/5 Commons, $56.5K (79.92 s of full plot) and 50 shards. Spawns up to Carrot; Luck is refused once Spawn Rarity reaches Carrot. Storage 5. Each Common merge pays 3 shards.
- **Stage 3 (2 rebirths).** Cap Pear, $71.2M/s. full(3) = 35 x 71.2M x 1.50 = $3.74B/s. To rebirth: 24/24 fruit, $8.11T (2,170 s, about 36 min of full plot) and 1,500 shards. The plot starts at Dandelion (level 5), spawns up to Eggplant (21); Luck stops at Eggplant. Storage 7. Each Rare merge pays 15 shards.
- **Stage 6 (5 rebirths).** Cap Dragon Pepper, $31.6Qa/s. full(6) = 35 x 31.6Qa x 2.25 = $2.49Qi/s. To rebirth: 47/47 fruit, $13.0Sx (5,224 s, about 87 min) and 20K shards. Start Cacao (32), ceiling Cactus (44). Storage 10. Each Mythic merge pays 155 shards.
- **Stage 9 (8 rebirths).** Cap Soul Fruit, $237Sx/s. full(9) = 35 x 237Sx x 3.00 = $24.9Sp/s. No further rebirth. Start Sarracenia (52), ceiling Zenflare (61). Storage 13. Each Transcendent merge pays 1,646 shards.

### Migration worked examples
- **Samarth's save** (rebirths 0, best Zenflare, cash 6.46e26). Zenflare is tier 9, so T = 9 and rebirths become min(9 - 1, 8) = 8. He lands on stage 9: Transcendent unlocked, cash x3.00 (was x1.00), storage 13. Every fruit stays. Cash stays at 6.46e26. If his Spawn Rarity level is 61 or lower nothing else changes. At level 62 he gets back 5.14Oc; at level 63 he gets back 5.14Oc + 13.4Oc = 18.5Oc (the prices of levels 62 and 63, `ReplicatedStorage.GardenConfig.lua:167-187`).
- **The dev panel's seed profile** (best Lemon, plot fruit at ranks 20, 35, 50, 64, `ServerScriptService.GardenService.DevPanel.lua:224-237`). Cactus (rank 20) is tier 6, so rebirths become 5 (stage 6). Its Spawn Rarity is 0, so nothing is refunded.

## UI

Every screen below is built from [[UI Kit Inventory]] images. Sizes are at the HUD reference of 1600 x 900 (`ReplicatedStorage.GardenConfig.lua:1871`). Panels are Scale-sized with an aspect lock, like every existing panel; nothing new joins the Offset HUD (see [[Consistent Image-Based UI Kits in Roblox]], section A6).

### U1. Index: Sealed card
- **Where:** each cell in `IndexMenu.List` (built at `StarterPlayer.StarterPlayerScripts.IndexMenu.lua:301-361`).
- **Kit parts:** the tier's card frame (`Config.RarityFrames`) with `ImageColor3 = Config.IndexSealedTint` (150, 150, 150). A new `Padlock` ImageLabel (asset A3) centred over the viewport area, 44% of the cell width, ZIndex 10 (the cell is 9, the Transcendent accent 11, the selected cell 20, `IndexMenu.lua:200`, `:343`). FruitName unchanged. The Rarity label (normally "1/2.3Qa") reads "Rebirth N".
- **States:** sealed, locked ("???", existing), unclaimed, claimed. Sealed is checked after "has an entry" and before locked.
- **Overlap:** the padlock replaces the code-drawn "???" pane (`IndexMenu.lua:95-188`); the two are never shown together. The viewport is hidden while sealed.

### U2. Index: info box for a Sealed fruit
- **Where:** `IndexMenu.InfoBox` (`IndexMenu.lua:243-273`).
- **Kit parts:** the existing InfoBox art (101725900357985). The Padlock (A3) at 60% of the box width over `InfoBox.Model`. FruitName as today. Rarity reads "Unlocks at Rebirth N". Tier keeps its tier colour. Rate keeps the $/s, so the player sees how big the next rarity is.
- **Rewards box:** chips show the rewards as a preview. Claim is grey (`Config.IndexClaimedTint`), exactly as for a locked fruit.

### U3. Index: "Found n/m"
- **Where:** `IndexMenu` title bar, left third. AnchorPoint (0, 0.5), left edge at 3.5% of the bar, width 20% of the bar, height 60% of the bar.
- **Text:** FredokaOne, white, black UIStroke rescaled with the panel like every Index stroke (`IndexMenu.lua:608-619`). "Found 12/13".
- **Overlap:** the INDEX title image (201 of 938 px) is centred and spans about 39% to 61% of the bar; the red X sits at the right end. The label ends at 23.5%.

### U4. Rebirth menu (rebuilt)
- **Where:** `GardenUI.RebirthMenu`, opened by the Rebirth HUD button (right column). The name still ends in "Menu", so the loading screen pre-draws it (`ReplicatedFirst.LoadingScreen.lua:411-416`).
- **Shell:** a copy of the IndexMenu shell, so every rectangle is the one already measured from the mockup: TitleBlue (95532041481396), TitleFrame (110138832714236), Body (82827190351667), Inline (110403403749726), ScrollBox (83403190365135), InfoBox (101725900357985), RewardsBox (126819257706583) and IndexMenu's red square X. The title image becomes "REBIRTH" (asset A1). Same aspect lock (1831 x 969, `ReplicatedStorage.GardenConfig.lua:424`). Same open and close tweens (0.22 s Back out, 0.14 s Quad in) and the same "close the other panels" list as today (`StarterPlayer.StarterPlayerScripts.RebirthMenu.lua:46-51`).
- **ScrollBox, "what you unlock":**
  - Header line, MaxTextSize 26: "Rebirth 1 unlocks 8 Uncommon fruits". The tier word uses `Config.RarityColors`, as the Index info box already does (`IndexMenu.lua:259`).
  - A grid of the next tier's fruit: clones of `IndexMenu.Templates.Cell`, 6 columns, pixel cell heights (the scroll rule at `IndexMenu.lua:526-545`). They show the normal "???" look, because they are undiscovered. 4 to 11 cells, so at most 2 rows.
  - Footer line, MaxTextSize 18: "Your plot starts again from Blueberry" (rebirth 1) or "Your plot will grow from Dandelion" (head start, rebirth 2+).
  - Second footer line, MaxTextSize 18, only when the next rebirth unlocks a shop item: "Also unlocks: Melon Mortar, Big TNT" (the items whose unlock stage is the next stage, read from `WeaponConfig` and the explosive `Config.Tools` entries). This answers the [[Weapons and Artillery]] and [[Explosives and Fruit Health]] requests and makes "rebirth to unlock more" visible, as the brain dump asks. Added by [[Integration Review]].
- **InfoBox, "NEEDS":** header "NEEDS", then three rows. Each row is a chip image at 34 px, a value label, and a 22 px tick (asset A6) when met.
  - Index chip (asset A4): "12/13".
  - Money chip (82056408182031): "$41.2K / $56.5K".
  - Shards chip (78947574768646): "40 / 50".
  - An unmet value uses `Config.UpgradeUnaffordable` red (`ReplicatedStorage.GardenConfig.lua:271`), the kit's "cannot afford" colour. A met value is white.
- **RewardsBox, "REWARDS":** three chips with a value label under each, like the Index rewards box.
  - Bonus chip (89282702522748): "+25%".
  - Storage chip (asset A5): "+1".
  - Bat chip ([[Bats]] asset request 5, the tintable bat in the reward-chip frame, tinted the next tier's colour): label "Power 56". Hidden until Bats ships; the other two chips then centre.
  - The action button is the existing green REBIRTH pill (86156862866253) in place of Claim, 80% of the box width. Grey (`IndexClaimedTint`) and inactive until ready. A press while grey shows the client toast for the first unmet need.
- **Final stage:** the ScrollBox reads "Every rarity is unlocked" over the 4 Transcendent cards. NEEDS shows Index 64/64 only. Rewards and the button are hidden.
- **Confirm dialog:** the existing `RebirthConfirm` (ClearConfirm art 112127226387884, Yes 71592791133553, No 105863640938118). Title "Rebirth?". Body: "Resets cash, upgrades, your plot and stored fruit. Keeps your Index, shards, tools, pets and Plot Size." A second line appears only when the player owns explosives: "Your explosives turn back into N shards."
- **Retired:** the old panel art (123255751134677), the LockedOverlay (71043479073116) and the cash and shard reward boxes. The overlay hid the panel until the Index was full, which hides exactly what the brain dump wants shown.
- **Overlap:** a full panel; opening it hides the other four panels, as today.

### U5. Max Fruit popup
- **Where:** new `GardenUI.MaxFruitPopup`, a full-screen transparent frame with a centred card, built like `OfflinePopup` (modal, with a Blocker, `StarterPlayer.StarterPlayerScripts.OfflinePopup.lua:89-103`).
- **Card size:** 36% of screen width, aspect 520:290, UISizeConstraint min 340 x 190 and max 620 x 346. Centred (AnchorPoint 0.5, 0.5).
- **Kit parts:** TitleBlue bar across the top 12% of the card with the "MAX FRUIT" title image (asset A2) and IndexMenu's red square X at the right end. The Body image (82827190351667) below.
- **Content:**
  - Left 30%: one Index cell clone showing the cap fruit, unlocked look.
  - Right: line 1, MaxTextSize 26: "Apple is your best fruit". Line 2, MaxTextSize 18: "Two Apples cannot merge until you rebirth". Line 3, MaxTextSize 18: "Rebirth 1 unlocks 8 Uncommon fruits" (tier word in its tier colour).
  - Bottom right: the green REBIRTH pill (86156862866253), 38% of the card width. It closes the popup and opens the Rebirth menu.
- **Stage 9 variant** (added by the player-path pass, [[Endgame - Transcendent]]). At stage 9 the cap fruit is Soul Fruit and no rebirth is left, so the stage 1 to 8 lines would be false. Line 1: "Soul Fruit is the best fruit in the game". Line 2: "You have unlocked every rarity". Line 3 is hidden. The REBIRTH pill is hidden; the red X closes the card. Rules 7 and 4 still apply: the popup opens once, on the first Soul Fruit.
- **States:** hidden, open, queued. It waits while any panel ending in "Menu", OfflinePopup, ClearConfirm or RebirthConfirm is visible, then opens.
- **Overlap:** a modal in the centre. The WARN toast lands at 76 to 104 px from the top (`ReplicatedStorage.GardenToast.lua:53-54`), well above the card's top edge (about 290 px: a 576 x 321 card centred on a 900 px tall screen).

### U6. Fruit prompt at the cap
- The fruit prompts are Roblox's default ProximityPrompt look today (`StarterPlayer.StarterPlayerScripts.GardenClient.lua:300-310`). Only the text changes (rule 5). ObjectText is cleared again on the Pick Up and Merge branches.

### U7. Spawn Rarity card at the ceiling
- **Where:** the Spawn Rarity card in `UpgradeMenu` (`StarterPlayer.StarterPlayerScripts.UpgradeMenu.lua:57-85`).
- **Rows:** Level "Level 3/3" (the shown maximum is the ceiling + 1). Stat "Up to Carrot". The Price row reads "Rebirth to raise" in `Config.UpgradeStatColor` gold (`ReplicatedStorage.GardenConfig.lua:269`). At stage 9 it reads "Max for every rarity" instead, because no rebirth is left (player-path pass, [[Endgame - Transcendent]]).
- **Button:** the LOCKED plate (asset A7) in the Maxed plate's rectangle (`Config.UpgradeButtonRect`, `Config.UpgradeMaxedHeight`). Until A7 exists, the Maxed plate (76326716689640) tinted with `IndexClaimedTint`.
- **Overlap:** uses the existing Price and Maxed slots only.

### U8. Storage counters
- The Store button badge (`GardenClient.lua:365-412`) and the Inventory title count (`StarterPlayer.StarterPlayerScripts.StoredFruitsMenu.lua:448-471`) read `Config.StoredCapFor(R)` instead of the constant 5. No layout change.

### U9. Rebirth HUD badge
- The existing AlertBadge art (79539007098845) on RebirthButton. Logic change only (rule 19).

### U10. Toasts (all Config strings, sent through `Notify` or `Notice`)

| Key | Channel | Text |
|---|---|---|
| `MergeCapText` | red | "%s is your best fruit. Rebirth to unlock %s fruit." |
| `MergeBestText` | red | "Soul Fruit is the best fruit in the game." |
| `MergeAllCapNothing` | red | "Nothing can merge. %s is your best fruit until you rebirth." |
| `MergeAllCapNothingFinal` (stage 9) | red | "Nothing can merge. Soul Fruit is the best fruit in the game." |
| `MergeAllDone` (extended) | green | "Merged %d fruit into %d" + " (+%s shards)" + ". %d %s pairs wait for a rebirth" (the last part is left out at stage 9) |
| `SpawnCapText` | red | "Spawn Rarity is at the %s limit. Rebirth to raise it." |
| `SpawnCapTextFinal` (stage 9) | red | "Spawn Rarity is maxed for good." |
| `RebirthNeedIndex` | red | "Find every unlocked fruit first (%d/%d)" |
| `RebirthNeedCash` | red | "You need $%s cash to rebirth" |
| `RebirthNeedShards` | red | "You need %s shards to rebirth" |
| `RebirthAllUnlocked` | red | "Every rarity is already unlocked" |
| `RebirthMerging` | red | "Wait for your merge to finish" |
| `RebirthDone` | green | "Rebirth %d complete. %s fruit unlocked, cash x%s." |
| `RarityMigratedNotice` | green | "Rarities now unlock with rebirths. You are on Rebirth %d: %s fruit unlocked." When Spawn Rarity was refunded, " Spawn Rarity refund: $%s." is added (rule 22). |

`RebirthDone` today formats the multiplier with `%d` (`ReplicatedStorage.GardenConfig.lua:1037`), which cannot print 1.25. The new one formats it like the HUD does: two decimals, trailing zeros trimmed (`StarterPlayer.StarterPlayerScripts.CashClient.lua:106-116`).

### U11. Shard popup over a merged fruit
- **Where:** world space, above the fruit a hand merge just made in tier S.
- **Look:** the EarnPopups billboard (`StarterPlayer.StarterPlayerScripts.EarnPopups.lua:30-54`): Highway font, `Config.ShardPopupColor` (196, 140, 255), text "+7". Same rise and life as the cash popups (`ReplicatedStorage.GardenConfig.lua:1099-1105`).
- **Overlap:** starts 1.5 studs higher than the "+$" popup so the two never share a line. Merge All shows no popups; its shard total goes in its green toast.

## World and assets

- **Models:** none. This system is rules and screens.
- **Rebirth burst (proposal):** copy `Stars-01` from the `Anime` effect pack in the staging area ([[Existing VFX and Props]]) into ServerStorage as `RebirthBurst`. On rebirth, clone it to the plot centre, Emit once, and remove it after 2 s, the same pattern as the Clear puff (`ServerScriptService.GardenService.Merging.lua:222-248`).
- **Cap moment:** the existing first-discovery sparkle already plays on the cap fruit (`ServerScriptService.GardenService.FruitFactory.lua:148-195`). Nothing new.
- **Sound (proposal):** on rebirth, play the existing merge sound (`Config.MergeSoundId`, `ReplicatedStorage.GardenConfig.lua:741`) on the plot at PlaybackSpeed 0.8. No new upload.
- **Creator Store packs:** none needed ([[Creator Store Packs]]).
- **To build in Studio:** the rebuilt `RebirthMenu` and the new `MaxFruitPopup` inside `StarterGui.GardenUI`, from the images above.

## Asset requests for Samarth

Images only. **No animations to publish. No Robux products.**

How to make each one (from [[Consistent Image-Based UI Kits in Roblox]], section E): attach the named kit images as references in this order, one asset per generation, transparent PNG (ChatGPT) or flat green background plus a 2-3 px white outline (Gemini, then key out the green). Remove halos, alpha-bleed about 4 px, keep it at or under 1024 x 1024, then import in Creator Hub and send me the id.

| # | Asset | Config key | Shown at | Export canvas |
|---|---|---|---|---|
| A1 | "REBIRTH" title image | `Config.RebirthTitleImage` | about 260 x 74 | 520 x 148 |
| A2 | "MAX FRUIT" title image | `Config.MaxFruitTitleImage` | about 330 x 74 | 660 x 148 |
| A3 | Padlock icon | `Config.PadlockIcon` | 40 to 110 px | 256 x 256 |
| A4 | Index chip | `Config.IndexChip` | 47 x 50 | 188 x 200 |
| A5 | Storage chip | `Config.StorageChip` | 47 x 50 | 188 x 200 |
| A6 | Green tick | `Config.TickIcon` | 22 px | 128 x 128 |
| A7 | LOCKED upgrade plate | `Config.UpgradeLockedButton` | 289 x 83 | 578 x 166 |

**A1 "REBIRTH" title.** Easiest and most exact: in Photopea, open the INDEX title image (IndexMenu.TitleText, rbxassetid://103221050515090), copy its layer style onto the word REBIRTH in a heavy italic font, export. If you use ChatGPT instead:
> Image 1 is the exact style to copy: the word INDEX from my Roblox game's panel title (rbxassetid 103221050515090). Make the single word REBIRTH in exactly that lettering: the same heavy italic slant, the same white-to-light-grey vertical gradient, the same thick dark outline relative to letter height, the same small bevel, the same letter spacing. Transparent background. The word centred with 6% padding. No drop shadow outside the outline, no other shapes, no other words. Canvas 520 x 148.

Check the spelling before uploading.

**A2 "MAX FRUIT" title.** Same as A1 with the words MAX FRUIT on one line, canvas 660 x 148, reference again IndexMenu.TitleText (103221050515090).

**A3 Padlock icon.**
> Image 1 is my Roblox game's Lock button (TopButtons.LockButton, rbxassetid 118392145629817): copy its golden padlock's shape and colours. Image 2 is my Index button (rbxassetid 118941334639047): copy its thick dark outline, glossy highlight at the upper left and saturated cartoon shading. Draw ONE closed padlock icon only, front view, centred: gold body with a darker keyhole, steel-grey shackle, thick dark outline, glossy highlight top left. No text, no background, no shadow. Canvas 256 x 256 with 12% padding.

**A4 Index chip.**
> Images 1 to 3 are the three small reward chips from my game's Index rewards box (rbxassetid 89282702522748, 82056408182031, 78947574768646). Copy their exact square frame: border colour, corner rounding, inner shading and the 47 x 50 proportions. Image 4 is my Index button (rbxassetid 118941334639047): take ONLY its green book with the red apple on the cover. Make ONE new chip in that same frame with the green book centred, filling 70% of the inside. No text. Transparent outside the chip. Canvas 188 x 200.

**A5 Storage chip.** Same prompt as A4, but Image 4 is the Inventory button (rbxassetid 108477171933167) and the icon is ONLY its wicker basket of fruit.

**A6 Green tick.**
> One green check mark icon. Image 1 is my game's green Claim button (rbxassetid 132804991293392): use its exact green and glossy top highlight. Image 2 is my Index button (rbxassetid 118941334639047): use its thick dark outline. Transparent background, centred, 10% padding, no text. Canvas 128 x 128.

**A7 LOCKED upgrade plate.**
> Image 1 is the grey MAXED plate from my upgrade cards (rbxassetid 76326716689640). Make the same plate: same shape, 289 x 83 proportions, same bevel, outline and grey. Replace its word with a small gold padlock on the left (style of Image 2, my padlock icon A3) and the word LOCKED in white heavy rounded letters with a thick black outline, matching the lettering on Image 1. Transparent outside the plate. Canvas 578 x 166.

If the AI misspells LOCKED, make the plate blank (padlock only) and I will add "LOCKED" in FredokaOne in code.

**Also needed, but requested by another note:** the bat chip for the REWARDS box comes from [[Bats]] (its asset request 5). The REBIRTH and MAX FRUIT titles (A1, A2) are made in the same Figma batch as every other INDEX-style title ([[King UI Overhaul]] section 7.3, which lists all of them). See the merged asset list in [[Integration Review]].

## Code plan

Follows the GardenService pattern: each server module is `return function(G)`, required in order, and shares state only through `G` (`ServerScriptService.GardenService.lua:88-98`). Design only: nothing here has been edited.

### Load order
1. New remote in `ServerScriptService.GardenService.lua` after line 83, using the existing `remote()` helper (`:49-57`): `local rarityRemote = remote("RarityGate")`, put into `G.rarityRemote` with the other shared values before the first module loads.
2. New module `ServerScriptService.GardenService.RarityGate` (ModuleScript under the GardenService Script). Required right after `require(script.Upgrades)(G)` (`ServerScriptService.GardenService.lua:413`) and before Carrying (`:487`). It takes `Config`, `FruitIndex`, `Store`, `balances`, `discover`, `setShards`, `indexFolder`, `levelOf`, `upgradeAttr`, `notifyRemote`, `noticeRemote`, `rarityRemote` from G. Merging, RebirthLock and Persistence load later, so they can copy its functions at require time.

### Shared rules in `ReplicatedStorage.GardenConfig.lua`
GardenConfig already requires FruitIndex (`ReplicatedStorage.GardenConfig.lua:14`), and server and client both read it, so the stage maths lives here and cannot drift between them.

| Change | Where |
|---|---|
| `Config.RebirthCashBonus = 0.25` (was 1.00) and its comment | `:995-1000` |
| Keep `RebirthNeedsAll = true`; it now means "all unlocked fruit" | `:993` |
| Retire `RebirthLocked`, `RebirthBoxCash`, `RebirthBoxShards`, `RebirthAmountY`, `RebirthShardsText`, `RebirthNotReady` | `:1017-1036` |
| Replace `RebirthDone` and add every string in U10 | `:1036-1039` |
| New block "rarity gate" after `:1039`: `MaxRebirths = 8`, `MergeOnlyTop = 2`, `StoredPerRebirth = 1`, `RebirthCashSeconds = {79.92, 241.8, 2170, 2996, 4375, 5224, 7091, 11300}`, `RebirthShards = {50, 250, 1500, 3500, 7500, 20000, 30000, 100000}`, `ShardDropBase = 3`, `ShardDropGrowth = 2.2`, `RarityGateVersion = 1`, `RarityOrder` (the 9 tier names in order, used by [[Plot Overhaul and Environment]] and [[Bats]]), `IndexSealedTint`, `ShardPopupColor`, `CapToastGap = 1.5`, the 7 image keys from the asset table as `"rbxassetid://..."` strings (so the loading screen preloads them, `ReplicatedFirst.LoadingScreen.lua:328-345`) | new |
| New functions beside the accessors: `StageFor(r)`, `CapRankFor(r)`, `FirstRankOfTier(t)`, `SpawnCeilingFor(r)`, `HeadStartLevelFor(r)`, `NicePrice(x)`, `ShardPrice(units, r)` and `ShardReward(units, r)` (the one shared shard price and reward, D-017; [[Potions]], [[Explosives and Fruit Health]], [[Weapons and Artillery]], [[Bats]] and [[Royal Combat]] all call these and keep no copy of the D-010 table), `StoredCapFor(r)`, `RebirthCashNeed(stage)` (requires FruitEconomy inside the function, so there is no load cycle), `RebirthShardNeed(stage)`, `ShardDropFor(stage)`, `UnlockedCount(stage)`, and `RebirthStatus(r, indexFolder, cash, shards)` returning `{stage, final, indexHave, indexNeed, cashNeed, shardNeed, indexOk, cashOk, shardsOk, ok, reason}` | near `:1901-1933` |
| `Config.UpgradePrice(key, level, rebirths)`: optional third argument. For `spawnTier` it returns nil when `level >= SpawnCeilingFor(rebirths)`. nil already means "maxed" everywhere | `:1929-1933` |
| `Config.UpgradeLevelText(key, level, rebirths)`: for `spawnTier` the shown maximum is the ceiling + 1 | `:1996-1998` |
| `Config.StoredCap = 5` stays as the base; `StoredCapFor` adds `StoredPerRebirth x min(r, 8)` | `:533` |

`Config.MaxLevel("spawnTier")` stays 63 on purpose. Save loading and `publishUpgrades` clamp with it (`ServerScriptService.GardenStore.lua:482`, `ServerScriptService.GardenService.Upgrades.lua:294-298`), as [[Config and Fruit Data]] warns.

### New module: `ServerScriptService.GardenService.RarityGate.lua`

| Function (put into G) | What it does |
|---|---|
| `rarityAllowsMerge(player, promoted)` | true if `promoted.rank >= Config.CapRankFor(R)` |
| `rarityCapHit(player, fruitName)` | red toast (rate-limited by `CapToastGap`), then `rarityRemote:FireClient(player, {kind = "attempt", fruit = fruitName})` |
| `rarityOnMerge(player, promoted, firstSighting, at)` | if `promoted.tier == stage`: `setShards(+ShardDropFor(stage))` and fire `{kind = "shards", amount = n, at = at}`. If `promoted.rank == capRank` and `firstSighting`: fire `{kind = "reached", fruit = promoted.name}` |
| `rarityOnMergeAll(player, produced, firstSeen)` | same rules over the cascade's `produced` counts; returns the shards paid, for the toast |
| `rebirthStatus(player)` | `Config.RebirthStatus(R, indexFolder(player), balances[player], Shards)`; the server copy uses the server balance |
| `migrateRarityGate(profile)` | the pure, yield-free migration of rule 22; returns a report `{before, after, refund, changed}` or nil |
| per-player `capToastAt` table | cleared on leave beside the other per-player tables (`ServerScriptService.GardenService.lua:716-733`) |

### Changes to existing server scripts

| File:line | Change |
|---|---|
| `ServerScriptService.GardenService.Merging.lua:179-180` | After `MergeInto`, before `releaseHold` at `:183`: if `promoted` is nil (Soul Fruit), send `MergeBestText` and return. If `not rarityAllowsMerge(player, promoted)`, call `rarityCapHit(player, carried.Name)` and return. The carried fruit stays in hand. |
| `Merging.lua:205-213` | After `recordMerge` at `:211`, call `rarityOnMerge(player, promoted, firstSighting, at)`. |
| `Merging.lua:289-346` | `cascade(counts, capRank)`: at `:314-316` promote only if `promoted.rank >= capRank`. After the walk, also return `capPairs = floor(working[capName] / 2)`. Keep the positional insert at `:326` (gotcha 12 in [[Carrying, Merging, Storage, Rebirth, Tools and Saving]]). |
| `Merging.lua:483-487` | Pass `Config.CapRankFor(R)`. When `remove` is empty and `capPairs > 0`, send `MergeAllCapNothing` instead of `MergeAllNothing` (`MergeAllCapNothingFinal` at stage 9). |
| `Merging.lua:522-533` | After discovery, call `rarityOnMergeAll(player, produced, firstSeen)`. |
| `Merging.lua:574` | Extend the `MergeAllDone` text with shards paid and cap pairs left. |
| `ServerScriptService.GardenService.Upgrades.lua:309-311` | `Config.UpgradePrice(key, level, R)`. On nil for `spawnTier` below level 63, send `SpawnCapText` instead of returning silently (`SpawnCapTextFinal` at stage 9). |
| `Upgrades.lua:372-379` | `spawnPlan`: `level = min(levelForPlot(plot, "spawnTier"), SpawnCeilingFor(R))`; with Luck, return `min(level + tierBonus, SpawnCeilingFor(R))` (the same change [[Potions]] lists; build it once). The tilt branch stays as a fallback that the gate never reaches. |
| `ServerScriptService.GardenService.Progression.lua:190-199` | `discoverThroughTier` clamps `top` to `SpawnCeilingFor(R)` as well as `MaxLevel`. |
| `Progression.lua:311-320` | `indexComplete(player, maxTier)` counts only fruit with tier at most `maxTier`; returns `ok, have, need`. |
| `Progression.lua:324-326` | `rebirthMultiplier` uses `min(R, Config.MaxRebirths)`. |
| `ServerScriptService.GardenService.Carrying.lua:185-193` | The "spawnable" check uses `min(levelOf(player, "spawnTier"), SpawnCeilingFor(R))`. |
| `ServerScriptService.GardenService.Storage.lua:43-45` | `Config.StoredCapFor(R)` in place of `Config.StoredCap`. |
| `ServerScriptService.GardenService.RebirthLock.lua:15-26` | Also take `G.rebirthStatus`, `G.pendingFor`, `G.stowTool`, `G.upgradeAttr`, `G.indexComplete`. |
| `RebirthLock.lua:50-55` | Replace the gate with `rebirthStatus(player)`. On failure send `status.reason`. Add the `pendingFor(plot) > 0` refusal (rule 14). |
| `RebirthLock.lua:59-61` | Delete the payout (D-009). |
| `RebirthLock.lua:71-73` | If `held[player].tool`, call `stowTool(player)`; else `releaseHold(player, true)`. |
| `RebirthLock.lua:90-92` | Delete the Index wipe (D-009). |
| `RebirthLock.lua:95-97` | `publishUpgrades(player, { plotSize = levelOf(player, "plotSize") })` (Plot Size is kept, D-020) and remove `applyPlotSize(plot, 0)`. Then set `upgradeAttr("spawnTier")` to `Config.HeadStartLevelFor(count)`. Compute `count` before this point (today at `:117`). |
| `RebirthLock.lua:113-115` | Before `publishRebirths`: `local refund = G.refundExplosives and G.refundExplosives(player) or 0` ([[Explosives and Fruit Health]] rule 4, D-020). Then `setShards(player, shards + refund - status.shardNeed)`. |
| `RebirthLock.lua:127-128` | Keep `seedPlot` after the head start, still inside the yield-free block. Clone and emit `RebirthBurst` after the block. |
| `RebirthLock.lua:144-145` | New `RebirthDone` text with the trimmed multiplier. |
| `ServerScriptService.GardenService.Persistence.lua:196` | First lines of `applyProfile`: `local report = G.migrateRarityGate(profile)`. It is pure table work, so the yield-free contract holds (`Persistence.lua:35-37`). If `report.changed` or `report.after >= 1`, send `RarityMigratedNotice` at `Config.RefundNoticeDelay` + 4 s, after the refund notice and in the same way (`Persistence.lua:243-253`), with the refund amount from `report.refund` when it is above 0 (rule 22). |
| `Persistence.lua:47-118` | `captureProfile` stamps `profile.gateVersion = Config.RarityGateVersion`. Every live session has already passed through `applyProfile`, and `commit` refuses a profile that never loaded (`ServerScriptService.GardenStore.lua:564-566`). |
| `ServerScriptService.GardenStore.lua:220-283` | `Store.Default()` gains `gateVersion = 0`. |
| `GardenStore.lua:451-456` area | `sanitise` copies `gateVersion` when it is a whole number from 0 to `Config.RarityGateVersion`; anything else is a repair. Required: a field missing here is dropped on the next load (`GardenStore.lua:447-450`). |
| `GardenStore.lua:1154-1209` area | Self-test cases: `gateVersion` round-trips; the migration report for a synthetic Samarth profile (see test T12). |
| `ServerScriptService.GardenService.lua:631-684` | Two Studio-only dev doors with the existing `devSet` helper: `DevSetRebirths` (clamp 0 to 8, `publishRebirths`, `publishMultiplier`, clamp Spawn Rarity to the new ceiling) and `DevFillIndex` (discover every unlocked fruit). Both are gated by `RunService:IsStudio()` like the others. |
| `ServerScriptService.GardenService.Crown.lua:312-313` | Comment only: "A rebirth clears the Index" is no longer true. The logic at `:316` stays. |
| `ReplicatedStorage.CrownConfig.lua:38` | `C.MinDiscovered = 5` (was 10), so a 0-rebirth player who has found all 5 Commons can hold or challenge for the crown. This note owns the change ([[King UI Overhaul]] section 4.3 only prints `RoyalWhy`). Decision D-026. |
| `RebirthLock.lua:36-39` | After the busy check, the arena and royal-busy refusal of rule 14 (owned by [[Royal Combat]] rule 54; build it once). |

### Client changes

| File:line | Change |
|---|---|
| `StarterPlayer.StarterPlayerScripts.GardenClient.lua:300-310` | Cap-aware prompt (rule 5). Reset `ObjectText` to "" on the other branches. |
| `GardenClient.lua:314-318` | Do not add a match highlight when the carried fruit is at the cap. This also saves Highlights from the 24 budget (`ReplicatedStorage.GardenConfig.lua:1889`). |
| `GardenClient.lua:397-411` | Badge uses `Config.StoredCapFor(R)`; also refresh on `Rebirths` changes. |
| `StarterPlayer.StarterPlayerScripts.StoredFruitsMenu.lua:465-471` | Same, and call `refreshCount` on `Rebirths` changes (today it only runs inside `rebuild`). |
| `StarterPlayer.StarterPlayerScripts.IndexMenu.lua:236-241` | `stateOf` returns `"sealed"` (rule 20). |
| `IndexMenu.lua:95-188`, `:294-299`, `:301-361` | Sealed look (U1): tint, padlock, "Rebirth N", hide the viewport and the LockMask. |
| `IndexMenu.lua:243-273` | Sealed info box (U2). |
| `IndexMenu.lua` (new wiring near `:433-442`) | Listen to `Rebirths`: repaint every cell, the selected info box and the Found label (U3). |
| `StarterPlayer.StarterPlayerScripts.AlertBadges.lua:56-65` | Pass R into `Config.UpgradePrice`. |
| `AlertBadges.lua:87-95` | `rebirthWaiting` = `Config.RebirthStatus(...).ok and not final`. |
| `AlertBadges.lua:105-108` | Also listen to `Shards` and `Rebirths`. |
| `StarterPlayer.StarterPlayerScripts.UpgradeMenu.lua:53-85` | Cap-aware card (U7); refresh on `Rebirths`. |
| `StarterPlayer.StarterPlayerScripts.RebirthMenu.lua:1-256` | Rewritten for U4. Keeps the open/close pattern, the confirm Blocker (`:174-188`) and stroke scaling (`:214-225`). Reads only `Config.RebirthStatus`, so the menu, the badge and the server share one rule. Exposes a BindableEvent `RebirthMenu.Open` for the popup. Refreshes on `Cash`, `Shards`, `Rebirths` and Index folder changes. |
| New `StarterPlayer.StarterPlayerScripts.MaxFruitPopup.lua` | U5. Listens to `RarityGate` kinds `"reached"` and `"attempt"`. Keeps `shownThisSession[stage]`. Validates every payload field type before use. |
| `StarterPlayer.StarterPlayerScripts.EarnPopups.lua:30-54`, `:87-108` | Add a listener for `RarityGate` kind `"shards"` that floats "+N" at `at` (U11). |

### Remotes

| Name | Direction | Payload | Validation |
|---|---|---|---|
| `RarityGate` (new) | server to client | `{kind = "reached" or "attempt" or "shards", fruit = string?, amount = number?, at = Vector3?}` | One-way. The server never connects `OnServerEvent`, so a client firing it does nothing. The client checks `kind` and every field type before use. |
| `Rebirth` (existing) | client to server | none | Server: profile loaded, owns a plot, not already rebirthing (`RebirthLock.lua:33-48`), no merge mid-animation, `rebirthStatus(player).ok`, R below 8. Nothing is read from the client. |
| `Upgrade` (existing) | client to server | upgrade key | Cap-aware price; nil gets `SpawnCapText`. |
| `MergeAll` (existing) | client to server | none | The cascade takes the server's cap rank. |
| Fruit prompt (existing, not a remote) | client to server | the fruit | `merge` checks the cap before consuming anything. |

### Save fields

| Field | Type | Default | Notes |
|---|---|---|---|
| `gateVersion` (new) | whole number | 0 | 0 = not migrated. Set to 1 by the migration and stamped by every capture. |
| `rebirths` (existing) | whole number | 0 | The migration may raise it, never lower it. Meaningful from 0 to 8. |
| `upgrades.spawnTier` (existing) | whole number | 0 | The migration may lower it to the ceiling, with a cash refund. Rebirth sets it to the head start. |
| `index`, `shards`, `cash`, `stored`, `plot` (existing) | unchanged | | The Index is no longer wiped on rebirth. |

**How old saves migrate.** No schema bump is needed: adding a field is forward and backward compatible (`ServerScriptService.GardenStore.lua:8-12`), and bumping `SCHEMA` would kick players whose document is newer than their server (`GardenStore.lua:764-772`). The migration runs in `applyProfile` on the first load by a server that has this code. If an older server is still running during the update and saves the profile without `gateVersion`, the next new server simply migrates again; the rule is idempotent (rule 23). Recommended anyway: publish with "Shut down all servers" so no old server is left.

### Attributes
- Player `Rebirths` (existing, `ServerScriptService.GardenService.Progression.lua:95-101`): the one source the client derives stage, cap, ceilings and storage from.
- Player `Cash`, `Shards` (existing): read by the client copy of `RebirthStatus`.
- Player `DevSetRebirths`, `DevFillIndex` (new, Studio only, one-shot like `DevSetCash`).
- No new fruit or plot attributes.

## Edge cases and abuse

- **Firing Rebirth at will.** Every term is re-checked on the server (rules 12 to 14). The client menu, the badge and the server all call `Config.RebirthStatus`, so an honest client never offers a press the server refuses. The client's cash comes from the `Cash` attribute, refreshed every 5 s income tick (`ServerScriptService.GardenService.Ambient.lua:450-470`); if it is a few seconds stale, the server's toast names the shortfall.
- **Forcing a merge past the cap.** `merge` checks the cap on the server before consuming the carried fruit. A modified client prompt changes nothing.
- **Merge All past the cap.** The cascade takes the server's cap rank. It cannot promote a cap pair.
- **Luck past the cap.** `spawnPlan` bounds Luck by `SpawnCeilingFor(R)`, two ranks below the cap fruit, and the server refuses the drink at the ceiling ([[Potions]] rule 26).
- **Discovering fruit above the cap.** Every discovery path is server-side: pick-up of a spawnable or Lucky fruit, hand merge, Merge All, and Spawn Rarity backfill. All of them are bounded by the ceilings above. The Index rule only counts unlocked fruit.
- **Farming shards.** Shards come only from newest-rarity merges, which need two fruit the plot had to grow, so the rate is bounded by the spawn rate (at best one fruit per 2.5 s, `ReplicatedStorage.GardenConfig.lua:60-62`). This is the rate the simulation priced. The old free shards from repeating rebirths (the Index repay) are gone.
- **E spam at the cap.** One toast per 1.5 s; one popup per session per stage.
- **Rebirth with a tool in hand.** The tool goes back to the bag (fixes the live loss at `RebirthLock.lua:71-73`).
- **Rebirth during a merge animation.** Refused while the merge ledger holds the new fruit, about 0.26 s from the press to the white flash (`Config.MergeSpinTime`, `ReplicatedStorage.GardenConfig.lua:720`). Fixes the live race (rule 14).
- **Rebirth during Merge All.** Both run on the server and neither yields while it changes the plot, so one finishes before the other starts.
- **Leaving mid-rebirth.** The rebirth block is yield-free (`RebirthLock.lua:63-67`). The leave path saves after it.
- **Leaving with a cap fruit in hand.** Existing behaviour: it is dropped on the plot before the save (`ServerScriptService.GardenService.Persistence.lua:337-359`).
- **Rejoining after the cap moment.** The Index already has the cap fruit, so "reached" never fires again. An "attempt" shows the popup once in the new session.
- **Potions across a rebirth.** Effects survive a rebirth today (gotcha 16 in [[Carrying, Merging, Storage, Rebirth, Tools and Saving]]). An active Luck applies to the new stage's ceiling at once.
- **Claiming Index rewards after a rebirth.** Allowed. A claim pays 60 s of that fruit's own income (`ReplicatedStorage.GardenConfig.lua:394`), which is small against the next stage's prices.
- **Old-rule rebirthers.** A player with R = 2 from the old system keeps R = 2, but their multiplier drops from x3 to x1.50 (D-008). Their old Index was wiped by each old rebirth, so they may need to find lower-rarity fruit again before rebirth 3. Spawn Rarity backfill on load refills much of it (`Persistence.lua:288-290`).
- **Old saves with more than 8 rebirths.** Kept as saved; every rule uses `min(R, 8)`.
- **No stealing.** Nothing in this system touches another player's plot, fruit, pets or currency.
- **Mobile.** The cap prompt works with a tap like every fruit prompt. The Max Fruit card has a 340 x 190 minimum size. The Rebirth menu uses the Index shell, which is already used on phones; its 6-column preview cells are not buttons, so their small size on a phone costs nothing. The REBIRTH pill keeps its art proportions (381:134, `ReplicatedStorage.GardenConfig.lua:1016`).

## Performance

Limits are from [[Roblox Platform Constraints for the Expansion]].
- **Parts:** no permanent parts. The rebirth burst is one anchored host part for 2 s per rebirth.
- **Server work per frame:** none. The cap check is two number comparisons per merge. The cascade adds one comparison per rung.
- **Client work:** `GardenClient.refresh` already walks every fruit 5 times a second (`StarterPlayer.StarterPlayerScripts.GardenClient.lua:269-320`); it gains one rank comparison per fruit. The Index repaints 64 cells once per rebirth. Fewer Highlights are drawn at the cap.
- **Network:** at most one `RarityGate` event per hand merge, and every hand merge needs a pick-up press and a merge press, under 100 bytes each. That is far below the about 500 events per second per client limit and the 50 KB/s per-client download budget.
- **Memory:** the Rebirth menu adds at most 11 ViewportFrames, the popup 1. The Index already builds 64 (`IndexMenu.lua:624-637`).
- **Saving:** one extra number per profile. The self-test keeps the worst payload under 8,192 bytes (`ServerScriptService.GardenStore.lua:1209`).

## Test plan

Before every playtest run `workspace:SetAttribute("StudioNoSave", true)` in the command bar, or the playtest writes the real save (`ServerScriptService.GardenStore.lua:572-575`). Dev doors (Studio only) are set on your Player, for example `game.Players.<you>:SetAttribute("DevSetRebirths", 0)`.

| # | Rule | Steps | Pass when |
|---|---|---|---|
| T1 | 2, 3, 5, 7 | DevSetRebirths 0, DevSetCash 1e4. Buy Spawn Rarity to the ceiling. Merge up to Apple. Then carry an Apple to another Apple and press E. | Popup opens once on the first Apple. The prompt reads "Max Fruit / Rebirth to merge". No green glow on the Apple pair. E gives the red toast; the Apple stays in hand. |
| T2 | 8, 9 | At stage 1, watch the dev panel SPAWN row and 100 spawns. Press Buy on Spawn Rarity at the ceiling. | No Tomato or Apple spawns. The buy press gives `SpawnCapText`; cash unchanged. Card shows U7. |
| T3 | 10 | DevSetShards 7500. With Spawn Rarity one level below the ceiling, drink Luck and watch 5 min. Then buy Spawn Rarity to the ceiling and drink again. | First: Carrots appear with the gold glint and enter the Index on pick-up; no Tomato or Apple ever spawns. Second: refused with "Spawn Rarity is maxed...", the potion stays in the bag. |
| T4 | 6 | Plot with 4 Apples and 2 Carrots: press Merge All. Then a plot with only Apple pairs: press again. | First press: Carrots become a Tomato, Apples stay, toast says "2 Apple pairs wait for a rebirth". Second press: `MergeAllCapNothing`, `MergeAllReadyAt` unchanged. |
| T5 | 12, 19 | Stage 1 with the Index at 4/5. Open Rebirth. Then DevFillIndex; then DevSetCash 56500; then DevSetShards 50. | U4 shows the 8 Uncommon cards. Each need turns green as it is met. Button and HUD badge appear only after the third. Each earlier press gives the matching toast. |
| T6 | 11, 15, 16, 17 | Buy Plot Size level 2, then rebirth from stage 1. Count IntValues in `IndexState` before and after. | R = 1, cash 0, shards down 50, Index count unchanged, storage badge "0/6", `CashMultiplier` = 1.25 x Index bonus, Spawn Rarity level 0, Plot Size still level 2 and the pen not rebuilt. |
| T7 | 11 | DevSetRebirths 1, then meet stage 2 needs and rebirth. | Spawn Rarity level is 5 (Dandelion); 3 starter fruit are Dandelion or lower. |
| T8 | 18 | Stage 2: merge two Daffodils; merge two Strawberries. Then Merge All a plot with several Uncommon pairs. | +7 shards and a purple "+7" for the Uncommon merge; +0 for the Common one; Merge All toast states the shards paid. |
| T9 | 17 | DevSetRebirths 3. Store fruit until refused. | The 9th store is refused with "8/8". |
| T10 | 20, 21 | Stage 1: open the Index and scroll. Select a Sealed card. | Uncommon to Transcendent cards show the padlock and "Rebirth 1" to "Rebirth 8". Info box reads "Unlocks at Rebirth N"; Claim grey. Title "Found n/5". |
| T11 | 22, 23 | StudioNoSave off, API access off (in-memory store). Dev panel Seed, then Reload. Then Reload again. | After the first Reload: Rebirths 5, notice toast, fruit restored (overflow in storage). After the second: nothing changes. |
| T12 | 22 | GardenStore self-test case: migrate `{rebirths = 0, best = "Zenflare", cash = 6.46e26, upgrades = {spawnTier = 63}}`. | rebirths 8, spawnTier 61, cash 6.46e26 + 5.14Oc + 13.4Oc, gateVersion 1. |
| T13 | 15 | Hold a Speed Potion, then rebirth. Repeat holding a TNT. | Potion: the bag count is unchanged and the carry row is gone. TNT: it is stowed, then refunded at the pre-rebirth price with the `ExplosivesRefunded` notice. |
| T14 | 14 | Start a hand merge and fire Rebirth from the command bar within 0.2 s. | Refused with `RebirthMerging`; no extra fruit on the new plot. |
| T15 | 13 | DevSetRebirths 8. Open Rebirth; fire the remote. | "Every rarity is unlocked"; the server refuses. |
| T16 | 24 | Device Emulator: a phone, a tablet and a 1600 x 900 desktop, at Actual Resolution. Open the Index, the Rebirth menu, the popup. | No element overlaps another; no text below its readable size; no emoji anywhere. |

## Dependencies and conflicts

- **Rarity Gating and Rebirth:** this note.
- **[[Bats]]:** the bat tier is R and its colour is the newest rarity's colour (the UI Kit says the tier colours become the bat colours). Bats owns the knockback numbers and the bat chip art shown in the REWARDS box. Rebirth keeps the bat (D-009). No conflict.
- **[[Explosives and Fruit Health]]:** TNT never destroys fruit: at 0 HP a fruit is stunned for 20 s and earns nothing (its rule 29). It cannot block a rebirth, because the Index rule counts discoveries, not fruit owned. Explosives cost shards through `ShardPrice`, which compete with `shardNeed(S)`. Rebirth refunds explosives at the pre-rebirth price (rule 15, D-020).
- **[[Weapons and Artillery]]:** same as explosives. No direct hook.
- **[[Underground and Ores]]:** the 100K-shard underground competes with the stage 8 shard need (100K). The [[Integration Check]] simulation puts Floor 1 at about 8.5 h (stage 8), later than the Stage Model's "stage 6 to 7". The ore Index category does not count toward the rebirth Index rule (open question 4), and "Found n/m" counts fruit only. Ore income is a share of current fruit income (7.5 / 9 / 10.5% by floor, fruit only, D-018), not pinned to `full(S)`, because a fixed share would pay a whole stage's money right after a rebirth. The underground is kept on rebirth.
- **[[Pets]]:** "rarer animals ... will match the rarity of the max fruit that you own". The best fruit a player can own is now bounded by `CapRankFor(R)`, so pet requests stay inside the unlocked rarities. Pet income follows the run-best fruit, capped at `CapRankFor(R)` (Pets P25 to P27, D-018), not `full(S)`: at the plateau that is the same 2.86% of `full(S)` per unit, and early in a stage it stays a climb. Pets are kept on rebirth.
- **[[Boats and Swimming]]:** the 1,000-shard boat competes with early shard needs (stage 2 needs 250, a median player holds 1.5K at that gate). The boat is kept on rebirth.
- **[[Potions]]:** this note owns how Luck behaves at the cap (rule 10). Other potions are unaffected. Running potions survive a rebirth.
- **[[Plot Overhaul and Environment]]:** a rebirth no longer shrinks the plot (Plot Size is kept, its rule E1, D-020), so the pen is not rebuilt on rebirth. Only the gate banners recolour. The rebirth burst plays at the soil centre.
- **[[King UI Overhaul]]:** resolved here. Holding or challenging for the crown needed 10 Index fruit or 1 rebirth (`ServerScriptService.GardenService.Crown.lua:314-323`, `ReplicatedStorage.CrownConfig.lua:38`), and only 5 fruit exist at 0 rebirths. This note sets `C.MinDiscovered = 5` (code plan, D-026). Royal shard rewards become `ShardReward(units, R)` ([[Royal Combat]], D-017), so they stay a few minutes of merging at every stage.
- **[[Royal Combat]]:** blade skins cost `ShardPrice` shards and compete with rebirth shard needs. Rebirth is refused while fighting (rule 14). Blade skins and the `royal` table are kept on rebirth.

**Decisions.** This note implements D-006 to D-011 as written. D-009 means removing today's Index wipe and shard repay. D-008 lowers the multiplier for anyone who rebirthed under the old rules (open question 7). D-006 changes "start from the beginning" for rebirths 2 and later (open question 6). The [[Integration Review]] added D-017 (one shared shard price and reward), D-019 (Luck refused at the ceiling), D-020 (rebirth keeps Plot Size and refunds explosives) and D-026 (`MinDiscovered = 5`, owned here); this note follows them.

## Open questions for Samarth

Each has a default, so building can start now.

1. **Migration.** Default: Placement (rule 22). Every existing player's rebirth count rises to cover the fruit they own, and they keep everything. The other choice is a full reset for everyone, which deletes real progress for a rule change.
2. **Luck at the cap.** Resolved in the [[Integration Review]] (C6) in favour of [[Potions]] rule 26: Luck adds one level up to the spawn ceiling and is refused at the ceiling, so D-007 stays strict (D-019). The alternative (Luck grows the second-best fruit) is listed in the review's open questions.
3. **After Transcendent.** Default: no rebirth past 8. A later prestige layer would need a shrinking reward formula ([[Rebirth Economies in Merge and Incremental Games]], proposal 12).
4. **Ores and pets in the Index.** Default: their Index categories do not count toward the rebirth rule. Only fruit count.
5. **Crown rule.** Default: set `C.MinDiscovered` to 5 and keep "1 rebirth is worthy". This note owns the change (code plan); the [[King UI Overhaul]] only prints the requirement.
6. **Head start (D-006).** You wrote "they start from the beginning". Default: rebirth 1 starts from Blueberry, rebirth 2+ start from the first fruit of the rarity below the new one, to avoid about 80 extra minutes of replay per stage.
7. **Old-rule rebirthers lose multiplier.** Default: accept it (x2 per rebirth becomes +25% per rebirth). Lifetime visits were 119 on 4 Oct 2026, so very few players are affected.
8. **Your own save.** Default: migrate it like everyone's (you land on Rebirth 8). To play the new progression, use a Studio playtest with StudioNoSave and `DevSetRebirths 0`, or an alt account.
9. **The Max Fruit popup.** Default: modal, once per stage, plus once per session after a cap attempt. The other choice is a non-blocking card under the top pills.
10. **The purple "+N" shard popup.** Default: on, for hand merges only. Merge All reports shards in its toast.
11. **Index rule.** Default: "discovered" (found, claimed or not), the same bar the game uses today.
12. **Locked plate (A7).** Default: until the art arrives, the grey-tinted Maxed plate stands in.
