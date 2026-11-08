---
tags: [fruit-empire, project, pets, eggs, fuse-machine]
status: in-progress
updated: 2026-11-08
code: ~/fruit-empire-pets
handoffs: [6, 7, 8]
---
# Pets, eggs and the Fuse Machine

Back to [[Project Status Board]].

## What players get (as designed)

Wild adult animals live in the forest in families, with nests of eggs every player shares. You sneak in and take an egg; its parent chases you, and if it catches you it knocks you back and walks the egg home. Your bat can bonk animals (a cartoon daze, then a knock-out poof) and knock a carried egg loose from another player. You carry the egg overhead back to your plot and hatch it on a Nest Shelf. Pets give multipliers. Three pets of one species and size fuse into a bigger one at the **Fuse Machine** in the sky castle, which wakes when you find all 12 Sky Stars in one day. Every 600 s the Mist sweeps the forest and sends everyone home.

Source: `~/fruit-empire-pets/handoff-draft/DECISIONS-rows.md` (30 Oct); full design `~/fruit-empire-pets/DESIGN.md`.

## Samarth's ask (29 Oct 2026)

Add pets to the forest; adults lay eggs; steal one and the adult chases you like in Steal an Egg; realistic spawns ("not like 50 common pets or something orlike 4 of the same adult animal"); many polished animations; hatch on your plot; pets give multipliers; a Fuse Machine in the Aether unlocked by all 12 Sky Stars with a great unlock; bats hit players to take eggs and hit adults; "make it 10x better", "non AI slop". His exact words: `~/fruit-empire-pets/BRIEF.md`. (memory `project_fruit_empire_pets.md`)

## Status

> [!warning] Install not confirmed
> The whole system was built and tested offline (30 Oct): 28 new scripts and patches to 36 live scripts, all groups' tests passing. The Pet and Egg chat took the Studio turn at 11:35 PDT on 30 Oct (backup `ServerStorage.Backup_2026-10-30_before_pets`). The deploy snippets were generated at 12:43 that day. No pets handoff has been published, and the handoff page on the local branch is still blank. **Whether the pets are installed in Studio is not recorded in any source read for this vault.** Check STATE.md and `99_status.lua` before assuming either way.
> Sources: the queue note (`studio-turn-rule.md`); `git log` in `~/fruit-empire-pets`; `~/fruit-empire-handoffs/handoffs/2026-10-30-samarth-pets/HANDOFF.md`.

Already in the place since 18 Oct: the **Aether Fuse Machine model**, 538 studded parts from Samarth's own model, at the west end of the sky castle's courtyard; decoration only so far ([PR #6](https://github.com/svaka2000/fruit-empire-handoffs/pull/6)). The 135 animal models are in `ServerStorage.ForestAnimals` ([[Forest Animals]]).

## Where it lives

- **Repo:** `~/fruit-empire-pets` (local git). Read in this order: `BRIEF.md` (his words), `RESEARCH.md`, `DESIGN.md` (from a 4-designer, 3-judge panel), `BUILD.md` (who owns which file), `INSTALL-PLAN.md`, `PLAYTEST-PLAN.md`, `NEEDS-tracker.md`, `NOTES-after-build.md`, `NOTES-fuse-machine.md`.
- `ref/scripts/` is an exact export of all 330 live scripts on 29 Oct (useful as a read-only reference for any project).
- **How it runs:** wild adults are server data only (no Humanoid, no physics). The server sends 12-byte snapshots over an unreliable remote; clients draw pooled, animated rigs (`WildNet`, `WildView`, `PetAnimator`, `WildSteer` for steering round 3,807 trunks). (memory `project_fruit_empire_pets.md`)
- **New scripts** (from `tools/status_compact.lua`): `GardenConfig.Pets`, `PetRules`, `WildNet`, `PetAnimator`, `WildView`, `PetStateClient`, `PetUI`, `GardenService.PetModels`, `.WildSteer`, `.Wildlife`, `.ForestSites`, `.Forest`, `.ForestBrain`, `.CarryGuard`, `.EggCarry`, `.Hatchery`, `.Pets`, `.Fuse`, `ZZTools.NestPropBuilder` and the client panels.
- **Canvas:** [Fruit Empire Pets UI](https://claude.ai/artifact/GApyrQArNP1fabtFW5sPeh) (15 screens). See [[Claude Design Canvases]].

## Waiting on Samarth

- **One Import 3D** of the rigged animal GLBs in `~/Downloads/forest-animals-rigged` (135 files), like on 19 and 21 Oct. Until then the single meshes are the fallback.
- **Upload the Pets button art** (`~/fruit-empire-pets/ui/icon/PetsButton.png`).
- **19 open design calls** in DESIGN.md section 16.2, each built on a default.
- No plotless spawn exists; the Plaza Nest Box stands near the beanstalk. Add a SpawnLocation if plotless players should start there.

Source: `~/fruit-empire-pets/NEEDS-tracker.md` items 32-34; memory `project_fruit_empire_pets.md`.

## Open questions and risks

- Weilin's OK on the new screens (the Fuse panel's purple striped title bar is new).
- Save room: about 870 bytes of headroom in the worst case, so pets need save schema v3. (memory `project_fruit_empire_pets.md`)
- The HUD's left column: the Pets button needs room on 375-tall phones; check 812 x 375 and 667 x 375 in Studio. (NEEDS-tracker item 1)
- The pets patch to `GardenService.MineDig` (8 edits) must also land in the dig repo, `~/fruit-empire-mine/dig/src`, and must not change `MineDigSync`'s answer or limit (the map reads it). (NEEDS-tracker items 35-36)
- Egg bounties were set to 1.6 times the design's numbers so a raid pays about as much as mining. Samarth has not confirmed. (DECISIONS-rows)
- The old pacing tool is gone, so the design's economy re-run needs a new model. (memory `project_fruit_empire_pets.md`)

Related: [[Forest Animals]], [[Sky Stars]], [[Nine Bats]], [[Mine Digging and Cave-in]] (mine eggs go into the same save field).
