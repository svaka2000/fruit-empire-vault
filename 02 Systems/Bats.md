---
tags: [system, design]
date: 2026-10-04
status: proposed
depends_on: [Rarity Gating and Rebirth, Royal Combat, Plot Overhaul and Environment]
---
# Bats

Every player carries one bat. It gets stronger and changes colour each time they rebirth. It pushes other players, and nothing else. This note is the full design: rules, numbers, UI, assets, code plan and tests. Everything here is a **proposal** unless it cites existing code or research.

**Words used in this note**
- **Bat**: a Roblox `Tool` held in the right hand. A swing pushes one player away.
- **Tier**: the bat's level, 1 to 9. It equals the player's **stage** from the [[Stage Model]] (stage 1 = Common at 0 rebirths, stage 9 = Transcendent at 8 rebirths).
- **Knockback**: the push a hit gives, as a launch speed in studs per second. This is the same unit the Royal Arena already uses (`ReplicatedStorage.CrownConfig.lua:92-145`).
- **Power**: the sideways part of the knockback (studs/s). Shown on the bat card.
- **Lift**: the upward part of the knockback (studs/s). Not shown.
- **Speed**: swings per minute. Shown on the bat card. Cooldown = 60 / Speed seconds.
- **Stun**: how long the hit player tumbles before getting up (seconds).
- **Safe zone**: a place where nobody can hit or be hit. Every plot is one.
- **Shield**: a short time after joining or respawning when a player cannot hit or be hit.
- **Immunity**: a short time after being hit when a player cannot be hit again.
- **Pair cap**: a limit on how often one player can hit the same other player.
- **Knock grace**: 10 seconds after a hit during which a fall cannot kill you.
- **Golden Crate**: a neutral world item that falls from the sky. Whoever carries it onto their own plot opens it for shards. A bat hit makes the carrier drop it.
- **Frontier merge**: a merge that makes a fruit of your newest rarity. Each one pays shards (decision D-010 in [[Decisions Log]]).
- **HUD cluster**: a top-level element of the `GardenUI` ScreenGui that `HudScale` resizes (`StarterPlayer.StarterPlayerScripts.HudScale.lua:60-160`).
- **Kit**: Samarth's image-based UI kit, listed in [[UI Kit Inventory]].

## Why

Samarth's lines that ask for this ([[Brain Dump 2026-10-04]]):

> "also add bats to the game. every time you rebirth your bat upgrades and does more knockback like in steal a pet."

> "also rarity in the index should be somethng that you have to unlock by rebirthing. this will match with the color of the bat that you unlock."

> "make this super planned and tell me exactly what assets you need dont make slop unless it looks clean and studded. the animations need to be top tier."

> "boats can fit up to two people and can also be broken by the artillery i mentioned earlier like weopons"

> "also all Uis you make should follow the UI that we created, if you need any UI stuff then we can create them using chatgpt or gemini image generation following references of current UI"

> "instead everything should be structured. like for players its going to be confusing. all the UI should never overlap and also everything should be super clean."

> "I dont like the emoji-slop that you pushed on the sky island kings feature."

What the system is for. In Steal a Pet and Steal a Brainrot, the bat's job was to stop thieves ([[Steal a Pet - Bats and Rebirth Knockback]]). Stealing left Fruit Empire on 3 Oct 2026, so the bat needs a new job. Here it is a **movement toy and a contest tool**: knock players off sky-island edges, boats and the throne platform, and fight over neutral Golden Crates. The bat's colour shows a player's stage at a glance, and its Power and Speed grow with every rebirth, so the rebirth feels like an upgrade you can hold. A hit can never cost anyone fruit, pets, potions, shards or cash.

## Player view

1. **You join.** When your garden finishes loading, a bat appears in a new slot at the bottom of the screen, to the right of where the carry buttons show (on a phone it sits next to the jump button instead, UI 3). It is grey-blue: Common, because you have 0 rebirths. A small "!" sits on it and a green toast says "Bat unlocked: Common Bat (Power 32, Speed 60)".
2. **You equip it.** Press 1 on a keyboard, L1 on a controller, or tap the slot on a phone. Your character holds the bat in the right hand. One small bar appears above the slot: "POWER 32  SPEED 60".
3. **Your plot is safe.** While you stand on any plot, the second bar reads "SAFE ZONE". You can swing for practice, but you cannot hit anyone there, and nobody can hit you. Merging is never interrupted.
4. **You swing.** Walk onto the plaza between plots. Click, press R2, or tap the Swing button next to the jump button on a phone. The bat swings with a short trail in its colour. If a player stands in front of you within reach, they fly back about 11 studs and tumble for 0.3 s. They cannot be hit again for 1.2 s.
5. **Your hands are one slot.** Picking up a fruit puts the bat away by itself. Put the fruit down and press 1 again.
6. **You rebirth.** The bat changes colour at once to your newest rarity (Uncommon green) and gets stronger: Power 40, Speed 63. A toast says so. The Rebirth panel showed this reward before you confirmed (owned by [[Rarity Gating and Rebirth]]).
7. **Edges matter.** On the sky islands, in the Royal Capital and on the throne platform, players knock each other off. Players sitting in a boat seat cannot be hit (B12m); someone standing on a deck can. A fall caused by a bat never kills you and never costs your potions. You only lose your spot and the climb back.
8. **Golden Crates.** About every 7 to 10 minutes, when 3 or more players are in the server, a Golden Crate falls onto the plaza, the Cloud Garden or the shallows. A toast says where. Hold E to lift it. You walk 15% slower while carrying it. Reach your own plot and it opens for shards: as many as 20 frontier merges pay at your stage. If someone bats you on the way, you drop it, and they can take it.
9. **On a phone.** Tap the slot to equip. A Swing button appears next to the jump button and greys out while it cools down.

## Rules

All checks run on the server. The client only asks and draws.

### Owning and equipping
1. **B1.** Every player owns exactly one bat once `ProfileLoaded` is true (`ServerScriptService.GardenStore.lua:800`). It is a `Tool` named `Bat` in the player's Backpack. It is never sold, never dropped (`CanBeDropped = false`), never stored, never counted in `StoredTools`, and never saved as an item.
2. **B2.** Bat tier `T = min(1 + Rebirths, 9)`. Tier 1 is Common and tier 9 is Transcendent, in the order of `Config.RarityColors`.
3. **B3.** The bat's colour is `Config.RarityColors[tierName]` (`ReplicatedStorage.GardenConfig.lua:582-592`). It shows on the bat's band, its trail, the HUD icon and the inventory card frame.
4. **B4.** Equip and unequip toggle with key `One`, gamepad `ButtonL1`, a tap or click on the BatSlot, or a tap on the bat card in Inventory. The server does every equip and unequip.
5. **B5.** An equip is refused while the player: carries anything (`CarryingFruit` is set), is `InArena`, has the Invisible effect, sits in a `VehicleSeat`, is dead, or has not loaded. **One hand:** equipping the bat first holsters a drawn weapon ([[Weapons and Artillery]], `G.holsterWeapon`) or stops mining ([[Underground and Ores]]). Decision D-025. A refusal for carrying shows the toast `BatHandsFull`, at most once per 5 s. Other refusals are silent.
6. **B6.** The bat unequips by itself when: `CarryingFruit` becomes set (fruit, tool or crate picked up or equipped), `InArena` becomes true, Invisible starts, the player sits in a `VehicleSeat`, a weapon is drawn (`WeaponDrawn` set), or mining starts (`MiningNode` set).
7. **B7.** While `InArena` is true, the bat is taken out of the Backpack. It is given back within 1 s after `InArena` clears.

