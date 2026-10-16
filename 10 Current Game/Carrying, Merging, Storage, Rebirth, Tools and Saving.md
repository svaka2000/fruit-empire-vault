---
tags: [current-game, code-map]
date: 2026-10-04
sources:
  # the subsystem, each read in full
  - ServerScriptService.GardenService.Carrying.lua
  - ServerScriptService.GardenService.Merging.lua
  - ServerScriptService.GardenService.Storage.lua
  - ServerScriptService.GardenService.RebirthLock.lua
  - ServerScriptService.GardenService.Tools.lua
  - ServerScriptService.GardenService.Persistence.lua
  - ServerScriptService.GardenStore.lua
  # read only where this subsystem calls into them
  - ReplicatedStorage.GardenConfig.lua
  - ReplicatedStorage.Fruits.FruitIndex.lua
  - ServerScriptService.GardenService.lua
  - ServerScriptService.GardenService.Progression.lua
  - ServerScriptService.GardenService.Upgrades.lua
  - ServerScriptService.GardenService.FruitFactory.lua
  - ServerScriptService.GardenService.PlotRegistry.lua
  # client scripts, only the cited lines (found by search)
  - StarterPlayer.StarterPlayerScripts.GardenClient.lua
  - StarterPlayer.StarterPlayerScripts.StoredFruitsMenu.lua
  - StarterPlayer.StarterPlayerScripts.ClearMenu.lua
  - StarterPlayer.StarterPlayerScripts.MergeAllButton.lua
  - StarterPlayer.StarterPlayerScripts.LockButton.lua
  - StarterPlayer.StarterPlayerScripts.RebirthMenu.lua
  - StarterPlayer.StarterPlayerScripts.CashClient.lua
  - StarterPlayer.StarterPlayerScripts.OfflinePopup.lua
  - StarterPlayer.StarterPlayerScripts.FeatherFall.lua
  - StarterPlayer.StarterPlayerScripts.TopEarnersBoard.lua
---

# Carrying, Merging, Storage, Rebirth, Tools and Saving

Code map of the script export in `/Users/samarthvaka/fruit-empire/src-export-2026-10-04`, taken 4 Oct 2026. Vision source: [[Brain Dump 2026-10-04]].

**How to read the citations.** `File.lua:LINE` is a file in that export folder and a line in it. A range like `:160-203` means those lines. Only code that was re-opened and checked is cited.

## Terms

- **Plot**: one player's garden pen. Every fruit on it carries a `PlotName` attribute naming the plot (`ServerScriptService.GardenService.FruitFactory.lua:26-34`).
- **Rank**: a fruit's place on the 64-fruit ladder. Rank 1 (Soul Fruit) is the best and rank 64 (Blueberry) is the worst. Two fruit of rank R merge into one of rank R-1 (`ReplicatedStorage.Fruits.FruitIndex.lua:5`).
- **Tier**: the rarity group, a number from 1 (COMMON) to 9 (TRANSCENDENT), stored as `tier` and `tierName` on every fruit (`ReplicatedStorage.Fruits.FruitIndex.lua:10-73`).
- **Carry**: holding one fruit or one tool welded above your head. Each player has one carry slot, kept in the server table `held` (`ServerScriptService.GardenService.lua:128-132`).
- **Busy**: an attribute the server puts on a fruit while it is mid-drop or mid-merge. While it is set, nobody can pick that fruit up or merge into it.
- **G**: the shared table the main GardenService script hands to every module. A module copies the helpers it needs out of G and writes its own functions back into G (`ServerScriptService.GardenService.lua:88-98`).
- **Profile**: the one saved document per player.
- **Session lock**: a stamp inside the profile that says which server is currently playing it.
- **Touch**: pulling the next autosave earlier, instead of writing straight away.
- **Yield-free**: code that never pauses (no `task.wait`, no web call), so no other code can run in the middle of it.
- **Shards**: the second currency. Earned from the Index, spent on tools and cosmetics, and kept through a rebirth.
- **Index**: the catalogue of fruit a player has discovered. It lives in a folder named `IndexState` on the player.
- **Tool**: a shop item bought with shards (TNT and potions). A tool is not a fruit.
- **Effect**: a timed buff from drinking a potion.
- **Remote**: a RemoteEvent in `ReplicatedStorage.GardenRemotes`. Every remote this subsystem uses is a RemoteEvent, made by the `remote()` helper (`ServerScriptService.GardenService.lua:49-57`). The game's one RemoteFunction, `RoyalQuery`, belongs to the Royal System (`ServerScriptService.GardenService.Crown.lua:55-60`).

## 1. What it does

This subsystem is everything a player does with one item in their hands, plus how the whole account is saved. You press E on your own fruit to lift it, press E on a matching fruit to merge the pair into the next rank, or drop it back onto your plot. You can bank the carried fruit in a 5-slot storage bag and take it out later. Shards buy tools: TNT destroys fruit on whatever plot you stand on, and potions give timed buffs. Rebirth wipes the run (plot, cash, upgrades, storage, Index) for a permanent cash multiplier, and lockdown raises a dome that blocks TNT for 60 seconds. GardenStore saves all of it to one DataStore document per player, guarded by a session lock, with autosave, leave and shutdown paths.

## 2. Scripts

| Script | Role | Lines |
|---|---|---|
| `ServerScriptService.GardenService.Carrying.lua` | Pick up, hold above the head, drop with an arc. Owns `holdFruit`, which tools reuse. | 359 |
| `ServerScriptService.GardenService.Merging.lua` | Hand merge and its animation, the Clear button, the Merge All gamepass cascade. | 583 |
| `ServerScriptService.GardenService.Storage.lua` | Store the carried fruit; equip a stored fruit. | 92 |
| `ServerScriptService.GardenService.RebirthLock.lua` | Rebirth reset and the lockdown dome. | 246 |
| `ServerScriptService.GardenService.Tools.lua` | Storage and tool folders, buy, equip, stow, drink a potion, set off TNT. | 318 |
| `ServerScriptService.GardenService.Persistence.lua` | Turn a live player into a profile and back; offline earnings; reclaim the carried item on leave. | 365 |
| `ServerScriptService.GardenStore.lua` | The DataStore layer: schema, sanitise, session lock, the single write path, autosave, shutdown, self-test. | 1269 |

## 3. How it works

### 3.1 Wiring

- Every module is `return function(G)`. GardenService requires them in this order: Carrying, Tools, Storage, Merging, RebirthLock, Persistence (`ServerScriptService.GardenService.lua:487-510`).
- The order matters. A module copies helpers out of G when it is required, so it can only take what an earlier module already put in. Carrying needs `stowTool` and `useTool` from Tools, which loads after it, so Carrying looks them up through G at call time instead (`ServerScriptService.GardenService.Carrying.lua:289`, `:352-353`).
- Inputs are routed in `ServerScriptService.GardenService.lua:522-546`. The table is in section 4.3.

### 3.2 Pick up

