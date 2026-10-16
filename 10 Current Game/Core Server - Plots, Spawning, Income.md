---
tags: [current-game, code-map]
date: 2026-10-04
sources:
  - ServerScriptService.GardenService.lua
  - ServerScriptService.GardenService.PlotRegistry.lua
  - ServerScriptService.GardenService.Stats.lua
  - ServerScriptService.GardenService.Progression.lua
  - ServerScriptService.GardenService.FruitFactory.lua
  - ServerScriptService.GardenService.Upgrades.lua
  - "context only: ServerScriptService.GardenService.Ambient.lua (spawn loop and income tick), ReplicatedStorage.GardenConfig.lua, ReplicatedStorage.FruitEconomy.lua, ReplicatedStorage.Fruits.FruitIndex.lua, ReplicatedStorage.IndexRewards.lua, ServerScriptService.GardenService.Persistence.lua, ServerScriptService.GardenService.RebirthLock.lua, ServerScriptService.GardenService.Tools.lua, ServerScriptService.GardenService.Carrying.lua, ServerScriptService.GardenService.Merging.lua"
---
# Core Server: Plots, Spawning, Income

Exported 2026-10-04 from `/Users/samarthvaka/fruit-empire/src-export-2026-10-04`. Vision source: [[Brain Dump 2026-10-04]].

Terms used below:
- **Plot**: one garden pen under `Main Island.Plots`. One player owns one plot.
- **Soil**: the flat part of a plot that fruit sit on.
- **Rank**: a fruit's place on the merge ladder. Rank 64 is Blueberry (the worst). Rank 1 is Soul Fruit (the best). Two fruit of rank R merge into one of rank R-1 (`FruitIndex.lua:95`).
- **Rarity tier**: the colour group a fruit belongs to (COMMON up to TRANSCENDENT), stored as `tierName` in `FruitIndex.lua`.
- **Upgrade level**: a 0-based number per upgrade. Level 0 means "bought nothing".
- **Spawn tier** (shown in game as "Spawn Rarity"): the upgrade that decides which fruit can spawn.
- **Profile**: the player's saved document. `GardenStore` reads and writes it. `Persistence` turns it into game state.
- **Attribute**: a named value set with `SetAttribute` on an Instance. Attributes replicate to clients.
- **Remote**: a `RemoteEvent` that carries a message between server and client.
- **G**: one shared Lua table. The main script passes it to every module so they can see each other's functions (`GardenService.lua:88-98`).
- **Store.IsReady(player)**: true once the player's profile has finished loading.

## What it does

