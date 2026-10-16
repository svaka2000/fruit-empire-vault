---
tags: [system, design]
date: 2026-10-04
status: proposed
depends_on: [Rarity Gating and Rebirth, Boats and Swimming, Explosives and Fruit Health, Bats, Royal Combat, Plot Overhaul and Environment]
---
# Weapons and Artillery

Everything in this note is a **proposal** unless it cites live code (`File.lua:LINE`, in `~/fruit-empire/src-export-2026-10-04`), a decision (D-0xx in [[Decisions Log]]) or a research note. Source of truth: [[Brain Dump 2026-10-04]].

## Why

Samarth's lines that ask for this, verbatim from [[Brain Dump 2026-10-04]]:

> "also add artilery and combat like tnt, nukes, etc. make this super planned and tell me exactly what assets you need dont make slop unless it looks clean and studded. the animations need to be top tier."

> "boats can fit up to two people and can also be broken by the artillery i mentioned earlier like weopons - look at the game called sky rivals to get ideas for the artillery you can create."

> "also all Uis you make should follow the UI that we created, if you need any UI stuff then we can create them using chatgpt or gemini image generation following references of current UI"

> "all the UI should never overlap and also everything should be super clean."

> "I dont like the emoji-slop that you pushed on the sky island kings feature."

> "everything needs to correspond with the stage of the game a player is in. you can use rebirths to control and organize this."

What the system is for:
- It adds **four ranged weapons** that work **only at sea**, between the main island and the forest. They break boats and push players, the way SkyRivals' fights are won by knockback and broken structures ([[Sky Rivals - Weapons and Artillery]]).
- It never touches anyone's fruit, cash, shards, pets or Index. Stealing was removed from the game on 3 Oct 2026, and this system does not bring it back in another form.
- Rebirths unlock the stronger weapons, and prices follow the stage, so the system grows with the player like the rest of the economy ([[Stage Model]]).
- Player-versus-player fights are **opt-in** through a flag, so a new player crossing to the forest is never sunk against their will.

### Words used in this note
- **Stage (s):** 1 + the player's rebirths. Stage 1 is Common, stage 9 is Transcendent ([[Stage Model]], rule 1).
- **M_s:** shards dropped per merge into the newest rarity at stage s: 3, 7, 15, 32, 70, 155, 340, 748, 1,646 (D-010). This note prices everything in multiples of M_s.
- **R_s:** shards a player earns per minute while in stage s. It comes from the same model as the Stage Model (see Numbers).
- **Full-plot $/s (P_s):** the most a plot earns in stage s ([[Stage Model]]). Shown for reference; weapons are not priced in cash.
- **Weapon:** one of the four items below. **Charge:** one shot (one bomb, one shell, one call). **Pack:** what one purchase gives.
- **Combat Zone:** the sea ring where weapons work (Rule 6). **No-Fire Zone:** everywhere else, plus marked boxes at the docks.
- **War Flag:** an on/off switch per player. Only **flagged** players can hit each other or each other's boats. A **flagged boat** is a boat whose driver is flagged.
- **Projectile:** a bomb or shell in flight. It exists only as numbers on the server; each client draws its own copy.
- **Splash radius (r):** how far a blast reaches. **Falloff:** a blast is weaker away from its centre. **Direct hit:** the projectile itself touched the target.
- **Knockback:** a push applied to a player's body. Weapons push; they never lower health.
- **BoatMaxHP:** a boat's full health, set by [[Boats and Swimming]]. Weapon damage to boats is written as a percent of it.
- **PvE target:** something the game owns, not a player: the practice buoys and the Rotten Fort (Phase 3).
- **Practice Range:** a pad by the island dock where every weapon fires free practice shots at buoys.
- **Draw / holster:** take the weapon into your hands / put it away.

## Player view

1. **Stage 1 (0 rebirths).** The Shop has a new **Weapons** tab. All four cards are visible but greyed, each with "Rebirth 1", "Rebirth 2", "Rebirth 4" or "Rebirth 6" where the price would be. At the dock there is a **Practice Range**: stand on the pad and the Weapon Bar appears with all four weapons, free, aimed at six floating target buoys. The player sees exactly what rebirthing unlocks.
2. **After rebirth 1 (stage 2).** The Cherry Bomb card turns on, next to the garden TNT card that unlocks at the same rebirth. A pack of 4 bombs costs 85 shards. The player buys one.
3. **Going to sea.** The player swims or takes a boat out. Once past the beach into open water, the **Weapon Bar** slides up at the bottom centre: a flag slot, then one slot per weapon they own, each with its charge count.
4. **Firing.** They press 2 (or tap the slot) to draw the Cherry Bomb; key 1 is always the bat. Holding the mouse (or the Fire button on a phone) shows a thin white arc and a green landing ring on the water. Releasing throws the bomb. It bursts with a splash.
5. **Without the flag,** bombs only matter to PvE targets. Other players' bombs cannot touch this player or their boat, and the reverse.
6. **Raising the War Flag.** They press the flag slot. A 5-second countdown runs, then a red flag appears over their head and a red pennant on their boat. Now flagged players can push them and break their boat, and they can do the same back.
7. **A fight.** A flagged rival fires a Melon Mortar. A red warning ring appears on the water where the shell will land, and its inner disc fills over about 2 seconds. The player steers away. A shell that lands drains their boat's health bar by up to 35%. Three good shells sink a boat. The sunk boat's riders drop into the water and swim ([[Boats and Swimming]]). The shooter gets a toast "You sank Mia's boat." Mia gets "Kai sank your boat." Mia's flag drops at once, so she is not hunted again on the way home.
8. **Lowering the flag.** Pressing the flag again starts a 10-second countdown. Any hit dealt or taken restarts it. The flag is always down when a player joins.
9. **Later stages.** Rebirth 2 unlocks the Melon Mortar, rebirth 4 the Coconut Cannon, rebirth 6 the Starfruit Barrage: a call-in strike where five starfruit shells fall from the sky onto a marked circle after a 3-second siren.
10. **Phase 3, the Rotten Fort.** Every 20 minutes or so a toast says a Rotten Fort is drifting at sea. Anyone, flagged or not, can shoot it. It lobs shells back at boats near it. When it breaks, everyone who hit it gets shards scaled to their own stage.

## Rules

**A. The weapons and how to get them**
1. There are exactly four weapons: Cherry Bomb, Melon Mortar, Coconut Cannon, Starfruit Barrage. Their combat numbers are in the Numbers section and are the same for every player at every stage.
2. Weapon w can be bought only when the buyer's stage is at least its unlock stage: Cherry Bomb 2, Melon Mortar 3, Coconut Cannon 5, Starfruit Barrage 7. Stage only rises, so an unlocked weapon never locks again.
3. Weapons are bought only with shards, in the Shop's Weapons tab or at the Weapons Booth (which opens the same tab; renamed from "Armory booth" so it is never confused with the Royal Armory of [[Royal Combat]]). One purchase gives one pack. Price = `Config.ShardPrice(k_w, rebirths)` = k_w x M_s rounded to 2 significant figures (the one shared shard price, D-017), with s the buyer's stage at the moment of purchase and k_w = 12, 20, 24, 40. No Robux, gamepass or random box sells weapons or charges.
4. The server refuses a purchase when: the profile has not loaded; the player is mid-rebirth (`rebirthing[player]`, `ServerScriptService.GardenService.RebirthLock.lua:57`); the stage is too low; shards are below the price; or charges + pack size would exceed the weapon's maximum held (40, 30, 60, 5).
5. Charges are saved. Rebirth, death and leaving keep them. A charge is spent only when the server accepts a shot.

**B. Where weapons work**
6. The Combat Zone is every point whose horizontal distance from the world origin (0, 0) is from 400 to 985 studs. A shooter must also be at height y = -10 to 40. Projectiles may travel up to y = 260. The beach edge is r 380 (`ServerStorage.ShallowsBuilder.lua:212`) and the forest ground starts at r 995 (`ReplicatedStorage.GardenConfig.lua:1263-1266`), so the zone is open water only. Plots, the plaza, the island, the forest, the sky islands, the Royal Capital and the underground are all outside it.
7. Parts tagged `NoFireZone` mark boxes inside the ring where weapons are inert. Each boat summon pad gets one box reaching 40 studs around it.
8. A blast affects only targets that are themselves inside the Combat Zone and outside every `NoFireZone` box. A player on the beach next to a blast is not pushed.

**C. Drawing and firing**
9. A weapon can be drawn only when all of these hold: the profile is loaded; the character is alive; the root is in the Combat Zone or on the Practice Range pad; the player is not carrying a fruit or tool (`held[player]` is nil); the player is not `InArena`; `G.royalBusy(player)` is nil; the Invisibility effect is not active; and the player owns at least one charge of that weapon (the pad skips this last check). **One hand:** drawing a weapon first unequips the bat ([[Bats]] B6); equipping the bat holsters the weapon (D-025).
10. Drawing or switching takes 0.35 s. Only one weapon is drawn at a time. Leaving the zone or the pad holsters the weapon within 0.25 s.
11. While a weapon is drawn, the server refuses fruit pick-up, tool equip and stored-fruit equip.
12. The server accepts a shot only when: the weapon is the drawn one; its cooldown has passed; at least 0.15 s passed since the player's last accepted shot; the player has fewer than 3 projectiles of their own in flight; the server has fewer than 40 in flight; the aim is inside the weapon's range limits; and the aim point is inside the Combat Zone (practice shots: inside `PracticeRangeArea`).
13. The client sends only the aim point and a sequence number. The server computes the launch point from its own copy of the character and the launch velocity from `Ballistics` and `WeaponConfig`. A client-sent origin or speed is never used.
14. An Invisible player cannot fire. A driver can fire. A passenger can fire.

