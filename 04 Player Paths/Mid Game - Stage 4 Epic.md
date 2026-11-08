---
tags: [player-path]
date: 2026-10-04
---
# Mid Game - Stage 4 Epic

Maya is 14 and plays on a PC. She has rebirthed 3 times, so she is in stage 4 (Epic). She comes back the next day for one 70-minute session: she finishes her stage 4 plot, uses a potion, takes her boat to the forest, tames an Epic animal, trades TNT with a rival, and decides when to rebirth.

Every rule cited is in the spec notes in [[00 Systems Index|02 Systems]]. Each friction is ranked in [[00 Paths Summary]].

**Words used here**
- **Stage 4**: Epic unlocked (3 rebirths). Cap fruit Watermelon ($59.4B/s each). Spawn ceiling Pumpkin. Honeydew and Watermelon can only be merged, never grown (D-007).
- **Full plot**: 35 cap fruit times the rebirth bonus: $3.64T/s at stage 4, before the Index bonus ([[Stage Model]]).
- **Plateau**: the plot is full of cap fruit. Nothing new spawns (the plot is full), nothing can merge (cap fruit do not merge) and Clear is refused (it will not clear a plot where every fruit is the same kind, `Merging.lua:396-399`).
- **Last merge**: the last merge the player can make before the plateau.
- **Effective stage**: for explosives, the lower of the attacker's and the garden owner's stage ([[Explosives and Fruit Health]] rule 20).

**How the times were made.** Times are minutes of the session (h:mm). Stage 4 events come from `~/fruit-empire/sim/integration_check.py` (the resolved design: pets, forest trips, 3 s merges): the last merge comes 36.5 minutes into stage 4 and the rebirth at 73.0 minutes. `~/fruit-empire/sim/plateau_wait.py` (new, written for this pass) measures the wait for every stage.

**Starting state**: 25 minutes into stage 4. Spawn Rarity at Mini Pumpkin (2 levels under the ceiling). Index 30/32. Cash $1.4Qa of $10.9Qa. 7,900 shards (the need is 3,500). Boat owned since stage 2. 4 pet slots (3 rebirths) with a Big Rare Sheep, a Baby Rare Deer, a Huge Uncommon Turtle and a Baby Rare Hyena. Bag: 1 Luck, 2 Aqua Affinity, 1 Speed. Storage 0/8.

---

### 0:00 Joins
- **Sees:** the "Welcome back" card: offline earnings for 5 hours (the cap) at 3% of her live income, about $0.5Qa, under 5% of the rebirth cash. Her plot is restored from its saved counts. Her bat is purple (Epic, Power 56).
- **Decides:** dismisses it.
- **Rule:** offline pay ([[Core Server - Plots, Spawning, Income]]: 3%, 5 hours); [[Pets]] P28 (pets count offline); [[Bats]] B2.

### 0:01 Checks the Rebirth menu
- **Sees:** NEEDS: Index 30/32 in red, cash in red, Shards 7.9K / 3.5K with a tick. The left box: "Rebirth 4 unlocks 7 Legendary fruits". Second footer: "Also unlocks: Coconut Cannon".
- **Decides:** her plan is to make Honeydew and Watermelon, then wait for cash.
- **Rule:** [[Rarity Gating and Rebirth]] rule 12, U4.
- PAYOFF: one glance tells her exactly what is left.

### 0:02 Luck
- **Sees:** in Inventory the Luck card has a Drink button. She taps it. A gulp, a gold puff over her head, a gold badge bottom right with "5:00".
- **Decides:** drinks it now, while Spawn Rarity is still 2 levels under the ceiling.
- **Why:** last stage she learned that Luck is refused once Spawn Rarity is maxed.
- **Rule:** [[Potions]] rules 12, 26; [[Rarity Gating and Rebirth]] rule 10.
- PAYOFF: the next fruit up (Pumpkin) starts spawning with a gold glint. The [[Potions]] model says Luck drunk early saves about 2.5 to 3.4 minutes.

