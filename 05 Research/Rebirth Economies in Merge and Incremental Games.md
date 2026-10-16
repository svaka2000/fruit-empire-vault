---
tags: [research]
topic: Rebirth (prestige) economy design for merge, idle and incremental games, and how to apply it to Fruit Empire
date: 2026-10-04
---

# Rebirth Economies in Merge and Incremental Games

Related: [[Brain Dump 2026-10-04]] (source of truth for what Samarth wants).

## Summary

- **Costs grow exponentially and income grows slower. That gap is the whole engine.** The standard cost formula is `cost = base × r^owned`. The growth rate `r` is usually between 1.07 and 1.15 [1][6]. Cookie Clicker uses 1.15. Clicker Heroes uses 1.07 [6].
- **Endless-prestige games shrink the reward with a root or a log.** AdVenture Capitalist pays `150 × sqrt(lifetime / 10^15)` angels, and each angel adds +2% income [3][8]. Cookie Clicker needs `X^3` trillion cookies for prestige level X, and each level adds +1% [9]. Pecorella's rule of thumb: players reset when the reset would raise their prestige currency by about +50% to +200% [4].
- **The big Roblox hits use short, hand-made rebirth ladders, not endless formulas.** Steal a Brainrot has 19 rebirths. Each costs cash plus specific brainrots. Almost every rebirth gives +1 slot and +10 s of base lock. The multiplier climbs by about 1 per rebirth [10]. Cash cost grows about 4x per rebirth (geometric mean, computed from [10]).
- **Rebirth growth mostly comes from unlocking new tiers, not from the multiplier.** In Steal a Brainrot the multiplier grows in a straight line but costs grow about 4x per rebirth. New rarities (about 10x more income each) carry the growth [10][11]. Merge a Mini Army and Merge a Nuke work the same way: rebirth raises the merge cap and adds a money multiplier [23][24].
- **Top Roblox games made their resets softer after launch.** Steal a Brainrot now takes only the required brainrots, not your whole base. It also lowered 8 rebirth costs and raised 2 [10]. Grow a Garden changed Ascension from "lose all Sheckles" to "pay the cost and keep the rest". It also cut the cooldown from 24 h to 4 h [18][19].
- **Merge math sets a hard floor on how fast value must grow.** In a merge-2 game (2 items make 1 of the next tier), a tier-L item costs `2^(L-1)` base items [25]. So each fruit tier should be worth at least 2x the one below it. Otherwise merging loses money. This makes the jump between rarity stages much steeper than in Steal a Brainrot (my math, shown below).
- **Side systems stay fair when they multiply the main income or are pinned to the player's current tier.** Flat amounts do not stay fair. Precedents: AdVenture Capitalist Newspapers multiply other businesses [4], Grow a Garden mutations multiply crop price [22], and Pet Simulator 99 rebirths add +75% pet strength per tier [14].
- **Fruit Empire already has a rebirth** (local code). It needs a full Index, wipes the run including the Index, keeps shards and tools, pays the Index shards again, and gives +100% cash per rebirth. It has no rarity cap and no storage growth yet. Its stage jumps are very uneven (45x to 7,240x), and a full Index pays only 26,528 shards (section 9).
- **Measure, don't guess.** Roblox's own docs say total currency sources minus sinks should stay near zero. They also say to watch the average wallet balance [30]. Funnel events can show where players stall on the way to each rebirth [31].

## How it works

### Terms used in this note

- **Rebirth / prestige**: you reset most progress (cash, items on your plot) for a permanent bonus that makes the next run faster [1].
- **Prestige currency**: a currency you get only by resetting. It controls the bonus (angels, heavenly chips) [4].
- **Generator**: anything that makes money over time. In Fruit Empire, each fruit on the plot is a generator [1].
- **Multiplier**: a fixed boost to income, like "x2 profit" [4].
- **Sink**: something that removes currency from the game (purchases, fees). **Source**: something that adds it [29].
- **Stage**: Samarth's word for the rarity tier of the best fruit a player can reach (Common to Transcendent) [Brain Dump].
- **Payback time**: the item's price divided by its income per second. It is how long an item takes to earn back its cost (my definition, used for the calculations below).

### 1. The core seesaw: exponential costs, slower income

