---
tags: [current-game, code-map]
date: 2026-10-04
sources:
  - ServerScriptService.GardenService.Crown.lua
  - ServerScriptService.GardenService.CrownArena.lua
  - ServerScriptService.GardenService.CrownContest.lua
  - ServerScriptService.GardenService.CrownDecree.lua
  - ReplicatedStorage.CrownConfig.lua
  - ServerScriptService.GardenService.lua (wiring and applyWalkSpeed only)
  - ServerScriptService.GardenService.Progression.lua (refreshKing and cashMultiplier only)
  - ServerScriptService.GardenService.Persistence.lua (save and load lines only)
  - ServerScriptService.GardenStore.lua (royal sanitise only)
  - ReplicatedStorage.Fruits.FruitIndex.lua (COMMON fruits only)
---
# Royal System - Server

All files are in `/Users/samarthvaka/fruit-empire/src-export-2026-10-04`. Citations use the export file name and line, for example `ServerScriptService.GardenService.Crown.lua:314`. Related: [[Brain Dump 2026-10-04]].

## What it does

The Royal System is the "King's Throne". It is the server side of the King feature in the sky castle. One player at a time holds the crown. When the throne is empty, the eligible player with the highest income per second claims it (`ServerScriptService.GardenService.Crown.lua:796-817`). After that the crown only changes hands three ways: a Crown Duel is won, a Crown Contest is won, or the King leaves the server (`ServerScriptService.GardenService.Crown.lua:4-9`). A Crown Duel is a 1v1 sword fight in the Royal Arena, scored by ring-outs. A ring-out means knocking the other fighter off the ring. A Crown Contest is a public brawl over a zone in the middle of the ring. The King can also start Royal Decrees, which are short server-wide events that pay shards to the players who take part. The system pays shards and Royal Reputation, never cash, and it never takes fruit, cash, shards, plots or Index entries from anyone (`ReplicatedStorage.CrownConfig.lua:15-21`).

## Scripts

| Script | Role | Lines |
|---|---|---|
| ServerScriptService.GardenService.Crown.lua | Who holds the crown. Eligibility, Empire Power, the crown accessory, castle colours, in-world boards, Royal Reputation, titles, crown styles, achievements, saving, reign records in DataStores, the royal remotes and the castle prompts. | 1223 |
| ServerScriptService.GardenService.CrownArena.lua | The Royal Arena. Fighters, the sword, combat actions, knockback, knights, the challenge queue, Crown Duels, Royal Trials, practice bouts, the Royal Gate, ring-out detection. | 1428 |
| ServerScriptService.GardenService.CrownContest.lua | The Crown Contest. The calendar, joining, the brawl, the Crown Zone, Kingdom Control, Claim the Throne, the final duel, the King's three contest abilities. | 590 |
| ServerScriptService.GardenService.CrownDecree.lua | Royal Decrees. Merge Rush, Fruit Frenzy, Royal Race, Crown Hunt, Empire Build. | 378 |
| ReplicatedStorage.CrownConfig.lua | Every tuning number, text string, sound, colour and style. Shared by the server and the clients RoyalUI and RoyalCombat (`ReplicatedStorage.CrownConfig.lua:11-13`). | 486 |

## How it works

### Terms used below

- **G**: the shared table every GardenService module receives. Each module is a `return function(G)` and reads and writes functions on G (`ServerScriptService.GardenService.Crown.lua:28`).
- **RoyalState**: a `Configuration` instance in ReplicatedStorage. The server writes HUD facts as attributes on it, so clients watch one object (`ServerScriptService.GardenService.Crown.lua:62-73`).
- **Marker**: a named part in `workspace["Sky Islands"]["Royal Capital"].Markers`. The server uses markers as teleport targets and prompt anchors (`ServerScriptService.GardenService.Crown.lua:80-85`).
- **Fighter record**: a Lua table for one fighter (a player or a knight). It holds stagger, cooldowns, block state, shields and contest power (`ServerScriptService.GardenService.CrownArena.lua:116-126`).
- **Bout**: one duel in the arena. Kinds are `crown`, `trial`, `spar` and `final` (`ServerScriptService.GardenService.CrownArena.lua:18-23`).
- **Crown Point**: one ring-out scored in a bout.
- **Stagger**: a number that grows with each hit you take. Knockback is multiplied by `1 + stagger / 100` (`ReplicatedStorage.CrownConfig.lua:88-90`, `ServerScriptService.GardenService.CrownArena.lua:546`).
- **Empire Power**: a player's `CashPerSecond` attribute (`ServerScriptService.GardenService.Crown.lua:296-298`).
- **Busy**: what royal event a player is in, such as `queue`, `duel`, `contest` or `spar` (`ServerScriptService.GardenService.Crown.lua:110`, `342-346`).

### 1. Loading

1. GardenService requires the four modules in this order: Crown, CrownArena, CrownContest, CrownDecree. They load after Persistence (`ServerScriptService.GardenService.lua:512-518`).
2. Crown creates or finds the remotes `RoyalRequest`, `RoyalEvent` and `RoyalQuery` in `ReplicatedStorage.GardenRemotes` (`ServerScriptService.GardenService.Crown.lua:43-60`).
3. Crown creates `ReplicatedStorage.RoyalState` and wipes any old attributes on it (`ServerScriptService.GardenService.Crown.lua:64-70`).
4. CrownArena creates the `RoyalCombat` remote and reads the ring size from the `ArenaCenter` marker (`ServerScriptService.GardenService.CrownArena.lua:38-65`).
5. Later modules plug into earlier ones through G. For example Crown asks `G.royalMode` (made in CrownArena) and `G.royalQueries` (made in CrownDecree) only at call time (`ServerScriptService.GardenService.Crown.lua:805`, `964-965`).

### 2. Players and saving

1. On join, `onPlayer` stamps the join time and publishes the royal attributes. It also re-attaches the crown on respawn if the player is King (`ServerScriptService.GardenService.Crown.lua:1045-1061`).
2. Persistence calls `G.royalApply(player, profile.royal)` on load (`ServerScriptService.GardenService.Persistence.lua:269`). `royalApply` copies only valid numbers, known achievements and a known style (`ServerScriptService.GardenService.Crown.lua:270-285`).
3. Persistence calls `G.royalCapture(player)` on save (`ServerScriptService.GardenService.Persistence.lua:81`). `royalCapture` adds the live reign into longestReign, totalReign and bestStreak if the player is King right now (`ServerScriptService.GardenService.Crown.lua:252-269`).
4. GardenStore cleans the `royal` table again on every load and save (`ServerScriptService.GardenStore.lua:306-307`, `409-436`).

### 3. Claiming an empty throne

1. `crownRefresh` runs once a second from the main loop and on every cash change (`ServerScriptService.GardenService.Crown.lua:1155`, `ServerScriptService.GardenService.Progression.lua:54`).
2. If a King reigns, it only checks that the King is still in the server (`ServerScriptService.GardenService.Crown.lua:800-804`).
3. If no King reigns and the arena is not in use (`G.royalMode()` is nil), it waits `C.ClaimDelay` seconds, then picks `strongestWorthy()` (`ServerScriptService.GardenService.Crown.lua:805-816`).
4. `worthy(player)` needs a loaded profile and either 1+ rebirths or at least `C.MinDiscovered` fruits in the Index folder (`ServerScriptService.GardenService.Crown.lua:314-323`).
5. `stronger(a, b)` ranks by Empire Power, then cash, then the lower UserId (`ServerScriptService.GardenService.Crown.lua:303-309`).
6. The winner is crowned with `how = "wealth"`, or `"heir"` if anyone was crowned before in this server (`ServerScriptService.GardenService.Crown.lua:813`).

### 4. Crowning

