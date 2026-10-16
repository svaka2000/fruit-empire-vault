---
tags: [current-game, code-map]
date: 2026-10-04
sources:
  - src-export-2026-10-04/StarterPlayer.StarterPlayerScripts.RoyalUI.lua
  - src-export-2026-10-04/StarterPlayer.StarterPlayerScripts.RoyalCombat.lua
  - src-export-2026-10-04/StarterPlayer.StarterPlayerScripts.KingClient.lua
  - src-export-2026-10-04/StarterPlayer.StarterPlayerScripts.PlayerVision.lua
  - src-export-2026-10-04/StarterPlayer.StarterPlayerScripts.SkyIslandLife.lua
  - src-export-2026-10-04/StarterPlayer.StarterPlayerScripts.IslandWater.lua
  - src-export-2026-10-04/StarterPlayer.StarterPlayerScripts.MusicPlayer.lua
  - src-export-2026-10-04/StarterPlayer.StarterPlayerScripts.FeatherFall.lua
  - src-export-2026-10-04/ServerScriptService.GardenService.Ambient.lua
  - src-export-2026-10-04/ServerScriptService.GardenService.CrownArena.lua (sword, swing, slam and push sections only)
  - src-export-2026-10-04/ReplicatedStorage.CrownConfig.lua (keys this subsystem reads)
  - src-export-2026-10-04/ReplicatedStorage.GardenConfig.lua (keys this subsystem reads)
  - src-export-2026-10-04/ServerScriptService.GardenService.Crown.lua (publishCrown only)
  - src-export-2026-10-04/ServerScriptService.GardenService.Progression.lua (refreshKing only)
---

# Client World, Royal Client, Music and Ambient

Part of [[Brain Dump 2026-10-04]] planning. Code map of the scripts listed in the frontmatter, from the 4 Oct 2026 export.

## 1. What it does

This subsystem is the code that runs on each player's own device (the "client") to draw the world and the King's Throne screens, plus one server module that runs the game's always-on loops. RoyalUI and RoyalCombat draw the King's Throne HUD, menus and arena controls; they only show what the server publishes and send requests the server may refuse (RoyalUI.lua:4-7, RoyalCombat.lua:16-18). KingClient draws the King's equipped aura on every client (KingClient.lua:2-8, 76-80). PlayerVision, MusicPlayer, SkyIslandLife, IslandWater and FeatherFall decide how the world looks and sounds: hazard blackouts, the day cycle, biome music, sky-island life, moving water textures and the Feather Falling glide. Ambient is a server module that runs the drown, fall, cloud-sink and thin-air sweep, the fruit spawn timer, the income tick and the plot-lock timer (Ambient.lua:1-4).

## 2. Scripts

| Script | Role | Lines |
|---|---|---|
| RoyalUI.lua | King's Throne HUD: King chip, announcements, duel board, contest card, challenge prompt, menus, titles over heads, world FX, event handlers | 1547 |
| RoyalCombat.lua | Arena input (mouse, keys, gamepad, touch buttons), local cooldowns, local dash, charge ring | 374 |
| KingClient.lua | Builds the King's aura locally and makes it follow the body | 267 |
| PlayerVision.lua | Blur and colour effects for hazards and the sky biome, heartbeat sound, day/night lighting, airborne half of the sea currents, publishes music levels | 505 |
| SkyIslandLife.lua | Sky-island ambience: drifting pieces, cloud crumbs, petals, fireflies, passing clouds, lamps, tree sway, flags, bird ambience | 794 |
| IslandWater.lua | Scrolls tagged water textures so ponds and the waterfall look like they move | 48 |
| MusicPlayer.lua | Playlist plus four looped biome tracks (sky, forest, Royal Capital, fight), ducking and mute | 375 |
| FeatherFall.lua | Caps downward speed while the Feather Falling potion runs | 41 |
| GardenService.Ambient.lua (server) | Drown / fall / cloud-sink / thin-air sweep, feathers, ragdoll, spawn sweep, income tick, lockdown timer | 519 |

## 3. How it works

### Terms used

- **Attribute**: a named value stored on a Roblox object with SetAttribute. Values the server sets reach every client. Values a client sets stay on that client.
- **RoyalState**: an object in ReplicatedStorage. The server writes the throne's state onto its attributes, and RoyalUI reads them (RoyalUI.lua:41, 54).
- **RemoteEvent / RemoteFunction**: Roblox objects that carry messages between client and server. A RemoteFunction waits for a reply.
- **Tween**: a TweenService animation that changes a property smoothly over a set time.
- **RenderStepped / Heartbeat**: callbacks that run every frame.
- **Ring-out**: knocking a fighter off the arena ring. Duels are won by ring-outs, not damage (CrownArena.lua:4-7).
- **Stagger**: a per-fighter number that grows with each hit. Knockback is multiplied by (1 + stagger / 100) (CrownConfig.lua:88-90).
- **toolanim**: a StringValue named "toolanim" placed inside a Tool. Roblox's default Animate script reads it and plays its own built-in tool animation (CrownArena.lua:169-171).
- **Conveyor plate**: a part whose velocity pushes whatever stands on it. The sea current and the return lanes are made of these (GardenConfig.lua:1310, 1313-1319).
- **Hazard**: PlayerVision's 0-to-1 measure of how blind the player is getting (sea, thin air) (PlayerVision.lua:8-12, 352).

### RoyalUI (the King's Throne on screen)