- Cost of the next copy of an item: `cost_next = cost_base × r^owned` [1].
- Income: `production = base_production × owned × multipliers` [1]. This is linear in the number owned. An exponential cost always beats linear or polynomial income in the end [4].
- Example: an AdVenture Capitalist Lemonade Stand has base cost 4, `r = 1.07` and 1.67 income/s. The 11th stand costs `4 × 1.07^10 = 7.87` [1].
- Typical `r` values: Clicker Heroes 1.07 for all heroes, Cookie Clicker 1.15 for all buildings, AdVenture Capitalist 1.07 to 1.15 by business, and Steam's Monster game up to 2.5 [6]. The base cost of each new generator tier is often about half an order of magnitude (about 3x) above the last [6].
- Eric Guan's example uses x1.15 cost and x1.1 production per level [7]. Derivative Clicker uses `5 × 1.1^n` [2].
- Buying `n` at once when you own `k`: `total = b × r^k × (r^n − 1) / (r − 1)` [1].
- Most you can afford with currency `c`: `max = floor( log( c(r−1) / (b × r^k) + 1 ) / log(r) )` [1].
- **Milestones** are multipliers at set counts. They help income keep up with costs and give small wins [4]. AdVenture Capitalist doubles a business at 25 and 50 owned [1]. Clicker Heroes gives x4 for every 25 of a hero [4].

### 2. Prestige formulas in endless idle games

| Game | Prestige currency formula | Bonus per unit | Earn this much more to double the currency |
|---|---|---|---|
| AdVenture Capitalist | `150 × sqrt(lifetime_earnings / 10^15)` [3] | +2% income per angel, added together. 50 angels = x2, 1000 angels = x21 [8] | about 4x [3] |
| Realm Grinder | `(sqrt(1 + 8 × max_earnings/10^12) − 1) / 2` [3] | not covered | 4x [3] |
| Cookie Clicker | `cbrt(lifetime / 10^12)`. Level X needs `X^3` trillion cookies [3][9] | +1% CpS (cookies per second) per level [9] | about 8x [3] |
| Egg, Inc. | `(earnings_this_run / 10^6)^0.14` [3] | not covered | 128x [3] |

What the Kongregate/GDC material recommends [4]:
- Use a log or a fractional power (like a square root) so prestige currency grows much slower than money.
- Players tend to reset when it would add about **+50% to +200%** prestige currency.
- Make sure players hit a worthwhile reset point **regularly**.
- The start of each new run should be **fast**. Feeling yourself rip through old content is the reward. Pecorella praises Nonstop Knight's speed-up through early content after a reset.
- Runs can get faster, slower or vary. If runs get slower, you need a meta prestige (a second, higher reset layer), or the game becomes tiresome.
- One of his example models used a 5x boost per prestige. That is a modeling choice, not a rule.

### 3. How the Roblox hits do rebirth (short, hand-made ladders)

**Steal a Brainrot** (19 rebirths, current wiki table) [10]:

| Rebirth | Cash needed | Multiplier | Other rewards |
|---|---|---|---|
| 1 | 500K (was 1M) | x0.5 | +10 s lock, gear |
| 2 | 1.5M (was 3M) | x1 | +1 slot (2nd floor) |
| 3 | 12.5M | x2 | +1 slot |
| 5 | 100M | x4 | +1 slot |
| 10 | 125B (was 250B) | x9 | +1 slot (3rd floor) |
| 15 | 100T (was 500T) | x15 | +1 slot |
| 19 | 30Qa | x19 | +1 slot |

- Every rebirth also needs 1 or 2 specific brainrots. Those brainrots get more valuable each step: Mythic, then Brainrot God, then Secret [10].
- Every rebirth gives +10 s of base lock (a timed shield). Every rebirth except the 1st gives +1 slot. Each also gives a cash bonus, from $5K at rebirth 1 up to $250T at rebirth 19 [10].
- **Cost growth (computed from [10]):** step ratios run from 2x to 10x. The median is 3.5x. The geometric mean over all 18 steps is about **3.97x per rebirth**, and it is about the same in the first and second half.
- **Multiplier growth:** about +1 per rebirth, a straight line [10]. Sources disagree on whether "x0.5" means a +0.5x bonus or the total multiplier. Game Rant writes it as "+0.5x" [12]. Beebom lists different numbers [13]. Not confirmed.
- **Changes after launch:** the wiki shows 8 rebirth costs lowered (rebirths 1, 2, 9, 10, 11, 12, 14, 15) and 2 raised (rebirths 3 and 4). Before Update 66, rebirth took every brainrot in your base. Now it takes only the required ones. One required brainrot (Ballerino Lololo) was dropped from the requirements because it spawned too rarely [10].
- What resets: required brainrots and most cash. You keep coin-shop gear unlocks, multipliers and base upgrades [10][13].

