---
tags: [roadmap, m1, verification]
date: 2026-10-04
---
# M1 Rarity Gating and Rebirth: verification

M1 is built and live in Studio (not yet published). Both halves were tested in Play with `StudioNoSave` on, so his real save was never written. Spec: [[Rarity Gating and Rebirth]]. Numbers: [[Stage Model]]. Exact script lengths and hashes are in `~/fruit-empire/NOTES.md`.

## Server half
- **Migration:** an old save with the full Index becomes Rebirth 8 (×3 cash) and keeps its Spawn Rarity, with no refund. A one-time notice arrives about 9 s after joining. It does not repeat on later logins (fixed after review).
- **Stage 1 cap:**
  - Two Tomatoes make an Apple and pay +3 shards.
  - Two Apples are refused, and the Apple stays in hand.
  - Merge All stops at Apple and says how many pairs wait.
  - Spawn Rarity cannot be bought past Carrot.
- **Rebirth:**
  - It is refused with the exact missing need.
  - When it succeeds: cash goes to 0, the stage's shards are spent, and the Index and Plot Size are kept.
  - A held tool goes back to the bag.
  - From Rebirth 2 the plot starts higher (head start).
- **Guards:** no rebirth mid-merge; no rebirth past 8.
- **Stage 9:** Clear on Soul Fruit keeps the plot growing (D-038).

## Client half (screens)
| Screen | What it shows |
|---|---|
| ![[m1-index-sealed.jpg]] | **Index at Rebirth 0.** "Found 5/5" counts only the unlocked fruit. Uncommon cards are sealed: dimmed, "LOCKED", "Rebirth 1". |
| ![[m1-max-fruit-popup.jpg]] | **Max Fruit popup.** Opens the first time a player tries to merge two of their best fruit. A REBIRTH button leads to the Rebirth menu. |
| ![[m1-upgrade-spawn-rarity-gated.jpg]] | **Spawn Rarity at the ceiling.** "Rebirth to raise" with a greyed plate, not a plain "Maxed". |
| ![[m1-rebirth-menu-ready.jpg]] | **Rebirth menu, every need met.** Shows the next rarity's 8 fruit as "???", the three needs in green, the rewards (+25% cash, +1 storage) and a lit pill. |
| ![[m1-rebirth-confirm.jpg]] | **Confirm.** Names the shard cost and what resets and what stays. |
| ![[m1-after-rebirth.jpg]] | **After the rebirth.** A burst on the player, the toast "Rebirth 1 complete. Uncommon fruit unlocked, cash x1.25", and a plot restarting at Blueberry. |
| ![[m1-rebirth-menu-r1.jpg]] | **Rebirth menu at Rebirth 1.** All 11 Rare fruit fit in 7 + 4 columns. Needs: 13/13 Index, $104M, 250 shards. |

## Bugs found on screen and fixed
1. **Preview cards too big.** They now size to fit the list with no scrolling. The grid was cloned from the Index, which caps a row at 5 cards. So at Rebirth 1 the 11th card was cut off until the row cap followed the chosen column count.
2. **Confirm text said shards are kept.** Rebirth spends the stage's shard need, so the dialog now names it ("Spends 250 shards").

## Still placeholder (waits on art from [[Asset Requests]])
- **R1 titles REBIRTH and MAX FRUIT:** both panels use a plain text title for now (`Config.RebirthTitleImage`, `Config.MaxFruitTitleImage`).
- **R2 padlock:** sealed Index cards show the word "LOCKED" for now (`Config.PadlockIcon`).
- **R5 green tick:** met needs are green text only for now (`Config.TickIcon`).
- **R6 LOCKED plate:** a gated upgrade uses the normal Maxed plate tinted grey for now (`Config.UpgradeLockedButton`).
- Each slot switches to the art as soon as its id is typed into `GardenConfig`.

## What Samarth does before players see it
1. Ctrl+S, then File → Publish to Roblox.
2. Shut down all servers, so no old server ever loads a migrated save.
