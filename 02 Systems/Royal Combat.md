---
tags: [system, design]
date: 2026-10-04
status: proposed
depends_on: [Rarity Gating and Rebirth, King UI Overhaul]
---
# Royal Combat

The fighting inside the Royal Arena: Crown Duels, Royal Trials, practice bouts, contest finals and the Crown Contest brawl. This note redesigns how the sword moves, how hits feel, adds health bars, rebuilds the Royal Slam and adds blade skins picked right before a fight. It changes the existing arena code; it does not replace the crown, the queue or the contest rules ([[Royal System - Server]]).

Everything here is a **proposal** unless it is marked "live" (meaning: it is in the 4 Oct 2026 script export today). Numbers marked "start at" are playtest starting points, not research facts.

## Why

Samarth's words ([[Brain Dump 2026-10-04]]):

> "for the royal sword during duels make it acctually animate like using a weopon and not just swinging forward and down everything i want like slashes and cool stuff. also make there by health drain as well. make the animations of it all look a lot better. same with the effects. the royal slam looks super AI, make it better. also offer like weopons they can choose from, like weopon skins, you select right before fighting."

> "https://create.roblox.com/store/asset/77935648543779/Sword-Animations"

> "https://create.roblox.com/store/asset/4697087599/Statue-With-Sword"

> "the animations need to be top tier."

> "make this super planned and tell me exactly what assets you need dont make slop unless it looks clean and studded."

> "what i noticed with the king feature in the sky island is that it was a bunch of dopamaine slop. instead everything should be structured. like for players its going to be confusing. all the UI should never overlap and also everything should be super clean."

> "notice how my Ui font is different from the clauded text you use. i want you to match my font and UI style. I dont like the emoji-slop that you pushed on the sky island kings feature."

> "also all Uis you make should follow the UI that we created, if you need any UI stuff then we can create them using chatgpt or gemini image generation following references of current UI"

> "everything needs to correspond with the stage of the game a player is in."

What the system is for. Today the duel uses Roblox's built-in tool "Slash" and "Lunge" (one forward-and-down chop), a sound per swing and a few sparkles per hit (`CrownArena.lua:229-243`, `RoyalUI.lua:1405-1421`). There is no health at all; only ring-outs score (`CrownArena.lua:4-7`). Royal Combat turns that into a readable sword fight: a 4-hit slash combo from the Sword Animations pack, real slash arcs, hit stop, layered sound, block and parry, health bars that drain, a Royal Slam that leaps and cracks the ground, and blade skins chosen on the pads before the countdown. The arena stays fair: nothing bought or earned changes damage, reach or speed (`CrownConfig.lua:16-19`).

### Words used here
- **Royal Arena**: the floating ring in the Royal Capital. **Fighter**: a player or a knight inside it.
- **Bout**: one duel in the arena. Kinds are `crown`, `trial`, `spar` (practice against a knight) and `final` (end of a contest) (`CrownArena.lua:18-23`, live).
- **Crown Point**: one point in a bout. **Ring-out**: knocking a fighter off the ring (live).
- **HP**: health points, 0 to 100, only inside the arena. **Knockdown (KO)**: HP reaching 0.
- **Combo**: taps of Attack chained in a row. **Step**: one swing of the combo (1, 2, 3, then 4, the **finisher**).
- **Heavy**: hold Attack, then release (live move).
- **Hitstun**: a short time after being hit when you cannot attack. **Stun**: a longer lock after your hit is parried.
- **Block**: hold the block key to take less damage and push (live move). **Parry**: a block pressed just before a hit lands. **Riposte**: your next hit after a parry, which does extra damage.
- **Hit stop**: freezing the attacker's swing animation for a few hundredths of a second when it connects ([[Top-Tier Sword Combat and Animation in Roblox]] §2).
- **Damage number**: a small floating number over the fighter who was hit.
- **Skin**: a blade look. It changes the sword model, its trail and spark colours, never the numbers.
- **Armory**: the panel where you pick or buy skins. **Pick phase**: up to 10 seconds on the pads, before the title card, when fighters choose a skin.
- **KeyframeSequence**: raw animation data that Studio can edit but a live game cannot play until it is published ([[Creator Store Packs]]). **Track**: a loaded animation you can play, stop and speed up. **Segment**: a time range inside one track. **Contact frame**: the moment in a swing where the blade crosses the target line.
- **Stage**: the newest rarity a player has unlocked; stage = rebirths + 1, from 1 (Common) to 9 (Transcendent) ([[Stage Model]]).
- **FM (frontier merge shards)**: the shards one merge into your newest rarity pays: 3, 7, 15, 32, 70, 155, 340, 748, 1,646 for stages 1 to 9 (decision D-010 in [[Decisions Log]]). Every shard amount in this note is a count of FM, so it scales with the player's stage.

## Player view

1. You challenge the King, answer a Royal Trial, or press the Sparring Post. The game teleports both fighters onto their pads in the arena (live).
2. The **Armory** panel opens in the middle of your screen with a 10-second timer in its title bar. It shows 9 blade cards. Blades you own are bright. Locked ones are dimmed and say what unlocks them, for example "Win 10 Crown Duels (4/10)".
3. You tap a blade. It appears in your character's hand on the pad at once. You press **READY**. When both fighters are ready (or the timer ends) the panel closes.
4. The title card plays ("CROWN CHALLENGE", live), your fighter does a short sword flourish, then 3, 2, 1, FIGHT (live).
5. You tap Attack. Your fighter slashes across, slashes back, cuts upward, then swings a wide finisher. Each landed hit: the swing freezes for a blink, a slash arc and a spark appear, a small number floats up, and the opponent's health bar drops, leaving a red chunk that drains away half a second later. The finisher throws them back hard.
6. You hold Attack and let go for a **heavy** strike, which also breaks through a block.
7. You hold F (or BLOCK on a phone) to block: you take a fifth of the damage and much less push. If you press it just before a hit lands, you **parry**: a clang, a white glint, the attacker is stunned for almost a second, and your next hit does 1.5 times damage.
8. You press E for the **Royal Slam**. You leap, a ring of dust rises at the edge of the slam's reach as a warning, you plunge, the stone cracks and everyone within 11 studs is thrown outward.
9. The lower your HP, the further hits throw you. You lose a Crown Point if you are knocked off the ring or if your HP reaches 0. After every point both fighters return to full HP. First to 3 points wins (live).
10. Outside fights you can look at every blade, buy shard blades and set your default at the **Royal Armory** stall beside the arena gate, or from a card in the Shop's King tab.

Spectators see the duel board at the top of their screen with both names, the score and both health bars, a small health bar over each fighter, the slashes, sparks and damage numbers.

## Rules

All rules are proposals. "Server" means the game server decides; a client only asks.

### A. Health and knockdown
1. Every fighter starts each bout, each Crown Point, each contest life and each new body with HP = 100 (`C.Combat.MaxHP`).
2. HP changes only on the server, only in `applyHit` (`CrownArena.lua:539-558`).
3. Damage dealt = move damage x block factor x riposte factor, rounded to the nearest whole number, at least 1, for every hit that is not parried, guarded or shielded.
4. A parried hit, a hit during Crown Guard and a hit during the respawn shield (`C.RespawnShield` 1.5 s, live) deal 0 damage and 0 push.
5. Knockback multiplier m = 1 + 1.6 x (1 - HP/100). This replaces stagger: m is 1.0 at full HP and 2.6 at 0 HP, the same range as today's `MaxStagger` 160 (`CrownConfig.lua:95`). Lift multiplier = min(1 + 0.8 x (1 - HP/100), 1.5), the same shape as today's lift rule (`CrownArena.lua:552`).
6. HP reaching 0 is a knockdown. In a bout, the other fighter scores one Crown Point through the same path as a ring-out, with reason `"ko"`. In a contest, the player is out for 1.5 s and returns at a contest spawn with full HP. A guard knight at 0 HP is removed, like a guard knight ring-out today (`CrownContest.lua:320-329`).
7. At every Crown Point, both fighters return to 100 HP.
8. The system never changes `Humanoid.Health`. (A death takes the "died" path, which waits up to 12 s for a new body: `CrownArena.lua:805-808`.)
9. The stagger percentage tag over fighters and the STAGGER line on the duel board are removed. Health bars replace them.

### B. The combo
10. Tapping Attack (released before 0.45 s) asks for the next combo step. Steps run 1, 2, 3, 4 (finisher), then back to 1.
11. A step is accepted only when its gap has passed since the previous swing (see Numbers). The next tap must arrive within 0.6 s after the previous segment ends, or the combo starts again at step 1.
12. After step 4, a new step 1 is refused for 0.6 s.
13. The server keeps the combo step. The step the client sends is only a hint and is ignored when it differs.
14. Each step resolves its hits after its own hit delay (the contact frame divided by playback speed). Targets use the live range, arc and 7-stud height checks (`CrownArena.lua:482-505`). Each target is hit at most once per swing.
15. A landed step 1, 2 or 3 gives the victim 0.30 s of hitstun (0.15 s in a contest). During hitstun the victim cannot attack, dash or slam and walks at half speed. The victim **can** block and parry, so every gap in a combo can be answered.
16. A finisher that hits nobody stuns the attacker for 0.45 s.
17. A swing whose attacker is stunned before its hit delay ends does nothing.

### C. Heavy strike
18. The heavy stays as today: hold 0.45 s, release, hit after 0.12 s (`CrownConfig.lua:105-114`). It now deals 16 damage. Through a block it deals 50% damage and 65% push (push is live).

### D. Block and parry
19. Block keeps its live limits: up to 1.6 s held, walking at 55% (`CrownConfig.lua:121-128`).
20. A parryable hit that lands within 0.30 s after the server received "block on" is parried. Combo steps, the heavy and knight strikes are parryable. The Royal Slam and the King's Shockwave are not.
21. On a parry: the defender takes 0 damage and 0 push; the attacker is stunned 0.8 s (no attack, dash, slam or block; walk at 50%) and their combo resets; the defender's next landed hit within 1.5 s deals 1.5 times damage.
22. Releasing block inside the 0.30 s window without having parried sets the block cooldown to 1.2 s instead of the live 0.9 s.
23. A blocked (not parried) hit deals 20% damage (combo steps 1 to 3) or 50% (finisher, heavy, slam) and pushes 30% (light) or 65% (heavy-class), using the live block numbers.

