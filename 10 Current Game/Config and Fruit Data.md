---
tags: [current-game, code-map]
date: 2026-10-04
sources:
  - ReplicatedStorage.GardenConfig.lua
  - ReplicatedStorage.FruitEconomy.lua
  - ReplicatedStorage.Fruits.FruitIndex.lua
  - ReplicatedStorage.IndexRewards.lua
  - ReplicatedStorage.GardenToast.lua
  - ReplicatedFirst.LoadingScreen.lua
---

# Config and Fruit Data

Code export: `/Users/samarthvaka/fruit-empire/src-export-2026-10-04`. The plan these notes serve: [[Brain Dump 2026-10-04]].

The six sources above were read in full. A few citations point at scripts outside this subsystem, to show where its data is used. Those were checked with grep and short reads only, not read in full.

## What it does

This subsystem is the game's shared data layer. It holds almost no live state; most other scripts read from it. FruitIndex is the list of all 64 fruit, and GardenConfig holds every tuning number, text string and image id, and builds the upgrade ladders from that list when it loads. FruitEconomy turns a rank into income per second and formats money, and IndexRewards turns a rank into Index rewards. GardenToast draws the red warning line and the green good-news line, and LoadingScreen covers the join while it preloads art and waits for the save to load. The terms used here are defined just below.

### Terms used in this note

- **ModuleScript**: a Roblox script that returns a table. Other scripts load it with `require`. All four ReplicatedStorage files here are ModuleScripts, so the server and every client load the same copy.
- **Rank**: a fruit's place on the merge ladder. Rank 1 is the best fruit (Soul Fruit). Rank 64 is the worst (Blueberry) (`ReplicatedStorage.Fruits.FruitIndex.lua:10`, `ReplicatedStorage.Fruits.FruitIndex.lua:73`).
- **Merge ladder**: the chain of fruit. Two fruit of rank R merge into one fruit of rank R-1 (`ReplicatedStorage.Fruits.FruitIndex.lua:5`).
- **Tier / tierName**: a rarity band of fruit. There are 9, from tier 1 COMMON to tier 9 TRANSCENDENT (`ReplicatedStorage.Fruits.FruitIndex.lua:10-73`).
- **Level**: how many times a player has bought one upgrade. Levels are stored from 0 and shown from 1 (`ReplicatedStorage.GardenConfig.lua:46-48`).
- **Spawn Rarity**: the upgrade that decides which fruit a plot grows. The code and saves still call it `spawnTier` (`ReplicatedStorage.GardenConfig.lua:68-69`).
- **Index**: the in-game catalogue of all 64 fruit. A fruit is "discovered" the first time you merge into it, or pick up one your plot can spawn (`ReplicatedStorage.GardenConfig.lua:382-384`).
- **Shards**: the second currency. Index claims pay them (`ReplicatedStorage.IndexRewards.lua:4-5`, `ReplicatedStorage.IndexRewards.lua:76-85`). Tools and king effects cost them (`ReplicatedStorage.GardenConfig.lua:1595`, `ReplicatedStorage.GardenConfig.lua:1225-1226`).
- **Rebirth**: trading a whole run for a permanent cash multiplier (`ReplicatedStorage.GardenConfig.lua:989-992`).
- **Attribute**: a named value set on a Roblox instance with `SetAttribute`. It copies from server to clients on its own.
- **IntValue**: a Roblox instance that holds one whole number. Folders of them are used where an attribute cannot hold a table (`ReplicatedStorage.GardenConfig.lua:525-527`).
- **Remote**: a RemoteEvent, the channel scripts use to send messages between server and client.
- **Toast**: the one-line message that slides onto the screen (`ReplicatedStorage.GardenToast.lua:2`).
- **Preload**: asking Roblox to download art before it is shown, so it does not pop in later (`ReplicatedFirst.LoadingScreen.lua:11-13`).

## Scripts

| Script | Role | Lines |
|---|---|---|
| ReplicatedStorage.GardenConfig.lua | Every tunable, string and image id. Builds the four upgrade ladders and their prices at load. Shared helper functions. | 2117 |
| ReplicatedStorage.FruitEconomy.lua | Income per rank, derived from the ladder. Money formatting ("1.2K"). | 86 |
| ReplicatedStorage.Fruits.FruitIndex.lua | The 64-fruit ladder: rank, name, tier, size. Lookups and `MergeInto`. Auto-generated. | 99 |
| ReplicatedStorage.IndexRewards.lua | Index rewards per rank: coin bonus, cash, shards, flavour rarity. | 100 |
| ReplicatedStorage.GardenToast.lua | Client-only toast lines: `Warn` (red, from top) and `Notice` (green, from bottom). | 170 |
| ReplicatedFirst.LoadingScreen.lua | Join splash. Preloads art, waits for the profile, pre-draws menus, then fades out. | 450 |

## How it works

### 1. Load order

1. FruitIndex requires nothing. GardenConfig depends on that, so there is no require cycle (`ReplicatedStorage.GardenConfig.lua:10-14`).
2. GardenConfig requires only FruitIndex (`ReplicatedStorage.GardenConfig.lua:14`).
3. FruitEconomy requires FruitIndex (`ReplicatedStorage.FruitEconomy.lua:21`).
4. IndexRewards requires GardenConfig, FruitEconomy and FruitIndex (`ReplicatedStorage.IndexRewards.lua:13-15`).
5. GardenToast requires GardenConfig. Only client scripts may require it, because it reaches into PlayerGui (`ReplicatedStorage.GardenToast.lua:17`, `ReplicatedStorage.GardenToast.lua:31`).
6. LoadingScreen lives in ReplicatedFirst, so it runs before any other client script (`ReplicatedFirst.LoadingScreen.lua:4-6`). It requires GardenConfig inside a `pcall`, with a 20 second wait (`ReplicatedFirst.LoadingScreen.lua:328-330`).

### 2. FruitIndex: the ladder

- One list of 64 entries. Each has `rank`, `name`, `display`, `tier`, `tierName`, `diameter` (`ReplicatedStorage.Fruits.FruitIndex.lua:9-74`).
- It is generated from MERGE_ORDER.txt by an importer. The header says do not hand-edit it (`ReplicatedStorage.Fruits.FruitIndex.lua:2-3`).
- It returns `List`, `ByName`, `Count`, `Get`, `Template` and `MergeInto` (`ReplicatedStorage.Fruits.FruitIndex.lua:89-99`).
- `MergeInto(name)` returns the entry one rank better, or nil at rank 1. It has no cap of any kind (`ReplicatedStorage.Fruits.FruitIndex.lua:95-98`).
- `Template(name)` finds the fruit's MeshPart under any Folder inside ReplicatedStorage.Fruits (`ReplicatedStorage.Fruits.FruitIndex.lua:79-87`).
- Fruit size runs from 1.4 studs (Blueberry) to 6.0 studs (Soul Fruit) (`ReplicatedStorage.Fruits.FruitIndex.lua:73`, `ReplicatedStorage.Fruits.FruitIndex.lua:10`).

The nine rarity tiers, counted from the list:

| Tier | tierName | Ranks | Fruit | Best fruit in tier |
|---|---|---|---|---|
| 1 | COMMON | 60-64 | 5 | Apple (`ReplicatedStorage.Fruits.FruitIndex.lua:69`) |
| 2 | UNCOMMON | 52-59 | 8 | Blue Lollipop (`ReplicatedStorage.Fruits.FruitIndex.lua:61`) |
| 3 | RARE | 41-51 | 11 | Pear (`ReplicatedStorage.Fruits.FruitIndex.lua:50`) |
| 4 | EPIC | 33-40 | 8 | Watermelon (`ReplicatedStorage.Fruits.FruitIndex.lua:42`) |
| 5 | LEGENDARY | 26-32 | 7 | Tropical Melon (`ReplicatedStorage.Fruits.FruitIndex.lua:35`) |
| 6 | MYTHIC | 18-25 | 8 | Dragon Pepper (`ReplicatedStorage.Fruits.FruitIndex.lua:27`) |
| 7 | ASTRAL | 13-17 | 5 | Paradise Petal (`ReplicatedStorage.Fruits.FruitIndex.lua:22`) |
| 8 | ETERNAL | 5-12 | 8 | Solar Bloom (`ReplicatedStorage.Fruits.FruitIndex.lua:14`) |
| 9 | TRANSCENDENT | 1-4 | 4 | Soul Fruit (`ReplicatedStorage.Fruits.FruitIndex.lua:10`) |

### 3. GardenConfig: tables built when it loads

These run once, at require time, in this order:

