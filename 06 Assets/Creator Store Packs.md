---
tags: [assets, creator-store]
date: 2026-10-04
---
# Creator Store packs (linked by Samarth, 4 Oct 2026)

All seven were loaded with `game:GetObjects` into `ServerStorage.AssetQuarantine`, one folder each, named `Q_<asset id>`. Each folder carries a `SourceAsset` attribute. **None of the seven contains a script**, so there was nothing to strip. Copies for looking at sit in `Workspace._AssetViewer`, at (6000, 1000, 6000), far from the map. That model is temporary and gets deleted before he saves.

Source of the request: [[Brain Dump 2026-10-04]].

| Pack | Store id | Creator | What is inside | Planned use |
|---|---|---|---|---|
| Stud-Themed Animal Pack [LITE] | 97050466582003 | DropKit | 6 rigged animals, 233 parts, 227 Motor6D joints, an AnimationController on each. **No animations.** | Forest pets ([[Pets]]) |
| Statue With Sword | 4697087599 | RehakrTV | One studded statue holding a sword, 8 parts, with a `Fire` effect | Decor: Royal Capital, arena entrance, plot statues |
| Sword Animations | 77935648543779 | Fancy Cat Games | 6 R15 dummies with katanas, each holding one `KeyframeSequence` (raw animation data). 521 keyframes. | Royal duel sword: idle, walk, run, block, slash combo |
| Stud Shop | 12340318191 | p_0kx | A studded shop booth with a shopkeeper NPC, 30 parts | Boat dock shop, potion stall, pet tamer stall |
| Stud Bridge | 71814553242028 | Moarter2 | A small studded plank bridge, 11 parts | Bridges over the shallows and the forest river |
| Studded Pack | 86601833587701 | DustyBlueSkys | 11 models, 538 parts, 672 stud `Texture`s: House, 3 studded trees, 3 basic trees, Tent, CampFire, **Logs**, Mushroom | **Logs** replace every round log. Trees, tent and campfire for the forest and the pet area. |
| Studed Mountain | 105342635739603 | alecdixi | One stepped mountain, 768 Parts with Studs on top, grey (Smoky grey + Fossil) | Replaces the border mountains. Recolour to match the island. |

## Screenshots
![[pack-animals.jpg]]
*The six animals. Left to right: Deer, Turtle, Sheep, Hyena, Giraffe, Basilisk (Mythic).*

![[pack-sword-animations.jpg]]
*The six sword animation rigs, each holding a katana.*

![[pack-statue-with-sword.jpg]]
![[pack-stud-shop.jpg]]
![[pack-stud-bridge.jpg]]
![[pack-studded-pack.jpg]]
![[pack-studded-logs-closeup.jpg]]
*The studded logs: square, studded sides, light end grain. One long log and two short ones.*

![[pack-studded-mountain.jpg]]

## The animals, in detail
| Model | Size (studs, X×Y×Z) | Joints | Notes |
|---|---|---|---|
| Sheep | 2.5 × 3.1 × 4.4 | 21 | Wool blocks are separate parts, so wool can grow with merges |
| Deer | 1.4 × 5.2 × 4.0 | 32 | Upper and lower legs plus hooves: the best rig for a walk cycle |
| Turtle | 2.5 × 1.8 × 4.0 | 22 | Its PrimaryPart is `Shell`, not `Body` |
| Hyena | 1.2 × 4.1 × 4.3 | 36 | Crest parts along the back |
| Giraffe | 2.5 × 13.0 × 8.7 | 31 | Tall: 13 studs. It needs a scale-down or a big enclosure |
| Basilisk (Mythic) | 2.4 × 1.4 × 5.4 | 85 | A serpent made of 14+ segments (Seg0…Seg13); its creator already labels it Mythic |

The store page says the animals are "already rigged to be animated as you like". The pack has no Animation objects. So the pets need movement from one of two sources:
1. **Procedural animation (recommended for v1).** Code swings each leg's Motor6D (`Transform`) on a sine wave tied to walking speed. The serpent wave works the same way, joint by joint. Nothing has to be published, and blocky animals look right with it.
2. Samarth animates them in Studio's Animation Editor and publishes each one. That gives better quality but costs him hours per animal.

The description says 5 animals; the file has 6.

### Rig check: where the joints pivot (measured 4 Oct 2026)
A Motor6D joint turns its part around the joint's origin. For a walk cycle, a leg must turn at its top (the hip), not its middle.

| Animal | Joints | Pivot at the part's end (hip-style) | Pivot at the part's centre |
|---|---|---|---|
| Sheep | 21 | 10 (all four legs) | 4 |
| Turtle | 22 | 16 (all four legs) | 4 |
| Giraffe | 31 | 18 (all four legs) | 11 |
| Deer | 32 | 1 | 31, **legs included** |
| Hyena | 36 | 0 | 36, **legs included** |
| Basilisk | 85 | 0 | 85 (every segment) |

**Fix for Deer, Hyena and Basilisk (no visual change):** re-pivot each joint once when the model loads. Keep the rest pose by keeping `C0 * C1⁻¹` the same:
- `C1' = CFrame.new(0, part.Size.Y / 2, 0) * C1.Rotation` (move the origin to the top of Part1);
- `C0' = C0 * C1:Inverse() * C1'`.

After that, `Motor6D.Transform` rotations swing the leg from the hip. Checked numerically on the Deer's front-left leg: the rest pose stays exactly the same (0.00000 studs off). With the old rig, a 30° swing moves the top of the leg 0.225 studs off the body; with the new rig, 0.00000. For the Basilisk, put the origin at each segment's front face along its length instead (the axis the body runs along), so the wave bends at the joints between segments. All six models have an anchored PrimaryPart (`Body`, or `Shell` for the Turtle). The client mover must unanchor the joints' parts, or keep the PrimaryPart anchored and move it by CFrame.



