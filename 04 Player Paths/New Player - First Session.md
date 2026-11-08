---
tags: [player-path]
date: 2026-10-04
---
# New Player - First Session

Ava is 11. She plays on a PC with a mouse and keyboard. She has never played Fruit Empire. She joins an 8-player server and plays for 40 minutes: her plot, her first merges, the Apple cap, the Rebirth menu, her first rebirth and her first Uncommon fruit. She also tries one trip to the forest.

Every rule cited is in the spec notes in [[00 Systems Index|02 Systems]]. Each friction is ranked in [[00 Paths Summary]].

**Words used here**
- **Stage**: the newest rarity a player has unlocked. Stage 1 is Common (0 rebirths).
- **Cap fruit**: the best fruit of the stage. Two cap fruit cannot merge until the next rebirth. At stage 1 it is Apple.
- **Spawn ceiling**: the best fruit the plot can grow. At stage 1 it is Carrot. Tomato and Apple must be merged.
- **Frontier merge**: a merge that makes a fruit of your newest rarity. It pays shards: 3 at stage 1, 7 at stage 2.
- **Notice**: the green one-line toast at the bottom. **Warning**: the red one-line toast at the top. Each has one label; a new line replaces the old one.
- **FRICTION**: a moment of confusion, dead time, unfairness or bug risk, with a fix. **PAYOFF**: a moment that should feel great.

**How the times were made.** Times are minutes and seconds of play (m:ss) since she joined. They come from a stage 1 and 2 simulation built on `~/fruit-empire/sim/economy.py` (6 s per merge, a 3-minute trip away from the plot, the two Plot Size buys), with about 1 to 2 minutes added for a first-time player learning the controls. The simulation put the first Apple at 0:56 to 1:52 and the rebirth at 16:20 to 17:45 across 7 seeds.

**Starting state**: 0 rebirths, $0, 0 shards, 3 Blueberries on a level-0 plot, a Common bat, storage 0/5.

---

### 0:00 Joins
- **Sees:** her plot: studded hedges, a stone gate with grey-blue banners and her name on the plaque ([[Plot Overhaul and Environment]] D1, D2). Three Blueberries on the soil. On the HUD: Inventory, Index, Shop and Pets on the left; Rebirth, Merge All and Clear on the right; Lock and Upgrade pills at the top; cash, shards and boost bottom left; the royal chip top left ("King" and a name). A bat slot sits bottom centre-right with a red "!". A green notice: "Bat unlocked: Common Bat (Power 32, Speed 60)".
- **Decides:** nothing yet. She walks around.
- **Why:** nothing on screen says what the game is.
- **Rule:** [[Bats]] B1, B26 (20 s join shield), B30; [[King UI Overhaul]] rule 11 (chip zone).
- FRICTION: **P1-1 (high)** The first thing the game says is about a bat. No spec has a first-session guide, and the HUD shows 12 buttons and counters at once. The brain dump warns that "for players its going to be confusing". **Fix:** a calm goal line for brand-new accounts only, one green notice per step, at most four steps, ending at the first Apple: "Walk to a fruit and press E to pick it up", "Press E on a matching fruit to merge them", "Open Upgrade and buy Spawn Rarity", "Make an Apple". Hold the `BatUnlocked` toast for new accounts until after the first Apple or 3 minutes. (spec: [[Rarity Gating and Rebirth]], new section; [[Bats]] B30)

### 0:25 First pick-up and merge
- **Sees:** the fruit prompt "Pick Up". She carries a Blueberry over another one. It glows green and the prompt reads "Merge". She presses E. A Strawberry appears with a sparkle. A purple "+3" floats up, 1.5 studs above the "+$" popup.
- **Decides:** keeps merging.
- **Why:** the green glow shows what pairs.
- **Rule:** [[Rarity Gating and Rebirth]] rule 18 (every merge into tier 1 pays 3 shards at stage 1), U11 (purple shard popup).
- PAYOFF: the first merge gives three rewards at once: a new fruit, a discovery sparkle and a purple "+3".

