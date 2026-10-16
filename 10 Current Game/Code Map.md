---
tags: [current-game, code-map, index]
date: 2026-10-04
---
# Code Map

This is the index for the code map of Fruit Empire as it is today. It sums up the seven subsystem notes in this folder and joins them up: every remote, the whole save document, the attributes, where each planned feature hooks in, and the gotchas. The notes hold the detail. Part of [[Fruit Empire - Home]]. The plan it serves is [[Brain Dump 2026-10-04]].

All code is the export in `~/fruit-empire/src-export-2026-10-04`, taken 4 Oct 2026.

**How to read the citations.** `File.lua:LINE` is a line in that export. A range like `:160-203` means those lines. File names are shortened like this:
- `GardenService.lua` is `ServerScriptService.GardenService.lua` (the main server script).
- `Merging.lua`, `Crown.lua`, `Ambient.lua` and so on are the modules `ServerScriptService.GardenService.<Name>.lua`. `DevPanel.lua` is `ServerScriptService.GardenService.DevPanel.lua`.
- `GardenStore.lua` is `ServerScriptService.GardenStore.lua` (the save layer).
- `GardenConfig.lua`, `CrownConfig.lua`, `FruitIndex.lua`, `FruitEconomy.lua`, `IndexRewards.lua`, `GardenToast.lua` are in `ReplicatedStorage` (FruitIndex is under `ReplicatedStorage.Fruits`). `LoadingScreen.lua` is in `ReplicatedFirst`.
- Client scripts such as `GardenClient.lua`, `RoyalUI.lua`, `ShopMenu.lua` are `StarterPlayer.StarterPlayerScripts.<Name>.lua`.
- Builders: `ForestBuilder.lua`, `ShallowsBuilder.lua`, `SaturationPass.lua`, `CloudBuilder.lua` are in `ServerStorage`. `RoyalCapitalBuilder.lua`, `SkyIslandBuilder.lua`, `SkyIslandDecor.lua`, `BeanstalkCrown.lua`, `ExportParts.lua` are in `ServerStorage.ZZTools`. `DevPanel.Server.lua` and `DevPanel.Client.lua` are in `ServerStorage.DevPanel`.

**Terms used in this note.**
- **Server / client**: the server is Roblox's machine that runs the game for everyone. The client is each player's own device.
- **Script / LocalScript / ModuleScript**: a Script runs on the server. A LocalScript runs on one client. A ModuleScript is a script that returns a value; other scripts load it with `require`.
- **Remote**: a `RemoteEvent` (a one-way message between server and client) or a `RemoteFunction` (a question that waits for an answer).
- **Attribute**: a named value set on a Roblox object with `SetAttribute`. A value the server sets on a player copies to clients on its own.
- **Plot**: one player's garden pen. There are 12, in a ring round the beanstalk.
- **Soil**: the flat part of a plot that fruit sit on.
- **Rank**: a fruit's place on the 64-fruit merge ladder. Rank 64 (Blueberry) is the worst. Rank 1 (Soul Fruit) is the best. Two fruit of rank R merge into one of rank R-1 (`FruitIndex.lua:95-98`).
- **Tier**: the rarity group, 1 (COMMON) to 9 (TRANSCENDENT), stored as `tier` and `tierName` on each fruit (`FruitIndex.lua:9-74`).
- **Level**: how many times an upgrade was bought, counted from 0.
- **Spawn Rarity**: the upgrade that decides which fruit can spawn. The code and saves call it `spawnTier`.
- **Index**: the in-game catalogue of fruit you have discovered. Claiming an entry pays cash and shards.
- **Shards**: the second currency. Index claims and royal events pay them. Tools and King auras cost them.
- **Profile**: the one saved document per player.
- **G**: the shared Lua table the main server script hands to every server module, so modules can call each other.
- **Busy**: an attribute on a fruit that is mid-drop or mid-merge. Nobody can pick up or merge a Busy fruit.
- **King's Throne**: the Royal System in the sky castle. One player holds the crown.
- **Ring-out**: knocking a fighter off the arena ring. Duels are scored by ring-outs, not damage.

## 1. How the game works today

Every player who joins gets a free plot in a ring of 12 plots round a beanstalk. Fruit grow on your plot on a timer. You press E to lift a fruit over your head, and press E on an identical fruit to merge the two into one fruit a rank better. The ladder has 64 fruit in 9 rarity tiers, from Blueberry up to Soul Fruit (`FruitIndex.lua:9-74`). Every fruit on your plot earns cash every 5 seconds, and each rank earns 2.1 to 2.6 times the rank below it, so better fruit are worth far more (`FruitEconomy.lua:25-27`, `GardenConfig.lua:372`). Cash buys four upgrades: more room on the plot, faster spawning, better spawns (Spawn Rarity) and a bigger plot (`GardenConfig.lua:37`). Each new fruit you find goes in the Index. Claiming it pays cash and shards (`IndexRewards.lua:88-98`). Shards buy TNT, which blows up fruit on a plot, and six potions with timed buffs (`GardenConfig.lua:1615-1705`). Once every fruit is discovered you can rebirth: you lose the run (plot, cash, upgrades, storage, Index) and keep a cash bonus of +100% per rebirth (`RebirthLock.lua:35-146`, `GardenConfig.lua:1000`). Up in the sky castle, one player at a time is King. The crown changes hands only by a won sword duel, a won Crown Contest, or the King leaving (`Crown.lua:4-9`). Royal events pay shards and Royal Reputation, never cash (`CrownConfig.lua:15-21`).

In code, one server Script, `GardenService.lua`, loads 17 ModuleScripts in a fixed order. They share state through the table G (`GardenService.lua:88-116`). The server owns every rule. Client scripts only read attributes the server writes, and they ask for actions through remotes; the server may refuse (`UpgradeMenu.lua:4-7`). One DataStore document per player holds the save, with a session lock so two servers never write it at once (`GardenStore.lua:56-63`). The HUD and menus are mostly pre-made images from Samarth's UI kit, filled in by code ([[UI Kit Inventory]]). Separate client scripts draw the world: lighting, the day cycle, music, sky-island life, and the King's Throne screens. The map itself was made by builder scripts that someone runs by hand in Studio. The game never runs them (`ForestBuilder.lua:2-4`).

## 2. The subsystem notes

| Note | What it covers |
|---|---|
| [[Core Server - Plots, Spawning, Income]] | The main server script and the plot, spawn, pricing, upgrade and Index modules: claiming a plot, spawning fruit, building fruit, income, the four upgrades, walk speed and invisibility. |
| [[Carrying, Merging, Storage, Rebirth, Tools and Saving]] | What you do with the one item in your hands (pick up, merge, drop, store), Clear, Merge All, tools and potions, rebirth, lockdown, and the whole save system. |
| [[Config and Fruit Data]] | The shared data layer: the 64-fruit ladder, every tuning number and string in GardenConfig, income per rank, Index rewards, toasts and the loading screen. |
| [[Royal System - Server]] | The King's Throne on the server: who holds the crown, sword duels scored by ring-outs, Crown Contests, Royal Decrees, royal saving. |
| [[Client UI - HUD and Menus]] | The 18 client scripts that draw the HUD, the five panels, dialogs, buff timers and the world signs, and how far each follows the image kit. |
| [[Client World, Royal Client, Music and Ambient]] | The King's Throne screens and arena controls, the King's aura, lighting and hazards, music, sky-island life, and the server's always-on loops (Ambient). |
| [[World Builders and Dev Tools]] | The nine Studio builder tools that made the map, and the developer panel used to test saving. |

Also in this folder: [[World Tour]], screenshots of the map today.

## 3. All remotes

Every remote lives in `ReplicatedStorage.GardenRemotes`, except the two dev panel remotes. All are RemoteEvents except `RoyalQuery`, which is a RemoteFunction.

| Name | Type | Direction | Defined in | Used by |
|---|---|---|---|---|
| `Drop` | RemoteEvent | client to server, no payload | `GardenService.lua:59` | Server: `dropFromPlayer` (`GardenService.lua:528`). If a tool is carried, Drop means "use it" (`Carrying.lua:297-305`). Client: Drop and Use buttons (`GardenClient.lua:351-360`). |
| `Store` | RemoteEvent | client to server, no payload | `GardenService.lua:60` | Server: `storeCarried` (`GardenService.lua:533`). Client: Store button (`GardenClient.lua:362-364`). |
| `Equip` | RemoteEvent | client to server, stored fruit name | `GardenService.lua:61` | Server: `equipStored` (`GardenService.lua:534`). Client: inventory Equip (`StoredFruitsMenu.lua:298`). |
| `BuyTool` | RemoteEvent | client to server, tool name | `GardenService.lua:62` | Server: `buyTool` (`GardenService.lua:529`). Client: shop Buy (`ShopMenu.lua:373`). |
| `EquipTool` | RemoteEvent | client to server, tool name | `GardenService.lua:63` | Server: `equipTool` (`GardenService.lua:530`). Client: inventory tool card (`StoredFruitsMenu.lua:439`). |
| `KingBuy` | RemoteEvent | client to server, King aura key | `GardenService.lua:64` | Server: `buyKingEffect` (`GardenService.lua:531`, `Progression.lua:245`). Client: Shop King tab (`ShopMenu.lua:562`). |
| `KingEquip` | RemoteEvent | client to server, King aura key | `GardenService.lua:65` | Server: `equipKingEffect` (`GardenService.lua:532`, `Progression.lua:271`). Client: Shop King tab (`ShopMenu.lua:583`). |
| `Notify` | RemoteEvent | server to client, a warning line | `GardenService.lua:66` | Fired by many server modules, for example `Ambient.lua:296` and the Crown modules through G (`Crown.lua:30-34`). Client: red toast via `GardenToast.Warn` (`GardenClient.lua:415-419`). |
| `Notice` | RemoteEvent | server to client, a good-news line | `GardenService.lua:70` | Client: green toast via `GardenToast.Notice` (`GardenClient.lua:423-427`). |
| `IndexClaim` | RemoteEvent | client to server, fruit name | `GardenService.lua:71` | Server: `claimIndex` (connected `Progression.lua:374`, defined `:345`). Client: Claim button (`IndexMenu.lua:447`). |
| `Income` | RemoteEvent | server to the plot owner, `(multiplier, dt)` | `GardenService.lua:76` | Fired by the income tick (`Ambient.lua:478`). Client: floating "+$" numbers (`EarnPopups.lua:87-88`). |
| `Upgrade` | RemoteEvent | client to server, upgrade key | `GardenService.lua:77` | Server: `buyUpgrade` (connected `Upgrades.lua:345`, defined `:301`). Client: upgrade cards (`UpgradeMenu.lua:328`) and the pen sign, which sends `"plotSize"` (`PenUpgradeSign.lua:230`). |
| `Lock` | RemoteEvent | client to server, no payload | `GardenService.lua:78` | Server: `lockPlot` (`GardenService.lua:546`). Client: `LockButton.lua:48`. |
| `MergeAll` | RemoteEvent | client to server, no payload | `GardenService.lua:80` | Server: `mergeAllOn` (`GardenService.lua:536`). Client: `MergeAllButton.lua:58`. |
| `Rebirth` | RemoteEvent | client to server, no payload | `GardenService.lua:81` | Server: `doRebirth` (`GardenService.lua:537`). Client: rebirth confirm Yes (`RebirthMenu.lua:208`). |
| `Clear` | RemoteEvent | client to server, no payload | `GardenService.lua:83` | Server: `clearLowest` (`GardenService.lua:535`). Client: Clear confirm Yes (`ClearMenu.lua:200`). |
| `RoyalRequest` | RemoteEvent | client to server, `(action, arg)`; action is at most 24 characters | `Crown.lua:53` | Server: the `handlers` table (`Crown.lua:861-880`), which CrownArena, CrownContest and CrownDecree add to. Actions: style, passage, challenge, respond, cancelChallenge, trial, joinTrial, watch, joinContest, leaveContest, decree. Client: `RoyalUI.lua` (for example `:342`, `:615`, `:693`). |
| `RoyalEvent` | RemoteEvent | server to one client or all, `(kind, data)` | `Crown.lua:54` | Fired by all four Crown modules. Client: `handlers[kind]` in `RoyalUI.lua:1301-1498` (29 kinds, including `knockback`, `swing`, `slam`, `duelIntro`). |
| `RoyalQuery` | RemoteFunction | client asks, server answers with a table | `Crown.lua:55-60` | Server: `Crown.lua:927-968`, plus `decrees` from `CrownDecree.lua:345-357`. Questions: history, me, king, power, decrees. Client: `RoyalUI.lua:840-842`, `:893`, `:920`. |
| `RoyalCombat` | RemoteEvent | client to server, `(action, arg)` | `CrownArena.lua:39-44` | Server: `CrownArena.lua:650-661`; King contest actions in `CrownContest.lua:379-432`. Actions: strike, charge, heavy, dash, block, slam, shockwave, guard, summon. Client: `RoyalCombat.lua:68-143`. |
| `DevChannel.Feed` | RemoteEvent, in the dev's own PlayerGui | server to the dev's client, a list of readout sections | `DevPanel.Server.lua:245-247` | Client: `DevPanel.Client.lua`. Sent every 1 s while the panel is open (`DevPanel.Server.lua:272-287`). |
| `DevChannel.Action` | RemoteEvent, in the dev's own PlayerGui | dev's client to server, `(name, value)` | `DevPanel.Server.lua:249-251` | Server: `handleAction` (`DevPanel.Server.lua:197-231`). Client: the nine buttons and F4 (`DevPanel.Client.lua:147-155`, `:262`). |