1. `Capacities[level] = 6 + level`, for levels 0 to 29 (`ReplicatedStorage.GardenConfig.lua:50-56`).
2. `SpawnRates[level] = 10 - 0.5 * level`, for levels 0 to 15 (`ReplicatedStorage.GardenConfig.lua:58-66`).
3. `SpawnTiers[level]` is a pool of the bottom `level + 1` fruit. Each fruit's weight is `0.6 ^ (steps below the best)` (`ReplicatedStorage.GardenConfig.lua:85-96`). Level L's best fruit is rank `64 - L` (`ReplicatedStorage.GardenConfig.lua:83-84`, `ReplicatedStorage.GardenConfig.lua:90`). The pool is listed worst first, best last.
4. `PlotSizes` is written by hand, levels 0 to 4 (`ReplicatedStorage.GardenConfig.lua:117-123`).
5. `UpgradePrices[key][n] = roundPrice(base * growth ^ (n - 1))`. The number of steps is `#` of each 0-based ladder (`ReplicatedStorage.GardenConfig.lua:159-187`, `ReplicatedStorage.GardenConfig.lua:169-175`). `roundPrice` keeps 3 significant figures (`ReplicatedStorage.GardenConfig.lua:161-165`).
6. `KingEffectByKey` indexes the king effects by key (`ReplicatedStorage.GardenConfig.lua:1243-1244`). `ShopTabs` is picked once from `KingEnabled` (`ReplicatedStorage.GardenConfig.lua:1210-1212`).

The accessor functions at the bottom all take a level and clamp it, so a bad level from a corrupt save becomes the nearest real level instead of an error (`ReplicatedStorage.GardenConfig.lua:1895-1903`):

| Function | Returns | Line |
|---|---|---|
| `Config.MaxLevel(key)` | number of price steps, or 0 for a key with no prices | `ReplicatedStorage.GardenConfig.lua:1905-1908` |
| `Config.SpawnTable(level)` | the weighted spawn pool | `ReplicatedStorage.GardenConfig.lua:1910-1912` |
| `Config.SpawnInterval(level)` | seconds between spawns | `ReplicatedStorage.GardenConfig.lua:1914-1916` |
| `Config.Capacity(level)` | fruit the plot holds | `ReplicatedStorage.GardenConfig.lua:1918-1920` |
| `Config.PlotSize(level)` | soil size as Vector2 | `ReplicatedStorage.GardenConfig.lua:1922-1924` |
| `Config.UpgradePrice(key, level)` | next price, or nil when maxed. nil is the one "maxed" signal for server and client | `ReplicatedStorage.GardenConfig.lua:1926-1933` |
| `Config.UpgradeLevelText(key, level)` | "Level n/max", 1-based | `ReplicatedStorage.GardenConfig.lua:1996-1998` |
| `Config.UpgradeStat(key, level)` | card's second line, e.g. "Up to Apple" | `ReplicatedStorage.GardenConfig.lua:2001-2021` |

Other shared helpers:

| Function | Job | Line |
|---|---|---|
| `Config.StoredTotal(folder)` | counts stored fruit; only names in FruitIndex count | `ReplicatedStorage.GardenConfig.lua:539-549` |
| `Config.ClockTimeAt(serverTime)` | time of day from the shared server clock | `ReplicatedStorage.GardenConfig.lua:1485-1498` |
| `Config.DurationText(seconds)` | "20s", "5 min", "5 min 30s" for potions | `ReplicatedStorage.GardenConfig.lua:1805-1811` |
| `Config.Duration(seconds)` | "2h 14m" for the offline popup | `ReplicatedStorage.GardenConfig.lua:1969-1976` |
| `Config.Plural(name)` | "Blueberries", "Tomatoes" | `ReplicatedStorage.GardenConfig.lua:1978-1994` |
| `Config.HudScaleFor(viewport, touch)` | HUD scale factor, or nil while the screen is not real yet | `ReplicatedStorage.GardenConfig.lua:1949-1957` |

### 4. FruitEconomy: income per rank

1. Rank 64 earns `Base` = 1 per second (`ReplicatedStorage.FruitEconomy.lua:25`, `ReplicatedStorage.FruitEconomy.lua:41`).
2. Each rank up multiplies by a step. The step slides from 2.10 at the bottom to 2.60 at the top (`ReplicatedStorage.FruitEconomy.lua:26-27`, `ReplicatedStorage.FruitEconomy.lua:42-46`).
3. Each value is rounded to 3 significant figures before the next one is built from it (`ReplicatedStorage.FruitEconomy.lua:33-37`, `ReplicatedStorage.FruitEconomy.lua:45`).
4. `ForRank(rank)` and `ForName(name)` look the result up, or return 0 (`ReplicatedStorage.FruitEconomy.lua:52-58`).
5. `Format(value)` writes money with suffixes K, M, B, T, Qa, Qi, Sx, Sp, Oc, No, Dc (`ReplicatedStorage.FruitEconomy.lua:62-84`).

The design rule: every step must be worth more than 2x. Otherwise holding two fruit would beat merging them (`ReplicatedStorage.FruitEconomy.lua:11-13`).

### 5. IndexRewards: what an Index entry pays

1. `RarityDenominator(rank) = 2 ^ (65 - rank)`. It is flavour text only; nothing rolls against it (`ReplicatedStorage.IndexRewards.lua:23-28`). `RarityText` prints it as "1/2" up to "1/18.4Qi" (`ReplicatedStorage.IndexRewards.lua:30-34`).
2. `Bonus(rank)`: each fruit gets weight `2 ^ (tier - 1)`. Weights are divided by the total of the whole ladder, so all 64 together equal `IndexBonusTotal` (`ReplicatedStorage.IndexRewards.lua:38-56`).
3. `Money(rank)` = that fruit's income times `IndexMoneySeconds` (`ReplicatedStorage.IndexRewards.lua:67-71`).
4. `Shards(rank)` climbs geometrically from `IndexShardsLow` to `IndexShardsHigh`, then rounds to 25s, 5s or 1s (`ReplicatedStorage.IndexRewards.lua:76-85`). Shards do not follow income on purpose (`ReplicatedStorage.IndexRewards.lua:73-75`).
5. `For(rank)` bundles all of these. The server pays from it on claim and the client prints the card from it, so the two cannot disagree (`ReplicatedStorage.IndexRewards.lua:4-5`, `ReplicatedStorage.IndexRewards.lua:88-98`).

### 6. GardenToast: the message lines

1. Server code fires the `Notify` remote for a warning or `Notice` for good news (`ServerScriptService.GardenService.lua:66`, `ServerScriptService.GardenService.lua:70`).
2. GardenClient forwards those to `Toast.Warn` and `Toast.Notice` (`StarterPlayer.StarterPlayerScripts.GardenClient.lua:415-426`). Some client scripts call `Toast.Warn` directly, for example `StarterPlayer.StarterPlayerScripts.LockButton.lua:38`.
3. `show()` builds the label the first time, under PlayerGui.GardenUI (`ReplicatedStorage.GardenToast.lua:83-119`, `ReplicatedStorage.GardenToast.lua:126`).
4. It bumps that channel's generation counter and cancels the channel's old tweens (`ReplicatedStorage.GardenToast.lua:128-130`).
5. It slides the label in with a Back/Out ease and fades the text and outline in (`ReplicatedStorage.GardenToast.lua:141-143`).
6. After `ToastDropTime + ToastHold`, it fades out in place, but only if no newer message has taken the label (`ReplicatedStorage.GardenToast.lua:145-157`).

The two channels: WARN is red, drops from the top and lands 76 px down. NOTICE is green, rises from the bottom and lands 150 px up (`ReplicatedStorage.GardenToast.lua:49-67`).

### 7. LoadingScreen: the join splash

1. Removes Roblox's own loading screen at once (`ReplicatedFirst.LoadingScreen.lua:35`).
2. Builds its GUI in code, because StarterGui has not arrived yet when ReplicatedFirst runs (`ReplicatedFirst.LoadingScreen.lua:22-24`). DisplayOrder 1000 puts it over GardenUI (`ReplicatedFirst.LoadingScreen.lua:53-59`). A 24-size blur goes on Lighting (`ReplicatedFirst.LoadingScreen.lua:77-82`).
3. Wobbles the logo and stage every frame with sine waves (`ReplicatedFirst.LoadingScreen.lua:252-264`).
4. `gather()` collects what to preload (`ReplicatedFirst.LoadingScreen.lua:279-349`):
   - every MeshPart, Decal, Texture, ImageLabel and ImageButton in ReplicatedStorage (`ReplicatedFirst.LoadingScreen.lua:298-309`);
   - every image in PlayerGui.GardenUI (`ReplicatedFirst.LoadingScreen.lua:313-318`);
   - every string in Config, at any depth, that starts with `rbxassetid://` and a digit (`ReplicatedFirst.LoadingScreen.lua:328-345`);
   - the logo (`ReplicatedFirst.LoadingScreen.lua:347`).
5. Preloads with a per-asset callback. The preload fills the bar from 0 to 75% (`ReplicatedFirst.LoadingScreen.lua:354-377`). A failed preload is skipped, not fatal (`ReplicatedFirst.LoadingScreen.lua:371-374`).
6. At the same time, waits for the `ProfileLoaded` player attribute, for at most 45 seconds (`ReplicatedFirst.LoadingScreen.lua:381-392`). GardenStore sets that attribute (`ServerScriptService.GardenStore.lua:800`).
7. When both are done, `warmMenus()` shows each hidden GardenUI child whose name ends in "Menu" for two frames, one at a time, so its art is drawn once (`ReplicatedFirst.LoadingScreen.lua:399-422`, `ReplicatedFirst.LoadingScreen.lua:424-426`).
8. Waits until at least 1.6 seconds have passed, shows "Ready!", fades everything over 0.5 seconds and destroys itself (`ReplicatedFirst.LoadingScreen.lua:397`, `ReplicatedFirst.LoadingScreen.lua:427-449`).