1. The player presses E on a fruit's prompt. `PromptTriggered` calls `merge` if the player already holds something, otherwise `pickUp` (`ServerScriptService.GardenService.lua:522-526`).
2. `pickUp` (`ServerScriptService.GardenService.Carrying.lua:160-203`) quietly refuses if: the player already holds something (`:161`); the profile has not loaded (`:164`); the fruit is not in the fruit folder or is already held (`:165`); the fruit is Busy (`:169`); the fruit is not on the player's own plot (`:173`, using `ownsFruit` at `:153-156`); or the fruit is farther than `Config.MaxReach` (`:179`).
3. If the player's spawn tier can grow this fruit, or it is a Lucky fruit from the Luck potion, the pick-up discovers it in the Index (`:185-196`). Bottom-of-ladder fruit are never the result of a merge, so this pick-up, plus the spawn-tier backfill (`ServerScriptService.GardenService.Progression.lua:190-199`), is how they enter the Index.
4. It sets `HeldBy` to the player's UserId and turns the prompt off (`:198-200`), then calls `holdFruit`.
5. `holdFruit` (`:95-143`) moves the fruit above the head, parents it into the character, zeroes its velocity, gives network ownership to the player, and only then welds it (`:122-139`). It starts the arms-up animation and sets the `CarryingFruit` attribute (`:141-142`).
6. The fruit keeps its `PlotName` while carried, so the spawner still counts it as taking a slot (`ServerScriptService.GardenService.Upgrades.lua:271-288`).

### 3.3 Merge

1. `merge` (`ServerScriptService.GardenService.Merging.lua:153-214`) runs when E is pressed while carrying. It refuses if: the player holds a tool (`:158`); not loaded (`:159-162`); the target is not in the folder, is held, or is Busy (`:164`, `:167`); the names differ (`:168`); the target is not on your plot (`:172`); it is out of reach (`:177`); or the target is rank 1 and has nothing above it (`:179-180`).
2. The merge commits. The carried fruit is destroyed (`:183`). The target's prompt goes off and its `PlotName` is cleared, so it stops counting and earning (`:191-193`).
3. A landing spot is chosen where the target stood (`:195-197`). `discover` is asked before the animation, because it only answers "first time" once (`:199-205`).
4. `mergeTransition` (`:66-151`) books the promoted fruit in the pending-merge ledger straight away (`:76`), with a backup release on a timer (`:80`). The old fruit spins, swells and washes to white (`:84-97`), is destroyed (`:99`), and the new fruit is placed at the white peak (`:103`). The ledger entry is released (`:106`). The new fruit stays Busy while it settles (`:112`, `:134`).
5. `recordMerge` updates Best Merge (`ServerScriptService.GardenService.Merging.lua:211`, defined at `ServerScriptService.GardenService.Progression.lua:129-140`). If the King's Throne code is loaded, `G.crownOnMerge` counts the merge for the Royal Decree merge rush (`ServerScriptService.GardenService.Merging.lua:213`).

**The pending-merge ledger** (`pendingMerge`, `ServerScriptService.GardenService.lua:134-169`). For about a quarter second the promoted fruit exists nowhere in the world. The ledger remembers it, so a save landing in that window still writes it (`ServerScriptService.GardenService.Persistence.lua:99-105`). Each booking returns a release that only works once (`ServerScriptService.GardenService.lua:146-169`).

### 3.4 Drop

1. The Drop remote calls `dropFromPlayer(player)` (`ServerScriptService.GardenService.lua:528`; `ServerScriptService.GardenService.Carrying.lua:291-347`).
2. If the held thing is a tool, Drop means use it (`useTool`). A silent drop (death, despawn, leaving) puts the tool back in the bag instead (`:297-306`).
3. No plot: refuse and keep the fruit in hand (`:309-316`).
4. A normal drop needs you standing on your own plot (`:325-328`) and a free slot. `#fruitOfPlot(plot) >= capacityFor(plot)` refuses (`:333-336`). A silent drop skips both checks (`:339-342`).
5. The target point is `Config.DropForwardStuds` in front of the player (`:344-346`). `dropAt` (`:205-281`) releases the hold, picks a free spot before tagging the fruit (`:215`), sets `PlotName` (`:216`), parents it to the fruit folder, takes ownership back and anchors it (`:222-225`). It sets Busy (`:242`) and tweens up, down, a hop, and settle (`:257-276`). Busy is cleared on every exit path (`:278`).

### 3.5 Storage

- `storeCarried` (`ServerScriptService.GardenService.Storage.lua:24-51`). A carried tool goes back to the tool bag (`:30-33`). A fruit is refused when `Config.StoredTotal(...) >= Config.StoredCap` (`:43-47`). Otherwise the fruit is destroyed and its count goes up by one (`:49-50`).
- Storage is a folder of IntValues on the Player, one per fruit name. No fruit model is kept (`ServerScriptService.GardenService.Tools.lua:24-49`).
- `equipStored` (`ServerScriptService.GardenService.Storage.lua:56-87`) checks the name (`:57`), refuses while carrying (`:63-69`), needs a count above 0 (`:71-72`), builds a fresh fruit with `createFruit` (`:78`), lowers the count (`:83-84`), and holds it (`:86`). This fruit has no `PlotName` until it is dropped.

### 3.6 Clear and Merge All

- `clearLowest` (`ServerScriptService.GardenService.Merging.lua:357-431`), Clear remote. The server picks the worst rank (highest rank number) on your plot (`:384-391`). It refuses if every fruit is the same kind (`:396-399`). It destroys every fruit of that rank that is not held, with a smoke puff each (`:402-412`). The cooldown starts only on success (`:420`) and is published as `ClearReadyAt` (`:425`).
- `mergeAllOn` (`:441-575`), MergeAll remote. Gated on the gamepass (`:452-463`), whose answer is cached per player (`:259-280`). It counts fruit by name (`:477-481`) and runs `cascade` (`:289-346`). `cascade` is pure arithmetic: lowest rung first, pairs promote, and a newly reached rung is inserted right after the current one so it is still processed (`:326`). Then it removes the consumed fruit (`:494-505`), records discoveries and best merge for every rung passed through (`:522-528`), and places the new fruit rarest first with a flash (`:510-514`, `:535-559`). The cooldown starts only on success (`:568`) and is published as `MergeAllReadyAt` (`:572`).

### 3.7 Tools

- `buyTool` (`ServerScriptService.GardenService.Tools.lua:91-112`) checks the name against `Config.Tools` (`:93-94`), checks shards (`:100-104`), subtracts the price and adds one to the tool's count (`:106-107`), and Touches the save (`:110`).
- `equipTool` (`:116-159`) refuses while carrying (`:124-127`), needs a template at `ReplicatedStorage.Tools.<name>` (`:133-139`), takes one from the bag only after every check passed (`:143`), clones the template, shrinks it to `carrySize` if the spec has one (`:149-151`), and reuses `holdFruit` (`:155`). It marks `held[player].tool` and sets `CarryingTool` (`:156-158`).
- `useTool` (`:244-310`) is reached through the Drop remote.
  - `kind == "consume"` goes to `consumeTool` (`:186-242`). The effect must exist in `Config.Effects` (`:187-192`). The new end time is the later of now or the current end, plus the duration, so potions stack (`:209-214`). It writes `<Effect>EndsAt` and `<Effect>Span` (`:217`, `:223`), re-applies speed and visibility (`:225-226`), and plays a drink sound (`:230-238`).
  - Every other kind runs the TNT code. You must stand on some plot, anyone's (`:259-263`; `plotUnder` at `:163-168`). A locked plot refuses (`:267-270`). Every fruit within `spec.radius` of the player that is not Busy is untagged and destroyed (`:278-290`). A cosmetic Explosion with zero blast pressure plays (`:292-298`). Both the user and the plot owner get a toast (`:300-309`).
- `stowTool` (`:172-181`) puts a carried tool back. The Store button, dying and leaving all use it.

### 3.8 Rebirth

`doRebirth` (`ServerScriptService.GardenService.RebirthLock.lua:35-146`), Rebirth remote, no payload.

