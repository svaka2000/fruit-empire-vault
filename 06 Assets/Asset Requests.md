---
tags: [assets, requests]
date: 2026-10-04
---
# Asset requests: what Samarth makes or does

This note lists everything only Samarth can make or do for the expansion: images, animations, one optional game pass, Studio saves and a few decisions. It merges the asset sections of the 11 specs in `02 Systems` and the merged list in [[Integration Review]] section 7. Items are grouped by the first milestone in [[Roadmap]] that shows them.

**Nothing here blocks building.** Every image and animation slot has a fallback, so Claude builds each milestone first and types the id in later. The one exception is M16 (the real sword animations), which cannot start without the six animation ids.

**Words used here**
- **Kit**: your image-based UI set, listed in [[UI Kit Inventory]]. Every new image must look like it.
- **Id**: the number Roblox gives an uploaded image or animation (an rbxassetid).
- **Fallback**: what the game shows until the id exists, for example Fredoka text on the blue title bar.
- **Figma batch**: one Figma session that makes every title from the frame you used for the INDEX title.
- **M1, M2, ...**: the milestones in [[Roadmap]].
- **PENDING**: asked for before today and not made yet. **NEW**: first asked for in the specs. **OPTIONAL**: the game is fine without it. **CHECK FIRST**: Claude checks in Studio whether it is needed before you make it.

## How to make and send any image

1. One image per message. No sheets to cut.
2. Attach the kit pieces each prompt names. Use your original PNG if you have it. If not, crop it from the file in the table below.
3. ChatGPT: the prompts already ask for a transparent PNG. Gemini: replace "transparent background" with `flat pure green #00FF00 background, and a 3 px white outline around the object`. Claude keys the green out.
4. Save the result with the exact file name and folder given. Every folder is under `~/fruit-empire/art/`.
5. Claude trims it to the canvas, cleans the edges (removes halos, 4 px alpha bleed), then uploads it through the Studio bridge or asks you to import it (Asset Manager, Import, signed in as the account that owns the game). After moderation passes, Claude types the id into the config key.
6. If an image comes back with text, a background or a wrong word, reply: `Redo only this image. Same style, same canvas, transparent background, no text.` If a baked word is misspelled twice, ask for the image with that area left empty and add the word in Figma (the word step in R12).

### Where to get each reference image

| Kit piece (name in [[UI Kit Inventory]]) | Id | If you do not have the PNG, crop it from |
|---|---|---|
| IndexMenu.TitleText (the word INDEX), IndexMenu.TitleBlue (blue title bar) | 103221050515090, 95532041481396 | vault `Images/ui-reference-index-menu.jpg` |
| RewardsBox chips (bonus, money, shards), RewardsBox.ClaimButton, Templates.Cell.Selection (white card outline) | 89282702522748, 82056408182031, 78947574768646, 132804991293392, 113285622864672 | `Images/ui-reference-index-menu.jpg` |
| HUD buttons Inventory, Index, Shop | 108477171933167, 118941334639047, 71485100495903 | `Images/ui-reference-hud-left.png` |
| HUD buttons Rebirth, Merge All, Clear (with the three above) | 115846161107162, 99980479639840, 88888506311530 | `~/fruit-empire/art/royal-icons/style-reference.png` (the same picture as `Images/ui-reference-hud-icons.png`) |
| TopButtons.LockButton, TopButtons.UpgradeButton | 118392145629817, 116083144480280 | `Images/throne-kingdom-menu.jpg` |
| CornerHud.CashBar, ShardsBar, BoostBar (Friend Boost) | 100015543922655, 140489943167779, 85577747844362 | `Images/throne-kingdom-menu.jpg` |
| CarryInfo.RateBar, CarryInfo.NameBar | 120297790302429, 123470450245311 | a screenshot in game while you carry a fruit |
| ShopMenu.ToolsTab (vertical tab), the shop card's Buy button, ShopMenu.Panel.CloseButton (red X) | 110917091182913, 97067256017679, 107111219044453 | a screenshot of the open Shop |
| UpgradeMenu Maxed button | 76326716689640 | a screenshot of the Upgrade panel with a maxed card |
| Potion of Aether bottle, Aether badge, Speed tile, Jump tile (your potion art, not in the inventory table yet) | 123282319597102, 91730920675016, 85746922401157, 103629048118784 | `~/fruit-empire/art/potion-icons/style-reference-potions.png` |

## Already requested and still pending (checked 4 Oct)

| Request | Asked in | Status | Where it is in this note |
|---|---|---|---|
| Royal icons: HUD icons, decree icons, medals, crown styles | `~/fruit-empire/art/royal-icons/PROMPTS.md` (3 Oct, four sheets) | **PENDING.** The folder holds only `PROMPTS.md` and `style-reference.png`. | R8 to R11. The one-image-per-message prompts there replace the sheet prompts ([[King UI Overhaul]] 7.11). If you already made the sheets, send them: Claude cuts them, and only B11 and B12 are still needed (B11 is the padlock, R2). |
| Potion art: Feather Falling and Luck, shop art and badge each | `~/fruit-empire/art/potion-icons/PROMPTS.md` (3 Oct) | **PENDING.** The folder holds only Claude's placeholders and the style reference. The game shows the placeholders. | R13 |
| Six sword animations | [[Decisions Log]] D-005 ("Needs Samarth") | **PENDING** | R34. Start now: it is the only request that blocks a milestone (M16). |

## Merged and withdrawn

- **One padlock for two jobs.** [[Rarity Gating and Rebirth]] A3 (Sealed cards) and [[King UI Overhaul]] B11 (royal Lock icon) are the same gold padlock at 256 x 256. Make it once (R2). Claude puts the same id in `Config.PadlockIcon` and `C.Icons.Lock`.
- **One Figma batch for every INDEX-style title** (R1): REBIRTH, MAX FRUIT, KINGDOM, CROWN CHALLENGE, 7 stingers, 14 castle signs, ARMORY, MINE, PETS, BOAT DOCK. The ChatGPT ARMORY title from [[Royal Combat]] is withdrawn (conflict C4).
- **One bar pair for every bar** (R7, A8 and A9, decision D-027): fruit health, ore nodes, boat HP and stamina, fighter HP, royal timers. The four other bar requests are withdrawn (conflict C2).
- **One combat button set** (R12, F1 to F7). The [[Royal Combat]] 3 x 3 sheet is withdrawn.
- **Aqua Affinity art** is owned by [[Potions]] (R23). The [[Boats and Swimming]] versions are withdrawn (D-029).
- **The Pets tab icon** is one of the three Index tab icons (R25), made together with Fruits and Ores.
- **Words go through Figma where possible**, because AI tools misspell words ([[King UI Overhaul]] 7.1). The "Bat" and "Swing" labels (R19, R20) can be made in the same Figma step as the combat-button words.
- **The royal icon sheets** from 3 Oct are replaced by one image per message (R8 to R11).

---

## Needed for M1: Rarity Gating and Rebirth

### R1. Figma title batch: 29 title images in one sitting (NEW)
- **What:** words in the exact lettering of your INDEX title. M1 needs only REBIRTH and MAX FRUIT. The rest serve later milestones, but one sitting is faster than six.
- **Fallback until then:** the blue title bar with FredokaOne text.
- **Folder:** `~/fruit-empire/art/titles/`
- **Figma steps** ([[King UI Overhaul]] 7.3):
```
In Figma, duplicate the frame you used to make the INDEX title (IndexMenu.TitleText, asset 103221050515090). Keep the same font, italic, white fill, dark outline and drop shadow. Change only the words. Export each as PNG at 2x with a transparent background, 148 px tall, width = the text plus 24 px of padding. Name each file exactly as in the table.
```
- **No Figma frame?** Use ChatGPT, one title per message. Attach IndexMenu.TitleText. Replace WORDS with the words from the table. Check the spelling before you save.
```
Image 1 is the exact style to copy: the word INDEX from my Roblox game's panel title (IndexMenu.TitleText, rbxassetid 103221050515090). Make the words WORDS in exactly that lettering: the same heavy italic slant, the same white-to-light-grey vertical gradient, the same thick dark outline relative to letter height, the same small bevel, the same letter spacing. Transparent background. The words centred on one line with 6% padding, 148 px tall. No drop shadow outside the outline, no other shapes, no other words.
```