**D. Projectiles**
15. Projectiles are simulated only on the server, along a ballistic path stepped every Heartbeat.
16. A projectile bursts at the first contact with: a boat hull part; a part in `Main Island.Shallows` (stones, driftwood, islets; `ServerStorage.ShallowsBuilder.lua:233-246`) or a part tagged `SeaSolid`; a flagged at-sea character's root (a sphere of radius 2.5 studs); a buoy or the Fort; or the sea surface at y = -2.2 (`ServerStorage.ShallowsBuilder.lua:109`).
17. A projectile that leaves the Combat Zone, enters a `NoFireZone` box, or lives 6 s is deleted with a small puff and no effect (a "fizzle").
18. Practice projectiles hit only buoys and the sea surface. They skip the zone test of Rule 17 (the pad itself is on the beach, outside the zone) and instead fizzle when they leave the Practice Range box: one invisible part tagged `PracticeRangeArea` that covers the pad and the buoy water.

**E. Effects of a blast**
19. Each target within splash radius r of the burst point gets the weapon's centre value times f(d) = 1 - 0.7 x min(d / r, 1), where d is the distance from the burst point. A direct hit gets 100%.
20. **Fruit:** weapons never damage fruit. Rules 6 and 17 keep every projectile at r 400 or more, while the plots sit about 200 studs from the centre ([[World Tour]]), so the nearest plot edge is more than 100 studs away. The blast code also passes a fruit scale of 0.
21. **Players:** no health is ever lost. Knockback velocity = horizontal direction away from the burst x K_w x f(d), plus an upward lift of L_w x f(d). It applies only when the shooter and the victim are both fully flagged, the victim is not seated, not `InArena`, and has not been knocked by any weapon in the last 1.5 s. A shooter is never knocked by their own blast.
22. **Boats:** damage = B_w x f(d) x `MaxHP`, sent through `G.damageBoat(boat, amount, shooter, key)` ([[Boats and Swimming]] R44). It applies only when the shooter is flagged, the boat's driver is flagged, the shooter is not aboard that boat, the boat has a driver, the boat's spawn protection (`ProtectedUntil`, 15 s after being summoned, Boats R30) is over, and the boat is not in a dock zone (40 studs from a pad). Seated riders are never knocked, by weapons or bats; their boat takes the hit.
23. A flagged passenger in an unflagged boat fires shots that affect only PvE targets.
24. PvE targets take damage from every shooter, flagged or not.
25. Pets are never targets of anything in this system.

**F. The War Flag**
26. The flag is down on every join and is never saved.
27. Raising: press the flag slot, wait 5 s, the flag is up. During those 5 s the player can neither hit nor be hit by players.
28. Lowering: press again, wait 10 s, the flag is down. Any player hit dealt or taken during the 10 s restarts the countdown. The flag stays up until the countdown completes.
29. When a player's boat is sunk by another player, that driver's flag drops at once, with no countdown.
30. A flagged player shows the Flag Up icon above their head to everyone within 180 studs. A flagged boat shows a red pennant.

**G. Practice Range**
31. On the Practice Range pad, all four weapons can be drawn, locked or not, owned or not. Practice shots cost no charges, use the normal cooldowns, and collide only with buoys and water. They never push or damage anything.

**H. Rewards and messages**
32. Player-versus-player actions pay nothing and move no currency.
33. A boat sunk by players credits the last player who damaged it within the previous 10 s. That player gets a Notice toast; the driver gets a Notify toast naming them. Nothing else is given or taken.
34. The Melon Mortar and the Starfruit Barrage show a warning ring at the landing point to every client, from the moment of firing until impact.
35. Only one Starfruit Barrage resolves at a time per server. A new one needs 20 s since the last one ended and 60 s since the same player's last call.

**I. The Rotten Fort (Phase 3)**
36. A Fort spawns at one of six markers (`SeaFortSpot1..6`, at r 650 to 900, each at least 150 studs from a summon pad) every 20 min +/- 3 min, first at 8 min after server start. It needs 2 or more players in the server. It does not spawn within 180 s of `RoyalState.NextContestAt` (`ServerScriptService.GardenService.CrownContest.lua:50`) or while a Crown Contest runs; it waits 60 s and checks again.
37. A Notice toast announces it 60 s before it appears. It stays 4 minutes, then sinks.
38. Fort HP = 150 x clamp(players in server, 2, 12) fort units. After 6 s with no damage it regains 2% of its maximum per second.
39. Every 3.5 s it lobs one shell at an occupied boat within 140 studs (else at a swimmer within 140 studs) with a 1.6 s warning ring: 12% of BoatMaxHP to the boat, knockback 60 / lift 25 to a swimmer. These shells ignore flags, because the Fort is PvE.
40. Pay, to each player present when it breaks or sinks: min(60 x M_s, 0.18 x M_s x units they dealt), rounded down, plus 6 x M_s if it broke and they dealt at least 30 units. s is that player's stage at payout. M_s comes from `Config.ShardDropFor` through `Config.ShardReward` (D-017); WeaponConfig keeps no copy of the D-010 table.

## Numbers

### Inputs from the Stage Model
| Stage | Rarity | Rebirths | Full-plot $/s (P_s) | M_s (D-010) | R_s, shards per minute in this stage* | Weapon unlocked here |
|---|---|---|---|---|---|---|
| 1 | Common | 0 | 707 | 3 | 15.7 | none (Practice Range only) |
| 2 | Uncommon | 1 | 430K | 7 | 39.2 | Cherry Bomb |
| 3 | Rare | 2 | 3.7B | 15 | 103 | Melon Mortar |
| 4 | Epic | 3 | 3.6T | 32 | 172 | |
| 5 | Legendary | 4 | 1.8Qa | 70 | 315 | Coconut Cannon |
| 6 | Mythic | 5 | 2.5Qi | 155 | 759 | |
| 7 | Astral | 6 | 265Qi | 340 | 921 | Starfruit Barrage |
| 8 | Eternal | 7 | 506Sx | 748 | 2,600 | |
| 9 | Transcendent | 8 | 24.9Sp | 1,646 | 6,670 (first hour) | |

*R_s = shards earned inside the stage / minutes in the stage, median of 7 seeds, attentive player using pets and ores, run with `~/fruit-empire/sim/economy.py` and the same requirements as `final.py` on 4 Oct 2026. Stage 9 has no rebirth gate, so its value is the first 60 minutes (median of 5 seeds). Weapons are priced in shards, not cash: the rebirth gate measured in seconds of P_s runs from 80 s at stage 1 to 11,265 s at stage 8 (D-011 divided by P_s), so a fixed number of seconds of income would cost a stage-1 player 37% of their rebirth for one 30-second pack. M_s stays a near-constant share of shard income (R_s / M_s = 2.7 to 6.9).

### Combat numbers (the same at every stage)
Flat on purpose. A stage-9 player and a stage-2 player sink the same boat with the same number of hits. This mirrors the arena rule that nothing bought changes a fight (`ReplicatedStorage.CrownConfig.lua:16-19`).

| | Cherry Bomb | Melon Mortar | Coconut Cannon | Starfruit Barrage |
|---|---|---|---|---|
| Kind | thrown lob | artillery lob | direct-fire launcher | strike called from the sky |
| Aim | landing point 15 to 70 studs away | landing point 40 to 160 | direction | landing circle centre 30 to 200 away |
| Flight | tau = 0.6 + 0.8 x (d / 70) s: 0.77 to 1.4 s | tau = 1.4 + 0.8 x (d / 160) s: 1.6 to 2.2 s | 170 studs/s, lives 1.4 s (about 238 studs) | first shell lands 3.0 s after the call (each shell falls its last 220 studs at 140 studs/s), the other four 0.32 s apart |
| Gravity (studs/s^2) | 140 | 120 | 30 | none (straight down) |
| Splash radius r | 6 | 12 | 5 | 8 per shell; shells land within 9 studs of the centre |
| Boat damage B (share of BoatMaxHP at centre) | 10% | 35% | 20% direct, 8% splash | 30% per shell |
| Knockback K / lift L at centre (studs/s) | 50 / 22 | 80 / 34 | 100 / 28 direct, 40 / 14 splash | 70 / 30 per shell |
| Cooldown | 1.0 s | 4.0 s | 0.5 s between shots, clip of 3, 3.5 s reload | 60 s per player; one per server; 20 s server gap |
| Pack | 4 bombs | 3 shells | 6 shots | 1 call |
| Max held | 40 | 30 | 60 | 5 |
| Warning ring for everyone | no | yes | no | yes |
| Fort units at centre (Phase 3) | 28 | 63 | 38 | 76 per shell |

For scale, the arena's moves push 40 / 16 (Strike), 74 / 26 (Heavy) and 92 / 32 (Slam) (`ReplicatedStorage.CrownConfig.lua:92-145`). The Cannon's direct hit (100) is the strongest single push in the game, and it needs a direct hit on a moving target. SkyRivals' Fireball is about 50 HP on a direct hit with a 2-block break radius, but its numbers are wiki guesses ([[Sky Rivals - Weapons and Artillery]]), so these are tuned for Fruit Empire, not copied.

### Prices (shards), price = `Config.ShardPrice(k_w, rebirths)` at the buyer's stage
Rounded to 2 significant figures (nearest 5 below 100), the one shared shard price (D-017). "min" = the price divided by R_s from the [[Integration Check]] re-run (15.2, 36.8, 89.6, 142.5, 246, 577, 758, 2,023 and about 5,500 shards per minute for stages 1 to 9).

