---
tags: [research]
topic: Roblox platform limits that shape the Fruit Empire expansion (saving, streaming, performance, NPCs, assets, prompts, random-reward and age policy)
date: 2026-10-04
---

# Roblox Platform Constraints for the Expansion

Terms used in this note:
- **DataStore**: Roblox's built-in save system. Each saved record is a **key** (for example one key per player) holding a **value** (the saved table).
- **Request budget**: how many save or load calls Roblox lets you make per minute before it starts queuing or failing them.
- **CCU**: concurrent users, meaning players online right now across all servers.
- **StreamingEnabled / streaming**: the engine only sends a player the parts of the map near them, and unloads far parts.
- **Replication**: the server sending changes (positions, properties) to every player's client.
- **Assembly**: a group of parts welded together that physics moves as one object.
- **Network ownership**: which computer (the server or one player's client) runs the physics for an unanchored part.
- **Anchored**: a part that physics never moves. Only scripts move it.
- **Draw call**: one batch of drawing work the device sends to its graphics chip. Fewer is faster.
- **Paid random item**: anything bought with Robux (directly, or with a currency bought with Robux) that gives a random result. Loot boxes, egg hatches and luck boosts count.
- **PolicyService**: a Roblox API that tells your game, per player, which rules apply to them (for example whether they may buy random items).

## Summary

- **Saving is not the bottleneck.** One key can hold 4,194,304 characters (about 4 MB). A server gets 60 + 40 × players reads and writes per minute. Total storage is 500 MB + 1 MB per lifetime player [1].
- **Moving things over the network is the real bottleneck.** A widely cited budget (linked from Roblox's own docs) is under 50 KB/s download per client. Each moving physics object costs about 0.4 to 0.9 KB/s, so only about 40 to 60 can move by physics at once [10][7].
- **Pets and ores should be server data, drawn by the client.** Roblox's docs say to avoid Humanoids for NPCs (non-player characters, like wandering pets), to play NPC animations on the client, and to pool models instead of respawning them [6].
- **Skip PathfindingService for base wander.** Paths cap at 3,000 studs and 20,000 nodes [15]. Creators report it slowing down badly under many calls [16][17][18]. Simple random-point wander inside a plot needs none of it.
- **Use Atomic streaming for each pet, ore and fruit model. Avoid Persistent.** Roblox says Persistent models are for "very rare circumstances" and overuse hurts performance [4][5].
- **Audio and animations are locked to whoever is allowed to use them.** Since July 31, 2025, animations work like audio: the experience must be granted permission, and they can never be made Open Use [31][32][28].
- **Free random rewards are fine. Paid random rewards need odds shown and a PolicyService check.** Earned-only randomness (free ore drops, free pet rolls) is exempt. Anything touched by Robux must show exact odds and hide or replace itself for restricted players [37][38].
- **Reaching kids is now gated.** A game is first shown only to age-checked players 16+. It needs 250 plays from "highly engaged" age-checked players in 60 days (or a 50,000 Robux expedited review) to reach Roblox Kids (5 to 8) and Roblox Select (9 to 15) accounts [45][46]. Fruit Empire had 119 lifetime visits on 2026-10-04 [48].
- **Fruit Empire's own settings** (Roblox Games API, read 2026-10-04) [48]: owned by the user account `hikeALT12212` (not a group), avatars forced to R15 (`MorphToR15`), and **50 max players per server**. The map has only 12 plots [49], so a full server could hold 38 players with no plot.

## How it works

### 1. DataStores (saving)

Size limits per entry [1]:
- Key name: 50 characters. DataStore name: 50 characters. Scope: 50 characters.
- Value: 4,194,304 characters per key. Length is measured on the JSON-encoded value [1].
- Metadata: key name 50 characters, value 250 characters, 300 characters total [1].

Per-server request budgets (requests per minute, standard DataStores) [1]:

| Request | Budget |
|---|---|
| Read (GetAsync, UpdateAsync read) | 60 + players × 40 |
| Write (SetAsync, IncrementAsync, UpdateAsync write) | 60 + players × 40 |
| List | 5 + players × 2 |
| Remove | 60 + players × 40 |

- Example: a 10-player server gets 460 writes per minute.
- `UpdateAsync` uses both one read and one write from the budget [1].
- New servers get a one-time startup burst of extra budget [1].
- Ordered DataStores (used for leaderboards) have a smaller write budget: 30 + players × 5 [1].

Per-experience limits (shared across all servers and Open Cloud) [1]:
- Standard read: 300 + CCU × 40. Standard write: 300 + CCU × 20. List: 300 + CCU × 2 [1].

Per-key throughput: 25 MB per minute read, 4 MB per minute write [1]. The current limits page lists this throughput limit. It does not list a per-key cooldown in seconds.

Throttling: when you go over, requests queue. Each queue holds 30 requests. Past that, requests fail with error codes 301 to 306 [1].

Storage: `500 MB + 1 MB × lifetime user count`, measured on the compressed size of the latest version of each key. Roblox compresses for you and says not to pre-compress [1].

Safety notes:
- A failed write does not always mean the write did not happen. Verify with a fresh read [1].
- Prefer `UpdateAsync` over `SetAsync` when more than one server might write the same key [2].

Memory stores (fast shared memory, for things like a live cross-server duel queue): quota is 64 KB + 1.2 KB × users. A single sorted map or queue tops out at 100 MB and 1 million items. Default expiry is 45 days [3].

### 2. Streaming and ModelStreamingMode

Defaults [4]:
- `StreamingMinRadius` default 64 studs. Roblox recommends keeping it.
- `StreamingTargetRadius` default 1,024 studs.
- `StreamOutBehavior`: `LowMemory` is the default (only unload under memory pressure). Roblox recommends `Opportunistic`, which unloads anything past the target radius.
- StreamingEnabled is on by default for new places and can only be set in Studio, not from a script [4].
- Roblox recommends `ModelStreamingBehavior = Improved` for default (Nonatomic) models [4].

ModelStreamingMode values [5][4]:

| Mode | What it does |
|---|---|
| Default (0) | "Default behavior (subject to change)." |
| Atomic (1) | The model and all its children arrive together when any part is in range. It leaves only when all parts are out of range. |
| Persistent (2) | Sent soon after join, before `Workspace.PersistentLoaded` fires. Never streamed out. |
| PersistentPerPlayer (3) | Persistent for players added with `Model:AddPersistentPlayer()`, Atomic for everyone else. |
| Nonatomic (4) | Children are sent except part children. Parts stream on their own. |

Key rules from the streaming page [4]:
- Persistent models are for "very rare circumstances". They are "not intended to circumvent streaming, and overuse may negatively impact performance."
- When something streams out it is parented to `nil`, not destroyed. Local-only property changes can be lost when it streams back.
- Unanchored assemblies stream in as complete units. Roblox warns: "Avoid creating moving assemblies with unnecessarily large numbers of instances."
- Client physics only runs in streamed-in areas, even for Persistent instances.
- Extra replication focus points cost server work. One player with nine moving foci can cost about as much as ten players.
- Streaming only applies to things inside `Workspace`. ReplicatedStorage is not streamed.

### 3. Performance budgets (parts, moving objects, memory)

Official numbers:
- Frame budget at 60 FPS is 16.67 ms [7].
- Roblox's design page gives an example target of under 1,000 draw calls and 1,000,000 triangles on a chosen baseline device. It is an example, not a platform rule [7].
- Server memory = 6.25 GiB + 100 MiB × peak connected players. Keep usage under 50%. A 30-player server has about 9.18 GiB. Servers do not shrink when players leave [8].
- Investigate if client crash rate goes above 2 to 3% [8].
- Place file limit is 100 MB [11].
- A single mesh cannot exceed 20,000 triangles [12].
- Textures can be up to 4096×4096 [13]. The performance page says most images should be no more than 512×512 and minor ones under 256×256. A 1024×1024 texture uses four times the memory of 512×512 [6].
- Identical meshes with the same texture are drawn in one batch ("instancing"). Decals, textures and particles do not batch well [6].
- Server-side tweens replicate every frame. Creating and destroying big instance trees at runtime is network heavy [6].
- `Workspace:BulkMoveTo()` moves many parts at once without the cost of separate property sets. Roblox says plain CFrame setting is fast enough "in the majority of cases" [9].
- Set `CanCollide`, `CanTouch` and `CanQuery` to false on parts that do not need them. Use `Box` collision fidelity for small anchored meshes [6].

No official hard cap on part count was found. Not confirmed.

Community budget (MrChickenRocket, DevForum, August 2024; linked from Roblox's design page [7]). He calls these "my opinion" for 60 FPS on 90%+ of phones [10]:
- 500,000 triangles and 500 draw calls in view.
- Under 1.3 GB client memory, to support 2 GB phones.
- Under 50 KB/s network download per client. Above about 40 KB/s, physics updates start getting throttled.
- Each moving physics object costs about 0.4 to 0.9 KB/s, so about 40 to 60 moving physics assemblies is a safe budget.
- His demo ran 30 pets per player (360 visible in a 12-player server). Pets existed on the server only as table data. Remote events told clients what each pet was doing. Clients positioned pets with `:PivotTo` every frame, so pet movement used no bandwidth. He calls a full Humanoid rig per pet "quite a heavy thing" [10].
- His coins used `BulkMoveTo` on the client and a spatial grid for pickup checks [10].

### 4. Humanoids, NPCs and animation cost

From Roblox's performance page [6]:
- A Humanoid "comes with a significant computation cost."
- Static NPCs should use an `AnimationController` instead of a Humanoid.
- Moving NPCs can use your own movement code plus an `AnimationController`.
- Playing NPC animations from the server means the server simulates and replicates them. Roblox suggests creating the `Animator` on the client and only animating NPCs near the player.
- Disable unused Humanoid states. Pool NPC models instead of destroying and respawning them.

### 5. PathfindingService vs simple wander

Official limits [15]:
- Straight-line distance from start to finish must not exceed 3,000 studs.
- A computation can run out at 20,000 nodes before that distance.
- Waypoints below Y = -65,536 or above Y = 65,536 are ignored.
- Agent defaults: radius 2 studs, height 5 studs, waypoint spacing 4 studs.
- `ComputeAsync` must be called again each time the path needs updating. Result status is `Success` or `NoPath` [47].
- No official per-call rate limit was found in the docs. Not confirmed.

Creator reports (anecdotal, not official):
- `ComputeAsync` sometimes taking up to 45 seconds instead of under 0.2 seconds, freezing every NPC at once (2024) [16].
- Paths slowing down after about 150 `ComputeAsync` calls (June 2026) [17].
- PathfindingService script activity jumping from 3 to 5% up to 15 to 40% (November 2025 bug report) [18].

### 6. ParticleEmitter cost

From the particle docs [14]:
- One emitter makes at most 400 particles per second (100 per second on mobile).
- Max particle lifetime is 20 seconds. Higher values are capped.
- Flipbooks run at most 30 FPS. They cost memory and turn off automatically on low-memory phones.
- Cost comes from overdraw (many see-through layers stacked) and fill rate (how much of the screen they cover).
- `LightEmission = 1` uses additive blending, which makes particles glow even in dark places.
- Changing ParticleEmitter properties at runtime "can have a dramatic impact on performance" [6].

No official performance numbers were found for PointLight or SurfaceLight counts. Not confirmed.

Light range: the cap for PointLight, SpotLight and SurfaceLight was 60 studs. Roblox announced a doubling to 120 on 23 Sep 2025 [52]. The API dump shows a Studio-only `Lighting.ExtendLightRangeTo120` rollout setting [36]. Whether it is on by default is not confirmed.

### 7. Network ownership and server authority

From the network ownership docs [19][20]:
- The server owns all parts by default. The engine hands unanchored parts near a player to that player's client.
- Anchored parts are always server owned. You cannot change that.
- `SetNetworkOwner(nil)` forces server ownership. Roblox warns it can make physics jittery for players.
- The vehicle example sets the driver as owner when they sit, then `SetNetworkOwnershipAuto()` when they leave [19].
- A client that owns a part can teleport it, fling things with extreme velocity, and fake or skip `Touched` events [20].
- `ApplyImpulse` on a part a client owns automatically (like a player's character) can be called from a server script [21]. In practice a standing Humanoid resists the push, so knockback built only on a server impulse often looks like nothing happened [50]. See "Bats, swords and duels" below for the pattern the game already uses.

Server Authority (full release July 9, 2026) [22][23]:
- The server becomes the only source of truth. Clients predict their own inputs and get corrected.
- It is opt-in. It needs `AuthorityMode = Server`, `StreamingEnabled`, Deferred signals, fixed simulation and next-generation replication [23].
- Backpacks and Tools are supported at full release [22].

Remote events [24][25]:
- About 500 requests per second per client, shared across all RemoteEvents (and separately across all UnreliableRemoteEvents).
- UnreliableRemoteEvent payloads over 1,000 bytes are dropped.

Explosions [26]:
- `Explosion.DestroyJointRadiusPercent` (0 to 1) controls how much of the blast radius breaks joints. `BlastPressure` controls push force. `Explosion.Hit` fires for each part in the radius with its distance.

### 8. Asset upload, moderation and privacy

Moderation [27]:
- Every imported asset is checked by humans and automated systems. It "generally happens within a few hours."
- If an asset is still in the queue when you publish, players cannot see or hear it until it is approved.

Upload limits:

| Asset | Limits |
|---|---|
| Audio (Studio import) | `.mp3`, `.ogg`, `.wav`, `.flac`; under 20 MB and 7 minutes; 2,000 per 30 days if ID-verified, 100 per 30 days if not [29] |
| Audio (Open Cloud API) | Up to 7 minutes; 100 uploads per month if ID-verified, 10 if not [30] |
| Image / Decal (Open Cloud API) | `.png`, `.jpeg`, `.bmp`, `.tga`; smaller than 8000×8000; 20 MB per file [30] |
| Animation (Open Cloud API) | `.rbxm` / `.rbxmx`; files edited outside Studio may fail [30] |
| Mesh | 20,000 triangles max per mesh [12] |

The two audio pages give different limits. They cover different upload paths, but this is not confirmed.

Privacy and ownership [28][31][32]:
- Two privacy types: **Restricted** (owner must grant permission) and **Open Use** (anyone can use it).
- The Asset Privacy setting only changes the default for Images, Decals and Meshes. These are Open Use by default unless you turn Asset Privacy on [28].
- Audio, Video, Animations and others have their own defaults [28]. Imported audio starts private to you. You can grant friends and experiences permission [29].
- Open Use is permanent. A grant to a game is permanent and cannot be revoked [28].
- Animations (July 31, 2025): they are "Restricted" and "cannot be changed to Open Use". Once an experience has permission, every player can load them, even if the experience has a different owner [31].
- Animations (August 21, 2025): they can now be shared with friends ("connections") and groups from the asset's Permissions page [32].
- The animation editor still says to pick the group as Creator if the animation is for a group-owned game [33].

### 9. ProximityPrompts

- Default activation distance is 10 studs (API dump) [36]. Visibility depends on `MaxActivationDistance`, `RequiresLineOfSight` (on by default) and `Exclusivity` [34].
- Default `Exclusivity` is `OnePerButton`. Only one prompt per key is visible at a time. The camera direction picks which one [34][36].
- `ProximityPromptService.MaxPromptsVisible` defaults to 16 (from the client API dump, not stated on the docs page) [35][36].
- `MaxIndicatorsVisible` (small distance hints) defaults to 16 and is clamped to 0 to 64 [35].
- No limit on how many prompts can exist was found. Not confirmed.

### 10. Random rewards, gambling and age rules

Gambling [39][44]:
- Roblox prohibits "both simulated and actual gambling activities." No Robux or items of value may be used in a game of chance [39].
- Depicting gambling you cannot play is allowed but pushes the rating to Moderate or higher [44].

Paid random items [37][38][39]:
- Covers four types: capsules (spin, hatch, open a chest), enhancements (a potion with a random duration, an upgrade that might fail), combinations (fuse two eggs for a better roll), and probability modifiers (luck boosts, pity systems, better drop rates) [37].
- Bundling a random item into a paid pack makes the whole pack a paid random item [38].
- You must show every possible outcome with numeric percent odds that add up to 100%, before purchase. A standalone (i) icon without a word like "Details" is not enough [37].
- Paid luck boosts must state their effect in numbers. Displayed odds must update live while the boost is active [37].
- Every outcome must give something. "There cannot be an outcome where the user simply loses their payment" [38].
- Earned-only randomness is exempt. Roblox's example: a chest earned and opened only through gameplay or earned currency is not a paid random item [38][37].

Who is restricted [40][41][42][43]:
- Check `ArePaidRandomItemsRestricted` from `PolicyService:GetPolicyInfoForPlayerAsync()`. If true, the player cannot touch paid random items at all [43].
- Allowed fixes: give a free earnable path, show outcomes in a fixed disclosed order, sell the outcome directly at its expected value, hide it, block it with a message, or teleport the player away [37].
- Regions named in the May 26, 2026 post: Australia, Belgium, Netherlands, United Kingdom and Brazil [38].
- UK users under 18 since August 13, 2024 [40]. All Australian users since September 20, 2024 [41].
- Brazil: under 18 from March 17, 2026, and adults who are not age-verified from March 31, 2026. This comes from a creator quoting Roblox's message on the DevForum [42].
- Trading paid items needs a separate check, `IsPaidItemTradingAllowed` [37][43].

Under-13: no global rule aimed at under-13 players and random rewards was found. The rules above are by region and by whether Robux is involved. Not confirmed.

Age tiers and ratings [44][45][46]:
- Account tiers: **Roblox Kids** (ages 5 to 8), **Roblox Select** (ages 9 to 15, and anyone self-declared 9+ until age-checked), **Roblox** (age-checked 16+) [45].
- Ratings: Minimal or Mild games can reach Kids and Select. Moderate games can reach Select and 16+. Restricted is 18+ only [44].
- Violence: "Mild" means unrealistic consequences, such as "bodies disappearing the moment their health reaches zero." Any realistic consequence, even once, makes it Moderate or higher [44].
- "Repeated" violence includes rare moments with many violent events in quick succession [44].
- Animal abuse is not allowed at any rating [44].
- To reach players under 16, the creator needs an age check, account verification (facial age estimation if under 18), 2-Step Verification, and either a 2+ month Plus or Premium subscription or a one-time refundable 1,000 Robux fee [45][46].
- The game then goes through an evaluation. First it is shown only to age-checked 16+ players. It becomes eligible after 250 unique plays by highly engaged age-checked players in 60 days [45]. (Re-checked in the docs source on 2026-10-04: still 250.)
- A "highly engaged player" must meet rules on account age, playtime in your game, and platform spend: a purchase anywhere on Roblox in the last 60 days, plus time in your game in that window. They do not have to spend in your game [45].
- An expedited review costs a refundable 50,000 Robux [45][46].
- The two pages describe the fee refund differently. The Kids and Select page says the publishing fee is refunded automatically 90 days after the game becomes eligible, or 90 days after payment if it never does [45]. The publishing page said it is refunded if the game keeps 25 highly engaged players for 60 days [46]. The publishing page was not re-checked, so which rule applies is not confirmed.
- If an update changes any questionnaire answer, you must resubmit the questionnaire [44].

## What Fruit Empire should take

Saving
- Keep one key per player holding one table: fruit on plot, index, shards, rebirths, pets, ores, boat owned, weapon skins, potions. Even a full 64-fruit plot plus pets and ores is tiny next to 4 MB [1].
- Save with `UpdateAsync` on a timer (for example every 60 to 120 seconds), on leave, and on server shutdown. Do not save on every merge. Each `UpdateAsync` costs one read and one write [1][2].
- Store fruit, ore and pet health only while it matters. Health from TNT damage can live in server memory and reset on rejoin. That keeps saves small.
- Use an Ordered DataStore for leaderboards only, and update it rarely. Its write budget is 30 + 5 × players [1].

Streaming
- Set `StreamOutBehavior = Opportunistic` and `ModelStreamingBehavior = Improved` [4].
- Make each fruit, ore, pet and boat an Atomic model. Then a client script waits for the model, not for each part [4].
- Do not mark plots, sky islands or the forest ring Persistent. If a script needs something always present (like a plot sign), keep it tiny [4].
- The underground base sits under the plot. Check it stays inside the streaming range when a player goes down. If not, call `RequestStreamAroundAsync` at the mine entrance [4]. The exact depth that works is not confirmed. Test it.
- Sky islands and the King's Throne arena are far from plots. Expect them to stream out. Client UI that points at them must handle "not loaded yet" [4].

Pets that wander and earn
- Treat pets as server data only: type, tier, owner, which plot. Earnings come from that data on the server, never from where the pet model is [10].
- Draw pets on the client. Each client picks random points inside the owner's plot bounds and moves anchored pet models with `PivotTo` or `BulkMoveTo`. Turn off `CanCollide`, `CanTouch` and `CanQuery` [10][9][6].
- Use `AnimationController`, not Humanoid. Animate on the client, and only for pets near the camera [6].
- Do not use PathfindingService for wandering on a flat plot. If forest pets ever need real paths (crossing the forest ring), share one computed path between many pets and limit how often `ComputeAsync` runs [15][16][17].
- Pool pet models when merging. Hide the two old ones and reuse them instead of destroying and making new ones [6].

Ores and fruit health (TNT, nukes, artillery)
- Keep fruit and ores anchored and server owned. Then clients cannot fake their position [19].
- Do all damage on the server. On impact, find targets in the radius, then subtract health by distance. If you use an `Explosion` for the look, set `DestroyJointRadiusPercent = 0` so it does not break models, and use `Hit` with distance for damage [26].
- Send one remote event for "TNT landed here", and let each client play the VFX. Do not tween health bars on the server. Server tweens replicate every frame [6].
- Glowing higher ores: use the Neon material or a low-rate emitter with `LightEmission = 1`. Keep rates low. Mobile caps each emitter at 100 particles per second anyway [14].

Boats
- A boat is one unanchored assembly. Make the driver the network owner on sit, and call `SetNetworkOwnershipAuto()` on exit, like Roblox's vehicle example [19].
- Boat health and breaking happen on the server only. A client that owns the boat could fake its position, so artillery hits on boats must be checked on the server [20].
- Keep boat part counts low. Big moving assemblies cause spikes when they stream in [4].
- If many boats will move at once, remember the community budget of about 40 to 60 moving physics objects per client [10].

Bats, swords and duels
- Check hits on the server. Rate-limit swings yourself with a cooldown. The platform cap (about 500 remote calls per second per client) is far too loose for anti-cheat [24].
- Knockback: reuse the Crown arena's push. The server fires a remote so the victim's own client sets the velocity at once, and adds a 0.12 s `LinearVelocity` (MaxForce 1e6) on the victim's root as a backup [51]. A plain server `ApplyImpulse` is allowed by the docs [21], but a grounded Humanoid fights it unless its state is changed first [50].
- Consider turning on Server Authority for the King's Throne duels. It is opt-in, released July 9, 2026, and supports Tools. It needs StreamingEnabled and Deferred signals, so test the whole game with it before shipping [22][23].
- Keep violence cartoonish. Health bars drain and the loser poofs or respawns, with no realistic injuries. That keeps the game "Mild" and open to Kids and Select accounts [44].

Assets
- Decide who owns the game (Samarth, Weilin, or a group) before uploading audio and animations. Then grant each audio and animation to the experience. Grants to a game are permanent [28][31][32].
- Upload each icon and texture once and reuse the ID. Duplicate IDs load twice into memory [6].
- Keep UI icons at 512×512 or smaller, and tiny icons at 256×256 [6].
- Upload new art a few hours before a release. Assets still in moderation are invisible to players [27].

Prompts
- Many fruit, ores and pets on one plot will crowd the screen. Only 16 prompts show at once by default, and only one per key with the default exclusivity [36][34]. Use different keys for different actions (feed pet vs mine ore), or set `Exclusivity` to `OneGlobally` near crowded areas [34].

Random rewards (stealing stays removed)
- Free random results are allowed with no extra rules: ore drop rolls from mining, which pet a forest animal turns into, free daily crates [38].
- If anything Robux-bought touches randomness, it becomes a paid random item. That includes a luck potion that boosts ore or pet rarity, a random-duration potion, a paid crate, or shards bought with Robux and spent on a roll [37][38].
- Paid random items then need: odds for every outcome before purchase, live-updating odds with boosts, no "nothing" outcome, and a PolicyService check with an alternative for restricted players [37][38][43].
- Simplest safe path: sell only fixed items for Robux (a specific weapon skin, a fixed-duration Aqua Affinity potion, the boat). Keep all randomness free [37].
- **Watch the Luck Potion.** Today the only Robux item in the code is the Merge All gamepass, and shards are not sold for Robux [53]. So the Luck Potion (7,500 shards, 5 minutes, raises the odds of better fruit spawns) counts as earned-only randomness [53][38]. If shards are ever sold for Robux, that potion becomes a paid probability modifier and needs shown odds, live odds while active, and the PolicyService check [37].
- Fill out the Maturity & Compliance questionnaire again whenever weapons, nukes or paid items change the answers [44].

Reaching young players
- Without passing the Kids and Select evaluation, Fruit Empire is only shown to age-checked 16+ players. Plan for the 250 highly engaged plays in 60 days, or the 1,000 Robux refundable fee path plus evaluation [45][46].

## What to avoid

- Do not make wandering pets Humanoid NPCs with server-run animations [6][10].
- Do not move hundreds of unanchored physics parts (fruit, ores, pets) and rely on replication. The community budget is about 40 to 60 moving physics objects per client [10].
- Do not mark big areas Persistent to "fix" streaming bugs [4].
- Do not run `ComputeAsync` for every pet every second [15][16][17].
- Do not trust the client for damage, knockback amounts, boat health or merge results [20].
- Do not save to the DataStore on every merge, hit or tick [1].
- Do not sell random rewards for Robux without odds, live boost odds and the PolicyService path [37][38].
- Do not let artillery or bats hurt tamed pets. Roblox bans animal abuse at every rating. Whether cartoon pets taking damage would count is not confirmed [44].
- Do not add realistic blood or injuries to sword duels. Any realistic consequence makes the game Moderate or higher and cuts off Roblox Kids [44].
- Do not change ParticleEmitter properties every frame [6].

## Open questions

- Does `ArePaidRandomItemsRestricted` return true for Roblox Kids or Select accounts outside the named regions? Not confirmed.
- ~~What is the max player count per Fruit Empire server?~~ Answered: 50 [48]. New question: should it drop to 12 to match the plots, or what do plotless players do? Server memory and DataStore budgets both scale with it [1][8].
- How deep can the underground base go before it streams out at the default radii? Needs a test in a live server [4].
- Does turning on Server Authority break current plot, carry-and-merge or sky-island code? It requires Deferred signals, which changes event timing [23].
- Does PathfindingService still slow down after many calls in October 2026? The reports are from 2024 to June 2026 [16][17][18].
- Do many PointLights on glowing ores cost much on mobile? No official numbers were found.
- ~~Who owns the game?~~ Answered: the user account `hikeALT12212` [48]. Still open: is that Samarth's account or Weilin's, and should the game move to a group before animations and audio are uploaded [28][31]?
- The two docs pages describe the 1,000 Robux fee refund differently. Which one applies [45][46]?

## Sources

1. https://create.roblox.com/docs/cloud-services/data-stores/error-codes-and-limits
2. https://create.roblox.com/docs/cloud-services/data-stores/best-practices
3. https://create.roblox.com/docs/cloud-services/memory-stores
4. https://create.roblox.com/docs/workspace/streaming
5. https://create.roblox.com/docs/reference/engine/enums/ModelStreamingMode
6. https://create.roblox.com/docs/performance-optimization/improve
7. https://create.roblox.com/docs/performance-optimization/design
8. https://create.roblox.com/docs/performance-optimization/identify
9. https://create.roblox.com/docs/reference/engine/classes/WorldRoot#BulkMoveTo
10. https://devforum.roblox.com/t/real-world-building-and-scripting-optimization-for-roblox/3127146
11. https://create.roblox.com/docs/projects/place-files
12. https://create.roblox.com/docs/art/modeling/specifications
13. https://create.roblox.com/docs/art/modeling/texture-specifications
14. https://create.roblox.com/docs/effects/particle-emitters
15. https://create.roblox.com/docs/characters/pathfinding
16. https://devforum.roblox.com/t/computeasync-pathfinding-calculation-taking-absurd-amounts-of-time/3044789
17. https://devforum.roblox.com/t/computeasync-pathfinding-starts-lagging-after-creating-enough-paths/4677784
18. https://devforum.roblox.com/t/degraded-performance-of-pathfindingservice-immense-slowdowns/4090905
19. https://create.roblox.com/docs/physics/network-ownership
20. https://create.roblox.com/docs/scripting/security/network-ownership
21. https://create.roblox.com/docs/reference/engine/classes/BasePart#ApplyImpulse
22. https://devforum.roblox.com/t/full-release-ship-fair-and-competitive-games-with-server-authority/4727993
23. https://create.roblox.com/docs/projects/server-authority
24. https://create.roblox.com/docs/reference/engine/classes/RemoteEvent
25. https://create.roblox.com/docs/reference/engine/classes/UnreliableRemoteEvent
26. https://create.roblox.com/docs/reference/engine/classes/Explosion
27. https://create.roblox.com/docs/projects/assets
28. https://create.roblox.com/docs/projects/assets/privacy
29. https://create.roblox.com/docs/audio/assets
30. https://create.roblox.com/docs/cloud/guides/usage-assets
31. https://devforum.roblox.com/t/improving-animation-asset-permissions/3852101
32. https://devforum.roblox.com/t/sharing-animation-assets-with-connections-and-groups/3892540
33. https://create.roblox.com/docs/animation/editor
34. https://create.roblox.com/docs/ui/proximity-prompts
35. https://create.roblox.com/docs/reference/engine/classes/ProximityPromptService
36. https://raw.githubusercontent.com/MaximumADHD/Roblox-Client-Tracker/roblox/Full-API-Dump.json (community mirror of the Roblox client API dump; used for default values)
37. https://create.roblox.com/docs/production/monetization/paid-random-items
38. https://devforum.roblox.com/t/clarifying-requirements-for-paid-random-items/4654622
39. https://about.roblox.com/community-standards
40. https://devforum.roblox.com/t/update-on-paid-random-items-restriction-for-uk-users-under-18/3072183
41. https://devforum.roblox.com/t/update-on-paid-random-items-restriction-for-australian-users/3153602
42. https://devforum.roblox.com/t/i-got-this-message-and-don%E2%80%99t-know-what-to-do/4526581
43. https://create.roblox.com/docs/reference/engine/classes/PolicyService
44. https://create.roblox.com/docs/production/promotion/content-maturity
45. https://create.roblox.com/docs/production/publishing/kids-and-select
46. https://create.roblox.com/docs/production/publishing/publish-games-and-places
47. https://create.roblox.com/docs/reference/engine/classes/Path
48. Roblox Games API, Fruit Empire universe 10764283455 (read 2026-10-04): https://games.roblox.com/v1/games?universeIds=10764283455
49. Local code: `~/fruit-empire/src-export-2026-10-04/ReplicatedStorage.GardenConfig.lua` (comment "measured on all twelve plots", line 107)
50. DevForum, "Knockback pretty uneffective on humanoids" (28 Dec 2024): https://devforum.roblox.com/t/knockback-pretty-uneffective-on-humanoids/3324305
51. Local code: `~/fruit-empire/src-export-2026-10-04/ServerScriptService.GardenService.CrownArena.lua` (the `push` function, about lines 507–535)
52. DevForum announcement, "Extended Light Ranges: Doubling the Limit to 120" (23 Sep 2025): https://devforum.roblox.com/t/extended-light-ranges-doubling-the-limit-to-120/3954367
53. Local code: `~/fruit-empire/src-export-2026-10-04/ReplicatedStorage.GardenConfig.lua` (`MergeAllGamepassId`; `LuckPotion` price 7500, duration 300) and a search of the export for `MarketplaceService` (only the Merge All gamepass uses it)

## Review (critic pass, 2026-10-04)

- **Verified in the docs source:** DataStore limits (60 + 40 × players per server; 300 + 40 × CCU read and 300 + 20 × CCU write per experience; 4,194,304 characters per key; 25 MB/min read and 4 MB/min write per key; 500 MB + 1 MB per lifetime user), the `ApplyImpulse` ownership wording, and the Kids and Select threshold of 250 plays in 60 days.
- **Added:** Fruit Empire's real owner, avatar type and server size [48]; the 50-players-vs-12-plots gap; the "highly engaged player" definition; the light-range change [52]; the practical knockback problem and the arena's working pattern [50][51].
- **Corrected:** the knockback advice. "Apply `ApplyImpulse` from the server" is allowed but weak on standing characters.
- **Still not confirmed:** whether `ArePaidRandomItemsRestricted` is true for Kids and Select accounts outside the named regions, the safe underground depth for streaming, PointLight cost on mobile, and which fee-refund rule applies.
