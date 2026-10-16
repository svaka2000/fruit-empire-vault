---
tags: [system, review]
date: 2026-10-04
---
# Integration review: the 11 expansion systems together

The 11 system notes in this folder were written one at a time. This review reads them side by side, finds where they disagree or collide, and fixes the notes so they agree. The economy numbers behind it are in [[Integration Check]]. New decisions are D-013 to D-033 in the [[Decisions Log]].

**Words used here**
- **Spec**: one system note in this folder.
- **Conflict**: two specs saying different things about the same rule, number, name or screen space.
- **Stage**: the newest rarity a player has unlocked, rebirths + 1, from 1 (Common) to 9 (Transcendent).
- **D**: the shards one merge into your newest rarity pays (D-010): 3, 7, 15, 32, 70, 155, 340, 748, 1,646.
- **HUD**: the buttons and bars always on screen. **Reference screen**: 1600 x 900, HUD scale 1.0. Phones use scale 0.55.
- **Hands stack**: the bottom-centre elements that show what is in your hands: carry buttons, carry info bars, and at sea the Weapon Bar.
- **Notice lane**: where the green toast lands, H - 178 to H - 150 px, not scaled (`GardenToast.lua:64`).

## 1. Brain dump coverage

Every request in [[Brain Dump 2026-10-04]], and where it is answered.

| # | Request (his words, shortened) | Covered by | Section | Status |
|---|---|---|---|---|
| 1 | "install the music ... now" | [[Decisions Log]] D-002 | - | Live |
| 2 | "install ... the royal icons now" | [[King UI Overhaul]] | 7.6 (B1 to B12), D-001 | Waiting on his art; D-001 change is open question 40 |
| 3 | "add more potions", "only add the potions it makes sense to add", "dont overdo anything", "nobody would pay for a potion of decaying" | [[Potions]] | Why (candidate table), Rules 1 to 27 | Covered: 2 added (Aqua Affinity now, Haste with the second ore band) |
| 4 | "add bats ... every time you rebirth your bat upgrades and does more knockback like in steal a pet" | [[Bats]] | Rules B1 to B30, Numbers | Covered |
| 5 | "artilery and combat like tnt, nukes, etc." | [[Weapons and Artillery]], [[Explosives and Fruit Health]] | Rules; Weapons Q13 explains "nukes" | Covered |
| 6 | "super planned and tell me exactly what assets you need" | every spec's asset section; merged list in section 7 below | - | Covered |
| 7 | "dont make slop unless it looks clean and studded" | [[Plot Overhaul and Environment]] J; every model table | - | Covered |
| 8 | "the animations need to be top tier" | [[Royal Combat]] (6 sword clips), [[Bats]] (2 swings), [[Weapons and Artillery]] (5), [[Pets]] (procedural, D-004) | Asset requests | Covered; needs his publishing (D-005) |
| 9 | "tnt shouldnt instantly get rid of your fruits ... drain the health ... stronger tnt cost more" | [[Explosives and Fruit Health]] | Rules 19 to 29, N2, N4 | Covered |
| 10 | "a base upgrade ... for 100k shards ... underground ... look super good ... extend it further" | [[Underground and Ores]] | Rules 1 to 8, 5.2, 7.3 to 7.5 | Covered: 3 floors |
| 11 | "base upgrades scale vertically ... later ... towers upwards" | [[Plot Overhaul and Environment]] C3 (reserved tower slot); [[Underground and Ores]] 13 (reuse the floor table) | - | Reserved for later, as asked |
| 12 | "mine and merge ore ... coal which merge to iron ... make this a category in the index" | [[Underground and Ores]] | Rules 9 to 19, U4, U5 | Covered |
| 13 | "higher tier ores should be glow" | [[Underground and Ores]] | 5.1 glow ramp | Covered for later bands; Iron gets a faint glint |
| 14 | "in the forest we will add pets, merging two ... bigger ... more money ... equip ... walk around your base" | [[Pets]] | Rules P1 to P35 | Covered |
| 15 | "StudThemed Animal Pack ... feed it the requested fruit ... rarer animals ... match the rarity of the max fruit" | [[Pets]] | P1, P7, P9, P25 | Covered |
| 16 | "really research, dont create slop ... detailed features" (animals) | [[Pets]]; [[Pets - Taming, Merging and Wandering]] | whole note | Covered |
| 17 | "improve the look of the default plots by 10x" | [[Plot Overhaul and Environment]] | A to E | Covered |
| 18 | "switch out all round logs for the studded logs" | [[Plot Overhaul and Environment]] F; D-003 | - | Covered |
| 19 | "turn the current lily pads into a studded texture" | [[Plot Overhaul and Environment]] G; D-021 | - | Covered (where they go is open question 46) |
| 20 | "remove the currents ... boats ... buy with shards ... 1000 shards ... swim ... fatigues ... aqua affinity potion" | [[Boats and Swimming]], [[Potions]] | R1 to R60; Potions rule 24 | Covered |
| 21 | "boats can fit up to two people ... broken by the artillery ... look at sky rivals" | [[Boats and Swimming]] R31, R44; [[Weapons and Artillery]] | - | Covered (who can break boats is open question 26) |
| 22 | "all Uis ... follow the UI that we created ... chatgpt or gemini ... references" | every spec's UI section; [[King UI Overhaul]] 7 | - | Covered |
| 23 | "king stuff ... doesnt follow the UI ... massive rehual ... figma or chatgpt prompts" | [[King UI Overhaul]] | whole note | Covered |
| 24 | "rarity in the index ... unlock by rebirthing ... color of the bat" | [[Rarity Gating and Rebirth]] U1; [[Bats]] B3 | - | Covered |
| 25 | "zero rebirths the max fruit ... apple ... cant merge past that" | [[Rarity Gating and Rebirth]] rules 1 to 7 | - | Covered |
| 26 | "+1 storage everytime you rebirth" | [[Rarity Gating and Rebirth]] rule 17 | - | Covered |
| 27 | "to rebirth ... fill out your index + shards + money. dont make it easy" | [[Rarity Gating and Rebirth]] rule 12, Table 2 and 3; D-011 | - | Covered |
| 28 | "slight boost ... start from the beginning ... access to both common and uncommon" | [[Rarity Gating and Rebirth]] rules 11, 16; D-006, D-008 | - | Covered (head start is open question 2) |
| 29 | "obvious ... to unlock more fruits ... they need to rebirth" | [[Rarity Gating and Rebirth]] U1 to U5; the Weapons tab ordered by unlock (D-022); "Also unlocks" line in the Rebirth menu | - | Covered |
| 30 | "plan ... structures and flow ... write down and simulate paths the player can go through and decisions" | [[Stage Model]] (4 player types); [[Integration Check]] Check F (a full simulated path) | - | Covered: seven player paths in `04 Player Paths`, ranked in [[00 Paths Summary]], plus Check F |
| 31 | "dopamaine slop ... structured ... UI should never overlap ... super clean" | [[King UI Overhaul]] rules 11 to 24; section 4 below | - | Covered; the HUD collisions found here (C21 to C25, C40) are fixed |
| 32 | "match my font ... I dont like the emoji-slop" | every spec (FredokaOne, no emoji); [[King UI Overhaul]] rules 1 to 10 | - | Covered |
| 33 | "the castle ... looks really AI ... main issue is with the text font" | [[King UI Overhaul]] 5.11 | - | Covered |
| 34 | "royal sword ... slashes ... health drain ... effects ... royal slam ... weopon skins, you select right before fighting" | [[Royal Combat]] | Rules A to G | Covered |
| 35 | Statue With Sword, Sword Animations, Stud Shop, Stud Bridge, Studded Pack: "figure out the uses" | [[Plot Overhaul and Environment]] I; [[Royal Combat]] (blades, statues, stall); [[Boats and Swimming]] (docks, shop); [[Weapons and Artillery]] (pier, booth); [[Underground and Ores]] (timber); [[Pets]] (camp) | - | Covered |
| 36 | "Studed-Mountain ... replace the mountains ... on the border" | [[Plot Overhaul and Environment]] H | - | Covered |
| 37 | "ores ... and animals ... reasonable in the amount of money ... match what stage" | [[Pets]] P25 to P27; [[Underground and Ores]] 5.3; [[Integration Check]] Checks A and E | - | Covered; budget fixed (D-018) |
| 38 | "stages ... the rarity of fruit the player is on ... common ... to transcendent" | [[Stage Model]]; [[Rarity Gating and Rebirth]] | - | Covered |