1. Gates: not already rebirthing (`ServerScriptService.GardenService.RebirthLock.lua:36-39`), loaded (`:40-43`), has a plot (`:44-48`), and, when `Config.RebirthNeedsAll` is true, every fruit in the ladder discovered (`:52-55`). The check itself is `indexComplete` (`ServerScriptService.GardenService.Progression.lua:311-320`).
2. The payout is the shards already claimed from the Index (`ServerScriptService.GardenService.RebirthLock.lua:61`). It is worked out before anything is wiped. The sum is `indexShardsEarned` (`ServerScriptService.GardenService.Progression.lua:296-307`).
3. The yield-free block (`ServerScriptService.GardenService.RebirthLock.lua:63-128`): destroys a carried fruit (`:71-73`); clears the plot and its pending ledger (`:75-79`); empties storage (`:83`); keeps tools (`:85-88`); empties the Index (`:92`); resets every upgrade level and the plot size to 0 (`:95-97`); clears Best Merge (`:102-105`); sets cash to 0 (`:109-111`); adds the payout to shards (`:115`); adds one to `Rebirths` and republishes the multiplier (`:117-119`); resets the spawn timer (`:124`); and seeds the starter fruit (`:127`).
4. It forces a save on its own thread (`ServerScriptService.GardenService.RebirthLock.lua:134-142`) and sends the notice (`:144-145`).

The cash multiplier is `1 + Rebirths * Config.RebirthCashBonus` (`ServerScriptService.GardenService.Progression.lua:324-326`).

### 3.9 Lockdown

`lockPlot` (`ServerScriptService.GardenService.RebirthLock.lua:195-240`), Lock remote. It refuses if not loaded, no plot, already locked, still cooling down, or not standing on your own plot (`:196-226`). It clones `ServerStorage.ForcefieldDome`, stretched to overhang the soil by 35 studs across and 22 along (`:158-188`). It sets `plot.lockedUntil` (`:230`), which is the value `useTool` checks. The cooldown runs from when the dome drops, not when it rises (`:236`). It publishes `LockEndsAt` and `LockReadyAt` on the player and `LockedUntil` on the plot model (`:233`, `:238-239`). The dome does not collide with anyone (`:183`).

### 3.10 Saving

- **Startup.** `Store.Init` gets `serialize = captureProfile` and `restore = applyProfile` (`ServerScriptService.GardenService.lua:757-761`). It tests the DataStore with a real write to a separate store (`ServerScriptService.GardenStore.lua:183-216`) and starts one autosave loop that wakes every second (`:1257-1266`).
- **Join.** `watchPlayer` claims a plot without seeding it, then starts `Store.Load` (`ServerScriptService.GardenService.lua:688`, `:697`). Load waits up to 10 s for this player's previous write to finish (`ServerScriptService.GardenStore.lua:694-698`). It takes the lock with UpdateAsync, retrying up to `LockWaitAttempts` times `LockWaitDelay` apart (`:717-735`). It kicks on failure (`:737-743`) or if the document's schema is newer than this server's (`:764-772`). It runs `sanitise` (`:775`) and then `applyProfile` (`:781`), then marks the session ready and sets `ProfileLoaded` (`:793-800`). The first autosave is staggered 60 to 120 s out (`:796`).
- **applyProfile** (`ServerScriptService.GardenService.Persistence.lua:196-332`) restores, in order: cash, best merge, storage, tools, Index, shards, any refund, the tester floor, rebirths, king look, royal record, upgrade levels, plot size. It backfills the Index on the next frame (`:288-290`). Then it either seeds a new player's plot (`:299-302`) or clears and restores the saved plot, sending overflow to storage (`:308-314`). Last, it pays offline earnings (`:324-331`).
- **Autosave.** `tickScheduler` captures and commits each ready player every `SaveInterval` (`ServerScriptService.GardenStore.lua:876-890`). `Store.Touch` pulls the next one forward to `SaveTouch` seconds (`:860-866`). `Store.ForceSave` writes now (`:868-874`); rebirth uses it.
- **commit** (`:563-659`) is the only write path. It refuses if the profile never loaded (`:564-566`). In Studio it does nothing when `workspace.StudioNoSave` is true (`:572-575`). It refuses a second write while one is in flight (`:578-580`). It encodes and decodes the profile first, so a bad payload never overwrites a good save (`:585-591`). It gives up if another server holds the lock (`:616-620`), and losing the lock disables saving for the rest of that session (`:634-641`).
- **Leave.** `teardownPlayer` (`ServerScriptService.GardenService.lua:703-735`) calls `reclaimCarried` first (`ServerScriptService.GardenService.Persistence.lua:337-359`). A tool goes back in the bag. A fruit is dropped on the plot centre so the plot snapshot counts it. With no plot, the fruit's name is passed along as the "orphan" and added to `profile.plot` (`:113-116`). Then `Store.SaveAndRelease` snapshots synchronously and writes with the lock released (`ServerScriptService.GardenStore.lua:816-845`).
- **Shutdown.** `BindToClose` runs the leave path for everyone, then `Store.Shutdown` waits up to `ShutdownDeadline` for the writes (`ServerScriptService.GardenService.lua:770-781`; `ServerScriptService.GardenStore.lua:892-928`).

## 4. Data

### 4.1 Saved fields

The stored document is `{ v = SCHEMA, lock = {owner, placeId, stamp} or nil, data = profile }` (`ServerScriptService.GardenStore.lua:621-625`, `:521-523`). Store name `GardenProfiles`, key `p_<UserId>`, schema 1 (`:56`, `:62-63`). `Store.Default()` is the only place the shape is declared (`:220-283`).

| Field | Shape | Written (captureProfile) | Read (applyProfile) | Checked in sanitise |
|---|---|---|---|---|
| `cash` | number | `Persistence.lua:49` | `Persistence.lua:197` | `GardenStore.lua:318-326` |
| `best` | fruit name, or nil | `Persistence.lua:51-53` | `Persistence.lua:207-213` | `GardenStore.lua:328-332` |
| `stored` | {fruit name = count} | `Persistence.lua:55-59` | `Persistence.lua:215-219` | `GardenStore.lua:334` |
| `plot` | {fruit name = count} | `Persistence.lua:96-111`, `:114-116` | `Persistence.lua:308-314` | `GardenStore.lua:335` |
| `index` | {fruit name = 0 or 1} | `Persistence.lua:69-73` | `Persistence.lua:227-236` | `GardenStore.lua:343-354` |
| `tools` | {tool name = count} | `Persistence.lua:63-67` | `Persistence.lua:221-225` | `GardenStore.lua:358-380` |
| `kingOwned` | {effect key = true} | `Persistence.lua:77` | `Persistence.lua:268` | `GardenStore.lua:389-399` |
| `kingEquipped` | effect key | `Persistence.lua:78` | `Persistence.lua:268` | `GardenStore.lua:400-406` |
| `royal` | numbers, achievements, style | `Persistence.lua:81` | `Persistence.lua:269` | `GardenStore.lua:410-438` |
| `shards` | whole number | `Persistence.lua:74` | `Persistence.lua:237` | `GardenStore.lua:440-445` |
| `rebirths` | whole number | `Persistence.lua:75` | `Persistence.lua:267` | `GardenStore.lua:451-456` |
| `lastSeen` | os.time() | `Persistence.lua:87` | `Persistence.lua:324` | `GardenStore.lua:462-467` (clamped to now) |
| `upgrades` | {key = level} | `Persistence.lua:92-94` | `Persistence.lua:270` | `GardenStore.lua:475-497` |
| `savedAt`, `saveCount` | debug numbers | `GardenStore.lua:555-556` | `GardenStore.lua:777` | `GardenStore.lua:502-503` |
| `refundShards` | load-only, never saved | none | `Persistence.lua:243-253` | `GardenStore.lua:501` |