### 0:05 Spawn Rarity reaches the ceiling
- **Sees:** she buys the last two Spawn Rarity levels. The card turns to LOCKED, "Up to Pumpkin". Luck's badge still shows 2:10 left.
- **Rule:** [[Potions]] rule 26 (Luck is capped at the ceiling).
- FRICTION: **P2-2 (low)** The last 2 minutes of Luck now do nothing, and the badge keeps counting as if they did. **Fix:** while Spawn Rarity is at the ceiling, Luck's timer pauses and the badge reads "Paused"; the seconds left resume after the next rebirth. This also fits the refusal line "Save it for after your next rebirth!". (spec: [[Potions]] rule 26)

### 0:07 to 0:11 The last discoveries
- **Sees:** a Honeydew, then a Watermelon, each with a sparkle and a purple "+32". The MAX FRUIT card opens: "Watermelon is your best fruit", "Rebirth 4 unlocks 7 Legendary fruits". The Index reads "Found 32/32".
- **Rule:** [[Rarity Gating and Rebirth]] rules 7, 18, U5.
- PAYOFF: the Index tick turns green. Only cash is left.

### 0:12 The plot fills
- **Sees:** the plot is 35 fruit, almost all Watermelons. Within two minutes the last pair merges. Merge All answers "Nothing can merge. Watermelon is your best fruit until you rebirth." Clear is refused because every fruit is the same kind. Nothing spawns because the plot is full. The cash row says $2.2Qa of $10.9Qa (the simulation has about 20% of the need banked at the last merge).
- **Decides:** she has about 36 minutes to wait.
- **Rule:** [[Rarity Gating and Rebirth]] rule 6; [[Stage Model]] (plateau).
- FRICTION: **P2-1 (critical)** The plateau wait. The cash need for stage 4 is about 50 minutes of full-plot income, but the plot is full 36.5 minutes into the stage, so the second half of the stage has no merging at all. The same holds for every later stage, and it grows:

