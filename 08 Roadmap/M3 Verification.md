---
tags: [roadmap, m3, verification]
date: 2026-10-05
---
# M3 Royal Combat core: verification

M3 is built and in Studio (not yet published). Spec: [[Royal Combat]]. Every changed or new script in Studio matches `~/fruit-empire/src-m2` at commit 95c07bd (13 of 13, by length and hash). Tested overnight in Play with `StudioNoSave` on. Samarth's save was never written (rebirths 8, saveCount 608, shards 58,204 before and after).

The screen was locked all night, so there are **no screenshots**. Everything below was checked with scripts and the server's combat log. The look of it is on the "verify tomorrow" list.

## What M3 put in
- **Health instead of stagger.** Every fighter has 100 HP. 0 HP is a knockdown: the fighter freezes, flashes white, and the other side scores a Crown Point. The less HP you have, the further a hit throws you, so ring-outs still happen.
- **A 4-hit combo.** Tap Attack: 7, 7, 8, then a 12 finisher. A finisher that hits nobody stuns you for 0.45 s.
- **Heavy strike.** Hold Attack, then let go: 16 damage.
- **Block and parry.** Holding block takes a fifth of a light hit. Blocking just before a hit lands parries it: no damage, the attacker is stunned, and your next hit does 1.5x.
- **The Royal Slam, rebuilt.** A leap, a dust ring warning, then a ground pound: 14 damage within 11 studs.
- **Fight effects** (`ReplicatedStorage.RoyalFX`): slash arcs, sparks, damage numbers, the white hit flash, the slam's crack and dust, hit stop, camera shake. All come from the effect packs already in the place, copied by a tool into `ReplicatedStorage.RoyalFXAssets`.
- **Health bars** on the duel card (both fighters) and over every fighter and knight.
- **Controls:** touch buttons, a charge ring, and on PC an ability bar and a key legend. They use kit art stand-ins until the real button art exists.
- **Royal rewards by stage.** Every royal shard reward is now counted in units of the player's own stage. At stage 9 a Defense pays 16,460 shards and a Victory 41,150.
- **Repeat pairs.** The same two players fighting again within 30 minutes get no reward.
- **No rebirth mid-fight.** Pressing Rebirth in the arena is refused.

## Checked in Play (spec test numbers)
| # | What | Result |
|---|---|---|
| 1 | A practice bout starts at full health | 100 / 100 on both sides |
| 2 | The four combo steps on a knight | 100 → 93 → 86 → 78 → 66; numbers 7, 7, 8 white and 12 gold |
| 3 | A tap every 1.2 s | step 1 every time, even when the client claims steps 2 and 3 |
| 4 | A tap 0.3 s after a finisher | refused ("strike refused (gap)") |
| 5 | A finisher that hits nobody | 0.45 s stun; a strike during it is refused, the next one after it works |
| 6 | Push at half health | heavy push 74 → 133 sideways, lift 26 → 36: about 2.5x the air distance |
| 7 | A hit at 5 HP | knockdown, point scored as "ko", both back to 100 after the pause |
| 8 | Parry the knight's swing | 0 damage, knight stunned 0.8 s, next hit 11 instead of 7 |
| 9 | A block tapped too short | refused 0.5 s later, allowed again 1.4 s later |
| 10 | Holding block when a knight hit lands | 1 damage, marked blocked |
| 12 | Royal Slam next to the knight | warning first, impact 0.334 s later (spec: 0.32 ± 0.05), 14 damage; no Neon parts in the effects folder |
| 16 | Health tags | the knight's tag shows its HP; your own tag stays hidden |
| 24 | Rebirth inside the arena | "Finish your royal fight first"; rebirths stay 8 |
| 33 | A hit 1.6 s after a parry | normal 7 damage |
| — | Heavy strike | 16 damage |
| — | A combo step that arrives a little early | held by the server and swung when its gap opens (not refused) |
| — | Consoles | no errors from any M3 script on the server or the client |

Before Studio, the build passed 151 server checks and full client checks (PC and touch) in plain-Luau test harnesses. Two review rounds found 21 real problems. All were fixed before or during the overnight test.

## Bugs found on the night and fixed
1. **The thin air killed fighters at the ring's edge.** The garden kills anyone above height 418 who isn't standing on sky ground. The ring floats at 452. A fighter knocked over its edge was killed by the thin air before the arena could catch the fall. Arena fighters are now exempt; the arena already scores the fall as a ring-out. This was in the game before M3 too.
2. **Camera shake drifted.** Each frame's shake stayed on the camera, so big hits left the view a few degrees off. The shake is now taken off every frame before the camera updates.
3. **Controls stopped when the window wasn't drawing.** The combat state now updates every frame, whether or not the screen is drawing.
4. **Small timing edges** around a block cooldown, a slam after being hit, and a combo step arriving early over the network.

## Eyes-on check (5 Oct, screen unlocked)
| Screen | What it shows |
|---|---|
| ![[m3-duel-hud-fixed.jpg]] | **Practice bout, after the fixes.** The duel card with both health bars and "First to 3", the knight's health tag, a damage number ("7") beside the tag, the PC ability bar (F, Q, E) and the key legend ("Hold Heavy" now fits). |
| ![[m3-slam-effects.jpg]] | **Slam and hit effects.** The slam's dust-ring warning, the flying rocks and the hit burst on the knight. |

Found by looking, and fixed (commit a4b7ec6, in Studio):
1. **The legend plates and ability slots used art with an apple baked in.** The kit's NameBar image (from the carry info bar) has an apple at its left end, so every plate and slot showed it, and the slots stretched it into blobs ([[m3-duel-hud-before.jpg|before]]). They now use the kit's plain InfoBox.
2. **"Hold Heavy" and "Sudden death" were cut off** at the Larger Text Size. Plates and the duel card's centre column now grow to fit their words.
3. **Damage numbers sat on top of the knight's health tag.** They now start beside it, left or right, at any distance.

Screenshot note: Studio's screen capture does not draw billboards set to "always on top", so damage numbers never show in a capture. They do show in play (checked by switching that setting off for one test).

## Verify tomorrow (needs eyes, not scripts)
- The red recent-damage chunk on the duel card, and speed lines on a dash.
- The controls: touch buttons, the charge ring, and the PC ability bar and legend. Also the overlap check at 1400x900, 1920x1080 and a phone size (test 27).
- The classic sword swing still plays with every animation id at 0 (test 29).
- That the camera shake feels right and leaves no drift.
- Network use in a real duel (test 31: under 5 KB/s added).
- Tests that need 2 players: the repeat-pair rule (23), contests (32, 34) and sound scope.

## Before players see it
1. In Studio, run `workspace:SetAttribute("StudioNoSave", nil)` in the command bar. It is still on from the tests. Live servers ignore it, but Studio play tests won't save while it's on.
2. Close the extra Studio window. Overnight there were two Studio processes; close one normally (never force-quit).
3. Play a practice bout yourself and go through the "Verify tomorrow" list.
4. Publish. This ships M1, M2 and M3 together.

## Found later (M4 Studio pass, 4 Oct)
- **The arena could go black.** A fighter standing on the very edge of a perch (feet on it, body's centre past it) got a black screen: PlayerVision's single downward ray found nothing and read it as open air at 450 studs with no Aether. The server's thin-air check already skipped fighters (fa3527c); the client's vision did not. Fixed in `PlayerVision`: a fighter in the arena always counts as standing on sky ground. See [[M4 Verification]], fix 3.