(File names in this table are shortened: `Persistence.lua` is `ServerScriptService.GardenService.Persistence.lua` and `GardenStore.lua` is `ServerScriptService.GardenStore.lua`.)

- Retired fields are read only to pay a refund, then dropped: `upgrades.fruitStand`, `tools.PlotAlarm`, `alarm`. The old `ripening` field is ignored (`ServerScriptService.GardenStore.lua:73-76`, `:337-339`, `:368-371`, `:384`, `:489-494`).
- **Not saved:** potion effects (`ServerScriptService.GardenService.lua:177-180`); the Clear, Merge All and Lock cooldowns and the gamepass cache, which are runtime tables cleared on leave (`ServerScriptService.GardenService.lua:724-728`); fruit positions, rotations and the Lucky flag, because the plot is saved as counts only (`ServerScriptService.GardenService.Persistence.lua:98`).

### 4.2 Attributes and instances

Player attributes:

| Attribute | Written at | Read by | Meaning |
|---|---|---|---|
| `CarryingFruit` | `Carrying.lua:142` (set), `:77` (clear) | `StarterPlayer.StarterPlayerScripts.GardenClient.lua:211` | Name of the carried item. Also holds a tool's name while a tool is carried. |
| `CarryingTool` | `Tools.lua:158` (set), `:177`, `:198`, `:276` (clear) | `StarterPlayer.StarterPlayerScripts.GardenClient.lua:225` | Tool name; swaps the client's carry buttons. |
| `ClearReadyAt` | `Merging.lua:425` | `StarterPlayer.StarterPlayerScripts.ClearMenu.lua:146` | Server time when Clear is allowed again. |
| `MergeAllReadyAt` | `Merging.lua:572` | `StarterPlayer.StarterPlayerScripts.MergeAllButton.lua:49` | Server time when Merge All is allowed again. |
| `LockEndsAt`, `LockReadyAt` | `RebirthLock.lua:238-239` | `StarterPlayer.StarterPlayerScripts.LockButton.lua:36`, `:42` | Dome end time; next allowed lock. |
| `<Effect>EndsAt`, `<Effect>Span` | `Tools.lua:217`, `:223` | `StarterPlayer.StarterPlayerScripts.GardenClient.lua:562`, `:579`; `StarterPlayer.StarterPlayerScripts.FeatherFall.lua:26` | Buff deadline and the length of the bar. Cleared on death and expiry (`ServerScriptService.GardenService.lua:560-563`, `:803-804`). |
| `OfflineEarned`, `OfflineAway`, `OfflineCounted` | `Persistence.lua:203-205`, `:328-330` | `StarterPlayer.StarterPlayerScripts.OfflinePopup.lua:113` | Cash paid for time away. |
| `ProfileLoaded` | `GardenStore.lua:800` | `StarterPlayer.StarterPlayerScripts.CashClient.lua:76`; `StarterPlayer.StarterPlayerScripts.OfflinePopup.lua:109` | The profile finished loading. |
| `Shards` | through `setShards` (`Progression.lua:216-218`) from `Tools.lua:106`, `RebirthLock.lua:115`, `Persistence.lua:237`, `:245`, `:265` | `Tools.lua:100`, `RebirthLock.lua:115`, `Persistence.lua:74` | Shard balance. |
| `Rebirths` | through `publishRebirths` (`Progression.lua:95-101`) from `RebirthLock.lua:118`, `Persistence.lua:267` | `RebirthLock.lua:117`, `Persistence.lua:75`, `Progression.lua:325` | Rebirth count. |
| `CashPerSecond` | `RebirthLock.lua:110` (zeroed) | `StarterPlayer.StarterPlayerScripts.TopEarnersBoard.lua:188` | Earning rate. |
| `CashMultiplier` | through `publishMultiplier` (`Progression.lua:341-343`) from `RebirthLock.lua:119` | `StarterPlayer.StarterPlayerScripts.CashClient.lua:107` | All cash bonuses as one number. |
| `BestMerge` | cleared `RebirthLock.lua:103`; set `Progression.lua:133` | (no client reader found by search) | Display name of the best fruit merged. |
| `KingEffect` | read `Persistence.lua:78` | | Equipped crown effect. |
| `Upgrade_<key>` | through `publishUpgrades` from `RebirthLock.lua:95`, `Persistence.lua:270` | `levelOf` (`Upgrades.lua:34-37`) | Upgrade levels. |

(Short names: `Carrying.lua`, `Tools.lua`, `Merging.lua`, `RebirthLock.lua`, `Persistence.lua`, `Progression.lua`, `Upgrades.lua` are the `ServerScriptService.GardenService.*.lua` files; `GardenStore.lua` is `ServerScriptService.GardenStore.lua`.)

Instance attributes:

| On | Attribute | Written at | Meaning |
|---|---|---|---|
| fruit | `HeldBy` | `ServerScriptService.GardenService.Carrying.lua:198` (set), `:209` (clear); `ServerScriptService.GardenService.Storage.lua:80` | UserId of the carrier. |
| fruit | `Busy` | `ServerScriptService.GardenService.Carrying.lua:242`, `:278`; `ServerScriptService.GardenService.Merging.lua:112`, `:134` | Mid-drop or mid-merge. |
| fruit | `PlotName` | `ServerScriptService.GardenService.Carrying.lua:216`; `ServerScriptService.GardenService.Merging.lua:193`, `:502`; `ServerScriptService.GardenService.Tools.lua:286`; `ServerScriptService.GardenService.FruitFactory.lua:359-367` | Which plot owns it. Cleared before a fruit is destroyed. |
| fruit | `Lucky` | read `ServerScriptService.GardenService.Carrying.lua:185`; set `ServerScriptService.GardenService.Upgrades.lua:384` | Grown by the Luck potion. |
| plot model | `LockedUntil` | `ServerScriptService.GardenService.RebirthLock.lua:233` | Cosmetic. The server reads `plot.lockedUntil` instead. |
| workspace | `StudioNoSave` | read `ServerScriptService.GardenStore.lua:572` | Studio only: skip every write. |

Instances:

- Folders on the Player: `StoredFruits` (`Config.StoredFolder`), `StoredTools` (`Config.ToolsFolder`), `IndexState` (`Config.IndexFolder`). Each holds one IntValue per name (`ServerScriptService.GardenService.Tools.lua:27-81`; `ServerScriptService.GardenService.Progression.lua:160-168`).
- `CarryWeld`, a Weld inside the carried item (`ServerScriptService.GardenService.Carrying.lua:134-139`).
- `MergeFlash`, a Highlight used for the white flash (`ServerScriptService.GardenService.Merging.lua:34-45`).
- Templates it clones: `ServerStorage.ClearSmoke`, `ServerStorage.ForcefieldDome`, `ReplicatedStorage.Tools.<name>`, and `ReplicatedStorage.CarryPose` as a Studio-only fallback.

### 4.3 Remotes

All are RemoteEvents in `ReplicatedStorage.GardenRemotes` (`ServerScriptService.GardenService.lua:43-86`).

| Remote | Direction | Payload | Handler |
|---|---|---|---|
| (ProximityPrompt `Interact`, not a remote) | client to server | the fruit | `merge` or `pickUp` (`ServerScriptService.GardenService.lua:522-526`) |
| `Drop` | client to server | none | `dropFromPlayer` (`:528`) |
| `Store` | client to server | none | `storeCarried` (`:533`) |
| `Equip` | client to server | fruit name (string) | `equipStored` (`:534`) |
| `BuyTool` | client to server | tool name (string) | `buyTool` (`:529`) |
| `EquipTool` | client to server | tool name (string) | `equipTool` (`:530`) |
| `Clear` | client to server | none | `clearLowest` (`:535`) |
| `MergeAll` | client to server | none | `mergeAllOn` (`:536`) |
| `Rebirth` | client to server | none | `doRebirth` (`:537`) |
| `Lock` | client to server | none | `lockPlot` (`:546`) |
| `Notify` | server to client | warning text (string) | red toast (`:66`) |
| `Notice` | server to client | good-news text (string) | green toast (`:70`) |
| `PromptGamePassPurchaseFinished` (Roblox signal) | Roblox to server | player, pass id, purchased | marks Merge All owned (`:541-545`) |