### Swinging and hitting
8. **B8.** A swing is `MouseButton1` (not over a GUI), `ButtonR2`, or the Swing button. Only while the bat is equipped.
9. **B9.** The client waits `Cooldown(T)` between swings. The server accepts a swing only if at least `0.85 x Cooldown(T)` has passed since that player's last accepted swing. Faster swings are dropped with no reply.
10. **B10.** One swing hits at most one player.
11. **B11.** The attacker's client picks the target at the swing's contact moment: the nearest player within `Reach` (7.5 studs, measured flat), with a height difference of 7 studs or less, within 60 degrees of the attacker's facing, who is not in a safe zone, shielded, immune or invisible as far as the client can tell. It sends that `Player`, or `nil` for a miss.
12. **B12.** The server accepts the hit only if **all** of these hold, using the server's own positions:
    - a. The attacker is loaded, alive, and has the `Bat` tool parented to their character.
    - b. The attacker carries nothing, is not `InArena`, is not invisible, and is not in a `VehicleSeat`.
    - c. The swing passed B9.
    - d. The victim is a `Player` in the server, is not the attacker, and is alive.
    - e. The victim is not `InArena`, not invisible, not shielded, not immune.
    - f. The victim is not the King seated on the throne (`RoyalSeat`, `ServerScriptService.GardenService.Crown.lua:1020-1041`).
    - g. Neither attacker nor victim is in a safe zone.
    - h. Flat distance between the two root parts is at most `Reach + LagSlack` = 7.5 + 4 = 11.5 studs.
    - i. Height difference is at most 7 studs (the arena's rule, `ServerScriptService.GardenService.CrownArena.lua:495`).
    - j. The victim is within 120 degrees of the attacker's facing (generous on purpose, to cover lag).
    - k. A raycast from attacker root to victim root, ignoring all characters, hits no world geometry.
    - l. The pair cap allows it (B19).
    - m. The victim is not sitting in a boat seat (a seat with the `BoatSeat` attribute, [[Boats and Swimming]]). Boat riders are only ever affected through boat damage by flagged artillery ([[Weapons and Artillery]] rule 22). Added by the [[Integration Review]] (conflict C33).
13. **B13.** Aim help. When a swing starts and no target is inside the 60-degree front arc, the client turns its own character toward the nearest valid player within `Reach + 1.5` studs and within 75 degrees of the camera direction, then swings. The client owns its character, so the turn replicates.
14. **B14.** A refused hit still spends the swing. The attacker alone sees a small Guard flash on the target. No toast, except B23.

### What a hit does
15. **B15.** Knockback velocity = flat unit direction from attacker root to victim root, times `Power(T)`, plus `(0, Lift(T), 0)`. If the two roots are closer than 0.5 studs, the attacker's facing is used. `T` is the **attacker's** tier. The victim's tier does not matter.
16. **B16.** The server: sets `Humanoid.Sit = false` if the victim is seated; sends `BatKnock` to the victim's client; adds a `LinearVelocity` (MaxForce 1e6) to the victim's root for 0.12 s, exactly like the arena's push (`ServerScriptService.GardenService.CrownArena.lua:510-536`); sets knock grace; starts immunity; records the pair hit; tells nearby clients to play the hit effects.
17. **B17.** The victim's client: sets the Humanoid state to `Physics`, writes the root velocity, and after `Stun(T)` seconds sets `GettingUp`. A standing Humanoid fights a plain push, so the victim's own client applies it, the way the arena already does ([[Roblox Platform Constraints for the Expansion]], `StarterPlayer.StarterPlayerScripts.RoyalUI.lua:1482-1490`).
18. **B18.** Immunity: after an accepted hit, that victim cannot be hit by any bat for 1.2 s.
19. **B19.** Pair cap: one attacker can land at most 4 hits on the same victim in any rolling 20 s. The 5th is refused (B14).
20. **B20.** A hit never changes: fruit on any plot or in hand, tools, stored items, pets, ores, boats, cash, shards, potions or other effects, the Index, upgrades or royal stats. A carried fruit or tool stays in hand. The only exception is a carried Golden Crate (C7).
21. **B21.** Knock grace: a hit sets `KnockGraceUntil = now + 10 s` on the victim's character. While it is in the future, the fall sweep does not remember the fall (no landing kill) and the thin-air kill is skipped. So a bat can never kill, and so never clears anyone's potions (death clears them, `ServerScriptService.GardenService.lua:554-570`, `:601-604`).

### Where bats work
22. **B22.** Safe zones:
    - a. **Every plot**, owned or not: `isOnPlot(plot, pos)` (`ServerScriptService.GardenService.FruitFactory.lua:63-68`) and `plot.topY - 240 <= pos.Y <= plot.topY + 40`. The band covers all three underground floors (Floor 3's floor slab ends 216 studs down, [[Underground and Ores]] 7.1) and future towers, and stops well below the sky deck at y 443.
    - b. **Tagged parts**: any BasePart with the CollectionService tag `BatSafeZone`. A position inside its box is safe. Tagged at launch: plotless SpawnLocations (16-stud box), the beanstalk climb, the Royal Capital arrival pad (`CourtyardArrival` marker, 24 x 24 x 12 box), every mine floor's interior box (tagged by [[Underground and Ores]], because a cave is wider than a level-0 soil), every Boat Dock's 40-stud dock zone ([[Boats and Swimming]]), and every shop stall other systems build (Boat Shop, Weapons Booth, Royal Armory).
23. **B23.** A hit is refused when the attacker or the victim is in a safe zone. The first refusal of this kind shows the toast `BatSafeZone`, at most once per 60 s per player.
24. **B24.** Bats work everywhere else: the plaza and paths between plots, the forest, the shallows and the sea edge, every sky island, the Royal Capital outside the arrival pad, the throne room and its dais, and players standing on boat decks (never seated riders, B12m).
25. **B25.** No bat hit can involve a player who is `InArena`. A spectator can never knock a fighter off the ring, which would score a point (`ServerScriptService.GardenService.CrownArena.lua:1231-1246`).

### Protection
26. **B26.** Shields: 20 s after `ProfileLoaded`, and 5 s after each respawn. A shielded player cannot hit or be hit. Swinging ends your own shield at once.
27. **B27.** An invisible player cannot equip, swing or be hit.
28. **B28.** The King cannot be hit while seated on the throne. Anyone standing on the dais can be.
29. **B29.** Bats hit players only. Never knights, pets, crates, fruit, ores or boats.
30. **B30.** Rebirth: when `Rebirths` changes, the server updates `BatTier`, the colour and the stats in the same frame, even if the bat is equipped. If the new tier is above `BatSeen`, the toast `BatUnlocked` fires once and the slot shows "!" until the bat is next equipped. That equip sets `BatSeen = T` and saves. **Timing:** after a rebirth the toast is sent 3 s after the `RebirthDone` notice; on join (a new player, or an old save meeting its first bat) it is sent at `Config.RefundNoticeDelay` + 8 s, after the refund and migration notices. The green toast has one label and a new line replaces the one on screen (`ReplicatedStorage.GardenToast.lua`, "A second message on the same channel restarts that label"), so two notices sent in the same frame lose one. (Added by the player-path pass, [[New Player - First Session]] and [[Existing Player Migration]].)

### Golden Crate
31. **C1.** The server drops a crate first at 300 s after server start, then after a random 420 to 600 s gap. It only drops when at least 3 players are loaded and own a plot, and no crate exists. Otherwise it retries every 30 s.
32. **C2.** The spot is a random `CrateSpot` marker. A spot inside a safe zone is skipped.
33. **C3.** The crate falls from 120 studs above the spot over 2.5 s. A vertical beam marks it until someone lifts it. Every player gets the toast `CrateFalling` with the spot's `Label` attribute.
34. **C4.** Lifting uses a ProximityPrompt (E or ButtonX, hold 0.5 s, range 10). It needs: loaded, owns a plot, carries nothing, not `InArena`, not invisible, and not inside the 2 s drop lock (C7). Lifting unequips the bat (B6).
35. **C5.** While carried, the crate is welded above the head by `holdFruit` (`ServerScriptService.GardenService.Carrying.lua:95-143`). `CarryingCrate` is true, walk speed is x0.85, the bat cannot be used, and the carry row shows only Drop.
36. **C6.** The carrier claims it the moment they stand on their own plot (`standingOn`, `ServerScriptService.GardenService.FruitFactory.lua:79-85`). The server checks every 0.2 s while a crate is carried. The claimer gets `CrateShards(S)` (Numbers), the toast `CrateReward`, and every player gets `CrateClaimed`. The crate is removed.
37. **C7.** Drops: (a) the Drop button drops it at the feet; (b) a bat hit drops it at the feet and the victim cannot lift it again for 2 s; (c) death, leaving, rebirth and entering the arena drop it at the last position. If that position is below y -50 or the server reads it as deep water, the crate goes back to its first spot.
38. **C8.** One carry lasts at most 90 s. Then the crate drops at the feet.
39. **C9.** A crate lives 150 s in total on the ground (the clock pauses while it is carried). Then it fades out.
40. **C10.** A claim is refused if the straight-line distance from where it was lifted to where it is claimed, divided by the carry time, is above 45 studs/s. The crate then goes back to its first spot. Normal top speed is 20 x 1.6 (Speed potion) x 0.85 = 27.2 studs/s (`ReplicatedStorage.GardenConfig.lua:943`, Speed potion x1.6 in [[Carrying, Merging, Storage, Rebirth, Tools and Saving]]).
41. **C11.** Nobody owns a crate until it is claimed. Its shards are new shards, never taken from anyone.
42. **C12.** The stage used for the reward is the claimer's stage at the moment of the claim: `S = min(1 + Rebirths, 9)`.

## Numbers

### The bat ladder (proposal)
Formulas, with `T` = tier = stage:
- `Power(T) = 24 + 8T` studs/s
- `Lift(T) = 12 + 2T` studs/s
- `Speed(T) = 57 + 3T` swings per minute; `Cooldown(T) = 60 / Speed(T)` s
- `Stun(T) = 0.25 + 0.05T` s
- Distance estimate on flat ground, `D(T) = Power x (0.12 + t)`, where `t = (Lift + sqrt(Lift^2 + 2g x 0.12 x Lift)) / g`. The 0.12 s is the server's LinearVelocity hold. `g` = 196.2, Roblox's default `workspace.Gravity`. The place's real value must be read in Studio. Sliding after landing is not counted.

| Stage | Rarity | Rebirths | Colour (RGB) | Full plot $/s (from [[Stage Model]]) | Power | Lift | Speed | Cooldown s | Stun s | Est. distance studs |
|---|---|---|---|---|---|---|---|---|---|---|
| 1 | Common | 0 | 151, 169, 189 | 707 | 32 | 14 | 60 | 1.000 | 0.30 | 10.9 |
| 2 | Uncommon | 1 | 78, 194, 101 | 430K | 40 | 16 | 63 | 0.952 | 0.35 | 14.5 |
| 3 | Rare | 2 | 78, 164, 230 | 3.7B | 48 | 18 | 66 | 0.909 | 0.40 | 18.5 |
| 4 | Epic | 3 | 170, 93, 222 | 3.6T | 56 | 20 | 69 | 0.870 | 0.45 | 22.9 |
| 5 | Legendary | 4 | 242, 171, 48 | 1.8Qa | 64 | 22 | 72 | 0.833 | 0.50 | 27.6 |
| 6 | Mythic | 5 | 232, 66, 60 | 2.5Qi | 72 | 24 | 75 | 0.800 | 0.55 | 32.6 |
| 7 | Astral | 6 | 71, 222, 217 | 265Qi | 80 | 26 | 78 | 0.769 | 0.60 | 38.0 |
| 8 | Eternal | 7 | 237, 81, 195 | 506Sx | 88 | 28 | 81 | 0.741 | 0.65 | 43.7 |
| 9 | Transcendent | 8 | 252, 247, 227 | 24.9Sp | 96 | 30 | 84 | 0.714 | 0.70 | 49.7 |

Worked values:
- **Stage 1** (T = 1): Power 24 + 8 = 32. Lift 12 + 2 = 14. Speed 57 + 3 = 60, so cooldown 1.000 s. Stun 0.30 s. About 11 studs of push.
- **Stage 3** (T = 3): Power 48, Lift 18, Speed 66 (cooldown 0.909 s), Stun 0.40 s. About 18 studs.
- **Stage 6** (T = 6): Power 72, Lift 24, Speed 75 (cooldown 0.800 s), Stun 0.55 s. About 33 studs.
- **Stage 9** (T = 9): Power 96, Lift 30, Speed 84 (cooldown 0.714 s), Stun 0.70 s. About 50 studs.

Why these values:
- They sit on the arena's scale. A knight pushes 34/14, a Strike 40/16, a Heavy 74/26 and the Royal Slam 92/32 (`ReplicatedStorage.CrownConfig.lua:92-164`). The research note proposed "a Bat 0 below the Strike and a top bat near the Slam" ([[Steal a Pet - Bats and Rebirth Knockback]]). Tier 1 is 32 and tier 9 is 96.
- Every tier pushes visibly further (about +4 to +6 studs per tier, 4.6 times as far at tier 9). Players complained that Steal a Brainrot slaps "mostly do the same thing" ([[Steal a Pet - Bats and Rebirth Knockback]]).
- Stun stays below immunity (0.70 s < 1.2 s), so a victim always gets at least 0.5 s of control between hits.
- Knockback is not money, so it is not scaled to income. It is tied to the stage by tier.

### What a bat costs
| Stage | Bat price |
|---|---|
| 1 to 9 | 0 cash and 0 shards (0% of that stage's rebirth cash) |

Steal a Brainrot and Steal a Pet sell each new slap for about 1% and 5% of the rebirth it follows ([[Steal a Pet - Bats and Rebirth Knockback]]). Samarth's line says the bat simply "upgrades" when you rebirth, so the rebirth itself is the price. No Robux item gives Power or Speed (B-rules; [[Steal a Pet - Bats and Rebirth Knockback]] warns against pay-to-win knockback).

### Fixed rule numbers (not stage-based, because they are timing and distance, not money)
| Name | Value | Where it comes from |
|---|---|---|
| Reach | 7.5 studs | same as the arena Strike range (`ReplicatedStorage.CrownConfig.lua:92-145`) |
| LagSlack (server only) | 4 studs | covers about 150 ms of lag at walk speed 26 (proposal) |
| Max height difference | 7 studs | arena rule (`ServerScriptService.GardenService.CrownArena.lua:495`) |
| Client arc / server arc | 60 / 120 degrees each side | proposal |
| Aim help cone | 75 degrees, Reach + 1.5 studs | proposal |
| Server swing tolerance | 0.85 x Cooldown(T) | proposal |
| Push hold | LinearVelocity 0.12 s, MaxForce 1e6 | arena push (`ServerScriptService.GardenService.CrownArena.lua:510-536`) |
| Immunity | 1.2 s | proposal |
| Pair cap | 4 hits per 20 s per attacker-victim pair | proposal |
| Join shield / respawn shield | 20 s / 5 s | Steal a Brainrot uses a 30 s join lock ([[Steal a Pet - Bats and Rebirth Knockback]]) |
| Knock grace | 10 s | a 643-stud fall takes 2.6 s at g 196.2 |
| Safe band below / above a plot | 240 / 40 studs | covers all 3 mine floors ([[Underground and Ores]] 7.1); raised from 160 in the [[Integration Review]] |
| Effects range | 150 studs | proposal |
| Equip rate limit | 4 per second | proposal |
| Swing animation / contact marker | 0.40 s / 0.14 s | proposal (fallback contact 0.12 s) |
| Hit stop (attacker, published animation only) | 0.05 s | shorter than Capcom's 8-frame light hit ([[Top-Tier Sword Combat and Animation in Roblox]]) |
| Camera shake attacker / victim | 0.6 deg for 0.12 s / (0.8 + 0.15T) deg for 0.25 s | proposal, based on the arena shake (`StarterPlayer.StarterPlayerScripts.RoyalUI.lua:1126-1137`) |

### Golden Crate reward (proposal)
Formula: `CrateShards(S) = 20 x ShardDrop(S)`, computed as `Config.ShardReward(20, Rebirths)` (the shared helper, D-017), where `ShardDrop(S)` is the shards one frontier merge pays at stage S (D-010: 3, 7, 15, 32, 70, 155, 340, 748, 1,646). If D-010 changes, the crate follows it.

The last three columns come from a run of the stage model (7 careers, 3 s merges, pets and ores on; the same settings as `~/fruit-empire/sim/final.py`).

| Stage | Rarity | Full plot $/s | Shards per frontier merge | Crate shards | Share of that stage's rebirth shard cost | Minutes of frontier merging it equals | Share of all frontier shards in that stage |
|---|---|---|---|---|---|---|---|
| 1 | Common | 707 | 3 | **60** | 120% of 50 | 4.4 | 29.9% |
| 2 | Uncommon | 430K | 7 | 140 | 56% of 250 | 3.8 | 11.2% |
| 3 | Rare | 3.7B | 15 | **300** | 20% of 1.5K | 3.1 | 5.0% |
| 4 | Epic | 3.6T | 32 | 640 | 18% of 3.5K | 3.9 | 5.2% |
| 5 | Legendary | 1.8Qa | 70 | 1,400 | 19% of 7.5K | 4.6 | 5.2% |
| 6 | Mythic | 2.5Qi | 155 | **3,100** | 16% of 20K | 4.2 | 4.0% |
| 7 | Astral | 265Qi | 340 | 6,800 | 23% of 30K | 7.6 | 6.3% |
| 8 | Eternal | 506Sx | 748 | 14,960 | 15% of 100K | 5.9 | 3.9% |
| 9 | Transcendent | 24.9Sp | 1,646 | **32,920** | (final stage) | - | - |

Worked values: stage 1 = 20 x 3 = 60; stage 3 = 20 x 15 = 300; stage 6 = 20 x 155 = 3,100; stage 9 = 20 x 1,646 = 32,920.

What it means:
- A crate is worth 3 to 8 minutes of frontier merging. That makes a short detour worth it, but camping crates is not.
- Shards only, no cash. Cash is what blocks a rebirth ([[Stage Model]]), so the crate cannot skip a stage.
- Stage 1 looks large (120% of the 50-shard cost), but a stage-1 player already holds a median of 232 shards at the gate, so shards never block that rebirth ([[Stage Model]]).
- Crates per server: 3,600 / 510 s average gap = about 7 per hour. With 6 players that is about 1.2 per player per hour. At stage 3 that is 360 shards an hour against about 5,900 from frontier merges: +6%.

### Golden Crate timing
| Key | Value |
|---|---|
| First drop | 300 s after server start |
| Gap between drops | random 420 to 600 s |
| Minimum players | 3 loaded players who own a plot |
| Fall height / fall time | 120 studs / 2.5 s |
| Lift prompt | hold 0.5 s, range 10 studs |
| Carry speed | x0.85 |
| Longest single carry | 90 s |
| Ground life | 150 s total |
| Drop lock after being hit | 2 s, for the dropper only |
| Claim check | every 0.2 s while carried |
| Anti-teleport limit | 45 studs/s average |

## UI

All sizes are at the HUD reference viewport of 1600 x 900 (`ReplicatedStorage.GardenConfig.lua:1871`). `HudScale` scales every cluster and its offsets by one factor (`StarterPlayer.StarterPlayerScripts.HudScale.lua:101-105`, `:137-160`). All text is `Font.new("rbxasset://fonts/families/FredokaOne.json")`, white, with a black `UIStroke` of 2.5 to 3 px and `TextScaled = true`, like the rest of the kit (`StarterPlayer.StarterPlayerScripts.ShopMenu.lua:47`). No emoji and no non-ASCII characters in any UI string.

### 1. BatSlot (new HUD cluster, bottom centre-right; mouse, keyboard and gamepad only)
- **What:** the bat's equip button. It works like a one-slot hotbar. Key `1` is always the bat; the [[Weapons and Artillery]] Weapon Bar uses keys 2 to 5, so the two read as one hotbar (D-025).
- **Touch devices:** the BatSlot is not drawn in the bottom row. Its job moves to the touch action slot (UI 3), next to the jump button. On a 667 x 375 phone a bottom-row slot would sit on top of the action slot ([[Integration Review]], conflict C21).
- **Structure:** a transparent `Frame` named `BatSlot` (the cluster, so HudScale gives it a UIScale) holding an `ImageButton` named `Button`. HudScale's own comment says button clusters must have no children (`StarterPlayer.StarterPlayerScripts.HudScale.lua:77-84`), so the cluster is a Frame and the button sits inside it.
- **Art:** the HUD buttons are cartoon icons with a baked white label, not boxes (see `ui-reference-hud-icons.png`). The slot follows that: `Icon` = the new tintable bat illustration (asset request 1), tinted with `ImageColor3` = the tier colour; `Label` = the new baked "Bat" label image (asset request 2), never tinted. **Until the art exists:** `Icon` is a `ViewportFrame` showing the real tinted bat mesh at a 30-degree angle, and `Label` is a FredokaOne TextLabel "Bat" with a 3 px black stroke. Both are kit-compliant: no code-drawn frames or boxes.
- **Size and position:** Frame 96 x 92, `AnchorPoint (0.5, 1)`, `Position UDim2(0.5, 320, 1, -21)`. On screen: x 1072 to 1168, y 787 to 879. It shares the bottom baseline (-21) with the carry buttons.
- **Children:**
  - `Selection`: the kit's card selection outline (113285622864672), 112 x 108, centred, behind the icon. Visible only while equipped.
  - `Key`: a 26 x 26 FredokaOne TextLabel at the top-left corner, max text size 22. Shows "1" with a keyboard, "L1" with a gamepad, hidden on touch.
  - `Badge`: the kit alert badge "!" (79539007098845), 31 x 35, at the top-right corner. It pops in with a child UIScale named `Pop`, the same way AlertBadges does it (`StarterPlayer.StarterPlayerScripts.AlertBadges.lua:36-45`). Visible while `BatTier > BatSeen`.
- **States:**
  - Ready: full colour.
  - Equipped: `Selection` shown.
  - Hands full (carrying anything): `Button.ImageTransparency` and `Icon` transparency 0.45. A click shows `BatHandsFull`.
  - Hidden: while `InArena`, or while the character is missing.
- **Behaviour:** `ButtonAnimator` adopts the inner button on its own (`StarterPlayer.StarterPlayerScripts.ButtonAnimator.lua:207-217`). It is not added to the magnetic side-button list (`StarterPlayer.StarterPlayerScripts.ButtonAnimator.lua:52-55`).

### 2. BatInfo (new HUD cluster, above the BatSlot)
- **What:** one bar that shows the bat while it is equipped. (It was two bars; the top one would have crossed the green notice toast lane at y H - 178 to H - 150, so the tier name moved to the bat's colour and the Inventory card. Changed by the [[Integration Review]], conflict C22.)
- **Art:** the kit's carry rate bar, CarryInfo.RateBar (120297790302429, 168 x 27). No new art.
- **Size and position:** 168 x 27, `AnchorPoint (0.5, 1)`, `Position UDim2(0.5, 320, 1, -119)`. On screen: x 1036 to 1204, y 754 to 781. That is 6 px above the BatSlot and 4 px below the notice toast lane (y 722 to 750 at 900 tall).
- **Text:** "POWER 48   SPEED 66". It reads "SAFE ZONE" while you stand in a safe zone, and "SHIELD 14" while your shield is up (seconds left).
- **Visible** only while equipped, on devices with a keyboard or gamepad, and while no full-screen panel is open (Shop, Index, Inventory, Upgrades, Rebirth). Hidden on touch-only devices, where it would collide with the Swing button (overlap table below). Phone players see Power and Speed on the inventory card.

### 3. Touch action slot: Bat, Swing (touch only, own ScreenGui)
- **What:** the one phone action button next to the jump button. It is the same spot the arena's Attack button uses, so "attack" is always under the same thumb (`StarterPlayer.StarterPlayerScripts.RoyalCombat.lua:259`, `:284-315`). The slot is shared with [[Weapons and Artillery]] (decision D-025). It shows exactly one thing, in this order:
  1. In the arena: nothing (RoyalCombat's buttons own the jump area; `StarterPlayer.StarterPlayerScripts.RoyalCombat.lua:320-331`).
  2. A weapon is drawn: the weapon Fire button ([[Weapons and Artillery]] UI 5).
  3. Seated in a `VehicleSeat` (driving a boat) with no weapon drawn: hidden. B5 refuses the bat there, so a Bat button would always be refused. (Fixed by the player-path pass, [[Mobile Player]].)
  4. The bat is equipped: **Swing** (tap to swing; hold 0.6 s to put the bat away).
  5. Otherwise, while not carrying: **Bat** (tap to equip). This replaces the bottom-row BatSlot on touch, with the same "!" badge.
  6. Carrying anything: hidden (the carry row is the hands).
- **Art:** the bat illustration (asset request 1), tinted the tier colour, turned 30 degrees, with the baked "Swing" label (asset request 3) or "Bat" label (asset request 2) under it. Until the art exists: the ViewportFrame bat plus a FredokaOne label.
- **Size and position:** 128 x 128 (x0.72 when the shorter screen side is 500 px or less). Placed relative to the real jump button: `(jumpCentre.X - (r + 80)k, jumpCentre.Y - (r x 0.2 + 30)k)`, the same formula as `layout()` in `StarterPlayer.StarterPlayerScripts.RoyalCombat.lua:284-315`. Re-laid out when the viewport size changes.
- **Cooldown:** a second copy of the icon, `ImageColor3` black at 0.5 transparency, inside a clipping Frame whose height is the fraction of cooldown left. Image-based, so no code-drawn shapes.
- **Safe zone:** the whole button sits at 0.5 transparency while you are in a safe zone or shielded.
- **ScreenGui:** `BatControls`, `DisplayOrder` 3 (the HUD band of [[King UI Overhaul]] section 4.7: above RoyalCombat's 2, below the modal band at 20), `ResetOnSpawn` false, enabled only on touch devices and not `InArena`. BatHud owns this ScreenGui; WeaponClient asks it to show Fire through the `WeaponDrawn` attribute instead of making its own button.

### 4. Bat card in Inventory
- **Where:** the first card in the Inventory panel, `LayoutOrder` -2000, before tool cards (-1000) and fruit (`StarterPlayer.StarterPlayerScripts.StoredFruitsMenu.lua:384-443`).
- **Art:** the tier's rarity card frame from `Config.RarityFrames` (Common 131104257655900 ... Transcendent 130274939131058). The same art fruit cards use, so no new art is needed.
- **Content:** a ViewportFrame of the tinted bat; a title "RARE BAT" (stroke 2.5); "POWER 48"; "SPEED 66". When equipped, the kit's card selection outline (113285622864672).
- **Tap:** toggles equip (sends `BatEquip`) and closes the panel, like equipping a tool. There is no overlay, so it does not add to the code-drawn Equip overlay the kit audit flagged (`StarterPlayer.StarterPlayerScripts.StoredFruitsMenu.lua:203-276`).
- The storage count "n/cap" only counts fruit (`ReplicatedStorage.GardenConfig.lua:539-549`), so the bat card does not change it.

### 5. Toasts
They go through the existing `Notify` (red) and `Notice` (green) remotes and GardenToast (`ServerScriptService.GardenService.lua:66-70`). Every string is a Config key.

| Key | Channel | Text | Who | Limit |
|---|---|---|---|---|
| `BatUnlocked` | Notice | "Bat unlocked: %s Bat (Power %d, Speed %d)" | the player | once per new tier (B30) |
| `BatSafeZone` | Notify | "Plots are safe zones. Bats work everywhere else." | the attacker | once per 60 s |
| `BatHandsFull` | Notify | "Put down what you are carrying first" | the player | once per 5 s |
| `CrateFalling` | Notice | "A Golden Crate is falling on %s" | everyone | once per crate |
| `CrateClaimed` | Notice | "%s brought the Golden Crate home" | everyone except the claimer | once per crate |
| `CrateReward` | Notice | "Golden Crate: +%s Shards" | the claimer | once per crate |
| `CrateNoPlot` | Notify | "You need a plot to carry the crate home" | the player | once per 10 s |
| `CrateNoStore` | Notify | "The crate cannot be stored. Carry it home." | the player | once per 5 s |

### 6. Golden Crate in the world and in the carry row
- **Billboard:** a BillboardGui on the crate root, 6 x 1.4 studs, 3.5 studs above it, FredokaOne "GOLDEN CRATE", white with a 3 px stroke, `MaxDistance` 300. It stays on the crate while carried, so other players can see who has it. Crates never land on plots, so they never meet fruit name tags.
- **Carry row:** while `CarryingCrate` is true, GardenClient shows only the kit Drop button (104911876480159). The NameBar reads "GOLDEN CRATE" and the RateBar "TAKE IT HOME". Store is hidden.

### 7. What is deliberately not added
No centre-screen banners, no confetti, no damage numbers, no hit counters, no red Highlight flash. Highlights are capped at 24 and shared with merge glow (`ReplicatedStorage.GardenConfig.lua:1887-1889`). The calm, structured look is what Samarth asked for.

### 8. Overlap check
At 1600 x 900 (desktop). The full game-wide table is in [[Integration Review]] section 4.

| Element | Rect (x, y) | Neighbour | Neighbour rect | Result |
|---|---|---|---|---|
| BatSlot | 1072-1168, 787-879 | CarryButtons (bottom centre, y -21, buttons 168 x 56) | even a 3-button row (about 520 wide) ends at x 1060 | clear by 12 px; **measure in Studio** |
| BatSlot | same | Weapon Bar (sea only, 448 x 102, bottom centre) | x 576-1024 | clear by 48 px |
| BatSlot | same | EffectStack (bottom right, 262 x 275 block from [[Potions]]) | x 1322-1584 | clear by 154 px |
| BatSlot | same | right column (Rebirth, Merge All, Clear) | x 1488 and up | clear |
| BatSlot | same | CornerHud (bottom left, 222 x 161) | x 16-238 | clear |
| BatInfo | 1036-1204, 754-781 | notice toast lane (centre, unscaled) | y 722-750 | clear by 4 px |
| BatInfo | same | CarryInfo (y -87, 168 wide, centred) | x 716-884 | clear |
| BatInfo | same | Weapon Bar | x 576-1024, y 777-879 | clear by 12 px across |
| BatInfo | same | full-screen panels (centred, up to about 948 x 534) | bottom about y 717 | clear, and BatInfo hides while any panel is open |
| BatSlot, BatInfo | - | RoyalCombat ability bar and legend | only while `InArena` | never both: bat UI hidden in the arena |

On a phone (667 x 375 and 812 x 375, HUD factor 0.55, action slot k = 0.72; jump button at the Roblox small-screen default):

| Element | Rect at 667 x 375 | Neighbour | Neighbour rect | Result |
|---|---|---|---|---|
| Touch action slot | x 478-570, y 247-339 | Weapon Bar (sea) | x 210-456, y 307-364 | clear by 22 px |
| Touch action slot | same | jump button | x 572-642, y 285-355 | clear (formula places it left of the jump) |
| Touch action slot | same | right column (Clear bottom) | x 605-658, y up to 238 | clear |
| Touch action slot | same | potion effect block | moved to the top left on touch ([[Potions]] UI 3) | clear |
| BatSlot, BatInfo | - | - | not drawn on touch | - |

On a tablet (1024 x 768, factor 0.65) the action slot sits left of the jump button the same way; nothing else is in that corner on touch.

The real jump button position is read at runtime, so the action slot follows it. Test plan T19 checks all three screens.

## World and assets

### The bat model
- **Source:** the existing `Workspace.Bat` Tool: classic bat mesh 54983181, texture 54983107, mesh scale (3, 3, 1.5), grip -1.7 on Z, one Trail with a 0.10 s lifetime ([[Existing VFX and Props]]). Today it lies loose in Workspace at (18, 157, 227). A Tool in Workspace is picked up by whoever touches it, so moving it also removes a stray pickup.
- **What must be built (Studio, by the builder, not Samarth):**
  - Move it to `ServerStorage.BatTemplate`. Set `Name = "Bat"`, `CanBeDropped = false`, `RequiresHandle = true`, `ManualActivationOnly = true`, `ToolTip = ""`.
  - Add a **Band**: a cylinder Part 0.55 x 0.25 x 0.55, Neon, `CanCollide`, `CanQuery` and `CanTouch` false, Massless, welded around the barrel 1.2 studs from the tip. It carries the tier colour clearly at every tier, including the pale ones.
  - Tint the wood: `SpecialMesh.VertexColor` = 55% of the way from white to the tier colour. Acceptance check: in a screenshot at 25 studs, a tester can tell all 9 tiers apart by band plus wood. If not, raise the tint to 80%.
  - Trail: two Attachments on the Handle (base and tip). Colour = tier colour fading to transparent. `Enabled` only during a swing, on each client.
  - Sparkle: tier 5 and up get one ParticleEmitter copied from `Anime > Shiny-01` on the barrel. Rate 4 per second (6 at tier 9), `LightEmission` 1, tier colour. This gives a visible "higher tiers glow" step, matching what Samarth asked for ores.
  - Sounds (already in the Tool): `Swoosh` 3755636638, `Hit` 175024455, `Hit2` 3932505023, `Ouch` 4306991691, `Ouch2` 4459572527, `Equip` 769464514.

### Effects, all from packs already in the place ([[Existing VFX and Props]])
They are copied once into a new folder `ReplicatedStorage.BatFX`. Clients clone from there and colour each clone once. No emitter property is changed every frame, which the performance docs warn about ([[Roblox Platform Constraints for the Expansion]]).

| Moment | Effect | Source model | Tiers | Size of burst |
|---|---|---|---|---|
| Swing | bat trail, tier colour | the Bat's own Trail | all | 0.10 s ribbon |
| Swing | crescent slash | `Anime > Slash-01` | 4 and up | Emit 1 per emitter |
| Hit | impact and flash | `vfx pack > f` (impact, flash and one crescent emitter only) | all | Emit 1 / 1 / 6 |
| Hit | stars | `Anime > Stars-01` | 7 and up | Emit 8 |
| Victim flight | speed lines on the victim's root for the stun time | `vfx pack > Part` (speed lines) | 5 and up | authored rate, 30/s or less |
| Refused hit | small shield flash, attacker only | `Anime > Shield-01` at 0.6 scale | all | 0.3 s |
| Landing after a knock-off | dust puff | `Anime > Smoke-01` | all | Emit 6 |
| Bat at rest | sparkle | `Anime > Shiny-01` | 5 and up | rate 4 to 6/s |

Effects never show before the contact frame. Feedback threads say early particles look wrong ([[Top-Tier Sword Combat and Animation in Roblox]]).

### Sounds
| Moment | Sound | Settings |
|---|---|---|
| Equip | `Equip` 769464514 | volume 0.5, owner only |
| Swing | `Swoosh` 3755636638 | PlaybackSpeed x random 0.95-1.05 |
| Hit | `Hit` and `Hit2`, alternating | PlaybackSpeed `1.10 - 0.03 x (T - 1)` x random 0.95-1.05 (heavier at higher tiers) |
| Victim | `Ouch` and `Ouch2`, alternating | volume 0.45, at most once per 2 s per victim |

The three layers (whoosh, impact, reaction) and the pitch randomness follow the research ([[Top-Tier Sword Combat and Animation in Roblox]]). Audio risk: these are old ids. Roblox audio needs permission to play in an experience ([[Roblox Platform Constraints for the Expansion]]). Each one must be heard once in a live server (T21). A silent one is swapped for a Creator Store licensed sound.

### Animations
- **Swing A** (forehand) and **Swing B** (backhand) alternate. R15, upper body only, `Action` priority, 0.40 s, with a `Hit` marker at 0.14 s. They are published by Samarth (asset request 4), the same constraint as decision D-005.
- **Hold pose:** the default R15 Animate script's tool-hold pose plays by itself when a Tool is equipped. No asset is needed.
- **Fallback until the ids exist:** the client puts a `toolanim` StringValue with value `Slash` into the bat, so the default Animate script plays its built-in slash. This is what the Royal Sword does today (`ServerScriptService.GardenService.CrownArena.lua:229-238`). The client then sends the swing 0.12 s after input.
- **Victim tumble:** the `Physics` Humanoid state (B17). No animation asset.

### The Golden Crate model (must be built)
- One root Part, 3.2 x 3.2 x 3.2, Wood material, colour (161, 110, 56). Six gold trim Parts (Metal, (242, 171, 48)) along the edges. A lid Part with `TopSurface = Studs`, matching the classic studded look of the sky builders (`ServerStorage.ZZTools.SkyIslandBuilder.lua:155`). All welded to the root. 8 Parts in total.
- One `Anime > Shiny-01` emitter (rate 8, gold), one vertical Beam (40 studs wide at the base, fading up, gold) shown while uncarried, and the billboard (UI 6).
- Saved as `ServerStorage.GoldenCrate`. The root carries the attribute `GoldenCrate = true`. **No child may be named `NameTag`**, because GardenClient's prompt handler looks for that name on any prompt's parent (`StarterPlayer.StarterPlayerScripts.GardenClient.lua:124-167`).
- No pack has a crate. The Studded Pack's logs and campfire are the wrong shape ([[Creator Store Packs]]).

### Markers and tags (must be placed)
- `workspace.CrateSpots`: 8 anchored, invisible, non-colliding marker Parts named `CrateSpot`, each with a `Label` attribute. Four on the plaza between plots ("the plaza"), outside every plot's safe band. Two on the Cloud Garden deck at y 443, away from the arena opening at (-20, -170) r 52 ("the Cloud Garden"; `ServerStorage.ZZTools.SkyIslandBuilder.lua:52-53`). Two on shallows islets ("the shallows"). Coordinated with [[Plot Overhaul and Environment]], which owns the plaza and decor.
- `BatSafeZone` tag on invisible boxes (Anchored, CanCollide, CanQuery and CanTouch false): every plotless SpawnLocation (16 x 12 x 16), one tall box around `workspace.beanstalk` (the climb), and the `CourtyardArrival` landing in the Royal Capital (24 x 12 x 24). Each future shop stall gets one from the system that builds it.

## Asset requests for Samarth

Four images, two animations. No Robux products. Audio: none, unless a sound fails the live check (T21).

**1. Bat icon (tintable).** 512 x 512 PNG, transparent background. Upload it and send the id. It goes in `Config.Bats.Images.Icon`.
> Prompt (ChatGPT or Gemini; attach `ui-reference-hud-icons.png` and the "Clear" HUD button image, Roblox asset 88888506311530): "Draw one cartoon baseball bat icon for a Roblox game HUD, 512 x 512 pixels, transparent background. Copy the exact art style of the attached 'Clear' broom icon: the same thick dark outline, the same glossy cartoon shading, the same chunky rounded proportions and the same 3/4 angle. The bat points from lower left (handle) to upper right (barrel), filling about 80% of the canvas. Colour the whole bat body in white to light grey only (shading from #FFFFFF down to #C8C8C8), because the game will tint it a different colour for each rarity. Give the handle a dark charcoal grip tape (#2B2B2B) with two thin wraps, and keep the outline dark. Add one soft white shine streak along the barrel. No text, no sparkles, no background, no emoji."

**2. "Bat" label.** 256 x 96 PNG, transparent background. Goes in `Config.Bats.Images.Label`.
> Prompt (attach the same two images): "Make only the word 'Bat' as a game UI label, 256 x 96 pixels, transparent background. Match the attached HUD labels 'Clear' and 'Shop' exactly: the same heavy rounded font, white letters with the same slight white-to-light-grey vertical gradient, the same thick dark outline and the same small drop shadow. Title case: capital B, lower-case 'at'. Centre it. No icon, no other text, no emoji."

**3. "Swing" label.** 320 x 96 PNG, transparent background. Goes in `Config.Bats.Images.SwingLabel`.
> Prompt (attach the same two images): "Make only the word 'Swing' as a game UI label, 320 x 96 pixels, transparent background. Match the attached HUD labels 'Clear' and 'Shop' exactly: the same heavy rounded font, white letters with the same slight vertical gradient, the same thick dark outline and drop shadow. Title case. Centre it. No icon, no other text, no emoji."

If tinting the white bat looks flat in game, the fallback is 9 pre-coloured copies of image 1, one per rarity colour in [[UI Kit Inventory]]. That is open question 8.

**5. Bat chip (for the Rebirth menu's REWARDS box).** 188 x 200 PNG, transparent outside the chip. Goes in `Config.Bats.Images.Chip`. [[Rarity Gating and Rebirth]] U4 shows it tinted the next tier's colour. Added by the [[Integration Review]] (the Rebirth note pointed here for it, and no request existed).
> Prompt (attach the three reward chips 89282702522748, 82056408182031, 78947574768646 and image 1): "Images 1 to 3 are the small reward chips from my game's Index rewards box. Copy their exact square frame: border colour, corner rounding, inner shading and 47 x 50 proportions. Image 4 is my white bat icon. Put ONE copy of that bat, white to light grey only so the game can tint it, centred and filling 70% of the chip. No text. Transparent outside the chip. Canvas 188 x 200."

**4. Two swing animations.** Publish them and send the two ids. They go in `Config.Bats.Anim.SwingA` and `SwingB`. An id of 0 keeps the `toolanim Slash` fallback, the same pattern as decision D-001.
1. In Studio, Avatar tab, Rig Builder, insert a **Block R15** rig. The game forces R15 ([[Top-Tier Sword Combat and Animation in Roblox]]).
2. Copy `ServerStorage.BatTemplate.Handle` into the rig. Add a Motor6D named `BatGrip` with `Part0 = RightHand`, `Part1 = Handle`, `C0 = RightHand.RightGripAttachment.CFrame`, `C1 = CFrame.new(0, 0, -1.7)`. This only shows where the bat is. Do not key BatGrip.
3. Open the Animation Editor on the rig. **Key only UpperTorso, Head, both arms and both hands.** Never key LowerTorso or the legs, so the walk keeps playing under the swing.
4. **Swing A, forehand, 0.40 s:**
   - 0.00 s: the tool-hold pose (right arm forward, bat up).
   - 0.08 s: wind-up. UpperTorso turned 25 degrees right, right upper arm pulled back 40 degrees, bat behind the right shoulder. Easing Quad Out.
   - 0.14 s: contact. Torso square to the front, right arm straight out at shoulder height, bat level and pointing forward-left. Easing Cubic In, so the swing snaps. Add an Animation Event marker named `Hit` here.
   - 0.24 s: follow-through. Torso turned 30 degrees left, bat over the left shoulder. Easing Quad Out.
   - 0.40 s: back to the hold pose.
5. **Swing B, backhand:** the mirror image, starting from the left side. `Hit` marker at 0.14 s.
6. Set Priority to **Action** and turn Looping off for both.
7. Publish each one (... menu, Publish to Roblox). The game is owned by the user account `hikeALT12212` ([[Roblox Platform Constraints for the Expansion]]). Publish from that account, or publish from yours and grant the experience permission in Creator Hub (Configure, Permissions). Animations no longer need the same owner as the game once permission is granted ([[Top-Tier Sword Combat and Animation in Roblox]]).
8. Time saver: the Sword Animations pack's `swordcombo1` KeyframeSequence (in `ServerStorage.AssetQuarantine`) has a slash combo ([[Creator Store Packs]]). Its first two slashes can be copied as A and B, then retimed to the steps above. Publish these in the same session as the Royal Combat animations.

## Code plan

Every module follows the GardenService pattern: `return function(G)`, required in order, sharing state through `G` (`ServerScriptService.GardenService.lua:88-98`). A module can only take what earlier modules put into G. Anything defined later is looked up through G at call time (`ServerScriptService.GardenService.Carrying.lua:352-353`).

### New files
| File | Kind | Job |
|---|---|---|
| `ReplicatedStorage.BatRules` | ModuleScript (shared) | `Tier(rebirths)`, `Stats(tier)` (Power, Lift, Speed, Cooldown, Stun), `TierName(tier)`, `Colour(tier)`, `InTaggedZone(pos)` (box test over `BatSafeZone` parts), `PlotZoneClient(pos)` (client guess for plots, using the same largest-part soil rule as `StarterPlayer.StarterPlayerScripts.PenUpgradeSign.lua:238-247`). One module, so client and server cannot drift apart. The UI note lists drift as a known trap (gotcha 13 in [[Client UI - HUD and Menus]]). |
| `ServerScriptService.GardenService.Bats` | ModuleScript, `return function(G)` | Remotes, bat granting, tier updates, equip, swing checks, knockback, shields, immunity, pair cap, safe zones, Studio test hooks. |
| `ServerScriptService.GardenService.GoldenCrate` | ModuleScript, `return function(G)` | Crate timer, spawn, lift prompt, carry, claim, drop, timeouts. |
| `StarterPlayer.StarterPlayerScripts.BatClient` | LocalScript | Input, swing animation, target pick, aim help, hit stop, shakes, sounds, effects, the `BatKnock` handler. |
| `StarterPlayer.StarterPlayerScripts.BatHud` | LocalScript | BatSlot and BatInfo states, the Swing button and its layout, key hints, and turning off the default Backpack bar. |
| `StarterGui.GardenUI.BatSlot`, `StarterGui.GardenUI.BatInfo` | GUI authored in Studio | The two kit clusters from the UI section. |
| `ServerStorage.BatTemplate`, `ServerStorage.GoldenCrate`, `ReplicatedStorage.BatFX`, `workspace.CrateSpots` | Instances built in Studio | See World and assets. |

### Load order
Insert after the four Crown modules (`ServerScriptService.GardenService.lua:515-518`) and before the wiring block:
```lua
require(script.Bats)(G)
require(script.GoldenCrate)(G)
```
Bats needs `G.crownHolder` (`ServerScriptService.GardenService.Crown.lua:784`), `G.isOnPlot` and `G.standingOn` (`ServerScriptService.GardenService.FruitFactory.lua:371-373`), `G.plots`, `G.plotFor`, `G.held`, `G.effectActive` (`ServerScriptService.GardenService.lua:391-392`), `G.notifyRemote`, `G.noticeRemote`, `G.Store` and `G.royalDev`. All of them exist before line 518. Ambient loads last and needs nothing from Bats, because it reads a character attribute.

### `Bats` module, function by function
- `grant(player)`: runs on `CharacterAdded` once `ProfileLoaded` is true, and again when `InArena` clears. It clones `ServerStorage.BatTemplate`, sets the Tool attributes `BatOwner` (UserId) and `BatTier`, calls `paint`, and parents it to the Backpack. The Backpack is emptied on every respawn, so each new body gets a fresh bat.
- `paint(tool, tier)`: sets the band colour, `VertexColor`, trail colour and the sparkle (tier 5 and up).
- `refresh(player)`: runs on `player:GetAttributeChangedSignal("Rebirths")`. It sets `BatTier`, repaints any bat the player has, and handles B30. `publishRebirths` is the one writer of `Rebirths` (`ServerScriptService.GardenService.Progression.lua:95-101`), so no change to `RebirthLock` is needed, and rebirth stays yield-free (`ServerScriptService.GardenService.RebirthLock.lua:63-67`).
- `equip(player, want)`: handles the `BatEquip` remote. Checks B5, then `Humanoid:EquipTool(tool)` or `Humanoid:UnequipTools()` on the server, which is how the arena equips its sword (`ServerScriptService.GardenService.CrownArena.lua:209`). Writes `BatEquipped`. On an equip with `BatTier > BatSeen`, sets `BatSeen = BatTier` and calls `Store.Touch(player, Config.SaveTouch)`.
- Auto-unequip listeners (B6): `CarryingFruit`, `InArena`, `InvisibleEndsAt` and `Humanoid.SeatPart`. On `InArena = true`, the bat is moved to a server folder (B7).
- `safeAt(pos)`: plot test (B22a) over `G.plots`, plus a box test over a cached list of `BatSafeZone` parts. The cache is refreshed on `GetInstanceAddedSignal` and `GetInstanceRemovedSignal`.
- `canHit(attacker, victim)`: the list in B12. Returns `ok, reason`.
- `onSwing(player, victim)`: handles `BatSwing`. Rate check (B9), then sends the swing effect to nearby players, then `canHit`. On success it calls `knockPlayer` and runs every function in `G.batHitHooks`.
- `knockPlayer(victim, vel, opts)`: **exported as `G.knockPlayer`** for [[Weapons and Artillery]]. `opts = { stun, kind = "bat" or "blast", source }`. It unseats, sends `BatKnock`, adds the 0.12 s LinearVelocity (copied from `ServerScriptService.GardenService.CrownArena.lua:519-531`; CrownArena itself is not changed), sets `KnockGraceUntil`, and starts immunity per `kind`. It multiplies `vel` by `G.knockScale(victim)` if another system defines it (default 1). That is the hook for any future potion that changes knockback.
- Shields: `shieldUntil[player]` from `ProfileLoaded` and `CharacterAdded`, mirrored to the attribute `BatShieldUntil`.
- `PlayerRemoving`: clears every per-player table (cooldowns, immunity, pair history, shields, refusal-toast timers).
- Exports: `G.knockPlayer`, `G.batSafeAt`, `G.batTierOf`, `G.batHitHooks` (a list).
- Studio test hooks on `G.royalDev`, which exists for exactly this (`ServerScriptService.GardenService.Crown.lua:1188-1222`): `RoyalDev:Invoke("batRebirths", userId, n)` sets `Rebirths` live, and `RoyalDev:Invoke("batStats")` returns accepted, refused and per-reason counts.

### `GoldenCrate` module
- A timer loop (C1), `spawnCrate()` (C2, C3), and a `PromptTriggered` handler that only acts when `prompt.Parent:GetAttribute("GoldenCrate")` is true. GardenService's own prompt handler ignores anything outside the fruit folder (`ServerScriptService.GardenService.lua:522-526`).
- `lift(player)`: checks C4, then `G.holdFruit(player, character, head, crateRoot)` (it "touches nothing fruit-specific", `ServerScriptService.GardenService.Tools.lua:152-155`). Then sets `held[player].crate = true` and `CarryingCrate = true`, and calls `applyWalkSpeed`.
- `G.crateDrop(player, silent, why)`: calls `G.releaseHold(player, false)`, which keeps the item (`ServerScriptService.GardenService.Carrying.lua:73-89`). Then it anchors the crate at the feet (or its first spot, per C7), clears `CarryingCrate`, re-runs `applyWalkSpeed`, and starts the 2 s drop lock if `why == "hit"`.
- A claim loop every 0.2 s while carried (C6, C8, C10). The reward goes through `setShards`, the single shard writer (`ServerScriptService.GardenService.Progression.lua:216-218`), then `Store.Touch`.
- Registers into `G.batHitHooks`: if the victim carries a crate, `crateDrop(victim, false, "hit")`.
- Adds `G.royalDev.crate` so a tester can drop a crate on demand.

### Changes to existing scripts
| File and line | Change |
|---|---|
| `ServerScriptService.GardenService.lua:518` | Add the two `require` lines above. |
| `ServerScriptService.GardenService.lua:215` (`humanoid.WalkSpeed = speed`) | Before it: `if player:GetAttribute("CarryingCrate") then speed *= Config.Crate.CarrySpeed end`. `applyWalkSpeed` is the only WalkSpeed writer, and the arena override above it still wins (`:197-208`). |
| `ServerScriptService.GardenService.Ambient.lua:281-283` | Add `local graced = (character:GetAttribute("KnockGraceUntil") or 0) > workspace:GetServerTimeNow()` and change `if feathered then` to `if feathered or graced then`, so the fall is not remembered. `feathers()` still gets only `feathered`, so no feather effect shows. |
| `ServerScriptService.GardenService.Ambient.lua:353` | Add `and not graced` to the thin-air condition. |
| `ServerScriptService.GardenService.Carrying.lua:291-306` (`dropFromPlayer`) | Before the `if state.tool` branch: `if state.crate then G.crateDrop(player, silent, "drop") return end`. |
| `ServerScriptService.GardenService.Storage.lua:24-33` (`storeCarried`) | Before the tool branch: crate means `Notify CrateNoStore`, then return. |
| `ServerScriptService.GardenService.Merging.lua:158` (`merge`) | `if state.tool or state.crate then return end`. |
| `ServerScriptService.GardenService.RebirthLock.lua:70-73` | Before `releaseHold(player, true)`: `if held[player] and held[player].crate then G.crateDrop(player, true, "rebirth") end`. Otherwise rebirth would destroy the crate. `crateDrop` does not yield. |
| `ServerScriptService.GardenService.Persistence.lua:337-347` (`reclaimCarried`) | Before the tool branch: `if state.crate then G.crateDrop(player, true, "leave") return nil end`. |
| `ServerScriptService.GardenService.Persistence.lua:75` | `profile.batSeen = player:GetAttribute("BatSeen") or 0`. |
| `ServerScriptService.GardenService.Persistence.lua:267` | After `publishRebirths`: `player:SetAttribute("BatSeen", profile.batSeen or 0)`. |
| `ServerScriptService.GardenStore.lua:264` (`Store.Default`) | Add `batSeen = 0`. |
| `ServerScriptService.GardenStore.lua:451-456` (`sanitise`, after rebirths) | Copy `batSeen` if it is a number: floor it, clamp 0 to 9; otherwise count a repair. **Required:** a field missing from sanitise is dropped on the next load (`ServerScriptService.GardenStore.lua:447-450`). |
| `ServerScriptService.GardenService.DevPanel.lua:162-183` (`clearPlayerResidentState`) | Clear `BatSeen`, `BatTier`, `BatEquipped`, `BatShieldUntil` and destroy the Bat tool, or the Round trip and Reload tests pass on old state. |
| `ReplicatedStorage.GardenConfig.lua:1863-1867` (`HudClusters`) | Add `"BatSlot"` and `"BatInfo"`, or they keep their desktop size (`ReplicatedStorage.GardenConfig.lua:1859-1862`). |
| `ReplicatedStorage.GardenConfig.lua` (new section) | `Config.Bats` (all numbers in Numbers, `Images = { Icon = 0, Label = 0, SwingLabel = 0 }`, `Anim = { SwingA = 0, SwingB = 0 }`, the sound ids), `Config.Crate`, and the toast strings. |
| `StarterPlayer.StarterPlayerScripts.GardenClient.lua:225-261` (carry row refresh) | When `CarryingCrate` is true: Drop only, NameBar "GOLDEN CRATE", RateBar "TAKE IT HOME", Store hidden. Also refresh on `CarryingCrate` changes (`:337-349`). |
| `StarterPlayer.StarterPlayerScripts.GardenClient.lua:124-167` (PromptShown) | Return early when the prompt's parent has the `GoldenCrate` attribute: no white outline, no NameTag lookup. |
| `StarterPlayer.StarterPlayerScripts.StoredFruitsMenu.lua:474-496` (`rebuild`) | Build the bat card first. Rebuild on `BatTier` and `BatEquipped` changes. |

### Remotes
All live in `ReplicatedStorage.GardenRemotes` and are created by Bats with the same find-or-create pattern as `remote()` (`ServerScriptService.GardenService.lua:49-57`).

| Name | Type | Direction | Payload | Server check |
|---|---|---|---|---|
| `BatEquip` | RemoteEvent | client to server | `want: boolean` | `typeof == "boolean"`; at most 4 per second; B5 |
| `BatSwing` | RemoteEvent | client to server | `victim: Player` or `nil` | must be nil or an instance whose `Parent == Players`; B9 and B12 |
| `BatKnock` | RemoteEvent (reliable) | server to the victim | `{ vel: Vector3, stun: number, tier: number }` | sent only by the server |
| `BatFX` | UnreliableRemoteEvent | server to players within 150 studs | `(kind: "swing" / "hit" / "guard", attackerId: number, victimId: number?, pos: Vector3?, tier: number)`, under 60 bytes | cosmetic. Unreliable payloads over 1,000 bytes are dropped ([[Roblox Platform Constraints for the Expansion]]); this one is far below that. `guard` goes to the attacker only. |

The crate uses its ProximityPrompt and the existing `Drop` remote, so it adds no remote.

### Save field
| Field | Type | Default | Meaning |
|---|---|---|---|
| `batSeen` | whole number 0 to 9 | 0 | the highest bat tier this player has been told about |

- **Migration:** old saves have no field, so they load as 0. The first join after the update shows one `BatUnlocked` toast for the player's current tier, which doubles as the intro. Players with more than 8 rebirths get tier 9.
- Adding a field needs no schema bump (`ServerScriptService.GardenStore.lua:8-12`). One small integer keeps the self-test's 8,192-byte worst case safe (`ServerScriptService.GardenStore.lua:1209`).
- The bat tier itself is **not saved**. It is worked out from `rebirths`, which rebirth keeps (decision D-009).

### Attributes
| On | Attribute | Type | Written by | Read by |
|---|---|---|---|---|
| Player | `BatTier` | number 1-9 | Bats | BatHud, BatClient, StoredFruitsMenu |
| Player | `BatSeen` | number 0-9 | Persistence (load), Bats (equip) | Bats, BatHud (badge) |
| Player | `BatEquipped` | boolean | Bats | BatHud, StoredFruitsMenu |
| Player | `BatShieldUntil` | server time | Bats | BatHud, BatClient (target guess) |
| Player | `CarryingCrate` | boolean | GoldenCrate | GardenClient, applyWalkSpeed, BatHud |
| Character | `KnockGraceUntil` | server time | Bats | Ambient |
| Tool `Bat` | `BatOwner`, `BatTier` | number | Bats | BatClient (effects on others' bats) |
| Crate root | `GoldenCrate` | boolean | template | GoldenCrate, GardenClient |
| CrateSpot | `Label` | string | Studio | GoldenCrate |

Times that reach clients use `workspace:GetServerTimeNow()`. Server-only cooldowns use `os.clock()`. Never compare the two, the same rule the Crown code follows ([[Royal System - Server]], gotcha "Two clocks").

### Client flow (BatClient)
1. On `CharacterAdded`: load the two swing tracks once and reuse them. `LoadAnimation` on every swing makes a new track each time ([[Top-Tier Sword Combat and Animation in Roblox]]).
2. Input: `UserInputService.InputBegan`, skipping `processed`, as RoyalCombat does (`StarterPlayer.StarterPlayerScripts.RoyalCombat.lua:148-158`). Key `One` / `ButtonL1` send `BatEquip`. `MouseButton1` / `ButtonR2` swing when equipped. RoyalCombat also reads `MouseButton1` and `ButtonR2`, but only while `InArena` (`StarterPlayer.StarterPlayerScripts.RoyalCombat.lua:148-150`), when the bat is gone. Apart from that, the only key bindings in the export are the prompt key `E` and the dev panel's `F4`.
3. Swing: check the local cooldown, run aim help (B13), play track A or B (or put in `toolanim`), turn on the trail, play the whoosh. At the `Hit` marker (or after 0.12 s), pick the target (B11) and `FireServer(victim)`. If the target looked valid locally, hit-stop for 0.05 s with `AdjustSpeed(0)` on the published track, spark at the contact point, shake 0.6 degrees.
4. `BatFX` from the server: play the swing or hit effects and sounds for other players' bats. Ignore your own swing echo.
5. `BatKnock`: unseat, `ChangeState(Physics)`, write `AssemblyLinearVelocity`, shake `(0.8 + 0.15T)` degrees, add speed lines at tier 5 and up, and after `Stun` call `ChangeState(GettingUp)`. When the next `Landed` state follows a knock, puff dust.

### Client flow (BatHud)
1. Turn off the default Backpack bar with `StarterGui:SetCoreGuiEnabled(Enum.CoreGuiType.Backpack, false)`, retrying for 5 s until it succeeds. The default bar sits at the bottom centre and would collide with the carry row. No script turns it off today (search of the export). **BatHud is the only script that touches this setting**, once, at start. [[Weapons and Artillery]] and [[Underground and Ores]] do not toggle it (an on/off toggle when leaving a mine would bring the bar back for everyone; [[Integration Review]] conflict C35).
2. Drive the BatSlot and BatInfo states from attributes. Poll the safe-zone guess 5 times a second only while equipped.
3. Touch: lay out the touch action slot (UI 3) relative to the real jump button, re-lay it out on `ViewportSize` changes, and pick its content (Fire, Swing, Bat or hidden) from `WeaponDrawn`, `BatEquipped`, `CarryingFruit` and `InArena`.
4. Never write the Position or Size of `BatSlot` or `BatInfo`. HudScale is their only writer (`StarterPlayer.StarterPlayerScripts.HudScale.lua:116-128`). Only `Visible` and children are touched.

## Edge cases and abuse

**Exploits**
- **Hits from far away:** the server checks distance, height, arc and line of sight on its own positions (B12). The client's target choice is only a suggestion. `Touched` is never used, because a hacked client can fake it ([[Steal a Pet - Bats and Rebirth Knockback]]).
- **Fast-fire scripts and auto-clickers:** the server's own cooldown (B9). The platform's limit of about 500 remote calls a second is far too loose to rely on ([[Roblox Platform Constraints for the Expansion]]).
- **Anti-knockback scripts:** the server's 0.12 s LinearVelocity still moves the victim. Fully stopping position cheats needs Server Authority, which is a game-wide change and out of scope ([[Roblox Platform Constraints for the Expansion]]).
- **Teleporting next to someone:** the exploiter controls their own character's position, so this is possible. Immunity and the pair cap limit it to 4 pushes per 20 s, and a push takes nothing.
- **Teleporting a crate home:** refused by C10, and the crate goes back to its spot.
- **Remote spam with bad payloads:** type checks first, then silent drops.

**Griefing**
- **Chasing one player:** 1.2 s immunity, a pair cap of 4 per 20 s, safe zones one walk away, and no loss of anything.
- **Knocking new arrivals off the sky island:** the arrival pad is a safe zone. A knock-off costs time only, and the Aether potion keeps running because nobody dies (B21).
- **Blocking the beanstalk climb:** the beanstalk is a safe zone (B22b).
- **Hitting the King off his throne:** refused while he sits (B28).
- **Invisible griefers:** an invisible player cannot equip or swing (B27).
- **Drive-bys from a boat:** a `VehicleSeat` driver cannot swing. Passengers can.
- **Arena spectators:** no hits involving `InArena` players (B25).

**Leaving, rejoining and rebirthing mid-action**
- Leaving mid-swing: nothing to clean up beyond the per-player tables.
- Leaving while knocked: the knock grace lives on the character and dies with it.
- Leaving with a crate: dropped at the last position (C7c), through `reclaimCarried`.
- Rejoining resets cooldown, immunity and pair history. Rejoining takes much longer than 20 s, so it cannot be used to beat the pair cap faster.
- Rebirth while equipped: recoloured on the same frame (B30).
- Rebirth while carrying a crate: dropped first (code plan change at `ServerScriptService.GardenService.RebirthLock.lua:70-73`).
- Death while carrying a crate: `humanoid.Died` already calls `dropFromPlayer(player, true)` (`ServerScriptService.GardenService.lua:601-604`), which reaches the crate branch.
- Entering the arena with a crate: `enterArena` calls the silent drop (`ServerScriptService.GardenService.CrownArena.lua:268`), so the crate drops where the player stood.

**Mobile and controllers**
- No hover is needed. Equip is one tap, swing is the Swing button.
- The world tap does not swing: `ManualActivationOnly = true`, so dragging the camera never swings by accident.
- Aim help (B13) makes standing swings land on a phone.
- Gamepad: R2 swings, L1 toggles. The default Backpack bar is off, so L1 and R1 no longer cycle hotbar tools.

**Server authority and other cases**
- The client predicts only effects. The server decides every hit, push, shield, claim and reward.
- Plotless players: 50 can join but there are 12 plots ([[Roblox Platform Constraints for the Expansion]]). Plotless players can bat, are safe at their spawn box, and cannot lift crates (C4).
- Streaming: a client may not have far plots loaded, so its safe-zone guess can be wrong far away. The server is the authority, so the only cost is a wasted spark.
- Fall into the deep sea: the bat's top push is about 50 studs, and the sea only kills past radius 2,825 (`ServerScriptService.GardenService.Ambient.lua:376-381`), so it cannot happen.
- Respawn: the Backpack is emptied, the new body gets a new bat, and a 5 s shield starts.
- The Studio playtest writes the real save unless `workspace.StudioNoSave = true` (`ServerScriptService.GardenStore.lua:572-575`).

## Performance

Measured against [[Roblox Platform Constraints for the Expansion]]:
- **Parts:** a bat is 2 Parts (Handle and Band). Only equipped bats are in Workspace, so at most 100 Parts at the 50-player cap. One crate is 8 Parts, and at most one exists per server.
- **Particles:** 1 sparkle emitter per equipped bat at tier 5 and up, 4 to 6 per second. A hit burst uses 3 emitters and about 30 particles at most. The crate has 1 emitter at 8 per second and 1 Beam. Mobile caps emitters at 100 per second anyway.
- **Lights and Highlights:** none. This keeps the shared Highlight budget of 24 for merge glow.
- **Per-frame work:** BatClient has no RenderStepped loop except during a shake or hit stop (under 0.3 s). BatHud polls the safe zone 5 times a second only while equipped. That is 12 plot tests and up to about 20 box tests: microseconds. The server has no per-frame loop. The crate loop runs at 5 Hz only while a crate is carried.
- **Server cost per swing:** a handful of comparisons and one raycast.
- **Network:**
  - `BatSwing`: at most 1.4 messages a second per player, about 16 bytes each.
  - `BatFX`: unreliable, under 60 bytes, only to players within 150 studs.
  - `BatKnock`: reliable, to one victim, at most once per 1.2 s.
  - Worst case, 12 players batting in one spot: 12 x 1.4 x 11 receivers x 60 bytes is about 11 KB/s out of the server, about 1 KB/s per client. That is under 2% of the 50 KB/s per-client budget.
- **Physics:** pushed characters stay client-owned. At most one LinearVelocity per victim at a time, removed after 0.12 s by Debris.
- **Animations:** 2 tracks per character, loaded once.
- **DataStore:** one small integer. A write happens only when `BatSeen` changes, at most 9 times in a player's life, through `Store.Touch`, never a direct write.
- **Memory:** the effects reuse textures already loaded from the packs. The new images are 512 px or smaller, as the performance page recommends.

## Test plan

Setup for every test: Studio, Test tab, **Clients and Servers**, Local Server with 2 players (Player1 attacks, Player2 is the target). In the server's command bar, first run `workspace:SetAttribute("StudioNoSave", true)`. Set tiers with `game.ServerStorage.RoyalDev:Invoke("batRebirths", userId, n)`, or the rebirth test buttons from [[Rarity Gating and Rebirth]]. Count results with `RoyalDev:Invoke("batStats")`.

| Id | Rule | Steps | Pass when |
|---|---|---|---|
| T1 | B1-B3 | Set rebirths 0 to 8 and then 12. | `BatTier` reads 1 to 9 and then 9. The band colour equals `Config.RarityColors` for each tier. |
| T2 | B4-B6 | Pick up a fruit with the bat equipped, then press 1 while carrying. | The bat unequips. Pressing 1 shows `BatHandsFull` once, and not again within 5 s. |
| T3 | B9 | Spam the mouse for 10 s at tier 1. | Accepted swings are 10 to 11, never closer than 0.85 s. |
| T4 | B12h-k | Hit from 12 studs, from behind a castle wall, and from 8 studs below. | All refused, with the reasons logged in `batStats`. |
| T5 | B15-B17 | On flat plaza ground, hit Player2 at tiers 1, 3, 6 and 9. Measure the root's displacement. | Each tier pushes further than the one below. Tier 9 travels at least 3.5 times tier 1. Values are within 35% of the table. If not, tune `Power` only and update the table. Player2 gets up and walks within Stun + 0.5 s. |
| T6 | B18 | Add a third client. Player1 and Player3 hit Player2 within 1 s. | The second hit is refused (immune). |
| T7 | B19 | Player1 hits Player2 5 times in 20 s, waiting out immunity. | The 5th hit is refused with the Guard flash. |
| T8 | B26 | Rejoin Player2, then hit at 10 s. Then have Player2 swing at 12 s, and hit them at 13 s. Respawn Player2 and hit at 3 s. | Hit at 10 s refused; hit at 13 s lands; hit at 3 s after respawn refused. |
| T9 | B21 | Give Player2 Aether. Stand at the Cloud Garden edge and knock them off. | Player2 lands alive, no `FellHard` attribute, `AetherEndsAt` still set. |
| T10 | B21 | Repeat T9, but have Player2 drink nothing and arrive by Royal Passage teleport. | No thin-air death during the 10 s grace. |
| T11 | B16, B5, B12m | Player2 sits on a plain Seat and gets hit. Player1 sits in a VehicleSeat and tries to swing. Player2 sits in a boat's passenger seat and gets hit. | Plain seat: Player2 is unseated and pushed. Player1's swing is refused. Boat seat: the hit is refused with the Guard flash. |
| T12 | B28 | `RoyalDev:Invoke("crown", Player2.UserId)`. Player2 sits on the throne. Hit them. Then they stand on the dais. Hit again. | First hit refused. Second hit lands. |
| T13 | B7, B25 | Player1 starts a practice bout at the Sparring Post. Player2 stands in the stands and swings at Player1. | The bat leaves Player1's Backpack during the bout and comes back within 1 s after. Player2's hits are refused. |
| T14 | B27 | Player2 drinks the Invisibility Potion. | Their bat unequips. They cannot equip. Player1 cannot hit them. |
| T15 | B30, save | With the bat equipped, set rebirths +1. Then run the dev panel's Reload test. | Colour and stats change on the same frame. The toast shows once. The "!" clears on the next equip. After Reload, no new toast (`BatSeen` round-trips). |
| T16 | B20 | Player2 walks off their plot carrying a fruit, then carrying TNT, and gets hit each time. | The fruit and TNT stay in hand. Plot fruit counts, shards, cash and effects are unchanged. |
| T17 | B22-B23 | Hit Player2 while they stand on any plot, on a SpawnLocation box, on the beanstalk and on the arrival pad. Then have Player1 swing from a plot at Player2 just outside it. | All refused. `BatSafeZone` toast at most once per 60 s. |
| T18 | C1-C12 | `RoyalDev:Invoke("crate")`. Lift it, measure walk speed, get hit, try to re-lift within 2 s, claim it on your plot, check shards. Then: let one sit 150 s; carry one for 90 s; pivot a carrier home through the command bar. | Speed 17 (20 x 0.85). Drop on hit. Re-lift refused for 2 s. Shards rise by exactly `20 x ShardDrop(stage)`. Ground timeout fades it. 90 s carry drops it. The teleport claim is refused and the crate returns. |
| T19 | UI | Device emulator at 1600 x 900, 1280 x 720, a tablet 1024 x 768 and a phone 812 x 375. Carry a fruit, drink 4 potions, equip and unequip, open each full-screen panel. | No element in the overlap tables touches another. BatInfo hides with panels open and on touch. |
| T20 | Mobile | Phone emulation. | Tap the slot equips. The Swing button sits by the jump button, shows the cooldown sweep and dims in safe zones. Camera drags never swing. |
| T21 | Audio | One **live** server visit after publishing. | Equip, Swoosh, Hit, Hit2, Ouch and Ouch2 can all be heard. Replace any silent one. |
| T22 | Performance | During T3 with 3 clients, open the Stats panel (Shift+F3). | Added network receive under 5 KB/s per client. No frame spikes in the MicroProfiler from BatClient. |

## Dependencies and conflicts

- **[[Rarity Gating and Rebirth]]**: the bat tier is the stage, `min(1 + Rebirths, 9)`. It must stay the same function that system uses for the newest rarity. Ask: the Rebirth panel shows the next bat as a reward line ("NEXT BAT: EPIC  POWER 56  SPEED 69") using the bar art from UI 2. If that system grandfathers old saves with more than 8 rebirths, the bat clamps at tier 9. The crate reward reads its frontier shard drop (D-010), so both must read one Config table.
- **[[Royal Combat]]**: the arena is sword-only. Bats skip `InArena` players and leave the Backpack during bouts. Ask: publish the bat swings in the same animation session (one owner, one permission grant). If that system turns on Server Authority for the whole place, re-test bat knockback, which assumes the victim client applies its own velocity. If it renames or reshapes the `knockback` RoyalEvent, bats are unaffected, because they use their own `BatKnock`.
- **[[Weapons and Artillery]]**: any weapon or blast that pushes players calls `G.knockPlayer(victim, vel, { kind = "blast" })`. That gives it the same safe zones, shields and fall grace, with immunity kept separate per kind (bat 1.2 s, blast 1.5 s). Bats never damage boats or fruit: breaking boats is artillery's job, as Samarth's line says. Bats need no War Flag; only weapons do (a bat is a short shove you can walk away from). Resolved in the [[Integration Review]]: weapons use the Weapon Bar with keys 2 to 5 (key 1 stays the bat), drawing a weapon holsters the bat and the reverse (B5, B6), and on touch the Fire button uses this note's action slot (UI 3). The old reserved slot at `UDim2(0.5, 424, 1, -21)` is no longer needed.
- **[[Explosives and Fruit Health]]**: bats never touch fruit health. If TNT ever pushes players (today its Explosion has zero blast pressure, `ServerScriptService.GardenService.Tools.lua:292-298`), it should go through `G.knockPlayer`. No other overlap.
- **[[Underground and Ores]]**: the three mine floors reach 216 studs below the soil, so the safe band is now 240 studs down (B22a), and Underground also tags each floor's interior box `BatSafeZone` (B22b), because a cave is wider than a level-0 soil. The pickaxe shares the one hand: starting to mine unequips the bat, and equipping the bat stops mining.
- **[[Pets]]**: bats never hit pets (B29). Pets are drawn on the client, not physical, so nothing can hit them anyway. Roblox bans animal abuse at every rating ([[Roblox Platform Constraints for the Expansion]]).
- **[[Boats and Swimming]]**: resolved in the [[Integration Review]] (C33): players sitting in a boat seat cannot be hit by a bat (B12m), so nobody can bat a boater into the sea without the War Flag. Players standing on a deck can be hit. Drivers in a `VehicleSeat` cannot swing. Bats never damage boats. Every dock's 40-stud zone and the Boat Shop are tagged `BatSafeZone`. Note for that system: it also removes the "fast means water" test (`ServerScriptService.GardenService.Ambient.lua:363-364`).
- **[[Potions]]**: Invisibility blocks bats both ways (B27); the Potions note now says the same (its rule 27, decision D-030). Speed makes chasing easier, which immunity and the pair cap bound. Feather Falling and knock grace can both be on at once with no conflict. Aether keeps running after a knock-off because nobody dies. Any future potion that changes knockback hooks in through `G.knockScale(victim)`, not by editing Bats.
- **[[Plot Overhaul and Environment]]**: the plot safe zone follows the soil part, which is the largest part by footprint (`ServerScriptService.GardenService.PlotRegistry.lua:21-30`). A new base part bigger than the soil moves the safe zone with it. Ask: place the 8 `CrateSpot` markers, keep the plaza open and walkable, and tag shop stalls and decor spawns with `BatSafeZone`.
- **[[King UI Overhaul]]**: all bat UI is kit art with no emoji, so it does not add to the King UI debt. The seated King is immune (B28). If the throne room is rebuilt, keep the `RoyalSeat` attribute and the `CourtyardArrival` marker, which the safe zone uses.

## Open questions for Samarth

Each has a default, so building can start now.

1. **Are all plots safe, or only your own?** Default: **all plots**. Being hit on someone else's plot feels like griefing, and plots are where people merge.
2. **Golden Crate in the first release?** Default: **yes, as a second step right after the bat core**. It gives the bat a job on the main island, not only on edges.
3. **Crate size: 20 frontier merges' worth?** Default: **20** (3 to 8 minutes of merging).
4. **Can a player pick an older tier's colour?** The UI kit note says "one bat per unlocked tier". Default: **no, the bat is always the newest tier**, which matches "this will match with the color of the bat that you unlock".
5. **The "Ouch" voice on hits?** Default: **on, at 0.45 volume, at most once per 2 s per player**.
6. **Tumble (Physics state) or a plain push?** Default: **tumble**. It makes the push carry and reads like the classic slap.
7. **BatSlot at the bottom centre-right, or in the right column under Clear?** The UI kit lists "the right edge between Clear and EffectStack" as free, but the EffectStack grows into that space with 3 or more potions. Default: **bottom centre-right**, next to where the carry buttons are, since both are "what's in your hands".
8. **Tint one white bat image, or 9 coloured images?** Default: **tint one image**. Switch to 9 if the tinted look is flat.
9. **Robux bat skins later?** Default: **not in v1**. If added later, cosmetic only, never Power or Speed ([[Steal a Pet - Bats and Rebirth Knockback]]).
10. **Log these as decisions?** Logged by the [[Integration Review]] as D-013 (bat tier = stage, free, from rebirths), D-014 (every plot, mine floor, dock zone and shop stall is a bat safe zone; boat seats cannot be hit), D-015 (bat and weapon knocks never kill) and D-016 (the Golden Crate pays `ShardReward(20, R)`, and it is the only crate system in v1). D-012 was already taken by the lily pad decision. This note does not contradict D-001 to D-011. It uses D-009 (rebirth keeps the bat) and D-010 (frontier shard drop) as given.
