---
tags: [player-path]
date: 2026-10-04
---
# Underground Unlock - Stage 6-7

Leo is 15 and plays on a PC. He is a saver: since stage 2 he has bought only the boat, a few potions and his first pet slot. At the start of stage 6 (Mythic) he decides to save for the underground. This path follows him across stage 6 and into stage 7 (Astral): the saving, the purchase, the first ores, the Ores Index, and a TNT warning while he is down in the mine.

Every rule cited is in the spec notes in [[02 Systems]]. Each friction is ranked in [[00 Paths Summary]].

**Words used here**
- **Mine**: a player's underground, bought one floor at a time with shards. Floor 1 costs 100,000 (Samarth's number).
- **Node**: a studded rock in the mine. Mining empties its HP and it drops one Coal. **Ore**: Coal or Iron. Two Coal merge into one Iron. Iron is the top ore in v1.
- **Ore Bonus**: every ore lying in the mine adds a share to fruit income: Coal +0.3%, Iron +0.75%, at most 7.5% with Floor 1 (D-018).
- **Plateau**: the plot is full of cap fruit and nothing can merge until the rebirth (see [[Mid Game - Stage 4 Epic]], P2-1).
- **Frontier merge**: a merge into your newest rarity. It pays 155 shards at stage 6 and 340 at stage 7 (D-010).

**How the times were made.** Times are hours and minutes of total play since his first join (h:mm). They come from `~/fruit-empire/sim/integration_check.py` re-run with a saver's spending (boat 1,000, potions 2,000, first pet slot 10,000, nothing else), median of 7 careers. In that run he holds 14.9K shards entering stage 6, 73.4K at the stage 6 gate, and passes 100K about 26 minutes into stage 7. Shards in stage 7 move like this (every 5 minutes, thousands): 53.4, 53.4, 54.4, 64.0, 85.1, 100.2, 124.9, 141.9, then flat at 141.9 for the remaining 83 minutes.

**Starting state (4:31)**: 5 rebirths, stage 6 just started. 14.9K shards. Boat, 6 pet slots (5 base, 1 bought), pets near 25% of a full plot at each plateau. Bat red (Mythic).

---

### 4:31 Stage 6 begins
- **Sees:** the plot restarts at Cacao (the head start). The banners are red. The Rebirth menu reads "Rebirth 6 unlocks 5 Astral fruits"; NEEDS include 20K shards.
- **Rule:** [[Rarity Gating and Rebirth]] rule 11, Table 1, Table 3.

### 4:32 The goal
- **Sees:** at the back right of his lot, stepping stones lead to the stone hut with locked doors and a gold padlock. Inside, the iron cage of the Mine Lift and a MINE sign. He presses E "Open Mine". The Mine panel opens like the Index: three floor cards, "Floor 1, 100K" with a Buy button, Floors 2 and 3 greyed with "Buy Floor 1 first". The status strip: "Shards 14.9K".
- **Decides:** saves. No potions (3,900 each now), no Mega TNT (16,000), no Celestial Saber (31,000).
- **Why:** the mine is the biggest thing he can buy, and the hut has been locked on his lot since day one.
- **Rule:** [[Underground and Ores]] rules 1, 4, 5, U1, U2; [[Plot Overhaul and Environment]] D3, "Underground entrance".
- PAYOFF: the locked hut is a goal he has walked past for 4 hours. The brain dump's "base upgrade" is visible long before anyone can afford it.
- FRICTION: **P3-6 (low)** The Index has no Ores tab until Floor 1 is bought, so the ore category the brain dump asked for ("make this a category in the index too") is invisible as a goal for the first 6 to 8 hours. **Fix:** show the Ores tab from the start with two sealed cards (padlock, "Buy the Mine"), the same look as sealed fruit cards. (spec: [[Underground and Ores]] U4)

### 4:40 to 5:11 Stage 6 merging
- **Sees:** Mythic fruit arrive with purple "+155" popups. A Golden Crate on the plaza gives 3,100. His shard count climbs from 14.9K to about 74K.
- **Rule:** [[Rarity Gating and Rebirth]] rule 18; [[Bats]] C6, Numbers.
- PAYOFF: at stage 6 one frontier merge pays 155 shards, three times the whole shard need of the first rebirth. The numbers feel big, and the mine feels reachable.

### 5:11 to 6:13 The stage 6 plateau
- **Sees:** his plot fills with Dragon Peppers 40 minutes into the stage. From here nothing merges and his shard count stops at about 74K. The cash need is 62 minutes away.
- **Decides:** fills the time in the forest, hunting the Mythic Basilisk (4 in 100 tier 6 rolls), and with a royal decree.
- **Rule:** [[Pets]] P2, P3 (Basilisk from tier 6, weight 4).
- FRICTION: the plateau wait (P2-1 in [[Mid Game - Stage 4 Epic]]): 62 minutes here.
- PAYOFF: at 5:40 a Basilisk shows up in a clearing. It slithers with a travelling S-wave. Tamed, it pays twice what a Sheep pays.

