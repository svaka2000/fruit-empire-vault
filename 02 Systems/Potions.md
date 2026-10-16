---
tags: [system, design]
date: 2026-10-04
status: proposed
depends_on: [Rarity Gating and Rebirth, Boats and Swimming, Underground and Ores]
---
# Potions

Source of truth: [[Brain Dump 2026-10-04]]. Economy backbone: [[Stage Model]]. Decisions already made: [[Decisions Log]]. UI rules: [[UI Kit Inventory]].

## Why

Samarth's own lines that drive this system:

> "also add more potions cus these r too little."

> "otherwise they have to swim through it, swimming fatigues the player, to swim super fast they can drink an aqua affinity potion. otherwise they can buy a boat for 1000 shards."

> "this will give you acccess to ores. you can mine and merge ore to get better ore"

> "everything needs to correspond with the stage of the game a player is in."

> "all the UI should never overlap and also everything should be super clean."

From the same day's follow-ups (Brain Dump, "Follow-ups" section):

> "only add the potions it makes sense to add… think about this smartly", "dont overdo anything", "but still make them extremely high quality", "nobody would pay for a potion of decaying".

What the system is for. Potions are short, timed boosts that a player buys with shards and drinks on themselves. They exist to solve a real need the player has right now: crossing the sea, getting to the sky, falling safely, growing rarer fruit sooner. This spec adds only the two potions that the new systems create a need for, re-prices every potion so it costs the same share of a player's shard income at every stage, and cleans up how potions are bought, drunk and shown.

### Words used here

