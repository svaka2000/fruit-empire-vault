---
tags: [research]
topic: Swimming fatigue, boats, breakable boats, and boat games in Roblox (for replacing Fruit Empire's sea currents)
date: 2026-10-04
---

# Swimming Fatigue and Boats in Roblox

## Summary

- Roblox has no built-in swim fatigue or drowning. Games script their own. The engine's "Swimming" state (a Humanoid state, meaning the character mode the engine runs) only triggers in **Terrain water**, the voxel water drawn with Studio's terrain tools [1][11]. Fruit Empire's sea is made of parts, not Terrain water. So nothing will "swim" or float on it by default [1][2][45][46].
- Swim stamina works the same way in most games. A bar drains while you swim. It refills on land, usually after a short delay. Genshin Impact waits 1.5 s and then refills 25 stamina per second, and the bar hides when full [40]. At zero, games either make you drown slowly (Minecraft: 2 HP per second) or wash you up on nearby land (Zelda: Breath of the Wild) [42][43].
- Roblox boat games use a plain HP pool (HP = hit points, the damage something can take before it breaks). Grand Piece Online ships have 100 / 750 / 2000 HP. A damaged ship gets slower in proportion to the HP it has lost, and you repair it with planks (10 HP each) [35]. Blox Fruits boats have 100 to 2500 HP. The HP bar sits above the hotbar, and the boat sinks at 0 [37].
- A purchased boat should be a one-time unlock that you **summon** at a dock. Blox Fruits puts a Boat Dealer on almost every island [38]. SharkBite 2 has you pick a boat at the start of each round, and you can invite others onto it [31].
- The usual boat setup is one VehicleSeat for the driver plus one regular Seat for a passenger. A VehicleSeat is a seat that reads the driver's WASD keys. Its input values are `ThrottleFloat` and `SteerFloat`; the older `Throttle` and `Steer` are deprecated and no longer replicated. A VehicleSeat only drives motor joints such as wheel hinges, so a boat needs its own push force [6].
- **Network ownership** means which computer runs the physics for a part. The driver should own the boat. Roblox's own docs use this exact 2-seat case: if the passenger sits first, the passenger owns the boat and the driver lags. The fix is to give ownership to the driver when they sit [7]. Client-owned parts can be faked by exploiters, so damage must be worked out on the server [7].
- Fruit Empire's current plates are anchored parts with a velocity set. An anchored part with velocity works like a conveyor belt: anything touching it gets moved [5][46]. A boat hull touching those plates would be dragged at 42 studs/s. Either remove the plates or put them in a collision group that boats pass through [9].
- Build a Boat for Treasure makes its river water a **part** that only collides with boat blocks. Players sink through it and walk on the sand below [21]. That is the closest match to Fruit Empire's setup, and it shows the trick works in a big game.

## How it works

### 1. Swimming stamina and fatigue

**Terms.** *Stamina* is a meter that drains during hard actions and refills at rest. *Fatigue* is what happens when it runs low or hits zero.

**What Roblox gives you.**
- `HumanoidStateType.Swimming` is described as "swimming in Terrain water" [1]. `BuoyancySensor`, the newer part that reports whether something is in water, also only reads Terrain water [2].
- DevForum drowning systems are all hand-written. One approach reads the terrain voxels around the head to tell if it is underwater. Another runs a stamina value that drops every second and starts dealing damage at zero. Example timing given there: `TakeDamage(100/30)` once a second kills a 100 HP character in 30 s [11].
- To get a swim animation in part water, force the Swimming state and disable the GettingUp state. Otherwise "the roblox character will automatically get up if it is swimming and not in terrain water" [13]. One reply drives the body with `MoveDirection * WalkSpeed + (0, 4, 0)` (the 4 is a small upward push) to stop sinking [14].
- A common bug: stamina drains faster in the live game than in Studio. It comes from looping `task.wait()` with no time argument, so the speed depends on frame rate. The fix is to scale by delta time (the seconds since the last frame) on Heartbeat [12].

**How other games do it (numbers where confirmed).**

| Game | Drain | At zero | Refill | UI |
|---|---|---|---|---|
| Genshin Impact (not Roblox, the model many devs copy) | 100 base stamina. Swimming costs 4 per stroke (tied to the animation, not per second). Swim dash costs 2, then 10.2/s [40] | Swimming needs stamina; deep water triggers swim mode [41]. Exact zero-stamina penalty not confirmed in the pages read | After 1.5 s idle, 25/s [40] | Yellow bar beside the character. Shows only while draining or refilling, hides when full [40] |
| Zelda: Breath of the Wild | Swim uses the Stamina Wheel [42] | "Link is unable to swim without Stamina and will instead sink into the water only to appear on nearby land" [42] | Refills at rest [42] | Wheel near the character [42] |
| Minecraft | Mobs hold breath 15 s. Respiration (a helmet enchant) adds about 15 s per level [43] | 2 HP damage per second [43] | 1 bubble per 0.2 s once out of water [43] | Bubble bar. A pop sound rises in pitch as air runs out [43] |
| Grand Piece Online (Roblox) | A bubble timer over your head [34] | You start drowning and take damage. Devil Fruit users take damage right away and cannot swim [34] | Not confirmed | Bubble above the head (red for fruit users) [34] |
| Fisch (Roblox) | Oxygen drains once your head is fully underwater [39] | Death by asphyxiation [39] | Refills automatically after surfacing [39] | Oxygen meter. Gear extends it: Basic 150 s (3,000 C$), Advanced 230 s (15,000 C$), Water Bubble 540 s (25,000 C$) [39] |
| Blox Fruits (Roblox) | No stamina. Fruit users take rapid damage in water; players without a fruit take none. Sand fruit users take 2x [36] | Death | Holding jump skips across the surface and heals [36] | Health bar |
| Build a Boat for Treasure (Roblox) | No stamina. River water deals about 20 damage per second. Shallow water is safe but still moves you [21] | Death | Leave the water | Health bar shows only when below max [22] |

Fisch's base breath time without gear: not confirmed.

**Ways to soften or skip the cost.** Blox Fruits sells three accessories with 60 to 90% water-damage resistance [36]. GPO sells a Bubble that lets you swim longer, but it pops if you lose 10% of max HP [34]. Fisch sells diving gear tiers [39].

**Name check for "Aqua Affinity".** In Minecraft, Aqua Affinity is a helmet enchantment. It removes the 5x mining slowdown underwater. It does not touch swim speed or breath [44]. Fruit Empire can still use the name for a swim potion. Just know it means something else to kids who play Minecraft.

### 2. How boats are built in Roblox

**VehicleSeat vs a custom controller.**
- A VehicleSeat welds the player in and passes their input to motor joints connected to it. It does not work through a hinge to a separate part [6]. Boats have no wheels, so a script reads `ThrottleFloat` (-1 to 1, forward or reverse) and `SteerFloat` and applies the force itself [6].
- Roblox's modern movers replace the old body movers. `LinearVelocity` (with `Line` / `Plane` / `Vector` modes) replaces BodyVelocity. `AlignOrientation` replaces BodyGyro. `AlignPosition` replaces BodyPosition and has a `PerAxis` mode, so you can push on Y only. `VectorForce` replaces BodyForce [8].

**Buoyancy (floating): Terrain water vs part water.**
- In Terrain water, floating depends on density. Water is 1 RMU/stud³ (RMU is Roblox's mass unit) [3]. Plastic is 0.7 and WoodPlanks 0.35, so both float [4].
- On part water nothing floats by itself. DevForum methods:
  - Archimedes' formula: buoyant force = submerged volume × water density × gravity. Bare force makes the boat bounce, so you add a water drag term to stop the oscillation [15].
  - Sample the water height at 3 or 4 points on the hull, then drive one position mover and one orientation mover. Several separate position movers fight each other [16].
  - Build a Boat for Treasure uses a collision-filtered water part. Its water "only has collision with blocks", so boats rest on it and players sink to the sand [21].
- A *collision group* is a named set of parts. You choose which groups touch which. Parts in groups set to not collide pass through each other even with `CanCollide` on [9].

**Players riding the boat.**
- "Only objects moved by physics will carry other objects with them." Tweening or setting CFrame (moving a part by script) will not carry a standing player [17].
- Fling problems (the boat or the players getting thrown hard) show up when players walk around on a stabilized boat. The suggested fixes: more boat mass, a stronger orientation mover, replacing the deprecated BodyGyro with AlignOrientation, or massless characters. The thread never says which one worked [18].
- GPO notes players "flinging or drowning" on ships, and adds a P key that attaches a player to the boat [35].

**Network ownership (who runs the boat's physics).**
- By default the server owns parts. The engine hands unanchored parts to nearby clients automatically [7].
- The official script: when someone sits in the VehicleSeat, call `SetNetworkOwner(player)`. When they leave, call `SetNetworkOwnershipAuto()`. Also give the driver ownership of any loose parts on the boat [7].
- Security: "Roblox cannot verify physics calculations when a client has ownership." Clients can teleport or fly parts they own. `Touched` events (the event that fires when parts touch) on owned parts can be faked [7].

**Spawning, despawning, and stopping boat spam.**
- Tag each spawned vehicle with the owner's UserId through CollectionService (Roblox's tagging tool). Destroy the old one before spawning a new one. Destroy all of them on `PlayerRemoving`. Do not rely on names, which can be exploited [19].
- Keep spawned boats in their own folder. Look them up by owner, and destroy the old boat whatever its type [20].
- Blox Fruits: boats are spawned from dealer NPCs at docks for a set price. The free Dinghy is the starter [37][38]. Whether spawning a second boat removes the first: not confirmed.

### 3. Breakable boats

| Game | Boat HP / durability | How it takes damage | At zero / repair |
|---|---|---|---|
| Grand Piece Online | Rowboat 100 HP, free, 50 speed. Caravel 750 HP, 2,000 Peli. Galleon 2,000 HP, 15,000 Peli, 8 cannons [35] | Cannons, sea monsters. "Large amounts of damage every second" if the ship is on land [35] | Gets slower in proportion to the HP lost. Repair with a Hammer plus planks, 10 HP per plank. A Robux Golden Hammer repairs about 300% faster [35] |
| Blox Fruits | Dinghy 100 HP, 2 seats, free. Sloop 1,000 HP (Pirate) or 2,500 HP (Marine), 3 seats. Grand Brigade 2,500 HP, 7 seats [37] | Cannons (damage scales with player level). Sea events [37] | Orange HP bar above the hotbar, or above the ship when you are far away. "The boat will sink when the health hits 0." Repair needs the Shipwright unlock plus Wooden Planks [37] |
| SharkBite Classic | Durability rated Bad to Max (Titanic is Max, 20,000 teeth) [28] | Sharks ram boats. Explosives can fling them [28]. The shark wiki says a touch destroys a boat "almost instantly" [32] | Destroyed |
| SharkBite 2 | HP per boat not listed. Speeds run 80 to 145 SPS (studs per second). The Green Jetski has 2 seats, costs 800 teeth, does 105 SPS [29][30] | Shark ramming. In Pass the Bomb, a bomb destroys the boat on a timer [31] | Defend the Ship mode: the cargo ship has 100 HP and loses 2 per container knocked off [31] |
| Build a Boat for Treasure | Each block has its own HP: Wood 1, Huge Wheel 40. Shield Generators add 20 each [23] | Obstacles. In PvP, the Cannon does 6 damage to any block on a 10 s reload [24] | Blocks break one at a time, so the boat falls apart piece by piece [23] |

**PvP rules.** Build a Boat's PvP Mode is opt-in, and both players must turn it on. TNT and Dynamite cannot hurt players directly but can damage opponents' boats [25].

**Engine notes for explosions (artillery).**
- A default `Explosion` breaks welds and kills Humanoids, by breaking the neck joint, inside its radius. Roblox recommends setting `DestroyJointRadiusPercent = 0` and handling damage yourself in the `Hit` event [10].
- `BlastPressure` does not weaken with distance, so you have to code falloff yourself [10].
- Explosions are not blocked by walls. Parts behind cover still get hit [10].

**Sinking effects.** Blox Fruits boats sink at 0 HP [37]. I found no source with a standard sinking recipe. The recipe under "What Fruit Empire should take" is my own suggestion.

### 4. Boat game examples

- **Build a Boat for Treasure.** You build in a plot. "Launching" floods the plot and unanchors your blocks (sets them free to move), sending the boat downriver [26]. Water pushes boats forward [21]. Blocks welded together move as one, so a well-built 10,000-block ship moves about as fast as a 10-block one. Unseated players and loose blocks slow it down [27].
- **SharkBite 1 and 2.** One player is the shark and everyone else is on boats. Survivors get a 10 s head start [31]. The shark sees moving boats as blue propeller icons and swimmers as red rhombuses [31]. SharkBite 2 adds a boat builder: a Speed Boat hull is 145 SPS for 14,000 teeth, and the free hull is 95 SPS [33].
- **Blox Fruits and Grand Piece Online.** These are sea-travel RPGs. Boats are the main way across water, and swimming is slow or harmful [34][36].

### 5. What Fruit Empire's part-based sea means (from the project's own files)

- The outward current is 24 invisible `CurrentPlate` parts, each 300 × 0.2 × 670 studs. Their tops sit at y −3.8. Each pushes outward at 42 studs/s. The return lanes (`ReturnPlate`) push home at 48 studs/s [45][46].
- `Water.Ocean` is walk-through and cannot be hit by raycasts (`CanQuery` off). A downward ray over open sea hits `Seabed` at y −4.0, and the ocean surface is y −2.2 [46]. So the water is only about 1.8 studs deep. Players **wade**; they do not swim (worked out from those two numbers).
- "Am I in the sea?" is already answered by a 12-stud downward ray. If it lands on `CurrentPlate` or `Seabed`, you are at sea. A stepping stone, the dock, the beach or a boat deck is not sea [45].
- Distances: island beach edge r ≈ 380, last stepping stone r 642, forest beach (Terrain) r ≈ 994–1001 [45][46]. The current ride takes 14.8–15.2 s, and the return lane 14.3 s [46]. Walk speed is 20 studs/s [45].
- A distance-based "drown" fade already exists: the screen darkens, then the server kills you and you respawn at home. Its radii now sit beyond the forest wall (2600 / 2775 / 2825) [45].

## What Fruit Empire should take

These are proposals. The numbers are starting points to tune, not research findings.

**Swim fatigue (replacing the currents)**
1. **Reuse the existing sea test.** Drain stamina only while the 12-stud ray says "sea" [45]. Standing on a stone, the dock, the beach or a boat deck counts as dry, so stamina refills. A sunk boat's passengers drop straight into the stamina system with no extra code.
2. **Make the crossing possible but tiring.** The forest is where pets get tamed, so a brand-new player with 0 shards must still be able to reach it. Example tuning: swim speed 12 studs/s (walk is 20) and 100 stamina lasting about 35 s of swimming. The open water from the last stone (r 642) to the forest beach (r ~994) is about 352 studs, which takes about 29 s at 12 studs/s. A careful swimmer arrives with a little stamina left. A careless one, starting from the beach without resting on stones, does not. Add one or two rest islets if playtests show too many failures.
3. **Refill like Genshin:** a short delay (about 1.5 s) after leaving the water, then a fast refill [40]. Stepping stones become real rest stops, which matches how they already work as the way home [45].
4. **At zero, use the Zelda approach, not Minecraft's.** Slow the player to a crawl and start the existing vision fade. If they are still in the water after a few seconds, wash them up at home through the existing respawn [42][45]. Do not take away fruit or cash. Stealing was removed to keep the game non-punishing, and drowning should not become a new way to lose things.
5. **Stamina UI:** a small bar in Fruit Empire's UI kit, near the character or above the hotbar. Show it only while draining or refilling, the way Genshin and BABFT hide full bars [22][40]. Flash it red below 25%. Add a rising-pitch "bubble" tick as it empties, like Minecraft [43].
6. **Swim feel:** cheapest option first. Lower WalkSpeed and play a swim animation over the seabed. If a true swim pose is wanted, force the Swimming state and disable GettingUp on the client [13]. A deeper sea is a separate map job, since the water is only about 1.8 studs deep [46].
7. **Code rule:** scale all stamina changes by delta time on Heartbeat. Never loop a bare `task.wait()` [12].

**Aqua Affinity potion**
8. Make it a consumable sold for shards, cheaper than a boat. For scale, today's potions all cost shards: Speed, Jump, Invisibility and Feather Falling 5,000 each, Luck 7,500, Potion of Aether 10,000 [50]. Those prices are higher than the 1,000-shard boat, so an Aqua Affinity potion "cheaper than a boat" would be the cheapest potion in the shop by far. It doubles swim speed (about 24 studs/s) and cuts stamina drain to 0–25% for its duration. Give it the same timer UI as the Potion of Aether. Optional rule from GPO's Bubble: in PvP areas, it ends early if you take a big hit [34].

**Boats (1000 shards, 2 seats)**
9. **Buy once, summon many times.** Save ownership to the player's data. Put summon pads at the island dock and the forest beach, like Blox Fruits' dock dealers [38].
10. **Seats:** one VehicleSeat (owner only) and one Seat. Fill the passenger seat through a "Ride with <name>" ProximityPrompt or an invite, the way SharkBite 2 does it [31]. Strangers cannot jump in and take control.
11. **Movement:** read `ThrottleFloat` and `SteerFloat` [6]. Drive a `LinearVelocity` in `Plane` mode for thrust, `AngularVelocity` or `AlignOrientation` for turning and staying upright, and `AlignPosition` in PerAxis mode holding Y at the waterline [8]. The sea is flat, so this beats a real Archimedes calculation [15][16].
12. **Floating backup:** add an invisible "SeaSurface" part in a `BoatWater` collision group that only collides with `Boats`. This is BABFT's trick [21]. Players still pass through to the seabed [9].
13. **Deal with the current plates.** Delete them when boats ship (the owner asked for the currents to go). If any stay, put them in a group that boats do not touch. Otherwise the 42 studs/s conveyor drags every boat [5][9][46].
14. **Move boats only with physics**, never by tweening CFrame, so riders move with the boat [17]. Keep riders seated.
15. **Network ownership:** use the official driver-ownership script [7]. Check on the server every 0.2 s or so. If a boat's speed is above the max (for example 1.5× the top speed) or it is outside the sea ring, snap it back or despawn it [7].
16. **Spam rules:** one boat per player, tracked by an `OwnerUserId` tag. A new summon destroys the old boat. All of a player's boats are destroyed on `PlayerRemoving` [19][20]. Also: summon only on a clear dock pad, a 15 s summon cooldown, despawn after 60 s with nobody aboard, and a per-server cap.
17. **Speed target:** 42–48 studs/s, the same as today's currents [45], so the crossing still takes about 15 s. SharkBite 2's fast boats run 105–145 SPS on a much bigger map [29][33].

**Breakable boats (artillery)**
18. **One HP number with a health-drain bar**, not block-by-block like BABFT. This matches the "drain health, don't delete" model the owner wants for artillery. Show it above the hotbar while driving and above the boat for others, like Blox Fruits [37]. Start around 500 HP and scale artillery damage by tier.
19. **Slow the boat as HP drops,** like GPO, so damage is felt before the boat sinks [35].
20. **Artillery:** spawn `Explosion` with `DestroyJointRadiusPercent = 0`. Do damage in `Hit` on the server, with your own distance falloff. Add a raycast if cover should block it. The default settings would kill passengers and break the boat's welds [10]. Never take damage from a client-side `Touched` [7].
21. **At 0 HP (my suggestion):** turn off the float, tilt the boat and sink it over about 2 s with splash and bubble particles, then despawn. Riders drop into the water and the stamina system takes over. Since the boat is a permanent purchase, use a respawn cooldown (for example 60 s) instead of re-buying. A shard repair fee is optional.
22. **Who can break a boat:** stealing was removed so players cannot grief each other's progress. So limit boat damage to opt-in PvP, duel zones, or the environment (sea mines, rock cannons). BABFT's both-players-opt-in rule is the model [25].

## What to avoid

- Relying on the Swimming state, `BuoyancySensor`, or density to float. None of them work without Terrain water [1][2][3].
- Moving boats by tween or CFrame. Riders slide off or get left behind [17].
- Leaving current plates and boats able to touch. The conveyor will drag boats [5].
- Default explosions on boats. They kill Humanoids and break welds [10].
- Server damage based on `Touched` from a client-owned boat. It can be faked [7].
- Frame-rate-based stamina code, which drains faster live than in Studio [12].
- Making a boat the only way to reach the forest. With 0 shards, a new player would be locked out of pet taming.
- Unlimited boat summons, or boats left behind after the owner leaves [19][20].
- Drowning that takes away fruit or cash. It goes against the no-stealing decision.
- Boats ramming or flinging swimmers. Fling problems are common on Roblox boats [18][35]. Test boat-vs-swimmer collisions and consider making boats not push characters in the water.

## Open questions

- Is there any Terrain water in the place? Old project notes mention a "terrain thin film Y0" near the fishing water. If players touch Terrain water anywhere, the Humanoid will enter the real Swimming state there.
- ~~How long does a new player take to earn 1000 shards?~~ Partly answered from the local code [48][49]. The Index pays 5 to 2,500 shards per fruit, and a full Common page pays only 31. With the planned rarity gating and today's rebirth rule (the Index is paid again at each rebirth), a player passes 1,000 Index shards only around rebirth 3 (about 1,248). Decree and Crown rewards can add more (a Merge Rush decree pays 400, at most once per 6 minutes). So swimming has to be the main early route to the forest, as proposals 2 and "What to avoid" already say. Real play time to 1,000 shards is not confirmed and needs a playtest.
- Should boats be damageable by other players outside duels and opt-in PvP? This is a call for the owner, given the no-stealing rule.
- What happens to a fruit someone is carrying when they run out of stamina or their boat sinks?
- Do the stepping stones stay, as studded stones, as the swimmer's rest stops? Their positions set the stamina math.
- Aqua Affinity price and duration. Can it stack with the Potion of Aether?
- After a boat is destroyed: cooldown only, or a shard repair cost like GPO's planks [35]?
- Not confirmed in this research: SharkBite 2 per-boat HP values, Fisch's base breath time without gear, Genshin's exact zero-stamina penalty, and whether Blox Fruits removes your old boat when you summon a new one.

## Sources

1. https://create.roblox.com/docs/reference/engine/enums/HumanoidStateType
2. https://create.roblox.com/docs/reference/engine/classes/BuoyancySensor
3. https://create.roblox.com/docs/physics/units
4. https://create.roblox.com/docs/parts/materials
5. https://create.roblox.com/docs/reference/engine/classes/BasePart (Velocity: "Setting the Velocity of a part that is Anchored will cause it to act like a conveyor belt")
6. https://create.roblox.com/docs/reference/engine/classes/VehicleSeat
7. https://create.roblox.com/docs/physics/network-ownership
8. https://create.roblox.com/docs/physics/mover-constraints
9. https://create.roblox.com/docs/workspace/collisions
10. https://create.roblox.com/docs/reference/engine/classes/Explosion
11. https://devforum.roblox.com/t/how-would-you-make-a-drowning-system/1491997
12. https://devforum.roblox.com/t/stamina-drains-and-restores-much-faster-in-game/3614380
13. https://devforum.roblox.com/t/making-a-character-swim-outside-of-terrain-water/951412
14. https://devforum.roblox.com/t/how-to-create-terrain-water-behavior-without-terrain-water/579536
15. https://devforum.roblox.com/t/calculating-custom-buoyancy/240104
16. https://devforum.roblox.com/t/custom-water-ships/1387577
17. https://devforum.roblox.com/t/moving-platforms-working-but-not-moving-the-player/797190
18. https://devforum.roblox.com/t/stop-players-being-able-to-fling-vehicle/2694969
19. https://devforum.roblox.com/t/vehicle-not-despawning/2062691
20. https://devforum.roblox.com/t/help-with-spawning-a-model/2090456
21. https://build-a-boat-for-treasure.fandom.com/wiki/Water
22. https://build-a-boat-for-treasure.fandom.com/wiki/Health
23. https://build-a-boat-for-treasure.fandom.com/wiki/Blocks
24. https://build-a-boat-for-treasure.fandom.com/wiki/Cannon
25. https://build-a-boat-for-treasure.fandom.com/wiki/PVP_Mode
26. https://build-a-boat-for-treasure.fandom.com/wiki/Launching
27. https://build-a-boat-for-treasure.fandom.com/wiki/Boats
28. https://sharkbite.wiki/wiki/SharkBite_Classic:Boats
29. https://sharkbite.wiki/wiki/SharkBite_2:Boats
30. https://sharkbite.wiki/wiki/SharkBite_2:Green_Jetski
31. https://sharkbite-rblx.fandom.com/wiki/SharkBite_2
32. https://sharkbite-rblx.fandom.com/wiki/Sharks
33. https://sharkbite.wiki/wiki/SharkBite_2:Boat_builder
34. https://grand-piece-online.fandom.com/wiki/Water
35. https://grand-piece-online.fandom.com/wiki/Ships
36. https://blox-fruits.fandom.com/wiki/Water
37. https://blox-fruits.fandom.com/wiki/Boats
38. https://blox-fruits.fandom.com/wiki/Boat_Dealer
39. https://fischipedia.org/wiki/Oxygen
40. https://genshin-impact.fandom.com/wiki/Stamina
41. https://genshin-impact.fandom.com/wiki/Swimming
42. https://zelda.fandom.com/wiki/Stamina_Wheel
43. https://minecraft.wiki/w/Drowning
44. https://minecraft.wiki/w/Aqua_Affinity
45. Project file: ~/fruit-empire/src-live-2026-10-03/ReplicatedStorage.GardenConfig.lua (sea and current block around lines 1324–1390, forest distances around 1509–1525, drown radii 1619–1645, WalkSpeed line 989)
46. Project file: ~/fruit-empire/NOTES.md (lines 90, 111, 140, 1539, 1566, 1588)
47. Owner request: Fruit Empire/01 Vision/Brain Dump 2026-10-04.md, line 23
48. Local code: `~/fruit-empire/src-export-2026-10-04/ReplicatedStorage.IndexRewards.lua` (`Rewards.Shards`) and `ServerScriptService.GardenService.RebirthLock.lua` (rebirth re-pays Index shards)
49. Local code: `~/fruit-empire/src-export-2026-10-04/ReplicatedStorage.CrownConfig.lua` (`C.DecreeCooldown = 360`, Merge Rush `reward = 400`, `C.Rewards`)
50. Local code: `~/fruit-empire/src-export-2026-10-04/ReplicatedStorage.GardenConfig.lua` (`Config.Tools`, potion prices and durations, about lines 1616–1705)

## Review (critic pass, 2026-10-04)

- **Verified:** VehicleSeat `Throttle` and `Steer` are deprecated and NotReplicated in the docs source [6]. The sea numbers still hold in the 2026-10-04 export: `CurrentSpeed = 42` (line 1310), `ReturnSpeed = 48` (line 1319), drown fade 2600 / 2775 (lines 1556–1557), `WalkSpeed = 20` (line 941). The line numbers in [45] are from the older src-live copy.
- **Added:** a shard-supply answer to the "1000 shards" question [48][49].
- **Checked, still open:** no script in the 2026-10-04 export fills Terrain with water. `ForestBuilder` uses Terrain only for the sand shelf. Whether the place file itself has Terrain water is not confirmed.
- **Still not confirmed:** SharkBite 2 boat HP, Fisch's base breath time, Genshin's zero-stamina penalty, and whether Blox Fruits removes an old boat on a new summon.