| Stage | Cherry Bomb, 12 x M_s (4 bombs) | Melon Mortar, 20 x M_s (3 shells) | Coconut Cannon, 24 x M_s (6 shots) | Starfruit Barrage, 40 x M_s (1 call) |
|---|---|---|---|---|
| **1** | locked | locked | locked | locked |
| 2 | 85 (2.3 min) | locked | locked | locked |
| **3** | **180 (2.0 min)** | **300 (3.3 min)** | locked | locked |
| 4 | 380 (2.7 min) | 640 (4.5 min) | locked | locked |
| 5 | 840 (3.4 min) | 1,400 (5.7 min) | 1,700 (6.9 min) | locked |
| **6** | **1,900 (3.3 min)** | **3,100 (5.4 min)** | **3,700 (6.4 min)** | locked |
| 7 | 4,100 (5.4 min) | 6,800 (9.0 min) | 8,200 (10.8 min) | 14,000 (18.5 min) |
| 8 | 9,000 (4.4 min) | 15,000 (7.4 min) | 18,000 (8.9 min) | 30,000 (14.8 min) |
| **9** | **20,000 (3.6 min)** | **33,000 (6.0 min)** | **40,000 (7.3 min)** | **66,000 (12.0 min)** |

Stage 7 reads high because Astral has only 5 fruits, so fewer frontier merges drop shards there (R_7 / M_7 = 2.2, the lowest).

**Effect on the Stage Model.** A player who buys two packs of their best weapon per stage spends, for example, 600 shards at stage 3 (about 12% of the 5,098 the re-run earns there) and 7,400 at stage 6 (about 13% of 58,965). Shards never gate a rebirth ([[Stage Model]]), so stage lengths do not change. A heavy buyer reaches the 100K underground a few minutes later. Weapons are also a late-game shard sink, which the Stage Model asks for ("Late shards inflate").

### Time to sink one boat (one shooter, every shot at centre, no repair)
| Weapon | Hits | Time | Packs | Cost at stage 6 |
|---|---|---|---|---|
| Cherry Bomb | 10 | about 10 s (10 throws 1.0 s apart, plus flight) | 2.5 | 4,750 |
| Melon Mortar | 3 | about 10 s (shells at 0, 4 and 8 s, plus 2 s flight) | 1 | 3,100 |
| Coconut Cannon | 5 direct | about 6 s (3 shots, 3.5 s reload, 2 shots, plus flight) | 0.83 | 3,100 |
| Starfruit Barrage | 4 of 5 shells | about 4 s from the call | 1 | not unlocked (14,000 at stage 7) |

Real numbers are slower. A boat at 42 to 48 studs/s ([[Swimming Fatigue and Boats in Roblox]], proposal 17) moves 67 studs during a 1.4 s bomb flight and 99 studs during a 2.2 s mortar flight, and the mortar shows its ring the whole time. [[Boats and Swimming]] R46 regenerates a boat at 20 HP per second (2% per second) after 8 s without damage, and 100 HP per second in a dock zone, so a lone Cherry Bomb thrower who misses often cannot sink a boat that keeps moving.

### Limits
| Limit | Value |
|---|---|
| Shooter height band / projectile ceiling | y -10 to 40 / y 260 |
| Combat Zone radius | 400 to 985 studs from (0, 0) |
| No-fire box around each summon pad | 40 studs |
| Own projectiles in flight / server-wide | 3 / 40 |
| Gap between accepted shots (any weapon) | 0.15 s |
| Gap between any two WeaponRequest calls | 0.1 s (extra calls dropped) |
| Draw / switch time | 0.35 s |
| Knockback immunity after a weapon push | 1.5 s |
| Boat spawn protection | 15 s after summon (owned by [[Boats and Swimming]] R30) |
| Flag raise / lower | 5 s / 10 s (restarts on any player hit) |
| Projectile lifetime | 6 s |
| Clients draw projectiles and blasts within | 600 studs of their camera |

### The Rotten Fort (Phase 3)
Fort units per hit at centre = round(9.5 x k_w / pack size), so every weapon returns about 7.1 units per M_s spent at an average falloff of 0.75. Pay per unit = 0.18 x M_s, so shooting the Fort returns about 1.28 M_s per M_s of ammo, plus the completion bonus, up to a cap.

| Stage | Pay per unit | Cap per Fort (60 x M_s) | Bonus (6 x M_s) | Net at the cap (pay - ammo) | Cap in minutes of R_s |
|---|---|---|---|---|---|
| **3** | 2.7 | 900 | 90 | about 285 | 8.7 |
| **6** | 27.9 | 9,300 | 930 | about 2,970 | 12.3 |
| **9** | 296 | 98,760 | 9,876 | about 31,500 | 14.8 |

Net at the cap: reaching the cap takes 333 units, which costs about 46.8 M_s of ammo; pay is 60 M_s + 6 M_s. So an active Fort fighter gains about 19 M_s per Fort, about 4 minutes of their stage's shard income every 20 minutes. Fort HP for 4 players is 600 units (150 each at an equal share), so a group of 4 breaks it with about 10 mortar shells each. The regen only starts after 6 s without damage, to avoid the "Why does it keep healing?" problem from Merge a Nuke's cities ([[Merge a Nuke - Explosives and Health Drain]]).

## UI

All new UI uses the image kit in [[UI Kit Inventory]]: FredokaOne via the family string `Font.new("rbxasset://fonts/families/FredokaOne.json")` (`StarterPlayer.StarterPlayerScripts.ShopMenu.lua:47`), white text, black `UIStroke` 2.5 to 3, `TextScaled`. No emoji or non-ASCII characters anywhere, including toast strings. Sizes below are at the HUD reference viewport of 1600 x 900; `HudScale` scales them (`ReplicatedStorage.GardenConfig.lua:1872`).

### 1. Shop: the Weapons tab
- **Tab:** `ShopMenu.WeaponsTab`, a copy of `ShopMenu.ToolsTab` (kit image 110917091182913, 100 x 132), with the label or icon child that ToolsTab uses swapped for "Weapons" (art request A1 if it is an icon). `Config.ShopTabs` becomes `{ "Potions", "Weapons", "King", "Robux" }`: today's Tools tab is renamed Potions ([[Potions]]), and this tab holds every combat item (decision D-022). Four tabs fit with the column rule in [[Potions]] UI 1 (0.194 of the panel each).
- **Page:** `ShopMenu.Panel.WeaponsPage`, a copy of `ToolsPage`, inside the kit Shop panel (93845982220486). ShopMenu fills it with a 3-column grid, using the same offset-and-explicit-canvas sizing as `fitTools` (`StarterPlayer.StarterPlayerScripts.ShopMenu.lua:162-190`). It holds 8 cards in one list sorted by unlock stage: Cherry Bomb and TNT (stage 2), Melon Mortar (3), Big TNT (4), Coconut Cannon (5), Mega TNT (6), Starfruit Barrage (7), Nuke (8). The four garden explosives come from [[Explosives and Fruit Health]] (its section 6.1) and buy through `BuyTool`; the four sea weapons buy through `WeaponRequest`. Every rebirth from 1 to 7 unlocks one or two cards here, which makes "rebirth to unlock more" visible.
- **Card art:** the kit rarity frame of the item's unlock tier, so a weapon reads as "the Rare weapon": Cherry Bomb UNCOMMON 136126447232134, Melon Mortar RARE 99182487208693, Coconut Cannon LEGENDARY 74773635384081, Starfruit Barrage ASTRAL 103897622776526 (explosives: TNT UNCOMMON, Big TNT EPIC 134546200735728, Mega TNT MYTHIC 125641842226730, Nuke ETERNAL 122828478750063). Cards keep the aspect of the Index cell template so the frames are not stretched.
- **Card layout (fractions of the card):** title at y 0.10 (height 0.12, Fredoka, stroke 2.5, max 30 px), e.g. "Melon Mortar"; spinning ViewportFrame preview at y 0.36 (0.42 x 0.34), built the way tool cards are (`StarterPlayer.StarterPlayerScripts.ShopMenu.lua:252-297`); stats line at y 0.58 (height 0.07, max 18 px), always starting with where the item works, e.g. "Sea | 3 shells, Boat 35%" (explosive cards read "Garden | Radius 14, 2 hits"); owned line at y 0.66 (height 0.07, max 20 px), "Owned: 7"; kit Buy button (97067256017679, shard crystal baked on its left) at y 0.82, 0.80 wide, price text offset right as on tool cards (`StarterPlayer.StarterPlayerScripts.ShopMenu.lua:299-330`).
- **States:**
  - Unlocked: as above. The price updates when `Rebirths` changes.
  - Locked: card `ImageColor3` (0.55, 0.55, 0.55); preview model drawn black at 0.6 transparency (the Index locked look); Buy button greyed, its text "Rebirth 2" (the rebirth count needed). Pressing it shows a local warning toast "Rebirth to Rare to unlock".
  - Full (charges at max held): Buy greyed, text "Full".
  - Not enough shards: the server answers with the existing "Not enough shards!" (`ReplicatedStorage.GardenConfig.lua:1849`).
- **No overlap:** the page lives inside the existing Shop panel, which already hides the other panels when it opens (`StarterPlayer.StarterPlayerScripts.ShopMenu.lua:612-633`).

### 2. The Weapon Bar (HUD, sea and Practice Range only)
- **Where:** a new transparent Frame `GardenUI.WeaponBar`, anchor (0.5, 1), position (0.5, 0, 1, -21): the bottom-centre slot that `CarryButtons` uses. Added to `Config.HudClusters` (`ReplicatedStorage.GardenConfig.lua:1863-1867`) so HudScale sizes it.
- **When it shows:** root in the Combat Zone or on the pad, **and** not carrying (`CarryingFruit` attribute nil), **and** not `InArena`. Rule 9 forbids drawing while carrying, so the Weapon Bar and the carry row are never on screen together: the bottom-centre slot always shows "what is in your hands". `CarryInfo` (y -87) is also hidden at those times, because it only shows while carrying (`StarterPlayer.StarterPlayerScripts.GardenClient.lua:231-232`).
- **Size:** slots of 80 x 80 with 12 px gaps; at most 5 slots (flag + 4 weapons) = 448 x 80, plus a 22 px label row above = 448 x 102. It spans y -21 to -123 from the bottom.
- **Slots, left to right:**
  1. **Flag slot.** Art A3: Flag Down or Flag Up. While a countdown runs, the remaining whole seconds show in the centre (Fredoka, max 30 px, stroke 3).
  2. **One slot per weapon with charges above 0**, ordered by unlock stage. Art A2. Charge count bottom-right (34 x 22, Fredoka, max 22 px, stroke 2.5). Key hint "2" to "5" top-left (18 x 18, max 16 px, stroke 2), hidden on touch devices. Key 1 stays the bat ([[Bats]] BatSlot), so the bat slot and this bar read as one hotbar (D-025).
  3. **Empty state:** with no charges at all, the bar shows the flag slot and one slot using the HUD Shop button art (71485100495903) scaled to 80 x 80. It opens the Shop on the Weapons tab.
