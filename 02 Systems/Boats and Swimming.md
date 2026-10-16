---
tags: [system, design]
date: 2026-10-04
status: proposed
depends_on: [Rarity Gating and Rebirth, Weapons and Artillery, Potions, Plot Overhaul and Environment]
---
# Boats and Swimming

Source: [[Brain Dump 2026-10-04]]. Research: [[Swimming Fatigue and Boats in Roblox]], [[Sky Rivals - Weapons and Artillery]], [[Roblox Platform Constraints for the Expansion]]. Economy: [[Stage Model]]. Code map: [[Client World, Royal Client, Music and Ambient]], [[Core Server - Plots, Spawning, Income]], [[Carrying, Merging, Storage, Rebirth, Tools and Saving]], [[Client UI - HUD and Menus]], [[World Builders and Dev Tools]].

**How to read this note.** Everything in Rules, Numbers, UI, World and Code plan is a **proposal** unless it quotes Samarth or cites live code. Code citations are `File.lua:LINE` in `~/fruit-empire/src-export-2026-10-04`. Short names drop the prefix: `GardenService.lua` is `ServerScriptService.GardenService.lua`, `Ambient.lua` is `ServerScriptService.GardenService.Ambient.lua`, `PlayerVision.lua` is `StarterPlayer.StarterPlayerScripts.PlayerVision.lua`, `GardenConfig.lua` is `ReplicatedStorage.GardenConfig.lua`, `GardenStore.lua` is `ServerScriptService.GardenStore.lua`. Map facts from the project log are cited as `NOTES.md:LINE` (`~/fruit-empire/NOTES.md`).

**Words used here**
- **Sea**: the water ring between the island beach (radius r 380 from the map centre) and the forest's sand shelf (r 995). Radius means distance from the map centre in studs. A stud is Roblox's distance unit.
- **Bearing**: the compass angle of a point, `atan2(x, z)` in degrees, the same convention the shallows builder uses (`ServerStorage.ShallowsBuilder.lua:215`). A point at bearing b and radius r is `x = r sin b, z = r cos b`.
- **Current**: the invisible conveyor parts that carry players out to the forest (`CurrentPlate`) and back (`ReturnPlate`) today.
- **Stamina**: a meter, counted in seconds of swimming, that drains in the water and refills on dry ground.
- **Dry ground**: anything a player can stand on that is not the sea floor: the island, a stepping stone, driftwood, an islet, a dock, the forest beach.
- **Wash-up**: what happens when stamina runs out and the player does not reach dry ground in time. The screen dims, and the player wakes on the last dry ground they stood on. Nobody dies and nothing is lost.
- **Dock**: a studded wooden pier where boats are summoned. **Pad**: the patch of water next to a dock where a summoned boat appears.
- **Channel**: a clear lane through the stepping-stone field. The six old return lanes already are clear lanes (`NOTES.md:1572-1581`).
- **Rest islet**: a small rocky island with a tree, placed halfway across the open water so swimmers can rest.
- **HP (hit points)**: how much damage a boat takes before it breaks.
- **Network ownership**: which computer runs the physics for a moving object. The server, or one player's device ([[Roblox Platform Constraints for the Expansion]] section 7).
- **Assembly**: parts welded together that physics moves as one object.
- **Collision group**: a named set of parts. You choose which groups touch which.
- **Stage**: the newest rarity a player has unlocked, 1 (Common) to 9 (Transcendent). Stage = rebirths + 1 ([[Stage Model]] rule 1).

---

Section numbers follow the spec template, where section 1 is the frontmatter above.

## 2. Why

Samarth's words (verbatim, [[Brain Dump 2026-10-04]]):

> "also remove the currents that go to the place and back, instead make boats that a player can buy with shards. otherwise they have to swim through it, swimming fatigues the player, to swim super fast they can drink an aqua affinity potion. otherwise they can buy a boat for 1000 shards. boats can fit up to two people and can also be broken by the artillery i mentioned earlier like weopons - look at the game called sky rivals to get ideas for the artillery you can create."

> "in the forest we will add pets, merging two of the same pets makes the pet bigger and makes you more money"

> "make this super planned and tell me exactly what assets you need dont make slop unless it looks clean and studded."

> "also all Uis you make should follow the UI that we created, if you need any UI stuff then we can create them using chatgpt or gemini image generation following references of current UI"

> "all the UI should never overlap and also everything should be super clean."

> "I dont like the emoji-slop that you pushed on the sky island kings feature."

> "the ores merge and the animals merge should both be reasonable in the amount of moeny they give meaning they should match what stage of the game a player is in"

And from the potions follow-up: "dont overdo anything".

What the system is for. Today the sea is a free ride: step off the beach and a conveyor drops you on the forest beach in about 15 seconds, and six return lanes bring you back (`NOTES.md:1566`, `NOTES.md:1587`). This system removes that ride. Crossing the sea becomes a choice with a cost: swim for free and get tired, drink an Aqua Affinity potion and swim fast, or buy a 1,000-shard boat that seats two and can be broken by artillery. The forest, where pets will be tamed, is the main reason to cross.

---

## 3. Player view

A short walkthrough, in order.

1. **A new player wants to reach the forest.** They walk off the island beach into the sea. A small bar labelled "Stamina" appears at the bottom centre of the screen, above the carry buttons and the green message lane. Their character sinks to chest depth and swims at 12 studs per second, slower than walking (20). A green tip appears once: "Swimming tires you out. Rest on stones and islets."
2. **They hop between stepping stones.** On each stone the bar refills after a short pause. Out past the stones they see a ring of small rock islets with trees, halfway to the forest. They swim to one, rest until the bar is full, then swim on to the forest beach.
3. **If they push too far,** the bar turns red and reads "Tired", and they slow to 8. If it empties, it reads "Exhausted", they slow to a crawl, the screen dims and a heartbeat plays. If they reach a stone, islet, dock or beach within 4 seconds, they recover. If not, the screen goes dark for a moment and they wake on the last dry ground they stood on, with a full bar. They keep whatever they were carrying, and their potions.
4. **Around stage 2 they have about 1,000 shards.** A one-time green notice says they can buy a boat at a Boat Dock. There are three docks on the island and three on the forest beach. The main island dock has a studded shop booth (the Stud Shop pack) and a display boat.
5. **They press E on the dock post ("Buy Boat").** The Boat Dock panel opens: one card showing the Stud Skiff, four plain facts, and a Buy button showing 1000. They buy it. It is theirs for good, and rebirth never takes it.
6. **The dock post now says "Summon Boat".** They press E. The boat splashes onto the pad beside the pier and they are seated at the wheel. The bottom-centre bar now reads "Boat" with a full green bar.
7. **They drive.** W and S (or the thumbstick up and down) for throttle, A and D (or the thumbstick sideways) to steer. Top speed is 44 studs per second, about the same as the old current. Red and white studded buoys mark the clear channels through the stones.
8. **A friend on the pier sees "Board" (F) on the passenger seat** and hops in. Only the owner's Roblox friends can board.
9. **At a forest dock they press Space (or Jump) to leave.** They are placed on the pier. The empty boat is put away 90 seconds later. They can summon it again at any dock 15 seconds after the last summon.
10. **(Phase 2, not in v1.)** Sea Crates are off at launch (section J). The Golden Crate from [[Bats]] is the one crate players learn first.
11. **If another player's artillery hits the boat,** the bar drops, the boat slows, and at low HP it trails smoke. At 0 HP it breaks and sinks. The riders drop into the water with a full stamina bar. A red line says "Your boat was wrecked. Ready again in 60 s." Nothing they carried is lost.
12. **The Aqua Affinity potion** is in the Shop's Potions tab. Drinking it gives 10 minutes of swimming at 30 studs per second (2.5 times the normal swim) with no tiredness. Its timer shows in the effect block with every other potion.

---

## 4. Rules

Numbered so tests can point at them. Every value is in section 5. The measured map facts the rules rely on are in table 5.1.

### A. The currents are removed
- **R1.** No part in the place pushes players across the sea. The 24 `CurrentPlate` parts in `Main Island.AnimatedWater` (`NOTES.md:140`) and the six lanes' `ReturnPlate` and `ReturnRamp` parts (`NOTES.md:1572-1581`) are moved to `ServerStorage.Currents_backup_2026-10-04` (moved, not deleted).
- **R2.** No script pushes an airborne player over the sea. The two airborne pushes in `PlayerVision.lua:272-297` are deleted, and so is the "return" ground kind (`PlayerVision.lua:149-151`).
- **R3.** Nothing in the water looks like it flows outward. The lanes' turquoise `LaneWater` strips and `Foam` edges (scrolling at FlowV 48 and 34.6, `NOTES.md:1576-1579`) go to the same backup. The 120 radial `CausticEdge` beams slow from TextureSpeed 1.5 to 0.4 (`NOTES.md:141`), with the old value kept in an `OrigTextureSpeed` attribute.
- **R4.** The six forest-end `Gate` models keep their two rocks. Their "Current to the island" labels (`NOTES.md:1579`) go to the backup.

### B. Swimming
- **R5.** A player is **in the water** when the server's own 12-stud downward ray from the body's root lands on `Seabed` (or on nothing), the player is not standing on a sky island, and the player is not sitting in a boat seat. Any other part or Terrain under the ray is dry ground. The server decides this every `DrownTick` (0.2 s), reusing the ray it already casts for falls (`Ambient.lua:273-274`), and keeps the last answer while the player is airborne. It no longer trusts the client's `FloorMaterial` for this (the comment at `Ambient.lua:260-266` explains why that is unreliable).
- **R6.** Stamina is counted in seconds. Max 30. In the water it drains 1.0 per second while moving and 0.5 per second while still. "Moving" means horizontal speed above 1 stud/s.
- **R7.** On dry ground, stamina starts refilling 1.5 s after the player left the water, at 7.5 per second. Empty to full takes 4 s after the pause.
- **R8.** Swim state and speed: **Swim** (stamina above 7.5): 12 studs/s. **Tired** (0 < stamina <= 7.5): 8 studs/s. **Exhausted** (stamina 0): 4 studs/s. **Dry**: normal walking rules.
- **R9.** In the water the Speed potion is ignored. Only the swim speeds in R8 and Aqua Affinity (R13) apply.
- **R10.** While in the water the body sits lower: `HipHeight` drops so the root sits at about y -2.4, just under the ocean surface at y -2.2 (`NOTES.md:90`). Today a wading root sits at y -0.79 (`NOTES.md:150`). The client plays Roblox's own default swim animation (moving) and swim-idle animation (still), read from the character's `Animate` script.
- **R11.** Jumping works in the water at normal jump power, so players can climb onto stones and dock steps. Stamina keeps draining while airborne over the water.
- **R12.** **Exhaustion and wash-up.** When stamina reaches 0 the state is Exhausted. If the player is still in the water 4.0 s later, the server stamps `WashUpAt`, waits 0.6 s (the client fades to black over 0.5 s), then moves the character to its **last dry position** plus 3 studs up, with zero velocity, and sets stamina to 30. The last dry position is the root position from the most recent server tick where the player was supported (the fall ray at `Ambient.lua:273-274` hit something and vertical speed was under 12), moving slower than 30 studs/s sideways, standing on an anchored part or Terrain that is not `Seabed`, and not seated. A boat deck never counts. With no last dry position, the player goes to their plot's spawn point (`plot.spawnCF`, `GardenService.lua:575-578`).
- **R13.** Wash-up never kills, never clears potions and never drops the carried item. The carried fruit or tool is welded to the character and moves with it.
- **R14.** Nobody can die in the sea. The old drown kill stays where it is today, at r 2825, outside the forest's invisible wall at r 1470 (`GardenConfig.lua:1558`, `NOTES.md:1527`), so no swimmer can reach it.
- **R15.** The first time in each session that a player enters the water, they get one green notice: "Swimming tires you out. Rest on stones and islets."
- **R16.** Swimming exists only in the sea. Island ponds are not `Seabed`, so they read as dry ground (same ray rule as R5).

