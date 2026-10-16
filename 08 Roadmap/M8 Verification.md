---
tags: [roadmap, m8, verification]
date: 2026-10-06
---
# M8 Bats and the Golden Crate: verification

M8 is built, deployed to Studio and checked in a solo Play test (not yet published). Spec: [[Bats]]. Code: `~/fruit-empire/src-m2` tag `m8-deployed` (3a25061), contract `M8-CONTRACT.md` (section 6 = as built). Questions 17, 29, 30, 31, 34 and 44 are built on their defaults.

Every script M8 deployed matches `src-m2` (100 of 100 by length and hash after the deploy, then the three Studio-pass fixes below, each hash-checked). Samarth's save was never written: before and after both Play tests, `GardenProfiles` `p_1662286109` was 1,745 bytes, hash 2536028064. `StudioNoSave` was removed at the end.

## What M8 adds
- **A bat for every player.** It sits in the Backpack from the first join, with a slot on the HUD (bottom right). Click the slot (or press 1, or tap Bat on a phone) to hold it, click to swing. A hit knocks the other player back; it never takes fruit, shards or cash.
- **Nine bat tiers, one per rebirth stage.** Each tier pushes harder and swings faster. The bat's band takes the stage's rarity colour, and from Legendary up it sparkles.
- **Safe places.** Nobody can be hit on their own plot or pen, under the 20 s join shield or the 5 s respawn shield, on the arrival pad, or on the beanstalk.
- **The Golden Crate.** Every 7 to 10 minutes (once 3 players with plots are on), a crate falls from the sky onto one of 7 spots: 3 on the plaza, 2 on the Cloud Garden deck, 2 on shallows islets. Hold E to lift it. You walk at 85% speed and can't use the bat, and a bat hit makes you drop it. Carry it into your own pen for 20 crate units of shards (60 at stage 1).

## Builder run in Studio (Edit)
`BuildBats` built the bat template (from your existing `Workspace.Bat`, which went to `ServerStorage.Bats_backup_2026-10-06`), the effects, the Golden Crate template, 7 crate spots, the safe-zone boxes and the BatSlot/BatInfo HUD. Self-check ok. The fourth plaza spot was dropped automatically: the beanstalk overhangs it.

## Checked in Play (solo)
| What | Result |
|---|---|
| Tiers and colours (rebirths 0 to 8, then 12) | Tier 1 to 9, then 9. Every band colour matches the spec table exactly (Common 151,169,189 to Transcendent 252,247,227). Sparkle from tier 5. |
| Equip | Clicking the slot puts the bat in your hand; the "new" badge clears. |
| Swing rate at tier 1 | 15 swings sent over the ~15 s click run (one a second, the tier-1 cooldown), none refused. |
| Crate fall | 120 studs in about 2.5 s onto the spot. |
| Lift | Hold E: carried, walk speed 17 (20 x 0.85), bat put away, "GOLDEN CRATE / TAKE IT HOME" and a Drop button. |
| 90 s carry limit | A carry left alone for 90 s dropped the crate (it happened by accident while I investigated). |
| Claim | Walking into your pen through the gate paid exactly +60 shards and removed the crate. |
| Drop | The Drop button puts it at your feet, anchored, ready to lift again. |
| Console | No script errors. |

![[m8-bat-equipped.jpg]]
*The bat in hand at tier 9 (pale band, sparkle). The slot shows it equipped; the info bar says SAFE ZONE because you are on your plot.*

![[m8-crate-carried.jpg]]
*Carrying the Golden Crate: a gold frame and lid around wood panels.*

![[m8-crate-on-ground.jpg]]
*The crate on the plaza, with its label.*

## Found and fixed in the Studio pass
1. **No crate spots on the islets.** BuildBats took the two largest islets and tried 10 fixed points on each. On the real map the largest carry trees, and every rock is a stack of blocks with tops at different heights, so it found none (0 of 148 islets) and refused (changing nothing). Now it walks down the islets by size and searches a 0.5-stud grid on each rock; it found two.
2. **The crate's gold trim was left behind.** The crate is one part with its trim welded on, and setting a part's position in Roblox moves only that part. On the first carry the wooden body sat on your head while the lid and posts lay about 110 studs under the plaza (picture below). Now every move of the crate (the fall, the landing, the lift) moves the whole crate; checked in Play with 0.000 studs of drift.

![[m8-crate-carry-before-fix.jpg]]
*Before the fix: only the wooden body came along.*

Both fixes came with new offline checks that fail on the old code (the test worlds now behave like Roblox for these two things).

## Left for Samarth (needs two or three players)
Use Studio's Test tab, Clients and Servers, a Local Server with 2 or 3 players, and run `workspace:SetAttribute("StudioNoSave", true)` in the server's command bar first. The spec's test plan T4 to T17 covers hits, reach, walls, knockback per tier, immunity, the pair cap, shields, Aether, seats, the throne, the Sparring Post, Invisibility, keeping carried items and the safe zones. Then T19 and T20 on the device emulator and a phone, and T21 (hearing every bat sound) after publishing. `RoyalDev:Invoke("batRebirths", userId, n)` sets a tier; `RoyalDev:Invoke("crateAt", 5)` drops a crate on the east plaza spot.

## Known limits (written into the contract, section 6)
The server only sees the positions a player's client reports, so a carefully scripted cheat can still shape them: step heights timed to the server's checks, or hovering while dropping and re-lifting the crate. Honest play is never affected. The crate's 180 s carry budget and the bat stay as the backstop.

## Small things noticed
- The BatInfo bar shows a small green cash icon before "SAFE ZONE". It is harmless but looks out of place.
- The offline-earnings popup on join covers the bat slot until it is closed (normal for that popup).
