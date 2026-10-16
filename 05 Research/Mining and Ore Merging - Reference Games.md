---
tags: [research]
topic: Mining and ore-merging reference games (ore ladders, block health, mine expansion, glowing ores, cave lighting, ore income balance)
date: 2026-10-04
---

# Mining and Ore Merging: Reference Games

Terms used in this note:
- **Ore ladder**: the ordered list of ores from cheapest to best.
- **Block health**: how much mining a block takes before it breaks. Games call it "stone amount", "strength" or "mining time".
- **Mining power**: how much damage one pickaxe swing (or one mining cycle) does to a block.
- **Layer**: a depth band of the mine with its own stone color, block health and ores.
- **Merge chain**: the list of items you get by merging the same item again and again.
- **Carry-merge**: you pick up one item and walk it onto an identical item to merge them. This is how Fruit Empire merges fruit.
- **Node**: a single ore rock that sits in a fixed spot and regrows after you mine it (instead of a diggable wall of blocks).
- **Mine reset / collapse**: the whole mine regenerates and mined blocks come back.
- **Rebirth**: you give up cash and gear for a permanent bonus.
- **Neon**: a Roblox part material that looks like it glows.
- **PointLight**: a Roblox light that shines in every direction from one point.
- **Fan wiki**: a site made by players, not by the game's developer. Some newer ones are SEO sites with unchecked numbers. I flag those.

Games covered: Mining Simulator (833M visits) and Mining Simulator 2 (211M) by Rumble Studios [59][9]; Azure Mines (52M) and Miner's Haven (200M) by berezaa [25][60]; Ultimate Mining Tycoon (89M) [43]; Grab Ores! (11M) [28]; Mine a Mountain (146M) [37]; Sell Ores (44M) [40]; Merge & Dig Simulator (11M) [34]; Mining Merge Tycoon (0.4M) [32]; mobile Merge Dragons [45]. Merge a Mineral! (449K visits) merges ores but is built around stealing, which Fruit Empire removed [36]. I did not find a Roblox game called "Merge Mine", "Mine Merge" or "Ore Tycoon" with real traffic. The closest were "Merge & Mine" (80K visits, no description) and "Ore Tycoon 2" (64M visits, a factory tycoon, no merging). The merge-mining games I found are all small.

## Summary

- **No big Roblox game merges ores the way Fruit Empire merges fruit.** The closest is Mining Merge Tycoon. You "pick up one ore and walk up to another SAME ore and combine it", across 25 minerals. It is small (about 403K visits) [32]. Grab Ores! (11M visits) merges two identical ores into a better version of the *same* ore (Rocky, then Shiny, Polished, Refined Rocky) [29]. The big merge-and-dig hits merge **pickaxes**, not ores [34][35].
- **Ore ladders follow the same rough order in every game.** Common metals come first (coal, copper/tin, iron, silver). Gems come in the middle (gold, emerald, ruby, sapphire). Made-up glowing ores sit at the top (Glowstone, Uranium, Unobtainium, Rainbowite) [1][44][31].
- **Block health climbs steeply with depth.** In Mining Simulator, stone goes from 1 (grass) to 2, 5, 20, 50, 125, 250, ... 30,000 [2]. Ultimate Mining Tycoon stone strength goes 8, 16, 40, 70, 110, 175, 210 [43]. Pickaxe power climbs too: Mining Simulator's free pickaxe has 1 power and a 5.5M-coin staff has 300 [3].
- **Explosives are a standard mining tool tier.** Mining Simulator sells TNT (power 100, blast size 10), C4 (150, 12) and a Robux Nuke (500, 16) [3]. "+1 TNT Mining" (13.7M visits) is built around buying stronger TNT [42]. This lines up well with Fruit Empire's TNT tiers.
- **Higher ores are made to look special in four ways.** They use neon parts, real textures, animated color cycling, and stone that changes color by layer. Glowstone is "protruding, neon-yellow cones" [6]. Rainbowite has a rainbow texture [7]. Valhalum "flashes between rainbow versions of various other ores" [20]. Azure Mines and Ultimate Mining Tycoon both change the stone color every layer [13][43].
- **Ore income is held back in three ways.** Rebirth multipliers [4][9]. Ore spent on upgrades instead of only sold, as in Azure Mines where base upgrades eat thousands of ores [22]. Limited display slots, as in Grab Ores! where ores on display in your vault earn $/sec and you buy more slots [29].
- **Mines reset.** Azure Mines collapses after about 100,000 to 115,000 blocks (its two wiki pages disagree) [11][12]. Ultimate Mining Tycoon uses a 100-minute timer [43]. Mine a Mountain resets every hour [38]. Sell Ores skips digging. You place ores in 7 compartments per floor, and a drone mines them as they regrow [41].
- **Cave look:** set dark Ambient and swap the Atmosphere when the player goes underground [52][51]. Put a PointLight on an Attachment inside neon gems [55][61]. Change the stone color per layer. Roblox's LightingStyle "Soft" gives "a flat, retro-Roblox look" [53]. Shadows and particles cost performance, so use them sparingly [58].

## How it works

