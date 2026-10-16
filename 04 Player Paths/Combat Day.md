---
tags: [player-path]
date: 2026-10-04
---
# Combat Day

Kai is 13 and plays on a PC. He is in stage 6 (Mythic, 5 rebirths) and his plot has just filled with Dragon Peppers: 62 minutes of waiting before his rebirth cash is ready (see P2-1 in [[Mid Game - Stage 4 Epic]]). He spends the wait fighting: a Golden Crate fight with bats, a boat battle with artillery, a TNT attack on his garden and his defence, and a Crown Duel with a new blade. His friend Mia (stage 4) rides with him.

Every rule cited is in the spec notes in [[02 Systems]]. Each friction is ranked in [[00 Paths Summary]].

**Words used here**
- **Tier**: a bat's level, equal to the player's stage. A higher tier pushes further ([[Bats]] Numbers).
- **War Flag**: a per-player switch. Only flagged players can push each other with sea weapons, and a boat can be damaged only when its driver is flagged ([[Weapons and Artillery]] rules 21 to 30, D-024).
- **Effective stage**: an explosive's damage uses the lower of the attacker's and the owner's stage.
- **Arena focus**: in the Royal Arena the whole garden HUD hides except Mute (D-028).
- **One label per channel**: the green and red toasts each have one label; a new line replaces the one on screen (`ReplicatedStorage.GardenToast.lua`).

**How the times were made.** Times are minutes of the session (m:ss), starting when his plot fills. Push distances, damage and prices are the specs' own numbers at each player's stage.

**Starting state**: stage 6. 48K shards. Boat. Bat tier 6 (red, Power 72, Speed 75). Weapons: 8 Cherry Bombs, 6 Melon Mortar shells, 6 Coconut Cannon shots. Blades: Royal Sword, Rarity Blade (red), Knight's Longsword (3 practice wins), Statue Greatsword (bought at stage 3 for 2,300). Plot: 35 Dragon Peppers, nothing to merge.

---

### 0:00 The plot is full
- **Sees:** Merge All: "Nothing can merge. Dragon Pepper is your best fruit until you rebirth." The Rebirth menu's cash row is about 11% of the way.
- **Decides:** go and fight.
- **Rule:** [[Rarity Gating and Rebirth]] rule 6; `plateau_wait.py` (62 minutes at stage 6).

### 2:00 Golden Crate
- **Sees:** "A Golden Crate is falling on the plaza". A gold beam marks the spot.
- **Decides:** runs for it.
- **Rule:** [[Bats]] C1 to C3.

### 2:20 The crate fight
- **Sees:** he holds E for half a second and lifts the crate. His bat goes away by itself. He walks 15% slower. Rex (stage 8, pink bat, Power 88) catches him on the road and swings: Kai flies about 44 studs and tumbles for 0.65 s. The crate drops at his feet and he cannot lift it for 2 seconds. Rex lifts it.
- **Decides:** presses 1 for his bat and chases. One swing: Rex flies about 33 studs and drops the crate. Kai lifts it again and makes it to his plot.
- **Rule:** [[Bats]] B6, B12, B15 to B19, C4 to C7, Numbers.
- PAYOFF: "Golden Crate: +3,100 Shards" (20 frontier merges at his stage) and, to everyone else, "Kai brought the Golden Crate home". A back-and-forth that nobody can lose anything in.

### 6:00 Out to sea with Mia
- **Sees:** at the main dock he presses "Summon Boat". Mia, his Roblox friend, sees "Board" (F) on the passenger seat and hops in. He presses G. "War Flag up in 5 s", then a red flag over his head and a red pennant on the mast: "War Flag is up. Flagged players can hit you and your boat." The Weapon Bar shows the flag slot, Cherry Bomb 8, Mortar 6, Cannon 6, with keys 2 to 4.
- **Rule:** [[Boats and Swimming]] R27 to R32; [[Weapons and Artillery]] rules 26 to 30, UI 2.
- FRICTION: **P4-1 (medium)** Mia never raised a flag, but she rides a flagged boat. If it sinks she is dumped in the sea too, and nothing told her before she boarded. **Fix:** while the driver is flagged, the Board prompt's ObjectText reads "Kai (War Flag up)", and the passenger seat shows the red pennant before boarding. (spec: [[Boats and Swimming]] R32, 6.4)