Inputs that are not remotes but act like them:
- **Fruit prompt.** Each fruit has a ProximityPrompt named `Interact`. `ProximityPromptService.PromptTriggered` sends it to `merge` if you are carrying, else `pickUp` (`GardenService.lua:522-526`).
- **Royal prompts.** Prompts named `RoyalPrompt` with a `Kind` attribute call `handlers["prompt:" .. Kind]`. They are refused over the remote (`Crown.lua:874`, `:972-987`).
- **Gamepass.** `MarketplaceService.PromptGamePassPurchaseFinished` marks Merge All as owned (`GardenService.lua:541-545`).
- **RoyalDev.** A BindableFunction in ServerStorage for royal testing, Studio only (`Crown.lua:1186-1198`).

## 4. The save document

One DataStore, `GardenProfiles`. One key per player, `p_<UserId>`. Schema 1 (`GardenStore.lua:56`, `:62-63`). The stored value is an envelope: `{ v = schema, lock = {owner, placeId, stamp} or nil, data = profile }` (`GardenStore.lua:521-523`, `:621-625`). `Store.Default()` is the one place the profile's shape is declared (`GardenStore.lua:220-283`). `Persistence.captureProfile` builds the profile from the live game, and `Persistence.applyProfile` puts it back. `GardenStore` checks every field on load (the `sanitise` step).

| Field | What it means | Live owner (module) | Captured | Restored | Checked in sanitise |
|---|---|---|---|---|---|
| `cash` | Cash balance. | `balances[player]`, a local table in `GardenService.lua:122`. Income adds (`Ambient.lua:468`); upgrades spend (`Upgrades.lua:313-321`). | `Persistence.lua:49` | `Persistence.lua:197` | `GardenStore.lua:318-326` |
| `best` | Name of the best fruit merged this run. | `bestRank`, set by `recordMerge` (`Progression.lua:129-140`). | `Persistence.lua:51-53` | `Persistence.lua:207-213` | `GardenStore.lua:328-332` |
| `stored` | Storage bag: `{fruit name = count}`. | `StoredFruits` IntValues on the player (`Tools.lua:24-49`); Storage module. | `Persistence.lua:55-59` | `Persistence.lua:215-219` | `GardenStore.lua:334` |
| `plot` | Fruit on your plot as `{fruit name = count}`. Counts only, no positions. | Fruit in `SpawnedFruits` tagged with your `PlotName`, plus the pending-merge ledger, plus an orphan carried fruit (`Persistence.lua:96-116`). | `Persistence.lua:96-116` | `restorePlot`; overflow goes to storage (`Persistence.lua:308-314`) | `GardenStore.lua:335` |
| `index` | `{fruit name = 0 or 1}`: 0 discovered, 1 claimed, missing locked. | `IndexState` IntValues (`Progression.lua:160-185`). | `Persistence.lua:69-73` | `Persistence.lua:227-236` | `GardenStore.lua:343-354` |
| `tools` | `{tool name = count}`. | `StoredTools` IntValues; Tools module. | `Persistence.lua:63-67` | `Persistence.lua:221-225` | `GardenStore.lua:358-380` |
| `kingOwned` | King auras owned: `{key = true}`. | `KingOwned` attribute (`Progression.lua:234-243`). | `Persistence.lua:77` | `Persistence.lua:268` | `GardenStore.lua:389-399` |
| `kingEquipped` | The equipped King aura key. Default "Crown". | `KingEffect` attribute. | `Persistence.lua:78` | `Persistence.lua:268` | `GardenStore.lua:400-406` |
| `royal` | Royal record: `rep`, `crowns`, `defenses`, `duelWins`, `contestWins`, `longestReign`, `totalReign`, `decrees`, `bestStreak`, `spars`, `achievements`, `style`. | Crown module, through `G.royalCapture` / `G.royalApply` (`Crown.lua:252-285`). | `Persistence.lua:81` | `Persistence.lua:269` | `GardenStore.lua:306-307`, `:410-438` |
| `shards` | Shard balance. | `Shards` attribute via `setShards` (`Progression.lua:216-218`). | `Persistence.lua:74` | `Persistence.lua:237` | `GardenStore.lua:440-445` |
| `rebirths` | Rebirth count. | `Rebirths` attribute via `publishRebirths` (`Progression.lua:95-101`). | `Persistence.lua:75` | `Persistence.lua:267` | `GardenStore.lua:451-456` |
| `lastSeen` | `os.time()` at save. The only input to offline earnings. | Persistence. | `Persistence.lua:87` | `Persistence.lua:324` | `GardenStore.lua:462-467` (clamped to now) |
| `upgrades` | `{capacity, spawnRate, spawnTier, plotSize}` levels. | `Upgrade_<key>` attributes (`Upgrades.lua:294-299`). | `Persistence.lua:92-94` | `Persistence.lua:270`; plot size applied `:276-278` | `GardenStore.lua:475-497` (clamped to `Config.MaxLevel`) |
| `savedAt`, `saveCount` | Debug numbers. | GardenStore. | `GardenStore.lua:555-556` | `GardenStore.lua:777` | `GardenStore.lua:502-503` |
| `refundShards` | Load-only. Pays refunds for removed items; never written back. | Persistence. | never | `Persistence.lua:243-253` | `GardenStore.lua:501` |

- **Retired fields.** `upgrades.fruitStand`, `tools.PlotAlarm` and `alarm` are read only to pay a refund, then dropped. `ripening` is ignored (`GardenStore.lua:73-76`, `:337-339`, `:368-371`, `:384`, `:489-494`).
- **Not saved.** Potion effects (`GardenService.lua:177-180`). The Clear, Merge All and Lock cooldowns and the gamepass cache (`GardenService.lua:724-728`). Fruit positions, rotations and the `Lucky` flag, because the plot is counts only (`Persistence.lua:98`). The client's `MusicMuted` (`MuteButton.lua:10-13`).
- **Separate global stores, not in the profile.** `CrownHistory` (key `"eternal"`) and the ordered store `CrownRecords` (UserId to longest reign), scope `"studio"` or `"live"` (`Crown.lua:602-606`, `:663-677`).
- **Rules for a new field.** Add it to `Store.Default`, to `sanitise`, to `captureProfile` and to `applyProfile`. `sanitise` copies only the keys it names, so a field missing there saves once and is dropped on the next load (`GardenStore.lua:447-450`). Adding a field needs no schema bump; bumping the schema kicks players whose document is newer than their server (`GardenStore.lua:8-12`, `:764-772`). The self-test asserts the worst payload is under 8192 bytes (`GardenStore.lua:1209`). Names in `stored`, `plot` and `index` must be in `FruitIndex.ByName` or they are silently dropped (`GardenStore.lua:289`, `:345`; `Persistence.lua:56`, `:70`).

## 5. Attributes

"Set by" is the code that writes it. "Read by" is the code that uses it. Royal and builder attributes are grouped; the subsystem notes list every one.

### Player: economy and progress

| Attribute | Set by | Read by |
|---|---|---|
| `PlotName` | `claimPlot` (`GardenService.lua:437`); cleared on release (`:478`) | `GardenClient.lua:266`, `ClearMenu.lua:107`, `EarnPopups.lua:89`, `PenUpgradeSign.lua:254` |
| `Cash` | `publishCash` (`Progression.lua:82`) | `CashClient.lua:64`, `UpgradeMenu.lua:73`, `AlertBadges.lua:57`, `PenUpgradeSign.lua:202` |
| `CashPerSecond` | Income tick (`Ambient.lua:469`); zeroed on rebirth (`RebirthLock.lua:110`) | `Money/s` leaderstat (`Stats.lua:88-92`), `TopEarnersBoard.lua:188`, Empire Power for the throne (`Crown.lua:296-298`), Empire Build decree (`CrownDecree.lua:258-262`) |
| `CashMultiplier` | `publishMultiplier` (`Progression.lua:341-343`) | `CashClient.lua:107` |
| `FriendCount` | `Progression.lua:122` | `CashClient.lua:89` |
| `Rebirths` | `publishRebirths` (`Progression.lua:95-101`) | `rebirthMultiplier` (`Progression.lua:325`), `RebirthMenu.lua:84`, throne eligibility (`Crown.lua:316`) |
| `BestMerge` | `Progression.lua:133`; cleared on rebirth (`RebirthLock.lua:103`) | no client reader found |
| `Shards` | `setShards` (`Progression.lua:216-218`) | `CashClient.lua:65`, `Tools.lua:100`, `Crown.lua:245` |
| `KingOwned` (comma list), `KingEffect` (one key) | `Progression.lua:241-242` | Shop King tab (`ShopMenu.lua:465`, `:471`), King aura (`KingClient.lua:207`) |
| `Upgrade_capacity`, `Upgrade_spawnRate`, `Upgrade_spawnTier`, `Upgrade_plotSize` | `publishUpgrades` and `buyUpgrade` (`Upgrades.lua:294-299`, `:320`) | `levelOf` (`Upgrades.lua:34-37`), `UpgradeMenu.lua:54`, `AlertBadges.lua:60`, `PenUpgradeSign.lua:175` |
| `ProfileLoaded` | `GardenStore.lua:800` | `LoadingScreen.lua:388`, `CashClient.lua:76`, `OfflinePopup.lua:109`, `Crown.lua:315` |
| `OfflineEarned`, `OfflineAway`, `OfflineCounted` | `Persistence.lua:203-205`, `:328-330` | `OfflinePopup.lua:113-114` |
| `DevSetCash`, `DevSetShards`, `DevSetPlotSize` | set by hand in Studio | one-shot requests, Studio only (`GardenService.lua:631-684`) |

### Player: carrying, cooldowns and potions

| Attribute | Set by | Read by |
|---|---|---|
| `CarryingFruit` | `holdFruit` (`Carrying.lua:142`). Also holds a tool's name while a tool is carried. | `GardenClient.lua:211`, `StoredFruitsMenu.lua:294`, `DevPanel.lua:143` |
| `CarryingTool` | `Tools.lua:158`; cleared `:177`, `:198`, `:276` | `GardenClient.lua:225` |
| `ClearReadyAt` | `Merging.lua:425` | `ClearMenu.lua:146` |
| `MergeAllReadyAt` | `Merging.lua:572` | `MergeAllButton.lua:49` |
| `LockEndsAt`, `LockReadyAt` | `RebirthLock.lua:238-239`; `LockEndsAt` cleared by the lock sweep (`Ambient.lua:506`) | `LockButton.lua:36-42` |
| `<Effect>EndsAt`, `<Effect>Span` (Speed, Jump, Aether, Invisible, FeatherFalling, Luck) | `consumeTool` (`Tools.lua:217`, `:223`); Royal Race gives Aether (`CrownDecree.lua:125-126`); cleared on death and expiry (`GardenService.lua:560-563`, `:803-804`) | Buff stack (`GardenClient.lua:562`, `:579`), `FeatherFall.lua:26`, `PlayerVision.lua:157` (Aether), `RoyalUI.lua:973` and `KingClient.lua:180` (Invisible) |
| `FellHard` | Ambient after a deadly fall (`Ambient.lua:295`) | `PlayerVision.lua:209-211` |