## 2. Conflicts found and how each was resolved

"Changed" lists the notes that were edited. Every edited spot cites this review or the decision.

| # | Conflict | Resolution | Changed |
|---|---|---|---|
| C1 | Rarity colours written three ways: exact code values (Bats, Plot Overhaul), rounded 0-1 values (Boats, UI Kit table), and off-by-one hex (Weapons, Royal Combat) | Every system reads `Config.RarityColors` (`GardenConfig.lua:582-592`); exact RGB and hex listed once in [[UI Kit Inventory]] | UI Kit Inventory, Royal Combat, Weapons, Boats |
| C2 | Four different health-bar art requests (256 x 32 from RateBar; 336 x 54 from RateBar; 1024 x 256 from CashBar; King UI A8 and A9) | One pair, [[King UI Overhaul]] A8 and A9 (336 x 54, from RateBar), for fruit, nodes, boats, fighters and timers (D-027) | Explosives, Underground, Royal Combat, Boats, King UI |
| C3 | HP colour bands: fruit 50 / 20%, boat 50 / 25%, fighter 50 / 26%, King UI "tinted red" | Above 50% green, 25 to 50% gold, under 25% red, everywhere (D-027) | Explosives, King UI, Royal Combat |
| C4 | Duplicate art requests: ARMORY title by ChatGPT vs King UI's Figma titles; a combat-button sheet with code labels vs King UI F1 to F7 with baked words; BOAT DOCK title by ChatGPT | King UI owns F1 to F7, A10 and one Figma title batch that now also lists ARMORY (T10), REBIRTH, MAX FRUIT, MINE, PETS, BOAT DOCK | Royal Combat, King UI, Boats, Rarity Gating |
| C5 | Shop tabs: Weapons wanted {Tools, Weapons, King, Robux}, Potions {Tools, Potions, King, Robux}, Explosives kept TNT on Tools. Together that is 5 tabs at 0.15 of the panel | {Potions, Weapons, King, Robux}: Tools becomes Potions; explosives join the sea weapons on Weapons, ordered by unlock stage (D-022) | Potions, Weapons, Explosives |
| C6 | Luck at the cap: Rarity let Luck grow the second-best fruit (breaks D-007); Potions refused Luck at the ceiling | Potions' rule: refused at the ceiling, D-007 strict (D-019) | Rarity Gating |
| C7 | Four copies of the D-010 table (Rarity, Potions, Weapons, Royal Combat), three rounding rules (3 significant figures, 2 significant figures, none), two stage helpers with the same name | One block in GardenConfig: `StageFor`, `ShardDropFor`, `NicePrice`, `ShardPrice`, `ShardReward` (D-017). Price tables recomputed | Rarity Gating, Potions, Explosives, Weapons, Royal Combat, Bats |
| C8 | Invisibility: Potions said it does not stop bat hits; Bats and Weapons said invisible players cannot be hit, swing or fire | Bats and Weapons win (D-030) | Potions |
| C9 | Aqua Affinity: Boats said 250 shards, 30 studs/s, cyan, its own art requests, attribute `Swimming`; Potions said stage price, x2.0 (24 studs/s), blue | Potions owns price (75 to 500), duration, colour (40, 130, 255) and art; speed x2.5 = 30; attribute `SwimState` (D-029) | Boats, Potions |
| C10 | Boats vs Weapons: different damage numbers, falloff, function name (`G.boatDamage` / `G.damageBoat`), attribute names (`HP` / `BoatHP`), shield 15 s / 10 s, and Boats let any artillery sink any boat | Weapons owns the numbers, Boats owns the boat; one falloff; `G.damageBoat`; Boats' attribute names; 15 s; War Flag required (D-024) | Boats, Weapons |
| C11 | Explosives allowed TNT in your own mine; Underground forbade explosives in mines | Not in v1 (D-031) | Explosives |
| C12 | Rarity rule 15 and D-009: rebirth keeps tools; Explosives refunds explosives on rebirth | Refund explosives at the pre-rebirth price; keep potions and weapon charges (D-020) | Rarity Gating, Explosives |
| C13 | Plot Overhaul keeps Plot Size through rebirth; D-009, Rarity rule 15, Stage Model rule 4 and Pets reset it | Keep it (D-020, measured +0.05 h) | Rarity Gating, Plot Overhaul, Pets, Stage Model |
| C14 | Two underground entrances: Underground's Mine Lift on the plaza side; Plot Overhaul's hut in the back-right lot corner | One: the hut, with the lift inside (D-023). The plaza spot would stand in the new front yard | Underground, Plot Overhaul |
| C15 | Underground ownership attribute named `UndergroundFloors` (Underground) and `UndergroundLevel` (Plot Overhaul, Potions) | `UndergroundFloors` everywhere | Plot Overhaul, Potions |
| C16 | Pets 25% plus ores 17.5% as a cash-multiplier factor compound to 46.9% of full-plot $/s, not 35% | Pets 25%, ores 7.5 / 9 / 10.5% on fruit income only: 35.5% (D-018, [[Integration Check]] Check A) | Underground, Pets, Rarity Gating, Stage Model |
| C17 | Index categories: Pets used two title-bar pills, Underground used vertical tabs | One tab column, Fruits, Pets, Ores (D-026) | Pets, Underground |
| C18 | Two "Armory"s: Weapons' booth and Royal Combat's blade panel | Weapons' becomes the "Weapons Booth" | Weapons |
| C19 | Key 1 and gamepad L1 were the bat in Bats and the first weapon in Weapons | 1 / L1 = bat; 2 to 5 / R1 = weapons (D-025) | Weapons, Bats |
| C20 | Bat, weapon and pickaxe could all be "in hand" | One hand: each holsters the others (D-025) | Bats, Weapons |
| C21 | Touch bottom right: Potions put the effect block left of the jump button, where Bats put the Swing button and Weapons the Fire button; on a 667 x 375 phone the BatSlot also sat on the Swing button | One touch action slot (Bat, Swing or Fire); the touch effect block moves to the top left; no bottom-row BatSlot on touch (D-025) | Bats, Weapons, Potions |
| C22 | BatInfo (two bars, y -121 to -179) crossed the notice lane | One bar at y -119 to -146 | Bats |
| C23 | SeaStatus at y -126 to -200 crossed the notice lane; King UI suggested a top-centre `Kit.Slot` card for stamina instead | Bottom centre with a placement rule that skips the notice lane (D-025); `Kit.Slot` stays royal-only | Boats, King UI, Weapons |
| C24 | Duel board: Royal Combat kept it at 70 px, 560 x 112, own scale curve, ArmoryUI at DisplayOrder 5; King UI moved it to the 112 px event strip, 560 x 96, `Kit.scale`, layers 1 / 2 / 20 / 30 | King UI's layout; the Armory is in the modal band at 20 (D-028) | Royal Combat, King UI |
| C25 | Arena PC controls: Royal Combat's ability bar and text hint vs King UI's legend plates | Ability bar at 54 px, legend at 16 px, no hint line (D-028) | Royal Combat, King UI |
| C26 | Arena focus: Royal Combat kept CornerHud visible, King UI hid it; neither listed the new clusters | Hide everything but Mute, including the new clusters (D-028) | Royal Combat, King UI |
| C27 | Royal rewards: King UI proposed k = 33 / 67 / 200 anchored at stage 3; Royal Combat proposed 10 / 25 / 40 units; decree rewards stayed flat with each note pointing at the other | Royal Combat's units, and it now owns decree units too (D-017) | King UI, Royal Combat |
| C28 | `MinDiscovered` ownership: Rarity said King UI owns it, King UI said Rarity owns it | Rarity Gating owns and makes it 5 (D-026) | Rarity Gating, King UI |
| C29 | Royal Combat refuses rebirth while fighting; Rarity's rebirth checks did not | Added to Rarity rule 14 (D-033) | Rarity Gating |
| C30 | Two crate systems (Golden Crate, Sea Crate), and the Sea Crate paid cash outside the side-income budget | Golden Crate only in v1; Sea Crates later, shards only (D-016) | Boats |
| C31 | Bat safe band reached 160 studs below a plot; the mine's third floor reaches 216, and a cave is wider than a level-0 soil | Band 240 studs; every mine floor box tagged `BatSafeZone` (D-014) | Bats, Underground |
| C32 | Underground said "no mining potion"; Potions shipped Haste with the Underground, where the v1 mine fills in about 3 minutes | Haste waits for the second ore band (D-032) | Potions, Underground |
| C33 | Bats knocked riders off boats; Boats and Weapons said seated riders are never knocked | Seated boat riders cannot be hit by bats (D-014) | Bats, Boats |
| C34 | Lily pads: D-012 restores the original mesh pads in the sea with the Studs material; Plot Overhaul rebuilt pads from plates and put 19 in ponds. Samarth asked on 29 Sep to remove the sea pads | D-012's look (mesh plus Studs), Plot Overhaul's place (ponds; sea pads stay in the backup) (D-021) | Plot Overhaul |
| C35 | The Backpack hotbar was switched off by Bats and Weapons and switched on again by Underground when leaving a mine | Only BatHud touches it, once (D-025) | Bats, Weapons, Underground |
| C36 | Rarity's dependency notes were stale: "pin pets and ores to full(S)", "TNT can destroy a cap fruit" | Rewritten to match Pets, Underground and Explosives | Rarity Gating, Pets |
| C37 | Explosives' new `EmpirePower` attribute used only `plotBase x cashMultiplier`, dropping pet and ore income | Same formula as the income tick, counting stunned fruit | Explosives, Underground, King UI |
| C38 | Bats proposed decisions D-012 to D-015, but D-012 was already the lily pad decision | Logged as D-013 to D-016 | Bats, Decisions Log |
| C39 | The Rebirth menu shows a "bat chip" from Bats, but Bats requested no chip image | Bats asset request 5 added | Bats, Rarity Gating |
| C40 | On a 667 x 375 phone the King UI herald (y 173 to 208) crossed the notice toast (y 197 to 225) | The herald takes the strip card's place when it would cross the lane | King UI |
| C41 | Six new panels (Pets, Mine, Boat Dock, Armory, Max Fruit, Kingdom) each listed the others by hand in "hide the other panels" lists | One list, `UIKit.Modals` | King UI |
| C42 | Stage Model rules 4 and 6 no longer matched the specs | Updated in place, pointing to D-018 and D-020 | Stage Model |
| C43 | Four notes proposed their own "set rebirths" Studio door (`DevSetRebirths`, `batRebirths`, `stage`) | One door, `DevSetRebirths`, owned by Rarity Gating; others use it | Royal Combat (test), noted for Bats |
| C44 | [[Plot Overhaul and Environment]] rule F rebuilt the Studded Pack log, but D-003 already chose the square trunk-style log (Earth orange, studs on four faces, Burlap end grain) from a side-by-side | Follow D-003; the pack log stays behind the builder flag | Plot Overhaul |

