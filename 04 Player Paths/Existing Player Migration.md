---
tags: [player-path]
date: 2026-10-04
---
# Existing Player Migration

This is Samarth's own save on its first join after the expansion ships: 0 rebirths, best fruit Zenflare, cash 6.46e26 ($646Sp), about 34.7K shards. Under the old rules nothing was gated, so he merged up to Zenflare without ever rebirthing. The question is what the one-time migration does to this save, and what the first half hour feels like.

Every rule cited is in the spec notes in [[02 Systems]]. Each friction is ranked in [[00 Paths Summary]].

**Words used here**
- **Migration (Placement)**: the one-time step on first load that raises a save's rebirth count so its stage covers the best fruit it already owns ([[Rarity Gating and Rebirth]] rule 22).
- **`gateVersion`**: a new save field. 0 means "not migrated yet"; the migration sets it to 1 so it never runs twice.
- **Retired-item refund**: the existing load step that pays back items the game no longer sells. Old TNT is added to it (2,500 shards each, [[Explosives and Fruit Health]] rule 5).
- **Notice**: the green toast. It has one label; a second line sent in the same frame replaces the first.

**How the times were made.** Times are minutes and seconds since he joins (m:ss). Facts about his save are the three numbers above; anything else (his Spawn Rarity level, whether he holds old TNT, his Index) is unknown, so each is shown both ways.

**Starting state (old rules)**: rebirths 0, cash x1.00, best Zenflare, cash $646Sp, 34.7K shards, storage 5. Plot and Index as he left them.

---

### 0:00 Load and migration
- **What happens:** the update was published with "Shut down all servers", so no old server can save over him. His profile loads. `sanitise` finds no `gateVersion` and sets it to 0. In `applyProfile`, before anything else, the migration runs: the highest tier among his plot fruit, stored fruit, best fruit and Index entries is 9 (Zenflare), so `rebirths` becomes min(9 - 1, 8) = 8. Stage 9. Then the save is marked `gateVersion = 1`.
- **Spawn Rarity:** the stage 9 ceiling is level 61 (Zenflare). If his level is 61 or lower, nothing changes. If it is 62, he gets $5.14Oc back; at 63, $18.5Oc (5.14Oc + 13.4Oc), about 28 times his cash.
- **Old TNT:** if his bag holds any old `TNT`, each one is refunded at 2,500 shards and removed.
- **Rule:** [[Rarity Gating and Rebirth]] rules 22, 23, worked example "Samarth's save", test T12; [[Explosives and Fruit Health]] rule 5; [[Integration Review]] section 5.
- PAYOFF: nothing he owns is lost. Every fruit, his cash, shards, Index and potions stay.

### 0:02 First look
- **Sees:** cash $646Sp. Boost x3.00 times his Index bonus (it was x1.00): his income triples. If he was away, the "Welcome back" card pays offline earnings at the new x3.00, because the migration runs before offline pay.
- **Rule:** [[Rarity Gating and Rebirth]] rule 16 (`1 + 0.25 x 8`); Persistence order ([[Carrying, Merging, Storage, Rebirth, Tools and Saving]]).
- PAYOFF: the update starts with his income tripling.

### 0:02 to 0:14 The join notices
- **Sees (as the specs were written):** the retired-item refund notice and "Rarities now unlock with rebirths. You are on Rebirth 8: Transcendent fruit unlocked." were both sent at `Config.RefundNoticeDelay`, so one replaced the other on the green label. The bat's intro notice ("Bat unlocked: Transcendent Bat (Power 96, Speed 84)") had no set time. A Spawn Rarity refund, if any, was paid with no message at all.
- FRICTION: **P7-1 (medium)** He could miss the one line that explains the whole update, and a cash jump of up to 28 times would appear with no reason given. Spec errors, fixed: [[Rarity Gating and Rebirth]] rule 22 and U10 now send the migration notice at the refund delay + 4 s and add " Spawn Rarity refund: $18.5Oc." when there was one; [[Bats]] B30 sends the bat line at the refund delay + 8 s.
- **With the fixes:** three separate lines, 4 seconds apart, then the bat slot's "!".

### 0:20 The bat
- **Sees:** the bat slot, white-gold with a "!". He presses 1: Power 96, Speed 84, the strongest bat any player can have.
- **Rule:** [[Bats]] B2, B30, Numbers; [[Integration Review]] section 5 (`batSeen`: old saves get one intro toast).
- **Note:** on day one every placed veteran holds a stage 9 bat in servers full of stage 1 players. See P1-7 in [[New Player - First Session]].

### 0:40 The Rebirth menu
- **Sees:** "Every rarity is unlocked" over the four Transcendent cards. NEEDS: for example "Index 62/64" (Cursed Fruit and Soul Fruit are still "???", since his best is Zenflare). No button.
- **Rule:** [[Rarity Gating and Rebirth]] rule 13, U4 (final stage).
- FRICTION: **P7-2 (low)** The update's main new journey (eight rebirths, one rarity at a time) is already finished on his save before he plays it. That is the spec's chosen default (Placement), and the right one for the game's 119 lifetime visitors, but for Samarth it means his main save cannot show him his own progression. **Fix:** none for the save. Play the new journey in Studio with `workspace.StudioNoSave = true` and the dev door `DevSetRebirths 0`, as [[Rarity Gating and Rebirth]] open question 8 says. (spec: [[Rarity Gating and Rebirth]] Q1, Q8)

### 1:00 The Spawn Rarity card
- **Sees:** "Up to Zenflare", the LOCKED plate. As written, the price row read "Rebirth to raise".
- FRICTION: the stage 9 text errors (P6-3 in [[Endgame - Transcendent]]). Fixed: at stage 9 it reads "Max for every rarity".