1. Loads CrownConfig and quits if `C.Enabled` is false (RoyalUI.lua:33-34). Waits for the remotes RoyalRequest, RoyalEvent, RoyalQuery and the RoyalState object (RoyalUI.lua:37-41).
2. Sets one font for all text: `FONT = FredokaOne` (RoyalUI.lua:43). Every TextLabel is made by `label()`, which sets `FontFace = FONT` (RoyalUI.lua:176-205). The two TextButtons have empty text (RoyalUI.lua:257, 314). So no other font is used in this script.
3. Builds a small GUI kit: `frame()` (rounded box, black border, optional grey gradient), `label()` (scaled text with a black stroke and RichText), `button()` (a coloured face on a darker "lip", squashes 3 px when pressed, plays a Sparkle sound on click) (RoyalUI.lua:134-289). Everything scales by `fitScale()`, clamped 0.55 to 1.15 against a 1400x900 reference (RoyalUI.lua:119-125).
4. Icons: `iconId(key)` returns an image only if `C.Icons[key]` is a number above 0 (RoyalUI.lua:214-218). Every entry in `C.Icons` is 0 today (CrownConfig.lua:390-397), so every spot keeps its emoji. `dropEmoji()` strips only a leading emoji run, and only when an icon exists (RoyalUI.lua:222-224, 272).
5. Top-left stack: the King chip (who rules, title, a badge for vulnerable / protected / reign time), then CHALLENGE / KINGDOM / WATCH buttons, then the Royal Court line (RoyalUI.lua:299-346, 392-393). `refreshChip()` fills it (RoyalUI.lua:400-431). CHALLENGE shows only when not King, `RoyalEligible` is true and not `InArena` (RoyalUI.lua:425).
6. Top-centre stack: decree card, duel board, contest card (RoyalUI.lua:352-391, 554-617). The whole stack hides while the garden's OfflinePopup is visible (RoyalUI.lua:360-369).
7. Announcements: a queue of cards. Each slides in (0.35 s Back Out), holds 2.2 s (short) or 3.4 s, slides out (0.3 s Quad In) (RoyalUI.lua:483-523). A line equal to the last queued line is dropped (RoyalUI.lua:512-513).
8. Big centre text (`bigText`): starts at UIScale 1.8, tweens to 1 in 0.3 s Back Out and fades in over 0.15 s; after the hold it scales to 0.7 and fades out over 0.25 s (RoyalUI.lua:536-550).
9. Challenge prompt for the King (ACCEPT / DECLINE with a timer bar, ZIndex 20), and the Royal Trial call card (RoyalUI.lua:671-709). The timer bar is driven in Heartbeat (RoyalUI.lua:1537-1541).
10. Menus in one shared window (`menuRoot`): kingdom, status, styles, decrees (RoyalUI.lua:713-961). Status and styles and decrees call RoyalQuery and drop the reply if another menu opened meanwhile (`menuToken`, RoyalUI.lua:806-808, 838-843).
11. In the world: a BillboardGui title over the King's head (RoyalUI.lua:977-1002); a stagger tag and a Highlight outline over each fighter (gold for Crown Guard, blue for blocking) (RoyalUI.lua:1006-1049); the crown flying from old King to new King along a curve in 1.15 s (RoyalUI.lua:1140-1191); a 2.2 s camera look at the throne (RoyalUI.lua:1195-1215); spinning crowns and waving flags tagged with `Spin`, `Bob`, `Wave` attributes within 320 studs of the camera (RoyalUI.lua:1218-1269).
12. Server messages arrive on RoyalEvent as `(kind, data)` and go to `handlers[kind]` inside a pcall (RoyalUI.lua:1492-1498).
13. Refresh: any RoyalState attribute change re-runs the matching refresh by name prefix (`Duel`, `Contest`, `Decree`) (RoyalUI.lua:1502-1511). A loop also refreshes every 0.5 s (RoyalUI.lua:1521-1533).

### RoyalCombat (arena controls)

1. Quits if `C.Enabled` is false (RoyalCombat.lua:26-27). Uses the RoyalCombat remote and RoyalState (RoyalCombat.lua:32-34). Font is FredokaOne via its own `label()` (RoyalCombat.lua:36, 176-194).
2. `fighting()` is true only when the player is `InArena` and a duel is in "fight" or a contest is in "brawl" or "crown" (RoyalCombat.lua:41-46).
3. Inputs: Mouse1 / R2 attack, F / L2 block, Q / B dash, E / Y ability, R / X Crown Guard, T / DPadUp summon (RoyalCombat.lua:148-165).
4. Attack: press stores `chargeStart` and sends "charge"; release sends "heavy" if held 0.45 s or more, else "strike" (RoyalCombat.lua:65-84).
5. Dash runs on the client first: sets the root velocity to `dir * 70 + (0, 6, 0)`, then cuts sideways speed to 35% after 0.18 s, then tells the server (RoyalCombat.lua:86-110).
6. Ability: Royal Shockwave for the King in a contest, otherwise Royal Slam (RoyalCombat.lua:122-133).
7. Local cooldowns are only for the button sweeps; the server keeps its own (RoyalCombat.lua:53-59).
8. Touch: six round buttons placed around the jump button, smaller (x0.72) on a small screen (RoyalCombat.lua:204-264, 284-315). A ring around Attack fills while charging and turns orange when full (RoyalCombat.lua:267-281, 355-361). The GUI is on only while `InArena` (RoyalCombat.lua:320-331).

### Sword swing, exactly as coded today

- The sword is built on the server: a Tool "RoyalSword" with a Handle part 1x0.8x4 and the classic mesh `rbxasset://fonts/sword.mesh` with `SwordTexture.png` (CrownArena.lua:172-195). It is force-equipped and re-equipped if unequipped (CrownArena.lua:197-220).
- Tap strike: the server calls `animate(f, "Slash")`, fires the "swing" event with `heavy = false`, and checks hits 0.08 s later (CrownArena.lua:570-581).
- Heavy strike: `animate(f, "Lunge")`, "swing" with `heavy = true`, hits checked after `Heavy.windup` 0.12 s (CrownArena.lua:583-596). A release held less than `charge - 0.15` s is downgraded to a strike (CrownArena.lua:587).
- `animate()` puts a StringValue named "toolanim" with value "Slash" or "Lunge" into the sword and removes it after 1 s (CrownArena.lua:229-238). For players, Roblox's default Animate script plays its built-in tool slash or lunge from that. No custom animation asset is used for players.
- Knights (NPC fighters) instead play an Animator track `rbxassetid://129967390` with a 0.05 s fade (CrownArena.lua:239-241, 323). They play this same track for "Slash" and "Lunge" alike.
- On clients, the "swing" handler only plays the Swing or HeavySwing sound at the attacker (RoyalUI.lua:1417-1421). There is no trail, slash mesh or tween.
- On a hit, `handlers.hit` fires `burst()`: unblocked light hit 14 white sparkles at speed 13, size 0.7; heavy 26 orange (255,170,60) sparkles at speed 20, size 1; blocked 10 light-blue sparkles at speed 10, size 0.6 (RoyalUI.lua:1405-1413). `burst()` is a one-shot ParticleEmitter: sparkles_main.dds, LightEmission 0.9, lifetime 0.4-0.8 s, spread 180 degrees, removed after 1.5 s (RoyalUI.lua:1078-1094). The victim's camera shakes at strength 1.6 (heavy 3) for 0.25 s (RoyalUI.lua:1414). `shake()` adds a random X and Y rotation each frame that shrinks linearly to zero (RoyalUI.lua:1126-1137).

### Royal Slam, exactly as coded today

