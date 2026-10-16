---
tags: [ui, reference]
date: 2026-10-04
---
# UI kit inventory (the real components in GardenUI)

Read from the live GardenUI on 4 Oct 2026. **Every new screen must be built from these images**, not from code-drawn frames. That is the root cause of the King UI looking "off": the royal screens are flat `Frame` + `UICorner` + `UIStroke` boxes with gold text, while every Fruit Empire screen is image art.

![[ui-reference-index-menu.jpg]]
![[ui-reference-hud-left.png]]

## Type
- **Font:** `rbxasset://fonts/families/FredokaOne.json`. 239 of the 250 text objects use it (the other 11 are LegacyArial leftovers).
- **Text style:** white `TextColor3`, black `UIStroke` 2.5–3 px, `TextScaled = true`.
- **Panel titles are not text.** "INDEX" is an image (`TitleText`, heavy italic). Any new panel title needs its own title image made in the same style, or must reuse the blue title bar with Fredoka text.
- **HUD button labels are baked into the button art** (Inventory, Index, Shop, Rebirth, Merge All, Clear).

## Components and their images
| Role | Instance | Image id | Native size |
|---|---|---|---|
| Panel title bar (blue, glossy) | IndexMenu.TitleBlue | 95532041481396 | 938×63 |
| Panel title frame | IndexMenu.TitleFrame | 110138832714236 | 948×74 |
| Panel title text "INDEX" | IndexMenu.TitleText | 103221050515090 | 201×74 |
| Panel body (dark) | IndexMenu.Body | 82827190351667 | 940×460 |
| Panel inline frame | IndexMenu.Inline | 110403403749726 | 930×477 |
| Scroll area box | IndexMenu.ScrollBox | 83403190365135 | 688×391 |
| Info box (side panel) | IndexMenu.InfoBox | 101725900357985 | 192×225 |
| Rewards box | IndexMenu.RewardsBox | 126819257706583 | 192×160 |
| Green action button (Claim) | RewardsBox.ClaimButton | 132804991293392 | 128×28 |
| Chips (bonus / money / shards) | RewardsBox.*Chip | 89282702522748 / 82056408182031 / 78947574768646 | 47×50 |
| Card selection outline | Templates.Cell.Selection | 113285622864672 | — |
| Shop panel (whole) | ShopMenu.Panel | 93845982220486 | 846×529 |
| Shop tab (vertical) | ShopMenu.ToolsTab/KingTab/RobuxTab | 110917091182913 | 100×132 |
| Red X close (square) | ShopMenu.Panel.CloseButton | 107111219044453 | 59×59 |
| Red X close (rebirth) | RebirthMenu.CloseButton | 103021820090659 | 60×56 |
| Item card (tools/king) | ShopMenu …Card | 99120915117089 | 216×237 |
| Buy button | …Card.Buy | 97067256017679 | 173×61 |
| Upgrade panel | UpgradeMenu | 87274420074283 | 742×488 |
| Upgrade card | UpgradeMenu.Templates.UpgradeCard | 114196041045860 | 240×374 |
| Upgrade buy / maxed | …Buy / …Maxed | 112539925904292 / 76326716689640 | 179×47 |
| Storage panel | StoredFruitsMenu | 97577980253585 | 948×522 |
| Rebirth panel | RebirthMenu | 123255751134677 | 708×461 |
| Rebirth action button | RebirthMenu.RebirthAction | 86156862866253 | 241×85 |
| Rebirth locked overlay | RebirthMenu.LockedOverlay | 71043479073116 | 566×281 |
| Confirm panel | ClearConfirm.Panel | 112127226387884 | 565×373 |
| Yes / No buttons | ClearConfirm.Yes/NoButton | 71592791133553 / 105863640938118 | 203×68 |
| Popup panel + banner + close | OfflinePopup.Panel/Banner/Close | 117635448632940 / 103179819811131 / 129208837837255 | 391×317 |
| Top pill buttons | TopButtons.LockButton / UpgradeButton | 118392145629817 / 116083144480280 | 177×50 |
| Alert badge "!" | TopButtons.UpgradeButton.AlertBadge | 79539007098845 | 31×35 |
| Counter bars | CornerHud.CashBar / ShardsBar / BoostBar | 100015543922655 / 140489943167779 / 85577747844362 | 197×46 |
| Invite "+" | CornerHud.InvitePlus | 118611403979434 | 36×36 |
| Carry row | CarryButtons.Drop/Store/Use | 104911876480159 / 90968859583016 / 139324536295394 | 168×56 |
| Carry info bars | CarryInfo.NameBar / RateBar | 123470450245311 / 120297790302429 | 168×27 |
| HUD buttons | Inventory / Index / Shop / Rebirth / MergeAll / Clear / Mute | 108477171933167 / 118941334639047 / 71485100495903 / 115846161107162 / 99980479639840 / 88888506311530 / 123330597208905 | ~96×92 |

## Rarity frames (card art per tier) and colours
| Tier | Card frame | Colour (RGB 0–1) |
|---|---|---|
| COMMON | 131104257655900 | 0.59, 0.66, 0.74 (grey-blue) |
| UNCOMMON | 136126447232134 | 0.31, 0.76, 0.40 (green) |
| RARE | 99182487208693 | 0.31, 0.64, 0.90 (blue) |
| EPIC | 134546200735728 | 0.67, 0.36, 0.87 (purple) |
| LEGENDARY | 74773635384081 | 0.95, 0.67, 0.19 (orange-gold) |
| MYTHIC | 125641842226730 | 0.91, 0.26, 0.24 (red) |
| ASTRAL | 103897622776526 | 0.28, 0.87, 0.85 (cyan) |
| ETERNAL | 122828478750063 | 0.93, 0.32, 0.76 (pink) |
| TRANSCENDENT | 130274939131058 | 0.99, 0.97, 0.89 (pearl) |

These tier colours become the **bat colours** too (one bat per unlocked tier). See [[Bats]] and [[Rarity Gating and Rebirth]].

**Exact values.** The 0-1 numbers above are rounded. Every system reads the exact RGB from `Config.RarityColors` (`ReplicatedStorage.GardenConfig.lua:582-592`): Common 151,169,189; Uncommon 78,194,101; Rare 78,164,230; Epic 170,93,222; Legendary 242,171,48; Mythic 232,66,60; Astral 71,222,217; Eternal 237,81,195; Transcendent 252,247,227. Hex for art prompts: #97A9BD, #4EC265, #4EA4E6, #AA5DDE, #F2AB30, #E8423C, #47DED9, #ED51C3, #FCF7E3 ([[Integration Review]], conflict C1).

## Layout facts (the HUD the new UI must not collide with)
- **Left column (x=16):** Inventory (y center −181), Index (−78), Shop (+24).
- **Right column (x=−16):** Rebirth (−126), Merge All (−45), Clear (+45).
- **Top centre:** Lock + Upgrade pills (y 7, 416 wide). **Top right:** Mute.
- **Bottom left:** CornerHud (cash, shards, boost; 222×161).
- **Bottom right:** EffectStack (58 wide, grows upward).
- **Bottom centre:** CarryButtons (y −21) and CarryInfo (y −87).
- **Free zones** for new UI: the top-left corner under the safe area (the royal chip lives there now), top centre below the pills (y > 70), and the right edge between Clear and EffectStack.
- **After the expansion:** the full HUD layout, with every new cluster (Pets button, BatSlot, BatInfo, Weapon Bar, SeaStatus, the touch action slot, the re-laid effect block, the royal chip and event strip), is the table in [[Integration Review]] section 4. New UI must be added there.