- **Stage**: the newest rarity a player has unlocked, from 1 (Common, 0 rebirths) to 9 (Transcendent, 8 rebirths). Defined in [[Stage Model]].
- **D (stage shard unit)**: the Stage Model's "Shards per merge into this rarity" column (decision D-010): 3, 7, 15, 32, 70, 155, 340, 748, 1646 for stages 1 to 9. It is what one frontier merge pays, so it grows with the stage at the same pace as shard income.
- **U (unit price)**: 25 x D. The price of a standard potion. Every potion costs a whole or half multiple of U.
- **Effect**: a timed buff. The server keeps an absolute end time and publishes `<Effect>EndsAt` and `<Effect>Span` player attributes (`ServerScriptService.GardenService.Tools.lua:209-223`).
- **Effect badge**: the rounded pixel-art square that shows a running effect, with a draining bar and a countdown under it (`StarterPlayer.StarterPlayerScripts.GardenClient.lua:464-547`).
- **Effect block**: the group of effect badges at the bottom right of the screen (today's `EffectStack` frame).
- **Spawn ceiling**: the best fruit a plot can grow in the current stage (Stage Model column "Spawns up to"). The top 2 fruits of the newest rarity never spawn (D-007).
- **Swimming**: the state the [[Boats and Swimming]] system puts a player in while they are in the sea. This spec reads it from that note's character attribute `SwimState` (swimming = anything but `"Dry"`).

### Which potions, and why (the "think about this smartly" pass)

Today's six potions all stay. Removing one would delete every player's stock on load with no refund (`ServerScriptService.GardenStore.lua:372-375`). A candidate is added only if a new system creates a need for it.

| Candidate | Verdict | Reason |
|---|---|---|
| Aqua Affinity | **Add (v1)** | Samarth asks for it by name. Swimming fatigue (from [[Boats and Swimming]]) creates the need. It is the cheap early way across the sea. |
| Haste | **Add, ships with the second ore band (phase 2)** | Mining is a new activity. A faster pickaxe is a shortcut players want (Minecraft's Haste, Mining Simulator's speed boosts, [[Mining and Ore Merging - Reference Games]]). Only Underground owners can buy it. The v1 mine (Coal and Iron only) fills in about 3 minutes, so Haste waits for the second ore band, when mining is long enough to want it (decision D-032, [[Integration Review]]). |
| Night Vision | Reject | Nights are 30 s of each 330 s day cycle (`ReplicatedStorage.GardenConfig.lua:1475-1484`) and are lit bright on purpose (NOTES, 4 Oct). The Underground costs 100,000 shards. It must be readable without a potion. Selling sight inside a base the player paid for is a tax. |
| Strength (bat knockback) | Reject | It would let shards buy more bat push. That skips the rebirth bat ladder ("every time you rebirth your bat upgrades") and rewards griefing. |
| Resistance (less knockback or blast damage) | Reject | It sells the cure to a problem the game itself creates. Samarth's own bad example, Turtle Master, is a resistance potion. Fruit protection is the Lock dome's job. Bat abuse is fixed with [[Bats]] rules, not a paid potion. |
| Water Breathing | Reject | The sea is about 1.8 studs deep, so players wade and never go under ([[Swimming Fatigue and Boats in Roblox]]). Nobody runs out of breath. Fatigue is Aqua Affinity's job. |
| Fire Resistance | Reject | No fire or lava damage exists or is planned. The lava beams are decor ([[Existing VFX and Props]]) and the mining research says to avoid hazard ores. |
| Healing, Regeneration | Reject | Players have no health outside duels, and duels ignore potions. |
| Turtle Master, Slowness, Weakness, Poison, Decay, Withering, Harming | Reject | Negative effects. "nobody would pay for a potion of decaying". |
| Cash or shard multiplier potion | Reject | The cash boost is the rebirth's job (D-008). A potion that pays shards and is bought with shards is a loop. Luck is already the one money potion. |
| Levitation and other vertical movers | Reject | Jump, Feather Falling and Aether already cover going up and coming down. |

Result: 8 potions. Speed, Jump, Feather Falling, Potion of Aether, Aqua Affinity, Haste, Luck, Invisibility.

## Player view

A short walkthrough with a stage 3 player (Rare, 2 rebirths) holding 1,900 shards who wants to tame a pet in the forest across the sea.

1. She taps **Shop**. The Shop opens. On its left edge there are four tabs: **Potions**, Weapons, King, Robux. Potions is the first tab (it replaced the old Tools tab).
2. She sees 7 potion cards in three rows: Speed, Jump, Feather Falling / Aether, Aqua Affinity, Luck / Invisibility. Each card shows the bottle art, one line saying what it does ("Swim 2.5x, no tiring"), the duration ("10 min"), how many she owns, and the price. Haste is not on the shelf yet: it joins with the second ore band (rule 11, D-032), and then shows "Needs the Underground" where the Buy button would be for players without a mine. (This step listed 8 cards before; corrected by the player-path pass.)
3. Aqua Affinity costs 380 shards at her stage. The price is white because she can afford it. She taps **Buy**. A green line rises from the bottom: "Aqua Affinity Potion added to your inventory!". The card now says "Owned: 1".
4. At the beach she opens **Inventory**, taps the Aqua Affinity card and taps **Drink**. She did not have to put down the fruit she was carrying. She hears a gulp, a small blue puff pops at her head, and a green line says "Aqua Affinity for 10 min!".
5. A blue badge appears in the bottom-right corner with a full blue bar and "10:00".
6. She swims to the forest at 2.5 times the normal swim speed (30 studs per second, faster than walking) and her stamina never drains. A thin trail of bubbles follows her, and other players can see it.
7. When 2 minutes are left the bar turns red. At 0 the badge disappears.
8. If she dies, the potion ends (as today). If she leaves the game, the time left is saved and resumes when she comes back.
9. After her next rebirth (stage 4), a standard potion costs 800 instead of 380. Aqua Affinity stops rising at 500, half a boat.
10. Later, once she owns the Underground and the second ore band has shipped, the Haste card unlocks (28,000 shards at stage 8). Drinking it makes her pickaxe swing 1.5 times as fast for 5 minutes.
11. Early in each stage, Luck makes the next fruit up the ladder start growing on her plot, with a gold glint. Once her Spawn Rarity is maxed for the stage, the Luck card says "Spawn Rarity maxed", and drinking it is refused with "Spawn Rarity is maxed. Save it!" so she never wastes one.

## Rules

Every rule is a proposal and is testable (see the Test plan).

**Catalogue**
1. The shop sells exactly 8 potions: Speed, Jump, Feather Falling, Potion of Aether, Aqua Affinity (new), Haste (new, ships with the Underground), Luck, Invisibility. No other potion is part of this system.
2. Each potion has three parts: a `Config.Tools` entry (`kind = "consume"`, `category = "potion"`, `order`, `short`, `blurb`, `priceUnits`, `effect`, `duration`, and `image` once art exists); a `Config.Effects` entry (`display`, `bar`, `icon`, `order`, stat fields); and a template of the same name in `ReplicatedStorage.Tools`.
3. No potion changes another player's state. No potion takes fruit, pets, shards or cash from anyone.

**Price**
4. Stage s = min(Rebirths + 1, 9).
5. D(s) = 3, 7, 15, 32, 70, 155, 340, 748, 1646 for s = 1 to 9 (the Stage Model's "Shards per merge into this rarity", D-010).
6. Price = Nice(priceUnits x D(s)). Nice(x): below 100, round to the nearest 5. Otherwise round to 2 significant figures, halves up.
7. priceUnits: Speed, Jump, Feather Falling, Invisibility and Aqua Affinity = 25 (1 U). Luck and Haste = 37.5 (1.5 U). Potion of Aether = 50 (2 U). These keep today's price ratios (5,000 : 7,500 : 10,000).
8. Aqua Affinity never costs more than half a boat: price = min(Nice(25 x D), floor(0.5 x `Config.BoatPrice`)). With the boat at 1,000 shards the cap is 500.
9. The server works out the price at the moment of purchase from its own `Rebirths` attribute. The client sends only the tool name.
10. The shop shows the same computed price. It refreshes within one frame of a change to `Rebirths` or `Shards`. The price text is white when Shards >= price, and `Config.UpgradeUnaffordable` red (236, 96, 88) when Shards < price.
11. Haste can be bought only by a player who owns the Underground (`UndergroundFloors` is 1 or more, the attribute [[Underground and Ores]] publishes). Otherwise Buy is refused with the red line "Buy the Underground first!", and the card shows "Needs the Underground" in place of its Buy button. Haste is not in the shop at all until the second ore band ships (D-032).

**Drinking**
12. In the Inventory, a potion card's overlay button reads "Drink", not "Equip". Drink sends `DrinkTool(name)`. The server takes one potion from the bag and starts its effect. The player's hands do not need to be empty.
13. A potion already held in hand (equipped through the old `EquipTool` path) is still drunk with the Use button, with the same result as rule 12.
14. `DrinkTool` is refused, and the bag count is unchanged, when any of these is true: the profile is not loaded; the name is not a `consume` tool; the bag holds 0 of it; the player has `InArena` (red line "Potions do not work in the arena!"); the character is missing or dead; less than 0.3 s has passed since this player's last drink (silent); the potion's own condition fails (rules 25 and 26).
15. Stacking is unchanged: new end = max(now, current end) + duration, and Span = new end - now (`ServerScriptService.GardenService.Tools.lua:209-223`).
16. Durations: Speed 300 s, Jump 300 s, Feather Falling 600 s, Aether 1,800 s, Aqua Affinity 600 s, Haste 300 s, Luck 300 s, Invisibility 60 s.
17. Every potion effect is ignored while the player has `InArena`, as Speed, Jump, Invisibility and Feather Falling already are (`ServerScriptService.GardenService.lua:200-208`, `:266`; `ServerScriptService.GardenService.Ambient.lua:281`).
18. Death clears every effect (unchanged, `ServerScriptService.GardenService.lua:601-604`).
19. Rebirth does not clear effects. Potions in the bag survive rebirth (unchanged, D-009).
20. Leaving saves effects. Each effect with at least 30 s left is saved as seconds left, capped at the time the player's own drinks bought. On the next join it resumes with the same seconds left. Time does not pass while offline. Time granted by a Royal Race decree is not saved.

**Effects**
21. Speed: walk speed x 1.6 on land only. It never changes swim speed.
22. Jump: JumpPower 85 instead of 50 (unchanged).
23. Feather Falling and Potion of Aether: unchanged.
24. Aqua Affinity: while the player is swimming (the character's `SwimState` is not `"Dry"`, [[Boats and Swimming]]), swim speed x 2.5 (12 to 30 studs/s, the speed Boats R18 uses) and swim stamina drain x 0. It does nothing on land or on a boat. Its timer runs whether or not the player is in the water. (Was x 2.0; changed so both notes say 30, decision D-029.)
25. Haste: pickaxe swing rate x 1.5 while it runs. Drinking is refused without the Underground ("Buy the Underground first!").
26. Luck: the plot picks its spawns from Spawn Rarity level min(own level + 1, spawn ceiling level). If the player's own level is already at the spawn ceiling, drinking is refused with "Spawn Rarity is maxed. Save it!". (M4 review: shortened from "... Save it for after your next rebirth!", which at 557 px ran across the touch effect block on a 667 x 375 phone; UI 3 promises that corner never meets the toast text. Pending Samarth's OK.) At stage 9 there is no next rebirth, so there the line is "Spawn Rarity is maxed for good." and the card cannot be bought (UI 1). (Stage 9 case added by the player-path pass, [[Endgame - Transcendent]].) Luck never spawns a merge-only fruit (D-007) or a fruit of a locked rarity. Fruit above the player's own level carry the `Lucky` attribute and glint (unchanged, `ServerScriptService.GardenService.Upgrades.lua:383-398`).
27. Invisibility: unchanged look. An invisible player cannot equip or swing a bat and cannot be hit by one ([[Bats]] B27), and cannot fire a weapon ([[Weapons and Artillery]] rule 14). It does not protect your fruit from explosives, and it does not hide the explosive warning circle or toast. (Changed in the [[Integration Review]] to match those two notes, decision D-030.)

**Feedback and UI**
28. A drink plays the existing gulp sound (`Config.PotionDrinkSound`, `ReplicatedStorage.GardenConfig.lua:1797`), a one-shot 12-particle puff in the potion's bar colour at the head (seen by everyone), and the existing green line "<Effect> for <duration>!" (`ReplicatedStorage.GardenConfig.lua:1800`).
29. The Shop has four tabs, **Potions, Weapons, King, Robux** (decision D-022). Today's Tools tab becomes the Potions tab, first from the top. It lists the potions by `order`, 3 per row. TNT and the other explosives move to the Weapons tab with the sea weapons ([[Weapons and Artillery]] UI 1), so no tool is left on a Tools tab.
30. With a mouse, keyboard or gamepad, effect badges fill one block from the bottom-right corner leftward, 4 per row, at most 8 badges. On touch devices the block moves to the top left, right of the left HUD column and under the royal chip, 2 per row, at most 4 rows, because the area left of the jump button is the touch action slot ([[Bats]] UI 3, shared with the weapon Fire button; decision D-025). The block never overlaps the Clear button, the carry buttons, the carry info bars, the touch jump button or the touch action slot, at 1600x900, 1366x768, 812x375 and 667x375.
31. The effect block is hidden while the player has `InArena`.

## Numbers

### Formulas

```
stage      s     = min(Rebirths + 1, 9)
stage unit D(s)  = 3, 7, 15, 32, 70, 155, 340, 748, 1646          (D-010)
price            = Nice(priceUnits x D(s))
Aqua price       = min(Nice(25 x D(s)), floor(0.5 x BoatPrice))   (BoatPrice = 1000 -> cap 500)
Nice(x)          = x < 100 ? round(x / 5) x 5 : round to 2 significant figures, halves up
```

Worked example, stage 6 (Mythic, 5 rebirths): D = 155. Speed = Nice(25 x 155 = 3,875) = **3,900**. Luck = Nice(37.5 x 155 = 5,812.5) = **5,800**. Aether = Nice(50 x 155 = 7,750) = **7,800**. Aqua = min(3,900, 500) = **500**.

### The catalogue

| Potion (tool key) | Effect key | Does | Duration | priceUnits | Liquid / bar colour (RGB) |
|---|---|---|---|---|---|
| Speed (`SpeedPotion`) | `Speed` | walk x 1.6, land only | 300 s | 25 | 255, 196, 76 (today) |
| Jump (`JumpPotion`) | `Jump` | JumpPower 85 | 300 s | 25 | 122, 221, 122 (today) |
| Feather Falling (`FeatherFallingPotion`) | `FeatherFalling` | fall capped at 22 studs/s, safe landing | 600 s | 25 | 176, 196, 255 (today) |
| Potion of Aether (`AetherPotion`) | `Aether` | survive the thin air | 1,800 s | 50 | 94, 231, 255 (today) |
| Aqua Affinity (`AquaAffinityPotion`) **new** | `AquaAffinity` | swim x 2.5 (30 studs/s), stamina drain x 0 | 600 s | 25, capped at half a boat | 40, 130, 255 |
| Haste (`HastePotion`) **new, phase 2 (second ore band)** | `Haste` | pickaxe swing rate x 1.5 | 300 s | 37.5 | 170, 110, 255 |
| Luck (`LuckPotion`) | `Luck` | +1 Spawn Rarity level, capped at the spawn ceiling | 300 s | 37.5 | 255, 205, 60 (today) |
| Invisibility (`InvisibilityPotion`) | `Invisible` | hidden body and name, sparkle trail | 60 s | 25 | 206, 214, 240 (today) |

Today's colours are from `ReplicatedStorage.GardenConfig.lua:1717-1783`.

### Price at every stage, tied to the Stage Model

"Shards per min" is the median shard income of an attentive player who uses pets and ores (7 simulated careers, `~/fruit-empire/sim/economy.py`, same parameters as `final.py`; stage 9 is its first hour). "Speed in minutes" is how many minutes of that income one standard potion costs.

| Stage | Rarity | Rebirths | Full-plot $/s | D | Speed, Jump, Feather Falling, Invisibility (25 D) | Aqua Affinity | Luck, Haste (37.5 D) | Aether (50 D) | Shards per min | Speed in minutes |
|---|---|---|---|---|---|---|---|---|---|---|
| 1 | Common | 0 | 707 | 3 | 75 | 75 | 110 | 150 | 15.5 | 4.8 |
| 2 | Uncommon | 1 | 430K | 7 | 180 | 180 | 260 | 350 | 38.9 | 4.6 |
| 3 | Rare | 2 | 3.7B | 15 | 380 | 380 | 560 | 750 | 102.5 | 3.7 |
| 4 | Epic | 3 | 3.6T | 32 | 800 | 500 | 1,200 | 1,600 | 170.7 | 4.7 |
| 5 | Legendary | 4 | 1.8Qa | 70 | 1,800 | 500 | 2,600 | 3,500 | 315.4 | 5.7 |
| 6 | Mythic | 5 | 2.5Qi | 155 | 3,900 | 500 | 5,800 | 7,800 | 758.6 | 5.1 |
| 7 | Astral | 6 | 265Qi | 340 | 8,500 | 500 | 13,000 | 17,000 | 933 | 9.1 |
| 8 | Eternal | 7 | 506Sx | 748 | 19,000 | 500 | 28,000 | 37,000 | 2,600 | 7.3 |
| 9 | Transcendent | 8 | 24.9Sp | 1,646 | 41,000 | 500 | 62,000 | 82,000 | 6,674 | 6.1 |

Potions are priced in shards against D, so the cash column (full-plot $/s, before the Index bonus) does not enter any price. It is shown so every row can be matched to the Stage Model.

### Re-check of today's prices against the new shard flows

Today every potion has a flat price: 5,000 (Speed, Jump, Invisibility, Feather Falling), 7,500 (Luck), 10,000 (Aether) (`ReplicatedStorage.GardenConfig.lua:1631-1704`). Against the Stage Model's shard income that breaks at both ends.

| Stage | Today, 5,000-shard potion, in minutes of shard income | Proposed standard potion, in minutes | Proposed Luck, in minutes | Proposed Aether, in minutes | Proposed standard potion, share of the whole stage's shards |
|---|---|---|---|---|---|
| 1 | 322 (more than the stage's total of 232 shards) | 4.8 | 7.1 | 9.7 | 32% of a 15-minute stage |
| 3 | 48.8 | 3.7 | 5.5 | 7.3 | 6.0% |
| 6 | 6.6 | 5.1 | 7.6 | 10.3 | 4.9% |
| 9 | 0.75 (trivial) | 6.1 | 9.3 | 12.3 | 10.2% of one hour |

So today a new player cannot afford any potion until about stage 4, and a late player never notices the price. The proposal keeps a standard potion at roughly 4 to 9 minutes of shard income at every stage. Rebirth shard costs take about 19 to 27% of each stage's shards (D-011 against the model's stage totals, [[Stage Model]]), so potions stay a real choice against saving.

Aether becomes reachable early (150 shards at stage 1, 350 at stage 2). It still costs two thirds of stage 1's shards and a quarter of stage 2's, so the sky is an early goal, not a freebie. See Open questions.

### What Luck is worth (model)

Measured with a copy of `simulate_stage` that applies Luck (rule 26) for 300 s from a chosen minute. Median of 12 careers, attentive player.

| Stage | Stage length | Spawn ceiling reached at (model, `probe2.py`) | Luck drunk at minute 1 shortens the stage by | Luck drunk at 50% of the stage | After the spawn ceiling |
|---|---|---|---|---|---|
| 1 | 14.8 min | 0.9 min | 0.42 min | refused (ceiling reached) | refused |
| 3 | 60.7 min | 27.8 min | 2.50 min | 0.92 min (some careers still climbing) | refused |
| 6 | 105.2 min | 40.8 min | 3.21 min | refused | refused |
| 8 | 150.2 min | 38.0 min | 3.38 min | refused | refused |

Luck is a "climb" potion. It saves 2.5 to 3.4 minutes when drunk early in stages 3 to 8, and almost nothing after the spawn ceiling. The alternative rule (let Luck spawn the first merge-only fruit) saved only 0.2 to 1.0 minute late in a stage and breaks D-007, so it is not used. This replaces today's "tilt the odds at the top level" behaviour (`ServerScriptService.GardenService.Upgrades.lua:347-366`, `topFalloff` at `ReplicatedStorage.GardenConfig.lua:1787`).

### Effect numbers

| Effect | Stat | Value | Source |
|---|---|---|---|
| Speed | `walkMultiplier` | 1.6: walk 20 to 32 studs/s, land only | `ReplicatedStorage.GardenConfig.lua:1721`, `:943` |
| Jump | `jumpPower` | 85 (base 50): jump height 18.4 vs 6.4 studs, 2.9x (h = P^2 / (2 x 196.2), computed) | `ReplicatedStorage.GardenConfig.lua:1727`, `:1793` |
| Feather Falling | `fallSpeed` | 22 studs/s | `ReplicatedStorage.GardenConfig.lua:1772` |
| Aqua Affinity | `swimMultiplier`, `drainMultiplier` | 2.5 and 0. With the base swim of 12 studs/s this is 30 studs/s (1.5 times walking), so the 352-stud crossing from the last stone to the forest beach takes about 12 s instead of 29 s | [[Swimming Fatigue and Boats in Roblox]] (base numbers belong to [[Boats and Swimming]] R8 and R18) |
| Haste | `mineRateMultiplier` | 1.5 | proposal; base swing rate belongs to [[Underground and Ores]] |
| Luck | `tierBonus` | 1 level, capped at the spawn ceiling | rule 26 |
| Drink puff | particles, life | 12, 0.4 to 0.7 s, sprite `rbxasset://textures/particles/sparkles_main.dds` | same recipe as the Luck glint, `ServerScriptService.GardenService.Upgrades.lua:385-397` |
| Drink cooldown | per player | 0.3 s | rule 14 |
| Saved effect threshold | seconds left | at least 30 s; at most 86,400 s accepted on load | rule 20 |

## UI

All text is Fredoka One (`Font.new("rbxasset://fonts/families/FredokaOne.json")`), white, with a black stroke, as the kit requires ([[UI Kit Inventory]]). No emoji. No new HUD button. The only HUD change is re-laying the existing effect block.

### 1. Shop: the Potions tab (inside the Shop panel)

- **Tab.** `ShopMenu.PotionsTab`: today's `ShopMenu.ToolsTab`, renamed (the Tools tab has nothing left on it once explosives move to the Weapons tab, decision D-022). Kit component: **Shop tab (vertical)**, image 110917091182913 (100x132 native). The label is a Fredoka TextLabel rotated -90 at x = 0.25 of the tab, as on the other tabs (HANDOFF-weilin-2026-09, "The label is offset, not centred"). Text: "POTIONS". It sits first: Potions, Weapons, King, Robux (`Config.ShopTabs`, `ReplicatedStorage.GardenConfig.lua:1210-1212`; the Weapons tab is [[Weapons and Artillery]] UI 1). Selected / unselected states use the existing tint (`Config.ShopTabDim`, `ReplicatedStorage.GardenConfig.lua:1216`). Hover and press come from ButtonAnimator as for every button.
- **Tab column fit.** Four tabs at today's height (0.25 of the panel) do not fit: 4 x 0.25 + 3 x 0.03 = 1.09 of the panel. The visible panel body starts 48/439 down the art (`StarterPlayer.StarterPlayerScripts.ShopMenu.lua:19-24`). New rule in the column layout (`StarterPlayer.StarterPlayerScripts.ShopMenu.lua:75-85`): tab height = min(0.25, (1 - 48/439 - 0.04 - (n - 1) x 0.025) / n), centred on the visible body (y = (48/439 + 1) / 2 = 0.555). For n = 4 that is 0.194 of the panel per tab. For n = 3 the height stays 0.25. The tabs keep their tuck behind the panel's left border (`Config.ShopPanelLeftEdge`).
- **Page.** `ShopMenu.Panel.PotionsPage`: today's `ToolsPage`, renamed (ScrollingFrame with its UIPadding). Grid: 3 columns, gap 0.04 of content width, card aspect 162/178 (same as `StarterPlayer.StarterPlayerScripts.ShopMenu.lua:138-140`). 8 cards make rows of 3, 3 and 2 in `order`: Speed, Jump, Feather Falling / Aether, Aqua Affinity, Haste / Luck, Invisibility. Row 2 is the three places a potion opens: sky, sea, underground. Until Haste ships (D-032) there are 7 cards and Luck moves up into its place.
- **Card.** Kit component **Item card (tools/king)**, image 99120915117089 (`Config.ToolBuyBox`). Positions are fractions of the card:

| Element | Kit part | Centre Y | Size (W x H) | Text size |
|---|---|---|---|---|
| Title (`short`: "Speed", "Aqua Affinity", "Invisibility", ...) | Fredoka TextLabel | 0.12 | 0.82 x 0.13 | clamp(floor(cellH x 0.085), 12, 30) |
| Art (`image`; the 3D bottle viewport only until the art exists) | ImageLabel, ScaleType Fit | 0.34 | 0.40 x 0.30 | - |
| Effect line (`blurb`, 18 characters at most) | Fredoka TextLabel | 0.535 | 0.86 x 0.065 | clamp(floor(cellH x 0.065), 10, 20) |
| Info row: "10 min" or "10 min   Owned: 2" (one label) | Fredoka TextLabel | 0.61 | 0.86 x 0.055 | clamp(floor(cellH x 0.055), 9, 18) |
| Buy button with price | Kit **Buy button**, image 97067256017679 (`Config.ToolBuyButton`, shard crystal baked on the left) | 0.80 | 0.80 wide, aspect 114/40 (unchanged) | price clamp(floor(cellH x 0.075), 10, 24) |

  Text sizes are fixed per card from the grid's cell height, recomputed in the fit pass, with `TextScaled` off. Equal labels then render at equal sizes on every potion card, which is the fix [[Consistent Image-Based UI Kits in Roblox]] asks for (finding: `TextScaled` makes equal labels different sizes).
- **Row fit (M4 Studio pass).** Each row of words (titles, effect lines, info lines, prices) takes the size above, or smaller, down to the row's smallest, until every card's words in that row fit their box. The whole row shrinks together, so equal labels stay equal. With the Larger Text Size setting a 1529 x 736 window drew "Feather Falling" at 190 px in a 177 px box ("Feather..."); now every title is 18 there. Below the smallest size a line is still cut with "...".
- **Label boxes (M4 review).** The Size column's heights are not used for the four text labels: each box is `ceil(text size x 1.3) + 10` px tall (the kit's own box plus room for the Larger Text Size setting), rewritten with the text size. A box that is a share of a phone-sized cell (6.6 px for 10 px words) would let the kit's truncation drop the line. The boxes are transparent and centred on the same rows, so nothing moves.
- **Small-screen rule (M4 review, pending Samarth's OK).** The effect line and the info row are 0.075 of the card apart. On a card under about 156 px tall (every phone) that cannot hold the effect line's descenders, the info row's capitals and their two 2 px outlines, so the outlines merge. There the effect line is hidden and the info row moves to 0.58 (midway between the art's bottom, 0.49, and Buy's top, 0.672). The rule in code: hide when `cellH x 0.075 - 0.5 x effect size - 0.3 x info size` is less than the two outlines. The sizes here are the sizes DRAWN (the label's TextBounds height), not TextSize (M4 Studio pass): with the Larger Text Size setting every line is about 10 px taller, and on a desktop card the two lines touched while the rule saw room. So with Larger text a desktop card also shows one centred info line. (Design call for Samarth: the other way is to shrink the potion picture on those cards to keep the effect line.)
- **Effect lines** (`blurb`): Speed "Run 60% faster". Jump "Jump 2.9x higher". Feather Falling "Glide down safely". Aether "Breathe in the sky". Aqua Affinity "Swim 2.5x, no tiring". Haste "Mine 1.5x faster". Luck "Grow rarer fruit". Invisibility "Hide from players".
- **Card states.**
  - Normal: price white.
  - Too poor: price `Config.UpgradeUnaffordable` red. Buy still sends; the server answers "Not enough shards!" (`ReplicatedStorage.GardenConfig.lua:1849`).
  - Haste locked (no Underground): the Buy button is hidden. In its place a Fredoka label "Needs the Underground" (same box as the button). The art is tinted `Config.ShopTabDim`.
  - Luck maxed: the info row reads "Spawn Rarity maxed". Buy stays enabled so a player can stock up for the next stage. **At stage 9** (no next stage) the info row reads "Maxed for good" and the Buy button is hidden, so nobody pays 62,000 shards for a potion that can never be drunk.
- **Overlap.** Everything lives inside the Shop panel, which already hides the other panels when it opens (`StarterPlayer.StarterPlayerScripts.ShopMenu.lua:604-633`). No HUD footprint.

### 2. Inventory: the Drink button (inside the Inventory panel)

- Potion cards in `StoredFruitsMenu` keep the shop card art (`Config.TNTBuyBox`, `StarterPlayer.StarterPlayerScripts.StoredFruitsMenu.lua:398`), show `image` art instead of the 3D icon when it exists, use `short` as the title, and sort among tools by `order` (LayoutOrder = -1000 + order, replacing the flat -1000 at `:395`).
- The overlay button caption becomes "Drink" for `consume` tools (today "Equip", `:266`). It fires `DrinkTool` and does not run the "hands full" check (`:294-297`), because drinking from the bag needs no free hands. The panel closes after the press, as today (`:305`).
- Kit note: this overlay is still a code-drawn button (`:221-275`, listed as "to convert" in [[Client UI - HUD and Menus]] section 9). This spec changes only its caption and remote. When the UI conversion swaps it for the kit's **Green action button (Claim)**, image 132804991293392, Drink uses the same art. If that art has "CLAIM" baked in, the blank green button in Asset request 8 covers it.

### 3. Effect block (HUD: bottom right with a mouse, top left on touch)

- **Frame.** The existing `GardenUI.EffectStack` (already in `Config.HudClusters`, `ReplicatedStorage.GardenConfig.lua:1863-1867`, so HudScale scales it). New authored layout at the 1600x900 reference: AnchorPoint (1, 1), Position (1, -16, 1, -16), Size 262 x 180, BackgroundTransparency 1, Active false. 262 = 4 x 58 + 3 x 10. 180 = 2 x 85 + 10. (M4 Studio pass: this was 262 x 275, three rows. At most 8 badges at 4 per row only ever fill two, and on a 1529 x 736 window the empty third row reached the Clear button. `Config.EffectRows` = 2.) On touch, HudScale re-anchors and resizes it (next bullets).
- **Layout.** The bottom-aligned UIListLayout (`StarterPlayer.StarterPlayerScripts.GardenClient.lua:450-451`) is replaced by a UIGridLayout made in code: CellSize 58 x 85 (`Config.EffectIconSize` 58 plus bar 7 plus label 20), CellPadding 10 x 10 (`Config.EffectRowGap`), StartCorner BottomRight, FillDirection Horizontal, FillDirectionMaxCells 4 (2 on touch), SortOrder LayoutOrder with LayoutOrder = `Config.Effects[name].order`. The first effect sits in the corner, later ones fill leftward, and a fifth starts the row above. Hidden badges take no cell, so an expiry reflows the rest with no gaps.
- **Badge.** Unchanged build (`StarterPlayer.StarterPlayerScripts.GardenClient.lua:464-547`): 58 x 58 badge image, a 7 px draining bar in the effect's `bar` colour (red under 20%), and a Fredoka countdown (`s`, `m:ss`, `h:mm:ss`). New badge art for Aqua Affinity and Haste (Asset requests 2 and 4). Until it arrives, a placeholder in the same style, as was done for Feather Falling and Luck (`~/fruit-empire/art/potion-icons/placeholder-*.png`).
- **Touch devices (changed in the [[Integration Review]], conflict C21, decision D-025).** The area left of the jump button is the touch action slot ([[Bats]] UI 3: Bat, Swing, or the weapon Fire button), so the block no longer goes there. When Roblox's touch controls are in use (a touch-only device, or `PlayerGui.TouchGui` enabled with touch as the last input; M4 review: TouchGui alone is made at join on every touchscreen laptop, so a laptop played with the mouse keeps the mouse layout), HudScale anchors the block top-left (AnchorPoint (0, 0)): its left edge 8 px right of the left HUD column (Inventory, Index, Shop, Pets), its top edge 8 px under the royal chip cluster ([[King UI Overhaul]] 5.2, read live), StartCorner TopLeft, 2 per row, at most 4 rows (126 x 360 reference px). That corner is outside the centre third, so it never meets the event strip, the herald or the toast text, and it is far from the jump button.
- **Arena.** The whole block hides while `InArena` is set, because effects do nothing there and the RoyalCombat touch buttons ring the jump button (`StarterPlayer.StarterPlayerScripts.RoyalCombat.lua:283-302`).
- **Overlap check (worked, HUD scale from `Config.HudScaleFor`, `ReplicatedStorage.GardenConfig.lua:1949-1957`):**

| Screen | HUD scale | Block (occupied by 8 badges) | Nearest neighbour | Clearance |
|---|---|---|---|---|
| 1600 x 900, mouse | 1.00 | x 1322-1584, y 704-884 (2 rows of 4) | Clear button bottom edge y ~541; carry buttons right edge x ~975 | 163 px above, 347 px left |
| 1366 x 768, mouse | ~0.91 | 2 rows of 4, same corner | same | scales down with the HUD, same proportions |
| 812 x 375, touch | 0.55 (clamped) | x 70-140, y 46-251 (2 per row, top left) | left column ends x 62; event strip starts x 252; herald x 230 | 8 px / 112 px / 90 px |
| 667 x 375, touch | 0.55 | x 70-140, y 46-251 | left column ends x 62; herald starts x 157; CornerHud top y 277 | 8 px / 17 px / 26 px |

  On touch the block is in the top-left corner, so it shares no space with the jump button, the touch action slot, the right column or the carry row. The full HUD table is in the [[Integration Review]] section 4. The carry button and Clear positions come from [[UI Kit Inventory]] "Layout facts"; the Clear bottom edge is its centre (+45) plus half its ~92 px height [inferring the button height from the ~96x92 HUD button size].

### 4. Toasts (existing lines, new strings)

Success uses the green NOTICE line from the bottom. Refusals use the red WARN line from the top (`ReplicatedStorage.GardenToast.lua:49-67`). New strings, all plain ASCII: "Potions do not work in the arena!", "Buy the Underground first!", "Spawn Rarity is maxed. Save it!" (M4 review: shortened, rule 26). A red line must stay under about 390 px at size 24, or on a 667 x 375 phone it reaches the touch effect block (x 66-135, UI 3). Existing: "Not enough shards!", "%s added to your inventory!", "%s for %s!".

## World and assets

| What | From | Build |
|---|---|---|
| `ReplicatedStorage.Tools.AquaAffinityPotion` | Copy of `ReplicatedStorage.Tools.SpeedPotion` (a five-part welded bottle, `ReplicatedStorage.GardenConfig.lua:1628-1630`) | Rename. Set the part named `Liquid` to RGB 40, 130, 255. Keep its material. Same way the Feather Falling and Luck bottles were made (NOTES, 4 Oct). |
| `ReplicatedStorage.Tools.HastePotion` (phase 2) | Copy of `SpeedPotion` | Rename. `Liquid` to RGB 170, 110, 255. |
| `ServerStorage.PotionFX.AquaWake` | Copy of the ParticleEmitter(s) inside `Workspace.Anime["Water-01"]` in the staging area at y ~ -790 ([[Existing VFX and Props]]) | Copy only the emitters, not the staging part, so other systems keep the original. Tune: Rate 12, Lifetime 0.6-1.0 s, LockedToPart false, Speed 1-2. Server clones one into the root as `AquaWake`; on only while Swimming with Aqua Affinity, off while invisible (same rule as `FeatherTrail`, `ServerScriptService.GardenService.Ambient.lua:108-115`). |
| `ServerStorage.PotionFX.HasteSpark` (phase 2) | Copy of the emitter(s) inside `Workspace.Anime["Shiny-01"]` | Rate 0. The Underground's pickaxe hit code calls `Emit(6)` at the hit point while Haste runs. |
| Drink puff | none (code) | One-shot ParticleEmitter on the head, 12 particles, colour = the effect's `bar`, `sparkles_main.dds`, LightEmission 0.8, destroyed after 1.5 s. Same recipe as `markLucky` (`ServerScriptService.GardenService.Upgrades.lua:383-398`). |
| Sounds | existing | Drink: `Config.PotionDrinkSound` "gulp3" (`ReplicatedStorage.GardenConfig.lua:1797`). No new sounds. |
| `GardenUI.ShopMenu.PotionsTab`, `GardenUI.ShopMenu.Panel.PotionsPage` | Copies of `ToolsTab` and `ToolsPage` | Studio edit. Label "POTIONS". |
| `GardenUI.EffectStack` | existing | Studio edit: AnchorPoint (1, 1), Position (1, -16, 1, -16), Size 262 x 180 (was 262 x 275; M4 Studio pass), remove its UIListLayout. |

Nothing else is built. No new models in Workspace. The optional potion stall (a Stud Shop booth, [[Creator Store Packs]]) is an open question, not part of v1.

## Asset requests for Samarth

Images only. No animations to publish and no Robux products in v1.

**How to use these prompts** (same method as `~/fruit-empire/art/potion-icons/PROMPTS.md`):
1. Attach `~/fruit-empire/art/potion-icons/style-reference-potions.png` to the first message. It is your own kit art: the Potion of Aether shop bottle (`Config.Tools.AetherPotion.image`, rbxassetid://123282319597102) and the effect badges, including the Aether badge (`Config.Effects.Aether.icon`, rbxassetid://91730920675016). Those two are the style references for every image below.
2. Ask for each image on its own.
3. Save each result in `~/fruit-empire/art/potion-icons/` with the file name shown. Claude keys out the backing and uploads at 512x512, then sets `image` and `icon` in GardenConfig.
4. If one comes back with text or a background, reply: `Same image, but with a transparent background and no text.`

**1. Aqua Affinity: shop art** -> `aquaaffinity-shop.png`
```
Pixel art game icon in the exact style of the attached potion bottle: a chunky Minecraft-style glass potion bottle with a brown cork, glossy glass highlights, glowing liquid and small four-point sparkles. Transparent background, no text.

This one is an AQUA AFFINITY potion: the liquid is a bright ocean blue, with a small white wave curling inside it, and three or four round white bubbles rising out of the liquid and up past the cork.
```

**2. Aqua Affinity: effect badge** -> `aquaaffinity-badge.png`
```
Pixel art status icon in the exact style of the attached rounded badges: a rounded square tile with a vertical gradient, a thick black outline and a white outer rim, with one pixel-art symbol in the middle. No text.

This one: a deep ocean-blue gradient (bright azure at the top, navy at the bottom), and the symbol is a white-and-aqua pixel wave curling to the right, with three small round bubbles rising above it.
```

**3. Haste: shop art (phase 2)** -> `haste-shop.png`
```
Pixel art game icon in the exact style of the attached potion bottle: a chunky Minecraft-style glass potion bottle with a brown cork, glossy glass highlights, glowing liquid and small four-point sparkles. Transparent background, no text.

This one is a HASTE potion: the liquid is a rich royal purple, with a small pixel pickaxe inside it (light brown handle, bright gold head), and a few small gold sparks flying off the side of the bottle.
```

**4. Haste: effect badge (phase 2)** -> `haste-badge.png`
```
Pixel art status icon in the exact style of the attached rounded badges: a rounded square tile with a vertical gradient, a thick black outline and a white outer rim, with one pixel-art symbol in the middle. No text.

This one: a violet gradient (light lilac at the top, deep purple at the bottom), and the symbol is a pixel pickaxe with a light brown handle and a bright gold head, tilted to the right, with three short white speed lines behind it.
```

**5. Speed: shop art** -> `speed-shop.png` (today this card shows a spinning 3D bottle; the badge already exists)
```
Pixel art game icon in the exact style of the attached potion bottle: a chunky Minecraft-style glass potion bottle with a brown cork, glossy glass highlights, glowing liquid and small four-point sparkles. Transparent background, no text.

This one is a SPEED potion: the liquid is a warm orange-gold, with a small brown pixel boot inside it, and three short white speed lines streaking past the left side of the bottle.
```

**6. Jump: shop art** -> `jump-shop.png`
```
Pixel art game icon in the exact style of the attached potion bottle: a chunky Minecraft-style glass potion bottle with a brown cork, glossy glass highlights, glowing liquid and small four-point sparkles. Transparent background, no text.

This one is a JUMP potion: the liquid is a bright lime green, with a small white pixel up-arrow inside it, and two or three green sparkles rising above the cork.
```

**7. Invisibility: shop art** -> `invisibility-shop.png`
```
Pixel art game icon in the exact style of the attached potion bottle: a chunky Minecraft-style glass potion bottle with a brown cork, glossy glass highlights, glowing liquid and small four-point sparkles. Transparent background, no text.

This one is an INVISIBILITY potion: the liquid is a pale silvery white that fades to see-through toward the top of the bottle, with a small gold pixel hand mirror inside it and a few faint white sparkles.
```

**8. Only if needed: a blank green action button** -> `green-action-blank.png`. Needed only if the kit's Green action button (Claim, rbxassetid://132804991293392) has the word CLAIM baked into the image. Attach a screenshot of the Index panel's Claim button as the reference.
```
Game UI button in the exact style of the attached green CLAIM button: same glossy green face, same darker green bottom lip, same thick black outline, same corner radius and proportions (about 4.6 wide to 1 tall). Completely blank: no text, no icon. Transparent background.
```

**Still waiting from before:** Feather Falling and Luck shop art and badges. Their prompts are already in `~/fruit-empire/art/potion-icons/PROMPTS.md`.

## Code plan

Follows the GardenService module pattern: each module is `return function(G)`, loaded in order by `ServerScriptService.GardenService.lua:487-519`, sharing state through `G`. A module copies from `G` at require time only what earlier modules exported; anything from a later module is looked up through `G` at call time (`ServerScriptService.GardenService.Carrying.lua:352-353`).

### New module: `ServerScriptService.GardenService.Potions` (ModuleScript)

Required in `ServerScriptService.GardenService.lua` right after Tools (`:491-493`) and before Storage (`:495`):
```lua
require(script.Potions)(G)
```
It owns:
- `drink(player, name, spec)`: the effect half of today's `consumeTool` (`ServerScriptService.GardenService.Tools.lua:200-241`) unchanged in behaviour, plus: record `drunk[player][spec.effect] = endsAt`; play the drink puff; refuse (return false with a reason) when `G.arenaStats`/`InArena`, a dead character, or the potion's condition fails. Exported as `G.drink`.
- `potionUsable(player, spec)`: returns `ok, warningText`. Haste: Underground owned (`player:GetAttribute("UndergroundFloors")` >= 1, the [[Underground and Ores]] ownership attribute). Luck: own Spawn Rarity level < spawn ceiling level (from [[Rarity Gating and Rebirth]], proposed `Config.SpawnCeilingLevel(rebirths)`). Everything else: true. Exported as `G.potionUsable`.
- `drinkFromBag(player, name)`: the `DrinkTool` handler. Order: `typeof(name) == "string"`; `Config.Tools[name]` with `kind == "consume"`; `Store.IsReady`; 0.3 s per-player cooldown (`drinkReadyAt[player]`); `potionUsable`; `G.takeTool(player, name)` last, so a refusal never eats a potion; then `drink`. Then `Store.Touch(player, Config.SaveTouch)`.
- `captureEffects(player)` -> `{ [effect] = seconds }`: for each running effect, seconds = min(effects end, drunk end) - now; kept only if >= 30. Yield-free.
- `restoreEffects(player, saved)`: for each entry, end = now + seconds; writes `effects`, `drunk`, `<Effect>EndsAt`, `<Effect>Span` (= seconds); then `applyWalkSpeed` and `applyVisibility`. Yield-free.
- `potionSweep(player, character, root, humanoid)`: the Aqua wake emitter on/off (built once per character, then toggled, like `feathers`, `ServerScriptService.GardenService.Ambient.lua:77-116`).
- `potionForget(player)`: clears `drunk[player]` and `drinkReadyAt[player]`.
- Exports: `G.drink, G.potionUsable, G.drinkFromBag, G.captureEffects, G.restoreEffects, G.potionSweep, G.potionForget`.

### Changes to existing scripts

| Script | Where | Change |
|---|---|---|
| `ReplicatedStorage.GardenConfig.lua` | `:1210-1212` | `ShopTabs = { "Potions", "Weapons", "King", "Robux" }` (King still behind `KingEnabled`), decision D-022. |
| | `:1631-1704` | Each potion: drop `price`, add `category = "potion"`, `order`, `short`, `blurb`, `priceUnits`. Add `AquaAffinityPotion` and `HastePotion` (`requires = "Underground"`). `image` only once art is uploaded (an empty id would show a blank card). TNT keeps its flat `price`. |
| | `:1714-1789` | Add `order` to every effect. Add `AquaAffinity { display = "Aqua Affinity", bar = Color3.fromRGB(40,130,255), icon, order = 5, swimMultiplier = 2.5, drainMultiplier = 0 }` and `Haste { display = "Haste", bar = Color3.fromRGB(170,110,255), icon, order = 6, mineRateMultiplier = 1.5 }`. Keep `Luck.topFalloff` only for the pre-gating fallback. |
| | near `:1905-1933` (accessors) | New `Config.StageShardUnit`, `Config.StageFor`, `Config.NicePrice`, `Config.ToolPrice` (below). If [[Rarity Gating and Rebirth]] already defines a stage function or the D table, use theirs. |
| | `:1830-1832`, `:1848-1857` | Add `Config.EffectColumns = 4`, `Config.EffectColumnsTouch = 3`, `Config.EffectJumpGap = 8`, `Config.DrinkCooldown = 0.3`, `Config.EffectSaveMin = 30`, and the three new warning strings. |
| `ServerScriptService.GardenService.Tools.lua` | `buyTool`, `:91-112` | Price = `Config.ToolPrice(name, player:GetAttribute("Rebirths"))` instead of `spec.price` at `:101` and `:106`. Before the shard check: if `spec.requires` and `G.potionUsable` says no, send its warning and return. |
| | `consumeTool`, `:186-242` | Keep the effect check (`:187-192`) and the release (`:197-198`). Replace `:200-241` with `G.drink(player, name, spec)`, looked up at call time. If `drink` refuses after release, put the bottle back with `addTool` so nothing is lost. |
| | exports, `:314-317` | Also export `takeTool` as `G.takeTool`. |
| `ServerScriptService.GardenService.lua` | remotes, `:59-86` | `local drinkToolRemote = remote("DrinkTool") -- client -> server, one potion name`. Add it to `G` at `:101-115`. |
| | `:177-185` | Add `local drunk = {}` (runtime: [player][effect] = end time bought by drinks). Export `G.drunk` with the others at `:390-396`. |
| | `applyWalkSpeed`, `:210-215` | Speed applies only when `character:GetAttribute("SwimState")` is `"Dry"` or nil. Swim speed (base x Aqua multiplier) is written here by [[Boats and Swimming]] in the same function, so there is still one writer. Rename the local `haste` (it holds `Config.Effects.Speed`) to `swift` to avoid confusion with the new Haste effect. |
| | `clearEffects`, `:554-570` | Also `drunk[player] = nil`. Export `G.clearEffects`. |
| | wiring, `:528-537` | `drinkToolRemote.OnServerEvent:Connect(function(player, name) G.drinkFromBag(player, name) end)`. |
| | `teardownPlayer`, `:703-735` | After the save (`:711`), call `G.potionForget(player)` next to `effects[player] = nil` (`:721`). The save runs first, so effects are captured. |
| | expiry sweep, `:793-821` | When an effect expires, also clear `drunk[player][name]`. |
| `ServerScriptService.GardenService.Upgrades.lua` | `spawnPlan`, `:372-379` | Luck: `top = Config.SpawnCeilingFor(rebirths)` ([[Rarity Gating and Rebirth]]; the same change that note lists, built once), falling back to `Config.MaxLevel("spawnTier")` only before gating ships. If `level < top`, return `min(level + luck.tierBonus, top), false, true`. The tilt branch runs only when `G.spawnCeiling` does not exist (pre-gating build). |
| `ServerScriptService.GardenService.Ambient.lua` | header `:14-22`, sweep `:281-282` | Copy `G.potionSweep` and call it beside `feathers(...)` for each living character. |
| `ServerScriptService.GardenService.Persistence.lua` | header `:15-31`; capture after `:61-67`; apply after `:221-225` | `profile.effects = G.captureEffects(player)`; `G.restoreEffects(player, profile.effects or {})`. Both stay yield-free (`:35-37`). |
| `ServerScriptService.GardenStore.lua` | `Store.Default`, `:240-243` | `effects = {}` with a comment. |
| | `sanitise`, after `:358-380` | Keep a pair only if the key is a string in `Config.Effects`, the value is a number, not NaN, and 0 < value <= 86400. Anything else counts as one repair. |
| | self-test, `:1199-1209` | Add one entry per effect at 86400 to the worst payload. It must still be under 8,192 bytes (`:1209`). |
| `ServerScriptService.GardenService.DevPanel.lua` | `clearPlayerResidentState`, `:162-183` | Call `G.clearEffects(player)` and `G.potionForget(player)`, so a Round trip proves effects come back from the save. |
| `StarterPlayer.StarterPlayerScripts.ShopMenu.lua` | tab layout `:75-85` | Height and centre formula from UI section 1. |
| | grid `:138-196`, cards `:197-376`, build `:378-388` | Make the grid a per-page helper (`makeGrid(page)` returning `add` and `fit`). Potions (`category == "potion"`) go to `PotionsPage` (the renamed `ToolsPage`) sorted by `order`; explosives go to the Weapons page with the sea weapons ([[Weapons and Artillery]] code plan, D-022). Potion cards use the card layout and fixed text sizes from UI section 1. Price = `Config.ToolPrice(name, player:GetAttribute("Rebirths"))`, repainted on `Rebirths`, `Shards`, `UndergroundFloors` and `Upgrade_spawnTier` changes. |
| `StarterPlayer.StarterPlayerScripts.StoredFruitsMenu.lua` | `makeHover`, `:203-307`; `makeToolCard`, `:384-443` | `makeHover(card, name, remote, caption, needsFreeHands)`. Potion cards pass the `DrinkTool` remote, "Drink", and `false`. Use `image` art when present, `short` as the title, LayoutOrder -1000 + `order`. |
| `StarterPlayer.StarterPlayerScripts.GardenClient.lua` | effect block `:450-557`; loop `:559-609` | Replace the list layout with the UIGridLayout from UI section 3; sort by `order`; set FillDirectionMaxCells from the touch check; set `stack.Visible = not player:GetAttribute("InArena")` in the loop. |
| `StarterPlayer.StarterPlayerScripts.HudScale.lua` | `place`, `:116-127` | For `EffectStack` only: on touch, anchor it top-left 8 px right of the left HUD column and 8 px under the royal chip cluster, width 126 reference px (2 per row); otherwise bottom right, 262 px (UI section 3). HudScale stays the only writer of cluster position and size ([[Client UI - HUD and Menus]] gotcha 2). Re-place when the jump button's `AbsolutePosition` changes. |

The shared price function (load-bearing, so shown in full). It lives **once**, in the rarity-gate block of `GardenConfig` that [[Rarity Gating and Rebirth]] adds (decision D-017); this note only adds `ToolPrice`. Explosives, weapons and blade skins call `Config.ShardPrice` too, so every shard price in the game rounds the same way:
```lua
-- in the Rarity Gating block (one copy):
-- Config.StageFor(rebirths)  -> 1..9
-- Config.ShardDropFor(stage) -> 3, 7, 15, 32, 70, 155, 340, 748, 1646 (D-010)
function Config.NicePrice(x)
	if x < 100 then return math.floor(x / 5 + 0.5) * 5 end
	local mag = 10 ^ (math.floor(math.log10(x)) - 1)
	return math.floor(x / mag + 0.5) * mag
end
function Config.ShardPrice(units, rebirths)
	return Config.NicePrice(units * Config.ShardDropFor(Config.StageFor(rebirths)))
end
-- added by this note:
function Config.ToolPrice(name, rebirths)
	local spec = Config.Tools[name]
	if not spec then return nil end
	if not spec.priceUnits then return spec.price end -- flat-priced tools (none left after this expansion)
	local price = Config.ShardPrice(spec.priceUnits, rebirths)
	if spec.priceCapBoatShare and Config.BoatPrice then
		price = math.min(price, math.floor(spec.priceCapBoatShare * Config.BoatPrice))
	end
	return price
end
```

One new tool entry, as an example of the fields:
```lua
AquaAffinityPotion = {
	display = "Aqua Affinity Potion", short = "Aqua Affinity", blurb = "Swim 2x, no tiring",
	kind = "consume", category = "potion", order = 5,
	priceUnits = 25, priceCapBoatShare = 0.5,
	effect = "AquaAffinity", duration = 600,
	-- image = "rbxassetid://...", -- set when Samarth's art is uploaded
},
```
Orders: Speed 1, Jump 2, Feather Falling 3, Aether 4, Aqua Affinity 5, Haste 6, Luck 7, Invisibility 8. The same numbers go on the `Config.Effects` entries.

### Remotes

| Name | Direction | Payload | Server validation |
|---|---|---|---|
| `DrinkTool` (new RemoteEvent) | client to server | `name: string` (a tool key) | string type; `Config.Tools[name].kind == "consume"`; `Store.IsReady`; 0.3 s cooldown per player; not `InArena`; living character; `potionUsable`; bag count > 0 (taken last). Price, duration and effect come from Config, never from the client. |
| `BuyTool` (existing) | client to server | `name: string` | unchanged, plus the stage price and the `requires` check. |
| `EquipTool`, `Drop` (existing) | client to server | unchanged | unchanged. The client no longer offers Equip for potions, but the server path stays for compatibility. |

No new server-to-client remote. State reaches clients through attributes.

### Save document

| Field | Type | Default | Notes |
|---|---|---|---|
| `effects` | `{ [effect key: string] = seconds left: number }` | `{}` | Seconds left, paused while offline. Only entries with >= 30 s. Never more than the drinks bought. |

`tools` is unchanged: new potions save automatically because their names are in `Config.Tools` (`ServerScriptService.GardenStore.lua:358-367`). **Migration:** old documents have no `effects` field, so they load as `{}` through `Store.Default` and nothing is restored. Adding a field needs no schema bump (`ServerScriptService.GardenStore.lua:8-12`). Old tool counts above any number stay as they are. No potion is removed, so `RETIRED_PRICES` needs no new entry.

### Attributes

| Attribute | On | Written by | Read by |
|---|---|---|---|
| `AquaAffinityEndsAt`, `AquaAffinitySpan` | Player | Potions (`drink`, `restoreEffects`), expiry sweep | GardenClient effect block; Boats and Swimming |
| `HasteEndsAt`, `HasteSpan` | Player | same | GardenClient effect block; Underground mining code |
| `Rebirths` (existing) | Player | Progression | price on server and client |
| `Shards` (existing) | Player | Progression | shop affordability |
| `Upgrade_spawnTier` (existing) | Player | Upgrades | Luck card state |
| `InArena` (existing) | Player | CrownArena | drink refusal; effect block hide |
| `SwimState` (owned by Boats and Swimming; swimming means not `"Dry"`) | Character | Boats and Swimming | `applyWalkSpeed`, `potionSweep` |
| `UndergroundFloors` (owned by Underground and Ores) | Player | Underground and Ores | Haste buy and drink checks; Haste card state |

## Edge cases and abuse

- **Wrong name on the wire.** `DrinkTool("TNT")` or any non-consume name is refused (rule 14). `BuyTool` already rejects unknown names (`ServerScriptService.GardenService.Tools.lua:92-94`).
- **Spam.** Each drink costs one potion and is rate-limited to one per 0.3 s per player. No duplication path: the count is taken before the effect is applied, and only after every check passed.
- **Price tampering.** The client never sends a price. The server prices from its own `Rebirths` attribute.
- **Buying before a rebirth to dodge the price rise.** Allowed. Potions bought at stage s cost 1/2.2 of the stage s+1 price, so the most anyone saves is about 55% on the potions they pre-buy, bounded by their shards. It rewards planning and costs nothing to the economy.
- **Rebirth mid-purchase.** Remote handlers on the server run one at a time, and both `buyTool` and `doRebirth` are yield-free (`ServerScriptService.GardenService.RebirthLock.lua:63-67`), so a purchase uses one stage's price, never a mix.
- **Rebirth mid-effect.** The effect keeps running (rule 19). Luck re-reads the new stage's ceiling on every spawn.
- **Leave mid-effect.** Saved and resumed (rule 20). The save runs before `effects[player] = nil` (`ServerScriptService.GardenService.lua:711`, `:721`). Check in a test that `Humanoid.Died` does not fire on leave, or effects would be cleared before capture.
- **Server shutdown.** `BindToClose` runs the same leave path (`ServerScriptService.GardenService.lua:770-781`), so effects save.
- **Royal Race Aether.** The decree writes `effects[p].Aether` directly (`ServerScriptService.GardenService.CrownDecree.lua:111-141`). The `drunk` cap means only potion-bought time is saved.
- **Dying to keep a potion.** Dying still clears every effect (rule 18). Leaving to avoid a pending fall death is not useful: the fall kill lands 2 s after impact (`Config.FallHoldTime`), and leaving loses the session anyway.
- **Drinking while dead.** Refused, so a potion drunk in the death window cannot carry into the new body.
- **Arena.** Drinking is refused, effects are ignored and the badges are hidden (rules 14, 17, 31). Duels stay equal (`ReplicatedStorage.CrownConfig.lua:16-19`).
- **Haste without the Underground.** Cannot be bought. If a player still holds one (a dev grant), drinking is refused until they own the Underground, so it is never wasted.
- **Luck at the ceiling.** Drinking is refused with the "save it" line, so no Luck potion is ever wasted. The potion stays in the bag for the next stage.
- **Aqua Affinity on a boat or on land.** No effect; the timer still runs. If artillery sinks the boat, the player is Swimming and Aqua applies at once.
- **Speed in water.** Speed does not apply while Swimming, so Speed x Aqua cannot stack to 38 studs/s.
- **Speed and the old "swept" test.** Walk 20 x 1.6 = 32 studs/s is above `DrownSweptSpeed` 30 ([[Config and Fruit Data]] gotcha 24). Harmless today. [[Boats and Swimming]] removes that test with the currents.
- **Invisible players.** Cannot be hit by bats and cannot swing or fire (rule 27, matching [[Bats]] B27 and [[Weapons and Artillery]] rule 14). Their fruit can still be hit by explosives. Invisibility lasts 60 s, so it is a short escape, not a shield.
- **Griefing.** No potion acts on another player (rule 3), so potions add no griefing path. Stealing stays removed.
- **Mobile.** Drink is one tap in the Inventory. On touch the effect block sits at the top left (UI 3, D-025), away from the jump button and the touch action slot. Text sizes have floors of 9 to 12 px. (This line said "left of the jump button" before; corrected by the player-path pass.)
- **Corrupt save.** `sanitise` drops unknown effect names, non-numbers, NaN, values <= 0 and values over 86,400 s.
- **Legacy rebirth counts above 8.** `Config.StageFor` clamps to stage 9.
- **Server authority.** Every rule above is decided on the server. The client only draws attributes and sends tool names.

## Performance

Limits from [[Roblox Platform Constraints for the Expansion]]: under 50 KB/s per client, about 500 remote calls per second per client, mobile emitters capped at 100 particles per second, one DataStore key holds 4 MB.

- **Parts.** Two new templates of 5 parts each, in ReplicatedStorage (not streamed, not in Workspace). Zero new parts in Workspace.
- **Per-frame client work.** The effect block's Heartbeat loop already iterates every `Config.Effects` key (`StarterPlayer.StarterPlayerScripts.GardenClient.lua:559-609`): 8 entries instead of 6. Text writes stay guarded (`:603`).
- **Shop.** Today every potion card without art runs its own RenderStepped spin while the shop is open (`StarterPlayer.StarterPlayerScripts.ShopMenu.lua:289-297`); 5 of 6 potions do. With art for all 8 potions, potion cards run 0 spins. Only TNT keeps one.
- **Server.** The 0.25 s expiry sweep (`ServerScriptService.GardenService.lua:793-821`) and the 0.2 s Ambient sweep gain one table lookup per player. `potionSweep` toggles one emitter; it builds it once per character.
- **Particles.** Aqua wake: one emitter per swimming player with Aqua, 12 per second. Drink puff: 12 particles once. Haste spark: 6 per pickaxe hit. All under the mobile cap.
- **Network.** A drink writes 2 attributes. `DrinkTool` is at most about 3.3 calls per second per player because of the cooldown. Prices are computed locally from attributes already replicated, so no price traffic.
- **Save.** `effects` adds at most 8 numbers (under 200 bytes). The self-test ceiling of 8,192 bytes (`ServerScriptService.GardenStore.lua:1209`) still holds. No extra writes: drinks use `Store.Touch`, as today.

## Test plan

Set `workspace.StudioNoSave = true` before every Play test, because playtests write the real save otherwise (`ServerScriptService.GardenStore.lua:572-575`). Give shards with the Studio-only `DevSetShards` attribute (`ServerScriptService.GardenService.lua:653`). Set a test stage by setting the player's `Rebirths` attribute from the server command bar. Remove `StudioNoSave` at the end and confirm the real save is unchanged (byte length and hash, as in NOTES, 4 Oct).

| # | Checks rule | Steps | Pass when |
|---|---|---|---|
| T1 | 1, 2, 29 | Open Shop. | Four tabs: Potions, Weapons, King, Robux; Potions is selected first. 8 cards (7 until Haste ships) in order Speed, Jump, Feather Falling / Aether, Aqua Affinity, Haste / Luck, Invisibility. TNT is on the Weapons tab, not here. Console has no "skipping its card" warning. |
| T2 | 4-8 | For Rebirths 0, 2, 5, 8, read every card's price. | Matches the price table: 75/75/110/150, 380/380/560/750, 3,900/500/5,800/7,800, 41,000/500/62,000/82,000. |
| T3 | 9 | With 1,000 shards at Rebirths 5, fire `BuyTool("SpeedPotion")` from a test client. | Refused ("Not enough shards!"), shards still 1,000. With 3,900 shards it succeeds and leaves 0. |
| T4 | 10 | Shop open; change `Shards` above and below a price; change `Rebirths`. | Price colour flips white/red and prices change on the next frame. |
| T5 | 11, 25 | No Underground: tap Haste Buy, then fire `BuyTool("HastePotion")` and `DrinkTool("HastePotion")`. Then set `UndergroundFloors = 1`. | Card shows "Needs the Underground"; both remotes refused with "Buy the Underground first!". After the attribute, the card shows a price and buying works. |
| T6 | 12 | Carry a fruit. Open Inventory, tap Aqua Affinity, tap Drink. | Fruit still held. Count drops by 1. Badge appears. Green "Aqua Affinity for 10 min!". |
| T7 | 13 | Equip a potion through the old path (fire `EquipTool`), press Use. | Same result as T6. |
| T8 | 14 | Fire `DrinkTool` with: "TNT"; a potion you own 0 of; 10 times in one frame (`for i = 1, 10 do remote:FireServer("SpeedPotion") end`), then 10 times spread evenly over 0.5 s; while `InArena`; while dead. | Each refused with the right message (or silently for the cooldown). The one-frame burst gives exactly 1 drink (count -1). The 0.5 s spread gives exactly 2 drinks, one per 0.3 s cooldown (count -2). The count drops by exactly the number of drinks that succeeded. (M4 review: the old wording, "only the first legal call in the burst drinks" for calls spread over 0.5 s, contradicted the 0.3 s cooldown.) |
| T9 | 15, 16 | Drink Speed, wait 10 s, drink another. | `SpeedEndsAt - now` is about 590 s; the bar refills to full. Each duration matches rule 16. |
| T10 | 17, 31 | Drink Speed and Aqua, enter a duel. | Arena speed applies, not 32. Effect block hidden. Leaving the arena shows the block again with time still counting. |
| T11 | 18 | Drink Jump, reset the character. | All badges gone; `JumpEndsAt` nil. |
| T12 | 19 | Drink Aether, rebirth (with requirements met by dev cash). | Aether still running after rebirth; potions still in the bag. |
| T13 | 20 | Drink Aether (30 min). Wait 60 s. Use the Dev Panel Round trip. | After reload, `AetherEndsAt - now` is within 2 s of the time left before the trip. With 20 s left before the trip, nothing is restored. |
| T14 | 20 | Start a Royal Race decree with no Aether bought, Round trip mid-race. | Aether not restored after the trip. |
| T15 | 21, 24 | On land: drink Speed, read WalkSpeed (32). Enter the sea (`SwimState` not Dry). Then drink Aqua. | In water Speed does not apply; with Aqua, swim speed is 30 (2.5 x the base 12) and the stamina value does not drop over 30 s of swimming. On a boat seat nothing changes. |
| T16 | 26 | Spawn Rarity below the ceiling: drink Luck, watch 20 spawns. Then set Spawn Rarity to the ceiling and drink again. | First: spawns include the fruit one level up, marked `Lucky`, never a merge-only fruit, never a locked rarity. Second: refused with "Spawn Rarity is maxed...", count unchanged, card info reads "Spawn Rarity maxed". |
| T17 | 27 | Two-player test: player A invisible, player B swings at A with a bat (when [[Bats]] ships); then A tries to equip a bat and to draw a weapon at sea. | B's hit is refused with the Guard flash; A cannot equip the bat or draw a weapon. |
| T18 | 28 | Drink any potion with a second client nearby. | Both hear the gulp and see a puff in the potion's colour. |
| T19 | 30 | Drink all 8 potions. Check at 1600x900, 1366x768, and Device Emulator 812x375 and 667x375 (touch), carrying a fruit, then with the bat equipped (touch action slot showing Swing). Run the overlap check from [[Consistent Image-Based UI Kits in Roblox]] (compare every visible HUD rectangle). | No intersection between the effect block and ClearButton, CarryButtons, CarryInfo or JumpButton. 4 per row on mouse, 3 per row on touch. |
| T20 | 3 | Review: grep the new code for writes to any other player's attributes, folders or balances. | None. |
| T21 | save | After T13, inspect the saved document in Studio (with saving on in a throwaway test place, not the live place). | `effects` holds numbers only, under 86,400; payload self-test passes. |

## Dependencies and conflicts

- **[[Rarity Gating and Rebirth]].** Potions need its stage (rebirths + 1) and the spawn ceiling level for Luck. If it names them differently (for example its own D table or `G.spawnCeiling`), use its names. Rebirth keeps tools (D-009), so potions carry over. Prices rise one step per rebirth; the rebirth panel need not mention it. Legacy saves with more than 8 rebirths clamp to stage 9.
- **[[Bats]].** Invisibility blocks bat hits both ways (rule 27, Bats B27, D-030). Strength and Resistance potions are rejected. If bat griefing becomes a problem, fix it with bat rules (cooldowns, no-hit zones), not with a potion.
- **[[Explosives and Fruit Health]].** Explosives move to the Weapons tab and use the same `Config.ShardPrice` (D-017, D-022). No potion protects fruit; the Lock dome does that.
- **[[Weapons and Artillery]].** Shop tab budget: four tabs fit at 0.194 of the panel each. A fifth would shrink every tab to about 0.15 and the rotated labels would be hard to read. Resolved in the [[Integration Review]] (D-022): Potions, Weapons (sea weapons and garden explosives together), King (auras and the blade-skin card), Robux. If artillery sinks a boat, its riders are Swimming and Aqua Affinity applies.
- **[[Underground and Ores]].** Haste reads its pickaxe swing rate and its ownership attribute `UndergroundFloors`, and its mining hit code fires the Haste spark. Haste ships with the second ore band, not with Floor 1 (D-032). Requirement on that system: the Underground must be fully readable with no potion, because Night Vision is rejected.
- **[[Pets]].** No interaction. Potions never affect pets, taming odds or pet income.
- **[[Boats and Swimming]].** Owns the base swim speed, the stamina numbers, the `SwimState` attribute and `Config.BoatPrice`. Aqua Affinity multiplies its base (x 2.5 speed = 30 studs/s, x 0 drain; D-029). Its SeaStatus bar sits at the bottom centre above the hands stack, never in the effect block's rectangle. Swim speed is written inside `applyWalkSpeed` so there is one writer; Speed must not apply while Swimming.
- **[[Potions]].** This note.
- **[[Plot Overhaul and Environment]].** No dependency in v1. If the optional potion stall is approved, it is one Stud Shop booth placed by that system.
- **[[King UI Overhaul]].** Two touch points. (1) The Shop tab column changes from 3 to 4 tabs (Potions, Weapons, King, Robux); any King tab art change must keep the shared tab image and the rotated label. (2) The Inventory overlay button is still code-drawn; converting it to the kit's green action button belongs to the UI overhaul, and Drink inherits it. Found while reading: the Royal Race notice that grants Aether contains an emoji (`ServerScriptService.GardenService.CrownDecree.lua:130`); that string belongs to the King UI Overhaul's emoji sweep.
- **[[Royal Combat]].** Drinking is refused in the arena, effects are ignored there (unchanged), and the effect block hides so it cannot crowd the touch combat buttons. The Royal Race Aether grant is unchanged and is never saved.

## Open questions for Samarth

Each has a recommended default, so building can start now.

1. **Prices grow with your stage** (1 unit = 25 x the stage's shards per frontier merge). Today's flat 5,000 is unaffordable for new players and trivial late. **Default: yes, scale them.**
2. **Aether is affordable from stage 1** (150 shards, then 350 at stage 2). Today it costs 10,000. **Default: yes.** The sky and the King's Throne become an early goal that still costs two thirds of stage 1's shards.
3. **Aqua Affinity removes swim fatigue completely** (drain x 0) rather than reducing it. **Default: completely**, because you asked for it to make swimming "super fast" and fatigue is the problem it solves.
4. **Aqua Affinity never costs more than half a boat** (cap 500 at a 1,000-shard boat). **Default: yes.** Without the cap a single potion would cost more than a permanent boat from stage 4 on.
5. **Haste ships with the second ore band** (changed from "with the Underground" in the [[Integration Review]], D-032), and only Underground owners can buy it. The v1 mine fills in about 3 minutes, so Haste would be bought for nothing. **Default: yes.** Until then the catalogue is 7 potions.
6. **A Potions tab** in the Shop. **Default: yes, as Potions, Weapons, King, Robux** (D-022): the old Tools tab becomes Potions and every combat item goes to Weapons.
7. **Drink straight from the Inventory**, instead of Equip then Use. **Default: yes.** You can drink while carrying a fruit.
8. **Potion time is saved when you leave** and resumes when you rejoin (dying still ends it). **Default: yes.** Prices now reach 82,000 shards at stage 9, so losing a 30-minute Aether to a disconnect would feel like a bug.
9. **Luck is refused once your Spawn Rarity is maxed for the stage**, instead of today's "tilt the odds" at the top level. The model shows the tilt saves almost no time, and spawning merge-only fruit would break D-007. **Default: yes, refuse and say "save it for after your next rebirth".**
10. **Speed does not work in water.** **Default: yes**, so Speed and Aqua Affinity cannot stack into near-boat speed.
11. **Invisibility stays** even though stealing is gone. Its job now: slipping past bat players (it blocks bat hits both ways, D-030) and hiding the King's aura. **Default: keep it.**
12. **Shop art for Speed, Jump and Invisibility** (requests 5 to 7), so every potion card is drawn art instead of a spinning bottle. **Default: yes.**
13. **Robux potions.** **Default: none in v1.** If wanted later, sell only fixed potions (never Luck), because Luck changes odds and would become a paid random item needing shown odds and a PolicyService check ([[Roblox Platform Constraints for the Expansion]]).
14. **Haste strength** x 1.5 swing rate. **Default: 1.5.**
15. **A drinking animation.** **Default: no.** Drinking from the bag has no bottle in hand, and an animation would need you to publish it (D-005).
16. **A potion stall in the world** (a Stud Shop booth that opens the Potions tab). **Default: not in v1.** Revisit after the plot overhaul.

No decision in the [[Decisions Log]] is contradicted. The [[Integration Review]] added D-017 (one shared shard price), D-019 (Luck refused at the ceiling, this note's rule), D-022 (four Shop tabs), D-025 (touch layout), D-029 (Aqua 2.5x, priced here), D-030 (Invisibility blocks bats) and D-032 (Haste with the second ore band). Two older ones are used directly: D-010 (shards per frontier merge) is the price unit, and D-007 (the top 2 fruits never spawn) is why Luck is capped at the spawn ceiling. One addition for the [[Stage Model]]: its note that Luck "adds roughly one Spawn Rarity level for 5 minutes" can now say it shortens a stage by 2.5 to 3.4 minutes when drunk early (stages 3 to 8) and by almost nothing after the spawn ceiling.