- Client: E sends "slam" if not cooling down (10 s, or x0.8 for an Underdog) (RoyalCombat.lua:122-133, 57).
- Server: `animate(f, "Lunge")` (the same lunge as the heavy strike), fires "slamStart" with the position, waits `Slam.windup` 0.28 s, fires "slam" with position and radius 11, then hits every fighter within 11 studs in every direction (CrownArena.lua:632-646). The call is `targets(f, radius, 180, true)`, and the last `true` means "all around", so facing does not matter (CrownArena.lua:642, 482-505).
- Client "slamStart": an empty function (RoyalUI.lua:1432). Nothing is shown during the 0.28 s windup.
- Client "slam": `ring()` 2.6 studs below the position: an anchored Neon cylinder part, size (0.6, 2, 2), turned 90 degrees on Z so it lies flat, gold, Transparency 0.25; it tweens over 0.45 s (Quad Out) to size (0.6, 22, 22) and Transparency 1, then is removed at 0.55 s (RoyalUI.lua:1096-1103, 1425). Then 30 gold sparkles at speed 22, size 1.1 (RoyalUI.lua:1426), the Slam sound at 1.2x volume (RoyalUI.lua:1427), and a camera shake of 2.5 for 0.35 s for anyone within 30 studs (RoyalUI.lua:1428-1429).
- King's Shockwave: two rings, radius 15 pale gold over 0.5 s and radius 10.5 gold over 0.35 s, plus the Slam sound at 1.3x (RoyalUI.lua:1434-1440).
- Knockback: the server sends "knockback" with a velocity to the victim's own client (CrownArena.lua:510-514). RoyalUI sets the Humanoid to Freefall and writes the root velocity (RoyalUI.lua:1482-1490).

### KingClient (the King's aura)

1. Quits if `GardenConfig.KingEnabled` is false (KingClient.lua:30-33). Builds everything in a local folder "KingVisuals_Local" (KingClient.lua:38-40).
2. Reads `workspace.KingUserId` and the King's `KingEffect` attribute (KingClient.lua:199-208). Since 3 Oct the crown is a worn accessory from the server, so `buildVisual` returns at once for the "crown" kind (KingClient.lua:77-80).
3. Aura kinds: "limbs" copies emitters onto matching R15 or R6 limbs, "root" moves beam attachments into the root, "follow" is a loose model pinned to the root each frame (KingClient.lua:108-163, 236-261).
4. Hides the aura while the King is invisible, except in the arena (KingClient.lua:179-183, 210-223, 242-245).

### PlayerVision (sight, light, currents, music levels)

1. Creates its own BlurEffect and ColorCorrectionEffect on Lighting (PlayerVision.lua:47-60) and a looped Heartbeat sound (PlayerVision.lua:116-121).
2. Each frame, if standing on something, casts a 12-stud ray down. Sky Islands parts give "sky", `CurrentPlate` or `Seabed` give "water", `ReturnPlate` gives "return", anything else "land", and no hit at all gives "water" (PlayerVision.lua:141-153). Moving faster than 30 studs/s horizontally on non-sky, non-return ground counts as "water" (PlayerVision.lua:260-265). The answer is kept while airborne (PlayerVision.lua:174, 271).
3. Airborne in a return lane: adds speed along the lane up to 48 studs/s at 160 studs/s^2 (PlayerVision.lua:272-280). Airborne over "water": adds outward speed up to 42 studs/s at 20 studs/s^2 (PlayerVision.lua:281-297). This is the client half of the sea currents.
4. Hazard: sea blindness ramps with distance from the map centre between `DrownFadeStart` and `DrownFadeFull`; thin-air blindness ramps with height unless Aether is active. The worse one wins (PlayerVision.lua:316-324).
5. Sky biome glow ramps with height while Aether is active and is forced to 1 on sky ground (PlayerVision.lua:309-316). Forest level ramps with distance between 650 and 990 studs (PlayerVision.lua:330-342).
6. Publishes `VisionHazard`, `VisionSky`, `VisionForest` on the local player in steps of 0.01 for MusicPlayer (PlayerVision.lua:371-396).
7. Sets `Lighting.ClockTime` every frame from `Config.ClockTimeAt(server time)` and lowers Brightness and ambient colours at night (PlayerVision.lua:402-418).
8. Hard-fall blackout: waits `FallHoldTime` after the server's `FellHard` stamp, then fades (PlayerVision.lua:209-211, 422-433). Wakes up blurred after such a death (PlayerVision.lua:219-226).
9. Writes blur size and tint every frame from hazard, fall and sky values (PlayerVision.lua:438-469). Makes the Ocean opaque while the sky glow is above 0.5, and fades the neon OceanGlow at night (PlayerVision.lua:478-492). Heartbeat volume and speed follow the hazard (PlayerVision.lua:497-504).

### MusicPlayer

1. Quits if `Config.MusicTracks` is empty (MusicPlayer.lua:28-29). Creates the playlist Sound in SoundService (MusicPlayer.lua:31-35).
2. Creates four looped tracks at volume 0: SkyMusic, ForestMusic, RoyalMusic, FightMusic (MusicPlayer.lua:79-111).
3. `applyVolume()` is the only writer of these volumes. Mute is the `MusicMuted` attribute. Fight takes its share first, then the Royal Capital, then sky and forest split the rest, with square-root (equal-power) gains. Hazard ducks volume, lowers pitch and muffles highs (MusicPlayer.lua:120-194).
4. Royal music plays inside the Royal Capital's bounding box; fight music plays while RoyalState `Phase` is "duel" or "contest" and the player is `InArena` or within 140 studs of ArenaCenter (MusicPlayer.lua:201-236).
5. Heartbeat eases every level toward its target and calls `applyVolume` (MusicPlayer.lua:241-273).
6. At start it preloads each playlist id and drops ones that do not load (MusicPlayer.lua:278-304). Plays in order with `MusicGap` between tracks (MusicPlayer.lua:355-375). Biome ids get a one-time load check that only warns (MusicPlayer.lua:336-353).

### SkyIslandLife

1. Waits up to 60 s for `workspace["Sky Islands"]` (SkyIslandLife.lua:54-55). Effects live in a folder under the camera (SkyIslandLife.lua:60-62).
2. `setupIsland` reads size from the island model's attributes and finds its "Cloud" model (SkyIslandLife.lua:628-676). `rescan` finds Drift parts, CherryTree canopies (pink parts), NightLamp lights and Flag models (green cloth, pole taller than 10) (SkyIslandLife.lua:124-222).
3. Heartbeat: ambience volume by height over the island; lamps by night amount; near the island it moves drift, sway and flag parts with BulkMoveTo, spawns petals and fireflies, and makes cloud crumbs and crunch sounds under walking characters (SkyIslandLife.lua:691-794).

### IslandWater

Collects every Texture tagged "IslandWaterFlow" and scrolls its offsets by its `FlowU` and `FlowV` attributes each frame (IslandWater.lua:20-48). No part moves, so this water carries nobody (IslandWater.lua:5-9).

### FeatherFall

