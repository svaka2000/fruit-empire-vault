---
tags: [system, design]
date: 2026-10-04
status: proposed
depends_on: [Rarity Gating and Rebirth, Underground and Ores]
---
# Explosives and Fruit Health

Part of [[Fruit Empire - Home]]. Source of truth: [[Brain Dump 2026-10-04]]. Priced against [[Stage Model]]. Code facts come from [[Code Map]] and the export in `~/fruit-empire/src-export-2026-10-04`.

Everything under Rules, Numbers, UI, World and assets, and Code plan is a **proposal** until Samarth signs it off. Nothing here has been built.

## 2. Why

What Samarth asked for, word for word:

> "also add artilery and combat like tnt, nukes, etc. make this super planned and tell me exactly what assets you need dont make slop unless it looks clean and studded."

> "also tnt shouldnt instantly get rid of your fruits. similar to merge a nuke it should drain the health of the fruits. stronger tnt cost more."

> "this means that everything needs to correspond with the stage of the game a player is in. you can use rebirths to control and organize this."

> "also all Uis you make should follow the UI that we created, if you need any UI stuff then we can create them using chatgpt or gemini image generation following references of current UI"

> "instead everything should be structured. like for players its going to be confusing. all the UI should never overlap and also everything should be super clean."

> "I dont like the emoji-slop that you pushed on the sky island kings feature."

TNT already exists. It costs 2,500 shards and deletes every fruit within 18 studs on any plot, unless that plot is locked (`ServerScriptService.GardenService.Tools.lua:278-290`, `ReplicatedStorage.GardenConfig.lua:1616-1626`). This system replaces that delete with a health drain, adds three stronger explosives (Big TNT, Mega TNT, Nuke), and gives every fruit a health bar. It also sets the anti-grief rules, because stealing was removed on 3 Oct 2026 and an explosive must never become a new way to take things from other players. Finally, it is the one place fruit damage is decided. A later ore band in [[Underground and Ores]] can plug its nodes into the same blast code (not in v1, D-031), and any later weapon that can hurt fruit must go through it. ([[Weapons and Artillery]] weapons never hurt fruit: its rule 20.)

**Words used in this note**
- **Stage**: the newest rarity a player has unlocked. Stage = rebirths + 1, capped at 9. Stage 1 is Common, stage 9 is Transcendent ([[Stage Model]]).
- **HP (health points)**: how much damage a fruit can take before it is stunned.
- **Explosive**: one of the four shop items in this note: TNT, Big TNT, Mega TNT, Nuke.
- **Armed**: an explosive that has been put on the ground and is counting down.
- **Fuse**: the seconds between arming and the blast.
- **Blast centre**: the point where the explosive sits on the ground.
- **Radius (R)**: how far the blast reaches, in studs (Roblox's distance unit).
- **Falloff**: damage getting weaker the farther a fruit is from the blast centre.
- **Effective stage**: the lower of the attacker's stage and the garden owner's stage. Damage is computed from it.
- **Stunned**: what happens at 0 HP. The fruit earns nothing and cannot be touched for 20 seconds, then heals to full. It is never deleted.
- **Shielded**: the 30 seconds after a stun ends, when the fruit takes no damage.
- **Fresh garden**: a garden in its first 120 seconds after its owner joined or rebirthed. It cannot be targeted.
- **Lock**: the existing forcefield dome over your own plot (60 s up, 30 s recharge, `ReplicatedStorage.GardenConfig.lua:885-886`).
- **Shards per merge**: the shards a player gets for each merge into their newest rarity (D-010 in [[Decisions Log]]: 3, 7, 15, 32, 70, 155, 340, 748, 1,646 by stage). In code it is `Config.ShardDropFor(stage)` from [[Rarity Gating and Rebirth]].
- **Full-plot $/s**: the most a plot earns in a stage: 35 fruit, all the merge-cap fruit, times the rebirth bonus ([[Stage Model]]).

## 3. Player view

A walkthrough of one attack, from both sides.

1. Mia has rebirthed 3 times (stage 4). She opens the Shop and taps the Weapons tab. Its cards are ordered by the rebirth that unlocks them, garden explosives and sea weapons together; the four explosive cards are TNT, Big TNT, Mega TNT and Nuke. TNT and Big TNT show a price in shards. Mega TNT says "Rebirth 5 to unlock" and Nuke says "Rebirth 7 to unlock". Their Buy buttons are dimmed.
2. She buys a Big TNT for 1,300 shards. It goes into her Inventory like any tool.
3. She equips it. It sits above her head like a carried fruit. The carry bar reads "Big TNT" and "Radius 14, 2 hits".
4. She walks onto Leo's garden. Leo has rebirthed 4 times (stage 5). She presses Drop.
5. The Big TNT lands on the soil 3 studs in front of her. A red circle the size of the blast appears on the ground. A white number counts down above the TNT: 3, 2, 1. A fuse hiss plays.
6. Leo gets a red message at the top of his screen: "Mia placed a Big TNT in your garden! Lock to block it." A red "!" appears on his Lock button.
7. Leo is busy, so he does not lock in time. The TNT goes off with the classic Roblox explosion. Nobody is pushed or hurt.
8. Six of Leo's fruit inside the circle get health bars. Red numbers float up: "-98" at the centre, down to "-29" at the very edge. A Legendary fruit at the centre drops from 244 to 146 HP. (Damage uses Mia's stage 4, the lower of the two stages.)
9. Leo runs over and merges two damaged fruit. The new fruit is at full health. He also raises his Lock. Mia's next Big TNT is refused: "That plot is locked!" Her own Lock (if she had one up) dropped the moment she placed the first one.
10. Sixteen seconds after the last hit, Leo's damaged fruit start healing. Each bar fills back up and disappears when full.
11. If Mia had hit that Legendary fruit three times before it healed, it would have hit 0 HP. It would turn see-through, stars would circle it, and its bar would turn grey and slowly refill over 20 seconds. It earns nothing during that time and Leo cannot pick it up. Then it comes back at full health with a cyan bar for 30 seconds, during which nothing can hurt it.
12. Nobody gains cash, shards or fruit from hitting another garden. Leo never loses a fruit.

A nuke is the same flow with more warning. Every player in the server sees "NUKE incoming on Leo's garden in 10s". A siren plays. The red circle covers the whole plot. At zero, every fruit on Leo's plot drops to 40% health at once, then drains 10% per second for 4 seconds, so every fruit ends up stunned unless Leo locks during those 4 seconds. When it is over the owner gets a green message: "Your garden survived a nuke: +350 shards" (the amount for Leo's stage 5; it is 3,740 for a stage 8 garden). Only stage 8 players can place a nuke.

At stage 1 (0 rebirths) none of this exists for you. The explosive cards say "Rebirth 1 to unlock", and nobody can target your garden.

## 4. Rules

### A. The items
1. There are exactly four explosives. Shop key, name shown: `TNT1` "TNT", `TNT2` "Big TNT", `TNT3` "Mega TNT", `Nuke` "Nuke". Each is a `Config.Tools` entry with `kind = "place"` and `explosive = true`.
2. Each explosive unlocks at a stage: TNT at stage 2, Big TNT at stage 4, Mega TNT at stage 6, Nuke at stage 8 (1, 3, 5 and 7 rebirths). The server refuses to sell or place an explosive above the player's stage.
3. Price in shards = `Config.ShardPrice(N, rebirths)`, the one shared shard price (decision D-017, owned by [[Rarity Gating and Rebirth]]): N x the buyer's shards per merge, rounded to 2 significant figures (nearest 5 below 100). N = 15 (TNT), 40 (Big TNT), 100 (Mega TNT), 250 (Nuke). The price changes only when the buyer rebirths.
4. Rebirth turns every explosive the player owns back into shards, at the current (pre-rebirth) price x count, then removes them. This includes an explosive held in the hands. Potions and weapon charges are still kept, as today (`ServerScriptService.GardenService.RebirthLock.lua:85-88`). This is the one exception to "rebirth keeps tools", logged as decision D-020 and written into [[Rarity Gating and Rebirth]] rule 15.
5. The old `TNT` tool is retired. A saved bag that still holds `TNT` is paid back 2,500 shards per TNT on first load, and the entry is dropped (the existing retired-item refund path, `ServerScriptService.GardenStore.lua:73-76`, `:368-371`).

### B. Placing
6. An explosive is placed with the existing Drop button. Drop on a carried explosive arms it. It never goes off by itself if you die, despawn or leave while carrying it; it goes back in the bag (existing rule, `ServerScriptService.GardenService.Carrying.lua:297-306`).
7. You may place an explosive only where you stand on another player's garden soil, inside the existing 12-stud height band (`standingOn`, `ServerScriptService.GardenService.FruitFactory.lua:79-85`). Everything else is refused: your own surface garden, any mine (rule 34), the plaza, the sea, the sky, and anywhere while the `InArena` attribute is set.
8. You must own a plot yourself to place an explosive. A player with no plot (the server holds 50 players, the map has 12 plots) cannot attack.
9. Target gates, all checked at placement: the garden owner is stage 2 or higher; the garden is not fresh; the plot is not locked; the garden owner is not in the Royal Arena (`InArena`), because arena focus hides the Lock button ([[King UI Overhaul]] rule 16, D-028), so a fighter could not defend (added by the player-path pass, [[Combat Day]], decision D-035). For a Nuke, also: no nuke was placed anywhere on this server in the last 600 s, and this plot was not nuked in the last 1,800 s.
10. Attacker gates, checked at placement: your personal explosive cooldown has ended; you have no other armed explosive; fewer than 12 explosives are armed on the server.
11. The blast centre is 3 studs in front of the placer (`Config.DropForwardStuds`, `ReplicatedStorage.GardenConfig.lua:307`), on the soil surface (`plot.topY`), clamped inside the soil rectangle. The client never sends a position.
12. Placing an explosive on another garden ends your own Lock at once, if it is up, and starts your 30 s Lock recharge from that moment.
13. Fuse: TNT 3 s, Big TNT 3 s, Mega TNT 4 s, Nuke 10 s. Personal cooldown, counted from placement and shared by all four explosives: 6 s after a TNT, 10 s after a Big TNT, 15 s after a Mega TNT, 600 s after a Nuke.
14. On placement the owner gets a red warning toast naming the attacker and the item, and a red "!" on their Lock button until the blast. Every player within 300 studs sees the warning circle and the countdown.

