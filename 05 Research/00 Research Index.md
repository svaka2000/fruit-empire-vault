---
tags: [research, index]
topic: Index of the Fruit Empire expansion research, with the most important findings across all notes
date: 2026-10-04
---

# Research Index

Source brief: [[Brain Dump 2026-10-04]]. Every note below had a critic pass on 2026-10-04. Each note now ends with a "Review (critic pass)" section that lists what was verified, what was corrected, and what is still not confirmed.

Terms used here:
- **Live code** means the newest local export of the game, `~/fruit-empire/src-export-2026-10-04/`.
- **Stage** means the rarity tier of the best fruit a player can reach, from Common to Transcendent.
- **Shards** are the second currency. Today they come mainly from claiming Index entries.

## Notes

| Note | One-line takeaway |
|---|---|
| [[Steal a Pet - Bats and Rebirth Knockback]] | Steal games gate one slap per rebirth at 1–5% of the rebirth's price but never show knockback numbers, so Fruit Empire's bats should show real push numbers, reuse the Crown arena's working knockback code, and never cost anyone owned fruit. |
| [[Merge a Nuke - Explosives and Health Drain]] | Merge a Nuke drains HP over 2+ hits (HP ≈ 2 × same-tier damage) with damage growing only about 9% per tier while cash doubles; Fruit Empire's live TNT instead deletes fruit on any plot at once, so the redesign changes an existing cross-plot weapon. |
| [[Sky Rivals - Weapons and Artillery]] | SkyRivals (R6, 12-player sky PvP) wins fights through knockback into the void and small single-use explosives; most of its stats are wiki guesses, and lobbed artillery in Roblox is built with raycast projectiles (FastCast2) plus a Beam arc preview. |
| [[Mining and Ore Merging - Reference Games]] | No big Roblox game carry-merges ores, so use fixed regrowing ore nodes, reuse the TNT health-drain system underground, ramp glow by rarity, and pin ore income to the player's stage; the 100k-shard underground is beyond what the Index pays today. |
| [[Pets - Taming, Merging and Wandering]] | Feed-to-tame is proven (99 Nights), merges must pay more than 2x, pets should be server data drawn by clients with no Humanoid, and a "nap" rule keeps pets from breaking the rebirth reset. |
| [[Rebirth Economies in Merge and Incremental Games]] | Growth must come from unlocking the next rarity, not from the multiplier; the live game already has a full-wipe rebirth, its stage jumps run from 45x to 7,240x, and a full Index pays only 26,528 shards. |
| [[Roblox Platform Constraints for the Expansion]] | Saving is cheap but moving physics objects is not; keep fruit, ores and pets anchored and server-authoritative, keep Robux sales fixed (no paid randomness), and note the game allows 50 players on a 12-plot map. |
| [[Swimming Fatigue and Boats in Roblox]] | Roblox has no swimming or floating on part water, the current plates would drag boats like a conveyor, and swimming must stay the main early route to the forest because shards are scarce early. |
| [[Top-Tier Sword Combat and Animation in Roblox]] | Top combat games end strings with a launch, use short parry windows and hit stop; Fruit Empire's duel is already an R15 ring-out fight with stagger and no health, and new animations must be published under the experience owner. |
| [[Consistent Image-Based UI Kits in Roblox]] | Build every new screen from 9-sliced kit prefabs in fixed DisplayOrder bands, stop using `TextScaled`, and keep panels in Scale and the HUD in Offset + `HudScale`, as the game already does. |

## Cross-cutting findings

These are the 10 findings that matter most for the design, ranked by how much they change the plan.

1. **TNT already exists and already hits other players' plots.** The live TNT costs 2,500 shards, works on any plot the player stands on, and deletes every fruit within 18 studs at once. Only the 60-second Lock stops it. So "drain health instead of deleting" changes an existing cross-plot weapon. The owner still has to decide whether explosives should hit other players' fruit at all now that stealing is gone. See [[Merge a Nuke - Explosives and Health Drain]] and [[Sky Rivals - Weapons and Artillery]].

2. **Shard supply is far below the planned shard prices.** A full Index pays 26,528 shards, and the Common page pays only 31. With rebirth-gated rarities and today's "pay the Index again on rebirth" rule, a player has about 1,248 Index shards after rebirth 3 and about 96,000 with the Transcendent page full. Today's prices are boat 1,000, TNT 2,500, potions 5,000 to 10,000, and underground 100,000. Either add a shard source or re-price before building. See [[Rebirth Economies in Merge and Incremental Games]], [[Mining and Ore Merging - Reference Games]] and [[Swimming Fatigue and Boats in Roblox]].

