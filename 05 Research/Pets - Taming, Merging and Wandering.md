---
tags: [research]
topic: "Pet systems in top Roblox games: taming by feeding, merging duplicates, pets wandering a base, pet income, rarity gating, equip limits, performance"
date: 2026-10-04
---
# Pets: taming, merging and wandering

Source brief: [[Brain Dump 2026-10-04]] ("in the forest we will add pets, merging two of the same pets makes the pet bigger and makes you more money, you can equip the pets and they will walk around your base").

**Terms used in this note**
- **Tame**: turn a wild animal into a pet you own.
- **Equip**: put an owned pet into the world so it follows you or walks around your plot. An **equip slot** is one space for an equipped pet. Unequipped pets sit in a list and do nothing.
- **Merge / fuse**: combine several copies of a pet into one stronger copy.
- **Rarity tier**: a quality band, like Common, Rare, Legendary.
- **BPS / $/s**: money earned per second.
- **Hunger / bond**: meters some games use. They go down over time and you refill them by feeding or caring for the pet.
- **Run-best fruit** (a new term for Fruit Empire): the highest fruit a player has made since their last rebirth.

## Summary
- **Feeding-to-tame is proven, and 99 Nights in the Forest does it best.** You play a short flute minigame. Then the animal asks for specific food, and you feed it once per "stage". Stronger animals need a better flute and up to 5 stages [2][3]. This is the closest match to "feed it the requested fruit".
- **Feeding fruit to pets is already normal on Roblox.** Grow a Garden pets eat harvested crops [1]. In Bee Swarm Simulator every bee has a favorite treat, and the treats are fruits (strawberry, pineapple, blueberry) [19].
- **Merging duplicates is common, but the ratios differ a lot.** Adopt Me fuses 4 identical full-grown pets into a Neon, then 4 Neons into a Mega Neon [9][10]. Pet Simulator 99 needs 10 identical pets for a Golden (about 13% stronger), then 10 Goldens for a Rainbow (about 27% stronger) [4][5]. Fruit Empire's 2-into-1 rule only works if every merge pays **more than 2x**. The game's own economy code already says so for fruit [41].
- **Pet income works best when it is flat cash on your base.** Catch and Tame pets earn $/s in your pen, scaled by rarity [11][16][17]. Steal a Brainrot units earn $/s in bands. The top of each band is about 4 to 17 times the top of the band below [25]. Grow a Garden pets do not pay cash. They boost crops instead [1].
- **Equip limits are small at the start and grow slowly.** Grow a Garden: 3 slots, up to 8 [1]. 99 Nights: 2 tamed animals, 3 with the best flute [2]. Catch and Tame: 8 animals in your pen at first [17]. Pet Simulator 99 is the outlier, going up to 99 [6][7].
- **Rarity is gated by a tool tier or by tasks, never by luck alone.** Catch and Tame compares lasso strength with animal strength [12]. 99 Nights uses flute tiers [3]. Adopt Me makes rarer pets need more tasks to grow: Common 25, Legendary 183 [10].
- **Performance rule from the DevForum and Roblox docs:** the server tracks which pets you own. Each client draws and moves the pets. Skip Humanoids, because Roblox's docs call them expensive to compute. Use AnimationController and AlignPosition or CFrame lerping instead [30][31][32][34][36][38].
- **Recommendation for Fruit Empire:** wild animals in the forest ask for a fruit from their own tier. You tame one by paying that fruit. The pet earns flat $/s worth more than the fruit it ate. Two identical pets merge into one bigger pet that earns more than both together. Equipped pets wander your plot. A pet naps (earns nothing) whenever its tier is above your run-best fruit, so rebirth still resets the economy. Details are below.

## How it works

### 1. Taming by feeding

**99 Nights in the Forest: the best reference**
- You need a Taming Flute. It has three tiers: Old, Good and Strong [3].
- Using the flute starts a minigame with a 30-second time limit. It works like the fishing minigame: you keep a bar inside a green zone. Harder animals have a smaller green zone [2].
- After the minigame the animal turns neutral and asks for food. Feeding it finishes one **stage**. The number of faces above its head shows how many stages are left. When all faces turn into pink hearts, the animal is tamed [2].
- Stages and foods scale with the animal [3]:

