---
tags: [fruit-empire, project, potions, troll-potion]
status: built
updated: 2026-11-08
code: ~/fruit-empire-troll
handoffs: [19]
---
# Troll Potion

Back to [[Project Status Board]].

## What players get

Drink it and a MORPH panel opens: turn into any of the 64 fruit or the 45 forest animals (adult or baby) for 10 minutes. Morphed players can be hit by bats, carry fruit and the crate, and swing their bat. Animal morphs **bite** other players' fruit: about 4 bites stun a fruit, and the biter gets nothing (no stealing; the Lock blocks it). Fruit morphs are real fruit size and, only while standing still, hide their name tag and vanish from maps. Taunts (Ahooga, Whistle, Squeak, Toot, from Pro Sound Effects) play on T, one per 1.5 s. (memory `project_fruit_empire_troll_potion.md`; DECISIONS.md)

## Samarth's answers (22 Oct)

He first asked for a 49-Robux potion, one per buy, 10 minutes. Then: "for now just make it cost shards in the shop like any other potion". So it costs 50 price units (the same as the Aether potion) and dying ends it like any potion. The Robux version (a developer product, kept through death) is parked in `~/fruit-empire-troll/later-robux`. (memory `project_fruit_empire_troll_potion.md`)

## Where it lives

- **In the place:** `GardenService.Troll`, `GardenConfig.TrollPotion`, `StarterPlayerScripts.MorphMenu`, `ReplicatedStorage.Tools.TrollPotion`. Patched: GardenService, Potions (`G.afterDrink`), GardenConfig, UIKit (`Kit.Modals`), Explosives, MapMenu and CosmeticsClient (attribute `TrollHidden`). Backup `ServerStorage.Backup_2026-10-22_before_troll_potion`.
- **Local:** `~/fruit-empire-troll` (`NOTES.md`, `code/PATCHES.md`, `art/`, `later-robux/`).
- **Handoff:** [PR #19](https://github.com/svaka2000/fruit-empire-handoffs/pull/19).

## Open items

- Tested solo only. Not tested with two players (others seeing the costume, taunts, real bites, the map hiding), while carrying, at sea, on the Cloud Garden, in the mine, on a phone, or at the five screen sizes.
- Once in a Play test the costume came off on its own with time left; not seen again.
- Weilin's OK on the MORPH panel and on bites stunning other gardens. The badge is a placeholder.
- The Robux version waits for Samarth.

Sources: [STATE.md](https://github.com/svaka2000/fruit-empire-handoffs/blob/main/STATE.md) "22 Oct: the Troll Potion" and "Open items"; memory `project_fruit_empire_troll_potion.md`.

Related: [[Forest Animals]] (the animal costumes), [[Robux Shop]].