### 1:30 Index claims
- **Sees:** a red "!" on the Index button. Inside, Blueberry and Strawberry have Claim buttons. Every card from Dandelion on is dimmed with a padlock and "Rebirth 1" to "Rebirth 8". The title bar reads "Found 2/5".
- **Decides:** claims both: $60 and $127 cash, 5 and 6 shards.
- **Why:** claims are her biggest early cash.
- **Rule:** [[Rarity Gating and Rebirth]] rules 20 and 21, U1, U3.
- PAYOFF: the sealed cards say "Rebirth 1" on day one. The goal "rebirth to unlock more" is visible before she knows what a rebirth is, as the brain dump asks.

### 2:00 Spawn Rarity, twice
- **Sees:** the Upgrade panel. Spawn Rarity costs $250. Then $650 for "Up to Carrot". After that the card reads "Level 3/3", "Up to Carrot", a grey LOCKED plate and the gold line "Rebirth to raise".
- **Decides:** buys both, then buys Capacity ($100, $190) as cash comes in.
- **Why:** more and better fruit.
- **Rule:** [[Rarity Gating and Rebirth]] rules 8 and 9, U7.
- FRICTION: **P1-2 (low)** Two minutes in, an upgrade says LOCKED and "Rebirth to raise" before she has heard the word rebirth. It can read as a broken button. **Fix:** at stage 1 the price row reads "Max for Common. Rebirth to raise." so it ties the limit to the rarity she can see in the Index. (spec: [[Rarity Gating and Rebirth]] U7)

### 3:10 First Apple: the cap
- **Sees:** two Tomatoes merge into an Apple with the discovery sparkle. A centred card opens: the MAX FRUIT title, an Apple card and three lines: "Apple is your best fruit", "Two Apples cannot merge until you rebirth", "Rebirth 1 unlocks 8 Uncommon fruits". A red X and a green REBIRTH button.
- **Decides:** presses REBIRTH to see what it means.
- **Why:** the card is the first time the game explains a goal.
- **Rule:** [[Rarity Gating and Rebirth]] rule 7, U5.
- PAYOFF: the cap is explained once, calmly, in the kit style, at the exact moment it matters.

### 3:20 The Rebirth menu
- **Sees:** a panel built like the Index. Left: "Rebirth 1 unlocks 8 Uncommon fruits" and 8 "???" cards. Footer: "Your plot starts again from Blueberry". Second footer: "Also unlocks: Cherry Bomb, TNT". NEEDS: Index 5/5 with a green tick, "$900 / $56.5K" in red, "Shards 41 / 50" in red. REWARDS: +25%, +1 storage, and a green bat chip "Power 40". The REBIRTH pill is grey.
- **Decides:** closes it and goes back to merging.
- **Why:** she can see she is short on cash, by a lot.
- **Rule:** [[Rarity Gating and Rebirth]] rule 12, U4.
- PAYOFF: every need is on one screen, with ticks. She knows exactly what blocks her.
- FRICTION: **P1-3 (medium)** The cash row is the only goal for the next 15 minutes, and it is a bare number: "$900 / $56.5K". Nothing says how long that is. **Fix:** under the cash row, a small line "About 15 min at your income now", computed from `CashPerSecond`, refreshed every income tick. Also name the side goals on the same panel when cash is the last unmet need: "While you wait: tame a pet in the forest". (spec: [[Rarity Gating and Rebirth]] U4)

### 3:30 to 8:30 Merging under the cap
- **Sees:** Apples pile up. Carrying an Apple to another Apple shows "Max Fruit" and "Rebirth to merge" with no green glow. She presses E anyway: red warning "Apple is your best fruit. Rebirth to unlock Uncommon fruit." The Apple stays in her hands. Shards pass 50 at about 4:30; the shards row turns white.
- **Decides:** buys Capacity levels and Spawn Rate as cash allows. Buys Plot Size level 1 ($2.5K) on the pen sign by her gate at 5:00.
- **Why:** more fruit on the plot means more cash.
- **Rule:** [[Rarity Gating and Rebirth]] rules 3, 5; [[Plot Overhaul and Environment]] E1, E2, UI 1.
- PAYOFF: the cap prompt always answers. A press never silently fails.
- FRICTION: **P1-4 (low)** The pen sign says "Level 2/5" and "39 x 63 studs". It reads like "more fruit", but Plot Size only adds room; fruit slots come from Capacity. Level 2 costs $20K, 35% of the rebirth cash, and buying it pushed the simulated rebirth back by about 1.3 minutes. **Fix:** a fourth line on the sign: "More room to walk. Fruit slots come from Capacity." (spec: [[Plot Overhaul and Environment]] UI 1)

