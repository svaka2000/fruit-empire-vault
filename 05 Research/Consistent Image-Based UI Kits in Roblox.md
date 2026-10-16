---
tags: [research]
topic: Consistent, non-overlapping image-based UI kits in Roblox (9-slice, component library, safe areas, scaling, fonts, z-order, AI-made UI art)
date: 2026-10-04
---
# Consistent Image-Based UI Kits in Roblox

Terms used in this note:
- **HUD** (heads-up display): the buttons and counters that stay on screen while you play.
- **Modal**: a panel that opens over the game, like the Shop or Index.
- **Toast**: a short pop-up message that fades away on its own.
- **9-slice**: one image cut into a 3×3 grid. The corners never stretch. The edges stretch one way. The middle stretches both ways. So one panel image can be any size without warping its border.
- **Safe area**: the part of the screen not covered by the phone notch or Roblox's own top-bar buttons.
- **AnchorPoint**: the point on a UI object (0 to 1 on each axis) that its Position and Size are measured from.
- **ZIndex / DisplayOrder**: numbers that decide which UI draws on top.
- **Token**: a named value (a colour, a size, a font) that many UI objects share, so you change it in one place.

## Summary
- Roblox has a built-in 9-slice mode. Set an ImageLabel's `ScaleType` to `Slice`, then set `SliceCenter` (or use Studio's 9-Slice Editor). Corners stay at source size, edges stretch, centre fills [1][2]. This is how one panel image becomes every panel size.
- Roblox now ships a real "design system" layer. **UI Styling** (StyleSheets, StyleRules, tokens, themes, state selectors like Hover/Press, and screen-size queries) reached full release on 20 Jan 2026 [10][11][12]. It is the engine-level way to stop every screen drifting to its own colours, sizes and fonts.
- Overlap is mostly a safe-area and layout problem. Interactive ScreenGuis should use `ScreenInsets = CoreUISafeInsets` (the default) [3][4]. Mobile reserves the bottom-left and bottom-right corners for the thumbstick and jump button [6]. Layout objects (`UIListLayout`, `UIGridLayout`) should place siblings instead of hand-placed positions [6][9].
- Two scaling schools exist. Roblox's docs push Scale sizes plus `UIAspectRatioConstraint` [7][19]. Many experienced devs design in Offset (pixels) at one reference resolution and multiply everything with a single `UIScale` [25][26]. Either works if the whole kit uses one school.
- One font family is the norm [24][31]. Fruit Empire already uses Fredoka One on 239 of 250 text objects ([[UI Kit Inventory]]). Fredoka One and Luckiest Guy are both built-in Roblox font families [13].
- `TextScaled = true` makes labels of different lengths render at different sizes [27]. It also ignores the player's Text Size accessibility setting [21]. That is a hidden consistency bug in the current kit. A search of the 2026-10-04 export finds `TextScaled = true` 30 times in 14 scripts, most in TopEarnersBoard (6), StoredFruitsMenu (6) and ShopMenu (4) [42]. Labels built in Studio and saved in the place file are not in the scripts, so the true count is likely higher (not confirmed).
- Use `ZIndexBehavior = Sibling` (ZIndex only orders siblings) and give each ScreenGui layer its own `DisplayOrder` band [3][14][15].
- AI image tools can match an existing kit if you feed real component images as references and give each reference a job [33][35]. They still drift between outputs and are weak at exact text and layout [34][37]. Gemini did not produce true transparency for one developer, so every asset needs a background-removal and edge-cleanup pass [36][23].

## How it works

### 1. 9-slice panels and buttons
- One image is split into 9 parts. Corners (1, 3, 7, 9): no scaling. Top and bottom edges (2, 8): stretch horizontally. Left and right edges (4, 6): stretch vertically. Centre (5): stretches both ways [1].
- To use it: pick an `ImageLabel` or `ImageButton` with an image, set `ScaleType = Slice`, click `SliceCenter`, then the "…" button. The editor shows 4 red lines and the source image size. Drag the lines or type exact offsets [1].
- Watch out: the numbers typed in the editor are **pixels from each edge**. They are not the same as the `SliceCenter` property values [1].
- `SliceScale` (default 1.0) grows the edges "as if you'd uploaded a new version of the texture upscaled" [2]. This lets one image give thin borders on phone and thicker ones on TV without new art.
- Roblox's docs say the main reason to use it is UI that must fit text of unknown length, such as translated text [1].
- `ImageButton` has `Image`, `HoverImage` and `PressedImage` slots, so a kit button can ship normal, hover and pressed art [40].

