---
tags: [fruit-empire, project, animals, models]
status: built
updated: 2026-11-08
code: ~/fruit-empire-pets/forest-animals
handoffs: [7, 8]
---
# Forest animals (the models)

The 45 animal models the pets system and the Troll Potion use. How they behave is [[Pets, Eggs and Fuse Machine]]. Back to [[Project Status Board]].

## What exists

- **45 animals, 5 per rarity, each as an adult, a baby and an egg: 135 models.** Each animal carries a fruit from its own tier of the ladder. The first nine: Berryhog (Common, blueberry hedgehog), Lollisnail (Uncommon, lollipop snail), Capsnout (Rare, mushroom pig), Grapetail (Epic, grape squirrel), Crownmane (Legendary, pineapple lion), Pepperatops (Mythic, chili-pepper triceratops), Lumoose (Astral, glowshroom moose), Moonmochi (Eternal, moon rabbit), Soulmane (Transcendent, qilin with a rainbow halo).
- Adults are 3.6 to 9.3 studs tall, babies up to 3.42, eggs up to 4.2; 424 to 6,284 triangles; studded voxels with vertex colours, no textures.

Source: memory `project_fruit_empire_forest_animals.md`.

## Samarth's idea (18 Oct)

Populate the forest with animals; players bonk them with the bat; animals drop eggs; players take the eggs and hatch a baby that gives a boost. He wanted the models first, then the rules. Steal an Egg is a style reference for the eggs only. He asked for "at least 4-5 per rarity" on 19 Oct. (memory `project_fruit_empire_forest_animals.md`)

## Where it lives

- **In the place:** `ServerStorage.ForestAnimals.Adults`, `.Babies`, `.Eggs` (45 each), with attributes `Animal`, `Stage`, `Tier`, `TierIndex`, `File`. **The display:** `Workspace["baseplate slop"].ForestAnimalsLineup`, all 45 in rows by rarity, rebuilt by `lineup_builder.luau`. Samarth: "keep the display, it's under the map so it's fine". (PLACE.md; STATE.md "20 to 21 Oct")
- **Local:** `~/fruit-empire-pets/forest-animals` (specs per model are the source of truth; `import/` GLBs; sheets; `tools/`). Do not re-run `tools/make_final_ORIGINAL_do_not_rerun.py`. Import copies in `~/Downloads/forest-animals`; rigged versions for the pets system in `~/Downloads/forest-animals-rigged`.
- **How they are made:** `~/pet-pipeline-test/voxel_builder.py`, then `roblox_vertex_colours.py` (linear to sRGB) before Import 3D. See [[Art Style - Studded Voxels]].
- **Handoffs:** [PR #7](https://github.com/svaka2000/fruit-empire-handoffs/pull/7) (first 27), [PR #8](https://github.com/svaka2000/fruit-empire-handoffs/pull/8) (all 135 and the display).

## Calls made

- The faceted "Voidhorn Behemoth" (13,310 triangles) is not one of the 45; suggested as a separate "Grove Guardian" set piece.
- Glow is PointLights and particles outside the mesh.
- A bonk is a cartoon startle, never injury (Roblox bans animal abuse).

## Open taste calls for Samarth

Waxberry's mask looks like sunglasses; Tardigrand reads pig-like; Dewbelly could read as a tick. Flying animals need a lift and bob script (the pets build's `PetAnimator` covers fliers). (memory `project_fruit_empire_forest_animals.md`, `project_fruit_empire_pets.md`)

Related: [[Pets, Eggs and Fuse Machine]], [[Troll Potion]], [[Mine Digging and Cave-in]] (mine eggs are copies of these eggs).