### 4.4 Config keys and constants (non-numeric)

Numeric values are in section 5. All `Config.*` lines are in `ReplicatedStorage.GardenConfig.lua`.

| Key | Value or role | Line |
|---|---|---|
| `CarryPoseName` | `"CarryPose"`, fallback animation | 350 |
| `ClearTemplate` | `"ClearSmoke"` | 1061 |
| `LockTemplate` | `"ForcefieldDome"` | 890 |
| `StoredFolder` | `"StoredFruits"` | 528 |
| `ToolsFolder` | `"StoredTools"` | 1604 |
| `ToolTemplates` | `"Tools"` | 1605 |
| `IndexFolder` | `"IndexState"` | 390 |
| `UpgradeKeys` | capacity, spawnRate, spawnTier, plotSize | 37 |
| `KingDefault` | `"Crown"` | 1235 |
| `MergeFlashColor` | white | 726 |
| `PotionDrinkSound` | `rbxassetid://120043778768093` | 1797 |
| `Tools` | the tool specs | 1615-1705 |
| `Effects` | the effect specs | 1714-1789 |
| `SpawnTable(level)` | spawn pool for a tier level | 1910-1912 |
| `StoredTotal(folder)` | total stored fruit; counts only real fruit names | 539-549 |
| `MaxLevel(key)` | top level of an upgrade ladder | 1905-1908 |
| `Plural`, `DurationText` | text helpers for toasts | 1978, 1805-1811 |
| `TesterShards` | shard floor per tester UserId | 410-413 |
| `KingEffectByKey` | lookup used by sanitise | 1243-1244 |
| Warning strings | `LoadingWarning` 1113, `NoPlotWarning` 1119, `OffPlotWarning` 322, `PlotFullWarning` 328, `EquipWarning` 1110, `StoredFull` 534 | |
| Clear / Merge All strings | `ClearOnlyKind`, `ClearEmptyPlot`, `ClearWait`, `ClearedToast` 1084-1087; `MergeAllWait`, `MergeAllNothing`, `MergeAllDone`, `MergeAllUnowned`, `MergeAllNoPass` 975-979 | |
| Rebirth strings | `RebirthNotReady`, `RebirthDone`, `RebirthNoPlot`, `RebirthBusy` 1036-1039 | |
| Lock strings | `LockCooldownWarning`, `LockNoPlot`, `LockOffPlot`, `LockAlreadyUp`, `LockTNTBlocked` 915-919 | |
| Tool strings | `ToolNotEnoughShards` through `TNTVictim` 1849-1857; `EffectDrunk` 1800 | |
| Save strings | `LoadFailedKick`, `LockedKick` 1114-1115; `NotDurableWarning` 1123-1124; `RefundNotice` 1151 | |

GardenStore constants (`ServerScriptService.GardenStore.lua`): `STORE_NAME = "GardenProfiles"` (`:56`), `PROBE_STORE = "GardenProbe"` (`:61`), `KEY_PREFIX = "p_"` (`:62`), `SCHEMA = 1` (`:63`), `MAX_COUNT = 1e6` (`:65`), `RETIRED_PRICES` (`:73-76`), `SESSION_ID` (`:80-82`), `ROYAL_NUMBERS` (`:306-307`).

### 4.5 Functions this subsystem puts into G

- Carrying: `dropAt`, `dropFromPlayer`, `holdFruit`, `ownsFruit`, `pickUp`, `releaseHold` (`ServerScriptService.GardenService.Carrying.lua:356-358`).
- Tools: `addStored`, `addTool`, `buyTool`, `equipTool`, `storedFolder`, `stowTool`, `toolsFolder`, `useTool` (`ServerScriptService.GardenService.Tools.lua:314-317`).
- Storage: `equipStored`, `storeCarried` (`ServerScriptService.GardenService.Storage.lua:91`).
- Merging: `clearLowest`, `clearReady`, `merge`, `mergeAllOn`, `mergeAllReady`, `ownsMergeAll` (`ServerScriptService.GardenService.Merging.lua:579-582`).
- RebirthLock: `doRebirth`, `lockPlot`, `lockReadyAt`, `rebirthing` (`ServerScriptService.GardenService.RebirthLock.lua:244-245`).
- Persistence: `applyProfile`, `captureProfile`, `reclaimCarried` (`ServerScriptService.GardenService.Persistence.lua:363-364`).

## 5. Numbers

All `Config.*` lines are in `ReplicatedStorage.GardenConfig.lua`.

| What | Value | Config key | Line |
|---|---|---|---|
| Server reach for pick up and merge | 16 studs | `MaxReach` | 305 |
| Prompt range | 12 studs | `PromptDistance` | 304 |
| Gap between head and carried fruit | 0.6 studs | `CarryGap` | 306 |
| Drop lands this far in front | 3 studs | `DropForwardStuds` | 307 |
| Drop arc height / rise / fall | 2.2 studs / 0.16 s / 0.22 s | `DropLift`, `DropRiseTime`, `DropFallTime` | 312-314 |
| Landing hop | 0.35 studs, 0.14 s each way | `DropHop`, `DropHopTime` | 315-316 |
| Slack for "standing on your plot" | 2.5 studs sideways, 12 studs up | `PlotStandMargin`, `PlotStandBand` | 321, 940 |
| Carry animation | asset 121986414905125, 0.2 s fade | `CarryAnimationId`, `CarryFadeTime` | 349, 351 |
| Plot capacity | 6 at level 0, up to 35 | `CapacityStart`, `CapacityTop` | 51-52 |
| Starter fruit after rebirth or for a new player | 3 | `InitialFruits` | 287 |
| Storage cap (all kinds together) | 5 | `StoredCap` | 533 |
| Merge wind-up / settle | 0.26 s / 0.32 s | `MergeSpinTime`, `MergeSettleTime` | 720-721 |
| Merge spins / settle spins / dip / punch | 1.75 / 0.6 / 0.88x / 1.18x | `MergeSpins`, `MergeSettleSpins`, `MergeDip`, `MergePunch` | 722-725 |
| Pending-merge backup release | spin + settle + 5 s | (code) | `ServerScriptService.GardenService.Merging.lua:80` |
| Clear cooldown | 5 s | `ClearCooldown` | 1056 |
| Clear puff particles / host life | 3 / 2 s | `ClearPuffs`, `ClearSmokeLife` | 1062-1063 |
| Merge All gamepass | id 1982540690 (comment says 79 Robux; the price lives on the pass) | `MergeAllGamepassId` | 950, 961 |
| Merge All cooldown | 300 s | `MergeAllCooldown` | 973 |
| Merge All puff particles | 2 per fruit | `MergeAllPuffs` | 985 |
| Rebirth needs the full Index | true | `RebirthNeedsAll` | 993 |
| Cash bonus per rebirth (stacks) | +1.00 (x2, x3, ...) | `RebirthCashBonus` | 1000 |
| Lockdown length / cooldown after it ends | 60 s / 30 s | `LockDuration`, `LockCooldown` | 885-886 |
| Dome overhang | +35 across, +22 along | (code) | `ServerScriptService.GardenService.RebirthLock.lua:173` |
| TNT | 2500 shards, radius 18, carried at 2.5 studs | `Tools.TNT` | 1616-1626 |
| Speed Potion | 5000 shards, 300 s, walk x1.6 | `Tools.SpeedPotion`, `Effects.Speed` | 1631-1639, 1721 |
| Jump Potion | 5000 shards, 300 s, jump power 85 | `Tools.JumpPotion`, `Effects.Jump` | 1640-1649, 1727 |
| Potion of Aether | 10000 shards, 1800 s | `Tools.AetherPotion` | 1655-1666 |
| Invisibility Potion | 5000 shards, 60 s | `Tools.InvisibilityPotion` | 1667-1678 |
| Feather Falling Potion | 5000 shards, 600 s, fall cap 22 studs/s | `Tools.FeatherFallingPotion`, `Effects.FeatherFalling` | 1685-1691, 1772 |
| Luck Potion | 7500 shards, 300 s, +1 spawn level (0.35 tilt at the top) | `Tools.LuckPotion`, `Effects.Luck` | 1698-1704, 1786-1787 |
| Drink sound volume / range | 0.65 / 60 studs | `PotionDrinkVolume`; range in code | 1798; `ServerScriptService.GardenService.Tools.lua:234` |
| Offline rate / cap / minimum | 3% / 5 h / 1 cash | `OfflineRate`, `OfflineCapSeconds`, `OfflineMinimum` | 753-754, 763 |
| Autosave interval / Touch delay | 60 s / 8 s | `SaveInterval`, `SaveTouch` | 1131-1132 |
| Write retries / base delay | 5 / 1 s, doubling with jitter | `SaveRetryCount`, `SaveRetryBase` | 1133-1134 |
| Shutdown retries / deadline | 2 / 25 s | `ShutdownRetryCount`, `ShutdownDeadline` | 1135-1136 |
| Stale lock / wait between tries / tries | 120 s / 7 s / 20 | `LockStaleSeconds`, `LockWaitDelay`, `LockWaitAttempts` | 1144-1146 |
| Refund toast delay | 6 s | `RefundNoticeDelay` | 1152 |
| Tester shard floors | 260000 and 100000 | `TesterShards` | 410-413 |
| Retired refunds | Fruit Stand 1000 + 5000, Plot Alarm 2500 | `RETIRED_PRICES` (GardenStore) | `ServerScriptService.GardenStore.lua:73-76` |
| Max count per name before it is called corrupt | 1,000,000 | `MAX_COUNT` (GardenStore) | `ServerScriptService.GardenStore.lua:65` |