This subsystem is the server's core for each player's garden. It hands every joining player a free plot and frees it when they leave. It decides which fruit can spawn on a plot, where it lands, and builds the fruit part with its prompt, name tag and aura. It prices a plot (sum of each fruit's income) and the cash multiplier (friends x Index x rebirths) that the income tick uses. It sells the four cash upgrades (capacity, spawn rate, spawn tier, plot size) and rebuilds the soil and fence when the plot grows. The actual loops that spawn fruit and pay cash live in `Ambient.lua`, but they only call functions defined here.

## Scripts

| Script | Role | Lines |
|---|---|---|
| `ServerScriptService.GardenService.lua` | Entry Script. Creates remotes, holds shared state, loads every module in order, claims and releases plots, walk speed and invisibility, player join and leave, shutdown save. | 824 |
| `ServerScriptService.GardenService.PlotRegistry.lua` | Builds the plot table from `Island.Plots`. Finds each soil, places the spawn pad, builds the owner headshot. | 179 |
| `ServerScriptService.GardenService.Stats.lua` | Leaderboard columns (`leaderstats`), the `watched` and `conns` registries, and the dev counters. | 99 |
| `ServerScriptService.GardenService.Progression.lua` | Cash and rebirth publishing, the crown hand-off, friend counts, Best Merge, the Index (discover, claim, bonus), shards, king effects, and the cash multiplier. | 394 |
| `ServerScriptService.GardenService.FruitFactory.lua` | Free-space search on a plot, plot pricing (`plotBase`), and building a fruit part (prompt, name tag, aura, sparkle, merge sound). | 377 |
| `ServerScriptService.GardenService.Upgrades.lua` | Upgrade levels and purchase, plot resize with fence rebuild, plot load and capacity, spawn pick (with the Luck potion), and `trySpawnIn`. | 431 |

## How it works

### 1. Startup (module load order matters)
1. The main script finds `Main Island` and its `Plots` folder (`GardenService.lua:34-35`). It destroys any old `SpawnedFruits` folder and makes a fresh one (`GardenService.lua:37-41`). All live fruit, on every plot, sit in this one folder.
2. It makes the `GardenRemotes` folder and creates each RemoteEvent if missing (`GardenService.lua:43-86`).
3. It fills `G` and requires modules one by one, in this order: PlotRegistry (`:117`), Stats (`:398`), Progression (`:402`), FruitFactory (`:409`), Upgrades (`:413`), then Carrying, Tools, Storage, Merging, RebirthLock, Persistence, the four Crown modules (`:487-518`), DevPanel (`:751`) and Ambient last (`:824`). Each module can only use what earlier modules put into `G`, unless it reads `G` late at call time.
4. **PlotRegistry** loops over every Model in `Plots` (`PlotRegistry.lua:141-160`). For each it:
   - picks the soil as the BasePart with the biggest X times Z area (`soilOf`, `PlotRegistry.lua:21-30`);
   - records `name, model, soil, topY, owner, sizeLevel` (`PlotRegistry.lua:145-152`);
   - places an invisible `PlotSpawn` SpawnLocation near the plaza end, facing down the plot (`placeSpawn`, `PlotRegistry.lua:112-139`);
   - builds a hidden round headshot billboard (`buildOwnerIcon`, `PlotRegistry.lua:38-92`).
   Plots are then sorted by name (`PlotRegistry.lua:161`), and `claimPlot` walks them in that order (`GardenService.lua:432`), so Plot0 is handed out first (`Upgrades.lua:81`).
5. **Upgrades** copies one Post, Rail and Kerb to use as fence templates, picked by majority vote across all plots (`FENCE_STYLE`, `Upgrades.lua:84-112`). This runs once, before any plot is resized.

### 2. A player joins
1. `watchPlayer` (`GardenService.lua:608-698`) guards against running twice (`:612-613`). It builds leaderstats (`setupStats`, `Stats.lua:43-93`), the stored and tools folders, and a zero balance (`GardenService.lua:615-618`).
2. In Studio only, it wires three one-shot dev attributes: `DevSetCash`, `DevSetShards`, `DevSetPlotSize` (`GardenService.lua:631-684`).
3. `claimPlot(player, false)` (`GardenService.lua:688`) takes the first plot with no owner (`GardenService.lua:431-448`). It stamps `OwnerUserId` and `OwnerName` on the plot model, `PlotName` on the player, sets `RespawnLocation` to the pad, shows the headshot, and clears any leftover fruit (`:434-440`). `false` means "do not seed": the profile decides what goes on the plot. If all plots are taken it warns and returns nil (`:446-447`).
4. Character hooks are connected (`GardenService.lua:690-694`). `watchCharacter` moves the body to the plot's stand point (`:575-578`), stamps `BaseHipHeight` (`:585`), and re-runs `applyWalkSpeed` and `applyVisibility` (`:592-596`). On death it drops the carried fruit and clears potions (`:601-604`).
5. `refreshFriendCounts` runs (`GardenService.lua:695`), then `Store.Load` starts in a new thread (`:697`). When the profile lands, `Persistence.applyProfile` publishes cash, rebirths, upgrades and plot size, then restores fruit (`Persistence.lua:196-309`). A brand-new player gets `seedPlot`, which calls `trySpawnIn` `Config.InitialFruits` times (`GardenService.lua:423-425`, `Persistence.lua:299-300`).

### 3. Spawning (loop in Ambient, logic here)
1. Every `SPAWN_TICK` = 0.5 s (`Upgrades.lua:292`) the loop wakes (`Ambient.lua:412`).
2. It only touches plots whose owner is loaded (`Ambient.lua:419`). Each plot keeps its own `nextSpawnAt` timer, set from `Config.SpawnInterval(spawnRate level)` divided by `G.spawnBoost` (the Royal Decree "Fruit Frenzy", 1 otherwise) (`Ambient.lua:421-426`). A plot with no owner, or whose owner is still loading, has its timer cleared (`Ambient.lua:438`).
3. When due, it calls `trySpawnIn(plot)` (`Upgrades.lua:403-422`), which:
   - returns `"full"` if `plotLoad(plot) >= capacityFor(plot)` (`Upgrades.lua:406`). `plotLoad` counts fruit on the soil, plus fruit from this plot in someone's hands, plus fruit a merge animation has eaten but not yet replaced (`Upgrades.lua:286-288`). Capacity is `Config.Capacity(owner's capacity level)` (`Upgrades.lua:46-48`).
   - asks `spawnPlan` for the level and Luck state (`Upgrades.lua:372-379`). With Luck active and below the top level, the pool is one level higher (`tierBonus`). At the top level, the same pool is tilted toward its best fruit.
   - picks a name by weighted roll (`pickSpawnName`, `Upgrades.lua:351-366`). The pool is `Config.SpawnTable(level)`: level L holds the bottom L+1 fruit, each step down `SpawnRarityFalloff` (0.6) times as likely (`GardenConfig.lua:85-96`).
   - finds a random free spot, up to 40 tries (`randomSpotIn`, `FruitFactory.lua:96-107`). Two fruit must be at least `(r1 + r2) * MinFruitGap` apart (`isFreeIn`, `FruitFactory.lua:51-58`). Returns `"nospace"` if all 40 fail.
   - builds and places the fruit (`placeFruit`, `FruitFactory.lua:359-367`). It is tagged with `PlotName`, put on the soil surface with a random spin, and parented to `SpawnedFruits`.
   - if Luck made this fruit possible, it gets a `Lucky` attribute and a gold glint (`markLucky`, `Upgrades.lua:383-398`, `:418`). Carrying reads `Lucky` so picking it up still enters the Index (`Carrying.lua:184-185`).
4. The reason string is stored for the dev panel (`Ambient.lua:427`, `Stats.lua:28-36`).

### 4. Building a fruit (`createFruit`, `FruitFactory.lua:224-357`)
1. Clones the MeshPart template from `ReplicatedStorage.Fruits` (`FruitIndex.Template`).
2. Sets it Anchored, `CanCollide`, `CanQuery`, `CanTouch` all false, Massless (`FruitFactory.lua:232-236`).
3. Adds a ProximityPrompt named `Interact`, key E, no hold, range `PromptDistance` (`FruitFactory.lua:238-250`).
4. Adds a hidden `NameTag` billboard coloured by `RarityColors[tierName]` (`FruitFactory.lua:256-320`). Legendary and up get a two-colour gradient the client animates (`FruitFactory.lua:293-309`).
5. Ranks listed in `Config.FruitAuras` (ranks 1 and 2, `GardenConfig.lua:664-674`) get emitters cloned from ServerStorage (`FruitFactory.lua:324-354`).
6. Separately, `sparkleOn` plays a one-time sparkle on a first discovery (`FruitFactory.lua:148-195`) and `playMergeSound` plays the merge sound on the new fruit (`FruitFactory.lua:204-222`).

### 5. Income (loop in Ambient, pricing here)
1. Every `Config.IncomeTick` = 5 s (`Ambient.lua:450`), for each loaded player:
2. `publishMultiplier` writes `CashMultiplier` (`Ambient.lua:462`, `Progression.lua:341-343`).
3. `plotBase(plot)` sums `Economy.ForName(fruit.Name)` over every fruit in the folder tagged with this plot (`FruitFactory.lua:39-45`, `Ambient.lua:465`). Fruit in hands do not earn, because carrying moves the fruit out of the folder (`Upgrades.lua:262-265`).
4. `cashMultiplier` = `(1 + FriendBonus x friends) x (1 + indexBonus) x (1 + Rebirths x RebirthCashBonus)` (`Progression.lua:324-336`). `indexBonus` sums only CLAIMED Index entries (`Progression.lua:203-214`).
5. The balance grows by `rate x dt` (`Ambient.lua:466-468`). `CashPerSecond` is set (`Ambient.lua:469`), which redraws the `Money/s` leaderstat (`Stats.lua:88-92`). `publishCash` writes the `Cash` attribute and leaderstat and re-runs the crown check (`Progression.lua:80-88`).
6. The owner gets the `Income` remote with `(multiplier, dt)` only (`Ambient.lua:477-478`). The client prices each fruit itself.
7. Offline pay reuses `plotBase` with Index and rebirth bonuses but NOT friends (`Persistence.lua:191`).

### 6. Buying an upgrade (`buyUpgrade`, `Upgrades.lua:301-343`)
1. Client fires `Upgrade` with a key string (`Upgrades.lua:345`). The key must be in `Config.UpgradePrices` (`:303`) and the profile must be loaded (`:304-307`).
2. Price is `Config.UpgradePrice(key, level)`. nil means maxed (`:309-311`).
3. Cash is checked and taken from `balances` (`:313-321`). The level attribute goes up by one (`:320`).
4. Side effects: `spawnRate` clears the plot's timer (`:325`). `spawnTier` discovers every fruit up to the new tier in the Index (`:335`). `plotSize` resizes the plot now (`:338`).
5. A save is forced (`:342`).

### 7. Plot resize (`applyPlotSize`, `Upgrades.lua:242-260`)
1. Clamps the level and returns early if `plot.sizeLevel` already matches (`:243-244`).
2. Sets the soil size from `Config.PlotSize(level)` about its centre. The CFrame is never touched (`:245-253`).
3. `buildFence` deletes the old fence and kerbs and rebuilds them from the soil's new size, using the cloned templates (`Upgrades.lua:160-228`).
4. `placeSpawn` moves the pad and stand point (`:256`). `PlotSizeLevel` is stamped on the model (`:259`).
5. Callers: purchase (`Upgrades.lua:338`), profile load (`Persistence.lua:277`), plot release (`GardenService.lua:456`), rebirth (`RebirthLock.lua:97`), Studio dev door (`GardenService.lua:665`).

### 8. Index, shards and king effects (`Progression.lua`)
- `discover(player, name)` adds an IntValue (0) under `player.IndexState` the first time, and pulls the next autosave closer (`Progression.lua:173-185`). It ignores names not in `FruitIndex` (`:174`).
- `claimIndex` turns a 0 into a 1 and pays money plus shards from `IndexRewards.For(rank)`, then forces a save (`Progression.lua:345-372`).
- `indexComplete` says whether all 64 fruit are discovered. Rebirth is gated on it (`Progression.lua:311-320`, `RebirthLock.lua:52`).
- `buyKingEffect` / `equipKingEffect` spend shards on crown auras and publish `KingOwned` and `KingEffect` (`Progression.lua:226-288`).
- `refreshKing` hands off to `G.crownRefresh` when the Crown module is loaded (`Progression.lua:54`). The old "richest wears the crown" code below it only runs if that is missing.

### 9. A player leaves
`teardownPlayer` (`GardenService.lua:703-735`) saves FIRST, then destroys. It reclaims the carried fruit and saves (`:707-712`), then `releaseHold` and `releasePlot` (`:714-715`). `releasePlot` clears fruit, resets plot size to 0, clears owner fields, removes any lock dome, and drops the merge ledger (`GardenService.lua:450-480`). All per-player tables are cleared (`:716-733`). `onPlayerLeaving` must take exactly one parameter (`GardenService.lua:737-744`). On shutdown every player goes through the same path (`GardenService.lua:770-781`).

## Data

### Saved fields (profile)
This subsystem never writes the DataStore itself. It owns the live values that `Persistence.captureProfile` reads and that `applyProfile` writes back.

| Profile field | Live source in this subsystem | Captured | Restored |
|---|---|---|---|
| `cash` | `balances[player]` (`GardenService.lua:122`) | `Persistence.lua:49` | `Persistence.lua:197` then `publishCash` |
| `best` | `bestRank[player]` (`GardenService.lua:122`, set by `recordMerge`, `Progression.lua:129-140`) | `Persistence.lua:51-53` | `Persistence.lua:212` via `recordMerge` |
| `index` | IntValues under `player.IndexState` (`Progression.lua:160-185`) | `Persistence.lua:69-73` | `Persistence.lua:228-236` |
| `shards` | `Shards` attribute (`setShards`, `Progression.lua:216-218`) | `Persistence.lua:74` | `Persistence.lua:237` |
| `rebirths` | `Rebirths` attribute (`publishRebirths`, `Progression.lua:95-101`) | `Persistence.lua:75` | `Persistence.lua:267` |
| `kingOwned`, `kingEquipped` | `KingOwned`, `KingEffect` attributes (`publishKing`, `Progression.lua:234-243`) | `Persistence.lua:77-78` | `Persistence.lua:268` |
| `upgrades.capacity / spawnRate / spawnTier / plotSize` | `Upgrade_<key>` attributes (`Upgrades.lua:30-32`, `:294-299`) | `Persistence.lua:92-94` | `Persistence.lua:270`, plot size `:277` |
| `plot` (fruit name to count) | fruit in `SpawnedFruits` with this `PlotName` (`fruitOfPlot`, `FruitFactory.lua:26-34`) plus `pendingMerge[plot]` (`GardenService.lua:144`) | `Persistence.lua:98-105` | `restorePlot`, `Persistence.lua:309` |

### Player attributes

| Attribute | Written / read | Where |
|---|---|---|
| `PlotName` | written on claim, cleared on release | `GardenService.lua:437`, `:478` |
| `Cash` | written | `Progression.lua:82` |
| `CashPerSecond` | written by the income tick; read for the `Money/s` column | `Ambient.lua:469`, `Stats.lua:88-92` |
| `CashMultiplier` | written | `Progression.lua:342` |
| `FriendCount` | written | `Progression.lua:122` |
| `Rebirths` | written; read by `rebirthMultiplier` | `Progression.lua:96`, `:325` |
| `BestMerge` | written (display name) | `Progression.lua:133` |
| `Shards` | written (floored) | `Progression.lua:217` |
| `KingOwned` (comma list), `KingEffect` (one key) | written | `Progression.lua:241-242` |
| `Upgrade_capacity`, `Upgrade_spawnRate`, `Upgrade_spawnTier`, `Upgrade_plotSize` | written and read | `Upgrades.lua:36`, `:296-297`, `:320` |
| `<Effect>EndsAt`, `<Effect>Span` | cleared on death and expiry (set in Tools) | `GardenService.lua:560-563`, `:803-804` |
| `InArena` | read (no invisibility in the arena) | `GardenService.lua:266` |
| `DevSetCash`, `DevSetShards`, `DevSetPlotSize` | Studio-only one-shot requests | `GardenService.lua:649-683` |

### Other instance attributes

| Instance | Attribute | Where |
|---|---|---|
| Plot model | `OwnerUserId`, `OwnerName` | `GardenService.lua:435-436`, `:459-460` |
| Plot model | `LockedUntil` (cleared on release) | `GardenService.lua:470` |
| Plot model | `PlotSizeLevel` | `Upgrades.lua:259` |
| Fruit part | `PlotName` (which plot owns it) | `FruitFactory.lua:362`, read `:29` |
| Fruit part | `Lucky` | `Upgrades.lua:384` |
| Character | `BaseHipHeight` (written), `CloudSink` (read) | `GardenService.lua:585`, `:243-246` |
| Character parts | `PreInvis`, `PreInvisOn`, `PreInvisShadow` | `GardenService.lua:293-331` |
| Humanoid | `PreInvisNames` | `GardenService.lua:343-351` |
| workspace | `KingUserId`, `KingName` (legacy crown path) | `Progression.lua:46-47`, `:71-72` |

Runtime plot table fields (not attributes): `name, model, soil, topY, owner, sizeLevel` (`PlotRegistry.lua:145-152`), `spawnCF`, `pad` (`PlotRegistry.lua:120`, `:138`), `icon`, `iconImage` (`PlotRegistry.lua:86-91`), `nextSpawnAt` (`Ambient.lua:422-425`), `dome`, `lockedUntil` (`GardenService.lua:467-469`).

### Remotes
All are `RemoteEvent`s in `ReplicatedStorage.GardenRemotes`, made at startup if missing (`GardenService.lua:43-86`: the `remote()` helper at `:49-57`, the calls at `:59-83`). `GardenService.lua` makes no RemoteFunctions. The game's one RemoteFunction, `RoyalQuery`, is made by the Royal System (`Crown.lua:55-60`).

| Remote | Direction | Payload | Handler |
|---|---|---|---|
| `Drop` | client to server | none | `dropFromPlayer` (`GardenService.lua:528`) |
| `Store` | client to server | none | `storeCarried` (`:533`) |
| `Equip` | client to server | stored fruit name | `equipStored` (`:534`) |
| `BuyTool` | client to server | tool name | `buyTool` (`:529`) |
| `EquipTool` | client to server | tool name | `equipTool` (`:530`) |
| `KingBuy` | client to server | king effect key | `buyKingEffect` (`:531`, `Progression.lua:245`) |
| `KingEquip` | client to server | king effect key | `equipKingEffect` (`GardenService.lua:532`, `Progression.lua:271`) |
| `IndexClaim` | client to server | fruit name | `claimIndex` (`Progression.lua:374`) |
| `Upgrade` | client to server | upgrade key | `buyUpgrade` (`Upgrades.lua:345`) |
| `Lock` | client to server | none | `lockPlot` (`GardenService.lua:546`) |
| `MergeAll` | client to server | none | `mergeAllOn` (`:536`) |
| `Rebirth` | client to server | none | `doRebirth` (`:537`) |
| `Clear` | client to server | none | `clearLowest` (`:535`) |
| `Notify` | server to client | warning text (red, top) | `GardenService.lua:66-70` |
| `Notice` | server to client | good-news text (green, bottom) | `GardenService.lua:70` |
| `Income` | server to owner | `(multiplier, dt)` | fired at `Ambient.lua:478` |

Fruit pick-up and merge do not use a remote. They use the fruit's ProximityPrompt, caught by `ProximityPromptService.PromptTriggered` (`GardenService.lua:522-526`).

### Leaderstats (`Stats.lua:43-93`)
`Money/s` (StringValue, `:56-59`), `Cash` (StringValue, `:63-66`), `Best Merge` (StringValue, `:68-71`), `Rebirths` (IntValue so it sorts as a number, `:76-79`).

### Config keys read

| Area | Keys |
|---|---|
| World | `IslandName`, `PlotsFolder`, `FruitFolder` |
| Plot and placement | `InitialFruits`, `SpawnMargin`, `MinFruitGap`, `PlotStandMargin`, `PlotStandBand`, `PlotSize()`, `PlotSizes` |
| Owner icon | `OwnerIconStuds`, `OwnerIconHeight`, `OwnerIconStroke`, `OwnerIconMaxDistance` |
| Upgrades | `UpgradeKeys`, `UpgradePrices`, `UpgradePrice()`, `MaxLevel()`, `Capacity()`, `SpawnTable()`, `SpawnInterval()` (in Ambient), `UpgradeTooPoor`, `LoadingWarning` |
| Luck potion | `Effects.Luck.tierBonus`, `Effects.Luck.topFalloff` |
| Movement and potions | `WalkSpeed`, `JumpPower`, `Effects.Speed.walkMultiplier`, `Effects.Jump.jumpPower`, `Effects.Invisible.trail*`, `SkyJumpY`, `SkyJumpMultiplier`, `CloudSinkDrop` |
| Fruit look | `PromptDistance`, `NameTagStuds`, `NameTagHeight`, `NameTagLift`, `NameTagForward`, `NameTagMaxDistance`, `NameTagFont`, `NameTagMaxTextSize`, `NameTagStroke`, `NameTagSweepTiers`, `RarityColors`, `RarityGradients`, `FruitAuras`, `DiscoverTemplate`, `DiscoverScaleMin/Max`, `DiscoverHold`, `DiscoverFade`, `MergeSoundId`, `MergeSoundVolume`, `MergeSoundRange`, `MergeSoundMaxLife` |
| Economy and Index | `FriendBonus`, `RebirthCashBonus`, `IndexFolder`, `SaveTouch`, `IncomeTick` (in Ambient) |
| King | `KingEnabled`, `KingEffects`, `KingEffectByKey`, `KingDefault`, `KingYouAre`, `KingIs`, `KingBought`, `KingEquipped`, `KingNotOwned`, `KingAlreadyOwned`, `ToolNotEnoughShards` |
| Gamepass | `MergeAllGamepassId` (`GardenService.lua:542`) |

### Constants in code

| Constant | Value | Where |
|---|---|---|
| `SPAWN_TICK` | 0.5 s | `Upgrades.lua:292` |
| Fence shape (`POST_OUT`, `KERB_OUT`, `POST_SPAN`, `GATE_HALF`, post, rail, kerb sizes) | 3.03, 0.61, 17.69, 9.68, ... | `Upgrades.lua:61-64` |
| Fence fallback colours | brown RGB | `Upgrades.lua:115-119` |
| Random spot tries | 40 | `FruitFactory.lua:100` |
| Nearby spot search | 8 rings x 8 points, ring step `radius * 1.5` | `FruitFactory.lua:112-114` |
| Spawn pad | 6 x 1 x 6, stand point 5 studs inside the plaza edge, 3.5 up | `PlotRegistry.lua:118-119`, `:128` |
| Effect sweep | every 0.25 s | `GardenService.lua:795` |

## Numbers

| What | Value | Config key / source |
|---|---|---|
| Capacity | 6 at level 0, +1 per level, max 35 (29 buys) | `CapacityStart`, `CapacityTop` (`GardenConfig.lua:51-56`) |
| Spawn interval | 10 s at level 0, -0.5 s per level, floor 2.5 s (15 buys) | `SpawnRateStart`, `SpawnRateStep`, `SpawnRateFloor` (`GardenConfig.lua:60-66`) |
| Spawn tier | 64 levels (0..63), level L spawns the bottom L+1 fruit | `SpawnTiers` (`GardenConfig.lua:86-96`) |
| Spawn odds falloff | 0.6 per step down (top fruit about 40%) | `SpawnRarityFalloff` (`GardenConfig.lua:68-85`) |
| Plot size | 29.5 x 47.1 at level 0 up to 48 x 78 at level 4 | `PlotSizes` (`GardenConfig.lua:117-123`) |
| Upgrade price | `base x growth^(n-1)`, rounded to 3 figures | `UpgradePriceBase` (`GardenConfig.lua:143`), `UpgradePriceGrowth` (`:157`) |
| Price bases | capacity 100, spawnRate 150, spawnTier 250, plotSize 2,500 | `UpgradePriceBase` |
| Price growth | capacity 1.90, spawnRate 3.40, spawnTier 2.60, plotSize 8.0 | `UpgradePriceGrowth` |
| Price ladders (computed from the two lines above) | capacity 100 to about 6.4e9 (total about 1.35e10); spawnRate 150 to about 4.1e9 (total about 5.9e9); spawnTier 250 to about 1.3e28 (total about 2.2e28); plotSize 2.5k, 20k, 160k, 1.28M | derived; ladder lengths stated at `GardenConfig.lua:126-127`, plotSize at `:153` |
| Starter fruit | 3 on claim or rebirth | `InitialFruits` (`GardenConfig.lua:287`) |
| Spawn edge margin | 0.6 studs | `SpawnMargin` (`GardenConfig.lua:288`) |
| Fruit spacing | 1.15 x summed radii | `MinFruitGap` (`GardenConfig.lua:289`) |
| Prompt range | 12 studs | `PromptDistance` (`GardenConfig.lua:304`) |
| On-plot slack | 2.5 studs sideways, 12 studs vertical band | `PlotStandMargin` (`GardenConfig.lua:321`), `PlotStandBand` (`:940`) |
| Fruit income | Blueberry 1 $/s; each rank up x2.10 (bottom) ramping to x2.60 (top); Soul Fruit about 2e23 $/s; Apple about 20 $/s (computed) | `Economy.Base`, `StepLow`, `StepHigh` (`FruitEconomy.lua:18`, `:25-27`) |
| Income tick | 5 s | `IncomeTick` (`GardenConfig.lua:372`) |
| Friend bonus | +10% per friend in server | `FriendBonus` (`GardenConfig.lua:357`) |
| Rebirth bonus | +100% per rebirth, additive (x2, x3, ...) | `RebirthCashBonus` (`GardenConfig.lua:1000`) |
| Full Index bonus | +50% (weighted by tier, doubling per tier) | `IndexBonusTotal` (`GardenConfig.lua:393`), `IndexRewards.lua:38-56` |
| Index claim money | 60 s of that fruit's income | `IndexMoneySeconds` (`GardenConfig.lua:394`) |
| Index claim shards | 5 (commonest) to 2,500 (rarest) | `IndexShardsLow`, `IndexShardsHigh` (`GardenConfig.lua:395-396`) |
| Offline pay | 3% of live rate, max 5 hours | `OfflineRate`, `OfflineCapSeconds` (`GardenConfig.lua:753-754`) |
| King aura prices (shards) | Fire 4,000, Water 6,000, Slashes 10,000, Rift 20,000; Crown free | `KingEffects` (`GardenConfig.lua:1236-1242`) |
| Luck potion | +1 spawn tier level; at top level falloff 0.35 instead of 0.6 | `Effects.Luck` (`GardenConfig.lua:1781-1788`) |
| Save pull-forward | next autosave within 8 s after a merge, discovery or purchase | `SaveTouch` (`GardenConfig.lua:1132`) |
| Walk speed | 20 | `WalkSpeed` (`GardenConfig.lua:943`) |
| Jump power | 50; x1.6 above Y = 430 | `JumpPower` (`GardenConfig.lua:1793`), `SkyJumpY`, `SkyJumpMultiplier` (`:1525-1526`) |
| Owner icon | 7 studs wide, 25 studs above soil, hidden past 400 studs | `OwnerIcon*` (`GardenConfig.lua:293-300`) |

## Extension points for the expansion

### 1. Rebirth-gated rarity
- **Spawn ceiling.** Rarity tiers by rank: COMMON = ranks 60-64, UNCOMMON = 52-59 (`FruitIndex.lua:61-73`). Apple is rank 60, the 5th fruit up, so "best fruit is Apple" means spawn tier level 4 at 0 rebirths. Clamp the level in `levelOf` (`Upgrades.lua:34-37`) or in `spawnPlan` (`Upgrades.lua:372-379`) to a rebirth-based cap. `buyUpgrade` must also refuse to buy past the cap (`Upgrades.lua:309-311`), or players pay for a level that does nothing.
- **Luck.** `spawnPlan` lets Luck go one level above the owner and tilt at `MaxLevel("spawnTier")` (`Upgrades.lua:376-378`). Both must use the rebirth cap, not the global max, or Luck leaks next-tier fruit.
- **Index gate.** `indexComplete` counts all 64 fruit (`Progression.lua:311-320`). With a rebirth cap a new player can never complete it, so rebirth would be impossible. Change it to count only fruit in unlocked tiers. `discoverThroughTier` (`Progression.lua:190-199`) will then only reach the cap, which is fine.
- **Shard and money cost.** Rebirth reads `indexComplete` at `RebirthLock.lua:52`. Shard and cash costs would be added there. `setShards` (`Progression.lua:216-218`) and `balances` are the writers.
- **Slight boost.** Today the boost is +100% per rebirth (`Progression.lua:324-326`, `GardenConfig.lua:1000`). "Slight" means lowering `RebirthCashBonus`. It also changes offline pay (`Persistence.lua:191`).
- **Merge cap.** Merging stops only at rank 1 today (`Merging.lua:179`). That is outside this subsystem, but it must use the same cap function.
- **+1 storage.** Lives in Storage, outside this subsystem.
- **Risk.** Existing saves already hold fruit and upgrade levels above the new caps. `restorePlot` will put them back as-is (`Persistence.lua:124-168`). Decide on a migration.

### 2. Bats coloured by rarity
- **Colour.** Use `Config.RarityColors[tierName]` like the name tag does (`FruitFactory.lua:283`). That keeps bat colour and fruit colour the same.
- **Upgrade trigger.** Every rebirth count change goes through `publishRebirths` (`Progression.lua:95-101`): load (`Persistence.lua:267`), rebirth (`RebirthLock.lua:118`), dev wipe (`DevPanel.lua:167`). Hook bat tier updates there, or read the `Rebirths` attribute.
- **Risk.** Knockback must not write Humanoid speed directly. `applyWalkSpeed` is the only writer (`GardenService.lua:187-190`) and re-runs on respawn, expiry and the Ambient sweep, so a direct write gets overwritten.

### 3. Explosives that drain fruit health
- **Health value.** Stamp a health attribute in `createFruit` (`FruitFactory.lua:224-357`) or `placeFruit` (`FruitFactory.lua:359-367`). Every fruit, spawned or restored, passes through `placeFruit`.
- **Hit detection.** Fruit are `CanQuery = false` and `CanTouch = false` (`FruitFactory.lua:232-236`). Raycasts and Touched will not hit them. Use distance, as TNT does today (`Tools.lua:279-283`).
- **Health bar.** The `NameTag` billboard (`FruitFactory.lua:256-273`) is the natural place. It is hidden by default and the client toggles it.
- **Death.** A drained fruit must lose `PlotName` before `Destroy`, the same order TNT uses (`Tools.lua:284-287`). Otherwise `plotLoad` and `plotBase` still count it for a moment.
- **Income.** `plotBase` pays full income regardless of damage (`FruitFactory.lua:39-45`). Decide if damaged fruit earn less.
- **Risk.** Saves store only name to count (`Persistence.lua:98`), so health resets to full on rejoin unless the save format changes. `Busy` fruit (mid-drop or mid-merge) are skipped by TNT (`Tools.lua:280-282`); keep that rule.

### 4. Underground base (100k shards) and ores
- **New upgrade key.** Adding a key to `Config.UpgradeKeys` (`GardenConfig.lua:37`) makes `publishUpgrades` (`Upgrades.lua:294-299`) and `captureProfile` (`Persistence.lua:92-94`) save it for free. It needs a price list in `Config.UpgradePrices`; only keys with a step count are generated (`GardenConfig.lua:170-186`).
- **Shards, not cash.** `buyUpgrade` always charges `balances` (`Upgrades.lua:313-320`). `Config.UpgradeCurrency` exists (`GardenConfig.lua:190`) but nothing reads it. Add a currency branch that uses `setShards`.
- **Rebirth wipes it.** Rebirth calls `publishUpgrades(player, nil)`, which zeroes every key in `UpgradeKeys` (`RebirthLock.lua:95`, `Upgrades.lua:294-299`). A 100k-shard purchase would be lost on rebirth unless excluded.
- **Geometry.** Follow `plotSize`: one apply function, called from purchase (`Upgrades.lua:338`), load (`Persistence.lua:276-278`), release (`GardenService.lua:456`), rebirth (`RebirthLock.lua:97`) and the dev door (`GardenService.lua:657-683`).
- **Ores must not go in `SpawnedFruits`.** Anything there with a `PlotName` counts toward capacity (`Upgrades.lua:286-288`), is priced by `plotBase` (unknown names earn 0, `FruitEconomy.lua:56-57`), and is routed to fruit pick-up or merge by the prompt handler (`GardenService.lua:522-526`). Use a separate folder and a separate factory modelled on `randomSpotIn` and `placeFruit`.
- **Ores Index category.** `discover` drops any name not in `FruitIndex` (`Progression.lua:174`), and so do save and load (`Persistence.lua:70`, `:230`). Ores need their own folder and save field.
- **Vertical checks.** `isOnPlot` is X/Z only (`FruitFactory.lua:63-68`), so a player underground still counts as on the plot for drops (`Carrying.lua:325`). `standingOn` adds the 12-stud band (`FruitFactory.lua:79-85`), so TNT and lock will not see players deeper than that.
- **Risk.** An error inside `claimPlot` aborts `watchPlayer` before `Store.Load` (`PlotRegistry.lua:87-90`, `GardenService.lua:688-697`). Do not build fragile geometry there.

### 5. Forest pets
- **Earnings.** Income today is `plotBase x cashMultiplier` (`Ambient.lua:465-466`). Add a separate pet term inside, or beside, `plotBase` (`FruitFactory.lua:39-45`). That way live income and offline income (`Persistence.lua:191`) stay the same formula.
- **Stage.** The "stage" is the player's top unlocked rarity. Read it from the `Rebirths` attribute or the spawn tier level (`levelOf`, `Upgrades.lua:34-37`). `bestRank` (`GardenService.lua:122`) gives the best fruit merged this run, for "the requested fruit".
- **Wandering.** Use `isOnPlot` and the soil size (`FruitFactory.lua:63-68`, `:96-107`) for targets. Keep pets out of `SpawnedFruits` for the same reasons as ores.
- **Risk.** Pet income must respect the friend-bonus rule. Offline pay excludes friends on purpose (`Persistence.lua:173-175`, `:191`).

### 6. Boats, swimming fatigue, Aqua Affinity
- **Speed.** `applyWalkSpeed` (`GardenService.lua:191-248`) is the one function that sets WalkSpeed and JumpPower. Swim fatigue and Aqua Affinity must be folded into it, after the arena override (`:200-208`), or they will be overwritten.
- **Trigger.** It only re-runs on events: drinking, expiry (`GardenService.lua:808-813`), respawn (`:592`) and the Ambient sweep. Entering or leaving water needs its own call.
- **Aqua Affinity.** A `Config.Effects` entry works with the existing expiry sweep (`GardenService.lua:793-821`) and death clearing (`:554-570`) with no extra code.
- **Boats.** Shard purchase, seats and breaking are outside this subsystem.

### 7. More potions
- **Pattern.** A timed potion is `effects[player][name] = endsAt` plus `<name>EndsAt` and `<name>Span` attributes (`Tools.lua:210-223`). This subsystem expires them (`GardenService.lua:793-821`) and clears them on death (`:554-570`). Effects are never saved (`GardenService.lua:177-180`).
- **Hooks by kind.** Movement: `applyWalkSpeed` (`GardenService.lua:191-248`). Visibility: `applyVisibility` (`:258-387`). Spawn luck or rate: `spawnPlan` (`Upgrades.lua:372-379`) or the interval at `Ambient.lua:422-425`. Income: `cashMultiplier` (`Progression.lua:332-336`).
- **Risk.** A cash potion in `cashMultiplier` is also shown on the HUD via `CashMultiplier` (`Progression.lua:338-343`). It must not reach offline pay, which builds its own formula (`Persistence.lua:191`).

### 8. Plot visual overhaul
- **Soil detection.** The soil is the BasePart with the largest X times Z area in the plot model (`PlotRegistry.lua:21-30`). Any new decoration larger than the soil becomes the "soil". Fruit would then spawn on it and the resize would stretch it.
- **Fence rebuild deletes things.** `buildFence` destroys every child named `Fence` or `FenceFront`, and every BasePart named `Part` that is not the soil (`Upgrades.lua:168-173`). New decorations left with the default name `Part` vanish on the first resize.
- **Fence look.** Rebuilt fences clone the majority Post, Rail and Kerb (`Upgrades.lua:84-112`). Restyle the authored fences in Studio and rebuilt ones follow. Shape numbers are hard-coded (`Upgrades.lua:61-64`); a new fence shape must update them.
- **Plaza direction.** Spawn pad and fence frame both assume the plaza is at world origin (`PlotRegistry.lua:115`, `Upgrades.lua:143-145`).
- **Size ladder.** `PlotSizes` was measured to fit inside the plaza band (`GardenConfig.lua:98-116`). New plot art must respect the level 4 footprint.
- **Logs, lily pads, mountains.** Not in this subsystem.

### 9. UI overhaul
- **Fonts sent from the server.** The fruit name tag uses `Config.NameTagFont`, which is `Config.FruitTextFont` = `Enum.Font.Highway` (`GardenConfig.lua:25`, `:595`, `FruitFactory.lua:279`). Switching to FredokaOne is a Config change.
- **Server text.** Toasts are plain strings sent through `Notify` and `Notice` (`GardenService.lua:66-70`). In this subsystem every toast string is a Config key, for example the king strings at `Progression.lua:75`, `:266`, `:286`. Wording changes belong in Config, not in code.
- **Owner headshot.** Built in code, not from an asset (`PlotRegistry.lua:38-92`).
- **King shop.** The server endpoints are `KingBuy` and `KingEquip` (`Progression.lua:245-288`). A new King UI only needs to fire the same keys.

### 10. Royal combat
- **Arena movement.** Fighters get arena speed and jump, overriding potions (`GardenService.lua:196-208`). Fighters are never invisible (`:266`). New combat effects that change movement should go through `G.arenaStats`, not a direct Humanoid write.
- **The rest** (swords, slashes, duel health, Royal Slam, skins) lives in the Crown modules, outside this subsystem.

## Gotchas

1. **Save before destroy.** `teardownPlayer` must capture the profile before `releaseHold` and `releasePlot`, which destroy the fruit (`GardenService.lua:704-715`). A line added above the save stops fruit saving.
2. **One parameter only.** `onPlayerLeaving` must take exactly one argument. `PlayerRemoving` passes an exit reason that would land in `skipSave` and skip the save (`GardenService.lua:737-744`). Ambient asserts this at startup (`Ambient.lua:397-400`).
3. **`claimPlot` errors block loading.** A throw in `claimPlot` aborts `watchPlayer` before `Store.Load` (`PlotRegistry.lua:87-90`).
4. **Module order.** A module only sees what earlier modules put in `G`. Late-bound reads like `G.arenaStats` (`GardenService.lua:200`), `G.crownRefresh` (`Progression.lua:54`) and `G.spawnBoost` (`Ambient.lua:421`) work because they are looked up at call time.
5. **`publishMultiplier` is a module global.** Progression defines it with `function publishMultiplier` and no `local` (`Progression.lua:341`). The main script's `local publishMultiplier` (`GardenService.lua:126`) is a different variable, filled from `G` at `:407`.
6. **Three counts, three meanings.** Spawn uses `plotLoad` = soil + hands + pending merge (`Upgrades.lua:286-288`). Drop uses only fruit on the soil (`Carrying.lua:331-333`). Income uses only fruit on the soil (`FruitFactory.lua:39-45`).
7. **The merge ledger.** `pendingMerge` covers the moment a merge has eaten fruit but not yet made the new one (`GardenService.lua:134-169`). It must be cleared on release and rebirth (`GardenService.lua:475`, `RebirthLock.lua:76-79`) or fruit duplicate.
8. **Release resets size.** `releasePlot` shrinks the plot to level 0 (`GardenService.lua:456`). The next owner's size is applied only when their profile lands (`Persistence.lua:276-278`).
9. **Claim always clears.** `claimPlot` destroys any fruit on the plot, even if `seed` is false (`GardenService.lua:440`).
10. **Startup wipes the fruit folder.** Anything saved inside `SpawnedFruits` in Studio is destroyed on server start (`GardenService.lua:37-38`).
11. **`applyPlotSize` skips same level.** It returns early if `plot.sizeLevel` already matches (`Upgrades.lua:244`). Editing the soil by hand will not trigger a rebuild.
12. **Stale comment.** `SPAWN_TICK` says the fastest rung is 4 s (`Upgrades.lua:290-291`). The real floor is 2.5 s (`GardenConfig.lua:61`), and Fruit Frenzy divides it further (`Ambient.lua:423`). 0.5 s is still well below both.
13. **Spawn search cost.** `trySpawnIn` calls `randomSpotIn` without a pre-built list (`Upgrades.lua:412`). Each of up to 40 tries walks the whole server's fruit folder (`FruitFactory.lua:51-53`, `:26-34`).
14. **Buying a tier fills the Index.** `buyUpgrade` with `spawnTier` discovers every fruit up to the new tier (`Upgrades.lua:327-335`), and load back-fills the same way (`Persistence.lua:280-290`). This matters for any rarity-gated rebirth rule.
15. **Friends do not pay offline.** Live income uses friends (`Progression.lua:333`). Offline does not (`Persistence.lua:191`). Rebirth and Index bonuses pay in both.
16. **Rebirth resets every upgrade.** `publishUpgrades(player, nil)` zeroes every key in `UpgradeKeys` (`Upgrades.lua:294-299`, `RebirthLock.lua:95`). Any new upgrade added there is wiped on rebirth too.
17. **The crown runs every income tick.** `publishCash` calls `refreshKing` (`Progression.lua:87`), and the income tick calls `publishCash` for every player every 5 s (`Ambient.lua:470`).
18. **Friend counts are slow.** `refreshFriendCounts` calls `IsFriendsWith` for every pair of players on every join and leave (`Progression.lua:105-127`). It checks `player.Parent` before writing so a leaver is not pinned in memory (`:117-124`).
19. **Studio dev doors are Studio-only.** `DevSetCash`, `DevSetShards` and `DevSetPlotSize` are gated by `RunService:IsStudio()` (`GardenService.lua:631`). Cash cannot be set from outside the main script otherwise, because `balances` is a local table (`GardenService.lua:622-626`).
