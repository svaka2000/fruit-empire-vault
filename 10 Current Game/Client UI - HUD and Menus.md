---
tags: [current-game, code-map]
date: 2026-10-04
sources:
  - StarterPlayer.StarterPlayerScripts.GardenClient.lua
  - StarterPlayer.StarterPlayerScripts.ShopMenu.lua
  - StarterPlayer.StarterPlayerScripts.IndexMenu.lua
  - StarterPlayer.StarterPlayerScripts.StoredFruitsMenu.lua
  - StarterPlayer.StarterPlayerScripts.UpgradeMenu.lua
  - StarterPlayer.StarterPlayerScripts.RebirthMenu.lua
  - StarterPlayer.StarterPlayerScripts.CashClient.lua
  - StarterPlayer.StarterPlayerScripts.HudScale.lua
  - StarterPlayer.StarterPlayerScripts.ButtonAnimator.lua
  - StarterPlayer.StarterPlayerScripts.AlertBadges.lua
  - StarterPlayer.StarterPlayerScripts.ClearMenu.lua
  - StarterPlayer.StarterPlayerScripts.LockButton.lua
  - StarterPlayer.StarterPlayerScripts.MergeAllButton.lua
  - StarterPlayer.StarterPlayerScripts.MuteButton.lua
  - StarterPlayer.StarterPlayerScripts.OfflinePopup.lua
  - StarterPlayer.StarterPlayerScripts.EarnPopups.lua
  - StarterPlayer.StarterPlayerScripts.TopEarnersBoard.lua
  - StarterPlayer.StarterPlayerScripts.PenUpgradeSign.lua
  - ReplicatedStorage.GardenConfig.lua (tuning values, read in parts)
  - ServerScriptService.GardenStore.lua (save schema, Store.Default only)
  - ReplicatedStorage.GardenToast.lua (searched for font only)
  - ServerScriptService.GardenService.Progression.lua (searched for attribute writers only)
  - ServerScriptService.GardenService.Persistence.lua (searched for attribute writers only)
  - ServerScriptService.GardenService.Merging.lua (searched for attribute writers only)
  - ServerScriptService.GardenService.RebirthLock.lua (searched for attribute writers only)
  - ServerScriptService.GardenService.Ambient.lua (searched for attribute writers only)
  - ServerScriptService.GardenService.Carrying.lua (searched for attribute writers only)
  - ServerScriptService.GardenService.Tools.lua (searched for attribute writers only)
  - ServerScriptService.GardenService.lua (searched for attribute writers only)
---
# Client UI - HUD and Menus

Related: [[UI Kit Inventory]], [[Brain Dump 2026-10-04]], [[World Tour]].

Terms used below:
- **HUD** (heads-up display): the buttons and counters that are always on screen.
- **GardenUI**: the one ScreenGui (a screen-space GUI container) in StarterGui that holds the HUD and every menu. Every script here waits for it, for example `GardenClient.lua:68`.
- **Attribute**: a named value stored on a Roblox instance. When the server sets one on a Player, it copies (replicates) to that player's client.
- **RemoteEvent**: a Roblox object that sends a one-way message between server and client. All remotes here live in `ReplicatedStorage.GardenRemotes`.
- **Config**: the shared module `ReplicatedStorage.GardenConfig`. Both server and client read it.
- **ViewportFrame**: a GUI element that renders a small 3D scene, used here for fruit and tool previews.
- **UIScale / UIStroke**: a UIScale multiplies the size of a GUI subtree; a UIStroke draws an outline around text or a frame.
- **Tween**: a smooth animation of a property from one value to another (TweenService).
- **ProximityPrompt**: the "press E" prompt Roblox shows near a part.
- **Kit**: Samarth's image-based UI kit, listed in [[UI Kit Inventory]].

## 1. What it does

These 18 LocalScripts draw everything the player sees on screen: the HUD counters and buttons, the carry row, five full-screen panels (Shop, Index, Inventory, Upgrades, Rebirth), three modal dialogs, and the buff timers. Three of them also draw UI in the 3D world: floating "+$" numbers, the Top Earners board, and the pen upgrade sign. None of them decide anything. The server owns every rule; the client reads replicated attributes and value folders and only asks the server through RemoteEvents (stated in `UpgradeMenu.lua:4-7`, `RebirthMenu.lua:8-11`, `IndexMenu.lua:4-7`). Most panel shells are pre-made kit images in StarterGui, and the scripts fill them with cards built in code.

## 2. Scripts

| Script | Role | Lines |
|---|---|---|
| GardenClient.lua | Fruit prompts and highlights, carry row (Drop/Use/Store), carry info bars, storage badge, toast relay, buff effect stack | 610 |
| ShopMenu.lua | Shop panel: tab switching, tool cards, King aura cards | 648 |
| IndexMenu.lua | Index panel: 64 fruit cells, info box, Claim button | 638 |
| StoredFruitsMenu.lua | Inventory panel: stored fruit and tool cards with Equip overlay, storage count | 576 |
| UpgradeMenu.lua | Upgrades panel: capacity, spawn rate, spawn rarity cards | 458 |
| RebirthMenu.lua | Rebirth panel, locked overlay, rebirth confirm dialog | 256 |
| CashClient.lua | Bottom-left counters: cash, shards, friend boost, cash multiplier, invite button | 137 |
| HudScale.lua | Shrinks the pixel-sized HUD clusters on small screens; sole writer of their Position | 188 |
| ButtonAnimator.lua | Hover/press/release animation for every button; "magnet" lean on side buttons | 217 |
| AlertBadges.lua | Red "!" badges on Upgrade, Index and Rebirth buttons | 137 |
| ClearMenu.lua | Clear (broom) button and its confirm dialog | 201 |
| LockButton.lua | Plot lock button and its countdown label | 89 |
| MergeAllButton.lua | Merge All gamepass button | 59 |
| MuteButton.lua | Music mute toggle | 38 |
| OfflinePopup.lua | "Welcome back" offline earnings dialog | 124 |
| EarnPopups.lua | Floating "+N$" billboards over your own fruit on each income tick | 108 |
| TopEarnersBoard.lua | Plaza leaderboard built as a SurfaceGui on the board mesh | 239 |
| PenUpgradeSign.lua | Owner-only "Upgrade Pen" sign that sells plot size | 313 |

## 3. How it works

### 3.1 Startup order
1. Each script waits for `PlayerGui.GardenUI` and the named children it needs (`GardenClient.lua:68-78`, `ShopMenu.lua:38-42`).
2. HudScale registers each name in `Config.HudClusters`, waiting up to 5 s per cluster (`HudScale.lua:64`). It records the authored Position and Size once (`HudScale.lua:70-75`), then applies a scale factor from `Config.HudScaleFor` (`HudScale.lua:137-160`). It polls for up to 10 s until the viewport reports a real size (`HudScale.lua:181-188`).
3. ButtonAnimator adopts every existing and future GuiButton under GardenUI (`ButtonAnimator.lua:207-217`).
4. Menus build their cards at load, not on first open: IndexMenu builds all cells (`IndexMenu.lua:637`), UpgradeMenu builds its cards (`UpgradeMenu.lua:455`), StoredFruitsMenu rebuilds (`StoredFruitsMenu.lua:516`), ShopMenu builds tool cards (`ShopMenu.lua:378-388`) and the King page (`ShopMenu.lua:400-594`).
5. When the save loads, the server sets `ProfileLoaded` (`GardenStore.lua:800`). CashClient snaps its counters on that frame instead of rolling up (`CashClient.lua:73-77`). OfflinePopup waits for it with no timeout (`OfflinePopup.lua:109-111`). IndexMenu waits for the `IndexState` folder with no timeout (`IndexMenu.lua:433-442`).