- **Slot states:** drawn = kit Card selection outline (113285622864672) over the slot at 1.12x; cooling down = slot `ImageColor3` (0.45, 0.45, 0.45) with a `UIGradient` transparency sweep that uncovers full colour bottom to top as the cooldown ends (no numbers, no extra frame); Cannon reload uses the same sweep.
- **Label row:** a single Fredoka line on the kit Carry info NameBar (123470450245311, 168 x 27) centred above the slots. It shows "PRACTICE" on the pad, and the drawn weapon's name at sea.
- **Spacing check on a small phone:** at 667 x 375 the HUD scale floors at 0.55 (`ReplicatedStorage.GardenConfig.lua:1949-1957`), so the bar is 246 px wide and spans x 210 to 456. CornerHud (222 x 0.55 = 122 px from the left) and EffectStack (58 x 0.55 = 32 px at the right edge) stay clear of it.
- Slots are ImageButtons under GardenUI, so ButtonAnimator gives them the standard hover and press feel (`StarterPlayer.StarterPlayerScripts.ButtonAnimator.lua:207-217`). They are not in the magnetic side-button list.

### 3. Aiming
- **Arc preview (shooter only, world):** a Beam from a muzzle attachment to the landing point, width 0.3, white, transparency 0.35, with `CurveSize0` and `CurveSize1` computed from the launch velocity (Code plan, Ballistics). This is the DevForum Beam-parabola method cited in [[Sky Rivals - Weapons and Artillery]] [43].
- **Landing ring (shooter only, world):** a flat Neon cylinder, diameter 2r, transparency 0.55, in the weapon's tier colour from the kit table (Uncommon green, Rare blue, Legendary orange-gold, Astral cyan). It turns grey and Fire is blocked while the aim point is outside the zone or the range.
- **Warning ring (everyone, world):** for the Mortar and the Barrage. A Mythic-red (0.91, 0.26, 0.24) ring of radius r on the water, plus an inner disc that grows from 0 to r over the flight time. The Barrage adds a dark shadow disc that darkens over its 3.0 s warning.
- **Reticle (screen centre):** art A4, 36 x 36. Shown while the Cannon is drawn, and for every weapon on touch devices. **Hit marker:** art A5, 36 x 36, for 0.15 s when the server reports a hit.

### 4. Controls
| Input | PC | Gamepad | Touch |
|---|---|---|---|
| Select / draw weapon | 2 to 5 (1 is the bat) | R1 cycles weapons (L1 is the bat) | tap a slot |
| Aim | mouse point on the water | camera centre | camera centre (reticle) |
| Fire | hold left mouse to show the arc, release to fire (Cannon: click) | hold and release R2 | hold and release the Fire button |
| Holster | Q | B | tap the drawn slot |
| War Flag | G | hold D-pad down 1 s | tap the flag slot |

Keys are bound with ContextActionService only while the Weapon Bar is visible, so they never collide with E prompts or the arena's keys (`StarterPlayer.StarterPlayerScripts.RoyalCombat.lua:148-165`), which only work `InArena`, where weapons are always holstered.

### 5. The Fire button (touch only)
- Art A6, 128 x 128 (x0.72 on small screens), shown only when the Weapon Bar is visible and a weapon is drawn.
- It is drawn in the **touch action slot** that [[Bats]] UI 3 owns: one button left of the jump button, at the same spot the arena's Attack button uses (`StarterPlayer.StarterPlayerScripts.RoyalCombat.lua:284-315`). The slot shows Fire while a weapon is drawn, Swing while the bat is equipped, and Bat otherwise, so the two are never on screen together and nothing else needs to dodge the jump button (decision D-025, [[Integration Review]] conflict C21). The potion effect block moved to the top left on touch ([[Potions]] UI 3), so it no longer sits here.
- Not in `Config.HudClusters`, because its position is computed from the jump button, like the arena buttons. RoyalCombat's arena buttons only show `InArena` (`StarterPlayer.StarterPlayerScripts.RoyalCombat.lua:320-331`), when no weapon can be drawn.

### 6. World labels
- **Flag over the head:** a BillboardGui on the Head, 1.6 x 1.6 studs, StudsOffset (0, 3.4, 0), Flag Up icon (A3), MaxDistance 180, not AlwaysOnTop. If the head already carries the royal title billboard (`StarterPlayer.StarterPlayerScripts.RoyalUI.lua:977-1002`), it moves to StudsOffset (2.4, 2.6, 0), beside the title instead of on it.
- **Boat pennant:** a small red studded flag part on the boat's mast, shown while the driver is flagged (a model part, not UI).
- **Practice Range sign:** the existing `Stud Sign` model ([[Existing VFX and Props]]) copied to the pier, its SurfaceGui text "PRACTICE RANGE" in Fredoka.
- **Fort health (Phase 3):** a BillboardGui 14 x 2.4 studs, 18 studs above the Fort. Background = kit BoostBar (85577747844362, 197 x 46); fill = a second copy of the same image tinted Mythic red with a UIGradient cut at the health fraction; title "Rotten Fort" in Fredoka above it.

### 7. Toasts
All messages go through the existing `Notify` (red) and `Notice` (green) toasts drawn by GardenToast (`StarterPlayer.StarterPlayerScripts.GardenClient.lua:415-427`), which already follow the kit. Strings live in `WeaponConfig.Text`, with no emoji:
- "Weapons work here. Raise your War Flag to fight other players." (first time entering the zone, once per session)
- "War Flag up in %d s." / "War Flag is up. Flagged players can hit you and your boat." / "War Flag down."
- "You sank %s's boat." / "%s sank your boat. Your War Flag is down."
- "Weapons only work at sea." / "Your hands are full!" (existing `Config.ToolEquipWarning`, `ReplicatedStorage.GardenConfig.lua:1851`)
- "Rebirth to %s to unlock." / "You cannot fire while invisible." / "Target is outside the combat zone."
- Phase 3: "A Rotten Fort is drifting at sea!" / "Rotten Fort broken! +%s shards" / "The Rotten Fort sank. +%s shards"

## World and assets

| Thing | Source | What to build |
|---|---|---|
| Cherry Bomb tool | build from Parts with Studs surfaces | Two red studded block-cherries on a green stem with a short fuse, about 8 parts, as a Roblox `Tool` with a Handle (Fruit Empire is R15, `MorphToR15`, so the default R15 grip and tool-hold pose apply) |
| Melon Mortar tool | the Studded Pack `Logs` model (86601833587701, [[Creator Store Packs]]) for its base | A short studded tube on a log cradle, about 20 parts |
| Coconut Cannon tool | build | A brown studded shoulder tube with a coconut in the barrel, about 18 parts |
| Starfruit Barrage tool | build | A small golden horn, about 8 parts; the shells come from the sky |
| Projectiles | build | Cherry pair (4 parts), striped watermelon shell (6), coconut (4), starfruit shell (5). Client-only visuals, pooled |
| Muzzle smoke, shell trail | `vfx pack` > `Projectile` (smoke and spark trail) | trimmed to 2 emitters |
| Small blast (Bomb, Cannon) | `vfx pack` > `Explosion` (shockwave, sparks, stars) at 0.5 to 0.6 scale, plus `vfx pack` > `f` impact and flash | trimmed to 5 emitters, `Emit()` bursts only |
| Mortar blast | `vfx pack` > `Explosion` at 1.0 scale plus `f` rocks | trimmed to 6 emitters |
| Barrage shell blast | `Big` > `Explosion-01` at 0.7 scale | trimmed to 5 emitters |
| Water splash | `Anime` > `Water-01` (large) and `Water-02` (small) | used for every burst on the sea surface |
| Cannon speed lines | `vfx pack` > `Part` (speed lines) | on the coconut |
| Fizzle | `Anime` > `Smoke-01` | one small puff |
| Projectile trails | the bat's `Trail` (0.10 s lifetime) from `Workspace.Bat` ([[Existing VFX and Props]]) | recoloured per weapon tier |
| Draw / throw / hit sounds | the bat's `Equip` 769464514, `Swoosh` 3755636638, `Hit` 175024455 | reuse as is |
| Launch thump, shell whistle, explosion, splash, siren | not in the place | builder picks from Roblox's licensed audio library, the way the music was picked (D-002); no upload needed |
| Practice Range pier | `Stud Bridge` (71814553242028) | extend into a pier by the island dock |
| Target buoys (6) | build | studded barrels about 10 parts each, one in each kit tier colour, bobbing on the client; tagged `RangeBuoy` |
| Weapons Booth | `Stud Shop` (12340318191), decor only, no scripts | a recoloured copy by the pier, with a ProximityPrompt "Weapons" that opens the Shop on the Weapons tab; tagged `BatSafeZone` ([[Bats]] B22b) |
| Rotten Fort (Phase 3) | Studded Pack `Logs`, `Tent`, `CampFire` | a log raft with a log palisade, a tent, a campfire and rotten fruit crates, at most 250 parts, `ModelStreamingMode = Atomic` |
| Camera shake | the RbxCameraShaker presets from [[Merge a Nuke - Explosives and Health Drain]]: "Bump" (2.5, 4, 0.1, 0.75) for the Mortar within 60 studs; "Explosion" (5, 10, 0, 1.5) for Barrage shells within 80 studs, scaled down with distance | client code |