While `FeatherFallingEndsAt` is in the future and the player is not `InArena`, it caps downward speed at `Effects.FeatherFalling.fallSpeed` (22) each Heartbeat (FeatherFall.lua:18-41). Sideways speed is never touched (FeatherFall.lua:30-31).

### Ambient (server loops)

1. Takes what it needs from `G`, the shared runtime table of GardenService (Ambient.lua:12-22).
2. Every `DrownTick` (0.2 s), for each living character (Ambient.lua:215-223):
   - Sky ground: a 12-stud ray down; standing on Sky Islands sets `OnSkyGround` (Ambient.lua:230-248).
   - Feathers: one FeatherTrail emitter on the root, on while gliding, a puff on landing (Ambient.lua:77-116, 281-282).
   - Landing: tracks the highest point in `FallPeak`. On landing after a drop of more than 220 studs it stamps `FellHard`, sends a Notify warning and ragdolls the body; the kill comes `FallHoldTime` later (Ambient.lua:271-301, 118-184). Feather Falling clears `FallPeak` every sweep (Ambient.lua:283-284).
   - Sky legs: flips `SkyLegs` at y 430 and asks `applyWalkSpeed` to re-read (Ambient.lua:309-313).
   - Cloud sink: standing still on cloud raises `CloudSink` from 0 to 1 after 3 s over 4 s; stone with `SolidGround` is exempt (Ambient.lua:320-342, 197-205).
   - Thin air: above y 418 off sky ground and without Aether, Notify plus death (Ambient.lua:353-357).
   - Deep water: sets `InDeepWater` if carried faster than 30 studs/s or over a water floor or nothing (Ambient.lua:359-375). Kills only past radius 2825 (Ambient.lua:376-381).
3. Hooks player join and leave, with an assert that `onPlayerLeaving` takes exactly one parameter (Ambient.lua:389-401).
4. Spawn sweep every `SPAWN_TICK` (0.5 s, Upgrades.lua:292): each owned, loaded plot spawns when its own interval is due, divided by `G.spawnBoost` (Fruit Frenzy decree) (Ambient.lua:405-444).
5. Income tick every 5 s: `rate = plotBase(plot) * cashMultiplier(player)`, added to `balances[player]`, written to `CashPerSecond`, and sent to the owner as `(multiplier, dt)` on the Income remote (Ambient.lua:448-484).
6. Lock sweep every 0.15 s: removes expired plot domes and clears the lock attributes. TNT refusal is not here; it is in useTool (Ambient.lua:486-513).

### Where the water currents and swimming really live

- The currents are not built by any exported script. They are saved parts in the place: outward `CurrentPlate` plates in Main Island.AnimatedWater at 42 studs/s, and `ReturnPlate` plates in six lanes (Main Island.ReturnLanes) at 48 studs/s toward the island (GardenConfig.lua:1310, 1313-1319). The only script mention of `CurrentPlate` outside config is a builder filter (ShallowsBuilder.lua:249).
- Code that exists only because of the currents: PlayerVision's return-lane and outward airborne pushes (PlayerVision.lua:266-297), the swept-speed test in PlayerVision and Ambient (PlayerVision.lua:260-265, Ambient.lua:363-364), the "return" ground kind (PlayerVision.lua:149-151), and the forest music ramp, which assumes distance equals progress along the ride (PlayerVision.lua:326-335).
- Swimming: no script in the export handles swimming. A search for "swim" finds nothing. The code treats `Seabed` and `CurrentPlate` as the floor under a player in the sea (GardenConfig.lua:1327, PlayerVision.lua:148), and the Ocean is a single transparent part about six studs above the seabed (PlayerVision.lua:62-71). Whether any Terrain water exists cannot be seen from the scripts.

## 4. Data

### Saved fields

| Field | Read / write | Where |
|---|---|---|
| `profile.cash` (through `balances[player]`) | Ambient adds income to `balances` every tick; Persistence saves it as `profile.cash` | Ambient.lua:468, Persistence.lua:49 |

No other script here reads or writes the save document. RoyalQuery's history and records come from the server's DataStores `CrownHistory` and `CrownRecords` (CrownConfig.lua:345-346).

### RoyalState attributes (written by the server, read here)

| Attribute | Read by |
|---|---|
| KingUserId, KingName, KingTitle, KingStyle, ReignStart, ProtectedUntil, VulnerableUntil, Court, Defenses | RoyalUI.lua:396, 401-421, 426, 814 |
| DuelState, ContestState, GateOpen | RoyalUI.lua:429-430, RoyalCombat.lua:43-45 |
| DecreeKey, DecreeTitle, DecreeText, DecreeEndsAt, DecreeGoal, DecreeProgress | RoyalUI.lua:435-459 |
| DuelKind, DuelAId, DuelBId, DuelAName, DuelBName, DuelAScore, DuelBScore, DuelSudden, DuelTarget | RoyalUI.lua:577-591 |
| ContestEndsAt, ContestEntrants, ContestClaim, ContestKing, ContestZone, ContestControl, ContestLeader, ContestLeaderPower | RoyalUI.lua:628-665 |
| TrialOpen | RoyalUI.lua:1506-1508 |
| Phase ("duel", "contest") | MusicPlayer.lua:229-230 |

### Player attributes

| Attribute | Set by | Read here |
|---|---|---|
| RoyalEligible, RoyalWhy, RoyalBusy, InArena, CrownPower, Stagger, Underdog, CrownGuard, Blocking | server (Crown modules) | RoyalUI.lua:425, 430, 568-569, 637, 651, 872, 1031-1037, 1507; RoyalCombat.lua:42, 57, 66 |
| InvisibleEndsAt | server (potions) | RoyalUI.lua:973, KingClient.lua:180 |
| KingEffect | server (Progression.lua:242) | KingClient.lua:207 |
| FeatherFallingEndsAt | server (potions) | FeatherFall.lua:26 |
| AetherEndsAt | server (potions) | PlayerVision.lua:157 |
| FellHard (server time) | Ambient.lua:295 | PlayerVision.lua:209-211 |
| CashPerSecond | Ambient.lua:469 | other HUD scripts |
| LockEndsAt (cleared) | Ambient.lua:506 | other scripts |
| VisionHazard, VisionSky, VisionForest (0..1, local only) | PlayerVision.lua:376, 386, 395 | MusicPlayer.lua:242-244 |
| MusicMuted | mute button (elsewhere) | MusicPlayer.lua:121, 238 |

### Character attributes (server, Ambient)

| Attribute | Meaning | Where |
|---|---|---|
| OnSkyGround | standing on Sky Islands, kept through a jump | Ambient.lua:243-247 |
| FallPeak | highest Y since leaving support | Ambient.lua:271, 284-290 |
| Ragdolled | ragdoll already applied | Ambient.lua:125-126 |
| SkyLegs | above `SkyJumpY` | Ambient.lua:309-312 |
| CloudSink | 0..1 sink into cloud; read by SkyIslandLife for crumbs | Ambient.lua:339-341, SkyIslandLife.lua:370 |
| InDeepWater | in the sea (carried, over water, or over nothing) | Ambient.lua:372-377 |

