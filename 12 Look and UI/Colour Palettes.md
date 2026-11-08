---
tags: [fruit-empire, colour, palette, reference]
status: current
updated: 2026-11-08
---
# Colour palettes

The colours the game uses, copied from the scripts that hold them, with the date of the copy. If a builder changes, its file is the truth. Back to [[Knowledge Base - Start Here]].

## Rarity colours

`Config.RarityColors` in `ReplicatedStorage.GardenConfig` (29 Oct export: `~/fruit-empire-pets/ref/scripts/ReplicatedStorage.GardenConfig.lua`, lines 622-632). The order is `Config.RarityOrder`.

| Rarity | RGB |
|---|---|
| COMMON | 151, 169, 189 |
| UNCOMMON | 78, 194, 101 |
| RARE | 78, 164, 230 |
| EPIC | 170, 93, 222 |
| LEGENDARY | 242, 171, 48 |
| MYTHIC | 232, 66, 60 |
| ASTRAL | 71, 222, 217 |
| ETERNAL | 237, 81, 195 |
| TRANSCENDENT | 252, 247, 227 |

## UI text colours

`Kit.Colors` in `ReplicatedStorage.UIKit`: White 255, 255, 255; Red 226, 52, 52 (unmet need); Green 86, 200, 96 (met need); Ink 0, 0, 0 (outline); Gold 255, 202, 58 (bar fills only). See [[UI Kit and Text Rules]].

## The mine: rock, timber, iron

From `MineBuilder` v3, 22 Oct 2026 (`~/fruit-empire-mine/cave/v3/ServerStorage.ZZTools.MineBuilder.lua`, lines 104-134). These are the colours *before* `ServerStorage.SaturationPass` lifts them. The woods are the plot kit's own, so the mine's timber matches the plot gates.

| Use | Name | RGB |
|---|---|---|
| Rock | Rock / RockDark / RockLight | 113, 115, 135 / 89, 91, 108 / 138, 139, 158 |
| Floor | Floor / FloorDark | 110, 102, 96 / 90, 86, 86 |
| Timber | Wood / Wood2 / Wood3 / Wood4 | 124, 92, 70 / 136, 101, 76 / 112, 82, 62 / 129, 98, 64 |
| Timber | WoodDark / EndGrain | 88, 61, 45 / 74, 51, 38 |
| Iron | Metal / MetalDark / Rust / Bolt | 84, 88, 100 / 54, 56, 66 / 118, 72, 50 / 42, 44, 52 |
| Stone footings | Stone | 106, 106, 116 |
| Rope | Rope | 150, 122, 88 |
| Shown exactly (glow, water) | Gold / Lamp / Water / Shaft light / Sign text | 222, 152, 38 / 226, 168, 58 / 50, 158, 222 / 170, 210, 255 / 255, 236, 196 |

- v2 rock was darker (Rock 64, 66, 84). v3 lifted it a step so rock out of a lantern's reach does not sink into black.
- **Cavern glows** (`MineBuilder.Caverns`): rift glow 255, 176, 72; crystal glow purple 176, 120, 255 or teal 80, 230, 220; pool light 110, 180, 255; mushroom glow 90, 240, 220; roof crack light 170, 210, 255.
- **Ore glow:** #7C3ECC (geode floor #5E2FA6). Reads pink in the mine's light; Samarth kept it. (memory `project_fruit_empire_mine.md`)
- **The safe pockets** being built on 8 Nov use their own rock (97, 100, 129 and three neighbours) and wood (107, 68, 41 and up) tables. (`~/fruit-empire-mine/pockets/MinePocketBuilder.lua`, work in progress)
- **The mine's light:** `Config.Mine` fill 112, 114, 130, Exposure 0.35, and a mine-only grade (`Lighting.MineGrade`, Contrast -0.24) that undoes part of the place's Contrast 0.5. (memory `project_fruit_empire_mine.md`, 22 Oct)

## The Fruit Castle (plot upgrades)

From `~/fruit-empire-plot-upgrades/src/PlotPenDecor.luau` (24 Oct 2026).

| Level (sign) | Stone body / corner stones / trim | Banners and flags |
|---|---|---|
| 1 Garden | the pen's old look | bunting |
| 2 Stone Keep | 132, 134, 148 / 104, 106, 120 / 190, 190, 200 | red 226, 52, 52 |
| 3 Fort | 124, 126, 142 / 96, 98, 114 / 186, 186, 196 | blue 52, 130, 230 |
| 4 Castle | 150, 152, 168 / 118, 120, 138 / 208, 208, 218 | purple 150, 70, 220 |
| 5 Fruit Palace | white 246, 244, 236 / 226, 220, 204 / gold 255, 200, 60 | purple 150, 70, 220 |

Shared colours (`C` table): wood 150, 98, 56 and 98, 60, 34; roof red 196, 64, 48 and 150, 44, 38; gold 255, 196, 46 and 214, 148, 20; flames 255, 120, 20 and 255, 214, 80; window glass 255, 200, 90 (Glass material); water 70, 170, 255.

## The Mega Rebirth floor styles

Each style's look is described in its own file (`~/fruit-empire-mega-rebirth/src/ServerStorage.PlotFloorDecor.*.lua`). Their exact colour tables were not copied here.

- **Sky Keep:** the Fruit Castle itself grows upward, one storey per Mega Rebirth, each storey an arcade of big round arches on piers.
- **Hanging Gardens:** a tower of white-and-gold terrace gardens on golden columns; at Plot Size level 0 the piers are sandstone.
- **Sky Platform:** crystal slabs floating on beams of gold light in a cyan glow; deep violet crystal edges with a gold band, cyan light lines, glass rails in cyan and lavender.

## The place-wide light (18 Oct 2026)

From [STATE.md](https://github.com/svaka2000/fruit-empire-handoffs/blob/main/STATE.md) "18 Oct: lighting pass": Ambient 124, 155, 184; Brightness 4; Lighting Style Realistic; Outdoor Ambient 157, 178, 255; Shadow Softness 1; Bloom size 56; ColorCorrection Contrast 0.5, Saturation 0.5, Tint 245, 234, 255; sky "Anime sky" (13107361022). On the cloud island `PlayerVision` turns it down: Brightness 2, `EnvironmentSpecularScale` 0.4, exposure -0.8. Grey stone and far mountains read lavender under it. See [[Light, Day and Sound]].

## Pen themes before the castle (21 to 24 Oct)

Garden, Blossom, Tropical, Crystal, Golden Empire, set in `GardenConfig.PlotPenThemes`. They come back if `Themes.Style` is set to nil. (STATE.md "21 Oct: base-upgrade show"; memory `project_fruit_empire_plot_castle.md`)

Related: [[Art Style - Studded Voxels]], [[Mine]], [[Plot Castle]], [[Mega Rebirth]].