### 1. Ore ladders (names and order)

**Mining Simulator, Earth (value in coins, layer where it starts)** [1]:
Fossils 15 (L2), Dinosaur Bones 30 (L2), Coal 45 (L3), Copper 80 (L3), Silver 120 (L4), Quartz 300 (L4), Skeleton Stone 450 (L4), Gold 220 (L5), Pezzottaite 700 (L5), Obsidian 5 (L6), Emerald 550 (L6), Ruby 700 (L6), Lavastone 1,000 (L6), Sapphire 1,200 (L6), Glowstone 1,250 (L6), Uranium 3,500 (L6), Platinum 4,500 (L6), Lapis 5,000 (L6), Krixanium 5,000 (L6), Red Rock 6,500 (L6), Dravite 20,000 (L6), Diamond 2,500 (L7), Unobtainium 5,000 (L7), Mythic Stone 75,000 (L7), Legendary Stone 90,000 (L7), Breadstone 6,666 (L8), Shadow Stone 160,000 (L9), Illuminite 250,000 (L9), Rainbowite 500,000 (L9).
- Note: value is **not** strictly in ladder order. Gold (220) is worth less than Quartz (300). Diamond (2,500) is worth less than Uranium (3,500). Mining Simulator uses depth for rarity and value only roughly.

**Ultimate Mining Tycoon (base value, required strength)** [44]:
Tin $10 (6), Iron $20 (6), Lead $30 (7), Cobalt $50 (12), Aluminium $65 (18), Silver $150 (18), Uranium $180 (18), Vanadium $240 (22), Tungsten $300 (45), Gold $350 (22), Titanium $400 (24), Molybdenum $600 (75), Plutonium $1,000 (99), Palladium $1,200 (120), Mithril $2,000 (200), Thorium $3,200 (270), Iridium $3,700 (180), Adamantium $4,500 (300), Rhodium $15,000, Unobtainium (wiki search snippet says $30,000; not checked on the page itself).
- This ladder is close to steady: each step is roughly 1.2x to 3x the one before.

**Azure Mines (start of the ladder)** [16][17][18]:
Stone (mining time 1, sells for 0.5 to 1), Coal (mining time 2, sells for 4), Iron (mining time 6). Azure, the ore the game is named after, has mining time 250 and sells for 8,400. Azure Mines uses ores mainly as crafting and upgrade material, not only for cash. The full game needs 1,130 Coal and 4,710 Iron [16][17].

**Grab Ores! (income per second while on display)**. This comes from an **unofficial fan site**, so treat the numbers as unconfirmed [31]:
Stone $5, Iron $45, Copper $120, Marble $180, Silver $450, Crystal $750, Platinum $1,200, Amber $1,450, Jade $1,800, Gold $3,500, Sapphire $5,200, Obsidian $7,200, Emerald $8,000, Garnet $9,500, Ruby $25,000, Diamond $50,000 (said to be "1 in 7,594,737").
- One number does match a YouTube video title, "Grabbing a 1 in 7,594,737 Ore". The rest are not confirmed. In the gameplay video, a normal Sapphire earned "108 per second" and a rainbow Sapphire "379 per second" [29].

**Ore Island** (a tiny game, 13K visits) says in its own description: "merge 3 into a rarer crystal ... Aquamarine → Emerald → Moonstone… 10 crystals to discover in the Codex" [33].

### 2. How mining works (tools, block health, respawn)

**Mining Simulator** [3][2]:
- Mining Power is how much a tool removes per cycle. "A tool with 1,000 Mining Power will take 1,000 Stone from a 20,000 block of Stone." Speed Boost is how fast cycles go. Efficiency = Power × Speed.
- Each stone layer has a block "Stone Amount": Layer 0 = 1, L1 = 2, L2 = 5, L3 = 20, L4 = 50, L5 = 125, L6 = 250, L7 = 350, L8 = 500, L9 = 1,000, L10 = 2,000, L11 = 5,000, L12 = 10,000, L13 = 20,000, L14 = 30,000.
- Shop pickaxes (price, power, speed): Wooden (free, 1, x1), Stone (25, 2, x2), Metal (75, 3, x2.5), Mattock (250, 5, x3), Shovel (600, 7, x5) ... Power Saw (120,000, 50, x15) ... Zues's Staff (5.5M, 300, x200).
- Explosives have "Blast Size" instead of speed: TNT 250,000 coins (power 100, blast 10), C4 360,000 (150, 12), Nuke 400 Robux (500, 16).
- Each ore also needs a minimum tool. For example, Glowstone needs a Power Saw or better [6].
- Backpack storage caps a trip. The free backpack holds 10 blocks. Later backpacks hold 25, 50, 145 ... up to 500,000 for 8M coins [5].

**Mining Simulator 2** [10]: Mining Power is damage per hit. Enchants add +5% speed or power, up to +50%. "Ore Duplicator" gives +1 ore per block, up to +4.

