---
tags: [player-path]
date: 2026-10-04
---
# Player paths: summary

Seven players were walked through the 11 expansion specs in [[00 Systems Index|02 Systems]], minute by minute, against the [[Stage Model]] and the [[Brain Dump 2026-10-04]]. This note ranks every friction they hit, says what fixes it and which spec owns the fix, and lists the spec errors that were fixed in place.

| Path | Player | Covers |
|---|---|---|
| [[New Player - First Session]] | Ava, 11, PC, first 40 minutes | plot, first merges, Apple cap, Rebirth menu, first rebirth, first Uncommon, first forest trip |
| [[Mid Game - Stage 4 Epic]] | Maya, 14, PC, stage 4 | Luck, the plateau, boat, taming at the plateau, TNT both ways, the rebirth decision |
| [[Underground Unlock - Stage 6-7]] | Leo, 15, PC, a saver | saving 100K, buying Floor 1, mining, Ores Index, a TNT warning in the mine |
| [[Combat Day]] | Kai, 13, PC, stage 6 | crate fight with bats, boat battle, Starfruit Barrage, TNT defence, Crown Duel with a blade skin |
| [[Mobile Player]] | Noah, 10, 667 x 375 phone | HUD fit, touch targets, boat driving, firing, mining, performance |
| [[Endgame - Transcendent]] | Rin, 16, PC, stage 9 | what is left after the last rarity |
| [[Existing Player Migration]] | Samarth's save | first join after the update: rebirths 0, best Zenflare, $646Sp, 34.7K shards |

**Words used here**
- **Severity.** *Critical*: breaks the pacing for every player, for a long time. *High*: a promised action fails, a fairness hole, or a big purchase or trip wasted, for many players. *Medium*: confusion or a few minutes lost, with a workaround. *Low*: polish.
- **Plateau**: the plot is full of cap fruit. Nothing spawns, nothing merges, and Clear is refused (it will not clear a plot where every fruit is one kind, `Merging.lua:396-399`). Only the rebirth cash is left to wait for.
- **Fixed** in the Status column: a spec error, already corrected in the spec. **Proposed**: a design change that needs Samarth's yes.

## The three findings that matter most

1. **About half of a career is spent on a full plot with nothing to merge.** In every stage from 3 on, the plot fills 33 to 40 minutes in, but the rebirth cash (D-011) takes 57 to 145 minutes. Measured over stages 1 to 8: 6.3 of 10.6 hours (59%) for the attentive player, 5.3 of 11.8 hours (45%) for a 6-second merger. Offline pay is 3%, and Roblox disconnects a player after 20 minutes with no input (platform behaviour, not in the vault research). Pets, ores, combat, crates and the crown are what fill this time, which is why several other frictions sit inside the plateau. Re-run: `cd ~/fruit-empire/sim && python3 plateau_wait.py` (new, read-only on the existing scripts).
2. **At stage 9 the plot freezes for good**, about 33 minutes after Transcendent unlocks: no spawns, no merges, no shards from merging, no rebirth, and cash with nothing to buy. Every migrated veteran meets this on day one.
3. **The explosive defence did not work away from home.** The warning says "Lock to block it", but the live Lock only works while you stand on your own plot, and the arena hides it. Fixed in the spec (D-034, D-035), along with a regen timer that let Mega TNT targets heal between blasts (D-036).

## Every friction, ranked