## Data

### Saved fields

This subsystem writes no save data itself. It supplies the key lists and name sets that the save code checks against.

| Save field | What this subsystem supplies | Where it is used |
|---|---|---|
| `upgrades.<key>` | the key list `Config.UpgradeKeys`; levels clamped to `Config.MaxLevel` | `ReplicatedStorage.GardenConfig.lua:37`, `ServerScriptService.GardenService.Persistence.lua:92-93`, `ServerScriptService.GardenStore.lua:476-483` |
| `stored.<fruit name>` | valid names = `FruitIndex.ByName` | `ServerScriptService.GardenStore.lua:289` |
| `index.<fruit name>` (0 or 1) | valid names = `FruitIndex.ByName` | `ServerScriptService.GardenStore.lua:345` |
| `tools.<tool name>` | valid names = keys of `Config.Tools` | `ServerScriptService.GardenService.Persistence.lua:64`, `ServerScriptService.GardenStore.lua:361` |
| `shards` (tester floor) | `Config.TesterShards`, by user id | `ReplicatedStorage.GardenConfig.lua:410-413`, `ServerScriptService.GardenService.Persistence.lua:263` |

### Replicated folders (named here, live on the Player)

| Config key | Folder name | Contents | Line |
|---|---|---|---|
| `IndexFolder` | IndexState | one IntValue per discovered fruit: 0 = rewards waiting, 1 = claimed, missing = locked | `ReplicatedStorage.GardenConfig.lua:386-390` |
| `StoredFolder` | StoredFruits | IntValue count per fruit name | `ReplicatedStorage.GardenConfig.lua:525-528` |
| `ToolsFolder` | StoredTools | stored tools | `ReplicatedStorage.GardenConfig.lua:1604` |

### Attributes this subsystem reads or documents

| Attribute | On | Meaning | Line |
|---|---|---|---|
| `ProfileLoaded` | Player | true once the save is applied. LoadingScreen reads it | `ReplicatedFirst.LoadingScreen.lua:388` |
| `Upgrade_<key>` | Player | upgrade level. Config says levels replicate as Player attributes; the name comes from Upgrades | `ReplicatedStorage.GardenConfig.lua:29-31`, `ServerScriptService.GardenService.Upgrades.lua:30-31` |
| `<Effect>EndsAt` | Player | potion end time, absolute, in server time | `ReplicatedStorage.GardenConfig.lua:1709-1713` |
| `MusicMuted` | Player | mute preference | `ReplicatedStorage.GardenConfig.lua:813-814` |
| `VisionSky` | not stated here | drives the sky music crossfade | `ReplicatedStorage.GardenConfig.lua:823-824` |
| `MergeAllReadyAt` | not stated here | Merge All cooldown deadline, server time | `ReplicatedStorage.GardenConfig.lua:967-969` |
| `KingUserId` | workspace | who holds the crown | `ReplicatedStorage.GardenConfig.lua:1221-1222` |
| `KingEffect` | Player | equipped king effect | `ReplicatedStorage.GardenConfig.lua:1221-1222` |
| `KingOwned` | Player | owned king effects, a comma list of keys | `ReplicatedStorage.GardenConfig.lua:1198-1199`, `ServerScriptService.GardenService.Progression.lua:222` |
| `LockedUntil` | plot Model | shield end time; the server never trusts it | `ReplicatedStorage.GardenConfig.lua:911-913` |
| `Lucky` | fruit | grown by the Luck potion; lets it into the Index on pick-up | `ReplicatedStorage.GardenConfig.lua:1778-1780` |
| `PlotName` | fruit | the fruit's plot tag; tools never carry it | `ReplicatedStorage.GardenConfig.lua:282`, `ReplicatedStorage.GardenConfig.lua:1596-1597` |

### Remotes

This subsystem defines no remotes. It references four:

| Remote | Direction | Payload | Line |
|---|---|---|---|
| `Notify` | server to client | one warning string | `ReplicatedStorage.GardenToast.lua:17-20`, `ServerScriptService.GardenService.lua:66` |
| `Notice` | server to client | one good-news string | `ServerScriptService.GardenService.lua:70`, `StarterPlayer.StarterPlayerScripts.GardenClient.lua:423-426` |
| `Upgrade` | client to server | one upgrade key | `ReplicatedStorage.GardenConfig.lua:2086-2088`, `ServerScriptService.GardenService.lua:77` |
| `Drop` | client to server | not stated in this subsystem | `ReplicatedStorage.GardenConfig.lua:1613-1614`, `ServerScriptService.GardenService.lua:59` |

### Every GardenConfig key

GardenConfig has 434 top-level keys and functions. Grouped by job. Line numbers are where each group sits. Some keys sit far from their section header; see Gotchas.