## 6. Extension points for the expansion

Feature numbers match the brain dump list. Lines marked "suggestion" are proposals, not existing code.

### 1. Rebirth-gated rarity, +1 storage per rebirth, harder rebirth, slight boost

- **Merge cap.** Hook in `merge` right after `FruitIndex.MergeInto` (`ServerScriptService.GardenService.Merging.lua:179`) and before the commit at `:183`. Refuse when the promoted fruit's tier is above the player's unlocked tier. Suggestion: unlocked tier = `1 + Rebirths`, which gives COMMON only at 0 rebirths. Apple is rank 60, the top COMMON fruit (`ReplicatedStorage.Fruits.FruitIndex.lua:69`), so that matches "max fruit is Apple". Nine tiers means eight rebirths to reach TRANSCENDENT (`ReplicatedStorage.Fruits.FruitIndex.lua:10-73`).
- **Merge All.** `cascade` (`ServerScriptService.GardenService.Merging.lua:289-346`) takes only counts. It needs a top-allowed-rank argument so pairs at the cap stay paired. Its caller is `:483`.
- **Spawning.** Spawn tiers run 0 to 63 and level L grows the bottom L+1 fruit (`ReplicatedStorage.GardenConfig.lua:68-96`), so Apple is reachable at level 4. The spawn-tier purchase (`ServerScriptService.GardenService.Upgrades.lua:301-343`) and the Luck potion's +1 level (`ServerScriptService.GardenService.Upgrades.lua:372-379`) both need the cap. The pick-up discovery (`ServerScriptService.GardenService.Carrying.lua:185-196`) and the load-time backfill (`ServerScriptService.GardenService.Persistence.lua:288-290`) both discover by spawn tier, so they also need it.
- **Index gate.** `indexComplete` counts all 64 fruit (`ServerScriptService.GardenService.Progression.lua:311-320`). Under the cap that is impossible at 0 rebirths. It must become "every fruit at or below your unlocked tier".
- **Shard and money cost.** Add checks next to the Index gate (`ServerScriptService.GardenService.RebirthLock.lua:52-55`), before `rebirthing[player] = true` (`:57`). Cash is set to 0 anyway (`:109`), so a money cost is a threshold, not a deduction. Subtract the shard cost where the payout is added (`:115`). Keep it yield-free (`:63-67`).
- **+1 storage.** `storeCarried` reads the constant `Config.StoredCap` (`ServerScriptService.GardenService.Storage.lua:43-44`). Both client counters read the same constant (`StarterPlayer.StarterPlayerScripts.GardenClient.lua:399-400`, `StarterPlayer.StarterPlayerScripts.StoredFruitsMenu.lua:467-468`). Suggestion: one shared function in GardenConfig that takes the rebirth count, used by all three.
- **Slight boost.** Today it is +100% per rebirth, stacking (`ReplicatedStorage.GardenConfig.lua:995-1000`), and it also scales offline pay (`ServerScriptService.GardenService.Persistence.lua:191`).
- **What could break.** Existing saves hold fruit far above Apple, and `restorePlot` puts them all back (`ServerScriptService.GardenService.Persistence.lua:124-168`), so a grandfathering rule is needed. The rebirth payout is the Index's claimed shards (`ServerScriptService.GardenService.RebirthLock.lua:61`); a tier-limited Index pays far less early on. The notice formats the multiplier with `%d` (`ReplicatedStorage.GardenConfig.lua:1037`; `ServerScriptService.GardenService.RebirthLock.lua:144-145`) and the Rebirth menu formats the bonus with `+%d%%` (`ReplicatedStorage.GardenConfig.lua:1032`; `StarterPlayer.StarterPlayerScripts.RebirthMenu.lua:88`), so a fractional boost will not display correctly.

### 2. Bats that upgrade each rebirth

- **Where.** `equipTool` (`ServerScriptService.GardenService.Tools.lua:116-159`) and the kind branch in `useTool` (`:250`).
- **Change.** Add a new kind branch before the TNT code starts at `:255`. Do not reuse `holdFruit` as is: it welds the item above the head (`ServerScriptService.GardenService.Carrying.lua:122-139`), which is wrong for a held bat. Derive the bat level and colour from the `Rebirths` attribute (`ServerScriptService.GardenService.RebirthLock.lua:117-118`) so nothing new needs saving. Rebirth already keeps tools (`:85-88`).
- **What could break.** Any kind that is not `"consume"` runs the TNT code (`ServerScriptService.GardenService.Tools.lua:250-310`), and a spec with no `radius` errors at `:283`. The Drop remote means "use" for any carried tool (`ServerScriptService.GardenService.Carrying.lua:297-305`). The existing code deliberately never flings players (`ServerScriptService.GardenService.Tools.lua:295`), so knockback is new ground. Lockdown only blocks TNT (`:265-270`).

### 3. Explosives that drain fruit health