### Workspace and instance attributes, tags

| Name | On | Where |
|---|---|---|
| KingUserId | workspace (written by Crown.lua:694) | KingClient.lua:199, 263 |
| Spin, Bob | Models that turn and bob | RoyalUI.lua:1226-1228 |
| Wave | flag BaseParts | RoyalUI.lua:1229-1231 |
| CrownRole ("band", "gem", "cap") | parts of KingAssets.RoyalCrown | RoyalUI.lua:1152-1153 |
| BobAmp, BobSpeed, BobPhase, SpinSpeed | Drift parts | SkyIslandLife.lua:137-140 |
| Deck, CenterX, CenterZ, Radius, MinX, MaxX, MinZ, MaxZ, HoleX, HoleZ | each sky-island model | SkyIslandLife.lua:638-649 |
| FlowU, FlowV | water Textures | IslandWater.lua:43-44 |
| tag "IslandWaterFlow" | water Textures | IslandWater.lua:20, 33-35 |
| SolidGround | castle / capital ancestors | Ambient.lua:201 |
| LockedUntil (cleared) | plot model | Ambient.lua:505 |

### Remotes (all in ReplicatedStorage.GardenRemotes)

| Remote | Type, direction | Payload |
|---|---|---|
| RoyalRequest | RemoteEvent, client to server | `(action, arg)`: "challenge", "watch", "joinContest", "respond" + bool, "joinTrial", "trial", "passage", "style" + key, "decree" + key (RoyalUI.lua:342, 346, 615, 693, 698, 708, 820, 829, 906, 939) |
| RoyalEvent | RemoteEvent, server to client | `(kind, data)`. Kinds handled: crowned, vacant, announce, challenge, prompt, declined, duelIntro, countdown, fight, point, duelEnd, hit, swing, slam, slamStart, shockwave, dash, crownGuard, fall, contestAnnounce, crownAppears, contestEnd, decree, decreeEnd, achievement, openMenu, throne, huntSpot, knockback (RoyalUI.lua:1301-1490) |
| RoyalQuery | RemoteFunction | `InvokeServer("king" / "history" / "me" / "decrees")` returns a table (RoyalUI.lua:840-842, 893, 920) |
| RoyalCombat | RemoteEvent, client to server | `(action, arg)`: "charge", "strike", "heavy", "dash" + direction, "block" + bool, "slam", "shockwave", "guard", "summon" (RoyalCombat.lua:68-143; server CrownArena.lua:650-661) |
| Income | RemoteEvent, server to owner | `(multiplier, dt)` (Ambient.lua:477-478; created GardenService.lua:76) |
| Notify | RemoteEvent, server to client | a warning string (Ambient.lua:296, 355, 379; created GardenService.lua:66) |

### Config keys read

| Config | Keys | Read by |
|---|---|---|
| CrownConfig | Enabled, Colors, Sounds, Icons, Text, CrownStyles, CrownStyleByKey, DefaultStyle, Decrees, AchievementByKey, LongestReignTitle, ClaimPower, ResponseTime, HoverTitle, CameraEmphasis, EmphasisRange | RoyalUI.lua (throughout) |
| CrownConfig | Combat (Strike, Heavy, Dash, Block, Slam, Underdog), KingAbilities (Shockwave, Guard) | RoyalCombat.lua:28-29 |
| GardenConfig | KingEnabled, KingAssets, KingDefault, KingEffectByKey, KingCrownWidth, KingHoverHeight, KingBob, KingBobSpeed, KingSpinSpeed | KingClient.lua |
| GardenConfig | IslandName, SkyModels, DrownGroundProbe, DrownWaterFloors, ReturnFloors, ReturnSpeed, ReturnAirAccel, CurrentSpeed, CurrentAirAccel, DrownSweptSpeed, DrownFadeStart, DrownFadeFull, AltitudeFadeStart, AltitudeFadeFull, SkyBiomeY, SkyBiomeFull, ForestBiomeStart, ForestBiomeFull, DrownCurve, DrownOnset, DrownRecover, SkyEase, ForestEase, Fall* (HoldTime, FadeIn, FadeOut, WakeBlur, WakeEase), DrownBlur, DrownTintFloor, DrownDim, DrownContrast, DrownDesaturate, Sky* (Blur, Tint, Brightness, Contrast, Saturation), Heartbeat* (Sound, Volume, Floor, SpeedUp), ClockTimeAt, NightSunDay, NightSunNight, NightBrightness, NightOutdoorAmbient, NightAmbient | PlayerVision.lua |
| GardenConfig | MusicTracks, MusicVolume, MusicTrackGain, MusicGap, MusicDuck* (Curve, Floor, Ease), MusicPitchFloor, MusicMuffleHigh, MusicMuffleMid, Sky/Forest/Royal/FightMusicId, ...Volume, ...Ease, FightMusicRange | MusicPlayer.lua |
| GardenConfig | Effects.FeatherFalling (fallSpeed, featherTexture, featherRate, featherFallSpeed, landingPuff) | FeatherFall.lua:19, Ambient.lua:79 |
| GardenConfig | DrownTick, DrownGroundProbe, SkyModels, FallSettleSpeed, FallKillDrop, FallWarning, FallHoldTime, SkyJumpY, CloudStillSpeed, CloudHoldTime, CloudSinkTime, AltitudeKill, AltitudeWarning, DrownSweptSpeed, DrownWaterFloors, DrownKill, DrownWarning, SpawnInterval, IncomeTick, Capacity, FriendBonus | Ambient.lua |

## 5. Numbers

### Royal combat (CrownConfig.Combat, CrownConfig.lua:92-145)

| Key | Value |
|---|---|
| Combat.WalkSpeed / JumpPower / MaxStagger | 19 / 54 / 160 |
| Strike range / arc / knockback / lift / cooldown / stagger | 7.5 / 70 deg / 40 / 16 / 0.42 s / 9 |
| Heavy charge / windup / range / arc / knockback / lift / cooldown / stagger | 0.45 s / 0.12 s / 8.5 / 60 deg / 74 / 26 / 1.25 s / 18 |
| Dash speed / time / cooldown / knockbackScale | 70 / 0.18 s / 2.2 s / 0.6 |
| Block maxTime / cooldown / knockbackScale / staggerScale / walkScale / heavyBreak | 1.6 s / 0.9 s / 0.3 / 0.25 / 0.55 / 0.65 |
| Slam windup / radius / knockback / lift / cooldown / stagger | 0.28 s / 11 / 92 / 32 / 10 s / 22 |
| Underdog deficit / dealt / taken / speed / cooldowns | 2 / 1.15 / 0.9 / 1.06 / 0.8 |
| KingAbilities.Shockwave radius / knockback / lift / cooldown / stagger | 15 / 88 / 30 / 8 s / 15 (CrownConfig.lua:149) |
| KingAbilities.Guard duration / cooldown | 2.5 s / 16 s (CrownConfig.lua:150) |
| Strike hit delay (hard-coded) | 0.08 s (CrownArena.lua:575) |