**Azure Mines** [11][12][14]:
- The public mine is 26x26 blocks. Blocks are 6 studs [15]. It has 7 layers down to 5,833m, where the player dies.
- Some ores are hazards. Boomite can explode and kill you [11]. Darkmatter can explode when mined [19].
- Caves generate when you break into them. Zombies spawn in some caves.
- The mine collapses after a set number of blocks. One page says 100,000 mined [11]. The Mine Condition page says "up to 115,000 blocks can be generated" and that it counts unmined generated blocks. It shows four warning levels: Safe 0–48%, Risky 48–73%, Dangerous 73–92.5%, CRITICAL 92.5%+. On collapse, "all players are teleported to the surface" [12].
- One ore (Titanium) gets more common as the mine nears collapse. Its spawn rate is 2^(lifetime/10) / 512,000, which tops out at 0.2% [19].

**Ultimate Mining Tycoon** [43]: The mine has 8 layers. It has "a collapse timer of 100 minutes". Players can extend it by 30 minutes for 225 Robux, with no limit.

**Mine a Mountain** [38]: You climb instead of digging. Height zones are locked behind "Warmth" upgrades. Pickaxes differ in power and "mine size". "The mountain resets every hour, restoring all mined blocks and crystal spawns." Bombs are sold in a shop.

**Grab Ores!** [28][29]: You rope down a cave and grab ores as you go. Weights make you sink faster and belts make you climb faster. Oxygen limits how long you stay down, and "health drains at zero oxygen".

**Sell Ores** [40][41]: There is no digging. You pull a lever to roll for an ore, buy it, and place it in one of "seven mining compartments" per floor. A drilling drone mines the ore into a crate that you sell. Upgrades cover ore luck, drill yield, drill speed and ore regen speed.

### 3. How ores are merged and what merging gives

- **Mining Merge Tycoon** (Roblox description): "Pick up one ore and walk up to another SAME ore and combine it. Click on ores to get minerals faster. Sell minerals or smelt them for money." It has 25 minerals [32]. This is the same carry-merge as Fruit Empire. I could not confirm its exact mineral list.
- **Grab Ores!** (from the gameplay video [29]):
  - Two identical ores merge into the next tier of the **same ore**. The video shows "a shiny Rocky. So, that's what the second tier is", then "a tier three Rocky ... a polished Rocky ... tier four. A refined Rocky."
  - Merges have a cooldown ("Just got to wait for the cool down").
  - A shiny ore will not merge with a normal one ("Can that merge? No, cuz this one's shiny").
  - "Two marbles" merged gave "30 per second".
  - Rare **rainbow** versions spawn naturally ("a rainbow sapphire ... it was changing colors").
  - A fan site says merging two identical ores "creates a shiny version of the ore with significantly higher value". It gives no multiplier [30].
- **Merge Dragons** (mobile) [45][46]:
  - 3 identical items merge into 1 of the next level. 5 identical items merge into **2**. The wiki says to "always merge in multiples of 5".
  - The stone chain shows how value grows: level 1 Stone Brick = 1, level 2 = 5, level 3 = 20, level 4 = 75, level 5 = 250, level 6 = 800. Merging 3 level-1 bricks (worth 3) gives one level-2 brick (worth 5), which the wiki lists as "+2" profit. **Merging always pays more than the inputs were worth.**
- **Merge & Dig Simulator / Merge & Dig!** merge pickaxes: "Two of the exact same pickaxe become one stronger pickaxe" [35][34]. Ores are only cash.
- **Sell Ores** has no merging. It uses **mutations**, which are paid coatings that multiply one ore's value: Rusty x1.5, Frozen x1.75, Crystal x2, Void x2.25, Alien x3.25, Galaxy x5 [41].
- **Miner's Haven** has no merging. Ore value goes up as ore rides past "upgraders". One example is the Large Ore Upgrader. It gives x1.5 under $5k, x1.2 from $5k to $10k, x1.05 from $10k to $15k and x1.01 from $15k to $50k, with a hard cap at $50k [26]. This "only helps cheap ore" cap keeps early items from carrying late game. Rebirth in Miner's Haven wipes the base and hands out stronger "Reborn" items instead of a plain multiplier [27].

### 4. How underground areas are expanded

- **Rebirth-gated zones (Mining Simulator)** [4]: Candy Land needs 1 rebirth, Dino Land 2, Atlantis 5, The Beach 10, Crystal Cavern 20, Magic Forest 45, Lava World 60.
- **Upgrade-gated height (Mine a Mountain)**: Warmth upgrades unlock higher, rarer zones [38]. Grab Ores! does the same with weights and oxygen for depth [29].
- **Pay ore to unlock fast travel (Azure Mines Layer Elevator)** [24]. Scary Caverns costs 25,000 coins plus layer ores (for example 500 Boomite, 175 Platinum). Azure Caverns costs 75,000 coins plus ores (for example 200 Diamond, 75 Azure).
- **Personal mine (Azure Mines Personal Tunnel)** [23]: costs 300 uC. It gives a 3x3 entrance "vs. the 26x26 surface of the public mine". But it "is connected with the Public Mine", so other players can dig into it. The wiki calls it "practically useless" for that reason. The Public Mine page also says players can "sneak into someone's base" by digging up through their tunnel [11].
- **Buy more floors (Sell Ores)**. Each floor adds 7 compartments and a drone [41]. "Base expansion multiplies the throughput of every drone you own" (fan wiki snippet, not confirmed).
- **Buy more display slots (Grab Ores!)**. "Get more ore slots by expanding our vault" [29]. Mine a Mountain also sells more base slots for luck crystals [38].