**Pet Simulator 99** (BIG Games' official database) [14]:
- 9 rebirth tiers. They unlock at areas 25, 50, 75, 99, 125, 150, 175, 199 and 219.
- Each tier adds **+75% pet damage**, stacking in a straight line: +75%, +150% ... up to +675%.
- Rebirth 1 also unlocks features: teleport, auto-hatch and the Castle Key (trading).
- You keep pets and unlocks. Coins and area progress reset [15].

**Grow a Garden "Garden Ascension"** (added Aug 2025):
- At launch: 1 trillion Sheckles plus 1 to 3 fruits (sometimes with a mutation). Reward: 10 Garden Coins. Ascending reset all Sheckles [16][17].
- Update of 9 Sep 2025: the cost is 1T, then 3T, 5T, 7T (+2T each time, a straight line). You keep any Sheckles left over. The cooldown dropped from 24 h to 4 h. Fruit and mutation requirements got easier [18][19].
- Garden Coins buy capacity, not power: 1 save slot (100 coins), plus egg, pet, plant and cosmetic slots (40 or 20 coins each, up to 5 times), plus fences [18].
- A fan site says Ascension was added to drain inflated Sheckles from players holding "quintillions" [20]. The developers have not confirmed this.
- Scale context: the game peaked at about 22.3M concurrent players on 23 Aug 2025 [21].

**Merge a Mini Army** (Roblox merge game, article dated 16 Aug 2026) [23]:
- Unit tiers are powers of two. "Tier 4.19M" is 2^22, and the top unit at the time, "Tier 2.14B", is 2^31.
- Rebirth 1 needs the 22nd unit in the index. It gives **+100% cash** and unlocks the next 5 tiers.
- Rebirth 2 needs the 27th unit. It gives **+200% cash** and 5 more tiers.
- Rebirth resets cash, upgrades and units. A cash "Spawn Tier" upgrade raises the tier of units that spawn.

**Merge a Nuke** (the game Samarth named for health drain) [24]:
- Nukes earn income based on their tier. Upgrades: spawn tier, max spawn (base capacity) and lock-base time. Base lock has a 30 s cooldown.
- Rebirth resets nukes, cash and upgrades. It gives permanent money multipliers and access to higher-tier nukes. Only 3 rebirths existed when the Sportskeeda article was written (date not shown).
- **Newer detail, from [[Merge a Nuke - Explosives and Health Drain]]:** a guide site says each rebirth checks the highest nuke you actually own (T30, then T35, T40 ...), raises the merge cap by 5 tiers, and adds +25% cash. The Fandom wiki lists 12 rebirths, and its Nukes page has sections up to Rebirth 14. Nuke damage = 300 × 1.0905^(tier − 1), and a nuke's HP is about 2 × its own tier's damage (the guide calls the HP an estimate). These come from fan and guide sites, not from the developer.

### 4. Income scaled to stage (rarity tiers)

Steal a Brainrot income per second by rarity [11], with payback times I computed:

| Rarity | Income/s range | Price range | Payback (price ÷ income) |
|---|---|---|---|
| Common | $1 to $14 | $25 to $1.7K | 25 s to 121 s |
| Rare | $15 to $75 | $2K to $9.7K | about 130 s |
| Epic | $75 to $325 | $10K to $47.5K | about 135 to 145 s |
| Legendary | $200 to $1.9K | $35K to $347.5K | about 175 to 185 s |
| Mythic | $2K to $18.7K | $350K to $6.2M | about 175 to 330 s |
| Brainrot God | $19K to $323K | $5M to $47.5M | about 150 to 265 s |
| Secret | $315K to $425M | $80M to $425B | about 250 to 1000 s |
| OG | $450M to $1B | $450B to $1T | about 1000 s |

- Income jumps by roughly one order of magnitude per rarity. Between the low ends of each range the jump is 3x to 17x [11].
- **Price rises in step with income.** So the time to earn back a purchase stays around 2 to 5 minutes across most tiers. Only the top tiers stretch toward about 17 minutes (computed from [11]).
- Grow a Garden uses multipliers on top of crop value. Mutations multiply a crop's sell price (Chilled x2, for example) [22]. So a mutation is worth more on a better crop, and it scales with stage by itself.

### 5. Merge-specific math

- In a merge-2 game, one level-L item is worth `2^(L−1)` level-1 items. Raising a request by one level doubles what it costs in base items [25].
- Board efficiency is the share of what you produce that actually moves you forward. Clutter and blocked merges lower it [25].
- If a boost would roll an item above the max level, it gets "clipped". The extra value is wasted. The authors suggest refunds, or boosts the player can switch on per generator [26].
- **My math (not from a source):** let fruit tier k be worth `v(k) = v1 × g^(k−1)`.
  - Two tier-k fruits earn `2 × v(k)`. Merged, they earn `g × v(k)` and free one storage slot.
  - With `g = 2`, merging keeps income the same and only saves space. With `g > 2`, merging also raises income. With `g < 2`, merging loses money.
  - If each rarity has `F` fruits, the next rarity's first fruit takes `2^F` copies of this rarity's first fruit. It earns `g^F` times more.
  - So the income jump per rarity is **at least 2^F**. With F = 8 and g = 2.4, that is about 1,100x per rarity. That is much steeper than Steal a Brainrot's ~10x, and it has to be. Merging forces it.
  - With 64 fruits and g = 2.4, the top fruit is worth about 10^24 × the first fruit. Luau numbers are doubles, which go up to about 1.8 × 10^308, so this fits. You will need number suffixes (K, M, B, T, Qa...).

### 6. Keeping older content useful (avoiding dead content)

- Players tend to ignore small generators that barely matter, even when they are technically "optimal" [4].
- Pecorella's fixes [4]:
  - Use strong multipliers.
  - Give bonuses for owning things. AdVenture Capitalist Newspapers multiply other businesses.
  - Tie prestige currency to how much you own. Clicker Heroes gives 1 prestige currency per 2,000 generators owned.
  - Use milestone multipliers based on count.
- AbyssRium ties milestone multipliers to buying fish, which builds a collection system into the economy [4].
- A Quantic Foundry survey of players of 3 idle games found their top motivations were **Completion** and **Power** [4]. A collection book (Index) fits this genre.
- A small indie simulation study found that a new mode which only speeds you up made total playtime *shorter* (3.7 h down to 3.1 h). A new system needs goals of its own [34].

### 7. Inflation and sinks

- Track sources and sinks. Their difference should stay near zero. If the average wallet keeps growing, add sinks [30].
- Before launching a random reward, work out its expected value (EV). EV is each outcome's value times its chance, added up. Underestimating an event's payout floods the economy [29].
- Keep event rewards in a separate event currency so the main economy is not touched [29].
- Endless rebirth with a fixed percent increase explodes. `1.1^n` passes 10^400 by 10,000 rebirths, and `1.05^n` passes 10^200 [27]. The fix in that thread: start at +5% per rebirth and halve the percent at 100, 200, 400, 800 ... rebirths [27].
- On the DevForum, a curved (parabolic) cost curve was suggested over a straight line for a "slow start, big growth later" feel. Upgrades should feel earned, not handed out [28].

### 8. Pacing: time to the next rebirth

- Confirmed guidance is qualitative. Reset points should come regularly, the start of each run should be fast, and slower runs need a meta layer [4].
- Lower-confidence numbers from studio and blog posts:
  - Well-tuned prestige gives about a 2x to 4x speedup on the second run [32].
  - Players reset when progress slows to about 10% to 20% of peak speed [33].
  - Offline earnings caps are often 2 to 24 h [32].
- Pecorella says Egg, Inc.'s 2-hour offline cap was a mistake. He says he quit largely because of it [4].
- Public time-per-rebirth data for Steal a Brainrot, Pet Simulator 99 or Grow a Garden: **not confirmed**. I found requirements, but no measured times.
- Pecorella shared his balancing spreadsheets publicly (link in slides [4]). Roblox funnel events can log each step on the way to a rebirth and show where players stall [31].

### 9. What Fruit Empire already has (local code, 2026-10-04 export)

The game already ships a rebirth. Everything below changes it rather than building one from nothing. [35][36][37][38]
- **Gate:** every fruit in the Index must be discovered (`RebirthNeedsAll = true`). There is no cash or shard cost yet.
- **What resets:** fruit on the plot, stored fruit, the whole Index, every upgrade level, plot size, cash and "Best Merge".
- **What stays:** shards and tools (TNT and potions, because they were bought with shards).
- **Reward:** the shards the Index already paid are paid a second time. Cash becomes x(1 + rebirths): x2 after one, x3 after two (`RebirthCashBonus = 1.00`, added, not multiplied).
- **No rarity cap.** A player at 0 rebirths can merge all the way to Soul Fruit today. The brief's "max fruit Apple at 0 rebirths" is new work.
- **Storage** is a flat 5 (`StoredCap = 5`). The brief's "+1 per rebirth" is new work.
- **Index rewards:** each fruit pays 60 s of its own income in cash once, a shard amount from 5 (commonest) to 2,500 (rarest) on a smooth curve, and a share of a +50% coin bonus for the full Index.

**The real fruit ladder** (from `FruitEconomy` and `FruitIndex`) [35][36]:
- 64 fruits in 9 tiers: Common 5, Uncommon 8, Rare 11, Epic 8, Legendary 7, Mythic 8, Astral 5, Eternal 8, Transcendent 4.
- Each rank earns 2.10x the one below at the bottom of the ladder, rising smoothly to 2.60x at the top. So every merge already pays more than 2x (section 5).
- **The jump between stages is very uneven,** because the tiers have different sizes (my math, top fruit of one tier to the top fruit of the next):

| From → to | Income jump | Copies of the stage's first fruit needed for its top fruit |
|---|---|---|
| Common → Uncommon | 487x | Common: 16 |
| Uncommon → Rare | 7,240x | Uncommon: 128 |
| Rare → Epic | 834x | Rare: 1,024 |
| Epic → Legendary | 429x | Epic: 128 |
| Legendary → Mythic | 1,240x | Legendary: 64 |
| Mythic → Astral | 96x | Mythic: 128 |
| Astral → Eternal | 1,740x | Astral: 16 |
| Eternal → Transcendent | 45x | Eternal: 128 |
| | | Transcendent: 8 |

- So the Rare stage needs about 8 times as many merges as the stages around it, and Astral and Transcendent are short. Rebirth costs and pacing targets have to follow this table, not a single smooth ratio.

**Shard supply** (from `IndexRewards`) [37]:
- A full Index pays 26,528 shards. By stage, counting everything up to and including that stage: Common 31, Uncommon 125, Rare 468, Epic 1,078, Legendary 2,203, Mythic 4,903, Astral 8,103, Eternal 17,853, Transcendent 26,528.
- Other sources: Crown rewards (100 to 3,000, only for players who fight), and the "Merge Rush" decree (400 to each player who merged, at most once per 6 minutes) [38].
- With rarity gating and today's "wipe the Index and pay it again" rule, Index shards add up to about 62 after rebirth 1, 1,248 after rebirth 3, 3,404 after rebirth 4, 69,528 after rebirth 8, and 96,056 with the Transcendent page full (my math).

## What Fruit Empire should take

Stealing is removed, so nothing below relies on taking other players' fruit. Numbers marked **(proposal)** are my starting values for playtesting, not sourced facts.

1. **One rebirth = one new rarity stage.** This is Samarth's design, and it matches Merge a Mini Army and Merge a Nuke [23][24].
   - With 0 rebirths, the merge cap is Apple (top of Common). Rebirth 1 opens Uncommon, and so on up to Transcendent.
   - The Index shows locked rarities greyed out with the rebirth number that opens them. Then "rebirth to unlock more fruit" is obvious.

2. **Fruit value ladder: keep the existing one.** The game already uses a step that ramps from 2.10 to 2.60 (section 9), which meets the "above 2" rule (section 5). An earlier draft of this note proposed a single `g` of 2.2 to 2.5. That is not needed. The thing to plan around is the uneven stage jumps in the section 9 table (45x to 7,240x), not the step size.

3. **Spawn tier must climb inside a run.** Spawning only tier-1 fruit makes high tiers impossible: 2^63 merges to reach fruit 64.
   - Copy the "Spawn Tier" upgrade from Merge a Nuke and Merge a Mini Army [23][24].
   - Price it `base × r^owned` with r = 1.10 to 1.15 [1][6].
   - After a rebirth, let the old rarities be cheap to skip. This gives Pecorella's "fast start of each run" [4].
   - Optional perk (proposal): rebirth n starts spawning at the first fruit of rarity n−1.

4. **Rebirth requirements: a hand-made table, built from a formula.**
   - **Index:** every fruit in the unlocked rarities. Make every entry reachable by merging, never pure luck. Steal a Brainrot dropped a too-rare requirement [10]. If the planned gacha+pity merge RNG ships, pity must guarantee each Index entry.
   - **Cash:** `C_s = top-of-stage income/s × T_s seconds`. The cost then follows income. The ratio between rebirths becomes about `g^F × (T_{s+1}/T_s)`. Do not copy Steal a Brainrot's average ~4x per rebirth [10]. Its rarities are only ~10x apart, and ours will be about 2^F apart or more.
   - **Shards:** charge a share of the shards that stage's Index pays out. The supply is now known (section 9). Rebirth 1 can only ask for a few dozen shards, because a full Common page pays 31. A rebirth must never need more shards than the player can have earned by then.
   - "Hard to rebirth" should come from the **Index and cash time**, not from luck or grinding a cooldown. Grow a Garden had to cut its cooldown from 24 h to 4 h [19].

5. **What a rebirth gives (in order of weight):**
   - (a) The next rarity: the real growth [23].
   - (b) **+1 storage**. Steal a Brainrot gives +1 slot per rebirth [10]. Grow a Garden spends its Ascension currency on capacity [18].
   - (c) A small permanent cash bonus, **+50% per rebirth, added (proposal)**. The live game gives +100% per rebirth, added (section 9), and the brief asks for "a slight boost". For comparison: Pet Simulator 99 gives +75% per tier [14], Steal a Brainrot about +1x [10], Merge a Mini Army +100% then +200% [23]. Added bonuses shrink in relative terms (x9 to x10 is only +11%). That is fine because (a) carries the growth.
   - (d) A bat tier whose color matches the new rarity.
   - (e) Defense that scales with rebirth, like Steal a Brainrot's +10 s lock per rebirth [10]. Example: more fruit max health against TNT drain, or more shield seconds.
   - (f) A starting cash gift scaled to the stage. Steal a Brainrot's grows from $5K to $250T [10].

6. **Soft reset: decide clearly what is kept.** Proposal:
   - **Reset:** fruit, cash and cash upgrades.
   - **Keep:** Index, shards, pets, underground base, boat, bat and storage.
   - Both top Roblox hits moved this way after launch [10][19]. Pet Simulator 99 keeps pets [15].
   - **Conflict with the live code:** today a rebirth wipes the Index and pays its shards again (section 9). Keeping the Index would remove that payout and cut total shard supply to 26,528 for the whole game. If the Index is kept, "fill out your index" in the brief means filling the newly unlocked pages only. Decide this together with the shard prices.

7. **Pets: pin their income to the current stage.**
   - Samarth's rule: a rarer animal is tamed with, and pays like, the rarity of your best fruit.
   - Formula (proposal): `pet_income = pet_factor(level) × v(best fruit on plot this run)`. Or make pets a % boost on fruit income. AdVenture Capitalist Newspapers [4] and Grow a Garden mutations [22] are the models.
   - After a rebirth, the best fruit falls back to Common, so pets cannot flatten the early game. They grow back as the run goes on.
   - **Taming payback:** the fed fruit is a sink. A pet should earn back that fruit's income in about 2 to 5 minutes. That borrows Steal a Brainrot's typical payback (computed from [11]).
   - **Merging pets:** use the same merge-2 rule (2 make 1 bigger). Use g > 2 per level so merging pays. Cap pets' total share of income, for example at 25% to 50% of fruit income (proposal), so fruit stays the main game.

8. **Ores: tie the ore ladder to the fruit ladder.**
   - Each new rarity adds ore tiers. Formula (proposal): an ore tier opened at stage s is worth about `v(first fruit of stage s) × g_ore^(j−1)`.
   - Coal to Iron is the stage-1 pair. Glowing ores belong to higher rarities.
   - The underground costs 100k shards. With Index shards only, that is more than a player has even with the Transcendent page full (about 96k, section 9). Either add a shard source or lower the price. Then make ore income matter at the stage where players can actually afford it.

9. **Shop upgrades:** use `base × r^owned` with r = 1.07 to 1.15 [1][6]. Add milestone doublings, like AdVenture Capitalist's x2 at 25 and 50 [1]. These are the small wins between rebirths.

10. **Pacing targets (proposal):**
    - First rebirth after about 45 min of active play.
    - Each later stage takes about 1.4x as long as the one before.
    - With 8 rebirths (Common up to Eternal; Transcendent is the last stage) that is about 26 h of play in total.
    - The merge counts in section 9 are uneven (the Rare stage is about 8x longer than its neighbours). Spawn Tier prices per stage have to absorb that, or the 1.4x curve will not hold.
    - Check it by simulation before shipping (Pecorella's sheets [4]). After launch, check it with a Roblox progression funnel that logs each rebirth [31].

11. **Artillery and TNT in the economy.**
    - Price stronger TNT relative to the stage, not as flat cash.
    - Stealing is removed, so destroying fruit gives the attacker no loot. Each TNT is a pure sink for the attacker and a loss for the victim.
    - Health drain gives the victim time to react. Merge a Nuke's lock/cooldown is a reference [24].

12. **After Transcendent:** if rebirths continue, use a root-based formula like AdVenture Capitalist's [3][8], or a shrinking percent per rebirth [27]. Never use a flat `1.1^n`.

## What to avoid

- **A fixed percent cost growth on endless rebirths.** It goes past 10^400 in a few thousand steps [27].
- **Flat cash rewards for pets, ores or quests.** They are huge at stage 1 and worthless at stage 6. Use multipliers or stage-pinned values [4][22].
- **Rebirth requirements that depend on luck.** Steal a Brainrot removed a too-rare requirement [10].
- **Wipe-everything resets.** Steal a Brainrot and Grow a Garden both softened theirs [10][19].
- **Leaning on the rebirth multiplier for growth.** In Steal a Brainrot it grows by about +1 per rebirth while costs grow about 4x [10]. The new rarity has to carry the growth.
- **Wasted boosts at the merge cap.** A potion or boost that would create a fruit above the rebirth cap should refund or convert, not vanish [26].
- **Value per tier below 2x in a merge-2 game.** Merging would then lose income (section 5).
- **Slow cycles with no meta goal.** Pecorella says this gets tiresome [4].
- **Long real-time cooldowns as the main gate.** Grow a Garden cut its cooldown from 24 h to 4 h [19].
- **Harsh offline caps.** Pecorella says he quit Egg, Inc. largely over its 2 h cap [4].
- **Unlimited shard sources against fixed shard prices** (boat 1,000, underground 100k). If shards keep coming in, prices lose meaning over time [30].

## Open questions

- ~~How many rarities sit between Common and Transcendent, and how many fruits does each have?~~ Answered in section 9: 9 tiers with 5/8/11/8/7/8/5/8/4 fruits.
- ~~Are shards earned once per Index entry or over and over?~~ Answered in section 9: once per run, and a rebirth pays the claimed amount again. Whether to keep this rule is open (proposal 6).
- Do pets, ores and the underground carry through a rebirth? (The proposal above says yes.)
- The live game already has a rebirth (section 9). Players who rebirthed under the old rule, and players holding fruit above their new rarity cap, need a migration rule. The Roblox API shows 119 lifetime visits on 2026-10-04 [39], so few players are affected, but the rule is still needed.
- How do offline earnings interact with rebirth pacing?
- What does Steal a Brainrot's "x0.5 ... x19" multiplier actually mean: a bonus or a total? Sources disagree [10][12][13].
- Merge a Nuke's rebirth multipliers, rebirth count and nuke health formula now have guide-site and fan-wiki numbers (section 3). None come from the developer, so they are still not confirmed.
- How long players actually take per rebirth in Steal a Brainrot, Grow a Garden or Pet Simulator 99: not confirmed. No public data found.

## Sources

1. Anthony Pecorella, "The Math of Idle Games, Part I" (Kongregate, via Game Developer), Oct 2016. https://www.gamedeveloper.com/design/the-math-of-idle-games-part-i
2. Anthony Pecorella, "The Math of Idle Games, Part II" (Kongregate). https://www.kongregate.com/en/pages/the-math-of-idle-games-part-ii
3. Anthony Pecorella, "The Math of Idle Games, Part III" (Game Developer). https://www.gamedeveloper.com/design/the-math-of-idle-games-part-iii
4. Anthony Pecorella, "Quest for Progress: The Math and Design of Idle Games", GDC Europe 2016 slides. https://media.gdcvault.com/gdceurope2016/presentations/Pecorella_Anthony_Quest%20for%20Progress.pdf
5. GDC Vault talk page for the same talk. https://www.gdcvault.com/play/1023876/Quest-for-Progress-The-Math
6. Alexander King, "Numbers Getting Bigger: The Design and Math of Incremental Games" (Envato Tuts+), Jun 2015. https://code.tutsplus.com/numbers-getting-bigger-the-design-and-math-of-incremental-games--cms-24023a
7. Eric Guan, "Idle Game Design Principles" (Substack), Jun 2025. https://ericguan.substack.com/p/idle-game-design-principles
8. AdVenture Capitalist Wiki, "Angel Investors". https://adventure-capitalist.fandom.com/wiki/Angel_Investors
9. Cookie Clicker Wiki, "Ascension". https://cookieclicker.fandom.com/wiki/Ascension
10. Steal a Brainrot Wiki, "Rebirth" (table through Rebirth 19). https://stealabrainrot.fandom.com/wiki/Rebirth
11. Steal a Brainrot Wiki, "Rarities". https://stealabrainrot.fandom.com/wiki/Rarities
12. Game Rant, "Roblox Steal a Brainrot Rebirth Guide" (updated Nov 29, 2025). https://gamerant.com/roblox-steal-a-brainrot-rebirth-guide/
13. Beebom, "Steal a Brainrot Rebirth Guide" (updated Nov 24, 2025). https://beebom.com/steal-a-brainrot-rebirth-guide-levels-and-rewards/
14. BIG Games database, "Rebirths" (Pet Simulator 99). https://db.biggames.io/wiki/rebirths
15. HoldToReset, "Pet Simulator 99 Rebirth System Explained" (updated Mar 11, 2026). https://holdtoreset.com/pet-simulator-99-rebirth-system-explained/
16. Grow a Garden official X account, Garden Ascension announcement. https://x.com/GrowaGardenRblx/status/1961794089285722260
17. Beebom, "Grow a Garden Ascend Guide" (Aug 30, 2025). https://beebom.com/grow-a-garden-ascend-guide-how-to-rebirth/
18. Game Rant, "How to Get Garden Coins in Grow a Garden" (Sep 11, 2025). https://gamerant.com/how-get-garden-coins-grow-a-garden-garden-ascension-shop-guide/
19. Finding Dulcinea, "Grow a Garden Ascension Update" (Sep 12, 2025; update released Sep 9, 2025). https://www.findingdulcinea.com/grow-a-garden-ascension-update/
20. GrowAGardenPro (fan site), "Rebirth System". https://growagardenpro.com/gamefeatures/rebirthmechanic/
21. Wikipedia, "Grow a Garden". https://en.wikipedia.org/wiki/Grow_a_Garden
22. Grow a Garden Wiki, "Crop Mutations". https://growagarden.fandom.com/wiki/Crop_Mutations
23. Sportskeeda, "Merge a Mini Army Rebirth guide" (Aug 16, 2026). https://www.sportskeeda.com/roblox-news/merge-a-mini-army-rebirth-guide
24. Sportskeeda, "Merge a Nuke: A beginner's guide" (date not shown). https://www.sportskeeda.com/roblox-news/merge-nuke-a-beginner-s-guide
25. Game Economist Consulting, "The Economics of Merge-2 Games" (Mar 25, 2026). https://www.gameeconomistconsulting.com/the-economics-of-merge-games/
26. Phillip Black, Game Economist Consulting, "What Merge-2 Economics Is Missing" (Mar 25, 2026). https://www.gameeconomistconsulting.com/more-merge-2-economics/
27. Roblox DevForum, "Good formula to calculate amount needed to rebirth" (Feb 2021). https://devforum.roblox.com/t/good-formula-to-calculate-amount-needed-to-rebirth/1047686
28. Roblox DevForum, "Proper scaling of stats/currency and prices for a simulator game" (Jul 2021). https://devforum.roblox.com/t/proper-scaling-of-statscurrency-and-prices-for-a-simulator-game/1354994
29. Roblox Creator Docs, "Balance virtual economies". https://create.roblox.com/docs/production/game-design/balance-virtual-economies
30. Roblox Creator Docs, "Economy events". https://create.roblox.com/docs/production/analytics/economy-events
31. Roblox Creator Docs, "Funnel events". https://create.roblox.com/docs/production/analytics/funnel-events
32. Dmytro Lunov, Game-Ace, "Idle game development: mechanics, cost, and LiveOps" (updated Sep 9, 2026). Studio blog, lower confidence. https://game-ace.com/blog/idle-game-development/
33. GridInc Blog, "Idle Games Best Practices" (Jan 28, 2025). Lower confidence. https://gridinc.co.za/blog/idle-games-best-practices
34. DEV Community, "I set an 18-hour playtime target for my idle game. A simulation says it has 3.7". Indie blog, lower confidence. https://dev.to/dhseadev/i-set-an-18-hour-playtime-target-for-my-idle-game-a-simulation-says-it-has-37-32a4
35. Local code: `~/fruit-empire/src-export-2026-10-04/ReplicatedStorage.FruitEconomy.lua` (Base 1, StepLow 2.10, StepHigh 2.60)
36. Local code: `~/fruit-empire/src-export-2026-10-04/ReplicatedStorage.Fruits.FruitIndex.lua` (64 fruits, tiers 1–9)
37. Local code: `~/fruit-empire/src-export-2026-10-04/ReplicatedStorage.IndexRewards.lua` and `ReplicatedStorage.GardenConfig.lua` (IndexBonusTotal 0.50, IndexMoneySeconds 60, IndexShardsLow 5, IndexShardsHigh 2500, StoredCap 5, RebirthNeedsAll, RebirthCashBonus 1.00)
38. Local code: `~/fruit-empire/src-export-2026-10-04/ServerScriptService.GardenService.RebirthLock.lua` (`doRebirth`) and `ReplicatedStorage.CrownConfig.lua` (`C.Rewards`, decrees)
39. Roblox Games API, Fruit Empire (read 2026-10-04): https://games.roblox.com/v1/games?universeIds=10764283455

## Review (critic pass, 2026-10-04)

- **Fixed:** the Merge a Nuke section said its multipliers and health formula were not confirmed. The sister note has guide-site numbers, now summarised here (still labelled as fan sources).
- **Added:** section 9, the live rebirth, the real fruit ladder, the uneven stage jumps and the shard supply, all from the local code [35]–[38]. Three open questions are now answered.
- **Changed:** proposal 2 (keep the existing 2.10→2.60 ladder) and proposals 4, 5, 6, 8 and 10 to match the code and the shard math.
- **Verified:** Grow a Garden's 22.3M peak on 23 Aug 2025 (RTC and esports.gg both report 22,346,725).
- **Still not confirmed:** what Steal a Brainrot's "x0.5 ... x19" means, and real time-per-rebirth data for any Roblox hit.