### 8:30 Into the sea
- **Sees:** she walks off the beach. Her character sinks to chest depth. A "Stamina" bar appears bottom centre. A green notice: "Swimming tires you out. Rest on stones and islets." Once she is in open water, a bar of slots slides up at the bottom: a flag slot and a slot with the Shop icon.
- **Decides:** swims straight out toward the trees on the horizon, past the last stepping stones.
- **Why:** the forest is the only new place she can see.
- **Rule:** [[Boats and Swimming]] R5 to R8, R15, 6.1; [[Weapons and Artillery]] UI 2 (empty state).
- FRICTION: **P1-5 (medium)** At stage 1 she owns no weapon and can buy none, yet the Weapon Bar shows a War Flag she can raise. With the flag up, flagged players can knock her around and she cannot fire back. **Fix:** at sea, show the Weapon Bar only when the player owns at least one weapon charge or already has the flag up, and refuse raising the flag at stage 1. The Practice Range keeps showing every weapon. (spec: [[Weapons and Artillery]] UI 2, rule 27)

### 9:40 Washed up
- **Sees:** past the stones the bar runs down. It turns red and reads "Tired", then "Exhausted". The screen dims and a heartbeat plays. Four seconds later the screen goes black and she wakes on the last stone she stood on, with a full bar. Red warning: "Too tired! You washed back to safety."
- **Decides:** tries again, hopping to the tree islet she can now see off to the side.
- **Why:** the islet is close and has a tree she can aim for.
- **Rule:** [[Boats and Swimming]] R12, R13, 5.3 (16 rest islets at r 820).
- PAYOFF: failing costs 30 seconds, not her fruit or potions. Nobody dies in the sea.

### 11:00 The forest, empty-handed
- **Sees:** she reaches the forest beach. Green notice: "Wild animals trade for fruit. Bring what they ask for." In the nearest clearing a small Sheep grazes. Over its head floats a mini Index card: a Strawberry in the middle, "Strawberry" on top, "0/1" below. Closer, the Sheep looks up at her. The prompt reads "Feed Strawberry".
- **Decides:** holds E.
- **Why:** the prompt invites it.
- **Rule:** [[Pets]] P5, P9, P12, P13 (g), U4, U7.
- **Result:** red warning "You need a Strawberry. Store one, then come back." Nothing happens.
- FRICTION: **P1-6 (high)** The first trip is wasted. The hint arrives on the forest beach, after a 65-second swim, and the fruit she needs is only shown in the forest. She has never used storage. **Fix:** move the one-time hint to the first time a player enters the sea, and name the fruit: "Wild animals in the forest want a Strawberry. Store one before you cross." Add a "Wanted" line to the Pets panel listing the requests of the player's 6 wild animals (the server already sends them in the `"wild"` snapshot). (spec: [[Pets]] U2, U7; [[Boats and Swimming]] R15)
- FRICTION: **P1-11 (medium)** As written, requests changed only when the best tier changed (P9). A request set when she joined (Blueberry, when Carrot was her best) would have stayed Blueberry for all of stage 1, so the card would ask for a fruit she had stopped growing, and test T4 (which expects Strawberry once Apple is her best) would fail. Spec error, fixed in [[Pets]] P9: unfed animals now recompute their request whenever her best fruit changes.

### 12:40 Home, store, back
- **Sees:** she swims home. She picks up a Strawberry and presses Store in the carry row. Inventory shows "1/5".
- **Decides:** stores two Strawberries and swims back through the islets (about 65 s).
- **Why:** she wants the Sheep now.
- **Rule:** [[Pets]] P13 (g), P14 (a feed may come from storage).

### 14:40 First tame
- **Sees:** she holds E. A Strawberry leaves her bag. The Sheep chews, hops twice, sparkles and fades. Green notice: "New pet: Common Sheep!" (the string was "Tamed a %s %s!", fixed in [[Pets]] U7). The Pets button gets a "!".
- **Decides:** swims home to see it.
- **Rule:** [[Pets]] P15, P19 (3 slots at 0 rebirths).
- PAYOFF: the Baby Sheep is already on her plot when she lands. It walks between fruit, grazes and turns to look at her. Every 5 seconds a small "+$25" floats over it (it pays like a quarter of an Apple, $5.05/s).