### 5. How higher ores are made to look special

- **Neon shapes**: Mining Simulator's Glowstone is "protruding, neon-yellow cones" [6].
- **Real textures**: Rainbowite is "the first ore to have a texture on the ore blocks", a rainbow pattern [7].
- **Animation**: Azure Mines' Valhalum "flashes between rainbow versions of various other ores", which makes it "the only ore with an animated texture". It spawns at 1 in 10,000 [20].
- **Rainbow variants** of normal ores in Grab Ores! change color while you hold them [29].
- **Light-emitting ores in dark layers**: Azure Mines says it is "hard to see any non-illuminated ore" with only a lantern below 600m. That implies some ores give off their own light [14].
- **Stone color per layer**:
  - Azure Mines goes Stone (gray, getting darker over 4 sub-layers from 1m to 600m), then Slate (black), Lapis (blue), Hellstone (red), Granite (green), Dreamstone (purple) [13].
  - Ultimate Mining Tycoon goes dark grey, light grey, dark grey, orange, green, dark yellow, brown, purple. It also switches between "flat" and "rough" stone textures [43].
- **A "magical/glowing" rarity tier**: the mobile game Drills Merge Master names its tiers Common (iron, copper), "Magical – glowing ores with special value", Rare (adamantite) and Epic [47].
- **The owner's reference image** (`Images/ore-reference.webp`, from [[Brain Dump 2026-10-04]]) shows studded gray rock clumps with small colored 1-stud gem cubes. The gems are black, cream, yellow, red, green, pink, cyan and tan. The bright gems read as glowing.

### 6. How ore income is balanced against the main loop

- **Rebirth multiplier plus price scaling (Mining Simulator)** [4]:
  - Rebirth n costs 10,000,000 × (n+1) coins.
  - Each rebirth "adds 1x" to the ore value multiplier, starting at 2x after the first.
  - Coin prices also scale: tools and backpacks cost (1 + rebirths) / 2 × base price.
  - The player keeps only items bought with Rebirth Tokens or Robux. Each rebirth gives 20 tokens.
- **Mining Simulator 2** [9]: the first rebirth costs 400,000,000 coins. The wiki says the multiplier "increases by 10x" per rebirth, and you get 150 gems.
- **Ore as upgrade material, not just cash (Azure Mines)**:
  - Coins mostly come from a base station, the Mining Operation. It pays 2 coins per 15s at level 1, 5 at level 2 and 9 at level 3 [21].
  - Ores are spent upgrading base stations. All stations together cost 26.1M coins plus thousands of each ore (300,000 Stone, 18,570 Copper, 4,704 Iron, ...) [22].
  - This keeps ore from flooding the cash economy.
- **Small fixed boosts, not big multipliers (Ultimate Mining Tycoon)** [44]: the Ore Cleaner adds +10 credits and the Polisher another +10. Smelting gives +20% value.
- **Display or sell (Mine a Mountain)** [38]: you either place a crystal on your base for a luck boost or sell it. Mine Antarctica copies this: "Place crystals in your garden for MORE digging luck!" [39].
- **Display slots as the cap (Grab Ores!)** [29]: displayed ores earn per second. Slots are limited and bought with cash. The first rebirth in the video needed "$110,000 and a stone" and gave "better climbing speed, more backpack space, and a better cash multiplier".

### 7. Building a mine and making caves look good in Roblox

**Generation and performance:**
- The standard "infinite mine" trick only makes a block a real Part when it touches air on one of its 6 faces (+X, -X, +Y, -Y, +Z, -Z). Every other block is just stored data [49][48][50].
- One developer saw memory reach "4.4K MB in under than minute" with 16x16x16 chunks of 1-stud blocks. Replies said to load chunks on the client, not the server [49].
- Azure Mines' cave generator used noise values between 0.57 and 0.86 to carve caves [14].

**Lighting:**
- **LightingStyle**: "Realistic" is the most advanced. "Soft produces a flat, retro‑Roblox look with softer lights and shadows" [53]. The old `Technology` property is deprecated. LightingStyle and PrioritizeLightingQuality replace it [54].
- **Ambient** "is the lighting hue applied to areas that are occluded from the sky, such as indoor areas" and defaults to black [54]. One builder made a crystal cave very dark with Ambient 0,0,0 [51].
- **Swap settings at the entrance.** Developers suggest a trigger box at the cave entrance that switches lighting settings. They also suggest removing or changing the Atmosphere while the player is inside, because Atmosphere fog "looks terrible" in caves [61][52]. Note that Lighting fog properties are hidden while an Atmosphere object exists [54]. Atmosphere Density sets "how much in-game objects/terrain will be obscured" [56].
- **Glowing crystals:**
  - "Try putting a pointlight on an attachment in the neon part, with shadows enabled" [62]. Roblox recommends Attachments as the parent for point lights [55].
  - One builder said crystals looked "a bit to bright". The suggested fix was glass plus a point light, or bloom [63].
  - BloomEffect "causes brighter colors to glow, similar to applying the neon Material to everything" [57].