### 8:00 Incoming mortar
- **Sees:** Rex, flagged, in his own boat 120 studs away. A red ring appears on the water ahead of Kai, with a disc filling it over 2 seconds.
- **Decides:** turns hard left.
- **Rule:** [[Weapons and Artillery]] rule 34, UI 3 (warning ring), Numbers (Mortar 35%).
- PAYOFF: the shell splashes where he was. Big artillery is telegraphed, so steering beats it.

### 8:30 Kai sinks Rex
- **Sees:** he presses 4 for the Coconut Cannon. A reticle appears. He clicks three times (one clip), waits 3.5 s for the reload, then two more. Five direct hits at 20% each. Rex's sail darkens, then his stern smokes. At 0 HP his boat tilts and sinks.
- **Rule:** [[Weapons and Artillery]] rules 19, 22, 29, 33, Numbers (Cannon: 20% direct, clip of 3); [[Boats and Swimming]] R37, R47, R48.
- PAYOFF: "You sank Rex's boat." Rex and his passenger drop into the sea with full stamina; Rex's flag drops at once so he is not hunted on the way home. Nobody loses anything but time.

### 12:00 Starfruit Barrage
- **Sees:** a siren. A dark disc grows on the water around his boat. Zed (stage 8, flagged) has called a Starfruit Barrage. Three seconds later five shells fall. Kai steers out of the centre; two shells still hit. "Boat" bar: 40%, gold. His top speed drops to about 33 studs/s.
- **Decides:** runs for the nearest dock.
- **Rule:** [[Weapons and Artillery]] rules 34, 35, Numbers (30% per shell); [[Boats and Swimming]] R37, R44, R46.
- PAYOFF: inside 40 studs of a pad the bar reads "Boat (safe)": no damage, and it regains 100 HP a second. A real escape.

### 14:00 TNT on his garden
- **Sees:** red warning: "Zed placed a Mega TNT in your garden! Lock to block it." A red "!" on his Lock pill. The fuse is 4 seconds. Kai is at the dock, about 250 studs from his plot.
- **Decides:** taps Lock.
- **Rule:** [[Explosives and Fruit Health]] rules 13, 14, 32.
- **As the specs were written:** refused with `LockOffPlot`. Live `lockPlot` accepts only a player standing on their own plot (`RebirthLock.lua:223`). The Mega TNT hit 7 of his fruit: 214 at the centre against 305-HP Dragon Peppers, no stun.
- FRICTION: **P4-2 (high)** The defence the toast names cannot be used by anyone away from home. Spec error, fixed in [[Explosives and Fruit Health]] rule 32 and the code plan (D-034): while an explosive is armed on your plot, the Lock can be raised from anywhere.
- **With the fix:** the dome rises over his plot. "Your Lock blocked Zed's Mega TNT!" The Mega TNT goes back to Zed's bag.
- PAYOFF: defending from across the sea, with 4 seconds to spare.
- FRICTION: **P4-3 (medium)** The red channel has one label. Any other red line in those 4 seconds (a boat or flag message, a refusal) replaces the TNT warning before he reads it. The "!" on the Lock pill stays, which helps, but the words are gone. **Fix:** `ExplosiveIncoming` and `NukeIncoming` are sticky: GardenToast holds them until the blast or the cancel, and other red lines wait in a queue of at most 2 behind them. (spec: [[Explosives and Fruit Health]] 6.5; GardenToast in [[Client UI - HUD and Menus]])

