---
tags: [system, design]
date: 2026-10-04
status: proposed
depends_on: [Rarity Gating and Rebirth, Boats and Swimming, Plot Overhaul and Environment, King UI Overhaul]
---
# Pets

Forest animals you tame by feeding them fruit, merge into bigger pets, and keep on your plot, where they walk around and earn money. Every rule and number below is a **proposal** unless it cites code (`File.lua:LINE`, in `~/fruit-empire/src-export-2026-10-04`) or a research note. Nothing has been built or edited.

## Why

Samarth's words, from [[Brain Dump 2026-10-04]]:

> in the forest we will add pets, merging two of the same pets makes the pet bigger and makes you more money, you can equip the pets and they will walk around your base.

> now back the the animals. i know this is easy to mess up but this is where you need to really research, dont create slop. make detailed features for this.
> https://create.roblox.com/store/asset/97050466582003/StudThemed-Animal-Pack-LITE
> we will start with these animals. to tame an animal you will have the feed it the requested fruit based on how rare the animal is it will be a different fruit. rarer animals that payout more money will match the rarity of the max fruit that you own.

> the ores merge and the animals merge should both be reasonable in the amount of moeny they give meaning they should match what stage of the game a player is in, this is because especcially since this is a merge game earnings scale exponentially, this means that everything needs to correspond with the stage of the game a player is in. you can use rebirths to control and organize this. the stages are of the game is the rarity of fruit the player is on starting from the common category all the way to transcendent.

> also all Uis you make should follow the UI that we created, if you need any UI stuff then we can create them using chatgpt or gemini image generation following references of current UI

> what i noticed with the king feature in the sky island is that it was a bunch of dopamaine slop. instead everything should be structured. like for players its going to be confusing. all the UI should never overlap and also everything should be super clean.

> I dont like the emoji-slop that you pushed on the sky island kings feature.

What the system is for. Pets give the player a second thing to build beside the plot: a small team of animals that earns money on top of the fruit. The forest across the sea becomes a place worth visiting, because that is where the animals are. Pet money is pinned to the player's best fruit, so a pet is always worth the same share of the plot at every stage, and a rebirth resets pet money in step with fruit money. Pets are calm decoration too: they wander the plot, graze and nap, and never get in the way of merging.

**Words used in this note**
- **Wild animal**: an animal standing in the forest that a player has not tamed. Each player has their own set; nobody else sees it.
- **Pet**: an animal a player owns. It has a **species** (Sheep, Deer, Turtle, Hyena, Giraffe, Basilisk), a **tier** (the 9 rarities, 1 Common to 9 Transcendent) and a **size** (Baby, Big, Huge, Giant).
- **Key**: the text name of one kind of pet, `Species|tier|size`, for example `Sheep|1|2` (a Big Common Sheep).
- **Feed / tame**: give a wild animal the fruit it asks for. When it has had enough feeds it is tamed and becomes a Baby pet.
- **Request**: the fruit a wild animal asks for.
- **Equip slot**: a space for a pet on the plot. Only equipped pets earn and walk around. The rest wait in the **bag**.
- **Merge (pets)**: two pets with the same key become one pet of the next size.
- **Stage S**: the newest rarity a player has unlocked, `S = rebirths + 1` ([[Rarity Gating and Rebirth]], [[Stage Model]]).
- **Best rank**: the best fruit this player has reached in this run (since the last rebirth), as a rank number. Rank 1 is Soul Fruit (best) and rank 64 is Blueberry (worst) (`ReplicatedStorage.Fruits.FruitIndex.lua:5`). **Best tier**: the tier of that fruit. **Best fruit income**: that fruit's $/s from `FruitEconomy.ForRank`.
- **Tier gap**: how many tiers a pet sits below the best tier. A pet at or above the best tier has gap 0.
- **Unit**: one "best fruit" worth of income. A pet worth 1.5 units earns as much as 1.5 of your best fruit.
- **Pet limit**: the most units all equipped pets can add together: 8.75, which is 25% of a full plot.
- **Full-plot $/s**: 35 cap fruit times the rebirth multiplier, the most a plot earns in a stage ([[Stage Model]]).
- **Plateau**: late in a stage, when the plot is nearly full of cap fruit and income stops rising.
- **Clearing**: one of the 18 open meadows in the forest ring, at radius 1,190 to 1,380 with an open radius of 38 to 72 studs, kept free of trees "for the animals later" (`ServerStorage.ForestBuilder.lua:131-145`; the count is in `~/fruit-empire/NOTES.md:1522`). Each has a marker part `Main Island.Forest.Clearings.ClearingNN` with a `Radius` attribute (`ServerStorage.ForestBuilder.lua:379-404`).
- **Bearing**: the compass angle of a point seen from the map centre, 0 to 360 degrees.
- **Rig / Motor6D**: the animal models are built from parts joined by Motor6D joints. Code can bend a joint by setting its `Transform`. **Procedural animation** means code computes those bends every frame, with no published animation (D-004 in [[Decisions Log]]).

## Player view

1. **First trip (stage 1).** A new player has merged up to Carrots. They cross the sea (swim or boat, [[Boats and Swimming]]). When they step onto the forest beach, a green notice appears once: "Wild animals trade for fruit. Bring what they ask for."
2. **A clearing.** In the nearest clearing a small Sheep grazes. Over its head floats a mini Index card: the Blueberry card frame, a Blueberry in the middle, "Blueberry" on top and "0/1" on the bottom. When they walk closer the Sheep lifts its head and turns to look at them.
3. **Feeding.** Within 10 studs a prompt reads "Feed Blueberry", with "Common Sheep" above it. They hold E for 0.6 s. A Blueberry leaves their storage bag. The Sheep lowers its head and chews, hops twice, sparkles and fades out. A green notice says "New pet: Common Sheep!"
4. **Home.** The new Baby Sheep is already on their plot, in its first free slot. It walks to a free spot between fruit, grazes, sits, looks around. On each 5 second income tick a small "+$5" floats over it, like the fruit popups (a Baby pays a quarter of a Carrot, about $1.1/s).
5. **The Pets button.** A new button sits under Shop in the left column. It opens the Pets panel, built like the Index. The top row shows 3 slots: one Sheep and two empty slots with a "+". Selecting the Sheep shows "Pays like 0.25 of a Carrot" and "$1/s now". When they make their first Apple, the same Sheep reads "Pays like 0.25 of an Apple" and "$5/s now", without doing anything.
6. **Merging.** Later they own two Baby Sheep. The Sheep card shows "x2" and the Merge button lights up. One press makes a Big Sheep. On the plot, the Sheep grows with a white flash and a sparkle. It now pays like 0.6 of their best fruit.
7. **Rarer animals.** After their first rebirth and their first Uncommon fruit, Uncommon animals appear. Some animals show a padlock card that says "Need UNCOMMON": a goal, not yet feedable. Later stages bring Turtles, Hyenas, Giraffes and, from Mythic, the rare Basilisk.
8. **Rebirth.** Pets are kept. Right after a rebirth their money drops, because it follows the best fruit, which is back near the start. It climbs again as the player merges. When the player's best fruit reaches a new tier, older pets count as one size smaller per tier behind, so it pays to tame the new tier.
9. **The Index.** The Index has category tabs on its left edge, Fruits, Pets and Ores (the same tab art as the Shop). The Pets page is a grid: one row per rarity, one column per species. Each first tame unlocks a card with a Claim button that pays cash and shards once.
10. **Later stages.** Animals of tier 4 to 6 need 2 feeds and tier 7 to 9 need 3, so a trip takes more fruit in storage. Storage also grows by 1 per rebirth ([[Rarity Gating and Rebirth]] rule 17). Extra slots can be bought with shards. When the pet total reaches the limit, the panel shows "(max)" next to the pet income.

## Rules

All rules are proposals. Numbers are worked out in the next section.