- **PointLight Range**: the cap used to be 60 studs. On 23 Sep 2025 Roblox announced it was doubling the limit to 120 studs for PointLight, SpotLight and SurfaceLight [65]. The current API dump lists a Studio-only `Lighting.ExtendLightRangeTo120` setting (a rollout switch) [66]. Whether 120 is on by default in Fruit Empire's place is not confirmed, so check that setting in Studio. Brightness does not make a light reach farther: it "is still limited to the light's defined range" [55].
- **Torches as an ore sink (Azure Mines)** [15]:
  - Torch: 1 Coal, lights 6 studs.
  - Uranium Stick: 1 Uranium + 2 Iron, 18 studs, green light.
  - Glass Candle: 3 Dragonglass, 36 studs, purple light.
- **Readability**: one builder turned off shadows instead of raising Ambient, because "changing the ambient makes it hard to distinguish surfaces" [64].

**Performance:**
- "Maps that contain a high number and density of light objects that cast shadows ... can have performance issues" [58].
- Particles "don't batch well", and property changes to ParticleEmitters "can have a dramatic impact on performance" [58].
- Shadows switch off completely below graphics quality 4 [58].

## What Fruit Empire should take

The owner's brief ([[Brain Dump 2026-10-04]]) asks for:
- a 100k-shard base upgrade that opens an underground floor, which can be extended further down;
- ores you mine and merge, starting with coal merging to iron;
- an Index category for ores;
- higher ores that glow;
- ore earnings that match the player's stage, from Common through Transcendent.

Stealing is removed, so the underground should be fully private. Suggestions (design proposals, not facts from sources):

1. **Use ore nodes, not a diggable voxel mine.**
   - Put studded rock clumps (like the reference image) in fixed slots on each underground floor. When mined, a node regrows after a timer. This copies Sell Ores' compartments and regen [41] and Mine a Mountain's hourly reset [38], scaled down to one plot.
   - It avoids the voxel memory problem [49]. It keeps the floor looking hand-built. It never kicks the player out with a mine collapse like Azure Mines does [12].
   - If digging is wanted later, use the 6-face rule [48][49] inside a small walled room only.
2. **Reuse the fruit health-drain system for nodes.**
   - Each node has a health bar. Pickaxe swings drain it. Nodes in higher tiers get much more health, the way Mining Simulator's block health climbs 1, 5, 20, 50, 125, 250, ... [2].
   - Let the existing TNT tiers work underground too: they drain health on every node in a radius. Mining Simulator already treats TNT, C4 and Nuke as mining tools with a blast size [3], and "+1 TNT Mining" shows the idea has an audience [42]. One TNT system then serves both the artillery feature and mining.
3. **Carry-merge ores exactly like fruit.**
   - Mined ore pops out as a carryable item. Walking it onto an identical ore merges it, as in Mining Merge Tycoon [32]. Coal + Coal = Iron, as the owner asked.
   - Use the same merge rule as fruit (2 into 1) so players learn nothing new.
   - Make every merge worth more than its inputs, as Merge Dragons does [46]. Players should never feel a merge lost them money.
4. **Ladder: start with 2 ores, plan 9 rarity bands.**
   - v1 ships Coal (black gems) → Iron (cream gems), both Common, no glow.
   - Later ores should take their gem color from the existing rarity colors ([[UI Kit Inventory]]): Uncommon green, Rare blue, Epic purple, Legendary gold, Mythic red, Astral cyan, Eternal pink, Transcendent pearl. This also matches most gem colors in the owner's reference image.
   - Possible names that follow the usual real-world order seen in every game [1][44]: Coal → Iron → Copper → Emerald → Sapphire → Amethyst → Gold → Ruby → (made-up top ores for Astral, Eternal and Transcendent).
   - Each rarity band unlocks with the same rebirth that unlocks that fruit rarity. This matches how Mining Simulator gates zones by rebirth count [4] and keeps ores tied to the player's stage.
5. **Glow ramp.**
   - Common: plain gems.
   - From Rare (or Epic): Neon gem cubes plus one PointLight of the same color on an Attachment [62][55].
   - Mythic and up: add a slow sparkle ParticleEmitter.
   - Transcendent: an animated color shimmer, like Valhalum's cycling texture [20].
   - Keep shadows off on ore lights and keep particle counts low [58].
6. **Tie ore earnings to the player's stage, not to the ore alone.**
   - Fruit Empire earnings are exponential. So an ore's payout (per tick on display, or as a sell price) should be set as a fraction of what the player's current top fruit tier earns. The fraction is to be decided.
   - That way a Coal node never matters to a Mythic player, and a lucky high ore cannot jump an early player ahead.
   - Ultimate Mining Tycoon's small flat boosts [44] and Miner's Haven's capped upgraders [26] are two existing ways to stop side income from running away.