## 3. Griefing check

Rule: no system may let a player take or permanently destroy another player's things, unless Samarth asked for it (draining fruit health with TNT was asked for).

| System | What it can do to another player | Takes or permanently destroys anything? | Protections | Consistent after this review |
|---|---|---|---|---|
| [[Bats]] | Push a player; make a Golden Crate carrier drop the crate | No. The crate is neutral until claimed | Safe zones (every plot and 240 studs under it, mine floors, docks, stalls, spawns, beanstalk, arrival pad), join and respawn shields, 1.2 s immunity, 4 hits per 20 s per pair, knock grace (no fall death, no potion loss), never seated boat riders, the seated King, invisible players or the arena | Yes |
| [[Explosives and Fruit Health]] | Drain fruit HP to a 20 s stun | No. Stun only (`ZeroHpMode = "stun"`); destroy would need Samarth's explicit yes | Stage 1 immune, 120 s fresh gardens, damage at the lower of the two stages, Lock blocks everything, 30 s shield after a stun, 1 armed per player and 12 per server, nuke limits, no mines, no player damage, attacker gains nothing | Yes |
| [[Weapons and Artillery]] | Push flagged players; damage flagged boats | No. A wrecked boat is rebuilt in 60 s; riders keep what they carry | War Flag on both sides, sea only, dock no-fire boxes, 15 s spawn protection, 1.5 s immunity, the flag drops when your boat is sunk, no pay for sinking, never fruit, pets or currency | Yes |
| [[Boats and Swimming]] | Nothing | No | Friends-only passengers, no boat or character collisions, wash-up instead of death | Yes |
| [[Pets]] | Nothing | No | Private wild animals, no trading or gifting, pets never take damage | Yes |
| [[Underground and Ores]] | Visit a mine and look | No | Owner-only prompts checked on the server, watchdog, visitors sent home when the owner leaves | Yes |
| [[Royal Combat]] | Duel for the crown (by challenge) | No. The crown is a title; rewards are shards from the game | Repeat-pair rule, forfeit pays nothing, arena-only | Yes |
| [[Potions]] | Nothing (self only) | No | Rule 3 | Yes |
| [[Plot Overhaul and Environment]], [[Rarity Gating and Rebirth]], [[King UI Overhaul]] | Nothing | No | - | Yes |