| Rank | Id | Severity | Friction | Fix | Spec | Status |
|---|---|---|---|---|---|---|
| 1 | P2-1 | critical | 45 to 59% of a career is waiting on a full plot (stage 8: 109 min of 145) | Either re-solve D-011's cash needs for stages 3 to 8 so the wait after the last merge is about 15 min (career about 6 h of play; shard flows unchanged because every frontier merge happens before the plateau), or keep D-011 and pay 50% offline while the plot is full of cap fruit, capped at the rebirth cash need | [[Stage Model]], D-011, [[Rarity Gating and Rebirth]] | Proposed |
| 2 | P6-1 | critical | At stage 9 the plot freezes for good about 33 min in; placed veterans hit it on day one | At stage 9 only, Clear works on a plot of Soul Fruit: it removes 4, new fruit spawn, and merging back up pays 1,646 per Transcendent merge (about 20K shards a cycle) | [[Rarity Gating and Rebirth]] | Proposed |
| 3 | P3-4, P4-2 | high | "Lock to block it" is refused away from your plot (mine, sea, forest, sky) | The Lock can be raised from anywhere while an explosive is armed on your plot (D-034) | [[Explosives and Fruit Health]] rule 32, code plan | Fixed |
| 4 | P4-6 | high | A fighter's garden has no defence: arena focus hides the Lock | Gardens of `InArena` owners cannot be targeted; armed explosives are returned (D-035) | [[Explosives and Fruit Health]] rules 9, 15 | Fixed |
| 5 | P1-1 | high | No first-session guide; 12 HUD items; the first toast is about a bat | Four calm goal notices for new accounts until the first Apple; hold the bat toast until then | [[Rarity Gating and Rebirth]] (new section), [[Bats]] B30 | Proposed |
| 6 | P1-6 | high | The first forest trip is wasted: the wanted fruit is only shown in the forest | Name the wanted fruit in a hint on first sea entry; a "Wanted" line in the Pets panel from the `"wild"` snapshot | [[Pets]] U2, U7; [[Boats and Swimming]] R15 | Proposed |
| 7 | P2-3 | high | At the plateau animals want fruit the full plot can no longer grow | A feed accepts the requested fruit or any better unlocked fruit | [[Pets]] P13 (g), P14 | Proposed |
| 8 | P3-5 | high | Shards spent after the plateau cannot be re-earned by merging, so a late purchase can stall the rebirth ("shards never block a rebirth" fails) | When the plot is full and the shards left would be under the next rebirth's need, the buy check shows a red line and dims Yes for 2 s; the plateau fix (rank 1) helps too | [[Underground and Ores]] U3, [[Stage Model]] | Proposed |
| 9 | P3-1 | high | 100K shards buys about 3 minutes of new play (Floor 1 fills with 10 Iron) | Ship a longer Floor 1 ladder (Coal, Iron, Copper, Emerald) with boosts re-solved to keep the 7.5% cap; the Mine Board names the next goal | [[Underground and Ores]] 5.1, 5.4, U10 | Proposed |
| 10 | P5-5 | high | On a phone, a habitual jump tap leaves the boat mid-sea | On touch, jump leaves only near a pad or below 6 studs/s; otherwise hold jump 0.5 s | [[Boats and Swimming]] R34 | Proposed |
| 11 | P6-2 | high | Cash has no use once the last upgrade is bought at stage 9 | A stage 9 cash sink: a Royal Treasury (60 s of full-plot income buys 1,646 shards, 60 times an hour) or cash cosmetics | [[Stage Model]] (new) | Proposed |
| 12 | P6-5 | high | No long-term goal once the stage 9 shard sinks are bought | Put the next beats in the plan: ore band on Floor 4, then the tower upgrade, each with an Index category; then decide on a capped prestige layer | [[Underground and Ores]] 5.1, [[Rarity Gating and Rebirth]] Q3, [[Plot Overhaul and Environment]] C3 | Proposed |
| 13 | P1-7 | medium | Veterans' bats push stage 1 players 38 to 50 studs; stage 1 gardens are protected from TNT but stage 1 players are not protected from bats | Stage 1 players cannot be hit by bats of tier 3 or higher | [[Bats]] B12 | Proposed |
| 14 | P2-8 | medium | Pet income falls about 300 times after a rebirth, unexplained | Confirm dialog line: "Pet and ore income follow your best fruit, so they start small again." | [[Rarity Gating and Rebirth]] U4, [[Pets]] P30 | Proposed |
| 15 | P2-10 | medium | Only the Mine's buy check warns about the next rebirth's shard need | One shared second line for every shard purchase of 1,000+: "You will have X left. Your next rebirth needs Y." | [[Rarity Gating and Rebirth]] (helper), [[Pets]] U3, [[Boats and Swimming]] 6.3, [[Royal Combat]] Armory | Proposed |
| 16 | P5-1 | medium | Drop, Store and the Lock and Upgrade pills are 27 to 31 points tall on a phone | On touch, a 44-point minimum for the carry row and the top pills | [[Integration Review]] section 4, [[King UI Overhaul]] UIKit | Proposed |
| 17 | P5-8 | medium | On a phone the Lock, now a 3-second defence, is the smallest button | While an explosive is armed on your plot, the touch action slot shows Lock | [[Bats]] UI 3, [[Explosives and Fruit Health]] 6.4 | Proposed |
| 18 | P4-3 | medium | One label per toast channel: the TNT warning can be overwritten within its fuse | `ExplosiveIncoming` and `NukeIncoming` stay until the blast; other red lines queue behind them | [[Explosives and Fruit Health]] 6.5, GardenToast | Proposed |
| 19 | P4-4 | medium | Regen started 10 s after a hit but Mega TNT's cooldown is 15 s, so its targets healed between blasts (3 hits, not 2) | Regen delay 16 s (D-036) | [[Explosives and Fruit Health]] rule 26, N6, 9.1 | Fixed |
| 20 | P4-5 | medium | A passenger is stranded when the driver dies or is pulled into a duel | Boat despawns 10 s later; R51 moves the passenger to a dock | [[Boats and Swimming]] R50 | Fixed |
| 21 | P2-6 | medium | TNT on a higher-stage garden does almost nothing, and the attacker cannot tell before spending | On a target garden the carry bar reads "98 of 244 HP at the centre" | [[Explosives and Fruit Health]] 6.3 | Proposed |
| 22 | P1-3 | medium | The cash need is a bare number for 15 minutes | Under the cash row: "About 15 min at your income now", plus a side goal | [[Rarity Gating and Rebirth]] U4 | Proposed |
| 23 | P1-5 | medium | A stage 1 swimmer gets a War Flag and no weapons | Weapon Bar at sea only with a charge or the flag up; no flag at stage 1 | [[Weapons and Artillery]] UI 2, rule 27 | Proposed |
| 24 | P5-6 | medium | Aim, fire and steer need three thumbs on a phone | On touch, Fire fires on a tap at the reticle and the arc shows whenever a weapon is drawn | [[Weapons and Artillery]] UI 4 | Proposed |
| 25 | P5-9 | medium | Every performance budget is set and tested on a PC | Halve PetWorld's caps on touch; add a low-end phone to T17 and T-Perf | [[Pets]] Performance, [[Boats and Swimming]] 12 | Proposed |
| 26 | P3-3 | medium | Ore income is invisible above ground | A second short line on the CornerHud boost bar: "Ores +7.5%" | [[Underground and Ores]] U15 | Proposed |
| 27 | P4-1 | medium | A friend boards a flagged boat without being told | Board prompt reads "Kai (War Flag up)" | [[Boats and Swimming]] R32, 6.4 | Proposed |
| 28 | P6-6 | medium | The 150K pet slot adds no money once pets are at the limit | The buy check says so when the limit is reached | [[Pets]] U3 | Proposed |
| 29 | P7-3 | medium | Placed veterans see shard prices jump 8 to 80 times overnight | A one-time Shop line for migrated players; optionally a one-time shard top-up (saved shards x 8.2 at stage 9) | [[Rarity Gating and Rebirth]] rule 22, [[Potions]] | Proposed |
| 30 | P1-8 | medium | The rebirth notice and the bat notice fired in the same frame; one was lost | Bat notice 3 s after `RebirthDone` | [[Bats]] B30 | Fixed |
| 31 | P7-1 | medium | Migration and refund notices fired together; the Spawn Rarity refund was silent | Migration notice at refund delay + 4 s with the refund amount; bat notice at + 8 s | [[Rarity Gating and Rebirth]] rule 22, U10; [[Bats]] B30 | Fixed |
| 32 | P1-11 | medium | Animal requests changed only with the best tier, so they went stale (and contradicted test T4) | Recompute unfed animals whenever the best fruit changes | [[Pets]] P9 | Fixed |
| 33 | P6-3 | medium | Stage 9 texts promised a rebirth (Max Fruit card, Merge All, Spawn Rarity) | Stage 9 variants of each | [[Rarity Gating and Rebirth]] U5, U7, U10 | Fixed |
| 34 | P6-4 | medium | At stage 9 Luck was still sold and refused with "after your next rebirth" | "Maxed for good", no Buy | [[Potions]] rule 26, UI 1; [[Rarity Gating and Rebirth]] rule 10 | Fixed |
| 35 | P3-2 | medium | "Merge ores to make room" shown when all ores are Iron, which cannot merge | Separate full-of-Iron message | [[Underground and Ores]] rule 12, U14 | Fixed |
| 36 | P5-4 | medium | The touch Bat button showed while driving, where the bat is always refused | Hidden while seated in a `VehicleSeat` with no weapon drawn | [[Bats]] UI 3 | Fixed |
| 37 | P2-2 | low | Luck keeps counting after Spawn Rarity reaches the ceiling | Pause Luck while at the ceiling; resume after the next rebirth | [[Potions]] rule 26 | Proposed |
| 38 | P2-5 | low | Bats cancel feeds in private forest clearings | Tag each clearing's inner circle `BatSafeZone` | [[Bats]] B22 | Proposed |
| 39 | P2-9 | low | The next stage's shard drop is never shown | Rebirth menu footer: "Legendary merges pay 70 shards" | [[Rarity Gating and Rebirth]] U4 | Proposed |
| 40 | P1-9 | low | After rebirth 1, Common merges stop paying shards without a word | `RebirthDone` adds "Uncommon merges now pay 7 shards." | [[Rarity Gating and Rebirth]] U10 | Proposed |
| 41 | P1-4 | low | Plot Size reads like more fruit; level 2 costs 35% of the first rebirth | Sign line: "More room to walk. Fruit slots come from Capacity." | [[Plot Overhaul and Environment]] UI 1 | Proposed |
| 42 | P1-2 | low | LOCKED and "Rebirth to raise" at minute 2 | Stage 1 price row: "Max for Common. Rebirth to raise." | [[Rarity Gating and Rebirth]] U7 | Proposed |
| 43 | P1-10 | low | The royal chip's "!" lights at minute 3 and invites a duel | Show the badge only after the first rebirth | [[King UI Overhaul]] rule 26 | Proposed |
| 44 | P2-7 | low | Placing TNT drops your own Lock silently | `ExplosiveArmed` adds "Your Lock is down." | [[Explosives and Fruit Health]] rule 12, 6.5 | Proposed |
| 45 | P2-11 | low | Sky crates are announced to players who cannot reach the sky | Drop and announce sky crates only to players who can reach them | [[Bats]] C2, C3 | Proposed |
| 46 | P3-6 | low | The Ores Index category is hidden until Floor 1 | Show the Ores tab from the start with sealed "Buy the Mine" cards | [[Underground and Ores]] U4 | Proposed |
| 47 | P4-7 | low | The pick phase shows blade prices it will not take | "Buy at the Royal Armory" in pick mode | [[Royal Combat]] Armory | Proposed |
| 48 | P5-2 | low | The HUD table's right column rects overlap (7 points on a phone) | Measure; trim hit areas or space centres 102 apart; add to `UIOverlapCheck` | [[Integration Review]] section 4 | Proposed |
| 49 | P5-7 | low | Node and ore prompts stack on a small screen | On touch, hide ore prompts while mining; show the nearest ore prompt only | [[Underground and Ores]] U13 | Proposed |
| 50 | P5-10 | low | The Index category tabs are likely to touch the left column on a phone | On touch, use the spec's own fallback row from the start | [[Underground and Ores]] U4 | Proposed |
| 51 | P5-3 | low | The cash corner's invite "+" sits under the thumbstick (live today) | Move or hide it on touch, after measuring | CornerHud | Proposed |
| 52 | P2-4 | low | "Tamed a %s %s!" printed "a Epic" | "New pet: %s %s!" | [[Pets]] U7 | Fixed |
| 53 | P7-2 | low | Samarth's own save has no rebirth journey left | Play the journey in Studio with `StudioNoSave` and `DevSetRebirths 0` | [[Rarity Gating and Rebirth]] Q1, Q8 | Accepted |