### C. The blast
15. At the end of the fuse the server checks again: the plot still has the same owner (by UserId); it is not locked; it is not fresh; the owner is still stage 2 or higher; the owner is not `InArena`. If any check fails, nothing is damaged, the explosive goes back into the attacker's bag (if they are still in the server), and both players get a toast saying why.
16. A blast can only damage fruit tagged with the target plot's name (`PlotName`). Fruit on neighbouring plots are never hit, even inside the radius. Carried fruit and stored fruit are never hit. Fruit that are `Busy` (mid-drop, mid-merge or stunned) or shielded are skipped.
17. Explosives never affect players, pets, boats, knights or decor. The visual `Explosion` uses `BlastPressure = 0` and `DestroyJointRadiusPercent = 0`, as today (`ServerScriptService.GardenService.Tools.lua:292-298`). Boats are damaged only by [[Weapons and Artillery]].
18. Distance `d` is measured flat (X and Z only) from the blast centre to the fruit's centre. A fruit more than 12 studs above or below the blast centre is not hit.
19. Damage to one fruit = `round(D x f(d))`, rounding halves up (`math.floor(x + 0.5)`), and at least 1 if `d <= R`. `D = ceil(m x HP(effective stage))`, with m = 0.35 (TNT), 0.50 (Big TNT), 0.70 (Mega TNT). Falloff `f(d) = 1 - 0.7 x min(d / R, 1)` for `d <= R`, and 0 beyond R, so the edge of the circle still takes 30%. This is the same function as `Ballistics.falloff` in [[Weapons and Artillery]], so every blast in the game falls off the same way.
20. Effective stage = min(attacker's stage when they placed it, owner's stage at the blast). A veteran never hits a newer garden harder than a player of that garden's own stage would. A newer player hitting a veteran's garden does very little.
21. Nuke: at the blast, every fruit on the target plot loses 60% of its own max HP, whatever its distance. Then at 1, 2, 3 and 4 s after the blast, every fruit on the plot loses 10% of its own max HP (the **fallout**). Fallout skips `Busy`, shielded and carried fruit, and stops at once if the plot becomes locked. A fruit untouched by the lock ends at 0 HP and is stunned.

### D. Health
22. Max HP depends only on the fruit's rarity tier: `HP(t) = round(100 x 1.25^(t - 1))`. Common 100, Uncommon 125, Rare 156, Epic 195, Legendary 244, Mythic 305, Astral 381, Eternal 477, Transcendent 596.
23. Every fruit starts at full HP: when it spawns, is merged, is restored on join, or is taken out of storage.
24. HP is never saved. Rejoining, storing a fruit, or merging it gives full HP.
25. A damaged fruit earns full income and can be picked up, carried, dropped and merged as normal. Carrying keeps its HP. Merging two damaged fruit makes a fruit at full HP.
26. Regen: a damaged fruit that has taken no damage for 16 s regains 10% of its max HP per second until full. 16 s is longer than the longest gap between one attacker's blasts (Mega TNT, 15 s cooldown), so a lone attacker's chain is never undone by regen. (Was 10 s. With 10 s a Mega TNT fruit healed for 5 s between blasts and needed 3 centre hits, not the 2 in N2. Fixed by the player-path pass, [[Combat Day]], decision D-036.) Regen does not run while the fruit is carried or stunned.

### E. Zero HP: stunned, never destroyed
27. At 0 HP a fruit is stunned for 20 s. While stunned it earns $0, cannot be picked up, merged into, or used by Merge All, and takes no damage. It still counts toward plot capacity, it is still saved, and the owner's Clear button may still remove it.
28. When the stun ends the fruit is back at full HP and shielded for 30 s.
29. No explosive ever deletes, moves or transfers a fruit. (A `ZeroHpMode` setting may be added for "destroy", but the default is stun. See Open questions.)

**Why stun and not destroy.** Samarth's words were "shouldnt instantly get rid of your fruits", which leaves both readings open. I recommend stun for four reasons:
- Stealing was removed on 3 Oct 2026 because losing things to other players feels bad. Deleting fruit is the same loss in a new form ([[Sky Rivals - Weapons and Artillery]], "What to avoid").
- The cost to the victim stays tiny. A full nuke costs the victim 20 seconds of income. Under destroy, the same nuke costs about 10 minutes of re-merging a full plot even with every upgrade bought (table N5).
- Rebirth timing is the spine of the game ([[Stage Model]]). Stun keeps attacked players within 1% of the modelled stage times. Destroy would make stage length depend on who is in your server.
- A late player holds hundreds of thousands of shards (439K at the stage 8 gate). With destroy, they could wipe new gardens over and over. Merge a Nuke's early version had exactly this problem: "How do new people win this?" ([[Merge a Nuke - Explosives and Health Drain]]).