### C. Aqua Affinity potion (coordinated with [[Potions]])
- **R17.** Aqua Affinity is a timed potion in the existing potion system: a `Config.Tools.AquaAffinityPotion` entry of kind `"consume"` and a `Config.Effects.AquaAffinity` entry (`GardenConfig.lua:1615`, `:1714`). Drinking, stacking, expiry and clearing on death all work through the existing code (`Tools.lua:186-242`, `GardenService.lua:554-570`, `:793-821`).
- **R18.** While it runs and the player is in the water: speed is 12 x `swimMultiplier` 2.5 = 30 studs/s, stamina does not drain, stamina refills at 7.5 per second with no pause, and the Tired and Exhausted states never apply. ([[Potions]] owns the multiplier and now uses 2.5, not 2.0; decision D-029.)
- **R19.** Price and duration belong to [[Potions]]: 600 s, priced `min(Config.ShardPrice(25, rebirths), 500)`, which is 75, 180, 380, then 500 shards from stage 4 on (half a boat at most). The earlier flat 250 is withdrawn (D-029).

### D. Owning a boat
- **R20.** One boat type in v1: the **Stud Skiff**. Price: 1,000 shards, as Samarth set. Bought once, kept forever, and kept through rebirth (D-009 in [[Decisions Log]] already lists the boat as kept).
- **R21.** It can only be bought at a dock: the player must be within 25 studs of a dock post. The server refuses if the profile is not loaded, the boat is already owned, or the player has fewer than 1,000 shards.
- **R22.** The first time in each session that a player who does not own a boat reaches 1,000 shards or more, they get one green notice: "You have enough shards for a boat. Visit a Boat Dock."
- **R23.** The boat's trim, sail stripe and flag use the rarity colour of the owner's stage (`Config.RarityColors`, `GardenConfig.lua:582`). That is the same colour as the owner's bat ([[Bats]]). It updates when the owner rebirths.

### E. Docks and summoning
- **R24.** There are six docks: island docks at bearings 76°, 200° and 325°, and forest docks at the same three bearings. These three bearings are three of the six old lanes, already clear of stones (`NOTES.md:1572-1581`). The 200° island dock is the main dock, with the Stud Shop booth and a display boat. The other three lanes (19°, 138°, 255°) stay clear as extra channels with no dock.
- **R25.** All six channels are widened from 20 to 30 studs: shallows pieces whose bounding box enters the 30-stud strip are moved to `ServerStorage.BoatChannels_cleared_2026-10-04` with a `ClearedForChannel` attribute, as was done for the lanes (`NOTES.md:1581`).
- **R26.** Each dock has two pads in a row along the channel, one dock post with a prompt, a sign, and two swim steps so swimmers can climb out.
- **R27.** Summoning is done only at a dock post (prompt "Summon Boat", owners only). There is no summoning at sea.
- **R28.** The server refuses a summon if: the player does not own a boat; `BoatReadyAt` is in the future (cooldown or wreck rebuild); the player is seated in any boat; both of that dock's pads hold a boat; or 16 boats already exist in the server. Each refusal sends a red line from section 6.
- **R29.** A summon first destroys the player's old boat, if any. Then it places a new one on the first free pad, facing away from the shore, with the hull centre at y -2.0. It restores the boat's remembered HP (R46) and seats the owner in the driver's seat. It sets `BoatReadyAt` to now + 15 s.
- **R30.** A summoned boat cannot be damaged for its first 15 s (spawn protection).

### F. Riding and driving
- **R31.** Two seats. The driver seat is a `VehicleSeat` that only the owner may use. The passenger seat is a `Seat` that only the owner's Roblox friends may use. Both seats have `CanTouch` off, so nobody sits by bumping into them. Only the server seats people, through prompts.
- **R32.** Prompts on a boat: "Drive" (E) on the driver seat, shown only to the owner. "Board" (F) on the passenger seat, shown only to the owner's friends, only while that seat is empty and the boat is slower than 4 studs/s. If anyone else ends up in a seat, the server unseats them at once.
- **R33.** Players may board, drive and ride while carrying a fruit or a tool. The carry row works as it does on land, and Drop is still refused off your plot (`Carrying.lua:325-328`).
- **R34.** Leaving: Jump (Space, the mobile jump button, or the gamepad A button) leaves the seat. If the boat is within 30 studs of a pad, the server places the player on that dock's exit point on the pier. Otherwise they drop into the water and R5 to R16 take over.
- **R35.** Controls come from the `VehicleSeat`'s `ThrottleFloat` and `SteerFloat` ([[Swimming Fatigue and Boats in Roblox]] section 2). Keyboard W/S/A/D and arrows, the gamepad left stick, and the mobile thumbstick all feed these through Roblox's default vehicle controls. The `VehicleSeat`'s own speed display is off (`HeadsUpDisplay = false`).
- **R36.** Top speed 44 studs/s forward, 10 in reverse. Acceleration 18 studs/s², braking 30 studs/s². Turn rate scales from 40°/s when stopped to 75°/s at top speed.
- **R37.** Damage slows the boat: top speed = 44 x (0.6 + 0.4 x HP / MaxHP). At 25% HP that is 30.8.

### G. Physics and authority
- **R38.** A boat is one unanchored assembly. All parts are welded to one invisible `Collider` part, the only part that collides. It is held at the waterline by an `AlignPosition` that pushes on the vertical axis only, kept upright and turned by an `AlignOrientation`, and pushed by a `LinearVelocity` in Plane mode ([[Swimming Fatigue and Boats in Roblox]] proposal 11). Boats never float by real buoyancy, because the sea is parts, not Terrain water.
- **R39.** **Network ownership.** While the owner sits in the driver seat, the owner's device runs the boat's physics (`SetNetworkOwner(owner)`), the standard Roblox vehicle setup ([[Roblox Platform Constraints for the Expansion]] section 7). With no driver, the server runs it (`SetNetworkOwner(nil)`) and holds it still. It is not set to automatic, so a passenger or a nearby player can never become the owner. When the driver leaves, the server resets the movers: speed target 0, and the turn target set to the boat's current heading so it does not swing back.
- **R40.** Collision groups: `Boats` (each boat's `Collider`) and `Characters` (every character part). Boats do not collide with Characters, so swimmers pass under or through boats and nobody gets flung. Boats do not collide with other boats, so nobody can ram or pin another boat. Boats do collide with everything else: stones, islets, docks, beaches, buoys.
- **R41.** The server checks every boat every 0.2 s. A check fails if: horizontal speed is over 66 studs/s; the boat moved more than 66 x dt + 4 studs since the last check; the `Collider` centre is outside y -3.0 to -1.0; the boat is outside r 372 to 1004; or it is tilted more than 30°. On a failure the server takes ownership, moves the boat back to its last good position with zero velocity, and gives ownership back 0.5 s later. Three failures within 30 s despawn the boat and set `BoatReadyAt` to now + 60 s.
- **R42.** Artillery never pushes a boat physically. It only lowers HP through the server function in R44. A push on a boat the driver owns would not work from the server and would trip R41.

### H. HP, damage, wreck and repair
- **R43.** MaxHP is 1,000 for every boat. HP is a server value shown on the boat as the `HP` attribute.
- **R44.** Only the server changes HP, through `G.damageBoat(boat, amount, attacker, source)` (one name for the whole expansion; this note used `G.boatDamage` before the [[Integration Review]], conflict C10), which [[Weapons and Artillery]] calls after it resolves a hit on the server. It does nothing if: the boat is spawn-protected (R30); the boat is within 40 studs of any pad (a dock safe zone); the attacker is the owner or the passenger; the attacker or the boat's driver has not raised the War Flag (`G.warFlagged`, [[Weapons and Artillery]] rule 22; the PvE Rotten Fort skips this check); or the same attacker and source hit the same boat in the last 0.1 s (no double counting).
- **R45.** Splash damage uses the one shared falloff, `Ballistics.falloff`: `f(d) = 1 - 0.7 x min(d / r, 1)`, so 100% at the centre and 30% at the edge. [[Weapons and Artillery]] applies it before calling R44 (D-024).
- **R46.** Repair. A boat that has taken no damage for 8 s regains 20 HP per second (2% per second). Inside a dock safe zone it regains 100 HP per second. HP is remembered per player for the session, even when the boat is put away, and summoning restores the remembered value. The server works out the regen from the time since the last hit, so a boat that is put away still heals.
- **R47.** Looks of damage (client only, read from `HP`): at 66% HP or less, the sail darkens; at 33% or less, a light smoke emitter turns on at the stern and the left trim turns charred grey.
- **R48.** **Wreck at 0 HP.** The server sets `Broken = true`, anchors the boat where it is, turns all its collisions off, and unseats both riders by removing the seat welds. Riders drop into the water with stamina set to 30. Each client plays the sink locally: the boat tilts 20° and drops 4 studs over 2.0 s, with splash and splinter effects. The server destroys it at 2.5 s. The owner's `BoatReadyAt` becomes now + 60 s and their remembered HP becomes full again for the next summon.
- **R49.** A wreck takes nothing from anyone. No fruit, pets, shards or cash move. The carried item stays in the rider's hands. (Stealing was removed on 3 Oct 2026, so nothing in this system takes another player's things.)

### I. Despawning
- **R50.** A boat despawns when: the owner summons again; the owner leaves the server; nobody has been aboard for 90 s; it has had no driver for 30 s while sitting on a pad (pad hogging); or the owner dies, or is taken out of the driver seat by a teleport (a royal duel or contest sets `InArena`), while a passenger is aboard: then it despawns 10 s later. Only the owner can drive, so without this a passenger would be left in a boat that never moves and never despawns. (Added by the player-path pass, [[Combat Day]].)
- **R51.** On any despawn that is not a wreck, anyone still aboard is moved to the exit point of the nearest dock. Nobody is ever stranded at sea by a despawn.
- **R52.** A boat never survives its owner leaving. Nothing about a boat is saved except ownership.

### J. Sea Crates (Phase 2, off in v1; see open question Q8)
The [[Integration Review]] (conflict C30) turned Sea Crates off for launch: two crate systems would confuse players, and the cash part (30 s of income every 180 s) was a cash source outside the 35% pets-and-ores budget. If they ship later, R56 pays shards only, through `Config.ShardReward(2, rebirths)`.
- **R53.** Each player has at most one personal Sea Crate, floating in the open sea between r 680 and 960. It is at least 60 studs from any pad and at least 20 studs from any rest islet. Only that player's device draws it, so nobody can take another player's crate.
- **R54.** The first crate appears 60 s after the player joins. After a claim, the next one appears 180 s later at a new random spot.
- **R55.** Claiming: when the player's root comes within 10 studs of their crate, their device asks the server. The server pays only if the crate exists and the root is within 14 studs of it, or within 18 studs if the player is seated in a boat.
- **R56.** (As first proposed; if Sea Crates ship in Phase 2 the cash part is dropped and only the shards below are paid.) A claim pays cash equal to 30 s of the player's live income (`CashPerSecond`, written by the income tick at `Ambient.lua:469`), never less than 30 s of one first fruit of their stage times their cash multiplier. It also pays shards equal to twice the per-merge shard drop of their stage (D-010). Cash goes through `balances` and `publishCash` (`Progression.lua:80-88`), shards through `setShards` (`Progression.lua:216-218`), then `Store.Touch`.

