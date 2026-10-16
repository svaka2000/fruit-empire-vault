---
tags: [roadmap, m5, verification]
date: 2026-10-05
---
# M5 Explosives and fruit health: verification

M5 is built and in Studio (not yet published). Spec: [[Explosives and Fruit Health]]. Every script in Studio matches `~/fruit-empire/src-m2` (87 of 87, by length and hash). Tested in Play with `StudioNoSave` on. Samarth's save was never written: before and after, `GardenProfiles` `p_1662286109` was 1,731 bytes, hash 3001582050, saveCount 609, rebirths 8, shards 58,204. `StudioNoSave` was removed at the end.

`ServerStorage.ZZTools.BuildExplosives` ran once in Edit. Its self-check said ok. It renamed the old `TNT` model to `TNT1`, built `TNT2`, `TNT3` and `Nuke`, made `ReplicatedStorage.WeaponFx` (8 effects) and the Lock alert badge. The old TNT model was copied to `ServerStorage.Explosives_backup_2026-10-05` first.

![[m5-models.jpg]]
*Left to right: TNT (the old TNT model, unchanged), Big TNT, Mega TNT, Nuke.*

## What M5 put in
- **Four explosives** on the Weapons tab: TNT, Big TNT, Mega TNT and Nuke. They unlock at rebirths 1, 3, 5 and 7. Prices are in shards and follow your stage.
- **Fruit health.** A blast takes HP off the fruit near it. A fruit at 0 HP is stunned for 20 s: it earns nothing and can't be picked up. Then it comes back full with a 30 s shield. A hurt fruit heals after 16 s without a hit.
- **The Lock blocks explosives**, and while one is armed on your garden you can raise it from anywhere.
- **The Nuke** hits the whole garden: 60% at once, then 10% a second for 4 s. The owner gets a survivor bonus in shards.
- **Old TNT is refunded**: each `TNT` in a saved bag becomes 2,500 shards on load.
- **A rebirth refunds explosives** in the bag at the price you paid.

## Checked in Play (spec test numbers)
| # | What | Result |
|---|---|---|
| T1, T2 | Weapons tab at rebirth 0 | Four cards. Buy says "Rebirth 1", "Rebirth 3", "Rebirth 5", "Rebirth 7", dimmed. Black previews. Pressing a locked Buy: "Unlocks at rebirth 1!" |
| T2, T3 | Rebirths 1 to 8 | Each card unlocks at its rebirth. Card prices: r1 110; r3 480 / 1.3K; r5 2.3K / 6.2K / 16K; r7 11K / 30K / 75K / 190K; r8 25K / 66K / 160K / 410K. All equal `Config.ToolPrice`. |
| T5 | His save had 1 old TNT | On load: shards 58,204 to 60,704 (+2,500), the `TNT` entry gone. (In Play only; never saved.) |
| T7 | Drop on your own garden, Studio door off | "You can't blow up your own garden!" The TNT stays in hand. |
| T9 | Drop on a garden right after its rebirth | "That garden was just planted. Try again in 59s." Still in hand. |
| T10 | Drop again within 6 s | "Explosives ready in 6s." The carry row says "Ready in 6s". Still in hand. |
| T11, T13 | Drop with the self-target door on | Armed 3 studs in front, on the soil. Red circle the size of the blast. Countdown 3, 2, 1 over the armed model. Fuse sparks. |
| T14 | The garden owner's warning | Red toast "imtheonewhoknocks placed a TNT in your garden! Lock to block it." (the display name). Lock "!" badge on until the blast, then off. |
| T18, T19 | Damage | 55, 36, 17 off at 0, 5, 10 studs at stage 3. A live blast at stage 3: all 35 fruit HP values matched the formula for their distance. 3 were in range. "Hit 3 fruit. 0 stunned." to the placer, "...'s TNT hit 3 of your fruit. 0 stunned." to the owner. No circle or countdown left after. |
| T26 | Healing | 50% damage door: half-full yellow bars on all 35 fruit. A 596 HP fruit healed 59.6 HP a second (10%), its HP cleared at full, and its bar hid 3 s later. |
| T27, T28 | Stun | 35 stunned. Income 5.5e23/s to 0. Empire power unchanged (5.5e23). Every fruit Busy, no pickup prompt. Fruit see-through with sparkles, bars grey and refilling. After 20 s: all 35 full with a cyan 30 s shield, income back. |
| T21 | Nuke at rebirth 7 (stage 8) | "NUKE incoming on imtheonewhoknocks's garden in 10s!" and the owner warning. The circle covered the whole soil. Blast at 10 s. HP 100, 40, 30, 20, 10, then stunned on 34 fruit (one went 40, 30, 19, 9 because each tick rounds up; it still ended stunned). Survivor bonus +3,740 (60,704 to 64,444), "Your garden survived a nuke: +3.7K shards". The Nuke card then read "Next nuke in 9:17". |
| T15, T32 | Lock from off the plot | Armed a Big TNT, walked off the plot, pressed Lock: the Lock went up. At the end of the fuse the blast was cancelled: "Blocked by a Lock. It went back in your bag." and "Your Lock blocked imtheonewhoknocks's Big TNT!" The Big TNT was back in the bag. Badge off. |
| T4 | Rebirth 3 to 4 holding 2 TNT and 1 Big TNT | Shards 64,444 to 63,204: +2,260 refund minus the 3,500 rebirth cost, exact. Explosives gone from the bag, potions kept. "Rebirth 4 complete..." then "Rebirth turned your explosives into 2.3K shards." |
| UI | "+$" popups | None over a damaged fruit (6 fruit, 6 s). |
| Console | Whole session | Only the dev door lines. No warnings, no errors. |