| Words | File | Used on (spec, section) | First needed |
|---|---|---|---|
| REBIRTH | title_rebirth.png | Rebirth menu ([[Rarity Gating and Rebirth]] U4, A1; `Config.RebirthTitleImage`) | M1 |
| MAX FRUIT | title_max_fruit.png | Max Fruit popup ([[Rarity Gating and Rebirth]] U5, A2; `Config.MaxFruitTitleImage`) | M1 |
| KINGDOM | title_kingdom.png | Kingdom panel ([[King UI Overhaul]] 5.6, T1) | M2 |
| CROWN CHALLENGE | title_crown_challenge.png | challenge dialog ([[King UI Overhaul]] 5.7, T2) | M2 |
| ROYAL GATE | sign_royal_gate.png | 2 gate signs (King UI 5.11, W1) | M2 |
| ROYAL ARENA | sign_royal_arena.png | arena arch, signpost, 2 scoreboards (W2) | M2 |
| THRONE ROOM | sign_throne_room.png | signpost (W3) | M2 |
| RETURN HOME | sign_return_home.png | Return Gate (W4) | M2 |
| CHALLENGE THE KING | sign_challenge_the_king.png | Altar board (W5) | M2 |
| KING'S CASTLE | sign_kings_castle.png | castle gate plaque (W6) | M2 |
| CURRENT RULER | sign_current_ruler.png | hall tablet (W7) | M2 |
| ROYAL RECORDS | sign_royal_records.png | hall tablet (W8) | M2 |
| HISTORY | sign_history.png | hall wall board (W9) | M2 |
| EMPIRE POWER | sign_empire_power.png | hall wall board (W10) | M2 |
| YOUR REIGN | sign_your_reign.png | chamber board (W11) | M2 |
| PRACTICE BOUT | sign_practice_bout.png | Sparring Post (W12) | M2 |
| THE KING | sign_the_king.png | portrait plate (W13) | M2 |
| ETERNAL KING | sign_eternal_king.png | portrait plate (W14) | M2 |
| FIGHT! | title_fight.png | stinger (King UI 5.5, T3) | M3 |
| CROWN POINT | title_crown_point.png | stinger (T4) | M3 |
| VICTORY | title_victory.png | stinger (T5) | M3 |
| DEFENDED | title_defended.png | stinger (T6) | M3 |
| DEFEAT | title_defeat.png | stinger (T7) | M3 |
| NEW KING | title_new_king.png | stinger (T8) | M3 |
| SUDDEN DEATH | title_sudden_death.png | stinger (T9) | M3 |
| BOAT DOCK | title_boat_dock.png | Boat Dock panel ([[Boats and Swimming]] 6.3, request 3) | M10 |
| PETS | title_pets.png | Pets panel ([[Pets]] U2, A1; `PetConfig.Images.Title`) | M11 |
| MINE | title_mine.png | Mine panel and lift sign ([[Underground and Ores]] U1, U11, A1; `Config.MineTitleImage`) | M13 |
| ARMORY | title_armory.png | Armory panel ([[Royal Combat]] request 2; King UI T10) | M17 |

### R2. Padlock icon (NEW, merged)
- **What:** one closed gold padlock. **Used:** Sealed Index cards and the Sealed info box ([[Rarity Gating and Rebirth]] U1, U2, A3; `Config.PadlockIcon`), the LOCKED plate (R6), the royal Lock icon ([[King UI Overhaul]] 7.6 B11; `C.Icons.Lock`).
- **File:** `~/fruit-empire/art/rebirth/padlock.png` (256 x 256)
- **Attach:** TopButtons.LockButton, HUD button Index.
```
Image 1 is my Roblox game's Lock button (TopButtons.LockButton, rbxassetid 118392145629817): copy its golden padlock's shape and colours. Image 2 is my Index button (rbxassetid 118941334639047): copy its thick dark outline, glossy highlight at the upper left and saturated cartoon shading. Draw ONE closed padlock icon only, front view, centred: gold body with a darker keyhole, steel-grey shackle, thick dark outline, glossy highlight top left. No text, no background, no shadow. Canvas 256 x 256 with 12% padding.
```

### R3. Index chip (NEW)
- **Used:** the NEEDS box of the Rebirth menu ([[Rarity Gating and Rebirth]] U4, A4; `Config.IndexChip`).
- **File:** `~/fruit-empire/art/rebirth/index-chip.png` (188 x 200)
- **Attach, in this order:** the three RewardsBox chips (bonus, money, shards), HUD button Index.
```
Images 1 to 3 are the three small reward chips from my game's Index rewards box (rbxassetid 89282702522748, 82056408182031, 78947574768646). Copy their exact square frame: border colour, corner rounding, inner shading and the 47 x 50 proportions. Image 4 is my Index button (rbxassetid 118941334639047): take ONLY its green book with the red apple on the cover. Make ONE new chip in that same frame with the green book centred, filling 70% of the inside. No text. Transparent outside the chip. Canvas 188 x 200.
```

### R4. Storage chip (NEW)
- **Used:** the REWARDS box of the Rebirth menu, the "+1 storage" line ([[Rarity Gating and Rebirth]] U4, A5; `Config.StorageChip`).
- **File:** `~/fruit-empire/art/rebirth/storage-chip.png` (188 x 200)
- **Attach, in this order:** the three RewardsBox chips, HUD button Inventory.
```
Images 1 to 3 are the three small reward chips from my game's Index rewards box (rbxassetid 89282702522748, 82056408182031, 78947574768646). Copy their exact square frame: border colour, corner rounding, inner shading and the 47 x 50 proportions. Image 4 is my Inventory button (rbxassetid 108477171933167): take ONLY its wicker basket of fruit. Make ONE new chip in that same frame with the basket centred, filling 70% of the inside. No text. Transparent outside the chip. Canvas 188 x 200.
```

### R5. Green tick (NEW)
- **Used:** next to each met need in the Rebirth menu ([[Rarity Gating and Rebirth]] U4, A6; `Config.TickIcon`).
- **File:** `~/fruit-empire/art/rebirth/tick.png` (128 x 128)
- **Attach:** RewardsBox.ClaimButton, HUD button Index.
```
One green check mark icon. Image 1 is my game's green Claim button (rbxassetid 132804991293392): use its exact green and glossy top highlight. Image 2 is my Index button (rbxassetid 118941334639047): use its thick dark outline. Transparent background, centred, 10% padding, no text. Canvas 128 x 128.
```

### R6. LOCKED plate (NEW)
- **Used:** the Spawn Rarity upgrade card at the spawn ceiling ([[Rarity Gating and Rebirth]] U7, A7; `Config.UpgradeLockedButton`). Fallback: the Maxed plate, tinted grey.
- **File:** `~/fruit-empire/art/rebirth/locked-plate.png` (578 x 166)
- **Attach:** UpgradeMenu Maxed button, your finished `padlock.png` (R2).
```
Image 1 is the grey MAXED plate from my upgrade cards (rbxassetid 76326716689640). Make the same plate: same shape, 289 x 83 proportions, same bevel, outline and grey. Replace its word with a small gold padlock on the left (style of Image 2, my padlock icon) and the word LOCKED in white heavy rounded letters with a thick black outline, matching the lettering on Image 1. Transparent outside the plate. Canvas 578 x 166.
```
If LOCKED comes out misspelled, make the plate with the padlock only. Claude adds the word in FredokaOne in code.

---

## Needed for M2: UI kit and King UI

