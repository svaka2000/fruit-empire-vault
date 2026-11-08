---
tags: [fruit-empire, lighting, sound, rules]
status: current
updated: 2026-11-08
---
# Light, day and sound

Back to [[Knowledge Base - Start Here]].

## Always day

> [!danger] Never change the sky or the clock
> Since 11 Oct 2026 the game is always day. Samarth: "make it always daytime, get rid of daylight cycle". `Config.DayNightCycle = false`, so the clock reads `Config.FixedClockTime`. No new script touches `Lighting`, `ClockTime`, `Atmosphere` or `Fog`.
> Sources: `~/fruit-empire-pets/DESIGN.md` lines 221 and 1348 (decision D8); GardenConfig line 1821 in the 29 Oct export; handoff [2026-10-24-samarth-plot-upgrades](https://github.com/svaka2000/fruit-empire-handoffs/blob/main/handoffs/2026-10-24-samarth-plot-upgrades/HANDOFF.md).

What this means in practice:

- Night-only features stay dark: the Fruit Castle's window glow and its level 5 gold beams are built but never show. (STATE.md "24 Oct: the Fruit Castle")
- The pets design gets its rhythm from animal behaviour, particles and sound, not from the sky. Day-or-night pet traits became plain traits. (`~/fruit-empire-pets/DESIGN.md` line 696)
- **Change the place's look in `Lighting` itself, not in scripts.** `StarterPlayerScripts.PlayerVision` takes the place's own Ambient, Outdoor Ambient and Brightness as the daytime look. It is the only script that writes Lighting. ([PLACE.md](https://github.com/svaka2000/fruit-empire-handoffs/blob/main/PLACE.md) "Where things live"; `~/fruit-empire-mine/CAVE-OVERHAUL.md` section 1)

## The one allowed exception: the mine's own light

While a player is in the mine, `PlayerVision` applies the mine's values from `Config.Mine`: fill light, exposure, and a mine-only colour grade (`Lighting.MineGrade`). The cave overhaul changed only these, never the place-wide Lighting. (`~/fruit-empire-mine/CAVE-OVERHAUL.md` sections 2-3; PLACE.md)

- Why the mine had dark spots: the rock was dark navy, the fill was about 13% grey, and the place's ColorCorrection Contrast 0.5 crushed the rest to black. The fix was a cool fill (112, 114, 130), Exposure 0.35, a grade with Contrast -0.24, and lighter rock: 14 test camera spots went from 67% to 12% almost-black. (memory `project_fruit_empire_mine.md`, 22 Oct)
- Samarth's rule for that fix: "you dont have to add more lanterns or something find a way to do it normally". New areas get light from what is in them (crystals, glowing water, cracks of light). (`~/fruit-empire-mine/CAVE-OVERHAUL.md`)

The place-wide light values (18 Oct) are in [[Colour Palettes]].

## Sounds

> [!important] Licensed library sounds
> Every sound picked so far comes from Roblox's licensed audio library (uploads by Pro Sound Effects and APM), because those play in any experience. The pets build's sound scout was limited to "ProSoundEffects / APMOfficial only".
> Sources: [[Decisions Log]] entry D-002 (4 Oct, music); `~/fruit-empire-pets/NOTES-after-build.md` line 24; [DECISIONS.md](https://github.com/svaka2000/fruit-empire-handoffs/blob/main/DECISIONS.md) (Troll Potion taunts from Pro Sound Effects). Samarth's own words for this rule were not found; it is the chats' practice. `[practice, not a quoted rule]`

- Sounds are pitched in code to make many effects from one sample (the pets design's marimba note and bell). (`~/fruit-empire-pets/DESIGN.md` line 1217)
- The Studded Pack's "Fire Crackling Sound" (9068935533) is **not approved**. ([PLACE.md](https://github.com/svaka2000/fruit-empire-handoffs/blob/main/PLACE.md) "Other rules")
- Music: the playlist and area themes picked on 4 Oct are in D-002 of the old [[Decisions Log]] (detail in `~/fruit-empire/docs/MUSIC-PICKS.md`). The royal and duel music went with the King's Throne on 9 Oct.
- **Nobody has listened to many of the new sounds yet:** the base-upgrade show, the Upgrade skill tree, the Fruit Castle show. (STATE.md "Open items")

Related: [[Hard Rules]], [[Art Style - Studded Voxels]].