### Player: royal

| Attribute | Set by | Read by |
|---|---|---|
| `RoyalRep`, `RoyalTitle`, `RoyalCrowns`, `RoyalDefenses`, `RoyalDuelWins`, `RoyalContestWins`, `RoyalLongest`, `RoyalTotal`, `RoyalStreak`, `CrownStyle`, `RoyalUnlocked`, `RoyalAchievements` | `Crown.lua:144-167` | RoyalUI |
| `RoyalBusy`, `IsKing`, `JoinedAt` | `Crown.lua:345`, `:712`, `:1047` | RoyalUI, Crown |
| `RoyalEligible`, `RoyalWhy` | once a second (`Crown.lua:1172-1173`) | CHALLENGE button (`RoyalUI.lua:425`, `:430`) |
| `InArena` | `enterArena` / `leaveArena` (`CrownArena.lua:269`, `:288`) | No invisibility (`GardenService.lua:266`), no feather fall (`Ambient.lua:109`, `:281`; `FeatherFall.lua:27`), `RoyalUI.lua:425`, `RoyalCombat.lua:42` |
| `Stagger`, `Blocking`, `CrownGuard`, `Underdog` | `CrownArena.lua:132`, `:618`, `:672` | `RoyalUI.lua:1031-1037`, `RoyalCombat.lua:57` |
| `CrownPower` | `CrownContest.lua:261`, `:279`, `:476` | RoyalUI contest card |

### Player: client-only (never reach the server)

| Attribute | Set by | Read by |
|---|---|---|
| `MusicMuted` | `MuteButton.lua:37` | `MusicPlayer.lua:121`, `:238` |
| `VisionHazard`, `VisionSky`, `VisionForest` | `PlayerVision.lua:376`, `:386`, `:395` | `MusicPlayer.lua:242-244` |

### Character

| Attribute | Set by | Read by |
|---|---|---|
| `BaseHipHeight` | `GardenService.lua:585` | `applyWalkSpeed` (`GardenService.lua:205`, `:243`) |
| `CloudSink` (0 to 1) | `Ambient.lua:339-341` | `applyWalkSpeed` (`GardenService.lua:243-246`), `SkyIslandLife.lua:370` |
| `OnSkyGround`, `FallPeak` | `Ambient.lua:243-247`, `:271`, `:284-290`; royal teleport (`Crown.lua:371-376`) | Ambient fall and sky checks |
| `Ragdolled`, `SkyLegs`, `InDeepWater` | `Ambient.lua:125-126`, `:309-312`, `:372-377` | Ambient; `SkyLegs` makes `applyWalkSpeed` re-read |
| `PreInvis`, `PreInvisOn`, `PreInvisShadow` (parts), `PreInvisNames` (Humanoid) | `applyVisibility` (`GardenService.lua:293-351`) | `applyVisibility` |

### Fruit, plot, workspace

| Attribute | On | Set by | Read by |
|---|---|---|---|
| `PlotName` | fruit | `placeFruit` (`FruitFactory.lua:362`), drop (`Carrying.lua:216`); cleared before destroy (`Merging.lua:193`, `:502`; `Tools.lua:286`) | `fruitOfPlot` (`FruitFactory.lua:29`), `GardenClient.lua:271`, `ClearMenu.lua:115`, `EarnPopups.lua:94`, dev panel (`DevPanel.lua:38-43`) |
| `HeldBy` | fruit | `Carrying.lua:198`, cleared `:209`; `Storage.lua:80` | `Merging.lua:406`, `ClearMenu.lua:116`, `EarnPopups.lua:95` |
| `Busy` | fruit | `Carrying.lua:242`, `:278`; `Merging.lua:112`, `:134` | `pickUp` (`Carrying.lua:169`), `merge` (`Merging.lua:167`), TNT (`Tools.lua:282`), `GardenClient.lua:295` |
| `Lucky` | fruit | `markLucky` (`Upgrades.lua:384`) | pick-up discovery (`Carrying.lua:185`) |
| `OwnerUserId`, `OwnerName` | plot model | `GardenService.lua:435-436`; cleared `:459-460` | |
| `LockedUntil` | plot model | `RebirthLock.lua:233`; cleared `GardenService.lua:470`, `Ambient.lua:505` | cosmetic only; the server reads the Lua field `plot.lockedUntil` |
| `PlotSizeLevel` | plot model | `Upgrades.lua:259` | |
| `KingUserId`, `KingName` | workspace | `publishCrown` (`Crown.lua:694-695`); a legacy copy in `Progression.lua:46-47`, `:71-72` runs only if Crown is missing | `KingClient.lua:199` |
| `StudioNoSave` | workspace | set by hand in Studio | skips every save write (`GardenStore.lua:572`) |

### Other groups (see the notes for the full lists)

- **`ReplicatedStorage.RoyalState`** (a Configuration object) carries about 60 attributes for the throne HUD: King, duel, gate, queue, trial, contest and decree state. Written by `Crown.lua:696-710`, `CrownArena.lua:678-697`, `CrownContest.lua:50`, `:66-89`, `CrownDecree.lua:41-56`. Read by RoyalUI, `RoyalCombat.lua:43-45` and `MusicPlayer.lua:229-230` (`Phase`). See [[Royal System - Server]].
- **Value folders on the Player.** `IndexState`, `StoredFruits` and `StoredTools`, one IntValue per name (`GardenConfig.lua:390`, `:528`, `:1604`). The server writes them; IndexMenu, StoredFruitsMenu, ShopMenu, GardenClient, RebirthMenu and AlertBadges read them.
- **Builder-made attributes and tags.** Sky island sizes on the Cloud Garden model, bob and spin on `DriftPiece`, ring radii on the `ArenaCenter` marker, `Kind` on each `RoyalPrompt`, `RoyalSeat` on the throne, and the tags `RoyalBanner`, `RoyalBoard`, `CrownVault` and others. Read by Crown, CrownArena, RoyalUI and SkyIslandLife. See [[World Builders and Dev Tools]] section 4.
- **GUI.** `Animated` on any GuiButton (a set value means ButtonAnimator leaves it alone) and `MagnetOffset` on side HUD buttons (`ButtonAnimator.lua:84`, `:126-129`; `HudScale.lua:119`).

## 6. Where each planned feature hooks in

Each feature merges the extension points from all seven notes into one ordered list. "Suggestion" marks a proposal, not existing code. Conflicts between features are flagged at the end of each section and summed up in 6.11.

### 6.1 Rebirth-gated rarity, +1 storage, harder rebirth, slight boost

The brain dump: at 0 rebirths the best fruit is Apple and merging stops there; each rebirth unlocks the next tier; +1 storage per rebirth; rebirth costs a full Index plus shards plus money; the boost is slight.

Facts that shape it: Apple is rank 60, the top COMMON fruit (`FruitIndex.lua:69`). Spawn Rarity level L grows the bottom L+1 fruit, so Apple is level 4 (`GardenConfig.lua:83-96`). There are 9 tiers, so 8 rebirths reach TRANSCENDENT (`FruitIndex.lua:10-73`). Today nothing knows about rebirths except the cash multiplier.

Changes, in order:
1. **One shared cap helper in GardenConfig.** Suggestion: `Config.MaxTierFor(rebirths)` returning `rebirths + 1`, clamped to 9, plus helpers for the best allowed rank and the top Spawn Rarity level. Put it near the accessors (`GardenConfig.lua:1893-1933`). GardenConfig already requires FruitIndex (`GardenConfig.lua:14`). Server and client both call it, so the rule lives in one place.
2. **Merge cap.** In `merge`, right after `FruitIndex.MergeInto` (`Merging.lua:179`) and before the commit (`:183`), refuse a merge whose result is above the cap. Do the same in the Merge All `cascade` (`Merging.lua:289-346`; its `MergeInto` is at `:315`, its caller at `:483`) by passing a top rank so pairs at the cap stay paired. Do not put the cap in FruitIndex; it is generated and hand edits are lost (`FruitIndex.lua:2-3`).
3. **Spawn cap.** Give `Config.UpgradePrice` an optional cap so it returns nil at the cap. nil already means "maxed" on the server and every client (`GardenConfig.lua:1926-1933`). Keep `Config.MaxLevel("spawnTier")` at 63, because save loading and `publishUpgrades` clamp with it (`GardenStore.lua:482`, `Upgrades.lua:294-298`). `buyUpgrade` then refuses past the cap (`Upgrades.lua:309-311`). Also clamp the level read in `levelOf` or `spawnPlan` (`Upgrades.lua:34-37`, `:372-379`), so an old save above the cap spawns only allowed fruit.
4. **Luck potion.** `spawnPlan` lets Luck go one level up, and tilt at `MaxLevel` (`Upgrades.lua:376-378`). Replace `MaxLevel` with the cap there.
5. **Discovery paths.** Buying a Spawn Rarity level discovers every fruit below it (`Upgrades.lua:327-335`). Load back-fills the same way (`Persistence.lua:280-290`, `discoverThroughTier` at `Progression.lua:190-199`). Pick-up discovers spawnable and Lucky fruit (`Carrying.lua:185-196`). All three must respect the cap.
6. **Index gate.** `indexComplete` counts all 64 fruit (`Progression.lua:311-320`). Change it to "every fruit at or below your cap". Two client copies test the same thing and must change in the same edit: `RebirthMenu.lua:57-65` and `AlertBadges.lua:87-95`. Update the `RebirthNotReady` text (`GardenConfig.lua:1036`).
7. **Shard and cash cost.** Add the checks next to the Index gate (`RebirthLock.lua:52-55`), before `rebirthing[player] = true` (`:57`). Cash is set to 0 by the rebirth anyway (`:109`), so a cash cost is a threshold, not a charge. Subtract shards where the payout is added (`:115`). Keep the block yield-free (`:63-67`). Put the cost numbers in the rebirth section of Config (`GardenConfig.lua:987-1039`).
8. **Show the cost.** The locked overlay is one baked image (`GardenConfig.lua:1017`). Showing "Index n/m, shards needed, cash needed" needs new art or live labels over it. Add Cash and Shards listeners to RebirthMenu (`RebirthMenu.lua:231-252`) and Shards and Rebirths listeners to AlertBadges (`AlertBadges.lua:105-135`).
9. **Slight boost.** Lower `RebirthCashBonus` (`GardenConfig.lua:1000`). Keep it inside `cashMultiplier` (`Progression.lua:332-336`), because the client only learns the multiplier from `rate / base` (`Ambient.lua:466`, `:477-478`). It also changes offline pay (`Persistence.lua:191`). Fix two number formats that print whole numbers only: `RebirthDone` uses `x%d` (`GardenConfig.lua:1037`, `RebirthLock.lua:144-145`) and `RebirthCashText` uses `+%d%%` (`GardenConfig.lua:1032`, `RebirthMenu.lua:88`).
10. **+1 storage per rebirth.** Replace the constant `Config.StoredCap = 5` (`GardenConfig.lua:533`) with a function of `Rebirths`. Three readers use the constant: `Storage.lua:43-45`, `GardenClient.lua:398-400` and `StoredFruitsMenu.lua:466-468`. Add Rebirths listeners to `refreshBadge` (`GardenClient.lua:397-411`) and `refreshCount` (`StoredFruitsMenu.lua:465-471`).
11. **Client display.** IndexMenu needs a fourth state, "rebirth to unlock", beside locked, unclaimed and claimed (`IndexMenu.lua:236-241`, drawing at `:95-188`), and a new Rebirths listener; it watches no attributes today. GardenClient must not offer "Merge" or the green glow at the cap (`GardenClient.lua:273`, `:304-307`). The Spawn Rarity card needs a "rebirth to unlock" state (`UpgradeMenu.lua:57-85`, `Config.UpgradeStat` at `GardenConfig.lua:2011-2015`), and `upgradeWaiting` must skip it (`AlertBadges.lua:56-65`).
12. **King's Throne eligibility.** You need 10 discovered fruit or 1 rebirth to hold or challenge for the crown (`Crown.lua:314-323`, `CrownConfig.lua:38`). Only 5 fruit are at or below Apple (`FruitIndex.lua:69-73`). Lower `C.MinDiscovered` to 5 or less, or tie it to the unlocked fruit.
13. **Old saves.** Existing saves hold Spawn Rarity levels, plot fruit, stored fruit and Index entries far above the new cap, and `restorePlot` puts them all back (`Persistence.lua:124-168`). Decide a grandfathering or clamping rule first. The dev panel's Seed test already writes fruit of ranks 20, 35 and 50 at 0 rebirths, so it is a ready test case (`DevPanel.lua:224-237`).
14. **Dev panel.** Add buttons: set rebirths, add shards, add cash, fill Index. Add readout rows for the rarity cap and the storage cap (`DevPanel.Server.lua:123-137`, `:162-171`). Recipe in [[World Builders and Dev Tools]] section 6.
15. **Retune.** The Spawn Rarity price growth of 2.60 was tuned for one long climb with no resets (`GardenConfig.lua:125-142`). Reaching Apple costs 6,980 cash in total (computed in [[Config and Fruit Data]]).