### Throne timing (CrownConfig)

| Key | Value |
|---|---|
| MinDiscovered | 10 fruits (CrownConfig.lua:38) |
| ResponseTime | 15 s (CrownConfig.lua:55) |
| DuelTarget / DuelIntroTime / DuelCountdown | 3 points / 3.5 s / 3 (CrownConfig.lua:69-73) |
| ClaimPower | 100 (CrownConfig.lua:199) |
| EmphasisRange | 260 studs (CrownConfig.lua:360) |

### RoyalUI hard-coded values

| What | Value | Where |
|---|---|---|
| UI scale clamp | 0.55 to 1.15 | RoyalUI.lua:123 |
| Announcement hold | 2.2 s short, 3.4 s normal | RoyalUI.lua:498 |
| Crown flight time | 1.15 s | RoyalUI.lua:1171 |
| Throne camera look | 0.8 s in, 2.2 s hold, 0.6 s out | RoyalUI.lua:1210-1212 |
| "Near arena" for big text | 220 studs | RoyalUI.lua:1114 |
| Spinner range | 320 studs | RoyalUI.lua:1252 |
| Slam shake range | 30 studs | RoyalUI.lua:1429 |
| Refresh loop | 0.5 s | RoyalUI.lua:1531 |

### King aura (GardenConfig.lua:1246-1250)

KingCrownWidth 2.0, KingHoverHeight 1.1, KingBob 0.18, KingBobSpeed 2.2, KingSpinSpeed 0.9. These only matter for the "crown" kind, which is now skipped (KingClient.lua:80).

### Sea, currents, air, fall (GardenConfig)

| Key | Value | Where |
|---|---|---|
| CurrentSpeed | 42 studs/s | GardenConfig.lua:1310 |
| CurrentAirAccel | 20 studs/s^2 | GardenConfig.lua:1311 |
| ReturnSpeed | 48 studs/s | GardenConfig.lua:1319 |
| ReturnAirAccel | 160 studs/s^2 | GardenConfig.lua:1324 |
| DrownSweptSpeed | 30 studs/s | GardenConfig.lua:1291 |
| DrownGroundProbe | 12 studs | GardenConfig.lua:1326 |
| DrownFadeStart / DrownFadeFull / DrownKill | 2600 / 2775 / 2825 | GardenConfig.lua:1556-1558 |
| DrownTick | 0.2 s | GardenConfig.lua:1559 |
| AltitudeFadeStart / Full / Kill | 220 / 405 / 418 | GardenConfig.lua:1348-1350 |
| SkyBiomeY / SkyBiomeFull | 405 / 440 | GardenConfig.lua:1443-1444 |
| ForestBiomeStart / Full | 650 / 990 | GardenConfig.lua:1461-1462 |
| FallKillDrop / FallSettleSpeed / FallHoldTime | 220 / 12 / 2.0 s | GardenConfig.lua:1364, 1371, 1384 |
| SkyJumpY | 430 | GardenConfig.lua:1525 |
| CloudStillSpeed / CloudHoldTime / CloudSinkTime | 2 / 3.0 s / 4.0 s | GardenConfig.lua:1538-1541 |
| DayLengthSeconds / NightSpeedup | 600 / 10 | GardenConfig.lua:1480, 1484 |

### Music (GardenConfig.lua:798-879)

| Key | Value |
|---|---|
| MusicTracks | 4 ids |
| MusicVolume / MusicGap | 0.35 / 1.5 s |
| SkyMusicVolume / ForestMusicVolume / RoyalMusicVolume / FightMusicVolume | 0.38 / 0.38 / 0.30 / 0.45 |
| FightMusicRange | 140 studs |
| MusicDuckFloor / MusicDuckCurve / MusicPitchFloor | 0.08 / 1.6 / 0.72 |
| MusicMuffleHigh / MusicMuffleMid | -22 dB / -7 dB |

### Potions that touch this subsystem (GardenConfig)

| Key | Value | Where |
|---|---|---|
| AetherPotion price / duration | 10000 / 1800 s | GardenConfig.lua:1655-1660 |
| FeatherFallingPotion price / duration | 5000 / 600 s | GardenConfig.lua:1685-1691 |
| Effects.FeatherFalling fallSpeed / featherFallSpeed / featherRate / landingPuff | 22 / 10 / 10 / 14 | GardenConfig.lua:1767-1776 |

### Garden loops

| What | Value | Where |
|---|---|---|
| SPAWN_TICK | 0.5 s | Upgrades.lua:292 |
| IncomeTick | 5 s | GardenConfig.lua:372 |
| LOCK_TICK | 0.15 s | Ambient.lua:490 |
| FriendBonus | +10% per friend | GardenConfig.lua:357 |

### SkyIslandLife local constants (SkyIslandLife.lua:39-44, 244, 407, 480, 551)

NEAR_RANGE 220, SWAY_RANGE 320, AMBIENCE_VOLUME 3, MAX_CRUMBS 90, MAX_PETALS 90, MAX_FLIES 40, CLOUD_COUNT 14.

## 6. Extension points for the expansion

**1. Rebirth-gated rarity.**
- Where: the King's Throne eligibility needs `MinDiscovered = 10` Index fruits (CrownConfig.lua:38). RoyalUI shows CHALLENGE only when `RoyalEligible` is true (RoyalUI.lua:425).
- Change: lower `MinDiscovered`, or base it on the unlocked tiers.
- Break risk: only five COMMON fruits sit at or below Apple (FruitIndex.lua:69-73). A 0-rebirth player could never reach 10 and could never challenge.
- Also: the "slight boost" should go through `cashMultiplier`. The income tick sends `rate / base` to the client (Ambient.lua:466, 477-478), so a boost applied anywhere else would not show on the client.

**2. Bats with knockback.**
- Where: today only the arena path pushes players. The server fires "knockback" to the victim (CrownArena.lua:510-514) and RoyalUI applies it on the client (RoyalUI.lua:1482-1490).
- Change: add a garden-side knockback that the victim's client applies the same way. RoyalUI quits when `CrownConfig.Enabled` is false (RoyalUI.lua:34), so do not depend on it.
- Break risk: a knock off a sky island into a drop of more than 220 studs kills (Ambient.lua:291-299).

