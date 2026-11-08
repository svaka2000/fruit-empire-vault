---
tags: [fruit-empire, project, aether, sky-stars]
status: built
updated: 2026-11-08
code: ~/fruit-empire
handoffs: []
---
# Sky Stars

Back to [[Project Status Board]].

## What players get

**12 gold stars** hidden around the Cloud Garden: 9 on the cloud, 3 on floating cloud puffs. Walk into one and it pops, sparkles, rings a chime (higher for each star in a row) and pays 3 shards (more after rebirths). All 12 pay a 24-shard bonus. They come back every day at 00:00 UTC, and each player has their own. A counter under Lock and Upgrade shows "found/12" while you are on the island and turns gold at 12/12. Five bounce mushrooms: three throw you onto the puffs, through their stars; two bounce you straight up. (old vault note [[MK Verification]], "Cloud Garden: Sky Stars")

Samarth's ask was "add more satisfaction to the cloud islands." It was built on 9 Oct, before the handoffs repo existed, so it has no handoff PR. (MK Verification; STATE.md "What was built up to 9 Oct")

## Where it lives

- **In the place:** `ServerScriptService.GardenService.SkyStars`, `StarterPlayerScripts.SkyStarsClient`, the places in `ReplicatedStorage.SkyGarden`, `SkyPuffs` and `SkyMushrooms` under the Cloud Garden. Test door: `RoyalDev:Invoke("skyStars", "reset")` or `"almost"`. (PLACE.md)
- **Design doc:** `~/fruit-empire/docs/mk/SKY-GARDEN.md`. (MK Verification)
- **Built by:** `ZZTools.SkyGardenBuilder` (has `run`, `undo`, `status`). (PLACE.md "Builders")

## Ties to other projects

- **The Fuse Machine wakes for good when a player finds all 12 in one day,** with a two-part unlock show (12 stars streak to the castle and light the machine). ([[Pets, Eggs and Fuse Machine]])
- A Mega Rebirth keeps Sky Stars. ([[Mega Rebirth]])
- The ore break reuses the Sky Stars chime, a step higher each time. (STATE.md "21 Oct: shard ore models")
- The counter's gold numbers at 12/12 are a built exception to the "gold is for bar fills" rule, waiting for Samarth. ([ui/README.md](https://github.com/svaka2000/fruit-empire-handoffs/blob/main/ui/README.md))

## Open items

- Samarth's "go" before publish (from the 9 Oct verification).
- Two Golden Crate spots on the Cloud Garden deck were not checked after the King's Throne came out. (STATE.md "Open items")

Related: [[Workspace Map]] (the sky section).