### F. Protection and the Lock
30. Stage 1 gardens (0 rebirths) can never be targeted, and stage 1 players cannot buy or place explosives.
31. A garden is fresh for 120 s after its owner joins or rebirths, and cannot be targeted then.
32. The Lock keeps its current numbers (60 s up, 30 s recharge from when it drops) and is raised the same way, with one change: while an explosive is armed on your plot (your `IncomingExplosive` attribute is in the future), you can raise the Lock from anywhere, including the mine, the sea, the forest and the sky. Today `lockPlot` refuses unless you stand on your own plot (`ServerScriptService.GardenService.RebirthLock.lua:196-226`), so the warning toast's "Lock to block it" could not be followed away from home (added by the player-path pass, [[Underground Unlock - Stage 6-7]] and [[Combat Day]], decision D-034). A locked plot refuses placement, cancels an armed explosive at its blast (the explosive goes back to the attacker), and stops nuke fallout.
33. The attacker gets nothing from a hit: no cash, no shards, no fruit, no items. The only explosive reward goes to the victim: when a nuke's fallout ends, the garden owner gets `5 x ShardsPerMerge(owner's stage)` shards.

### G. Underground
34. **No explosives in any mine in v1** ([[Underground and Ores]] rule 27, decision D-031). Placement is refused with "No explosives in the mine." The `G.registerBlastTarget("ore", provider)` hook stays in the code for a later ore band, when the [[Underground and Ores]] note decides node numbers. Fruit are never hit from underground.

### H. Server authority
35. Every check and every number above is decided on the server. The client only asks (Drop, BuyTool) and only draws what the server publishes through attributes and the `ExplosiveFx` remote.

## 5. Numbers

All numbers below are proposals. Each one is tied to the stage table in [[Stage Model]]. Damage and HP are deliberately small whole numbers (100 to 596) so players can read them. They grow 25% per stage, the same step as the rebirth cash bonus (D-008), while cash grows 96x to 7,243x per stage. That follows the Merge a Nuke lesson: damage grows slowly, earnings grow fast ([[Merge a Nuke - Explosives and Health Drain]]).

### N1. Fruit max HP by rarity tier
Formula: `HP(t) = round(100 x 1.25^(t - 1))`.

| Stage / tier | Rarity | Merge cap fruit | Full-plot $/s | Max HP |
|---|---|---|---|---|
| 1 | Common | Apple | 707 | 100 |
| 2 | Uncommon | Blue Lollipop | 430K | 125 |
| 3 | Rare | Pear | 3.7B | 156 |
| 4 | Epic | Watermelon | 3.6T | 195 |
| 5 | Legendary | Tropical Melon | 1.8Qa | 244 |
| 6 | Mythic | Dragon Pepper | 2.5Qi | 305 |
| 7 | Astral | Paradise Petal | 265Qi | 381 |
| 8 | Eternal | Solar Bloom | 506Sx | 477 |
| 9 | Transcendent | Soul Fruit | 24.9Sp | 596 |

### N2. The explosives

| Key | Name | Unlocks at stage (rebirths) | m (share of a same-stage top fruit's HP at the centre) | Radius R | Damage at the edge | Fuse | Personal cooldown | Price = N x shards per merge | Centre hits to stun a same-stage top fruit |
|---|---|---|---|---|---|---|---|---|---|
| TNT1 | TNT | 2 (1) | 0.35 | 10 | 30% of D | 3 s | 6 s | N = 15 | 3 |
| TNT2 | Big TNT | 4 (3) | 0.50 | 14 | 30% of D | 3 s | 10 s | N = 40 | 2 |
| TNT3 | Mega TNT | 6 (5) | 0.70 | 20 | 30% of D | 4 s | 15 s | N = 100 | 2 |
| Nuke | Nuke | 8 (7) | 60% of every fruit's own HP, then 10%/s for 4 s | whole plot | - | 10 s | 600 s (and 600 s per server) | N = 250 | all fruit on the plot |

Bigger explosives are not faster per fruit. A lone attacker drains a top fruit at about 5.8%/s with TNT (35% every 6 s), 5.0%/s with Big TNT, 4.7%/s with Mega TNT. What a bigger explosive buys is area: TNT covers 314 square studs, Big TNT 616, Mega TNT 1,257. A level-4 plot is 48 x 78 = 3,744 square studs (`ReplicatedStorage.GardenConfig.lua:117-123`).

### N3. Centre damage D by effective stage
`D = ceil(m x HP(effective stage))`. Edge damage is `round(0.3 x D)`.

| Effective stage | Rarity | Top fruit HP | TNT D (edge) | Big TNT D (edge) | Mega TNT D (edge) |
|---|---|---|---|---|---|
| 1 | Common | 100 | protected | protected | protected |
| 2 | Uncommon | 125 | 44 (13) | locked | locked |
| 3 | Rare | 156 | 55 (17) | locked | locked |
| 4 | Epic | 195 | 69 (21) | 98 (29) | locked |
| 5 | Legendary | 244 | 86 (26) | 122 (37) | locked |
| 6 | Mythic | 305 | 107 (32) | 153 (46) | 214 (64) |
| 7 | Astral | 381 | 134 (40) | 191 (57) | 267 (80) |
| 8 | Eternal | 477 | 167 (50) | 239 (72) | 334 (100) |
| 9 | Transcendent | 596 | 209 (63) | 298 (89) | 418 (125) |

"Locked" means no player at that stage owns the item yet. Because effective stage is the lower of the two stages, an item can still hit a garden whose row says "locked". Example: a stage 9 player's Mega TNT on a stage 3 garden uses stage 3's HP, so `D = ceil(0.70 x 156) = 110`. That stuns a stage 3 top fruit (156 HP) in 2 centre hits, the same as any stage 3 attacker with that item would.

### N4. Prices
`Price(item, stage) = Config.ShardPrice(N, rebirths)` = N x ShardsPerMerge(stage), rounded to 2 significant figures (D-017). ShardsPerMerge is D-010. Shard income per minute is from the [[Integration Check]] re-run of the [[Stage Model]] (attentive player, regular pet trips, ores, 7 careers, median; gross shards earned in the stage / minutes in the stage).

| Stage | Shards per merge | Shard income per min (model) | TNT | Big TNT | Mega TNT | Nuke |
|---|---|---|---|---|---|---|
| 1 | 3 | 15.2 | locked | locked | locked | locked |
| 2 | 7 | 36.8 | 110 (3.0 min) | locked | locked | locked |
| 3 | 15 | 89.6 | 230 (2.6 min) | locked | locked | locked |
| 4 | 32 | 142.5 | 480 (3.4 min) | 1,300 (9.1 min) | locked | locked |
| 5 | 70 | 246 | 1,100 (4.5 min) | 2,800 (11 min) | locked | locked |
| 6 | 155 | 577 | 2,300 (4.0 min) | 6,200 (11 min) | 16,000 (28 min) | locked |
| 7 | 340 | 758 | 5,100 (6.7 min) | 14,000 (18 min) | 34,000 (45 min) | locked |
| 8 | 748 | 2,023 | 11,000 (5.4 min) | 30,000 (15 min) | 75,000 (37 min) | 190,000 (94 min) |
| 9 | 1,646 | about 5,500 (first hour) | 25,000 | 66,000 | 160,000 | 410,000 |

The minutes in brackets are "minutes of that stage's shard income". Each step up the ladder costs about 2.5 to 2.7 times the one below: "stronger tnt cost more". A nuke at stage 8 costs about 94 minutes of stage 8 shard income (about two thirds of everything a player earns in shards during stage 8), so it is a rare, deliberate purchase. Explosives are optional spending. The model's planned shard spending (`SPEND` in `~/fruit-empire/sim/final.py:9`) does not include them, and shards are never what blocks a rebirth ([[Stage Model]]), so buying explosives cannot stall a stage.

### N5. What an attack costs the victim
A stun stops that fruit's income for 20 s. Cost = `stunned fruit / 35 x 20 s x full-plot $/s` on a full plot. "Big TNT, 6 fruit" assumes a Big TNT stuns 6 fruit (about 16% of a full level-4 plot). Stage times are from [[Stage Model]]. The destroy column uses a simulation of re-merging an empty plot to 90% of full income with every upgrade already bought, 3 s per merge: about 7.4 min at stage 1 and 9.8 to 9.9 min at stages 3, 6 and 9.

| Stage | Full-plot $/s | Big TNT stuns 6 fruit | Nuke stuns all 35 | Nuke as share of the stage's length | If fruit were destroyed instead (nuke) |
|---|---|---|---|---|---|
| 1 | 707 | protected | protected | protected | protected |
| 3 | 3.7B | 12.8B (3.4 s of income) | 74.8B (20 s) | 0.55% of 61 min | about 9.8 min of re-merging (16% of the stage) |
| 6 | 2.5Qi | 8.5Qi (3.4 s) | 49.8Qi (20 s) | 0.32% of 105 min | about 9.8 min (9%) |
| 9 | 24.9Sp | 85.3Sp (3.4 s) | 498Sp (20 s) | 0.56% of one hour of income | about 9.9 min |

### N6. Timers and limits

| What | Value | Note |
|---|---|---|
| Regen delay after the last hit | 16 s | Longer than the longest gap between one attacker's blasts (Mega TNT: 15 s), so one attacker chaining any explosive is not undone by regen. Was 10 s, which let Mega TNT targets heal 5 s between blasts ([[Combat Day]], D-036) |
| Regen rate | 10% of max HP per second | Server ticks once per second |
| Stun | 20 s | Fruit earns $0, cannot be touched |
| Shield after stun | 30 s | So a fruit can be stunned at most once every 50 s |
| Fresh garden | 120 s after the owner joins or rebirths | |
| Lock | 60 s up, 30 s recharge | Unchanged (`ReplicatedStorage.GardenConfig.lua:885-886`) |
| Nuke, server-wide | one per 600 s | Military Tycoon uses one per 10 minutes per server ([[Merge a Nuke - Explosives and Health Drain]]) |
| Nuke, same plot | once per 1,800 s | |
| Armed explosives | 1 per player, 12 per server | |
| Warning circle seen from | 300 studs | |
| Old TNT refund | 2,500 shards each, once | |

### N7. Nuke survivor bonus
`Bonus = 5 x ShardsPerMerge(owner's stage)`, paid when the fallout ends.

| Owner's stage | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 |
|---|---|---|---|---|---|---|---|---|
| Bonus (shards) | 35 | 75 | 160 | 350 | 775 | 1,700 | 3,740 | 8,230 |

The bonus is at most 8,230 shards, and the cheapest nuke is 190,000. Two friends nuking each other to farm the bonus would lose at least 95% of what they spend.

### Worked examples
- **Stage 1.** No explosives can be bought, and the garden cannot be targeted (rule 30).
- **Stage 3 vs stage 3, TNT (230 shards).** A Rare top fruit has 156 HP. `D = ceil(0.35 x 156) = 55`. A fruit 7 studs from the centre: `f(7) = 1 - 0.7 x 7 / 10 = 0.51`, so it takes 28. Three centre hits stun a top fruit. With the 6 s cooldown, one attacker stuns it at the third blast, 12 s after the first.
- **Stage 6 vs stage 6, Mega TNT (16,000 shards).** A Mythic top fruit has 305 HP. `D = ceil(0.70 x 305) = 214`. Two centre hits stun it. A fruit at the edge (20 studs) takes 64.
- **Stage 9 vs stage 9, Nuke (410,000 shards).** Every fruit drops to 40% at the blast and to 0% four seconds later, unless the owner locks. The owner loses 20 s of income (498Sp) and gains 8,230 shards.
- **Cross-stage.** A stage 9 player's TNT on a stage 3 garden deals 55 at the centre, the same as a stage 3 player's TNT. A stage 2 player's TNT on a stage 9 garden deals 44 against a 596 HP top fruit: 14 hits, so regen undoes it.

## 6. UI

Every element uses the kit in [[UI Kit Inventory]]: FredokaOne (`Font.new("rbxasset://fonts/families/FredokaOne.json")`), white text, black UIStroke, kit images. No emoji anywhere, in UI or in any string. **No new screen-space panel or HUD cluster is added**, so nothing can collide with the HUD zones. Everything new is either inside an existing component or drawn in the 3D world.

### 6.1 Shop cards (Weapons tab of the Shop)
Resolved in the [[Integration Review]] (conflict C5, decision D-022): the Shop has four tabs, **Potions, Weapons, King, Robux**. Today's Tools tab becomes the Potions tab ([[Potions]]), and every combat item, garden explosives and sea weapons, sits on the Weapons tab ([[Weapons and Artillery]] UI 1). One tab answers "what does the next rebirth give me to fight with".
- **Card art:** the Weapons-tab card from [[Weapons and Artillery]] UI 1: the kit rarity frame of the item's unlock tier (TNT UNCOMMON 136126447232134, Big TNT EPIC 134546200735728, Mega TNT MYTHIC 125641842226730, Nuke ETERNAL 122828478750063) and the kit Buy button `97067256017679` (shard crystal baked on its left).
- **Order:** one list for the whole Weapons page, sorted by unlock stage, explosives before sea weapons on a tie: Cherry Bomb and TNT (stage 2), Melon Mortar (3), Big TNT (4), Coconut Cannon (5), Mega TNT (6), Starfruit Barrage (7), Nuke (8). With 3 columns that is 3 rows. Each card keeps `order` in its config so the list never depends on key names.
- **Layout inside a card:** the Weapons-tab card layout (title 0.10, preview 0.36, stats line 0.58, owned line 0.66, Buy 0.82). The explosive stats line reads "Garden | Radius 10, 3 hits", "Garden | Radius 14, 2 hits", "Garden | Radius 20, 2 hits", "Garden | Whole garden", so it is clear these work on plots, while sea weapons read "Sea | ...".
- **Price text:** `Config.ShardPrice(N, rebirths)`, formatted with `Economy.Format` as today (`ShopMenu.lua:318`). It repaints when the `Rebirths` attribute changes. Buying still fires the existing `BuyTool` remote; sea weapon cards fire `WeaponRequest("buy", key)`.
- **States:**
  - Available: as today.
  - Locked: card tinted grey and the Buy text "Rebirth N" (the Weapons-tab locked look); pressing Buy only shows the refusal toast.
  - Nuke cooling down server-wide: the owned line reads "Next nuke in 6:12", from the `NukeReadyAt` attribute on `workspace`.
  - Not enough shards: unchanged (server toast "Not enough shards!").

### 6.2 Inventory
No change. Explosives get the existing tool cards (`StarterPlayer.StarterPlayerScripts.StoredFruitsMenu.lua:384-443`).

### 6.3 Carry row and carry info (bottom centre, existing)
- Drop button `104911876480159` (168x56) arms the explosive. It already shows for `kind = "place"` (`StarterPlayer.StarterPlayerScripts.GardenClient.lua:225-233`).
- Name bar `123470450245311` shows the item name. Rate bar `120297790302429` shows the tool's `note` (`GardenClient.lua:250`), the same text as the stats line.
- While `ExplosiveReadyAt` is in the future, the rate bar reads "Ready in 4s" instead. `refresh()` already runs every 0.2 s (`GardenClient.lua:430-435`).

### 6.4 Lock button alert (top centre, existing pills)
- A copy of the kit alert badge `79539007098845` (31x35), placed in Studio as `TopButtons.LockButton.AlertBadge`, with a `Pop` UIScale like the Upgrade badge (`StarterPlayer.StarterPlayerScripts.AlertBadges.lua:28-45`).
- It sits on the Lock pill's **outer** top corner, the corner away from the Upgrade pill, so it cannot touch the Upgrade pill or the `LockTimer` label under the Lock pill.
- Shown while the player's `IncomingExplosive` attribute is in the future. Hidden at the blast or the cancel.

### 6.5 Toasts (existing GardenToast)
All messages go through the existing `Notify` (red, drops from the top) and `Notice` (green, rises from the bottom) remotes (`ServerScriptService.GardenService.lua:66-70`). New Config strings, all plain ASCII:

| Key | Text | Channel, to whom |
|---|---|---|
| `ExplosiveIncoming` | "%s placed a %s in your garden! Lock to block it." | Notify, owner |
| `ExplosiveArmed` | "%s armed. %ds!" | Notice, attacker |
| `ExplosiveHit` | "Hit %d fruit. %d stunned." | Notice, attacker |
| `ExplosiveVictim` | "%s's %s hit %d of your fruit. %d stunned." | Notify, owner |
| `ExplosiveBlocked` | "Blocked by a Lock. It went back in your bag." | Notify, attacker |
| `ExplosiveDefended` | "Your Lock blocked %s's %s!" | Notice, owner |
| `ExplosiveCancelled` | "That garden is gone. It went back in your bag." | Notify, attacker |
| `ExplosiveOwnPlot` | "You can't blow up your own garden!" | Notify |
| `ExplosiveLockedStage` | "Unlocks at rebirth %d!" | Notify |
| `ExplosiveNewGarden` | "That garden is protected until its first rebirth!" | Notify |
| `ExplosiveFresh` | "That garden was just planted. Try again in %ds." | Notify |
| `ExplosiveCooldown` | "Explosives ready in %ds." | Notify |
| `ExplosiveOneAtATime` | "Wait for your last one to go off!" | Notify |
| `ExplosiveServerFull` | "Too many explosives out right now!" | Notify |
| `ExplosiveOwnerFighting` | "That player is in a royal fight. Try again later." | Notify, attacker (rules 9 and 15) |
| `ExplosiveNoPlot` | "You need a garden of your own first!" | Notify |
| `NukeServerCooldown` | "Next nuke in %s." | Notify |
| `NukeRecentlyHit` | "That garden was nuked recently!" | Notify |
| `NukeIncoming` | "NUKE incoming on %s's garden in %ds!" | Notify, every player |
| `NukeSurvivor` | "Your garden survived a nuke: +%s shards" | Notice, owner |
| `ExplosivesRefunded` | "Rebirth turned your explosives into %s shards." | Notice |
| `ToolOffPlot` (reworded, `ReplicatedStorage.GardenConfig.lua:1854`) | "Stand in another player's garden to use this!" | Notify |

`TNTDetonated`, `TNTNothing` and `TNTVictim` (`ReplicatedStorage.GardenConfig.lua:1855-1857`) are removed.

### 6.6 Fruit health bar (world, client-only)
- **Art:** the one shared bar pair, `UIKit.Images.BarTrack` and `UIKit.Images.BarFill` ([[King UI Overhaul]] assets A8 and A9, 336 x 54, made from CarryInfo.RateBar 120297790302429). Every bar in the expansion uses this pair: fruit health, ore node HP, boat HP, fighter HP and decree progress (decision D-027). Until they exist, the fallback uses the carry rate bar image `120297790302429` for both, the fill tinted and clipped. No code-drawn Frame plus UICorner bars.
- **Build:** one BillboardGui per visible bar, parented to PlayerGui with `Adornee = fruit`, the pattern EarnPopups uses (`StarterPlayer.StarterPlayerScripts.EarnPopups.lua:56-67`). Children: Track (ImageLabel, ScaleType Slice), FillClip (Frame with ClipsDescendants, width = HP / max), Fill (ImageLabel, full width, tinted). Pool of 40.
- **Size and place:** width `clamp(fruit.Size.X x 0.9, 2.2, 4.5)` studs, height 0.42 studs. `StudsOffsetWorldSpace = (0, fruit.Size.Y / 2 + 0.55, 0)`, so it spans 0.34 to 0.76 studs above the fruit's top. `StudsOffset = (0, 0, Config.NameTagForward)` (2), the same camera nudge the name tag uses. `MaxDistance` 60. `AlwaysOnTop` false. `LightInfluence` 0.
- **No overlap:**
  - The name tag sits at top + 1.75 with height 1.4, so its bottom edge is at top + 1.05 (`ReplicatedStorage.GardenConfig.lua:599-607`). The bar's top edge is at top + 0.76.
  - The bar hides while that fruit's E prompt is shown (PromptShown / PromptHidden, the same signals `GardenClient.lua:161-167` uses), so it never sits under the Roblox prompt.
  - Fruit with a visible bar show no "+$" earn popup. Their income still lands in the corner counter.
- **Fill colours, all from the kit rarity palette** (`ReplicatedStorage.GardenConfig.lua:582-592`): above 50% green (Uncommon 78, 194, 101); 25% to 50% orange-gold (Legendary 242, 171, 48); below 25% red (Mythic 232, 66, 60) (the same bands as boat HP and fighter HP, D-027); stunned grey (Common 151, 169, 189), refilling from 0 to full over the 20 s stun; shielded cyan (Astral 71, 222, 217), full, for 30 s. Cyan already means "protected" in this game: the Lock timer is cyan (`ReplicatedStorage.GardenConfig.lua:930`).
- **States:** hidden at full HP; shown when HP < max, while stunned and while shielded; hides 3 s after returning to full. The fill tweens 0.15 s on each change.

### 6.7 Damage numbers (world, client-only)
- Text "-98". Font `Config.FruitTextFont`, the same key the name tags and earn popups use (Highway today, `ReplicatedStorage.GardenConfig.lua:25`), so all text over fruit matches (see Open question 6). Colour red (Mythic 232, 66, 60), black stroke 2.5, max text 26.
- BillboardGui 3.0 x 1.0 studs, starting at top + 1.15, rising 2.0 studs over 0.9 s and fading over the second half, the same motion as earn popups (`EarnPopups.lua:69-84`).
- At most 12 per blast, nearest the camera first. Seen within 60 studs. None for nuke fallout ticks: there the bars visibly drain instead.

### 6.8 Warning circle and countdown (world, client-only)
- **Circle:** a flat Neon cylinder, red (Mythic 232, 66, 60), transparency 0.65, 0.12 studs thick, diameter 2R, at `plot.topY + 0.07`. CanCollide, CanQuery, CanTouch and CastShadow all off. One circle only, the size of the blast. It pulses (transparency 0.65 to 0.8) every 0.6 s, then every 0.2 s in the last second. The nuke circle covers the soil: radius = half the soil diagonal + 2.
- **Countdown:** BillboardGui 2.4 x 1.2 studs (3.6 x 1.8 for the Nuke) above the explosive. Fredoka, white, black stroke 3, max text 40. Whole seconds rounded up (3, 2, 1). Seen within 300 studs.

### 6.9 Stunned look (world, client-only)
`LocalTransparencyModifier = 0.45` on the fruit, and the `Stars-01` emitter from the `Anime` pack circling above it ([[Existing VFX and Props]]). No Highlight, because Highlights are capped at 24 and shared with the merge glow (`ReplicatedStorage.GardenConfig.lua:1887-1889`).

### 6.10 Nuke screen effect (client-only)
For players within 250 studs of the blast: a ColorCorrection flash (tint 255, 200, 150, brightness +0.15) that fades over 1.5 s, and a Blur of 8 that fades to 0 over 0.8 s. This is the Jailbreak pattern ([[Merge a Nuke - Explosives and Health Drain]]). No text on screen beyond the toast.

## 7. World and assets

### 7.1 Explosive models (to build in Studio)
Each template lives in `ReplicatedStorage.Tools` under its key. Each is one root BasePart with welded child parts, the shape potions use, because `equipTool` writes a cube Size only when `carrySize` is set (`ServerScriptService.GardenService.Tools.lua:146-151`). The look is clean and studded: Studs surfaces on the flat tops, the game's palette, no meshes.

| Key | Build | Parts | Carried size |
|---|---|---|---|
| TNT1 | The existing `ReplicatedStorage.Tools.TNT` cube, renamed. Keeps its current look. | 1 | 2.5 (`carrySize`) |
| TNT2 | Four TNT sticks in a 2 x 2 bundle, one dark band around the middle, Studs on top | 5 | about 3 studs tall, no `carrySize` |
| TNT3 | Nine TNT blocks in a 3 x 3 bundle, two bands, a bigger fuse cap | 11 | about 3.5 studs, no `carrySize` |
| Nuke | A studded bomb: cylinder body (Smoky grey), ball nose, four flat fins, one yellow band (Legendary 242, 171, 48) with two black stripes, Studs on the fins | 8 | about 3 x 3 x 4 studs, no `carrySize` |

When armed, the server places a fresh clone (anchored, CanCollide, CanQuery and CanTouch off) in a server folder `workspace.ArmedExplosives`. The carried one is destroyed by `releaseHold`, as today (`Tools.lua:275-276`).

### 7.2 VFX (already in the place)
All are in the staging area at y about -790 ([[Existing VFX and Props]]). Copies go into the shared `ReplicatedStorage.WeaponFx` folder that [[Weapons and Artillery]] creates (this build creates it if it comes first), so every client can clone them whether or not the staging area is streamed in, and no effect is copied twice.

| Use | Source |
|---|---|
| Classic explosion, every tier | A client-side Roblox `Explosion`, `BlastPressure = 0`, `DestroyJointRadiusPercent = 0`, `ExplosionType = NoCraters`, visual radius = min(R, 14) |
| Big TNT and Mega TNT extra burst | `vfx pack` > `Explosion` (shockwave, shards, sparks) |
| Fuse sparks while armed | `vfx pack` > `Projectile` (smoke and spark trail) |
| Nuke blast | `Big` > `Explosion-01`, plus `Big` > `Big-Crack-01` on the ground |
| Nuke cloud | `Big` > `Ball-01` on an invisible anchor that rises 30 studs over 3 s, then fades |
| Stunned fruit | `Anime` > `Stars-01` |
| Lock blocks a blast | `Anime` > `Shield-Break-01` at the explosive |
| Cancelled explosive fizzles | `Anime` > `Smoke-01` |

Particle budget: bursts only (`:Emit(n)`), at most 400 particles per blast. Burst size scales by tier: TNT 0.5x, Big TNT 0.75x, Mega TNT 1x, Nuke 1x of the pack's own counts.

### 7.3 Camera shake
A small client shake, not a library. Peak shake by tier: TNT 1.5, Big TNT 2.5, Mega TNT 3.5, Nuke 5. Roughness 10, fade-in 0 s, fade-out 1.5 s. These are the RbxCameraShaker "Explosion" preset values, with magnitude scaled down for the smaller tiers ([[Merge a Nuke - Explosives and Health Drain]]). Strength falls to zero at 4R from the blast (300 studs for the Nuke).

### 7.4 Sounds (to pick from Roblox's licensed library)
Picked the same way as the music in D-002, so they play in any experience and need nothing from Samarth.

| Sound | Volume | Heard within |
|---|---|---|
| Fuse hiss (loop, while armed) | 0.5 | 40 studs |
| Small boom (TNT) | 0.8 | 150 studs |
| Big boom (Big TNT, Mega TNT) | 1.0 | 250 studs |
| Nuke siren (loop, 10 s) | 0.6 | 600 studs |
| Nuke boom | 1.0 | 800 studs |
| Lock block (shield crack) | 0.7 | 80 studs |

## 8. Asset requests for Samarth

One optional image. No animations to publish, no Robux products.

**1. Fruit health bar.** No separate request any more. The fruit bar uses the shared bar pair made once for the whole expansion: [[King UI Overhaul]] assets A8 (`kit_bar_track.png`, 336 x 54) and A9 (`kit_bar_fill.png`, 336 x 54), both copied from CarryInfo.RateBar 120297790302429 (decision D-027). The earlier 256 x 32 "HealthBarTrack" / "HealthBarFill" request is withdrawn so you do not make the same bar four times ([[Integration Review]], conflict C2).

**2. (Optional) Nuke shop card art.** Only if you want the nuke card to show drawn art instead of the spinning model, the way the Potion of Aether card does (`ReplicatedStorage.GardenConfig.lua:1661-1665`).

> Make one transparent PNG, 512 x 512, for a Roblox shop card, matching the attached Potion of Aether card art in style (attach rbxassetid 123282319597102 as the reference): a chunky cartoon bomb built from Roblox-style studded plastic blocks, smoky grey body, rounded nose, four flat fins with studs, one yellow band with two black stripes, small lit fuse spark at the top, soft top-left lighting, thick dark outline. No text, no emoji, no background, centred with 10% padding.

**Robux products:** none. I recommend not selling anything that hurts other gardens or makes yours immune. See Open question 8.

## 9. Code plan

All modules follow the existing pattern: a ModuleScript that is `return function(G)`, required in order by `ServerScriptService.GardenService.lua`, taking helpers out of `G` and putting its own back (`ServerScriptService.GardenService.lua:88-98`).

### 9.1 New: `ReplicatedStorage.ExplosiveConfig` (ModuleScript, shared)
Shared by server and client so the shop, the bars and the server always agree. It holds every number in section 5:
- `HP(tier)`, `MaxHPFor(fruitName)` (tier from `FruitIndex.ByName`), `Stage(rebirths)` (uses `Config.StageFor` from [[Rarity Gating and Rebirth]] if present, else `math.min(rebirths, 8) + 1`).
- `Items[key] = { m, radius, fuse, cooldown, unlockStage, priceMerges, stats }`, plus `Nuke = { primary = 0.6, falloutPerTick = 0.1, falloutTicks = 4, serverCooldown = 600, plotCooldown = 1800 }`.
- `CentreDamage(key, effStage)`, `Falloff(d, R)` (calls `Ballistics.falloff` from [[Weapons and Artillery]] if that module exists, else the same formula), `Price(key, rebirths)` (uses `Config.ShardDropFor(stage)` from [[Rarity Gating and Rebirth]] if present, else the D-010 table 3, 7, 15, 32, 70, 155, 340, 748, 1646), `Unlocked(key, rebirths)`, `SurvivorBonus(stage)`.
- Timers: `RegenDelay = 16`, `RegenRate = 0.10`, `Stun = 20`, `Shield = 30`, `Fresh = 120`, `MaxArmed = 12`, `HeightBand = 12`, `OreDamageScale = 0.5`, `ZeroHpMode = "stun"`.

Kept out of `GardenConfig` on purpose: that file is already 111,613 characters, and a Script source caps at 200,000 (`ServerScriptService.GardenService.lua:96-97`). `Price` is a thin wrapper over `Config.ShardPrice(N, rebirths)` and `SurvivorBonus` over `Config.ShardReward(5, rebirths)` from [[Rarity Gating and Rebirth]] (D-017); this module keeps no copy of the D-010 table.

### 9.2 New: `ServerScriptService.GardenService.Explosives` (module)
Required right after Tools, at `ServerScriptService.GardenService.lua:491-493`, because it needs `stowTool`, `addTool`, `toolsFolder` and `plotUnder`. Functions it puts into G:

| Function | Job |
|---|---|
| `G.placeExplosive(player, name, spec)` | Rules 7 to 14. Runs every gate in the order listed, sends one toast on the first failure, keeps the item in hand on refusal. On success: `releaseHold(player, true)`, clears `CarryingTool` (as `Tools.lua:275-276` does), clones the template into `workspace.ArmedExplosives`, stores `{ id, key, attacker, attackerUserId, attackerStage, ownerUserId, plot, pos, detonateAt }`, sets the cooldown, drops the attacker's own Lock (writes `plot.lockedUntil = now` on their plot and `G.lockReadyAt[player] = now + Config.LockCooldown`, exported at `RebirthLock.lua:244-245`), sets `IncomingExplosive` on the owner, fires `ExplosiveFx("armed", ...)`, and schedules `detonate`. |
| `detonate(id)` (local) | Rule 15 re-checks, then rules 16 to 21. Applies damage with `G.damageFruit`, fires `ExplosiveFx("blast", ...)`, sends the hit toasts, and for a nuke starts the fallout and pays the survivor bonus when it ends. On a failed re-check it calls `addTool` for the attacker and fires `ExplosiveFx("cancel", ...)`. |
| `G.damageFruit(fruit, amount, now)` | Subtracts HP, records the last-hit time, adds the fruit to the `damaged` set (weak keys), writes the `HP` attribute, or stuns at 0. |
| `stun(fruit, now)` (local) | Sets `HP = 0`, `StunnedUntil = now + 20`, `Busy = true`. Reusing `Busy` means `pickUp` (`Carrying.lua:169`), `merge` (`Merging.lua:167`), the client prompt (`GardenClient.lua:295`) and the blast loop all refuse it with no change. |
| `G.blast(args)` | The one entry point for fruit damage: `{ centre, radius, m, attacker, attackerStage, plot }`. Runs rules 16 to 20. Explosives use it; any future system that can hurt fruit must call it. [[Weapons and Artillery]] does not hurt fruit today (its rule 20). |
| `G.registerBlastTarget(kind, provider)` | Lets [[Underground and Ores]] register `"ore"`. A provider is `{ collect(centre, radius, player) -> list, damage(target, amount) }`. Boats are not a provider: boat damage goes through `G.damageBoat` ([[Boats and Swimming]], [[Weapons and Artillery]]). |
| `G.refundExplosives(player)` | Rule 4. Sums `count x Price(key, rebirths)` over the explosive entries in `toolsFolder(player)`, adds it with `setShards`, destroys the entries, returns the total. Yield-free. |
| `G.isStunned(fruit)` | `fruit:GetAttribute("StunnedUntil") ~= nil`. |
| `G.explosivesForget(player)` | Clears that player's cooldown and armed record on leave. |

Its own loop, `task.spawn` with the body wrapped in `pcall` like Ambient's loops (`ServerScriptService.GardenService.Ambient.lua:403-404`), runs once per second. It walks only the `damaged` set: it drops fruit that were destroyed, skips carried ones (`Parent ~= fruitFolder`), ends stuns (`HP` nil, `StunnedUntil` nil, `Busy` nil, `ShieldUntil = now + 30`, but only if the fruit is still in the fruit folder), and regenerates fruit that have not been hit for 16 s, clearing `HP` to nil at full.

### 9.3 Changes to existing scripts

| File and line | Change |
|---|---|
| `ReplicatedStorage.GardenConfig.lua:1616-1626` | Replace `TNT` with `TNT1`, `TNT2`, `TNT3`, `Nuke`: `display`, `kind = "place"`, `explosive = true`, `order`, `note` (the stats text), `carrySize = 2.5` on TNT1 only. No `price` field: the price comes from `ExplosiveConfig.Price`. |
| `ReplicatedStorage.GardenConfig.lua:1631-1704` | Add `order = 10, 11, ...` to each potion so the shelf keeps a fixed order. |
| `ReplicatedStorage.GardenConfig.lua:1849-1857` | Remove `TNTDetonated`, `TNTNothing`, `TNTVictim`; add the strings in 6.5; reword `ToolOffPlot`. |
| `ServerScriptService.GardenService.lua:59-86` | Add `local explosiveFxRemote = remote("ExplosiveFx")` and put it into G at `:101-115`. |
| `ServerScriptService.GardenService.lua:491-493` | `require(script.Explosives)(G)` right after Tools. |
| `ServerScriptService.GardenService.lua:631-684` | Studio-only dev doors next to `DevSetShards` (`:653`): `DevSetRebirths` (calls `publishRebirths`, `Progression.lua:95-101`; the same door [[Weapons and Artillery]] proposes, built once) and `DevExplosivesSelf` (lets a Studio tester target their own plot). |
| `ServerScriptService.GardenService.lua:703-735` | In `teardownPlayer`, after the save (keeping the ordering rule at `:704-706`), call `G.explosivesForget(player)`. |
| `ServerScriptService.GardenService.Tools.lua:91-112` (`buyTool`) | For `spec.explosive`: refuse if `not ExplosiveConfig.Unlocked(name, rebirths)` (toast `ExplosiveLockedStage`), and charge `ExplosiveConfig.Price(name, rebirths)` instead of `spec.price`. |
| `ServerScriptService.GardenService.Tools.lua:244-310` (`useTool`) | After the consume branch (`:250-253`), add `if spec.explosive then return G.placeExplosive(player, name, spec) end`. Delete the old TNT body (`:255-310`). End with a `warn` for an unknown kind, so a new kind no longer falls into explosive code (today every non-consume tool runs TNT, `Tools.lua:250-310`). |
| `ServerScriptService.GardenService.Tools.lua:163-168`, `:314-317` | Export `plotUnder` into G (`G.plotUnder`). |
| `ServerScriptService.GardenService.FruitFactory.lua:39-45` (`plotBase`) | Skip fruit with `StunnedUntil`. Live income and offline pay both use `plotBase`, so they stay in step (`Persistence.lua:191`). Add an optional second argument `includeStunned` for the Empire Power line below. |
| `ServerScriptService.GardenService.FruitFactory.lua:87-94`, `:371-376` | Export `clampToPlot` into G for the blast centre. |
| `ServerScriptService.GardenService.Merging.lua:477-481` and `:494-505` (`mergeAllOn`) | Skip fruit with `StunnedUntil` when counting and when consuming. (Merge All ignores `Busy` today, see [[Code Map]] gotchas.) |
| `ServerScriptService.GardenService.RebirthLock.lua:71-73` | If the held item is a tool, call `G.stowTool(player)` instead of `releaseHold(player, true)`, so a carried explosive or potion is not silently destroyed. [[Rarity Gating and Rebirth]] rule 15 asks for the same change; build it once. |
| `ServerScriptService.GardenService.RebirthLock.lua:196-226` (`lockPlot`) | Skip the "standing on your own plot" check while the player's `IncomingExplosive` attribute is in the future (rule 32, D-034). Every other check stays (loaded, has a plot, not already locked, not cooling down). `LockButton.lua:30-49` needs no change: it only checks the timers before firing `Lock`. |
| `ServerScriptService.GardenService.RebirthLock.lua:113-118` | Inside the yield-free block and **before** `publishRebirths` (`:117-118`), so the old stage's price is used: `local refund = G.refundExplosives and G.refundExplosives(player) or 0`. After the notice at `:144-145`, send `ExplosivesRefunded` if `refund > 0`. |
| `ServerScriptService.GardenService.Ambient.lua:465-469` | Publish a second attribute `EmpirePower = (plotBase(plot, true) x (1 + OreBonus) + petBase) x cashMultiplier(player)` next to `CashPerSecond`: the same income formula as the live tick ([[Underground and Ores]] rule 21, [[Pets]] P27, D-018), but counting stunned fruit, so a stun cannot swing the throne (see Royal Combat in section 13). |
| `ServerScriptService.GardenService.Crown.lua:296-298`, `ServerScriptService.GardenService.CrownDecree.lua:260` | Read `EmpirePower` instead of `CashPerSecond`. |
| `ServerScriptService.GardenStore.lua:73-76` | Add `tnt = 2500` to `RETIRED_PRICES`. |
| `ServerScriptService.GardenStore.lua:358-380` | Add a branch like the Plot Alarm one (`:368-371`): a `tools.TNT` count adds `RETIRED_PRICES.tnt x count` to `refund`. The existing refund notice pays it (`Persistence.lua:243-253`). |
| `StarterPlayer.StarterPlayerScripts.ShopMenu.lua:381-386` | Sort by `order`, then by key. |
| `StarterPlayer.StarterPlayerScripts.ShopMenu.lua:197-376` | Explosive cards are built on the Weapons page by the same card builder as the sea weapons ([[Weapons and Artillery]] code plan), with the explosive stats line, price from `Config.ShardPrice`, the locked and nuke-cooldown states, a repaint on the `Rebirths` and `Shards` attributes and on `workspace` `NukeReadyAt`, and `BuyTool` on press. |
| `StarterPlayer.StarterPlayerScripts.GardenClient.lua:246-251` | For an explosive tool with `ExplosiveReadyAt` in the future, show "Ready in %ds". |
| `StarterPlayer.StarterPlayerScripts.EarnPopups.lua:93-95` | Skip fruit that have an `HP` or `StunnedUntil` attribute (rule in 6.6). |
| `StarterPlayer.StarterPlayerScripts.AlertBadges.lua:28-30` | Add `lockBadge` (`TopButtons.LockButton.AlertBadge`), shown while `IncomingExplosive` is in the future; re-check on the attribute change and with a `task.delay` to the deadline. |
| `ServerStorage.DevPanel.Server.lua:197-230` (`handleAction`) and its snapshot rows (for example `:141-148`) | New Studio actions: give 3 of each explosive, damage my plot 50%, stun my plot, reset my explosive cooldown. New snapshot row: damaged and stunned fruit on my plot, with their HP. |

### 9.4 New client script: `StarterPlayer.StarterPlayerScripts.FruitHealthClient`
- Watches `fruitFolder` ChildAdded and each fruit's `AttributeChanged` for `HP`, `StunnedUntil` and `ShieldUntil`. Event-driven, no per-frame loop for bars.
- Draws bars (6.6), damage numbers when `HP` drops (6.7), the stunned look (6.9).
- Listens to `ExplosiveFx`: draws circles and countdowns (6.8), plays VFX, sounds and shake (7.2 to 7.4) and the nuke screen effect (6.10). Ignores events farther than 300 studs (800 for nuke sound and toast-side effects).
- Hides a fruit's bar on `PromptShown` and shows it again on `PromptHidden`.

### 9.5 Remotes

| Name | Direction | Payload | Server validation |
|---|---|---|---|
| `Drop` (existing) | client to server | none | All of rules 6 to 14 in `placeExplosive`. The position is computed by the server. |
| `BuyTool` (existing) | client to server | tool key (string) | Key must be in `Config.Tools` (`Tools.lua:93-94`); explosive must be unlocked for the buyer's rebirths; price from `ExplosiveConfig.Price`; shards checked (`Tools.lua:100-104`). |
| `ExplosiveFx` (new RemoteEvent) | server to all clients | `(event, data)`. `"armed"`: `{ id, key, pos, radius, detonateAt, plot, by }`. `"blast"`: `{ id, key, pos, radius }`. `"fallout"`: `{ plot, endsAt }`. `"cancel"`: `{ id, reason }` where reason is `"locked"`, `"gone"`, `"fresh"`, `"stage"` or `"fighting"`. | Server-only sender. Clients check types and ignore bad data. About 100 bytes per event. |

### 9.6 Save document
- **No new fields.** HP, stuns, shields, cooldowns and armed explosives are runtime only, as the platform note recommends ("Health from TNT damage can live in server memory and reset on rejoin", [[Roblox Platform Constraints for the Expansion]]).
- **Changed contents of `tools`:** new keys `TNT1`, `TNT2`, `TNT3`, `Nuke` (whole-number counts, default absent = 0), saved and checked by the existing tools path (`ServerScriptService.GardenService.Persistence.lua:63-67`, `:221-225`; `ServerScriptService.GardenStore.lua:358-367`).
- **Migration:** the old `TNT` key is refunded at 2,500 shards each on first load and dropped. No schema bump is needed (`ServerScriptService.GardenStore.lua:8-12`).

### 9.7 Attributes

| On | Attribute | Type | Written by | Read by |
|---|---|---|---|---|
| fruit | `HP` | number, or nil at full | Explosives | FruitHealthClient, EarnPopups |
| fruit | `StunnedUntil` | server time, or nil | Explosives | `plotBase`, `mergeAllOn`, FruitHealthClient, EarnPopups |
| fruit | `ShieldUntil` | server time, or nil | Explosives | Explosives, FruitHealthClient |
| fruit | `Busy` | true while stunned (reused) | Explosives | existing readers |
| player | `ExplosiveReadyAt` | server time | Explosives | GardenClient |
| player | `IncomingExplosive` | server time of the next blast on your plot, or nil | Explosives | AlertBadges |
| player | `EmpirePower` | number | Ambient | Crown, CrownDecree |
| workspace | `NukeReadyAt` | server time | Explosives | ShopMenu |
| armed model | `Key`, `DetonateAt`, `OwnerUserId` | string, number, number | Explosives | debugging only |

## 10. Edge cases and abuse

| Case | What happens |
|---|---|
| Attacker leaves during the fuse | The server timer still runs and the blast happens. Toasts to the attacker are skipped. The explosive is not refunded: it was placed. |
| Owner leaves during the fuse | The plot is released (`ServerScriptService.GardenService.lua:450-480`). The re-check finds a different owner or none, cancels, and returns the explosive to the attacker. |
| A new player claims that plot during the fuse | The re-check compares UserIds, so it cancels and returns the explosive. |
| Owner rebirths during the fuse | Their garden becomes fresh, so the re-check cancels and returns the explosive. |
| Attacker rebirths during the fuse | The blast uses the attacker stage saved at placement. The armed one is not refunded. |
| Rebirth while carrying an explosive | `stowTool` puts it in the bag first, then the refund pays for it (code plan, `RebirthLock.lua:71-73`). |
| Death or despawn while carrying | Existing silent drop puts it back in the bag (`Carrying.lua:297-306`). |
| Server shutdown during a fuse | That one explosive is lost. It is at most 10 s of one item, on a rare event. Accepted. |
| Spamming Drop | One armed explosive per player, a server cooldown, and a server cap of 12. |
| Choosing the blast point | Impossible. The Drop remote has no payload and the server computes the point. |
| Teleport or speed exploit onto a plot | Same exposure as today's TNT. The owner still gets the toast, the "!" and 3 to 10 s to Lock. |
| Hoarding cheap explosives for a later stage | Rebirth refunds them at the price paid, so no profit. |
| Two friends farming the nuke bonus | Bonus at most 8,230 shards; nuke at least 190,000. They lose. |
| Rejoining to reset the personal cooldown | Rejoining takes longer than the 6 to 15 s cooldown. The nuke cooldown is per server, so rejoining does not help there. |
| Server-hopping to nuke many servers | Each nuke costs at least 190,000 shards. Accepted. |
| Veteran harassing a new player | Stage 1 is immune. Damage is capped at the victim's own stage. Each fruit can be stunned at most once per 50 s. Each stun costs the victim 20 s of that fruit's income. The Lock blocks everything. |
| Owner is AFK | They can be hit, with the same caps. The worst case is a stream of 20 s stuns. |
| Owner is underground, at sea, in the forest or in the sky | Their surface fruit can still be hit. While an explosive is armed on their plot they can raise the Lock from wherever they are (rule 32, D-034). |
| Plotless players (server of 50, 12 plots) | Cannot be targeted and cannot attack (rule 8). |
| Merge All during fallout | Skips stunned fruit (code plan). |
| Clear during a stun | Allowed: the owner removing their own fruit. |
| Storing a damaged fruit | It comes back at full HP. Storing is slow and limited, so this is allowed. |
| Fruit mid-drop or mid-merge | `Busy`, so skipped by every blast, as today (`Tools.lua:280-282`). |
| Royal arena | Placement refused with `InArena`. A garden whose owner is `InArena` cannot be targeted, and an armed explosive is cancelled and returned if its owner enters the arena before the blast (rules 9 and 15, D-035). Explosives never touch fighters or knights (`BlastPressure = 0`). |
| Mobile | Same Drop button. Bars and circles are sized in studs. Toasts are existing. Particle counts respect the mobile emitter cap of 100 per second ([[Roblox Platform Constraints for the Expansion]]). |
| Invisible attacker (Invisibility Potion) | The circle, the countdown and the toast with the attacker's name all still show. |
| Kids and Select rating | No player damage, no realistic injury, cartoon explosions. Keeps the "Mild" rating ([[Roblox Platform Constraints for the Expansion]]). |

## 11. Performance

Budgets from [[Roblox Platform Constraints for the Expansion]]: about 50 KB/s download per client, 16.67 ms per frame, at most 400 particles per second per emitter (100 on mobile), Highlights capped near 31 (the game caps itself at 24).

- **Parts.** At most 12 armed explosives per server, at most 11 parts each: 132 parts worst case. Each warning circle is 2 client-only parts. Fruit get no new parts.
- **Server work.** A blast walks `fruitOfPlot` once (it scans the shared fruit folder, at most 12 x 35 = 420 fruit) and damages at most 35. The 1 Hz loop walks only damaged fruit, typically under 50. Nuke fallout is 4 ticks over at most 35 fruit.
- **Network.** One `ExplosiveFx` event per arming and per blast, about 100 bytes, sent to everyone. Each damaged fruit replicates one `HP` change per hit and at most one per second while healing. Worst case of about 100 healing fruit is about 3 KB/s, well under 50 KB/s. No server tweens, which replicate every frame.
- **Client per-frame work.** None for bars (event-driven). Camera shake runs on RenderStepped only while active (at most 1.5 s). Countdown text updates once per second per armed explosive.
- **Client objects.** At most 40 pooled bar billboards (4 instances each) and 24 pooled damage numbers. No Highlights.
- **Particles.** Bursts only, at most 400 per blast, scaled by tier.

## 12. Test plan

Set `workspace.StudioNoSave = true` first, or playtests write the real save (`ServerScriptService.GardenStore.lua:572-575`). Use Studio's "Clients and Servers" test with 2 players, called A (attacker) and B (owner). Give shards with the existing `DevSetShards` door and set rebirths with the new `DevSetRebirths` door. For solo checks, the Studio-only `DevExplosivesSelf` door allows targeting your own plot.

| Rule | How to check |
|---|---|
| 1, 2 | A at 0 rebirths: all four cards say "Rebirth N to unlock"; Buy is refused with the toast. Set A to 1, 3, 5, 7 rebirths: TNT, Big TNT, Mega TNT, Nuke unlock in turn. |
| 3 | At each stage 2 to 9, the card prices match table N4 exactly. |
| 4 | A at stage 4 owns 2 TNT and 1 Big TNT (2 x 480 + 1,300 = 2,260). Rebirth: shards rise by exactly 2,260, the tool entries are gone, the toast shows. Repeat while holding one in hand. |
| 5 | Load a save that holds `tools.TNT = 3`: +7,500 shards, refund toast, no `TNT` entry left. |
| 6 | A dies while carrying TNT: it is back in the bag, nothing exploded. |
| 7, 8 | A presses Drop on their own plot, on the plaza, and on a plot with no owner: each is refused with the right toast, and the item stays in hand. A plotless player is refused. |
| 9 | B at 0 rebirths: refused. B just joined (under 120 s): refused with seconds left. B locked: refused. Nuke twice within 600 s: second refused. |
| 10 | A drops TNT, then drops another within 6 s: refused with "ready in". Two armed at once: refused. |
| 11 | The circle centre is 3 studs in front of A, on B's soil, inside the soil edges. |
| 12 | A has their own Lock up, then places on B: A's dome drops within 0.15 s; A's Lock recharge shows 30 s. |
| 13 | Time the fuses with the countdown: 3, 3, 4, 10 s. |
| 14 | B gets the red toast and the "!" on the Lock pill; the "!" clears at the blast. |
| 15, 32 | B raises the Lock during the fuse: nothing is damaged, A gets the item back and the "blocked" toast, B gets the "defended" toast. B leaves during the fuse: A gets the item back. |
| 16 | Put a fruit of the neighbour plot within R of the blast (place the blast near the shared edge): it takes no damage. Carry a fruit during the blast: it takes no damage. |
| 17 | Stand A and B inside the blast: no push, no health loss. |
| 18, 19 | Read HP from the new dev panel row. Both players at stage 3: fruit at 0, 5 and 10 studs from a TNT take 55, 36 and 17. |
| 20 | A at stage 9 and B at stage 3: TNT deals 55 at the centre. Swap stages: TNT deals 44. |
| 21 | Nuke on B: all fruit drop to 40% at once, then 30, 20, 10, 0 at 1-second steps, and all are stunned. Repeat and Lock at the 2-second mark: fruit stop at 20% and none are stunned. |
| 22, 23 | Each tier's fruit shows the HP in N1 when first damaged. A newly spawned, merged, restored or un-stored fruit has no `HP` attribute. |
| 24, 25 | Damage a fruit, merge it with another: the new fruit is full. Damage a fruit, store it, take it out: full. Damage a fruit, rejoin: full. Damage a fruit, carry it 30 s: same HP when dropped. |
| 26 | Damage a fruit, wait: no change for 16 s, then +10% per second until full; the bar hides 3 s later. Then chain two Mega TNT centre hits 15 s apart on a same-stage top fruit: it is stunned at the second blast. |
| 27 | A stunned fruit: no E prompt, Merge All skips it, the income tick drops by its $/s (check the dev panel's last rate), Clear can remove it, the plot capacity still counts it. |
| 28 | After 20 s: full HP, cyan bar for 30 s, a blast in that window does nothing to it. |
| 30, 31 | Covered in 1, 2 and 9. |
| 33 | After hits, A's cash and shards are unchanged. After a nuke, B gets exactly the N7 bonus. |
| 34 | With the Underground module in place: press Drop with TNT in A's own mine and in B's mine. | Both refused with "No explosives in the mine."; the TNT stays in hand. |
| 35 | Fire `Drop` and `BuyTool` from the command bar with bad keys and while not loaded: all refused, nothing changes. |
| UI no-overlap | Damage a fruit and walk up to it: the bar hides when the prompt shows, and the name tag never touches a bar. Earn popups do not appear over damaged fruit. Check the shop explosive card on a 1600x900 window and a phone-size window: title, preview, stats line, Owned line and Buy do not overlap. The Lock "!" does not touch the Upgrade pill or the Lock timer. |

## 13. Dependencies and conflicts

| System | How it interacts |
|---|---|
| [[Rarity Gating and Rebirth]] | **Depends on it.** Uses its stage helper `Config.StageFor(r)` and its shard drop `Config.ShardDropFor(stage)` (D-010) for prices and unlocks. Adds the explosive refund inside `doRebirth`, before the rebirth count goes up. Shares its fix for a tool held at rebirth (its rule 15). **Conflict to resolve:** D-009 and its rule 15 say rebirth keeps tools; this note refunds explosives at full price instead (potions are still kept). See Open question 5. The protection rule "stage 1 cannot be targeted" uses its stage definition. Fruit above the new caps in old saves simply use their own tier's HP. |
| [[Bats]] | Bats never damage fruit HP. Explosives never push players, so the two never stack into a combo. The Lock blocks explosives only; whether it blocks bats is the Bats note's call. Bats and explosives both go through `useTool`, which gets explicit branches here, so a bat kind must not fall into explosive code. |
| [[Weapons and Artillery]] | **Owns** the sea weapons (Cherry Bomb, Melon Mortar, Coconut Cannon, Starfruit Barrage). They never damage fruit (its rule 20) and only fire in the open-water Combat Zone, so the two systems never hit the same thing. If a weapon ever does hurt fruit, it must call `G.blast` so the same HP, stun, Lock, fresh-garden and effective-stage rules apply. **Shared:** the falloff `Ballistics.falloff` (adopted here), the VFX folder `ReplicatedStorage.WeaponFx`, the Studio door `DevSetRebirths`, and the price form `N x shards per merge`. Unlocks interleave, so every rebirth opens one combat item: weapons at stages 2, 3, 5, 7; explosives at 2, 4, 6, 8. Its Weapons tab and this note's Tools-page cards are separate pages, so they cannot overlap. |
| [[Underground and Ores]] | Resolved in the [[Integration Review]] (C11, D-031): no explosives in any mine in v1, matching its rule 27. The `"ore"` provider hook stays for a later ore band. **Needs from it:** the Lock must be raisable from inside your own mine, or an owner below ground cannot answer an attack. Today `standingOn` uses a 12-stud band (`FruitFactory.lua:79-85`). |
| [[Pets]] | Explosives never damage pets. Roblox bans animal abuse at every rating, and whether cartoon pets taking damage counts is not confirmed ([[Roblox Platform Constraints for the Expansion]]). Optional for the Pets note: pets walk away from a warning circle. |
| [[Boats and Swimming]] | Placed explosives cannot target boats (they need a garden under you). Boat damage comes only from [[Weapons and Artillery]], through `G.damageBoat`. |
| [[Potions]] | No potion changes HP in v1. The Invisibility Potion does not hide the circle or the toast. Potions are still kept on rebirth; only explosives are refunded. Explosives moved off the old Tools tab to the Weapons tab; the old Tools tab is now the Potions tab (D-022). |
| [[Plot Overhaul and Environment]] | The circle sits on `plot.topY` and the blast centre is clamped to the soil part, so the soil must stay one flat part (`PlotRegistry.lua:21-30`). Decor must not go in the fruit folder or be named so `buildFence` deletes it (`Upgrades.lua:168-173`). New decor near the soil should leave the soil surface clear so the circle is visible. |
| [[King UI Overhaul]] | No royal screen changes here. All new text uses the existing toasts, so if that note restyles GardenToast, explosive messages follow. The stats line and Lock badge follow the same kit. |
| [[Royal Combat]] | Explosives are refused for anyone with `InArena` and never touch fighters or knights. **Conflict found:** the throne's Empire Power reads `CashPerSecond` (`Crown.lua:296-298`), and so does the Empire Build decree (`CrownDecree.lua:258-262`). A nuke would lower a rival's number for 20 s. The code plan adds an `EmpirePower` attribute that ignores stuns. |

## 14. Open questions for Samarth

Each has a default, so building can start without waiting.

1. **At 0 HP, stun or destroy?** Default: **stun** (20 s, then full health). Reasons in section 4E. A `ZeroHpMode` setting can switch to destroy later.
2. **Should explosives hit other players' gardens at all, or only PvE targets and boats?** Default: **yes, other gardens**, with every protection in section 4F. It is the live behaviour today and what "your fruits" in the brain dump points to.
3. **Explosives unlock at the first rebirth** (today anyone can buy TNT at 0 rebirths)? Default: **yes**, which also makes stage 1 a safe zone for new players.
4. **Currency: shards scaled by stage, or cash?** Default: **shards** (`N x ShardsPerMerge`). Cash is what blocks a rebirth, so cash prices would lengthen stages in a way the model never tuned for. Shards already pay for every tool and need late-game sinks ([[Stage Model]]).
5. **Rebirth refunds explosives instead of keeping them** (an exception to D-009)? Default: **refund at full price** (logged as D-020; [[Rarity Gating and Rebirth]] rule 15 now says the same). Damage grows with your stage, so keeping cheap early TNT for a later stage would be a free upgrade. Nothing is lost.
6. **Font for numbers over fruit.** Fruit name tags and "+$" popups use Highway on purpose (`ReplicatedStorage.GardenConfig.lua:20-25`). Default: damage numbers use the same key, so all over-fruit text matches. If you want Fredoka everywhere, change `Config.FruitTextFont` once and all three switch.
7. **Nuke delivery.** Default: **carried and placed** like TNT, with a 10 s warning. The alternative is choosing a garden from a map and air-dropping it, which needs a new targeting screen.
8. **Robux.** Merge a Nuke sells "x2 Nuke Health" for 199 Robux. Default: **sell nothing** that hurts gardens or protects them.
9. **Explosives on your own ore nodes.** Default changed in the [[Integration Review]]: **not in v1** (D-031), matching [[Underground and Ores]] rule 27. The v1 mine fills in about 3 minutes with the pickaxe, so TNT there adds nothing. Revisit with the second ore band.
10. **Should the Lock grow with rebirths** (Merge a Nuke's shield does)? Default: **no**, keep 60 s and 30 s. With stun-only damage, a longer shield is not needed.
11. **Shop layout.** Default changed in the [[Integration Review]]: **the Weapons tab**, ordered by unlock stage with the sea weapons (D-022). The old Tools tab becomes the Potions tab, so the Shop keeps four tabs.
12. **Do damaged (not stunned) fruit earn less?** Default: **no**, full income until 0 HP. Simple to read: only the stun costs money.
13. **Nuke survivor bonus** (5 merges' worth of shards to the victim). Default: **on**. It turns the worst moment into a small win and cannot be farmed.
14. **Health bar art.** Default: the shared bar pair A8 and A9 from [[King UI Overhaul]] (D-027). Until then, the carry rate bar art stands in.