## Spec errors fixed in place

Each edited spot names the path that found it. The list is also in [[Integration Review]] section 9, and the three rule changes are logged as D-034 to D-036 in the [[Decisions Log]].

| Spec | What changed | Path |
|---|---|---|
| [[Explosives and Fruit Health]] | Lock from anywhere while an explosive is armed on your plot (rule 32, code plan row for `lockPlot`, edge cases); gardens of `InArena` owners cannot be targeted (rules 9 and 15, new toast `ExplosiveOwnerFighting`, cancel reason `"fighting"`); regen delay 16 s (rule 26, N6, `RegenDelay`, the server loop, test 26); player view step 10 | [[Underground Unlock - Stage 6-7]], [[Combat Day]] |
| [[Bats]] | Touch action slot hidden while driving (UI 3); `BatUnlocked` timing after `RebirthDone` and after the join notices (B30) | [[Mobile Player]], [[New Player - First Session]], [[Existing Player Migration]] |
| [[Boats and Swimming]] | Boat despawns 10 s after the owner dies or is teleported out with a passenger aboard (R50, edge cases) | [[Combat Day]] |
| [[Pets]] | Requests recompute whenever the best fruit changes (P9); "New pet: %s %s!" (U7, player view) | [[New Player - First Session]], [[Mid Game - Stage 4 Epic]] |
| [[Potions]] | Player view lists 7 cards (Haste waits for the second ore band); mobile edge case places the effect block top left; stage 9 Luck (rule 26, UI 1) | [[New Player - First Session]], [[Mobile Player]], [[Endgame - Transcendent]] |
| [[Rarity Gating and Rebirth]] | Stage 9 variants of the Max Fruit card (U5), the Spawn Rarity card (U7), Merge All and Spawn Rarity toasts (U10), the Luck refusal (rule 10); migration notice timing and refund text (rule 22, U10, code plan) | [[Endgame - Transcendent]], [[Existing Player Migration]] |
| [[Underground and Ores]] | Full-of-Iron message (rule 12, player view, U14) | [[Underground Unlock - Stage 6-7]] |
| [[Stage Model]] | "Shards are never the thing that blocks a rebirth" now says it holds only for a player who never spends below the next need; new section with the measured plateau wait per stage | [[Underground Unlock - Stage 6-7]], [[Mid Game - Stage 4 Epic]] |
| [[Integration Review]] | Brain dump row 30 now points here; new section 9 lists these fixes | all |