1. `crownPlayer(newKing, how)` ends the old reign with `endReign`, which records the reign, writes it to the DataStores and removes the old crown (`ServerScriptService.GardenService.Crown.lua:719-737`).
2. It resets protection, vulnerability, defenses and declines. It adds 1 to `crowns` and adds `C.Rep.Claim` for wealth or heir crownings (`ServerScriptService.GardenService.Crown.lua:741-756`).
3. `publishCrown` writes the King to RoyalState, to `workspace` attributes and to each player's `IsKing` attribute (`ServerScriptService.GardenService.Crown.lua:691-715`).
4. `paintCastle` recolours every tagged castle part in the King's crown style (`ServerScriptService.GardenService.Crown.lua:532-573`).
5. It fires the `crowned` event, then attaches the crown accessory 1.25 seconds later (`ServerScriptService.GardenService.Crown.lua:760-768`).
6. `attachCrown` clones `ReplicatedStorage.KingAssets.RoyalCrown`, paints it, scales it to the head and welds every part to an invisible handle (`ServerScriptService.GardenService.Crown.lua:429-517`).
7. It calls `G.royalOnCrowned`, which removes the new King from the challenge queue and voids any challenge aimed at the old King (`ServerScriptService.GardenService.Crown.lua:769`, `ServerScriptService.GardenService.CrownArena.lua:1107-1117`).

### 5. The main loop (once a second)

The loop is at `ServerScriptService.GardenService.Crown.lua:1144-1184`. Each tick it:
- runs `crownRefresh`;
- gives the King `C.Rep.PerReignTick` every `C.ReignTick` seconds (`ServerScriptService.GardenService.Crown.lua:1157-1160`);
- announces "open to challengers" once when protection ends, if 2+ players are in and no arena event runs (`ServerScriptService.GardenService.Crown.lua:1162-1168`);
- writes `RoyalEligible` and `RoyalWhy` on every player (`ServerScriptService.GardenService.Crown.lua:1170-1174`);
- redraws the in-world boards with `boardsTick` (`ServerScriptService.GardenService.Crown.lua:1085-1142`);
- reloads the global records every 300 seconds (`ServerScriptService.GardenService.Crown.lua:1176-1179`).

### 6. Challenging the King

1. A client sends `RoyalRequest("challenge")`, or a player uses the Altar prompt (`ServerScriptService.GardenService.Crown.lua:1014-1017`).
2. `handlers.challenge` checks there is a King, the player may challenge, the King is not protected, and the queue has room. Then it adds the player to the queue (`ServerScriptService.GardenService.CrownArena.lua:996-1018`).
3. `mayChallenge` adds to `worthy`: not the King, `C.MinServerTime` seconds in this server, no cooldown, and not busy (`ServerScriptService.GardenService.Crown.lua:328-339`).
4. `processQueue` takes the first valid challenger. It sets a pending challenge and sends the King a `prompt` event with a deadline of `C.ResponseTime` (`ServerScriptService.GardenService.CrownArena.lua:956-993`).
5. The King answers with `RoyalRequest("respond", true/false)` (`ServerScriptService.GardenService.CrownArena.lua:1020-1023`). No answer counts as a decline, unless declining is not allowed (`ServerScriptService.GardenService.CrownArena.lua:982-986`).
6. `resolve` starts the duel on accept. On decline it puts the challenger on `C.DeclinedCooldown` and calls `G.royalDeclined` (`ServerScriptService.GardenService.CrownArena.lua:909-954`).
7. `royalDeclined` returns `declined`, `vulnerable` (after `C.DeclinesToVulnerable` declines inside `C.DeclineMemory`), or `contest` (a decline while vulnerable) (`ServerScriptService.GardenService.Crown.lua:832-855`). A `contest` result starts a Crown Contest (`ServerScriptService.GardenService.CrownArena.lua:944-946`).

### 7. A bout

1. `startCrownDuel` makes two fighter records and calls `startBout` with target `C.DuelTarget` and an `onEnd` callback (`ServerScriptService.GardenService.CrownArena.lua:1063-1096`).
2. `startBout` sets `mode = "bout"`, runs `enterArena` on both, puts them on the `DuelPadA` and `DuelPadB` markers, and opens the Royal Gate (`ServerScriptService.GardenService.CrownArena.lua:743-755`).
3. `enterArena` drops any carried fruit, sets `InArena`, marks the player busy, resets stagger, gives the sword, and re-applies walk speed and visibility (`ServerScriptService.GardenService.CrownArena.lua:264-280`).
4. The bout runs intro (`C.DuelIntroTime`), then a countdown (`C.DuelCountdown`), then `fight` (`ServerScriptService.GardenService.CrownArena.lua:759-778`). Fighters are frozen in every state except `fight` (`ServerScriptService.GardenService.CrownArena.lua:140-144`).
5. `arenaTick` runs about 10 times a second while the arena is in use (`ServerScriptService.GardenService.CrownArena.lua:1306-1314`). It checks each fighter with `outOf`: no body, dead, fell below the ring, or outside the boundary too long (`ServerScriptService.GardenService.CrownArena.lua:1231-1246`).
6. A fall is caught at once. The root part is anchored so the fall sweep does not kill the player (`ServerScriptService.GardenService.CrownArena.lua:247-257`, `1259`).
7. `ringOutInBout` gives the other side a point, updates Underdog Momentum, and pauses for `C.PointPause` before placing both on their pads again (`ServerScriptService.GardenService.CrownArena.lua:783-818`).
8. After `C.DuelMaxTime` seconds the leader wins. A tie switches to sudden death (`ServerScriptService.GardenService.CrownArena.lua:1271-1282`).
9. Non-fighters who walk onto the ring are teleported to the stands (`ServerScriptService.GardenService.CrownArena.lua:1290-1303`).
10. `finishBout` calls `onEnd`, fires `duelEnd`, waits `C.ResultTime`, then clears the arena, sends everyone home, shuts the gate and calls `G.royalAfterBout`, which runs the queue again (`ServerScriptService.GardenService.CrownArena.lua:820-864`, `1100-1102`).
11. Crown Duel `onEnd`: if the King wins, `royalDefended` adds a defense and `C.DefenseProtection`; the King gets rep and shards; the challenger gets `C.ChallengeCooldown`. If the challenger wins, they get rep, shards and the crown (`how = "duel"`), and the old King gets `C.ChallengeCooldown` (`ServerScriptService.GardenService.CrownArena.lua:1070-1095`, `ServerScriptService.GardenService.Crown.lua:821-829`). A forfeit (leaving) pays no rep or shards but still counts the result (`ServerScriptService.GardenService.CrownArena.lua:1071-1089`).

### 8. Combat

1. The client sends `RoyalCombat(action, arg)`. The server ignores it unless the sender is a fighter, not out, and not frozen (`ServerScriptService.GardenService.CrownArena.lua:650-661`).
2. Actions: `strike`, `charge` then `heavy`, `dash`, `block`, `slam`, and the King's contest actions (`ServerScriptService.GardenService.CrownArena.lua:570-646`).
3. Every action checks a server cooldown with `ready(f, key, cd)` (`ServerScriptService.GardenService.CrownArena.lua:561-566`).
4. Hits are found by the server with `targets()`. It uses range, a facing arc, and a height difference under 7 studs. It does not use the sword's touch events (`ServerScriptService.GardenService.CrownArena.lua:482-505`).
5. `applyHit` is the one place a hit lands. It handles shields, Crown Guard, stagger, block, dash, Underdog, lift, and adds stagger (`ServerScriptService.GardenService.CrownArena.lua:539-558`).
6. `push` sends a `knockback` event to the victim's client and also adds a server `LinearVelocity` for 0.12 seconds (`ServerScriptService.GardenService.CrownArena.lua:510-536`).
7. There is no health damage. The only score is ring-outs. Knights have infinite health (`ServerScriptService.GardenService.CrownArena.lua:410-411`).
8. The sword is built in code from the classic sword mesh (`ServerScriptService.GardenService.CrownArena.lua:172-195`). Swing animations are the classic `toolanim` StringValues (`Slash`, `Lunge`) that the default Animate script plays (`ServerScriptService.GardenService.CrownArena.lua:229-243`).
9. Walk speed and jump inside the arena come from `G.arenaStats`, which `applyWalkSpeed` asks before any potion (`ServerScriptService.GardenService.CrownArena.lua:146-154`, `ServerScriptService.GardenService.lua:197-208`).

