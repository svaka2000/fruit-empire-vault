---
tags: [fruit-empire, art, models, style]
status: current
updated: 2026-11-08
---
# Art style: studded voxels

Every Fruit Empire model is blocky and studded, like Lego. Detail goes into shapes and shading, never into smooth surfaces. Words used here (voxel, MaterialVariant, Neon, Bloom) are in [[Fruit Empire Glossary]]. Back to [[Knowledge Base - Start Here]].

> [!important] Detailed but blocky, always studded
> "Go overboard" on detail, but keep the studs. "Realistic" means detailed (corner stones, framed windows, corbels, tiled roofs, real timber joints), never smooth.
> Sources: memory `feedback_game_models_go_overboard.md`; `~/fruit-empire-plot-upgrades/src/PlotPenDecor.luau` header; `~/fruit-empire-mine/CAVE-OVERHAUL.md` section 2.

## The material

- **Parts:** Plastic with the place's MaterialVariant **"Studs"**. (memory `reference_glb_to_roblox_parts.md`)
- **Imported meshes:** MeshPart `Color` white, Plastic + "Studs", colours carried in the mesh's vertex colours. (memory `project_fruit_empire_forest_animals.md`)
- **Terrain can be studded too:** a clone of a Studs variant with BaseMaterial LeafyGrass, set with `MaterialService:SetBaseMaterialOverride`. (memory `reference_roblox_studio_mcp.md`, 29 Sep)
- **Plants and rocks** come from the place's packs (Asset Pack V1, Studded Pack, Stud Shop, Stud Bridge, Studed Mountain), never built from plain parts. ([PLACE.md](https://github.com/svaka2000/fruit-empire-handoffs/blob/main/PLACE.md) "Other rules")

## How models are built

- **Voxels about 0.2 studs wide**, one stud per block face, like the pickaxes and ores. Detail comes from the shape and from per-block shading: lit and dark sides, gradients, highlight corners, glowing bits, variants, damage states, a break kit. (memory `feedback_game_models_go_overboard.md`)
- **Voxel builder:** `~/pet-pipeline-test/voxel_builder.py` turns JSON blocks into one mesh with vertex colours (COLOR_0) and box UVs, exported as GLB and FBX, with Blender preview renders. The mine's builder (`~/fruit-empire-mine/models/tools/build.py`) makes separate Neon "Glow" pieces. (memory `project_fruit_empire_forest_animals.md`, `project_fruit_empire_mine.md`)
- **A user's GLB into plain studded parts:** `~/fruit-empire/tools-fuse/convert.py` and `make_builder.py`, used for the Aether Fuse Machine (538 parts). (memory `reference_glb_to_roblox_parts.md`)
- **Show it moving.** A Blender clip of the model in motion helps Samarth judge it. (memory `feedback_game_models_go_overboard.md`)

## Import traps

- **Vertex colours come in too dark and red** unless converted first: a GLB's COLOR_0 is linear, Roblox reads it as sRGB. Convert with `~/pet-pipeline-test/roblox_vertex_colours.py`. (memory `reference_glb_to_roblox_parts.md`, `project_fruit_empire_forest_animals.md`)
- The importer wraps each piece in a `<piece>_Node` model, puts the pivot at the file's origin, and turns the model 180 degrees (the file's +Z front becomes Roblox's front). Scripts that find pieces by name need them lifted out of `_Node`. (memory `project_fruit_empire_mine.md`, 21 Oct)
- Import 3D with "Studio Default" uses stud units; models arrive under Workspace "Scene"; each file is also uploaded to his inventory. (memory `project_fruit_empire_forest_animals.md`)

## Glow and the place's light

> [!warning] Neon washes out under the bloom
> The place's light is strong: Lighting Brightness 4, Bloom Intensity 1, Size 56, Threshold 2. A big Neon face hides detail in front of it, and pale parts bloom to white.

- Keep Neon for small accents. Make big glowing faces studded Plastic or SmoothPlastic instead. (memory `reference_glb_to_roblox_parts.md`)
- The bats' fix: Handle colour 85% grey (it multiplies the vertex colours) and Neon at about 62% of its colour, in the builder's LOOK table. (memory `project_fruit_empire_bats.md`)
- Purple Neon reads hot pink (#9E52E0 on the Amethyst Pick, #7C3ECC on the ores). Samarth kept it. A darker #6024A8 was suggested. (memory `project_fruit_empire_mine.md`)
- For crystals in the dark mine, a near-white PointLight over the crystals worked better than more Neon: full Neon glared, dimmed Neon lost its studs. (memory `project_fruit_empire_mine.md`, PR #14)
- Glow on animals is PointLights and particles outside the mesh. (memory `project_fruit_empire_forest_animals.md`)
- The place's ColorCorrection makes creams look white and navy look royal blue; the vertex colours are right. (memory `project_fruit_empire_forest_animals.md`)
- Only a Play test shows the real look: Edit mode lights differently. (memory `reference_glb_to_roblox_parts.md`)

## Size and cost budgets seen so far

- Forest animals: 424 to 6,284 triangles; adults up to 9.3 studs, babies up to 3.42, eggs up to 4.2. A 13,310-triangle "Voidhorn Behemoth" was kept out of the set. (memory `project_fruit_empire_forest_animals.md`)
- A level 5 Fruit Castle pen: 725 parts; 12 castles cost about +1 ms render time. (memory `project_fruit_empire_plot_castle.md`)
- The v3 mine: 3,251 parts and 59 lights. (memory `project_fruit_empire_mine.md`)

Related: [[Colour Palettes]], [[Light, Day and Sound]], [[Samarth's Taste]].