### R7. Kit recolours A1 to A9: pills, royal chips and the one bar pair (NEW)
- **What:** blank versions of your own buttons and bars in new colours. Figma first (duplicate your component, delete the label and icon layers, recolour). ChatGPT only as a fallback.
- **Used:** every royal button and chip ([[King UI Overhaul]] 7.5; `UIKit.Images`). **A8 and A9 are the one bar pair for the whole expansion** (D-027): royal timers (M2), fighter HP (M3), fruit health (M5), stamina (M9), boat HP (M10), ore nodes (M13). Make A8 and A9 first.
- **Folder:** `~/fruit-empire/art/kit/`
- **ChatGPT chat setup.** Attach crops of `Images/throne-kingdom-menu.jpg` (the Lock and Upgrade pills, the Friend Boost, Shards and Cash bars) and `Images/ui-reference-index-menu.jpg` (the blue title bar and the green Claim button). Paste once:
```
I am making UI art for my Roblox game Fruit Empire. I will attach pieces of my existing UI kit. Copy their style exactly:
- glossy, chunky cartoon look, like a Roblox simulator game
- soft gradient shading with a bright white highlight at the top left
- saturated colours
- a thick near-black outline around the whole object, the same thickness as in my attached art
- front view, centred, with empty padding on every side
Rules for every image: one object only, transparent PNG background, no text, no letters, no numbers, no logos, nothing cut off at the edges, no shadow on the background. I will ask for one image per message. Keep the style identical across all of them.
Reply only "Ready" to this message. Do not make an image yet.
```

| Id | File (canvas) | Kit piece it copies | Figma route | ChatGPT message (paste after the setup) |
|---|---|---|---|---|
| A1 | kit_pill_green.png (360 x 88) | TopButtons.UpgradeButton (shape), RewardsBox.ClaimButton (green) | duplicate the Upgrade pill, delete arrow and word, fill with the Claim green | Image A1. Canvas 360x88. A blank pill-shaped button that fills the canvas with 6 px padding. Copy the exact shape, rounded ends, glossy top band, darker bottom lip and thick outline of my Upgrade top button (TopButtons.UpgradeButton, asset 116083144480280). Colour it the green of my Claim button (RewardsBox.ClaimButton, asset 132804991293392). The face must be empty: no arrow, no text. |
| A2 | kit_pill_blue.png (360 x 88) | TopButtons.UpgradeButton, IndexMenu.TitleBlue (blue) | same, fill with the title-bar blue | Image A2. The same pill as A1, coloured the glossy sky blue of my INDEX title bar (IndexMenu.TitleBlue, asset 95532041481396). Empty face. |
| A3 | kit_pill_red.png (360 x 88) | TopButtons.LockButton | duplicate the Lock pill, delete padlock and word | Image A3. The same pill as A1, coloured the red of my Lock top button (TopButtons.LockButton, asset 118392145629817). Empty face, no padlock. |
| A4 | kit_pill_gold.png (360 x 88) | TopButtons.UpgradeButton | duplicate the Upgrade pill, delete arrow and word | Image A4. The same pill as A1, in the yellow-gold of my Upgrade top button (asset 116083144480280). Empty face. |
| A5 | kit_pill_grey.png (360 x 88) | UpgradeMenu Maxed button | duplicate the Maxed button, delete its word, match the pill shape | Image A5. The same pill as A1, in the grey of my Maxed upgrade button (asset 76326716689640), slightly flatter so it reads as switched off. Empty face. |
| A6 | kit_royalchip_gold.png (572 x 124) | CornerHud.CashBar, ShardsBar, BoostBar | duplicate the Boost bar, widen to 572, delete the money icon, add an empty round socket at the left | Image A6. Canvas 572x124. A wide rounded HUD bar in the same style as my Cash and Shards counter bars (CornerHud.CashBar asset 100015543922655, CornerHud.ShardsBar asset 140489943167779): thick outline, glossy bevel, darker inner face. Dark royal green face with a gold rim, like my Friend Boost bar (asset 85577747844362). At the left end, an empty round socket 104 px across (a darker disc with a thin gold ring) where an icon will go. No icon, no text, no money symbol. |
| A7 | kit_royalchip_stone.png (572 x 124) | same as A6 | recolour A6 grey and silver | Image A7. The same bar as A6, but cool stone grey with a silver rim. Same empty socket. |
| A8 | kit_bar_track.png (336 x 54) | CarryInfo.RateBar | duplicate the rate bar, delete its text, darken the inside | Image A8. Canvas 336x54. An empty progress-bar track in the style of my fruit rate bar (CarryInfo.RateBar, asset 120297790302429): rounded ends, thick outline, dark recessed inside. Empty. |
| A9 | kit_bar_fill.png (336 x 54) | CarryInfo.RateBar | the inner shape of A8, 6 px smaller, white | Image A9. Canvas 336x54. The fill for A8: the same rounded shape, 6 px smaller on every side, pure white with a soft vertical light-grey gradient and a thin brighter line along the top. No outline. It must be white because the game tints it. |

A10 (the charge ring) is in R12, because M3 needs it.

### R8. Royal HUD icons B1 to B12 (PENDING)
- **Used:** the royal chip, action pill, Kingdom panel tabs and cards ([[King UI Overhaul]] 5.2 to 5.6, 7.6; keys in `C.Icons`). Fallback: no icon (an id of 0 shows nothing, open question 40), never an emoji.
- **Folder:** `~/fruit-empire/art/royal-icons/` (256 x 256 each)
- **Chat setup.** Attach `~/fruit-empire/art/royal-icons/style-reference.png` and crops of HUD buttons Inventory, Index and Shop from `Images/ui-reference-hud-left.png`. Paste once (this chat also does R9 to R11):
```
I am making UI art for my Roblox game Fruit Empire. I will attach pieces of my existing UI kit. Copy their style exactly:
- glossy, chunky cartoon look, like a Roblox simulator game
- soft gradient shading with a bright white highlight at the top left
- saturated colours
- a thick near-black outline around the whole object, the same thickness as in my attached art
- front view, centred, with empty padding on every side
Rules for every image: one object only, transparent PNG background, no text, no letters, no numbers, no logos, nothing cut off at the edges, no shadow on the background. I will ask for one image per message. Keep the style identical across all of them.
Style: exactly like my HUD button art (InventoryButton asset 108477171933167, IndexButton asset 118941334639047, ShopButton asset 71485100495903), attached. Canvas 256x256, the object fills about 80%.
Reply only "Ready" to this message. Do not make an image yet.
```

| Id | Key | File | Message to paste |
|---|---|---|---|
| B1 | Crown | icon_crown.png | Icon B1: a golden royal crown with five points, a small gold ball on each point, red and green gems on the band. |
| B2 | Throne | icon_throne.png | Icon B2: a royal throne seen from the front, carved gold frame, tall back, green velvet cushion. |
| B3 | Swords | icon_swords.png | Icon B3: two crossed silver swords with gold hilts and red grips. |
| B4 | Watch | icon_watch.png | Icon B4: a pair of golden binoculars with blue glass lenses. |
| B5 | Decree | icon_decree.png | Icon B5: a rolled parchment scroll, partly open, gold end caps, a red wax seal stamped with a crown. |
| B6 | Trial | icon_trial.png | Icon B6: a curved golden herald's horn with a small green banner hanging from it, the banner showing a gold crown. |
| B7 | Status | icon_status.png | Icon B7: a golden trophy cup with two handles and a small crown on top. |
| B8 | Passage | icon_passage.png | Icon B8: a small cartoon castle, grey stone walls, two round towers with green cone roofs and little gold flags. |
| B9 | Shield | icon_shield.png | Icon B9: a green royal shield with a gold rim and a gold crown emblem in the middle. |
| B10 | Reputation | icon_reputation.png | Icon B10: a gold five-point star medal with a small crown on top and a red ribbon underneath. |
| B11 | Lock | (use `padlock.png`, R2) | none: already made in R2 |
| B12 | Reign | icon_reign.png | Icon B12: a golden hourglass with blue sand, about half run through. |