### 9. Trials, practice and watching

- **Royal Trial**: the King sends `trial`. For 20 seconds the first player to send `joinTrial` who may challenge gets a duel of kind `trial` (`ServerScriptService.GardenService.CrownArena.lua:1033-1060`). A trial defense gives the King 10 extra rep (`ServerScriptService.GardenService.CrownArena.lua:1076`).
- **Practice bout (spar)**: the Sparring Post prompt spawns a knight and runs a bout to `C.TrialTarget`. The first win ever pays `C.Rewards.FirstSpar` (`ServerScriptService.GardenService.CrownArena.lua:1121-1142`). It does not check worthiness.
- **Knights**: hand-built R6 rigs with a tabard in the King's banner colour (`ServerScriptService.GardenService.CrownArena.lua:336-460`). `knightThink` walks to the nearest target and strikes in range, every `C.Knight.Reaction` seconds (`ServerScriptService.GardenService.CrownArena.lua:1179-1224`, `1284-1289`).
- **Watching**: `watch` and the Royal Gate send players to a random `StandSpot` marker while an arena event runs (`ServerScriptService.GardenService.CrownArena.lua:1144-1175`).

### 10. Crown Contest

1. A calendar loop checks every second. When the timer is due and at least `C.Contest.MinPlayers` players are in and no bout runs, it calls `announce("scheduled")` (`ServerScriptService.GardenService.CrownContest.lua:551-564`).
2. `announce` opens the gate and starts a `C.Contest.AnnounceTime` countdown (`ServerScriptService.GardenService.CrownContest.lua:498-516`). Players send `joinContest`; worthy players are added and teleported to the `ArenaGate` marker (`ServerScriptService.GardenService.CrownContest.lua:519-538`).
3. `begin` waits if a bout is running, calls it off with fewer than `MinEntrants`, then hands the ring to the arena with `G.royalBeginContest` (`ServerScriptService.GardenService.CrownContest.lua:436-473`, `ServerScriptService.GardenService.CrownArena.lua:1372-1379`).
4. States: `intro` (3 s), `brawl` (`BrawlTime`), `crown` (`MaxTime`) (`ServerScriptService.GardenService.CrownContest.lua:478-495`).
5. `zoneTick` runs every `C.Contest.Tick` seconds in the `crown` state (`ServerScriptService.GardenService.CrownContest.lua:236-315`). Kingdom Control starts at 100. Challengers alone drain it, with diminishing returns for more challengers. The King alone refills it. Both together freezes it. Each draining challenger gains Crown Power equal to their share of the drain (`ServerScriptService.GardenService.CrownContest.lua:270-285`).
6. When control hits 0, `startFinal` picks the challenger with the most Crown Power and runs a `final` bout to `C.FinalTarget` (`ServerScriptService.GardenService.CrownContest.lua:150-231`).
7. If time runs out first, the King holds (`ServerScriptService.GardenService.CrownContest.lua:292-302`).
8. With no King, it becomes Claim the Throne. A lone player on the zone gains `C.ClaimRate` power per second and is crowned at `C.ClaimPower` (`ServerScriptService.GardenService.CrownContest.lua:255-268`).
9. King abilities, contest only: `shockwave`, `guard` (Crown Guard: no knockback for a while) and `summon` (Royal Guard knights, once, when control is at or under `GuardAt`) (`ServerScriptService.GardenService.CrownContest.lua:379-432`).
10. A player ring-out returns after 1.5 seconds. A guard knight ring-out is removed for good (`ServerScriptService.GardenService.CrownContest.lua:319-346`).
11. `finish` pays everyone (`ContestEntry` to entrants, `ContestSurvive` or `ContestWin` to the winner) and schedules the next contest (`ServerScriptService.GardenService.CrownContest.lua:110-147`).

### 11. Royal Decrees

1. The King sends `RoyalRequest("decree", key)`. The server checks King, a valid key, no decree running, `C.DecreeCooldown`, and `C.DecreeMinPlayers` (`ServerScriptService.GardenService.CrownDecree.lua:317-343`).
2. `start` builds the decree, runs its starter, then ticks every 0.5 seconds until success or `endsAt` (`ServerScriptService.GardenService.CrownDecree.lua:282-304`).
3. Starters (`ServerScriptService.GardenService.CrownDecree.lua:79-278`):
   - **MergeRush**: Merging calls `G.crownOnMerge` (`ServerScriptService.GardenService.CrownDecree.lua:312-315`). At the goal, every merger except the King gets shards.
   - **FruitFrenzy**: sets the global `G.spawnBoost`, which the Ambient spawn timer reads.
   - **RoyalRace**: players below `startBelow` (not the King, not busy) get a free Aether pass for the race. The first three to reach a fixed box near the castle get shards.
   - **CrownHunt**: raycasts for a random spot on the main island, outside the plots, and places a golden crown with a beam. The first non-King to touch it wins.
   - **EmpireBuild**: measures total `CashPerSecond` in the server. At +10% every non-King gets shards.
4. `finish` resets `G.spawnBoost` to 1, runs cleanup, and on success gives the King `C.Rep.Decree` (`ServerScriptService.GardenService.CrownDecree.lua:58-75`).
5. If a starter fails at once (no spot for the crown, no income yet), the decree cooldown is given back (`ServerScriptService.GardenService.CrownDecree.lua:334-342`).

### 12. Leaving and shutdown

- Crown: a leaving King vacates the throne; per-player tables are cleared; the royal record is kept 30 seconds so the leave-save can read it (`ServerScriptService.GardenService.Crown.lua:1070-1079`).
- CrownArena: the leaver is dropped from the queue; a pending challenge is voided; a fighter in a bout forfeits; a fighter in a contest is handed to `G.royalContestLeft` (`ServerScriptService.GardenService.CrownArena.lua:1344-1366`).
- CrownContest: if the King flees during the crown phase, the challenger with the most power is crowned; otherwise it becomes a claim (`ServerScriptService.GardenService.CrownContest.lua:354-375`).
- CrownDecree: a leaving King fails their decree (`ServerScriptService.GardenService.CrownDecree.lua:360-362`).
- On server close, the running reign is written to the records (`ServerScriptService.GardenService.Crown.lua:1065-1068`).

## Data

### Saved fields (player profile)

| Field | Type | Read / written at |
|---|---|---|
| `royal.rep` | whole number | `ServerScriptService.GardenService.Crown.lua:107`, `252-285` |
| `royal.crowns` | whole number | same |
| `royal.defenses` | whole number | same |
| `royal.duelWins` | whole number | same |
| `royal.contestWins` | whole number | same |
| `royal.longestReign` | seconds | same |
| `royal.totalReign` | seconds | same |
| `royal.decrees` | whole number | same |
| `royal.bestStreak` | whole number | same |
| `royal.spars` | whole number | same |
| `royal.achievements` | set of achievement keys = true | `ServerScriptService.GardenService.Crown.lua:265-267`, `277-281` |
| `royal.style` | crown style key | `ServerScriptService.GardenService.Crown.lua:254`, `282` |
| `shards` | written through `setShards` on the `Shards` attribute | `ServerScriptService.GardenService.Crown.lua:243-248`, `ServerScriptService.GardenService.Persistence.lua:74` |
| `rebirths`, `index` | read only, through the `Rebirths` attribute and the Index folder | `ServerScriptService.GardenService.Crown.lua:289-292`, `316` |

The same ten number names are also listed in `ServerScriptService.GardenStore.lua:306-307`.

### DataStores (global, not per profile)