| Area | Keys | Lines |
|---|---|---|
| Fonts | FruitTextFont | 25 |
| Upgrade ladders | UpgradeKeys, UpgradePanelKeys, CapacityStart, CapacityTop, Capacities, SpawnRateStart, SpawnRateFloor, SpawnRateStep, SpawnRates, SpawnRarityFalloff, SpawnTiers, PlotSizes, UpgradePriceBase, UpgradePriceGrowth, UpgradePrices, UpgradeCurrency | 37-190 |
| Upgrade cards UI | UpgradeCards, UpgradeCardThemes, UpgradeButtonImage, UpgradeMaxedButton, UpgradeCardAspect, UpgradeRows, UpgradeButtonRect, UpgradeMaxedHeight, UpgradeFont, UpgradeTextStroke, UpgradeLevelColor, UpgradeStatColor, UpgradePriceColor, UpgradeUnaffordable, UpgradeMaxedText, UpgradeTooPoor | 197-274 |
| Plot | IslandName, PlotsFolder, FruitFolder, MaxFruitsPerPlot, InitialFruits, SpawnMargin, MinFruitGap, OwnerIconStuds, OwnerIconHeight, OwnerIconStroke, OwnerIconMaxDistance | 280-300 |
| Carrying | PromptDistance, MaxReach, CarryGap, DropForwardStuds, DropLift, DropRiseTime, DropFallTime, DropHop, DropHopTime, PlotStandMargin, OffPlotWarning, PlotFullWarning, CarryAnimationId, CarryPoseName, CarryFadeTime | 304-351 |
| Economy | FriendBonus, CashMultiplierText, IncomeTick, CashCatchUp | 357-378 |
| Index | IndexFolder, IndexBonusTotal, IndexMoneySeconds, IndexShardsLow, IndexShardsHigh, TesterShards, IndexCanvas, IndexTextStroke, IndexSelectRing, IndexSelectRingScale, IndexSelectFrom, IndexSelectTime | 390-435 |
| Index art | TranscendentAccent, TranscendentAccentScale, TranscendentAccentOffset, TranscendentAccentTransparency, TranscendentShimmer, TranscendentShimmerSeconds, IndexLockedColor, IndexLockedFade, IndexLockedMaskColor, IndexLockedMaskTransparency, IndexLockedText, IndexLockedMaskWiden, IndexClaimedTint | 442-521 |
| Storage | StoredFolder, StoredCap, StoredFull, StoredCountText, StoredTotal() | 528-549 |
| Rarity look | RarityFrames, CardAspect, RarityColors, RarityGradients | 563-641 |
| Name tags | NameTagFont, NameTagStuds, NameTagHeight, NameTagStroke, NameTagMaxTextSize, NameTagLift, NameTagMaxDistance, NameTagForward, NameTagSweepTiers, NameTagSweepSeconds | 595-652 |
| Auras and sparkle | FruitAuras, DiscoverTemplate, DiscoverScaleMin, DiscoverScaleMax, DiscoverHold, DiscoverFade, PromptHighlightColor | 664-709 |
| Merging | MergeSpinTime, MergeSettleTime, MergeSpins, MergeSettleSpins, MergeDip, MergePunch, MergeFlashColor, MergeSoundId, MergeSoundVolume, MergeSoundRange, MergeSoundMaxLife | 720-746 |
| Offline earnings | OfflineRate, OfflineCapSeconds, OfflineMinimum, OfflineMinimumAway, OfflineAmountText | 753-784 |
| Music | MusicTracks, MusicVolume, MusicTrackGain, MusicOnIcon, MusicOffIcon, MusicGap, SkyMusicId, SkyMusicVolume, SkyMusicEase, ForestMusicId, ForestMusicVolume, ForestMusicEase, RoyalMusicId, RoyalMusicVolume, RoyalMusicEase, FightMusicId, FightMusicVolume, FightMusicEase, FightMusicRange, MusicLoadTimeout, MusicDuckFloor, MusicDuckCurve, MusicPitchFloor, MusicMuffleHigh, MusicMuffleMid, MusicDuckEase | 798-879 |
| Lockdown | LockDuration, LockCooldown, LockTemplate, LockCooldownWarning, LockNoPlot, LockOffPlot, LockAlreadyUp, LockTNTBlocked, LockTimerText, LockTimerStroke, LockTimerColor, LockCooldownColor, PlotStandBand, WalkSpeed | 885-943 |
| Merge All | MergeAllGamepassId, MergeAllButton, MergeAllCooldown, MergeAllWait, MergeAllNothing, MergeAllDone, MergeAllUnowned, MergeAllNoPass, MergeAllPuffs | 961-985 |
| Rebirth | RebirthNeedsAll, RebirthCashBonus, RebirthButton, RebirthPanel, RebirthPanelAspect, RebirthAction, RebirthActionAspect, RebirthLocked, RebirthLockedAspect, RebirthClose, RebirthBoxCash, RebirthBoxShards, RebirthAmountY, RebirthCountY, RebirthActionY, RebirthCountText, RebirthCashText, RebirthShardsText, RebirthConfirmBody, RebirthNotReady, RebirthDone, RebirthNoPlot, RebirthBusy | 993-1039 |
| Clearing | ClearButton, ClearCooldown, ClearTemplate, ClearPuffs, ClearSmokeLife, ClearConfirmTitle, ClearConfirmBody, ClearStrokeCanvas, ClearEdgeStroke, ClearTextStroke, ClearOnlyKind, ClearEmptyPlot, ClearWait, ClearedToast | 1051-1087 |
| Earn popups | EarnPopupFont, EarnPopupColor, EarnPopupStroke, EarnPopupStuds, EarnPopupHeight, EarnPopupMaxTextSize, EarnPopupRise, EarnPopupTime, EarnPopupMaxDistance, EarnPopupStagger | 1093-1105 |
| General messages | EquipWarning, LoadingWarning, LoadFailedKick, LockedKick, NoPlotWarning, NotDurableWarning | 1110-1124 |
| Saving | SaveInterval, SaveTouch, SaveRetryCount, SaveRetryBase, ShutdownRetryCount, ShutdownDeadline, LockStaleSeconds, LockWaitDelay, LockWaitAttempts, RefundNotice, RefundNoticeDelay | 1131-1152 |
| Toasts | ToastColor, NoticeColor, ToastDropTime, ToastHold, ToastFadeTime | 1159-1167 |
| Shop | ShopButton, ShopPanel, ShopTab, ShopPanelLeftEdge, KingEnabled, ShopTabs, ShopTabDim | 1185-1216 |
| Crown | KingAssets, KingDefault, KingEffects, KingEffectByKey, KingCrownWidth, KingHoverHeight, KingBob, KingBobSpeed, KingSpinSpeed, KingIcons | 1234-1259 |
| Crown messages | KingYouAre, KingIs, KingBought, KingEquipped, KingNotOwned, KingAlreadyOwned | 1586-1591 |
| Sea and currents | DrownSweptSpeed, CurrentSpeed, CurrentAirAccel, ReturnSpeed, ReturnFloors, ReturnAirAccel, DrownGroundProbe, DrownWaterFloors | 1291-1327 |
| Drowning view | HeartbeatSound, HeartbeatVolume, HeartbeatFloor, HeartbeatSpeedUp, DrownFadeStart, DrownFadeFull, DrownKill, DrownTick, DrownCurve, DrownOnset, DrownRecover, DrownBlur, DrownTintFloor, DrownDim, DrownContrast, DrownDesaturate, DrownWarning | 1546-1584 |
| Thin air and falls | AltitudeFadeStart, AltitudeFadeFull, AltitudeKill, AltitudeWarning, FallKillDrop, FallWarning, FallSettleSpeed, FallHoldTime, FallFadeIn, FallFadeOut, FallWakeBlur, FallWakeEase | 1348-1393 |
| Sky and forest biomes | SkyModels, SkyBrightness, SkyContrast, SkySaturation, SkyBlur, SkyTint, SkyEase, SkyBiomeY, SkyBiomeFull, ForestBiomeStart, ForestBiomeFull, ForestEase, SkyJumpY, SkyJumpMultiplier, CloudHoldTime, CloudSinkTime, CloudSinkDrop, CloudStillSpeed | 1414-1463, 1525-1541 |
| Day and night | DayLengthSeconds, DayCycleOffset, DayStartHour, DayEndHour, NightSpeedup, ClockTimeAt(), NightBrightness, NightOutdoorAmbient, NightAmbient, NightSunDay, NightSunNight | 1480-1518 |
| Tools | ToolsFolder, ToolTemplates, Tools, ToolBuyBox, ToolBuyButton, TNTBuyBox, TNTBuyButton, UseButton, ToolPriceText, ToolNotEnoughShards, ToolBought, ToolEquipWarning, ToolOffPlot, TNTDetonated, TNTNothing, TNTVictim | 1604-1705, 1837-1857 |
| Effects | Effects, JumpPower, PotionDrinkSound, PotionDrinkVolume, EffectDrunk, DurationText(), EffectBarHeight, EffectBarInset, EffectBarTrack, EffectBarLow, EffectBarLowColor, EffectIconSize, EffectRowGap, EffectLabelHeight | 1714-1832 |
| HUD scaling | HudClusters, HudReference, HudScaleDampen, HudScaleMin, HudScaleMax, HudTouchScale, HudMinViewport, HudScaleFor() | 1863-1883, 1947-1957 |
| Match highlights | MaxHighlights, HighlightColor, RefreshInterval | 1889-1891 |
| Accessors and text | MaxLevel(), SpawnTable(), SpawnInterval(), Capacity(), PlotSize(), UpgradePrice(), Duration(), Plural(), UpgradeLevelText(), UpgradeStat() | 1905-2021 |
| Top earners board | BoardModel, BoardFace, BoardNormal, BoardTitle, BoardSubtitle, BoardEmpty, BoardRows, BoardRefresh, BoardCanvas, BoardPadding, BoardRowHeight, BoardRowGap, BoardFont, BoardTitleFont, BoardBack, BoardBackAlpha, BoardRowBack, BoardTitleColor, BoardSubtitleColor, BoardNameColor, BoardRateColor, BoardStroke, BoardPlaceColors, BoardPlaceDefault, BoardYouTint, BoardYouSuffix | 2032-2080 |
| Pen upgrade sign | PenSignTemplate, PenSignTitle, PenSignLevelText, PenSignMaxText, PenSignMaxButton, PenSignOut, PenSignSide, PenSignMaxDistance, PenSignCanvas, PenSignFont, PenSignBack, PenSignEdge, PenSignTitleColor, PenSignLevelColor, PenSignButtonTop, PenSignButtonBottom, PenSignButtonEdge, PenSignPoorTop, PenSignPoorBottom, PenSignMaxedColor | 2089-2115 |

### Constants outside GardenConfig

| Script | Constant | Value | Line |
|---|---|---|---|
| FruitEconomy | Base, StepLow, StepHigh | 1, 2.10, 2.60 | `ReplicatedStorage.FruitEconomy.lua:25-27` |
| FruitEconomy | SUFFIX | "" to "Dc" (12 entries) | `ReplicatedStorage.FruitEconomy.lua:62` |
| IndexRewards | COUNT | FruitIndex.Count | `ReplicatedStorage.IndexRewards.lua:19` |
| GardenToast | WARN hidden / shown | y -36 px / y 76 px | `ReplicatedStorage.GardenToast.lua:53-54` |
| GardenToast | NOTICE hidden / shown | 40 px below bottom / 150 px above | `ReplicatedStorage.GardenToast.lua:63-64` |
| GardenToast | INK_TWEEN | 0.18 s | `ReplicatedStorage.GardenToast.lua:37` |
| GardenToast | label size, ZIndex, font | 90% x 28 px, 100, FredokaOne | `ReplicatedStorage.GardenToast.lua:93-101` |
| GardenToast | MaxTextSize, stroke | 24, 2 | `ReplicatedStorage.GardenToast.lua:108`, `ReplicatedStorage.GardenToast.lua:113` |
| LoadingScreen | LOGO | rbxassetid://113441759317898 | `ReplicatedFirst.LoadingScreen.lua:40` |
| LoadingScreen | palette | BACK_TOP, BACK_BOTTOM, CARD, TRACK, FILL_LOW, FILL_HIGH | `ReplicatedFirst.LoadingScreen.lua:44-49` |
| LoadingScreen | DisplayOrder | 1000 | `ReplicatedFirst.LoadingScreen.lua:58` |
| LoadingScreen | blur size | 24 | `ReplicatedFirst.LoadingScreen.lua:82` |
| LoadingScreen | BAR_Y, BAR_H | 0.80, 0.085 | `ReplicatedFirst.LoadingScreen.lua:163-164` |
| LoadingScreen | font | FredokaOne | `ReplicatedFirst.LoadingScreen.lua:222` |
| LoadingScreen | PRELOAD_SHARE | 0.75 | `ReplicatedFirst.LoadingScreen.lua:354` |
| LoadingScreen | profile wait | 45 s | `ReplicatedFirst.LoadingScreen.lua:387` |
| LoadingScreen | MIN_SHOWN | 1.6 s | `ReplicatedFirst.LoadingScreen.lua:397` |
| LoadingScreen | end hold, fade | 0.45 s, 0.5 s | `ReplicatedFirst.LoadingScreen.lua:429`, `ReplicatedFirst.LoadingScreen.lua:432` |

## Numbers

Values marked "computed" were worked out by running the same formula as the cited line. They are not written in the code.

### Upgrades (paid in cash)