The VFX packs sit in the staging area under the map at y about -790 ([[Existing VFX and Props]]). The build copies trimmed versions into `ReplicatedStorage.WeaponFx`, so clients clone from there and never from Workspace. Every blast uses an `Explosion` only for its look, with `BlastPressure = 0` and `DestroyJointRadiusPercent = 0`, exactly like today's TNT (`ServerScriptService.GardenService.Tools.lua:292-298`), so no Explosion ever breaks joints or flings anyone.

## Asset requests for Samarth

**Images** (upload each once and reuse the id; keep at 512 x 512 or smaller, [[Roblox Platform Constraints for the Expansion]]):

- **A1. Weapons tab label or icon.** First check what `ShopMenu.ToolsTab` uses for its label (all three tabs share image 110917091182913, so the label is a child). If it is Fredoka text, no art is needed. If it is an icon, use this prompt:
  > "Use the attached Fruit Empire 'Shop' HUD button (a red basket full of fruit, Roblox image 71485100495903) as the exact style reference: a chunky cartoon game icon with a thick dark outline, glossy highlights and soft shading. Draw a short studded wooden mortar tube (Roblox-style blocks with round studs on top) with a striped watermelon shell in its mouth, angled up and to the right. 512x512 PNG, transparent background, centred, the same line weight and colour saturation as the reference. No text, no letters, no emoji."