The protections now agree across systems: one safe-zone list, one knock function (`G.knockPlayer`) with the same grace, one Lock, one rule that invisible players neither hit nor get hit, and one rule that seated boat riders are only affected through flagged boat damage.

## 4. Unified HUD layout

Positions at the 1600 x 900 reference (HUD scale 1.0) and on a 667 x 375 phone (scale 0.55, touch). "Unscaled" means the position does not follow the HUD scale. Rects are x range, y range in screen pixels. Every element below is checked by the [[King UI Overhaul]] `UIOverlapCheck`, which must include the new clusters.

| Element | Owner | Zone | Reference position and size | 1600 x 900 rect | 667 x 375 rect | Shown when | Fix in this review |
|---|---|---|---|---|---|---|---|
| Lock and Upgrade pills (TopButtons) | existing | top centre | y 7, 416 x 50 | 592-1008, 7-57 | 219-448, 4-31 | always (not in arena) | - |
| Lock alert "!" | [[Explosives and Fruit Health]] | on the Lock pill's outer top corner | 31 x 35 | about 577-608, 0-35 | about 211-228, 0-19 | an explosive is armed on your plot | - |
| Mute | existing | top right | kit HUD button | top-right corner | top-right corner | always, even in the arena | - |
| Warn toast (red) | existing | top centre | label 0.9 W x 28, unscaled | y 76-104 | y 76-104 | on a warning | - |
| Royal chip | [[King UI Overhaul]] | top left | (16, 8), 286 x 62 | 16-302, 8-70 | 9-166, 4-38 | always (not in arena or with a modal open) | - |
| Royal action pill | [[King UI Overhaul]] | top left | (16, 78), 180 x 44 | 16-196, 78-122 | dropped (rule 13) | Watch, Join or Answer | - |
| Event strip card | [[King UI Overhaul]] | top centre | top edge 112 unscaled, 560 x 96 | 520-1080, 112-208 | 179-487, 112-165 | a royal event | - |
| Herald | [[King UI Overhaul]] | under the strip | 640 x 64 | 480-1120, 216-280 | 157-509, 173-208, or at 112 when it would cross the notice lane | server news | C40 |
| Left column: Inventory, Index, Shop | existing | left edge | x 16, centres -181, -78, +24 from the middle; 96 x 92 | 16-112: 223-315, 326-418, 428-520 | 9-62: about 50-100, 107-157, 163-213 (after the Pets guard lifts the column 13 px) | always | - |
| Pets button | [[Pets]] | left column, 4th rung | x 16, centre +126 | 16-112, 530-622 | 9-62, about 219-269 | always | Guard keeps it 8 px above CornerHud |
| Right column: Rebirth, Merge All, Clear | existing | right edge | x -16, centres -126, -45, +45 | 1488-1584: 278-370, 359-451, 449-541 | 605-658: 93-144, 137-188, 187-238 | always | - |
| CornerHud (cash, shards, boost) | existing | bottom left | 222 x 161 | 16-238, 723-884 | 9-131, 278-366 | always (not in arena) | - |
| Effect block (potions) | [[Potions]] | bottom right (mouse) / top left (touch) | 262 x 180, 4 per row (was 262 x 275; M4 Studio pass) / 126 x 360, 2 per row | 1322-1584, 704-884 | 70-140, 46-250 | an effect runs | C21: moved off the jump area on touch |
| Carry buttons | existing | bottom centre (hands) | y -21, buttons 168 x 56 | about 540-1060, 823-879 | 238-428, 333-364 | carrying | - |
| Carry info bars | existing | bottom centre (hands) | y -87, 168 x 58 | 716-884, 755-813 | 287-380, 295-327 | carrying | - |
| Weapon Bar | [[Weapons and Artillery]] | bottom centre (hands) | y -21, 448 x 102 | 576-1024, 777-879 | 210-456, 307-364 | at sea or on the Practice pad, not carrying | Keys 2 to 5 (C19) |
| BatSlot | [[Bats]] | bottom centre-right | (0.5, 320, 1, -21), 96 x 92 | 1072-1168, 787-879 | not drawn on touch | mouse, keyboard or gamepad | C21 |
| BatInfo | [[Bats]] | above the BatSlot | (0.5, 320, 1, -119), 168 x 27 | 1036-1204, 754-781 | hidden on touch | bat equipped | C22: one bar |
| Notice toast (green) | existing | bottom centre | label 0.9 W x 28, bottom at H - 150, unscaled | y 722-750 | y 197-225 | on good news | lane kept clear (King UI rule 14) |
| SeaStatus (stamina or boat HP) | [[Boats and Swimming]] | bottom centre, above the hands stack | 218 x 74, placed by the gap rule | 691-909, 640-714 | 273-393, 246-287 | swimming or seated in a boat | C23 |
| Touch action slot (Bat, Swing, Fire) | [[Bats]] (Fire from [[Weapons and Artillery]]) | left of the jump button | 128 x 128, x0.72 on small screens | - | 478-570, 247-339 | touch, not in arena, not carrying | C21 |
| Roblox jump button | Roblox | bottom right | Roblox default | - | 572-642, 285-355 | touch | - |
| Reticle / hit marker | [[Weapons and Artillery]] | screen centre | 36 x 36 | 782-818, 432-468 | centre | Cannon drawn, or any weapon on touch | - |
| Centre panels: Index (with Fruits / Pets / Ores tabs), Shop (4 tabs), Inventory, Upgrades, Rebirth, Pets, Mine, Boat Dock, Kingdom, Armory | each owner | centre modal | Index shell, up to 948 x 534 | about 326-1274, 183-717 | scaled | one at a time (`UIKit.Modals`) | C17, C41 |
| Max Fruit popup | [[Rarity Gating and Rebirth]] | centre modal | 36% of width, aspect 520:290 | about 512-1088, 290-611 | min 340 x 190, centred | first cap fruit of a stage | - |
| Confirm dialogs (Clear, Rebirth, Buy Floor, Release, challenge) | each owner | centre, above panels | ClearConfirm art 565 x 373 | centred | centred | on demand | - |
| Travel fade | [[Underground and Ores]] | full screen | own ScreenGui | full | full | about 1 s per lift trip | - |
| Arena: touch combat buttons | [[Royal Combat]] / [[King UI Overhaul]] | around the jump button | F1 to F7 | - | around the jump | `InArena` only | - |
| Arena: ability bar | [[Royal Combat]] | bottom centre | 54 px up, 212 x 64 (360 x 64 for the King) | 694-906, 782-846 | hidden on touch | `InArena` only | C25 |
| Arena: PC legend | [[King UI Overhaul]] | bottom centre | 16 px up, plates 140 x 30 | centred, 854-884 | hidden on touch | `InArena` only | C25 |
| Arena: stinger | [[King UI Overhaul]] | centre band | 120 tall | between the strip and H - 186 | same, hides the card if short | in or near the arena | - |