### R9. Decree icons C1 to C5 (PENDING, King UI phase 2)
- **Used:** the decree menu and decree card ([[King UI Overhaul]] 5.3, 5.6, 7.7). **Same chat as R8.** Also attach HUD button Merge All before C1.

| Id | Key | File | Message to paste |
|---|---|---|---|
| C1 | MergeRush | icon_mergerush.png | Also match the sparkles and energy of my Merge All button (MergeAllButton, asset 99980479639840), attached. Icon C1: a red apple and an orange bumping together, a big yellow lightning bolt between them, a few small four-point sparkles. |
| C2 | FruitFrenzy | icon_fruitfrenzy.png | Same Merge All energy. Icon C2: a burst of mixed fruit flying outward (cherries, a strawberry, a banana, a bunch of grapes) with short motion lines. |
| C3 | RoyalRace | icon_royalrace.png | Same Merge All energy. Icon C3: a black-and-white checkered race flag waving on a gold pole, a small gold crown on top of the pole. |
| C4 | CrownHunt | icon_crownhunt.png | Same Merge All energy. Icon C4: a golden crown half hidden in a leafy green bush, a magnifying glass in front of it. |
| C5 | EmpireBuild | icon_empirebuild.png | Same Merge All energy. Icon C5: a tall stack of gold coins with a big green arrow beside it pointing up. Match the money green of my Cash chip (RewardsBox chip, asset 82056408182031). |

### R10. Achievement medals D1 to D8 (PENDING, King UI phase 2)
- **Used:** the achievements list, grey until earned ([[King UI Overhaul]] 5.6, 7.8). **Same chat.** Attach the three RewardsBox chips, then send this line first:
```
The next eight images are medals. All eight medals use the SAME round medal: a thick rim around a darker inner disc, a symbol in the middle. Same outline weight as my reward chips (RewardsBox chips, assets 89282702522748, 82056408182031, 78947574768646), attached. Only the rim colour and the symbol change. Reply only "Ready".
```

| Id | Key | File | Message to paste |
|---|---|---|---|
| D1 | FirstCrown | medal_firstcrown.png | Medal D1: bronze rim; symbol a small gold crown. |
| D2 | CrownKeeper | medal_crownkeeper.png | Medal D2: silver rim; symbol a green shield with a gold crown on it. |
| D3 | Tyrant | medal_tyrant.png | Medal D3: red rim; symbol a gold crown with a small hourglass in front. |
| D4 | Emperor | medal_emperor.png | Medal D4: purple rim; symbol a gold crown inside a green laurel wreath. |
| D5 | Dynasty | medal_dynasty.png | Medal D5: gold rim; symbol three gold crowns stacked. |
| D6 | Conqueror | medal_conqueror.png | Medal D6: dark steel rim; symbol two crossed swords behind a gold crown. |
| D7 | Undisputed | medal_undisputed.png | Medal D7: orange rim; symbol a gold crown surrounded by flames. |
| D8 | Legend | medal_legend.png | Medal D8: rainbow rim; symbol a glowing white-gold crown with small angel wings and tiny stars. |

### R11. Crown styles E1 to E6 (PENDING, King UI phase 2)
- **Used:** the Crown Style tab of the Kingdom panel ([[King UI Overhaul]] 5.6, 7.9). **Same chat.** Attach your finished `icon_crown.png` (B1), then send this line first:
```
The next six images are the SAME crown as icon B1 (attached): identical shape, size and angle. Only the colours change. Reply only "Ready".
```

| Id | Key | File | Message to paste |
|---|---|---|---|
| E1 | Golden | crown_golden.png | Crown E1: band gold (RGB 255, 198, 46), gems red (230, 40, 50). |
| E2 | Emerald | crown_emerald.png | Crown E2: band gold (255, 198, 46), gems bright green (40, 220, 110). |
| E3 | Ruby | crown_ruby.png | Crown E3: band warm orange-gold (255, 186, 60), gems pink-red (255, 30, 80). |
| E4 | Shadow | crown_shadow.png | Crown E4: band dark iron grey (60, 58, 70), gems glowing purple (150, 70, 255). |
| E5 | Celestial | crown_celestial.png | Crown E5: band white-silver (235, 245, 255), gems sky blue (90, 200, 255), a faint white glow. |
| E6 | Soul | crown_soul.png | Crown E6: band glowing hot pink (255, 120, 230), gems white, a soft pink glow. |

---

## Needed for M3: Royal Combat core

### R12. Combat buttons F1 to F7 with words, and the charge ring A10 (NEW)
- **Used:** the touch combat buttons in the arena ([[King UI Overhaul]] 5.9, 7.10; `C.CombatButtons`) and the heavy-strike charge ring ([[King UI Overhaul]] 7.5 A10; `UIKit.Images.RingCharge`). Fallback: kit pills with FredokaOne words.
- **Folder:** `~/fruit-empire/art/royal-icons/`
- **Step 1, icons (ChatGPT).** In the R8 chat, attach HUD buttons Merge All and Clear, then send this line first:
```
The next seven images are button icons. Canvas 256x256. Put the object in the top 70% only and leave the bottom 30% completely empty (I add a word there). Style like my Merge All and Clear buttons (MergeAllButton asset 99980479639840, ClearButton asset 88888506311530), attached. Reply only "Ready".
```

| Id | File | Message to paste | Word added in step 2 |
|---|---|---|---|
| F1 | btn_attack.png | Icon F1: a silver sword slashing diagonally with a short white swoosh arc. | Attack |
| F2 | btn_block.png | Icon F2: a round wooden shield with a steel rim and a gold centre boss. | Block |
| F3 | btn_dash.png | Icon F3: a green boot with small white wings and speed lines behind it. | Dash |
| F4 | btn_slam.png | Icon F4: a gold gauntlet fist hitting cracked stone ground, small rocks flying. | Slam |
| F5 | btn_wave.png | Icon F5: a gold crown sending out two golden rings of wave around it. | Wave |
| F6 | btn_guard.png | Icon F6: a gold crown in front of a glowing blue shield bubble. | Guard |
| F7 | btn_knights.png | Icon F7: a steel knight helmet with a green plume, two smaller helmets behind it. | Knights |

- **Step 2, words (Figma).** This is also the word step every other request points to:
```
Use the label style baked into my Clear button (ClearButton, asset 88888506311530): same font, white fill, same thick dark outline. Put the word centred in the empty bottom 30% of each F image. Export 256x256 PNG, transparent.
```
- **A10 charge ring.** Figma route: a circle stroke in the style of Templates.Cell.Selection. ChatGPT route (in the R7 chat, attach Templates.Cell.Selection). File `~/fruit-empire/art/kit/kit_ring_charge.png` (300 x 300):
```
Image A10. Canvas 300x300. A plain circular ring 18 px thick, in the white glossy style of my card selection outline (Templates.Cell.Selection, asset 113285622864672). White only, empty centre, thin dark outline on both edges.
```
The 7 stinger titles (FIGHT! to SUDDEN DEATH) for this milestone are in R1.

---

## Needed for M4: Potions

### R13. Feather Falling and Luck: shop art and badges (PENDING)
- **Used:** the Potions tab cards and the effect block ([[Potions]] UI 1 and 3; `Config.Tools.<name>.image`, `Config.Effects.<name>.icon`). Fallback: Claude's placeholder badges, already live.
- **Folder:** `~/fruit-empire/art/potion-icons/`
- **Attach to the first message:** `~/fruit-empire/art/potion-icons/style-reference-potions.png` (your Potion of Aether bottle and badges). Ask for each image on its own.