Risks:
- Missing the Merge All path or the Luck potion lets players jump the gate.
- If `indexComplete` still counts all 64, nobody can ever rebirth.
- The rebirth payout is the shards already claimed from the Index (`RebirthLock.lua:61`, `Progression.lua:296-307`). The whole COMMON tier pays only 31 shards (computed in [[Config and Fruit Data]]). So a shard cost at the first rebirth cannot come from the Index alone.
- Royal fights pay 500 to 3000 shards (`CrownConfig.lua:261-268`). If rebirth costs shards, royal fights become a rebirth input, against the stated rule that fighting must not beat the merge economy (`CrownConfig.lua:256-259`).

Conflicts:
- **Luck vs the cap.** Luck adds one level and tilts at the top (`Upgrades.lua:372-379`). A Lucky fruit enters the Index on pick-up (`Carrying.lua:185`). Without the cap in both places, Luck leaks next-tier fruit into the plot and the Index.
- **Spawn Rarity purchase vs the cap.** Buying a level discovers fruit (`Upgrades.lua:327-335`). An uncapped purchase both wastes cash and fills the Index past the cap.
- **Throne vs the cap.** See step 12.
- **Feature 4.** A 100k-shard base is about 3.8 full Index claims (26,528 shards each, computed). If rebirth also costs shards, the two compete for the same currency.

### 6.2 Bats that upgrade each rebirth

Changes, in order:
1. **Decide where the bat lives.** Any `Config.Tools` entry automatically gets a shop card, an inventory card, a Use button and a saved count (`ShopMenu.lua:378-388`, `StoredFruitsMenu.lua:484-488`, `GardenClient.lua:227`, `Persistence.lua:64`). A free bat probably should not be a `Config.Tools` entry. If it is, add a "not for sale" filter to the shop build loop and a third kind (`GardenConfig.lua:1607-1614`).
2. **Give it its own code path.** In `useTool`, every kind other than `"consume"` runs the TNT code and errors on a missing `radius` (`Tools.lua:250-310`, `:283`). Add a bat branch before the TNT code starts (`:255`). Do not reuse `holdFruit` as is: it welds the item above the head (`Carrying.lua:122-139`). Remember the Drop remote means "use" for any carried tool (`Carrying.lua:297-305`).
3. **Tier and colour from `Rebirths`.** Derive both from the `Rebirths` attribute with the shared cap helper from 6.1, so nothing new is saved. Every change to the count goes through `publishRebirths` (`Progression.lua:95-101`): load (`Persistence.lua:267`), rebirth (`RebirthLock.lua:118`), dev wipe (`DevPanel.lua:167`). Colour from `Config.RarityColors` (`GardenConfig.lua:582-592`), as the fruit name tag does (`FruitFactory.lua:283`). Rebirth already keeps tools (`RebirthLock.lua:85-88`).
4. **Knockback ladder.** Suggestion: a Config list keyed by rebirth count, shaped like `RebirthCashBonus` (`GardenConfig.lua:1000`).
5. **Deliver the push on the victim's client.** The only player push today is the arena's: the server sends `knockback` to the victim (`CrownArena.lua:510-514`) and RoyalUI sets the velocity (`RoyalUI.lua:1482-1490`). Build a garden version that does not depend on RoyalUI, which quits when `CrownConfig.Enabled` is false (`RoyalUI.lua:34`). Never write WalkSpeed directly; `applyWalkSpeed` is the only writer and overwrites it (`GardenService.lua:187-248`).
6. **Skip the arena.** Skip any attacker or victim with `InArena` set (`CrownArena.lua:269`, `:288`). A bat from the stands could knock a fighter off the ring, which scores a point (`CrownArena.lua:1237`, `:1263`). `enterArena` does not remove other tools (`CrownArena.lua:264-280`).
7. **HUD readout.** A bat level line fits in CornerHud by the multiplier (`CashClient.lua:106-116`). A new HUD cluster must go in `Config.HudClusters` (`GardenConfig.lua:1863-1867`).
8. **Dev panel.** A "give bat / set bat level" button, and clear any bat state in `clearPlayerResidentState` (`DevPanel.lua:162-183`).

Risks: a knock off a sky island into a drop of more than 220 studs kills (`Ambient.lua:291-299`). The lock dome blocks only TNT and pushes nobody (`GardenConfig.lua:897-900`), so bats hit players on locked plots unless they get a rule. The existing code never flings players on purpose (`Tools.lua:295`).

Conflicts: bats and explosives both go through `useTool`, which has only two branches today (6.3). The bat tier must use the same cap helper as 6.1.

### 6.3 Explosives that drain fruit health

Changes, in order:
1. **Health curve.** Suggestion: health per rank derived from FruitIndex, in a shared module, the way income is derived (`FruitEconomy.lua:39-50`).
2. **Stamp health on every fruit.** In `createFruit` or `placeFruit` (`FruitFactory.lua:224-357`, `:359-367`). Every spawned or restored fruit passes `placeFruit`. A fruit taken out of storage is built by `createFruit` (`Storage.lua:78`).
3. **Damage instead of destroy.** Replace the destroy loop in `useTool` (`Tools.lua:278-290`) with damage, and destroy only at 0. At 0, keep today's order: clear `PlotName`, then destroy, so the spawner sees the free slot (`Tools.lua:284-287`). Keep skipping Busy fruit (`Tools.lua:282`).
4. **Hit by distance.** Fruit are `CanQuery = false` and `CanTouch = false`, so raycasts and Touched never hit them (`FruitFactory.lua:232-236`). TNT already uses distance (`Tools.lua:279-283`).
5. **TNT tiers and nukes.** More `kind = "place"` entries in `Config.Tools` with `radius` and a new damage field, each with a template of the same name in `ReplicatedStorage.Tools` (`GardenConfig.lua:1601-1626`). The shop sorts by key name (`ShopMenu.lua:382-383`), so add an order field. Give each a `note`, which the carry bar shows (`GardenClient.lua:250`). Reword the strings that assume deletion (`GardenConfig.lua:1855-1857`).
6. **Artillery.** Needs a new targeting path. Today you must stand on the target plot and the blast centres on your own body (`Tools.lua:259-263`, `:272`). A nuke over two plots needs new plot targeting (`PlotStandBand`, `GardenConfig.lua:933-940`).
7. **Health bars.** The NameTag billboard is the natural place (`FruitFactory.lua:256-273`), but the client shows it only for the prompted fruit (`GardenClient.lua:138-159`). Damaged fruit need their own display pass.
8. **Saving.** The plot is saved as name counts (`Persistence.lua:98`), and `sanitiseCounts` accepts only numbers (`GardenStore.lua:285-304`). Health resets to full on rejoin unless a separate saved field is added.
9. **Income and merging.** `plotBase` pays full income whatever the damage (`FruitFactory.lua:39-45`). `merge` compares names only, so a damaged fruit merges like a fresh one (`Merging.lua:168`). Decide both.
10. **Effects.** RoyalUI's ring, burst and shake (`RoyalUI.lua:1078-1137`) are good patterns but are local functions. Suggestion: move them into a shared module.
11. **Leave the arena and knights alone.** Skip `InArena` players and knight models (`RoyalKnight` attribute, `CrownArena.lua:417`). Roblox `Explosion` objects could break knight Motor6Ds and crown Welds (`CrownArena.lua:375-380`, `Crown.lua:477-481`). Today's TNT uses zero blast pressure (`Tools.lua:292-298`).
12. **Dev panel.** A "give TNT tier N" button and a damaged-fruit count row (`DevPanel.Server.lua:141-148`). A health system that swaps a fruit for a copy must copy `PlotName`, or the panel flags it untagged (`DevPanel.lua:38-43`).

Risks: PlayerVision blacks out a death only if the hazard was already above 0.5 (`PlayerVision.lua:230-234`); keep explosion deaths out of that path.

Conflicts:
- **Clear and Merge All ignore Busy.** Only TNT skips Busy fruit. `clearLowest` checks only `HeldBy` (`Merging.lua:406`) and `mergeAllOn` checks neither (`Merging.lua:494-504`). Any per-fruit state (health) must be handled there too.
- **Lockdown.** The dome refuses only the TNT path (`Tools.lua:265-270`). New explosives and artillery must check `plot.lockedUntil` the same way.
- **Feature 6.** Breakable boats need the same health idea and a world-target path.

### 6.4 Underground base (100k shards) and ores

Changes, in order:
1. **The purchase.** `buyUpgrade` always charges cash (`Upgrades.lua:313-320`). `Config.UpgradeCurrency` exists but nothing reads it (`GardenConfig.lua:189-190`). The notes disagree on the best route:
   - [[Core Server - Plots, Spawning, Income]] suggests a new key in `Config.UpgradeKeys` (`GardenConfig.lua:37`), which saves for free, plus a shard branch in `buyUpgrade`.
   - [[Carrying, Merging, Storage, Rebirth, Tools and Saving]] says not to: rebirth zeroes every key in `UpgradeKeys` (`RebirthLock.lua:95` via `Upgrades.lua:294-299`), and sanitise clamps those levels to the cash price ladders (`GardenStore.lua:475-488`).
   - Recommendation: a separate saved field bought the way `buyTool` spends shards (`Tools.lua:91-112`). If `UpgradeKeys` is used anyway, exclude the key from the rebirth reset and from the sanitise clamp.
