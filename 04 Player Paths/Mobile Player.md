---
tags: [player-path]
date: 2026-10-04
---
# Mobile Player

Noah is 10 and plays on a phone held sideways. Its screen is 667 x 375 points (iPhone SE size), the smallest screen the specs check. He does not own the Merge All game pass, so he merges by hand, about one merge every 6 seconds. This path follows the same core loop as the PC paths, in three sessions: the first 22 minutes, a boat and taming session at stage 3, and a mining session at stage 8.

Every rule cited is in the spec notes in [[02 Systems]]. Screen rectangles are from the unified HUD table in [[Integration Review]] section 4. Each friction is ranked in [[00 Paths Summary]].

**Words used here**
- **HUD scale**: every HUD button is drawn at a fraction of its 1600 x 900 size. On this phone it is 0.55, the floor (`Config.HudScaleFor`).
- **Touch target**: the area a thumb can press. Apple's guideline is 44 x 44 points; the specs use 44 px as their minimum (for example [[Boats and Swimming]] 6.3, [[Underground and Ores]] U1).
- **Touch action slot**: the one button left of the jump button. It shows Fire, Swing or Bat, whichever applies (D-025).
- **Rect**: a rectangle on screen, written x range, y range, in points from the top left.

**How the times were made.** Times are hours and minutes of his total play (h:mm). With 6-second merges, `plateau_wait.py` gives 11.8 hours to Transcendent, about 1.2 hours more than a 3-second merger, nearly all of it in stages 3 to 8. Stage 1 takes the same 15 minutes either way, because cash, not merging speed, sets its length.

---

## Session A: the first 22 minutes (stage 1 to 2)

### 0:00 The HUD at 0.55
- **Sees:** Inventory, Index, Shop and Pets down the left edge (each about 53 x 51, the column lifted 13 points so Pets clears the cash corner). Rebirth, Merge All and Clear on the right. The Lock and Upgrade pills at the top centre. The royal chip top left. Bottom left, the cash, shards and boost bars. Left of the jump button, the touch action slot shows a grey-blue bat with a "!". There is no bat slot in the bottom row on a phone.
- **Rule:** [[Integration Review]] section 4; [[Pets]] U1 (the guard); [[Bats]] UI 1 and 3; D-025.
- PAYOFF: every new element has a place. The touch action slot (rect 478-570, 247-339) sits clear of the jump button (572-642, 285-355) and the right column.

### 0:01 Picking up and dropping
- **Sees:** fruit prompts are tappable. When he carries a fruit, the carry row appears at the bottom centre: Drop and Store, each about 92 x 31 points (rect 238-428, 333-364).
- **Decides:** taps Drop dozens of times over the next few minutes, and misses about one in ten.
- **Rule:** existing carry row, scaled by HudScale.
- FRICTION: **P5-1 (medium)** The two most-tapped buttons in the game (Drop and Store) are 31 points tall here, and the Lock and Upgrade pills are 27. Both are under the 44-point minimum the specs use elsewhere. The Lock pill is now also the explosive defence, with 3 to 4 seconds to react. **Fix:** on touch, give the carry row and the top pills a minimum height of 44 points (HudScale floors the carry buttons' scale at 0.79 and the pills' at 0.88), and move CarryInfo up to keep its 8-point gap; SeaStatus already places itself by measuring the hands stack. Check with the [[King UI Overhaul]] `UIOverlapCheck`. (spec: [[Integration Review]] section 4; [[King UI Overhaul]] UIKit)

### 0:03 The Max Fruit card
- **Sees:** the card opens at its minimum size, 340 x 190, centred. The green REBIRTH pill is about 129 x 45.
- **Rule:** [[Rarity Gating and Rebirth]] U5 (min 340 x 190).
- PAYOFF: the most important explanation in the first session fits a small phone with no clipping.

### 0:06 The right column
- **Sees:** he reaches for Rebirth and the game pass purchase box for Merge All opens instead.
- **Rule:** [[Integration Review]] section 4: Rebirth 93-144 and Merge All 137-188 on this phone (and 278-370 and 359-451 at 1600 x 900), so the two rects share 7 points.
- FRICTION: **P5-2 (low)** The table that every spec checks against lists overlapping rects for the right column, and nobody flagged them. If the art's transparent margins are part of the buttons, a tap near the edge can hit the wrong one, and one of them opens a Robux purchase. **Fix:** measure in Studio; if the hit areas overlap, either trim the buttons' hit areas to the visible art or space the centres 102 points apart like the left column, and add the right column to `UIOverlapCheck`. (spec: [[Integration Review]] section 4)