- **Where.** The destroy loop in `useTool` (`ServerScriptService.GardenService.Tools.lua:278-290`).
- **Change.** Replace destroy with damage, and destroy only at 0 health. Fruit have no health today; it would be set where fruit are built (`ServerScriptService.GardenService.FruitFactory.lua:224`, `:359-367`). TNT tiers and nukes are more `Config.Tools` entries with `kind = "place"`, a `radius`, and a new damage field (`ReplicatedStorage.GardenConfig.lua:1615-1626`). Ranged artillery needs a new target path: today you must stand on the target plot and the blast centres on your own body (`ServerScriptService.GardenService.Tools.lua:259-263`, `:272`).
- **What could break.** Health would not survive a rejoin. The plot is saved as name counts only (`ServerScriptService.GardenService.Persistence.lua:98`), and `sanitiseCounts` rejects anything that is not a number (`ServerScriptService.GardenStore.lua:285-304`). A separate saved field is safer than changing `plot`. `merge` compares names only (`ServerScriptService.GardenService.Merging.lua:168`), so a damaged fruit merges like a fresh one unless that changes. Income ignores health (`ServerScriptService.GardenService.FruitFactory.lua:39-45`). Keep skipping Busy fruit (`ServerScriptService.GardenService.Tools.lua:282`).

### 4. Underground base and ores

- **Where.** A shard purchase like `buyTool` (`ServerScriptService.GardenService.Tools.lua:91-112`), saved as a new profile field in `Store.Default` (`ServerScriptService.GardenStore.lua:220-283`), in `sanitise` (`:309-506`), and in `captureProfile` / `applyProfile`.
- **Do not put the unlock in `Config.UpgradeKeys`.** Rebirth resets every one of those levels (`ServerScriptService.GardenService.RebirthLock.lua:95`) and sanitise clamps them against the cash price ladders (`ServerScriptService.GardenStore.lua:475-488`).
- **Do not add ores to FruitIndex.** It is generated by an importer (`ReplicatedStorage.Fruits.FruitIndex.lua:1-3`). Spawn tiers are built from `FruitIndex.Count` (`ReplicatedStorage.GardenConfig.lua:86-96`), and the rebirth gate counts it (`ServerScriptService.GardenService.Progression.lua:315`).
- **Ores need their own saved tables and their own Index category.** Names outside FruitIndex are dropped from `stored` and `plot` on load (`ServerScriptService.GardenStore.lua:289`), on save (`ServerScriptService.GardenService.Persistence.lua:56`), and in the storage count (`ReplicatedStorage.GardenConfig.lua:543`). Index entries are filtered the same way (`ServerScriptService.GardenService.Persistence.lua:70`; `ServerScriptService.GardenStore.lua:345`).
- **Ore merging cannot reuse `merge` directly.** It uses `FruitIndex.MergeInto` (`ServerScriptService.GardenService.Merging.lua:179`) and plot ownership through `PlotName` (`ServerScriptService.GardenService.Carrying.lua:153-156`; `ServerScriptService.GardenService.PlotRegistry.lua:163-165`). The prompt router only handles parts in the fruit folder (`ServerScriptService.GardenService.lua:524`).
- **What could break.** Rebirth only wipes what it lists, so decide whether the underground and ores survive. The self-test asserts the worst payload is under 8192 bytes (`ServerScriptService.GardenStore.lua:1209`). Adding fields needs no schema bump (`ServerScriptService.GardenStore.lua:8-12`); bumping it kicks players whose document is newer than their server (`:764-772`).

### 5. Forest pets

- **Taming.** Feeding uses up the carried fruit the same way storing does: `releaseHold(player, true)` (`ServerScriptService.GardenService.Storage.lua:49`). Check the held fruit's name against the request first.
- **Prompts.** `PromptTriggered` sends prompts on fruit-folder parts to `merge` or `pickUp` (`ServerScriptService.GardenService.lua:522-526`). A pet needs its own handler.
- **Merging two pets.** Copy `merge`'s checks (ownership, Busy, the pending ledger) for a separate pet ladder.
- **Saving.** A new `pets` field in `Store.Default`, `sanitise`, `captureProfile` and `applyProfile` (see Gotchas on sanitise).
- **Earnings.** Offline pay counts only fruit on the plot (`ServerScriptService.GardenService.Persistence.lua:191`). Pet income must be added there and in the live income tick, or pets pay nothing offline. The cash multiplier is defined once in `ServerScriptService.GardenService.Progression.lua:332-336`.

### 6. Boats and Aqua Affinity

- **Buying.** Shards are spent the way `buyTool` does it (`ServerScriptService.GardenService.Tools.lua:100-107`).
- **Saving.** A boat in `Config.Tools` would be saved for free (`ServerScriptService.GardenStore.lua:358-367`; `ServerScriptService.GardenService.Persistence.lua:63-67`). But `equipTool` would weld it above the head (`ServerScriptService.GardenService.Tools.lua:155`), and `useTool` would run the TNT code on it (`:250-255`). Boats need their own kind and branch, or their own store.
- **Breaking boats with artillery.** `useTool` only hits fruit on the plot you stand on (`:259`, `:279`). Water targets need a new path.
- Aqua Affinity is a potion; see feature 7. Swim fatigue and removing currents are outside this subsystem.

### 7. More potions

- **Recipe.**
  - A `Config.Tools` entry with `kind = "consume"`, `display`, `price`, `effect`, `duration`, and no `carrySize` for bottles (`ReplicatedStorage.GardenConfig.lua:1615-1705`, `:1628-1630`; `ServerScriptService.GardenService.Tools.lua:149-151`).
  - A `Config.Effects` entry with `display`, `bar`, `icon` (`ReplicatedStorage.GardenConfig.lua:1714-1789`).
  - A template at `ReplicatedStorage.Tools.<name>` (`ServerScriptService.GardenService.Tools.lua:133-139`; `ReplicatedStorage.GardenConfig.lua:1601-1603`).
- `consumeTool` is generic (`ServerScriptService.GardenService.Tools.lua:186-242`). The client gets `<Effect>EndsAt` and `<Effect>Span` automatically (`:217`, `:223`).
- **The effect needs a reader, or it does nothing.** Speed and Jump are applied in `applyWalkSpeed` (`ServerScriptService.GardenService.lua:210-233`), Invisible in `applyVisibility` (`:258-387`), Luck in `spawnPlan` (`ServerScriptService.GardenService.Upgrades.lua:372-379`).
- **What could break.** Effects are not saved (`ServerScriptService.GardenService.lua:177-180`) and are lost on death (`:601-604`). Stacking has no cap (`ServerScriptService.GardenService.Tools.lua:200-208`). Removing a potion from `Config.Tools` later deletes everyone's stock on load (`ServerScriptService.GardenStore.lua:372-375`) with no refund, unless it is added to `RETIRED_PRICES` (`:73-76`).

### 8. Plot visual overhaul

- **Where.** The dome is sized from the soil plus a 35/22 overhang and sits on `plot.topY` (`ServerScriptService.GardenService.RebirthLock.lua:171-180`). Drops land on `plot.topY` (`ServerScriptService.GardenService.Carrying.lua:228`). Placement and "standing on plot" use `plot.soil` (`ServerScriptService.GardenService.FruitFactory.lua:63-121`).
- **Change.** Keep one flat soil part per plot doing the same job. Re-check the dome overhang against new fences.
- **What could break.** Decoration placed inside the fruit folder could be counted as fruit; the clear puff deliberately parents to workspace for this reason (`ServerScriptService.GardenService.Merging.lua:237-240`).

### 9. UI overhaul

