---
tags: [research]
topic: What makes Roblox sword combat feel top-tier, plus the animation ownership rules and weapon skin pickers
date: 2026-10-04
---

# Top-Tier Sword Combat and Animation in Roblox

Related: [[Steal a Pet - Bats and Rebirth Knockback]] · [[Sky Rivals - Weapons and Artillery]] · [[Brain Dump 2026-10-04]]

## Summary

- **The animation rules changed on 31 July 2025.** The old rule was "an animation only plays if the game's owner (or Roblox) owns it" [4][5]. Now animations are "Restricted" assets. The animation's owner (a user or a group) can give any experience permission to load it. Once given, that permission cannot be taken back [1]. It still cannot be made "Open Use" [1].
- **Animations from free models still break.** Only the animation's owner can grant permission [1]. A free model whose Animation objects point at someone else's IDs will fail in our game. The fix is to get the KeyframeSequence, open it in the Animation Editor, and use "Publish to Roblox" under our own account or group [6][7].
- **Fruit Empire is owned by a user account, not a group.** The Games API lists the owner as the user `hikeALT12212` [48]. A group-owned experience cannot import a user-owned animation, and Roblox staff say this is "by design" [2]. If the game ever moves to a group, publish the animations under that group or grant the group permission.
- **Fruit Empire's duel already exists and has no health.** It is an R15 ring-out fight with Strike, Heavy, Dash, Block and Royal Slam, plus a "stagger" meter that makes each hit push harder (section 15). Health drain has to be added on top of that.
- **Every top combat game ends a combo on purpose.** Deepwoken does 3 swings and then a kick it calls a "Flourish", which launches the target away [30]. The Strongest Battlegrounds (TSB) does 4 punches, and the 4th always ragdolls the target if it lands [31]. Missing that 4th hit stuns you for a full second [31].
- **Parry windows are short, and failing is punished.** Combat Warriors gives about 0.5 s. A missed parry stuns you for about 1.5 s. A landed parry stuns the enemy for 3 s [29]. Deepwoken's parry has a 1.2 s cooldown, and a landed parry gives 0.3 s of auto-parry [30]. Blade Ball's parry shield lasts 500 ms [32].
- **Hit stop sells weight.** Capcom's own example is 8, 12 and 16 frames of freeze for light, medium and heavy hits [23]. At 60 fps that is about 0.13, 0.20 and 0.27 s. In Roblox the freeze is `AnimationTrack:AdjustSpeed(0)` [11]. It must be called on the client that owns the character [13].
- **Good feel comes in layers, and each layer has a ready-made tool.** Use a swing trail (Trail) [14], a hit spark (ParticleEmitter `:Emit` bursts and flipbooks) [16], a short camera shake (CameraShaker's "Bump" preset) [19], a red flash on the target (Highlight) [18][20], 3-layer sound with ±5% pitch randomness [24][20], and a floating damage number (BillboardGui) [47].
- **Weapon skin pickers split the "look" from the "power".** Arsenal knife skins are cosmetic only [35]. Blade Ball tags each skin card with small icons for VFX, SFX, finisher and accessory [33]. RIVALS lists each skin's extras, like "Custom Sounds" and "Custom Animation" [38][39]. If skins are sold in random boxes for Robux, Roblox requires us to show the odds before purchase [41].

## How it works

### Terms used in this note

- **M1**: the basic attack. It is named after mouse button 1 (left click).
- **Combo / string**: a chain of M1s pressed in a row. Each press plays the next swing animation.
- **Finisher**: the last hit of a string. It usually knocks the target away so the string cannot loop forever.
- **Hit stop (hitstop)**: a very short freeze of the animation at the moment a hit lands. It makes the hit feel heavy.
- **Hitbox**: the invisible shape the game checks to decide whether a swing touched someone.
- **Stun**: a short time when the hit player cannot act.
- **Parry**: a timed block. Press it at the right moment and the attacker gets punished. Press it too early and you get punished.
- **Block**: holding guard. It reduces or stops damage but often has a cost, such as a posture bar filling up.
- **Posture**: Deepwoken's guard meter. When it fills, your guard breaks and you are stunned [30].
- **Knockback**: pushing a character away when hit.
- **Ragdoll**: the character goes limp and falls over for a moment.
- **Animation asset**: an uploaded animation with an asset ID, played through an `Animation` object.
- **KeyframeSequence**: the raw, editable animation data that Studio saves. It cannot play in a live game until it is published [9].
- **AnimationTrack**: what you get after loading an Animation onto a character's Animator. You call `:Play()`, `:Stop()` and `:AdjustSpeed()` on it [11].

### 1. Combo structure in the reference games

| Game | Basic string | How the string ends | Numbers found |
|---|---|---|---|
| Deepwoken | Usually 3 swings, then a "Flourish" kick (4 for rifles) [30] | The Flourish launches the target away, "essentially preventing an infinite combo" [30] | Players start with 220 Health. A guard break stuns for 1.05 s [30] |
| The Strongest Battlegrounds | Hold M1 for up to 4 hits [31] | The 4th hit "guarantees" a ragdoll. Special inputs turn the 4th hit into a downslam or uppercut [31] | 3% + 3% + 4% + 4% = 14% damage. Missing the 4th hit means a 1 s stun. After each M1 you briefly cannot block [31] |
| Combat Warriors | Weapon swings plus a kick [29] | A kick ragdolls the target. They cannot get up until they stop moving for about 2-3 s [29] | See parry numbers below [29] |
| BedWars | Sword swings with knockback, no fixed string [34] | Not confirmed | Swing cooldown 0.3 s. Damage by tier: Wood 20, Stone 25, Iron 30, Diamond 42, Emerald 55. Since 16 Aug 2024 swords drop one tier on death instead of resetting to wood [34] |
| Classic Roblox Linked Sword (2007) | One click slashes, a quick second click lunges [44] | The lunge also pushes the user forward a little [44] | 5 damage on touch, 10 on a slash, 30 on a lunge ("Lunging does 3x damage") [44] |

Extra combo tricks seen:
- **Wall combos.** In TSB, every character has a special combo near a wall. It deals 12% and has a 5 s cooldown [31].
- **Reward for perfect defense.** In TSB, perfectly blocking an M1 makes your next hit a "critical hit" for 3x damage, with a shattering sound. A second perfect block within about 4 s turns it into a "Black Flash" for 6x [31].
- **Directional slashes.** One DevForum solution reads mouse movement through `UserInputService.InputChanged` and compares `input.Delta.X` against `input.Delta.Y`. It counts as vertical when `AbsY * 3 > AbsX`. Then it picks an up, down, left or right slash animation. This works in shift-lock [42].
- **Generated slash animations.** An R6-only tool from November 2025 makes sword swings from numpad-style start and end points (for example 7 to 3 is a diagonal). It outputs data that converts to KeyframeSequences. The author calls it an MVP whose quality varies [43].

### 2. Hit stop (freeze frames)

- Capcom's own design column calls it "hit stop/block stop". It "helps to give the player a sense of the attack hitting as well as the weight of the attack". Its example is 8F, 12F and 16F for light, medium and heavy attacks [23].
- In Roblox: `AnimationTrack:AdjustSpeed(0)` pauses a track, and a positive value plays it again [11]. The docs also say you can use `TimePosition` together with `AdjustSpeed(0)` to "freeze the animation at a desired point" [11].
- Call it on the client that owns the character. Quenty (Jan 2025): "You need to call it on the animation's network owner, in this case, the player that owns the character" [13].
- Gotcha: `Play()` resets speed. If you call `AdjustSpeed` before `Play()`, `Play()` puts the speed back to the default [11].
- No Roblox-specific hit stop durations were found in any official or DevForum source. Not confirmed.

### 3. Camera shake and FOV

- CameraShaker (Sleitnick's port of the Unity "EZ Camera Shake") has presets [19]:
  - **Bump**: `CameraShakeInstance.new(2.5, 4, 0.1, 0.75)`. That is magnitude 2.5, roughness 4, fade-in 0.1 s, fade-out 0.75 s. Position influence 0.15, rotation influence 1 [19].
  - **Explosion**: `CameraShakeInstance.new(5, 10, 0, 1.5)`. Position influence 0.25, rotation influence (4, 1, 1) [19].
- DevForum advice: shake the camera both when you deal damage and when you take it. A quick FOV punch adds power. Raise the FOV during a charge to fake speed [20].

### 4. Slash VFX

- **Trails** draw a ribbon between two Attachments as they move. You need both Attachment0 and Attachment1, or nothing renders [14]. Use Lifetime, Color/Transparency sequences, `FaceCamera` and a slash texture for a sword arc [14].
- **Beams** draw a texture between two Attachments along a cubic Bézier curve. `CurveSize0` and `CurveSize1` bend it, and `Width0` and `Width1` set the width at each end [15]. This suits a fixed crescent slash that appears and fades.
- **Flipbook particles**: one ParticleEmitter texture holds a grid of frames. Layouts are 2x2 (4 frames), 4x4 (16) and 8x8 (64). A 1024x1024 image fits the 8x8 layout. `FlipbookFramerate` maxes out at 30 fps [16]. Modes are Loop, OneShot, PingPong and Random [17]. `LightEmission` 1 means additive glow [16].
- **Mesh slashes / mesh explosions**: a mesh that grows and fades for about 0.25 s, with particles on top for about 1 s. That is the "Hit Effect Handler" tutorial by Loca1t [21].
- **Impact frames**: a 1-2 frame black or white flash. One recipe is "speedline particles, then black color correction, a white highlight on the player, then more particles" [22].
- **Hit flash**: "add a red highlight to the enemy and make it fade out" [20]. Studio renders at most 255 Highlights at once on a client. Extra ones are silently ignored [18].
- **Timing rule from feedback threads**: particles that appear before the animation's contact frame look wrong [20].

### 5. Hit sparks

- Use `ParticleEmitter:Emit(count)` for a one-time burst at the hit point. Code samples use 16 [16].
- Make the particles bigger and push them outward from the contact point [20].
- For a retro, studded look, a OneShot flipbook spark at 30 fps fits the style [16][17]. This is a suggestion, not a rule.

### 6. Sound layering

- A Roblox asset-kit blog (KitsBlox, a third party) suggests three layers per attack: a swing whoosh, an impact on contact (only if it lands), and a victim reaction. It suggests multiplying `PlaybackSpeed` by a random 0.95-1.05, keeping 3-5 variations of each sound, and using SoundGroups for per-category volume [24].
- DevForum: use "very punchy, very impactful sounds" and vary them by "subtly changing the playbackspeed" [20]. Do not make them painfully loud with no volume setting [20].
- TSB plays a distinct "shattering sound" on a critical hit, plus a subtle cue on a perfect block [31]. This is audio used as feedback.
- Retro option: the classic Linked Sword uses built-in client sounds, `rbxasset://sounds/swordslash.wav` and `rbxasset://sounds/swordlunge.wav` [44].

### 7. Damage numbers and health bars

- Damage numbers: put a BillboardGui with a TextLabel on the hit part. Give it a small random offset, tween it up with `StudsOffsetWorldSpace`, fade `TextTransparency` to 1, and set `AlwaysOnTop`. The example used a Circular/Out easing over about 3 s. One poster used a special red color for headshots [47].
- Default Humanoid bars: `HealthDisplayType` can be DisplayWhenDamaged (the default), AlwaysOn or AlwaysOff [45].
- Fighting-game convention: a "little red chunk on your health bar that shows how much damage you've just taken". It is red because our eyes find red first [46]. This is the "health drain" look Samarth asked for.

### 8. Block and parry

| Game | Window | On success | On failure | Cooldown |
|---|---|---|---|---|
| Combat Warriors [29] | About 0.5 s | Enemy stunned 3 s (cannot use inventory or move) | You are stunned about 1.5 s, with WASD only | About 3 s after a miss, about 1 s after a hit |
| Deepwoken [30] | Starts on key press | Enemy staggered and takes posture damage, plus 0.3 s of auto-parry | You can hold F to block instead | 1.2 s, reset by a landed parry |
| Blade Ball [32] | 500 ms shield | Ball bounces to the next player and speeds up slightly | You are eliminated by the ball | Not confirmed |
| RIVALS Katana deflect [39] | 1 s | Bullets reflect back | Not confirmed | 4 s |
| TSB block [31] | Hold F, 180° front cover | Perfect blocks give critical hits | You move much slower while blocking. Some moves break the block | Not confirmed |

- Readable cues matter. Combat Warriors shows a "parry shield" model [29]. Deepwoken's posture bar glows red while the block is not yet full [30].
- Combat Warriors wiki tip: "A parry can only stun one player at a time", so a crowd can still hit you [29].

### 9. Knockback and ragdoll

- The server cannot verify physics on parts a client owns [27]. Characters near a player tend to be owned by that player [27]. The engine docs do allow a server script to call `ApplyImpulse` on a character the client owns automatically [54]. But a standing Humanoid resists pushes in its Running state, so a server impulse often looks like nothing happened [55]. DevForum fixes are: fire a remote so the victim's own client applies the push, change the Humanoid state for the push, or move network ownership to the server first [28][55]. Fruit Empire's arena already does the first, plus a short server `LinearVelocity` as backup (section 15) [53].
- `LinearVelocity` is recommended over raw impulses because the target "doesn't get flung" [28].
- Keep checks such as cooldowns and range on the server, because "exploiters can only manipulate things on their client" [28].
- TSB runs side and back dashes on the client, so they start instantly even with 150 ms ping. Forward dashes turn into attacks, so the server runs them, and they start after the full ping delay [31]. Use this split for anything movement-based.

### 10. Hit detection

- Raycast Hitbox (TeamSwordphin) casts rays each frame between "DmgPoint" attachments on the blade, from their last position to their new one. `HitStart` hits each target only once until `HitStop` resets it [25]. It beats `Touched` for fast swings [25].
- It is no longer supported. Its successor is **ShapecastHitbox**, which is lighter and supports bones and mesh deformation through a "DmgPoint" tag [26].

### 11. Animation playback basics

- Priorities, highest to lowest: Action4, Action3, Action2, Action, Movement, Idle, Core [6]. Swing animations should sit above Movement so walking does not override them.
- `Play(fadeTime = 0.1, weight = 1, speed = 1)` and `Stop(fadeTime = 0.1)` [11].
- **Animation events (markers)**: add named markers in the Animation Editor's event track. Read them with `GetMarkerReachedSignal("Name")`. A marker can carry a parameter string [12]. Put a "Hit" marker on the contact frame and fire the hitbox, sound, spark and hit stop from it.
- **Replication**: if the Animator is inside a player's character, animations started on that player's client replicate to the server and everyone else. NPC or duel-dummy animations must be loaded and played on the server. An Animator created locally will not replicate [10].
- Do not call `LoadAnimation()` every swing, because it creates a new track each time. Load once and reuse, or use `GetTrackByAnimationId()` [10].

### 12. Animation ownership rules

**Before 31 July 2025 (old rule):** an animation "can ONLY be played if you own them". Even a group animation would fail in a user-owned game (a 2023 DevForum answer) [4]. Roblox's own catalog animations, like Ninja Run `656118852`, work in any game. The official docs tell you to paste them straight into the default Animate setup [5].

**Since 31 July 2025 (current rule) [1]:**
- Animations are treated like Audio and Video. They are "Restricted" and cannot be made "Open Use" [1].
- "When an experience gets permission to load an Animation asset, any player will be able to load it when playing the experience and any collaborator will be able to load it during a playtest in Studio" [1].
- The owner can grant permission in five ways: Creator Hub (Configure > Permissions > Assets tab, enter the IDs), inserting from the Toolbox or Asset Manager, playtesting a game that loads it, copy-pasting Animation instances between places, and pasting IDs into properties [1].
- Experience permissions "are not revokable" [1].
- The game's creator and the animation's creator can now be different. Transferring a game no longer means re-uploading every animation, as long as the game already had permission [1].
- Group collaborators no longer need the old "Create and edit community experiences" role for animations to load in Studio [1].
- Edge case: a group-owned game cannot import a user-owned animation ("You do not have ownership of animation with the selected ID"). Staff said this is "by design". A community workaround is to grant the group permission in Creator Hub, or re-upload under the group [2].
- Known bug, now fixed: in November 2025, group animations stopped loading across games in the same group. Staff said it was patched, with no repeat for three months, as of 19 May 2026 [3].
- A "Failed to load animation - sanitized ID" error usually means the ID does not exist, was not published right, or has an ownership/permission problem [49].

**Free-model animations:** if a model ships Animation objects with someone else's IDs, our game will not have permission unless that owner grants it [1]. If the model ships KeyframeSequences instead, we can re-publish them ourselves (see section 13).

### 13. Importing KeyframeSequences and publishing them

1. When you save in the Animation Editor, Studio stores a KeyframeSequence in ServerStorage and adds a reference under the rig's **AnimSaves** folder [6].
2. To import one from a file, right-click the rig's AnimSaves folder, choose **Insert From File**, and pick the `.rbxm`. Then open the Animation Editor, click ⋯ > **Load**, and pick it [7]. If the rig has no AnimSaves folder yet, create and save any animation first [7].
3. For FBX from Blender or Maya, use Animation Editor ⋯ > **Import** > **From FBX Animation** [8].
4. To publish, click ⋯ > **Publish to Roblox**, enter a title, and pick the group in the **Creator** field if the game is group-owned. Then copy the new ID [6].
5. For testing only, `KeyframeSequenceProvider:RegisterKeyframeSequence()` makes a temporary ID that "cannot be used outside of Studio" [9].

### 14. Weapon skin selection UI patterns

| Game | What a skin changes | How you get it | UI detail worth copying |
|---|---|---|---|
| Arsenal [35] | Look only. The knife keeps the same stats | Equip in the Locker. Most knife skins have a golden version (not Fisticuffs or Ban Hammer) | Kill effects are a separate cosmetic |
| Blade Ball [33] | Custom VFX, SFX and slashes, depending on rarity | Sword crate for 80 coins: 89% Common, 10% Rare | Each card shows icons for VFX, SFX, Finisher, Accessory and Rare. Rarities run Common, Rare, Legendary, Limited, Unique, Secret |
| RIVALS [38][39][40] | Skins change shape and sometimes sounds and animations. "Wraps" change only the texture | Cases at 249 Robux, a daily shop (Common 299, Rare 449, Legendary 599 Robux), contracts for playtime, a ranked shop | Each skin lists its extras ("Flashy Effects, Custom Sounds, Custom Animation"). Inspect animations. Finishers are a separate cosmetic |
| Combat Warriors [36] | Weapon color pattern | Cases, battle pass, item shop | A rarity color box on each skin, from green (most common) to yellow (rarest) |
| Type Soul [37] | Weapon appearance | "Weapon Appearance Rerolls" from PvP and codes | The reroll is not used up if the weapon has no other skins |
| BedWars [34] | Tiers are real stat upgrades, not skins | Bought in order: Wood > Stone > Iron > Diamond > Emerald | Tier color shows strength |

Rule for paid random skins: odds must show for every possible item before purchase. Luck boosts must show their numbers. Free random rewards, like a key found in the world, do not need odds [41].

### 15. What Fruit Empire's duel already does (local code, 2026-10-04 export)

The proposals below change an existing fighting system, so here is what it is today [52][53].
- **It is a ring-out game with no health.** You win points by knocking the other player out of the arena. Nothing bought (Robux, potions, upgrades, shards) changes anything inside the arena.
- **Moves:**

| Move | Input | Reach | Push (sideways / up, studs/s) | Cooldown |
|---|---|---|---|---|
| Strike | tap Attack | 7.5 studs, 70° each side | 40 / 16 | 0.42 s |
| Heavy | hold 0.45 s, release | 8.5 studs, 60° each side | 74 / 26 | 1.25 s |
| Dash | Dash key | 70 studs/s for 0.18 s | takes 60% knockback mid-dash | 2.2 s |
| Block | hold, up to 1.6 s | — | takes 30% knockback, walks at 55% | 0.9 s after release |
| Royal Slam | Ability | 11-stud circle around you | 92 / 32 | 10 s |

- **Stagger is the comeback rule.** Each hit adds stagger (Strike 9, Heavy 18, Slam 22). Knockback is multiplied by (1 + stagger/100), up to 2.6x. A ring-out resets it. This works like the damage percent in Super Smash Bros. (comparison is my inference).
- **Underdog Momentum:** a fighter 2 points behind deals 15% more knockback, takes 10% less, walks 6% faster and gets 20% shorter dash and slam cooldowns.
- **Knockback is already built the robust way:** the victim's client applies the velocity, and the server adds a 0.12 s `LinearVelocity` backup [53]. See section 9.
- **What the brief adds:** health drain, better animations and effects, a new Royal Slam, and weapon skins picked before the fight [[Brain Dump 2026-10-04]]. The open design choice is how health fits with ring-outs and stagger. One option: health is a second way to win, and low health raises knockback the way stagger does (proposal).
- **Avatar type:** the game forces R15 (`MorphToR15`, Roblox API, read 2026-10-04) [48]. Swing animations must be authored on R15 rigs.

## What Fruit Empire should take

These are proposals for our game. Any number marked "start at" is a value to tune in playtests, not a researched fact.

**Royal Sword duel (King's Throne arena)**
1. **3 hits plus a finisher.** M1, M1, M1 slashes (left-to-right, right-to-left, overhead), then a 4th "Royal Slam" that knocks the opponent back. This copies the Deepwoken Flourish and TSB 4th-hit pattern [30][31]. Missing the finisher stuns the attacker. Start at 0.6 s (TSB uses 1 s [31]).
2. **Drive all feedback from markers.** Put one "Hit" marker on each swing's contact frame [12]. On that marker, the attacker's client:
   - freezes the swing with `AdjustSpeed(0)`. Start at 0.06 s for hits 1-3 and 0.12 s for the slam [11][13]. These are deliberately shorter than Capcom's 8-16F [23], because duel hits also cause network lag. That reasoning is my inference.
   - plays the Bump shake on a hit and a scaled-down Explosion shake on the slam [19].
   - bursts a OneShot flipbook spark [16][17].
   - flashes a red Highlight on the target and fades it out [18][20].
   - plays the whoosh, then the impact, then the hurt sound, with 0.95-1.05 pitch randomness [24].
   - pops a floating damage number [47].
3. **Use a Trail on the Royal Sword** between two blade Attachments. Fruit-tier skins can recolor it [14]. Add a Beam crescent for the overhead slash [15].
4. **Show health drain.** Use a custom duel health bar for both players at the top of the screen. It drops at once, with a red "recent damage" chunk that drains a moment later [46]. Turn off the default overhead bar during duels (`HealthDisplayType = AlwaysOff`) [45].
5. **Give parry a clear cue.** Use a short window (start at 0.4 s, close to Combat Warriors' 0.5 s [29]), show a visible shield or glint, stun the attacker on success, and give a short self-stun on a miss. Optional depth: a perfect parry makes your next hit a crit, like TSB [31].
6. **Run hit detection with ShapecastHitbox** using DmgPoint tags on the blade [26]. The attacker's client reports hits. The server checks range and cooldown before applying damage [27][28].
7. **Send knockback through the victim's client** using LinearVelocity [28]. Do not rely on a server impulse on a client-owned character [27].
8. **Pick a skin right before the duel.** Show a short carousel on the arena entry screen, with a rotating preview and the Blade Ball-style badges (VFX / SFX / Custom Animation) [33][38]. Skins should be **cosmetic only** in duels, like Arsenal [35]. Duels are about skill and the crown.
9. **Unlock skins through progress, not paid crates.** Give rarity colors that match our fruit tiers (Common > ... > Transcendent), unlocked by rebirth, Index completion and throne wins. This ties skins to the player's stage. If we ever sell random skin boxes, add the odds pop-up first [41].

**Bats (rebirth-tier knockback)**
- Copy BedWars' clear tier ladder (fixed steps, one color per tier) [34], but scale **knockback**, not fruit damage, because stealing is removed. The bat color should match the rarity tier the player has unlocked through rebirths, as Samarth asked in the brain dump.
- Reuse the duel feel stack, with a lighter shake and no hit stop on whiffs. Apply knockback on the victim's client [28].

**Animation pipeline (do this before building anything)**
- Author swings in the Animation Editor on the same rig type the game uses. Set priority to Action or higher [6]. Publish from the account that owns the experience (`hikeALT12212`) [48], or from a collaborator account followed by a Studio playtest, which grants permission automatically [1].
- Load each track once per character on the client. That client plays it, and it replicates [10]. Duel NPCs or a practice dummy must play on the server [10].
- Never ship Animation IDs from free models we do not own. Re-publish the KeyframeSequences instead [1][6][7].
- The "Sword Animations" asset Samarth linked (77935648543779, by the group Fancy Cat Games) is listed by the Toolbox API as a **Model with 90 MeshParts, 0 Animation instances and 0 scripts**, in the "3d weapons" category [50]. Re-checked 2026-10-04: same counts, and about 56,600 triangles in total. So it is probably a pack of sword meshes, not animations. Open it in Studio to check for KeyframeSequences before planning around it. If the swords become skins, keep only the few you use, since the whole pack is heavy.
- The retro option fits the 2006-2008 style: the classic Linked Sword's built-in slash and lunge sounds, and its "lunge does 3x" idea, could become the slam [44].

## What to avoid

- **Endless stun-locks.** The DevForum feedback on a stun-heavy 5-hit combo was that it did not make a fun challenge on its own [51]. Deepwoken and TSB both end strings with a launch [30][31].
- **`Touched` hit detection on fast swings.** It misses between physics frames [25]. Also avoid Raycast Hitbox for new work, because it is unsupported [26].
- **Server-only feedback.** Hit stop and animation speed changes must happen on the owning client [13]. Server-applied impulses on players often fail [27][28].
- **VFX that lead the animation.** Particles before the contact frame look off [20].
- **Painfully loud impacts, or flashing with no warning** on big hits or kills [20].
- **Paid random skin boxes without odds** break Roblox policy [41].
- **Pay-to-win duel skins.** RIVALS has some skins that affect gameplay [39], but for a crown duel that would feel unfair. Keep skin stats equal. This is a design opinion.
- **Knockback near the sea edge or another player's plot.** It turns bats into a griefing tool. This is an inference with no source. Consider safe zones or reduced knockback on your own plot.
- **Group or user permission mix-ups.** If the game moves to a group later, user-owned animations will not import [2].

## Open questions

- ~~Is Fruit Empire on **R6 or R15**?~~ Answered: R15. The Roblox API lists `universeAvatarType: MorphToR15` [48]. So the R6-only slash generator [43] cannot be used.
- How should health combine with ring-outs and stagger in the existing duel (section 15)?
- What is really inside asset 77935648543779? The Toolbox API shows no Animation instances [50]. KeyframeSequences are not counted in that API, so this is not confirmed.
- Which account will publish the animations: the owner `hikeALT12212` or Samarth's account? [48]
- Should duel hits be confirmed by the attacker's client with server checks (snappy), or decided only by the server (fair but laggy)? This needs a ping test.
- What exact hit stop and shake values feel right in Roblox? No Roblox source gives numbers, so we need to playtest.
- Type Soul's combat timings (M1 count, parry window) were not found on its wiki. Not confirmed.
- The public game title is still "[STEAL!] Fruit Empire" [48], even though stealing was removed. Should it be renamed?

## Sources

1. https://devforum.roblox.com/t/improving-animation-asset-permissions/3852101 (Roblox announcement, 31 Jul 2025)
2. https://devforum.roblox.com/t/cant-import-user-owned-animations-in-a-group-owned-experience/3989573
3. https://devforum.roblox.com/t/group-animations-no-longer-automatically-accessible-across-group-experiences-causing-critical-production-failures/4079089
4. https://devforum.roblox.com/t/cant-run-animations-on-someone-elses-game/2576003
5. https://create.roblox.com/docs/animation/using
6. https://create.roblox.com/docs/animation/editor
7. https://create.roblox.com/docs/education/build-it-play-it-island-of-move/sharing-animations
8. https://create.roblox.com/docs/art/characters/export-avatar-animations-from-maya
9. https://create.roblox.com/docs/reference/engine/classes/KeyframeSequenceProvider
10. https://create.roblox.com/docs/reference/engine/classes/Animator
11. https://create.roblox.com/docs/reference/engine/classes/AnimationTrack
12. https://create.roblox.com/docs/animation/events
13. https://devforum.roblox.com/t/animationtrackadjustspeed0-not-working/3390106
14. https://create.roblox.com/docs/effects/trails
15. https://create.roblox.com/docs/effects/beams
16. https://create.roblox.com/docs/effects/particle-emitters
17. https://create.roblox.com/docs/reference/engine/enums/ParticleFlipbookMode
18. https://create.roblox.com/docs/effects/highlighting
19. https://github.com/Sleitnick/RbxCameraShaker/blob/master/src/CameraShaker/CameraShakePresets.lua
20. https://devforum.roblox.com/t/how-would-i-go-about-making-my-combat-feel-more-impactful/2042214
21. https://devforum.roblox.com/t/hit-effect-handler-how-to-make-a-good-hit-effect-in-5-minutes-mesh-particles/1511337
22. https://devforum.roblox.com/t/advanced-impact-frames/2468478
23. https://game.capcom.com/cfn/sfv/column/131545?lang=en (Capcom SF Seminar, hit stop)
24. https://kitsblox.com/blog/how-to-add-sound-effects-roblox (third-party blog)
25. https://devforum.roblox.com/t/raycast-hitbox-401-for-all-your-melee-needs/374482
26. https://github.com/TeamSwordphin/ShapecastHitbox and https://devforum.roblox.com/t/shapecasthitbox-for-all-your-melee-needs-v025/3624241
27. https://create.roblox.com/docs/physics/network-ownership
28. https://devforum.roblox.com/t/should-i-handle-knockback-on-the-server-or-client/3738501
29. https://combat-warriors.fandom.com/wiki/Parry
30. https://deepwoken.fandom.com/wiki/Combat_Mechanics
31. https://the-strongest-battlegrounds-rblx.fandom.com/wiki/Basic_Combat
32. https://bladeball.fandom.com/wiki/Gameplay
33. https://bladeball.fandom.com/wiki/Sword_Skins
34. https://roblox-bedwars.fandom.com/wiki/Sword
35. https://arsenal.miraheze.org/wiki/Knife
36. https://combat-warriors.fandom.com/wiki/Skins
37. https://type-soul.fandom.com/wiki/Weapon_Skins
38. https://robloxrivals.fandom.com/wiki/Skins
39. https://robloxrivals.miraheze.org/wiki/Katana and https://robloxrivals.miraheze.org/wiki/Wraps (Miraheze copies that say they are copied from the Fandom wiki)
40. https://robloxrivals.fandom.com/wiki/Finishers
41. https://create.roblox.com/docs/production/monetization/paid-random-items
42. https://devforum.roblox.com/t/directional-slashes/2848168
43. https://devforum.roblox.com/t/sword-swing-animation-generator/4080523
44. https://roblox.fandom.com/wiki/Sword (classic Linked Sword)
45. https://create.roblox.com/docs/reference/engine/classes/Humanoid
46. https://queerqueenlywriting.wordpress.com/2019/03/09/the-art-of-fighting-ui/
47. https://devforum.roblox.com/t/whats-the-method-of-accomplishing-these-cool-damage-numbers/2984815
48. https://games.roblox.com/v1/games?universeIds=10764283455 (universe ID from https://apis.roblox.com/universes/v1/places/88602028667228/universe)
49. https://devforum.roblox.com/t/failed-to-load-animation-sanitized-id/2652038
50. https://apis.roblox.com/toolbox-service/v1/items/details?assetIds=77935648543779 and https://create.roblox.com/store/asset/77935648543779/Sword-Animations
51. https://devforum.roblox.com/t/what-makes-good-combat/1999076
52. Local code: `~/fruit-empire/src-export-2026-10-04/ReplicatedStorage.CrownConfig.lua` (`C.Combat`, `C.Rewards`, about lines 84–145 and 256–266)
53. Local code: `~/fruit-empire/src-export-2026-10-04/ServerScriptService.GardenService.CrownArena.lua` (the `push` function, about lines 507–535)
54. https://create.roblox.com/docs/reference/engine/classes/BasePart#ApplyImpulse
55. https://devforum.roblox.com/t/knockback-pretty-uneffective-on-humanoids/3324305 (28 Dec 2024)

## Review (critic pass, 2026-10-04)

- **Answered:** R6 or R15. The game is R15 [48].
- **Added:** section 15, the existing duel's moves, stagger and knockback code [52][53]. The proposals had been written as if no fighting system existed.
- **Corrected:** section 9 said a server `ApplyImpulse` on a player "often does nothing" as if it were an engine rule. The docs allow it [54]. The real cause is that standing Humanoids resist pushes [55].
- **Unsourced but labelled:** the hit stop, parry window and combo numbers in the proposals are "start at" values, not research.
- **Still not confirmed:** Roblox-specific hit stop durations, Type Soul combat timings, and what is inside asset 77935648543779 besides meshes.