| Store | Kind | Key | Value | Where |
|---|---|---|---|---|
| `CrownHistory` (`C.HistoryStore`) | DataStore | `"eternal"` | `{ name, userId, seconds, at }`, only raised | `ServerScriptService.GardenService.Crown.lua:605`, `663-670` |
| `CrownRecords` (`C.RecordsStore`) | OrderedDataStore | UserId as string | longest reign in seconds, only raised | `ServerScriptService.GardenService.Crown.lua:606`, `671-677` |

Scope is `"studio"` in Studio and `"live"` otherwise (`ServerScriptService.GardenService.Crown.lua:602`). Reigns under 60 seconds are not written (`ServerScriptService.GardenService.Crown.lua:664`).

### Remotes

| Name | Type | Direction | Payload | Where |
|---|---|---|---|---|
| `RoyalRequest` | RemoteEvent | client to server | `(action: string up to 24 chars, arg)` | `ServerScriptService.GardenService.Crown.lua:53`, `865-880` |
| `RoyalEvent` | RemoteEvent | server to one client or all | `(kind: string, data: table)` | `ServerScriptService.GardenService.Crown.lua:54`, `97-100` |
| `RoyalQuery` | RemoteFunction | client asks server | `(what)` returns a table | `ServerScriptService.GardenService.Crown.lua:55-60`, `927-968` |
| `RoyalCombat` | RemoteEvent | client to server | `(action, arg)` | `ServerScriptService.GardenService.CrownArena.lua:39-44`, `650-661` |
| `noticeRemote`, `notifyRemote` | from G | server to client | a text line | `ServerScriptService.GardenService.Crown.lua:30-34` |
| `RoyalDev` | BindableFunction in ServerStorage | Studio only | `(cmd, ...)` | `ServerScriptService.GardenService.Crown.lua:1191-1222` |

### RoyalRequest actions

| Action | arg | Handler |
|---|---|---|
| `style` | crown style key | `ServerScriptService.GardenService.Crown.lua:883-898` |
| `passage` | none (King only, throne room and back) | `ServerScriptService.GardenService.Crown.lua:902-923` |
| `challenge` | none | `ServerScriptService.GardenService.CrownArena.lua:996-1018` |
| `respond` | true to accept | `ServerScriptService.GardenService.CrownArena.lua:1020-1023` |
| `cancelChallenge` | none | `ServerScriptService.GardenService.CrownArena.lua:1025-1030` |
| `trial` | none (King only) | `ServerScriptService.GardenService.CrownArena.lua:1033-1048` |
| `joinTrial` | none | `ServerScriptService.GardenService.CrownArena.lua:1050-1060` |
| `watch` | none | `ServerScriptService.GardenService.CrownArena.lua:1144-1149` |
| `joinContest` | none | `ServerScriptService.GardenService.CrownContest.lua:519-538` |
| `leaveContest` | none | `ServerScriptService.GardenService.CrownContest.lua:540-547` |
| `decree` | decree key | `ServerScriptService.GardenService.CrownDecree.lua:317-343` |

Prompt-only handlers (names start with `prompt:`, refused over the remote at `ServerScriptService.GardenService.Crown.lua:874`): `ReturnGate`, `ChamberDoor`, `ChamberExit`, `StyleMirror`, `DecreeDesk`, `Altar` (`ServerScriptService.GardenService.Crown.lua:989-1017`), `Spar` (`ServerScriptService.GardenService.CrownArena.lua:1121`), `RoyalGate` (`ServerScriptService.GardenService.CrownArena.lua:1153`). They are wired from each marker's `RoyalPrompt` and its `Kind` attribute (`ServerScriptService.GardenService.Crown.lua:972-987`).

### RoyalCombat actions

| Action | arg | Code |
|---|---|---|
| `strike` | none | `ServerScriptService.GardenService.CrownArena.lua:570-581` |
| `charge` | none (starts the heavy hold) | `ServerScriptService.GardenService.CrownArena.lua:654` |
| `heavy` | none | `ServerScriptService.GardenService.CrownArena.lua:583-597` |
| `dash` | aim Vector3 (used for knights only) | `ServerScriptService.GardenService.CrownArena.lua:599-610` |
| `block` | true or false | `ServerScriptService.GardenService.CrownArena.lua:612-630` |
| `slam` | none | `ServerScriptService.GardenService.CrownArena.lua:632-646` |
| `shockwave`, `guard`, `summon` | none (King, contest only) | `ServerScriptService.GardenService.CrownContest.lua:379-432` |

### RoyalEvent kinds

| Kind | Sent to | Where |
|---|---|---|
| `achievement`, `throne`, `openMenu` | one player | `ServerScriptService.GardenService.Crown.lua:206`, `1029`, `1005`, `1012` |
| `crowned`, `vacant` | all | `ServerScriptService.GardenService.Crown.lua:760`, `779` |
| `announce` `{ text, sub, style, short }` | all | for example `ServerScriptService.GardenService.Crown.lua:1166`, `ServerScriptService.GardenService.CrownArena.lua:1080` |
| `knockback` `{ vel }` | the victim | `ServerScriptService.GardenService.CrownArena.lua:514` |
| `hit`, `swing`, `dash`, `slamStart`, `slam` | all | `ServerScriptService.GardenService.CrownArena.lua:543`, `557`, `574`, `590`, `603`, `637`, `641` |
| `duelIntro`, `countdown`, `fight`, `point`, `duelEnd` | all | `ServerScriptService.GardenService.CrownArena.lua:755`, `767`, `777`, `791`, `834` |
| `challenge`, `declined` | all | `ServerScriptService.GardenService.CrownArena.lua:976`, `950` |
| `prompt` `{ challengerId, challenger, power, expires, canDecline }` | the King | `ServerScriptService.GardenService.CrownArena.lua:977-980` |
| `contestAnnounce`, `crownAppears`, `contestEnd`, `fall`, `shockwave`, `crownGuard` | all | `ServerScriptService.GardenService.CrownContest.lua:510`, `495`, `135`, `332`, `386`, `396` |
| `decree`, `decreeEnd`, `huntSpot` | all | `ServerScriptService.GardenService.CrownDecree.lua:286`, `73`, `254` |

### RoyalQuery answers

| `what` | Returns | Where |
|---|---|---|
| `history` | last reigns, top reigns, the Eternal King | `ServerScriptService.GardenService.Crown.lua:928-934` |
| `me` | own rep, title, styles, achievements and stats | `ServerScriptService.GardenService.Crown.lua:935-947` |
| `king` | the King's name, reign, stats, court and power | `ServerScriptService.GardenService.Crown.lua:948-954` |
| `power` | top 8 players by Empire Power | `ServerScriptService.GardenService.Crown.lua:955-963` |
| `decrees` | decree list, cooldown, active key, isKing | `ServerScriptService.GardenService.CrownDecree.lua:345-357` |

### RoyalState attributes (on ReplicatedStorage.RoyalState)

| Attributes | Written by |
|---|---|
| `KingUserId`, `KingName`, `KingTitle`, `KingStyle`, `ReignStart`, `ProtectedUntil`, `VulnerableUntil`, `Defenses`, `ReignHow` | `ServerScriptService.GardenService.Crown.lua:696-710` |
| `EternalName`, `EternalSeconds`, `EternalUserId` | `ServerScriptService.GardenService.Crown.lua:630-632` |
| `Court` | `ServerScriptService.GardenService.Crown.lua:1169` |
| `DuelKind`, `DuelState`, `DuelAId`, `DuelAName`, `DuelBId`, `DuelBName`, `DuelAScore`, `DuelBScore`, `DuelTarget`, `DuelSudden`, `DuelStarted` | `ServerScriptService.GardenService.CrownArena.lua:678-697` |
| `DuelUnderdog`, `DuelCount` | `ServerScriptService.GardenService.CrownArena.lua:674`, `766`, `771` |
| `GateOpen`, `Phase` | `ServerScriptService.GardenService.CrownArena.lua:735`, `754`, `861` |
| `ChallengeQueue`, `PendingChallenger` | `ServerScriptService.GardenService.CrownArena.lua:879-882` |
| `TrialOpen`, `TrialEnds` | `ServerScriptService.GardenService.CrownArena.lua:1037-1038` |
| `NextContestAt`, `ContestState`, `ContestEndsAt`, `ContestControl`, `ContestZone`, `ContestClaim`, `ContestKing`, `ContestLeader`, `ContestLeaderPower`, `ContestEntrants` | `ServerScriptService.GardenService.CrownContest.lua:50`, `66-89` |
| `DecreeKey`, `DecreeTitle`, `DecreeEndsAt`, `DecreeProgress`, `DecreeGoal`, `DecreeBy`, `DecreeText`, `NextDecreeAt` | `ServerScriptService.GardenService.CrownDecree.lua:41-56` |