![[m5-weapons-locked.jpg]]
*Rebirth 0: every weapon locked.*

![[m5-weapons-rebirth1.jpg]]
*Rebirth 1: TNT unlocked at 110 shards.*

![[m5-fuse-cooldown.jpg]]
*The circle, the sparks, the cooldown toast and "Ready in 6s" on the carry row. The Lock badge is on.*

![[m5-bars-half.jpg]]
*Health bars at 50%.*

![[m5-stunned.jpg]]
*Stunned: see-through fruit, sparkles, empty bars.*

![[m5-shield.jpg]]
*After the stun: cyan shield bars.*

![[m5-nuke-fuse.jpg]]
*Nuke armed: the circle covers the garden.*

![[m5-nuke-fallout.jpg]]
*Nuke fallout: every bar red and draining.*

![[m5-weapons-nuke-clock.jpg]]
*The Nuke card's clock after a nuke.*

## Found and fixed in the Studio pass
1. **The weapon cards hid their stats line** ("Garden | Radius 10, 3 hits"). Studio's Play window draws a 216 x 272 card, and Larger text draws every label 10 px taller. The two lines under the picture missed fitting by 2 px, so the small-screen rule hid the stats line on every card.
   - Fix (`ShopMenu`): before hiding the stats line, the two lines are packed into the free space, and the owned line can step down a size or two (to 11 here). When the whole note still can't fit across the card, the card shows only what the weapon does ("Radius 10, 3 hits"). The Weapons tab already says it works on gardens. A full-size desktop card still shows the whole note. A phone card still hides the line.
   - Before: ![[m5-weapons-rebirth1.jpg]] After: ![[m5-weapons-stats-fixed.jpg]]
2. **The countdown number was small**, about name-tag size from a few studs away. It also sat behind the placer's own name tag.
   - Fix (`ExplosiveConfig`): the countdown is half as big again (3.6 x 1.8 studs; the Nuke's 5.4 x 2.7), and its text cap went from 40 px to 72 px. Without the higher cap the bigger box drew the same small number.
   - Before: ![[m5-fuse-countdown-old.jpg]] After: ![[m5-countdown-bigger.jpg]]

Every fix has a harness check. C1 check X19 fails on the old ShopMenu and passes on the new one.

## Plain-Luau harnesses (after the fixes)
S1 417/417, S2 all pass (RebirthLock 55, Ambient 48, Crown 21, GardenStore 25 and its self-test 70/70, DevPanel 57, S2 x S1 18), C1 (shop) 139/139, C2 (health client) 152/152, W (BuildExplosives) 394/394, M5 integration 279/279, the weak-keys exploit check PASS.

## Still to verify (needs something this pass couldn't do)
- **Merge All skipping stunned fruit.** In Studio, Merge All is a gamepass the test account doesn't own, so pressing it opened the purchase prompt. That prompt then blocked every mouse click until Play stopped. The S1 harness covers the skip.
- **The pickup prompt hiding a bar.** Fruit prompts are off while you carry a tool, so this didn't come up. The C2 harness covers it.
- **The 16 s wait before healing.** The heal rate and the hide were measured live; the wait was not (the sampling started late). The S1 harness covers it.
- **The Nuke's screen flash.** It is short and the screenshot missed it. The C2 harness covers it.
- **Two players**: a second player seeing the circle, a neighbour's fruit staying untouched, a placer at a higher stage than the owner (T16, T20). Covered by S1 and integration harnesses.
- **Phones.** The MCP tools can't switch Studio's device emulator. Phone cards are covered by the C1 harness.
- **Sounds** (fuse, booms, siren, shield break). Screenshots can't hear them.

## Design calls for Samarth
- **The Lock cancels an explosive at the end of its fuse,** not the moment you press it. That is the contract (each check runs again when the fuse runs out). The placer only finds out when it would have gone off.
- **TNT is the biggest model of the four.** It is your original TNT block (4 x 4 x 4). Big TNT is 2.7 x 3.4 and Mega TNT 3.7 x 3.7. The weakest explosive looks the largest on the ground. It could be shrunk to about 2.5.
- **The bigger countdown and the short stats line** are look calls. Both are one line in `ExplosiveConfig.Circle` and `ShopMenu` to undo.