**3. Explosives and artillery.**
- Where: the dome refusal for TNT lives in useTool, not in the lock sweep (Ambient.lua:486-489). The ring, burst and shake effects (RoyalUI.lua:1078-1137) are good patterns, but they are local functions inside RoyalUI.
- Change: put shared effect helpers in a module both can use.
- Break risk: PlayerVision blacks out a death only if the hazard was already above 0.5 (PlayerVision.lua:230-234). Keep explosion deaths out of that path.

**4. Underground base and ores.**
- Where: ore income would join the income tick (Ambient.lua:464-479). An underground music track would be one more `biomeTrack` (MusicPlayer.lua:101-111) with its own share in `applyVolume` (MusicPlayer.lua:142-153). Underground lighting must be folded into PlayerVision's single tint write (PlayerVision.lua:449-469).
- Break risk: a shaft with a drop of more than 220 studs kills on landing (Ambient.lua:291). A spot where the 12-stud ray hits nothing reads as "water" (PlayerVision.lua:144, Ambient.lua:370).
- Break risk: the Income payload is only `(multiplier, dt)` and the client prices fruit itself (Ambient.lua:473-478). Ore earnings would not appear on the client without a payload change.

**5. Forest pets.**
- Where: pet earnings join the income tick (Ambient.lua:464-479), with the same Income payload limit as ores. The forest is detected by distance 650-990 studs from the centre (PlayerVision.lua:330-335).
- Break risk: SkyIslandLife makes crumbs only for player characters (SkyIslandLife.lua:780-788), so pets on clouds leave no crumbs. This is harmless, but expect it.

**6. Boats, swimming fatigue, remove currents, Aqua Affinity.**
- Remove currents (place edit): delete or stop the `CurrentPlate` and `ReturnPlate` parts (GardenConfig.lua:1310, 1313-1319).
- Remove currents (code): delete PlayerVision's airborne pushes (PlayerVision.lua:266-297) and the "return" kind (PlayerVision.lua:149-151). Otherwise a player jumping in the sea is still pushed outward at up to 42 studs/s.
- Remove currents (config and comments): `ReturnSpeed`, `ReturnAirAccel`, `CurrentSpeed`, `CurrentAirAccel` and the `ReturnFloors` entry go. Ambient's drown comment (Ambient.lua:26-33) becomes wrong.
- Swimming: none exists today (see section 3). Fatigue fits the drown sweep, which already decides `InDeepWater` every 0.2 s (Ambient.lua:359-375). The pattern to copy is `CloudSink`: a character attribute that `applyWalkSpeed` reads (Ambient.lua:337-341).
- Aqua Affinity: copy FeatherFall for a client-side movement effect read off `AquaAffinityEndsAt` (FeatherFall.lua:25-28). Check it on the server with `effectActive(player, "AquaAffinity")` (Ambient.lua:281).
- Boats break risk: a boat faster than 30 studs/s makes its riders read as "water" in PlayerVision and as `InDeepWater` on the server (PlayerVision.lua:260-265, Ambient.lua:363-364). Raise `DrownSweptSpeed`, or exempt boat decks by name the way `ReturnFloors` is exempted.
- Forest music still works without the current, because it is distance-based (PlayerVision.lua:330-335).

**7. More potions.**
- Pattern: each effect is read off a `<Effect>EndsAt` player attribute against server time, with an arena exemption (FeatherFall.lua:25-28, KingClient.lua:179-183, RoyalUI.lua:972-975). Server visuals that everyone sees go in Ambient, the way the feathers do (Ambient.lua:77-116).
- Break risk: each new effect must repeat the `InArena` exemption, or it will leak into duels (FeatherFall.lua:27, Ambient.lua:281).

**8. Plot overhaul, studded logs and lily pads, border mountains.**
- New moving water must carry a Texture tagged "IslandWaterFlow" with `FlowU` / `FlowV`, or it will not animate (IslandWater.lua:20, 33-35, 43-44).
- GardenConfig says the lily pads were already replaced on 28 Sep 2026 by stones, driftwood logs and islets in Main Island.Shallows (GardenConfig.lua:1283-1284). Check in Studio which "lily pads" are meant.
- New border mountains need no code, as long as no part is named `CurrentPlate`, `Seabed` or `ReturnPlate` (PlayerVision.lua:141-152). The forest's invisible boundary wall at radius 1470 must stay (GardenConfig.lua:1268).
- Break risk: SkyIslandLife finds decor by name and colour: `CherryTree` plus pink parts, `Flag` with green cloth and a pole taller than 10, `NightLamp`, `Drift`, `Cloud` (SkyIslandLife.lua:120-122, 151-154, 179-195, 629). A studded re-skin that changes names or colours silently stops those animations.

**9. UI overhaul (no emoji, image kit).**
- Font: already FredokaOne in both scripts (RoyalUI.lua:43, 179; RoyalCombat.lua:36, 179). No other font is used here.
- Style: what differs from an image kit is built in code. That is the gradient `shade()`, the stroke `border()`, the lip `button()`, and text strokes of 2.5 to 5 (RoyalUI.lua:141-205, 242-289). The cleanest swap is to rewrite these kit functions so that every caller changes at once.
- Emoji sites in RoyalUI, by line:
  - 322 and 404: crown on the chip.
  - 341: crossed swords on CHALLENGE. 343: crown on KINGDOM. 345: eye on WATCH.
  - 419: warning sign on VULNERABLE. 420: shield on protection. 421: hourglass on reign time.
  - 427: crown on the Royal Court line. 449: scroll as the decree fallback.
  - 572: fire on Underdog. 582: crown before King's name. 583: crossed swords after challenger's name.
  - 614: crossed swords on JOIN. 616: eye on WATCH.
  - 620-621: crown and crossed swords in zone text. 634, 643, 644, 650, 657: crossed swords and crown in contest titles.
  - 675: crossed swords on the prompt title. 690: check mark on ACCEPT. 695: heavy X on DECLINE.
  - 704: crown on the trial card. 705: crossed swords on ANSWER THE CALL.
  - 717 and 811: crown on KINGDOM. 742: heavy X close button.
  - 816: scroll. 817: crossed swords. 823: crown. 824: bar chart. 826: castle.
  - 835: bar chart. 848, 852: crown. 858: scroll. 865: trophy. 876: trophy.
  - 880: check mark and white square for achievements. 888: crown. 902: check mark and lock for styles. 916: scroll.
  - 986 and 999: crown in the head title. 1369: crossed swords. 1400: trophy (VICTORY). 1466: trophy. 1475: crown (ROYAL MODE).