### Player attributes

| Attribute | Read or write | Where |
|---|---|---|
| `RoyalRep`, `RoyalTitle`, `RoyalCrowns`, `RoyalDefenses`, `RoyalDuelWins`, `RoyalContestWins`, `RoyalLongest`, `RoyalTotal`, `RoyalStreak`, `CrownStyle`, `RoyalUnlocked`, `RoyalAchievements` | write | `ServerScriptService.GardenService.Crown.lua:144-167` |
| `RoyalBusy` | write | `ServerScriptService.GardenService.Crown.lua:345` |
| `IsKing` | write | `ServerScriptService.GardenService.Crown.lua:712` |
| `JoinedAt` | write | `ServerScriptService.GardenService.Crown.lua:1047` |
| `RoyalEligible`, `RoyalWhy` | write | `ServerScriptService.GardenService.Crown.lua:1172-1173` |
| `InArena`, `Stagger`, `Blocking`, `CrownGuard`, `Underdog` | write | `ServerScriptService.GardenService.CrownArena.lua:132`, `269-271`, `288-292`, `618`, `672` |
| `CrownPower` | write | `ServerScriptService.GardenService.CrownContest.lua:261`, `279`, `476`; cleared at `ServerScriptService.GardenService.CrownArena.lua:1391` |
| `AetherEndsAt`, `AetherSpan` | write (Royal Race pass) | `ServerScriptService.GardenService.CrownDecree.lua:125-126`, `138` |
| `ProfileLoaded`, `Rebirths`, `CashPerSecond`, `Shards` | read | `ServerScriptService.GardenService.Crown.lua:315-316`, `297`, `245` |

### Other attributes, tags and markers

| Thing | Where |
|---|---|
| `workspace` attributes `KingUserId`, `KingName` (write) | `ServerScriptService.GardenService.Crown.lua:694-695` |
| Character attributes `FallPeak`, `OnSkyGround` (write on teleport) | `ServerScriptService.GardenService.Crown.lua:371-376` |
| Knight model attributes `RoyalKnight`, `Stagger` (write) | `ServerScriptService.GardenService.CrownArena.lua:417`, `134` |
| `ArenaCenter` marker attributes `RingTop`, `RingRadius`, `BoundRadius`, `ZoneRadius`, `ZoneTop` (read) | `ServerScriptService.GardenService.CrownArena.lua:56-65` |
| Part attribute `CrownRole` = band, gem or cap (read) | `ServerScriptService.GardenService.Crown.lua:402`, `541` |
| Crown template attribute `Width` (read) | `ServerScriptService.GardenService.Crown.lua:444` |
| Light attribute `BaseBrightness` (write once) | `ServerScriptService.GardenService.Crown.lua:523-530` |
| Tags `RoyalBanner`, `RoyalCushion`, `RoyalEmblem`, `RoyalGlow`, `RoyalLamp`, `CrownVault` (attr `Style`) | `ServerScriptService.GardenService.Crown.lua:532-573` |
| Tag `RoyalBoard` (attr `Board`), tag `RoyalPortrait` (attr `Portrait`) | `ServerScriptService.GardenService.Crown.lua:576-598`, `614-627` |
| Seat attribute `RoyalSeat` (the throne) | `ServerScriptService.GardenService.Crown.lua:1020-1041` |
| Markers `ThroneFront`, `ChamberInside`, `ChamberDoorN`, `ChamberDoorS`, `ChamberExitN` | `ServerScriptService.GardenService.Crown.lua:916`, `998-1002` |
| Markers `ArenaCenter`, `DuelPadA`, `DuelPadB`, `StandSpot1..12`, `ArenaGate`, `CourtyardArrival` | `ServerScriptService.GardenService.CrownArena.lua:56`, `701-702`, `74-82`, `850-855` |
| Markers `ContestSpawn1..8`, `KnightPost1..3` | `ServerScriptService.GardenService.CrownContest.lua:55`, `413` |
| Workspace model `Royal Gate` with children `Portal`, `Bars`, `RoyalPrompt` | `ServerScriptService.GardenService.CrownArena.lua:708-731` |
| Asset `ReplicatedStorage.KingAssets.RoyalCrown` | `ServerScriptService.GardenService.Crown.lua:434-435`, `ServerScriptService.GardenService.CrownDecree.lua:195-196` |

### Main G functions this subsystem exports

| G key | Where |
|---|---|
| `royalCapture`, `royalApply` | `ServerScriptService.GardenService.Crown.lua:252`, `270` |
| `crownRefresh`, `crownHolder`, `royalCrown`, `royalVacate`, `royalDefended`, `royalDeclined` | `ServerScriptService.GardenService.Crown.lua:818`, `784`, `772`, `782`, `821`, `832` |
| `royalWorthy`, `royalMayChallenge`, `empirePower`, `royalBusy`, `royalSetBusy` | `ServerScriptService.GardenService.Crown.lua:324`, `340`, `299`, `342-346` |
| `royalAddRep`, `royalAddStat`, `royalGiveShards`, `royalTeleport`, `royalHandlers` | `ServerScriptService.GardenService.Crown.lua:351-353`, `379`, `862` |
| `arenaStats`, `royalMode`, `royalFighterOf`, `royalApplyHit`, `royalActions`, `royalStartBout` | `ServerScriptService.GardenService.CrownArena.lua:146`, `90`, `100`, `559`, `648`, `781` |
| `royalContestState`, `royalContestFrozen`, `royalContestCanHit`, `royalKingActions`, `royalStartContest` | `ServerScriptService.GardenService.CrownContest.lua:92`, `94`, `99`, `379`, `517` |
| `spawnBoost`, `crownOnMerge`, `royalRacer`, `royalQueries` | `ServerScriptService.GardenService.CrownDecree.lua:39`, `312`, `307`, `345` |

### Config keys read from GardenConfig (not CrownConfig)

| Key | Where |
|---|---|
| `Config.SaveTouch` | `ServerScriptService.GardenService.Crown.lua:217`, `246`, `891` |
| `Config.IslandName`, `Config.PlotsFolder` | `ServerScriptService.GardenService.CrownDecree.lua:166`, `177` |

## Numbers

All `C.` keys are in `ReplicatedStorage.CrownConfig.lua`.

### Eligibility and the reign

| Key | Value | Line |
|---|---|---|
| `C.Enabled` | true | 31 |
| `C.MinDiscovered` (Studio: `StudioMinDiscovered`) | 10 fruits (Studio 0) | 38-39 |
| `C.MinServerTime` (Studio: `StudioMinServerTime`) | 180 s (Studio 5 s) | 40-41 |
| `C.ClaimDelay` | 6 s | 42 |
| `C.ProtectionTime` | 60 s | 51 |
| `C.DefenseProtection` | 120 s | 52 |
| `C.ChallengeCooldown` | 600 s | 53 |
| `C.DeclinedCooldown` | 90 s | 54 |
| `C.ResponseTime` | 15 s | 55 |
| `C.QueueMax` | 4 | 56 |
| `C.ReignTick` | 60 s | 57 |
| `C.DeclinesToVulnerable` | 2 | 63 |
| `C.VulnerableTime` | 300 s | 64 |
| `C.DeclineMemory` | 600 s | 65 |
| `C.PassageCooldown` | 20 s | 356 |

### The duel