### 15:30 Zed tries again
- **Sees:** Zed waits out the 60-second dome, then places a Mega TNT during the 30-second recharge, and another 15 seconds later.
- **Rule:** [[Explosives and Fruit Health]] rules 13, 19, 26, N2, N6.
- **As the specs were written:** the spec says 2 centre hits stun a same-stage top fruit. But regen started 10 s after a hit, and Mega TNT's cooldown is 15 s, so for 5 s between blasts each fruit regained 10% a second: 305, then 91 after blast 1, then 244 after 5 s of regen, then 30 after blast 2. No stun. Zed would need a third Mega TNT (16,000 shards each).
- FRICTION: **P4-4 (medium)** The spec's own N2 number was wrong for Mega TNT. Spec error, fixed in [[Explosives and Fruit Health]] rule 26, N6 and the config (D-036): regen now starts 16 s after the last hit, longer than every explosive's cooldown.
- **With the fix:** the second blast stuns the 3 fruit nearest the centre for 20 seconds. "Zed's Mega TNT hit 7 of your fruit. 3 stunned." They turn see-through with stars, then come back full with a cyan 30-second shield.
- PAYOFF: Kai loses 20 seconds of income from 3 of his 35 fruit and keeps every fruit. Zed spent 32,000 shards and gained nothing. Stealing stays out of the game.

### 20:00 The challenge
- **Sees:** the royal chip shows a red "!" (he is eligible). He taps it. The Kingdom panel's Throne tab shows King Ana. He presses Challenge. Green line: "You are #1 in line".
- **Decides:** drives Mia back toward the dock while he waits.
- **Rule:** [[King UI Overhaul]] rules 26 to 30.

### 21:00 Pulled into the arena mid-sea
- **Sees:** Ana accepts. Kai is teleported off the driver seat onto his arena pad.
- **Rule:** [[Royal Combat]] Player view 1; Crown teleports stand a seated player up (`Crown.lua:370`).
- FRICTION: **P4-5 (medium)** Mia is left alone in a boat only Kai can drive. As written, a boat despawns only when nobody is aboard for 90 s, so with Mia aboard it never moves and never despawns. Spec gap, fixed in [[Boats and Swimming]] R50: when the owner dies or is teleported out of the driver seat with a passenger aboard, the boat despawns 10 s later and R51 puts the passenger on the nearest dock.
- FRICTION: **P4-6 (high)** While Kai fights, arena focus hides his Lock button (D-028), so his garden had no defence and an attacker could time explosives to his duel. Spec gap, fixed in [[Explosives and Fruit Health]] rules 9 and 15 (D-035): a garden whose owner is in the arena cannot be targeted, and an armed explosive is returned if the owner enters the arena before the blast. The attacker reads "That player is in a royal fight. Try again later."

### 21:05 Pick phase
- **Sees:** every garden button is gone except Mute. The Armory opens in the centre, "FIGHT IN 10", with 9 blade cards. Owned ones are bright. The Celestial Saber (Astral frame) shows "31,000" with the shard crystal, but no button works in pick mode.
- **Decides:** picks the Statue Greatsword. It appears in his hand on the pad at once. READY.
- **Rule:** [[Royal Combat]] rules 38 to 43, 51, Armory panel.
- PAYOFF: choosing a blade right before the fight, as the brain dump asked, and seeing it in hand immediately.
- FRICTION: **P4-7 (low)** The Saber shows a price he could pay (he has 51K) in a panel where buying is blocked. **Fix:** in pick mode, an unowned shard blade shows "Buy at the Royal Armory" instead of its price. (spec: [[Royal Combat]] Armory states)

