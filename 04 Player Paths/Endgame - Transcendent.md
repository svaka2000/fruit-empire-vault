---
tags: [player-path]
date: 2026-10-04
---
# Endgame - Transcendent

Rin is 16 and plays on a PC. She is a regular player: pets at the 25% limit since stage 6, the mine with Floor 1 since stage 8. This path starts at her eighth rebirth, which unlocks Transcendent (stage 9), and asks one question: what keeps her playing once there is nothing left to unlock?

Every rule cited is in the spec notes in [[02 Systems]]. Each friction is ranked in [[00 Paths Summary]].

**Words used here**
- **Stage 9**: Transcendent, the last stage (8 rebirths). Cap fruit Soul Fruit ($237Sx/s each). Spawn ceiling Zenflare. Cursed Fruit and Soul Fruit are merge-only. There is no rebirth after it ([[Rarity Gating and Rebirth]] rule 13).
- **Frontier merge**: a merge into Transcendent. It pays 1,646 shards (D-010).
- **Plateau**: the plot is 35 cap fruit. Nothing spawns, nothing merges, and Clear is refused because every fruit is the same kind (`Merging.lua:396-399`).
- **Shard sink**: something that costs shards, so shards keep a use.

**How the times were made.** Times are hours and minutes of her total play (h:mm). They come from `~/fruit-empire/sim/integration_check.py` (resolved design, median of 7 careers): Transcendent at 10:36; she enters stage 9 with about 96K shards; stage 9 earns about 330K shards, all before her last merge 32.6 minutes in; after an hour she holds about 30K. `probe2.py` puts her first Soul Fruit about 21.7 minutes into stage 9.

**Starting state (10:35)**: 7 rebirths. Floor 1 and Floor 2 of the mine. 6 pet slots plus 1 bought, pets at the 8.75-unit limit. About 226K shards at the stage 8 gate.

---

### 10:35 Rebirth 8
- **Sees:** the rebirth confirm, then the burst. "Rebirth 8 complete. Transcendent fruit unlocked, cash x3." Three seconds later: "Bat unlocked: Transcendent Bat (Power 96, Speed 84)". Her banners, bat and boat trim turn white-gold. The plot starts again at Sarracenia.
- **Rule:** [[Rarity Gating and Rebirth]] rules 11, 15, 16; [[Bats]] B2, B30; [[Boats and Swimming]] R23.
- PAYOFF: the last colour of every system at once.

### 10:36 The Rebirth menu
- **Sees:** "Every rarity is unlocked" over the four Transcendent cards. NEEDS shows only "Index 60/64". No REWARDS and no button.
- **Rule:** [[Rarity Gating and Rebirth]] rule 13, U4 (final stage).
- PAYOFF: a clean finish line. The menu that has driven 10 hours says, plainly, that the journey is done.

### 10:45 to 11:08 The last climb
- **Sees:** Alien Carrot, Zenflare, Cursed Fruit: each a new discovery, each merge into them a purple "+1,646". Her shard count climbs by about 330K in 23 minutes.
- **Decides:** buys Floor 3 (300K) at about 11:05. The hut's third lamp lights; the stair slab on Floor 2 crumbles into a violet crystal grotto. Ore bonus 10.5%.
- **Rule:** D-010; [[Underground and Ores]] rules 1, 6, 5.2.
- PAYOFF: the biggest numbers of the career, and the last floor of the mine.

### 10:57 First Soul Fruit
- **Sees:** two Cursed Fruit become a Soul Fruit with the sparkle. The MAX FRUIT card opens.
- **Rule:** [[Rarity Gating and Rebirth]] rules 4, 7, U5.
- **As the specs were written:** the card read "Two Soul Fruits cannot merge until you rebirth" and "Rebirth 9 unlocks ...", with a REBIRTH button leading to a menu with no button.
- FRICTION: **P6-3 (medium)** At stage 9 the Max Fruit card, both Merge All refusals ("until you rebirth", "pairs wait for a rebirth"), the Spawn Rarity card ("Rebirth to raise") and `SpawnCapText` all promised a rebirth that does not exist. Spec error, fixed in [[Rarity Gating and Rebirth]] U5 (a stage 9 variant: "Soul Fruit is the best fruit in the game", "You have unlocked every rarity", no REBIRTH pill), U7 ("Max for every rarity") and U10 (`MergeAllCapNothingFinal`, `SpawnCapTextFinal`).
- **With the fix:** PAYOFF: the best fruit in the game, named as such, once.