### 3.2 HUD layer
- **Scaling.** `Config.HudScaleFor(viewport, touch)` takes the tighter of width/1600 and height/900, raises it to the power 0.6, multiplies by 0.85 on touch-only devices, and clamps to 0.55..1.0 (`GardenConfig.lua:1949-1957`). A Frame cluster gets a UIScale; a cluster that is itself a button gets its Size written directly, so the UIScale slot stays free for ButtonAnimator (`HudScale.lua:77-94`, `HudScale.lua:152-156`).
- **Position ownership.** `place(entry)` is the only Position write for HUD clusters. It folds ButtonAnimator's `MagnetOffset` attribute into the Position as a fraction of the parent size (`HudScale.lua:116-127`).
- **Button feel.** `animate(button)` tweens the button's UIScale to 1.08 on hover, 0.92 on press, and back to 1 with a Back ease on release (`ButtonAnimator.lua:28-34`, `ButtonAnimator.lua:143-183`). The six side buttons also lean toward the cursor; the lean is published as the `MagnetOffset` attribute, not written to Position (`ButtonAnimator.lua:52-55`, `ButtonAnimator.lua:82-85`). If a button's parent is hidden, its scale and lean reset (`ButtonAnimator.lua:188-204`).
- **Counters.** Every frame CashClient eases the shown cash and shards toward the real attributes, closing a fixed fraction of the gap per second (`CashClient.lua:50-58`, `CashClient.lua:63-86`). Friend Boost text is FriendCount x `Config.FriendBonus` x 100 (`CashClient.lua:88-91`). The multiplier line shows (CashMultiplier - 1) x 100 with trailing zeros trimmed (`CashClient.lua:106-116`). The "+" button opens the Roblox invite prompt (`CashClient.lua:121-137`).
- **Badges.** AlertBadges recomputes three booleans on every relevant signal: Upgrade (you can afford any panel upgrade, `AlertBadges.lua:56-65`), Index (any IndexState entry equals 0, `AlertBadges.lua:71-78`), Rebirth (every fruit discovered, `AlertBadges.lua:87-95`). A badge pops in by tweening its child UIScale named `Pop` from 0 to 1 (`AlertBadges.lua:36-45`).
- **Lock, Merge All, Mute.** LockButton warns locally if the lock is up or recharging, else fires `Lock` (`LockButton.lua:30-49`); a Heartbeat loop draws one label for both the lock and its cooldown (`LockButton.lua:57-89`). MergeAllButton sets its own image, warns if `MergeAllReadyAt` is in the future, else fires `MergeAll` (`MergeAllButton.lua:44-59`). MuteButton flips the client-only `MusicMuted` attribute and swaps the icon (`MuteButton.lua:25-38`).

### 3.3 Carry row and fruit prompts (GardenClient)
1. `refresh()` runs on `CarryingFruit`, `CarryingTool` and `PlotName` changes, on fruit added or removed, and every `Config.RefreshInterval` (0.2 s) (`GardenClient.lua:337-349`, `GardenClient.lua:430-435`).
2. It shows the carry row only while carrying. Drop shows for fruit and for tools of kind `"place"`; any other kind shows Use. Store always shows beside it (`GardenClient.lua:225-233`).
3. The name bar shows the fruit display name or tool display name. The rate bar shows the fruit's raw $/s, or for a tool the effect name plus `Config.DurationText(duration)`, or the tool's `note` (`GardenClient.lua:239-261`).
4. For every fruit in the shared fruit folder it sets the prompt locally: off if `Busy`, off if not on your plot, "Pick Up" if not carrying, "Merge" if carrying the same fruit name, else off (`GardenClient.lua:269-312`).
5. Matching fruit get a green Highlight, nearest first, up to `Config.MaxHighlights` (24) (`GardenClient.lua:314-334`).
6. Separately, `ProximityPromptService.PromptShown` shows the prompted fruit's `NameTag` billboard and adds a white outline Highlight (`GardenClient.lua:124-167`).
7. Drop and Use both fire the `Drop` remote; Store fires `Store` (`GardenClient.lua:351-364`).
8. A storage badge "n/cap" is built in code on the Store button and turns red when full (`GardenClient.lua:369-412`).
9. `Notify` and `Notice` remote strings become warning and notice toasts through GardenToast (`GardenClient.lua:415-427`).

### 3.4 Buff effect stack (GardenClient)
- One entry per `Config.Effects` key is built at load, sorted by name, then shown or hidden (`GardenClient.lua:549-557`). Each entry is an icon image, a draining bar and a Fredoka timer (`GardenClient.lua:464-547`).
- A Heartbeat loop reads `<Effect>EndsAt` (an absolute server-time deadline) and `<Effect>Span` (the length this drink bought) for each effect (`GardenClient.lua:559-581`). Under 20% left the bar turns red (`GardenClient.lua:584-587`). Text is `s`, `m:ss` or `h:mm:ss` (`GardenClient.lua:593-601`).

### 3.5 Full-screen panels
All five use the same open/close pattern: an `isOpen` flag separate from `Visible`, a UIScale tween from 0.85 to 1 on open and back to 0.85 on close, and hiding the other panels on open (`StoredFruitsMenu.lua:526-562`, `UpgradeMenu.lua:405-439`, `IndexMenu.lua:474-499`, `RebirthMenu.lua:105-129`, `ShopMenu.lua:604-633`). Each also resets `isOpen` when something else hides it (`ShopMenu.lua:640-642`).

- **ShopMenu.** Tabs come from `Config.ShopTabs`; each needs a `<Name>Tab` under ShopMenu and a `<Name>Page` under Panel, or it is warned and skipped (`ShopMenu.lua:55-65`). The tab column is re-centred vertically every run (`ShopMenu.lua:75-85`). `select(name)` shows one page and dims the other tabs with `Config.ShopTabDim` (`ShopMenu.lua:89-102`). `makeToolCard` builds one card per `Config.Tools` entry, sorted by key name (`ShopMenu.lua:197-376`, `ShopMenu.lua:378-388`). A card is the kit item-card image, a title, a preview (drawn art if `spec.image`, else a spinning ViewportFrame clone of the template), an "Owned: n" line, and the kit buy button with the price; Buy fires `BuyTool` with the tool name (`ShopMenu.lua:205-373`). The King page builds one card per `Config.KingEffects` entry; Buy fires `KingBuy`, Equip fires `KingEquip`, and cards repaint on `KingOwned` / `KingEffect` changes (`ShopMenu.lua:484-591`). The Robux page is shown and hidden but not filled by code.
- **IndexMenu.** `build()` clones `Templates.Cell` once per fruit, rank 64 (Blueberry) down to rank 1, sets the cell image to the tier's rarity frame, and fills a ViewportFrame (`IndexMenu.lua:301-361`). `stateOf(name)` returns `locked` (no IntValue), `unclaimed` (0) or `claimed` (1) (`IndexMenu.lua:236-241`). Locked fruit render as a faded black silhouette under a "???" pane (`IndexMenu.lua:95-188`). `paintInfo` fills the right-hand info and rewards boxes and greys Claim unless the fruit is unclaimed (`IndexMenu.lua:243-273`). Claim fires `IndexClaim` with the fruit name (`IndexMenu.lua:444-448`). When a claim lands, the selection moves to the next unclaimed fruit (`IndexMenu.lua:394-418`).
- **StoredFruitsMenu.** `rebuild()` destroys every card and rebuilds tool cards then fruit cards on any folder change, deferred to one pass per frame (`StoredFruitsMenu.lua:474-496`). Fruit cards use the rarity frame and sort by rank (`StoredFruitsMenu.lua:309-378`); tool cards use the shop card art and sort first with LayoutOrder -1000 (`StoredFruitsMenu.lua:384-443`). Tapping a card shows a code-drawn Equip overlay; Equip fires `Equip` or `EquipTool` and closes the panel (`StoredFruitsMenu.lua:203-307`). A "n/cap" label sits in the title bar (`StoredFruitsMenu.lua:448-471`).
- **UpgradeMenu.** `buildCards()` clones `Templates.UpgradeCard` per key in `Config.UpgradePanelKeys`, sets its art from `Config.UpgradeCards`, and wires Buy to fire `Upgrade` with the key (`UpgradeMenu.lua:295-341`). `refresh(key)` writes level, stat and price; a nil price means maxed (`UpgradeMenu.lua:57-85`). Prices turn red when unaffordable, so every `Cash` change refreshes all cards (`UpgradeMenu.lua:337-340`).
- **RebirthMenu.** `refresh()` writes the cash bonus, the shard payout and the rebirth count, and shows the locked overlay and blocker until every fruit is discovered (`RebirthMenu.lua:83-95`). The action button opens a confirm dialog; Yes closes both and fires `Rebirth` (`RebirthMenu.lua:195-209`).