### 0:08 Swimming
- **Sees:** at sea three things share the bottom: the Stamina bar (273-393, 246-287), the Weapon Bar's empty state with a flag slot and a Shop slot (210-456, 307-364) and the Bat button (478-570, 247-339). None overlap. The green notice line lands above the Stamina bar (y 197-225). The jump button climbs him onto stones.
- **Rule:** [[Boats and Swimming]] 6.1 (placement rule), R11; [[Weapons and Artillery]] UI 2.
- PAYOFF: the gap rule from the [[Integration Review]] (C23) works: the Stamina bar never sits in the notice lane.
- **Note:** the empty Weapon Bar is clutter for a stage 1 swimmer with no weapons (P1-5 in [[New Player - First Session]]).

### 0:09 Moving
- **Sees:** Roblox's thumbstick appears wherever his left thumb lands on the lower left. That is also where the cash corner and its "+" invite button sit (rect 9-131, 278-366).
- FRICTION: **P5-3 (low)** A thumb that lands on the "+" opens the invite dialog instead of moving. This is in the live game today, not new. **Fix:** on touch, move the invite "+" out of the cash corner, or hide it. Measure first. (spec: existing CornerHud; [[UI Kit Inventory]])

### 0:17 First rebirth
- **Sees:** the same Rebirth menu as on PC, in the Index shell. The REBIRTH pill keeps its art proportions.
- **Rule:** [[Rarity Gating and Rebirth]] U4, edge case "Mobile".
- PAYOFF: a slow merger loses no time in stage 1. Stage 1 is 15 minutes at 3 or 6 seconds per merge.

---

## Session B: boat, taming and weapons (stage 3, about 1:20 in)

### 1:20 Summoning the boat
- **Sees:** at the dock he taps the "Summon Boat" prompt. He is seated at the wheel. The "Boat" bar shows bottom centre. The thumbstick drives the boat: up for throttle, sideways to steer, through Roblox's default vehicle controls. His right thumb turns the camera.
- **Rule:** [[Boats and Swimming]] R27, R29, R35, edge case "Mobile".
- PAYOFF: driving needs no new controls. A 10-year-old who has driven any Roblox car can drive this boat.

### 1:21 The Bat button while driving
- **Sees:** the touch action slot still shows the Bat button. He taps it. Nothing happens: the server refuses a bat in a `VehicleSeat` (B5) and says nothing.
- FRICTION: **P5-4 (medium)** A button that is always refused. Spec error, fixed in [[Bats]] UI 3: the slot is now hidden while the player is seated in a `VehicleSeat` with no weapon drawn.

### 1:22 Jump
- **Sees:** halfway across, he taps jump out of habit (he jumps all the time on land). He is out of the seat and in the sea, 250 studs from either shore. The boat stops and sits empty.
- **Rule:** [[Boats and Swimming]] R34 (jump leaves the seat; away from a pad you drop into the water), R50.
- FRICTION: **P5-5 (high)** On a phone the jump button is the most-pressed button, so the boat's exit is one habitual tap away. He swims back to the boat and has to be in the driver seat again; only the owner can drive, and "Drive" is a prompt he must find on the bobbing boat. **Fix:** on touch, jump leaves the boat only when it is within 30 studs of a pad or slower than 6 studs/s; otherwise holding jump for 0.5 s leaves, with a small "Hold to leave" hint in the boat bar's name strip the first time. (spec: [[Boats and Swimming]] R34)

### 1:25 Taming
- **Sees:** at the forest he taps and holds the "Feed" prompt for 0.6 s. The request card is readable from 45 studs.
- **Rule:** [[Pets]] P12, U4, edge case "Mobile".
- PAYOFF: hold-to-feed works on touch with no extra button.

### 1:30 Pets panel
- **Sees:** the Pets panel in the Index shell. The slot cards are about 70 points wide. The Release button (the kit No art at 0.6 scale) is small, about 30 points.
- **Rule:** [[Pets]] U2.
- **Note:** a small Release button is acceptable here because Release always asks to confirm (U3).

### 1:40 Firing from the boat
- **Sees:** he owns Cherry Bombs and Melon Mortar shells (stage 3). He taps the Mortar slot (44 x 44) on the Weapon Bar. The Bat slot becomes a Fire button and a reticle appears at the screen centre. To aim, he must turn the camera with his right thumb; to fire, hold Fire and release; to steer, keep his left thumb on the stick.
- **Rule:** [[Weapons and Artillery]] UI 3, UI 4 (touch: aim with the camera centre, hold and release Fire), UI 5.
- FRICTION: **P5-6 (medium)** Aiming, firing and steering need three thumbs. He stops the boat to shoot, which makes him an easy target. **Fix:** on touch, Fire fires on a tap at the reticle (no hold), and the arc preview shows the whole time a weapon is drawn, so the camera drag is the only aiming input. (spec: [[Weapons and Artillery]] UI 4)

---

## Session C: mining (stage 8, about 9:40 in)