7. **Give ore a use besides cash.**
   - Fill the new Ore Index category, which should grant shards like the fruit index.
   - Possible sinks:
     - Underground floor extensions cost cash plus ore, like Azure Mines' Layer Elevator [24].
     - Craftable lanterns or torches from Coal, like Azure Mines [15].
     - Optional pet level-ups fed with ore. Mining Simulator levels pets with "specific ores" [8].
   - Sinks keep ore from flooding the money economy, as Azure Mines does with base stations [22].
8. **Make each new floor look deeper.**
   - Each floor bought shifts the stone color and gets a bit darker, like Azure Mines and Ultimate Mining Tycoon [13][43]. For example: warm gray, then dark slate, then blue, then purple crystal.
   - Use LightingStyle Soft for the retro look [53]. Swap to a dark Ambient and remove or thin the Atmosphere when the player goes down the ladder [52][61].
   - Use studded walls and pillars (the Studed-Mountain asset from the brief), hanging lanterns with warm PointLights, and a few large neon crystal clusters as landmarks.

9. **Check the 100k-shard price against how many shards exist.** (Worked out from the local code, not from a reference game.)
   - Today shards come mainly from the Index. Each fruit pays 5 to 2,500 shards once per run, 26,528 for all 64 fruit. A rebirth pays the claimed amount a second time [67][68]. The Crown pays extra shards to the few players who fight (100 to 3,000), and a "Merge Rush" decree pays 400 to each merging player, at most once per 6 minutes [69].
   - With the planned rarity gating (0 rebirths = Common only), the Index pays 31 shards for Common, 125 through Uncommon, 468 through Rare, 1,078 through Epic, and 26,528 through Transcendent.
   - If each run keeps the current rule (Index wiped and re-paid at rebirth), a player has about 1,248 Index shards after rebirth 3, 3,404 after rebirth 4, 69,528 after rebirth 8, and 96,056 once the Transcendent page is full. That is before spending any shards on rebirth requirements, boats or TNT.
   - So a 100k-shard underground is an end-game purchase unless shards get a new source. Decide this before tuning ore income "to the stage" where the underground opens.

## What to avoid

- **Connecting one player's mine to anyone else's.** Azure Mines' Personal Tunnel joins the public mine, so players can dig into other bases [23][11]. That would quietly bring back stealing and griefing.
- **Thousands of Part blocks.** Full voxel mines blow up memory unless only air-touching blocks exist [49].
- **Mine collapses that throw players out**, and Robux timers to delay a reset. Ultimate Mining Tycoon charges 225 Robux per 30 minutes [43]. This feels like paying to avoid a penalty.
- **Value out of ladder order.** Mining Simulator has Gold below Quartz and Diamond below Uranium [1]. In a merge game, the merged ore must always be worth more, or the merge feels like a loss.
- **Two kinds of the same tier that can't merge together.** Grab Ores! blocks shiny + normal merges [29]. If Fruit Empire adds rainbow variants, decide up front how they merge.
- **Hazard ores that kill or delete loot** (Boomite, Darkmatter) [11][19], and oxygen death [29]. Fruit Empire already has swimming fatigue as its hazard. Don't stack another punishment onto a cozy base floor at launch.
- **Bright neon everywhere.** Builders were told their neon crystals were too bright [63]. Too many shadow-casting lights and particles cost FPS [58].
- **Trusting fan-site numbers.** grabores.wiki, merge-dig-simulator.wiki and similar sites look auto-generated [31][35]. Use them for ideas, not for balancing.

## Open questions

- Nodes or diggable blocks? (Recommendation above: nodes.)
- Do ores earn per tick on display (like fruit and Grab Ores!), or are they only sold and indexed? If they earn per tick, do they share the fruit storage cap or get their own slots?
- Merge rule: 2→1 like fruit, or 3→1 and 5→2 like Merge Dragons [45]?
- What does each floor extension cost after the first 100k shards? Shards only, or shards + cash + ore?
- At which rarity does glow start (Rare or Epic)?
- Should TNT and nukes damage nodes underground in the first version, or only fruit on the surface?
- What fraction of fruit income should ore income be at each stage?
- I could not confirm Mining Merge Tycoon's 25-mineral list, Grab Ores!' shiny value multiplier, or Mining Simulator's exact mine-reset rule. A play session in each would settle these.

## Sources