### 21:20 The duel
- **Sees:** the title card "CROWN CHALLENGE", a short flourish, 3, 2, 1, FIGHT. One card at top centre: "Crown Duel", "Kai 0 - 0 Ana", "First to 3", and two health bars. A row of plates at the bottom: "LMB Strike", "Hold Heavy", "F Block", "Q Dash", "E Slam".
- **Decides:** taps for a combo: across, back, rising, then a wide finisher. Each landed hit freezes his swing for a blink, draws an ember slash arc and floats a white number. Ana's bar drops, leaving a red chunk that drains half a second later.
- **Rule:** [[Royal Combat]] rules 1 to 17, 30 to 36, Numbers (combo 7, 7, 8, 12).
- PAYOFF: a sword fight that reads: telegraphed hits, hit stop, real slashes, a health bar.

### 21:50 Parry and slam
- **Sees:** Ana's third swing; Kai presses F just before it lands. Clang, a white glint, "PARRY". She is stunned for 0.8 s and his next hit floats a gold 11. At 40 HP she is close to the edge; he presses E: he leaps, a dust ring rises at 11 studs as a warning, he plunges, the stone cracks, and she flies about 50 studs off the ring.
- **Rule:** [[Royal Combat]] rules 5, 20, 21, 24 to 28; Numbers (finisher throw by HP).
- PAYOFF: "CROWN POINT". Low HP throws further, so a hurt fighter near the edge is in real danger.

### 23:30 Victory
- **Sees:** 3 - 1. "VICTORY". Every other player sees one herald line for 3 seconds: "Kai took the crown". Green line: "+3,875 shards" (25 units at his stage). The garden HUD returns.
- **Rule:** [[Royal Combat]] rules 48, 49, Numbers; [[King UI Overhaul]] rules 16, 18, 19.
- PAYOFF: the crown, and a reward sized to his stage instead of the old flat 1,000.

### 24:00 King Kai
- **Sees:** his chip reads "You Are King". A red "!" for a ready decree. The Decrees tab offers five decree cards with green Issue pills.
- **Decides:** issues Crown Hunt.
- **Rule:** [[King UI Overhaul]] rules 25, 26, 29; [[Royal Combat]] rule 48 (decree units).

### 24:30 Mia
- **Sees:** Mia is on the island dock. Ten seconds after Kai vanished, the boat despawned and she was placed on the nearest dock exit (the R50 fix, then R51). Kai can summon the boat again at any dock.
- PAYOFF: nobody was stranded.

### 40:00 Still waiting
- **Sees:** his plot is still full of Dragon Peppers. The rebirth cash is 22 minutes away. He has 55K shards.
- **Decides:** a second duel against a challenger (Defense pays 1,550), then the forest.
- FRICTION: the plateau wait again (P2-1). Combat filled 40 minutes of it well, but only because three other players were online and willing.

---

## What this path found

| Id | Severity | Friction | Spec |
|---|---|---|---|
| P4-2 | high | Lock refused away from your plot (spec error, fixed, D-034) | [[Explosives and Fruit Health]] |
| P4-6 | high | Fighters' gardens had no defence during duels (spec gap, fixed, D-035) | [[Explosives and Fruit Health]] |
| P4-1 | medium | Passengers ride flagged boats without being told | [[Boats and Swimming]] |
| P4-3 | medium | The TNT warning can be overwritten by any other red line | [[Explosives and Fruit Health]], GardenToast |
| P4-4 | medium | Regen healed Mega TNT targets between blasts (spec error, fixed, D-036) | [[Explosives and Fruit Health]] |
| P4-5 | medium | Passenger stranded when the driver is pulled into a duel (spec gap, fixed) | [[Boats and Swimming]] |
| P4-7 | low | Pick phase shows prices it will not take | [[Royal Combat]] |

Spec errors fixed by this path: [[Explosives and Fruit Health]] rules 9, 15, 26, 32, N6 (D-034, D-035, D-036); [[Boats and Swimming]] R50.

Payoffs: the crate back-and-forth, telegraphed mortars, sinking a boat with the Cannon, the dock safe zone, defending from across the sea, stuns that cost seconds and never fruit, the blade pick, a readable sword fight with parry and slam, and stage-sized royal rewards.