**A. Species and tiers**
1. **P1.** There are 6 species, all from the Stud-Themed Animal Pack LITE: Sheep, Deer, Turtle, Hyena, Giraffe, Basilisk ([[Creator Store Packs]]).
2. **P2.** Each species has a first tier: Sheep 1, Deer 1, Turtle 2, Hyena 3, Giraffe 4, Basilisk 6 (the pack's creator already labels the Basilisk Mythic). A species exists in every tier from its first tier up to 9. That gives 43 species-tier pairs.
3. **P3.** Species income bonus: Sheep 1.00, Deer 1.00, Turtle 1.15, Hyena 1.30, Giraffe 1.50, Basilisk 2.00. Species spawn weight: 28, 28, 20, 13, 7, 4, renormalised over the species that exist in the rolled tier.

**B. Wild animals**
4. **P4.** Every player whose profile has loaded has 6 wild animals, except during a respawn gap (P10). The server holds them as data. Only that player's client draws them.
5. **P5.** Homes: each animal's home is a point in one of the 2 clearings whose bearing is closest to the player's bearing, 3 animals per clearing. The point is uniform within (`Radius` minus 8) studs of the clearing marker and at least 12 studs from the player's other homes (10 tries, then the last try is kept).
6. **P6.** Arrival: when the 1 s server sweep sees the player's root go from radius below 990 to radius 1,003 or more (the forest dry beach, [[Boats and Swimming]] table 5.1), every animal with 0 feeds gets a new home by P5. Partly fed animals keep theirs.
7. **P7.** Tier roll, with best tier b and stage S: 55% tier b; 25% tier b - 1 (tier b if b = 1); 8% a uniform tier from 1 to b - 2 (tier b if b is 2 or less); 12% tier b + 1 if b + 1 is at most S, otherwise tier b. The species is rolled after the tier (P3).
8. **P8.** **Locked animal**: an animal whose tier is above b. It cannot be fed. Its card says "Need" and the tier name. When b reaches its tier it becomes feedable and gets its request then.
9. **P9.** Request: the fruit of rank `min(64, bestRank + k)`, with k = 3 when the animal's tier is b, k = 5 when it is b - 1, and k = 7 when it is b - 2 or lower. Feeds needed: 1 for tiers 1 to 3, 2 for tiers 4 to 6, 3 for tiers 7 to 9. Every feed asks for the same fruit. The request is set when the animal spawns or unlocks, and recomputed for every animal with 0 feeds whenever `bestRank` changes (not only when b changes: test T4 expects the request to move from Blueberry to Strawberry when the best fruit goes from Carrot to Apple, both tier 1). Partly fed animals keep their request. (Fixed by the player-path pass, [[New Player - First Session]].)
10. **P10.** A tamed animal is replaced after 30 s by a new roll, homed by P5 at the player's bearing at that moment and at least 25 studs from the player. A rebirth rerolls all 6 at once. Joining rolls a new 6. Wild animals are never saved.
11. **P11.** Wild animals walk within 8 studs of home. They never collide and nothing can hit, push or harm them. When the owner comes within 14 studs, an animal stops, turns to face them and plays Alert.

**C. Taming**
12. **P12.** Each feedable animal has a ProximityPrompt made by the client: ActionText "Feed" plus the fruit's display name, ObjectText the tier name plus species ("Common Sheep"), key E, HoldDuration 0.6 s, MaxActivationDistance 10, RequiresLineOfSight false. Locked animals have no prompt.
13. **P13.** The server accepts a feed only if all of these hold, checked in this order. On any refusal nothing is consumed:
    - (a) the profile has loaded (`Store.IsReady`);
    - (b) the animal id belongs to this player and is not locked;
    - (c) the player's root is within 20 studs (horizontal) and 20 studs (vertical) of the animal's home;
    - (d) on the previous 1 s sweep sample, the root was within 80 studs of that home (no teleporting in);
    - (e) at least 0.5 s since this player's last accepted feed;
    - (f) if this feed completes the tame, the player owns fewer than 60 pets;
    - (g) the player carries the requested fruit, or has at least 1 in storage.
    Refusals (c), (f) and (g) send one red toast. (b), (d) and (e) are silent.
14. **P14.** A feed uses the carried fruit if its name equals the request. Otherwise it takes one from `StoredFruits`. Exactly one fruit per feed.
15. **P15.** When the feeds reach the number needed, the animal is tamed. The player gains one Baby of that species and tier. The first tame of a species-tier pair creates its Pets Index entry with state 0. If a slot is free, the Baby goes into the lowest-numbered free slot.

**D. Sizes and merging**
16. **P16.** Sizes are 1 Baby, 2 Big, 3 Huge, 4 Giant.
17. **P17.** Two pets with the same species, tier and size merge into one pet of the next size. Giants do not merge.
18. **P18.** Merges are server actions: Merge (one pair of the chosen key) and Merge All (from size 1 upward, every pair, until none are left). Both are instant and free. If an input was equipped, the result takes the lowest-numbered slot that held an input. Any other slot that held an input becomes free.

**E. Equipping and the bag**
19. **P19.** Slots = `min(6, 3 + floor(rebirths / 2)) + bought`, with bought from 0 to 2. That is 3 at 0 to 1 rebirths, 4 at 2 to 3, 5 at 4 to 5 and 6 from 6 on, up to 8 with both bought slots.
20. **P20.** The first bought slot costs 10,000 shards and the second 150,000. Bought slots are kept through rebirth.
21. **P21.** Only equipped pets earn and appear on the plot. A key can fill as many slots as the player owns of it.
22. **P22.** The bag holds at most 60 pets, equipped ones included. At 60, a taming feed is refused (P13 f). Merging and releasing still work.
23. **P23.** Release removes one pet of a key, unequipped copies first. It pays nothing and needs the confirm dialog.
24. **P24.** Equip Best fills every slot with the pets worth the most units now. Ties go to the higher tier, then to the species order in P1.

**F. Income**
25. **P25.** `bestRank = max(CapRankFor(rebirths), min(bestRankMerged or 64, 64 - spawnTierLevel))`. `bestRankMerged` is the server's existing per-run best merge (`bestRank[player]`, set by `recordMerge`, `ServerScriptService.GardenService.Progression.lua:129-140`, cleared on rebirth, `ServerScriptService.GardenService.RebirthLock.lua:102`). `64 - spawnTierLevel` is the best fruit the plot can grow (`ReplicatedStorage.GardenConfig.lua:83-96`). The `max` with the merge cap means a pet can never be priced above the stage. b is the tier of `bestRank`.
26. **P26.** Effective size `e = size - max(0, b - tier)`. Units = `U(e) x speciesBonus`, with U(4) = 3.5, U(3) = 1.5, U(2) = 0.6, U(1) = 0.25, U(0) = 0.10, U(-1) = 0.04 and U of -2 or less = 0. In words: each tier behind your best tier makes a pet count as one size smaller.
27. **P27.** `petBase = min(sum of equipped units, 8.75) x FruitEconomy.ForRank(bestRank)`. It is added to the plot's base income before the cash multiplier, so the friend, Index and rebirth bonuses apply to pets exactly as to fruit (`ServerScriptService.GardenService.Progression.lua:332-336`).
28. **P28.** Offline pay adds `petBase` the same way. Friends stay excluded offline, as for fruit (`ServerScriptService.GardenService.Persistence.lua:179-193`). Pets are a permanent property of the account, the rule Config states for what pays offline (`ReplicatedStorage.GardenConfig.lua:756-759`).
29. **P29.** Pets have no health, no hunger and no timers.
30. **P30.** **No economic nap.** A rebirth keeps every pet, every bought slot and the Pets Index (D-009). It resets `bestRank` through the head start, so pet income falls and rises with the player's best fruit. Pets play a sleep pose only as an idle animation (section "World and assets").

**G. On the plot**
31. **P31.** Equipped pets are drawn for every player within 220 studs of the plot. Their walking schedule comes from the shared server clock, so all clients see each pet within about 1 stud of the same spot when they see the same fruit.
32. **P32.** A pet stays inside the soil rectangle inset by its radius + 1.5 studs. At rest it is never within (fruit radius + pet radius + 0.5) of a fruit. While walking, its straight path never passes within (fruit radius + pet radius) of a fruit. Every pet part has CanCollide, CanQuery and CanTouch off.
33. **P33.** An idle pet whose owner is within 10 studs turns to face the owner. It then plays Happy with a 25% chance, at most once per 20 s per pet.
34. **P34.** While an explosive warning circle covers a pet ([[Explosives and Fruit Health]] section 6.8), the pet walks out of the circle at 1.5 times its speed. This is cosmetic: nothing can hurt a pet.
35. **P35.** Pets never leave their owner's plot. They never follow a player, so they never enter the sea, the forest, the sky islands or the arena.

**H. Index**
36. **P36.** The Index gets a Pets category with 43 cards: one row per tier, one column per species.
37. **P37.** A card is Sealed when its tier is above S ("Rebirth N", the same look as [[Rarity Gating and Rebirth]] rule 20), "???" when not tamed yet, found (0) after the first tame, and claimed (1) after Claim.
38. **P38.** Claim pays once per account: cash = 60 s of the income of the cap fruit of that tier (`IndexMoneySeconds`, `ReplicatedStorage.GardenConfig.lua:394`) and shards = `IndexRewards.Shards` of that cap fruit's rank (`ReplicatedStorage.IndexRewards.lua:76-85`).
39. **P39.** Pets Index cards do not count toward the rebirth Index rule (the same default as [[Rarity Gating and Rebirth]] open question 4).

**I. Look**
40. **P40.** Every screen uses kit images ([[UI Kit Inventory]]), FredokaOne white text with a black stroke, no emoji and no separator characters (middle dot, em dash, multiplication sign, the rule in [[King UI Overhaul]] rule 10). No new element overlaps an existing one at any HUD scale.

## Numbers

### Formulas
- `S = rebirths + 1`. `capRank(t)` is the best rank in tier t: 60, 52, 41, 33, 26, 18, 13, 5, 1 ([[Rarity Gating and Rebirth]] table 1).
- `bestRank = max(capRank(S), min(bestRankMerged or 64, 64 - spawnTierLevel))`; `b = tier(bestRank)`.
- `e = size - max(0, b - tier)`; `units = U(e) x bonus(species)`.
- `petBase = min(sum(units), 8.75) x ForRank(bestRank)`; shown $/s = `petBase x cashMultiplier`.
- At the plateau `bestRank = capRank(S)`, so `ForRank(bestRank) x mult(S) = full(S) / 35`. **One unit is 1/35 = 2.86% of the full-plot $/s of the player's stage**, and the pet limit (8.75 units) is **25%** of it.
- `request rank = min(64, bestRank + k)`, k = 3, 5 or 7 (P9).
- `slots = min(6, 3 + floor(rebirths / 2)) + bought`.
- Index claim: cash `60 x ForRank(capRank(t))`, shards `IndexRewards.Shards(capRank(t))`.

### Table 1: species

Parts and joints are from [[Creator Store Packs]] (233 parts and 227 Motor6D in total; each animal has one more part than joints). Baby scale is this note's proposal (see "The 13-stud Giraffe" below).

| Species | First tier | Spawn weight | Income bonus | Parts (joints) | Pack size X x Y x Z (studs) | Baby scale | Baby height | Giant height (x1.8) |
|---|---|---|---|---|---|---|---|---|
| Sheep | 1 Common | 28 | 1.00 | 22 (21) | 2.5 x 3.1 x 4.4 | 1.00 | 3.1 | 5.6 |
| Deer | 1 Common | 28 | 1.00 | 33 (32) | 1.4 x 5.2 x 4.0 | 0.90 | 4.7 | 8.4 |
| Turtle | 2 Uncommon | 20 | 1.15 | 23 (22) | 2.5 x 1.8 x 4.0 | 1.00 | 1.8 | 3.2 |
| Hyena | 3 Rare | 13 | 1.30 | 37 (36) | 1.2 x 4.1 x 4.3 | 0.95 | 3.9 | 7.0 |
| Giraffe | 4 Epic | 7 | 1.50 | 32 (31) | 2.5 x 13.0 x 8.7 | 0.38 | 4.9 | 8.9 |
| Basilisk | 6 Mythic | 4 | 2.00 | 86 (85) | 2.4 x 1.4 x 5.4 (long) | 1.00 | 1.4 (5.4 long) | 2.5 (9.7 long) |

Species per tier (Index row length): tier 1: 2, tier 2: 3, tier 3: 4, tier 4 and 5: 5, tier 6 to 9: 6. Total 43.

### Table 2: units by size and tier gap

| Size | Units at gap 0 | % of full plot at the plateau | Gap 1 counts as | Gap 2 counts as | Gap 3 counts as |
|---|---|---|---|---|---|
| Baby | 0.25 | 0.71% | 0.10 | 0.04 | 0 |
| Big | 0.60 | 1.71% | 0.25 | 0.10 | 0.04 |
| Huge | 1.50 | 4.29% | 0.60 | 0.25 | 0.10 |
| Giant | 3.50 | 10.0% | 1.50 | 0.60 | 0.25 |
| Pet limit | 8.75 | 25.0% | | | |

Each step is more than 2x the one below (2.4, 2.5, 2.33), so a merge always raises income, the same rule the fruit ladder follows (`ReplicatedStorage.FruitEconomy.lua:11-13`). Species bonus multiplies these: a Giant Basilisk at gap 0 is 7.0 units, 20% of a full plot on its own.

### Table 3: worked values at the plateau, every stage

$/s includes the rebirth multiplier (D-008) and leaves out the Index and friend bonuses. Money uses the game's suffixes. "Baby / fed" is the Baby's income divided by the income of all the fruit fed to tame it.

| S | Cap fruit ($/s) | full(S) | Baby | Big | Huge | Giant | Pet limit | Same-tier request (fruit $/s) | Feeds | Baby / fed |
|---|---|---|---|---|---|---|---|---|---|---|
| **1** | Apple (20.2) | 707 | 5.05 | 12.1 | 30.3 | 70.7 | 177 | Strawberry (2.11) | 1 | 2.39 |
| 2 | Blue Lollipop (9.83K) | 430K | 3.07K | 7.37K | 18.4K | 43.0K | 108K | Crocus (940) | 1 | 2.61 |
| **3** | Pear (71.2M) | 3.74B | 26.7M | 64.1M | 160M | 374M | 934M | Violet Corn (6.05M) | 1 | 2.94 |
| 4 | Watermelon (59.4B) | 3.64T | 26.0B | 62.4B | 156B | 364B | 910B | Mini Pumpkin (4.64B) | 2 | 1.60 |
| 5 | Tropical Melon (25.5T) | 1.78Qa | 12.8T | 30.6T | 76.5T | 178T | 446T | Red Mango (1.86T) | 2 | 1.71 |
| **6** | Dragon Pepper (31.6Qa) | 2.49Qi | 17.8Qa | 42.7Qa | 107Qa | 249Qa | 622Qa | Giant Pinecone (2.13Qa) | 2 | 1.85 |
| 7 | Paradise Petal (3.03Qi) | 265Qi | 1.89Qi | 4.54Qi | 11.4Qi | 26.5Qi | 66.3Qi | Candy Blossom (194Qa) | 3 | 1.30 |
| 8 | Solar Bloom (5.26Sx) | 506Sx | 3.62Sx | 8.68Sx | 21.7Sx | 50.6Sx | 127Sx | Moon Melon (313Qi) | 3 | 1.40 |
| **9** | Soul Fruit (237Sx) | 24.9Sp | 178Sx | 427Sx | 1.07Sp | 2.49Sp | 6.22Sp | Alien Carrot (13.6Sx) | 3 | 1.45 |

At the plateau the same-tier request is always the fruit one rank below the spawn ceiling (D-007), so it grows on the plot by itself and never needs a long merge chain. Lower-tier animals ask for cheaper fruit: at stage 6 a Legendary animal (k = 5) wants Golden Apple and an older one (k = 7) wants Passionfruit.

**Worked example, stage 1.** A player with 3 slots, best fruit Apple: a Baby Sheep pays $5.05/s, a Giant Sheep $70.7/s (10% of the $707/s full plot). Three Giants would be 10.5 units, above the 8.75 limit, so they pay $177/s (25%). That needs 24 tames, more than a 15-minute stage allows; the simulation below shows about 5.

**Worked example, stage 9.** Six slots of Giants at gap 0 would be 21 units; the limit holds pets to 8.75 units, $6.22Sp/s, 25% of $24.9Sp/s.

### Table 4: early in a stage (the reason pets follow the best fruit)

Right after rebirth 2 the head start puts Spawn Rarity on Dandelion ([[Rarity Gating and Rebirth]] rule 11), so `bestRank` = 59 and b = 2.

| Moment | Best fruit ($/s) | A Giant Uncommon Sheep (gap 0) | Pet limit | If pets were pinned to full(3) instead |
|---|---|---|---|---|
| Stage 3, first second | Dandelion (43.2) | 3.5 x 43.2 x 1.5 = $227/s | $567/s | 3.5 x 71.2M x 1.5 = $374M/s |
| Stage 3, first Rare fruit | Bee Balm (21.7K) | b is now 3, so gap 1: counts as Huge, 1.5 x 21.7K x 1.5 = $48.8K/s | $285K/s | $374M/s |
| Stage 3, midway | Bell Pepper (1.19M) | gap 1, counts as Huge: $2.68M/s (a Giant Rare Sheep at gap 0 would pay $6.25M/s) | $15.6M/s | $374M/s |
| Stage 6, first second | Cacao (140B) | $1.10T/s | $2.76T/s | $249Qa/s |

The three seeded fruit at the start of stage 3 earn about $113/s on average (expected value of three spawns at Spawn Rarity level 5, times 1.5, from `economy.py`). Pets pinned to the best fruit give that start a push (Pecorella's "fast start of each run", [[Rebirth Economies in Merge and Incremental Games]] section 2), and the push shrinks to one size smaller as soon as the first Rare fruit appears. Pets pinned to full(S) would pay about 3.3 million times the plot in the first second, which would remove the stage.

### Table 5: equip slots and shard prices

| Rebirths | Stage | Base slots | With 1 bought | With 2 bought |
|---|---|---|---|---|
| 0 to 1 | 1 to 2 | 3 | 4 | 5 |
| 2 to 3 | 3 to 4 | 4 | 5 | 6 |
| 4 to 5 | 5 to 6 | 5 | 6 | 7 |
| 6 to 8 | 7 to 9 | 6 | 7 | 8 |

Starting slots follow the research: 99 Nights starts at 2 and Grow a Garden at 3 ([[Pets - Taming, Merging and Wandering]] section 6).

| Bought slot | Price | Against median shards held at a rebirth gate ([[Stage Model]]) |
|---|---|---|
| First | 10,000 shards | 78% at the stage 4 gate (12.8K), 35% at the stage 5 gate (28.3K) |
| Second | 150,000 shards | 34% at the stage 8 gate (439K), after the 100K underground |

Both are optional sinks for late shards, which the Stage Model says inflate (about 439K at the stage 8 gate).

### Table 6: Pets Index rewards

| Tier | Cards | Shards per card | Tier shards | Running total | Cash per card (60 s of the cap fruit) |
|---|---|---|---|---|---|
| 1 Common | 2 | 7 | 14 | 14 | 1.21K |
| 2 Uncommon | 3 | 16 | 48 | 62 | 590K |
| 3 Rare | 4 | 50 | 200 | 262 | 4.27B |
| 4 Epic | 5 | 100 | 500 | 762 | 3.56T |
| 5 Legendary | 5 | 200 | 1,000 | 1,762 | 1.53Qa |
| 6 Mythic | 6 | 475 | 2,850 | 4,612 | 1.90Qi |
| 7 Astral | 6 | 775 | 4,650 | 9,262 | 182Qi |
| 8 Eternal | 6 | 1,675 | 10,050 | 19,312 | 316Sx |
| 9 Transcendent | 6 | 2,500 | 15,000 | 34,312 | 14.2Sp |

Through stage 8 the Pets Index adds about 19.3K shards, about 4% of the 439K a median player holds at the stage 8 gate. The fruit Index pays 26,528 in total ([[Config and Fruit Data]]). Cash per card is one minute of a single cap fruit, small against each stage's prices, like the fruit Index's own claims.

### Table 7: wild tier odds

| Situation | Tier b | b - 1 | Lower | b + 1 (locked) |
|---|---|---|---|---|
| Early in a stage (b = S - 1, b of 3 or more) | 55% | 25% | 8% | 12% |
| Late in a stage (b = S, b of 3 or more) | 67% | 25% | 8% | - |
| Stage 2 early (b = 1) | 88% | - | - | 12% |
| Stage 3 early (b = 2) | 63% | 25% | - | 12% |

Gating is by the player's own progress, never luck alone, the pattern the research found in every hit ([[Pets - Taming, Merging and Wandering]] section 5).

### Table 8: what pets do to stage length (simulation)

Method: the stage model (`~/fruit-empire/sim/economy.py`, imported read-only, D-011 cash and shard needs, D-010 shard drops, 3 s merges, 7 careers per row, median) with a pet layer added: the rules in this note, species rolled by weight, the player tames only tier-b animals, merges every pair, equips the best pets, and pays each request with fruit taken off the plot. A trip blocks merging for its travel time. Travel times come from [[Boats and Swimming]] table 5.3 (boat crossing about 14 s, plain swim about 65 s each way) plus walking from the plot and between two clearings. The script lives in this session's scratchpad (`petsim/final_pets.py`); it is about 150 lines and can be rebuilt from this paragraph.

| Player (assumed trip habit) | Total to stage 9 | Minutes per stage, S1 to S8 | Pets as % of full plot at each gate | Tames per stage |
|---|---|---|---|---|
| No pets | 12.95 h | 15, 34, 70, 88, 108, 126, 148, 189 | 0 | 0 |
| Casual: first trip at 6 min, then every 30 min, 4 min per trip | 11.79 h | 17, 38, 62, 78, 99, 112, 130, 170 | 4, 7, 9, 10, 12, 18, 16, 22 | 5, 8, 9, 12, 13, 16, 13, 21 |
| Regular: first at 4 min, every 15 min, 3 min per trip | 11.30 h | 17, 37, 58, 74, 98, 105, 123, 163 | 4, 16, 18, 15, 21, 25, 25, 25 | 5, 15, 21, 17, 23, 27, 22, 38 |
| Heavy: first at 3 min, every 8 min, 2.5 min per trip | 11.31 h | 18, 38, 57, 73, 93, 108, 122, 170 | 8, 25, 25, 25, 25, 25, 25, 25 | 9, 25, 36, 30, 41, 58, 40, 74 |
| Regular, trips twice as long | 12.33 h | 19, 41, 66, 80, 103, 119, 130, 180 | 5, 14, 19, 18, 22, 25, 25, 25 | 5, 15, 21, 20, 22, 32, 23, 42 |

What it shows:
- A regular pet user saves about 1.65 h of the 2.1 h the Stage Model credits to pets and ores together (12.95 h to 10.84 h). Ores, at 7.5 / 9 / 10.5% by floor (D-018), cover part of the rest from stage 8, when the [[Integration Check]] re-run says Floor 1 is bought.
- The 25% limit is what keeps heavy players in line: they hit it from stage 2 and gain nothing more from extra trips.
- **Stages 1 and 2 get 2 to 4 minutes slower** for anyone who goes to the forest, because a trip costs more time than its pets pay back inside such a short stage. The pets carry into stage 3, where every profile is faster. Open question 7.
- No stage drops below 80% of its no-pet length, so the D-011 rebirth needs stay valid.
- Travel time matters a lot: with trips twice as long, the saving falls from 1.65 h to 0.6 h. Boats and Aqua Affinity ([[Potions]]) are what make pets worth it.

## UI

Every piece uses [[UI Kit Inventory]] images or the two new images in section 8. Text is FredokaOne, white, with a black stroke, at the `UIKit` size steps (Title 30, Heading 24, Body 20, Small 17) from [[King UI Overhaul]] section 9; until `UIKit` ships, the same values are set by hand. Casing follows the King UI rule: titles upper case, buttons Title Case, sentences in sentence case and at most 48 characters. Sizes are at the HUD reference of 1600 x 900 (`ReplicatedStorage.GardenConfig.lua:1871`). Panels are Scale-sized with an aspect lock; HUD buttons are Offset-sized and scaled by HudScale ([[Consistent Image-Based UI Kits in Roblox]] section A6).

### U1. Pets HUD button (`GardenUI.PetsButton`)
- **Art:** new image A2 (section 8), 96 x 92, label "Pets" baked in like Inventory, Index and Shop. Until A2 has an id (0), a fallback: the kit green action button (ClaimButton, 132804991293392) at 96 x 40 with "Pets" text at Body size. The same "0 means fallback" pattern as D-001.
- **Where:** the left column, a fourth rung under Shop. X offset 16 like the column; Y = Scale 0.5, Offset +126 (the column's rungs are -181, -78 and +24, about 102 px apart, [[UI Kit Inventory]] layout facts).
- **States:** normal; hover 1.08 and press 0.92 from ButtonAnimator (it adopts every GuiButton, `StarterPlayer.StarterPlayerScripts.ButtonAnimator.lua:207-217`); the magnetic lean (add to `MAGNETIC`, `ButtonAnimator.lua:52-55`). The kit AlertBadge "!" (79539007098845, 31 x 35) at its top right when a slot is free and an unequipped pet exists, or when any key has 2 or more copies below Giant.
- **Overlap:** add `"PetsButton"` to `Config.HudClusters` (`ReplicatedStorage.GardenConfig.lua:1863-1867`) so HudScale sizes and places it. At 1600 x 900 its bottom edge is at 450 + 126 + 46 = 622 px, about 36 px above the top of CornerHud (my estimate from the screenshot in [[UI Kit Inventory]]). On a phone at 812 x 375 the HUD factor clamps to 0.55 (`GardenConfig.lua:1949-1957`) and the bottom edge lands at about 281 px against CornerHud's top at about 278 px (my estimate from the authored sizes). So HudScale gets a guard: after each `apply()`, if `PetsButton`'s bottom is within 8 px of `CornerHud`'s top, the four left buttons move up together by the difference. HudScale stays the only writer of positions (`StarterPlayer.StarterPlayerScripts.HudScale.lua:116-127`). The button is also added to the arena-focus hide list of [[King UI Overhaul]] rule 16.

### U2. Pets panel (`GardenUI.PetsMenu`)
- **Shell:** the Index panel pieces, cloned from `UIKitPrefabs.Panel` when it exists ([[King UI Overhaul]] section 9.1), or copied from `IndexMenu` in Studio: TitleFrame (110138832714236), TitleBlue (95532041481396), Body (82827190351667), Inline (110403403749726), ScrollBox (83403190365135), InfoBox (101725900357985), RewardsBox (126819257706583), red square X (107111219044453). Same Size, Position, AnchorPoint and aspect lock as `IndexMenu`, so it fits wherever the Index fits. The name ends in "Menu", so the loading screen pre-draws it (`ReplicatedFirst.LoadingScreen.lua:411-426`). Modal band, DisplayOrder 20 ([[Consistent Image-Based UI Kits in Roblox]] section B).
- **Title:** image A1 "PETS" where the INDEX title image sits. Until A1 exists, "PETS" at Title size on the blue bar (the inventory allows this).
- **Title bar, left:** two green pills (ClaimButton art, 150 x 34 at a 940-wide body): "Equip Best" and "Merge All". A pill with nothing to do is tinted `Config.ShopTabDim` (`ReplicatedStorage.GardenConfig.lua:1216`) and does nothing when pressed.
- **Title bar, right, left of the X:** the bag count "23/60" at Body size.
- **ScrollBox content** (`PetList`, a ScrollingFrame using the Index's pixel-height rules, `StarterPlayer.StarterPlayerScripts.IndexMenu.lua:526-578`, 5 columns, the rarity card aspect 268:338):
  - A header row at Heading size: "Equipped 3/4" on the left and "Pets $1.2M/s" on the right. When the limit applies it reads "Pets $3.4M/s (max)".
  - The slot cards, one per slot up to 8. A filled slot is a card in the pet's tier frame (`Config.RarityFrames`, `GardenConfig.lua:563`): pet viewport, top strip "Big Sheep", bottom strip "$12/s". An empty slot is the Common frame at ImageTransparency 0.65 with the kit "+" image (CornerHud.InvitePlus, 118611403979434) centred. A locked slot is the Common frame at 0.8 with "Rebirth 2" (next base slot) or "10K Shards" (a buyable slot; pressing it opens U3).
  - A header row "Your pets".
  - One card per key that has unequipped copies, sorted by units, then tier, then species. Top right of the card: "x3" when there are several.
  - Empty bag: one centred line, "Feed wild animals in the forest to tame them."
- **Selection:** the kit card selection outline (113285622864672) on the selected card, like the Index.
- **InfoBox:** the selected pet's viewport (3/4 view, yaw 35 degrees, pitch -12, framed to the model's bounding box, so a Giraffe fits), then:
  - Heading: "Big Common Sheep".
  - Body: "Pays like 0.6 of an Apple", or "Pays like 1.5 Pears" from 1 unit up (uses `Config.Plural`, `GardenConfig.lua:1978-1994`).
  - Body: "$12/s now".
  - Small, only with a tier gap: "1 tier behind: counts as Baby".
  - Small: "Merge 2 to make Huge" (hidden for a Giant).
- **RewardsBox, used for actions:** header "Actions" at Heading size, then three buttons stacked: Equip or Unequip (ClaimButton art), Merge (ClaimButton art, dimmed with `ShopTabDim` when fewer than 2 copies or Giant), Release (the kit No button, 105863640938118, at 0.6 scale).
- **Open and close:** an `isOpen` flag, UIScale from 0.85 to 1 in 0.22 s (Back out) and back in 0.14 s, like every panel ([[Client UI - HUD and Menus]] section 3.5). Opening it hides the five other panels, and each of them adds "PetsMenu" to its own list (Code plan).
- **Overlap:** a centred modal like the Index. Only one modal is ever open.

### U3. Release and buy-slot confirm
- The existing `ClearConfirm` art: panel 112127226387884, Yes 71592791133553, No 105863640938118, with its title line overwritten, the way RebirthMenu reuses it (`StarterPlayer.StarterPlayerScripts.RebirthMenu.lua:190-193`). A Blocker one ZIndex below eats clicks (`StarterPlayer.StarterPlayerScripts.ClearMenu.lua:84-98`).
- Release: title "Release?", body "Release 1 Big Common Sheep? You get nothing back."
- Buy slot: title "Buy Slot?", body "Buy a pet slot for 10,000 shards?" Yes fires the request; the price is re-checked on the server.

### U4. Request card over a wild animal
- **What:** a BillboardGui on the animal's root part, built by the client. It is one mini Index card, so it reads like the rest of the game.
- **Size:** 2.4 x 3.03 studs (the card aspect 268:338, `ReplicatedStorage.GardenConfig.lua:575`). `StudsOffsetWorldSpace` = (0, model height + 1.9, 0). `MaxDistance` 45. `LightInfluence` 0. `AlwaysOnTop` false, so trees hide it.
- **Feedable:** the frame of the requested fruit's tier (`Config.RarityFrames`); top strip the fruit's display name at Small size; centre a ViewportFrame with that fruit's model at a fixed 25 degree turn (calm, no spinning); bottom strip the feed count "0/2".
- **Locked:** the frame of the animal's own tier with `ImageColor3` = `Config.IndexSealedTint`, the padlock image A3 from [[Rarity Gating and Rebirth]] (or "???", `Config.IndexLockedText`, `GardenConfig.lua:514`, until A3 exists), bottom strip "Need EPIC".
- **Calm rules:** only the 3 nearest own animals within 45 studs show a card. Cards fade in and out over 0.2 s. Homes are at least 12 studs apart (P5), so two cards never overlap at normal camera distance.
- **Prompt:** Roblox's default ProximityPrompt look, the same look every fruit prompt uses today (`StarterPlayer.StarterPlayerScripts.GardenClient.lua:300-310`).

### U5. Index: Pets category
- **Tabs:** the Index category tabs that [[Underground and Ores]] U4 builds: kit Shop tab art 110917091182913 tucked behind the Index panel's left edge, three tabs, Fruits, Pets and Ores, with the icon art in Underground request A2 (which now includes a Pets icon). The selected tab is plain; the others are tinted `ShopTabDim`, the way the Shop dims its tabs (`StarterPlayer.StarterPlayerScripts.ShopMenu.lua:89-102`). One tab column for every Index category, instead of two pills in this note and tabs in the other (decision D-026, [[Integration Review]] conflict C17). The left third of the title bar belongs to "Found n/m" ([[Rarity Gating and Rebirth]] U3), which on the Pets page counts unlocked pet cards.
- **Grid:** a second ScrollingFrame `PetList` beside `List`, in the same place, so `fitCanvas` keeps counting only fruit cells (`IndexMenu.lua:571-578`). 6 columns (Sheep, Deer, Turtle, Hyena, Giraffe, Basilisk), 9 rows (tiers). A species that does not exist in a tier gets an empty transparent cell, so the columns stay lined up.
- **Cell:** a clone of `IndexMenu.Templates.Cell`: tier frame, the animal's viewport in its tier look, top strip the species, bottom strip the tier name. States by P37, with the Sealed look shared with fruit cards.
- **InfoBox:** viewport, "Rare Hyena", "Income bonus 1.30", "Found in forest clearings".
- **RewardsBox:** the money chip (82056408182031) and shards chip (78947574768646) with their amounts, and Claim (132804991293392). No bonus chip: pet cards pay no permanent bonus.
- **Badge:** the Index button's "!" also lights for an unclaimed pet card (`StarterPlayer.StarterPlayerScripts.AlertBadges.lua:71-78`).

### U6. Money popups over pets
- The owner sees one "+$" popup per equipped pet per income tick, using EarnPopups' existing billboard (`StarterPlayer.StarterPlayerScripts.EarnPopups.lua:30-54`). Highway font is the game's rule for money over objects in the world (`ReplicatedStorage.GardenConfig.lua:20-25`). It starts at the pet's height + 1 stud.

### U7. Toasts (strings in `PetConfig.Strings`, sent through `Notify` and `Notice`)

| Key | Channel | Text |
|---|---|---|
| `Tamed` | green | "New pet: %s %s!" (was "Tamed a %s %s!", which prints "a Epic" and "a Uncommon"; player-path pass, [[Mid Game - Stage 4 Epic]]) |
| `NeedFruit` | red | "You need a %s. Store one, then come back." |
| `TooFar` | red | "Get closer to feed it." |
| `BagFull` | red | "Your pet bag is full (60/60)." |
| `SlotsFull` | red | "All slots are full. Unequip a pet first." |
| `NotEnoughShards` | red | "You need %s shards." |
| `SlotBought` | green | "Pet slot bought. You have %d slots." |
| `ForestHint` | green, once per account | "Wild animals trade for fruit. Bring what they ask for." |

Every string is ASCII, with no emoji and no separator characters.

## World and assets

### Rigs (from the Stud-Themed Animal Pack LITE)
The six models sit in `ServerStorage.AssetQuarantine.Q_97050466582003` ([[Creator Store Packs]]). They have rigs and an AnimationController each, but no animations and no scripts.
- **Move** each into `ReplicatedStorage.PetModels` as `Sheep`, `Deer`, `Turtle`, `Hyena`, `Giraffe`, `Basilisk`.
- **Root:** PrimaryPart is the body (the Turtle keeps `Shell`, its existing PrimaryPart). The root is Anchored; every other part is unanchored and Massless, so the Motor6D joints still move them while the root is moved by code. CanCollide, CanQuery, CanTouch and CastShadow off on every part.
- **No Humanoid**, as Roblox's performance page advises for NPCs ([[Roblox Platform Constraints for the Expansion]] section 4).
- **Roles:** joint names are not recorded in the vault, so step 1 of the build is a dump of every Motor6D (Name, Part0, Part1, C0 position) per model. Each joint then gets a `PetRole` attribute: `HipFL`, `HipFR`, `HipBL`, `HipBR`, `KneeFL` (and so on), `HoofFL` (and so on), `Neck1`..`NeckN`, `Head`, `Jaw`, `EarL`, `EarR`, `Tail1`..`TailN`, `Wool`, `Crest1`..`CrestN`, `Seg0`..`Seg13`, `Spike`. Rules for the first pass: a leg joint's Part1 hangs below the body centre; side from the sign of C0.X; front or back from C0.Z compared with the neck joint; a joint whose Part0 is a leg part is the knee, the next one the hoof. Each role also gets `PetAxis` ("X" for a pitch swing by default) and `PetSign` (1 or -1), checked by eye with the dev pose test (Test plan T20).
- **Hip height:** an attribute `HipHeight` on each model: root centre to ground, at Baby scale.

### Tier look
- **Collar:** a new 1-part studded band (the `Studs` MaterialVariant) welded at the neck, coloured `Config.RarityColors[tierName]` (`ReplicatedStorage.GardenConfig.lua:582`). It is the one thing that always shows the tier, on every species.
- **Coat tint:** for tier 2 and up, each part tagged `Coat` takes `Color:Lerp(tierColour, 0.35)`. Common animals keep their natural colours.
- **Glow, tiers 7 to 9:** a PointLight on the root (Range 10 x size scale, Brightness 1.2, tier colour), on only within 120 studs of the camera, and a sparkle emitter (Rate 2, LightEmission 1, Size 0.3, tier colour). Giants of tiers 7 to 9 also carry `Auras.RNG-Aura-02`, the planned use in [[Existing VFX and Props]]. This mirrors "higher tier ores should glow" from the brain dump.

### Sizes, and the 13-stud Giraffe
- `Model:ScaleTo(babyScale x sizeScale)` on the client clone, with size scale 1.00, 1.25, 1.50, 1.80 for Baby, Big, Huge, Giant. `ScaleTo` scales the joints too ([[Pets - Taming, Merging and Wandering]] source 37).
- The pack's Giraffe is 13 studs tall and 8.7 long. Unscaled, a Giant would be 23 studs tall on a plot that can be as small as 29.5 x 47.1 studs (level 0, `ReplicatedStorage.GardenConfig.lua:117-123`; rebirth now keeps Plot Size, D-020, but a new player starts at level 0). Five fixes:
  1. **Baby scale 0.38:** a Baby Giraffe is 4.9 studs tall, a Giant 8.9 (Table 1), a little taller than a Giant Deer (8.4).
  2. **First tier Epic:** the Giraffe only appears from stage 4, when the player has had three stages to learn the plot.
  3. **Occlusion fade:** on the owner's client, any own pet taller than 6 studs whose bounding box is within 6 studs of the owner's root fades to `LocalTransparencyModifier` 0.6 over 0.2 s, and back when they step away. A Giraffe never hides the fruit the player is about to pick up.
  4. **Viewports frame by bounding box** (camera distance from the model's largest side at a 30 degree field of view), so a Giraffe fits its card like a Sheep.
  5. **Walking room:** the Giraffe's footprint at Giant size is 8.7 x 0.38 x 1.8 = 6.0 studs long, a radius of about 3 studs, so the P32 inset (radius + 1.5) keeps it 4.5 studs inside the soil edge.

### Procedural motion (D-004)

**Gait engine (client, every frame, per animated pet).**
- Phase `phi` advances by `v / stride x dt`, where v is the pet's ground speed this frame. When v drops to 0, phi eases to the nearest neutral point over 0.2 s and the idle state takes over.
- For each leg L with phase offset `o(L)`: `theta = 2 pi (phi + o(L))`.
  - Hip swing (pitch): `A_hip x sin(theta)`.
  - Knee bend: `A_knee x max(0, cos(theta))`, so the knee bends while the leg swings forward. Front knees bend backward (sign -1), hind knees forward (+1).
  - Hoof: `-0.6 x knee`, keeping the hoof roughly flat.
- Body (root offset): bob `y = H x (0.5 - 0.5 cos(4 pi phi))` (two bobs per cycle), roll `A_roll x sin(2 pi phi)`, pitch `A_pitch x sin(4 pi phi + pi/2)`.
- Neck nod `A_nod x sin(4 pi phi - pi/2)`, against the bob, which keeps the head steady.
- Tail yaw `A_tail x sin(2 pi f_tail t)` on its own clock.
- Ear flick (if ear joints exist): every 4 to 8 s (random), ear yaw +20 degrees and back in 0.25 s.
- Every joint eases toward its target with `alpha = 1 - e^(-dt / 0.12)`, so state changes never snap.
- Size: speed x sqrt(size scale) and stride x size scale, so bigger animals step more slowly and cover more ground. Amplitudes do not change.

**Leg phase offsets (fraction of a cycle)**

| Gait | Front left | Front right | Back left | Back right | Used by |
|---|---|---|---|---|---|
| 4-beat walk | 0.25 | 0.75 | 0.00 | 0.50 | Sheep, Deer |
| Trot (diagonal pairs) | 0.0 | 0.5 | 0.5 | 0.0 | Hyena |
| Crawl (trot timing, yaw swing) | 0.0 | 0.5 | 0.5 | 0.0 | Turtle |
| Pace (same-side pairs) | 0.0 | 0.5 | 0.0 | 0.5 | Giraffe (real giraffes pace at a walk) |

**Per-species walk (Baby size)**

| Species | Speed (studs/s) | Stride (studs) | Step rate | Hip swing | Knee bend | Bob | Roll | Pitch | Neck | Tail | Extra |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Sheep | 4.0 | 1.6 | 2.5 Hz | +-22 deg | none | 0.08 | +-2 deg | +-1.5 deg | +-5 deg | +-15 deg yaw at 2.0 Hz | `Wool` parts pitch +-2 deg at 2x step rate, 0.1 cycle behind (fluff bounce) |
| Deer | 5.5 | 2.4 | 2.3 Hz | +-20 deg | 0 to 40 deg | 0.10 | +-1.5 deg | +-2 deg | +-4 deg | still while walking | ears flick |
| Turtle | 2.0 | 0.8 | 2.5 Hz | +-28 deg yaw, +10 deg lift in swing | none | 0.04 | +-3 deg | 0 | neck yaw +-6 deg at 0.4 Hz | none | shell is the root |
| Hyena | 6.0 | 2.0 | 3.0 Hz | +-26 deg | 0 to 35 deg | 0.10 | +-1 deg | +-3 deg (rocking) | +-4 deg around a -8 deg base (head low) | +-18 deg yaw at 2x step rate | `Crest` parts pitch +-4 deg, a wave 0.08 cycle per part |
| Giraffe | 4.0 | 3.2 | 1.25 Hz | +-18 deg | 0 to 25 deg | 0.12 | +-4 deg (pace sway) | +-1 deg | each `Neck` joint +-5 deg at 2x step rate, 0.1 cycle behind the one below; head -0.8x the neck sum (stays level) | swish +-20 deg at 1.0 Hz, always | |
| Basilisk | 4.0 | wave 2.8 | 1.4 Hz | legs, if the dump finds any: +-25 deg yaw, leg on the outer side of the bend forward | none | 0 | 0 | 0 | head yaw = -0.5 x Seg1 (looks ahead) | last 3 segments 1.3x amplitude | see the serpent wave below |

**Basilisk serpent wave.** For segment joints `Seg0` to `Seg13` (i = 0 to 13): yaw `= 16 deg x sin(2 pi (phi - i / 7))`. That is two full waves along the body travelling backward from the head, at `f = v / 2.8` (1.4 Hz at 4 studs/s). Odd segments add pitch `+-2 deg` with the same phase, a small vertical ripple. `Spike` parts, if found, pitch +-3 deg in the same wave. The root moves on a straight line between targets; the travelling wave makes it read as slithering.

**Idle states (picked by weight when a walk ends; durations are random within the range)**

| State | Weight by day | Duration | Motion |
|---|---|---|---|
| Stand | 25 | 3 to 6 s | breathing: root y +-0.03 studs at 0.25 Hz, neck +-1.5 deg |
| Look around | 20 | 3 to 5 s | neck and head yaw +-35 deg at 0.3 Hz (Deer +-40 deg) |
| Eat (graze) | 25 | 2.5 s | neck to -40 deg in 0.35 s, chew: head +-4 deg at 3 Hz for 1.8 s, back up in 0.35 s |
| Sit | 15 | 5 to 8 s | hind hips +70 deg, root down 45% of hip height, body pitch -14 deg, head +5 deg; 0.4 s blend |
| Sleep | 15, x3 at night | 6 to 10 s | front hips -80 deg, hind hips +80 deg, root down 80% of hip height, neck -30 deg, head roll 8 deg, breathing root +-0.04 at 0.2 Hz |

Night is when `Config.ClockTimeAt` is outside 6 to 18 (`ReplicatedStorage.GardenConfig.lua:1482-1498`).

**Species overrides**
- **Turtle.** Sit and Sleep become Retract: the head and leg joints slide 0.25 studs (x size scale) inward along their axis using `Transform` position, over 0.5 s. Eat dips the head only -25 deg.
- **Giraffe.** Eat is the drinking pose: neck -55 deg, front legs roll outward +-12 deg. Sleep folds the legs, keeps the neck up at -10 deg, head nods +-2 deg at 0.15 Hz. Look around raises the neck +10 deg first (browsing).
- **Basilisk.** Sit is Rest: a still S-curve, Seg i yaw `22 deg x sin(2 pi i / 7)`, breathing +-2 deg at 0.3 Hz. Sleep is the same S at 32 deg with the head pitched -8 deg. Eat is a head dip of -20 deg with two lunges of Seg0 forward 0.25 studs. Look around raises the head +10 deg and turns it +-30 deg.
- **Sheep and Hyena.** Eat is a graze. The Hyena's Look around is Sniff: head -20 deg with quick nods +-3 deg at 5 Hz for 2 s.

**Event states**

| State | Length | Motion |
|---|---|---|
| Happy | 1.2 s | two hops, root y = 0.6 x size scale x sin(pi t / 0.28) each, 0.08 s apart; tail +-30 deg at 6 Hz. Hyena crest +15 deg; Deer ears +20 deg; Turtle head bob +-10 deg three times and a 30 deg turn; Basilisk wiggle at 24 deg and 3 Hz with the head +12 deg; Giraffe neck yaw +-15 deg for 2 cycles and a 0.3 stud hop |
| Alert (wild only) | 1.5 s | head +12 deg, ears +20 deg, turn to face the player over 0.5 s |
| Turn in place | until facing | yaw 180 deg/s, legs step at half hip swing as if v = 1.5 |
| Merge pop | 0.4 s | scale from old to new size over 0.35 s (Back out), a white Highlight flash, `Shiny-01` Emit(12) |
| Arrive or leave | 0.4 s | scale 0.6 to 1 (or 1 to 0.6) with a transparency fade, `Shiny-01` Emit(8) |
| Feed (wild) | 1.8 s | Eat, then (if tamed) Happy, `Shiny-01` Emit(12) and a 0.4 s fade out |

### Walking on the plot (client)
- **Schedule:** every equipped slot has a seed `hash(plotName, slot)`. Time is cut into 14 s legs: leg k runs from `k x 14 + slotOffset` seconds of `workspace:GetServerTimeNow()`. Leg k walks from target k - 1 to target k for at most 8 s, then idles for the rest. Every client computes target k from `Random.new(seed + k)`, so no network is needed and late joiners agree.
- **Target:** a point in the soil rectangle (`plot.soil`, the same part the server uses, `ServerScriptService.GardenService.PlotRegistry.lua:21-30`) inset by pet radius + 1.5, that passes P32 against the fruit in the fruit folder with this plot's `PlotName`, and is reachable within 8 s at up to 2x species speed. Up to 8 seeded candidates; if none passes, the pet idles where it is for that leg.
- **Obstacles:** parts tagged `PetObstacle` (CollectionService) are treated like fruit, so new plot decor on the soil ([[Plot Overhaul and Environment]]) and an underground hatch ([[Underground and Ores]]) are avoided.
- **Owner nearby (P33)** and **explosive circles (P34)** override the schedule locally until they end.
- **Ground height:** `plot.topY` (`PlotRegistry.lua:145-152`) plus the model's `HipHeight` x scale.

### Wild animals in the forest (client)
- Drawn only for their owner, only within 160 studs of the camera. The server sends home points with the marker's ground height. Wander targets (within 8 studs of home) get their height from a raycast down that only hits `workspace.Terrain`, the ground ForestBuilder seats things on (`ServerStorage.ForestBuilder.lua:59-66`). If the terrain under a point has not streamed in yet, the animal stays at home height until it has.
- Clearings hold grass, flowers and few rocks: about 0.08 rocks per 1,000 square studs (`ServerStorage.ForestBuilder.lua:309-317`), so short wanders rarely clip anything.

### VFX and sounds
| Use | Source | Notes |
|---|---|---|
| Tame, merge, arrive and leave | `Anime.Shiny-01` from the staging area ([[Existing VFX and Props]]) | copied to `ReplicatedStorage.PetFX`; Rate 0, `Emit(n)` only |
| Happy (small) | `Anime.Stars-01` | Emit(6) |
| Giant tier 7 to 9 aura | `Auras.RNG-Aura-02` | the planned use in [[Existing VFX and Props]] |
| Merge sound | existing `Config.MergeSoundId` (`ReplicatedStorage.GardenConfig.lua:741`) | client side, at the pet |
| Eat crunch, tame chime, poof, optional calls per species | Roblox licensed library, picked during the build | 2D for the owner (chime) or 3D with roll-off at 60 studs; volume 0.5 |

### Forest decor (optional)
- At the 3 forest docks ([[Boats and Swimming]] R24): one Studded Pack `Tent` and `CampFire` at the edge of the nearest clearing, so players can see where the animals are from the beach. The 200 degree dock already gets a campfire landmark in the Boats note, so that one is shared.

### What must be built
- `ReplicatedStorage.PetModels` (six prepared rigs), `ReplicatedStorage.PetFX` (Shiny-01, Stars-01, RNG-Aura-02, the collar part).
- `GardenUI.PetsMenu` and `GardenUI.PetsButton` in StarterGui (U1, U2), and `IndexMenu.PetList` plus the Pets tab (U5; the tab column itself is built once, by [[Underground and Ores]] U4).
- The scripts in the Code plan.

## Asset requests for Samarth

Two images. **No animations to publish (D-004). No Robux products at launch.** Method, as in [[Consistent Image-Based UI Kits in Roblox]] section E: attach the named kit images as references, one asset per generation, transparent PNG (ChatGPT) or flat green background with a 2 to 3 px white outline (Gemini, then key the green out). Keep words out of AI art and add them in an image editor with FredokaOne. Clean the edges (alpha-bleed about 4 px), keep it at or under 1024 x 1024, import it in Studio (Asset Manager, Import) under the account that owns the game, and send me the id.

| # | Asset | Config key | Shown at | Export canvas |
|---|---|---|---|---|
| A1 | "PETS" panel title | `PetConfig.Images.Title` | about 170 x 74 | 512 x 222 |
| A2 | Pets HUD button | `PetConfig.Images.HudButton` | 96 x 92 | 512 x 491 |

**A1 "PETS" title.** Most exact: in Photopea open the INDEX title image (IndexMenu.TitleText, rbxassetid 103221050515090), copy its layer style onto the word PETS in a heavy italic font, export. Or with ChatGPT:
> Image 1 is the exact style to copy: the word INDEX, the panel title from my Roblox game (rbxassetid 103221050515090). Make the single word PETS in exactly that lettering: the same heavy italic slant, the same white-to-light-grey vertical gradient, the same thick dark outline relative to letter height, the same small bevel and the same letter spacing. Transparent background. The word centred with 6% padding. No drop shadow outside the outline, no other shapes, no other words. Canvas 512 x 222.

Check the spelling before uploading.

**A2 Pets HUD button.** Make the icon with AI, then add the label yourself.
> Image 1 is my game's Index HUD button (rbxassetid 118941334639047) and Image 2 is my Inventory HUD button (rbxassetid 108477171933167). Copy their exact style: one chunky glossy cartoon object, thick dark outline, saturated colours, a bright highlight at the upper left, slight tilt. Image 3 is the blocky white Sheep from the Stud-Themed Animal Pack (the third animal from the left in the screenshot). Draw ONE object: that blocky white sheep, made of rounded cubes with soft studs on top, standing in a small wooden feed trough that holds one red apple with a green leaf (the same apple as on Image 1). Front three-quarter view, centred, filling the top 70% of the canvas. Leave the bottom 30% empty for a label. No text, no background, no sparkles, no shadow. Canvas 512 x 491.

Then, in an image editor, add the word "Pets" in the bottom 30%: FredokaOne, white fill, a thick black outline, the same size and position as the word "Index" on Image 1. Attach `ui-reference-hud-left.png` and `pack-animals.jpg` from the vault's Images folder as Images 1 to 3 if the ids cannot be attached directly.

**Sounds.** Nothing to make. During the build I will pick an eat crunch, a tame chime and a poof from Roblox's licensed library (they play in any experience, as with the music in D-002) and list the ids for you to approve.

**Later, optional (not at launch):** a "+1 Pet Slot" gamepass. If you want it: Creator Hub, your experience, Monetization, Passes, Create a Pass; send me the pass id. It would be a fixed item, not a random one, so no odds or PolicyService checks are needed ([[Roblox Platform Constraints for the Expansion]] section 10).

## Code plan

Follows the GardenService pattern: each server module is `return function(G)`, required in order, sharing state only through `G` (`ServerScriptService.GardenService.lua:88-98`). Design only; nothing has been edited.

### New: `ReplicatedStorage.PetConfig` (ModuleScript, shared)
Requires `GardenConfig`, `Fruits.FruitIndex`, `FruitEconomy`, `IndexRewards`. Holds data and pure functions, so the server and the client compute every number the same way.
- Data: `Species` (Table 1 rows: key, firstTier, weight, bonus, babyScale, speed, stride, gait), `Sizes` (name, scale), `Units` (U table), `LimitUnits = 8.75`, `WildCount = 6`, `RespawnSeconds = 30`, `WanderRadius = 8`, `FeedRange = 20`, `TeleportRange = 80`, `FeedCooldown = 0.5`, `HoldSeconds = 0.6`, `TierRoll = {same = 0.55, below1 = 0.25, below2 = 0.08, above = 0.12}`, `RequestOffset = {3, 5, 7}`, `SlotPrices = {10000, 150000}`, `BagCap = 60`, `RenderRange = 220`, `WildRenderRange = 160`, `Strings` (U7), `Images = {Title = 0, HudButton = 0}`.
- Functions: `Key(species, tier, size)`, `Parse(key)` (returns nil unless valid), `IndexKey(species, tier)`, `FeedsFor(tier)`, `SlotsFor(rebirths, bought)`, `BestRank(rebirths, bestRankMerged, spawnLevel)` (uses `Config.CapRankFor` from [[Rarity Gating and Rebirth]]), `UnitsFor(key, bestTier)`, `Income(equippedKeys, bestRank)` returning `base, units, capped`, `RequestRank(animalTier, bestTier, bestRank)`, `IndexReward(tier)` returning `money, shards`.

### New: `ServerScriptService.GardenService.Pets` (ModuleScript)
- **Required** right after `require(script.CrownDecree)(G)` (`ServerScriptService.GardenService.lua:518`), before the wiring at `:520`. It takes from G: `Config`, `FruitIndex`, `Economy`, `IndexRewards`, `Store`, `Players`, `ReplicatedStorage`, `RunService`, `balances`, `bestRank`, `held`, `releaseHold`, `storedFolder`, `plotFor`, `levelOf`, `publishCash`, `setShards`, `notifyRemote`, `noticeRemote` (all already in G: `GardenService.lua:390-395`, `Tools.lua:314-317`, `Carrying.lua:356-357`, `Upgrades.lua:426-428`, `Progression.lua:383-393`).
- **Remotes:** made inside the module the way the Crown module makes its own (`ServerScriptService.GardenService.Crown.lua:42-53`): `PetRequest` and `PetEvent` in `GardenRemotes`.
- **State:** `pets[player] = { owned = {[key] = n}, equipped = {[slot] = key}, index = {[indexKey] = 0 or 1}, bought, hint, wild = {[id] = animal}, nextId, feedAt, actionAt, lastPos, lastR }`.
- **Put into G (late-bound, like `G.royalCapture`, `Crown.lua:252-290`):**
  - `G.petBase(player)`: P25 to P27. Yield-free, at most 8 table reads.
  - `G.petsCapture(player)`: returns the save table.
  - `G.petsApply(player, saved)`: fills state, builds `PetInventory` and `PetIndex`, truncates `equipped` to the slot count, publishes attributes, rolls the wild set. Yield-free.
  - `G.petsOnRebirth(player)`: rerolls the wild set and republishes `PetBestRank`. Yield-free.
  - `G.petsTeardown(player)`: clears this player's tables and the `PetsEquipped` attribute.
- **Handlers** (each re-validates, `PetRequest` actions below). `claim` copies `claimIndex` (`ServerScriptService.GardenService.Progression.lua:345-372`): set 1, pay cash into `balances`, shards with `setShards`, `publishCash`, then `task.spawn(Store.ForceSave, player)`. `buySlot` copies `buyTool`'s shard check (`ServerScriptService.GardenService.Tools.lua:100-107`) and force-saves. `feed` consumes the carried fruit with `releaseHold(player, true)` (the same call storing uses, `ServerScriptService.GardenService.Storage.lua:49`) or decrements the `StoredFruits` IntValue (as at `Storage.lua:83-84`). Tame, merge, equip and release call `Store.Touch(player, Config.SaveTouch)` (`ReplicatedStorage.GardenConfig.lua:1132`).
- **Sweep:** one loop every 1 s, wrapped in pcall like the Ambient loops (`ServerScriptService.GardenService.Ambient.lua:403-404`): due respawns, arrival detection (P6), `lastPos` samples for P13 (d), and request refresh when b changes (P9).
- **Dev doors (Studio only, `RunService:IsStudio()`, one-shot attributes like `GardenService.lua:631-684`):** `DevPetGive` (a key string), `DevPetWildHere` (true: homes the 6 animals around the player), `DevPetRoll` (N: rolls N animals and prints the tier and species counts), `DevPetPose` (a role name: holds that joint at +30 degrees on the player's pets, to check `PetAxis` and `PetSign`).

### New: client scripts
- `StarterPlayer.StarterPlayerScripts.PetWorld` (LocalScript) with child ModuleScripts `PetAnimator` (gait engine, states, species tables) and `PetWander` (schedule, targets, avoidance). It draws every player's equipped pets from the `PetsEquipped` attribute on each Player (players are never streamed out, unlike plot models), keyed by that player's `PlotName`, and the local player's wild animals from `PetEvent` snapshots. Clones go into a client-only folder `workspace.PetsLocal`. It pools up to 8 rigs per species and reuses them. It moves all roots with one `workspace:BulkMoveTo` per frame and writes joint `Transform`s in `RunService.PreSimulation`.
- `StarterPlayer.StarterPlayerScripts.PetsMenu` (LocalScript): U1, U2, U3. Fires `PetRequest` only.

### Changes to existing scripts

| File:line | Change |
|---|---|
| `ServerScriptService.GardenService.lua:518` | Add `require(script.Pets)(G)` after the Crown modules. |
| `ServerScriptService.GardenService.lua:708-714` | After the save (`Store.SaveAndRelease`, `:711`) and before `releaseHold` (`:714`): `if G.petsTeardown then G.petsTeardown(player) end`. Clearing before the save would save an empty pet list. A separate `PlayerRemoving` handler in the module is not safe for the same reason: handlers run in either order (`ServerScriptService.GardenService.Crown.lua:256-258` notes this for the royal record). |
| `ServerScriptService.GardenService.Ambient.lua:465` | `local base = plotBase(plot) * (1 + (G.oreBoost and G.oreBoost(player) or 0)) + (G.petBase and G.petBase(player) or 0)` (one line shared with [[Underground and Ores]]: the Ore Bonus applies to fruit only, pets are added after it, D-018). Everything after it (rate, `CashPerSecond`, the `Income` payload `rate / base`, `:466-478`) stays right. |
| `ServerScriptService.GardenService.Persistence.lua:81` | `if G.petsCapture then profile.pets = G.petsCapture(player) end`. |
| `Persistence.lua:270` | Right after `publishUpgrades(player, profile.upgrades)`: `if G.petsApply then G.petsApply(player, profile.pets) end`. After it, not before: the wild roll inside `petsApply` reads the Spawn Rarity level through `levelOf`, which is only right once `publishUpgrades` has run, and `Rebirths` is already published at `:267`. It still runs before offline pay (`:324`), which needs the pets. |
| `Persistence.lua:191` | `local rate = (plotBase(plot) * (1 + (G.oreBoost and G.oreBoost(player) or 0)) + (G.petBase and G.petBase(player) or 0)) * (1 + indexBonus(player)) * rebirthMultiplier(player) * Config.OfflineRate` (the same shared line as [[Underground and Ores]]). |
| `ServerScriptService.GardenStore.lua:220-283` | `Store.Default()` gains `pets = { owned = {}, equipped = {}, index = {}, bought = 0, hint = false }`. |
| `GardenStore.lua:309-506` | `sanitise` copies `pets` (rules below). Required: a field only in `Default()` is dropped on the next load (`GardenStore.lua:447-450`). |
| `GardenStore.lua:1197-1209` | Self-test: add the worst pet payload (all 172 keys at 999, 8 equipped, 43 Index entries, 2 bought). Measured with Python JSON: the existing worst payload is 3,196 characters; with pets it is 6,764, under the 8,192 check. Raise the check to 16,384 when ores and bats add fields; the real limit is 4,194,304 per key ([[Roblox Platform Constraints for the Expansion]] section 1). |
| `ServerScriptService.GardenService.RebirthLock.lua:119` | After `publishMultiplier`, inside the yield-free block: `if G.petsOnRebirth then G.petsOnRebirth(player) end`. It must come after the head start is set (the [[Rarity Gating and Rebirth]] change at `RebirthLock.lua:95-97`) and after `bestRank` is cleared (`:102`). |
| `StarterPlayer.StarterPlayerScripts.GardenClient.lua:138-140` | In `setTags`, return unless `prompt.Parent` is in `fruitFolder`. Today the handler runs for every prompt in the game, so a wild animal's prompt would get the fruit outline Highlight (`GardenClient.lua:124-136`, [[Client UI - HUD and Menus]] gotcha 9). |
| `StarterPlayer.StarterPlayerScripts.EarnPopups.lua:87-108` | On `Income(multiplier, dt)`, also float one popup per own equipped pet: `PetConfig.Income` per pet, scaled down evenly when capped, x `dt x multiplier`. |
| `StarterPlayer.StarterPlayerScripts.IndexMenu.lua` | The Pets tab (U5, in the tab column [[Underground and Ores]] U4 adds) and `PetList`; `stateOf` for pet cards (`:236-241` pattern); info and rewards for pets (`:243-273` pattern); Claim on the Pets page fires `PetRequest("claim", indexKey)` instead of `IndexClaim` (`:447`). |
| `StarterPlayer.StarterPlayerScripts.AlertBadges.lua:71-78` | `indexWaiting` also looks for a 0 in `PetIndex`. New `petsWaiting` for the Pets button (U1). |
| `ReplicatedStorage.GardenConfig.lua:1863-1867` | Add `"PetsButton"` to `HudClusters`. |
| `StarterPlayer.StarterPlayerScripts.ButtonAnimator.lua:52-55` | Add `PetsButton = true` to `MAGNETIC`. |
| `StarterPlayer.StarterPlayerScripts.HudScale.lua:137-160` | The left-column guard in U1, at the end of `apply()`. |
| `ShopMenu.lua:616`, `IndexMenu.lua:47`, `StoredFruitsMenu.lua:541`, `UpgradeMenu.lua:421`, `RebirthMenu.lua:51` | Add `"PetsMenu"` to each "never two panels at once" list. |
| [[King UI Overhaul]] rules 15 and 16 | Add `PetsMenu` to the Kingdom panel's modal list and `PetsButton` to the arena-focus hide list. |

### Remotes

| Name | Direction | Payload | Server validation |
|---|---|---|---|
| `PetRequest` (new RemoteEvent) | client to server | `(action: string, arg: string or number or nil)` | `action` must be one of the list below; `Store.IsReady`; 0.1 s between actions per player (silent drop); then per action: |
| | | `"feed", id` | id is a whole number in this player's `wild`; all of P13 |
| | | `"equip", key` | `PetConfig.Parse(key)` valid; owned count > copies already equipped; a free slot (else `SlotsFull`) |
| | | `"unequip", slot` | whole number 1 to 8, slot filled |
| | | `"merge", key` | valid, owned count >= 2, size < 4 |
| | | `"mergeAll"` | none |
| | | `"equipBest"` | none |
| | | `"release", key` | valid, owned >= 1 |
| | | `"claim", indexKey` | a valid species-tier pair with `index[indexKey] == 0` |
| | | `"buySlot"` | bought < 2 and shards >= price |
| | | `"sync"` | none; replies with a `"wild"` snapshot (the client asks once on start, because `petsApply` can run before its listener exists) |
| `PetEvent` (new RemoteEvent) | server to the owner | `(kind, data)`: `"wild"` with up to 6 `{id, species, tier, x, y, z, request or nil, fed, feeds, locked}`; `"fed"` with `{id, fed}`; `"tamed"` with `{id, key}` | One-way; the server never connects `OnServerEvent`. The client checks every field's type before use. |

Strings longer than 32 characters, NaN numbers and wrong types are dropped silently. `Parse` accepts only a known species, a whole tier from the species' first tier to 9 and a whole size from 1 to 4.

### Save fields

| Field | Type | Default | Sanitise rule |
|---|---|---|---|
| `pets.owned` | table key -> whole number | `{}` | valid key and count 1 to 1,000; anything else is a repair |
| `pets.equipped` | list of keys | `{}` | at most 8 valid keys; each key at most its owned count (extras dropped) |
| `pets.index` | table indexKey -> 0 or 1 | `{}` | valid species-tier pair, value exactly 0 or 1 |
| `pets.bought` | whole number | 0 | clamped to 0 to 2 |
| `pets.hint` | boolean | false | true or false only |

**Old saves.** No player has pets yet, so a missing `pets` field loads as the default. No schema bump: adding a field is forward and backward compatible (`ServerScriptService.GardenStore.lua:8-12`), and a bump would kick players whose document is newer than their server (`GardenStore.lua:764-772`). The slot count is not saved; it is recomputed from rebirths and `bought`, and `petsApply` drops equipped entries past it (they stay owned).

### Attributes and replicated folders
- Player `PetsEquipped` (string): slot keys joined by commas, an empty field for a free slot, for example `"Sheep|1|2,,Deer|1|1"`. Read by every client to draw pets.
- Player `PetSlots` (whole number), `PetBestRank` (whole number), `PetUnits` (number, after the limit), `PetIncome` (number, base $/s before the multiplier), `PetBag` (whole number).
- Player folder `PetInventory`: one IntValue per owned key (value = count), like `StoredFruits` (`ServerScriptService.GardenService.Tools.lua:24-49`).
- Player folder `PetIndex`: one IntValue per found species-tier pair (0 found, 1 claimed), like `IndexState`.
- Studio only: `DevPetGive`, `DevPetWildHere`, `DevPetRoll`, `DevPetPose`.

## Edge cases and abuse

- **Stealing stays removed.** Wild animals belong to one player and only they see them. No action moves a pet between players. There is no trading or gifting at launch: Grow a Garden switched gifting off twice in July 2025 over a duplication exploit ([[Pets - Taming, Merging and Wandering]] section "What to avoid").
- **Duplication.** Every pet action is yield-free on the server, so a save cannot land between "remove two" and "add one". There is no animation gap on the server to cover (unlike fruit merges, which need the pending ledger, `ServerScriptService.GardenService.lua:134-169`). Capture reads the same table.
- **Feeding from far away or by teleport.** P13 (c) checks the root against the server's own home point, and (d) against the 1 s position sample. The worst a teleporter gains is skipping the trip; they still pay the fruit.
- **Remote spam.** 0.1 s between actions, 0.5 s between feeds, silent drops. The platform cap of about 500 events per second is far too loose to rely on ([[Roblox Platform Constraints for the Expansion]] section 7).
- **Rejoin to reroll for a Basilisk.** A rejoin rolls a new set (P10). That only saves walking to a new clearing; a Basilisk is still 4 in 100 species rolls at tier 6 and up. Accepted.
- **Leaving mid-feed.** Each feed is complete on its own. Partial progress is lost with the wild set; the fruit already fed is gone. Pets already tamed are saved (the save happens before teardown).
- **Rebirth mid-feed.** Rebirth and feed are both yield-free, so one finishes before the other starts. The rebirth rerolls the wild set, so partial progress is lost. The rebirth confirm already tells players their stored fruit resets ([[Rarity Gating and Rebirth]] U4).
- **Carrying a tool.** A held TNT is never fed: the carried path needs a fruit with the request's exact name, otherwise storage is used.
- **Bag full.** Checked before any fruit is taken (P13 f).
- **No plot (all 12 taken, 50-player servers).** Taming and the panel still work. Nothing is drawn and nothing is earned, exactly like fruit: the income tick only pays plot owners (`ServerScriptService.GardenService.Ambient.lua:464`).
- **Explosives, bats, artillery, boats.** Pets are client-only models with every collision and query flag off, and they are in nobody's target list ([[Bats]] B29, [[Weapons and Artillery]] rule 25, [[Explosives and Fruit Health]] rule 17). Roblox bans animal abuse at every rating ([[Roblox Platform Constraints for the Expansion]] section 10), so pets never take damage.
- **A bat knocks a player off an animal mid-hold.** The prompt releases and nothing is consumed. Bats work in the forest ([[Bats]] B24); the 0.6 s hold keeps the window short.
- **Pets and fruit clicks.** CanQuery off, so pets never block a prompt, a carry raycast or a drop. Pets have no prompts on the plot, so they never take one of the 16 visible prompt places or the one-per-key slot ([[Roblox Platform Constraints for the Expansion]] section 9).
- **Server authority.** Positions of pets on plots do not matter to the server. Income comes from the equipped list and the best rank, both server-owned. A client can only change what it draws for itself.
- **Mobile.** Hold-to-feed works with a tap and hold. The request card is world-sized and readable at 45 studs. The Pets panel uses the Index shell already used on phones. The left-column guard keeps the Pets button off CornerHud (U1). Tap targets keep the kit button sizes.
- **Streaming.** Pet and wild animal models are created on the client, so streaming never removes them. Plot soil and forest terrain can stream out; pets on a plot whose soil is not loaded are not drawn, and wild animals wait at home height (World and assets).
- **Night.** Pets sleep more at night but still earn. Sleep is never a penalty.
- **Old pets fading.** A pet three tiers behind earns little (Table 2). The bag cap pushes players to release or merge, and the Index keeps the record forever.

## Performance

Limits are from [[Roblox Platform Constraints for the Expansion]].
- **Parts.** Per rig: Sheep 23, Deer 34, Turtle 24, Hyena 38, Giraffe 33, Basilisk 87 (pack parts plus the collar). A typical view, the own plot and two neighbours each side with 6 pets each at about 35 parts, is about 1,050 parts. The worst own plot, 8 Giant Basilisks, is 696 parts. Wild animals add about 210 parts, and only in the forest, where plots are far out of range. The pack is plain parts with about 2,796 triangles in total ([[Pets - Taming, Merging and Wandering]] source 39), so triangle cost is negligible.
- **Per frame (client).** One `BulkMoveTo` for every drawn root (at most 48). Joint `Transform` writes only for rigs within 90 studs and on screen, at most 24 rigs and at most 1,200 joint writes per frame; past either cap the farthest rigs drop to root-only movement. Full rate within 40 studs, every other frame from 40 to 90, root only from 90 to 220, hidden past 220. Target: under 1.0 ms for PetWorld in the MicroProfiler on a PC, checked in T17. One developer's DevForum test moved 100 pets at 160 to 170 FPS on PC; this budget is a quarter of that ([[Pets - Taming, Merging and Wandering]] section 3).
- **Network.** Nothing per frame: walking is computed from the shared clock. A `"wild"` snapshot is about 400 bytes on join, after each tame and after a rebirth. `PetsEquipped` is under 160 bytes per change. The budget is 50 KB/s per client.
- **Server.** The income tick reads at most 8 table entries per player every 5 s. The 1 s sweep walks at most 50 players x 6 animals. No physics, no Humanoids, no PathfindingService (its slowdowns under many calls are reported in the platform note, section 5).
- **Lights and particles.** PointLights only on tier 7 to 9 pets within 120 studs. Emitters burst with `Emit` and never change properties per frame, which the docs warn against. Sparkle Rate 2 is far below the mobile cap of 100 per second per emitter.
- **Highlights.** One short Highlight per merge pop (0.4 s), sharing the 24 the game allows (`ReplicatedStorage.GardenConfig.lua:1887-1889`).
- **Memory.** Rigs are pooled and reused, the docs' advice for NPCs. Viewports: up to 8 slot cards, the visible bag cards and 3 request cards.
- **Saving.** About 3.6 KB in the worst case (Code plan); no extra DataStore calls (pets ride the existing autosave, `Store.Touch` and `ForceSave`).

## Test plan

Before every playtest run `workspace:SetAttribute("StudioNoSave", true)` in the command bar, or the playtest writes the real save (`ServerScriptService.GardenStore.lua:567-575`). Dev doors are attributes on your Player, for example `game.Players.<you>:SetAttribute("DevPetGive", "Sheep|1|1")`. Stage changes use `DevSetRebirths` from [[Rarity Gating and Rebirth]].

| # | Rules | Steps | Pass when |
|---|---|---|---|
| T1 | P2, P36, P37 | DevSetRebirths 0. Open the Index, press the Pets tab. | 9 rows x 6 columns. Row 1 has Sheep and Deer as "???", the other 4 cells empty. Rows 2 to 9 show the padlock and "Rebirth 1" to "Rebirth 8". |
| T2 | P7, P3 | DevSetRebirths 2, best fruit Dandelion. `DevPetRoll` 1000. Then reach a Rare fruit and roll 1000 again. | First: tier 2 about 63%, tier 1 about 25%, tier 3 (locked) about 12%, each within 3 points. Second: 67% / 25% / 8%. No Hyena below tier 3, no Basilisk below tier 6. |
| T3 | P5, P6, P10 | Swim to the forest at bearing 200. Read the `"wild"` snapshot. Leave, then arrive at bearing 76. | Homes are in the 2 clearings nearest 200, 3 each, 12+ studs apart. After the second arrival, unfed animals are in the 2 clearings nearest 76. |
| T4 | P9 | DevSetRebirths 0; best fruit Carrot, then Apple. Read requests of tier-1 animals. | Carrot: rank min(64, 65) = Blueberry. Apple: Strawberry (rank 63). |
| T5 | P12 to P15 | Store a Strawberry; feed a tier-1 animal. Then carry one and feed. Then try with none. | First: storage count drops by 1, Sheep or Deer tamed, toast, Baby on the plot in slot 1. Second: the carried fruit is used, storage untouched. Third: `NeedFruit` toast, nothing consumed. |
| T6 | P13 c, d, e | Stand 30 studs from an animal and fire `PetRequest:FireServer("feed", id)` from the client command bar. Then fire twice within 0.5 s from close by. | First: refused with `TooFar`, nothing consumed. Second: only one feed counted. |
| T7 | P13 f, P22 | `DevPetGive` 60 times, then complete a tame. | `BagFull` toast, fruit not consumed. Merge still works. |
| T8 | P17, P18 | Give two `Sheep|1|1`, equip both (slots 1 and 3), press Merge. Then give 8 Babies and press Merge All. | One `Sheep|1|2` in slot 1, slot 3 free. Merge All leaves exactly one Giant. The plot shows the merge pop. |
| T9 | P19, P20 | DevSetRebirths 0, 2, 4, 6, 8: read `PetSlots`. DevSetShards 10000, buy a slot; try a third purchase. | 3, 4, 5, 6, 6. After buying: +1 and shards 0. A third purchase is refused. |
| T10 | P25 to P27 | DevSetRebirths 2, Spawn Rarity at Eggplant, best merge Pear. Equip one Giant Rare Sheep. | `PetBestRank` 41. `PetIncome` = 3.5 x 71.2M = 249.2M (before the multiplier). Give 3 more Giants: `PetUnits` 8.75, popup totals capped. |
| T11 | P26 | Same player, equip a Giant Uncommon Sheep. | Units 1.5 (one tier behind counts as Huge). |
| T12 | P28 | API access off (in-memory store), StudioNoSave off. Equip pets, leave, change `lastSeen` 600 s back with the dev panel, rejoin. | `OfflineEarned` includes `petBase x 600 x 0.03` times the Index and rebirth bonuses. |
| T13 | P30, P10 | Own pets at stage 3, meet the needs and rebirth. | Pets, slots, Pets Index kept. `PetBestRank` = 51 (head start Bee Balm) at once. A new wild set arrives. |
| T14 | P31, P33 | Studio Test with 2 players. Watch player 1's pets from player 2. Walk player 1 next to an idle pet. | Same spots within about 1 stud on both screens. The pet turns to player 1; a Happy plays at most once per 20 s. |
| T15 | P32 | Fill a level-0 plot with 35 fruit, equip 8 Giants (`DevPetGive`). Watch 2 minutes; carry and merge fruit through the pets. | No pet rests on a fruit or leaves the soil. Every fruit prompt, pick-up and drop works through pets. |
| T16 | P34 | With [[Explosives and Fruit Health]] built, place a TNT on the plot. | Pets inside the red circle walk out before it goes off. Nothing happens to them at the blast. |
| T17 | Performance | 8 Giant Basilisks on your plot and 6 pets on each neighbour (`DevPetGive` on test players). MicroProfiler on PC, then the Device Emulator phone. | PetWorld under 1.0 ms per frame on PC; no frame-time spikes on equip changes. |
| T18 | P38 | Tame a Rare Hyena, press Claim on its card. | +50 shards and +4.27B cash once. A second Claim does nothing. |
| T19 | P40, U1 to U5 | Device Emulator: a phone (812 x 375), a tablet and a 1600 x 900 desktop, at Actual Resolution. Open the Pets panel, the Index Pets page, a confirm. Run the `UIOverlapCheck` from [[King UI Overhaul]]. | Zero overlap warnings. The Pets button never touches CornerHud. No emoji or separator characters anywhere. |
| T20 | Rigs | `DevPetPose` each role on each species. Then watch each species walk, idle, sit, sleep, eat, and play Happy. | Every joint bends the right way (fix `PetSign` or `PetAxis` where not). Legs follow the gait table; the Basilisk shows two travelling waves; the Giraffe paces. |
| T21 | Giraffe | A Giant Giraffe on a level-0 plot; stand next to it; open its card. | It fits the plot, fades to 0.6 when you stand within 6 studs, and fits its viewport. |
| T22 | Save | Run the GardenStore self-test. Then rejoin with pets equipped. | "worst payload is small" passes with pets added. Owned, equipped, Index and bought slots all come back. |

## Dependencies and conflicts

- **[[Rarity Gating and Rebirth]].** Supplies the stage, `CapRankFor`, the head start (which sets the Spawn Rarity level the best rank reads), the Sealed card look and padlock A3, `StoredCapFor` (trip capacity), and the rebirth hook point. **Conflict, resolved here (and that note now agrees, [[Integration Review]] D-018):** that note used to say "Pet income should be pinned to `full(S)`". Pinning to full(S) pays stage-S money from the first second of a stage, about 3.3 million times the starting plot at stage 3 (Table 4), which would remove the stage. This note pins to the best fruit instead, which equals full(S)/35 per unit at the plateau, so the stage share is the same where it matters and the early stage stays a climb. Its open question 4 (ores and pets out of the rebirth Index rule) matches P39.
- **[[Bats]].** Bats never hit pets (B29). Bats work in the forest (B24), so a player can be knocked away mid-feed; nothing is lost (Edge cases).
- **[[Explosives and Fruit Health]].** Explosives never affect pets (its rule 17). This note takes up its offer: pets walk out of warning circles (P34), reading its client circle data (its section 6.8).
- **[[Weapons and Artillery]].** Pets are never targets (its rule 25). Pets never go to sea (P35), so the sea combat zone never meets a pet.
- **[[Underground and Ores]].** Shares the side-income budget: the [[Stage Model]] assumes pets and ores add 35% of full-plot income at the plateau (rule 6). Resolved in the [[Integration Review]] (C16, D-018): pets keep this note's 25% limit, ores take 7.5 / 9 / 10.5% by floor, and the Ore Bonus applies to fruit income only, so the sum is 35.5% at the plateau and the two never compound. The underground entrance is a hut on the lot, outside the soil, so pets never meet it. The Index uses one tab column, Fruits, Pets and Ores (U5).
- **[[Pets]].** This note.
- **[[Boats and Swimming]].** Travel time decides what pets are worth (Table 8). Its forest docks (R24) and the islet route set the trip lengths used in the simulation. Feeding from storage means nothing has to be carried across the sea, but carried fruit works too, as its R13 and R33 require. A tamed pet never crosses the sea; it appears on the plot, as that note recommends.
- **[[Potions]].** No interaction (its own note says potions never affect pets). Aqua Affinity shortens trips, which raises pet value (Table 8, last row).
- **[[Plot Overhaul and Environment]].** Pets walk on `plot.soil`, now found by its `PlotSoil` attribute (that note's rule A3). Decor placed on the soil must carry the `PetObstacle` tag. Soil size is now kept through rebirth (its rule E1, D-020), but a new player still starts at level 0, which is why pet sizes are capped at x1.8.
- **[[King UI Overhaul]].** Its `UIKit` module, `UIKitPrefabs.Panel` and `UIOverlapCheck` are used here. Its rules 15 and 16 now list `PetsMenu` and `PetsButton` (added by the [[Integration Review]]). Pets use none of the royal zones (top-left chip, top-centre strip and herald). Its note says the arena should not hide pets and that [[Royal Combat]] decides the arena rule: pets never leave the plot (P35), so no pet is ever in the arena.
- **[[Royal Combat]].** It asks the Pets system to hide pets that follow an owner into the arena. Pets never follow anyone (P35), so there is nothing to hide.

**Decisions.** D-004 (procedural animation) is followed. D-009 (rebirth keeps pets) is followed. D-006 (head start) is why the research note's nap rule would never fire: at the start of every stage from 3 on, the head start puts the best tier at S - 1, and every pet a player owns is tier S - 1 or lower, so a "sleep until your best fruit reaches the pet's tier" rule wakes every pet at once; at stage 2 the pets are Common and the plot starts on Blueberry, also Common. P30 replaces the nap with the best-fruit pin. D-008 (rebirth bonus +25%, added) applies to pets through the cash multiplier. D-010 and D-011 are untouched; Table 8 shows the D-011 stage lengths still hold.

## Open questions for Samarth

Each has a default, so building can start now.

1. **What pet money follows.** Default: your best fruit this run (P25 to P27). The other choices: the full-plot money of your stage ([[Rarity Gating and Rebirth]]), which makes the start of every stage trivial (Table 4); or a fixed amount set when you tame ([[Pets - Taming, Merging and Wandering]] section 6), which makes every pet worthless one rebirth later.
2. **Nap rule.** Default: no money nap; pets nap only as an animation (P30). Reason in Dependencies (D-006).
3. **Pet limit.** Default: 25% of a full plot (8.75 best fruits), with ores at 7.5 / 9 / 10.5% on fruit income only, so the two together stay at about 35% (decided in the [[Integration Review]], D-018).
4. **Pets in the rebirth Index rule.** Default: no, only fruit count (P39).
5. **Which species unlocks when.** Default: Sheep and Deer Common, Turtle Uncommon, Hyena Rare, Giraffe Epic, Basilisk Mythic (Table 1).
6. **Private wild animals.** Default: each player sees only their own, so nobody races anyone for an animal. The other choice is shared animals, which brings back the "someone took mine" feeling stealing caused.
7. **Taming at stage 1.** Default: open from the start. The simulation says a forest trip makes stages 1 and 2 about 2 to 4 minutes longer; the pets pay off from stage 3. The other choice is "pets unlock at rebirth 1", which adds one more reason to rebirth.
8. **Shard slots.** Default: two, at 10,000 and 150,000 shards (Table 5).
9. **Robux.** Default: nothing at launch. A fixed "+1 Pet Slot" gamepass could come later (section 8).
10. **Release refund.** Default: nothing back, so tame-and-release can never farm anything.
11. **Pets follow you.** Default: no, pets stay on the plot ("they will walk around your base"). Following pets would need arena, sea and sky-island rules.
12. **Feeds per animal.** Default: 1 for Common to Rare, 2 for Epic to Mythic, 3 for Astral to Transcendent, the pattern 99 Nights in the Forest uses ([[Pets - Taming, Merging and Wandering]] section 1).
13. **Pet sizes on the plot.** Default: x1.00, x1.25, x1.50, x1.80. Bigger reads better but crowds a level-0 plot.
14. **Trading.** Default: none at launch (Edge cases).
15. **Bag size.** Default: 60 pets.
