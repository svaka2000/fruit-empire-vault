---
tags: [assets, vfx]
date: 2026-10-04
---
# VFX and props already in the place

These are already in the place file and contain no scripts. Most sit in a staging area under the map, around y = −790, so players never see them. They are stock for the expansion's combat and effects. The planned uses below are my proposals.

## The bat (`Workspace.Bat`, a Tool)
- Handle 0.4 × 0.4 × 4.2. Classic bat mesh `54983181` with texture `54983107`, mesh scale (3, 3, 1.5). Grip −1.7 on Z.
- Sounds: `Swoosh` 3755636638, `Hit` 175024455, `Hit2` 3932505023, `Ouch` 4306991691, `Ouch2` 4459572527, `Equip` 769464514.
- One `Trail` with a 0.10 s lifetime.
- It sits loose in Workspace at (18, 157, 227), not in `ReplicatedStorage.Tools`, so the game does not hand it out.
- **Planned use:** the base of the rebirth bat ([[Bats]]). A `SpecialMesh.VertexColor` tints the mesh's texture, so each rarity colour can tint the same mesh. The trail takes the same colour.

## Effect packs (staging area, y ≈ −790)
| Model | Contents | Planned use |
|---|---|---|
| `Anime` | 14 effects, 65 emitters: Wind-02, Blood-01, Water-02, Smoke-01, Lighting-01/02, ForceField-Break-01, Shiny-01, Shield-Break-01, **Slash-01**, Portal-01, Stars-01, Water-01, Shield-01 | Sword slash arcs, block/parry flash (Shield-01, Shield-Break-01), boat splash (Water-01/02) |
| `Big` | Explosion-01, Tornado-01, Ball-01, **Big-Crack-01**, Lighting-01 (25 emitters) | Royal Slam ground crack, nuke blast |
| `vfx pack` | `Explosion` (20 emitters: shockwave, shards, sparks, stars), `Projectile` (smoke + spark trail), `Part` (speed lines), `f` (18 emitters: crescents, rocks, impact, flash) | TNT blast, artillery shell trail, sword impact |
| `Beams` | Lava-01, Wind-01, Waterfall-01 (9 Beams, 12 emitters) | Underground lava/ore glow, waterfalls |
| `Auras` | RNG-Aura-02, RNG-Aura-03 (on dummies) | Pet aura for top-tier merged pets |
| `Heads` | Six face/eye effects (Surprised, Angry, Confused, Glowing-Eyes, Flaming-Eyes) | Pet reactions when tamed or fed |
| `Particle Storage` | 35 plain Parts | Spare holders |

## Other loose models in Workspace
- `Stud Sign`: a 2-part sign with a SurfaceGui, at (−155, 2.8, −44.5).
- `baseplate slop`: one textured Baseplate at y = −808 (under everything).
- `SEASONAL DECOR` (713 parts): holographic trees, lamp posts. It holds the only 4 scripts in Workspace (lamp physics, hologram spin, 84–144 characters each).
- `Asset Pack V1` (554 parts): the old nature pack, the source of the round logs and mesh lily pads (see [[Creator Store Packs]]).