Rules that keep this table true:
- **Hands stack** (bottom centre): the carry row and the Weapon Bar never show together, because nobody can draw a weapon while carrying. SeaStatus always sits above the hands stack and never in the notice lane (Boats 6.1).
- **One hand** (D-025): bat, weapon, pickaxe and carried items exclude each other, so the BatSlot, Weapon Bar and carry row each show their own state without fighting.
- **Touch** (D-025): one action slot left of the jump button; the effect block lives top left; BatSlot and BatInfo are not drawn.
- **Arena focus** (D-028): everything except Mute hides.
- **Modals** (rule 15 of the King UI): one at a time, from `UIKit.Modals`; while a modal is open the royal HUD layer hides and SeaStatus and BatInfo hide.

## 5. Unified save-document additions

No schema bump anywhere: adding fields is forward and backward compatible (`GardenStore.lua:8-12`). Every new field must be added to both `Store.Default()` and `sanitise`, or it is dropped on the next load (`GardenStore.lua:447-450`). No two systems use the same field name.

| Field | Type | Default | Owning system | Migration |
|---|---|---|---|---|
| `gateVersion` | whole number 0 to 1 | 0 | [[Rarity Gating and Rebirth]] | One-time Placement migration in `applyProfile` (raises `rebirths` to cover owned fruit, clamps `upgrades.spawnTier` with a cash refund); idempotent |
| `rebirths` (existing) | whole number | 0 | [[Rarity Gating and Rebirth]] | May be raised by the migration, never lowered; rules use `min(R, 8)` |
| `upgrades.spawnTier` (existing) | whole number | 0 | [[Rarity Gating and Rebirth]] | Clamped to the new spawn ceiling, removed levels refunded as cash |
| `upgrades.plotSize` (existing) | whole number 0 to 4 | 0 | [[Plot Overhaul and Environment]] | None; from now on kept through rebirth (D-020) |
| `index` (existing) | table | {} | [[Rarity Gating and Rebirth]] | None; no longer wiped on rebirth (D-009) |
| `batSeen` | whole number 0 to 9 | 0 | [[Bats]] | None; old saves get one intro toast |
| `tools` (existing): new keys `TNT1`, `TNT2`, `TNT3`, `Nuke`, `AquaAffinityPotion`, later `HastePotion` | whole-number counts | absent = 0 | [[Explosives and Fruit Health]], [[Potions]] | Old `TNT` refunded at 2,500 shards each through `RETIRED_PRICES`, then dropped |
| `effects` | `{ effect = seconds left }`, 0 < v <= 86,400 | {} | [[Potions]] | None |
| `weapons` | `{ key = 1 to maxHeld }` | {} | [[Weapons and Artillery]] | None; kept through rebirth |
| `ugFloors` | whole number 0 to 3 | 0 | [[Underground and Ores]] | None |
| `ores` | `{ ore name = count }` | {} | [[Underground and Ores]] | None; over-slot counts load and only block mining |
| `oreIndex` | `{ ore name = 0 or 1 }` | {} | [[Underground and Ores]] | None |
| `pets.owned`, `pets.equipped`, `pets.index`, `pets.bought`, `pets.hint` | tables, whole number 0 to 2, boolean | {}, {}, {}, 0, false | [[Pets]] | None |
| `boats` | `{ boat key = true }` | {} | [[Boats and Swimming]] | None |
| `royal.skin` | string, a key of `C.WeaponSkinByKey` | "RoyalSword" | [[Royal Combat]] | Absent: default; unknown: default plus one repair |
| `royal.skinsBought` | `{ key = true }`, shard skins only | {} | [[Royal Combat]] | Unknown or non-shard keys dropped |

Not saved anywhere (runtime only): fruit HP and stuns, War Flag, cooldowns, stamina, boat HP, node HP, ore positions, wild animals, crate state, bat tier (worked out from `rebirths`).

**Payload size.** The GardenStore self-test checks the worst payload against 8,192 characters. Pets measured 6,764 with pets alone; the other additions bring it to about 7,400 [estimate, not measured]. That passes with under 10% headroom, so raise the self-test ceiling to 16,384 when these fields land (the real DataStore limit is 4 MB per key).