2. **Save fields.** Add the unlock and the ore inventory to `Store.Default` (`GardenStore.lua:220-283`), `sanitise` (`:309-506`), `captureProfile` and `applyProfile`. No schema bump is needed (`GardenStore.lua:8-12`). Keep the payload under 8192 bytes (`GardenStore.lua:1209`).
3. **Geometry.** One apply function, called from the same places `applyPlotSize` is: purchase (`Upgrades.lua:338`), load (`Persistence.lua:276-278`), release (`GardenService.lua:456`), rebirth if it is wiped (`RebirthLock.lua:97`), and the Studio dev door (`GardenService.lua:657-683`). Do not build fragile geometry inside `claimPlot`: an error there stops the profile loading (`PlotRegistry.lua:87-90`).
4. **Ores in their own folder.** Anything in `SpawnedFruits` with a `PlotName` counts toward capacity (`Upgrades.lua:286-288`), earns 0 in `plotBase` (`FruitEconomy.lua:56-57`), and is routed to fruit pick-up and merge (`GardenService.lua:522-526`). Build an ore factory modelled on `randomSpotIn` and `placeFruit` (`FruitFactory.lua:96-107`, `:359-367`), with its own prompt handler and merge path.
5. **A parallel OreIndex.** Never append ores to FruitIndex. Its `Count` drives the spawn ladder (`GardenConfig.lua:87`), income (`FruitEconomy.lua:40-46`), Index bonus and shards (`IndexRewards.lua:42-50`, `:76-85`) and the rebirth gate (`Progression.lua:311-320`). Suggestion: an OreIndex with the same shape (`List`, `ByName`, `Count`, `Get`, `MergeInto`, `Template`, `FruitIndex.lua:89-99`) and its own income and reward curves. Coal then iron are its first rungs.
6. **Ores Index category.** Use a separate player folder. Ore names in `IndexState` would light the Index badge (`AlertBadges.lua:71-78`), collide with any fruit of the same name (`GardenConfig.lua:386-390`), and count toward the throne's `MinDiscovered` in-session (`Crown.lua:289-292`) but vanish on rejoin, because only fruit names are saved (`Persistence.lua:69-71`). In the UI, copy ShopMenu's tab pattern (`ShopMenu.lua:55-129`); `fitCanvas` counts every child, so keep hidden cells in a separate list (`IndexMenu.lua:571-575`).
7. **Ore storage.** `makeCard` returns nil for non-fruit names (`StoredFruitsMenu.lua:311-313`) and `Config.StoredTotal` ignores them (`GardenConfig.lua:543`). Add an ore card builder like `makeToolCard` (`StoredFruitsMenu.lua:384-443`).
8. **Ore income.** Add it to the income tick (`Ambient.lua:464-479`) and to offline pay (`Persistence.lua:191`). The `Income` remote carries only `(multiplier, dt)` and the client prices fruit itself (`Ambient.lua:473-478`), so ore earnings will not show without a payload change. Income in `CashPerSecond` also raises Empire Power for the throne (`Crown.lua:296-298`).
9. **Shard prices in the UI.** UpgradeMenu, PenUpgradeSign and AlertBadges hard-code "$" and compare against Cash (`UpgradeMenu.lua:73-74`, `PenUpgradeSign.lua:198-206`, `AlertBadges.lua:57-62`).
10. **Vertical checks.** `isOnPlot` checks X and Z only, so a player underground still counts as on the plot for drops (`FruitFactory.lua:63-68`, `Carrying.lua:325`). `standingOn` adds a 12-stud band (`FruitFactory.lua:79-85`), so TNT and lock do not see players deeper than that.
11. **World rules underground.** A drop of more than 220 studs kills (`Ambient.lua:291`). A spot where the 12-stud ground ray hits nothing reads as "water" (`PlayerVision.lua:144`, `Ambient.lua:370`). SaturationPass skips everything below y -500, where asset packs are parked (`SaturationPass.lua:57`, `:95`). Underground lighting must go into PlayerVision's single tint write (`PlayerVision.lua:449-469`). An underground music track is one more biome track (`MusicPlayer.lua:101-111`, `:142-153`).
12. **Dev panel.** An "add 100k shards" button; ore fields in `devSeed` and `clearPlayerResidentState` (`DevPanel.lua:162-183`, `:224-237`).

Conflicts: rebirth wipes only what it lists (`RebirthLock.lua:63-128`), so decide whether the base and ores survive a rebirth. A 100k-shard purchase wiped by rebirth would feel like theft. See also 6.1 (shards).

### 6.5 Forest pets

Changes, in order:
1. **Taming.** Check the carried fruit's name against the request, then use it up with `releaseHold(player, true)`, as storing does (`Storage.lua:49`). The request can come from `bestRank` (`GardenService.lua:122`) or the player's top tier in FruitIndex (`FruitIndex.lua:9-74`).
2. **Prompts.** Give pets their own prompt handler beside the fruit router (`GardenService.lua:522-526`). GardenClient's `PromptShown` handler runs for every ProximityPrompt in the game: it outlines the part and looks up `NameTag.Label` (`GardenClient.lua:124-167`, `:216`). A pet child named `NameTag` without a `Label` makes it error (`GardenClient.lua:148-151`). Filter by prompt parent.
3. **Merging pets.** Copy `merge`'s ownership check, Busy flag and pending ledger, on a separate pet ladder (`Merging.lua:153-214`).
4. **Saving.** A new `pets` field in `Store.Default`, `sanitise`, `captureProfile` and `applyProfile` (see section 4). Keep pets out of FruitIndex or save loading drops them (`GardenStore.lua:289`). Decide whether rebirth wipes pets; today it wipes only what it lists.
5. **Income.** Add a pet term inside or beside `plotBase` (`FruitFactory.lua:39-45`), so live income (`Ambient.lua:465-466`) and offline income (`Persistence.lua:191`) use one formula. Scale it with the player's stage, for example `Economy.ForRank` of the best rank (`FruitEconomy.lua:52-54`); flat numbers go stale within a few ranks. Decide the offline rule: permanent account properties pay offline, per-session ones do not (`GardenConfig.lua:756-759`). Pet income needs its own popup path (`EarnPopups.lua:93-106`) and an Income payload change (`Ambient.lua:473-478`).
6. **Wandering.** Equipped pets: use the soil bounds (`isOnPlot`, `randomSpotIn`, `FruitFactory.lua:63-68`, `:96-107`) and keep them out of `SpawnedFruits`. Wild pets: ForestBuilder made clearings "for the animals later" with `ClearingNN` markers that carry a `Radius` (`ForestBuilder.lua:132`, `:380-404`). Check the folder exists in the live place. Bushes, logs and rocks collide (`ForestBuilder.lua:309-317`), so pets need pathfinding or must stay in clearings. The knight NPCs are a working pattern: server-owned rig, `Humanoid:MoveTo` thinking (`CrownArena.lua:336-460`, `:1179-1224`).
7. **UI.** A Pets panel must be added to all five hand-written panel exclusion lists (`ShopMenu.lua:617`, `IndexMenu.lua:47`, `StoredFruitsMenu.lua:541`, `UpgradeMenu.lua:421`, `RebirthMenu.lua:51`), to `Config.HudClusters` (`GardenConfig.lua:1863-1867`) and, for a side button, to `MAGNETIC` (`ButtonAnimator.lua:52-55`). Name it `...Menu` directly under GardenUI so the loading screen pre-draws it (`LoadingScreen.lua:411-416`).

Risks: the arena's "nobody else on the ring" sweep moves only Players, so a pet following its owner in would stay (`CrownArena.lua:1291-1303`). Highlights are capped at 24 and shared with merge glows (`GardenConfig.lua:1887-1889`).

Conflicts: pet income in `CashPerSecond` raises Empire Power, which decides who claims an empty throne (`Crown.lua:296-298`). The "stage" should come from the same cap helper as 6.1.

### 6.6 Boats, swimming fatigue, no currents, Aqua Affinity

Changes, in order:
1. **Remove the currents (place edit).** They are saved parts, not scripts: outward `CurrentPlate` plates at 42 studs/s and `ReturnPlate` lanes at 48 studs/s (`GardenConfig.lua:1310`, `:1313-1319`).
2. **Remove the currents (code).** Delete PlayerVision's airborne pushes (`PlayerVision.lua:266-297`) and the `"return"` ground kind (`:149-151`). If only the plates go, a jump in the sea is still pushed outward at up to 42 studs/s. Retire `CurrentSpeed`, `CurrentAirAccel`, `ReturnSpeed`, `ReturnFloors`, `ReturnAirAccel` (`GardenConfig.lua:1310-1324`). Ambient's drown comment goes stale (`Ambient.lua:26-33`). After this, only `Seabed` marks the sea floor (`GardenConfig.lua:1292-1298`, `:1327`).
3. **Swimming fatigue.** No swimming code exists anywhere today. Suggestion: a fatigue value as a character attribute, updated in the drown sweep that already sets `InDeepWater` every 0.2 s (`Ambient.lua:359-375`), on the `CloudSink` pattern (`Ambient.lua:337-341`). `applyWalkSpeed` reads it after the arena override (`GardenService.lua:200-208`). `applyWalkSpeed` runs only on events, so call it when a player enters or leaves water. The drowning view keys can be reused for the warning look (`GardenConfig.lua:1546-1584`). A fatigue meter is not a timed effect, so it needs its own HUD widget in `Config.HudClusters` (`GardenClient.lua:559-563`).
4. **Aqua Affinity potion.** Follow the potion recipe in 6.7. Its reader goes in `applyWalkSpeed`. For a client-side movement change, copy FeatherFall, which reads `<Effect>EndsAt` (`FeatherFall.lua:25-28`). Check it on the server with `effectActive(player, "AquaAffinity")` (`Ambient.lua:281`).
5. **Boats.** Not a `Config.Tools` "place" tool: that kind needs a plot under you (`GardenConfig.lua:1852-1854`), `equipTool` would weld it above the head (`Tools.lua:155`), and `useTool` would run the TNT code on it (`Tools.lua:250-255`). Give boats their own 1000-shard purchase (spend like `Tools.lua:100-107`), a saved field, two seats, and health (6.3).
6. **Boat speed.** Moving faster than `DrownSweptSpeed` 30 marks a rider as "water" and `InDeepWater` (`PlayerVision.lua:260-265`, `Ambient.lua:363-364`). Raise it, or exempt boat decks by name the way `ReturnFloors` is exempted. A Speed potion walker already hits 32 (`GardenConfig.lua:943`, `:1721`, `:1291`).
7. **Lanes.** About 692 collidable stones, logs and islets fill the sea from r 381 outward (`ShallowsBuilder.lua:7-9`, `:60`, `:212`). Driftwood sits with its centre at -1.95, right at the ocean top of -2.2 (`ShallowsBuilder.lua:109-111`). The forest beach (r 1040 to 1072) has collidable logs and rocks (`ForestBuilder.lua:19`, `:310`). Plan lanes and docks.

Risks: the forest music still works without the current, because it is measured by distance (`PlayerVision.lua:330-335`). Royal teleports set `Sit = false`, so a player in a boat is lifted out when called to a duel (`Crown.lua:370`); the throne hook only acts on `RoyalSeat` seats (`Crown.lua:1023`).

Conflicts: the Shallows pieces that block boats are the same pieces feature 8 might restyle or replace (6.8). Breaking boats with artillery needs the world-target path from 6.3.

### 6.7 More potions

The recipe has three parts:
1. A `Config.Tools` entry: `display`, `kind = "consume"`, `price`, `effect`, `duration`; no `carrySize` for bottles; optional `image` for flat shop art (`GardenConfig.lua:1615-1705`, `:1628-1630`, `:1661-1665`).
2. A `Config.Effects` entry: `display`, `bar` colour, `icon`, and its stat fields (`GardenConfig.lua:1714-1789`).
3. A template part of the same name in `ReplicatedStorage.Tools` (`Tools.lua:133-139`, `GardenConfig.lua:1601-1605`).

What happens on its own: the save keeps the tool (`Persistence.lua:64`), the shop and inventory get cards (`ShopMenu.lua:381-386`, `StoredFruitsMenu.lua:484-488`), the carry row shows Use and "Effect 5 min" (`GardenClient.lua:241-247`), the buff stack gets a row (`GardenClient.lua:549-557`), `<Effect>EndsAt` and `<Effect>Span` are written (`Tools.lua:217`, `:223`), expiry and death-clearing work (`GardenService.lua:554-570`, `:793-821`), and an `rbxassetid://` icon is preloaded (`LoadingScreen.lua:338`).

What you must add: a **reader**, or the potion only shows a bar.
- Movement: `applyWalkSpeed` (`GardenService.lua:191-248`; Speed and Jump at `:210-233`).
- Visibility: `applyVisibility` (`GardenService.lua:258-387`).
- Spawn luck or rate: `spawnPlan` (`Upgrades.lua:372-379`) or the spawn interval (`Ambient.lua:422-425`).
- Income: `cashMultiplier` (`Progression.lua:332-336`).
- Client-side movement: copy `FeatherFall.lua:25-41`.
- Every reader must repeat the `InArena` exemption (`GardenService.lua:266`, `Ambient.lua:109`, `:281`, `FeatherFall.lua:27`), or the potion leaks into duels.