| Upgrade (key) | Ladder | Buys | Price rule | First buys | Last buy | All buys (computed) |
|---|---|---|---|---|---|---|
| Capacity (`capacity`) | 6 to 35 fruit, +1 each (`ReplicatedStorage.GardenConfig.lua:51-56`) | 29 | 100 x 1.90^(n-1) (`ReplicatedStorage.GardenConfig.lua:143`, `ReplicatedStorage.GardenConfig.lua:157`) | 100, 190, 361, 686 | 6.38e9 | 1.35e10 |
| Spawn rate (`spawnRate`) | 10 s down to 2.5 s, -0.5 each (`ReplicatedStorage.GardenConfig.lua:60-66`) | 15 | 150 x 3.40^(n-1) | 150, 510, 1730, 5900 | 4.14e9 | 5.87e9 |
| Spawn Rarity (`spawnTier`) | best fruit Blueberry up to Soul Fruit (`ReplicatedStorage.GardenConfig.lua:86-96`) | 63 | 250 x 2.60^(n-1) | 250, 650, 1690, 4390 | 1.34e28 | 2.18e28 |
| Plot size (`plotSize`) | 29.5 x 47.1 up to 48 x 78 studs (`ReplicatedStorage.GardenConfig.lua:117-123`) | 4 | 2500 x 8.0^(n-1) | 2.5k, 20k, 160k | 1.28M | 1.46M |

The code comments state the same totals: 13.5 billion, about 5.8e9 and 2.2e28 (`ReplicatedStorage.GardenConfig.lua:144-148`).

Spawn weights: `SpawnRarityFalloff` = 0.6 (`ReplicatedStorage.GardenConfig.lua:85`). At level 4 the five fruit spawn about 43.4%, 26.0%, 15.6%, 9.4%, 5.6%, best first (computed). The 40/24/14/9/5 in the comment is the limit for a long pool (`ReplicatedStorage.GardenConfig.lua:80-81`).

### Spawn Rarity level per rarity tier

| Rarity | Level whose best fruit tops the tier | Total Spawn Rarity cash to reach it (computed) | Income of that fruit, $/s (computed) |
|---|---|---|---|
| COMMON (Apple) | 4 | 6,980 | 20.2 |
| UNCOMMON | 12 | 1.49e7 | 9,830 |
| RARE | 23 | 5.48e11 | 7.12e7 |
| EPIC | 31 | 1.14e15 | 5.94e10 |
| LEGENDARY | 38 | 9.18e17 | 2.55e13 |
| MYTHIC | 46 | 1.92e21 | 3.16e16 |
| ASTRAL | 51 | 2.28e23 | 3.03e18 |
| ETERNAL | 59 | 4.76e26 | 5.26e21 |
| TRANSCENDENT (Soul Fruit) | 63 | 2.18e28 | 2.37e23 |

Income runs from 1/s at Blueberry to about 2e23/s at Soul Fruit, as the header says (`ReplicatedStorage.FruitEconomy.lua:18`).

### Index rewards

`IndexBonusTotal` = 0.50, the whole Index is worth +50% cash (`ReplicatedStorage.GardenConfig.lua:393`). `IndexMoneySeconds` = 60 (`ReplicatedStorage.GardenConfig.lua:394`). `IndexShardsLow` = 5, `IndexShardsHigh` = 2500 (`ReplicatedStorage.GardenConfig.lua:395-396`).

| Rarity | Fruit | Coin bonus per fruit (computed) | Bonus for whole tier (computed) | Shards for whole tier (computed) |
|---|---|---|---|---|
| COMMON | 5 | +0.017% | +0.09% | 31 |
| UNCOMMON | 8 | +0.035% | +0.28% | 94 |
| RARE | 11 | +0.070% | +0.77% | 343 |
| EPIC | 8 | +0.140% | +1.12% | 610 |
| LEGENDARY | 7 | +0.279% | +1.96% | 1,125 |
| MYTHIC | 8 | +0.559% | +4.47% | 2,700 |
| ASTRAL | 5 | +1.117% | +5.59% | 3,200 |
| ETERNAL | 8 | +2.234% | +17.87% | 9,750 |
| TRANSCENDENT | 4 | +4.468% | +17.87% | 8,675 |

A fully claimed Index pays 26,528 shards in total (computed).

### Cash, offline, rebirth, storage

| Key | Value | Line |
|---|---|---|
| FriendBonus | +10% per friend in the server | `ReplicatedStorage.GardenConfig.lua:357` |
| IncomeTick | income paid every 5 s | `ReplicatedStorage.GardenConfig.lua:372` |
| CashCatchUp | 2.5 per second counter ease | `ReplicatedStorage.GardenConfig.lua:378` |
| OfflineRate | 3% of live income | `ReplicatedStorage.GardenConfig.lua:753` |
| OfflineCapSeconds | 5 hours | `ReplicatedStorage.GardenConfig.lua:754` |
| OfflineMinimum, OfflineMinimumAway | popup only above $1 and 300 s away | `ReplicatedStorage.GardenConfig.lua:763`, `ReplicatedStorage.GardenConfig.lua:776` |
| RebirthNeedsAll | true: every fruit must be discovered | `ReplicatedStorage.GardenConfig.lua:993` |
| RebirthCashBonus | +100% per rebirth, stacking (x2, x3, ...), offline too | `ReplicatedStorage.GardenConfig.lua:995-1000` |
| StoredCap | 5 fruit in storage, all kinds together | `ReplicatedStorage.GardenConfig.lua:533` |
| InitialFruits | 3 seeded on a new plot | `ReplicatedStorage.GardenConfig.lua:287` |
| MaxFruitsPerPlot | 35, tied to CapacityTop | `ReplicatedStorage.GardenConfig.lua:286` |

### Cooldowns and reach

| Key | Value | Line |
|---|---|---|
| LockDuration, LockCooldown | 60 s shield, 30 s recharge after it ends | `ReplicatedStorage.GardenConfig.lua:885-886` |
| MergeAllCooldown | 300 s | `ReplicatedStorage.GardenConfig.lua:973` |
| ClearCooldown | 5 s | `ReplicatedStorage.GardenConfig.lua:1056` |
| PromptDistance, MaxReach | 12 studs, 16 studs | `ReplicatedStorage.GardenConfig.lua:304-305` |
| PlotStandMargin, PlotStandBand | 2.5 studs, 12 studs | `ReplicatedStorage.GardenConfig.lua:321`, `ReplicatedStorage.GardenConfig.lua:940` |
| WalkSpeed | 20 | `ReplicatedStorage.GardenConfig.lua:943` |
| MergeAllGamepassId | 1982540690 | `ReplicatedStorage.GardenConfig.lua:961` |

### Tools and effects (shards)

| Tool | Kind | Price | Duration | Effect stats | Lines |
|---|---|---|---|---|---|
| TNT | place | 2500 | - | radius 18, carried at 2.5 studs | `ReplicatedStorage.GardenConfig.lua:1616-1626` |
| SpeedPotion | consume | 5000 | 300 s | walk x1.6 | `ReplicatedStorage.GardenConfig.lua:1631-1639`, `ReplicatedStorage.GardenConfig.lua:1715-1722` |
| JumpPotion | consume | 5000 | 300 s | JumpPower 85 (base 50) | `ReplicatedStorage.GardenConfig.lua:1640-1649`, `ReplicatedStorage.GardenConfig.lua:1723-1728`, `ReplicatedStorage.GardenConfig.lua:1793` |
| AetherPotion | consume | 10000 | 1800 s | no stat; immunity to thin air | `ReplicatedStorage.GardenConfig.lua:1655-1666`, `ReplicatedStorage.GardenConfig.lua:1729-1740` |
| InvisibilityPotion | consume | 5000 | 60 s | trail rate 20, life 1.4, size 1.0 | `ReplicatedStorage.GardenConfig.lua:1667-1678`, `ReplicatedStorage.GardenConfig.lua:1741-1760` |
| FeatherFallingPotion | consume | 5000 | 600 s | fall capped at 22 studs/s | `ReplicatedStorage.GardenConfig.lua:1685-1691`, `ReplicatedStorage.GardenConfig.lua:1767-1777` |
| LuckPotion | consume | 7500 | 300 s | +1 Spawn Rarity level; at max level, falloff 0.35 | `ReplicatedStorage.GardenConfig.lua:1698-1704`, `ReplicatedStorage.GardenConfig.lua:1781-1788` |

King effects, in shards: Crown 0, Fire Aura 4000, Water Aura 6000, Slash Aura 10000, Rift Aura 20000 (`ReplicatedStorage.GardenConfig.lua:1236-1242`).

### World