| Key | Value | Line |
|---|---|---|
| `C.DuelTarget` | 3 points | 69 |
| `C.FinalTarget` | 2 points | 70 |
| `C.TrialTarget` (used for spar) | 3 points | 71 |
| `C.DuelIntroTime` | 3.5 s | 72 |
| `C.DuelCountdown` | 3 s | 73 |
| `C.DuelMaxTime` | 180 s | 74 |
| `C.PointPause` | 1.6 s | 75 |
| `C.RespawnShield` | 1.5 s | 76 |
| `C.RingOutDepth` | 12 studs | 77 |
| `C.OutOfBoundsTime` | 1.5 s | 78 |
| `C.ResultTime` | 6 s | 79 |

### Combat (`C.Combat`, lines 92-145)

| Move | range | arc | knockback | lift | cooldown | stagger | other |
|---|---|---|---|---|---|---|---|
| WalkSpeed / JumpPower | | | | | | | 19 / 54 |
| MaxStagger | | | | | | 160 | |
| Strike | 7.5 | 70 | 40 | 16 | 0.42 | 9 | |
| Heavy | 8.5 | 60 | 74 | 26 | 1.25 | 18 | charge 0.45, windup 0.12 |
| Dash | | | | | 2.2 | | speed 70, time 0.18, knockbackScale 0.6 |
| Block | | | | | 0.9 | | maxTime 1.6, knockbackScale 0.3, staggerScale 0.25, walkScale 0.55, heavyBreak 0.65 |
| Slam | radius 11 | all around (`ServerScriptService.GardenService.CrownArena.lua:642`) | 92 | 32 | 10 | 22 | windup 0.28 |
| Underdog | | | | | | | deficit 2, dealt 1.15, taken 0.9, speed 1.06, cooldowns 0.8 |

King abilities (`C.KingAbilities`, lines 148-152): Shockwave radius 15, knockback 88, lift 30, cooldown 8, stagger 15. Guard duration 2.5, cooldown 16. Summon count 3, duration 25, uses 1.

Knights (`C.Knight`, lines 155-164): WalkSpeed 15, StrikeRange 7, StrikeCooldown 1.1, Knockback 34, Lift 14, Stagger 6, Reaction 0.35, SparringSpeed 13.

### The contest (`C.Contest`, lines 172-200)

| Key | Value |
|---|---|
| `IntervalMin` / `IntervalMax` | 18 min / 24 min |
| `FirstDelay` | 10 min |
| `MinPlayers` | 3 |
| `MinEntrants` | 1 |
| `AnnounceTime` | 60 s |
| `BrawlTime` | 12 s |
| `MaxTime` | 150 s |
| `ZoneRadius` / `ZoneHeight` | 9 / 10 studs |
| `Tick` | 0.25 s |
| `DrainRate` / `RegenRate` | 7 / 5 control per second |
| `Diminishing` | 1.0, 1.3, 1.45, 1.55 for 1, 2, 3, 4+ challengers |
| `GuardAt` | 50 control |
| `C.ClaimPower` / `C.ClaimRate` | 100 / 10 per second |

### Rewards and reputation

| Key | Value | Line |
|---|---|---|
| `C.Rewards.Defense` | 500 shards | 262 |
| `C.Rewards.Victory` | 1000 shards | 263 |
| `C.Rewards.ContestWin` | 3000 shards | 264 |
| `C.Rewards.ContestSurvive` | 1500 shards | 265 |
| `C.Rewards.ContestEntry` | 100 shards | 266 |
| `C.Rewards.FirstSpar` | 150 shards | 267 |
| `C.Rep.PerReignTick` / `Claim` / `Defense` / `Victory` / `ContestWin` / `ContestSurvive` / `Decree` | 4 / 5 / 25 / 40 / 60 / 30 / 15 | 272-280 |
| `C.Titles` | KING 0, HIGH KING 250, EMPEROR 1000, SOVEREIGN 3000 rep | 283-288 |
| `C.CrownStyles` unlocks | Golden 0, Emerald 100, Ruby 300, Celestial 1000 rep; Shadow by Conqueror; Soul by Legend | 297-316 |
| `C.Achievements` | FirstCrown 1 crown (+10), CrownKeeper 5 defenses (+50), Tyrant 15 min (+60), Emperor 30 min (+120), Dynasty 10 crowns (+100), Conqueror 10 duel wins (+100), Undisputed 5-defense streak (+120), Legend 60 min (+300) | 328-337 |
| `C.HistoryLength` | 8 reigns shown | 344 |

### Decrees (`C.Decrees`, lines 209-254)

| Key | Time | Goal | Reward |
|---|---|---|---|
| `C.DecreeCooldown` / `C.DecreeMinPlayers` | 360 s / 1 player | | |
| MergeRush | 180 s | 25 merges | 400 shards each merger |
| FruitFrenzy | 60 s | spawnBoost 1.1 | 0 |
| RoyalRace | 240 s | 3 finishers, start below Y 120 | 900 / 500 / 250 |
| CrownHunt | 150 s | 1 find | 750 |
| EmpireBuild | 240 s | income x 1.10 | 350 each player |

### Hard-coded numbers in the server code

| Value | Meaning | Where |
|---|---|---|
| 0.15 s | minimum gap between any two RoyalRequest calls per player | `ServerScriptService.GardenService.Crown.lua:870` |
| 300 studs | `SKY_Y`: a teleport above this sets `OnSkyGround` | `ServerScriptService.GardenService.Crown.lua:364`, `372` |
| 260 studs | Royal Passage sends you home if you are this close to the throne | `ServerScriptService.GardenService.Crown.lua:918` |
| 1.25 s | crown attach delay after crowning | `ServerScriptService.GardenService.Crown.lua:766` |
| 20 | reigns kept in this server's history | `ServerScriptService.GardenService.Crown.lua:731` |
| 60 s | shortest reign written to the DataStores | `ServerScriptService.GardenService.Crown.lua:664` |
| 10% per friend | text on the Ruler board for the Royal Court | `ServerScriptService.GardenService.Crown.lua:1098` |
| 20 s | Royal Trial answer window | `ServerScriptService.GardenService.CrownArena.lua:1036-1041` |
| 12 s | longest wait for a dead fighter's new body | `ServerScriptService.GardenService.CrownArena.lua:807` |
| 0.12 s | life of the server knockback LinearVelocity | `ServerScriptService.GardenService.CrownArena.lua:530` |
| 0.08 s | strike hit delay after the swing | `ServerScriptService.GardenService.CrownArena.lua:575` |
| 7 studs | max height difference for a hit | `ServerScriptService.GardenService.CrownArena.lua:495` |
| 0.3 of ClaimPower | Claim the Throne timeout winner threshold | `ServerScriptService.GardenService.CrownContest.lua:298` |
| 1.5 s | player return after a contest ring-out | `ServerScriptService.GardenService.CrownContest.lua:333` |
| 60 to 320 studs, Y -5 to 60 | Crown Hunt search ring and height window | `ServerScriptService.GardenService.CrownDecree.lua:173-176` |
| X -18..40, Z -56..56, Y 438..470 | Royal Race finish box | `ServerScriptService.GardenService.CrownDecree.lua:148` |
| 8 s | Aether grace after the race | `ServerScriptService.GardenService.CrownDecree.lua:137` |

## Extension points for the expansion

### 1. Rebirth-gated rarity