### How the studded `Logs` are built (for the log-swap builder)
- 18 Parts, three logs (one long, two short). Each log is:
  - a **body**: Plastic, `Earth yellow`, cross-section 2 × 1, 2 or 6 long;
  - an **end-grain cap**: Plastic, `Sand yellow`, 1.5 × 0.5;
  - thin **bark strips**: SmoothPlastic, `Earth yellow`, 0.25–0.5 thick.
- Every part carries 6 `Texture`s (texture 1444886499, the studs tile) at **8 × 8 studs per tile**. A Texture tiles by studs, not by part size, so a body can be **resized to any length and the studs stay the same size**. A builder can therefore make a studded log of exactly the length and angle of each round log it replaces (inventory: `~/fruit-empire/world/round-logs-2026-10-04.json`) instead of stretching one model.
- The round logs are about 2.1–2.7 thick and 10–47 long, so the body cross-section scales up to about 2.5 and the caps and strips scale with it.

## The sword animations, in detail
| KeyframeSequence | Length | Loops | Priority |
|---|---|---|---|
| Sword Idle 1 | 3.00 s | yes | Idle |
| Sword Idle 2 | 3.00 s | yes | Action |
| Sword Walk | 0.93 s | yes | Movement |
| Sword Run | 0.67 s | yes | Movement |
| block | single pose | yes | Action |
| swordcombo1 | 2.82 s | yes | Idle |

A `KeyframeSequence` is the editor's raw data, not a playable animation. In a live game an `Animator` can only play an uploaded Animation asset. So these must be **published by Samarth**, under the account or group that owns the game: open each in the Animation Editor, then File → Publish to Roblox. That gives six animation ids. The combo also needs split points: three slashes as separate clips, or keyframe markers, so the game can deal damage on each hit (see [[Royal Combat]]). The rigs are **R15**. The duel's armour dummies in `CrownArena.lua` are R6 (`hum.RigType = Enum.HumanoidRigType.R6`), and `KingClient.lua` maps both rig types. **Checked in a playtest: player characters are R15** (16 parts, the default `Animate` script, HipHeight 2.01), so the R15 sword animations fit players directly. Only the duel's armour knights would need R15 rigs too.


### Where the hits are in `swordcombo1` (measured 4 Oct 2026)
The combo is one 2.82 s clip with 140 keyframes. I summed how fast the sword arm turns (RightUpperArm, RightLowerArm, RightHand and UpperTorso, in degrees per second between keyframes) and smoothed it. It shows **4 slashes**:

| Slash | Fastest moment (the hit) | Peak arm speed | Clip from → to (split at the still moments) |
|---|---|---|---|
| 1 | 0.38 s | 3,147 °/s | 0.00 → 0.80 s |
| 2 | 0.98 s | 3,407 °/s | 0.80 → 1.25 s |
| 3 | ~1.75 s | 2,168 °/s | 1.25 → 1.95 s |
| 4 | 2.35 s | 2,166 °/s | 1.95 → 2.82 s |

The median arm speed is 579 °/s, and the still moments between slashes drop to about 100–120 °/s (0.8, 1.9, 2.6 s).

**What to do in the Animation Editor before publishing:** add a keyframe marker named `Hit` at 0.38, 0.98, 1.75 and 2.35 s, and set the clip to not loop. The game then deals damage on `AnimationTrack:GetMarkerReachedSignal("Hit")` instead of on a timer. To make each slash its own clip (so a player can stop after one or two), export the 4 ranges above as 4 animations instead. The rigs are R15.

## Round logs and lily pads today
- **Round logs** are MeshParts from the old free pack `Workspace["Asset Pack V1"]`: models `Log1`, `Log2`, `VinesLog1`, `VinesLog2` and the MeshPart `HollowLog`. They are scattered through:
  - `Main Island.Forest.Undergrowth.Sector_000…345` (about 600 log parts);
  - `Main Island.Shallows.Driftwood` (423 log parts);
  - `Main Island.CoastDecor2` (37);
  - `Main Island.OuterRingV1` (22).
- **Studded logs:** the only ones in the game are the `Logs` model in the Studded Pack above (texture `rbxassetid://1444886499`, the studs tile). Nothing else in Workspace or ServerStorage is a studded log. So "the studded logs you already have" means this pack. → [[09 Decisions/Decisions Log|decision D-003]]
- **Lily pads** are the MeshParts `Lilypad1` (3.3 × 0.33 × 3.8) and `Lilypad2` from the same old pack. A MeshPart cannot take Studs surfaces. Making them studded means either a stud `Texture` on top, or rebuilding each pad from a flat Part with Studs on top plus a notch. The second matches the other studded decor better.
- The game already has four MaterialVariants: `Studs` (Plastic), `ForestStuds` (LeafyGrass), `BeachStuds` (Sandstone), `RidgeStuds` (Slate).

## Licence notes
- All seven are free Creator Store models. Roblox's terms let any experience use free store models.
- Statue With Sword's description says "All Rights Reserved". That is the creator's text on a model they released free on the store. Using it inside Roblox is what the store is for. Don't re-upload it as our own asset.
- Stud Shop says "Don't works need SCRIPTS": it is decor only. Our own code drives any shop.
- The Studded Pack campfire's `Fire Crackling Sound` (rbxassetid 9068935533) fails to load: "Asset is not approved for the requester". It is someone else's private audio. Replace it with a fire sound from Roblox's licensed library before using the campfire.