### 6:13 Rebirth 6
- **Sees:** shards 73.4K, minus 20K. Cyan banners and an Astral bat (Power 80). "Also unlocks: Starfruit Barrage" lit on the Weapons tab.
- **Rule:** [[Rarity Gating and Rebirth]] rule 15; [[Weapons and Artillery]] rule 2.
- **Shards now:** 53.4K.

### 6:23 to 6:39 The climb to 100K
- **Sees:** for 10 minutes no shards (his plot is still climbing through Mythic). Then Glowshroom, the first Astral fruit, and "+340" on every Astral merge. 64K, 85K, then 100.2K at 6:39.
- **Rule:** D-010; [[Rarity Gating and Rebirth]] rule 18.
- PAYOFF: 46K shards in 16 minutes. The last stretch of a long save goes fast.

### 6:39 Buying Floor 1
- **Sees:** he runs to the hut. "Open Mine". The Floor 1 card's price "100K" is white now. He presses Buy. The check: "Buy Floor 1? 100K shards. You will have 0.2K shards left. Your next rebirth needs 30K shards."
- **Decides:** Yes.
- **Why:** his plot is still merging, and each Astral merge pays 340. He expects to earn the 30K back before his plot fills.
- **Rule:** [[Underground and Ores]] rules 1, 2, U3.
- **Result:** he was right: the simulation earns him about 42K more before his last merge at about 6:48.
- FRICTION: **P3-5 (high)** The same press after his plot is full would have stranded his rebirth. Once the plot is at the plateau, no merge pays shards, so a player who buys Floor 1 with less than "100K plus the next rebirth's need" cannot earn the rest by merging. His cash would be ready at 8:11 with his shards stuck. Only crates, royal rewards, or storing cap fruit to free slots and merge again would get him there, and none of that is suggested. The [[Stage Model]] line "Shards are never the thing that blocks a rebirth" is only true for a player who never spends below the need. **Fix:** in this check, when the player's plot is full of cap fruit and the shards left would be under the next rebirth's need, show a red third line: "Your plot is full, so merges will not pay shards until you rebirth." and dim Yes for 2 seconds. Longer term, see the plateau fix in [[00 Paths Summary]]. (spec: [[Underground and Ores]] U3; [[Stage Model]])

### 6:39 The doors open
- **Sees:** the hut's doors swing open, the first depth lamp lights amber and a green notice says "Floor 1 unlocked!". The Index now has a third tab: Ores.
- **Rule:** [[Underground and Ores]] rules 5, U4, U14; [[Plot Overhaul and Environment]] D3.
- PAYOFF: the hut he has walked past since stage 1 opens on his own lot.

### 6:40 Going down
- **Sees:** he is carrying a Paradise Petal. He steps into the cage and presses "Go Down". Red warning: "Put down what you are carrying first." He drops it and presses again. The screen fades to black for about half a second. He arrives in a warm studded cave: stepped grey stone, timber beams, lanterns, amber crystals in the corners, a campfire by the lift, and six grey rocks dotted with black gems. The Mine Board by the lift reads "FLOOR 1 OF 3", "Ores 0 / 10", "Ore bonus +0.0%".
- **Rule:** [[Underground and Ores]] rules 6, 7, U10, U12, 7.4.
- PAYOFF: the cave is the showpiece the brain dump asked for ("the underground should look super good").

### 6:41 First Coal
- **Sees:** he walks to a rock. The prompt reads "Mine" (F). His bat goes away and a pickaxe comes out. It chops every half second; chips fly and a small amber bar drains. After 10 chops the rock bursts and a lump of Coal hops out; the rock leaves rubble that grows back in 25 seconds. He presses E on the Coal: a sparkle and "New ore: Coal".
- **Rule:** [[Underground and Ores]] rules 9, 10, 13, 14, 19, U8; D-025 (one hand).
- PAYOFF: mining is tactile: chips, a draining bar, a burst, a hop.

### 6:42 First Iron
- **Sees:** he carries the Coal to another Coal. It glows green. E "Merge". The two flash white and become Iron, with a faint glint and a discovery sparkle.
- **Rule:** [[Underground and Ores]] rules 15, 16, 5.1 (Iron glint).
- PAYOFF: the same merge feel as fruit, in a new place.

### 6:43 A warning from the surface
- **Sees:** red warning: "Rex placed a Big TNT in your garden! Lock to block it." A red "!" on his Lock pill. He taps Lock.
- **Rule:** [[Explosives and Fruit Health]] rules 14, 32.
- **As the specs were written:** the server refused with `LockOffPlot`, because live `lockPlot` only accepts a press from a player standing on their own plot (`RebirthLock.lua:223`), and the cave is 140 studs under it. The Big TNT went off: Rex is stage 8, so it hit at Leo's stage 7: 191 at the centre against his 381-HP Astral fruit. No stun, but six bars he could not stop.
- FRICTION: **P3-4 (high)** The toast told him to do something the game then refused. The same happens at sea, in the forest and in the sky. Spec error, fixed in [[Explosives and Fruit Health]] rule 32 and its code plan (decision D-034): while an explosive is armed on your plot, the Lock can be raised from anywhere. With the fix his dome rises and the Big TNT goes back to Rex's bag.