1. Mining Simulator Wiki, Ores: Earth table. https://mining-simulator.fandom.com/wiki/Mining_Simulator_Wiki:Tables/Ores:_Earth
2. Mining Simulator Wiki, Layers: Earth table (stone amount per layer). https://mining-simulator.fandom.com/wiki/Project:Tables/Layers:_Earth
3. Mining Simulator Wiki, Tools and Earth Shop tool table. https://mining-simulator.fandom.com/wiki/Tools and https://mining-simulator.fandom.com/wiki/Project:Tables/Tools:_Earth_Shop
4. Mining Simulator Wiki, Rebirth. https://mining-simulator.fandom.com/wiki/Rebirth
5. Mining Simulator Wiki, Backpacks and Earth Shop backpack table. https://mining-simulator.fandom.com/wiki/Backpacks and https://mining-simulator.fandom.com/wiki/Project:Tables/Backpacks:_Earth_Shop
6. Mining Simulator Wiki, Glowstone. https://mining-simulator.fandom.com/wiki/Glowstone
7. Mining Simulator Wiki, Rainbowite. https://mining-simulator.fandom.com/wiki/Rainbowite
8. Mining Simulator Wiki, Pets (evolving with ores). https://mining-simulator.fandom.com/wiki/Pets
9. Mining Simulator Wiki, Rebirths (MS2). https://mining-simulator.fandom.com/wiki/Rebirths_(MS2) and Roblox page https://www.roblox.com/games/9551640993
10. Mining Simulator Wiki, Tools (MS2). https://mining-simulator.fandom.com/wiki/Tools_(MS2)
11. Azure Mines Wiki, Public Mine. https://azuremines.fandom.com/wiki/Public_Mine
12. Azure Mines Wiki, Mine Condition. https://azuremines.fandom.com/wiki/Mine_Condition
13. Azure Mines Wiki, Stone (layer colors). https://azuremines.fandom.com/wiki/Stone
14. Azure Mines Wiki, Caves. https://azuremines.fandom.com/wiki/Caves
15. Azure Mines Wiki, Torches. https://azuremines.fandom.com/wiki/Torches
16. Azure Mines Wiki, Coal. https://azuremines.fandom.com/wiki/Coal
17. Azure Mines Wiki, Iron. https://azuremines.fandom.com/wiki/Iron
18. Azure Mines Wiki, Azure. https://azuremines.fandom.com/wiki/Azure
19. Azure Mines Wiki, Titanium (and Darkmatter: https://azuremines.fandom.com/wiki/Darkmatter). https://azuremines.fandom.com/wiki/Titanium
20. Azure Mines Wiki, Valhalum. https://azuremines.fandom.com/wiki/Valhalum
21. Azure Mines Wiki, Mining Operation. https://azuremines.fandom.com/wiki/Mining_Operation
22. Azure Mines Wiki, Base Stations. https://azuremines.fandom.com/wiki/Base_Stations
23. Azure Mines Wiki, Personal Tunnel. https://azuremines.fandom.com/wiki/Personal_Tunnel
24. Azure Mines Wiki, Layer Elevator. https://azuremines.fandom.com/wiki/Layer_Elevator
25. Azure Mines, Roblox game page (52M visits, berezaa Games Live). https://www.roblox.com/games/428114181/Azure-Mines
26. Miner's Haven Wiki, Large Ore Upgrader. https://minershaven.fandom.com/wiki/Large_Ore_Upgrader
27. Miner's Haven Wiki, Rebirth. https://minershaven.fandom.com/wiki/Rebirth
28. Grab Ores!, Roblox game page (11M visits, created 2026-03-28). https://www.roblox.com/games/82806856224736
29. YouTube, "Grabbing Super Rare Ores For Profit in Roblox" (Grab Ores! gameplay; auto-transcript read). https://www.youtube.com/watch?v=q0otw2naVIs
30. Rouniverse, "Grab Ores! Optimal Ore Merging & Progression" (fan article). https://rouniverse.com/articles/grab-ores-tips/
31. grabores.wiki, All Ores (unofficial fan site; numbers unconfirmed). https://grabores.wiki/ores/ ; video title matching the Diamond odds: https://www.youtube.com/watch?v=ml11zsjUACc
32. Mining Merge Tycoon, Roblox game page (403K visits). https://www.roblox.com/games/11255073656/Mining-Merge-Tycoon
33. Ore Island Merge Mining Simulator, Roblox game page (13K visits). https://www.roblox.com/games/100950607680214
34. Merge & Dig Simulator!, Roblox game page (11M visits). https://www.roblox.com/games/74569490529875 ; Merge & Dig! https://www.roblox.com/games/106480700466481
35. merge-dig-simulator.wiki, How to Merge Pickaxes (unofficial fan site). https://merge-dig-simulator.wiki/guides/how-to-merge-pickaxes/
36. Merge a Mineral!, Roblox game page (449K visits; includes stealing). https://www.roblox.com/games/87428660723605/Merge-a-Mineral
37. Mine a Mountain, Roblox game page (146M visits). https://www.roblox.com/games/125927821145949
38. Sportskeeda, "Mine a Mountain: A beginner's guide". https://www.sportskeeda.com/roblox-news/mine-mountain-a-beginner-s-guide
39. Mine Antarctica, Roblox game page ("Place crystals in your garden for MORE digging luck"). https://www.roblox.com/games/138686218420016
40. Sell Ores, Roblox game page (44M visits). https://www.roblox.com/games/122572082932179
41. Sportskeeda, "Sell Ores: A beginner's guide". https://www.sportskeeda.com/roblox-news/sell-ores-a-beginner-s-guide
42. +1 TNT Mining, Roblox game page (13.7M visits). https://www.roblox.com/games/101304595834078
43. Ultimate Mining Tycoon Wiki (Miraheze), The Mine. https://umt.miraheze.org/wiki/The_Mine ; Roblox page https://www.roblox.com/games/18680867089
44. Ultimate Mining Tycoon Wiki (Miraheze), Ores. https://umt.miraheze.org/wiki/Ores
45. Merge Dragons Wiki, Merging. https://mergedragons.fandom.com/wiki/Merging
46. Merge Dragons Wiki, Stone Bricks. https://mergedragons.fandom.com/wiki/Stone_Bricks
47. Drills Merge Master: Idle Mine, Google Play listing. https://play.google.com/store/apps/details?id=com.d954mas.drills.merge.master.idle
48. DevForum, "Procedural Infinite Mining Module (Depth-Based Ores + Auto-Generation)". https://devforum.roblox.com/t/procedural-infinite-mining-module-depth-based-ores-auto-generation/4091519
49. DevForum, "What I can do to reduce memory usage in infinite mining game?". https://devforum.roblox.com/t/what-i-can-do-to-reduce-memory-usage-in-infinite-mining-game/2331716
50. DevForum, "Infinite Mining Generation" tutorial. https://devforum.roblox.com/t/infinite-mining-generation/672149
51. DevForum, "Crystal Caves lighting". https://devforum.roblox.com/t/crystal-caves-lighting/797735
52. DevForum, "Atmosphere fog affecting cave lighting". https://devforum.roblox.com/t/atmosphere-fog-affecting-cave-lighting/3655036
53. Roblox Creator Docs, Global lighting (LightingStyle, Ambient). https://create.roblox.com/docs/environment/lighting
54. Roblox Creator Docs, Lighting class (Ambient, LightingStyle, Technology deprecated, FogEnd). https://create.roblox.com/docs/reference/engine/classes/Lighting
55. Roblox Creator Docs, Light sources. https://create.roblox.com/docs/effects/light-sources
56. Roblox Creator Docs, Atmosphere class. https://create.roblox.com/docs/reference/engine/classes/Atmosphere
57. Roblox Creator Docs, BloomEffect. https://create.roblox.com/docs/reference/engine/classes/BloomEffect
58. Roblox Creator Docs, Improve performance. https://create.roblox.com/docs/performance-optimization/improve
59. Mining Simulator, Roblox game page (833M visits, Rumble Studios). https://www.roblox.com/games/1417427737
60. Miner's Haven, Roblox game page (200M visits). https://www.roblox.com/games/258258996
61. DevForum, "How To Make a Dark Cave On a Map?" (trigger box; PointLight max range 60 claim). https://devforum.roblox.com/t/how-to-make-a-dark-cave-on-a-map/1768466
62. DevForum, "Feedback on lighting of cave" (PointLight on Attachment in neon part). https://devforum.roblox.com/t/feedback-on-lighting-of-cave/1780310
63. DevForum, "[Feedback] Crystal Cave" (neon too bright; glass + light or bloom). https://devforum.roblox.com/t/feedbackcrystal-cave/518368
64. DevForum, "Making a cave brighter" (shadows vs Ambient readability). https://devforum.roblox.com/t/making-a-cave-brighter/639027

65. DevForum announcement, "Extended Light Ranges: Doubling the Limit to 120" (23 Sep 2025). https://devforum.roblox.com/t/extended-light-ranges-doubling-the-limit-to-120/3954367
66. Roblox client API dump (community mirror, version 0.741), `Lighting.ExtendLightRangeTo120` (RolloutState, NotScriptable). https://raw.githubusercontent.com/MaximumADHD/Roblox-Client-Tracker/roblox/Full-API-Dump.json
67. Local code: `~/fruit-empire/src-export-2026-10-04/ReplicatedStorage.IndexRewards.lua` (`Rewards.Shards`) and `ReplicatedStorage.GardenConfig.lua` (`IndexShardsLow = 5`, `IndexShardsHigh = 2500`)
68. Local code: `~/fruit-empire/src-export-2026-10-04/ServerScriptService.GardenService.RebirthLock.lua` (rebirth wipes the Index and grants `indexShardsEarned` again)
69. Local code: `~/fruit-empire/src-export-2026-10-04/ReplicatedStorage.CrownConfig.lua` (`C.Rewards`, `C.DecreeCooldown = 360`, Merge Rush `reward = 400`)

Internal: [[Brain Dump 2026-10-04]] (owner's requests and the ore reference image), [[UI Kit Inventory]] (rarity tier colors), [[Steal a Pet - Bats and Rebirth Knockback]].

## Review (critic pass, 2026-10-04)

- **Corrected:** the PointLight range. The note said a 60-stud cap was not confirmed. Roblox did cap lights at 60 and has since raised the cap to 120 [65][66].
- **Added:** proposal 9, the shard-supply check for the 100k underground, from the local code [67][68][69].
- **Verified:** the rarity colours match [[UI Kit Inventory]] (Legendary is listed there as orange-gold).
- **Still not confirmed:** Mining Merge Tycoon's 25-mineral list, the Grab Ores! shiny multiplier and its fan-site ore values, Ultimate Mining Tycoon's Unobtainium value, and Mining Simulator's mine-reset rule.