### 2. A fixed component library
What Roblox's own design guide says:
- Good UI follows five principles: prioritization, attention, visual language, conventions, consistency [16].
- Its consistency examples: "Health" is always green; close buttons are always square, red, white X, top right only; unaffordable prices always show red [16].
- Write the visual language down in a **style guide** so the whole team applies it the same way [16].
- Real examples it cites: Jailbreak's season pass puts a gold stripe under premium rewards; Dragon Adventures makes the best coin bundle bigger with more padding; Tower Defense Simulator groups daily skins and daily crates by size and proximity [16].
- Conventions players already know: an X closes a dialog, grey means unusable, a lock icon means not unlocked yet [16].

How Roblox's engine now supports a library (UI Styling) [10][11][12]:
- **Tokens** are attributes on a token StyleSheet, for example one colour shared by `BackgroundColor3`, `TextColor3` and `UIStroke.Color` [10].
- **Themes** are swappable sets of the same tokens [10].
- **Rules** match UI by class (`Frame`), by CollectionService tag (`.ButtonPrimary`), by name, by modifier (`::UICorner`), or by state. The states are the four `GuiState` values: Idle, Hover, Press, NonInteractable [10].
- **Queries** apply styles only on certain screens. The Style Editor has built-in `@ViewportDisplaySizeSmall / Medium / Large` rules. Small means most phones and tablets; Large means most TVs [11].
- A `StyleLink` connects one StyleSheet to a ScreenGui and everything inside it. Only one StyleSheet applies per tree [10].
- Styled properties show a ⚠ in the Properties window. A bold value means that one object overrides the style [10].
- Known gap at full release: styling multiple pseudo-instances, "including UIStrokes", was "soon" [12].

What image-heavy simulator UIs do:
- A DevForum thread on Pet Simulator 99-style buttons says `UIStroke` borders get cluttered on small screens. Posters recommend a frame-behind-a-frame instead: inner frame `AnchorPoint (0.5, 0.5)`, size about 0.9 scale of the outer frame [29].
- A DevForum reply suggests PS99-style auto-fitting panels use `AutomaticSize` + `UIListLayout` + `UIFlexItem` [30]. Not confirmed that PS99 itself does this.
- A simulator-UI art tutorial (Photoshop side, not Roblox properties) uses an outside stroke of 6 to 15 px and about 50 px corner rounding, scaled to canvas size [24]. It also cleans exported PNG edges with PixelFix [24].

### 3. Safe areas and responsive layout
Safe-area settings on each `ScreenGui` (`ScreenInsets`) [3][4]:

| Setting | What it keeps clear | Use for |
|---|---|---|
| `CoreUISafeInsets` (default) | Roblox top-bar buttons and notches | Anything you can tap |
| `DeviceSafeInsets` | Notches only, not the top bar | Rarely |
| `TopbarSafeInsets` | The strip between the top-bar buttons and the right edge; flexes with top-bar content | A top strip of small chips |
| `None` | Nothing | Background art only |

- `ClipToDeviceSafeArea` defaults to `true`, so UI pushed off-screen stays hidden [4].
- `SafeAreaCompatibility` defaults to `FullscreenExtension`, but Roblox says to avoid it for new work and use `ScreenInsets` instead [4].
- `GuiService.TopbarInset` gives the free rectangle beside Roblox's top-left buttons. It changes at runtime, so listen with `GetPropertyChangedSignal` [5].
- `GuiService:GetInsetArea(Enum.ScreenInsets.None)` on one phone returned `-59, -58, 792, 334`, relative to the core-UI safe area [5].

Mobile reach [6]:
- Default touch controls fill the bottom-left and bottom-right corners. Keep important info and buttons out of them.
- A button 40% down from the top is reachable on a phone but "almost unreachable" on a tablet.
- Roblox's own example places a custom button 20 px left of the jump button, at 0.8× its size, by reading the jump button's position at runtime.
- Show buttons only when they matter (context-based UI), for example a proximity prompt instead of a permanent "open" button.
- Roblox's art-style lesson cites the WCAG touch target of at least 9×9 mm [17].
- Roblox's adaptive design guide: on small screens, move less-important buttons into a collapsible menu; let text boxes grow with their content; tie UI size to the player's accessibility settings; avoid thin or overly decorative fonts [20].