### 6:44 The floor is full
- **Sees:** about 3 minutes after his first swing the cave holds 10 Iron. The Board: "Ores 10 / 10", "Ore bonus +7.5%", "+$X/s". He presses Mine on a fresh rock. Red warning, as written: "Your ore floor is full. Merge ores to make room."
- **Decides:** looks for something to merge. There is nothing: two Iron do not merge in v1.
- **Rule:** [[Underground and Ores]] rules 12, 15, 20, 5.4 (about 3 minutes to fill Floor 1).
- FRICTION: **P3-2 (medium)** The warning sends him looking for a merge that does not exist. Spec error, fixed in [[Underground and Ores]] rule 12 and U14: with every ore at the top ore, the line is "Your floor is full of Iron, the best ore for now. More slots come with the next floor."
- FRICTION: **P3-1 (high)** 100K shards (about 2 hours of saving) bought about 3 minutes of new play. After that the cave has nothing to do: two ore kinds, ten slots, all full. The +7.5% is real, but it is passive. **Fix:** give Floor 1 a longer ladder at launch. The cheapest is to ship the Uncommon band (Copper, Emerald) on Floor 1 now, so the ladder is Coal, Iron, Copper, Emerald, and re-solve the boosts so the cap stays 7.5% (D-018 still holds). The Mine Board's last line should also name the next goal: "Floor 2: 150K". (spec: [[Underground and Ores]] 5.1, 5.4, U10)

### 6:45 Ores Index
- **Sees:** the Index button has a "!". The Ores tab shows Coal and Iron with Claim buttons. Coal pays 500 shards and 60 seconds of a Coal's income; Iron pays 1,000 and 60 seconds of an Iron's.
- **Rule:** [[Underground and Ores]] rule 19, 5.5, U5.
- PAYOFF: 1,500 shards back, 1.5% of the floor price: a thank-you, as the spec intends.

### 6:46 A visitor
- **Sees:** a friend presses "Visit Mine" on his lift and walks around his cave. She can look but has no prompts.
- **Rule:** [[Underground and Ores]] rule 8.
- PAYOFF: the mine is something to show off, and nothing in it can be taken.

### 6:47 Back up
- **Sees:** "Go Up", fade, his lot. On the HUD nothing has changed. The CornerHud boost still shows only the rebirth and Index bonus.
- **Rule:** [[Underground and Ores]] U15 (the Ore Bonus is not part of `CashMultiplier`, so the HUD does not show it).
- FRICTION: **P3-3 (medium)** Above ground there is no sign that the mine earns anything. The only readouts are the Mine Board in the cave and the Mine panel. **Fix:** when the Ore Bonus is above 0, the CornerHud boost bar gets a second short line, "Ores +7.5%", built from the same kit bar text; nothing new on the HUD otherwise. (spec: [[Underground and Ores]] U15)

### 6:48 to 8:11 The stage 7 plateau
- **Sees:** his plot fills with Paradise Petals 35 minutes into stage 7. Shards about 43.7K (0.2K + 42K from merges + 1.5K from the Ores Index). The rebirth needs 30K, so he is safe. Then 83 minutes of waiting with nothing to merge (P2-1).
- **Decides:** forest trips, the Practice Range with the Starfruit Barrage, crates.
- **Rule:** [[Stage Model]]; `plateau_wait.py`.

### 8:11 Rebirth 7
- **Sees:** the mine, its ores and the Ores Index stay. His ore income starts small again, because it is a share of fruit income and the plot is back at its head start.
- **Rule:** [[Underground and Ores]] rules 23, 24; D-009.
- **Next goal:** Floor 2 at 150K. Stage 8 frontier merges pay 748 each, so he passes 150K plus the 100K rebirth need during stage 8.

---

## What this path found

| Id | Severity | Friction | Spec |
|---|---|---|---|
| P3-1 | high | 100K buys about 3 minutes of new play | [[Underground and Ores]] |
| P3-4 | high | "Lock to block it" refused off-plot (spec error, fixed, D-034) | [[Explosives and Fruit Health]] |
| P3-5 | high | Shards spent after the plateau cannot be re-earned by merging; the rebirth can stall | [[Underground and Ores]], [[Stage Model]] |
| P3-2 | medium | "Merge ores to make room" when nothing can merge (spec error, fixed) | [[Underground and Ores]] |
| P3-3 | medium | Ore income invisible above ground | [[Underground and Ores]] |
| P3-6 | low | The Ores category is hidden until Floor 1 | [[Underground and Ores]] |

Spec errors fixed by this path: [[Explosives and Fruit Health]] rule 32 (Lock from anywhere while armed), [[Underground and Ores]] rule 12 (full-of-Iron message).

Payoffs: the locked hut as a long goal, big stage 6 and 7 merge numbers, the Basilisk, the hut opening, the cave look, tactile mining, the Coal to Iron merge, Ores Index claims, and visitors who can look but not take.