Risks:
- Effects are not saved and are lost on death (`GardenService.lua:177-180`, `:601-604`). Stacking has no cap (`Tools.lua:200-214`).
- A missing `Config.Effects` entry only warns (`Tools.lua:187-189`). A missing template skips the shop card with a warning (`ShopMenu.lua:198-203`) and the inventory card silently (`StoredFruitsMenu.lua:387-388`).
- Removing a potion later deletes everyone's stock on load with no refund, unless it is added to `RETIRED_PRICES` (`GardenStore.lua:372-375`, `:73-76`).
- A cash potion inside `cashMultiplier` shows on the HUD multiplier, but must not reach offline pay, which builds its own formula (`Persistence.lua:191`).
- A spawn-speed potion must not write `G.spawnBoost`; every decree finish resets it to 1 (`CrownDecree.lua:62`).
- Each active effect adds 85 px to the buff stack, which grows toward the Clear button column (`GardenClient.lua:473-474`). Feather Falling and Luck icons are still placeholders (`GardenConfig.lua:1770`, `:1784`).
- The dev panel's `clearPlayerResidentState` does not reset potion timers, so Round trip tests can pass on stale state (`DevPanel.lua:162-183`).

Conflict: a new Luck-like potion meets the same rebirth cap as 6.1.

### 6.8 Plot overhaul, studded logs, studded lily pads, border mountains

Plots:
1. **Keep one flat soil part.** The soil is the BasePart with the biggest X times Z area in the plot model (`PlotRegistry.lua:21-30`). The pen sign uses the same rule on the client (`PenUpgradeSign.lua:238-247`). Any bigger decoration becomes the soil: fruit spawn on it and resizes stretch it.
2. **Naming.** `buildFence` deletes every child named `Fence` or `FenceFront`, and every BasePart named `Part` that is not the soil, on the first resize (`Upgrades.lua:168-173`). Name new decorations.
3. **Fence style.** Rebuilt fences clone the majority Post, Rail and Kerb across all plots (`Upgrades.lua:84-112`). Restyle the authored fences in Studio. If the fence shape changes, update the shape constants (`Upgrades.lua:61-64`).
4. **Size.** Fit inside the level-4 footprint of 48 x 78 (`GardenConfig.lua:98-123`). If the plot model changes size, re-measure `PlotSizes[0]`; a wrong level 0 makes the first purchase nearly free and makes rebirth shrink plots (`GardenConfig.lua:109-112`, `:117-118`).
5. **Placement assumptions.** The spawn pad and fence assume the plaza is at world origin (`PlotRegistry.lua:115`, `Upgrades.lua:143-145`). The lock dome overhangs the soil by 35 and 22 studs and sits on `plot.topY` (`RebirthLock.lua:171-180`); drops land on `topY` (`Carrying.lua:228`). The pen sign sits 6.5 out and 13 to the side (`GardenConfig.lua:2095-2099`) and trusts its ground ray only within 1.5 studs of Y = 0 (`PenUpgradeSign.lua:286-289`).
6. **Colour.** Run `SaturationPass.apply()` after the edit, or new parts look duller than the rest (`Upgrades.lua:66-70`, `ShallowsBuilder.lua:404-407`).
7. **Keep these.** The `Royal Gate` model with `Portal`, `Bars` and `RoyalPrompt`, on the road between Plot0 and Plot11 (`CrownArena.lua:708-731`, `CrownConfig.lua:351-353`). The Top Earners board path `Main Island.TopEarnersBoard.Leaderboard`, or that script waits forever (`TopEarnersBoard.lua:30-32`). The Crown Hunt decree raycasts for spots 60 to 320 studs from origin at Y -5 to 60; tall decor there makes it fail after 40 misses (`CrownDecree.lua:165-184`).

Logs:
8. **Swap in place, do not rebuild.** Re-running `buildSector` re-rolls the trees (`ForestBuilder.lua:215-222`). Write a one-off script that replaces each log, keeping pivot and scale. Forest logs keep their template names under `Forest.Undergrowth` (`ForestBuilder.lua:122-123`, `:43-56`). Sea logs have `Role = "Driftwood"` under `Shallows.Driftwood` (`ShallowsBuilder.lua:87`, `:132`). Put the studded template in both `LOGS` lists, then run `F.finish()` and `B.finish()` for the colour lift (`ForestBuilder.lua:408-427`, `ShallowsBuilder.lua:426-464`).

Lily pads:
9. **Ask which pads.** The sea's lily pads were already replaced on 28 Sep by stones, driftwood and islets (`GardenConfig.lua:1283-1284`, `ShallowsBuilder.lua:7-9`). The originals sit in `ServerStorage.LilyPads_backup_2026-09-28` (`ShallowsBuilder.lua:226-231`). "Studded lily pads" can mean restoring restyled pads or studding the stones. Restored pads would overlap the pieces that replaced them (`ShallowsBuilder.lua:326`). If the stone field changes, re-measure the forest music start at r 642 (`GardenConfig.lua:1455`, `:1550-1555`).

Mountains:
10. **No builder makes them.** Keep the forest's invisible boundary wall at r 1470, or move the drown radii (`GardenConfig.lua:1263-1269`). Name no part `CurrentPlate`, `Seabed` or `ReturnPlate` (`PlayerVision.lua:141-152`). ForestBuilder's ground ray sees only terrain, so add new mountain models to its ray filter before rebuilding ridge sectors (`ForestBuilder.lua:59-62`).

Studs and decor:
11. **Pick one stud method per area.** The main map uses a MaterialVariant named `Studs`. The sky builders use `SurfaceType.Studs` (`Upgrades.lua:68-69`, `SkyIslandBuilder.lua:155`). SkyIslandLife finds decor by name and colour (`SkyIslandLife.lua:120-122`, `:151-154`, `:179-195`), so a re-skin that changes them silently stops the sway, flags and lamps. Moving water needs the `IslandWaterFlow` tag with `FlowU` / `FlowV` (`IslandWater.lua:20`, `:43-44`).

Conflict: the Shallows pieces are also the boat obstacles in 6.6.

### 6.9 UI overhaul to the image kit (FredokaOne, baked title images, no emoji)

Changes, in order:
1. **King and Royal screens first.** RoyalUI builds everything through three kit functions, `frame()`, `label()` and `button()` (`RoyalUI.lua:134-289`). Rewrite those and every caller changes at once. The font is already FredokaOne (`RoyalUI.lua:43`, `RoyalCombat.lua:36`).
2. **Icons.** Every `C.Icons` entry is 0, which means "no image yet, keep the emoji" (`CrownConfig.lua:385-397`). Fill them with kit image ids. `dropEmoji` strips only a leading emoji, and only when an icon exists (`RoyalUI.lua:222-224`).
3. **Emoji in client code.** The full line list is in [[Client World, Royal Client, Music and Ambient]] (section 6, item 9): about 50 sites in RoyalUI and 7 in RoyalCombat (`RoyalCombat.lua:259-264`, `:343`). Trailing emoji, such as the swords after a name, are never stripped (`RoyalUI.lua:583`).
4. **Emoji in data and server text.** A client-only reskin still shows these: most `C.Text` lines (`CrownConfig.lua:417-477`), decree `icon` fields (`CrownConfig.lua:213`, `:222`, `:231`, `:240`, `:248`), strings built in server code (`Crown.lua:207`, `:211`, `:228`, `:232`, `:1090-1117`; `CrownArena.lua:1039`; `CrownDecree.lua:93`, `:130`, `:153`, `:249`), and the Royal Capital board default texts, which need a builder re-run (`RoyalCapitalBuilder.lua:717` and others listed in [[World Builders and Dev Tools]]).
5. **Garden screens to convert.** These are still drawn in code: the King tab Equip pill (`ShopMenu.lua:567-583`), the inventory Equip overlay (`StoredFruitsMenu.lua:203-276`), the storage badge (`GardenClient.lua:369-395`), the buff bars (`GardenClient.lua:494-522`), the Index "???" pane (`IndexMenu.lua:105-125`), the Top Earners board with its LuckiestGuy title (`TopEarnersBoard.lua:36-183`, `GardenConfig.lua:2059`) and the pen sign (`PenUpgradeSign.lua:90-170`). Already kit-compliant: IndexMenu, RebirthMenu, ClearMenu, OfflinePopup, UpgradeMenu, CornerHud, TopButtons.
6. **King aura cards.** They use tinted Roblox particle sprites, not kit art (`GardenConfig.lua:1252-1259`).
7. **Kit rules.** Store image ids as `"rbxassetid://..."` strings so the loading screen preloads them (`LoadingScreen.lua:333-338`). Name new panels `...Menu` under GardenUI (`LoadingScreen.lua:411-416`). Add new HUD clusters to `Config.HudClusters` (`GardenConfig.lua:1859-1867`). Rescale strokes against panel width, as ClearMenu does (`ClearMenu.lua:179-193`). Use `Font.new("rbxasset://fonts/families/FredokaOne.json")` (`ShopMenu.lua:44-47`). Fruit name tags use Highway on purpose (`GardenConfig.lua:20-25`, `FruitFactory.lua:279`).
8. **Contracts to keep.** `announce` events carry a `style` of royal, alarm, crown, plain or decree, and the `decrees` query sends `icon` (`CrownDecree.lua:285`, `:349`); change both sides together. Board labels are filled by line name (`Crown.lua:592-596`); a baked title image must use a new name, because an ImageLabel has no `Text`.
9. **Out of scope.** Toast wording is all Config keys (`GardenService.lua:66-70`). The dev panel stays as it is (`DevPanel.Client.lua:69`).

Risks: ButtonAnimator adopts every new GuiButton; full-screen helper buttons must set `Animated` before parenting (`ButtonAnimator.lua:213-217`, `ClearMenu.lua:82-83`). Non-emoji, non-ASCII punctuation (middle dot, em dash, multiplication sign) appears in RoyalUI and RoyalCombat; check the kit font draws it (`RoyalUI.lua:427`, `RoyalCombat.lua:197`). The top-left royal stack once covered the Inventory button (`RoyalUI.lua:348-351`).

Conflict: re-running RoyalCapitalBuilder for the board texts is part of the builder chain (see gotcha G59).

### 6.10 Royal combat: animations, slashes, health drain, better Slam, weapon skins

Changes, in order:
1. **Real animations.** Today players get the default `toolanim` Slash and Lunge strings (`CrownArena.lua:229-238`). Knights play one track for everything (`CrownArena.lua:239-241`, `:323`). Replace `animate()` with Animator tracks. Animations must be published to the place owner's account; a runtime-registered one works in Studio and silently fails live (`GardenConfig.lua:338-345`).
2. **Slash effects.** Hook them on `handlers.swing`, which already gets the attacker id and the heavy flag (`RoyalUI.lua:1417-1421`). Today it only plays a sound.
3. **Better Royal Slam.** `handlers.slamStart` is empty (`RoyalUI.lua:1432`); fill the 0.28 s windup (`CrownConfig.lua:130`). The landing look is in `handlers.slam` (`RoyalUI.lua:1423-1431`). The server's `slam()` only sends events and hits every fighter within 11 studs in all directions (`CrownArena.lua:632-646`). It reuses the heavy's Lunge (`CrownArena.lua:635`), so it needs its own animation.
4. **Health drain.** Add an `hp` field in `newFighter` (`CrownArena.lua:116-126`), drain it in `applyHit`, the one place every hit lands (`:539-558`), publish it like `Stagger` (`:129-136`), score at 0 through `ringOutInBout(f, "ko")` (`:783`), and reset it where stagger resets (`:789`). Do not set `Humanoid.Health` to 0: death waits up to 12 s for a new body (`:805-808`). Contests have their own ring-out handler (`CrownContest.lua:319-346`). Knights have infinite MaxHealth (`CrownArena.lua:410-411`). Bars go in the duel board and fighter tags (`RoyalUI.lua:554-592`, `:1031-1033`).
5. **Weapon skins.** `makeSword()` builds the one sword and `giveSword(f)` hands it out (`CrownArena.lua:172-220`); read a skin key there. Keep the tool name `RoyalSword`, since three functions find it by name (`:201`, `:224`, `:231`). Reach comes from `CB.Strike.range`, not the blade. Skins stay cosmetic (`CrownConfig.lua:16-19`).
6. **Picking a skin before the fight.** The intro plus countdown is 3.5 s plus 3 steps (`RoyalUI.lua:1365-1376`, `CrownConfig.lua:72-73`). A new `RoyalRequest` action goes in the `handlers` table (`Crown.lua:861-862`); all actions share one 0.15 s rate limit (`Crown.lua:868-871`). Or use a new marker with a `RoyalPrompt` of a new `Kind` directly in `Royal Capital.Markers` (`RoyalCapitalBuilder.lua:490-508`, `Crown.lua:972-987`). For a shop, copy the King tab: an ownership attribute plus buy and equip remotes (`ShopMenu.lua:463-482`).
7. **Saving skins.** Put owned and chosen skins in `profile.royal`. Add them to `royalCapture` / `royalApply` (`Crown.lua:252-285`) and to GardenStore's royal sanitise (`GardenStore.lua:306-307`, `:410-438`). A new saved number goes in both `NUMBERS` and `ROYAL_NUMBERS` (`Crown.lua:107-108`).
8. **Movement in duels.** Any movement change goes through `G.arenaStats`, which `applyWalkSpeed` reads first (`CrownArena.lua:146-154`, `GardenService.lua:196-208`).
9. **Testing.** Use `RoyalDev` spar, bout, ringout and endBout (`CrownArena.lua:1410-1426`). Studio uses 0 fruit and 5 s rules, so live rules differ (`CrownConfig.lua:38-41`).