### 2:00 Merging pays shards now
- **Sees:** he carries a Zenflare to another Zenflare. A Cursed Fruit appears, his first ever, with the discovery sparkle and a purple "+1,646". Under the old rules merges paid no shards at all.
- **Decides:** merges every Transcendent pair on his plot. If he owns Merge All, one press: "Merged 24 fruit into 12 (+19,752 shards)".
- **Rule:** [[Rarity Gating and Rebirth]] rules 6, 18, U11; D-010.
- PAYOFF: the biggest shard numbers in the game, on his first minute of play after the update.

### 3:30 First Soul Fruit
- **Sees:** two Cursed Fruit merge into a Soul Fruit. With the stage 9 fix, the MAX FRUIT card reads "Soul Fruit is the best fruit in the game" and "You have unlocked every rarity". "Found 64/64".
- **Rule:** [[Rarity Gating and Rebirth]] rule 7, U5 (stage 9 variant).
- PAYOFF: the best fruit in the game, made for the first time, named as such.

### 5:00 The Shop
- **Sees:** the Potions tab: Speed 41,000 shards, Luck 62,000, Aether 82,000, Aqua Affinity 500. Yesterday Speed cost 5,000. The Weapons tab: all 8 cards lit (Cherry Bomb 20,000 for 4, TNT 25,000, Nuke 410,000).
- **Decides:** buys an Aqua Affinity and nothing else.
- **Rule:** D-017 (`ShardPrice` uses his stage); [[Potions]] rules 6 to 8; [[Weapons and Artillery]] rule 3; [[Explosives and Fruit Health]] rule 3.
- FRICTION: **P7-3 (medium)** Overnight, every shard price is 8 to 80 times higher for a placed veteran, and the shards he saved under the old prices buy a tenth of what they did. Nothing explains why. His Transcendent merges refill shards fast (see 2:00), so the shock passes, but the first Shop visit reads like a price hike. **Fix:** a one-time line the first time a migrated player opens the Shop: "Prices now follow your rarity. Transcendent merges pay 1,646 shards each." Optionally a one-time top-up for migrated saves: saved shards times the potion price ratio at their new stage (8.2 at stage 9). Samarth's call. (spec: [[Rarity Gating and Rebirth]] rule 22; [[Potions]] Numbers)
- **Also:** the Luck card at the stage 9 ceiling. As written it sold Luck that can never be drunk (P6-4 in [[Endgame - Transcendent]]). Fixed: "Maxed for good", no Buy.

### 7:00 A boat
- **Sees:** at the main dock, "Buy Boat". The Boat Dock panel: Stud Skiff, "Seats 2: you and a friend", "Kept when you rebirth", Buy 1000. He buys it. The trim, sail stripe and flag are white-gold.
- **Rule:** [[Boats and Swimming]] R20 to R23, 6.3.
- PAYOFF: the fixed 1,000 price is pocket change at stage 9, so the boat is his immediately.

### 10:00 The forest
- **Sees:** six wild animals in the two clearings nearest his bearing, almost all Transcendent and Eternal, with glowing collars. A Transcendent Giraffe wants "Alien Carrot 0/3" (his best fruit is now Soul Fruit, rank 1, so the request is rank 4; requests now follow the best fruit at once, the P9 fix).
- **Decides:** goes back to store three Alien Carrots (storage 13 at 8 rebirths).
- **Rule:** [[Pets]] P7, P9, P19 (6 slots at 8 rebirths); [[Rarity Gating and Rebirth]] rule 17.
- FRICTION: the wasted first trip again (P1-6 in [[New Player - First Session]]).

### 15:00 The mine
- **Sees:** the hut's padlock. "Open Mine": Floor 1, 100K. He has about 55K (34.7K, plus about 20K from his merges, minus the boat).
- **Decides:** merges a few more Transcendent pairs as they appear, then buys Floor 1 at about 25:00.
- **Rule:** [[Underground and Ores]] rules 1, 2, U1 to U3.

### 25:00 to 30:00 The plot freezes
- **Sees:** with every pair merged, his plot fills with Soul Fruit. Nothing spawns, nothing merges, Clear is refused.
- **Rule:** [[Rarity Gating and Rebirth]] rules 6, 13; `Merging.lua:396-399`.
- FRICTION: P6-1 in [[Endgame - Transcendent]], but on day one. Every placed veteran meets the permanent stage 9 freeze within their first session after the update. The endgame loop fix for P6-1 matters most for exactly these players.

---

## What this path found

| Id | Severity | Friction | Spec |
|---|---|---|---|
| P7-1 | medium | Join notices collide; the Spawn Rarity refund is silent (spec errors, fixed) | [[Rarity Gating and Rebirth]], [[Bats]] |
| P7-3 | medium | Shard prices jump 8 to 80 times overnight with no explanation | [[Rarity Gating and Rebirth]], [[Potions]] |
| P7-2 | low | His save has no rebirth journey left; test the progression in Studio | [[Rarity Gating and Rebirth]] |

Also hit on day one: P6-1 (the stage 9 freeze), P6-3 and P6-4 (stage 9 text errors, fixed), P1-6 (wasted forest trip), P1-7 (a stage 9 bat among new players).

Spec errors fixed by this path: [[Rarity Gating and Rebirth]] rule 22, U10 and the code plan (notice timing and refund text); [[Bats]] B30 (join timing).

Payoffs: nothing lost, income tripled, the strongest bat, the first Cursed and Soul Fruit with 1,646-shard merges, an instant boat, and every system unlocked at once.