- **Where**: `worthy(player)` (`ServerScriptService.GardenService.Crown.lua:314-323`) and `C.MinDiscovered` (`ReplicatedStorage.CrownConfig.lua:38`).
- **What breaks**: at 0 rebirths the best fruit would be Apple. Only 5 fruits are COMMON and at or below Apple: Apple, Tomato, Carrot, Strawberry, Blueberry (`ReplicatedStorage.Fruits.FruitIndex.lua:69-73`). The rule needs 10 (`ReplicatedStorage.CrownConfig.lua:38`). So no 0-rebirth player could ever hold or challenge for the crown. Only a rebirth makes them worthy (`ServerScriptService.GardenService.Crown.lua:316`).
- **What to change**: lower `C.MinDiscovered` to 5 or less, or make the need depend on the number of unlocked fruits. Keep the "1+ rebirths is worthy" line.
- **Balance risk**: if rebirth costs shards, the royal shard rewards (500 to 3000 per fight, `ReplicatedStorage.CrownConfig.lua:261-268`) become a rebirth input. The config says fighting should never beat the merge economy (`ReplicatedStorage.CrownConfig.lua:256-259`). Recheck those values.
- The King keeps the crown through his own rebirth. The crown only moves by duel, contest or leaving (`ServerScriptService.GardenService.Crown.lua:4-9`).

### 2. Bats that upgrade each rebirth

- **Where**: the arena fairness rule (`ReplicatedStorage.CrownConfig.lua:16-19`) and the `InArena` player attribute (`ServerScriptService.GardenService.CrownArena.lua:269`, `288`).
- **What to change**: bat hit code (outside this subsystem) must skip any attacker or victim with `InArena` set. Otherwise a player in the stands could knock a fighter off the ring, which scores a point (`ServerScriptService.GardenService.CrownArena.lua:1237`, `1263`).
- Inside the arena the sword is re-equipped whenever it is unequipped (`ServerScriptService.GardenService.CrownArena.lua:211-218`). `enterArena` does not remove other tools from the backpack (`ServerScriptService.GardenService.CrownArena.lua:264-280`). A bat equip would be undone, but only after a short `task.defer`.

### 3. Explosives and artillery

- **Where**: same as bats: skip `InArena` players and knight models (`RoyalKnight` attribute, `ServerScriptService.GardenService.CrownArena.lua:417`).
- **Risk**: knights are held together by Motor6D joints (`ServerScriptService.GardenService.CrownArena.lua:375-380`) and the crown by Welds (`ServerScriptService.GardenService.Crown.lua:477-481`). If explosives use Roblox `Explosion` objects, check that they do not break those joints.
- The "King never griefs" rule (`ReplicatedStorage.CrownConfig.lua:20-21`) means no royal power should fire explosives at plots.

### 4. Underground base and the Ores index

- **Where**: `discovered(player)` counts every child of the Index folder (`ServerScriptService.GardenService.Crown.lua:289-292`).
- **What breaks**: if ore entries go into the same folder, they count toward `C.MinDiscovered` in this session. Persistence only saves names that are in FruitIndex (`ServerScriptService.GardenService.Persistence.lua:69-71`), so after a rejoin the count would drop and eligibility could flip. Use a separate folder, or filter to fruit names in `discovered`.
- **Income**: if ore income feeds `CashPerSecond`, it raises Empire Power (`ServerScriptService.GardenService.Crown.lua:296-298`) and Empire Build progress (`ServerScriptService.GardenService.CrownDecree.lua:258-262`). That is probably wanted.

### 5. Forest pets

- **Income**: pet income in `CashPerSecond` raises Empire Power (`ServerScriptService.GardenService.Crown.lua:296-298`), which decides who claims an empty throne.
- **Pattern to reuse**: knights are server-owned NPCs with a hand-built R6 rig, `SetNetworkOwner(nil)`, R6 animations and `Humanoid:MoveTo` thinking (`ServerScriptService.GardenService.CrownArena.lua:336-460`, `1179-1224`). Wandering pets could copy that.
- **Risk**: the "nobody else on the ring" sweep only moves Players (`ServerScriptService.GardenService.CrownArena.lua:1291-1303`). A pet that follows its owner into the arena would not be removed.

### 6. Boats, swimming and Aqua Affinity

- `teleport()` sets `Humanoid.Sit = false` before moving a player (`ServerScriptService.GardenService.Crown.lua:370`). A player sitting in a boat is lifted out when they go to a duel or the stands.
- The throne hook only acts on seats with the `RoyalSeat` attribute (`ServerScriptService.GardenService.Crown.lua:1023`), so boat seats are not affected.
- **Pattern to reuse**: the Royal Race grants a timed effect by writing `effects[p].Aether` plus `AetherEndsAt` and `AetherSpan`, and only removes its own grant (`ServerScriptService.GardenService.CrownDecree.lua:121-141`). An Aqua Affinity grant from code could follow the same shape.

### 7. More potions

- **Where**: `G.arenaStats` (`ServerScriptService.GardenService.CrownArena.lua:146-154`) wins over every potion in `applyWalkSpeed` (`ServerScriptService.GardenService.lua:197-208`). Invisible and Feather Falling are switched off while `InArena` (`ServerScriptService.GardenService.lua:266`, `ServerScriptService.GardenService.Ambient.lua:109`, `ServerScriptService.GardenService.Ambient.lua:281`).
- **What to change**: any new potion that changes movement, size, gravity, knockback or visibility must add the same `InArena` check. Otherwise it breaks "same numbers for everyone" (`ReplicatedStorage.CrownConfig.lua:16-19`).
- **Risk**: `G.spawnBoost` is one global number. Decree `finish` always sets it back to 1 (`ServerScriptService.GardenService.CrownDecree.lua:62`). A spawn-speed potion must not write `G.spawnBoost`.

### 8. Plot visuals, logs, lily pads, border mountains

- **Crown Hunt**: it hides the crown by raycasting down onto the model named `Config.IslandName`, 60 to 320 studs from the world origin, and accepts only hits between Y -5 and 60 that are not inside `Config.PlotsFolder` (`ServerScriptService.GardenService.CrownDecree.lua:165-183`). New mountains or decor taller than Y 60 inside that ring make hits fail. After 40 failures the decree fails (`ServerScriptService.GardenService.CrownDecree.lua:184`). New decor outside the island model is never picked.
- **Royal Gate**: it must stay a workspace model named `Royal Gate` with `Portal`, `Bars` and a `RoyalPrompt` (`ServerScriptService.GardenService.CrownArena.lua:708-731`). It sits on the road between Plot0 and Plot11 (`ReplicatedStorage.CrownConfig.lua:351-353`). A plot rebuild must keep it.

### 9. UI overhaul (FredokaOne, image titles, no emoji)

- **Server text has emoji**: most `C.Text` lines (`ReplicatedStorage.CrownConfig.lua:417-477`), decree `icon` fields (`ReplicatedStorage.CrownConfig.lua:213`, `222`, `231`, `240`, `248`), and strings built in server code (`ServerScriptService.GardenService.Crown.lua:207`, `211`, `228`, `232`, `1090`, `1094`, `1096`, `1098`, `1101`, `1117`; `ServerScriptService.GardenService.CrownArena.lua:1039`; `ServerScriptService.GardenService.CrownDecree.lua:93`, `130`, `153`, `249`). A client-only reskin will still show these.
- **Icons**: every `C.Icons` entry is 0, meaning "no image yet, keep the emoji" (`ReplicatedStorage.CrownConfig.lua:385-397`). Fill these with image ids.
- **In-world boards**: the server only sets `Text` on labels found by tag (`ServerScriptService.GardenService.Crown.lua:576-598`). The font lives on the board instances, which are built elsewhere.
- **Contract to keep**: `announce` events carry a `style` of `royal`, `alarm`, `crown`, `plain` or `decree` (for example `ServerScriptService.GardenService.CrownDecree.lua:285`). The `decrees` query sends `icon` (`ServerScriptService.GardenService.CrownDecree.lua:349`). Change both sides together.

### 10. Royal combat (animations, slashes, health, Slam, weapon skins)