### 16:30 Golden Crate
- **Sees:** green notice "A Golden Crate is falling on the plaza". A gold crate drops with a beam. She runs over, holds E for half a second and lifts it. She walks 15% slower. Halfway home a stage 7 player swings an Astral bat at her. She flies about 38 studs and tumbles. The crate drops. For 2 seconds she cannot lift it. He takes it home.
- **Decides:** gives up on crates for today.
- **Rule:** [[Bats]] C1 to C7, B15 (push uses the attacker's tier), Numbers (Power 80 at tier 7).
- FRICTION: **P1-7 (medium)** A stage 1 player is pushed 38 to 50 studs by veterans and can push back only 11. Explosives protect stage 1 gardens completely ([[Explosives and Fruit Health]] rule 30), but nothing protects a stage 1 player from bats, so the plaza is where a new player first meets griefing. **Fix:** stage 1 players cannot be hit by bats of tier 3 or higher (a stage 1 or 2 player can still play with them), and they get the "Plots are safe zones" toast the first time anyone swings at them. (spec: [[Bats]] B12)

### 17:30 Plot Size level 2
- **Sees:** the pen sign: "Level 2/5", "39 x 63 studs", "$20K" in green.
- **Decides:** buys it.
- **Why:** she thinks a bigger garden holds more fruit.
- **Result:** her cash drops from $27K to $7K. The rebirth moves from about 19:15 to about 20:30. See P1-4.

### 20:30 Ready to rebirth
- **Sees:** a red "!" on the Rebirth button. The menu shows three green ticks and a lit REBIRTH pill. The confirm dialog: "Rebirth? Resets cash, upgrades, your plot and stored fruit. Keeps your Index, shards, tools, pets and Plot Size."
- **Decides:** Yes.
- **Rule:** [[Rarity Gating and Rebirth]] rules 12, 15, 19, U4.
- PAYOFF: a star burst at the plot centre, the merge sound at a lower pitch, her banners turn green, and the notice "Rebirth 1 complete. Uncommon fruit unlocked, cash x1.25." Her bat turns green and gets stronger (Power 40).
- FRICTION: **P1-8 (medium)** As written, `BatUnlocked` fired in the same frame as `RebirthDone`. The green toast has one label, so one of the two lines was lost. Spec error, fixed in [[Bats]] B30: the bat line now comes 3 s after `RebirthDone`.

### 20:45 Starting again
- **Sees:** three Blueberries. Cash $0. Spawn Rarity back at Blueberry. The Index reads "Found 5/13": the 8 Uncommon cards are now plain "???" cards. Storage "0/6". Her Sheep still walks on the plot, but its card reads "Pays like 0.25 of a Blueberry".
- **Decides:** rebuys Spawn Rarity ($250, $650, $1.7K, $4.4K) on the way up.
- **Rule:** [[Rarity Gating and Rebirth]] rules 11, 15, 17; [[Pets]] P25, P30.
- FRICTION: **P1-9 (low)** Her merges into Common fruit stop paying shards (only Uncommon merges pay now) and nothing tells her. **Fix:** add a second sentence to `RebirthDone`: "Uncommon merges now pay 7 shards." (spec: [[Rarity Gating and Rebirth]] U10)

### 23:30 First Uncommon
- **Sees:** two Apples merge into a Dandelion. It has the discovery sparkle and a purple "+7". The Index card fills in.
- **Decides:** keeps merging up the new ladder.
- **Rule:** [[Rarity Gating and Rebirth]] rules 1, 2, 18; D-010.
- PAYOFF: the reason she rebirthed pays off about 3 minutes later. Apple plus Apple, refused for 17 minutes, now makes a new fruit.

### 24:00 to 30:00 A new fruit every minute
- **Sees:** Daffodil, Pink Tulip, Orange Tulip, Crocus, Candy Sunflower, Red Lollipop, each with a sparkle and "+7". Spawn Rarity climbs past Dandelion ($11.4K) and Daffodil ($29.7K).
- **Rule:** [[Rarity Gating and Rebirth]] Table 1 (stage 2).
- PAYOFF: the discovery pace of the first two minutes comes back for six minutes.

### 30:45 Blue Lollipop: the second cap
- **Sees:** the MAX FRUIT card again: "Blue Lollipop is your best fruit", "Rebirth 2 unlocks 11 Rare fruits". The Rebirth menu's second footer reads "Also unlocks: Melon Mortar".
- **Decides:** checks the menu. Cash $1.1M of $104M.
- **Rule:** [[Rarity Gating and Rebirth]] rule 7, U4, U5.

### 31:30 The Shop
- **Sees:** the Shop's four tabs: Potions, Weapons, King, Robux. Potions cost 180 shards each (Aqua Affinity 180). On the Weapons tab, Cherry Bomb (85 shards) and TNT (110) are lit since her rebirth; the rest read "Rebirth 2" to "Rebirth 7".
- **Decides:** buys an Aqua Affinity for her next forest trip.
- **Note:** the Potions tab shows 7 cards. Haste is not on the shelf until the second ore band (D-032). The [[Potions]] player view still described 8 cards with Haste; corrected.
- **Why:** it makes the swim 20 seconds instead of 65.
- **Rule:** [[Potions]] rules 7, 8, 24; [[Weapons and Artillery]] UI 1; D-022.
- PAYOFF: every rebirth from here lights at least one new Weapons card, so "rebirth to unlock more" is visible in the Shop too.

### 32:00 The royal chip
- **Sees:** the royal chip top left has had a red "!" since about 3:00, when she found all 5 Commons. The Kingdom panel offers a red Challenge pill.
- **Decides:** ignores it.
- **Rule:** [[King UI Overhaul]] rule 26; [[Rarity Gating and Rebirth]] D-026 (`MinDiscovered = 5`).
- FRICTION: **P1-10 (low)** A 3-minute-old player is invited to duel the King before she understands her garden. The badge competes with the Rebirth goal. **Fix:** keep her eligible, but show the chip's "!" only after the first rebirth. (spec: [[King UI Overhaul]] rule 26)

### 40:00 Logs off
- **Sees:** cash about $10M of $104M, about 900 shards after the Aqua Affinity, one pet. The plot is still merging.
- **Decides:** leaves.
- **Rule:** offline pay is 3% of live income for up to 5 hours ([[Core Server - Plots, Spawning, Income]]).
- **Next session:** the boat (1,000 shards) is a few merges away. It is the stage 2 buy the [[Stage Model]] plans for, and it turns the 65-second swim into a 14-second ride.

---

## What this path found

| Id | Severity | Friction | Spec |
|---|---|---|---|
| P1-1 | high | No first-session guide; the first toast is about a bat | [[Rarity Gating and Rebirth]], [[Bats]] |
| P1-6 | high | First forest trip wasted: the wanted fruit is only shown in the forest | [[Pets]], [[Boats and Swimming]] |
| P1-3 | medium | Cash need shown as a bare number for 15 minutes | [[Rarity Gating and Rebirth]] |
| P1-5 | medium | Stage 1 swimmer gets a War Flag and no weapons | [[Weapons and Artillery]] |
| P1-7 | medium | Veterans' bats push stage 1 players 38 to 50 studs | [[Bats]] |
| P1-8 | medium | Rebirth toast and bat toast in the same frame (spec error, fixed) | [[Bats]] |
| P1-11 | medium | Animal requests went stale inside a tier (spec error, fixed) | [[Pets]] |
| P1-2 | low | LOCKED and "Rebirth to raise" at minute 2 | [[Rarity Gating and Rebirth]] |
| P1-4 | low | Plot Size reads like more fruit; level 2 delays the rebirth | [[Plot Overhaul and Environment]] |
| P1-9 | low | Common merges stop paying shards without a word | [[Rarity Gating and Rebirth]] |
| P1-10 | low | Royal "!" at minute 3 | [[King UI Overhaul]] |

Spec errors fixed by this path: [[Bats]] B30 (toast timing), [[Pets]] P9 (requests follow the best fruit), [[Potions]] player view (7 cards, not 8).

Payoffs: the first merge, the sealed Index cards, the Max Fruit card, the Rebirth menu's ticks, the no-death wash-up, the first pet appearing at home, the rebirth burst, the first Uncommon three minutes after the rebirth, and new Weapons cards on every rebirth.