### 11:00 Index 64/64
- **Sees:** the last Transcendent card fills. "Found 64/64". The Index bonus is at its full +50%.
- **Rule:** [[Rarity Gating and Rebirth]] U3; Index bonus ([[Config and Fruit Data]]).
- PAYOFF: the Index the brain dump built the whole rebirth system around is complete.

### 11:08 The plot freezes
- **Sees:** 35 Soul Fruit. Merge All: "Nothing can merge. Soul Fruit is the best fruit in the game." Clear: refused, every fruit is the same kind. Nothing spawns. Her income is about $50Sp/s with pets, ores and the Index bonus.
- **Decides:** looks for something to do.
- **Rule:** [[Rarity Gating and Rebirth]] rules 6, 13; `Merging.lua:396-399`.
- FRICTION: **P6-1 (critical)** From here the plot never changes again. Before stage 9 the plateau was a wait (P2-1 in [[Mid Game - Stage 4 Epic]]); at stage 9 it is permanent. No spawns, no merges, no shards from merging, and no rebirth to reset it. About 33 minutes after unlocking the last rarity, the core loop of a merge game is over. **Fix:** an endgame loop on the plot itself. At stage 9 only, Clear works on a plot of Soul Fruit: it removes 4 Soul Fruit (the cooldown applies), new fruit spawn into the gaps, and merging them back up pays 1,646 shards per Transcendent merge again. Each cycle is about 12 frontier merges (about 20K shards) for a few minutes of play, and the cash it costs does not matter at stage 9 (P6-2). (spec: [[Rarity Gating and Rebirth]], new stage 9 rule)

### 11:10 Cash with no use
- **Sees:** cash climbing by about $50Sp every second. Every cash upgrade is maxed: Capacity 35, Spawn Rate 2.5 s, Spawn Rarity at Zenflare. The Upgrade pill has no badge. The Shop takes shards only.
- **Rule:** [[Stage Model]]; [[Rarity Gating and Rebirth]] rule 13.
- FRICTION: **P6-2 (high)** The main currency of the game stops meaning anything about 3 minutes into stage 9, when the last cash upgrade is bought. **Fix:** a stage 9 cash sink that feeds the shard sinks. One option: a Royal Treasury stall where 60 seconds of full-plot income buys one frontier merge's worth of shards (1,646), at most 60 times an hour. Another: cash-priced cosmetics (banner styles, gate statues, boat sails). Samarth's call. (spec: [[Stage Model]]; new)

### 11:12 Shopping with 30K
- **Sees:** about 30K shards left after Floor 3. Speed potion 41,000. Luck 62,000. Statue Greatsword 250,000. Celestial Saber 330,000. Nuke 410,000. Second pet slot 150,000. Aqua Affinity 500 (capped).
- **Decides:** buys nothing.
- **Rule:** D-017 (`ShardPrice` at stage 9); [[Potions]] rule 8; [[Royal Combat]] skins; [[Pets]] P20.
- **Note:** with merging over (P6-1), her only shard income now is Golden Crates (32,920 each), royal rewards and one-time Index claims. Every item above is out of reach until royal play pays for it.

### 11:13 The Luck card
- **Sees:** as written, the Luck card read "Spawn Rarity maxed" with Buy still enabled "so a player can stock up for the next stage". Drinking was refused with "Save it for after your next rebirth!".
- FRICTION: **P6-4 (medium)** At stage 9 Luck can never be drunk again, the card still sold it for 62,000 shards, and the refusal pointed at a rebirth that does not exist. Spec error, fixed in [[Potions]] rule 26 and UI 1 and [[Rarity Gating and Rebirth]] rule 10: at the stage 9 ceiling the card reads "Maxed for good" with no Buy, and the refusal reads "Spawn Rarity is maxed for good."