### 9:40 The hut
- **Sees:** he taps "Go Down" in the hut. The fade covers the screen for about a second. The Mine panel (from the Mine Board) opens at 92% of the screen width so its buttons stay at least 44 points tall.
- **Rule:** [[Underground and Ores]] U1, U12, U13.
- PAYOFF: the one new panel that would have been cramped was widened on purpose.

### 9:41 Mining
- **Sees:** next to a rock with two ores lying near it, he sees three prompt buttons at once: "Mine" on the rock and "Pick Up" on each ore. He taps "Mine": the pickaxe chops by itself until the rock breaks.
- **Rule:** [[Underground and Ores]] rules 10, 11, U13 (nodes use F, ores use E, so both show).
- PAYOFF: one tap mines a whole node. Mining is easier on a phone than fruit merging.
- FRICTION: **P5-7 (low)** With 6 nodes and up to 10 ores in a small cave, prompts stack on a small screen and a tap meant for "Pick Up" can start mining. **Fix:** on touch, hide ore prompts while mining runs, and show only the nearest ore's prompt otherwise. (spec: [[Underground and Ores]] U13)

### 9:45 Carrying ores
- **Sees:** an ore over his head and the same 31-point carry row (P5-1).

### 9:50 A TNT warning in the mine
- **Sees:** red warning: "Rex placed a TNT in your garden! Lock to block it." He has to find the 27-point Lock pill at the top centre within 3 seconds.
- **Rule:** [[Explosives and Fruit Health]] rules 13, 14, 32 (with the fix, the Lock works from the mine, D-034).
- FRICTION: **P5-8 (medium)** On a phone the fastest defence in the game is its smallest button. **Fix:** while an explosive is armed on your plot, the touch action slot shows a Lock button (above Fire, Swing and Bat in its order), using the Lock pill's padlock art. (spec: [[Bats]] UI 3; [[Explosives and Fruit Health]] 6.4)

### 10:00 Frame rate
- **Sees:** in the cave the game runs smoothly: cave decor is built only for the floor he is on. Back on the surface, near his plot with 6 Giant pets and his neighbours' pets, and later in the forest, the frame rate dips.
- **Rule:** [[Pets]] Performance (budget: 24 animated rigs and 1,200 joint writes per frame, checked on PC only, T17); [[Underground and Ores]] 7.4.
- FRICTION: **P5-9 (medium)** Every performance budget in the specs is set and tested on a PC. A low-end phone draws the same 1,050 pet parts and the same joint writes. **Fix:** on touch devices, halve PetWorld's caps (12 rigs, 600 joint writes, full rate within 25 studs instead of 40) and add a real low-end phone to tests T17 ([[Pets]]) and T-Perf ([[Boats and Swimming]]). (spec: [[Pets]] Performance)

### 10:05 The Index on a phone
- **Sees:** the Index with its three category tabs (Fruits, Pets, Ores) tucked behind the panel's left edge.
- **Rule:** [[Underground and Ores]] U4 ("Test at 844 x 390 and 667 x 375 ... Fallback if they touch the Inventory/Index/Shop column").
- FRICTION: **P5-10 (low)** The spec leaves this to a test. On this screen the panel's left edge is close to the left column, so the tucked tabs are likely to touch it. **Fix:** on touch, use the spec's own fallback from the start: the three tabs as a row inside the panel, at 60% size. (spec: [[Underground and Ores]] U4)

---

## What this path found

| Id | Severity | Friction | Spec |
|---|---|---|---|
| P5-5 | high | Jump leaves the boat mid-sea | [[Boats and Swimming]] |
| P5-1 | medium | Carry row and top pills under 44 points | [[Integration Review]], [[King UI Overhaul]] |
| P5-4 | medium | Bat button shown while driving, always refused (spec error, fixed) | [[Bats]] |
| P5-6 | medium | Aim, fire and steer need three thumbs | [[Weapons and Artillery]] |
| P5-8 | medium | The Lock, now a 3-second defence, is the smallest button | [[Bats]], [[Explosives and Fruit Health]] |
| P5-9 | medium | No phone performance budget | [[Pets]], [[Boats and Swimming]] |
| P5-2 | low | Right column rects overlap in the HUD table | [[Integration Review]] |
| P5-3 | low | Invite "+" under the thumbstick (live today) | CornerHud |
| P5-7 | low | Prompt stacking in the mine | [[Underground and Ores]] |
| P5-10 | low | Index tabs likely to touch the left column | [[Underground and Ores]] |

Spec errors fixed by this path: [[Bats]] UI 3 (no Bat button while driving); [[Potions]] edge case "Mobile" (it still placed the effect block by the jump button, which D-025 moved to the top left).

Payoffs: a HUD where every new element has a place at 0.55, the Max Fruit card at its minimum size, the Stamina bar's gap rule, no stage 1 penalty for slow mergers, standard boat controls, hold-to-feed, a widened Mine panel, and one-tap mining.