3. **A rebirth already ships, and the brief changes most of it.** Today it needs a full Index, wipes the run including the Index, keeps shards and tools, pays the Index shards again, and gives +100% cash per rebirth. It has no rarity cap and no storage growth. The research proposal to keep the Index through rebirth would remove the shard re-payment, so it must be decided together with finding 2. Existing players also need a migration rule. See [[Rebirth Economies in Merge and Incremental Games]].

4. **"Scale to the player's stage" has to use a per-stage table.** The tiers have different sizes (5, 8, 11, 8, 7, 8, 5, 8 and 4 fruits), so the income jump from one stage's top fruit to the next runs from 45x to 7,240x. The Rare stage needs 1,024 copies of its first fruit to reach Pear, against 8 to 128 for other stages. Pet payouts, ore payouts, TNT prices, rebirth costs and pacing all need this table, not one smooth ratio. See [[Rebirth Economies in Merge and Incremental Games]], [[Pets - Taming, Merging and Wandering]] and [[Mining and Ore Merging - Reference Games]].

5. **Health drain should take several hits, and damage should grow slower than income.** Merge a Nuke sets a nuke's HP at about 2 × its own tier's damage, so a same-tier hit needs 2 shots. Its damage grows about 9% per tier while cash doubles. An earlier version with doubling damage let top tiers one-shot everything, and players complained. Use the docs' linear falloff, `BlastPressure = 0` and `DestroyJointRadiusPercent = 0`, and point the biggest blasts at a shared target that heals over time. See [[Merge a Nuke - Explosives and Health Drain]], [[Sky Rivals - Weapons and Artillery]] and [[Roblox Platform Constraints for the Expansion]].

6. **The Crown arena already has the combat core, so build on it.** Its knockback works: the victim's client applies the push, with a 0.12-second server `LinearVelocity` as backup. A plain server impulse barely moves a standing Humanoid. Bats should reuse that push and show their power in the same studs-per-second units (the arena's Strike is 40, the Slam 92). The duel itself is an R15 ring-out fight with a stagger meter and no health, so health drain has to be designed into it. Animations must be published by the experience owner, the user account `hikeALT12212`. See [[Steal a Pet - Bats and Rebirth Knockback]], [[Top-Tier Sword Combat and Animation in Roblox]] and [[Roblox Platform Constraints for the Expansion]].

7. **Pets need one income rule, and they must be drawn on the client.** Two notes propose different rules: income fixed at tame time with a "nap" rule, or income as a factor of the best fruit this run. Pick one. Either way, every merge must pay more than 2x, pets should be server data drawn by each client with an AnimationController (no Humanoid), and they should never block fruit carrying or be damageable. See [[Pets - Taming, Merging and Wandering]], [[Rebirth Economies in Merge and Incremental Games]] and [[Roblox Platform Constraints for the Expansion]].

8. **The part-based sea changes how swimming and boats work.** Roblox's swimming state and buoyancy only work in Terrain water, and Fruit Empire's sea is parts about 1.8 studs deep. The current plates push at 42 studs per second like a conveyor and would drag boats, so they must go or be put in a collision group. Boats need driver network ownership and server-side damage. Swimming must stay possible for new players. See [[Swimming Fatigue and Boats in Roblox]] and [[Roblox Platform Constraints for the Expansion]].

9. **New UI must come from the existing kit, with fixed rules.** Use 9-sliced kit panels as prefabs, one DisplayOrder band per layer, `ZIndexBehavior = Sibling`, and no bottom-corner buttons on mobile. `TextScaled = true` appears 30 times in 14 scripts and makes equal labels render at different sizes. New modals should follow the panels (Scale) and new HUD pieces should follow the HUD (Offset with `HudScale`). See [[Consistent Image-Based UI Kits in Roblox]].

10. **The public game listing does not match the design.** Roblox lists Fruit Empire as "[STEAL!] Fruit Empire", its description says "ROB everyone", and it allows 50 players per server on a map with 12 plots. To reach players under 16, the game also needs 250 plays from "highly engaged" age-checked players in 60 days (it had 119 lifetime visits on 2026-10-04). Keep every Robux sale fixed, not random. See [[Roblox Platform Constraints for the Expansion]] and [[Pets - Taming, Merging and Wandering]].

## Still not confirmed across the notes

- Real knockback forces in Steal a Pet and Steal a Brainrot.
- Merge a Nuke's blast radius, falloff and city pay formula.
- Any exact SkyRivals explosive numbers.
- The 5 species in the Stud-Themed Animal Pack.
- Whether the place has any Terrain water.
- Whether `Lighting.ExtendLightRangeTo120` is on in the place.
- The 1024×1024 image upload cap.
- Real play time to each rebirth, in Fruit Empire or any reference game.