| Key | Value | Line |
|---|---|---|
| DayLengthSeconds, DayStartHour, DayEndHour, NightSpeedup | 600, 6, 18, 10: 300 s of day and 30 s of night | `ReplicatedStorage.GardenConfig.lua:1475-1484` |
| CurrentSpeed, CurrentAirAccel | 42 studs/s out, 20 studs/s^2 while airborne | `ReplicatedStorage.GardenConfig.lua:1310-1311` |
| ReturnSpeed, ReturnAirAccel | 48 studs/s home, 160 studs/s^2 | `ReplicatedStorage.GardenConfig.lua:1319`, `ReplicatedStorage.GardenConfig.lua:1324` |
| DrownSweptSpeed | 30 studs/s counts as "being carried" | `ReplicatedStorage.GardenConfig.lua:1291` |
| DrownFadeStart, DrownFadeFull, DrownKill | r 2600, 2775, 2825 (past the forest wall at r 1470) | `ReplicatedStorage.GardenConfig.lua:1556-1558` |
| AltitudeFadeStart, AltitudeFadeFull, AltitudeKill | y 220, 405, 418 | `ReplicatedStorage.GardenConfig.lua:1348-1350` |
| FallKillDrop | a 220-stud drop kills | `ReplicatedStorage.GardenConfig.lua:1364` |
| SkyBiomeY, SkyBiomeFull | 405 to 440 | `ReplicatedStorage.GardenConfig.lua:1443-1444` |
| ForestBiomeStart, ForestBiomeFull | r 650 to 990 | `ReplicatedStorage.GardenConfig.lua:1461-1462` |
| SkyJumpY, SkyJumpMultiplier | above y 430, jump x1.6 | `ReplicatedStorage.GardenConfig.lua:1525-1526` |
| CloudHoldTime, CloudSinkTime | 3 s still, then 4 s to sink through | `ReplicatedStorage.GardenConfig.lua:1538-1539` |

### Saving and toasts

| Key | Value | Line |
|---|---|---|
| SaveInterval, SaveTouch | autosave every 60 s; a merge pulls it to 8 s | `ReplicatedStorage.GardenConfig.lua:1131-1132` |
| SaveRetryCount, SaveRetryBase | 5 tries, 1 s doubling | `ReplicatedStorage.GardenConfig.lua:1133-1134` |
| ShutdownRetryCount, ShutdownDeadline | 2 tries, 25 s | `ReplicatedStorage.GardenConfig.lua:1135-1136` |
| LockStaleSeconds, LockWaitDelay, LockWaitAttempts | 120 s, 7 s x 20 | `ReplicatedStorage.GardenConfig.lua:1144-1146` |
| ToastDropTime, ToastHold, ToastFadeTime | 0.32 s, 1.7 s, 0.45 s | `ReplicatedStorage.GardenConfig.lua:1165-1167` |

## Extension points for the expansion

All 10 planned features touch this subsystem. Most touch it lightly. Feature 1 changes it the most.

### 1. Rebirth-gated rarity

Today any player can buy Spawn Rarity up to Soul Fruit and merge to rank 1. Nothing here knows about rebirths.

- **Tier cap.** Add a function near the accessors, for example `Config.MaxTierFor(rebirths)`, returning `rebirths + 1` clamped to 9. Put it near `ReplicatedStorage.GardenConfig.lua:1893-1933`. GardenConfig already requires FruitIndex, so it can turn a tier into "best allowed rank" from the `tier` field (`ReplicatedStorage.GardenConfig.lua:14`). At 0 rebirths the best allowed fruit is Apple, rank 60 (`ReplicatedStorage.Fruits.FruitIndex.lua:69`). That is Spawn Rarity level 4 (`ReplicatedStorage.GardenConfig.lua:83-84`).
- **Merge cap.** `FruitIndex.MergeInto` has no cap (`ReplicatedStorage.Fruits.FruitIndex.lua:95-98`). Do not add the cap inside FruitIndex; the importer overwrites it (`ReplicatedStorage.Fruits.FruitIndex.lua:2-3`). Check the cap at both callers: the single merge (`ServerScriptService.GardenService.Merging.lua:179`) and the Merge All cascade (`ServerScriptService.GardenService.Merging.lua:315`). Missing the second lets Merge All jump the gate.
- **Spawn cap.** Keep `Config.MaxLevel("spawnTier")` at 63. Save loading clamps saved levels with it (`ServerScriptService.GardenStore.lua:482`), and so does `publishUpgrades` (`ServerScriptService.GardenService.Upgrades.lua:294-298`). Instead, give `Config.UpgradePrice` an optional cap so it returns nil at the cap. nil already means "maxed" on both server and client (`ReplicatedStorage.GardenConfig.lua:1926-1933`). `UpgradeLevelText` and `UpgradeStat` should show the cap too (`ReplicatedStorage.GardenConfig.lua:1996-2021`).
- **Luck potion.** It raises the spawn level by `tierBonus` up to `MaxLevel` (`ServerScriptService.GardenService.Upgrades.lua:372-379`). A Lucky fruit enters the Index on pick-up (`ServerScriptService.GardenService.Carrying.lua:185-193`). It must stop at the rebirth cap, or Luck sells fruit past the gate.
- **"Full Index" gate.** `indexComplete` counts all 64 ranks (`ServerScriptService.GardenService.Progression.lua:311-320`). A 0-rebirth player can only ever reach 5 fruit, so rebirth would be impossible. Bound it by the cap. Update `RebirthNeedsAll` and the `RebirthNotReady` text (`ReplicatedStorage.GardenConfig.lua:993`, `ReplicatedStorage.GardenConfig.lua:1036`).
- **Rebirth price.** No shard or cash cost exists. The only gate is the Index check (`ServerScriptService.GardenService.RebirthLock.lua:52`). Add per-rebirth cost keys in the rebirth section (`ReplicatedStorage.GardenConfig.lua:987-1039`).
- **"Slight boost".** `RebirthCashBonus` is +100% and stacks: x2, x3, x4 (`ReplicatedStorage.GardenConfig.lua:995-1000`, `ServerScriptService.GardenService.Progression.lua:325`). Lower it. The client text reads it too (`StarterPlayer.StarterPlayerScripts.RebirthMenu.lua:88`).
- **+1 storage per rebirth.** `StoredCap` is a flat 5 (`ReplicatedStorage.GardenConfig.lua:533`). Turn it into a function of the `Rebirths` attribute. Three readers use the constant directly and all must change: `ServerScriptService.GardenService.Storage.lua:43-45`, `StarterPlayer.StarterPlayerScripts.GardenClient.lua:398-400`, `StarterPlayer.StarterPlayerScripts.StoredFruitsMenu.lua:466-468`.
- **Price curve.** The 2.60 Spawn Rarity growth was tuned for one long climb with no resets (`ReplicatedStorage.GardenConfig.lua:125-142`). Reaching Apple costs 6,980 in total (computed). Retune once runs are short.
- **What could break.** Existing saves already hold Spawn Rarity levels, plot fruit, stored fruit and Index entries above the new cap. `discoverThroughTier` re-discovers everything up to the saved level on every load (`ServerScriptService.GardenService.Persistence.lua:289`, `ServerScriptService.GardenService.Progression.lua:192-199`). Decide how old saves are treated before shipping.

### 2. Bats that upgrade each rebirth

- No weapon exists in Config. Tools have two kinds, "place" and "consume", and the kind decides the button shown (`ReplicatedStorage.GardenConfig.lua:1607-1614`). A bat is neither, because it is not used up. Either add a third kind, with new branches where the kind is read (`StarterPlayer.StarterPlayerScripts.GardenClient.lua:226-227`), or keep the bat out of `Config.Tools` entirely.
- If a bat is put in `Config.Tools`, the save code will store it as a counted stack like any tool (`ServerScriptService.GardenService.Persistence.lua:64`).
- Knockback per rebirth: a new ladder keyed by rebirth count, like `RebirthCashBonus` (`ReplicatedStorage.GardenConfig.lua:1000`).
- Bat colour by rarity: reuse `RarityColors` or `RarityGradients`, both keyed by tierName (`ReplicatedStorage.GardenConfig.lua:582-592`, `ReplicatedStorage.GardenConfig.lua:631-641`).
- A bat model under ReplicatedStorage is preloaded with no extra work (`ReplicatedFirst.LoadingScreen.lua:298-309`).
- What could break: the lock dome blocks only TNT; it collides with nothing and pushes nobody (`ReplicatedStorage.GardenConfig.lua:897-900`). Bats would hit players inside a locked plot unless they get their own rule.

### 3. Explosives that drain fruit health

- TNT is `Config.Tools.TNT`: 2500 shards, radius 18 (`ReplicatedStorage.GardenConfig.lua:1616-1626`). It deletes every fruit in range that is not mid-animation (`ServerScriptService.GardenService.Tools.lua:278-290`).
- Fruit have no health anywhere in this subsystem. Add a health curve per rank, derived from FruitIndex the way income is (`ReplicatedStorage.FruitEconomy.lua:39-50`). Add damage fields to each explosive's `Config.Tools` entry.
- More TNT tiers and nukes are more `kind = "place"` entries, each with a template part of the same name in ReplicatedStorage.Tools (`ReplicatedStorage.GardenConfig.lua:1601-1605`).
- The strings assume deletion and must be reworded: `TNTDetonated` "Destroyed %d fruit!", `TNTNothing`, `TNTVictim` (`ReplicatedStorage.GardenConfig.lua:1855-1857`).
- Which plot TNT lands on uses `PlotStandBand` (`ReplicatedStorage.GardenConfig.lua:933-940`). A nuke big enough to cover two plots needs new targeting.
- The shop sorts tools by key name, so key names set shelf order (`StarterPlayer.StarterPlayerScripts.ShopMenu.lua:379-385`).
- What could break: when health hits zero, keep the current order: clear `PlotName` first, then destroy, so the spawner sees the free slot (`ServerScriptService.GardenService.Tools.lua:284-287`).

### 4. Underground base and ores