`featherfalling-shop.png`
```
Pixel art game icon in the exact style of the attached potion bottle: a chunky Minecraft-style glass potion bottle with a brown cork, glossy glass highlights, glowing liquid and small four-point sparkles. Transparent background, no text.

This one is a FEATHER FALLING potion: the liquid is a soft peach-cream color, with a small white feather floating inside it, and two or three white pixel feathers drifting down around the outside of the bottle.
```
`featherfalling-badge.png`
```
Pixel art status icon in the exact style of the attached rounded badges: a rounded square tile with a vertical gradient, a thick black outline and a white outer rim, with one pixel-art symbol in the middle. No text.

This one: a light periwinkle-blue gradient (lighter at the top), and the symbol is a white pixel feather drifting diagonally down, with a few tiny white sparkles.
```
`luck-shop.png`
```
Pixel art game icon in the exact style of the attached potion bottle: a chunky Minecraft-style glass potion bottle with a brown cork, glossy glass highlights, glowing liquid and small four-point sparkles. Transparent background, no text.

This one is a LUCK potion: the liquid is shining gold, with a small green four-leaf clover floating inside it, and a few gold sparkles around the bottle.
```
`luck-badge.png`
```
Pixel art status icon in the exact style of the attached rounded badges: a rounded square tile with a vertical gradient, a thick black outline and a white outer rim, with one pixel-art symbol in the middle. No text.

This one: a warm gold-to-orange gradient (lighter at the top), and the symbol is a bright green pixel four-leaf clover with a short stem, with a few tiny white sparkles.
```

### R14. Speed, Jump and Invisibility shop art (NEW, nice to have)
- **Used:** the Potions tab cards, which today show a spinning 3D bottle ([[Potions]] asset requests 5 to 7). The badges already exist.
- **Folder and chat:** same as R13.

`speed-shop.png`
```
Pixel art game icon in the exact style of the attached potion bottle: a chunky Minecraft-style glass potion bottle with a brown cork, glossy glass highlights, glowing liquid and small four-point sparkles. Transparent background, no text.

This one is a SPEED potion: the liquid is a warm orange-gold, with a small brown pixel boot inside it, and three short white speed lines streaking past the left side of the bottle.
```
`jump-shop.png`
```
Pixel art game icon in the exact style of the attached potion bottle: a chunky Minecraft-style glass potion bottle with a brown cork, glossy glass highlights, glowing liquid and small four-point sparkles. Transparent background, no text.

This one is a JUMP potion: the liquid is a bright lime green, with a small white pixel up-arrow inside it, and two or three green sparkles rising above the cork.
```
`invisibility-shop.png`
```
Pixel art game icon in the exact style of the attached potion bottle: a chunky Minecraft-style glass potion bottle with a brown cork, glossy glass highlights, glowing liquid and small four-point sparkles. Transparent background, no text.

This one is an INVISIBILITY potion: the liquid is a pale silvery white that fades to see-through toward the top of the bottle, with a small gold pixel hand mirror inside it and a few faint white sparkles.
```

### R15. Blank green action button (CHECK FIRST)
- **Used:** only if RewardsBox.ClaimButton has the word CLAIM baked into its image ([[Potions]] asset request 8). Claude checks in Studio and tells you. Do not make it before then.
- **File:** `~/fruit-empire/art/potion-icons/green-action-blank.png`. **Attach:** a screenshot of the Index panel's Claim button.
```
Game UI button in the exact style of the attached green CLAIM button: same glossy green face, same darker green bottom lip, same thick black outline, same corner radius and proportions (about 4.6 wide to 1 tall). Completely blank: no text, no icon. Transparent background.
```

---

## Needed for M5: Explosives and fruit health

### R16. Nuke shop card art (OPTIONAL)
- **Used:** the Nuke card on the Weapons tab, instead of the spinning model ([[Explosives and Fruit Health]] section 8, request 2). Fruit health bars use A8 and A9 (R7); no separate bar.
- **File:** `~/fruit-empire/art/explosives/nuke-card.png` (512 x 512). **Attach:** the Potion of Aether card art (123282319597102, in `style-reference-potions.png`).
```
Make one transparent PNG, 512 x 512, for a Roblox shop card, matching the attached Potion of Aether card art in style (rbxassetid 123282319597102): a chunky cartoon bomb built from Roblox-style studded plastic blocks, smoky grey body, rounded nose, four flat fins with studs, one yellow band with two black stripes, small lit fuse spark at the top, soft top-left lighting, thick dark outline. No text, no emoji, no background, centred with 10% padding.
```

---

## Needed for M6 and M7: plot and world builders

### R17. Approve the builder screenshots, then save and publish (Studio steps)
No images. [[Plot Overhaul and Environment]] reuses kit images that are already uploaded.
1. Claude runs one builder at a time and sends screenshots (the plaza, Plot0, Plot6, the beach at four bearings and the sky island deck).
2. Reply "go", or name what looks wrong. Every builder has an undo and a dated backup in ServerStorage.
3. After M7: in Explorer, delete `Workspace._AssetViewer` (the temporary pack viewer at 6000, 1000, 6000). If you plan to use its sword rigs for R34, do R34 first. The same rigs also stay in `ServerStorage.AssetQuarantine`.
4. File > Save to Roblox. Then File > Publish to Roblox.

---

## Needed for M8: Bats

### R18. Bat icon, tintable (NEW)
- **Used:** the BatSlot, the touch action slot and the bat card ([[Bats]] UI 1, 3, 4; asset request 1; `Config.Bats.Images.Icon`). The game tints it the rarity colour. Fallback: a ViewportFrame bat with a FredokaOne label.
- **File:** `~/fruit-empire/art/bats/bat-icon.png` (512 x 512). **Attach:** `Images/ui-reference-hud-icons.png` and HUD button Clear.
```
Draw one cartoon baseball bat icon for a Roblox game HUD, 512 x 512 pixels, transparent background. Copy the exact art style of the attached 'Clear' broom icon (HUD button Clear, Roblox asset 88888506311530): the same thick dark outline, the same glossy cartoon shading, the same chunky rounded proportions and the same 3/4 angle. The bat points from lower left (handle) to upper right (barrel), filling about 80% of the canvas. Colour the whole bat body in white to light grey only (shading from #FFFFFF down to #C8C8C8), because the game will tint it a different colour for each rarity. Give the handle a dark charcoal grip tape (#2B2B2B) with two thin wraps, and keep the outline dark. Add one soft white shine streak along the barrel. No text, no sparkles, no background, no emoji.
```
If the tint looks flat in game, the fallback is 9 pre-coloured copies, one per rarity colour (open question 31).