### K. Destinations that make sailing worth it
- **R57.** **The forest ring is the main destination.** Pets are tamed there ([[Pets]]), and the forest already has 18 open clearings at r 1190 to 1380 set aside for animals (`NOTES.md:1522`). Each forest dock's pier starts at r 1012, 60 studs short of where the forest edge begins (r 1072, `ServerStorage.ForestBuilder.lua:19`).
- **R58.** **Speed is the reward.** Island to forest takes about 13 to 14 s by boat, against about 63 to 68 s swimming with rests (table 5.3). A round trip to tame one pet saves about 100 s, and a friend rides free (R31).
- **R59.** **The open sea pays through other systems in v1:** Golden Crates on the shallows islets ([[Bats]]), the Practice Range and, in Phase 3, the Rotten Fort ([[Weapons and Artillery]]). Sea Crates (section J) are a Phase 2 option.
- **R60.** Nothing is placed out at sea that only boats can reach. Swimmers can reach every destination, so a stage 1 player with no shards is never locked out of the forest ([[Swimming Fatigue and Boats in Roblox]], "What to avoid").

### L. Always true
- **R61.** No new UI uses emoji or non-ASCII text. Every text label uses FredokaOne, white, with a black stroke ([[UI Kit Inventory]]).
- **R62.** No new HUD element overlaps an existing one at any HUD scale (section 6 and test T-UI-1).

---

## 5. Numbers

### 5.1 Map facts the rules rely on (measured, not proposals)

| Fact | Value | Source |
|---|---|---|
| Ocean surface | y -2.2 | `NOTES.md:90`, `ShallowsBuilder.lua:109` |
| Seabed top | y -4.0 (water is about 1.8 studs deep, so players wade today) | [[Swimming Fatigue and Boats in Roblox]] section 5 |
| Island beach edge | r 380, top y -1.0 | `NOTES.md:140`, `ShallowsBuilder.lua:212` |
| Stepping stones, driftwood, islets | about 692 pieces, r 381 to 643.3, anchored, collidable | `NOTES.md:1440-1455`, `ShallowsBuilder.lua:7-9`, [[World Builders and Dev Tools]] |
| Forest wet sand shelf | r 995 to 1003, surface y -2 | `NOTES.md:1495` |
| Forest dry beach | from r 1003, surface y +2 | `NOTES.md:1496` |
| Forest invisible wall | r 1470 | `NOTES.md:1527` |
| Old lanes (clear of stones, 20 wide) | bearings 19, 76, 138, 200, 255, 325 | `NOTES.md:1572-1581` |
| Old current ride, island to forest | 14.8 s | `NOTES.md:1566` |
| Walk speed | 20 studs/s | `GardenConfig.lua:943` |
| Wading root height | y -0.79 | `NOTES.md:150` |

### 5.2 Swimming (not money, so not stage-scaled)

| Quantity | Value | Why |
|---|---|---|
| Max stamina | 30 s of moving swim | One bar covers one leg between rests (table 5.3) |
| Drain | 1.0/s moving, 0.5/s still | Floating in place still costs something |
| Refill on dry ground | 7.5/s after a 1.5 s pause (empty to full in 5.5 s) | Genshin's pause-then-fast-refill ([[Swimming Fatigue and Boats in Roblox]] section 1) |
| Tired threshold | 7.5 (25% of the bar) | |
| Speeds | Swim 12, Tired 8, Exhausted 4, Aqua Affinity 30 (walk is 20) | Aqua is 1.5x walking and 2.5x swimming |
| Distance on one full bar | 22.5 s x 12 + 7.5 s x 8 = **330 studs**, plus 4 s x 4 = 16 while exhausted | |
| Exhausted grace before wash-up | 4.0 s | Enough to reach a stone you are next to |
| Wash-up timing | 0.5 s fade to black, move at 0.6 s, 1.0 s fade back | |
| Aqua Affinity | 600 s, 30 studs/s (12 x 2.5), no drain | R18, R19 ([[Potions]] owns price and duration) |

### 5.3 Crossing the sea: legs, reach and travel times

Rest islets: 16 at r 820 (proposal, section 7). With 16 spread evenly, the arc between neighbours is 2 x pi x 820 / 16 = 322 studs, so a swimmer is never more than 161 studs to the side of one.

| Leg | Straight out | Worst case (angled) | Share of one bar |
|---|---|---|---|
| Outer stones (r 643) to rest islet (r 820) | 177 studs, 14.8 s | sqrt(177² + 161²) = 239 studs, 19.9 s | 49% to 66% |
| Rest islet to forest shelf (r 995) | 175 studs, 14.6 s | 175 studs | 49% |
| Outer stones straight to forest, no islet | 352 studs | 352 studs | more than one bar plus the crawl (346): you wash back. The islets are the route. |

| Trip, island beach to forest beach (615 studs) | Time | Cost |
|---|---|---|
| Old current (removed) | 14.8 s (measured) | free |
| Boat, island pad (r ~430) to forest pad (r ~950), 520 studs | about 2.4 s to reach 44, then 44 studs/s, about 13 to 14 s | 1,000 shards once |
| Aqua Affinity swim | 615 / 30 = 20.5 s, about 22 s with stone detours | 75 to 500 shards per 10 min, by stage ([[Potions]]) |
| Plain swim with two rests | about 22 s through the stones + 15 to 20 s + 15 s + two 5.5 s rests, about 63 to 68 s | free, tiring |

### 5.4 Boat constants (not money)