**Attribute names that look alike but live on different instances** (safe, listed so nobody merges them): `HP` on fruit (Explosives), boat models (Boats) and ore nodes (Underground); `ShieldUntil` on fruit; `ProtectedUntil` on boat models and in RoyalState; `BatShieldUntil` on players.

## 6. Unified remotes

All live in `ReplicatedStorage.GardenRemotes`. No name collides with an existing remote. Every client-to-server remote is type-checked and re-validated on the server.

| Remote | New or changed | Direction | Payload | Owner | Server check (short) |
|---|---|---|---|---|---|
| `RarityGate` | new | server to client | `{kind, fruit?, amount?, at?}` | [[Rarity Gating and Rebirth]] | server never listens |
| `Rebirth` | changed | client to server | none | [[Rarity Gating and Rebirth]] | full `RebirthStatus`, no merge mid-animation, not in the arena or royal busy, R below 8 |
| `Upgrade` | changed | client to server | upgrade key | [[Rarity Gating and Rebirth]] | cap-aware price; `SpawnCapText` at the ceiling |
| `MergeAll` | changed | client to server | none | [[Rarity Gating and Rebirth]] | cascade stops at the cap rank |
| `BuyTool` | changed | client to server | tool key | [[Potions]], [[Explosives and Fruit Health]] | `Config.ShardPrice`; unlock stage for explosives; `requires` for Haste |
| `DrinkTool` | new | client to server | potion name | [[Potions]] | consume tool, loaded, 0.3 s cooldown, not in arena, `potionUsable`, bag count taken last |
| `Drop` | changed | client to server | none | [[Explosives and Fruit Health]], [[Bats]], [[Underground and Ores]] | arms an explosive (all placement gates), drops a crate or an ore |
| `ExplosiveFx` | new | server to all | `(event, data)` | [[Explosives and Fruit Health]] | server-only sender |
| `BatEquip` | new | client to server | boolean | [[Bats]] | 4 per second; B5 refusals; one hand |
| `BatSwing` | new | client to server | Player or nil | [[Bats]] | own cooldown; B12 checks a to m |
| `BatKnock` | new | server to victim | `{vel, stun, tier}` | [[Bats]] | server-only; also used by weapon blasts through `G.knockPlayer` |
| `BatFX` | new (unreliable) | server to nearby | `(kind, attackerId, victimId?, pos?, tier)` | [[Bats]] | cosmetic, under 60 bytes |
| `WeaponRequest` | new | client to server | `("buy", key)`, `("draw", key)`, `("holster")`, `("fire", key, aim, seq)`, `("flag", up)` | [[Weapons and Artillery]] | 0.1 s gate; rules 4, 9, 12 to 14, 27 to 29; server computes origin and velocity |
| `WeaponEvent` | new | server to all / shooter / victim | `shot`, `ack`, `nack`, `warn`, `boom`, `hit`, `knock`, `fort` | [[Weapons and Artillery]] | server-only |
| `UndergroundBuy` | new | client to server | floor number | [[Underground and Ores]] | equals `ugFloors + 1`, loaded, has plot, not rebirthing, shards |
| `OreIndexClaim` | new | client to server | ore name | [[Underground and Ores]] | known ore, state 0 |
| `PetRequest` | new | client to server | `(action, arg)` | [[Pets]] | action list, 0.1 s gate, 0.5 s feed gate, P13 checks |
| `PetEvent` | new | server to owner | `wild`, `fed`, `tamed` | [[Pets]] | server never listens |
| `BoatBuy` | new | client to server | boat key | [[Boats and Swimming]] | known key, loaded, not owned, near a dock, shards |
| `SeaCrate`, `SeaCrateClaim` | Phase 2 only, not created in v1 | - | - | [[Boats and Swimming]] | - |
| `RoyalCombat` | changed | client to server | `strike`, `charge`, `heavy`, `block`, `dash`, `slam` | [[Royal Combat]] | 20 per second, stun and hitstun gates, server combo |
| `RoyalRequest` | changed | client to server | adds `skinPick`, `skinReady`, `skinBuy`, `skinRobux`; panel sends `cancelChallenge` | [[Royal Combat]], [[King UI Overhaul]] | 0.15 s gate, ownership, `ShardPrice`, not in arena for buys |
| `RoyalQuery` | changed | client asks server | `decrees` answer carries this player's reward | [[King UI Overhaul]] | 6 calls per 2 s |
| `RoyalEvent` | changed | server to clients | `swing`, `hit`, `slamStart`, `point` changed; `pickOpen`, `pickClose`, `openArmory` new; `announce` gains `scope`, `icon` | [[Royal Combat]], [[King UI Overhaul]] | server-only |
| `Notice`, `Notify` | changed | server to client | plain ASCII only | every system | server-only |
| `Income` | unchanged shape | server to client | `(multiplier, dt)` | existing | base now = `plotBase x (1 + OreBonus) + petBase`, so `rate / base` is still the cash multiplier |

Prompts handled on the server (not remotes): Golden Crate lift ([[Bats]]), boat Dock / Shop / Drive / Board ([[Boats and Swimming]]), Mine Lift, nodes and ores ([[Underground and Ores]]), pet feeding uses `PetRequest`, Royal `Armory` stall ([[Royal Combat]]), Weapons Booth ([[Weapons and Artillery]]).

## 7. Merged asset requests

What Samarth makes, after removing duplicates. Each note keeps the full prompt.