| Animal | Flute needed | Stages | Food per stage |
|---|---|---|---|
| Bunny | Old | 1 | 1 Carrot |
| Kiwi | Old | 1 | 1 Berry |
| Frog | Old | 2 | 1 Mackerel |
| Wolf | Old | 3 | 1 Steak + 1 Morsel |
| Arctic Fox | Good | 3 | 1 Steak + 1 Chili |
| Alpha Wolf | Good | 4 | 1 Steak + 1 Mackerel + 1 Salmon |
| Bear | Strong | 5 | 1 Steak + 1 Pumpkin + 1 Salmon |
| Mammoth | Strong | 5 | 2 Pumpkin + 1 Cake |

- Using the flute on a second animal mid-tame resets the first one. Everything already fed is lost [2].
- You can only have 2 tamed animals, or 3 if you carry a Strong Flute. A few classes are exempt from this cap [2].
- A tamed animal has 4 actions: Follow, Sit, Release and Rename [2].
- Feeding a tamed animal the food it was tamed with heals it, and gives the flute XP [3].
- The wiki's own verdict is blunt. It says taming takes a lot of time for a small reward, and tamed animals get stuck on obstacles easily [3]. This is the biggest lesson. **The payoff has to be worth the trip.**

**Catch and Tame: same theme, but you catch with a lasso, not food**
- You throw a lasso at a roaming animal. Then you tap to fill a taming meter. A tamed animal goes into your pen and earns money [11][17].
- Each lasso has a Strength number, and the game compares it with the animal's strength. Lassos run from Basic (10 strength, free) to Stormrend (3,600 strength, 3.75B cash) [12].
- Food does not tame here. It levels pets up to a maximum of level 10. Each level makes the pet slightly bigger and raises its breeding luck. Food also shortens the breeding cooldown. Some special foods add a cosmetic mutation [13].
- Scale signal: Catch and Tame had 462,964,134 visits when checked on 3 Oct 2026 [29].

**Grow a Garden: you feed fruit, but you do not tame**
- Pets come from eggs. Feeding is for hunger [1].
- To feed, you hold a harvested crop near your pet and press Feed [1].
- A full hunger bar empties in about 12 hours. At zero hunger the pet stops gaining XP and stops aging. Hunger size differs by pet and can depend on rarity [1].

**Bee Swarm Simulator: favorite treats are fruits**
- Bees hatch from eggs. Feeding treats raises **bond**, and bond levels the bee up [18].
- Each bee has a favorite treat. Feeding it the favorite gives a tiny chance to become "Gifted", which means better stats plus a hive-wide bonus. The odds are 1 in 8,000 for Rare bees, 1 in 10,000 for Epic, 1 in 12,000 for Common and Legendary, and 1 in 24,000 for Mythic [18][19].
- Treats include Strawberry, Pineapple, Blueberry and Sunflower Seed. Each treat is the favorite of about 8 or 9 bees [19].

**Adopt Me: pets ask for things**
- Pets get needs like Hungry, Thirsty, Sleepy and Dirty. Meeting a need counts as a task [10].
- Sometimes a pet asks for one specific item. Fulfilling that exact request pays 3x Bucks and XP [10]. This "thought bubble with a request" is the UI pattern Fruit Empire's wild animals need.