- **Animations and slashes**: `animate(f, which)` drops a `toolanim` StringValue for the default Animate script (`ServerScriptService.GardenService.CrownArena.lua:229-243`). Every swing also broadcasts `swing { id, heavy }` (`ServerScriptService.GardenService.CrownArena.lua:574`, `590`). Real animations and slash effects can hang off that event on the client. Add a field (for example a combo step) if needed. **Risk**: the hit lands 0.08 s after a strike and `CB.Heavy.windup` after a heavy (`ServerScriptService.GardenService.CrownArena.lua:575`, `591`). Longer animations will look late compared to the hit.
- **Health drain**: `applyHit` is the single point every hit passes through, including knights and Shockwave (`ServerScriptService.GardenService.CrownArena.lua:539-558`). Add an `hp` field to `newFighter` (`ServerScriptService.GardenService.CrownArena.lua:116-126`), drain it in `applyHit`, publish it like `Stagger` (`ServerScriptService.GardenService.CrownArena.lua:129-136`), and score a point at 0 through `ringOutInBout(f, "ko")` (`ServerScriptService.GardenService.CrownArena.lua:783`). Reset it where stagger resets (`ServerScriptService.GardenService.CrownArena.lua:789`). **Do not** lower `Humanoid.Health` to 0. A death is the `died` path, which waits for a new body for up to 12 s (`ServerScriptService.GardenService.CrownArena.lua:1235`, `1257`, `805-808`). Contests use their own ring-out handler (`ServerScriptService.GardenService.CrownContest.lua:319-346`).
- **Better Royal Slam**: the server side is `slam(f)` (`ServerScriptService.GardenService.CrownArena.lua:632-646`). It only sends `slamStart` and `slam` events with position and radius. The look is all client side. Change the server only if the move itself changes (for example a leap).
- **Weapon skins chosen before a fight**: `makeSword()` builds one fixed sword and `giveSword(f)` is the only place it is handed out (`ServerScriptService.GardenService.CrownArena.lua:172-220`). Read a skin key there. The bout intro lasts `C.DuelIntroTime` (`ServerScriptService.GardenService.CrownArena.lua:760`), which is the natural pick window. A new `RoyalRequest` handler goes in the `handlers` table (`ServerScriptService.GardenService.Crown.lua:861-862`). To save the choice, follow `royal.style`: add it to `royalCapture`/`royalApply` (`ServerScriptService.GardenService.Crown.lua:252-285`) and to GardenStore's sanitise (`ServerScriptService.GardenStore.lua:409-436`). **Risks**: keep the tool name `RoyalSword`, since three functions find it by name (`ServerScriptService.GardenService.CrownArena.lua:201`, `224`, `231`). Hit reach comes from `CB.Strike.range`, not the blade, so a long blade will look like it reaches further than it hits. Skins must stay cosmetic (`ReplicatedStorage.CrownConfig.lua:16-19`).

## Gotchas

- **Load order matters.** Crown, CrownArena, CrownContest and CrownDecree load in that order (`ServerScriptService.GardenService.lua:515-518`). Later modules add G functions that earlier ones look up at call time, such as `G.royalMode` and `G.royalQueries` (`ServerScriptService.GardenService.Crown.lua:805`, `964`). CrownArena grabs `G.applyVisibility` and `G.applyWalkSpeed` once at load (`ServerScriptService.GardenService.CrownArena.lua:29-32`).
- **Two clocks.** Cooldowns, declines, join time and the claim timer use `os.clock()`. Reign start, protection and vulnerability use `workspace:GetServerTimeNow()` (`ServerScriptService.GardenService.Crown.lua:75`, `334-336`, `744-746`, `834-839`). Do not compare one with the other.
- **The "refuse while vulnerable starts a contest" rule looks unreachable.** A challenge made while the King is vulnerable has `canDecline = false` (`ServerScriptService.GardenService.CrownArena.lua:973-974`). `resolve` turns any answer to such a challenge into a duel (`ServerScriptService.GardenService.CrownArena.lua:932-939`). Vulnerability only starts inside `royalDeclined`, and only one challenge is pending at a time (`ServerScriptService.GardenService.CrownArena.lua:957`). So the `contest` branch (`ServerScriptService.GardenService.Crown.lua:842-847`) seems to never fire in normal play. A vulnerable King is simply forced to fight.
- **`C.Enabled` only stops the wealth claim.** The server checks it only in `crownRefresh` (`ServerScriptService.GardenService.Crown.lua:799`). The contest calendar does not check it (`ServerScriptService.GardenService.CrownContest.lua:551-564`), and Claim the Throne can still crown someone (`ServerScriptService.GardenService.CrownContest.lua:130`).
- **`C.GateOpenForEvents` is not read by the server.** `setGate` only ORs in `C.GateAlwaysOpen` (`ServerScriptService.GardenService.CrownArena.lua:733-737`).
- **The saved number list exists twice.** `NUMBERS` in Crown (`ServerScriptService.GardenService.Crown.lua:107-108`) and `ROYAL_NUMBERS` in GardenStore (`ServerScriptService.GardenStore.lua:306-307`). A new saved royal number must go in both, or GardenStore's sanitise drops it.
- **The leave-save and the end of the reign race.** Both are PlayerRemoving handlers. `royalCapture` folds in the live reign so either order is right (`ServerScriptService.GardenService.Crown.lua:256-264`). The record is kept 30 s after leaving (`ServerScriptService.GardenService.Crown.lua:1075-1078`). Do not shorten that.
- **`finishBout` clears the arena before anyone leaves it.** `leaveArena` re-applies walk speed through `G.arenaStats`, which would still freeze a listed fighter (`ServerScriptService.GardenService.CrownArena.lua:841-848`). Keep that order.
- **Do not swap the crown Welds for WeldConstraints.** A WeldConstraint measures its offset after `AddAccessory` moves the handle, which left the crown 180 studs away in a play test (`ServerScriptService.GardenService.Crown.lua:465-484`).
- **Falls are caught by anchoring.** The arena sits above a 400-stud drop and the garden fall sweep kills long falls. `catch` anchors the root until the fighter is placed again (`ServerScriptService.GardenService.CrownArena.lua:247-262`). Any new movement code must call `release` or `place` first.
- **One rate limit for all RoyalRequest actions.** Two requests from one player within 0.15 s drop the second silently, whatever the action (`ServerScriptService.GardenService.Crown.lua:868-871`).
- **Rep added in two places skips the notices.** `crownPlayer` adds `C.Rep.Claim` and `checkAchievements` adds achievement rep directly, not through `addRep` (`ServerScriptService.GardenService.Crown.lua:755`, `205`). Style unlock and new title notices do not fire for that rep.
- **The Royal Court board line is not a King perk.** The Ruler board shows "+10% income" per friend (`ServerScriptService.GardenService.Crown.lua:1098`). The real bonus is `Config.FriendBonus` and applies to every player (`ServerScriptService.GardenService.Progression.lua:332-335`). If `FriendBonus` changes, the board text goes wrong.
- **Practice bouts skip eligibility.** Anyone not busy can spar when the arena is free (`ServerScriptService.GardenService.CrownArena.lua:1121-1129`).
- **Empire Build pays everyone.** At success every non-King player gets shards, even players who did nothing (`ServerScriptService.GardenService.CrownDecree.lua:272-274`).
- **Hard-coded positions.** The Royal Race finish box (`ServerScriptService.GardenService.CrownDecree.lua:148`), the arena fallback centre (`ServerScriptService.GardenService.CrownArena.lua:57`), `SKY_Y` (`ServerScriptService.GardenService.Crown.lua:364`) and the 260-stud passage range (`ServerScriptService.GardenService.Crown.lua:918`) all assume the castle stays where it is.
- **The contest blocks the arena from announce to the last result screen.** `contestOn()` is true while any contest state exists, including the final bout (`ServerScriptService.GardenService.CrownArena.lua:875-877`). A scheduled contest is pushed back a minute while a bout runs (`ServerScriptService.GardenService.CrownContest.lua:556-560`).
- **Studio uses different rules.** Studio needs 0 fruits and 5 s in the server (`ReplicatedStorage.CrownConfig.lua:38-41`), writes to the `studio` DataStore scope (`ServerScriptService.GardenService.Crown.lua:602`), and creates the `RoyalDev` BindableFunction (`ServerScriptService.GardenService.Crown.lua:1192-1222`). A test that passes in Studio may still fail the live 10-fruit and 180-second rules.