- Emoji sites in RoyalCombat: touch buttons at 259 (crossed swords), 260 (shield), 261 (dash), 262 (collision), 263 (crown), 264 (shield), and the ability label at 343 (water wave or collision).
- Non-emoji, non-ASCII punctuation (middle dot, em dash, multiplication sign): RoyalUI.lua:427, 455, 458, 559, 584, 635, 664-665, 814, 849-850, 862, 869-871; RoyalCombat.lua:197, 264, 335-336. Check that the kit font draws these.
- Emoji also come from data, not only from these scripts:
  - Most `C.Text` strings (CrownConfig.lua:417-477).
  - Decree `icon` fields (CrownConfig.lua:213, 222, 231, 240, 248), drawn at RoyalUI.lua:449 and 935.
  - Server-sent announcements, for example CrownArena.lua:1039.
- Overlap: the top-left stack is 300x300 at (14, 6) (RoyalUI.lua:299-305). The decree card was already moved out of it once because it covered the Inventory button (RoyalUI.lua:348-351).

**10. Royal combat: real animations, slashes, health drain, better slam, weapon skins.**
- Swing: today it is the default "toolanim" Slash and Lunge for players (CrownArena.lua:229-238), and client FX are a sound only (RoyalUI.lua:1417-1421).
  - New slash VFX hook: `handlers.swing`, which already gets the attacker id and the heavy flag.
  - New animations: load them on the fighter's Animator in place of the toolanim StringValue inside `animate()` (CrownArena.lua:229-243). The local start could be in `attackDown` / `attackUp` (RoyalCombat.lua:65-84).
  - Knights' animation table: CrownArena.lua:323.
- Slam: `handlers.slamStart` is empty (RoyalUI.lua:1432). That is the windup hook, with 0.28 s to fill (CrownConfig.lua:130). The landing FX are in `handlers.slam` (RoyalUI.lua:1423-1431). The slam reuses the heavy strike's "Lunge" (CrownArena.lua:635), so it needs its own animation.
- Health drain:
  - Today there is no health. The duel board shows score and stagger only (RoyalUI.lua:554-592), and fighter tags show stagger % (RoyalUI.lua:1031-1033).
  - A health value would need a new server-written attribute, plus a bar in `refreshBoard` and in the fighter tags.
  - Break risk: knights are built with infinite MaxHealth (CrownArena.lua:410-411).
- Weapon skins:
  - The sword mesh is hard-coded in `makeSword` (CrownArena.lua:172-195).
  - A picker fits between "duelIntro" and "countdown" (RoyalUI.lua:1365-1376). That window is `DuelIntroTime` 3.5 s plus a 3-step countdown (CrownConfig.lua:72-73).
  - A King answering a challenge could pick next to ACCEPT (RoyalUI.lua:690-694).

## 7. Gotchas

- **Two KingUserId copies.** KingClient reads `workspace.KingUserId` (KingClient.lua:199). RoyalUI reads `RoyalState.KingUserId` (RoyalUI.lua:396). `publishCrown` writes both (Crown.lua:691-696). Progression also writes the workspace copy, but only when the throne module is absent (Progression.lua:50-53, 71).
- **Stale KingClient header.** It still says the crown goes to the richest player (KingClient.lua:2-3). The crown branch in `buildVisual` is dead code because of the early return (KingClient.lua:80, 87-107).
- **Two separate switches.** RoyalUI and RoyalCombat use `CrownConfig.Enabled`. KingClient uses `GardenConfig.KingEnabled` (RoyalUI.lua:34, RoyalCombat.lua:27, KingClient.lua:33).
- **Emoji stripping is narrow.** `dropEmoji` strips only a leading emoji, and only when that icon id is above 0 (RoyalUI.lua:222-224, 272). Trailing emoji such as the crossed swords after a name are never stripped (RoyalUI.lua:583). Crown styles pass `keepEmoji = true` (RoyalUI.lua:903).
- **Announcement dedupe.** It compares only with the last queued card, not the one on screen (RoyalUI.lua:512-513).
- **The refresh loop runs every 0.5 s.** Its comment says once a second (RoyalUI.lua:1520, 1531).
- **Cooldowns live on both sides.** Client and server keep separate cooldowns from the same CrownConfig numbers (RoyalCombat.lua:53-59, CrownArena.lua:570-596). Change the config, not one side.
- **Knockback must be applied on the client.** The server pushes a player by messaging their client (CrownArena.lua:507-514, RoyalUI.lua:1482-1490). A purely server-side push on a player's character will lag or fail.
- **Single owners, by design.**
  - PlayerVision is the only writer of its blur and tint and of Lighting ClockTime, Brightness and ambient colours (PlayerVision.lua:5-6, 402-418).
  - MusicPlayer is the only writer of the music volumes (MusicPlayer.lua:39-51, 61-63).
  - Add new looks and new tracks inside these scripts. A second writer will fight them every frame.
- **Music can switch everything off.** If `Config.MusicTracks` is empty, MusicPlayer returns before it creates the sky, forest, royal and fight tracks (MusicPlayer.lua:28-29, 79-111). A GardenConfig comment says three of the game's four music ids were dead when it was written (GardenConfig.lua:1795-1796). MusicPlayer skips dead ids with a warning (MusicPlayer.lua:293-300).
- **The Ocean's transparency is owned by PlayerVision.** It sets the Ocean to opaque in the sky biome and fades OceanGlow at night (PlayerVision.lua:478-492). A Studio edit to those parts' transparency is overwritten on each client.
- **Feather Falling has two halves.** The client caps the fall (FeatherFall.lua:32-41). The server forgets the fall height (Ambient.lua:281-284). Both must keep the same arena exemption.
- **The fall kill is server-measured.** Any drop of more than 220 studs kills, measured by a server ray (Ambient.lua:260-301). The arena already has to anchor knocked-out fighters because of this (CrownArena.lua:246-249).
- **Stale comment in Ambient.** It mentions `StarterPlayerScripts.DrownFade` (Ambient.lua:35-36). No such script is in the export. The fade is done by PlayerVision.
- **Late-arriving world.** PlayerVision waits up to 120 s for sky models, because a FindFirstChild once found nothing and made the island read as thin air (PlayerVision.lua:123-135). Keep using WaitForChild for any new world model a client script needs.
- **Spinners are collected after 4 s.** RoyalUI collects them then, and again on new `Spin` models (RoyalUI.lua:1235-1249). A new spinning prop must carry the `Spin` attribute on a Model, not on a Part.
- **onPlayerLeaving must take exactly one parameter.** Ambient asserts this at startup. The extra `exitReason` argument once broke saving (Ambient.lua:392-401).
- **The Income payload cannot carry new sources.** It is only `(multiplier, dt)`, and the client re-prices the fruit itself (Ambient.lua:473-478). Any income source that is not a fruit on the plot is invisible to the client unless the payload changes.