- No server logic needs to change. Every toast text is a `Config` string sent through `Notify` (red) or `Notice` (green) (`ServerScriptService.GardenService.lua:66-70`). Change the strings in Config, not in the scripts.
- A new UI must keep reading the attributes in section 4.2.
- **What could break.** The client copies `Config.StoredCap` (`StarterPlayer.StarterPlayerScripts.GardenClient.lua:399-400`; `StarterPlayer.StarterPlayerScripts.StoredFruitsMenu.lua:467-468`) and formats `RebirthCashBonus` itself (`StarterPlayer.StarterPlayerScripts.RebirthMenu.lua:88`).

### 10. Royal combat weapon skins

- **Where.** Owned and selected skins belong in `profile.royal`, captured through `G.royalCapture` (`ServerScriptService.GardenService.Persistence.lua:81`) and restored through `G.royalApply` (`:269`).
- **Change.** `sanitise` keeps only `ROYAL_NUMBERS`, known achievements and `style` (`ServerScriptService.GardenStore.lua:306-307`, `:410-438`). A new royal key must be added there.
- **What could break.** Skip that, and skins save once and vanish on the next load. Rebirth does not touch `royal`, so skins survive a rebirth.

## 7. Gotchas

1. **Rank runs backwards.** Rank 1 is the best fruit, so "lowest fruit" means the highest rank number. Reading a comparison the wrong way deletes the best fruit (`ServerScriptService.GardenService.Merging.lua:353-356`).
2. **The weld order is load-bearing.** Position, parent, zero velocity, hand over ownership, then weld. Changing it brings back the "teleported into the fruit" bug (`ServerScriptService.GardenService.Carrying.lua:102-121`). `dropAt` is the mirror image (`:218-221`).
3. **Busy is the real rule; `prompt.Enabled` is cosmetic.** GardenClient turns prompts back on five times a second (`ServerScriptService.GardenService.Carrying.lua:232-241`). Every exit path of an animation must clear Busy, or the fruit can never be picked up again (`:268-278`).
4. **Only TNT skips Busy fruit.** `useTool` does (`ServerScriptService.GardenService.Tools.lua:282`). `clearLowest` checks only `HeldBy` (`ServerScriptService.GardenService.Merging.lua:406`), and `mergeAllOn` checks neither (`:494-504`). So Clear and Merge All can consume a fruit that is still in its drop arc or still settling.
5. **Module order matters.** Modules copy from G when required (`ServerScriptService.GardenService.lua:487-510`). A function defined by a later module must be looked up through G at call time, as Carrying does (`ServerScriptService.GardenService.Carrying.lua:352-353`).
6. **`stowTool` and `useTool` are not local.** Tools.lua defines them with plain `function` (`ServerScriptService.GardenService.Tools.lua:172`, `:244`), so they are globals of that module. It works because the same module exports them to G (`:314-317`).
7. **`CarryingFruit` also holds tool names.** `holdFruit` sets it for whatever it holds (`ServerScriptService.GardenService.Carrying.lua:142`), and `equipTool` calls `holdFruit` (`ServerScriptService.GardenService.Tools.lua:155`). Read `CarryingTool` to tell them apart.
8. **Every non-"consume" tool is TNT.** `useTool` has only two branches (`ServerScriptService.GardenService.Tools.lua:250-310`).
9. **Drop on a tool means use.** A silent drop (death, leaving) stows it instead, so TNT never goes off by accident (`ServerScriptService.GardenService.Carrying.lua:297-306`).
10. **The pending-merge ledger must be cleared** on rebirth (`ServerScriptService.GardenService.RebirthLock.lua:79`) and when a plot is released (`ServerScriptService.GardenService.lua:475`). Otherwise a stale entry brings a fruit back on the next save.
11. **Ask `discover` before the merge animation.** It answers "first time" exactly once (`ServerScriptService.GardenService.Merging.lua:199-205`).
12. **`cascade` depends on a positional insert.** Appending instead leaves mergeable pairs on the plot with no error (`ServerScriptService.GardenService.Merging.lua:297-302`, `:326`).
13. **The storage cap only applies on the way in.** Restore overflow goes into storage with no cap (`ReplicatedStorage.GardenConfig.lua:530-532`; `ServerScriptService.GardenService.Persistence.lua:310-314`).
14. **A silent drop can overfill a plot by one.** It skips the capacity check on purpose (`ServerScriptService.GardenService.Carrying.lua:339-342`).
15. **Rebirth must stay yield-free.** No `task.wait` may go inside it, or an autosave could land between the wipe and the reward (`ServerScriptService.GardenService.RebirthLock.lua:63-67`). The payout must be computed before the Index is wiped (`:59-61`).
16. **Rebirth does not reset everything.** It keeps shards, tools, `kingOwned`, `royal`, any running potion, and the Clear and Merge All cooldowns. None of these appear in `doRebirth` (`ServerScriptService.GardenService.RebirthLock.lua:35-146`).
17. **A new saved field must also be added to `sanitise`.** `Default()` declares it, but `sanitise` copies only the keys it names, so a field missing there survives one save and is dropped on the next load (`ServerScriptService.GardenStore.lua:447-450`).
18. **Names are checked against FruitIndex everywhere.** Any non-fruit placed in `stored`, `plot` or `index` is silently thrown away (`ServerScriptService.GardenStore.lua:289`, `:345`; `ServerScriptService.GardenService.Persistence.lua:56`, `:70`).
19. **The plot is saved as counts.** Position, rotation, the Lucky flag and any future per-fruit state are lost on rejoin. Restore places fruit rarest first at random spots (`ServerScriptService.GardenService.Persistence.lua:98`, `:124-168`).
20. **Capture and apply must stay yield-free,** and the carried item must be reclaimed before the save (`ServerScriptService.GardenService.Persistence.lua:35-37`; `ServerScriptService.GardenService.lua:704-707`).
21. **Touch does nothing until the session is ready** (`ServerScriptService.GardenStore.lua:860-866`). `commit` refuses while another write is in flight (`:578-580`), so the rebirth ForceSave can fail and only logs a warning (`ServerScriptService.GardenService.RebirthLock.lua:134-142`).
22. **Losing the lock stops the session cold.** The session is marked failed and never saves again (`ServerScriptService.GardenStore.lua:634-641`). Every action that checks `Store.IsReady` (`:932-935`) then answers "Still loading your garden" for the rest of that visit.
23. **Cooldowns reset on rejoin.** The Merge All, Clear and Lock timers and the gamepass cache are cleared when the player leaves (`ServerScriptService.GardenService.lua:724-728`).
24. **One Config comment about the dome is stale.** Config says "Nobody else can walk in" (`ReplicatedStorage.GardenConfig.lua:883-884`). The code makes the dome non-colliding (`ServerScriptService.GardenService.RebirthLock.lua:183`), and the later Config note says it is only a TNT shield (`ReplicatedStorage.GardenConfig.lua:897-900`).
25. **TNT works on anyone's plot, including your own.** Only lockdown stops it (`ServerScriptService.GardenService.Tools.lua:161-168`, `:265-270`).
26. **The carry pose fallback works only in Studio.** Without `CarryAnimationId`, the runtime-registered id does not exist for real players (`ServerScriptService.GardenService.Carrying.lua:40-55`; `ReplicatedStorage.GardenConfig.lua:338-345`).
27. **Two different things are called "Store".** The `Store` remote banks the carried fruit (`ServerScriptService.GardenService.lua:60`). `Store` in code is the GardenStore save module (`:30`).
28. **Studio saving traps.** With API access off, Studio uses an in-memory store that forgets on Stop (`ServerScriptService.GardenStore.lua:197-203`). `workspace.StudioNoSave = true` skips every write (`:572-575`).
29. **A failed gamepass check is not cached,** so each press while Roblox's web call is failing makes a new call (`ServerScriptService.GardenService.Merging.lua:275-277`).