- **The 100k-shard purchase.** `Config.UpgradeCurrency` looks like the hook ("Anything not listed here costs cash"), but nothing reads it (`ReplicatedStorage.GardenConfig.lua:189-190`). `buyUpgrade` always spends cash (`ServerScriptService.GardenService.Upgrades.lua:301-319`). Either make `buyUpgrade` honour it, or build a separate purchase. A new upgrade key must be added to `UpgradeKeys` or it will not save (`ReplicatedStorage.GardenConfig.lua:37-43`). It also needs prices, or `MaxLevel` is 0 and the purchase is refused (`ReplicatedStorage.GardenConfig.lua:1905-1908`, `ServerScriptService.GardenService.Upgrades.lua:303`).
- **Ores index.** Do not append ores to FruitIndex. Its `Count` drives the spawn ladder (`ReplicatedStorage.GardenConfig.lua:87`), the income ladder (`ReplicatedStorage.FruitEconomy.lua:40-46`), Index bonus weights and shards (`ReplicatedStorage.IndexRewards.lua:42-50`, `ReplicatedStorage.IndexRewards.lua:76-85`) and the rebirth gate (`ServerScriptService.GardenService.Progression.lua:311-320`). Adding ores would re-price every one of these and make ores part of the rebirth gate. Build a parallel OreIndex with the same shape (`List`, `ByName`, `Count`, `Get`, `MergeInto`, `Template`, `ReplicatedStorage.Fruits.FruitIndex.lua:89-99`) and its own income and reward curves. Tools set the precedent: they are kept out of the fruit tables on purpose (`ReplicatedStorage.GardenConfig.lua:1595-1599`).
- Save loading drops any stored or Index name not in `FruitIndex.ByName` (`ServerScriptService.GardenStore.lua:289`, `ServerScriptService.GardenStore.lua:345`). Ores need their own save fields and their own check.
- The Index replicates as IntValues keyed by fruit name (`ReplicatedStorage.GardenConfig.lua:386-390`). An ore name equal to a fruit name would collide.
- Scale check: a full Index pays 26,528 shards (computed), so 100k shards is about 3.8 full Index claims.
- Ore glow can reuse `RarityFrames` and `RarityColors` if ores carry a tierName (`ReplicatedStorage.GardenConfig.lua:563-592`).

### 5. Forest pets

- Earnings "matched to the player's stage": derive pet income from `Economy.ForRank` of the player's best rank (`ReplicatedStorage.FruitEconomy.lua:52-54`). Each rank earns 2.1x to 2.6x the one below (`ReplicatedStorage.FruitEconomy.lua:26-27`), so any flat number goes stale within a few ranks.
- "Rarer animals match the rarity of the max fruit you own": read tier and tierName from FruitIndex (`ReplicatedStorage.Fruits.FruitIndex.lua:9-74`).
- Pets merging two-into-one is the same shape as `MergeInto`, but in a separate list. Pets are not fruit; keep them out of FruitIndex, or save loading drops them (`ServerScriptService.GardenStore.lua:289`).
- Decide whether pet income counts offline. The current rule: permanent account properties count offline, per-session ones do not (`ReplicatedStorage.GardenConfig.lua:756-759`).

### 6. Boats, swimming, no currents

- The currents are described by `CurrentSpeed` 42, `CurrentAirAccel`, `ReturnSpeed` 48, `ReturnFloors` and `ReturnAirAccel` (`ReplicatedStorage.GardenConfig.lua:1310-1324`). `CurrentSpeed` and `ReturnSpeed` must match the plate velocities in the world (`ReplicatedStorage.GardenConfig.lua:1310`, `ReplicatedStorage.GardenConfig.lua:1319`). These become dead when the plates go.
- The "in the sea" test is the floor under you: `CurrentPlate` or `Seabed` (`ReplicatedStorage.GardenConfig.lua:1292-1298`, `ReplicatedStorage.GardenConfig.lua:1327`). Delete the plates and only `Seabed` counts.
- `DrownSweptSpeed` 30 assumes walkers stay under it (`ReplicatedStorage.GardenConfig.lua:1286-1291`). A Speed potion already gives 20 x 1.6 = 32 (`ReplicatedStorage.GardenConfig.lua:943`, `ReplicatedStorage.GardenConfig.lua:1721`). If this test is reused for swimming or boats, a fast runner on a stone counts as swept.
- The forest music ramp is measured by distance from the island, so it still works with boats (`ReplicatedStorage.GardenConfig.lua:1446-1451`). The "about eight seconds" in the comment assumes the current (`ReplicatedStorage.GardenConfig.lua:1458-1459`).
- Swimming fatigue is new. The drowning view keys (heartbeat, tint, blur, desaturate) could be reused for its warning look (`ReplicatedStorage.GardenConfig.lua:1546-1584`).
- Aqua Affinity is a new potion; follow feature 7.
- A 1000-shard boat does not fit `kind = "place"`, which needs a plot under you (`ReplicatedStorage.GardenConfig.lua:1852-1854`). It needs its own purchase path. If artillery breaks boats, boats need health like feature 3.

### 7. More potions

The recipe is three parts:

1. A `Config.Tools` entry: `display`, `kind = "consume"`, `price`, `effect`, `duration`. Leave `carrySize` out for bottles (`ReplicatedStorage.GardenConfig.lua:1615-1705`, `ReplicatedStorage.GardenConfig.lua:1628-1630`). An `image` field shows flat art in the shop instead of a spinning model (`ReplicatedStorage.GardenConfig.lua:1661-1665`).
2. A `Config.Effects` entry: `display`, `bar` colour, `icon`, and its stat fields (`ReplicatedStorage.GardenConfig.lua:1714-1789`).
3. A template part with the same name in ReplicatedStorage.Tools (`ReplicatedStorage.GardenConfig.lua:1601-1605`).

What else happens on its own: save loading keeps the tool because its name is in `Config.Tools` (`ServerScriptService.GardenService.Persistence.lua:64`). The status stack gets a row for every `Config.Effects` key (`StarterPlayer.StarterPlayerScripts.GardenClient.lua:550-556`). The icon is preloaded because it is an `rbxassetid://` string (`ReplicatedFirst.LoadingScreen.lua:338`).

What does not happen on its own: the effect does nothing until code reads its stat. Speed and Jump are read in `ServerScriptService.GardenService.lua:211-217`, Luck in `ServerScriptService.GardenService.Upgrades.lua:372-379`. A consume tool whose `effect` is missing from `Config.Effects` only logs a warning (`ServerScriptService.GardenService.Tools.lua:187-189`). Two existing icons are still placeholders (`ReplicatedStorage.GardenConfig.lua:1770-1771`, `ReplicatedStorage.GardenConfig.lua:1784-1785`).

### 8. Plot visual overhaul, studded logs and lily pads, border mountains

- `PlotSizes[0]` is the measured size of the plot in the map, 29.5 x 47.1 (`ReplicatedStorage.GardenConfig.lua:117-118`). If the plot model changes size, re-measure it. A wrong level 0 made the first purchase buy half a stud and made a rebirth shrink plots below a fresh one (`ReplicatedStorage.GardenConfig.lua:109-112`).
- The biggest plot, fence included, was checked to fit the white band with clearance (`ReplicatedStorage.GardenConfig.lua:103-107`). A thicker fence or border eats that clearance.
- Things placed relative to the plot: owner headshot 25 studs above the soil (`ReplicatedStorage.GardenConfig.lua:294-298`), spawn margin 0.6 from planter walls (`ReplicatedStorage.GardenConfig.lua:288`), pen sign 6.5 out and 13 to the side of a 16-stud gate (`ReplicatedStorage.GardenConfig.lua:2095-2099`). The lock dome uses its own authored size, not the plot's (`ReplicatedStorage.GardenConfig.lua:891-895`).
- Lily pads: Config says stepping stones replaced them on 28 Sep 2026 (`ReplicatedStorage.GardenConfig.lua:1283-1285`). The forest music starts just past the last stone at r 642 (`ReplicatedStorage.GardenConfig.lua:1455`, `ReplicatedStorage.GardenConfig.lua:1461`). If the stone field is regrown, re-measure, as the comment asks (`ReplicatedStorage.GardenConfig.lua:1550-1555`).
- Border mountains: the forest has an invisible boundary wall at r 1470, and the drown radii sit outside it (`ReplicatedStorage.GardenConfig.lua:1263-1269`). New mountains must keep that wall or the drown radii must move.
- Studded look: sky lighting was toned down so white studs did not clip flat (`ReplicatedStorage.GardenConfig.lua:1422-1425`). Expect the same trade-off on studded plots.

### 9. UI overhaul to the image kit