| Batch | Images | Requested in |
|---|---|---|
| Figma: INDEX-style titles | KINGDOM, CROWN CHALLENGE, 7 stingers, 14 castle signs, ARMORY, REBIRTH, MAX FRUIT, MINE, PETS, BOAT DOCK | [[King UI Overhaul]] 7.3 (one frame, words changed) |
| Figma: combat buttons with baked words | F1 to F7 | [[King UI Overhaul]] 7.10 |
| Kit recolours (Figma first) | pills A1 to A5, royal chips A6 and A7, **bar track A8 and fill A9 (used by every bar)**, charge ring A10 | [[King UI Overhaul]] 7.5 |
| Royal icons | B1 to B12, decree C1 to C5, medals D1 to D8, crowns E1 to E6 | [[King UI Overhaul]] 7.6 to 7.9 |
| Rarity and rebirth | padlock, Index chip, Storage chip, green tick, LOCKED plate | [[Rarity Gating and Rebirth]] A3 to A7 |
| Bats | tintable bat icon, "Bat" label, "Swing" label, bat chip | [[Bats]] requests 1, 2, 3, 5 |
| Weapons | Weapons tab label (only if the tab uses an icon), 4 weapon tiles, 2 flag tiles, reticle, hit marker, Fire button | [[Weapons and Artillery]] A1 to A6 |
| Potions | Aqua Affinity shop art and badge, Haste art (with the second ore band), Speed / Jump / Invisibility shop art; Feather Falling and Luck still pending | [[Potions]] 1 to 8 |
| Underground | MINE title (in the Figma batch), 3 Index tab icons (Fruits, Pets, Ores), OWNED button | [[Underground and Ores]] A1 to A3 |
| Pets | PETS title (in the Figma batch), Pets HUD button | [[Pets]] A1, A2 |
| Explosives | optional Nuke card art | [[Explosives and Fruit Health]] 8 |
| Animations (publish under `hikeALT12212`) | 6 sword clips, 2 bat swings, 5 weapon clips (all with working fallbacks) | [[Royal Combat]], [[Bats]], [[Weapons and Artillery]] |
| Robux | Storm Edge game pass (optional) | [[Royal Combat]] |

Withdrawn as duplicates: Explosives' 256 x 32 health bar, Underground's A4 bar, Royal Combat's bar, combat-button sheet and ChatGPT ARMORY title, Boats' Aqua Affinity icon and shop art.

## 8. Open questions for Samarth

Deduplicated across all 11 notes and this review. Each has a recommended default, so building can start without waiting. "Was" names the notes that asked.

**Progression and economy**

| # | Question | Recommended default | Was |
|---|---|---|---|
| 1 | Existing players: placed at the rebirth their fruit prove, or a full reset? | Placement, keep everything | Rarity Q1 |
| 2 | Rebirth 2 and later start from the first fruit of the rarity below, not from Blueberry? | Yes (D-006); rebirth 1 starts from Blueberry | Rarity Q6 |
| 3 | Old-rule rebirthers drop from x2 per rebirth to +25% per rebirth? | Accept (119 lifetime visits) | Rarity Q7 |
| 4 | Your own save migrates to Rebirth 8? | Yes; test with StudioNoSave and `DevSetRebirths 0` | Rarity Q8 |
| 5 | Anything past 8 rebirths? | No | Rarity Q3 |
| 6 | Luck at the cap: refused at the spawn ceiling, or let it grow the second-best fruit? | Refused (D-019) | Rarity Q2, Potions Q9 |
| 7 | Pets and ores split of the 35%: pets 25% and ores 10.5%, or 17.5% each? | 25% / 10.5%, ores on fruit income only (D-018) | Pets Q3, Underground Q3 |
| 8 | Keep Plot Size through rebirth? | Yes (D-020) | Plot Overhaul Q2 |
| 9 | Refund explosives on rebirth (instead of keeping them)? | Yes, at the pre-rebirth price (D-020) | Explosives Q5 |
| 10 | Floor 1 arrives about 8.5 h in (stage 8), not stage 6 to 7. Keep 100K? | Keep 100K | Underground Q1 |
| 11 | Floors 2 and 3 at 150K and 300K; ores kept through rebirth? | Yes and yes | Underground Q4, Q5 |
| 12 | Stage-scaled shard prices for potions (Aether 150 shards at stage 1)? | Yes | Potions Q1, Q2 |
| 13 | Stage-scaled royal and decree rewards? | Yes (D-017) | Royal Combat Q5, Q14; King UI Q10 |
| 14 | Pet money follows your best fruit this run? | Yes | Pets Q1 |
| 15 | Taming open from stage 1? | Yes | Pets Q7 |
| 16 | Pet slots for 10K and 150K shards; bag of 60; no release refund; no trading? | Yes to all | Pets Q8, Q10, Q14, Q15 |
| 17 | Golden Crate in the first release, worth 20 frontier merges? Sea Crates later? | Yes; Sea Crates off in v1 (D-016) | Bats Q2, Q3; Boats Q8 |
| 18 | Ore and pet Index cards stay out of the rebirth rule and crown eligibility? | Yes (D-026) | Rarity Q4, Pets Q4 |
| 19 | Index rule counts "discovered", claimed or not? | Yes | Rarity Q11 |

**Combat and fairness**

| # | Question | Recommended default | Was |
|---|---|---|---|
| 20 | Fruit at 0 HP: stunned 20 s or destroyed? | Stunned | Explosives Q1 |
| 21 | Explosives can hit other players' gardens (with all protections)? | Yes | Explosives Q2 |
| 22 | Explosives unlock at rebirth 1 (stage 1 is a safe start)? | Yes | Explosives Q3 |
| 23 | Nuke carried and placed (not a map air-drop); survivor bonus on? | Yes and yes | Explosives Q7, Q13 |
| 24 | Explosives in your own mine? | Not in v1 (D-031) | Explosives Q9, Underground Q10 |
| 25 | Lock stays 60 s up, 30 s recharge; damaged (not stunned) fruit earn full income? | Yes and yes | Explosives Q10, Q12 |
| 26 | Who can break a boat? | Only flagged shooters, only flagged drivers' boats, plus the Phase 3 Fort (D-024) | Weapons Q1, Boats Q5 |
| 27 | Artillery never hurts fruit; sinking pays nothing; driver may fire? | Yes to all | Weapons Q2, Q6, Q12 |
| 28 | Weapon unlocks at stages 2 / 3 / 5 / 7; Practice Range lets you try locked ones; Rotten Fort as Phase 3? | Yes to all | Weapons Q4, Q5, Q7 |
| 29 | Are all plots bat safe zones, or only your own? Seated boat riders immune? | All plots; seated riders immune (D-014) | Bats Q1 |
| 30 | Bat tumble on hit, "Ouch" voice at low volume? | Yes and yes | Bats Q5, Q6 |
| 31 | Bat always the newest tier's colour; one tinted image? | Yes; switch to 9 images if the tint looks flat | Bats Q4, Q8 |
| 32 | Health replaces stagger in duels; 10 s blade pick phase; no ragdoll? | Yes to all | Royal Combat Q1, Q4, Q13 |
| 33 | No shards or rep for the same two duellists within 30 min? | Yes | Royal Combat Q6 |
| 34 | Invisibility blocks bat hits both ways and weapon firing, and stays in the shop? | Yes (D-030) | Potions Q11 |
| 35 | Robux: one cosmetic game pass (Storm Edge, 149 R$), nothing that changes damage, health or protection? | Yes | Royal Combat Q7, Explosives Q8, Bats Q9, Pets Q9, Potions Q13 |