### R19. "Bat" label (NEW)
- **Used:** under the bat on the BatSlot ([[Bats]] UI 1, request 2; `Config.Bats.Images.Label`).
- **File:** `~/fruit-empire/art/bats/label_bat.png` (256 x 96).
- **Best route:** Figma, with the word step from R12 (the Clear button's label style), word "Bat", canvas 256 x 96. **ChatGPT route** (attach `Images/ui-reference-hud-icons.png` and HUD button Clear):
```
Make only the word 'Bat' as a game UI label, 256 x 96 pixels, transparent background. Match the attached HUD labels 'Clear' and 'Shop' exactly: the same heavy rounded font, white letters with the same slight white-to-light-grey vertical gradient, the same thick dark outline and the same small drop shadow. Title case: capital B, lower-case 'at'. Centre it. No icon, no other text, no emoji.
```

### R20. "Swing" label (NEW)
- **Used:** the touch Swing button ([[Bats]] UI 3, request 3; `Config.Bats.Images.SwingLabel`).
- **File:** `~/fruit-empire/art/bats/label_swing.png` (320 x 96). Figma route as R19 with the word "Swing". ChatGPT route (same attachments):
```
Make only the word 'Swing' as a game UI label, 320 x 96 pixels, transparent background. Match the attached HUD labels 'Clear' and 'Shop' exactly: the same heavy rounded font, white letters with the same slight vertical gradient, the same thick dark outline and drop shadow. Title case. Centre it. No icon, no other text, no emoji.
```

### R21. Bat chip (NEW)
- **Used:** the REWARDS box of the Rebirth menu, tinted the next tier's colour ([[Bats]] request 5; [[Rarity Gating and Rebirth]] U4; `Config.Bats.Images.Chip`). Hidden until M8.
- **File:** `~/fruit-empire/art/bats/bat-chip.png` (188 x 200). **Attach, in this order:** the three RewardsBox chips, your finished `bat-icon.png` (R18).
```
Images 1 to 3 are the small reward chips from my game's Index rewards box (RewardsBox chips, rbxassetid 89282702522748, 82056408182031, 78947574768646). Copy their exact square frame: border colour, corner rounding, inner shading and 47 x 50 proportions. Image 4 is my white bat icon. Put ONE copy of that bat, white to light grey only so the game can tint it, centred and filling 70% of the chip. No text. Transparent outside the chip. Canvas 188 x 200.
```

### R22. Two bat swing animations (NEW; fallback is the default tool slash)
- **Used:** every bat swing ([[Bats]] Animations, request 4; `Config.Bats.Anim.SwingA`, `SwingB`). An id of 0 keeps the built-in `toolanim Slash`.
- **Studio steps:**
  1. Avatar tab > Rig Builder > insert a **Block R15** rig.
  2. Copy `ServerStorage.BatTemplate.Handle` into the rig (Claude builds `BatTemplate` in M8). Add a Motor6D named `BatGrip`: `Part0 = RightHand`, `Part1 = Handle`, `C0 = RightHand.RightGripAttachment.CFrame`, `C1 = CFrame.new(0, 0, -1.7)`. It only shows where the bat is. Do not key BatGrip.
  3. Open the Animation Editor on the rig. Key only UpperTorso, Head, both arms and both hands. Never key LowerTorso or the legs, so the walk keeps playing under the swing.
  4. **Swing A (forehand), 0.40 s:** 0.00 s the tool-hold pose (right arm forward, bat up). 0.08 s wind-up: UpperTorso turned 25 degrees right, right upper arm pulled back 40 degrees, bat behind the right shoulder, easing Quad Out. 0.14 s contact: torso square to the front, right arm straight out at shoulder height, bat level and pointing forward-left, easing Cubic In; add an animation event named `Hit` here. 0.24 s follow-through: torso turned 30 degrees left, bat over the left shoulder, easing Quad Out. 0.40 s back to the hold pose.
  5. **Swing B (backhand):** the mirror image, starting from the left side. `Hit` event at 0.14 s.
  6. Set Priority to **Action** and turn Looping off for both.
  7. Animation Editor "..." menu (top left of the editor) > **Publish to Roblox**. Not the main File menu: that one publishes the whole game. Names: `FE Bat Swing A`, `FE Bat Swing B`. Publish as the game's owner (see R34 step 1).
  8. Send the two ids.
- **Time saver:** the first two slashes of `swordcombo1` (R34) can be copied in as A and B, then retimed to the steps above. Do this in the same session as R34.

---

## Needed for M9: Swimming

### R23. Aqua Affinity shop art and badge (NEW)
- **Used:** the Aqua Affinity potion card and its badge in the effect block ([[Potions]] asset requests 1 and 2). Fallback: the 3D bottle on the card and a placeholder badge, as for Feather Falling and Luck.
- **Folder and chat:** same as R13.

`aquaaffinity-shop.png`
```
Pixel art game icon in the exact style of the attached potion bottle: a chunky Minecraft-style glass potion bottle with a brown cork, glossy glass highlights, glowing liquid and small four-point sparkles. Transparent background, no text.

This one is an AQUA AFFINITY potion: the liquid is a bright ocean blue, with a small white wave curling inside it, and three or four round white bubbles rising out of the liquid and up past the cork.
```
`aquaaffinity-badge.png`
```
Pixel art status icon in the exact style of the attached rounded badges: a rounded square tile with a vertical gradient, a thick black outline and a white outer rim, with one pixel-art symbol in the middle. No text.

This one: a deep ocean-blue gradient (bright azure at the top, navy at the bottom), and the symbol is a white-and-aqua pixel wave curling to the right, with three small round bubbles rising above it.
```

## Needed for M10: Boats
No new item. The BOAT DOCK title is in R1. After the docks are built, do the save and publish steps at the end of this note.

---

## Needed for M11 and M12: Pets

### R24. Pets HUD button (NEW)
- **Used:** the new Pets button in the left column under Shop ([[Pets]] U1, A2; `PetConfig.Images.HudButton`).
- **File:** `~/fruit-empire/art/pets/pets-hud-button.png` (512 x 491). **Attach, in this order:** HUD button Index, HUD button Inventory, vault `Images/pack-animals.jpg` (the Sheep is the third animal from the left).
```
Image 1 is my game's Index HUD button (rbxassetid 118941334639047) and Image 2 is my Inventory HUD button (rbxassetid 108477171933167). Copy their exact style: one chunky glossy cartoon object, thick dark outline, saturated colours, a bright highlight at the upper left, slight tilt. Image 3 is the blocky white Sheep from the Stud-Themed Animal Pack (the third animal from the left in the screenshot). Draw ONE object: that blocky white sheep, made of rounded cubes with soft studs on top, standing in a small wooden feed trough that holds one red apple with a green leaf (the same apple as on Image 1). Front three-quarter view, centred, filling the top 70% of the canvas. Leave the bottom 30% empty for a label. No text, no background, no sparkles, no shadow. Canvas 512 x 491.
```
Then add the word "Pets" in the bottom 30% in Figma: FredokaOne, white fill, a thick black outline, the same size and position as the word "Index" on the Index button.

### R25. Index tab icons: Fruits, Pets, Ores (NEW)
- **Used:** the category tabs on the left of the Index ([[Underground and Ores]] U4, A2; [[Pets]] U5; keys `Config.IndexTabFruitsIcon`, `IndexTabPetsIcon`, `IndexTabOresIcon`). First shown in M12. Fallback: FredokaOne words on the shop tab image.
- **Folder:** `~/fruit-empire/art/index/` (200 x 264 each). **Attach:** ShopMenu.ToolsTab, HUD button Index. Send the three messages in one chat.

`tab_fruits.png`
```
Create a transparent PNG, 200 x 264 pixels, to sit on top of my game's vertical shop tab (attached, ShopMenu.ToolsTab, asset 110917091182913). Style reference: my Index HUD button (attached, asset 118941334639047): a glossy cartoon object with a thick black outline, and the label baked underneath in white rounded bold text with a black stroke. Icon: one shiny red apple, label "Fruits".
```
`tab_pets.png`
```
Same canvas, style and layout as the last image, so the two line up exactly. Icon: the head of a blocky white studded sheep, label "Pets".
```
`tab_ores.png`
```
Same canvas, style and layout again, so all three line up. Icon: two chunky blocky grey stone ore lumps with square studs on top, one dotted with small black gem cubes and one with small cream gem cubes, label "Ores".
```
If a word comes out wrong, ask for the icon without the label and add the word in Figma (word step in R12).

---

## Needed for M13: Underground and ores

### R26. OWNED button (NEW)
- **Used:** the floor cards in the Mine panel once a floor is bought ([[Underground and Ores]] U2, A3; `Config.OwnedButtonImage`). Fallback: the green Claim art with FredokaOne "OWNED". The MINE title is in R1.
- **File:** `~/fruit-empire/art/mine/owned-button.png` (346 x 122). **Attach:** the shop card's Buy button, RewardsBox.ClaimButton.
```
Create a transparent PNG, 346 x 122 pixels. Copy the exact shape, gloss, border and drop shadow of my shop Buy button (attached, asset 97067256017679), but remove the shard crystal, change the fill to the calm green of my Claim button (attached, RewardsBox.ClaimButton, asset 132804991293392), and bake the word "OWNED" in the same white rounded bold text with a black stroke, centred.
```

### R27. Pickaxe swing animation (OPTIONAL)
- **Only if** the M13 playtest shows the default tool slash looks wrong ([[Underground and Ores]] A5, test for rule 10).
- **Studio steps:** Rig Builder > Block R15. Animation Editor: an overhead chop, 0.5 s, Priority Action, Looping off. "..." menu > Publish to Roblox as `FE Pickaxe Chop`, as the game's owner. Send the id.

---

## Needed for M14 and M15: Weapons

### R28. Weapons tab icon (CHECK FIRST)
- Probably not needed. The King tab was made by cloning the Tools tab and changing its label to KING (`~/fruit-empire/NOTES.md`, Shop entry), so the tab label is likely text. Claude confirms in Studio ([[Weapons and Artillery]] A1). Only if Claude asks: file `~/fruit-empire/art/weapons/tab_weapons.png`, attach HUD button Shop:
```
Use the attached Fruit Empire 'Shop' HUD button (a red basket full of fruit, Roblox image 71485100495903) as the exact style reference: a chunky cartoon game icon with a thick dark outline, glossy highlights and soft shading. Draw a short studded wooden mortar tube (Roblox-style blocks with round studs on top) with a striped watermelon shell in its mouth, angled up and to the right. 512x512 PNG, transparent background, centred, the same line weight and colour saturation as the reference. No text, no letters, no emoji.
```

### R29. Four weapon slot tiles (NEW)
- **Used:** the Weapon Bar slots and the Weapons tab cards ([[Weapons and Artillery]] UI 1 and 2, A2). Cherry Bomb and Mortar first show in M14, Cannon and Barrage in M15. The gradients are the kit's Uncommon, Rare, Legendary and Astral colours (`Config.RarityColors`).
- **Folder:** `~/fruit-empire/art/weapons/` (512 x 512 each). **Attach:** the Speed tile and the Jump tile (in `style-reference-potions.png`).

`tile_cherrybomb.png`
```
Style reference: the attached Fruit Empire potion effect icons (the Speed tile, Roblox image 85746922401157, and the Jump tile, Roblox image 103629048118784): a rounded square tile with a thick black outline and a thin white inner rim, a smooth vertical gradient background, and a centred pixel-art object with chunky pixels and a 1-pixel dark outline. One 512x512 PNG tile, transparent outside the tile, no text and no emoji. Cherry Bomb: two round red cherries joined by a green stem with a short lit fuse, gradient #4EC265 at the top to #378847 at the bottom.
```
`tile_mortar.png`
```
Same tile style as the last image, one 512x512 PNG, no text. Melon Mortar: a stubby studded wooden mortar tube with a watermelon shell, gradient #4EA4E6 at the top to #3772A1 at the bottom.
```
`tile_cannon.png`
```
Same tile style again, one 512x512 PNG, no text. Coconut Cannon: a brown shoulder launcher made of studded blocks with a coconut in the barrel, gradient #F2AB30 at the top to #A97822 at the bottom.
```
`tile_barrage.png`
```
Same tile style again, one 512x512 PNG, no text. Starfruit Barrage: a small golden horn with three yellow starfruit falling from above, gradient #47DED9 at the top to #329B98 at the bottom.
```

### R30. Two War Flag tiles (NEW)
- **Used:** the flag slot on the Weapon Bar ([[Weapons and Artillery]] UI 2, A3). **Same chat and folder as R29.**

`flag_down.png`
```
Same tile style as the weapon tiles (the Speed tile 85746922401157 and the Jump tile 103629048118784), one 512x512 tile, no text, no emoji. Flag Down: a white cloth flag hanging limp on a short studded wooden pole, gradient #97A9BD at the top to #697684 at the bottom.
```
`flag_up.png`
```
Same tile style, one 512x512 tile, no text, no emoji. Flag Up: a red cloth flag flying straight out from the same pole, gradient #E8423C at the top to #A22E2B at the bottom.
```

### R31. Reticle and hit marker (NEW)
- **Used:** the aim reticle and the hit confirmation at screen centre ([[Weapons and Artillery]] UI 3, A4 and A5).
- **Folder:** `~/fruit-empire/art/weapons/` (256 x 256 each). **Attach:** ShopMenu.Panel.CloseButton (the red X).

`reticle.png`
```
Style reference: the Fruit Empire red X close button (ShopMenu.Panel.CloseButton, Roblox image 107111219044453): a thick black outline around a white shape with a slight glossy highlight on top. An aiming reticle: a white ring with four short white ticks at the top, bottom, left and right, and a small white centre dot, every piece outlined in 6-pixel black. 256x256 PNG, transparent background, only white and black.
```
`hitmarker.png`
```
Same style as the reticle. A hit marker: four short white diagonal dashes forming an X around an empty centre, with the same 6-pixel black outline. 256x256 PNG, transparent background, only white and black.
```

### R32. Fire button for touch (NEW)
- **Used:** the touch action slot's Fire state at sea ([[Weapons and Artillery]] UI 5, A6).
- **File:** `~/fruit-empire/art/weapons/btn_fire.png` (512 x 512). **Attach:** HUD button Merge All.
```
Style reference: the Fruit Empire 'Merge All' HUD button (Roblox image 99980479639840): a chunky cartoon icon on top with the button's word baked underneath in a heavy rounded white font with a thick black outline (the same lettering as 'Merge All', 'Rebirth' and 'Clear'). Draw a round red cherry bomb with a lit fuse and two small yellow sparkle stars, and bake the word 'Fire' underneath in that exact lettering. 512x512 PNG, transparent background, icon in the top two thirds and the word in the bottom third, like the reference.
```

### R33. Five weapon animations (OPTIONAL; fallback is a code-driven arm swing)
- **Used:** throwing and firing ([[Weapons and Artillery]] Animations; `WeaponConfig.Animations`). Needed for M15 polish, never blocking.
- **Studio steps:** on an R15 dummy (Rig Builder > Block R15, or one of the pack's sword dummies), make each clip in the Animation Editor, add the named animation event, then "..." menu > Publish to Roblox as the game's owner and send the ids.

| Clip | Length, priority | Poses | Event |
|---|---|---|---|
| WeaponThrow | 0.55 s, Action | arm cocks back behind the head over 0.25 s, overhand release, follow-through | `Release` at 0.30 s |
| MortarFire | 0.6 s, Action | slight crouch, both hands steady the tube at chest height, recoil | `Fire` at 0.15 s |
| CannonFire | 0.35 s, Action | tube on the right shoulder, recoil kick back, settle | `Fire` at 0.08 s |
| BarrageCall | 0.8 s, Action | horn to the mouth, then point at the target | `Call` at 0.30 s |
| WeaponHold | looped, Idle | the two-handed carry pose for the Mortar and the Cannon | none |

---

## Needed for M16: Royal Combat 2 (real sword animations)

### R34. Publish the six sword animations (PENDING; start now)
- **What:** the six clips in the Sword Animations pack (store 77935648543779). Each R15 dummy holds one KeyframeSequence, which is raw editor data and cannot play in a live game until you publish it (D-005).
- **Used:** [[Royal Combat]] asset request 1 and "Splitting the combo clip"; keys `C.Anims.Idle`, `Idle2`, `Walk`, `Run`, `Block`, `Combo`. Until then every id is 0 and duels use today's classic swing (M3 works without them).
- **Studio steps:**
  1. Sign in to Studio as **`hikeALT12212`**, the account that owns the game (open question 53). If you publish from your own account instead, afterwards open each animation in Creator Hub > Configure > Permissions and add the Fruit Empire experience, then run one Studio playtest of the game.
  2. Open place 88602028667228. In Explorer open `ServerStorage > AssetQuarantine > Q_77935648543779`. Copy the six dummies into Workspace (or use the copies already in `Workspace._AssetViewer`, before it is deleted).
  3. Avatar tab > Animation Editor. Click the dummy that holds `Sword Idle 1`.
  4. In the Animation Editor, "..." menu > **Load**, and pick that dummy's KeyframeSequence. If the list is empty: save any tiny animation on that dummy first (this creates its `AnimSaves` folder), drag the pack's KeyframeSequence into `AnimSaves`, and Load again. Claude can do this part for all six over the bridge before you start.
  5. **Only for `swordcombo1`:** add four animation events named `Hit`. Move the playhead to 0.38 s (the nearest frame is fine), right-click the animation events row above the tracks, choose Add Animation Event Here, name it `Hit` and set its parameter to `1`. Repeat at 0.98 s (`2`), 1.75 s (`3`) and 2.35 s (`4`). These are the four slashes measured in [[Creator Store Packs]]. If the events row is hidden, turn on Show Animation Events in the editor's settings. The other five clips need no events. Leave priority and looping alone: the code sets them.
  6. "..." menu (top left of the Animation Editor) > **Publish to Roblox**. Not the main File menu: that one publishes the whole game. Use the name from the table. Copy the id it shows (it is also in Creator Hub > Creations > Animations).
  7. Repeat steps 3 to 6 for the other five dummies.
  8. Send the six ids in this form: `Idle=..., Idle2=..., Walk=..., Run=..., Block=..., Combo=...`

| Dummy holds | Publish as | Config key | Animation events |
|---|---|---|---|
| Sword Idle 1 (3.00 s) | FE Sword Idle 1 | `C.Anims.Idle` | none |
| Sword Idle 2 (3.00 s) | FE Sword Idle 2 | `C.Anims.Idle2` | none |
| Sword Walk (0.93 s) | FE Sword Walk | `C.Anims.Walk` | none |
| Sword Run (0.67 s) | FE Sword Run | `C.Anims.Run` | none |
| block (one pose) | FE Sword Block | `C.Anims.Block` | none |
| swordcombo1 (2.82 s) | FE Sword Combo | `C.Anims.Combo` | `Hit` at 0.38 (1), 0.98 (2), 1.75 (3), 2.35 (4) |

---

## Needed for M17: Royal Combat 3 (blade skins and Armory)
The ARMORY title is in R1, the combat buttons and ring in R12, the bars in R7.

### R35. Storm Edge game pass (OPTIONAL, recommended)
- **Used:** one look-only blade skin for Robux ([[Royal Combat]] request 5; `C.WeaponSkinByKey.StormEdge.unlock.gamepass`). It never changes damage or reach (open question 35).
- **Creator Hub steps:** Creator Hub > your experience > Monetization > Passes > Create a Pass. Name `Storm Edge`. Description: `A royal blade with crackling lightning. Looks only: it does not change damage or reach.` Price 149 Robux (proposal). Upload the icon below. Send the pass id.
- **Icon file:** `~/fruit-empire/art/royal-combat/stormedge-pass.png` (512 x 512). **Attach:** `~/fruit-empire/art/royal-icons/style-reference.png`.
```
Make a 512x512 PNG game pass icon with a transparent background: a pale cyan sword pointing up and slightly tilted, with small white-and-cyan lightning bolts crackling around the blade. Glossy chunky 3D cartoon style like the attached icons, thick near-black outline, saturated colours. No text, no letters, no numbers.
```

### R36. Richer royal sounds (OPTIONAL)
In the Creator Store audio tab, pick three licensed sounds and send their ids: search "stone impact boom" (slam), "metal ting short" (parry), "crowd ooh" (a knockdown in the stands) ([[Royal Combat]] request 6).

### R37. Hand-made Royal Slam animation (OPTIONAL)
Only if the leap built from the combo clip reads badly in playtest ([[Royal Combat]] request 7): a 0.6 s R15 jump with the sword raised and a downward plunge, Priority Action, published like R34 as `FE Royal Slam`.

---

## Later (not in the numbered milestones)
- **R38. Haste shop art and badge**, with the second ore band (D-032). Files `~/fruit-empire/art/potion-icons/haste-shop.png` and `haste-badge.png`. Same chat as R13.
```
Pixel art game icon in the exact style of the attached potion bottle: a chunky Minecraft-style glass potion bottle with a brown cork, glossy glass highlights, glowing liquid and small four-point sparkles. Transparent background, no text.

This one is a HASTE potion: the liquid is a rich royal purple, with a small pixel pickaxe inside it (light brown handle, bright gold head), and a few small gold sparks flying off the side of the bottle.
```
```
Pixel art status icon in the exact style of the attached rounded badges: a rounded square tile with a vertical gradient, a thick black outline and a white outer rim, with one pixel-art symbol in the middle. No text.

This one: a violet gradient (light lilac at the top, deep purple at the bottom), and the symbol is a pixel pickaxe with a light brown handle and a bright gold head, tilted to the right, with three short white speed lines behind it.
```
- **R39. "+1 Pet Slot" game pass (OPTIONAL).** Creator Hub > experience > Monetization > Passes > Create a Pass; send the id ([[Pets]] asset requests, "Later").

---

## Every milestone: save and publish (Studio steps)
Map and script edits made over the Studio bridge live only in the open Studio session until you save.
1. Claude tells you when a milestone passes its tests in Studio.
2. File > Save to Roblox.
3. File > Publish to Roblox.
4. When the milestone changes the save document (M1, M4, M5, M8, M9, M10, M11, M13, M14, M17), also shut down the old servers from Creator Hub (Shut Down All Servers, or Restart Servers for Updates). For **M1 this is required**: the rebirth migration must not meet a server running old code.

## Decisions to confirm
Each open question in [[Integration Review]] section 8 has a recommended default, so building never waits. Before each milestone, reply "defaults ok for Mx" or name the numbers you want changed.

| Milestone | Open questions |
|---|---|
| M1 Rarity Gating and Rebirth | 1, 2, 3, 4, 5, 6, 19, 43 |
| M2 UI kit and King UI | 39, 40, 41, 42 |
| M3 Royal Combat core | 13, 32, 33 |
| M4 Potions | 12, 36, 37 |
| M5 Explosives | 9, 20, 21, 22, 23, 25, 45 |
| M6 Plot pens | 8, 47, 48 |
| M7 World restyle | 46, 49 |
| M8 Bats | 17, 29, 30, 31, 34, 44 |
| M9 Swimming | 51 |
| M10 Boats | 26, 50 |
| M11 and M12 Pets | 7, 14, 15, 16, 18, 38 |
| M13 Underground | 10, 11, 24, 54 |
| M14 and M15 Weapons | 27, 28 |
| M16 Sword animations | 53 |
| M17 Blade skins | 35 |
| Later | 52 |

---

## Checklist (one screen)
**Now (longest lead time)**
- [ ] R34 Publish the 6 sword animations with the 4 `Hit` events; send the 6 ids
- [ ] R1 Figma title batch: 29 titles (REBIRTH and MAX FRUIT first)
- [ ] R8 to R11 Royal icons, one image per message (PENDING since 3 Oct)
- [ ] R13 Feather Falling and Luck art (PENDING since 3 Oct)
- [ ] Reply "defaults ok for M1" or name changes

**By milestone**
- [ ] M1: R2 padlock, R3 Index chip, R4 Storage chip, R5 tick, R6 LOCKED plate
- [ ] M2: R7 kit pills, royal chips, bar track and fill (A8 and A9 first)
- [ ] M3: R12 combat buttons F1 to F7 with words, charge ring A10
- [ ] M4: R14 Speed, Jump, Invisibility art (optional); R15 only if Claude asks
- [ ] M5: R16 Nuke card art (optional)
- [ ] M6 and M7: R17 approve screenshots, delete `_AssetViewer`, save, publish
- [ ] M8: R18 bat icon, R19 Bat label, R20 Swing label, R21 bat chip, R22 two swing animations
- [ ] M9: R23 Aqua Affinity art
- [ ] M11 and M12: R24 Pets button, R25 three Index tab icons
- [ ] M13: R26 OWNED button; R27 pickaxe animation only if needed
- [ ] M14 and M15: R29 weapon tiles, R30 flags, R31 reticle and hit marker, R32 Fire button; R33 weapon animations (optional); R28 only if Claude asks
- [ ] M17: R35 Storm Edge pass (optional), R36 and R37 (optional)

**Every milestone**
- [ ] Save to Roblox, then Publish (shut down old servers when the save changes; required for M1)