Risks: hits land 0.08 s after a strike and 0.12 s after a heavy (`CrownArena.lua:575`, `:591`), so long animations look late. Cooldowns live on both client and server; change the config, not one side (`RoyalCombat.lua:53-59`). Slam effects must fit the arena: pillars at r 21, perches at r 41, the ring at r 30 over an opening of r 52 (`RoyalCapitalBuilder.lua:58-59`, `:1370-1381`).

Conflicts: health drain adds a second way to score beside ring-outs; decide how they combine. Bats and explosives must skip `InArena` (6.2, 6.3).

### 6.11 Cross-feature conflicts at a glance

| Conflict | Features | What to do |
|---|---|---|
| Luck potion jumps the rarity cap | 1, 7 | Cap `spawnPlan` and the Lucky pick-up discovery (`Upgrades.lua:372-379`, `Carrying.lua:185`). |
| Buying Spawn Rarity fills the Index past the cap | 1 | Cap the purchase and its discovery loop (`Upgrades.lua:309-335`). |
| Merge All bypasses a merge cap | 1 | Cap `cascade` as well as `merge` (`Merging.lua:179`, `:315`). |
| Throne needs 10 fruit, cap allows 5 | 1 | Lower `C.MinDiscovered` or tie it to the cap (`CrownConfig.lua:38`). |
| Rebirth shard cost vs 100k-shard base vs tiny early Index payout | 1, 4 | Model the shard economy before picking numbers (`RebirthLock.lua:61`). |
| Rebirth wipes every `UpgradeKeys` level | 1, 4 | Keep the base out of `UpgradeKeys`, or exclude it (`RebirthLock.lua:95`). |
| Bats and explosives share `useTool`'s two branches | 2, 3 | Add explicit kinds and branches (`Tools.lua:250-310`). |
| Lock dome blocks only TNT | 2, 3 | Decide a rule for bats, new explosives and artillery (`Tools.lua:265-270`). |
| Clear and Merge All ignore Busy and per-fruit state | 3 | Handle health there too (`Merging.lua:406`, `:494-504`). |
| Pets, ores and boats add prompts that GardenClient's global handler touches | 4, 5, 6 | Filter by prompt parent (`GardenClient.lua:124-167`). |
| Non-fruit income is invisible to the client | 4, 5 | Extend the `Income` payload (`Ambient.lua:473-478`). |
| Shallows pieces block boats, and may be restyled | 6, 8 | Plan lanes and the lily pad decision together. |
| New movement effects leak into duels | 2, 6, 7, 10 | Repeat the `InArena` check everywhere (`GardenService.lua:266`). |

## 7. Load order and module pattern

The server side is one Script, `GardenService.lua`, plus ModuleScripts under it. It was split because a Script's source is capped at 200,000 characters (`GardenService.lua:96-97`).

**The pattern.** Every module is `return function(G)` (for example `Storage.lua:15`). At the top it copies what it needs out of G into locals (`Storage.lua:17-20`). At the bottom it writes its own functions back into G (`Storage.lua:91`). The main script does the same: after each `require(script.X)(G)` it copies the new functions it needs out of G (`GardenService.lua:117-118`, `:402-407`). Its own shared state goes into G in blocks marked "into the shared runtime" (`GardenService.lua:100-115`, `:390-395`, `:483-485`, `:747-749`).

**The order** (each step can use only what earlier steps put in G):

| Step | What loads | Line |
|---|---|---|
| 1 | Shared modules: GardenConfig, FruitIndex, FruitEconomy, GardenStore (as `Store`), IndexRewards, DevPanel.Server | `GardenService.lua:27-32` |
| 2 | Wipe and remake the `SpawnedFruits` folder; make the remotes | `GardenService.lua:34-41`, `:43-86` |
| 3 | Create G and fill it with services, Config and remotes | `GardenService.lua:98-115` |
| 4 | PlotRegistry | `:117` |
| 5 | Main script: balances, the held table, the pending-merge ledger, walk speed and visibility, then into G | `:120-395` |
| 6 | Stats, Progression, FruitFactory, Upgrades | `:398`, `:402`, `:409`, `:413` |
| 7 | Main script: claim, release and seed plots, then into G | `:431-485` |
| 8 | Carrying, Tools, Storage, Merging, RebirthLock, Persistence | `:487`, `:491`, `:495`, `:498`, `:504`, `:508` |
| 9 | Crown, CrownArena, CrownContest, CrownDecree | `:515-518` |
| 10 | Wiring: the fruit prompt and the remote handlers | `:520-546` |
| 11 | Main script: join and leave (`watchPlayer`, `teardownPlayer`, `onPlayerLeaving`) | `:608-744` |
| 12 | GardenService.DevPanel module | `:751` |
| 13 | `Store.Init` with `captureProfile` / `applyProfile`, then `DevPanel.Start` | `:757-768` |
| 14 | `BindToClose` shutdown save; the potion expiry sweep | `:770-781`, `:793-821` |
| 15 | Ambient, last. It hooks player join and leave and starts the spawn, income and lock loops | `:824`; `Ambient.lua:389-401` |

**Rules that come from this.**
- **Copy at require time, or look up late.** A module that needs a function from a later module must read it from G when it is called. Examples: `G.arenaStats` in `applyWalkSpeed` (`GardenService.lua:200`), `G.crownRefresh` in Progression (`Progression.lua:54`), `G.spawnBoost` in Ambient (`Ambient.lua:421`), `stowTool` and `useTool` in Carrying (`Carrying.lua:352-353`). CrownArena copies `applyWalkSpeed` and `applyVisibility` once at load (`CrownArena.lua:29-32`).
- **Three accidental globals.** `publishMultiplier` in Progression (`Progression.lua:341`), and `stowTool` and `useTool` in Tools (`Tools.lua:172`, `:244`), are declared without `local`. They work because each module also writes them into G. The main script's `local publishMultiplier` (`GardenService.lua:126`) is a different variable, filled at `:407`.
- **Single writers.** `applyWalkSpeed` is the only writer of WalkSpeed and JumpPower (`GardenService.lua:187-190`). PlayerVision is the only writer of blur, tint and Lighting (`PlayerVision.lua:5-6`). MusicPlayer is the only writer of music volumes (`MusicPlayer.lua:39-51`). HudScale is the only writer of HUD cluster Position and Size (`HudScale.lua:16-19`). Add new effects inside these, never beside them.

**Client side.** There is no G on the client. Each LocalScript runs on its own, waits for `PlayerGui.GardenUI` (`GardenClient.lua:68`), and loads the shared ReplicatedStorage modules. GardenConfig requires only FruitIndex, so there is no require cycle (`GardenConfig.lua:10-14`). `LoadingScreen` is in ReplicatedFirst, so it runs before every other client script (`LoadingScreen.lua:4-6`).

**Builders.** The ServerStorage builders are run by hand from the Studio command bar and never by the game (`ForestBuilder.lua:2-4`). Only the dev panel runs during play (`GardenService.lua:763-768`).

## 8. Known gotchas

Deduplicated across the seven notes. Numbered so other notes can refer to them.

**Saving**
- G1. **Save before destroy.** `teardownPlayer` reclaims the carried item and saves before `releaseHold` and `releasePlot` destroy the fruit. A line added above the save stops fruit saving (`GardenService.lua:703-715`).
- G2. **`onPlayerLeaving` takes exactly one parameter.** `PlayerRemoving` passes an exit reason that would land in `skipSave`. Ambient asserts this at startup (`GardenService.lua:737-744`, `Ambient.lua:397-400`).
- G3. **A new saved field must also go in `sanitise`,** or it saves once and vanishes on the next load (`GardenStore.lua:447-450`). The royal number list exists twice (`Crown.lua:107-108`, `GardenStore.lua:306-307`).
- G4. **Non-fruit names vanish.** `stored`, `plot` and `index` keep only names in `FruitIndex.ByName` (`GardenStore.lua:289`, `:345`; `Persistence.lua:56`, `:70`). Renaming a fruit's internal `name` deletes it from saves; changing `display` is safe.
- G5. **The plot is saved as counts.** Positions, rotations, `Lucky` and any future per-fruit state are lost on rejoin (`Persistence.lua:98`, `:124-168`).
- G6. **`captureProfile` and `applyProfile` must stay yield-free,** and `doRebirth` too, with the payout computed before the Index wipe (`Persistence.lua:35-37`, `RebirthLock.lua:59-67`).
- G7. **The pending-merge ledger must be cleared** on rebirth and plot release, or a fruit comes back on the next save (`RebirthLock.lua:79`, `GardenService.lua:475`).
- G8. **Losing the session lock stops saving for that visit,** and every `Store.IsReady` action then says "Still loading" (`GardenStore.lua:634-641`, `:932-935`). `commit` refuses while another write is in flight, so the rebirth ForceSave can fail with only a warning (`GardenStore.lua:578-580`, `RebirthLock.lua:134-142`). `Store.Touch` does nothing until the session is ready (`GardenStore.lua:860-866`).
- G9. **Studio saving traps.** With API access off, Studio uses an in-memory store that forgets on Stop. `workspace.StudioNoSave = true` skips writes (`GardenStore.lua:197-203`, `:572-575`). With API access on, the dev panel's Fresh profile deletes the dev's real key and Seed test overwrites it (`GardenStore.lua:1028-1035`, `:1049-1056`).
- G10. **Cooldowns and potions reset.** The Clear, Merge All and Lock cooldowns and the gamepass cache reset on rejoin; potions are never saved and end on death (`GardenService.lua:724-728`, `:177-180`, `:601-604`).

**Server wiring**
- G11. **Load order is the contract.** See section 7. Late lookups through G work only because they happen at call time.
- G12. **An error inside `claimPlot` stops the profile loading,** because it aborts `watchPlayer` before `Store.Load` (`PlotRegistry.lua:87-90`).
- G13. **Two things are called "Store".** The `Store` remote banks a fruit; `Store` in code is the GardenStore save module (`GardenService.lua:30`, `:60`).
- G14. **The crown check runs every income tick** for every player (`Progression.lua:87`, `Ambient.lua:470`). Friend counts call the yielding `IsFriendsWith` for every pair on each join and leave (`Progression.lua:105-127`).
- G15. **Cash can only be set inside the main script,** because `balances` is a local table. The `DevSetCash` door works only in Studio (`GardenService.lua:622-631`).