**UI**

| # | Question | Recommended default | Was |
|---|---|---|---|
| 36 | Shop tabs Potions, Weapons, King, Robux, with explosives on the Weapons tab? | Yes (D-022) | Potions Q6, Explosives Q11 |
| 37 | Phone layout: one action button by the jump button (Bat, Swing or Fire), potion timers top left? | Yes (D-025) | new |
| 38 | Index category tabs Fruits, Pets, Ores on the left edge? | Yes (D-026) | new |
| 39 | Hide everything but Mute while you fight in the arena? | Yes (D-028) | King UI Q12, Royal Combat Q10 |
| 40 | Change D-001: an icon id of 0 shows no icon instead of an emoji? | Yes | King UI Q2 |
| 41 | Figma for titles and words, ChatGPT for icons? | Yes | King UI Q1 |
| 42 | One Kingdom panel with 4 tabs; only the chip plus one pill on the HUD; camera takeover off; remove the "Royal Court +income" line? | Yes to all | King UI Q3 to Q6 |
| 43 | Max Fruit popup as a modal, once per stage? | Yes | Rarity Q9 |
| 44 | Turn off Roblox's default Backpack hotbar? | Yes, once, for the whole game | Weapons Q8 |
| 45 | Text over fruit (names, +$, damage numbers) stays Highway, or switches to Fredoka? | Keep Highway; one config key switches all three | Explosives Q6 |

**World and assets**

| # | Question | Recommended default | Was |
|---|---|---|---|
| 46 | Lily pads: your original pads with the Studs material, in the island ponds only (the sea ones stay removed)? | Yes (D-021) | Plot Overhaul Q1 |
| 47 | Underground entrance: the stone hut in the back-right lot corner, with the Mine Lift inside? | Yes (D-023) | Plot Overhaul Q5, Underground Q8 |
| 48 | Plot look: hedges on three sides, gate banners in your stage colour, display name on the gate, statue fire off, tower slot reserved back left? | Yes to all | Plot Overhaul Q3, Q4, Q8, Q9, Q11 |
| 49 | Mountains: 6 full pack copies plus 66 cheaper copies (8,208 parts); logs as D-003's trunk-style studded logs (not the pack's)? | Yes and yes | Plot Overhaul Q6, Q7 |
| 50 | Docks at bearings 76, 200 and 325; 16 rest islets; friends-only passenger; no boat collisions; no repair cost; one boat type? | Yes to all | Boats Q1 to Q4, Q6, Q14 |
| 51 | Aqua Affinity 30 studs/s for 10 min, priced 75 to 500 shards; Speed does nothing in water? | Yes (D-029) | Boats Q7, Q10; Potions Q3, Q4, Q10 |
| 52 | Haste waits for the second ore band? | Yes (D-032) | Potions Q5 |
| 53 | Publish the animations (6 sword, 2 bat, 5 weapon) from `hikeALT12212`, or from your account plus one playtest? | Owner account | Royal Combat Q3, Bats, Weapons |
| 54 | Next ore bands: one 2-ore band per rarity colour, each on a deeper floor? | Yes, decide before the next ore ships | Underground Q12 |

## 9. Fixes from the player-path pass

The seven notes in `04 Player Paths` walk players through these specs minute by minute. Every friction they found is ranked in [[00 Paths Summary]]. The ones that were spec errors were fixed in place; each edited spot names the path that found it.

| # | Spec error | Fix | Changed | Found by |
|---|---|---|---|---|
| X1 | Fruit regen started 10 s after a hit, but Mega TNT's cooldown is 15 s, so its target healed between blasts (3 centre hits, not 2) | Regen delay 16 s (D-036) | [[Explosives and Fruit Health]] rule 26, N6, 9.1, 9.2, test 26 | [[Combat Day]] |
| X2 | "Lock to block it" could not be followed away from your plot: live `lockPlot` refuses off-plot | Lock from anywhere while an explosive is armed on your plot (D-034) | [[Explosives and Fruit Health]] rule 32, code plan, edge cases | [[Underground Unlock - Stage 6-7]], [[Combat Day]] |
| X3 | A fighter's garden could be hit while arena focus hides the Lock | Owners `InArena` cannot be targeted; armed explosives are returned (D-035) | [[Explosives and Fruit Health]] rules 9 and 15, toasts, edge cases | [[Combat Day]] |
| X4 | Touch action slot showed "Bat" while driving, a press B5 always refuses | Hidden while seated in a `VehicleSeat` with no weapon drawn | [[Bats]] UI 3 | [[Mobile Player]] |
| X5 | `BatUnlocked` fired in the same frame as `RebirthDone` (and with the join notices) on a one-label toast | Sent 3 s after `RebirthDone`; on join after the other notices | [[Bats]] B30 | [[New Player - First Session]], [[Existing Player Migration]] |
| X6 | A passenger was stranded in a driverless boat when the owner died or was teleported to a duel | Boat despawns 10 s later; R51 moves the passenger to a dock | [[Boats and Swimming]] R50, edge cases | [[Combat Day]] |
| X7 | P9 recomputed requests only when the best tier changed, but test T4 expects a change inside a tier | Recompute for unfed animals whenever `bestRank` changes | [[Pets]] P9 | [[New Player - First Session]] |
| X8 | "Tamed a %s %s!" prints "a Epic" | "New pet: %s %s!" | [[Pets]] U7, player view | [[Mid Game - Stage 4 Epic]] |
| X9 | Potions player view showed 8 cards with Haste, but Haste is not in the shop until the second ore band; the mobile edge case still put the effect block by the jump button | Player view and edge case corrected | [[Potions]] | [[New Player - First Session]], [[Mobile Player]] |
| X10 | At stage 9 the Luck refusal says "after your next rebirth", the card still sells Luck, and the Max Fruit popup, Merge All toasts and Spawn Rarity card all say "rebirth" | Stage 9 variants of every one of these | [[Rarity Gating and Rebirth]] rule 10, U5, U7, U10, code plan; [[Potions]] rule 26, UI 1 | [[Endgame - Transcendent]] |
| X11 | Migration notice sent at the same delay as the refund notice; Spawn Rarity refund was silent | Notice at refund delay + 4 s; refund named in the notice | [[Rarity Gating and Rebirth]] rule 22, U10, code plan | [[Existing Player Migration]] |
| X12 | "Merge ores to make room" shown when all 10 ores are Iron, which cannot merge | Separate message for a floor full of the top ore | [[Underground and Ores]] rule 12, player view, U14 | [[Underground Unlock - Stage 6-7]] |