- **A2. Four weapon slot icons.**
  > "Style reference: the attached Fruit Empire potion effect icons (the Speed tile, Roblox image 85746922401157, and the Jump tile, Roblox image 103629048118784; also the vault image ui-reference-potion-icons.png): a rounded square tile with a thick black outline and a thin white inner rim, a smooth vertical gradient background, and a centred pixel-art object with chunky pixels and a 1-pixel dark outline. Make 4 separate tiles, each 512x512 PNG, transparent outside the tile, no text and no emoji. (1) Cherry Bomb: two round red cherries joined by a green stem with a short lit fuse, gradient #4EC265 at the top to #378847 at the bottom. (2) Melon Mortar: a stubby studded wooden mortar tube with a watermelon shell, gradient #4EA4E6 to #3772A1. (3) Coconut Cannon: a brown shoulder launcher made of studded blocks with a coconut in the barrel, gradient #F2AB30 to #A97822. (4) Starfruit Barrage: a small golden horn with three yellow starfruit falling from above, gradient #47DED9 to #329B98."

  (The four gradients are the kit's Uncommon, Rare, Legendary and Astral colours, the exact `Config.RarityColors` values (`ReplicatedStorage.GardenConfig.lua:582-592`); the flags use Common and Mythic.)

- **A3. Two War Flag icons.**
  > "Same tile style and the same references as the four weapon tiles (the Speed tile 85746922401157 and the Jump tile 103629048118784). Two 512x512 tiles, no text, no emoji. (1) Flag Down: a white cloth flag hanging limp on a short studded wooden pole, gradient #97A9BD to #697684. (2) Flag Up: a red cloth flag flying straight out from the same pole, gradient #E8423C to #A22E2B."

- **A4. Reticle** and **A5. Hit marker.**
  > "Style reference: the Fruit Empire red X close button (Roblox image 107111219044453): a thick black outline around a white shape with a slight glossy highlight on top. (A4) An aiming reticle: a white ring with four short white ticks at the top, bottom, left and right, and a small white centre dot, every piece outlined in 6-pixel black. (A5) A hit marker: four short white diagonal dashes forming an X around an empty centre, the same 6-pixel black outline. Each 256x256 PNG, transparent background, only white and black."

- **A6. Fire button (touch).**
  > "Style reference: the Fruit Empire 'Merge All' HUD button (Roblox image 99980479639840): a chunky cartoon icon on top with the button's word baked underneath in a heavy rounded white font with a thick black outline (the same lettering as 'Merge All', 'Rebirth' and 'Clear'). Draw a round red cherry bomb with a lit fuse and two small yellow sparkle stars, and bake the word 'Fire' underneath in that exact lettering. 512x512 PNG, transparent background, icon in the top two thirds and the word in the bottom third, like the reference."

**Animations** (only uploaded animations owned by the game's owner play in a live game, D-005; the game is owned by the account `hikeALT12212` per [[Roblox Platform Constraints for the Expansion]]):
- Make these five on an R15 dummy (the Sword Animations pack dummies are R15, [[Creator Store Packs]]) in Studio's Animation Editor, add the named keyframe markers, then File > Publish to Roblox under the game owner's account, and paste the ids into `WeaponConfig.Animations`:
  1. `WeaponThrow`, 0.55 s, Action priority: arm cocks back behind the head over 0.25 s, overhand release, marker `Release` at 0.30 s, follow-through.
  2. `MortarFire`, 0.6 s: slight crouch, both hands steady the tube at chest height, recoil, marker `Fire` at 0.15 s.
  3. `CannonFire`, 0.35 s: tube on the right shoulder, recoil kick back, marker `Fire` at 0.08 s, settle.
  4. `BarrageCall`, 0.8 s: horn to the mouth, marker `Call` at 0.30 s, then point at the target.
  5. `WeaponHold`, looped, Idle priority: the two-handed carry pose for the Mortar and the Cannon.
- Until they are published, the build uses Roblox's default tool-hold pose plus a code-driven arm swing on the client (the procedural approach of D-004), so nothing waits on this.

**Robux products:** none. Weapons and charges are never sold for Robux (Rule 3), so there is also no paid-random-item work and no PolicyService check ([[Roblox Platform Constraints for the Expansion]]).

## Code plan

### Build phases
1. **Phase 1:** `WeaponConfig`, `Ballistics`, the `Weapons` server module (buy, draw, holster, flag, fire, projectiles, blasts on boats and players), save fields, `WeaponClient` (Weapon Bar, aiming, Shop page, flags), `WeaponFx`. Cherry Bomb and Melon Mortar only. Until [[Boats and Swimming]] ships, test against a Studio-only stand-in `ServerStorage.TestBoat` that carries the boat contract below.
2. **Phase 2:** Coconut Cannon, Starfruit Barrage, the Practice Range and Weapons Booth, swap in the published animations.
3. **Phase 3:** the Rotten Fort (`SeaFort` module and its visuals).

### New modules
| Module | Kind | Job |
|---|---|---|
| `ReplicatedStorage.WeaponConfig` | ModuleScript, shared | Every number in this note, the `Text` strings, asset ids, unlock stages, `k_w` price multipliers (prices come from `Config.ShardPrice`, which reads the one D-010 table in GardenConfig, D-017), zone radii, `Animations`. Read by server and clients, like `CrownConfig` (`ReplicatedStorage.CrownConfig.lua:11-13`). `StudioUnlockAll = false` (honoured only when `RunService:IsStudio()`). |
| `ReplicatedStorage.Ballistics` | ModuleScript, shared | Pure math: `position(o, v, g, t)`, `velocity(v, g, t)`, `solveLob(o, target, tau, g)`, `lobTime(spec, d)`, `beamCurve(o, v, g, tau)`, `falloff(d, r)`, `inZone(pos, forShooter)`. Server and clients use the same functions, so the drawn path and the real path match. |
| `ServerScriptService.GardenService.Weapons` | ModuleScript, `return function(G)` | Purchase, draw and holster (a real Roblox `Tool` equipped with `Humanoid:EquipTool`, as the arena does, `ServerScriptService.GardenService.CrownArena.lua:197-220`), War Flag, fire validation, the projectile table and its Heartbeat step, blast resolution, practice mode, save capture and apply, cleanup on leave and death. Exports `G.weaponsCapture`, `G.weaponsApply`, `G.weaponDrawn`, `G.holsterWeapon`, `G.warFlagged`. |
| `ServerScriptService.GardenService.Knockback` | ModuleScript, `return function(G)` | **Only if [[Bats]] has not already shipped `G.knockPlayer`.** Contract: `G.knockPlayer(victim, velocity, source)` fires the victim's client to set Freefall and write `AssemblyLinearVelocity`, and adds a 0.12 s server `LinearVelocity` (MaxForce 1e6) as a backup. This copies `push` (`ServerScriptService.GardenService.CrownArena.lua:510-536`) without depending on RoyalUI, which quits when `CrownConfig.Enabled` is false (`StarterPlayer.StarterPlayerScripts.RoyalUI.lua:34`). |
| `StarterPlayer.StarterPlayerScripts.WeaponClient` | LocalScript | Weapon Bar, input binding, aim preview, the shooter's predicted projectile, the Shop Weapons page, flag billboards, the Fire state of the touch action slot (drawn by [[Bats]] BatHud), the knockback handler if `Knockback` is ours. It does not touch the Backpack hotbar setting: BatHud turns it off once for the whole game (D-025). |
| `StarterPlayer.StarterPlayerScripts.WeaponFx` | LocalScript | Draws every projectile, warning ring and blast from `WeaponEvent`; pooled visuals (40 maximum); camera shake; buoy bobbing. |
| `ServerScriptService.GardenService.SeaFort` | ModuleScript, `return function(G)` | Phase 3: the Fort schedule, health, regen, return fire and payout. |

Studio instances added during the build: `ReplicatedStorage.WeaponModels` (the four Tools and the projectile models), `ReplicatedStorage.WeaponFx` (trimmed emitter sets), `GardenUI.WeaponBar`, `ShopMenu.WeaponsTab`, `ShopMenu.Panel.WeaponsPage`, `Workspace["Main Island"].PracticeRange` (pier, pad part tagged `PracticeRange`, buoys tagged `RangeBuoy`, Weapons Booth, sign), and `NoFireZone` parts at each summon pad.

### Changes to existing scripts
| File and line | Change |
|---|---|
| `ServerScriptService.GardenService.lua:49-86` | Create `WeaponRequest` and `WeaponEvent` with the existing `remote()` helper (`:49-57`) and add them to G (`:101-115`). |
| `ServerScriptService.GardenService.lua:518` | After `require(script.CrownDecree)(G)`, add `require(script.Weapons)(G)` (and in Phase 3 `require(script.SeaFort)(G)`). After the royal modules, so `G.royalBusy` exists. Persistence reaches the new functions through G at call time, the way it reaches `G.royalCapture` (`ServerScriptService.GardenService.Persistence.lua:81`). |
| `ServerScriptService.GardenService.Persistence.lua:63-67` | After the tools loop: `if G.weaponsCapture then profile.weapons = G.weaponsCapture(player) end`. |
| `ServerScriptService.GardenService.Persistence.lua:221-225` | After tools are restored: `if G.weaponsApply then G.weaponsApply(player, profile.weapons) end`. Yield-free, like the rest of `applyProfile`. |
| `ServerScriptService.GardenStore.lua:239-243` | In `Store.Default()`, beside `tools`: `weapons = {}`. |
| `ServerScriptService.GardenStore.lua:358-380` | A sanitise block for `weapons` modelled on the tools block: name in `WeaponConfig.Weapons`, whole number from 1 to that weapon's `maxHeld`; anything else counts as a repair and is dropped. Without this the field survives one save and vanishes on the next load (`ServerScriptService.GardenStore.lua:447-450`). |
| `ServerScriptService.GardenStore.lua:1200-1209` | Add all four weapons at max held to the worst-case self-test payload so the 8,192-byte check still covers the document. |
| `ServerScriptService.GardenService.CrownArena.lua:264-268` | In `enterArena`, before `giveSword`: `if p and G.holsterWeapon then G.holsterWeapon(p, "arena") end`. |
| `ServerScriptService.GardenService.Carrying.lua:160-161` | `pickUp` returns early if `G.weaponDrawn and G.weaponDrawn(player)`. The zone rule already keeps drawn weapons off plots; this is a second guard. |
| `ServerScriptService.GardenService.Tools.lua:124-127` | `equipTool` refuses with `Config.ToolEquipWarning` while a weapon is drawn. |
| `ServerScriptService.GardenService.Storage.lua:63-69` | `equipStored` refuses while a weapon is drawn, with its existing `Config.EquipWarning`. |
| `ReplicatedStorage.GardenConfig.lua:1210-1212` | `Config.ShopTabs` becomes `{ "Potions", "Weapons", "King", "Robux" }` (and `{ "Potions", "Weapons", "Robux" }` with the King off), decision D-022. The Weapons page is built by ShopMenu from one list: `WeaponConfig.Weapons` plus the `Config.Tools` entries with `explosive = true`, sorted by unlock stage. |
| `ReplicatedStorage.GardenConfig.lua:1863-1867` | Add `"WeaponBar"` to `Config.HudClusters`. |
| `StarterPlayer.StarterPlayerScripts.ShopMenu.lua:89-102`, `:612-633` | Watch a local-player attribute `OpenShopTab`: when set, open the Shop, call `select(name)`, clear the attribute. The Weapon Bar's Shop slot and the Weapons Booth prompt use it. |
| `StarterPlayer.StarterPlayerScripts.GardenClient.lua:163-169` | The global `PromptShown` handler skips prompts whose parent is not in the fruit folder, so the Weapons Booth prompt does not get the fruit outline or the `NameTag` lookup (the gotcha in [[Client UI - HUD and Menus]], item 9). |

No change to `applyWalkSpeed` (`ServerScriptService.GardenService.lua:190-248`): weapons never slow anyone. No change to the drown sweep: a push faster than 30 studs/s briefly reads as "carried" (`ServerScriptService.GardenService.Ambient.lua:363-364`), but the drown kill only applies past r 2825 (`:376-381`), far outside the zone.

### Remotes
Both are RemoteEvents in `ReplicatedStorage.GardenRemotes`. Any two `WeaponRequest` calls from one player less than 0.1 s apart drop the second, the same idea as RoyalRequest's 0.15 s gate (`ServerScriptService.GardenService.Crown.lua:868-871`). Bad payloads are dropped silently.

| Remote | Direction | Payload | Server validation |
|---|---|---|---|
| `WeaponRequest` | client to server | `("buy", key: string)` | key in `WeaponConfig.Weapons`; Rule 4 checks; price computed on the server from the `Rebirths` attribute; `setShards` then add charges; `Store.Touch` like `buyTool` (`ServerScriptService.GardenService.Tools.lua:91-112`) |
| | | `("draw", key)` | Rule 9 checks; 0.35 s since the last draw |
| | | `("holster")` | none needed |
| | | `("fire", key, aim: Vector3, seq: integer)` | Rules 12 to 14; `aim` must be a finite Vector3, `seq` an integer; the server computes origin and velocity |
| | | `("flag", up: boolean)` | a boolean; Rules 27 to 29 |
| `WeaponEvent` | server to all | `("shot", {id, key, o, v, g, t0, by, practice})` | o and v are Vector3; t0 is `workspace:GetServerTimeNow()` at launch |
| | server to the shooter | `("ack", {seq, id, o, v, t0})` or `("nack", {seq, reason})` | |
| | server to all | `("warn", {id, at, r, tEnd})` | Mortar and Barrage only |
| | server to all | `("boom", {id, at, kind})` | kind = water, boat, solid, player, buoy, fort, fizzle |
| | server to the shooter | `("hit", {})` | hit marker |
| | server to the victim | `("knock", {vel})` | only if the Knockback module is ours |
| | server to all | `("fort", {state, pos, hp, max})` | Phase 3 |

### How a shot works (server authority with client prediction)
1. The shooter's client checks the zone, its local copy of the cooldown (`WeaponReadyAt`) and the charge count, so honest clients do not spam. It spawns a **predicted** projectile at its own muzzle at once, plays the launch sound and animation, and sends `("fire", key, aim, seq)`.
2. The server validates (Rules 9 to 14). It sets the origin to `root.CFrame * spec.muzzle`. For a lob it finds the flat distance d, sets tau with `lobTime`, and solves the launch velocity. For the Cannon it uses `unit(aim - o) * 170`. It spends one charge (not on the pad), writes `WeaponReadyAt` and `WeaponClip`, adds the projectile record `{id, owner, key, o, v, g, t0, practice}`, sends `ack` to the shooter and `shot` to everyone (plus `warn` for the Mortar).
3. On `ack`, the shooter's predicted visual blends onto the server path over 0.15 s. On `nack` it fizzles and the reason shows as a toast.
4. Every other client draws the projectile at `position(o, v, g, now - t0)` with `now = workspace:GetServerTimeNow()`, so it appears where the server has it, already skipped ahead by the network delay. This is the fire-position-and-time fix from the DevForum thread cited in [[Sky Rivals - Weapons and Artillery]] [42].
5. Each Heartbeat the server moves every projectile from `position(t)` to `position(t + dt)` and tests, in order: zone exit (for practice shots, leaving `PracticeRangeArea`), `NoFireZone`, lifetime; a raycast along the segment with an Include list (the boats folder, `Main Island.Shallows`, `SeaSolid` parts, buoys, the Fort); the sea surface crossing (y = -2.2, solved directly, no raycast, since `Ocean` cannot be raycast, [[Swimming Fatigue and Boats in Roblox]]); and the closest distance from the segment to each flagged at-sea character root (2.5 studs). The earliest contact wins.
6. On contact the server resolves the blast (Rules 19 to 25) by calling `G.damageBoat(boat, B * f(d) * BoatMaxHP, owner, "weapon")` ([[Boats and Swimming]]), `G.knockPlayer(victim, vel, owner)`, and the buoy or Fort handlers, then sends `boom`. Clients snap their visual to `at` and play the blast.
7. Clients never report hits. There is no hit remote.

### Ballistics math
- Position: `p(t) = o + v t + 0.5 g t^2`, with `g = (0, -G_w, 0)`.
- Lob launch velocity for a landing point T after tau seconds: `v = (T - o) / tau - 0.5 g tau`.
- Arc preview: a parabola from p0 = o with start velocity v0 over tau is exactly a cubic Bezier with control points `p0 + v0 tau / 3` and `p1 - v1 tau / 3`, where `p1 = p(tau)` and `v1 = v0 + g tau`. So `CurveSize0 = |v0| tau / 3` with Attachment0's axis along v0, and `CurveSize1 = |v1| tau / 3` with Attachment1's axis along v1.
- Boat distance: the distance from the burst point to the nearest point of the boat's bounding box (`GetBoundingBox`), so a long hull is hit along its length.
- Falloff: `f(d) = 1 - 0.7 * math.min(d / r, 1)`. This is the linear falloff from Roblox's Explosion docs sample ([[Merge a Nuke - Explosives and Health Drain]]) with a 30% floor. It lives once, in `Ballistics.falloff`, so [[Explosives and Fruit Health]] can use the same function.

### The boat contract this note needs from Boats and Swimming
- Boats live in one folder (proposed `Workspace.Boats`), one Model per boat; hull parts tagged `BoatHull` and queryable (`CanQuery = true`).
- Model attributes, named by [[Boats and Swimming]] (its 9.6, which owns them): `OwnerUserId`, `DriverUserId` (nil when nobody drives), `HP`, `MaxHP`, `ProtectedUntil` (server time, summon + 15 s), `Broken`. The `Collider` part carries the `BoatHull` tag and has `CanQuery` true.
- `G.damageBoat(boat, amount, sourcePlayer, kind)` lowers HP on the server, starts the sinking at 0, and tells Weapons who sank it (for Rule 33 and Rule 29). Boats R44 adds its own refusals (spawn protection, dock zone, owner or passenger as the source, double counting). One name for the whole expansion: `G.damageBoat` (Boats renamed its `G.boatDamage`, [[Integration Review]] conflict C10).

### Saved fields
| Field | Type | Default | Notes and migration |
|---|---|---|---|
| `weapons` | table `{[weaponKey] = whole number}`, 1 to `maxHeld` | `{}` | Captured from a `Weapons` folder of IntValues on the Player (the same pattern as `StoredTools`, `ServerScriptService.GardenService.Tools.lua:59-81`). A folder on the Player lives until the Player instance is gone, so the leave-save can always read it. Old saves have no `weapons` key: sanitise leaves Default's `{}` and the player owns none. Adding a field needs no schema bump (`ServerScriptService.GardenStore.lua:8-12`). Rebirth does not touch it, matching D-009's "keeps tools". |

Not saved: the War Flag (always down on join), cooldowns, projectiles in flight, Fort progress.

### Attributes and tags
| On | Name | Type | Written by | Read by |
|---|---|---|---|---|
| Player | `WarFlag` | boolean | Weapons | WeaponClient, WeaponFx (billboards) |
| Player | `WarFlagChangeAt` | number, server time a pending change lands, or nil | Weapons | WeaponClient (countdown) |
| Player | `WeaponDrawn` | weapon key or nil | Weapons | WeaponClient |
| Player | `WeaponReadyAt` | number, server time | Weapons | WeaponClient (cooldown sweep) |
| Player | `WeaponClip` | number (Cannon shots left in the clip) | Weapons | WeaponClient |
| Player | `InCombatZone` | boolean, checked 4 times a second | Weapons | WeaponClient (bar visibility) |
| Player folder | `Weapons` | IntValue per key = charges | Weapons | WeaponClient, Shop page |
| Boat model | `OwnerUserId`, `DriverUserId`, `HP`, `MaxHP`, `ProtectedUntil`, `Broken` | | Boats | Weapons |
| Fort model | `FortHP`, `FortMax` | number | SeaFort | WeaponFx |
| Tags | `NoFireZone`, `PracticeRange` (the pad), `PracticeRangeArea` (the box practice shots stay inside), `RangeBuoy`, `SeaSolid`, `BoatHull` | | Studio / Boats | Weapons, WeaponFx |

## Edge cases and abuse

- **Faked aim or origin.** The client sends only an aim point. The server uses its own origin and its own speeds, clamps the range and checks the zone. A teleporting exploiter still fails the zone check made on the server's copy of their position.
- **Rapid fire.** The per-weapon cooldown, the 0.15 s shot gap, 3 in flight per player and 40 per server. The 0.1 s request gate drops floods long before Roblox's own limit of about 500 requests per second ([[Roblox Platform Constraints for the Expansion]]).
- **Firing from safety.** Shooting from the beach, the dock or a plot is refused (Rule 9). A projectile fired from the zone toward the beach fizzles at the boundary (Rule 17). A blast near the boundary affects only targets inside the zone (Rule 8).
- **Hurting players who did not agree.** Impossible: knockback and boat damage need both sides flagged (Rules 21 to 22).
- **Gunboat exploit.** A flagged passenger in an unflagged friend's boat would be untouchable. Rule 23 makes their shots PvE-only.
- **Flag dodging.** Raising takes 5 s with no player hits either way. Lowering takes 10 s and restarts on any hit. Nobody can flip the flag to win a fight in progress.
- **Hunting one player.** A boat sunk by a player drops its driver's flag at once (Rule 29). The 15 s spawn protection and the 40-stud no-fire boxes stop camping at summon pads.
- **Juggling.** The 1.5 s knockback immunity stops one player being pinned in the air by chained blasts. If [[Bats]] uses the same `G.knockPlayer`, the window is shared.
- **Self-knockback by the water.** A shooter is never pushed by their own blast, the problem SkyRivals' shotgun kick has ([[Sky Rivals - Weapons and Artillery]], "What to avoid").
- **Leaving mid-flight.** Projectiles already fired finish and resolve. A leaver gets no toast or reward. Charges are already spent and saved.
- **Rejoining.** The flag is down, nothing is drawn, charges load from the save.
- **Rebirth mid-action.** Purchases are refused while `rebirthing[player]` is set. A drawn weapon stays drawn; charges are kept; prices and unlocks update from the new `Rebirths` value. Stage only rises, so nothing locks.
- **Death mid-aim.** `CharacterRemoving` holsters the weapon; nothing is spent. Projectiles in flight still resolve.
- **Dying costs potions.** `clearEffects` runs on death (`ServerScriptService.GardenService.lua:554-570`, `:601-604`). Weapons never kill, and the zone never reaches the 220-stud fall kill (`ServerScriptService.GardenService.Ambient.lua:291-299`) or the r 2825 drown kill. [[Boats and Swimming]] must also wash tired swimmers home with a teleport, not a death, or a sunk player loses their potions.
- **Carrying fruit at sea.** A carrier cannot draw (Rule 9) and is not knocked unless flagged. A push never drops a carried fruit.
- **Arena teleports.** `enterArena` holsters the weapon first. Crown teleports already stand a seated player up (`ServerScriptService.GardenService.Crown.lua:370`).
- **Invisible attacker.** Invisible players cannot fire (Rule 14).
- **Client-owned boats.** The driver's client runs the boat's physics, so an exploiter could teleport a boat out of a shell's path ([[Swimming Fatigue and Boats in Roblox]]). Boats owns the speed and position checks; Weapons always uses the server's view of the boat.
- **Streaming.** A target far from the shooter may be streamed out on the shooter's client. Only the visual is affected; the server decides the hit.
- **Mobile.** Aim by camera with the reticle, one large Fire button beside the jump button, no hover states needed, slot taps for switching. Weapon VFX use `Emit()` bursts sized for the mobile particle cap.
- **Two currencies.** All prices are shards, matching every other shop item and the shard crystal baked into the kit Buy button (`StarterPlayer.StarterPlayerScripts.ShopMenu.lua:315`).
- **Pets and animals.** Projectiles have no pet in their Include list and no pet is ever a blast target, so the platform's animal-abuse rule cannot be touched ([[Roblox Platform Constraints for the Expansion]]).
- **Content rating.** No blood, no deaths, no health loss: cartoon fruit bursts and pushes, which keeps the game in the "Mild" band ([[Roblox Platform Constraints for the Expansion]]).
- **Studio.** Tests write the real save unless the workspace attribute `StudioNoSave` is true (`ServerScriptService.GardenStore.lua:572-575`).

## Performance

Budgets from [[Roblox Platform Constraints for the Expansion]]: under 50 KB/s download per client, about 40 to 60 moving physics assemblies per client, 400 particles per second per emitter (100 on mobile), at most about 24 Highlights.

- **No physics projectiles.** A projectile is a server table and a client-drawn anchored visual moved by `CFrame`, so it costs no physics replication and does not count toward the 40 to 60 moving assemblies. Boats are the only moving assemblies this system touches.
- **Server work per frame.** At most 40 projectiles, each one raycast against a short Include list plus one closest-point test per flagged character at sea (at most 12 in practice). That is well under 1 ms per frame [inferring, not measured].
- **Network.** A `shot` event is about 80 bytes (two Vector3s and four numbers). Worst case, 12 players each firing twice a second, is 24 events per second, about 2 KB/s per client, plus `boom` events of about 40 bytes. Far under 50 KB/s. Flag and cooldown state ride on attributes that change only on actions.
- **Client drawing.** Visuals are pooled (40 maximum) and culled past 600 studs. One Beam for the shooter's own preview. Rings are single Neon parts. No Highlights at all, so the 24-Highlight budget stays with merges.
- **Particles.** Every blast uses `Emit()` bursts from trimmed emitter sets: about 60 particles for a small blast, about 150 for a Mortar or Barrage shell. No emitter changes properties every frame.
- **Parts.** Weapon tools 8 to 20 parts each; projectile visuals 4 to 6 parts; Practice Range about 150 parts (pier, 6 buoys of about 10, the 30-part Stud Shop, the 2-part sign); Fort at most 250 parts, Atomic streaming.
- **Saving.** One small table in the existing document, written by the normal autosave plus a `Store.Touch` after a purchase. No extra DataStore calls.

## Test plan

Set the workspace attribute `StudioNoSave` to true before every playtest (`ServerScriptService.GardenStore.lua:572-575`). Use the Studio dev door `DevSetShards` for shards (`ServerScriptService.GardenService.lua:653`). For stages, set `WeaponConfig.StudioUnlockAll = true`, or add a Studio-only `DevSetRebirths` door next to `DevSetShards` (proposal). Player-versus-player tests use Test > Clients and Servers with 2 players.

| Rule | How to verify |
|---|---|
| 1, 2 | At 0 rebirths, open Shop > Weapons: eight cards in unlock order (four sea weapons and the four garden explosives), all greyed with "Rebirth 1" to "Rebirth 7". Set rebirths to 1: only the Cherry Bomb and TNT cards turn on. Set 2: the Mortar too. |
| 3, 4 | At stage 3 with 1,000 shards, buy a Cherry Bomb pack: shards drop by exactly 180, "Owned: 4". Buy until 40 held: the next press shows "Full" and shards do not change. With 0 shards: "Not enough shards!". Press Buy and Rebirth together: the purchase is refused while rebirthing. |
| 5 | Buy, rejoin (with saving on in a throwaway account, or use the dev panel round trip): charges are back. Rebirth: charges kept. Die at sea with a weapon drawn: charges unchanged. |
| 6, 7, 8 | Stand on the beach (r < 400): no Weapon Bar, the draw key does nothing. Walk into the sea past r 400: the bar appears. Fire a bomb toward the beach: it fizzles at the boundary. Stand inside a summon pad's box: drawing is refused. Burst a bomb at r 404 next to a flagged player standing at r 395: they are not pushed. |
| 9, 10, 11 | At sea, carry a fruit: drawing is refused with "Your hands are full!". Drink Invisibility: drawing is refused. Walk out of the zone with a weapon drawn: it holsters within 0.25 s. With a weapon drawn, try `EquipTool` from the Inventory: refused. |
| 12, 13 | Fire the Mortar twice quickly: the second is refused until 4.0 s. From the command bar, fire `WeaponRequest` with an aim 500 studs away: `nack`. Fire with an aim inside a plot: `nack`. Fire 4 bombs as fast as allowed: never more than 3 in flight. |
| 14 | As a passenger, fire: accepted. As a driver, fire: accepted. While invisible: refused with the toast. |
| 15 to 18 | Throw at a stepping stone: bursts on the stone. Throw into open water: bursts at y -2.2 with a splash. Fire the Cannon straight out to sea: fizzles at about 238 studs or at r 985. On the pad, fire at a buoy: the buoy reacts; fire past a player: nothing happens to them. |
| 19 | Two flagged players: burst a Mortar shell 6 studs from the victim (r 12): the push is 80 x 0.65 = 52 studs/s plus lift 22. Measure with a debug print of the knock velocity. |
| 20 | Fire every weapon at max range toward the island: no fruit on any plot changes (fruit count per plot before and after is equal). |
| 21 | Flagged shooter, unflagged victim: no push. Both flagged, victim seated in a boat: no push, the boat takes damage. Two blasts within 1.5 s: the second does not push. The shooter stands in their own blast: not pushed. |
| 22, 23 | Unflagged driver: shells do 0 boat damage. Flagged driver: one centre Mortar hit takes 35% of `MaxHP`. Summon a boat and shoot it at once: no damage for 15 s. Flagged passenger in an unflagged boat fires at a flagged boat: no damage. |
| 24, 25 | Unflagged player shoots a buoy (and the Fort in Phase 3): it takes damage. Put a pet near a blast in Studio: nothing happens to it. |
| 26 to 30 | Rejoin: flag down. Press the flag: 5 s countdown, then the billboard appears over the head for the other client. Press again, get hit at 7 s: the countdown restarts at 10. Sink a flagged driver's boat: their flag drops at once with the toast. |
| 31 | At stage 1, stand on the pad: all four weapons fire with no charge change. |
| 32, 33 | Sink a boat: both toasts appear; both players' shards and cash are unchanged. |
| 34, 35 | Fire a Mortar: the second client sees the red ring for the whole flight. Call two Barrages from two players: the second is refused until 20 s after the first ends. |
| 36 to 40 | Phase 3 with 2 players: the Fort spawns at a marker after the 60 s toast; its HP is 300; stop shooting for 6 s and HP rises 6 per second; it shells a nearby boat with a ring; break it at stage 6 with 333 units dealt: pay is 9,290 + 930 shards. |
| UI overlap | Check at 1600 x 900, 1280 x 720 and the phone emulator at 667 x 375 (Studio device emulation): Weapon Bar, CornerHud, EffectStack, the Fire button and the jump button never cover each other. Carry a fruit at sea: the carry row shows and the Weapon Bar hides. |

## Dependencies and conflicts

- **[[Rarity Gating and Rebirth]].** Stage = 1 + `Rebirths` decides unlocks and prices; nothing else here reads the rarity gate. Weapons are kept through rebirth, matching D-009. Request: the Rebirth panel names the weapon the next rebirth unlocks ("Next rebirth unlocks: Melon Mortar"), which makes "rebirth to unlock more" obvious, as the brain dump asks. Buying is blocked while `rebirthing[player]` is set.
- **[[Bats]].** Both systems push players through one `G.knockPlayer`, with immunity kept per kind (bat 1.2 s, blast 1.5 s). Resolved in the [[Integration Review]]: bats need no War Flag (a bat is a short shove you can walk away from); weapons do (artillery hits from 160 studs and breaks boats). One hand (D-025): drawing a weapon unequips the bat and equipping the bat holsters the weapon (Rule 9, Bats B5 and B6). Keys: 1 = bat, 2 to 5 = weapons. Touch: the Fire button lives in the Bats touch action slot. Seated boat riders cannot be hit by bats either (Bats B12m).
- **[[Explosives and Fruit Health]].** That system owns fruit health and placed TNT on plots. Weapons never call fruit damage (Rule 20). Both use `Ballistics.falloff` and the same Explosion setup (`BlastPressure = 0`, `DestroyJointRadiusPercent = 0`, `ServerScriptService.GardenService.Tools.lua:292-298`). If that note adds a placed Nuke, the Starfruit Barrage must not look like it (shells from the sky, not a ground mushroom cloud). The two should share one camera-shake helper so a Nuke and a Barrage never stack screen shake.
- **[[Underground and Ores]].** No overlap: the underground sits under the plots (about 200 studs from the centre, [[World Tour]]), outside the zone. A pickaxe and a weapon are never held together.
- **[[Pets]].** Pets are never targets (Rule 25). Pets wander on plots, outside the zone. If pets ever follow players to sea, projectiles still ignore them.
- **[[Boats and Swimming]].** The biggest link. Resolved in the [[Integration Review]] (C10, D-024): this note owns the weapon list and the damage numbers (Boats 5.7 now copies this note's table), Boats owns the boat (HP 1,000, spawn protection 15 s, regeneration 20 HP per second after 8 s, dock zones, attribute names) and calls the shared falloff. Contract: folder `workspace.Boats`, `BoatHull` tag on the `Collider`, attributes `HP`, `MaxHP`, `ProtectedUntil`, `DriverUserId`, `OwnerUserId`, function `G.damageBoat`. Boats are breakable only through the War Flag (both sides flagged), or by the Phase 3 Fort. The SeaStatus bar sits above the Weapon Bar and the notice toast lane (Boats 6.1); wash-up is a teleport, not a death; the currents are removed. The 1,000-shard boat price is Samarth's and is not touched.
- **[[Potions]].** Invisibility blocks firing (Rule 14), and Potions rule 27 now says so too (D-030). Aqua Affinity needs no change. Any new potion that changes movement must keep its `InArena` exemption as today. On touch the potion effect block moved to the top left ([[Potions]] UI 3), so the Fire button no longer needs to dodge it. The Shop's old Tools tab is now Potions; both notes use `{ Potions, Weapons, King, Robux }`.
- **[[Plot Overhaul and Environment]].** The zone radii assume the beach edge at r 380 and the forest ground at r 995; if the overhaul moves either, update `WeaponConfig.Zone`. Studded restyles of the stones, driftwood and islets must stay inside `Main Island.Shallows`, which is the projectiles' collision set. The dock layout must place the Boats summon pad, the Practice Range pier and the Weapons Booth together, at least 30 studs apart. The Stud Shop pack is used for the Boat Shop, the Weapons Booth, the pet-tamer booth and the Royal Armory; each is recoloured differently.
- **[[King UI Overhaul]].** Weapons UI stays out of the royal zones: the top-left royal stack (300 x 300 at (14, 6), `StarterPlayer.StarterPlayerScripts.RoyalUI.lua:299-305`) and the top-centre royal cards (`:352-391`). The King UI's only bottom-centre elements are the arena PC legend and the Royal Combat ability bar, both shown only `InArena`, when the Weapon Bar is hidden. The flag billboard steps aside for the royal head title.
- **[[Royal Combat]].** Weapons are holstered on `enterArena` and never hit `InArena` players, so duels stay "same numbers for everyone" (`ReplicatedStorage.CrownConfig.lua:16-19`). Royal Combat's weapon skins are for the duel sword only. Disabling Roblox's default Backpack hotbar removes the sword's hotbar slot in duels; the server still equips the sword (`ServerScriptService.GardenService.CrownArena.lua:209`), so duels work (Royal Combat's `giveSword` also unequips other tools first). The Weapons Booth is never called "Armory", which is Royal Combat's blade panel. RoyalCombat's touch buttons and the weapon Fire button are never visible at the same time.

## Open questions for Samarth

1. **Who can break a boat?** You said boats "can also be broken by the artillery". **Default: only boats whose driver raised the War Flag, and only by flagged players.** Reason: a 0-shard new player has to swim or boat to the forest to tame pets, and being sunk on the way would be griefing. Flip `WeaponConfig.RequireFlag = false` and every boat at sea becomes breakable.
2. **Should artillery ever hurt fruit?** **Default: never.** Stealing was removed on purpose, and ranged fruit damage would bring back loss caused by other players from a safe distance. TNT on plots stays with [[Explosives and Fruit Health]].
3. **Which currency?** **Default: shards, priced as k x M_s.** Cash prices break across stages (Numbers), and no weapon is sold for Robux.
4. **Unlock stages 2 / 3 / 5 / 7?** **Default: yes.** Stage 1 (the first 15 minutes) stays calm, and each weapon arrives with its rarity colour.
5. **Is the Rotten Fort wanted now?** **Default: build Phases 1 and 2 first and the Fort as Phase 3.** It gives weapons a use that is not player-versus-player.
6. **Should sinking another player pay a reward?** **Default: no.** Rewards for player-versus-player invite farming with alt accounts.
7. **Can the Practice Range try locked weapons?** **Default: yes.** It shows exactly what rebirthing unlocks.
8. **Hide Roblox's default hotbar?** **Default: yes,** so it never sits over the bottom-centre slot.
9. **Animations.** **Default: ship with the default hold pose plus code-driven arm swings, and swap in your five published animations when they are ready** (Asset requests).
10. **Sunk players' flag drops at once?** **Default: yes.** It stops one player being hunted repeatedly.
11. **Cosmetic launcher skins later** (fixed price, shards or fixed-price Robux, never random boxes)? **Default: not in v1.** They would be a good late-game shard sink if wanted.
12. **Can the driver fire?** **Default: yes,** so a solo player can fight. If two-person boats should feel different, make it passengers only.
13. **What does "nukes" mean here?** **Default: the Starfruit Barrage is this system's big strike (a siren, a marked circle, five shells from the sky), and a placed Nuke, if you want one, is the top tier of TNT in [[Explosives and Fruit Health]].** That keeps every plot-side explosive in one system and every sea weapon in this one.