Layout tools [6][7][9]:
- `AnchorPoint` default is `(0, 0)` (top-left). `(0.5, 0.5)` makes the object grow out from its centre [6].
- `UIListLayout.Padding` is space **between** items. `UIPadding` is space **around** the content [9].
- Flex (`HorizontalFlex`) fills a tab bar equally no matter how many tabs. Flex has "a slight performance cost," so do not add it without reason [9].
- `ItemLineAlignment = Stretch` makes uneven tiles fill their row [9].
- `UISizeConstraint` and `UIAspectRatioConstraint` **override** a layout's sizing when both apply [7].
- Bug history: `AutomaticSize` on a parent plus Scale-sized children is a circular dependency (parent size depends on children; children depend on parent). Roblox staff confirmed it in April 2025. One commenter said it looked fixed by mid-June 2025. Workarounds were offset sizing or filler frames [28].

### 4. Scaling: Scale + aspect ratio vs Offset + UIScale
- `UIScale.Scale` multiplies the parent's `AbsoluteSize`. Scale 0.5 turns 200×50 into 100×25 [2]. It also scales child `UIStroke` and `UICorner` [7].
- Roblox's UI curriculum puts a `UIAspectRatioConstraint` on most pieces (examples: `AspectRatio` 7, 13.78, 0.895) so they keep shape on any screen [19].
- "Scaler" (DevForum, March 2021) designs at one resolution (example 720 px tall) and drives tagged `UIScale`s from the viewport size. Its stated benefit: you can size everything in Offset [25].
- The `usePx` hook (roblox-ts React) uses a base of 1280×832, a 50/50 blend of width and height ratios on a log scale, and a minimum scale of 0.75 [26].
- Test with Studio's Device Emulator at **Actual Resolution**, with at least one Phone, Tablet, Desktop and Console device [19].