## Decisions Samarth needs to make

Ranked by impact. Each is a "Proposed" row above.

1. **The plateau (P2-1).** Shorter late stages (re-solve D-011, career about 6 hours of play) or higher offline pay while the plot is full. This one choice shapes how every session after stage 3 feels.
2. **The stage 9 endgame (P6-1, P6-2, P6-5).** An endgame loop on the plot, a use for cash, and the next content beats.
3. **The mine's first floor (P3-1).** 100K shards for 3 minutes of play, or a longer ladder at launch.
4. **New-player protection (P1-1, P1-7).** A four-step goal line, and no bat hits on stage 1 players from tier 3 and up.
5. **Taming at the plateau (P2-3, P1-6).** Better fruit accepted as feed, and the wanted fruit shown before the trip.
6. **Phone controls (P5-5, P5-1, P5-6, P5-8).** Boat exit, 44-point buttons, tap-to-fire, Lock in the action slot.
7. **Veteran price shock (P7-3).** A one-time line, or a one-time shard top-up.

## What should feel great (payoffs worth protecting)

- The Max Fruit card and the Rebirth menu: the cap, every need and every reward on one calm screen ([[Rarity Gating and Rebirth]] U4, U5).
- The first Uncommon about 3 minutes after the first rebirth, and a new fruit every minute after it.
- Sealed Index cards and new Weapons cards on every rebirth: "rebirth to unlock more" is always visible.
- Failing at sea costs time, never fruit or potions; explosives stun, never delete; sinking a boat takes nothing. Stealing stays out of the game.
- The boat turning a 65-second swim into 14 seconds.
- The locked hut on every lot from day one, and the cave when it opens.
- Pets that pay in plain words ("Pays like 0.38 of a Watermelon") and push the start of every stage.
- Telegraphed artillery, dock safe zones, a sword duel with hit stop, parry and a slam, and blade picks right before the fight.
- Stage-scaled rewards that make the crown worth fighting for at stage 9.
