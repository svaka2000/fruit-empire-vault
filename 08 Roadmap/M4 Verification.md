---
tags: [roadmap, m4, verification]
date: 2026-10-04
---
# M4 Potions and the Shop tabs: verification

M4 is built and in Studio (not yet published). Spec: [[Potions]]. Every script in Studio matches `~/fruit-empire/src-m2` (83 of 83, by length and hash). Tested in Play with `StudioNoSave` on. Samarth's save was never written: before and after, `GardenProfiles` `p_1662286109` was 1,731 bytes, hash 3001582050, saveCount 609, rebirths 8, shards 58,204. `StudioNoSave` was removed at the end.

## What M4 put in
- **Four Shop tabs:** Potions, Weapons, King, Robux. Potions opens first. TNT moved to Weapons.
- **Six potions** priced by the player's stage: Speed, Jump, Feather Falling, Aether, Luck, Invisibility. The price is worked out on the server the same way the card shows it.
- **Drink from the Inventory.** A potion card's button says Drink, not Equip, and works while you carry a fruit. TNT still says Equip.
- **Potion time is saved.** Time left on your own potions comes back when you rejoin. A Royal Race's free Aether is never saved.
- **Luck can't be wasted.** At the top Spawn Rarity level for your stage, Luck is refused and the card says so. At stage 9 it can't be bought at all.
- **The effect block.** Running potions show as badges in one block: bottom right with a mouse (4 per row), top left on touch (2 per row). It hides in the arena.

## Checked in Play (spec test numbers)
| # | What | Result |
|---|---|---|
| T1 | Open the Shop | Tabs Potions, Weapons, King, Robux in that order. Potions first. Six cards in spec order. TNT on Weapons. No warnings. |
| T2 | Prices at Rebirths 0, 2, 5, 8 | 75 / Luck 110 / Aether 150; 380 / 560 / 750; 3.9K / 5.8K / 7.8K; 41K / 62K / 82K. TNT 2.5K at stage 9. |
| T3 | Buy Speed at Rebirths 5 | With 1,000 shards: "Not enough shards!", still 1,000. With 3,900: bought, 0 left. |
| T4 | Price colour and repaint | White when affordable, red when not, the same moment shards change. "Owned: 1" appeared at once. |
| T6 | Drink while carrying a fruit | Pressed the real Drink button in the Inventory. Fruit still held, Jump 9 to 8, badge shown, "Speed for 5 min!" notice. |
| T8 | Bad DrinkTool calls | TNT, a potion owned 0 of, nil, a number, a table, a made-up name: all ignored, nothing taken. 10 calls in one frame: 1 drink. 10 over 0.5 s: 2 drinks. In the arena: "Potions do not work in the arena!", nothing taken. While dead: nothing taken. |
| T9 | Stacking | Jump drunk 4 times: about 20 min left, the bar refilled. |
| T10 | Arena | Walk speed 19 (the arena's), not Speed's 32. Block hidden. After the bout the block came back, time still counting, walk speed 32 again. |
| T11 | Dying | Every effect and badge gone. |
| T12 | A real rebirth (7 to 8) | Aether still had 1,786 s. The bag was untouched. |
| T16 | Luck | Below the ceiling: 6 of 17 spawns were the next level's fruit, marked Lucky. At the ceiling: "Spawn Rarity is maxed. Save it!", nothing taken, card "Spawn Rarity maxed". At stage 9: "Spawn Rarity is maxed for good.", card "Maxed for good", Buy hidden. |
| T19 | Six potions at once | Two rows from the bottom right, in potion order. UIOverlapCheck silent after the fix below. |

## Found and fixed in the Studio pass
1. **"Feather Falling" was cut to "Feather..."** with Studio's Larger text setting (Larger draws every label about 10 px taller). The card's title box is 177 px; the title drew at 190.
   - Fix (`ShopMenu`): each row of words (titles, effect lines, info lines, prices) now takes the largest size at which every card's words fit. All titles stay the same size, as the spec asks (UI 1).
   - Before: ![[m4-shop-before-title-cut.jpg]] After: ![[m4-shop-titles-fit.jpg]]
2. **The effect line and the info line touched** with Larger text. The spec's small-screen rule (hide the effect line when the two would touch) read the nominal text size, so it missed the extra 10 px.
   - Fix (`ShopMenu`): the rule now reads the size actually drawn. With Larger text on a desktop card, the card shows one centred info line.
   - ![[m4-shop-larger-text.jpg]]
   - **Design call for Samarth:** this means Larger-text players don't see the one-line description ("Run 60% faster"). The other choice is to shrink the potion picture on those cards to make room.
3. **The arena went black.** This is an M3 bug, found while testing T10. A fighter standing on the very edge of a perch had their feet on it but their body's centre past it. PlayerVision's single downward ray found nothing, read "open air at 450 studs with no Aether", and blacked out the screen. The server already skipped fighters for its thin-air check (M3 fix fa3527c); the client never did.
   - Fix (`PlayerVision`): a fighter in the arena always counts as standing on sky ground, which the ring is.
   - Before: ![[m4-arena-black-before.jpg]] After, pushed off the same edge: ![[m4-arena-lit-after.jpg]]
4. **The effect block's frame was one row too tall.** The spec gave it 3 rows (262 x 275), but at most 8 badges at 4 per row only ever fill 2. On Studio's 1529 x 736 window, the empty top row reached the Clear button (UIOverlapCheck).
   - Fix (`GardenConfig`): `EffectRows = 2` (262 x 180). The badges don't move; the frame now ends 74 px below Clear.
   - Before: ![[m4-effects-six.jpg]] After: ![[m4-effects-two-rows.jpg]]

Every fix has a harness check that fails on the old file and passes on the new one.

## Plain-Luau harnesses (after the fixes)
C1 (shop) 103/103, HudScale 63/63, effect grid 41/41, inventory 38/38, M4 integration 262/262 (`Store.SelfTest` 65/65), S1 286/286, S2 60/60, Royal Race Aether 13/13.

## Still to verify (needs something this pass couldn't do)
- **T13, T14, T21 (save round trip).** The spec runs these with Studio API access off, and that is a security setting only Samarth changes. The S2 and integration harnesses cover capture, sanitise and restore.
- **T18 (a second player sees the puff)** and **the Royal Race keeping a bought Aether (review fix R5).** Both need two players: the race skips the King, and a solo player is always the King. Covered by `decree_aether.luau` (13/13).
- **Phones (667 x 375, 812 x 375).** The MCP tools can't switch Studio's device emulator. Phone sizes are covered by the C1 and HudScale harnesses.
- **The inventory card** also shows Drink for a potion. The art is still the code-drawn button (spec: "to convert").