### 5. Fonts, strokes and text size
- `Font` = family + weight + style. `Font.fromName("FredokaOne")` maps to `rbxasset://fonts/families/FredokaOne.json`. Luckiest Guy is `rbxasset://fonts/families/LuckiestGuy.json` [13].
- Community guidance: one font family across the whole UI [31]. Roblox's guidance: stylized fonts sparingly (titles, alerts); text on a contrasting colour or with a stroke [17].
- `UIStroke` on a text object outlines the letters. On any other object it draws a border. `ApplyStrokeMode` can switch a text object to a border, and two strokes can do both [8].
- `StrokeSizingMode = ScaledSize` makes a text stroke's thickness relative to font size [8].
- Do not tween `UIStroke.Thickness` on text. It renders many glyph sizes per frame and can flicker [8].
- `UIShadow` draws a box shadow, not a text shadow [8]. RDC 2026 announced "blur, text shadows, and improved gradients" [39]. Release status in Studio: not confirmed.
- `UITextSizeConstraint.MinTextSize` should never be below 9 [7].
- `TextScaled` fits each label to its own box, so "10 robux" and "10000 robux" end up different sizes [27]. `TextScaled` text also ignores `GuiService.PreferredTextSize` (the player's Text Size setting: Medium, Large, Larger, Largest) [21][5].

### 6. Z-order rules
- `ZIndexBehavior`: **Global** sorts all objects by ZIndex first. **Sibling** sorts by hierarchy first, then ZIndex among siblings only [14]. Sibling arrived on 11 Jan 2018, pitched as "more intuitive," and you no longer have to copy ZIndex down to every child [15].
- Default `ScreenGui` behaviour renders children above parents [6].
- `ScreenGui.DisplayOrder`: higher draws on top of lower. Roblox's example is a settings modal on a higher DisplayOrder than the main HUD [3][4].
- Multiple ScreenGuis can be toggled with `Enabled`. When `false`, they don't render, take input or update [3].
- `UIStroke` has its own `ZIndex`. Two strokes with the same ZIndex have undefined order [8].

### 7. Image pipeline limits
- A DevForum user (Jan 2024) states the max upload without downscaling is 1024×1024 [22]. Staff confirmation: not found.
- Roblox's `ScaleType` doc says transparent pixels are set to black on upload, which can give a dark outline around scaled images [2].
- The fix is **alpha bleed**: copy the nearby edge colours into the fully transparent pixels so texture filtering has good colour to sample. One 2026 tool applies a 4 px bleed by default [23]. PixelFix is the older tool for the same job [24].
- `ResampleMode = Pixelated` keeps hard pixel edges. The default smooths [2].
- Roblox says an image scales to fit its label but "looks best when it displays at its native resolution" [41].

### 8. Making matching UI art with AI tools
What the tools can do (vendor docs, fetched 3 Oct 2026):

| Tool | Reference images | Transparency | Other limits |
|---|---|---|---|
| Gemini 3.1 Flash Image | up to 10 object images, 4 character images [32] | not stated in docs; one dev got no true alpha [36] | ratios 1:1, 3:2, 2:3, 3:4, 4:3, 4:5, 5:4, 9:16, 16:9, 21:9; SynthID watermark [32] |
| Gemini 3 Pro Image | up to 6 objects, 5 characters [32] | same | — |
| OpenAI image API | multiple input images; max not stated [34] | `background: "transparent"` with PNG or WebP [34] | sizes 1024×1024, 1536×1024, 1024×1536; custom sides multiples of 16, ratio 1:3 to 3:1, max 3840 px; admits weak text placement, consistency and layout control [34] |

How teams keep outputs consistent:
- **Give each reference a job.** Google's own tip: "Use Image A for the character's pose, Image B for the art style…" [33].
- **Put the most important reference first.** With OpenAI's `input_fidelity="high"`, all inputs are kept with high fidelity, "only the first one" gets "extra richness in texture" [35].
- **Do not trust one prompt to stay on-model.** A vendor blog (it sells its own tool) lists drift: colours shift, outlines change thickness, sizes change between generations [37].
- **Plan for background removal.** One developer's fix for Gemini: generate on a flat green background and ask for a 2 to 3 px white outline around each object, so anti-aliasing blends into white, not green. Then key out green by HSV (hue ±22°, saturation ≥ 0.3, value ≥ 0.3) [36].
- **Presets beat free generation.** Bloxsmith, an AI UI tool for Roblox, keeps style by shipping fixed presets (colours, type, spacing, strokes, gradients, plus a library of matching images) and only experiments with new image generation. Users still reported misaligned output and cut-off code [38].

## What Fruit Empire should take

Fruit Empire already has a real image kit ([[UI Kit Inventory]]). The King UI looked "off" because it was built from flat `Frame` + `UICorner` + `UIStroke` boxes instead of kit images. Everything below is about making every new screen (bats, explosives, mine, pets, boats, swimming, rebirth gates, duels, weapon skins) come out of that same kit.

### A. Turn the kit into a library (one-time setup)
1. **Make every kit panel 9-sliced.** Open Body (940×460), Inline, ScrollBox, InfoBox, RewardsBox, Confirm panel, Popup panel and the item card in the 9-Slice Editor. Record each image's 4 edge insets in [[UI Kit Inventory]]. Then a 600×300 "Boat purchase" panel and a 940×460 "Mine Index" panel use the same art [1].
2. **Make one `UIKit` folder of prefabs** in ReplicatedStorage: `Panel`, `TitleBar` (blue bar + Fredoka text), `CloseX`, `TabVertical`, `Card` (per rarity frame), `BuyButton`, `ActionButtonGreen`, `Chip`, `CounterBar`, `AlertBadge`, `Toast`. New screens clone prefabs. Nobody draws a `Frame` from scratch.
3. **Move shared values into UI Styling tokens** [10][11]: the 9 rarity colours (Common grey-blue … Transcendent pearl), white text, black stroke, the 2.5 to 3 px stroke width, the Fredoka font, and the text-size steps. Tag prefabs (`.KitTitle`, `.KitBody`, `.KitPrice`) so rules style them. Note: multi-stroke styling was not supported at release [12], so keep strokes on the prefab itself.
4. **Finish the font job.** Swap the last 11 LegacyArial labels to Fredoka One [13][31].
5. **Fix text sizing.** Replace `TextScaled = true` with a small fixed size ladder (proposal: Title / Body / Small / Price) under one root `UIScale`. Reasons: equal labels stop rendering at different sizes [27], and the player's Text Size setting starts working [21]. Keep `UITextSizeConstraint.MinTextSize` ≥ 9 [7]. Set text strokes to `StrokeSizingMode = ScaledSize` [8].
6. **Pick one scaling school per layer, and match what the game already does.** The local code already splits it this way [42]:
   - **Panels** (Index, Upgrade, Inventory) are sized in **Scale**.
   - **HUD clusters** are pixel art sized in **Offset**. `HudScale.lua` is the only script that writes their positions. It puts a `UIScale` on container frames, and writes `Size` directly on clusters that are themselves buttons.
   - **Gotcha:** `ButtonAnimator` animates each button's own `UIScale` for hover. Roblox honours only one `UIScale` per object, so a second `UIScale` on a button would fight it [42].
   - So new modals should follow the panels (Scale + `UIAspectRatioConstraint` [19]), and new HUD pieces should follow the HUD (Offset + `HudScale`) [25][26]. Drift comes from mixing schools inside one layer.
7. **Clean every new image before upload.** Alpha-bleed (about 4 px) [23], keep it ≤ 1024×1024 [22], and upload at or above the size it will display.

### B. Layering plan (proposal)
| DisplayOrder band | ScreenGuis |
|---|---|
| 0 | HUD (left column, right column, CornerHud, top pills, EffectStack) |
| 10 | Context UI (carry row, mining prompt, pet "wants a fruit" bubble, boat seat prompt, swim fatigue meter, duel health bars) |
| 20 | Modals (Shop, Index, Rebirth, Bats, Mine, Pets, Boat, Skins). Only one open at a time. |
| 30 | Confirm dialogs (Yes/No) |
| 40 | Toasts and reward pops |
| 50 | Tutorial arrows |

Set every ScreenGui to `ZIndexBehavior = Sibling` [14][15]. Inside a screen, use small ZIndex steps only between siblings.

### C. Where new things go (uses the inventory's free zones)
- **Swim fatigue meter, Aqua Affinity timer:** top centre below the pills (y > 70), or into EffectStack. Never bottom corners: those are the mobile thumbstick and jump zones [6].
- **Duel health bars:** top centre below the pills. In a duel, hide the merge HUD columns (context-based UI) [6][16].
- **Fruit health bars (explosives drain health):** in-world BillboardGuis, not the ScreenGui. Use one 9-sliced bar image and colour the fill by fruit rarity token.
- **Pet tame request:** a BillboardGui speech bubble over the pet showing the requested fruit icon. No HUD space used.
- **New modals:** centred, `AnchorPoint (0.5, 0.5)`, built from Panel + TitleBar + CloseX (top right, always) [16].
- **Rebirth gate checklist (full index + shards + money):** reuse RebirthMenu + LockedOverlay. Grey for unmet, green tick for met, a lock icon on the button. These are conventions players know [16].
- **Bats, TNT tiers, nukes, weapon skins:** reuse the Shop item card (216×237) + Buy button (173×61). Card frame = rarity frame. Unaffordable price in red, every time [16].
- **No steal UI anywhere.** Stealing is removed. Duels are opt-in in the sky arena only.

### D. The "new screen" checklist
Follow this for every new screen before it ships:
1. Wireframe it with grey boxes **next to Roblox's core UI** on a phone, tablet, desktop and console in Device Emulator at Actual Resolution [18][19].
2. List which kit prefabs it uses. If a piece is not in the kit, make the art first (section E). Do not draw it with Frames.
3. Put it in the right ScreenGui band (table B). `ScreenInsets = CoreUISafeInsets` if anything is tappable [3].
4. Anchor the root to the corner or centre it hugs. Position = that corner in Scale plus a fixed margin.
5. Let layouts place children. `UIListLayout` / `UIGridLayout` for rows and grids, `UIPadding` for inner margins [9]. No hand-placed siblings.
6. Do not mix `AutomaticSize` parents with Scale-sized children [28]. Remember aspect and size constraints override layouts [7].
7. Text: Fredoka One, white, black stroke, a ladder size (not `TextScaled`). Titles: the blue title bar, or a title image made in the same style.
8. Button states: Hover and Press art (or a style rule on `GuiState`) [10]. A press scale tween on a `UIScale`, never on stroke thickness [8].
9. Tap targets at least about 9×9 mm [17]. Frequent buttons inside thumb reach, not 40% down on a tablet [6].
10. Run an overlap check. Proposal: a dev-only script that walks every visible GuiObject in the HUD and Context ScreenGuis, compares `AbsolutePosition` + `AbsoluteSize` rectangles, and prints any two that intersect that are not parent and child. Run it on each emulated device.
11. Test with the player Text Size set to Largest [21].
12. Add the screen and its new images to [[UI Kit Inventory]].

### E. Making new kit art with ChatGPT or Gemini
1. Pick 2 to 4 real kit images as references. Good ones in the vault: `ui-reference-hud-icons.png`, `ui-reference-potion-icons.png`, `ui-reference-index-menu.jpg`, plus one exported button PNG.
2. Name each reference's job in the prompt: "Image 1 = exact outline thickness, gloss and bevel. Image 2 = colour palette. Image 3 = icon shape language." [33] Put the closest match first [35].
3. Ask for **one asset per generation**, front-on, flat, centred, with padding. State the native size you will display it at.
4. Background: on OpenAI ask for a transparent PNG [34]. On Gemini ask for flat green plus a 2 to 3 px white outline, then key it out [36].
5. Keep text out of AI art. The tools are weak at exact text [34]. Add labels in an image editor with Fredoka One to match the baked-in HUD labels, or use a Studio TextLabel.
6. Post-process: remove background, alpha-bleed, resize to ≤ 1024 [22][23]. Put the new asset beside the old ones on one sheet and compare outline thickness, highlight direction and saturation before uploading.
7. Keep a "style card" image (the best 6 kit pieces on one sheet) and attach it to every future prompt, so references never change.
8. Style anchors for this game: retro 2006 to 2008 Roblox, studded plastic surfaces, glossy cartoon icons, thick dark outline. Ores: higher tiers glow, so ask for an outer glow only on Rare+ ore icons.

## What to avoid
- Flat `Frame` + `UICorner` + `UIStroke` "placeholder" screens shipped as final. That is what made the King UI look foreign.
- `TextScaled` on every label [27][21].
- Global ZIndex and climbing ZIndex numbers (ZIndex 999 wars) [15].
- Putting buttons or meters in the bottom-left or bottom-right corners on mobile [6].
- `ScreenInsets = None` on anything tappable [3].
- Mixing Scale-based and Offset+UIScale-based pieces inside one layer (panels are Scale, the HUD is Offset + `HudScale`).
- A second `UIScale` on a button that `ButtonAnimator` already animates [42].
- `AutomaticSize` parents with Scale-sized children [28].
- Tweening stroke thickness on text [8].
- Uploading AI PNGs straight from the generator. You get halos and drift [23][36][37].
- Letting the AI write in-image text or lay out whole screens. Composition and text control are its weak points [34][38].
- Too many accent colours. Roblox advises limiting the colour theme and not relying on colour alone (pair it with icons or shapes) [17].

## Open questions
- What are the exact 9-slice insets for each kit image? This needs a pass in the 9-Slice Editor.
- ~~Do the current screens use Scale or Offset sizing?~~ Answered from the code: panels use Scale, the HUD uses Offset with `HudScale` [42]. See A6.
- Were the kit images uploaded at or above their display size on tablets and TVs?
- Can UI Styling tokens drive the `Image` property (for example swap a rarity frame by token)? Not confirmed.
- Are RDC 2026 text shadows live in Studio yet [39]? If so, they might replace some text strokes.
- What share of Fruit Empire players are on mobile? Check Creator Dashboard analytics by device. Not confirmed here.
- Does Pet Simulator 99 really build panels with AutomaticSize + UIListLayout? Only a community guess [30].
- Roblox's moderation stance on AI-made UI images: not confirmed.

## Sources
1. https://create.roblox.com/docs/ui/9-slice
2. https://create.roblox.com/docs/reference/engine/classes/ImageLabel (SliceCenter, SliceScale, ScaleType, ResampleMode) and https://create.roblox.com/docs/reference/engine/classes/UIScale
3. https://create.roblox.com/docs/ui/on-screen-containers
4. https://create.roblox.com/docs/reference/engine/classes/ScreenGui
5. https://create.roblox.com/docs/reference/engine/classes/GuiService
6. https://create.roblox.com/docs/ui/position-and-size
7. https://create.roblox.com/docs/ui/size-modifiers
8. https://create.roblox.com/docs/ui/appearance-modifiers
9. https://create.roblox.com/docs/ui/list-flex-layouts
10. https://create.roblox.com/docs/ui/styling
11. https://create.roblox.com/docs/ui/styling/editor
12. https://devforum.roblox.com/t/full-release-ui-styling-is-officially-released/4275082
13. https://create.roblox.com/docs/reference/engine/datatypes/Font
14. https://create.roblox.com/docs/reference/engine/enums/ZIndexBehavior
15. https://devforum.roblox.com/t/new-zindexbehavior-property-is-now-live/76051
16. https://create.roblox.com/docs/production/game-design/ui-ux-design
17. https://create.roblox.com/docs/tutorials/curriculums/user-interface-design/choose-an-art-style
18. https://create.roblox.com/docs/tutorials/curriculums/user-interface-design/wireframe-your-layouts
19. https://create.roblox.com/docs/tutorials/curriculums/user-interface-design/implement-designs-in-studio
20. https://create.roblox.com/docs/production/publishing/adaptive-design
21. https://create.roblox.com/docs/production/publishing/accessibility
22. https://devforum.roblox.com/t/warn-user-about-resolution-limits-before-uploading-an-image/2779568
23. https://devforum.roblox.com/t/automated-pixelfix-for-ui-run-and-forget/4812286
24. https://devforum.roblox.com/t/how-to-make-ui-styled-for-simulator-detailed-tutorial/2895762
25. https://devforum.roblox.com/t/scaler-using-uiscale-to-scale-your-ui/1105672
26. https://gist.github.com/littensy/519d6b88a8aa9c9aace4da947a5fb1d8
27. https://devforum.roblox.com/t/making-text-size-consistent/2700490
28. https://devforum.roblox.com/t/automaticsize-with-uilistlayouts-is-completely-broken/3630449
29. https://devforum.roblox.com/t/make-buttons-like-pet-simulator/3984742
30. https://devforum.roblox.com/t/how-would-i-go-about-making-a-gui-rescaling-thing-like-in-pet-simulator-99/3244721
31. https://devforum.roblox.com/t/ui-design-guide/2975811
32. https://ai.google.dev/gemini-api/docs/image-generation
33. https://blog.google/products-and-platforms/products/gemini/prompting-tips-nano-banana-pro/
34. https://developers.openai.com/api/docs/guides/image-generation
35. https://developers.openai.com/cookbook/examples/generate_images_with_high_input_fidelity
36. https://roboticape.com/2026/03/07/generating-game-sprites-with-gemini-image-generation-nano-banana-pro-lessons-learned/
37. https://gamelabstudio.co/blog/why-ai-cant-make-game-ready-spritesheets (vendor blog)
38. https://devforum.roblox.com/t/bloxsmith-%E2%80%94-ai-ui-generation-with-real-style-presets-assets/4526615
39. https://about.roblox.com/newsroom/2026/09/rdc-2026-the-world-needs-more-play
40. https://create.roblox.com/docs/ui/buttons
41. https://create.roblox.com/docs/ui/labels
42. Local code: `~/fruit-empire/src-export-2026-10-04/StarterPlayer.StarterPlayerScripts.HudScale.lua` (header comment on Scale panels vs Offset HUD, and the one-`UIScale`-per-object rule) and a search of the export for `TextScaled = true`

## Review (critic pass, 2026-10-04)

- **Answered:** the Scale-or-Offset question, from `HudScale.lua` [42]. A6 and the "avoid" list now say "one school per layer" instead of "one school for the whole game", because the game already splits them on purpose.
- **Added:** where `TextScaled = true` is used today [42].
- **Re-checked, still not confirmed:** the 1024×1024 upload cap. The Roblox docs pages on assets, labels and textures do not state it. Only a DevForum user does [22].
- **Still not confirmed:** whether RDC 2026 text shadows are live in Studio, whether UI Styling tokens can drive `Image`, the exact 9-slice insets, the mobile share of players, and Roblox's stance on AI-made UI images.