**Not feed-to-tame games (checked so I don't assume)**
- Dragon Adventures: dragons come from eggs and breeding. Bonding means bathing, brushing, petting and napping. Food of the "Bond" type also gives bond XP. The bond meter drops 3 points every 30 seconds while the dragon is equipped. Below 40 the dragon becomes "Lonely" [21]. A player post says the "Wild/Tame" label only shows whether a dragon was hatched or bred. That post is low confidence [22].
- Creatures of Sonaria is a creature survival game [23]. Feather Family is a role-play game where you play as a bird [24]. I did not find a taming mechanic in either one.

### 2. Merging duplicates into a stronger pet

| Game | Inputs | Output | Strength gain | Cost |
|---|---|---|---|---|
| Pet Simulator 99, Golden Machine | 10 identical normal pets | 1 Golden | about +13% [4] | free [4] |
| Pet Simulator 99, Rainbow Machine | 10 identical Golden pets | 1 Rainbow | about +27% [5] | free [5] |
| Adopt Me, Neon Cave | 4 identical Full Grown pets | 1 Neon (glows) | cosmetic glow; keeps Fly/Ride [9] | aging time |
| Adopt Me, Neon Cave | 4 identical Luminous Neons | 1 Mega Neon (cycles colors) | cosmetic [9][10] | aging time |
| Catch and Tame, breeding | 2 *different* pets from a recipe | an egg with a chance of a new pet | new species | odds from 9% down to 0.3% [15] |

- Pet Simulator 99 also has a separate Fuse Machine that combines pets [8].
- Pure "merge pets" tycoons do exist. Pet Merge Tycoon has 589,077 visits [27][29] and Pet Merge Simulator has 140,429 [28][29]. Both are tiny next to Catch and Tame's 462M [29]. *[inferring, not cited]* The hits wrap pets in a bigger loop (catching, a pen, breeding). Merging alone does not carry a game.
- Visual payoff matters as much as the number. Adopt Me's merge result is a glow [9]. Grow a Garden's "Huge" pets (a 0.1% hatch chance) get a 4x weight modifier [1]. Bigger pets read instantly as better.

### 3. Pets walking around a base

- Grow a Garden pets roam the garden. The Cat's trait has it nap at a random spot on the farm, with a size aura around it [1]. The game later added a Pet Menu so players can feed and pick up pets without chasing them around [1]. This tells us wandering pets are fun to watch but annoying to click.
- Catch and Tame sells a "Pet Post" for $10K. It ties an animal in place [14]. That means pets roam by default, and some players want them to stay still.
- Grow a Garden pets all use classic studs. The community voted for the studded look before pets launched [1]. This fits Fruit Empire's retro studded style.

**Performance and how to build it (Roblox)**
- Draw pets on the client. The server only says which pets each player owns and when they are equipped. Clients do all the visuals [30][32].
- One developer measured a client loop moving pets in Pet Simulator 99 style. With 100 pets they got 160–170 FPS while moving on PC. With 200 pets (2 players) they got about 100 FPS. They worried about phones [31]. That is one person's test, not a benchmark.
- Several replies suggest AlignPosition (a physics constraint that pulls a part toward a target) instead of a Lua loop. The physics engine then does the work [31][33][36]. Another reply says lerping beats creating tweens every frame [32].
- Roblox docs say a Humanoid adds a large computation cost. They recommend AnimationController for NPCs that only need to animate, playing NPC animations on the client, and only spawning NPCs when players are nearby [34][38].
- PathfindingService computes waypoints. Its defaults are a 2-stud agent radius and a 4-stud waypoint gap, and paths must be recomputed when blocked [35]. A flat plot does not need it.
- `Model:ScaleTo()` exists for making a merged pet visibly bigger [37].

### 4. Pet income

- **Catch and Tame:** each pet in the pen earns money over time, and the amount depends on rarity [11][17]. The Chicken (Common) has a base income of 1 on the wiki [16]. The game also earns while you are offline [11].
- **Steal a Brainrot** (units, not pets, but the same idea of "things on your base earn $/s") [25]:

| Rarity | $/s range |
|---|---|
| Common | 1 – 14 |
| Rare | 15 – 75 |
| Epic | 75 – 325 |
| Legendary | 200 – 1.9K |
| Mythic | 2K – 18.7K |
| Brainrot God | 19K – 323K |
| Secret | from 315K |

- Steal a Brainrot rebirths take your required units and most of your money. You get a bigger multiplier, and from rebirth 2 on, **+1 base slot** per rebirth. Each rebirth needs a cash amount plus specific units [26]. This is almost exactly Fruit Empire's planned rebirth (+1 storage, needs a full index plus shards plus money).
- **Grow a Garden:** pets do not pay cash. They boost growth, size and mutation chance, and their power scales with **weight**. Weight goes up with age: W = BaseWeight × (1 + 0.1 × Age) [1].
- **Bee Swarm:** bees collect pollen and turn it into honey at the hive [18].

### 5. Rarity tiers and how rare pets are gated

| Game | Tiers | Gate |
|---|---|---|
| Grow a Garden | Common, Uncommon, Rare, Legendary, Mythical, Divine, Prismatic [1] | which egg you buy; 0.1% Huge roll [1] |
| Adopt Me | Common, Uncommon, Rare, Ultra-Rare, Legendary [10] | tasks to Full Grown: 25 / 36 / 54 / 107 / 183 (cut from 56 / 80 / 120 / 160 / 275 on 12 Sep 2025) [10] |
| Catch and Tame | Common → Rare → Epic → Legendary → Mythical → Godly → Divine [12] | lasso strength vs animal strength; breeding-only pets [12][15] |
| 99 Nights | none named | flute tier + number of stages [2][3] |
| Bee Swarm | Common, Rare, Epic, Legendary, Mythic, Event [18] | egg tier [18] |
| Steal a Brainrot | 7 main tiers [25] | spawn odds (Common spawns 55% of the time) + price [25] |

### 6. Equip limits

| Game | Start | Max | How it grows |
|---|---|---|---|
| Grow a Garden | 3 equipped | 8 | trade in pets aged 20/30/45/60/75, or Robux (249 to 1,699 per slot) [1] |
| Grow a Garden inventory | 60 | 300 | game passes, crafting, Ascension shop [1] |
| 99 Nights | 2 tamed | 3 | carry a Strong Flute [2] |
| Catch and Tame | 8 in pen | not confirmed | fence and base upgrades [17] |
| Bee Swarm | 25 hive slots | 50 | 3M honey for the first extra slot, about 2.17T for the last [20] |
| Pet Simulator 99 | 4 (inferred: the first machine upgrade takes you to 5) | 84 by machine, 99 with a gamepass | diamonds + rank, then a 375 Robux pass [6][7] |

Grow a Garden also saves up to 6 pet loadouts, with a 5-second swap cooldown [1].

## What Fruit Empire should take

These are design proposals built on the research above. **All numbers here are starting values to test in the economy simulation (03 Economy). None are final.** Stealing stays removed. Nothing in this design lets a player take, hurt or block another player's pets.

### What Fruit Empire already has (from the local code)
- 64 fruits in 9 tiers. Common 5, Uncommon 8, Rare 11, Epic 8, Legendary 7, Mythic 8, Astral 5, Eternal 8, Transcendent 4. Apple is the top Common [42].
- Fruit income starts at 1/s for Blueberry. Each rank up multiplies it by 2.10 at the bottom, ramping to 2.60 at the top. Soul Fruit earns about 2.37e23/s [41]. The top fruit of each tier, worked out from that code: Apple (Common) 20.2/s, Blue Lollipop (Uncommon) 9.83K/s, Pear (Rare) 71.2M/s, Watermelon (Epic) 59.4B/s, Tropical Melon (Legendary) 2.55e13/s, Dragon Pepper (Mythic) 3.16e16/s, Paradise Petal (Astral) 3.03e18/s, Solar Bloom (Eternal) 5.26e21/s, Soul Fruit (Transcendent) 2.37e23/s [41][42]. Re-checked against the 2026-10-04 export: same constants, same numbers.
- Income ticks every 5 seconds. Storage holds 5 fruits [43]. There are 12 plots, and a starting plot is about 29.5 × 47.1 studs [43].
- The animal models will come from the "Stud-Themed Animal Pack [LITE]" by the verified group DropKit, uploaded 28 Jun 2026. It is 5 rigged animals with no scripts, no animations and no MeshParts, about 2,796 triangles in total [39]. Zero MeshParts suggests the animals are built from plain parts, which suits the studded look (inference). The listing does not name the 5 species.

### The loop in one line
Go to the forest. Find an animal whose request bubble shows a fruit you can make. Pay that fruit to tame it. The pet walks home to your plot and earns $/s. Tame a second identical one, merge the two, and you get a bigger pet that earns more than both did.

### 1. Wild animals (the forest ring)
- **Each player sees their own wild animals.** The server keeps a list per player (id, species, tier, request). The client draws them. Two players never fight over one animal. *[inferring, not cited]* Shared animals would bring back the "someone took mine" feeling that stealing caused.
- About 8 animals per player at once. When one is tamed, a new one appears somewhere else in the forest after 30–60 s.
- **The tier of each animal is rolled per player:**
  - It is never above the tier your rebirths unlock (0 rebirths = Common only).
  - Most rolls land at your run-best tier or lower.
  - About 1 in 8 rolls is one tier above your run-best but still unlocked. That animal shows a **locked** bubble ("Needs a Rare fruit"). It is a goal, like 99 Nights' stronger animals that need a better flute [3].
- **The look:** 5 species from the LITE pack × 9 tier colors. Each tier color matches the bat color for that rarity, as the brain dump asks. Astral, Eternal and Transcendent animals glow, the same rule the ore brief uses ("higher tier ores should glow"). Adopt Me shows that a glow alone reads as "better" [9].

### 2. The request rule (what fruit an animal wants)
The brain dump says rarer animals should "match the rarity of the max fruit that you own". The rule below does that and never asks for your single best fruit:

| Animal's tier compared with your run-best | It asks for | Why |
|---|---|---|
| Below your run-best tier | the top fruit of its own tier (a Common animal wants an Apple) | cheap; fills the Index; low payout |
| Same as your run-best tier | a fruit 1–2 ranks **below** your run-best fruit | payout matches your stage; you keep your best fruit |
| Above your run-best tier (but unlocked) | locked; shows the tier's lowest fruit as the goal | a reason to keep merging |

- **Stages, like 99 Nights [2][3]:** Common to Rare animals want 1 fruit. Epic to Mythic want 2 (two separate requests, one after the other). Astral to Transcendent want 3.
- **The request is fixed per animal per player.** It does not reroll when you walk away, so you can go home and make the fruit.
- **The bubble** is a BillboardGui showing the fruit's own Index icon, the count ("1/2") and a tier-colored frame. Use Fredoka One with a black stroke, as the UI kit requires. No emoji.

### 3. Taming (the action)
- **Pay from storage.** Walk up and hold the prompt: "Feed Apple". The fruit comes out of your StoredFruits. Grow a Garden also feeds from what you hold [1]. Carrying fruit across the sea while swimming would be clunky. Carrying the fruit to the animal can stay as a second way, because it matches the merge verb.
- **No minigame at launch.** 99 Nights and Catch and Tame use one [2][17], but the brain dump only asks for feeding. A minigame can come later if taming feels too instant.
- On tame: a heart burst plus a short happy animation. The pet then walks to your plot, or teleports there if the plot is across the sea. A toast in the existing style confirms it.

### 4. Merging pets
- **Identical means the same species, the same tier and the same size.** That is the same idea as matching fruit.
- **4 sizes:** Baby → Big → Huge → Giant. That is 3 merges, so 8 Babies make 1 Giant. Each size up scales the model by about 1.3x with `Model:ScaleTo()` [37].
- **Pay more than 2x per merge.** The fruit economy code explains why: at 2x or less, the best play is to never merge [41]. Proposed: merged income = 1.15 × (income A + income B). Two equal pets give 2.3x each.
- **How to merge:** drop one pet onto another of the same kind on your plot, or press a "Merge" button in the pet panel when two identical pets exist. A "Merge All" option like the fruit HUD button is fine.
- Avoid PS99's 10-to-1 ratio with only +13% [4]. Fruit Empire has few equip slots, so every merge has to feel big.

### 5. Equipping and wandering
- **Equip slots: 2 at 0 rebirths** (99 Nights starts at 2 [2], Grow a Garden at 3 [1]). +1 slot at rebirths 1, 3 and 6, plus one slot bought with shards. Cap at 6. Unequipped pets wait in a pet list (start at 20 spaces) and earn nothing.
- **How a pet moves:**
  1. Pick a random point inside your plot, kept away from the edge.
  2. Reject any point within about 3 studs of a fruit. If the plot is full, walk the plot's border loop instead.
  3. Walk there at about 6–8 studs/s with a small bob.
  4. On arrival, idle for 3–8 s: sit, sniff, sleep or eat. Grow a Garden's napping cat is the model [1].
  5. If the owner comes within about 12 studs, there is a chance the pet walks over and looks at them.
- **Build it client-side** [30][32]:
  - The server stores the equipped list per plot (pet id, species, tier, size, a random seed).
  - Each client spawns the models for nearby plots and moves them with CFrame lerping or AlignPosition [31][36].
  - No Humanoid. Use AnimationController + Animator [34][38].
  - Hide pets on plots far away [34].
  - No pathfinding on a flat plot [35].
- Pets are **CanCollide off and CanQuery off**. That way they never block carrying fruit, clicking fruit, merging fruit or prompts. Grow a Garden added a Pet Menu so players would not have to chase their pets [1].
- **The budget is tiny.** 12 plots × 6 pets = at most 72 pets, plus about 8 wild animals per client. That is well under the 100–200 pets in the DevForum test [31].
- **Animations needed** (the pack has none [39]): walk, idle, sit, sleep, eat, happy and merge-pop, for each of the 5 rigs.

### 6. Earnings that scale with the player's stage
- **Flat $/s, set at tame time:** pet income = 1.5 × the summed income of the fruits it ate. It shows on the pet card. It also pops above the pet on each 5-second income tick, using the existing EarnPopups [43].
- **The cost sets the payout.** A pet always costs a fruit from your current stage, so it always pays at your current stage. Old pets fade in value the same way old fruit do. That is the brain dump's rule: "everything needs to correspond with the stage of the game a player is in".
- **Worked examples** (fruit incomes from [41][42]; pet numbers use the proposed 1.5x and 1.15x):

| Pet | Fed | Baby | Big | Huge | Giant | Compare |
|---|---|---|---|---|---|---|
| Common | Apple (20.2/s) | 30.3/s | 69.7/s | 160/s | 369/s | Blue Lollipop 9.83K/s |
| Rare | Green Apple (31.2M/s) | 46.8M/s | 108M/s | 248M/s | 569M/s | Pear 71.2M/s |

  A Common Giant is a solid early boost that is tiny by mid-Uncommon. A Rare Giant takes 8 Green Apples and pays about 8x one Pear. That is strong but bounded by equip slots.
- **The nap rule makes rebirth work.** A pet earns only while your run-best fruit is at or above the pet's tier. Otherwise it sleeps on your plot with a "Wakes at EPIC" tag.
  - Pets are kept through rebirth, so the collection is not lost.
  - The economy still resets, like Steal a Brainrot taking units and money on rebirth [26].
  - Climbing back up wakes old pets one by one. That gives each new run its own small goal.
  - The rule uses **run-best** and not "current best owned", so feeding away a fruit can never make a pet fall asleep.
- **Alternative to test in the simulation:** pets pay a percentage of plot income. That scales with stage on its own, but it reads less clearly than "$X/s" and keeps low-tier pets useful forever.
- **Conflict to settle:** [[Rebirth Economies in Merge and Incremental Games]] (proposal 7) suggests a different rule: pet income = a factor × the best fruit on the plot this run, with pets capped at 25–50% of fruit income. This note's rule fixes the income when the pet is tamed and uses naps. Pick one before building.

### 7. Index and rebirth
- Add a **Pets** category to the Index: 5 species × 9 tiers = 45 entries. Only show the tiers your rebirths have unlocked. The index already pays money worth 60 seconds of a fruit's income (`IndexMoneySeconds`) [43]. Reuse that for each first tame.
- Decide whether pet entries count toward the "full index" rebirth requirement. See Open questions.

## What to avoid
- **A weak payoff for a long trip.** 99 Nights' own wiki calls its taming "extremely time consuming for little reward" [3]. Every tame must pay at the player's current stage.
- **Pets that leave the plot or get stuck.** 99 Nights pets get stuck on obstacles [3]. Keep pets inside the plot rectangle, and use no physics collisions.
- **Long grinds to grow or merge.** Adopt Me cut the tasks needed to grow pets by more than half for Common (56 to 25) and by a third for Legendary (275 to 183) in Sep 2025 [10]. Keep merges to 2-into-1 with only 3 steps.
- **Chores and decay.** Dragon Adventures' bond drops every 30 s while equipped [21], and Grow a Garden pets stop aging when hunger hits zero [1]. Do not add hunger that turns off income. Do not punish players who go offline.
- **Trading or gifting at launch.** Grow a Garden turned off pet gifting twice, in July 2025, because of a duplication exploit [1]. Add trading only after the pet save format is locked down.
- **Tiny merge bonuses or long odds as the main path.** PS99's 10-to-1 for about +13% [4], Catch and Tame's 0.3% breeding recipes [15] and Bee Swarm's 1-in-24,000 Gifted rolls [18] are fine as side goals. They are wrong for Fruit Empire's core 2-into-1 loop.
- **Humanoids on pets, or moving pets on the server.** Both cost performance [30][32][34].
- **Pets that can be hurt.** Artillery drains fruit health. Pets should be immune to TNT, nukes and boats breaking, and other players' pets should not be interactive. This keeps griefing out, for the same reason stealing was removed.
- **Pets in the way of fruit.** Never let a pet block the carry raycast, a fruit click or a ProximityPrompt.
- **A UI that breaks the kit.** Bubbles, the pet panel and the merge screen must use the GardenUI images and Fredoka One. No emoji, no code-drawn gold boxes (see [[UI Kit Inventory]]).

## Open questions
1. Which 5 animals are in the LITE pack? The store listing only says it has 5 animals [39]. Open it in Studio to name the species before writing the Index.
2. Should a tame be paid from storage, by carrying the fruit, or both? Storage holds only 5 fruits at 0 rebirths [43].
3. Where do equip slots come from: rebirths, shards, or both? What is the cap?
4. Do pet Index entries count toward "fill out your index" for rebirth? If yes, rebirth gets much harder.
5. Nap rule, or reset pets on rebirth? This note recommends the nap rule. The simulation should confirm it does not make later runs too fast.
6. The 1.5x tame payout and 1.15x merge payout are guesses. Tune them in 03 Economy against plot income at each stage.
7. Should rare "Huge" wild animals exist (like Grow a Garden's 0.1% Huge, with 4x weight [1])? Suggest leaving them for a later update.
8. Should the game add a short taming minigame later (99 Nights, Catch and Tame [2][17])? Only if instant taming feels flat in playtests.
9. Side finding: the Roblox games API still lists the game as "[STEAL!] Fruit Empire" with maxPlayers 50 (re-checked 2026-10-04) [40]. The public description also still says "ROB everyone" and "Steal fruits from enemy bases". Both no longer match the game now that stealing is gone. The map has only 12 plots [43], so up to 38 players in a full server would have no plot to put pets on.

## Sources
1. Grow a Garden Wiki, "Pets": https://growagarden.fandom.com/wiki/Pets
2. 99 Nights in the Forest Wiki, "Taming": https://99-nights-in-the-forest.fandom.com/wiki/Taming
3. 99 Nights in the Forest Wiki, "Taming Flute": https://99-nights-in-the-forest.fandom.com/wiki/Taming_Flute
4. Pet Simulator Wiki, "Golden Machine (Pet Simulator 99)": https://pet-simulator.fandom.com/wiki/Golden_Machine_(Pet_Simulator_99)
5. Pet Simulator Wiki, "Rainbow Machine (Pet Simulator 99)": https://pet-simulator.fandom.com/wiki/Rainbow_Machine_(Pet_Simulator_99)
6. Pet Simulator Wiki, "Pet Equip Machine (Pet Simulator 99)": https://pet-simulator.fandom.com/wiki/Pet_Equip_Machine_(Pet_Simulator_99)
7. Pet Simulator Wiki, "+15 Pets Gamepass (Pet Simulator 99)": https://pet-simulator.fandom.com/wiki/%2B15_Pets_Gamepass_(Pet_Simulator_99)
8. Pet Simulator Wiki, "List of Machines (Pet Simulator 99)": https://pet-simulator.fandom.com/wiki/List_of_Machines_(Pet_Simulator_99)
9. Adopt Me! Wiki, "Neon Pets": https://adoptme.fandom.com/wiki/Neon_Pets
10. Adopt Me! Wiki, "Gameplay" (objectives, rarities, task counts, Neon/Mega Neon): https://adoptme.fandom.com/wiki/Gameplay
11. Roblox game page, "Catch And Tame!": https://www.roblox.com/games/96645548064314/Catch-And-Tame
12. Catch and Tame Wiki, "Lassos": https://catch-and-tame-rblx.fandom.com/wiki/Lassos
13. Catch and Tame Wiki, "Food": https://catch-and-tame-rblx.fandom.com/wiki/Food
14. Catch and Tame Wiki, "Buildings": https://catch-and-tame-rblx.fandom.com/wiki/Buildings
15. Catch and Tame Wiki, "Breeding Hints": https://catch-and-tame-rblx.fandom.com/wiki/Breeding_Hints
16. Catch and Tame Wiki, "Chicken": https://catch-and-tame-rblx.fandom.com/wiki/Chicken
17. Sportskeeda, "Catch and Tame: A beginner's guide" (8 animals at start, fence upgrades): https://www.sportskeeda.com/roblox-news/catch-tame-a-beginner-s-guide
18. Bee Swarm Simulator Wiki, "Bees": https://bee-swarm-simulator.fandom.com/wiki/Bees
19. Bee Swarm Simulator Wiki, "Treats": https://bee-swarm-simulator.fandom.com/wiki/Treats
20. Bee Swarm Simulator Wiki, "Hive Slot": https://bee-swarm-simulator.fandom.com/wiki/Hive_Slot
21. Dragon Adventures Wiki, "Bonding": https://dragon-adventures.fandom.com/wiki/Bonding
22. Dragon Adventures Wiki discussion post on Wild/Tame potions (player post, low confidence): https://dragon-adventures.fandom.com/f/p/4400000000000708601
23. Creatures of Sonaria Wiki, "Creatures of Sonaria": https://creatures-of-sonaria-official.fandom.com/wiki/Creatures_of_Sonaria
24. Feather Family Wiki, "Feather Family": https://feather-family.fandom.com/wiki/Feather_Family
25. Steal a Brainrot Wiki, "Rarities": https://stealabrainrot.fandom.com/wiki/Rarities
26. Steal a Brainrot Wiki, "Rebirth": https://stealabrainrot.fandom.com/wiki/Rebirth
27. Roblox game page, "Pet Merge Tycoon": https://www.roblox.com/games/12797469624/Pet-Merge-Tycoon
28. Roblox game page, "Pet Merge Simulator!": https://www.roblox.com/games/14795295750/Pet-Merge-Simulator
29. Roblox Games API, visit counts checked 3 Oct 2026 (Catch And Tame, Pet Merge Tycoon, Pet Merge Simulator): https://games.roblox.com/v1/games?universeIds=9091133975,4481371079,5099090262
30. Roblox DevForum, "Render pets on the client or server?": https://devforum.roblox.com/t/render-pets-on-the-client-or-server/442739
31. Roblox DevForum, "How to make a very optimized Pet Follower?": https://devforum.roblox.com/t/how-to-make-a-very-optimized-pet-follower/3235258
32. Roblox DevForum, "Should my pets following system be client or server?": https://devforum.roblox.com/t/should-my-pets-following-system-be-client-or-server/2494410
33. Roblox DevForum, "Optimize Pet Follow?": https://devforum.roblox.com/t/optimize-pet-follow/3166935
34. Roblox Creator Docs, "Improve performance": https://create.roblox.com/docs/performance-optimization/improve
35. Roblox Creator Docs, "Character pathfinding": https://create.roblox.com/docs/characters/pathfinding
36. Roblox Creator Docs, AlignPosition: https://create.roblox.com/docs/reference/engine/classes/AlignPosition
37. Roblox Creator Docs, Model (ScaleTo, GetScale): https://create.roblox.com/docs/reference/engine/classes/Model
38. Roblox Creator Docs, AnimationController: https://create.roblox.com/docs/reference/engine/classes/AnimationController
39. Creator Store, "Stud-Themed Animal Pack [LITE]" (and toolbox details API): https://create.roblox.com/store/asset/97050466582003/StudThemed-Animal-Pack-LITE · https://apis.roblox.com/toolbox-service/v1/items/details?assetIds=97050466582003
40. Roblox Games API, Fruit Empire universe 10764283455: https://games.roblox.com/v1/games?universeIds=10764283455
41. Local code: `~/fruit-empire/src-latest/ReplicatedStorage.FruitEconomy.lua` (Base 1/s, StepLow 2.10, StepHigh 2.60, the "more than 2x" rule)
42. Local code: `~/fruit-empire/src-latest/ReplicatedStorage.Fruits.FruitIndex.lua` (64 fruits, 9 tiers)
43. Local code: `~/fruit-empire/src-latest/ReplicatedStorage.GardenConfig.lua` (IncomeTick 5, StoredCap 5, IndexMoneySeconds 60, PlotSizes, 12 plots). Same values in `~/fruit-empire/src-export-2026-10-04/`.

## Review (critic pass, 2026-10-04)

- **Verified:** every local-code number (tier sizes 5/8/11/8/7/8/5/8/4, income constants, tier-edge incomes, IncomeTick, StoredCap, IndexMoneySeconds) against the 2026-10-04 export. Added the three missing tier edges (Legendary, Astral, Eternal).
- **Added:** the animal pack's creator, upload date and part makeup from the toolbox API [39]; the public description still advertising stealing [40]; a pointer to the conflicting pet-income rule in the rebirth note.
- **Unsourced but labelled:** the per-player wild animals, the 1.5x tame and 1.15x merge payouts, and the slot counts are proposals, and the note says so.
- **Still not confirmed:** the 5 species in the pack, Catch and Tame's pen cap, and the Pet Simulator 99 starting equip count.