| Quantity | Value |
|---|---|
| Seats | 2 (driver = owner, passenger = owner's friend) |
| Top speed / reverse | 44 / 10 studs/s (between the old current's 42 and the old return lane's 48, `GardenConfig.lua:1310`, `:1319`) |
| Acceleration / braking | 18 / 30 studs/s² |
| Turn rate | 40°/s stopped to 75°/s at top speed |
| Speed at reduced HP | 44 x (0.6 + 0.4 x HP / MaxHP): 100% HP 44, 50% 35.2, 25% 30.8 |
| MaxHP | 1,000 |
| Regen | 20 HP/s after 8 s with no damage; 100 HP/s in a dock safe zone |
| Spawn protection | 15 s |
| Dock safe zone | 40 studs from any pad centre |
| Summon cooldown | 15 s |
| Rebuild after a wreck | 60 s |
| Despawn | 90 s with nobody aboard; 30 s with no driver while on a pad |
| Server cap | 16 boats |
| Validation | every 0.2 s; speed <= 66; jump <= 66 x dt + 4; Collider centre y -3.0 to -1.0; r 372 to 1004; tilt <= 30°; 3 strikes in 30 s = despawn |

### 5.5 Boat and Aqua Affinity prices against the Stage Model

Formula: minutes of shard income = price / shard income per minute in that stage. Shard income per minute is from the [[Integration Check]] re-run (attentive player, regular pet trips, ores; gross shards earned in the stage divided by its length). Shards held at the gate are the [[Stage Model]] medians.

| Stage | Rarity | Full plot $/s | Shard income / min | Shards held at the gate | Boat (1,000): % of gate holdings | Boat: minutes of shard income | Aqua Affinity price ([[Potions]]) | Aqua Affinity: minutes of shard income |
|---|---|---|---|---|---|---|---|---|
| **1** | Common | 707 | 15.2 | 232 | 431% | 66 | 75 | 4.9 |
| 2 | Uncommon | 430K | 36.8 | 1.5K | 65% | 27 | 180 | 4.9 |
| **3** | Rare | 3.7B | 89.6 | 6.6K | 15% | 11 | 380 | 4.2 |
| 4 | Epic | 3.6T | 142.5 | 12.8K | 7.8% | 7.0 | 500 | 3.5 |
| 5 | Legendary | 1.8Qa | 246 | 28.3K | 3.5% | 4.1 | 500 | 2.0 |
| **6** | Mythic | 2.5Qi | 577 | 86.2K | 1.2% | 1.7 | 500 | 0.87 |
| 7 | Astral | 265Qi | 758 | 111K | 0.9% | 1.3 | 500 | 0.66 |
| 8 | Eternal | 506Sx | 2,023 | 439K | 0.2% | 0.5 | 500 | 0.25 |
| **9** | Transcendent | 24.9Sp | about 5,500 | none (final stage) | - | 0.18 | 500 | 0.09 |

What this means:
- **The boat is a stage 2 purchase.** A stage 1 player holds about 232 shards at the gate, so they cannot afford it and must swim. That is fine: stage 1 lasts 15 minutes. In stage 2 the boat costs about 27 minutes of a 34-minute stage. The Stage Model already planned exactly this: its stage 2 spend is 1,000 shards, for the boat (`sim/final.py`, `SPEND = {2: 1000, ...}`).
- **From stage 3 on, the boat is cheap,** and from stage 6 on it costs under two minutes of shards. The price is Samarth's fixed number. It is a one-time unlock, so becoming cheap later breaks nothing.
- **Aqua Affinity is affordable from stage 1** (75 shards) and becomes a convenience from stage 4 on, where it stops at half a boat.

### 5.6 Sea Crate rewards against the Stage Model (Phase 2 only; off in v1, shards only if shipped)

Formulas:
- `crateCash = 30 x CashPerSecond` (live), with a floor of `30 x firstFruitIncome(stage) x rebirthMultiplier(stage)`.
- `crateShards = 2 x ShardDrop(stage)`, where ShardDrop is D-010's 3, 7, 15, 32, 70, 155, 340, 748, 1,646.
- At most one crate every 180 s, so sailing nonstop adds at most 30 / 180 = **16.7%** of live income on top of the plot, which keeps earning while you sail.

| Stage | Full plot $/s | Crate cash at the plateau (30 x full plot) | Crate cash floor (first fruit) | Crate shards | Crate shards as a share of that stage's shard income, at 1 crate / 3 min |
|---|---|---|---|---|---|
| **1** | 707 | 21.2K | 30 (Blueberry 1 $/s x 1.0) | 6 | 12.7% |
| 2 | 430K | 12.9M | 1.62K (Dandelion 43.2 x 1.25) | 14 | 11.9% |
| **3** | 3.7B | 112B | 976K (Bee Balm 21.7K x 1.5) | 30 | 9.7% |
| 4 | 3.6T | 109T | 8.56B | 64 | 12.4% |
| 5 | 1.8Qa | 53.5Qa | 8.4T | 140 | 14.8% |
| **6** | 2.5Qi | 74.7Qi | 4.14Qa (Passionfruit 61.4T x 2.25) | 310 | 13.5% |
| 7 | 265Qi | 7.95Sx | 5.86Qi | 680 | 24.6% |
| 8 | 506Sx | 15.2Sp | 628Qi | 1,496 | 19.2% |
| **9** | 24.9Sp | 747Sp | 1.22Sp (Alien Carrot 13.6Sx x 3.0) | 3,292 | about 19% (extrapolated) |

First-fruit incomes are from `sim/stage_reference.json` (`first_inc`). The rebirth multiplier is +25% per rebirth, added (D-008). Pets and ores aim for 35% of full-plot income at the plateau ([[Stage Model]] rule 6). Crates are capped well below that, and only while you are out at sea.

### 5.7 Artillery against boats (owned by [[Weapons and Artillery]]; copied here so both notes show the same numbers)

Damage is in boat HP. Every boat has 1,000, so these numbers do not depend on stage. They only apply when the shooter and the boat's driver both have the War Flag up (R44), or when the PvE Rotten Fort fires (Phase 3).

| Weapon | Boat damage at the centre | % of MaxHP | Hits to wreck from full (no regen) |
|---|---|---|---|
| Cherry Bomb | 100 | 10% | 10 |
| Melon Mortar | 350 | 35% | 3 |
| Coconut Cannon | 200 direct, 80 splash | 20% / 8% | 5 direct |
| Starfruit Barrage | 300 per shell, 5 shells | 30% | 4 shells |
| Rotten Fort shell (Phase 3, PvE) | 120 | 12% | 9 |
| Placed TNT ladder and Nuke | 0 (they only work standing on a plot, [[Explosives and Fruit Health]] rule 7) | - | - |

Falloff (R45): the shared `f(d) = 1 - 0.7 x min(d / r, 1)`. At the edge (d = r) that is 30% of the centre value. Regen pauses for 8 s after any hit, so steady fire wins. The earlier list in this section (Cherry Bomb 120, Melon Mortar 300, Seed Gatling, Golden Nuke) is withdrawn ([[Integration Review]], conflict C10).

---

## 6. UI

All of it is built from the kit in [[UI Kit Inventory]]. FredokaOne (`rbxasset://fonts/families/FredokaOne.json`), white text, black `UIStroke` 2.5 px, `TextScaled`. No emoji, no non-ASCII characters, no code-drawn rounded boxes.

### 6.1 Sea Status bar (new HUD cluster `GardenUI.SeaStatus`)

One slot that shows either stamina (while swimming) or boat HP (while seated in a boat). The two never show at the same time, because a seated player is never in the water (R5). It copies the look of `CarryInfo`, which is a name bar above a rate bar.

| Part | Kit image | Size at HUD scale 1 | Content |
|---|---|---|---|
| `SeaStatus` (frame) | none (transparent) | 218 x 74 | AnchorPoint (0.5, 1); its bottom edge is placed by HudScale with the rule below (authored at `UDim2.new(0.5, 0, 1, -194)` for the 1600 x 900 reference) |
| `SeaStatus.NameBar` | `CarryInfo.NameBar` 123470450245311 (native 168 x 27, shown at 1.3x) | 218 x 35, top of the frame | Text: "Stamina", "Boat", or "Boat (safe)" |
| `SeaStatus.Bar` (track) | the shared bar track `UIKit.Images.BarTrack` ([[King UI Overhaul]] A8, D-027); until it exists, `CarryInfo.RateBar` 120297790302429 with ImageColor3 (90, 90, 90) | 218 x 35, 4 px under NameBar | |
| `SeaStatus.Bar.Clip` | transparent Frame, `ClipsDescendants = true`, width = fill fraction | 0 to 218 x 35 | |
| `SeaStatus.Bar.Clip.Fill` | the shared bar fill `UIKit.Images.BarFill` (A9); until it exists, the same RateBar image, ImageColor3 = state colour | 218 x 35 | Tinted and clipped |
| `SeaStatus.Bar.Text` | none | full bar | "74%", "Tired", "Exhausted", "Wrecked" |

States and colours (exact `Config.RarityColors` values, `GardenConfig.lua:582-592`; the HP bands are the shared ones from D-027):

| State | Fill colour | Text | Shown when |
|---|---|---|---|
| Stamina, Swim | Rare blue (78, 164, 230) | percent | in the water, or refilling after leaving it; hidden 1 s after reaching full |
| Stamina, Tired | Mythic red (232, 66, 60), a slow pulse of Transparency 0 to 0.25 at 1 Hz | "Tired" | stamina 7.5 or less |
| Stamina, Exhausted | empty | "Exhausted" | stamina 0 |
| Boat, HP above 50% | Uncommon green (78, 194, 101) | percent | seated in any boat |
| Boat, HP 25% to 50% | Legendary orange (242, 171, 48) | percent | |
| Boat, HP under 25% | Mythic red | percent | |
| Boat, wrecked | empty | "Wrecked" | for 3 s after R48, then hidden |
| Boat (safe) | same as HP colour | percent | NameBar reads "Boat (safe)" during spawn protection or inside a dock safe zone |

How it avoids overlap:
- It sits in the bottom-centre column above the "hands" stack: `CarryButtons` (y -21), `CarryInfo` (y -87, two bars, top about -145) and, at sea, the [[Weapons and Artillery]] Weapon Bar (y -21 to -123). Players may carry while swimming or sailing (R33), so SeaStatus is stacked above them, never on top of them.
- **Placement rule (set in the [[Integration Review]], conflict C23).** The green notice toast is not scaled: it always lands at H - 178 to H - 150 (`GardenToast.lua:64`), and the [[King UI Overhaul]] keeps H - 186 to the bottom of the centre third clear for it. HudScale places SeaStatus in the first gap that fits, measured live: (1) between the top of the visible hands stack + 8 px and the notice lane's bottom (H - 150) - 8 px, if that gap is at least the bar's scaled height; otherwise (2) with its bottom edge at H - 186. At 1600 x 900 that is rule 2: y 640 to 714, x 691 to 909. On a 667 x 375 phone (scale 0.55) it is rule 1: y 246 to 287, x 273 to 393. Either way it never touches the hands stack, the notice toast or the Weapon Bar. `CarryInfo` never moves.
- It is added to `Config.HudClusters` (`GardenConfig.lua:1863-1867`) so HudScale scales and places it ([[Client UI - HUD and Menus]] gotchas 2 and 3). It has no buttons, so ButtonAnimator never touches it.
- It hides while any centre panel is open (Shop, Index, Inventory, Upgrades, Rebirth, Boat Dock).
- Bottom left (CornerHud), bottom right (EffectStack), the side button columns and the top-centre royal stack are all clear of it.

Sound with it (client): while Tired, a soft bubble tick every 1.0 s, with pitch rising from 1.0 to 1.5 as stamina falls from 7.5 to 0 ([[Swimming Fatigue and Boats in Roblox]] section 1, the Minecraft cue).

### 6.2 Exhaustion and wash-up look (inside PlayerVision)

PlayerVision is the only writer of the blur and tint (`PlayerVision.lua:5-6`, [[Client World, Royal Client, Music and Ambient]] gotchas). So exhaustion is one more term in its hazard value, not a new effect:
- While Exhausted, the hazard rises from 0 to 0.45 over the 4 s grace. It is capped at 0.45, under the 0.5 that latches a blackout on death (`PlayerVision.lua:233`). Music ducks and the heartbeat plays through the existing hazard path (`MusicPlayer.lua:120-194`, `PlayerVision.lua:497-504`).
- On a `WashUpAt` change: fade to black over 0.5 s, hold until the character has moved, then clear over 1.0 s. This uses the same pattern as the `FellHard` stamp (`PlayerVision.lua:209-211`).

### 6.3 Boat Dock panel (new, `GardenUI.BoatDockMenu`)

This panel is the shop for boats. It opens when a non-owner presses the dock post prompt ("Buy Boat") or anyone presses "Boat Shop" on the Stud Shop keeper. Owners use the prompt to summon and do not need the panel.

| Part | Kit component | Size (at the panel's own scale 1) | Notes |
|---|---|---|---|
| Title frame | `IndexMenu.TitleFrame` 110138832714236 | 620 x 48 | scaled from 948 x 74 |
| Title bar | `IndexMenu.TitleBlue` 95532041481396 | 612 x 41 | Fredoka text "BOAT DOCK" until the baked title image arrives (asset request 3) |
| Body | `IndexMenu.Body` 82827190351667 | 612 x 300 | |
| Inline | `IndexMenu.Inline` 110403403749726 | 606 x 310 | |
| Close | Red X `ShopMenu.Panel.CloseButton` 107111219044453 | 46 x 46 | top-right corner of the title frame |
| Boat card | Item card 99120915117089 | 216 x 237 | ViewportFrame of the Stud Skiff turning at 20°/s. Fredoka title "Stud Skiff" |
| Facts box | Info box `IndexMenu.InfoBox` 101725900357985 | 192 x 225 | Four Fredoka lines: "Seats 2: you and a friend", "Hull 1000", "Top speed 44", "Kept when you rebirth" |
| Buy button | Buy `…Card.Buy` 97067256017679 | 173 x 61 | Text "1000" (the shard crystal is baked into the art, [[Client UI - HUD and Menus]]). Owned: ImageColor3 grey (150, 150, 150), text "Owned", not clickable |

Behaviour:
- Centre screen. Same open and close pattern as the other panels: an `isOpen` flag and a UIScale tween from 0.85 to 1 (`ShopMenu.lua:604-633`). Uses a full-screen `Blocker` ImageButton marked `Animated` before parenting (`ClearMenu.lua:84-98`).
- Added to every panel's "hide the others" list: `ShopMenu.lua:616`, `IndexMenu.lua:47`, `StoredFruitsMenu.lua:541`, `UpgradeMenu.lua:421`, `RebirthMenu.lua:51`. Those five panels go in its own list too.
- Closes on X, or when the player walks more than 20 studs from the dock post.
- On a touch screen the Buy button must stay at least 44 px tall. Check at 812 x 375.

### 6.4 Prompts

ProximityPrompts use Roblox's default prompt style, the same style as the fruit prompts (`FruitFactory.lua:238-250`), so they match what players already see.

| Prompt | Where | Key (keyboard / gamepad) | Shown to | Text |
|---|---|---|---|---|
| Dock | each dock post | E / ButtonX | everyone; the client sets the text | "Summon Boat" if owned, else "Buy Boat" |
| Boat Shop | Stud Shop keeper, main dock | E / ButtonX | everyone | "Boat Shop" |
| Drive | boat driver seat | E / ButtonX | owner only (client hides it for others) | "Drive" |
| Board | boat passenger seat | F / ButtonY | owner's friends only, seat empty, boat under 4 studs/s | "Board", ObjectText = owner's name |

Board uses F so it never fights an E prompt near a dock. The default "one prompt per key" rule would otherwise hide one of them ([[Roblox Platform Constraints for the Expansion]] section 9). All prompts: hold time 0, `RequiresLineOfSight = false`, range 10 studs.

### 6.5 In the world

| Element | Kit image | Size | Rule |
|---|---|---|---|
| Boat name and HP (BillboardGui in the boat template, on `HPAnchor`) | Name: Fredoka text. HP: `CarryInfo.RateBar` 120297790302429 with the same clip-fill trick as 6.1 | 6 x 0.8 studs each, 4.5 studs above the deck | Drawn only within 120 studs, and only when HP is below max or the viewer is within 40 studs. `AlwaysOnTop = false` |
| Sea Crate marker (client-local, Phase 2 only) | Money chip `RewardsBox` money chip 82056408182031 | 2.5 x 2.6 studs, 3 studs above the crate | Only on the owner's device, within 300 studs |
| Dock sign | Clone of `Workspace["Stud Sign"]` (a 2-part sign with a SurfaceGui, [[Existing VFX and Props]]) | as authored | SurfaceGui text "BOAT DOCK", Fredoka, white, black stroke |
| Stud Shop keeper's name | none | - | Set the NPC's `Humanoid.DisplayDistanceType = None`, so Roblox's default name tag (not kit UI) never shows |

### 6.6 Effect stack and toasts

- Aqua Affinity gets an effect-block badge automatically from its `Config.Effects` entry (icon, bar colour) (`GardenClient.lua:549-557`). Bar colour and badge art belong to [[Potions]]: ocean blue (40, 130, 255) and its `aquaaffinity-badge.png` (D-029).
- Text lines use the existing channels: red `Notify` for problems, green `Notice` for good news (`GardenService.lua:66-70`). All are new `Config` keys (proposal wording, ASCII only):

| Key | Channel | Text |
|---|---|---|
| `SwimTip` | Notice | "Swimming tires you out. Rest on stones and islets." |
| `SwimWashedUp` | Notify | "Too tired! You washed back to safety." |
| `BoatAffordable` | Notice | "You have enough shards for a boat. Visit a Boat Dock." |
| `BoatBought` | Notice | "Boat bought! Summon it at any Boat Dock." |
| `BoatNotOwned` | Notify | "Buy a boat at the Boat Dock first." |
| `BoatWait` | Notify | "Your boat is ready in %d s." |
| `BoatPadBusy` | Notify | "This dock is busy. Try again in a moment." |
| `BoatServerFull` | Notify | "All boats are out. Try again soon." |
| `BoatInBoat` | Notify | "Leave this boat first." |
| `BoatTooFar` | Notify | "Stand at a Boat Dock to buy a boat." |
| `BoatFriendsOnly` | Notify | "Only %s's friends can ride this boat." |
| `BoatWrecked` | Notify | "Your boat was wrecked. Ready again in 60 s." |
| `BoatReturned` | Notify | "The boat was put away. You are back on the dock." |
| `SeaCrateClaimed` (Phase 2 only) | Notice | "Sea Crate: +%d shards" |
| `ToolNotEnoughShards` | Notify | existing: "Not enough shards!" (`GardenConfig.lua` Tools strings) |

---

## 7. World and assets

### 7.1 From packs and props already in the place

| Use | Source (by name) | Notes |
|---|---|---|
| Boat Shop booth and keeper at the 200° island dock | **Stud Shop** (store 12340318191, in `ServerStorage.AssetQuarantine.Q_12340318191`) | 30 parts, decor only, no scripts ([[Creator Store Packs]]). Our code drives the prompt. Anchor every part. |
| Pier decks | **Stud Bridge** (71814553242028) | Its studded planks set the plank size and colours for the piers and for the boat's deck. |
| Pier pilings, boat mast | **Studded Pack** `Logs` (86601833587701) | Square studded logs (D-003). Stood upright as pilings. The mast copies the look. |
| Landmark campfire at the 200° forest dock | **Studded Pack** `CampFire` | One only. Visible from the sea as a target. |
| Dock signs | `Workspace["Stud Sign"]` | Clone per dock. |
| Rest islets | `ShallowsBuilder` tree islets (`B.islet(rng, x, z, 12, true)`, `ShallowsBuilder.lua:148-196`), then `B.finish` (`:426`) | The same Asset Pack V1 rocks and plants as the stepping stones, so they match. Tree islets are 11 to 13 studs wide, with a tree that shows from far away. |
| Wake and bow splash, summon splash, wreck splash | `Anime` pack `Water-01`, `Water-02` (staging area, y about -790) | Cloned emitters on attachments. |
| Damaged-boat smoke | `Anime` pack `Smoke-01` | Low rate, on at 33% HP or less. |
| Wreck splinters | `vfx pack` `f` (rocks and impact emitters) tinted brown (124, 92, 70) | One burst. |
| Forest dock markers | The six `Gate` models' two Big Rocks each (`NOTES.md:1579`) | Labels removed (R4). |
| Aqua Affinity bottle | Built by [[Potions]] (a clone of `SpeedPotion`, liquid (40, 130, 255)) | Nothing to build here. |

### 7.2 To build (Claude, in Studio, during the build)

**Stud Skiff template** (`ServerStorage.Boats.StudSkiff`). Front is -Z. About 22 parts, one assembly, `ModelStreamingMode = Atomic` ([[Roblox Platform Constraints for the Expansion]] streaming advice). Blocky, studded, no curved parts (Samarth: "studded texture doesnt match with curves it just ends up looking like metal", `NOTES.md:1597`).

| # | Part | Size (X x Y x Z) | Look | Collides |
|---|---|---|---|---|
| 1 | `Collider` (PrimaryPart) | 6.6 x 2.4 x 15.6 | invisible | yes, group `Boats`, `CanQuery` true, `CanTouch` false, density set so the assembly weighs about 300 |
| 2 | HullBottom | 6 x 0.8 x 14 | dark wood (105, 64, 40), studs on top | no |
| 3-4 | SideL, SideR | 0.8 x 2.2 x 14 | dark wood, studs on top | no |
| 5 | Stern | 6 x 2.2 x 0.8 | dark wood | no |
| 6-7 | BowStep1, BowStep2 | 4.4 x 2.2 x 1.2 and 2.8 x 2.2 x 1.2 | dark wood, stepped bow, studs on top | no |
| 8-12 | Plank1 to Plank5 | 5.2 x 0.4 x 2.4 | alternating (124, 92, 70) and (160, 120, 80), studs on top (Stud Bridge style) | no |
| 13 | DriverSeat (`VehicleSeat`) | 2 x 0.8 x 2 | trim colour; `HeadsUpDisplay` false, `MaxSpeed` 0, `Torque` 0, `CanTouch` false, attribute `BoatSeat = "Driver"` | no |
| 14 | PassengerSeat (`Seat`) | 2 x 0.8 x 2 | trim colour; `CanTouch` false, attribute `BoatSeat = "Passenger"` | no |
| 15-16 | TrimL, TrimR | 0.4 x 0.4 x 14.2 | owner's stage rarity colour (R23) | no |
| 17 | Mast | 0.8 x 7 x 0.8 | studded square log look | no |
| 18 | Sail | 0.2 x 4.5 x 4.5 | cream (248, 240, 220) | no |
| 19 | SailStripe | 0.22 x 0.8 x 4.5 | rarity colour | no |
| 20 | Flag | 0.1 x 1 x 1.6 | rarity colour | no |
| 21 | BowLamp | 0.8 cube, Neon, warm white | `PointLight` range 12, on at night only | no |
| 22 | HPAnchor | 0.2 cube, invisible | holds the BillboardGui | no |

All parts except `Collider` are `Massless`, with `CanCollide`, `CanTouch` and `CanQuery` off. They are welded to `Collider` with WeldConstraints. On `Collider`'s root attachment: `LinearVelocity` (Plane mode, world X and Z axes, MaxForce = mass x 150), `AlignPosition` (OneAttachment, PerAxis, force on Y only = mass x gravity x 4, Responsiveness 30, target y -2.0), `AlignOrientation` (OneAttachment, Responsiveness 25). Attachments at the stern corners and bow hold the wake and splash emitters.

**Display skiff** at the main dock: an anchored copy with prompts and movers removed.

**Docks** (6). Placement uses the bearing formula in the glossary.

| Dock | Bearing | Shore | Pier (radius from, to) | Deck height | Pads (centre radius) | Extras |
|---|---|---|---|---|---|---|
| D1 | 76° | Island | 372 to 412 | y 0.7 | 429, 449 (facing out) | sign |
| D2 (main) | 200° | Island | 372 to 412 | y 0.7 | 429, 449 | Stud Shop on the beach at r about 360; display skiff moored beside the pier |
| D3 | 325° | Island | 372 to 412 | y 0.7 | 429, 449 | sign |
| D4 | 76° | Forest | 1012 to 972 | y 1.0, one step up to the y +2 beach | 955, 935 (facing in) | Gate rocks |
| D5 | 200° | Forest | 1012 to 972 | y 1.0 | 955, 935 | Gate rocks, campfire |
| D6 | 325° | Forest | 1012 to 972 | y 1.0 | 955, 935 | Gate rocks |

Each dock: a pier 8 wide and 40 long (Stud Bridge planks), 6 log pilings, a dock post with the prompt, a sign, an exit point (an invisible part on the deck), two pad boxes (9 x 4 x 18, invisible, `CanCollide`, `CanQuery` and `CanTouch` off, tag `BoatPad`) with four thin studded mooring rails outlining each pad at the waterline, and two swim steps beside the pier end (tops at y -1.6 and y -0.4, so each step up is 1.6 studs or less). The island pier root sits on the beach (top y -1.0) with a two-step stair up to the deck. The 12 `FillsDockGap` pieces at the old south dock (`NOTES.md:1483`) stay, because no dock goes there.

**Rest islets** (16): tree islets at r 815 to 825, bearings 11.25° + 22.5° x k. Any islet whose centre is within 40 studs (measured sideways) of a channel's centre line is turned along the ring until it is 40 studs clear. Attribute `Role = "RestIslet"`.

**Channel buoys** (12): two per channel at r 655 (where the stones end), 16 studs either side of the centre line. Each is three stacked studded boxes 2 x 1.2 x 2 (red, white, red) plus a 1 x 0.6 x 1 Neon top with a `PointLight` (range 10, on at night). Anchored.

**Sea Crate template** (`ReplicatedStorage.SeaAssets.SeaCrate`, cloned by the client): a 3 x 3 x 3 studded crate body (160, 120, 80), four darker bands, a 3-stud pole and a small flag in the owner's rarity colour. 7 parts, all non-colliding. Bobs 0.3 studs at 0.5 Hz on the client.

### 7.3 Sounds (picked from Roblox's licensed library in Studio, each id checked to load before it is written down, as was done for music in D-002)

| Sound | Use | Fallback already in the place |
|---|---|---|
| Soft water lapping loop | boat moving, volume by speed | none |
| Splash | summon, wreck, entering the water at speed | none |
| Wood crack | wreck | none |
| Hull thud | boat hit | Bat `Hit2` 3932505023 ([[Existing VFX and Props]]) |
| Bubble tick | stamina Tired | none |
| Soft whoosh | wash-up fade | none |
| Swim stroke loop | swimming | none |
| Drink | Aqua Affinity | existing `PotionDrinkSound` (`GardenConfig.lua:1797`), automatic |

---

## 8. Asset requests for Samarth

Only these need you. Everything else (boat, docks, islets, buoys, crate, bottle, sounds, code) is built in Studio from parts and packs already in the place.

**1 and 2. Aqua Affinity icon and shop art:** no separate request. [[Potions]] owns both images (`aquaaffinity-badge.png` and `aquaaffinity-shop.png`, its asset requests 1 and 2, in the potion pixel style). The two prompts that were here are withdrawn so the same potion is not drawn twice in two styles ([[Integration Review]], conflict C9).

**3. "BOAT DOCK" panel title (nice to have).** Made in the same Figma batch as every other INDEX-style title ([[King UI Overhaul]] section 7.3, which now lists it). Until it exists, the panel uses the kit's blue title bar with Fredoka text, which the kit allows.

**4. After the build: Save to Roblox, then publish.** Map edits made over the Studio bridge live only in the open session until you save (`NOTES.md:1567`).

**Animations to publish: none.** Swimming uses Roblox's own default swim animations, read from each character's `Animate` script. The boat has no animation.

**Robux products: none.** The boat costs shards, as you asked.

---

## 9. Code plan

Follows the GardenService module pattern: each module is `return function(G)`, loaded in order by `GardenService.lua`, with shared state passed through `G` ([[Carrying, Merging, Storage, Rebirth, Tools and Saving]] section 3.1).

### 9.1 New server modules (children of `ServerScriptService.GardenService`)

Load order: after the Crown modules (`GardenService.lua:518`) and before the wiring block (`:522`):
```
require(script.Swimming)(G)
require(script.Boats)(G)
-- require(script.SeaCrates)(G)   -- Phase 2 only (section J)
```
`Ambient` still loads last (`GardenService.lua:824`), so it can call `G.swimTick`.

**`GardenService.Swimming`** (about 250 lines)
- Takes from G: `Config, Players, applyWalkSpeed, effectActive, notifyRemote, noticeRemote, plotFor`.
- `state`: a weak table keyed by character (`setmetatable({}, {__mode = "k"})`), holding `{stamina, dryFor, exhaustedFor, lastDry, washing}`. Keyed by character for the same reason as `cloudStill` (`Ambient.lua:186-190`): it dies with the body.
- `G.swimTick(player, character, root, humanoid, dt, footing, supported)`, called from Ambient's sweep:
  1. Wet or dry (R5): if `supported`, wet = `footing` is nil or `Config.DrownWaterFloors[footing.Instance.Name]`, and the player is not on sky ground and not in a seat with a `BoatSeat` attribute. If not supported, keep the last value.
  2. Stamina (R6, R7, R18), scaled by `dt` (never a bare `task.wait()` loop; [[Swimming Fatigue and Boats in Roblox]] proposal 7).
  3. Last dry position (R12): set when dry, supported, not seated, horizontal speed under 30, and `footing.Instance` is anchored or is Terrain.
  4. State (R8) and exhaustion timer (R12). On a state change, write `SwimState` and call `applyWalkSpeed(player)`, only on the change, as `SkyLegs` does (`Ambient.lua:309-313`).
  5. Write `Stamina` rounded to 0.1, only when it changes.
  6. Wash-up: stamp `WashUpAt`, then `task.delay(0.6, ...)`: if still alive and still exhausted, `character:PivotTo(lastDry + Vector3.new(0, 3, 0))` (fallback `plotFor(player).spawnCF`), zero the root velocity, set stamina to 30, and send `Notify SwimWashedUp`.
  7. First water entry this session: `Notice SwimTip`.
- `G.swimRefill(character)`: sets stamina to 30 (used by wrecks, R48).
- Its own `Players.PlayerRemoving` handler (one parameter) clears the per-session tip flag.

**`GardenService.Boats`** (about 600 lines)
- Takes from G: `Config, Players, Store, notifyRemote, noticeRemote, setShards, plotFor, ProximityPromptService`.
- Remotes, made with the Crown module's `remoteEvent` pattern (`Crown.lua:43-51`): `BoatBuy`.
- Collision groups at startup: `PhysicsService:RegisterCollisionGroup("Boats")` and `("Characters")`. `Boats` does not collide with `Characters` or with `Boats` (R40). The module connects `Players.PlayerAdded`, then `player.CharacterAdded`, and sets every BasePart of the character to `Characters`, including parts added later (accessories load late, `GardenService.lua:254-256` comment).
- Docks registry: CollectionService tags `BoatDock` (models with attributes `DockId`, `Shore`), `BoatPad`, and `BoatPrompt` (attribute `Kind` = `Dock`, `Shop`, `Drive` or `Board`).
- Ownership: `owned[player] = {StudSkiff = true}`. `G.boatCapture(player)` returns the table. `G.boatApply(player, saved)` fills it and publishes `BoatsOwned`. Both are late-bound from Persistence, exactly like `G.royalCapture` and `G.royalApply` (`Persistence.lua:81`, `:269`; `Crown.lua:252`, `:270`).
- `buyBoat(player, key)` on `BoatBuy` (R21): same shape as `buyTool` (`Tools.lua:91-112`): validate, `setShards`, add to the owned set, publish, `Store.Touch(player, Config.SaveTouch)`, `Notice BoatBought`. It is yield-free.
- Its own `ProximityPromptService.PromptTriggered` connection, for prompts tagged `BoatPrompt` only. The existing handler already ignores anything that is not a fruit (`GardenService.lua:522-526`).
  - `Dock` from an owner: `summon(player, dock)` (R28, R29). From a non-owner: nothing (the client opens the panel).
  - `Drive`: owner only, `seat:Sit(humanoid)`.
  - `Board`: friend check (R31), seat empty, boat speed under 4 studs/s, then `seat:Sit(humanoid)`. Friend results come from `player:IsFriendsWith(ownerId)` inside `pcall`, cached per pair for the session.
- Seats: on `VehicleSeat` `Occupant` changes: owner sits = `SetNetworkOwner(owner)` and `DriverUserId` set. Anyone else = unseat (destroy the `SeatWeld`). Empty = `SetNetworkOwner(nil)` and the movers reset (R39). On `Seat` `Occupant` changes: unseat anyone who is not a friend. On any occupant leaving: if within 30 studs of a pad, `PivotTo` the dock's exit point (R34).
- `task.spawn` loop every 0.2 s over at most 16 boats: validation (R41), despawn timers (R50), pad-hogging timer, regen bookkeeping (R46).
- `G.damageBoat(boat, amount, attacker, source)` (R44), `G.boatsNear(position, radius)` (returns boats whose `Collider` is within `radius`), `G.boatOf(player)`. Exported for [[Weapons and Artillery]]. The template's `Collider` carries the CollectionService tag `BoatHull` (Weapons raycasts against it).
- `wreck(boat)` (R48). It calls `G.swimRefill` on both riders' characters.
- When `Shards` changes, the first time in the session that a non-owner reaches the price: `Notice BoatAffordable` (R22).
- Its own `Players.PlayerRemoving` handler (one parameter): despawn the boat (R51), clear the runtime tables (HP, cooldowns, friend cache). It never touches the save, so it does not interfere with the save-first order in `teardownPlayer` (`GardenService.lua:703-715`).

**`GardenService.SeaCrates`** (about 150 lines, Phase 2 only, off in v1; shards only if shipped)
- Takes from G: `Config, Players, Store, balances, publishCash, setShards, cashMultiplier, noticeRemote`.
- Remotes: `SeaCrate` (server to owner: `Vector3` or `nil`) and `SeaCrateClaim` (client to server, no payload).
- A per-player table `{position, nextAt}`, a 1 s loop that spawns due crates (R53, R54), and claim handling (R55, R56).
- Stage: `math.min(9, (player:GetAttribute("Rebirths") or 0) + 1)` until [[Rarity Gating and Rebirth]] exports a shared `G.stageOf`. Then use that.
- First-fruit income: `Economy.ForName(firstFruitOfStage) * cashMultiplier(player)` (`FruitEconomy.lua`, `Progression.lua:324-336`).

### 9.2 New client scripts (`StarterPlayer.StarterPlayerScripts`)

**`SeaClient`** (about 250 lines)
- The SeaStatus bar in stamina mode (6.1), driven by the character's `Stamina` and `SwimState`. It eases toward each 0.2 s server value so the bar moves smoothly.
- The swim animation: loads `Animate.swim.Swim` and `Animate.swimidle.SwimIdle` from the character's `Animate` script (verify these child names in Studio). Priority `Movement`, so it beats the default walk (Core priority) and loses to the carry pose, which is `Action` (`Carrying.lua:67`).
- Bubble ticks while Tired. A surface splash emitter (`Water-01` clone) is enabled only while swimming and moving, switched on and off, never rate-tweaked every frame ([[Roblox Platform Constraints for the Expansion]] section 6).
- Sea Crates: listens to `SeaCrate`, clones the crate template locally, bobs it, shows the chip, and fires `SeaCrateClaim` once when within 10 studs.

**`BoatClient`** (about 450 lines)
- Driver controller: when the local player sits in a `VehicleSeat` with `BoatSeat = "Driver"` and is the owner, each `Heartbeat` reads `ThrottleFloat` and `SteerFloat` and writes the movers on its own copy (R36, R37). The driver owns the physics, so the result replicates. It stops when unseated.
- The SeaStatus bar in boat mode, from the boat's `HP`, `MaxHP`, `ProtectedUntil` and the dock-zone distance.
- Prompt texts and visibility: Dock prompt text by `BoatsOwned`; Drive shown only to the owner; Board shown only to friends (`LocalPlayer:IsFriendsWith`, cached).
- The Boat Dock panel (6.3) and the `BoatBuy` remote.
- For every boat in `workspace.Boats`: billboard visibility, damage looks (R47), the wake emitter switched by speed bands (off, under 15, 15 and up), the water loop volume by speed, and the local sink animation when `Broken` turns true (R48).
- A one-line `Notice` the first time per session the player drives: "Press Space (Jump) to leave the boat." On touch screens: "Press Jump to leave the boat."

### 9.3 Changes to existing scripts

| File and lines | Change |
|---|---|
| `GardenService.lua:191-248` (`applyWalkSpeed`) | After the arena block (`:200-208`): if the character's `SwimState` is not `Dry`, `speed = Config.Swim.speed[state]`, or `Config.Swim.speed.Swim x Config.Effects.AquaAffinity.swimMultiplier` (12 x 2.5 = 30) while that effect is active, and the Speed potion is skipped (R9). Hip height (`:243-247`): subtract `Config.Swim.sink` (1.6) while swimming, never below 0.2. This stays the only writer of Humanoid speed and hip height. |
| `GardenService.lua:518` | Add the three `require` lines (9.1). |
| `Ambient.lua:359-375` | Replace the `FloorMaterial`-gated `InDeepWater` decision with the server-ray rule in R5, reusing `footing` and `supported` from `:273-274`. Delete the swept-speed test (`carried`, `:362-364`): with no conveyor it has no job, and boats go faster than 30. Then call `G.swimTick(player, character, root, humanoid, Config.DrownTick, footing, supported)`. |
| `Ambient.lua:26-33` | Rewrite the comment: there is no current, and the kill past r 2825 only catches someone outside the map. |
| `PlayerVision.lua:141-153` (`groundUnder`) | Delete the `ReturnFloors` branch (`:149-151`). Treat a seat with a `BoatSeat` attribute as land. |
| `PlayerVision.lua:255-265` | Delete the swept-speed override. |
| `PlayerVision.lua:272-297` | Delete both airborne pushes (R2). |
| `PlayerVision.lua:316-324` and `:438-469` | Add the exhaustion term (capped at 0.45) and the wash-up fade (6.2). |
| `GardenConfig.lua:1261-1327` | Delete `CurrentSpeed` (`:1310`), `CurrentAirAccel` (`:1311`), `ReturnSpeed` (`:1319`), `ReturnFloors` (`:1320`), `ReturnAirAccel` (`:1324`) and `DrownSweptSpeed` (`:1291`) once the code above no longer reads them. Set `DrownWaterFloors = { Seabed = true }` (`:1327`). Rewrite "the deep" comment. |
| `GardenConfig.lua:1584` | `DrownWarning` becomes "You swam too far out!" (it only fires outside the map). |
| `GardenConfig.lua:1615-1705` | `Tools.AquaAffinityPotion` is added by [[Potions]] (its code plan: `priceUnits = 25`, `priceCapBoatShare = 0.5`, `duration = 600`). This note adds nothing there. |
| `GardenConfig.lua:1714-1789` | `Effects.AquaAffinity` is added by [[Potions]] (`bar = Color3.fromRGB(40, 130, 255)`, `swimMultiplier = 2.5`, `drainMultiplier = 0`). `applyWalkSpeed` reads `Config.Swim.speed.Swim x swimMultiplier` = 30 while it runs. |
| `GardenConfig.lua:1863-1867` | Add `"SeaStatus"` to `HudClusters`. |
| `GardenConfig.lua` (new blocks) | `Config.Swim`, `Config.Boats`, `Config.BoatRules`, `Config.BoatPrice = 1000` (read by [[Potions]] for the Aqua cap), `Config.SeaCrate` (Phase 2, `Enabled = false`) (values from section 5) and the strings in 6.6. |
| `Persistence.lua:81` | Add `if G.boatCapture then profile.boats = G.boatCapture(player) end`. |
| `Persistence.lua:269` | Add `if G.boatApply then G.boatApply(player, profile.boats) end`. |
| `GardenStore.lua:220-283` (`Store.Default`) | Add `boats = {}`. |
| `GardenStore.lua` sanitise, next to `kingOwned` (`:389-406`) | Copy only keys that exist in `Config.Boats` with value `true`. This is required: a field missing from sanitise is dropped on the next load (`GardenStore.lua:447-450`). |
| `ShopMenu.lua:616`, `IndexMenu.lua:47`, `StoredFruitsMenu.lua:541`, `UpgradeMenu.lua:421`, `RebirthMenu.lua:51` | Add `"BoatDockMenu"` to each panel's "hide the others" list. |

Note: the shop sorts tool cards by key name (`ShopMenu.lua:382-383`), so `AquaAffinityPotion` will show first. If [[Potions]] adds an order field, use it.

### 9.4 Remotes

All live in `ReplicatedStorage.GardenRemotes`.

| Remote | Direction | Payload | Server validation |
|---|---|---|---|
| `BoatBuy` | client to server | boat key (string) | `typeof == "string"`; `Config.Boats[key]` exists; `Store.IsReady(player)`; not already owned; root within 25 studs of a `BoatDock` post; `Shards >= price`. Yield-free from the check to `setShards`. |
| `SeaCrate` (Phase 2, not created in v1) | server to owner | `Vector3` or `nil` | (server-sent only) |
| `SeaCrateClaim` (Phase 2, not created in v1) | client to server | none | crate exists for this player; root within 14 studs (18 if seated in a boat); loaded. Then pay and schedule the next crate. A claim with no crate does nothing. |

Prompts (not remotes) are handled server-side through `ProximityPromptService.PromptTriggered`, with the rules in R27 to R34. The wash-up fade needs no remote: it uses the `WashUpAt` attribute.

### 9.5 Saved fields

| Field | Type | Default | Migration |
|---|---|---|---|
| `boats` | table `{[boat key] = true}` | `{}` | Old saves have no field, so they load as `{}` (does not own a boat). No schema bump: adding fields needs none (`GardenStore.lua:8-12`). The self-test's worst-case payload limit of 8,192 bytes (`GardenStore.lua:1209`) gains about 30 bytes. |

Not saved, on purpose: stamina, swim state, last dry position, boat HP, `BoatReadyAt`, Sea Crate position. All of them are runtime only and reset on rejoin.

### 9.6 Attributes

| On | Attribute | Type | Written by | Read by |
|---|---|---|---|---|
| Player | `BoatsOwned` | string, comma list (like `KingOwned`) | Boats | BoatClient |
| Player | `BoatReadyAt` | number, server time | Boats | BoatClient (dock prompt text, wait message) |
| Player | `AquaAffinityEndsAt`, `AquaAffinitySpan` | number | existing potion code (`Tools.lua:217`, `:223`) | GardenClient effect stack, Swimming, `applyWalkSpeed` |
| Character | `Stamina` | number 0 to 30, 0.1 steps | Swimming | SeaClient |
| Character | `SwimState` | `"Dry"`, `"Swim"`, `"Tired"`, `"Exhausted"` | Swimming | `applyWalkSpeed`, SeaClient, PlayerVision |
| Character | `WashUpAt` | number, server time | Swimming | PlayerVision |
| Character | `InDeepWater` | bool (existing, now decided by R5) | Ambient | Ambient |
| Boat model | `OwnerUserId`, `OwnerName`, `BoatKey`, `HP`, `MaxHP`, `Broken`, `ProtectedUntil`, `DriverUserId`, `TrimTier` | number, string, bool | Boats | BoatClient, Weapons (these are the contract names; Weapons uses them as written, D-024) |
| `Collider` part | CollectionService tag `BoatHull` | tag | template | Weapons |
| Seat parts | `BoatSeat` | `"Driver"` or `"Passenger"` | template | Swimming, Ambient, PlayerVision, BoatClient |
| Dock models | `DockId`, `Shore` | string | place | Boats |
| Prompts | `Kind` | `"Dock"`, `"Shop"`, `"Drive"`, `"Board"` | place or template | Boats, BoatClient |
| Workspace | folder `Boats` | - | Boats | BoatClient |

### 9.7 Place edits (Studio, during the build)

1. Move the 24 `CurrentPlate`s, the six `ReturnPlate`s and `ReturnRamp`s, the `LaneWater` and `Foam` strips, and the six Gate labels to `ServerStorage.Currents_backup_2026-10-04`, each with a `WasIn` attribute (R1, R3, R4).
2. CausticEdge beams: TextureSpeed 1.5 to 0.4, keeping `OrigTextureSpeed` (R3).
3. Widen the six channels to 30 studs (R25).
4. Build the six docks, 16 rest islets, 12 buoys, the display skiff and the Stud Shop (section 7.2). Then run `ShallowsBuilder` `B.finish` for the islets. It is place-locked and needs a clone because Edit-mode `require` caches ([[World Builders and Dev Tools]] gotchas 2 and 3).
5. Templates: `ServerStorage.Boats.StudSkiff`, `ReplicatedStorage.SeaAssets.SeaCrate`, `ReplicatedStorage.Tools.AquaAffinityPotion`.
6. `GardenUI.SeaStatus` and `GardenUI.BoatDockMenu` in StarterGui.

---

## 10. Edge cases and abuse

| Case | What happens |
|---|---|
| Driver leaves the seat at sea | The boat holds still under server physics (R39). The passenger stays. Despawn after 90 s with nobody aboard. |
| Driver resets or dies while seated | The seat weld breaks, same as leaving. They respawn on their plot (existing). Potions clear on death (existing, `GardenService.lua:601-604`). With a passenger aboard, the boat despawns 10 s later and the passenger goes to the nearest dock exit (R50, R51). |
| Owner leaves the game mid-sea | The boat is destroyed (R52). The passenger goes to the nearest dock exit (R51). |
| Passenger leaves the game | The seat frees. Nothing else changes. |
| Wreck with riders carrying fruit | The fruit stays welded to them (R49). They swim with full stamina. The nearest rest point is at most about 200 studs away (stones to r 643, islets at r 820, the forest at r 995), well within one bar (330 studs). |
| Exhausted while carrying | Wash-up moves the character with the fruit (R13). |
| Rebirth while sailing or swimming | Allowed. Rebirth never touches the boat or the character (`RebirthLock.lua:63-128` lists what it resets). The trim colour updates to the new stage. Boat ownership is kept (D-009). |
| Rebirth during a boat purchase | The purchase is yield-free, so a rebirth cannot land halfway through it. |
| Rejoin mid-wash-up or mid-wreck | All of this is runtime. The player rejoins on their plot. The wreck cooldown resets on rejoin (accepted; see Q12). |
| Summon spam | 15 s cooldown, one boat per player, server cap of 16 (R28, R29). |
| Pad hogging | A boat with no driver on a pad despawns after 30 s (R50). |
| Exploiter flies or teleports a boat they drive | Caught within 0.2 s by R41, snapped back, despawned after 3 strikes. |
| Exploiter sits in someone else's seat | Seats have `CanTouch` off and the server unseats any occupant who is not allowed (R31, R32). |
| Exploiter fakes stamina or speed | Stamina is server-only. Swim speed is set by the server, but a client can still change its own WalkSpeed. That is an existing risk for all movement and is out of scope here. Optional: log swimmers above 39 studs/s (1.3 x 30) in the dev counters. |
| Exploiter claims a crate from far away | The server checks distance (R55). One crate per player, 180 s apart. |
| Exploiter spams `BoatBuy` | It is refused once owned. No shards move twice. |
| Griefing by ramming | Boats do not collide with boats or characters (R40). |
| Griefing by shelling boats at docks | Docks are safe zones and new boats have 15 s protection (R30, R44). |
| Repeatedly wrecking one player | It costs them a 60 s rebuild and a swim, and nothing else (R49). Open question Q5 covers limiting who may shoot boats. |
| Bat knockback on a seated player | Seated players are skipped by the bat's push ([[Bats]]). Bats do no boat damage. |
| Swimmer under a boat | No collision, no fling (R40). |
| Boat stuck on stones | The driver leaves and summons again at a dock. The old boat is destroyed on summon. |
| Mobile | The thumbstick drives through Roblox's vehicle controls (verify in the device emulator). Jump leaves the seat. Prompts are tapped. SeaStatus scales with HudScale. The Buy button stays 44 px tall or more. |
| Server authority overall | Purchases, HP, damage, wrecks, stamina, wash-ups and crates are all decided on the server. The client only draws, and drives physics inside R41's limits. |
| Plotless players (50-player servers, 12 plots) | They can swim and use boats. Wash-up with no last dry position falls back to the plot spawn. With no plot, use the main island dock's exit point. |
| Sky islands, the arena | No water there. The arena override in `applyWalkSpeed` comes first (`GardenService.lua:200-208`), unchanged. |

---

## 11. Performance

Budgets from [[Roblox Platform Constraints for the Expansion]]: about 40 to 60 moving physics objects per client, under 50 KB/s download per client, Atomic streaming for moving models, no per-frame server tweens.

| Item | Count | Cost |
|---|---|---|
| Boats | at most 16 per server, about 22 parts each, one assembly each | 16 moving assemblies, inside the 40 to 60 budget. At 0.4 to 0.9 KB/s each, all 16 in view cost about 6 to 14 KB/s. |
| Docks | 6 x about 50 parts, plus the Stud Shop's 30 and the display skiff's 20 | about 350 anchored parts |
| Rest islets | 16 x about 8 parts | about 130 anchored parts |
| Buoys | 12 x 5 parts | 60 anchored parts, 12 night lights (range 10) |
| Removed | 24 CurrentPlates + 6 lanes x (plate, ramp, strip, 2 foam, label) | about 60 fewer parts, and two scrolling textures per lane gone |
| Net change | | about +480 parts on a workspace of about 71,700 (`NOTES.md` forest entry) |

Per-frame and per-tick work:
- **Server:** swim logic runs inside the existing 0.2 s drown sweep, reusing its ray (no new raycast per player). The boat loop runs every 0.2 s over at most 16 boats. The crate loop runs every 1 s. No server tweens: the wreck sink is played by each client (R48).
- **Client:** the driver controller runs only on the driver's device. The SeaStatus bar updates only while visible. Billboards and damage looks change only on attribute changes. The wake emitter switches between three speed bands instead of tweaking its rate every frame.
- **Network:** `Stamina` changes at most 5 times a second per swimmer, and only while it is changing. `HP` changes only on hits and regen steps (regen writes at most once a second). Crate positions go only to their owner.
- **Particles:** wake rate 20 per second at most (mobile caps each emitter at 100 per second anyway).

---

## 12. Test plan

Run every playtest with `workspace.StudioNoSave = true` (`GardenStore.lua:572-575`), so tests never write real saves. Use the Studio dev doors `DevSetShards` (Studio only, `GardenService.lua:631-684`) to give test shards. For two-player tests, use Studio's local server with 2 players.

| Test | Rules | Steps | Pass when |
|---|---|---|---|
| T1 No current | R1, R2 | Stand still in the open sea at 8 bearings, then jump 10 times in place | Horizontal drift is 0.0 studs over 5 s at every bearing, standing and airborne |
| T2 No flowing look | R3, R4 | Screenshot a lane and the caustics from the island and the forest | No turquoise strips, no "Current to the island" labels; caustics barely drift |
| T3 Wet and dry | R5, R16 | Stand on the seabed, a stone, driftwood, an islet, a dock, the forest shelf, an island pond, and sit in a boat | `InDeepWater` and `SwimState` are wet only on the seabed |
| T4 Drain and refill | R6, R7 | From full: swim 10 s, then stand on a stone | `Stamina` reads 20.0 plus or minus 0.3 after 10 s; refill starts 1.5 s after landing and is full 4 s later |
| T5 Still drain | R6 | Float still for 10 s | Stamina drops by 5.0 plus or minus 0.3 |
| T6 Speeds | R8, R9 | Read `Humanoid.WalkSpeed` in each state, with and without the Speed potion | 12 / 8 / 4, the Speed potion has no effect in water, normal walking on land |
| T7 Body height and animation | R10 | Swim, moving and still | The root sits at y -2.4 plus or minus 0.2; swim and swim-idle animations play; the carry pose still shows the arms up when carrying |
| T8 Jump out | R11, R26 | Swim to a stone, a swim step and the island beach edge | All are reachable with one jump |
| T9 Exhaustion recovery | R12 | Let the bar empty next to a stone, then climb on within 4 s | No wash-up; refill begins |
| T10 Wash-up | R12, R13, R14 | Carrying a fruit, with a Speed potion running, swim straight out from the outer stones until the bar empties and wait | At 4.0 s the screen fades, the character appears on the last stone it stood on, stamina is 30, the fruit is still held, the potion timer is unchanged, health is unchanged |
| T11 Tips | R15, R22 | Enter the water twice in one session; set shards from 900 to 1,000 | Each green notice shows exactly once |
| T12 Aqua Affinity | R17, R18 | Drink it, swim 60 s, then drink a second | Speed 30 (12 x 2.5), stamina never drops, the timer stacks to about 20 min, the effect block shows the badge; it clears on death |
| T13 Buying | R20, R21 | Try buying with 999 shards, far from a dock, then with 1,000 at a dock, then again | Refused, refused, bought (shards drop by 1,000, `BoatsOwned = "StudSkiff"`), refused |
| T14 Kept on rebirth | R20 | Buy, then rebirth with the dev tools | Still owned; trim colour changes to the new stage's rarity colour (R23) |
| T15 Save round trip | 9.5 | Turn `StudioNoSave` off only in a separate test experience with its own DataStore (never in place 88602028667228), with API access on: buy, leave, rejoin | Still owned. Then a profile saved before this change loads with `boats = {}` |
| T16 Summon rules | R27 to R30 | Summon, summon again before 15 s, fill both pads with two players, spawn 16 boats with a test command | Each refusal shows its exact red line; a new summon destroys the old boat; spawn protection blocks damage for 15 s |
| T17 Seats | R31, R32 | A non-friend and a friend try to board; force a non-owner into the driver seat with the command bar | Non-friend sees no Board prompt and a server-side sit is undone; the friend boards; the forced driver is unseated |
| T18 Carrying aboard | R33 | Board while carrying a fruit; press Drop at sea | Boarding works; Drop is refused with the existing off-plot warning |
| T19 Leaving | R34 | Leave near a pad, then mid-sea | Near a pad you land on the pier exit; mid-sea you drop into the water and start swimming |
| T20 Controls | R35, R36, R37 | Drive with keyboard, gamepad and the emulated phone thumbstick; time 0 to top speed; damage the boat to 25% and re-measure | Top speed 44 plus or minus 1; about 2.4 s to reach it; 30.8 plus or minus 1 at 25% HP; no VehicleSeat speed readout on screen |
| T21 Ownership | R39 | Print `Collider:GetNetworkOwner()` while driving, after leaving, and with only the passenger aboard | Owner, then nil, then nil; the boat does not turn back toward its spawn heading after the driver leaves |
| T22 No ramming | R40 | Drive through a swimmer and through another boat | No collision, nobody flung |
| T23 Validation | R41 | From the driver's client, set the boat's CFrame 200 studs away, then repeat 3 times | Snapped back each time; despawned on the third strike; `BoatReadyAt` is 60 s out |
| T24 Damage rules | R43 to R45 | Call `G.damageBoat` from the command bar with both players flagged: at the centre and at the edge of a 20-stud blast, inside a dock zone, from the owner, twice within 0.1 s; then once with the driver unflagged | 300 and 90 for a 300 base; 0; 0; counted once; 0 when unflagged |
| T25 Regen | R46 | Damage to 500, wait 8 s, then 10 s more; put the boat away and summon it again | Regen starts at 8 s and gives about 200 HP in 10 s; the HP value carries over when re-summoned |
| T26 Wreck | R48, R49 | Damage to 0 with a passenger carrying a fruit | Riders drop with stamina 30 and the fruit still held; the boat sinks over 2 s and is gone at 2.5 s; 60 s rebuild; no shards or cash change for anyone |
| T27 Despawn | R50 to R52 | Leave a boat empty mid-sea for 90 s; leave one on a pad with only a passenger for 30 s; the owner leaves the game with a passenger aboard | Despawns at 90 s and 30 s; the passenger is placed on the nearest dock each time |
| T28 Crates (Phase 2 only) | R53 to R56 | With `Config.SeaCrate.Enabled` true in a Studio copy: join, wait 60 s, sail to the crate; try claiming from 30 studs with the command bar; check the second player cannot see it | Crate appears; claim pays exactly 2 x the stage's shard drop and no cash; the far claim is refused; the other player sees nothing. In v1, confirm no crate ever appears |
| T-UI-1 No overlap | R62, 6.1 | At 1600 x 900, 1280 x 720, 812 x 375 and 667 x 375 (touch emulation), show SeaStatus with the carry row visible, then with the Weapon Bar visible, and fire a green notice; compare `AbsolutePosition` and `AbsoluteSize` of SeaStatus, CarryInfo, CarryButtons, the Weapon Bar, the notice label, CornerHud, EffectStack and the side buttons (the [[King UI Overhaul]] `UIOverlapCheck` does this) | No two rectangles intersect; SeaStatus is at least 8 px from the hands stack and from the notice lane |
| T-UI-2 Kit only | R61 | Byte-scan the new scripts and Config strings for non-ASCII; check every TextLabel's FontFace | No non-ASCII bytes; FredokaOne everywhere |
| T-UI-3 Panel | 6.3 | Open the Boat Dock panel, then the Shop, then the Index | Only one panel is visible at a time; walking 20 studs away closes it |
| T-Perf | 11 | 16 boats moving with 2 clients | Client receive under 50 KB/s in the Network stats; no new server script above 1% activity |

---

## 13. Dependencies and conflicts

| System | How it touches this one |
|---|---|
| [[Rarity Gating and Rebirth]] | The boat is kept through rebirth (D-009). The trim colour uses the player's stage (`Config.StageFor`). The Stage Model plans the boat as the stage 2 spend (`sim/final.py`). Sea Crates are off in v1; if they ship, re-run the [[Integration Check]] script with crate shards added. |
| [[Bats]] | Same rarity colour for trim and bat. Bats skip players sitting in a boat seat (its rule B12m, D-014). Every dock's 40-stud zone and the Boat Shop are tagged `BatSafeZone`. Bats do no boat damage. Swimmers and players standing on a deck can be knocked like anyone on land, with no stamina effect. |
| [[Explosives and Fruit Health]] | TNT only works on plots (its rule 7), so it never reaches boats. Boats and explosives use the one shared falloff, `Ballistics.falloff` (R45). |
| [[Weapons and Artillery]] | The main link. Resolved in the [[Integration Review]] (C10, D-024): Weapons owns the weapon list and damage numbers (5.7 copies them), this note owns the boat (HP 1,000, spawn protection 15 s, regeneration, dock zones, attribute names). Weapons resolves hits on the server, applies the shared falloff and calls `G.damageBoat` and `G.boatsNear`. Boats break only when both sides have the War Flag up (R44), or from the Phase 3 Fort. Weapons never applies physical force to boats (R42). |
| [[Underground and Ores]] | No world overlap: the underground is under the plots at r about 200, the sea starts at r 380. Shards compete: 1,000 for the boat against 100K for the underground. That is negligible after stage 2 (table 5.5). |
| [[Pets]] | The forest is the reason to cross. Taming needs the requested fruit carried to the forest, so carrying while swimming and sailing must work (R33, R13), and nothing is lost at sea (R49). Pets never ride boats. Recommended: a tamed pet goes straight to the owner's plot, so it never needs a sea crossing. |
| [[Potions]] | Owns Aqua Affinity's price (`min(ShardPrice(25), 500)`), duration (600 s), multiplier (2.5, giving this note's 30 studs/s), colour and art (D-029). This note owns the swim mechanics (R18) and the `SwimState` attribute that Potions reads. The Speed potion does nothing in water (R9). Invisibility hides the character, not the boat. |
| [[Plot Overhaul and Environment]] | It re-skins the stepping stones, driftwood (studded logs, D-003) and lily pads. These are the swim route's rest stops, so they must stay anchored and collidable, with tops above y -1.2, and never be named `Seabed`. The rest islets, buoys and docks should get the same studded look. The caustic speed change (R3) and the Gate labels (R4) are in that system's area too; agree who does them. |
| [[King UI Overhaul]] | Same kit, no emoji. The Boat Dock panel uses its `UIKitPrefabs.Panel` shell. SeaStatus stays at the bottom centre (not the top-centre `Kit.Slot` the King UI note first suggested), placed by the rule in 6.1 so it never touches the notice toast lane. The BOAT DOCK title is in its Figma title batch. |
| [[Royal Combat]] | No contact: the arena is in the sky, and its speed override comes first in `applyWalkSpeed`. |

---

## 14. Open questions for Samarth

Each has a default, so building can start without waiting.

| # | Question | Default |
|---|---|---|
| Q1 | Where do the docks go? | Three island and three forest docks at bearings 76°, 200° and 325° (three of the old lanes); the 200° island dock is the main one with the Stud Shop. |
| Q2 | Add 16 small rest islets halfway across, so swimming is possible but tiring? Without them, one bar of stamina does not cover the open water. | Yes, 16 tree islets at r 820. |
| Q3 | Who can ride as the passenger? | The owner's Roblox friends only. This avoids strangers hopping in, with no extra buttons. |
| Q4 | Should boats bump into each other? | No. That stops ramming and pile-ups at docks. |
| Q5 | Who can break a boat? You asked for artillery to break boats. | Changed in the [[Integration Review]] to match [[Weapons and Artillery]]: only when the shooter and the boat's driver both have the War Flag up (plus the Phase 3 PvE Fort), never at docks and never in a boat's first 15 s. A new player crossing to tame pets is never sunk against their will. A wreck takes nothing, so it is not stealing. Flip `WeaponConfig.RequireFlag = false` to make every boat at sea breakable. |
| Q6 | Should repairing cost shards? | No. Boats heal over time, faster at docks; after a wreck you wait 60 s. |
| Q7 | Aqua Affinity price and length? | [[Potions]] decides: 75 shards at stage 1 up to 500 (half a boat) from stage 4, for 10 minutes, swimming at 30 (2.5 times the normal swim) with no tiredness. (In Minecraft, Aqua Affinity means faster mining underwater, so players who know Minecraft may expect that. The name is yours, so it stays.) |
| Q8 | Ship the Sea Crates (personal floating crates)? | Changed in the [[Integration Review]]: **not in v1**. The Golden Crate ([[Bats]]) is the one crate system at launch, and the cash part would have been income outside the pets-and-ores budget. If you want them later, they pay shards only (2 x the stage's per-merge drop). |
| Q9 | Should swimmers sink to chest depth and use a swim animation, or just wade slowly? | Sink and swim, using Roblox's default swim animation, so there is nothing to publish. |
| Q10 | Should the Speed potion work in water? | No, so Aqua Affinity keeps its point. |
| Q11 | Should the boat also be listed in the main Shop? | No. Docks only, plus one notice when you can afford it. |
| Q12 | Should a wreck's 60 s wait survive leaving and rejoining? | No. Rejoining takes time anyway. |
| Q13 | Keep the old lane gate rocks on the forest beach? | Keep the rocks, remove the "Current to the island" labels. |
| Q14 | More boat types later (bigger, faster, skins)? | One boat in v1. The save already allows more. |

Decisions log check: nothing here contradicts D-001 to D-011. D-009 already keeps the boat through rebirth. D-010's per-merge shard drops are reused for crate shards. D-011's shard costs and the model's stage 2 boat purchase line up with the 1,000 price. The [[Integration Review]] added D-016 (one crate system in v1), D-024 (boats are broken only through the War Flag; Weapons owns the numbers, this note owns the boat) and D-029 (Aqua Affinity priced and tuned by [[Potions]], 30 studs/s); this note follows them.