| Stage | Length (min) | Last merge at (min) | Waiting with nothing to merge (min) |
|---|---|---|---|
| 3 | 56.8 | 32.8 | 24.0 |
| 4 | 73.0 | 36.5 | 36.5 |
| 5 | 90.9 | 36.5 | 54.4 |
| 6 | 102.1 | 40.2 | 61.9 |
| 7 | 117.8 | 34.4 | 83.4 |
| 8 | 145.4 | 36.1 | 109.3 |

  Over stages 1 to 8 that is 6.3 of 10.6 hours (59%) for the attentive player and 5.3 of 11.8 hours (45%) for a 6-second merger (`plateau_wait.py`, median of 7 careers). Logging off does not help: offline pay is 3%. Standing still does not work either: Roblox disconnects a player after 20 minutes with no input (platform behaviour, not measured in the vault). So the design asks players to fill half their career with side activities, or to leave. Most of the frictions below happen inside this wait, while Maya tries to fill it. **Fix (pick one, Samarth's call):** (a) re-solve D-011's cash needs for stages 3 to 8 with `calibrate.py` so the wait after the last merge is about 15 minutes; stages become about 48 to 55 minutes and the career about 6 hours of play; shard flows do not change, because every frontier merge happens before the plateau anyway; or (b) keep D-011 and, while the plot is full of cap fruit, raise offline pay from 3% to 50% (capped at the rebirth cash need), so the wait becomes "come back later". (spec: [[Stage Model]], D-011, [[Rarity Gating and Rebirth]])

### 0:15 First trip, nothing to give
- **Sees:** she walks to the nearest Boat Dock (about 10 s), presses E "Summon Boat". The boat splashes onto the pad and she is at the wheel. A "Boat" bar at full green appears bottom centre. She drives 14 seconds to the forest dock and walks to the nearest clearing. The cards: an Epic Giraffe wants "Mini Pumpkin 0/2", an Epic Sheep "Mini Pumpkin 0/2", a Rare Hyena "Grape 0/1".
- **Decides:** goes back for fruit.
- **Why:** her storage is empty and her plot only has Watermelons.
- **Rule:** [[Boats and Swimming]] R27 to R29, R36; [[Pets]] P7, P9 (requests are your best fruit plus 3, 5 or 7 ranks), U4.
- PAYOFF: the boat. A crossing that was 65 seconds of swimming at stage 1 is now 14 seconds, with a wake and a studded sail in her rarity colour.
- FRICTION: **P2-3 (high)** At the plateau every request asks for a fruit that her plot can no longer grow: the plot is full of Watermelons, so nothing new spawns. The only way is to store cap fruit to free slots and wait for random spawns. **Fix:** a feed accepts the requested fruit or any better unlocked fruit, so a Watermelon pays any request at stage 4; plus the "Wanted" line in the Pets panel (P1-6) so players can store requests before the plateau. (spec: [[Pets]] P13 (g), P14)

### 0:17 Back home, the workaround
- **Sees:** she summons the boat at the forest dock (the old one is destroyed), drives home, picks up two Watermelons and presses Store. Two slots open. New fruit drop in every few seconds: Pumpkin, Loquat, Mini Pumpkin, Pumpkin, Mini Pumpkin, Grape. She stores two Mini Pumpkins and a Grape.
- **Rule:** [[Boats and Swimming]] R29; [[Pets]] P14; spawn odds ([[Config and Fruit Data]]: each step down the pool weighs 0.6, so Mini Pumpkin is about 1 spawn in 4).
- **Cost:** about 2.5 minutes and two trips she should not have needed.

### 0:21 First Epic pet
- **Sees:** she feeds the Giraffe twice and the Hyena once. "New pet: Epic Giraffe!" The old string printed "Tamed a Epic Giraffe!".
- **Rule:** [[Pets]] P12 to P15, U7.
- FRICTION: **P2-4 (low)** "Tamed a %s %s!" read "a Epic" and "a Uncommon". Spec error, fixed in [[Pets]] U7 ("New pet: %s %s!").
- PAYOFF: the Giraffe. It is the first Epic species (it only exists from stage 4), it walks with a pacing gait, and it bends its neck down to drink.

### 0:22 Knocked off an animal
- **Sees:** while she holds E on the Giraffe's second feed, another player's bat sends her 20 studs back. The prompt lets go. Nothing is used up.
- **Decides:** walks back and feeds again.
- **Rule:** [[Bats]] B24 (bats work in the forest); [[Pets]] edge cases.
- FRICTION: **P2-5 (low)** Wild animals are private, so there is nothing to fight over in a clearing, yet bats work there and can cancel a feed. **Fix:** tag each clearing's inner circle (`Radius` minus 8 studs) `BatSafeZone`. (spec: [[Bats]] B22 (b))

### 0:24 Pets panel
- **Sees:** 6 pets for 4 slots. The header reads "Equipped 4/4" and "Pets $110B/s". She presses Equip Best. Her Rare pets now read "1 tier behind: counts as Baby".
- **Decides:** merges her two Baby Rare Deer into a Big Rare Deer, then Equip Best again.
- **Rule:** [[Pets]] P17, P18, P24, P26, U2.
- PAYOFF: one button picks the best four, and every card says in words what it is worth ("Pays like 0.38 of a Watermelon").

### 0:26 TNT on a rival
- **Sees:** Shop, Weapons tab: TNT 480, Big TNT 1,300, Cherry Bomb 380, Melon Mortar 640; Coconut Cannon reads "Rebirth 4", Mega TNT "Rebirth 5". She buys a Big TNT. Equipped, it sits over her head; the carry bar reads "Big TNT", "Garden | Radius 14, 2 hits".
- **Decides:** walks onto Leo's garden. Leo is stage 5. She presses Drop.
- **Rule:** [[Explosives and Fruit Health]] rules 2, 3, 6 to 14, 6.1, 6.3.
- **Result:** a red circle, a 3-2-1 countdown, a fuse hiss. Leo, on his plot, taps Lock in time. "Blocked by a Lock. It went back in your bag."
- PAYOFF (for Leo): the warning, the red "!" on his Lock pill and 3 seconds are enough to defend.

### 0:28 The second try
- **Sees:** Leo's dome drops after 60 seconds. She places the Big TNT during his 30-second recharge. It hits 6 fruit: "-98" at the centre. Leo's Legendary fruit drops from 244 to 146 HP. Green line: "Hit 6 fruit. 0 stunned." Sixteen seconds later his bars refill.
- **Rule:** [[Explosives and Fruit Health]] rules 19, 20 (her stage 4 sets the damage), 22, 26.
- FRICTION: **P2-6 (medium)** She spent 1,300 shards (about 9 minutes of stage 4 shard income) for no effect: against a higher-stage garden her damage uses her own stage, and nothing told her before she bought or placed it. **Fix:** while she carries an explosive and stands on a target garden, the carry rate bar reads the real numbers for that garden: "98 of 244 HP at the centre". (spec: [[Explosives and Fruit Health]] 6.3)
- FRICTION: **P2-7 (low)** Placing the first TNT would have dropped her own Lock if it had been up, and the armed toast does not say so. **Fix:** `ExplosiveArmed` adds "Your Lock is down." when it was up. (spec: [[Explosives and Fruit Health]] rule 12, 6.5)

### 0:31 Leo answers
- **Sees:** red warning: "Leo placed a Big TNT in your garden! Lock to block it." A red "!" on her Lock pill. She is standing on her plot. She taps Lock. Cyan dome. Green line: "Your Lock blocked Leo's Big TNT!"
- **Rule:** [[Explosives and Fruit Health]] rules 14, 32.
- PAYOFF: a clean, readable defence.
- **Then:** Leo waits out her Lock and hits during her recharge. 7 fruit hit, 2 Watermelons stunned (98 + 98 against 195 HP, with no regen between blasts 10 s apart). They turn see-through with circling stars for 20 seconds, then come back full with a cyan bar. It costs her about 40 seconds of two fruit's income. "Leo's Big TNT hit 7 of your fruit. 2 stunned."

### 0:36 A crate she cannot reach
- **Sees:** "A Golden Crate is falling on the Cloud Garden". The Cloud Garden is on the sky islands; above y 418 she needs the Aether potion, and hers ran out last week.
- **Decides:** ignores it.
- **Rule:** [[Bats]] C2, C3; thin air ([[Client World, Royal Client, Music and Ambient]]).
- FRICTION: **P2-11 (low)** Two of the eight crate spots are in the sky, where most players cannot go without a 1,600-shard potion, but the toast goes to everyone. **Fix:** drop a crate on a sky spot only when at least one player who owns a plot is on the sky islands, and send its toast only to players who can reach it. (spec: [[Bats]] C2, C3)

### 0:38 to 0:47 Practice Range and a contest
- **Sees:** on the Practice Range pad by the main dock, the Weapon Bar shows all four sea weapons, free. She tries the Coconut Cannon she will unlock next. Then a Crown Contest opens; she joins from the event strip (entry pays 4 units, 128 shards at stage 4) and is knocked out in the brawl.
- **Rule:** [[Weapons and Artillery]] rules 18, 31; [[Royal Combat]] rule 48, Numbers.
- PAYOFF: she feels the Cannon before she can buy it. "Rebirth to unlock more" becomes something in her hands.

### 0:48 The rebirth decision
- **Sees:** a red "!" on Rebirth: cash $10.9Qa reached. Shards 8,000. The Pets panel shows a locked slot card "10K Shards". She has 6 pets for 4 slots.
- **Decides:** rebirth now, buy the slot later.
- **Why:** the slot (10,000) and the rebirth (3,500) cost more together than she has. After the rebirth, Legendary merges pay 70 shards each, so the slot comes quickly in stage 5. The price of the slot is fixed, so waiting costs nothing.
- **Rule:** [[Rarity Gating and Rebirth]] rules 12, 15, 18; [[Pets]] P20, U3.
- FRICTION: **P2-9 (low)** Her reasoning depends on a number the game never shows: stage 5 merges pay 70 shards, not 32. **Fix:** a REWARDS footer line in the Rebirth menu: "Legendary merges pay 70 shards". (spec: [[Rarity Gating and Rebirth]] U4)
- FRICTION: **P2-10 (medium)** Had she bought the slot first, its confirm ("Buy a pet slot for 10,000 shards?") would not have warned her that she could no longer rebirth. Only the Mine's confirm states the shards left and the next rebirth's shard need. **Fix:** every shard purchase of 1,000 or more uses the same second line: "You will have 6,500 shards left. Your next rebirth needs 3,500." One helper in GardenConfig, used by pet slots, the boat, floors, skins and Shop items. (spec: [[Pets]] U3; [[Boats and Swimming]] 6.3; [[Royal Combat]] Armory; [[Rarity Gating and Rebirth]] owns the helper)

### 0:49 Rebirth 4
- **Sees:** the confirm: "Resets cash, upgrades, your plot and stored fruit. Keeps your Index, shards, tools, pets and Plot Size." Her 2 stored Watermelons and the Grape go. Burst, orange banners, "Rebirth 4 complete. Legendary fruit unlocked, cash x2." Three seconds later: "Bat unlocked: Legendary Bat (Power 64, Speed 72)". The Weapons tab lights the Coconut Cannon.
- **Rule:** [[Rarity Gating and Rebirth]] rules 11, 15, 16; [[Bats]] B30; [[Weapons and Artillery]] rule 2.
- PAYOFF: three visible upgrades in one moment: banners, bat and a new weapon.
- FRICTION: **P2-8 (medium)** Her pet income falls from about $120B/s to about $380M/s, because pet money follows her best fruit and the plot now starts at Lemon ($163M/s against Watermelon's $59.4B/s). It climbs back as she merges, but nothing tells her that, and it looks like the rebirth broke her pets. **Fix:** one more line in the confirm dialog when she owns pets or ores: "Pet and ore income follow your best fruit, so they start small again." (spec: [[Rarity Gating and Rebirth]] U4; [[Pets]] P30)

### 0:50 to 1:05 Stage 5
- **Sees:** the plot climbs from Lemon. Her Epic pets count fully again (no tier gap until her first Legendary fruit). The first Cacao arrives with a purple "+70".
- **Rule:** [[Pets]] P25, P26, Table 4; [[Rarity Gating and Rebirth]] rule 11.
- PAYOFF: pets pinned to the best fruit give the start of each stage a real push, as the [[Pets]] note planned.

### 1:06 The slot
- **Sees:** 10,100 shards. She presses the locked slot card: "Buy Slot? Buy a pet slot for 10,000 shards?" Yes. "Pet slot bought. You have 5 slots."
- **Rule:** [[Pets]] P19, P20.
- PAYOFF: all her pets fit for the first time.

### 1:10 Logs off
Cash and shards keep climbing toward the stage 5 needs ($7.81Qi, 7,500 shards). The plateau will arrive about 36 minutes into stage 5, and then she will wait 54 minutes (P2-1).

---

## What this path found

| Id | Severity | Friction | Spec |
|---|---|---|---|
| P2-1 | critical | Half the career is waiting on a full plot with nothing to merge | [[Stage Model]], [[Rarity Gating and Rebirth]] |
| P2-3 | high | At the plateau, animals want fruit the full plot cannot grow | [[Pets]] |
| P2-6 | medium | TNT on a higher-stage garden does nothing and the attacker cannot tell | [[Explosives and Fruit Health]] |
| P2-8 | medium | Pet income collapses after a rebirth with no explanation | [[Rarity Gating and Rebirth]], [[Pets]] |
| P2-10 | medium | Only the Mine warns that a purchase leaves too few shards to rebirth | [[Pets]], [[Boats and Swimming]], [[Royal Combat]] |
| P2-2 | low | Luck keeps counting after the ceiling | [[Potions]] |
| P2-4 | low | "Tamed a Epic" (spec error, fixed) | [[Pets]] |
| P2-5 | low | Bats cancel feeds in private clearings | [[Bats]] |
| P2-7 | low | Placing TNT drops your Lock silently | [[Explosives and Fruit Health]] |
| P2-9 | low | The next stage's shard drop is never shown | [[Rarity Gating and Rebirth]] |
| P2-11 | low | Sky crates announced to players who cannot reach the sky | [[Bats]] |

Spec errors fixed by this path: [[Pets]] U7.

Payoffs: the Rebirth menu at a glance, Luck drunk early, the boat crossing, the first Epic species, Equip Best and plain-words pet cards, a Lock that defends in 3 seconds, the Practice Range, and the rebirth that changes banners, bat and Shop in one moment.