**Fruit, plots and tools**
- G16. **Rank runs backwards.** Rank 1 is the best fruit, so the "lowest" fruit has the highest rank number. Reading it the wrong way in Clear deletes the best fruit (`Merging.lua:353-356`, `GardenConfig.lua:1048-1050`).
- G17. **Busy is the real rule.** `prompt.Enabled` is overwritten five times a second by GardenClient. Every animation exit path must clear Busy (`Carrying.lua:232-241`, `:268-278`). Only TNT skips Busy fruit; Clear and Merge All do not (`Tools.lua:282`, `Merging.lua:406`, `:494-504`).
- G18. **Three fruit counts.** Spawning counts soil + hands + pending merges (`Upgrades.lua:286-288`). Dropping counts soil only (`Carrying.lua:331-333`). Income counts soil only (`FruitFactory.lua:39-45`). A silent drop can overfill a plot by one (`Carrying.lua:339-342`).
- G19. **The weld order in `holdFruit` is load-bearing:** position, parent, zero velocity, network owner, then weld. `dropAt` mirrors it (`Carrying.lua:102-121`, `:218-221`).
- G20. **`CarryingFruit` also holds tool names.** Read `CarryingTool` to tell them apart (`Carrying.lua:142`, `Tools.lua:155`).
- G21. **Every tool kind that is not "consume" runs the TNT code,** and a missing `radius` errors (`Tools.lua:250-310`, `:283`). Drop on a carried tool means "use"; death and leaving stow it instead (`Carrying.lua:297-306`).
- G22. **TNT works on any plot, including your own.** Only lockdown blocks it, and the dome collides with nobody. A Config comment saying nobody can walk in is stale (`Tools.lua:161-168`, `:265-270`; `RebirthLock.lua:183`; `GardenConfig.lua:883-884`, `:897-900`).
- G23. **Fruit are `CanQuery = false` and `CanTouch = false`,** so raycasts and Touched never hit them (`FruitFactory.lua:232-236`).
- G24. **The soil is the largest part.** Any decoration bigger than the soil becomes the soil, and `buildFence` deletes children named `Fence`, `FenceFront` or `Part` (`PlotRegistry.lua:21-30`, `Upgrades.lua:168-173`).
- G25. **Plot lifecycle quirks.** Startup wipes the whole `SpawnedFruits` folder (`GardenService.lua:37-38`). `claimPlot` always clears fruit (`:440`). `releasePlot` shrinks the plot to level 0, and the new owner's size lands only with their profile (`:456`, `Persistence.lua:276-278`). `applyPlotSize` skips a level that already matches, so a hand-edited soil never rebuilds (`Upgrades.lua:244`).
- G26. **Spawn search is costly.** Each of up to 40 spot tries walks the whole server's fruit folder (`Upgrades.lua:412`, `FruitFactory.lua:26-34`, `:51-53`).
- G27. **The storage cap applies only on the way in.** Restore overflow goes into storage uncapped (`GardenConfig.lua:530-532`, `Persistence.lua:310-314`).
- G28. **`cascade` depends on a positional insert.** Appending leaves mergeable pairs with no error (`Merging.lua:297-302`, `:326`). Ask `discover` before the merge animation; it answers "first time" once (`Merging.lua:199-205`).

**Economy and config**
- G29. **Rebirth resets every `UpgradeKeys` level,** including any new key (`RebirthLock.lua:95`, `Upgrades.lua:294-299`). It keeps shards, tools, King auras, the royal record, running potions and cooldowns.
- G30. **`Config.UpgradeCurrency` does nothing.** Every upgrade costs cash (`GardenConfig.lua:190`, `Upgrades.lua:313-320`).
- G31. **Friends do not pay offline.** Rebirth and Index bonuses do (`Progression.lua:333`, `Persistence.lua:191`).
- G32. **Changing `FruitIndex.Count` re-prices everything:** spawn ladder, income, Index bonus, shards, and even bonuses already claimed, which the server re-sums live (`GardenConfig.lua:87`, `IndexRewards.lua:38-41`, `Progression.lua:203-214`). FruitIndex is generated; hand edits are lost (`FruitIndex.lua:2-3`).
- G33. **Config clamps silently.** A level of 999 reads as max with no error (`GardenConfig.lua:1895-1903`). Ladders are 0-based and `#` counts steps, so adding `PlotSizes[5]` adds a formula-priced purchase (`GardenConfig.lua:169-175`). `UpgradeKeys` is the save list (`GardenConfig.lua:39-43`).
- G34. **Config section headers are unreliable.** Search by key name (`GardenConfig.lua:1169-1194`, `:1859-1883`).
- G35. **Stale comments.** `SPAWN_TICK` says 4 s is the fastest rung; it is 2.5 s (`Upgrades.lua:290-291`, `GardenConfig.lua:61`). "A plot can hold 40 fruit"; the cap is 35 (`GardenConfig.lua:1102`, `:52`). devSeed says "a cap of 40" (`DevPanel.lua:230`). CashClient says shards buy only cosmetics (`CashClient.lua:83-84`). KingClient's header says the richest player is King, and its crown branch is dead code (`KingClient.lua:2-3`, `:80`, `:87-107`). Ambient names a `DrownFade` script that does not exist (`Ambient.lua:35-36`).
- G36. **Number formats.** `FruitEconomy.Format` floors values under 1000, so 0.5 shows "0" (`FruitEconomy.lua:67-68`). The rebirth texts use `%d` and cannot show a fraction (`GardenConfig.lua:1032`, `:1037`).
- G37. **Speed potion beats the swept-speed line.** Walking at 32 is above `DrownSweptSpeed` 30; harmless only while the drown radii sit far out (`GardenConfig.lua:943`, `:1721`, `:1291`, `:1556-1558`).

**Client UI**
- G38. **ButtonAnimator adopts every new GuiButton** and takes its UIScale slot. Set `Animated` before parenting a full-screen helper (`ButtonAnimator.lua:213-217`, `ClearMenu.lua:82-83`). Never give a button cluster a UIScale (`HudScale.lua:27-33`).
- G39. **HudScale owns HUD Position and Size.** A cluster not in `Config.HudClusters` keeps its desktop size; HudScale waits only 5 s per cluster (`HudScale.lua:16-19`, `:64-67`).
- G40. **Panel exclusion lists are hand-written and uneven,** and IndexMenu caches its list at load (`IndexMenu.lua:46-50`, `StoredFruitsMenu.lua:541`, `UpgradeMenu.lua:421`). `Visible` is not "open"; toggles read `isOpen` (`ShopMenu.lua:606-610`).
- G41. **Client copies of server rules.** The rebirth test lives in RebirthMenu, AlertBadges and the server; the "maxed" test in UpgradeMenu, AlertBadges, PenUpgradeSign and the server; `StoredCap` in GardenClient, StoredFruitsMenu and the server (`AlertBadges.lua:49-51`, `PenUpgradeSign.lua:182-184`, `GardenClient.lua:399-400`). Change them together.
- G42. **The prompt handler is global.** Every ProximityPrompt gets GardenClient's outline and `NameTag` lookup (`GardenClient.lua:124-167`, `:216`). Highlights are capped at 24 (`GardenConfig.lua:1887-1889`).
- G43. **Scrolling grids need pixel heights and an explicit canvas** (`IndexMenu.lua:526-545`). The inventory rebuilds every card on any change (`StoredFruitsMenu.lua:474-496`). The Index builds 64 cells at load (`IndexMenu.lua:624-637`).
- G44. **Loading screen limits.** It preloads only `rbxassetid://` strings, gives up after 45 s, and pre-draws only `...Menu` children of GardenUI (`LoadingScreen.lua:338`, `:383-392`, `:411-416`). GardenToast waits forever for GardenUI and must never be required by server code (`GardenToast.lua:17-24`, `:84`).
- G45. **Mute is not saved** (`MuteButton.lua:10-13`). Modals do not close panels; a Blocker sink stops clicks (`ClearMenu.lua:74-83`).

**Royal**
- G46. **Two clocks.** Cooldowns use `os.clock()`; reign times use `workspace:GetServerTimeNow()`. Do not compare them (`Crown.lua:75`, `:334-336`, `:744-746`).
- G47. **Knockback on a player must be applied by that player's client** (`CrownArena.lua:507-514`, `RoyalUI.lua:1482-1490`).
- G48. **Falls over 220 studs are killed by a server ray,** so the arena anchors knocked-out fighters until placed again; new movement code must release first (`Ambient.lua:260-301`, `CrownArena.lua:247-262`).
- G49. **Two switches.** RoyalUI and RoyalCombat obey `CrownConfig.Enabled`; KingClient obeys `GardenConfig.KingEnabled` (`RoyalUI.lua:34`, `KingClient.lua:33`). On the server, `C.Enabled` stops only the wealth claim (`Crown.lua:799`). `C.GateOpenForEvents` is never read (`CrownArena.lua:733-737`).
- G50. **Two `KingUserId` copies,** on workspace and on RoyalState, both written by `publishCrown` (`Crown.lua:691-696`).
- G51. **One 0.15 s rate limit covers every RoyalRequest action;** a second quick request is dropped silently (`Crown.lua:868-871`).
- G52. **`finishBout` must clear fighters before `leaveArena`,** or players stay frozen (`CrownArena.lua:841-848`). Keep the crown's Welds; WeldConstraints left it 180 studs away (`Crown.lua:465-484`).
- G53. **Hard-coded positions assume the castle never moves** (`CrownDecree.lua:148`, `CrownArena.lua:57`, `Crown.lua:364`, `:918`).
- G54. **Studio rules differ.** 0 fruit and 5 s in Studio, a `"studio"` DataStore scope, and the `RoyalDev` tool (`CrownConfig.lua:38-41`, `Crown.lua:602`). A Studio pass does not prove the live rules.
- G55. **Small royal quirks.** The "decline while vulnerable starts a contest" branch looks unreachable (`Crown.lua:842-847`, `CrownArena.lua:973-974`). The Ruler board hard-codes "+10% income" per friend (`Crown.lua:1098`). Practice bouts skip eligibility (`CrownArena.lua:1121-1129`). Empire Build pays idle players too (`CrownDecree.lua:272-274`).

**World, music and builders**
- G56. **The currents are saved parts,** not scripts. No swimming code exists (`GardenConfig.lua:1310`, `:1313-1319`; `PlayerVision.lua:266-297`).
- G57. **The Income payload is only `(multiplier, dt)`.** Any income that is not a fruit on the plot is invisible to the client (`Ambient.lua:473-478`).
- G58. **If `Config.MusicTracks` is empty, MusicPlayer quits** before making the sky, forest, royal and fight tracks (`MusicPlayer.lua:28-29`). PlayerVision owns the Ocean's transparency (`PlayerVision.lua:478-492`).
- G59. **Builder chain.** Run SkyIslandBuilder, then SkyIslandDecor, then RoyalCapitalBuilder; each later step destroys and rebuilds what it owns. Run BeanstalkCrown first if the crown changes (`SkyIslandBuilder.lua:451-454`, `SkyIslandDecor.lua:36-39`, `RoyalCapitalBuilder.lua:561-582`).
- G60. **Studio caches `require`.** Require a clone, or a second run builds nothing (`ShallowsBuilder.lua:14-15`). Three builders refuse to run outside place 88602028667228 (`RoyalCapitalBuilder.lua:35`, `ForestBuilder.lua:195`, `ShallowsBuilder.lua:222`). BeanstalkCrown waits forever if the beanstalk is missing (`BeanstalkCrown.lua:30`).
- G61. **Builder re-runs differ.** ForestBuilder re-rolls trees against today's neighbours (`ForestBuilder.lua:215-222`). ShallowsBuilder consumes the pads, so a second run does nothing (`ShallowsBuilder.lua:386-389`). SkyIslandDecor does not know about the Royal Capital (`SkyIslandDecor.lua:239-244`). The Royal Gate must stay outside `Sky Islands` (`RoyalCapitalBuilder.lua:1647-1649`).
- G62. **SaturationPass is global** and skips everything below y -500. Never undo it with Ctrl+Z (`SaturationPass.lua:19-20`, `:57`, `:90-104`).
- G63. **Dev panel limits.** One UserId only (`DevPanel.Server.lua:36-38`). Reload throws away unsaved progress, on live servers too (`DevPanel.lua:214`). `clearPlayerResidentState` is a fixed list that misses `Royal*` attributes and potion timers (`DevPanel.lua:162-183`).
