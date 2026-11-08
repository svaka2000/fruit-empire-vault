---
tags: [fruit-empire, ui, kit, reference]
status: current
updated: 2026-11-08
---
# UI kit and text rules

How every Fruit Empire screen is built, so it looks like the rest of the game. Words used here (9-slice, words pass, Larger text) are in [[Fruit Empire Glossary]]. Back to [[Knowledge Base - Start Here]].

> [!important] Read the source rules too
> The shared rules live in [ui/README.md](https://github.com/svaka2000/fruit-empire-handoffs/blob/main/ui/README.md), with reference screenshots. This note adds the kit's numbers and the lessons the chats learned. The older inventory of the kit's images is [[UI Kit Inventory]] (4 Oct).

## The kit

- **Where:** `ReplicatedStorage.UIKit` (the module) and `ReplicatedStorage.UIKitPrefabs`. Every new panel and HUD piece is built from it. It was made in M2 (4 Oct 2026). Source: [STATE.md](https://github.com/svaka2000/fruit-empire-handoffs/blob/main/STATE.md) "What was built up to 9 Oct".
- **Art, not boxes.** Every panel, card, button and bar is an image from Samarth's UI kit art. No visible Frame backgrounds, no `UICorner`, no `UIGradient` on boxes. (ui/README.md)
- **Helpers** (read from the 29 Oct export, `~/fruit-empire-pets/ref/scripts/ReplicatedStorage.UIKit.lua`): `Kit.text`, `Kit.textWidth`, `Kit.image`, `Kit.button`, `Kit.icon`, `Kit.title`, `Kit.pill`, `Kit.bar`, `Kit.panelFrame`, `Kit.clone`, `Kit.fit`, `Kit.openModal`. Lists: `Kit.Modals` (full panels; opening one closes the others) and `Kit.Dialogs` (popups).
- **Motion** (`Kit.Motion`): panels open in 0.22 s (Back, Out) from 0.85 scale, close in 0.14 s (Quad, In).

## Text

- **One font:** FredokaOne (`Kit.Font`), white, with a black `UIStroke` outline sized by `Kit.strokeFor(size)`. One exception waits for Samarth: text floating over fruit uses Highway (`Config.FruitTextFont`, question 45 in DECISIONS.md). (ui/README.md)
- **Four sizes only** (`Kit.Size`, at a 1600 x 900 screen): Title 30, Heading 24, Body 20, Small 17. Make labels with `Kit.text(parent, step, text)`. New UI never uses `TextScaled`.
- **Colours** (`Kit.Colors`): white for all text; red (226, 52, 52) for an unmet need or a timer under 5 s; green (86, 200, 96) for a met need; a rarity's own colour for that rarity's labels. Gold (255, 202, 58) is for bar fills, not text.
- **Plain ASCII only:** no emoji, no middle dot, em dash or multiplication sign. An icon whose image id is still 0 shows no icon, never an emoji.
- **Casing:** title images UPPER CASE, buttons and labels Title Case, sentences in sentence case and 48 characters at most.
- **Panel titles are images** (like INDEX). Until one exists, the title is Fredoka text on the blue title bar.

## The "Larger" text size and the words pass

> [!warning] Samarth's Studio is set to Larger text
> `GuiService.PreferredTextSize = Larger` adds about 10 screen pixels to every fixed-size label (17 becomes 27, 20 becomes 30). The extra does **not** shrink under a `UIScale`. So a panel that fits at normal text cuts names off for him.

- **In game scripts**, `TextService:GetTextSize` already includes the extra. In the Studio command bar or the MCP's `execute_luau` it does not. So a size measured from the console can be wrong in game. (memory `reference_roblox_text_size_setting.md`)
- **The fix, the "words pass":** measure every label's words with `Kit.textWidth(text, step, within)` and size the layout from that. It measures with a real hidden label under the same `UIScale`. The ShopMenu was the first panel built this way. (memory `project_fruit_empire_mine.md`, 22 Oct; UIKit export lines 432-458)
- To keep a designed size anyway: `TextSize = design - extra / scale` (at least 1). (memory `reference_roblox_text_size_setting.md`)
- A label's `TextBounds` measures the text *after* truncation, so a cut line looks like it fits. Measure the full string instead. A hidden label's `TextBounds` can be stale. (memory `reference_roblox_studio_mcp.md`, 5 Oct; `reference_roblox_ui_measuring.md`)

## 9-slice numbers

Insets are in the art's own pixels; `scale` is the default `SliceScale`. They were measured from the art's pixels on 5 Oct 2026 (UIKit export, `Kit.Slice`).

| Kit image | Inset (px) | Default scale | Note |
|---|---|---|---|
| PanelBody | 24 | 1 | corner 16 |
| PanelInline | 16 | 1 | corner 11 |
| TitleBar | 20, 24, 20, 30 | 1 | left, top, right, bottom: keeps the gloss and the shade |
| ScrollBox | 22 | 1 | corner 16 |
| InfoBox, RewardsBox | 28 | 1 | corner 22 |
| ItemCard | 48 | 0.25 | 4x redrawn art (10 Oct sharp-shop fix) |
| ShopTab | 64 | 0.25 | 4x art |
| NameBar, RateBar | 36 | 1 | corner 33 |
| PillGreen, PillBlue, PillRed, PillGold, PillGrey | 43 | 0.5 | `Kit.pill` |
| ChipGold, ChipStone | 30 | 0.5 | |
| BarTrack, BarFill | 26 | 0.5 | `Kit.bar` (art still a placeholder) |

- **Never guess an image's pixel size from a screenshot.** Read it with `AssetService:CreateEditableImageAsync(...)` and `.Size`. Wrong sizes make `SliceCenter` sample the wrong pixels: dark bands, see-through middles. (memory `reference_roblox_ui_measuring.md`)
- `Kit.image` shrinks `SliceScale` by itself when the corners would not fit the element.
- The Rebirth panel scales its sliced boxes with the panel: `SliceScale = base x panelWidth / 984`. (memory `project_fruit_empire_rebirth_panel.md`)

## Rarity frames and pills

- Rarity colours come from `Config.RarityColors` (table in [[Colour Palettes]]).
- Cards use rarity-coloured frames (the mine's Pickaxe Shop cards, the Index cards). Status pills use `Kit.pill` in green, blue, red, gold or grey. The Rebirth panel's need bars are gold while short and green once met, with white numbers. (memory `project_fruit_empire_mine.md`, `project_fruit_empire_rebirth_panel.md`)

## Screen sizes and overlap

- Every HUD piece is listed in `Config.HudClusters`, so `HudScale` shrinks it with the rest. A piece left out keeps its full size and collides. (ui/README.md)
- `StarterPlayer.StarterPlayerScripts.UIOverlapCheck` checks overlap. **Check five sizes:** 1600 x 900, 1280 x 720, a tablet, 812 x 375 and 667 x 375 (the last two are phones held sideways). (ui/README.md)
- Studio sizes can't be emulated from `execute_luau`. Work them out with `Config.HudScaleFor` or ask Samarth to use Test > Device. (memory `reference_roblox_studio_mcp.md`, 21 Oct)
- The strip just under the top buttons is the red Warn line's (`GardenUI.Toast`, at y 76). A HUD piece placed there gets drawn over. (memory `project_fruit_empire_mine.md`, egg glow, 22 Oct)

## Where HUD pieces go

Left: Inventory, Index, Shop. Right: Rebirth, Merge All, Clear. Top centre: Lock, Home, Upgrade, with the Sky Stars counter under them. Top right: Map button, Settings gear, Mute. Bottom left: money. Bottom centre: carry row and stamina bar. Bottom right: bat slot and potion timers. (ui/README.md; [[Map Panel]] for the Map button.)

> [!note] The top buttons
> Samarth wants all three top buttons in the old glossy look (`ui/08-top-buttons-wanted.png`), not the flat studded one (`ui/09-top-buttons-not-this.png`). Lock and Upgrade were restored on 18 Oct. Home still needs a new image in that style, made by Samarth. (STATE.md "Next up")

## New art

Missing art is made by Samarth: in Figma for titles and anything with words, with ChatGPT image generation for icons, each based on a named kit piece. The id is typed in after it passes Roblox moderation. Kit art can be pulled out of Studio read-only with `EditableImage` (HttpService stays off). (ui/README.md; memory `project_fruit_empire_rebirth_panel.md`)

Related: [[Samarth's Taste]], [[Claude Design Canvases]], [[Colour Palettes]].
