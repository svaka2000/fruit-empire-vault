---
tags: [roadmap, m9, verification]
date: 2026-10-09
---
# M9 Swimming without currents: verification

M9 is built, deployed to Studio and checked in a solo Play test. It is not published yet. Spec: [[Boats and Swimming]]. Code: `~/fruit-empire/src-m2` tag `m9-verified` (8b7a331), deployed together with the King removal (tag `mk-deployed`, 8af160d). Contract: `M9-CONTRACT.md` (section 8 = as built).

Every deployed script matches `src-m2` by length and hash: 26 of 26, checked in Studio after the deploy. Samarth's save was never written. Before and after both Play tests, `GardenProfiles` `p_1662286109` was 270 bytes, hash 2947522014, saveCount 662.

## What M9 changes
- **No more currents.** The fast water that pushed you out to the forest and the six lanes that carried you home are gone. The sea between the island and the forest is calm. You cross it by swimming.
- **A stamina bar.** In the water a bar counts down: 30 seconds of swimming, or 60 seconds of floating still. You swim at 12 studs per second. When the bar is low you are "Tired" (speed 8). When it is empty you are "Exhausted" (speed 4).
- **Wash-up, not drowning.** If you stay exhausted in the water for 4 seconds, the screen fades and you wake up on the last dry ground you stood on, with a full bar. You never die and never lose anything.
- **Places to rest.** Standing on anything solid (a stone, a log, an islet, a buoy, the beach) refills the bar in about 5 seconds. 16 new rest islets sit in a ring between the shallows and the forest. 12 red-and-white buoys mark where the six old lanes were.
- **Aqua Affinity potion.** A new potion in the shop. For 10 minutes you swim at 30 studs per second and the bar never drains. A blue wake follows you. Its price is never more than half a boat (75, 180, 380, then 500 shards).

## Tools run in Studio (Edit)
- `BuildAquaPotion` made the potion bottle (a copy of the Speed Potion with blue liquid) and its wake effect. Self-check ok.
- `SeaBuilder` moved the 60 current parts into `ServerStorage.Currents_backup_2026-10-04` (nothing deleted). It slowed the sea's shimmer, and built the 16 rest islets, the 12 buoys, the stamina bar's frame and the splash. Every check passed. The closest buoy is 26 studs from a stone, and the closest new part is 247 studs from a crate spot.

## Checked in Play (solo)
| What | Result |
|---|---|
| No currents left | 0 current plates anywhere; the seabed does not move. |
| Standing still in open sea | 0.00 studs of drift in 5 s, and 0.00 after 5 jumps. |
| Floating still | The bar drops 2.5 in 5 s (0.5 a second). |
| Swimming | About 1 a second, at 11.4 studs per second (12 set). |
| Body height | The root sits at y -2.59, just under the surface. |
| Running out | Tired (speed 8), then Exhausted (speed 4); the fade starts after 3.9 s; at 4.6 s you are back on your plot with a full bar. |
| Aqua Affinity | 600 s of effect, 30 studs per second, no drain, the wake on, the badge counting down. |
| Console | No game warnings. |

![[m9-swim-stamina-bar.jpg]]
*Swimming in open sea. The stamina bar sits above your hands, 53% left.*

![[m9-aqua-affinity-wake.jpg]]
*With Aqua Affinity: the blue wake behind you, the bar stays full, and the badge (bottom right) counts down from 10:00.*

## Before and after (same cameras)
![[m9-before-return-lane.jpg]]
![[m9-after-return-lane.jpg]]
*A return lane at the forest's beach: the turquoise strip and its sign are gone.*

![[m9-before-shallows-above.jpg]]
![[m9-after-shallows-above.jpg]]
*The shallows from above: calm water, the stones, and the new buoys (red and white).*

![[m9-before-swimmer-view.jpg]]
![[m9-after-swimmer-view.jpg]]
*What a swimmer sees heading out: no current strips.*

## Left for Samarth
- The full crossing at three bearings with a stopwatch (the Lune test says 59 to 71 s one way, 22 s with Aqua).
- Two players: the wake as seen by someone else.
- Phone screen: the bar against a long green notice.
- Design calls already built on their defaults: a carried Golden Crate also slows you in water (x0.85); the potion blurb reads "Swim 2.5x, nonstop"; the islets alternate between r 825 and r 815; swimming straight out beside an old lane is longer than one bar, so you wash back to the last stone.