### 3.6 Modal dialogs
ClearMenu, the rebirth confirm and OfflinePopup each add a full-screen transparent ImageButton named `Blocker` one ZIndex below the panel. It eats clicks that would fall through to the HUD. It is marked `Animated` before parenting so ButtonAnimator leaves it alone (`ClearMenu.lua:84-98`, `RebirthMenu.lua:174-188`, `OfflinePopup.lua:89-103`).
- **ClearMenu.** On press it checks `ClearReadyAt`, then `survey()` counts your lowest-rank fruit (`ClearMenu.lua:106-170`). The title reads "Clear all N Fruits?". Yes fires `Clear` (`ClearMenu.lua:197-201`).
- **OfflinePopup.** Opens once if `OfflineEarned >= Config.OfflineMinimum` and `OfflineAway >= Config.OfflineMinimumAway` (`OfflinePopup.lua:113-123`). The X only dismisses; the money is already banked (`OfflinePopup.lua:13-14`).

### 3.7 World-space UI
- **EarnPopups.** On each `Income(multiplier, dt)` event it prices each of your fruit as `Economy.ForName(name) x dt x multiplier` and floats a Highway-font "+N$" billboard over it after a random delay (`EarnPopups.lua:87-108`, `EarnPopups.lua:56-85`).
- **TopEarnersBoard.** Builds a SurfaceGui on `Main Island.TopEarnersBoard.Leaderboard` and every 2 s ranks all players by the `CashPerSecond` attribute, top 5, with your own row tinted (`TopEarnersBoard.lua:30-47`, `TopEarnersBoard.lua:191-232`).
- **PenUpgradeSign.** Clones `ReplicatedStorage.PenSignTemplate` locally, so only the owner sees it (`PenUpgradeSign.lua:7-11`, `PenUpgradeSign.lua:40-44`). It places the sign off the soil (the plot's largest part by footprint) toward the plaza, rechecked every second (`PenUpgradeSign.lua:238-293`, `PenUpgradeSign.lua:307-313`). Pressing it fires `Upgrade` with `"plotSize"` (`PenUpgradeSign.lua:226-232`).

### 3.8 Built in code versus pre-made art, and kit compliance

| Menu / element | Shell | What code builds | Follows the kit? |
|---|---|---|---|
| IndexMenu | Pre-made (panel, InfoBox, RewardsBox, Claim, `Templates.Cell`) | Cells cloned from the template; rarity frame image per tier (`IndexMenu.lua:321`); "???" lock pane (`IndexMenu.lua:105-125`) | Yes. Only the lock pane is a code-drawn TextLabel with UICorner. |
| RebirthMenu + RebirthConfirm | Pre-made (panel, action, locked overlay, amounts, confirm panel) | Only the Blocker sink (`RebirthMenu.lua:174-188`) | Yes. Confirm reuses the Clear dialog art (`RebirthMenu.lua:211-213`). |
| ClearMenu | Pre-made `ClearConfirm` | Blocker only | Yes |
| OfflinePopup | Pre-made | Blocker only; amount text | Yes. All words except the amount are baked into the art (`OfflinePopup.lua:4-6`). |
| UpgradeMenu | Pre-made panel and `Templates.UpgradeCard` | Card image swapped per key (`UpgradeMenu.lua:304-306`). Code-drawn fallback `paintCard` (`UpgradeMenu.lua:232-293`) | Yes. All three panel keys have art (`GardenConfig.lua:197-211`), so `paintCard` never runs today. |
| ShopMenu | Pre-made root, Panel, tabs, pages, close | Tool and King cards from kit images `Config.ToolBuyBox` / `Config.ToolBuyButton` (`ShopMenu.lua:207`, `ShopMenu.lua:302`, `ShopMenu.lua:487`, `ShopMenu.lua:551`) | Mostly. The King page Equip button is a code-drawn green TextButton pill with UICorner and UIStroke (`ShopMenu.lua:567-583`). |
| StoredFruitsMenu | Pre-made panel | Cards from rarity frames and shop card art; storage count label | Mostly. The Equip overlay is a code-drawn dark Frame plus a two-tone green TextButton (`StoredFruitsMenu.lua:203-276`). |
| CornerHud, TopButtons, MergeAll, Mute, AlertBadge | Pre-made kit art | Text and image swaps only | Yes |
| Carry row / carry info | Pre-made kit art | Storage badge: dark rounded TextLabel (`GardenClient.lua:369-395`) | Mostly. The badge is not kit art. |
| EffectStack | Pre-made empty frame | Entries: icon image plus code-drawn bar Frames with UICorner (`GardenClient.lua:481-522`) | Partly. Icons are images; bars are code. |
| TopEarnersBoard | None | Entire SurfaceGui: flat Frames, UICorner, LuckiestGuy title (`TopEarnersBoard.lua:36-183`) | No |
| PenUpgradeSign | Template part only | Entire SurfaceGui: flat Frames, gradient button, hand-drawn bracket icon (`PenUpgradeSign.lua:90-170`) | No |
| EarnPopups | None | BillboardGui in Highway font (`EarnPopups.lua:30-54`) | No, by design: Config reserves Highway for text over fruit (`GardenConfig.lua:20-25`). |

**Fonts.** Two spellings of Fredoka One are used:
- `Enum.Font.FredokaOne`: `GardenClient.lua:377`, `IndexMenu.lua:108`, `StoredFruitsMenu.lua:265`, `StoredFruitsMenu.lua:344`, and via `Config.UpgradeFont`, `Config.BoardFont`, `Config.PenSignFont` (`GardenConfig.lua:266`, `GardenConfig.lua:2058`, `GardenConfig.lua:2103`). GardenToast also uses it (`GardenToast.lua:95`).
- `Font.new("rbxasset://fonts/families/FredokaOne.json", Regular)`, the family string the kit names: `GardenClient.lua:457`, `ShopMenu.lua:47`. ShopMenu warns that `Font.fromId` with a numeric id silently falls back to the default font (`ShopMenu.lua:44-46`).
- Exceptions: LuckiestGuy for the board title and place numbers (`GardenConfig.lua:2059`, `TopEarnersBoard.lua:70`, `TopEarnersBoard.lua:148`); Highway for earn popups (`GardenConfig.lua:1093`).

**Text style.** White text (`Color3.fromRGB(255, 255, 255)`), `TextScaled = true`, black UIStroke. Size caps use UITextSizeConstraint (for example 30 / 24 / 20 on shop cards, `ShopMenu.lua:225`, `ShopMenu.lua:328`, `ShopMenu.lua:344`). No emoji or non-ASCII characters appear in any of the 18 scripts (checked with a byte search).

**Stroke weights.**

| Where | Thickness | Citation |
|---|---|---|
| Inventory card title and count | 2.5 | `StoredFruitsMenu.lua:351-354`, `StoredFruitsMenu.lua:369-372` |
| Inventory title-bar count | 3 | `StoredFruitsMenu.lua:460-463` |
| Inventory Equip button outline | 4 | `StoredFruitsMenu.lua:238-241` |
| Shop card title | 2.5 | `ShopMenu.lua:226-227` |
| Shop price, Owned, King labels | 2 | `ShopMenu.lua:329-330`, `ShopMenu.lua:345-346`, `ShopMenu.lua:457` |
| King Equip pill border (Border mode) | 2.5 | `ShopMenu.lua:580` |
| Storage badge border (Border mode) | 2 | `GardenClient.lua:385-389` |
| Effect timer text | 2.5 | `GardenClient.lua:541-544` |
| Index: every stroke rescaled | 5 x panel width / 1831, min 1 | `IndexMenu.lua:608-619` |
| Upgrade cards: text / code border | 3 and 6 x card width / 320 | `UpgradeMenu.lua:384-389` |
| Clear dialog: text / button edges | 6 and 10 x width / 720 | `ClearMenu.lua:179-190` |
| Rebirth confirm text | 6 x width / 720 | `RebirthMenu.lua:214-223` |
| Offline amount | 4 x width / size-cap width (720 default) | `OfflinePopup.lua:45-58` |
| Top Earners title and place | 2, 35-40% transparent | `TopEarnersBoard.lua:76-80`, `TopEarnersBoard.lua:154-158` |
| Pen sign text / price / edge / button edge | 5 / 6 / 10 / 8 | `PenUpgradeSign.lua:86`, `PenUpgradeSign.lua:170`, `PenUpgradeSign.lua:96-100`, `PenUpgradeSign.lua:126` |
| Earn popup | 2.5 (`Config.EarnPopupStroke`) | `EarnPopups.lua:51-54` |

UIStroke thickness is in real pixels, so the image-based panels rescale it against the panel's on-screen width (`ClearMenu.lua:172-178`).

## 4. Data

### Remotes (all RemoteEvents in `ReplicatedStorage.GardenRemotes`; no RemoteFunctions)

| Name | Direction | Payload | Citation |
|---|---|---|---|
| Drop | client to server | none (Drop and Use both) | `GardenClient.lua:351-360` |
| Store | client to server | none | `GardenClient.lua:362-364` |
| Notify | server to client | warning string | `GardenClient.lua:415-419` |
| Notice | server to client | good-news string | `GardenClient.lua:423-427` |
| BuyTool | client to server | tool name | `ShopMenu.lua:373` |
| KingBuy | client to server | effect key | `ShopMenu.lua:562` |
| KingEquip | client to server | effect key | `ShopMenu.lua:583` |
| IndexClaim | client to server | fruit name | `IndexMenu.lua:447` |
| Equip | client to server | fruit name | `StoredFruitsMenu.lua:298` |
| EquipTool | client to server | tool name | `StoredFruitsMenu.lua:439`, `StoredFruitsMenu.lua:298` |
| Upgrade | client to server | upgrade key (`"plotSize"` from the sign) | `UpgradeMenu.lua:328`, `PenUpgradeSign.lua:230` |
| Rebirth | client to server | none | `RebirthMenu.lua:208` |
| Clear | client to server | none | `ClearMenu.lua:200` |
| Lock | client to server | none | `LockButton.lua:48` |
| MergeAll | client to server | none | `MergeAllButton.lua:58` |
| Income | server to client | (multiplier, dt) numbers | `EarnPopups.lua:87-88` |

### Player attributes read (server-written unless noted)

| Attribute | Read by | Server writer seen |
|---|---|---|
| CarryingFruit | GardenClient refresh, StoredFruitsMenu equip guard (`GardenClient.lua:211`, `StoredFruitsMenu.lua:294`) | `Carrying.lua:142` |
| CarryingTool | GardenClient (`GardenClient.lua:225`) | `Tools.lua:158` |
| PlotName | GardenClient, UpgradeMenu, ClearMenu, EarnPopups, PenUpgradeSign (`GardenClient.lua:266`, `ClearMenu.lua:107`, `EarnPopups.lua:89`, `PenUpgradeSign.lua:254`) | `GardenService.lua:437` |
| `<Effect>EndsAt`, `<Effect>Span` | Effect stack (`GardenClient.lua:562`, `GardenClient.lua:579`) | not searched |
| Cash | CashClient, UpgradeMenu, AlertBadges, PenUpgradeSign (`CashClient.lua:64`, `UpgradeMenu.lua:73`, `AlertBadges.lua:57`, `PenUpgradeSign.lua:202`) | `Progression.lua:82` |
| Shards | CashClient (`CashClient.lua:65`) | `Progression.lua:217` |
| ProfileLoaded | CashClient, OfflinePopup (`CashClient.lua:76`, `OfflinePopup.lua:109`) | `GardenStore.lua:800` |
| FriendCount | CashClient (`CashClient.lua:89`) | `Progression.lua:122` |
| CashMultiplier | CashClient (`CashClient.lua:107`) | `Progression.lua:342` |
| KingOwned (comma list), KingEffect (one key) | ShopMenu King page (`ShopMenu.lua:465`, `ShopMenu.lua:471`) | `Progression.lua:241-242` |
| `Upgrade_<key>` | UpgradeMenu, AlertBadges, PenUpgradeSign (`UpgradeMenu.lua:54`, `AlertBadges.lua:60`, `PenUpgradeSign.lua:175`) | not searched |
| Rebirths | RebirthMenu (`RebirthMenu.lua:84`, `RebirthMenu.lua:252`) | `Progression.lua:96` |
| ClearReadyAt | ClearMenu (`ClearMenu.lua:146`) | `Merging.lua:425` |
| MergeAllReadyAt | MergeAllButton (`MergeAllButton.lua:49`) | `Merging.lua:572` |
| LockEndsAt, LockReadyAt | LockButton (`LockButton.lua:36-42`, `LockButton.lua:61-65`) | `RebirthLock.lua:238-239` |
| OfflineEarned, OfflineAway | OfflinePopup (`OfflinePopup.lua:113-114`) | `Persistence.lua:328-329` |
| CashPerSecond (on every Player) | TopEarnersBoard (`TopEarnersBoard.lua:188`) | `Ambient.lua:469` |
| MusicMuted | MuteButton reads and WRITES it on the client; not replicated, not saved (`MuteButton.lua:10-13`, `MuteButton.lua:37`) | none |

### Instance attributes

| Attribute | On | Read or written | Citation |
|---|---|---|---|
| PlotName | fruit part | read | `GardenClient.lua:271`, `ClearMenu.lua:115`, `EarnPopups.lua:94` |
| Busy | fruit part | read (prompt off while set) | `GardenClient.lua:295` |
| HeldBy | fruit part | read (skip held fruit) | `ClearMenu.lua:116`, `EarnPopups.lua:95` |
| Animated | any GuiButton | written by ButtonAnimator and the Blockers; a set value means "do not animate" | `ButtonAnimator.lua:126-129`, `ClearMenu.lua:95` |
| MagnetOffset (Vector2, pixels) | magnetic HUD buttons | written by ButtonAnimator, read by HudScale | `ButtonAnimator.lua:84`, `HudScale.lua:119` |
| Fruit / Tool | inventory cards | written, not read by anything in this subsystem | `StoredFruitsMenu.lua:332`, `StoredFruitsMenu.lua:401` |

Local-only property writes: ProximityPrompt `Enabled`, `ActionText`, `HoldDuration`, `ClickablePrompt` and NameTag `Enabled` (`GardenClient.lua:12-14`, `GardenClient.lua:285-310`).

### Replicated value folders read

| Folder on Player | Config key | Meaning | Read by |
|---|---|---|---|
| IndexState | `Config.IndexFolder` (`GardenConfig.lua:390`) | IntValue per fruit: 0 discovered, 1 claimed, absent locked | IndexMenu, RebirthMenu, AlertBadges (`IndexMenu.lua:237-240`, `RebirthMenu.lua:58`, `AlertBadges.lua:72`) |
| StoredFruits | `Config.StoredFolder` (`GardenConfig.lua:528`) | IntValue per fruit = count | GardenClient badge, StoredFruitsMenu (`GardenClient.lua:370`, `StoredFruitsMenu.lua:44`) |
| StoredTools | `Config.ToolsFolder` (`GardenConfig.lua:1604`) | IntValue per tool = count | ShopMenu Owned label, StoredFruitsMenu (`ShopMenu.lua:351-353`, `StoredFruitsMenu.lua:48`) |

### Saved fields (read only through their replicated mirrors; the client writes none)

| Save field (`GardenStore.Default`) | Mirror the UI reads |
|---|---|
| `cash` (`GardenStore.lua:225`) | Cash attribute |
| `shards` (`GardenStore.lua:259`) | Shards attribute |
| `rebirths` (`GardenStore.lua:264`) | Rebirths attribute |
| `index` (`GardenStore.lua:233-238`, same 0/1/absent meaning) | IndexState folder |
| `stored` (`GardenStore.lua:231`, "from the StoredFruits IntValues") | StoredFruits folder |
| `tools` (`GardenStore.lua:239-243`) | StoredTools folder |
| `kingOwned`, `kingEquipped` (`GardenStore.lua:244-249`) | KingOwned, KingEffect attributes |
| `upgrades` (`GardenStore.lua:276`; levels "replicate as attributes on the Player", `GardenConfig.lua:29-30`) | `Upgrade_<key>` attributes |
| `lastSeen` (`GardenStore.lua:266-271`, "the ONLY input to offline earnings") | OfflineEarned, OfflineAway (computed by the server) |

### Config keys read, by script
- GardenClient: DurationText, EffectBarHeight, EffectBarInset, EffectBarLow, EffectBarLowColor, EffectBarTrack, EffectIconSize, EffectLabelHeight, Effects, FruitFolder, HighlightColor, IslandName, MaxHighlights, MergeSoundId, NameTagSweepSeconds, PromptHighlightColor, RefreshInterval, StoredCap, StoredCountText, StoredFolder, StoredTotal, Tools.
- ShopMenu: KingAssets, KingDefault, KingEffectByKey, KingEffects, KingEnabled, KingIcons, ShopTabDim, ShopTabs, TNTBuyBox, TNTBuyButton, ToolBuyBox, ToolBuyButton, ToolTemplates, Tools, ToolsFolder (ShopPanelLeftEdge only in a comment).
- IndexMenu: IndexCanvas, IndexClaimedTint, IndexFolder, IndexLockedColor, IndexLockedFade, IndexLockedMaskColor, IndexLockedMaskTransparency, IndexLockedMaskWiden, IndexLockedText, IndexSelectFrom, IndexSelectTime, IndexTextStroke, RarityColors, RarityFrames, RarityGradients, TranscendentAccent, TranscendentAccentOffset, TranscendentAccentScale, TranscendentAccentTransparency, TranscendentShimmer, TranscendentShimmerSeconds.
- StoredFruitsMenu: CardAspect, EquipWarning, RarityFrames, StoredCap, StoredCountText, StoredFolder, StoredTotal, TNTBuyBox, ToolTemplates, Tools, ToolsFolder.
- UpgradeMenu: MaxLevel, UpgradeCardAspect, UpgradeCardThemes, UpgradeCards, UpgradeFont, UpgradeLevelText, UpgradePanelKeys, UpgradePrice, UpgradePriceColor, UpgradeStat, UpgradeUnaffordable.
- RebirthMenu: ClearStrokeCanvas, ClearTextStroke, IndexFolder, RebirthCashBonus, RebirthCashText, RebirthConfirmBody, RebirthCountText, RebirthShardsText.
- CashClient: CashCatchUp, CashMultiplierText, FriendBonus (IncomeTick only in a comment).
- HudScale: HudClusters, HudScaleFor. ButtonAnimator: HudClusters.
- AlertBadges: IndexFolder, MaxLevel, UpgradeKeys, UpgradePanelKeys, UpgradePrice (RebirthNeedsAll only in a comment).
- ClearMenu: ClearConfirmBody, ClearConfirmTitle, ClearEdgeStroke, ClearEmptyPlot, ClearOnlyKind, ClearStrokeCanvas, ClearTextStroke, ClearWait, FruitFolder, IslandName, Plural.
- LockButton: LockAlreadyUp, LockCooldownColor, LockCooldownWarning, LockTimerColor, LockTimerText.
- MergeAllButton: MergeAllButton, MergeAllWait. MuteButton: MusicOffIcon, MusicOnIcon.
- OfflinePopup: OfflineAmountText, OfflineMinimum, OfflineMinimumAway.
- EarnPopups: EarnPopupColor, EarnPopupFont, EarnPopupHeight, EarnPopupMaxDistance, EarnPopupMaxTextSize, EarnPopupRise, EarnPopupStagger, EarnPopupStroke, EarnPopupStuds, EarnPopupTime, FruitFolder, IslandName, NameTagForward.
- TopEarnersBoard: all `Board*` keys (BoardBack, BoardBackAlpha, BoardCanvas, BoardEmpty, BoardFace, BoardFont, BoardModel, BoardNameColor, BoardNormal, BoardPadding, BoardPlaceColors, BoardPlaceDefault, BoardRateColor, BoardRefresh, BoardRowBack, BoardRowGap, BoardRowHeight, BoardRows, BoardStroke, BoardSubtitle, BoardSubtitleColor, BoardTitle, BoardTitleColor, BoardTitleFont, BoardYouSuffix, BoardYouTint), IslandName.
- PenUpgradeSign: all `PenSign*` keys, IslandName, MaxLevel, PlotsFolder, UpgradePrice.

Shared modules used: FruitIndex (ladder lookups), FruitEconomy (`Format`, `ForName`, `ForRank`), IndexRewards (`For`, `Shards`, `RarityText`), GardenToast (`Warn`, `Notice`).

## 5. Numbers

| Value | Number | Key and citation |
|---|---|---|
| Storage cap (all fruit kinds together) | 5 | `Config.StoredCap`, `GardenConfig.lua:533` |
| Counter easing rate | 2.5 per second | `Config.CashCatchUp`, `GardenConfig.lua:378` |
| Friend boost per friend | +10% | `Config.FriendBonus`, `GardenConfig.lua:357` |
| Income tick (drives earn popups) | 5 s | `Config.IncomeTick`, `GardenConfig.lua:372` |
| Prompt refresh poll | 0.2 s | `Config.RefreshInterval`, `GardenConfig.lua:1891` |
| Max merge highlights | 24 | `Config.MaxHighlights`, `GardenConfig.lua:1889` |
| Name tag sweep period | 5 s | `Config.NameTagSweepSeconds`, `GardenConfig.lua:652` |
| Effect entry size | icon 58 + bar 7 + label 20 px | `Config.EffectIconSize/EffectBarHeight/EffectLabelHeight`, `GardenConfig.lua:1824-1832` |
| Effect bar warning threshold | 20% left | `Config.EffectBarLow`, `GardenConfig.lua:1827` |
| HUD reference viewport | 1600 x 900 | `Config.HudReference`, `GardenConfig.lua:1871` |
| HUD dampening exponent / min / max | 0.6 / 0.55 / 1.0 | `GardenConfig.lua:1876-1878` |
| HUD touch multiplier | 0.85 | `Config.HudTouchScale`, `GardenConfig.lua:1883` |
| Smallest viewport treated as real | 240 px | `Config.HudMinViewport`, `GardenConfig.lua:1947` |
| Button hover / press scale | 1.08 / 0.92 | `ButtonAnimator.lua:28-29` |
| Button tween times hover / press / release | 0.15 / 0.07 / 0.28 s | `ButtonAnimator.lua:31-34` |
| Magnet pull / reach / rate | 0.3 / 8% of size / 14 per s | `ButtonAnimator.lua:56-58` |
| Panel open / close tween | 0.22 s Back / 0.14 s Quad, from scale 0.85 | `IndexMenu.lua:456-457` (same in Stored, Upgrade, Rebirth, Clear, Offline) |
| Shop open / close tween | 0.18 s / 0.12 s | `ShopMenu.lua:49-50` |
| Grid columns: Index / Inventory / Shop / Upgrades | 5 / 4 / 3 / 3 | `IndexMenu.lua:546`, `StoredFruitsMenu.lua:71`, `ShopMenu.lua:138`, `UpgradeMenu.lua:349` |
| Index selection bounce | from 1.25, 0.24 s | `Config.IndexSelectFrom/IndexSelectTime`, `GardenConfig.lua:434-435` |
| Index stroke reference | 5 px at 1831 px wide | `Config.IndexTextStroke/IndexCanvas`, `GardenConfig.lua:424-425` |
| Transcendent shimmer period | 5.5 s | `Config.TranscendentShimmerSeconds`, `GardenConfig.lua:495` |
| Tool prices (shards) and durations | TNT 2500 (radius 18); Speed 5000 / 300 s; Jump 5000 / 300 s; Aether 10000 / 1800 s; Invisibility 5000 / 60 s; Feather Falling 5000 / 600 s; Luck 7500 / 300 s | `Config.Tools`, `GardenConfig.lua:1616-1703` |
| King aura prices (shards) | Crown 0, Fire 4000, Water 6000, Slash 10000, Rift 20000 | `Config.KingEffects`, `GardenConfig.lua:1237-1241` |
| Upgrade price base | capacity 100, spawnRate 150, spawnTier 250, plotSize 2500 | `Config.UpgradePriceBase`, `GardenConfig.lua:143` |
| Upgrade price growth per level | 1.90 / 3.40 / 2.60 / 8.0 | `Config.UpgradePriceGrowth`, `GardenConfig.lua:157` |
| Capacity ladder | 6 to 35 fruit | `GardenConfig.lua:51-52` |
| Spawn interval ladder | 10 s down to 2.5 s, step 0.5 | `GardenConfig.lua:60-62` |
| Plot sizes | 29.5 x 47.1 up to 48 x 78 studs | `Config.PlotSizes`, `GardenConfig.lua:117-123` |
| Rebirth cash bonus shown | +100% per rebirth | `Config.RebirthCashBonus = 1.00`, `GardenConfig.lua:1000`; drawn at `RebirthMenu.lua:88` |
| Lock duration / cooldown | 60 s / 30 s | `GardenConfig.lua:885-886` |
| Clear cooldown | 5 s | `Config.ClearCooldown`, `GardenConfig.lua:1056` |
| Merge All cooldown | 300 s | `Config.MergeAllCooldown`, `GardenConfig.lua:973` |
| Offline popup gates | at least $1 and 300 s away | `GardenConfig.lua:763`, `GardenConfig.lua:776` |
| Offline rate / cap (server) | 3% of live / 5 h | `GardenConfig.lua:753-754` |
| Earn popup life / rise / stagger / draw distance | 1.25 s / 2.6 studs / 0.35 s / 120 studs | `GardenConfig.lua:1099-1105` |
| Top Earners refresh / rows / draw distance | 2 s / 5 / 220 studs (hard-coded) | `GardenConfig.lua:2039`, `GardenConfig.lua:2044`, `TopEarnersBoard.lua:45` |
| Pen sign offset out / side / draw distance | 6.5 / 13 / 140 studs | `GardenConfig.lua:2098-2100` |
| Pen sign placement poll | 1 s | `PenUpgradeSign.lua:309` |

## 6. Extension points for the expansion

### 1. Rebirth-gated rarity, +1 storage per rebirth, harder rebirth
- **Rebirth gate.** `indexComplete()` (`RebirthMenu.lua:57-65`) tests "every fruit in FruitIndex discovered". It must become "every fruit in the tiers unlocked so far, plus enough shards and cash". `rebirthWaiting()` (`AlertBadges.lua:87-95`) is a copy of the same test and must change in the same edit, or the badge will invite players into a locked panel. AlertBadges only listens to Cash, `Upgrade_*` and the Index folder (`AlertBadges.lua:105-135`), so add Shards and Rebirths signals. RebirthMenu only listens to the Index folder and Rebirths (`RebirthMenu.lua:231-252`), so add Cash and Shards if it shows progress. The server's own test lives outside this subsystem; all three must agree.
- **Requirement display.** The locked overlay is a baked image (`Config.RebirthLocked`, `GardenConfig.lua:1017`; kit id 71043479073116). Showing "index n/m, shards needed, cash needed" needs new art or live labels drawn over it.
- **Slight boost.** The cash box prints `Config.RebirthCashBonus * 100` through `"+%d%%"` (`RebirthMenu.lua:88`, `GardenConfig.lua:1032`). `%d` is for whole numbers, so a fractional percent (for example 12.5) will not print as intended. Use a whole percent or change the format.
- **Index tier lock.** `stateOf()` (`IndexMenu.lua:236-241`) has three states. A fourth, "tier locked until rebirth N", is needed so a player can tell "not found yet" from "rebirth to unlock". `fillViewport()` and `applyLockMask()` (`IndexMenu.lua:95-188`) draw the locked look; `build()` sets the frame per tier (`IndexMenu.lua:321`). IndexMenu watches no player attributes today, so it must start listening to Rebirths to repaint.
- **Storage +1 per rebirth.** `Config.StoredCap` is a constant 5 (`GardenConfig.lua:533`). It is read directly at `GardenClient.lua:399-400` and `StoredFruitsMenu.lua:467-468`. Replace it with a function of Rebirths, shared with the server, and add a Rebirths listener to `refreshBadge` (`GardenClient.lua:397-411`) and `refreshCount` (`StoredFruitsMenu.lua:465-471`, which today only runs inside `rebuild`).
- **Merge stops at the cap.** `refresh()` offers "Merge" and the green glow for any matching fruit on your plot (`GardenClient.lua:273`, `GardenClient.lua:304-307`). If the server refuses merges at the tier cap, the client will still offer them. Add a cap check here.
- **Spawn rarity card.** "Up to X" comes from `Config.UpgradeStat` (`GardenConfig.lua:2011-2015`) and a nil price is the only "maxed" signal (`UpgradeMenu.lua:69-84`). If the spawn ceiling is capped by rebirth tier, the card needs a third state ("rebirth to unlock") and `upgradeWaiting()` (`AlertBadges.lua:56-65`) must skip that level.

### 2. Bats that upgrade per rebirth, coloured by tier
- If a bat is added to `Config.Tools`, ShopMenu builds a shop card for it automatically (`ShopMenu.lua:381-386`), the inventory shows a tool card (`StoredFruitsMenu.lua:484-488`), the carry row shows Use for any kind other than `"place"` (`GardenClient.lua:227`), and the save keeps a count of it (`GardenConfig.lua:1601-1603`). A free, per-rebirth bat probably should not be a `Config.Tools` entry, or ShopMenu needs a "not for sale" filter in its build loop.
- Tier colours already exist as `Config.RarityColors` (`GardenConfig.lua:582`; used at `IndexMenu.lua:259`). A bat level readout would fit in CornerHud next to the multiplier line (`CashClient.lua:106-116`). A new HUD cluster must be added to `Config.HudClusters` (`GardenConfig.lua:1863-1867`).

### 3. Explosives that drain fruit health
- New TNT tiers and nukes as `Config.Tools` entries with `kind = "place"` get shop and inventory cards and the Drop button with no client code (`ShopMenu.lua:378-388`, `GardenClient.lua:225-233`). Each needs a template in `ReplicatedStorage.Tools`, or its shop card is skipped with only a warning (`ShopMenu.lua:198-203`).
- The shop sorts by key name (`ShopMenu.lua:382-383`), so tiers will not appear in price order. Add an explicit order field and sort on it.
- A shop card shows only name and price. A damage or radius line needs a new label; the `Owned` line sits at 60% height (`ShopMenu.lua:340`).
- The carry info rate bar shows `toolSpec.note` for a tool with no timed effect (`GardenClient.lua:250`). Give each explosive a `note` (for example its damage) or the bar reads "n/a".
- Fruit health bars: the only per-fruit UI here is the NameTag shown for the prompted fruit (`GardenClient.lua:138-159`). Showing damage on every hurt fruit needs its own pass. The `refresh()` loop already walks every fruit 5 times a second (`GardenClient.lua:269-320`) but is not the right place for per-frame visuals.

### 4. 100k-shard underground base, ores, Ores index category
- **Purchase.** UpgradeMenu hard-codes "$" and compares against Cash (`UpgradeMenu.lua:73-74`). So do PenUpgradeSign (`PenUpgradeSign.lua:198-206`) and AlertBadges (`AlertBadges.lua:57-62`). A shard-priced upgrade needs currency-aware text and affordability. `Config.UpgradeCurrency` exists but is empty and unused here (`GardenConfig.lua:190`).
- **Ores in the Index.** IndexMenu is one grid built from FruitIndex (`IndexMenu.lua:307-355`), keyed by fruit name in IndexState. A category switch can copy ShopMenu's tab pattern (`ShopMenu.lua:55-129`). `fitCanvas()` counts every GuiObject child of the list (`IndexMenu.lua:571-575`), so cells for a hidden category must live in a separate list or the canvas will be too tall. `indexWaiting()` lights the Index badge for any 0 in IndexState (`AlertBadges.lua:71-78`); ores stored there will light it too. The rebirth test iterates FruitIndex only (`RebirthMenu.lua:60-63`), so ores will not count toward rebirth unless added.
- **Ores in storage.** `makeCard()` returns nil for names not in FruitIndex (`StoredFruitsMenu.lua:311-313`), and `Config.StoredTotal` ignores them (`GardenConfig.lua:543`). Ores need a third card builder like `makeToolCard` (`StoredFruitsMenu.lua:384-443`).

### 5. Forest pets
- **New panel.** Every panel hard-codes the list of panels it hides: `ShopMenu.lua:617`, `IndexMenu.lua:47`, `StoredFruitsMenu.lua:541`, `UpgradeMenu.lua:421`, `RebirthMenu.lua:51`. A Pets panel must be added to all five, and must copy the `isOpen` / 0.85 scale pattern (section 3.5).
- **HUD button.** Add it to `Config.HudClusters` (`GardenConfig.lua:1863-1867`) or it will not scale, and to `MAGNETIC` (`ButtonAnimator.lua:52-55`) if it sits in a side column. Use a free zone from [[UI Kit Inventory]].
- **Taming prompt.** GardenClient's PromptShown handler runs for every ProximityPrompt in the game (`GardenClient.lua:161-167`). It will put a white outline on a pet until the next refresh clears it (`GardenClient.lua:216`). If a pet has a child named `NameTag` without a `Label` child, `setTags` will error (`GardenClient.lua:148-151`). Filter by prompt parent before touching it.
- **Pet income popups.** EarnPopups only prices fruit in the fruit folder (`EarnPopups.lua:93-106`). Pet earnings need their own popup path.

### 6. Boats, swimming fatigue, Aqua Affinity potion
- Aqua Affinity follows the potion path in feature 7.
- A boat as a `Config.Tools` entry of kind `"place"` would get the Drop button and a spinning viewport card framed off the model's size (`ShopMenu.lua:249-297`). The shop price label is the bare number (`ShopMenu.lua:318`); the shard crystal is baked into the button art (`ShopMenu.lua:315`), so this only works for shard prices.
- A swim fatigue meter is not a timed effect, and the effect stack only reads `<Effect>EndsAt` (`GardenClient.lua:559-563`). It needs its own widget, placed in a free zone, and listed in `Config.HudClusters`.

### 7. More potions
- Client path with no new code: a `Config.Tools` entry (`kind = "consume"`, `price`, `effect`, `duration`, optional `image`) (`GardenConfig.lua:1615`), a `Config.Effects` entry (`display`, `bar` colour, `icon`) (`GardenConfig.lua:1714`), and a template in `ReplicatedStorage.Tools`. That gives a shop card (`ShopMenu.lua:381-386`), an inventory card (`StoredFruitsMenu.lua:484-488`), a Use button and "Effect 5 min" info text (`GardenClient.lua:241-247`), and an effect stack entry (`GardenClient.lua:549-557`).
- Risks: a missing template skips the shop card (warn only, `ShopMenu.lua:198-203`) and the inventory card (silently, `StoredFruitsMenu.lua:387-388`). Each active effect adds 85 px to the stack, which grows upward from the bottom right (`GardenClient.lua:473-474`), toward the Clear button column. Feather Falling and Luck icons are still placeholders (`GardenConfig.lua:1770`, `GardenConfig.lua:1784`). Each viewport card adds its own RenderStepped spin connection (`ShopMenu.lua:289-297`); a potion with `image` avoids that.

### 8. Plot visual overhaul
- `soilOf()` picks the plot's largest part by footprint (`PenUpgradeSign.lua:238-247`). A new decorative part larger than the soil (a big studded base) would move the sign. The comment says PlotRegistry uses the same rule (`PenUpgradeSign.lua:236-237`), so the server's soil detection would move too.
- The sign trusts the ground raycast only within 1.5 studs of Y = 0 (`PenUpgradeSign.lua:286-289`). A raised plot floor makes it fall back to Y = 0.1.
- `plotModels` matches any workspace Model named `Plot<number>` (`UpgradeMenu.lua:34-37`), and `plotIcon` clones the whole plot (`UpgradeMenu.lua:126-140`). It is unused in the panel today, but a heavier plot makes it costlier if revived.
- TopEarnersBoard waits with no timeout for `Main Island.TopEarnersBoard.Leaderboard` (`TopEarnersBoard.lua:30-32`). Renaming or moving the board during a map rework silently stops the script.

### 9. UI overhaul to the image kit
- Already compliant: IndexMenu, RebirthMenu and its confirm, ClearMenu, OfflinePopup, UpgradeMenu, CornerHud, TopButtons, side HUD buttons (table in 3.8).
- To convert: the King page Equip pill (`ShopMenu.lua:567-583`), the inventory Equip overlay (`StoredFruitsMenu.lua:203-276`; the kit's green Claim button is the nearest existing art), the storage badge (`GardenClient.lua:369-395`), the effect bars (`GardenClient.lua:494-522`), the Index "???" pane (`IndexMenu.lua:105-125`), and the two world signs (`TopEarnersBoard.lua:36-183`, `PenUpgradeSign.lua:90-170`). The board's LuckiestGuy title (`GardenConfig.lua:2059`) is the one non-Fredoka screen-readable font besides the deliberate Highway popups.
- Use the family string `Font.new("rbxasset://fonts/families/FredokaOne.json")` for new text, as ShopMenu does (`ShopMenu.lua:47`).
- New image panels should rescale their strokes against panel width the way ClearMenu does (`ClearMenu.lua:179-193`).
- The Royal / King screens Samarth flagged are mostly in `RoyalUI.lua` and `KingClient.lua`, which are not part of this subsystem and were not read for this note. The only King UI here is the Shop's King tab.

### 10. Royal combat weapon skins
- If skins are bought or picked from the Shop, add a tab: an entry in `Config.ShopTabs` (`GardenConfig.lua:1210-1212`) plus a `<Name>Tab` and `<Name>Page` in StarterGui (`ShopMenu.lua:56-65`). The column re-spaces itself (`ShopMenu.lua:75-85`).
- The King page is the pattern to copy: ownership as a comma-list attribute, buy and equip remotes, and cards that repaint on attribute change (`ShopMenu.lua:463-482`, `ShopMenu.lua:589-591`).
- A picker shown right before a duel and the duel health bars belong to the royal combat scripts, outside this subsystem.

## 7. Gotchas

1. **ButtonAnimator grabs every new GuiButton.** It adopts on `DescendantAdded` and takes the button's UIScale slot (`ButtonAnimator.lua:131-135`, `ButtonAnimator.lua:213-217`). A full-screen helper button must have `Animated` set before it is parented, or the whole screen hover-scales (`ClearMenu.lua:82-83`).
2. **Only HudScale may set HUD cluster Position or Size.** It captures the authored values once and rescales from them (`HudScale.lua:52-53`, `HudScale.lua:16-19`). Another script writing Position is overwritten on the next resize.
3. **A HUD cluster missing from `Config.HudClusters` keeps its desktop size** while everything else shrinks (`GardenConfig.lua:1859-1862`). HudScale waits only 5 s per cluster, then warns and skips it forever (`HudScale.lua:64-67`).
4. **Never give a button cluster a UIScale.** HudScale writes button Size directly because ButtonAnimator would overwrite a shared UIScale on first hover (`HudScale.lua:27-33`).
5. **Scrolling grids must use pixel heights and an explicit canvas.** A vertical Scale in a ScrollingFrame resolves against the canvas and can diverge (6000 px canvas took a cell to 464,240 px) (`IndexMenu.lua:526-545`; same rule `StoredFruitsMenu.lua:87-104`, `ShopMenu.lua:159-161`).
6. **`Visible` is not "open".** Panels stay visible during the close tween; toggles must read `isOpen` (`ShopMenu.lua:606-610`).
7. **Panel exclusion lists are hand-maintained and uneven.** StoredFruitsMenu and UpgradeMenu do not list IndexMenu (`StoredFruitsMenu.lua:541`, `UpgradeMenu.lua:421`); IndexMenu covers that by watching the others (`IndexMenu.lua:510-522`). IndexMenu caches its list with `FindFirstChild` at load (`IndexMenu.lua:46-50`), so a panel that is still replicating at that moment is never in it (HudScale documents that GardenUI children can arrive late on a slow join, `HudScale.lua:57-63`).
8. **Shop tweens differ** from every other panel: 0.18 / 0.12 s versus 0.22 / 0.14 s (`ShopMenu.lua:49-50`, `IndexMenu.lua:456-457`).
9. **The prompt handler is global.** Any new ProximityPrompt (pets, ores, boats) gets GardenClient's outline for up to 0.2 s and its `NameTag` lookup (`GardenClient.lua:124-167`, `GardenClient.lua:216`).
10. **Highlights are capped** at 24 because Roblox stops drawing past about 31 (`GardenConfig.lua:1887-1889`). New Highlight effects (ore glow, pet focus) share that budget with merge highlights.
11. **`Busy` must be checked first** in the prompt chain; otherwise a player can catch their own thrown fruit (`GardenClient.lua:289-296`).
12. **Rank runs backward.** Rank 1 is the rarest fruit (Soul Fruit) and rank 64 is Blueberry (`IndexMenu.lua:370-371`). Inventory sorts by rank so the best fruit is first (`StoredFruitsMenu.lua:326`).
13. **Client copies of server rules.** The rebirth test exists in RebirthMenu, AlertBadges and the server; the upgrade "maxed" test in UpgradeMenu, AlertBadges, PenUpgradeSign and the server (`UpgradeMenu.lua:9-10`, `AlertBadges.lua:49-51`, `PenUpgradeSign.lua:182-184`). Change them together.
14. **UpgradeMenu's code-drawn card path is dead today.** All three panel keys have art (`GardenConfig.lua:197-211`), and only `plotSize` has `window = true` but it is not a panel key (`GardenConfig.lua:44`, `GardenConfig.lua:225-231`). `paintCard`, `plotIcon` and `ceilingIcon` (`UpgradeMenu.lua:97-293`) never run.
15. **Inventory rebuilds everything** on any count change, one ViewportFrame per card (`StoredFruitsMenu.lua:474-496`). Fine at 5 slots; heavier as storage grows per rebirth.
16. **Index builds 64 cells at load** (about 890 instances and 64 ViewportFrames; 38 ms if done on open) (`IndexMenu.lua:624-637`). Adding ores multiplies this.
17. **The inventory Equip button outline probably does not draw.** The UIStroke on that TextButton keeps the default ApplyStrokeMode and the button's Text is empty (`StoredFruitsMenu.lua:230`, `StoredFruitsMenu.lua:238-241`). ShopMenu and GardenClient set `Border` explicitly for button outlines (`ShopMenu.lua:580`, `GardenClient.lua:386`). [inferring from Roblox UIStroke rules, not checked in Studio]
18. **Stale comment.** CashClient says shards are "spent only on cosmetics" (`CashClient.lua:83-84`), but tools cost shards (`GardenConfig.lua:1619`).
19. **Mute is not saved.** It is a client-set attribute and resets on rejoin (`MuteButton.lua:10-13`).
20. **CashClient writes both counter texts every frame** with no change guard (`CashClient.lua:82-85`). Harmless now; worth guarding if more counters are added.
21. **Modals do not close panels.** Clear, rebirth confirm and offline dialogs sit on top of whatever is open; the Blocker sink is what stops clicks reaching the HUD underneath (`ClearMenu.lua:74-83`).
22. **The rebirth confirm reuses the Clear art**, whose header bakes "ARE YOU SURE?", so its Title line is overwritten with "Rebirth?" (`RebirthMenu.lua:190-193`). New confirm dialogs for big purchases (100k-shard base, boats) can reuse it the same way.