### 11:20 Transcendent pets
- **Sees:** in the forest, Transcendent animals with glowing collars and sparkles. A Transcendent Basilisk wants "Alien Carrot 0/3". Her plot holds only Soul Fruit, so she has no Alien Carrot.
- **Rule:** [[Pets]] P9 (request = best fruit + 3 ranks, 3 feeds at tiers 7 to 9), World and assets (tier glow).
- FRICTION: the same as P2-3 in [[Mid Game - Stage 4 Epic]], but permanent: at stage 9 her plot will never grow an Alien Carrot again. The P2-3 fix (a feed accepts any better unlocked fruit) lets her pay with Soul Fruit.

### 11:25 A slot that adds nothing
- **Sees:** the Pets panel header reads "Pets $6.2Sp/s (max)". She has 7 slots and 9 pets. The locked eighth slot card offers "150K Shards".
- **Rule:** [[Pets]] P19, P20, P27 (pets add at most 8.75 units).
- FRICTION: **P6-6 (medium)** Her pets are already at the limit, so the second bought slot (150,000 shards, the price of half her stage 9 income) adds no money at all, only one more animal walking on the plot. The buy check does not say so. **Fix:** when the pet total is at the limit, the buy check adds: "Your pets are at the income limit. This slot adds room, not money." (spec: [[Pets]] U3)

### 11:30 to 13:00 What is left
- **Sees and does:**
  - Pets Index: the Transcendent row, 6 cards at 2,500 shards each (15,000 in all). A Giant Transcendent Basilisk carries the RNG aura.
  - Royal play: as a challenger, a Victory pays 41,150 shards; as King, each Defense pays 16,460 and a Contest win 65,840 (25, 10 and 40 units at her stage). Royal play is now the biggest shard source in the game.
  - Sea fights with the War Flag, Golden Crates (32,920), the Practice Range.
  - Decrees, if she is King.
- **Rule:** [[Pets]] Table 6; [[Royal Combat]] rules 48, 49, Numbers; [[Bats]] C1 to C12; [[Weapons and Artillery]].
- PAYOFF: the crown matters most at the end. Stage-scaled royal rewards (D-017) make the throne worth fighting for at stage 9, which the old flat rewards did not.
- PAYOFF: collecting: the Giant Transcendent Basilisk, the last Pets Index row, every blade skin.

### 13:00 and later
- **Sees:** at about 50K shards an hour from crates and royal rewards (if she wins often), the sinks above take her roughly 20 to 25 more hours. After that, nothing in the specs is left to earn. The open questions say there is no prestige layer ([[Rarity Gating and Rebirth]] Q3) and the next ore band is a sketch ([[Underground and Ores]] 5.1, Q12).
- FRICTION: **P6-5 (high)** No long-term goal after the last purchase. **Fix:** commit to a content cadence in the plan itself: the next ore band with Floor 4 (Copper and Emerald), then the tower upgrade the brain dump reserved ("towers upwards"), each with its own Index category. Decide whether a capped prestige layer comes after that ([[Rebirth Economies in Merge and Incremental Games]], proposal 12). (spec: [[Underground and Ores]] 5.1, 14; [[Rarity Gating and Rebirth]] Q3; [[Plot Overhaul and Environment]] C3)

---

## What this path found

| Id | Severity | Friction | Spec |
|---|---|---|---|
| P6-1 | critical | At stage 9 the plot freezes for good about 33 minutes in | [[Rarity Gating and Rebirth]] |
| P6-2 | high | Cash has no use after the last upgrade | [[Stage Model]] |
| P6-5 | high | No long-term goal after the shard sinks | [[Underground and Ores]], [[Rarity Gating and Rebirth]], [[Plot Overhaul and Environment]] |
| P6-3 | medium | Stage 9 texts promise a rebirth (spec error, fixed) | [[Rarity Gating and Rebirth]] |
| P6-4 | medium | Luck sold and refused with a "next rebirth" line at stage 9 (spec error, fixed) | [[Potions]], [[Rarity Gating and Rebirth]] |
| P6-6 | medium | The 150K pet slot adds no money once pets are at the limit | [[Pets]] |

Spec errors fixed by this path: [[Rarity Gating and Rebirth]] rule 10, U5, U7, U10 and the code plan; [[Potions]] rule 26 and UI 1.

Payoffs: the last colours, the "Every rarity is unlocked" finish line, the biggest merge numbers of the career, the last mine floor, the complete Index, stage-scaled royal rewards that make the crown the endgame, and the Transcendent pets.