- **Fonts.** HUD and menus use FredokaOne. World text over fruit uses Highway (`ReplicatedStorage.GardenConfig.lua:20-25`). Font keys: `UpgradeFont`, `BoardFont`, `PenSignFont` (FredokaOne) and `BoardTitleFont` (LuckiestGuy, the one non-Fredoka UI face) (`ReplicatedStorage.GardenConfig.lua:266`, `ReplicatedStorage.GardenConfig.lua:2058-2059`, `ReplicatedStorage.GardenConfig.lua:2103`). GardenToast and LoadingScreen hard-code FredokaOne (`ReplicatedStorage.GardenToast.lua:95`, `ReplicatedFirst.LoadingScreen.lua:222`).
- **Baked images are already Config ids.** Swapping art is an id change: `UpgradeCards` (`ReplicatedStorage.GardenConfig.lua:197-215`), `RarityFrames` (`ReplicatedStorage.GardenConfig.lua:563-573`), rebirth panel art (`ReplicatedStorage.GardenConfig.lua:1008-1019`), shop art (`ReplicatedStorage.GardenConfig.lua:1185-1189`), tool buy box and Use button (`ReplicatedStorage.GardenConfig.lua:1837-1846`).
- **King UI.** The king effect cards use tinted Roblox particle sprites, not kit art (`ReplicatedStorage.GardenConfig.lua:1252-1259`). Effect names and prices are in `KingEffects` (`ReplicatedStorage.GardenConfig.lua:1236-1242`). Crown messages are at `ReplicatedStorage.GardenConfig.lua:1586-1591`. None of the six files here contain emoji or any non-ASCII text (checked by byte scan), so the emoji Samarth saw come from other scripts.
- **Preload rule.** Store new kit ids in Config as `"rbxassetid://..."` strings. The loading screen only picks up strings with that prefix (`ReplicatedFirst.LoadingScreen.lua:333-338`).
- **Pre-draw rule.** Only direct children of GardenUI whose names end in "Menu" are warmed (`ReplicatedFirst.LoadingScreen.lua:411-416`). A panel in another ScreenGui, or with another name, will pop in on first open.
- **HUD scale rule.** A new HUD cluster must be added to `HudClusters`, or it keeps its pixel size while the rest shrinks (`ReplicatedStorage.GardenConfig.lua:1859-1867`).
- **Toasts.** Positions are tuned around TopButtons and the carry stack (`ReplicatedStorage.GardenToast.lua:42-48`). Moving those means moving the toasts. Colours and timings are Config keys (`ReplicatedStorage.GardenConfig.lua:1159-1167`).
- **Loading screen.** Its palette and logo are hard-coded (`ReplicatedFirst.LoadingScreen.lua:40-49`). A kit-style version must still be built in code, not in StarterGui (`ReplicatedFirst.LoadingScreen.lua:22-24`).
- **Strokes.** UIStroke thickness does not scale, so panels divide by a reference canvas (`ReplicatedStorage.GardenConfig.lua:420-425`, `ReplicatedStorage.GardenConfig.lua:1071-1082`). New kit panels need the same.

### 10. Royal combat

- Combat tuning is not in GardenConfig; a search for duel, slam, sword and arena finds only music. Config holds the royal and fight music: `RoyalMusicId`, `FightMusicId`, `FightMusicRange` 140 (`ReplicatedStorage.GardenConfig.lua:843-853`). The crown itself lives in CrownConfig (`ReplicatedStorage.GardenConfig.lua:1205-1207`), which is outside this subsystem.
- Weapon skins picked before a fight can copy the `KingEffects` pattern: a list of `key`, `display`, `price`, `asset`, `kind`, plus a by-key index (`ReplicatedStorage.GardenConfig.lua:1236-1244`), with ownership on a player attribute like `KingOwned` (`ServerScriptService.GardenService.Progression.lua:222`).
- Sword animations: the carry pose shows the rule. An animation must be published to the place owner's account. A runtime-registered one works in Studio and silently fails live (`ReplicatedStorage.GardenConfig.lua:338-345`).
- Feather Falling is switched off in the Royal Arena (`ReplicatedStorage.GardenConfig.lua:1766`). Any new potion needs the same decision.

## Gotchas

1. **Rank runs opposite to level.** Rank 1 is the best fruit; level 0 is the start. Getting this backwards in Clear deletes your best fruit (`ReplicatedStorage.GardenConfig.lua:1048-1050`, `ReplicatedStorage.GardenConfig.lua:83-84`).
2. **FruitIndex is generated.** Hand edits are lost when the importer runs again (`ReplicatedStorage.Fruits.FruitIndex.lua:2-3`).
3. **Changing the fruit count re-prices everything.** Spawn ladder, income, Index bonus, shards and rarity text all derive from `Count` (`ReplicatedStorage.GardenConfig.lua:87`, `ReplicatedStorage.FruitEconomy.lua:40`, `ReplicatedStorage.IndexRewards.lua:19`).
4. **Already-claimed Index bonuses change too.** `Bonus` re-normalises over the whole ladder (`ReplicatedStorage.IndexRewards.lua:38-41`). The server re-sums claimed entries live, so old players' bonus moves when tiers change (`ServerScriptService.GardenService.Progression.lua:203-214`).
5. **Renaming a fruit's `name` deletes it from saves.** Save loading drops stored and Index names not in `ByName` (`ServerScriptService.GardenStore.lua:289`, `ServerScriptService.GardenStore.lua:345`). Changing `display` is safe.
6. **`UpgradeKeys` is the save list.** Remove a key and that level silently stops saving (`ReplicatedStorage.GardenConfig.lua:39-43`).
7. **`UpgradeCurrency` does nothing.** It is declared empty and never read; every upgrade costs cash (`ReplicatedStorage.GardenConfig.lua:190`, `ServerScriptService.GardenService.Upgrades.lua:313-319`).
8. **Ladders are 0-based and `#` counts steps.** Adding `PlotSizes[5]` adds a purchase priced by the formula: 2500 x 8^4, rounded to 10.2M (computed) (`ReplicatedStorage.GardenConfig.lua:169-175`, `ReplicatedStorage.GardenConfig.lua:161-165`).
9. **Clamping hides bugs.** A level of 999 reads as max level with no error (`ReplicatedStorage.GardenConfig.lua:1895-1903`).
10. **Income rounding compounds.** Each rank is rounded, then the next is built from the rounded value (`ReplicatedStorage.FruitEconomy.lua:45`).
11. **`Format` floors small numbers.** Below 1000 it prints a whole number, so 0.5 shows "0" (`ReplicatedStorage.FruitEconomy.lua:67-68`). The suffix list ends at Dc (`ReplicatedStorage.FruitEconomy.lua:62`, `ReplicatedStorage.FruitEconomy.lua:72`).
12. **Config sections are not reliable.** The "hud scaling" header holds shop keys (`ReplicatedStorage.GardenConfig.lua:1169-1194`); the real HUD keys are at `ReplicatedStorage.GardenConfig.lua:1859-1883`. Drown radii sit inside "the day" (`ReplicatedStorage.GardenConfig.lua:1556-1558`). Crown messages sit after the drown keys (`ReplicatedStorage.GardenConfig.lua:1586-1591`). Search by key name.
13. **Some comments sit above the wrong function.** The 1-based level note belongs to `UpgradeLevelText` (`ReplicatedStorage.GardenConfig.lua:1935-1937`, `ReplicatedStorage.GardenConfig.lua:1996`). The plural note sits above `Duration` (`ReplicatedStorage.GardenConfig.lua:1959-1969`).
14. **Stale comments.** "A plot can hold 40 fruit" (`ReplicatedStorage.GardenConfig.lua:1102`) but the cap is 35 (`ReplicatedStorage.GardenConfig.lua:52`). "The card art is the same for all three" tools (`ReplicatedStorage.GardenConfig.lua:1834-1836`), but there are seven tools.
15. **`RebirthAction` is not read at runtime.** The GardenUI instance holds its own image; keep the two in step by hand (`ReplicatedStorage.GardenConfig.lua:1011-1015`).
16. **The carry animation belongs to user 1015320113.** Move the place to a group and it must be republished (`ReplicatedStorage.GardenConfig.lua:343-345`).
17. **Tester shards are a floor, by user id.** A listed account is topped up to the number on every join (`ReplicatedStorage.GardenConfig.lua:400-413`).
18. **Number ids are not preloaded.** The sweep takes only `rbxassetid://` strings, so music ids, the merge sound and the carry animation (all plain numbers) are skipped (`ReplicatedFirst.LoadingScreen.lua:338`, `ReplicatedStorage.GardenConfig.lua:349`, `ReplicatedStorage.GardenConfig.lua:741`, `ReplicatedStorage.GardenConfig.lua:798-803`).
19. **The loading screen gives up after 45 seconds.** It drops even if the profile never arrived (`ReplicatedFirst.LoadingScreen.lua:383-392`).
20. **GardenToast waits forever for GardenUI.** `WaitForChild` has no timeout here. Rename GardenUI and every toast hangs (`ReplicatedStorage.GardenToast.lua:84`).
21. **One label per channel.** A new message on the same channel replaces the old one at once, so quick messages hide earlier ones (`ReplicatedStorage.GardenToast.lua:22-24`).
22. **Server scripts must not require GardenToast.** It touches PlayerGui (`ReplicatedStorage.GardenToast.lua:17-20`).
23. **`ShopTabs` is decided once, at load.** Flipping `KingEnabled` later does not change the tab list (`ReplicatedStorage.GardenConfig.lua:1210-1212`).
24. **A Speed potion beats the swept-speed line.** Walk 20 x 1.6 = 32 is above `DrownSweptSpeed` 30 (`ReplicatedStorage.GardenConfig.lua:943`, `ReplicatedStorage.GardenConfig.lua:1721`, `ReplicatedStorage.GardenConfig.lua:1291`). Harmless today because the drown radii are far out (`ReplicatedStorage.GardenConfig.lua:1556-1558`).