### E. Royal Slam
24. The slam keeps its live input (E, gamepad Y, touch SLAM), 10 s cooldown and Underdog cooldown cut (`CrownConfig.lua:129-136`, `:143`).
25. The slammer's own client leaps: up 50 studs/s and forward 10 studs/s at once, then down 95 studs/s at 0.20 s (start at). Knights do not slam.
26. The server resolves the slam 0.32 s after the request (start at), from the ground point under the slammer (a downward ray of up to 20 studs, ignoring fighters; if it hits nothing, the slammer's position minus 3 studs).
27. Every hittable fighter within 11 studs across and 7 studs up or down of that ground point takes 14 damage, push 92, lift 32, both scaled by m.
28. From the request to the impact, every client shows a dust ring at radius 11 on the ground as a warning.
29. The gold neon disc and the gold sparkle burst (`RoyalUI.lua:1425-1426`) are removed.

### F. Feedback
30. Hit stop freezes only the attacker's own animation track, on the attacker's own client: 0.06 s for steps 1 to 3, 0.10 s for the finisher, heavy and a parry, 0.12 s for the slam. It never pauses server logic.
31. The attacker's client predicts its own hits at the contact frame with the same range math. A predicted hit plays its hit stop, spark and impact sound at once. The server's `hit` event then adds the damage number and the bar change. If the server says it missed, nothing more happens (the prediction was cosmetic).
32. Damage numbers show for every hit that deals damage, to the attacker, the victim and anyone within 120 studs (60 studs in a contest). White for normal hits, Legendary gold for finisher, heavy, slam and riposte, Rare blue for blocked chip damage. A parry shows the word PARRY in white instead.
33. Camera shake on the victim: 0.6 degrees for steps 1 to 3, 1.2 for finisher and heavy, 2.0 for the slam (fading to 0 at 30 studs away). On the attacker: 0.25 and 0.5. Each lasts 0.18 to 0.35 s.
34. Sound layers: a whoosh on every swing; an impact only on a landed hit; a low thud added for finisher, heavy and slam. Each layer's pitch is randomised between 0.95 and 1.05. At most 3 sounds play per hit.
35. No spark, arc or number appears before the contact frame ([[Top-Tier Sword Combat and Animation in Roblox]] §4).
36. Each fighter shows a small health bar over the head to everyone except that fighter's own player, who reads it on the duel board.
37. No effect uses the `Blood-01` effect in the Anime pack, and nothing shows injuries. This keeps the game in the "Mild" violence band ([[Roblox Platform Constraints for the Expansion]] §10).

### G. Skins
38. Skins are cosmetic only. Range, arc, damage and timing never read the model. Every blade is 3.8 to 4.6 studs long from the grip, so no skin looks longer than its real reach.
39. A fighter can use only a skin they own. The server decides ownership from the unlock rules in Numbers.
40. Your equipped skin is saved (`royal.skin`) and is preselected in every pick phase.
41. Crown, trial and spar bouts begin with a pick phase of 10 s, before the title card. It ends early when every player fighter has pressed READY. Final bouts skip it (finalists keep their contest blade).
42. In a Crown Contest, the Armory opens for each entrant when they join, and for the King when the contest is announced. The blade selected when the brawl starts is used.
43. During the pick phase fighters stand frozen on their pads (`CrownArena.lua:140-144`, live), and nothing can be bought from the panel.
44. Outside the arena you can equip and buy skins from the Royal Armory stall prompt or the Shop's King tab card.
45. A shard skin costs `Config.ShardPrice(units, rebirths)`: its FM count x FM(your stage at the moment of purchase), rounded to 2 significant figures like every shard price (decision D-017). It is refused below its minimum stage.
46. A Robux skin is a game pass. Ownership is checked on join and when a purchase finishes.
47. Rebirth keeps every skin and `royal.skin` (decision D-009 keeps the royal record).

### H. Rewards
48. Every royal shard reward from duels, contests **and decrees** = `Config.ShardReward(units, rebirths)`: its FM count x FM(the receiving player's stage at payout), exact. No cash is ever paid (`CrownConfig.lua:256-259`, live rule). Decree units are in Numbers (added by the [[Integration Review]], conflict C27).
49. If the same two players already had a rewarded crown or trial duel against each other in the last 30 minutes (server memory), the next one pays no shards, no reputation and no duel-win count to either of them.
50. A forfeit pays nothing (live, `CrownArena.lua:1071-1089`).

### I. Screen and safety
51. While you are in the arena, the whole garden HUD except Mute is hidden: the list in [[King UI Overhaul]] rule 16 (side buttons including Pets, the Lock and Upgrade pills, CornerHud, the carry row, the effect block, the bat slot and bar, the Weapon Bar, SeaStatus and the touch action slot). All come back when you leave. (This note first kept cash and shards visible; changed to match the King UI rule in the [[Integration Review]], conflict C26.)
52. While the Armory panel is open, royal announcements wait in their queue and the royal top-centre stack is hidden on that screen.
53. While the Armory panel is open, the combat buttons and the ability bar are hidden.
54. Rebirth is refused while you are in the arena or busy in a duel, contest or practice bout.

## Numbers

### Combat moves (the same for every player, by design)
These are not tied to stage: the arena rule says everyone fights with the same numbers (`CrownConfig.lua:16-19`, live). Live values are marked; the rest start at these values.

| Move | Input | Hit delay after input | Damage | Push (studs/s) | Lift | Range (studs) | Arc (deg either side) | Gap before next | Victim hitstun |
|---|---|---|---|---|---|---|---|---|---|
| Combo step 1 | tap | 0.24 s | 7 | 18 | 6 | 7.5 | 70 | 0.40 s | 0.30 s |
| Combo step 2 | tap within chain | 0.26 s | 7 | 20 | 6 | 7.5 | 70 | 0.44 s | 0.30 s |
| Combo step 3 | tap within chain | 0.28 s | 8 | 24 | 10 | 8.0 | 55 | 0.46 s | 0.30 s |
| Finisher (step 4) | tap within chain | 0.31 s | 12 | 70 | 26 | 8.5 | 80 | 0.60 s reset | none (launch) |
| Heavy | hold 0.45 s, release | 0.12 s (live) | 16 | 74 (live) | 26 (live) | 8.5 (live) | 60 (live) | 1.25 s (live) | none |
| Royal Slam | E | 0.32 s | 14 | 92 (live) | 32 (live) | radius 11 (live) | all round (live) | 10 s (live) | none |
| King's Shockwave (contest only) | E | 0 | 10 | 88 (live) | 30 (live) | radius 15 (live) | all round | 8 s (live) | none |
| Knight strike | AI | 0.15 s (live) | 6 | 34 (live) | 14 (live) | 7 (live) | 70 | 1.1 s (live) | 0.20 s |

- A full combo deals 7 + 7 + 8 + 12 = **34**. Hits to knock down from full HP: 3 full combos, or 7 heavies, or 8 slams.
- Blocked: step 1 = 1, step 2 = 1, step 3 = 2, finisher = 6, heavy = 8, slam = 7. Riposte step 1 = 11.
- Parry: window 0.30 s, attacker stun 0.8 s, riposte x1.5 for 1.5 s, whiff cooldown 1.2 s.
- Hit stop: 0.06 / 0.10 / 0.12 s. Camera shake: see rule 33.
- Dash, Underdog and Crown Guard keep their live numbers (`CrownConfig.lua:115-152`).

### HP and how far a finisher throws
m = 1 + 1.6 x (1 - HP/100); lift factor = min(1 + 0.8 x (1 - HP/100), 1.5). Air distance assumes the default gravity of 196.2 studs/s^2 (no script changes it; the place value is not visible from the export) and ignores sliding.

| HP | m | Finisher speed | Finisher lift | Air time | Air distance |
|---|---|---|---|---|---|
| 100 | 1.0 | 70 | 26.0 | 0.27 s | 19 studs |
| 75 | 1.4 | 98 | 31.2 | 0.32 s | 31 studs |
| 50 | 1.8 | 126 | 36.4 | 0.37 s | 47 studs |
| 25 | 2.2 | 154 | 39.0 | 0.40 s | 61 studs |

The ring has radius 30 and the pads sit 17 studs from the centre (`RoyalCapitalBuilder.lua:1595-1596`). So below about 75 HP a finisher toward the near edge is a ring-out. Combo steps 1 to 3 at full HP move the victim only 1 to 3 studs, so the combo stays in reach. Knockdowns end fights where both players hold the centre.

### Duel length (estimate to check in playtest)
The target is 45 to 90 seconds per duel (`docs/KINGS-THRONE.md:287`, the live tuning note). Estimate: with about half of swings landing, one point takes 10 to 20 s (two to three combos, or a ring-out once HP is under 75). Three points plus two 1.6 s pauses (live) gives about **35 to 70 s** of fighting. The pick phase adds at most 10 s before the title card. Tune with `C.Combat.Combo.steps[n].damage` first, then the finisher push.

### Splitting the combo clip (swordcombo1, 2.82 s)
The clip is one 2.82 s KeyframeSequence that loops at Idle priority ([[Creator Store Packs]]). The game plays it as 4 segments of one track. Code forces `Priority = Action2` and `Looped = false` at load, so Samarth does not need to change those settings before publishing.

**How to find the split points (done by the builder in Studio, from the quarantined pack, no publishing needed).** Register the KeyframeSequence for a test with `KeyframeSequenceProvider:RegisterKeyframeSequence` on an R15 dummy ([[Top-Tier Sword Combat and Animation in Roblox]] §13), play it at speed 1 and sample the right hand's world position every 1/60 s. Each swing is a peak in hand speed. Put each **hit** at a speed peak and each **split** at the speed low point between two peaks. Write the times into the config table. Optional: Samarth may also add Animation Editor markers named `Hit` (value `1` to `4`) at the same times; the config stays the source of truth because the server needs the times too.

The values below assume 4 evenly spaced swings. They are placeholders until measured. Playback speed 1.15 (start at); hit delay = (hit - start) / 1.15.

| Step | Segment start | Contact (hit) | Segment end | Hit delay | Slash arc roll (deg) | Direction (to confirm) |
|---|---|---|---|---|---|---|
| 1 | 0.00 s | 0.28 s | 0.62 s | 0.24 s | 0 | across, left to right |
| 2 | 0.62 s | 0.92 s | 1.30 s | 0.26 s | 180 | back, right to left |
| 3 | 1.30 s | 1.62 s | 2.00 s | 0.28 s | 60 | rising diagonal |
| 4 | 2.00 s | 2.36 s | 2.82 s | 0.31 s | 90 | wide overhead finisher |

- Gap before the next step = segment length / 1.15 x 0.75.
- Heavy: while charging, the client holds the step 4 pose frozen 0.05 s into the segment; on release it plays from (step 4 hit - 0.12 x 1.15), so the blade connects exactly at the live 0.12 s windup.
- Slam: plays step 3 from its start, freezes at the top of the swing at 0.14 s, then resumes at speed 1.6 when the plunge starts at 0.20 s.
- If the clip has only 3 swings, step 3 becomes the finisher and the combo is 3 steps (see Open questions).

The other five clips: `Sword Idle 1` (3.00 s) replaces the arena idle; `Sword Idle 2` (3.00 s, Action priority) plays once as the flourish during the 3.5 s title card; `Sword Walk` (0.93 s) and `Sword Run` (0.67 s) replace walk and run; `block` (one pose) plays while blocking.

### The shard unit, by stage
FM comes from decision D-010. Merging rate is measured from the stage model's simulation (the attentive player who uses pets and ores, median of 7 careers; script in the scratchpad, using `~/fruit-empire/sim/economy.py` unchanged). Stage 9 was not simulated; its rate is assumed equal to stage 8.

| Stage | Rarity | Full-plot $/s | FM (shards per frontier merge) | Frontier merges per minute | Shards per minute from merging |
|---|---|---|---|---|---|
| 1 | Common | 707 | 3 | 4.54 | 13.6 |
| 2 | Uncommon | 430K | 7 | 5.21 | 36.5 |
| 3 | Rare | 3.7B | 15 | 6.51 | 97.6 |
| 4 | Epic | 3.6T | 32 | 5.12 | 163.7 |
| 5 | Legendary | 1.8Qa | 70 | 4.33 | 302.9 |
| 6 | Mythic | 2.5Qi | 155 | 4.73 | 732.8 |
| 7 | Astral | 265Qi | 340 | 2.63 | 894.7 |
| 8 | Eternal | 506Sx | 748 | 3.39 | 2,535 |
| 9 | Transcendent | 24.9Sp | 1,646 | 3.39 (assumed) | 5,580 (assumed) |

Full-plot $/s is shown for reference only: the arena pays no cash.

### Royal rewards (replace the flat 100 to 3,000 shards in `CrownConfig.lua:261-268`)
Formula: shards = units x FM(receiver's stage).

| Reward | Units | Stage 1 | Stage 2 | Stage 3 | Stage 4 | Stage 5 | Stage 6 | Stage 7 | Stage 8 | Stage 9 | Live today |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Defense (King wins a duel) | 10 | 30 | 70 | 150 | 320 | 700 | 1,550 | 3,400 | 7,480 | 16,460 | 500 |
| Victory (challenger takes the crown) | 25 | 75 | 175 | 375 | 800 | 1,750 | 3,875 | 8,500 | 18,700 | 41,150 | 1,000 |
| Contest win | 40 | 120 | 280 | 600 | 1,280 | 2,800 | 6,200 | 13,600 | 29,920 | 65,840 | 3,000 |
| Contest survive (King holds) | 20 | 60 | 140 | 300 | 640 | 1,400 | 3,100 | 6,800 | 14,960 | 32,920 | 1,500 |
| Contest entry | 4 | 12 | 28 | 60 | 128 | 280 | 620 | 1,360 | 2,992 | 6,584 | 100 |
| First practice win (once ever) | 10 | 30 | 70 | 150 | 320 | 700 | 1,550 | 3,400 | 7,480 | 16,460 | 150 |
| Decree: Merge Rush (each merger) | 10 | 30 | 70 | 150 | 320 | 700 | 1,550 | 3,400 | 7,480 | 16,460 | 400 |
| Decree: Royal Race 1st / 2nd / 3rd | 25 / 15 / 8 | 75 / 45 / 24 | 175 / 105 / 56 | 375 / 225 / 120 | 800 / 480 / 256 | 1,750 / 1,050 / 560 | 3,875 / 2,325 / 1,240 | 8,500 / 5,100 / 2,720 | 18,700 / 11,220 / 5,984 | 41,150 / 24,690 / 13,168 | 900 / 500 / 250 |
| Decree: Crown Hunt (finder) | 20 | 60 | 140 | 300 | 640 | 1,400 | 3,100 | 6,800 | 14,960 | 32,920 | 750 |
| Decree: Empire Build (each player) | 10 | 30 | 70 | 150 | 320 | 700 | 1,550 | 3,400 | 7,480 | 16,460 | 350 |

Why these counts: the live config says fighting must never pay more than playing the economy (`CrownConfig.lua:256-259`). One reward equals a few minutes of frontier merging at the same stage: a defense is 1.5 to 3.8 minutes, a victory 3.8 to 9.5 minutes, contest entry under 1.5 minutes. Sustained, a King can defend at most once per duel plus 120 s of protection (`CrownConfig.lua:52`, live), about every 4 minutes: 10 / 4 = 2.5 FM per minute, below the merging rate at every stage (2.63 to 6.51). Contests run every 18 to 24 minutes (`CrownConfig.lua:173-174`, live), so a contest winner averages at most 40 / 18 = 2.2 FM per minute. Rule 49 stops two friends from trading wins.

### Skins and how they unlock
| Skin | Card frame (rarity art) | Model | Look | Unlock |
|---|---|---|---|---|
| Royal Sword (default) | COMMON | classic sword mesh | today's sword; white trail; gold sparks | free |
| Rarity Blade | your stage's rarity | classic sword mesh | tinted to your stage's rarity colour; trail and sparks in that colour; from stage 7 a faint glint | free; recolours on every rebirth |
| Knight's Longsword | UNCOMMON | built from studded parts | knight grey blade, gold guard and pommel; white trail | win 3 practice bouts (`royal.spars` >= 3) |
| Katana | RARE | katana from the Sword Animations rigs | silver; pale trail; finisher wind puff | Royal Reputation 100 |
| Shadow Blade | EPIC | katana, recoloured | dark steel; purple trail; smoke wisps | Conqueror achievement (10 Crown Duel wins, live achievement) |
| Crown Blade | LEGENDARY | classic sword mesh | gold; trail in your crown style's gem colour | First Crown achievement (be King once, live achievement) |
| Statue Greatsword | MYTHIC | blade from Statue With Sword | studded stone blade; ember trail; small flame | 150 FM, from stage 3 |
| Celestial Saber | ASTRAL | classic sword mesh | white-blue; blue trail; star motes | 200 FM, from stage 6 |
| Storm Edge | ETERNAL | classic sword mesh | pale cyan; crackling lightning; cyan trail | Robux game pass, 149 R$ (proposal) |

Rarity colours: read `Config.RarityColors` (`ReplicatedStorage.GardenConfig.lua:582-592`), the same table the bat, gate banners and boat trim use: Common 151,169,189; Uncommon 78,194,101; Rare 78,164,230; Epic 170,93,222; Legendary 242,171,48; Mythic 232,66,60; Astral 71,222,217; Eternal 237,81,195; Transcendent 252,247,227. The Rarity Blade uses the same colour as the player's bat ([[Bats]]).

Shard skin prices, worked (percent = share of the median shards held at that stage's rebirth gate, [[Stage Model]]):

| Skin | Units | Min stage | Stage 1 | Stage 3 | Stage 6 | Stage 7 | Stage 8 | Stage 9 |
|---|---|---|---|---|---|---|---|---|
| Statue Greatsword | 150 | 3 | locked | 2,300 (35%) | 23,000 (27%) | 51,000 (46%) | 110,000 (25%) | 250,000 |
| Celestial Saber | 200 | 6 | locked | locked | 31,000 (36%) | 68,000 (61%) | 150,000 (34%) | 330,000 |

That is 23 to 44 minutes of frontier merging for the Greatsword and 42 to 76 minutes for the Saber. Both are sinks for the late-stage shard surplus the Stage Model expects ("cosmetics (bat skins, weapon skins)").

## UI

All UI uses [[UI Kit Inventory]]: FredokaOne, white text with a black `UIStroke` of 2.5 px, `TextScaled`, kit images for every panel, card and button. No emoji anywhere, including the strings in `CrownConfig.lua` that this system shows (the King UI Overhaul removes the rest). No middle-dot characters in new strings (the code map flags that the font may not draw them).

### Scale and layering
- Sizes below are design pixels at the HUD reference of 1600 x 900. Every new screen element uses `Kit.scale()` from the [[King UI Overhaul]] (`Config.HudScaleFor`, 0.55 to 1.0), the same curve as the garden HUD, so it scales with the royal elements it must avoid. (This note first used RoyalUI's own 1400 x 900 `fitScale`; changed in the [[Integration Review]], conflict C24.)
- New ScreenGui `ArmoryUI`, DisplayOrder 20: the modal band, with the Kingdom panel ([[King UI Overhaul]] section 4.7). One modal at a time: opening the Armory hides the other panels in `Kit.Modals`.
- The panel closes itself when the King's challenge card arrives (`prompt` event), the same way RoyalUI hides its menu (`RoyalUI.lua:1356`).

### Duel board health bars (top centre)
- The duel board is the [[King UI Overhaul]] event strip card: top edge at 112 px (not scaled), 560 x 96, with the kit blue title bar as its 28 px header. This system owns only the bars and lines inside it. (This note first kept RoyalUI's old stack at 70 px and grew the board to 560 x 112; the King UI layout wins, conflict C24.)
- Two health bars sit in the card's 20 px sub row: A at the left, B at the right, each **196 x 20**, so they never touch the centre score column (x 213 to 347 of the card).
- Each bar: the shared track and fill, `UIKit.Images.BarTrack` and `BarFill` ([[King UI Overhaul]] A8 and A9, decision D-027). Fill colour: Uncommon green above 50 HP, Legendary gold from 25 to 50, Mythic red under 25 (the same bands as fruit and boat bars). Behind the fill, a Mythic red "recent damage" chunk holds for 0.35 s, then shrinks to the fill over 0.25 s.
- The HP number sits inside the bar (A at its right end, B at its left end), FredokaOne, Small (17).
- "Underdog" moves under the bar on the underdog's side, Small, no emoji, white (the King UI text colour rule).
- The main row's sub line reads "Choosing blades 8" during the pick phase, then "First to 3" or "Sudden death" (live).
- Until the bar art arrives: track and fill = the CarryInfo RateBar image (120297790302429), the fill tinted and clipped.

### Health bar over each fighter
- Replaces the stagger tag BillboardGui (`RoyalUI.lua:1006-1049`). Size 90 x 12 px, 2.6 studs above the head, `MaxDistance` 160, `LightInfluence` 0, not always on top.
- The same frame and fill images as the board, same colour bands. No text.
- Shown for every fighter, including knights (found by the `RoyalKnight` model attribute, `CrownArena.lua:417`), except your own character.
- The block and Crown Guard outlines stay (live). A stunned fighter gets a grey outline for the stun time.

### Damage numbers
- BillboardGui 90 x 40 px on the victim's root part, `AlwaysOnTop`, `MaxDistance` 120 (60 in a contest).
- Starts at a random offset of up to 1 stud sideways, 2.4 studs up; rises 2.5 studs over 0.7 s (Quad Out); fades out over the last 0.25 s.
- FredokaOne, `TextScaled`, max text size 30 (36 for gold numbers), black stroke 2.5. Colours from rule 32.
- A pool of 12 billboards per client is reused; when all 12 are busy the oldest is recycled.

### Armory panel (pick mode and browse mode)
Centred at (0.5, 0.56), **760 x 430** total. Every piece is a kit image (native size scaled by 0.80):

| Part | Kit image | Position and size |
|---|---|---|
| Title frame | IndexMenu.TitleFrame 110138832714236 | top, 760 x 59 |
| Title bar | IndexMenu.TitleBlue 95532041481396 | inside the frame, 752 x 50 |
| Title word "ARMORY" | new art, made like IndexMenu.TitleText 103221050515090 | same place as INDEX in the Index panel, 161 x 59. Fallback until delivered: FredokaOne "ARMORY" on the blue bar (allowed by the kit note) |
| Timer (pick mode) | FredokaOne text "FIGHT IN 8", 180 x 36 | right end of the title bar |
| Close (browse mode) | Red X close (square) 107111219044453 | right end of the title bar, 48 x 48 |
| Body | IndexMenu.Body 82827190351667, with IndexMenu.Inline 110403403749726 | below the title, 754 x 369 |
| Blade grid | IndexMenu.ScrollBox 83403190365135 | (16, 18) in the body, 552 x 313 |
| Blade cell | the rarity card frame of the skin (Numbers table) | 100 x 100, 5 columns x 2 rows, 10 px across, 14 px down; name label under each, 100 x 20, max text 16 |
| Cell preview | ViewportFrame inside the cell, slowly turning blade (ShopMenu pattern, `ShopMenu.lua:251-292`) | 80 x 80 |
| Selected cell | Card selection outline 113285622864672 | over the cell |
| Info box | IndexMenu.InfoBox 101725900357985 | (584, 18), 154 x 180: large preview 130 x 110, name, unlock line ("Win 3 practice bouts (1/3)"), extras line ("Trail  Sparks  Glow") |
| Action box | IndexMenu.RewardsBox 126819257706583 | (584, 206), 154 x 128 |
| READY / EQUIP / EQUIPPED / LOCKED | UpgradeMenu Buy button art 112539925904292 (blank green; UpgradeMenu writes its text in code, `UpgradeMenu.lua:74`) with FredokaOne label | 144 x 38 in the action box. EQUIPPED and LOCKED use the Index's grey tint (`IndexMenu.lua:270`) |
| Shard price | ShopMenu Buy button 97067256017679 (shard crystal baked on the left, price in code, `ShopMenu.lua:300-330`) | 144 x 51; price text turns red when you cannot afford it, like UpgradeMenu (`UpgradeMenu.lua:73-75`) |
| Robux price | UpgradeMenu Buy button art with FredokaOne "R$ 149" | 144 x 38 |

States:
- Owned cell: full colour. Locked cell: image tinted grey, preview darkened, unlock line visible. Equipped cell: a FredokaOne "EQUIPPED" line in Uncommon green under the name.
- Pick mode: only READY or LOCKED appear in the action box. No buying. A small "OPPONENT READY" line (FredokaOne 14) appears in the action box when the other fighter is ready.
- Browse mode: EQUIP, EQUIPPED, a shard price, a Robux price, or a progress line for reputation and achievement skins.
- Open and close use the kit panels' tween: UIScale 0.85 to 1 (`ShopMenu.lua:604-633`). Opening the Armory closes any open garden panel.

Entry points: the pick phase (automatic), the Royal Armory stall prompt (`Kind = "Armory"`), and one card on the Shop's King page (`ShopMenu.lua:400-594`) built like the King aura cards, with the Katana as its preview.

### Touch combat buttons
- Positions stay exactly as today, around the jump button (`RoyalCombat.lua:284-315`): Attack 128, Block 92, Dash 88, Slam 92, Guard 78, Summon 78 px, x0.72 on small screens.
- Each becomes an ImageButton with the [[King UI Overhaul]] combat images F1 to F7 (icon with the word baked in: Attack, Block, Dash, Slam or Wave, Guard, Knights). The emoji labels at `RoyalCombat.lua:259-264` and `:343` go. (This note's own image-sheet request with code labels is withdrawn, conflict C4.)
- Cooldown: the same image tinted black at 45% inside a clipping frame that shrinks from the top (replaces the code-drawn frame at `RoyalCombat.lua:226-238`).
- Charge ring: King UI image A10, 150 x 150, replaces the code-drawn ring (`RoyalCombat.lua:267-281`).
- Hidden during the pick phase and while the Armory is open.

### PC ability bar and legend
- Ability bar, bottom centre, anchor (0.5, 1), 54 px above the bottom edge: 3 slots of 64 x 64 with 10 px gaps (212 x 64): BLOCK (F), DASH (Q), SLAM (E). In a contest the King gets 5 slots (360 x 64): BLOCK, DASH, WAVE (E), GUARD (R), KNIGHTS (T).
- Each slot: the same F image as the touch button, the key letter in FredokaOne 18 at its lower right, and the same cooldown sweep.
- Legend: the [[King UI Overhaul]] rule 38 row of name plates at 16 px up (16 to 46 px) replaces this note's text hint line, so the two never overlap. Full opacity during pick, title card and countdown; 50% transparency during the fight. Hidden on touch. Replaces `RoyalCombat.lua:197-198` and `:335-336`.

### Arena focus (why nothing overlaps)
While `InArena` is true, the arena client hides the clusters in [[King UI Overhaul]] rule 16 by `Visible`: every garden HUD cluster except Mute, including the expansion's PetsButton, BatSlot, BatInfo, WeaponBar, SeaStatus and the BatControls ScreenGui (names from `Config.HudClusters`, `GardenConfig.lua:1863-1867`). One list, one owner (the King UI's `RoyalUI.Hud` module); this script does not keep its own copy. HudScale only writes Position and Size, so it does not fight this (`HudScale.lua:116-127`).

This fixes a real overlap: on a phone in landscape (844 x 390), the live DASH and GUARD buttons land on the garden's Clear and Merge All buttons in the right column.

| Element | Where | Checked against | Result |
|---|---|---|---|
| Armory panel (pick) | centre, top edge 0.56 x H - 215 x scale | top-left royal chip cluster (King UI: chip hidden in the arena by rule 16; outside, chip ends at x 302) | the chip cluster is hidden while any modal is open (King UI rule 15) |
| Armory panel | centre | royal top-centre stack | the stack is hidden while the panel is open (rule 52) |
| Armory panel | centre | announcement cards (0.16 x H) | announcements wait while the panel is open (rule 52) |
| Armory panel | centre | King challenge card (0.5, 0.55) | the panel closes when the card arrives |
| Armory panel (browse) | centre | garden left column (x 16) and CornerHud | at 1400 x 900 the panel spans x 320 to 1080; the left column ends at about x 112 |
| Ability bar and legend | bottom centre, 54 to 118 px and 16 to 46 px | CarryButtons, CarryInfo, Weapon Bar | all hidden in the arena (rule 51); bar and legend 8 px apart |
| Touch buttons | around the jump button | garden right column | hidden in the arena (rule 51) |
| Board health bars | inside the board | score column | 5 px clear on each side |

## World and assets

### Blades
| Skin | Source in the place | Build work |
|---|---|---|
| Royal Sword, Rarity Blade, Crown Blade, Celestial Saber, Storm Edge | classic sword mesh `rbxasset://fonts/sword.mesh` + `SwordTexture.png`, as built today (`CrownArena.lua:188-191`) | tint with `SpecialMesh.VertexColor` (the same trick planned for the bat, [[Existing VFX and Props]]) |
| Katana, Shadow Blade | the katana held by each rig in `ServerStorage.AssetQuarantine.Q_77935648543779` (Sword Animations) | copy one katana into a Tool. Its grip offset should match the source rig's right hand, because the clips were made for it. Shadow Blade recolours its parts to 60,58,70 |
| Statue Greatsword | the sword parts of `Q_4697087599` (Statue With Sword), plus its `Fire` | copy the blade, scale to 4.5 studs, keep the Fire at Size 1.5 and Heat 3, on only while fighting |
| Knight's Longsword | none | build 4 parts with Studs on top, like the knights (`CrownArena.lua:339-350`): blade 0.3 x 0.8 x 3.6 in armour grey 150,150,158; guard 2.0 x 0.3 x 0.4 in gold 255,198,46; grip 0.35 x 0.35 x 0.9 in wood brown; pommel 0.5 ball in gold. Knights carry this blade too |

Every skin Tool keeps the name `RoyalSword` (three functions find it by name, `CrownArena.lua:201`, `:224`, `:231`) and carries two attachments, `TrailBase` (0.6 studs above the grip) and `TrailTip` (at the tip), for its Trail.

### Effects (all already in the place, [[Existing VFX and Props]])
They sit in a staging area around y = -790 in Workspace. Copy the ones listed into a new folder `ReplicatedStorage.RoyalFXAssets`, because ReplicatedStorage is never streamed out and Workspace can be ([[Roblox Platform Constraints for the Expansion]] §2). Every emitter is set to `Rate = 0` and fired with `:Emit(n)` unless the table says "looping".

| Effect | From | Used for | Emit |
|---|---|---|---|
| Slash-01 | Anime | the slash arc on each combo step, rolled per step (arc roll column), recoloured to the skin's trail colour | 1 burst per step |
| Shield-01 | Anime | block: a faint shield in front of the blocker; Crown Guard: the same in gold for 2.5 s | looping while held, rate 6 |
| Shield-Break-01 | Anime | parry glint | 1 burst |
| Smoke-01 | Anime | slam warning ring (12 puffs at radius 11); Shadow Blade wisps | 12 puffs; wisps looping at rate 4 |
| Wind-02 | Anime | Katana finisher puff; King's Shockwave swirl | 1 burst |
| Shiny-01 | Anime | Rarity Blade glint from stage 7; Crown Blade glint | looping, rate 3 |
| Stars-01 | Anime | Celestial Saber motes | looping, rate 3 |
| Lighting-01, Lighting-02 | Anime | Storm Edge crackle (01, looping rate 2); its finisher burst (02) | as stated |
| Portal-01 | Anime | Royal Guard knights appearing at their posts | 1 burst |
| Big-Crack-01 | Big | Royal Slam ground crack, laid flat at the ground point | 1 burst |
| Explosion (shockwave, shards and sparks emitters only) | vfx pack | Royal Slam dust ring and stone chips, recoloured to stone grey and dust beige (not gold) | 1 burst each |
| f (impact, flash and rocks emitters) | vfx pack | hit spark (impact + flash, skin colour), slam rocks | 8 sparks + 1 flash per hit |
| Part (speed lines) | vfx pack | dash, and a finisher victim's first 0.2 s in the air | 0.18 s burst |
| Trail (new instance) | none | one per blade, between TrailBase and TrailTip; lifetime 0.14 s; transparency 0.2 to 1; on only from 0.12 s before to 0.10 s after each contact | per swing |
| PointLight (new instance) | none | slam impact flash: brightness 6 to 0 over 0.2 s, range 16 | per slam |

Not used: `Blood-01` (rating), and the RoyalUI gold neon ring (`RoyalUI.lua:1096-1103`) for the slam and the Shockwave.

### Sounds
Existing ids only for the first build. Each layer's pitch is randomised 0.95 to 1.05.

| Moment | Layers |
|---|---|
| Combo swing | `Swing` 123796710194563 (0.6) + `rbxasset://sounds/swordslash.wav` (0.3) |
| Finisher or heavy swing | `HeavySwing` 137628815514180 (0.6) + `rbxasset://sounds/swordlunge.wav` (0.35) |
| Landed hit | `Hit` 126976659766267 (0.65) |
| Finisher, heavy or slam landed | add the bat's `Hit` 175024455 (0.4) as the low thud (candidate; confirm by ear) |
| Block | `Block` 9125740334 (0.5) |
| Parry | `Block` (0.6, pitch 1.25) + `Sparkle` 4612374393 (0.35, pitch 1.5) |
| Royal Slam impact | `Slam` 79476956608476 (1.2) + `swordlunge.wav` (0.4, pitch 0.75) |
| Dash | `Dash` 138890894929151 (live) |

The `C.Sounds` ids are licensed Roblox audio checked on 3 Oct 2026 (`CrownConfig.lua:363-383`). The `rbxasset://` sounds ship with every client ([[Top-Tier Sword Combat and Animation in Roblox]] §6).

### The Royal Armory stall
- A clone of the **Stud Shop** booth (`Q_12340318191`, 30 parts) placed on the other side of the arena gate from the Sparring Post. The Sparring Post is at (gate x + 22, gate z + 6) (`RoyalCapitalBuilder.lua:1611`); the stall goes at (gate x - 22, gate z + 6), checked in Studio for clearance. Roof recoloured royal green 40,127,71 with gold trim 255,198,46. All parts anchored; any Humanoid in the shopkeeper removed (a static NPC does not need one, [[Roblox Platform Constraints for the Expansion]] §4).
- A weapon rack to build: one studded plank 12 x 6 x 0.6 studs with 9 pegs, holding an anchored display copy of each blade (CanCollide, CanQuery and CanTouch off).
- A marker part `Armory` in `Royal Capital.Markers` with a `RoyalPrompt` (Kind `Armory`, "Open the Armory", hold 0.4 s, distance 10), using the same marker and prompt conventions as `RoyalCapitalBuilder.lua:490-508`. Crown wires it on load (`Crown.lua:972-987`, live).
- Two **Statue With Sword** copies (`Q_4697087599`, 8 parts each) flank the arena path entrance, the planned use in [[Creator Store Packs]].

### What must be built (summary)
The Knight's Longsword (4 parts), the weapon rack (10 parts), the stall placement, the `ArmoryBuilder` dev script, the `RoyalFXAssets` folder copies, and the 9 skin Tools.

## Asset requests for Samarth

### 1. Publish the six sword animations (needed; decision D-005)
Only the account or group that owns the game can make these playable ([[Top-Tier Sword Combat and Animation in Roblox]] §12). The game is owned by the user `hikeALT12212`.
1. In Studio, open `ServerStorage.AssetQuarantine.Q_77935648543779`. Each of the 6 rigs holds one KeyframeSequence.
2. Avatar tab, Animation Editor, click a rig, then the three-dot menu, Load, and pick that rig's KeyframeSequence.
3. Three-dot menu, Publish to Roblox. Title them `FE Sword Idle 1`, `FE Sword Idle 2`, `FE Sword Walk`, `FE Sword Run`, `FE Sword Block`, `FE Sword Combo`. Creator: the owner account. If you publish from your own account instead, run one Studio playtest of the game afterwards so the game is granted permission.
4. Send me the 6 ids. They go into `C.Anims`. Until then every id stays 0 and the duel falls back to today's swing, with all new effects working.
You do not need to change priority, looping or add markers; the code sets those.

### 2. Image: "ARMORY" panel title
Made in the [[King UI Overhaul]] Figma title batch as T10 (`title_armory.png`, its section 7.3), with every other INDEX-style title. The ChatGPT prompt that was here is withdrawn ([[Integration Review]], conflict C4). Until it exists, FredokaOne "ARMORY" on the blue bar.

### 3. Combat buttons and charge ring
No separate request. The [[King UI Overhaul]] makes them: F1 to F7 (icon plus the word baked in, its section 7.10) and A10 (the charge ring, section 7.5). The 3 x 3 sheet prompt that was here is withdrawn so the same buttons are not made twice in two styles.

### 4. Health bar frame and fill
No separate request. Every bar in the expansion uses the [[King UI Overhaul]] pair A8 and A9 (336 x 54, from CarryInfo.RateBar 120297790302429; decision D-027). The CashBar-based prompt that was here is withdrawn.

### 5. Robux game pass: Storm Edge (optional; recommended)
Creator Hub, the game, Monetization, Passes, Create a Pass. Name "Storm Edge", description "A royal blade with crackling lightning. Looks only: it does not change damage or reach." Price 149 Robux (proposal). Icon prompt (attach `style-reference.png`):
```
Make a 512x512 PNG game pass icon with a transparent background: a pale cyan sword pointing up and slightly tilted, with small white-and-cyan lightning bolts crackling around the blade. Glossy chunky 3D cartoon style like the attached icons, thick near-black outline, saturated colours. No text, no letters, no numbers.
```
Send me the pass id. It goes into `C.WeaponSkinByKey.StormEdge.unlock.gamepass`. It is a fixed item, not a random one, so no odds screen is needed ([[Roblox Platform Constraints for the Expansion]] §10).

### 6. Sounds (optional)
If you want richer impacts later, pick three licensed sounds in the Creator Store audio tab and send their ids: search "stone impact boom" (slam), "metal ting short" (parry), "crowd ooh" (a ring-out or knockdown in the stands). The first build works without them.

### 7. A hand-made Royal Slam animation (optional, only if needed)
Only if the leap built from the combo clip reads badly in playtest: a 0.6 s R15 animation of a jump with the sword raised and a downward plunge, published like step 1, Action priority.

## Code plan

Follows the GardenService pattern: each server module is `return function(G)`, loaded in order, sharing state through `G` (`Crown.lua:28`, `GardenService.lua:515-518`). Earlier modules look up later `G` functions only at call time.

### File names used below
| Short name | Export file |
|---|---|
| `CrownArena.lua` | ServerScriptService.GardenService.CrownArena.lua |
| `CrownContest.lua` | ServerScriptService.GardenService.CrownContest.lua |
| `Crown.lua` | ServerScriptService.GardenService.Crown.lua |
| `CrownConfig.lua` | ReplicatedStorage.CrownConfig.lua |
| `RoyalUI.lua` | StarterPlayer.StarterPlayerScripts.RoyalUI.lua |
| `RoyalCombat.lua` | StarterPlayer.StarterPlayerScripts.RoyalCombat.lua |
| `GardenStore.lua` | ServerScriptService.GardenStore.lua |
| `GardenService.lua` | ServerScriptService.GardenService.lua |
| `RebirthLock.lua` | ServerScriptService.GardenService.RebirthLock.lua |
| `ShopMenu.lua` | StarterPlayer.StarterPlayerScripts.ShopMenu.lua |
| `HudScale.lua` | StarterPlayer.StarterPlayerScripts.HudScale.lua |
| `GardenConfig.lua` | ReplicatedStorage.GardenConfig.lua |
| `RoyalCapitalBuilder.lua` | ServerStorage.ZZTools.RoyalCapitalBuilder.lua |
| `Merging.lua` | ServerScriptService.GardenService.Merging.lua |

### Load order
`GardenService.lua:515-518` becomes: Crown, CrownArena, **CrownArmory** (new), CrownContest, CrownDecree.

### New modules
1. **`ReplicatedStorage.RoyalFX`** (ModuleScript, used by clients). The shared effect helpers the code map asks for ("put shared effect helpers in a module both can use", [[Client World, Royal Client, Music and Ambient]] extension 3). Bats and explosives can reuse `shake`, `spark` and `number`. Functions:
   - `slash(character, step, color)`: emits Slash-01 on an attachment 2.5 studs in front of the root, rolled by the step's arc roll.
   - `spark(pos, kind, color)`: `f` impact and flash; kinds `light`, `heavy`, `blocked`, `parry`.
   - `flash(model)`: one reused Highlight per fighter, fill white 0.5 to 1 over 0.15 s.
   - `number(adornee, text, kind)`: the pooled damage numbers.
   - `shake(degrees, seconds, falloffFrom)`: decaying random rotation, like the live `shake` (`RoyalUI.lua:1126-1137`), with distance falloff.
   - `hitStop(track, seconds)`: `AdjustSpeed(0)`, then back to the saved speed.
   - `telegraph(groundPos, radius, seconds)`, `slamImpact(groundPos, radius)`, `shockwave(pos, radius)`, `speedLines(root, seconds)`.
   - `sound(layers, pos)`: plays up to 3 layered sounds with pitch jitter, positioned like the live `playAt` (`RoyalUI.lua:86-106`).
   - `bladeFX(tool, skinKey, on)`: trail colour, looping skin emitters.
   - Templates come from `ReplicatedStorage.RoyalFXAssets`; effects are parented to RoyalUI's `RoyalFX_Local` folder (`RoyalUI.lua:1057-1059`).
2. **`ServerScriptService.GardenService.CrownArmory`** (ModuleScript, `return function(G)`):
   - `G.royalSkinsOf(player)`: the owned key list from free, stage, `royal.spars`, `RoyalRep`, achievements, `royal.skinsBought` and game pass ownership (cached with `UserOwnsGamePassAsync` in a pcall, the Merge All pattern, `Merging.lua:255-273`).
   - `G.royalPublishSkins(player)`: writes `RoyalSkins` and `WeaponSkin`. Called from `publishRoyal` (`Crown.lua:144-166`), after a purchase, on rebirth (the Rarity Blade colour) and on achievement or rep change.
   - `G.royalSkinFor(f)`: the fighter's skin key (knights always `KnightsLongsword`).
   - `G.royalMakeSword(f)`: builds the skin's Tool named `RoyalSword` with its attachments, tint and emitters.
   - `G.royalOpenPick(bout)`, `G.royalPickDone(bout)`, `G.royalOpenPickFor(player, endsAt)` (contests).
   - Handlers: `skinPick`, `skinReady`, `skinBuy`, `skinRobux`, `prompt:Armory` (validation in the Remotes table).
   - `G.royalOnGamepass(player, id)`: called from the purchase-finished hook.
   - Studio-only `G.royalDev` commands: `skin(userId, key)`, `grant(userId, key)`, `hp(side, n)`, `stage(userId, n)` (sets the `Rebirths` attribute; use only with StudioNoSave).
3. **`StarterPlayer.StarterPlayerScripts.ArmoryMenu`** (LocalScript): builds `ArmoryUI` from the kit images in the UI section; pick and browse modes; listens to RoyalEvent kinds `pickOpen`, `pickClose`, `openArmory` and `prompt` (to close); reads `RoyalSkins`, `WeaponSkin`, `Shards`, `Rebirths`, `RoyalRep` and RoyalState `DuelState`, `DuelPickEnds`, `DuelAReady`, `DuelBReady`; sets the local-only player attribute `ArmoryOpen`.
4. **`ServerStorage.ZZTools.ArmoryBuilder`** (dev tool, run once in Studio): places the stall, rack, display blades, two statues and the `Armory` marker with its prompt. Place-locked to 88602028667228 like `RoyalCapitalBuilder.lua:35`. It only adds a `RoyalArmory` model and one marker, so it does not need a full RoyalCapitalBuilder re-run (which rebuilds the whole capital, [[World Builders and Dev Tools]]).

### Changes to existing scripts

**`CrownConfig.lua`**
- `C.Combat` (`:92-145`): add `MaxHP = 100`; `Combo` (clip speed, chain 0.6, finisher reset 0.6, whiff stun 0.45, hitstun 0.30, contest hitstun 0.15, gap fraction 0.75, and the 4 step rows from Numbers); `damage` on Heavy (16) and Slam (14); Slam `windup = 0.32`, `leapUp = 50`, `leapForward = 10`, `plungeAt = 0.20`, `plungeSpeed = 95`; Block `damageScale = 0.2`, `heavyDamage = 0.5`; `Parry = { window = 0.30, stun = 0.8, stunWalk = 0.5, riposte = 1.5, riposteTime = 1.5, whiffCooldown = 1.2 }`; `HitStop`; `Shake`.
- `C.KingAbilities.Shockwave` (`:149`): `damage = 10`, `parryable = false`.
- `C.Knight` (`:155-164`): `Damage = 6`, `MaxHP = 100`, `BlockChance = 0.25`, `BlockTime = 0.8`, `R15 = false`.
- New `C.Anims = { Idle = 0, Idle2 = 0, Walk = 0, Run = 0, Block = 0, Combo = 0 }`: 0 means "not published yet, use the classic swing", the same rule as `C.Icons` (decision D-001).
- New `C.PickTime = { crown = 10, trial = 10, spar = 10, final = 0 }`.
- `C.Rewards` (`:261-268`) becomes `C.RewardUnits = { Defense = 10, Victory = 25, ContestWin = 40, ContestSurvive = 20, ContestEntry = 4, FirstSpar = 10 }`; new `C.RepeatPairWindow = 1800`. No copy of the D-010 table: rewards call `Config.ShardReward(units, rebirths)` and skin prices `Config.ShardPrice(units, rebirths)` from the [[Rarity Gating and Rebirth]] block (D-017).
- `C.Decrees` (`:209-254`): each `reward` becomes `rewardUnits` (Merge Rush 10, Crown Hunt 20, Empire Build 10) and Royal Race `rewards` becomes `rewardUnits = { 25, 15, 8 }`. `CrownDecree.lua` pays through `G.royalGiveStageShards` per receiving player and sends each viewer their own amount in the `decrees` query ([[King UI Overhaul]] section 8.3).
- New `C.WeaponSkins` list and `C.WeaponSkinByKey`, one row per skin: `key`, `display`, `frame`, `model` (`classic`, `katana`, `longsword`, `statue`), `tint`, `trail`, `sparks`, `fx`, `unlock` (`{ kind = "free" }`, `{ kind = "stage" }`, `{ kind = "spars", need = 3 }`, `{ kind = "rep", need = 100 }`, `{ kind = "achievement", key = "Conqueror" }`, `{ kind = "shards", units = 150, minStage = 3 }`, `{ kind = "robux", gamepass = 0 }`), `grip` (a CFrame per model).
- `C.Sounds` (`:366-383`): add `SwordSlash = rbxasset://sounds/swordslash.wav`, `SwordLunge = rbxasset://sounds/swordlunge.wav`, `Thud = rbxassetid://175024455`.

**`CrownArena.lua`**
- `newFighter` (`:116-126`): add `hp`, `maxHp`, `combo = { step = 0, lastEnd = 0, nextAt = 0, resetUntil = 0 }`, `stunUntil`, `hitstunUntil`, `riposteUntil`, `parriedThisBlock`, `skin`.
- `setStagger` (`:129-136`) becomes `setHP(f, value)`: clamps 0 to maxHp, keeps `f.stagger = 160 x (1 - hp/maxHp)` so the live push formulas still read it, writes `RoyalHP` on the player or knight model, and calls `publishBout` for bout fighters. Stops writing `Stagger`. Exported as `G.royalSetHP`.
- `G.arenaStats` (`:146-154`): multiply walk by 0.5 while stunned or in hitstun. Stun and hitstun start and end call `applyWalkSpeed` so the change lands.
- `makeSword` and `giveSword` (`:172-220`): `giveSword` asks `G.royalMakeSword(f)` first and falls back to today's sword. Add `hum:UnequipTools()` before equipping, so a bat or other tool never stays in hand. New `G.royalSwapSword(f)` destroys and re-gives the sword when a skin is picked during the pick phase.
- `animate` (`:229-243`): for players, return early when `C.Anims.Combo > 0` (their own client animates); keep the `toolanim` fallback otherwise; knights unchanged.
- `enterArena` (`:264-280`): `setHP(f, maxHp)`; set `WeaponSkin` from the saved skin. `leaveArena` (`:283-301`): clear `RoyalHP` and `RoyalStun`.
- `targets` (`:482-505`): new optional `origin` argument for the slam's ground point; the height test uses `origin.Y + 3`.
- `applyHit` (`:539-558`): in this order: shield or guard (0 damage, live); parry (rule 21, sends `hit` with `parried = true`); compute m from HP; live push scaling with block, dash and Underdog; damage with block and riposte factors; `setHP`; hitstun for steps 1 to 3; `hit` event with `dmg`, `hp`, `step`, `seq`, `crit`; if HP is 0, `task.defer(knockdown, v)`. `knockdown` calls `ringOutInBout(v, "ko")` in a bout, `G.royalContestRingOut(v, "ko")` in a contest.
- `strike` (`:570-581`): the combo rules B10 to B17 with the step table; fires `swing { id, step, seq, skin }`; resolves at the step's hit delay; applies the finisher whiff stun.
- `heavy` (`:583-597`): add damage; `swing` gets `step = "heavy"`.
- `setBlock` (`:612-630`): reset `parriedThisBlock` on press; on release inside the window without a parry, use the 1.2 s cooldown.
- `slam` (`:632-646`): ground ray, `slamStart { id, pos, ground, radius }`, resolve at 0.32 s from the new ground point with `targets(..., origin)`, damage 14, `parryable = false`.
- Remote handler (`:650-661`): a per-player limit of 20 messages per second; refuse every action while stunned, block included (rule 21); refuse attack, dash and slam during hitstun (block allowed); type-check every argument (Remotes table).
- `publishBout` (`:678-697`): add `DuelAHP`, `DuelBHP`, `DuelPickEnds`, `DuelAReady`, `DuelBReady`.
- `startBout` (`:743-780`): when `C.PickTime[kind] > 0`, set `state = "pick"`, call `G.royalOpenPick`, wait until every player fighter is ready or the time ends, then run today's intro. Move the `duelIntro` event (`:755`) to after the pick.
- `ringOutInBout` (`:783-818`): replace `setStagger(f, 0)` (`:789`) with full HP for both fighters.
- `startCrownDuel` (`:1063-1096`): rewards through `G.royalGiveStageShards` (`:1077`, `:1088`); the repeat-pair check (rule 49) before rep, shards and `duelWins`.
- Spar (`:1121-1142`): `FirstSpar` through stage shards; the notice (`:1136`) shows the real amount.
- `spawnKnight` (`:422-460`) and `knightThink` (`:1179-1224`): knights get HP and the knight damage spec (`:1209`), carry the Knight's Longsword, and block with `BlockChance` for `BlockTime` when the player is in range and their strike is cooling. With `C.Knight.R15 = true` (phase 4), knights are built with `Players:CreateHumanoidModelFromDescription` as R15, armour is welded on, and the server plays the published clips on them (NPC animations must be played on the server, [[Top-Tier Sword Combat and Animation in Roblox]] §11).
- `onCharacter` (`:1320-1336`): full HP on the new body.
- Studio tools (`:1408-1427`): add `hp(side, n)`.

**`CrownContest.lua`**
- Rewards (`:118`, `:123`, `:129`, `:201`, `:207`, `:214`): through `G.royalGiveStageShards` with `C.RewardUnits`.
- `G.royalContestRingOut` (`:319-346`): treat `why == "ko"` like a fall; replace the stagger reset (`:337-338`) with `G.royalSetHP(f, f.maxHp)`. `G.royalContestRespawn` (`:348-352`): full HP.
- `announce` (`:498-516`): open the Armory for the King. `handlers.joinContest` (`:519-538`): open it for the new entrant until the brawl begins.
- Shockwave (`:380-390`): damage comes from `KA.Shockwave.damage` through `applyHit`. Guard (`:391-400`) already zeroes hits through `guardUntil` (`CrownArena.lua:542`).

**`Crown.lua`**
- `rec` (`:112-120`): defaults `skin = "RoyalSword"`, `skinsBought = {}`.
- New `G.royalGiveStageShards(player, units)` beside `giveShards` (`:243-248`, exported at `:353`): shards = `Config.ShardReward(units, Rebirths)`. Used by duels, contests and decrees. New `G.royalSpendShards(player, amount)` with the same save touch.
- `G.royalCapture` (`:252-269`): add `skin` and a clean copy of `skinsBought`. `G.royalApply` (`:270-285`): accept a known `skin`; accept `skinsBought` keys that exist and whose unlock kind is `shards`.
- `publishRoyal` (`:144-166`): call `G.royalPublishSkins(player)` if it exists.

**`GardenStore.lua`**
- `Store.Default` (`:254`): `royal = { achievements = {}, skinsBought = {} }`.
- Royal sanitise (`:409-436`): validate `skin` against `CrownConfig.WeaponSkinByKey` and `skinsBought` against shard skins; a bad value present counts as one repair, an absent one does not ("Absent is not damaged").
- Self-tests (`:1109-1121`): add a skin round trip and junk cases (unknown skin dropped, non-shard key in `skinsBought` dropped).
- The `ROYAL_NUMBERS` list (`:306-307`) is unchanged: no new number fields.

**`GardenService.lua`**
- Load order (`:515-518`): add `require(script.CrownArmory)(G)` after CrownArena.
- Purchase hook (`:541-545`): also call `G.royalOnGamepass(player, id)` when it exists.

**`RebirthLock.lua`**
- `doRebirth` (`:35-57`): after the busy check (`:36-39`), refuse with "Finish your royal fight first" if `InArena` is set or `G.royalBusy(player)` is `duel`, `contest` or `spar` (looked up at call time, because RebirthLock loads before Crown, `GardenService.lua:504` and `:515`).

**`RoyalCombat.lua`** (client)
- New arena animation loader, run when `InArena` turns true: load the 6 tracks once on the character's Animator, force `Combo` to `Action2` and not looped, `Block` to `Action2`, `Idle2` to `Action`. Swap the default Animate script's `idle`, `walk`, `run` and `toolnone` animation ids for the pack's and restore the originals when `InArena` clears or the character is removed (the documented way to replace default animations, [[Top-Tier Sword Combat and Animation in Roblox]] §12).
- `attackDown` and `attackUp` (`:65-84`): local combo step with the same rules; play the step's segment at once; schedule the local hit prediction (rule 31) and `RoyalFX.hitStop`; send `("strike", step, seq)`. Charging holds the step 4 pose.
- `dash` (`:86-110`): add speed lines.
- `block` (`:112-120`): play the block pose while held; mirror the whiff cooldown locally.
- `ability` (`:122-133`): the slam leap and plunge on your own root (like the live client-first dash), and the slam segment.
- Buttons (`:204-264`, `:267-281`): kit images and FredokaOne labels; no emoji (`:259-264`, `:343`).
- Loop (`:320-368`): hide buttons and the ability bar during the pick phase and while `ArmoryOpen`; draw the PC ability bar; hide and restore the GardenUI clusters (rule 51).
- Hint (`:197-198`, `:335-336`): the new text and opacity rule.

**`RoyalUI.lua`** (client)
- Announcement queue (`:510-523`): wait while `ArmoryOpen` is true. Top-centre stack (`:360-369`): also hidden while `ArmoryOpen`.
- Board (`:554-562`, `:566-592`): health bars, pick foot line; remove the stagger text and the emoji at `:572`, `:582-583`.
- Fighter tags (`:1006-1049`): health bars, knights included, own character skipped, grey outline while stunned.
- `handlers.hit` (`:1405-1415`): `RoyalFX.number`, `flash`, `spark` in the attacker's skin colour, sound layers, role-based shake; skip the spark and impact sound when this client is the attacker and already predicted that `seq`.
- `handlers.swing` (`:1417-1421`): whoosh layers for everyone; for other people's swings, the slash arc at the step's hit delay.
- `handlers.slamStart` (`:1432`, empty today): `RoyalFX.telegraph`. `handlers.slam` (`:1423-1431`): `RoyalFX.slamImpact` with hit stop for the slammer, light flash and shake by distance.
- `handlers.shockwave` (`:1434-1440`): `RoyalFX.shockwave` (dust ring, Wind-02 swirl, a few Shiny-01 gold motes) instead of two gold discs.
- `handlers.dash` (`:1442-1446`): add speed lines. `handlers.crownGuard` (`:1448-1452`): the gold Shield-01 for the guard time.
- `handlers.point` (`:1384-1388`): unchanged; a knockdown point also shows "CROWN POINT".

**`ShopMenu.lua`**
- King page (`:400-594`): one Armory card that opens `ArmoryUI` in browse mode (it fires a local BindableEvent the ArmoryMenu script listens to) and closes the Shop.

### Remotes
No new RemoteEvent objects. New actions on the existing ones.

| Remote | Direction | Payload | Server validation |
|---|---|---|---|
| `RoyalCombat` (live) | client to server | `("strike", step, seq)` | sender is a fighter, not out, not frozen (live checks, `CrownArena.lua:650-652`); not stunned, not in hitstun, not blocking; 20 messages per second per player; `step` a number 1 to 4 or ignored; `seq` a whole number 0 to 2^31 or 0; the server's own combo rules decide the step |
| `RoyalCombat` | client to server | `("charge")`, `("heavy")` | as above; heavy needs a charge of at least 0.30 s (live, `CrownArena.lua:587`) |
| `RoyalCombat` | client to server | `("block", on)` | `on` must be a boolean; refused while stunned |
| `RoyalCombat` | client to server | `("dash", dir)`, `("slam")` | refused while stunned or in hitstun; `dir` must be a Vector3 with finite parts (used for knights only, live) |
| `RoyalRequest` (live) | client to server | `("skinPick", key)` | the live 0.15 s limit and 24-character action check (`Crown.lua:868-871`); `key` a string of at most 24 characters in `C.WeaponSkinByKey` and owned; inside the arena only during a pick phase for that fighter (then the sword swaps on the pad); outside the arena always; saves `royal.skin` |
| `RoyalRequest` | client to server | `("skinReady")` | the sender is a player fighter in a bout whose state is `pick` |
| `RoyalRequest` | client to server | `("skinBuy", key)` | unlock kind `shards`; profile loaded (`Store.IsReady`); not in the arena; stage at least `minStage`; not already owned; shards at least the server-computed price; then spend, add to `skinsBought`, publish, save touch |
| `RoyalRequest` | client to server | `("skinRobux", key)` | unlock kind `robux`; pass id above 0; not in the arena; not owned; then `PromptGamePassPurchase` |
| `RoyalEvent` (live) | server to clients | changed and new kinds below | |
| ProximityPrompt `RoyalPrompt`, Kind `Armory` | world | none | handled as `prompt:Armory`, which cannot be called over the remote (`Crown.lua:874`); sends `openArmory` to that player |

RoyalEvent kinds:

| Kind | To | Payload | New or changed |
|---|---|---|---|
| `swing` | all | `{ id, step, seq, skin }` | changed (adds step, seq, skin) |
| `hit` | all | `{ pos, victim, attacker, seq, step, dmg, hp, blocked, parried, guarded, crit }` | changed |
| `slamStart` | all | `{ id, pos, ground, radius }` | changed (adds ground) |
| `slam` | all | `{ id, pos = ground, radius }` | same shape |
| `point` | all | `why` may now be `"ko"` | changed |
| `pickOpen` | one fighter | `{ mode = "bout" or "contest", kind, endsAt }` | new |
| `pickClose` | one fighter | `{}` | new |
| `openArmory` | one player | `{}` | new |

### Save fields
| Field | Type | Default | How old saves migrate |
|---|---|---|---|
| `royal.skin` | string, a key of `C.WeaponSkinByKey` | `"RoyalSword"` | absent: default, no repair counted; unknown key: default, one repair |
| `royal.skinsBought` | table, `[key] = true`, shard skins only | `{}` | absent: empty, no repair; keys that do not exist or are not shard skins are dropped, one repair each |

Everything else is derived on load (free, stage, spars, reputation, achievements, game pass), so nothing else is saved. Health, stun and combo state live only in server memory. Rebirth does not touch the `royal` table (decision D-009).

### Attributes
| Name | On | Type | Written by | Read by |
|---|---|---|---|---|
| `RoyalHP` | Player (only while `InArena`), knight Model | whole number 0 to 100 | CrownArena `setHP` | RoyalUI tags, RoyalCombat |
| `RoyalStun` | Player | server time the stun ends | CrownArena | RoyalCombat (grey buttons), RoyalUI (grey outline) |
| `WeaponSkin` | Player | skin key | CrownArmory | RoyalCombat (trail), RoyalUI (spark colour), ArmoryMenu |
| `RoyalSkins` | Player | comma-separated owned keys | CrownArmory | ArmoryMenu |
| `ArmoryOpen` | Player, set on the client only | boolean | ArmoryMenu | RoyalUI, RoyalCombat |
| `DuelAHP`, `DuelBHP` | RoyalState | whole number | CrownArena `publishBout` | RoyalUI board |
| `DuelPickEnds` | RoyalState | server time | CrownArena | RoyalUI board, ArmoryMenu |
| `DuelAReady`, `DuelBReady` | RoyalState | boolean | CrownArena | ArmoryMenu |
| `Stagger` | Player, knight Model | no longer written | (was CrownArena `:132-134`) | readers in RoyalUI `:571`, `:1031` are removed |

### Build order
1. **Combat core** (no art or publishing needed): HP, damage, knockdown, combo rules on the classic swing fallback, parry, hitstun, the slam leap and new impact, RoyalFX with existing effects, damage numbers, board bars on the temporary frame, stage-scaled rewards, the repeat-pair rule, the rebirth guard, arena focus.
2. **Animations** (after Samarth publishes): `C.Anims` ids, measured split times, the Animate swap, hit stop on the real clip.
3. **Skins and Armory** (needs the title, icon and bar images): CrownArmory, ArmoryMenu, the 9 skin Tools, the stall, the Shop card, the game pass.
4. **Polish**: R15 knights, knight blocking, optional new sounds.

## Edge cases and abuse

- **Fast or fake strikes.** The server keeps the step and every gap; the 20 per second limit drops floods. A client that claims step 4 gets whatever step the server computes.
- **Parry spam.** A missed parry costs a 1.2 s block cooldown, and the window opens only on a fresh press after the cooldown. Holding block does not extend the window.
- **Fake hits.** The client's predicted hits are cosmetic. HP changes only on the server from server positions.
- **Speed or teleport cheats for reach.** Range and arc use the server's positions; the live out-of-bounds and fall checks stay (`CrownArena.lua:1231-1246`).
- **Unowned or overpriced skins.** `skinPick` checks ownership; `skinBuy` computes the price on the server and ignores anything the client sends.
- **Trading wins with a friend.** Rule 49 pays nothing for a repeat pair inside 30 minutes. The loser's 600 s challenge cooldown (live, `CrownConfig.lua:53`) still applies. Defenses still count for the King's protection (live) but the pair rule stops shards and reputation.
- **Leaving during the pick phase or a fight.** Today's rule applies: the leaver forfeits, the other fighter wins, and a forfeit pays no shards or rep (`CrownArena.lua:1359-1362`, `:1071-1089`). The other fighter's Armory closes on `pickClose`.
- **Rejoin.** Arena state is not restored (live). The equipped skin and bought skins are saved.
- **Reset (death) mid-fight.** The live died path gives up the point and waits for the new body (`CrownArena.lua:803-808`); the new body gets full HP.
- **Rebirth mid-fight.** Refused (rule 54). A player in the challenge queue may still rebirth; rewards use the stage at payout.
- **Stage changes between click and purchase.** The server reads the stage when it handles `skinBuy`.
- **Knockdown and ring-out in the same tick.** The first one processed scores; the second is ignored because the bout is no longer in `fight` (`CrownArena.lua:785`). Two hits landing at once can bring both to 0; the first processed scores.
- **Hits during the point pause.** Frozen fighters' actions are refused (live), and pending swings check `frozen()` before resolving (live).
- **Animations not published or failing to load.** Ids of 0, or a pcall failure on load, fall back to today's swing with all new effects, so nothing breaks before D-005 is done.
- **Game pass check fails.** Treated as not owned, with a notice; checked again on the next join and on purchase finished.
- **Equipped skin no longer owned** (for example a refunded pass). The server falls back to Royal Sword.
- **A bat or other tool in hand.** `giveSword` unequips other tools first; the live re-equip rule keeps the sword in hand (`CrownArena.lua:211-218`).
- **Contest pile-ons.** Contest hitstun is half as long (0.15 s), and every finisher launches, so a crowd cannot lock one player in place.
- **Spectators.** The live sweep teleports anyone else off the ring (`CrownArena.lua:1291-1303`). Other systems' bats, explosives and artillery must skip `InArena` players (see Dependencies).
- **Pets.** Pets never leave their owner's plot ([[Pets]] P35), so no pet is ever in the arena or hit.
- **Mobile.** Every action has a touch button; parry is a tap of BLOCK. On a 0.55 scale the smallest touch target is Guard at 78 x 0.72 = 56 px.
- **High ping.** The parry window is measured on the server from when "block on" arrives, so a player with 150 ms ping must press about 75 ms earlier. No ping compensation in the first build; tune with playtests.
- **Server authority.** Damage, HP, stun, combo steps, rewards and ownership are all server decisions. Knockback keeps the live victim-client push plus a 0.12 s server LinearVelocity backup (`CrownArena.lua:510-536`).

## Performance

Budgets from [[Roblox Platform Constraints for the Expansion]]: under 50 KB/s of download per client, about 500 remote requests per second per client, 400 particles per second per emitter (100 on mobile), up to 255 Highlights rendered at once ([[Top-Tier Sword Combat and Animation in Roblox]] §4).

- **Parts.** Stall about 30, rack 10, display blades about 20, two statues 16: about 76 new anchored parts, with CanCollide, CanQuery and CanTouch off on decor. Each fighter's sword is 1 to 5 parts.
- **Per fighter.** 1 Trail, 2 attachments, at most 1 looping emitter at 2 to 6 particles per second, 1 health bar billboard, up to 2 Highlights (outline and flash, reused).
- **Per hit, on each client.** At most 3 sounds, one burst of at most 20 particles, one pooled damage number (12 at most alive).
- **Per frame, client.** RoyalCombat's existing RenderStepped loop plus up to 6 cooldown sweeps; RoyalUI's existing Heartbeat tag pass, which now resizes bars only when `RoyalHP` changes. Damage numbers and bar chunks use client tweens. No server tweens (server tweens replicate every frame).
- **Server.** The 10 per second arena tick is unchanged (`CrownArena.lua:1306-1314`). Each swing adds one delayed task and one pass over at most a dozen fighters; each slam adds one raycast.
- **Network.** A `swing` event is about 60 bytes and a `hit` event about 120 bytes. A two-fighter duel peaks near 1 KB/s per client; a 12-fighter contest near 6 KB/s. HP travels as small attribute changes.
- **Animations.** Six tracks per fighter, loaded once per arena entry on that player's client; never `LoadAnimation` per swing.
- **Armory.** 10 ViewportFrames that render only while the panel is open (`ShopMenu.lua:292` pattern).
- **Mobile.** Emit counts stay at or under 20 and looping emitters at or under 6 per second.

## Test plan

Setup for every test: in the Studio command bar run `workspace:SetAttribute("StudioNoSave", true)` before pressing Play, so no real save is written (`GardenStore.lua:568-572`). Studio already uses 0 Index fruits and 5 s server time for eligibility (`CrownConfig.lua:38-41`). Practice bouts: `game.ServerStorage.RoyalDev:Invoke("spar")`. Two-player duels: Test tab, Clients and Servers, Local Server with 2 players; crown one with `RoyalDev:Invoke("crown", userId)`. Set `C.Knight.BlockChance = 0` for tests 2 to 7.

| # | Rule | Steps | Pass when |
|---|---|---|---|
| 1 | A1 | start a spar | your `RoyalHP` and the knight model's `RoyalHP` are 100; the board shows 100 |
| 2 | B10, B14 | land steps 1 to 4 on the knight | knight HP 93, 86, 78, 66; numbers 7, 7, 8 white and 12 gold |
| 3 | B11 | tap once every 1.2 s | every `swing` event has step 1 |
| 4 | B12 | tap again within 0.6 s of a finisher | no `swing` event |
| 5 | B16 | whiff a finisher | `RoyalStun` set for 0.45 s; strikes in that time are refused (server log) |
| 6 | A5 | `RoyalDev:Invoke("hp", "B", 50)`, then a finisher; repeat at 100 | the knight's air distance is about 2.5 times larger at 50 HP (log root positions) |
| 7 | A6, A7 | `hp("B", 5)`, then any hit | `point` event with `why = "ko"`; both HP back to 100 after the pause |
| 8 | D20, D21 | press F just before a knight strike | `hit` has `parried = true`; the knight does not strike for 0.8 s; your next hit deals 11 instead of 7 |
| 9 | D22 | tap F and release inside 0.3 s with nothing to parry | block refused for 1.2 s |
| 10 | D23 | hold F past 0.3 s and take a step 1 | 1 damage, about 30% push |
| 11 | C18 | heavy into a held block | 8 damage |
| 12 | E25 to E29 | Royal Slam near the knight | leap seen; dust ring during the windup; impact logged within 0.05 s of 0.32 s; 14 damage within 11 studs; no Neon cylinder in `workspace.RoyalFX_Local` |
| 13 | D20 | try to parry a slam | full blocked slam damage (7), no parry |
| 14 | F30 | log the combo track's `TimePosition` around contact | unchanged for 0.06 s after a predicted hit |
| 15 | F32 | hit 20 times quickly | never more than 12 damage-number billboards exist |
| 16 | F36 | look at the knight and at yourself | the knight has a bar; your own character has none |
| 17 | G41 | 2-player crown duel | both see the Armory with a 10 s timer; READY on both ends it early; `duelIntro` fires after; the picked blade is in hand; `G.royalCapture` shows `skin` |
| 18 | G39 | from the client console fire `skinPick` with a locked key | nothing changes |
| 19 | G41 | run a contest to the final (`RoyalDev:Invoke("contestNow")`) | no pick phase in the final |
| 20 | G42 | join a contest during its announce | the Armory opens until the brawl starts |
| 21 | G45 | `DevSetRebirths` 2 (stage 3), give 2,300 shards, buy Statue Greatsword; then try at stage 2 and with too few shards | first: shards 0 and owned; second and third: refused with a notice |
| 22 | G46 | buy Storm Edge in Studio's test purchase dialog | it unlocks without rejoining |
| 23 | H48, H49 | `stage(userId, 6)`, win a crown duel as challenger; repeat with the same pair | first: +3,875 shards; repeat: +0 shards, no rep, no duel win |
| 24 | I54 | press Rebirth while in the arena | refused with "Finish your royal fight first" |
| 25 | edge | leave during the pick phase | the other fighter wins by forfeit; no shards |
| 26 | I51 | enter and leave the arena | side buttons hide, then all return |
| 27 | UI | screenshot at 1400 x 900, 1920 x 1080 and the 844 x 390 phone emulator during pick, fight and browse | nothing overlaps (UI overlap table) |
| 28 | I52 | start a decree while the Armory is open | its announcement shows only after the panel closes |
| 29 | fallback | all `C.Anims` ids 0 | combos play the classic swing and still deal damage |
| 30 | save | load a profile saved before this change | `royal.skin` is Royal Sword; GardenStore self-tests pass |
| 31 | performance | Developer Console network stats during a duel | under 5 KB/s added per client |
| 32 | A6 | summon the Royal Guard in a contest and knock one down by HP | that knight is removed |
| 33 | D21 | wait 1.6 s after a parry, then hit | normal damage |
| 34 | A4 | hit the King during Crown Guard | 0 damage, no number |

## Dependencies and conflicts

- **[[Rarity Gating and Rebirth]]** (depends on). Provides the stage (`Config.StageFor`), the D-010 frontier shard table (`Config.ShardDropFor`) and the shared `Config.ShardPrice` and `Config.ShardReward` that all rewards and prices use (D-017), and the rarity colour per stage for the Rarity Blade. It also owns `MinDiscovered = 5` (D-026). Its rebirth code must keep the `royal` table (D-009). It also owns eligibility: `C.MinDiscovered = 10` means no 0-rebirth player can ever duel, since only 5 Common fruits exist ([[Royal System - Server]], extension 1). Royal Combat adds the rebirth refusal while fighting.
- **[[Bats]]**. Bats must skip any attacker or victim with `InArena`. Otherwise a spectator could push a fighter off the ring and score a point (live risk, [[Royal System - Server]] extension 2). The bat and the Rarity Blade share one colour table. Bats can reuse `RoyalFX.shake`, `spark` and the victim-client push. A bat in hand is unequipped on arena entry.
- **[[Explosives and Fruit Health]]**. No explosive may affect `InArena` players or knight models (`RoyalKnight`). The health bar frame and fill images are shared, so fruit bars and fighter bars look the same. Slam and TNT can both use Big-Crack-01; scale and colour keep them distinct.
- **[[Weapons and Artillery]]**. Artillery must not reach the Royal Capital arena or hit `InArena` players. Naming: that system's "weapons" are world tools; this system's blades are called "skins" or "blades" in all UI so players do not confuse them.
- **[[Underground and Ores]]**. No direct interaction. Ore entries must not go into the Index folder that eligibility counts (`Crown.lua:289-292`).
- **[[Pets]]**. Pets are never hit by anything here. Pets following an owner into the arena should hide while `InArena` is set (the live ring sweep only moves players). Pets must never take damage (animal-harm rule, [[Roblox Platform Constraints for the Expansion]] §10).
- **[[Boats and Swimming]]**. The live teleport un-seats a player (`Crown.lua:370`), so a boat driver called to a duel leaves the boat behind. Swimming fatigue must not carry into the arena; the arena walk speed already overrides other effects (`GardenService.lua:197-208`).
- **[[Potions]]**. Every potion must keep the `InArena` exemption (live pattern). No potion may change HP, damage or knockback in the arena.
- **[[Plot Overhaul and Environment]]**. The Stud Shop booth and Statue With Sword may also be used as plot or dock decor; each use is its own copy. The Royal Gate model and its name must stay (`CrownArena.lua:708-731`).
- **[[King UI Overhaul]]** (depends on). It owns the royal kit restyle and every screen position: the duel board is its event strip card, the HUD scale is `Kit.scale()`, the layers are its section 4.7 (ArmoryUI at 20), the touch buttons are its F1 to F7, the PC legend is its rule 38, the bars are its A8 and A9, and arena focus is its rule 16. Royal Combat owns what those parts do and the numbers inside them (health, damage numbers, the Armory panel's content, the ability bar, rewards). Resolved in the [[Integration Review]] (C4, C24 to C27).
- **Royal Combat** (this note). The decree rewards (350 to 900 flat, `CrownConfig.lua:209-254`) are not combat and are not changed here; see Open questions.

## Open questions for Samarth

Each has a recommended default so building can start now.

1. **Should health replace stagger?** HP drains with hits, 0 HP is a lost Crown Point, low HP makes you fly further, and the stagger % goes away. Default: **yes**.
2. **Does `swordcombo1` contain 4 swings?** I will measure it in Studio. Default: 4 steps if it has 4 swings; if it has 3, step 3 becomes the finisher and the heavy stays as the big hit.
3. **Which account publishes the 6 animations?** Default: the game's owner account `hikeALT12212`. If you publish from your own account, one Studio playtest grants the game permission.
4. **Pick phase length.** Default: **10 seconds**, ending early when both press READY.
5. **Stage-scaled royal shards** instead of the flat 500 / 1,000 / 3,000 (a proposed new decision, next to D-010 and D-011). Default: **yes**, with the unit table above.
6. **Anti-farm rule:** no shards or rep for the same two players within 30 minutes. Default: **yes**.
7. **Robux skins.** Default: **one game pass, Storm Edge, 149 Robux**, and no paid random boxes ever.
8. **The skin list and unlocks** (9 blades: 2 free, 4 earned by fighting or reign, 2 for shards, 1 for Robux). Default: **as listed**. Add more later through `C.WeaponSkins`.
9. **Where to browse skins outside fights.** Default: **the Royal Armory stall by the arena gate plus one card in the Shop's King tab**, because players without the Aether potion cannot reach the sky.
10. **Hide the garden buttons while you fight.** Default: **yes** (it is also what stops the phone overlap).
11. **R15 knights** so practice bouts use the new animations too. Default: **yes, in the last build step**, after the player combat ships.
12. **Damage number colours** (white normal, gold big hits, blue blocked). Default: **yes**.
13. **Knockdown look.** Default: **no ragdoll**; the fighter freezes in the point pause with a white flash, which keeps the game in the Mild rating.
14. **Decree rewards** (350 to 900 flat) get the same FM treatment. **Decided in the [[Integration Review]]: yes, owned by this note** (Merge Rush 10, Royal Race 25 / 15 / 8, Crown Hunt 20, Empire Build 10 units; Numbers). The King UI shows the number the server sends.
15. **Ping compensation for parry.** Default: **none in the first build**; revisit if players with high ping complain.
