---
tags: [fruit-empire, project, bats]
status: built
updated: 2026-11-08
code: ~/fruit-empire-bats
handoffs: [18]
---
# Nine bats

Back to [[Project Status Board]].

## What players get

Everyone has a bat. It knocks other players back but never takes anything they own. Since 22 Oct there are **nine bats, one per stage**, each themed on a fruit and studded like the pickaxes, each with its own swing trail and hit effect: Applewood, Sprout, Blue Corn, Grape, Pineapple, Dragon Pepper, Glowshroom, Lunar Cherry, Soul Fruit. They are 4.4 to 7 studs long; the grip tape is the rarity colour; glowing pieces start at stage 3. A toast and a card show the new bat after a rebirth. (memory `project_fruit_empire_bats.md`; STATE.md "22 Oct: nine bats")

Earlier, M8 (6 Oct) added the single bat and the Golden Crate. (STATE.md "What was built up to 9 Oct")

## Samarth's ask

He liked the pickaxe ladder and asked for a "massive rehaul on bat designs" with Claude Design, then "use the suggested answers in BAT-PLAN section 8" and "just go ahead". (memory `project_fruit_empire_bats.md`)

## Where it lives

- **In the place:** `ServerStorage.BatModels` (the 9 sources), `ServerStorage.BatTiers` (the 9 Tools), `ReplicatedStorage.BatFX.Hit<Bat>`, `GardenConfig.BatTiers` (child module), `GardenService.Bats`, `BatClient`, `BatHud`, `StoredFruitsMenu`. Bat rules in `ReplicatedStorage.BatRules`. Backup `ServerStorage.Backup_2026-10-22_before_bat_tiers`. (PLACE.md; memory)
- **Local:** `~/fruit-empire-bats` (`BAT-PLAN.md`; builders in `code/tools`; models in `models/tools`; particle pictures and their uploaded ids in `fx/`). GLBs in `~/Downloads/fruit-empire-bat-models`.
- **Canvas:** [bat board](https://claude.ai/artifact/AKMm89kKmG6ZrgheXhoHyh). **Handoff:** [PR #18](https://github.com/svaka2000/fruit-empire-handoffs/pull/18).

> [!tip] Change a bat the right way
> Edit `BatModels` or the builder's LOOK and FX tables and re-run `build_bat_tools.luau`. Never edit a Tool by hand. (PLACE.md; memory `project_fruit_empire_bats.md`)

## Look lesson

Under the place's light (Brightness 4, Bloom Threshold 2), pale parts and Neon bloomed white. The fix: Handle colour 85% grey and Neon at about 62% of its colour. Particle pictures took about an hour to pass Roblox's approval. (memory `project_fruit_empire_bats.md`) See [[Art Style - Studded Voxels]].

## Open items

- Not tested with two players (real hits), on a phone (frame rate with a stage 7 to 9 bat) or after a real rebirth.
- Samarth's own look in Play; Weilin to say if nine bats replacing M8's one is OK.
- The pets design widens what bats do: bonk forest animals, and knock a carried forest egg loose (changing bat rule B29, "bats hit players only"). See [[Pets, Eggs and Fuse Machine]].
- Old leftovers: "InArena" and "RoyalSeat" checks remain in Bats and BatClient since the King's Throne was removed. (STATE.md "Open items")

Sources: [STATE.md](https://github.com/svaka2000/fruit-empire-handoffs/blob/main/STATE.md); memory `project_fruit_empire_bats.md`.
