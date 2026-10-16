---
tags: [system, design]
date: 2026-10-04
status: proposed
depends_on: [Royal Combat, Rarity Gating and Rebirth]
---
# King UI Overhaul

Every King's Throne screen, button, card, overhead title and castle sign, rebuilt from Samarth's own image kit. Sources: [[Brain Dump 2026-10-04]], [[UI Kit Inventory]], [[Client World, Royal Client, Music and Ambient]], [[Royal System - Server]], [[Client UI - HUD and Menus]], [[Consistent Image-Based UI Kits in Roblox]], [[Top-Tier Sword Combat and Animation in Roblox]], [[Roblox Platform Constraints for the Expansion]].

All code citations are in `/Users/samarthvaka/fruit-empire/src-export-2026-10-04`. Short names are used: `RoyalUI.lua` means `StarterPlayer.StarterPlayerScripts.RoyalUI.lua`, `Crown.lua` means `ServerScriptService.GardenService.Crown.lua`, `Builder.lua` means `ServerStorage.ZZTools.RoyalCapitalBuilder.lua`, and so on.

---

## 1. Why

Samarth's words, verbatim:

> install the music and the royal icons now

> make this super planned and tell me exactly what assets you need dont make slop unless it looks clean and studded.

> also all Uis you make should follow the UI that we created, if you need any UI stuff then we can create them using chatgpt or gemini image generation following references of current UI - look at the image in the screenshot all the king stuff in the sky islands doesnt follow the UI i want.

> i want a massive rehual on the UI of all the king stuff btw because it currently goes completely against my UI. if you want i can make everything on figma or you can give me chatgpt prompts and tell me which component in my UI i should use as the reference

> what i noticed with the king feature in the sky island is that it was a bunch of dopamaine slop. instead everything should be structured. like for players its going to be confusing. all the UI should never overlap and also everything should be super clean.

> notice how my Ui font is different from the clauded text you use. i want you to match my font and UI style. I dont like the emoji-slop that you pushed on the sky island kings feature.

> the king feature also showed me that the castle you made looks really AI. to be honest my main issue is with the text font everything else is not bad. preferable use my UI style.

What this system is for:
- It replaces every code-drawn royal box with Samarth's kit images, and removes every emoji.
- It gives each royal element one fixed screen zone, shows fewer things at once, and makes sure nothing overlaps the garden HUD or the toasts.
- It fixes the castle signs, which read as "AI" because of how their text is drawn, not because of the buildings.
- It gives Samarth an exact list of images to make (Figma for anything with words, ChatGPT or Gemini for icons), each tied to a named kit component.

### What is wrong today (read from the code)

**Terms.** *Kit*: Samarth's image-based UI components listed in [[UI Kit Inventory]]. *HUD* (heads-up display): the buttons and counters always on screen. *Emoji*: picture characters such as a crown or crossed swords typed inside text.

![[throne-kingdom-menu.jpg]]
![[royal-gate-shut.jpg]]

1. **The font family is already right; the treatment around it is wrong.** Every royal label already uses Fredoka One: `RoyalUI.lua:43`, `RoyalCombat.lua:36`, and every castle board at `Builder.lua:292`. What makes it look like "clauded text":
   - **Emoji.** 118 code lines put emoji or picture symbols (such as a down arrow) inside strings, across 7 files: `RoyalUI.lua` 51 lines, `CrownConfig.lua` 29, `Builder.lua` 16, `Crown.lua` 10, `RoyalCombat.lua` 7, `CrownDecree.lua` 4, `CrownArena.lua` 1 (counted by a scan of string literals). Emoji are drawn by the device's emoji font, not Fredoka.
   - **Colour.** Gold, brown, cream and pale green text (for example `RoyalUI.lua:325`, `:767`, `:378`; `Builder.lua:1125-1131`). The kit uses white text with a black outline ([[UI Kit Inventory]]).
   - **Thin or missing outlines.** Labels get a 2.5 px outline on text that can grow to 60 px (`RoyalUI.lua:195`, `:201`), about 4% of letter height. The parchment boards and menu lines have no outline at all (`Builder.lua:1129-1131`, `:1296-1301`; `RoyalUI.lua:767`). Samarth's HUD labels have an outline of roughly 10 to 15% of letter height (estimated by eye from `ui-reference-hud-left.png`, not measured).
   - **Titles typed as text.** The kit's panel titles are images, for example "INDEX" is image 103221050515090. Every royal title is live text.
   - **Casing.** Royal text is ALL CAPS ("KINGDOM", "WATCH", "CHALLENGE"). Kit buttons are Title Case ("Inventory", "Merge All", "Lock", "Upgrade").
   - **Sizes jump.** `TextScaled = true` on every royal label (`RoyalUI.lua:180`, `Builder.lua:294`), so two labels of different length render at different sizes ([[Consistent Image-Based UI Kits in Roblox]] section 5).
2. **The boxes are not kit art.** Panels, cards and buttons are flat `Frame` + `UICorner` + `UIStroke` + `UIGradient` built in code (`RoyalUI.lua:134-289`; `RoyalCombat.lua:204-257`). Castle boards are flat colour Frames (`Builder.lua:268-273`).
3. **Real overlaps exist.**
   - The top-centre royal stack starts at y 70 (`RoyalUI.lua:356`). The red warning toast lands at y 76 (`GardenToast.lua:54`). They cover each other.
   - The Royal Trial card sits at bottom centre, 150 px up (`RoyalUI.lua:702`). The green notice toast lands at exactly the same spot (`GardenToast.lua:64`).
   - Announcements drop in at 16% of screen height (`RoyalUI.lua:469`), on top of the duel board and contest card in the same area.
   - The royal menu does not close when the Index, Shop or Inventory opens, and does not close them. Two panels can be open at once.
   - The royal stack can show up to 3 cards at once (decree, duel, contest, `RoyalUI.lua:376`, `:554`, `:596`) plus JOIN and WATCH hanging below.
4. **"Dopamine slop", measured.**
   - A sound plays on every royal button press (`RoyalUI.lua:284`).
   - Big centre text pops from 1.8x size (`RoyalUI.lua:539`).
   - The camera is taken over for 3.6 s when anyone is crowned near the castle (`RoyalUI.lua:1195-1215`).
   - Personal news (achievements) uses the same giant banner as server news (`RoyalUI.lua:1465-1467`).
   - The HUD shows a chip, up to 3 buttons and a court line in the top-left (`RoyalUI.lua:299-346`, `:392`).
5. **One HUD line is false.** "ROYAL COURT, +X% EMPIRE INCOME" (`RoyalUI.lua:427`, `Crown.lua:1098`) presents the friend bonus as a King perk. Every player gets it (`Progression.lua:333`).
6. **The arena scoreboards are never written.** Two boards are built (`Builder.lua:1552-1570`) but no script sets their lines (searched the whole export). They always say "THE ROYAL ARENA".
7. **The royal UI scales on a different curve from the HUD.** RoyalUI clamps 0.55 to 1.15 against 1400x900 (`RoyalUI.lua:119-125`). The HUD uses `Config.HudScaleFor`: 1600x900, power 0.6, 0.85 on touch, 0.55 to 1.0 (`GardenConfig.lua:1949-1957`). On big monitors the royal chip grows while the HUD does not.

---

## 2. Player view

**Terms.** *Chip*: the small bar at the top-left that says who is King. *Pill*: a rounded kit button. *Event strip*: the one card at top centre that shows the royal event happening now. *Herald*: a one-line banner for server news. *Stinger*: a big title image in the middle of the screen during a fight ("FIGHT!"). *Modal*: a panel that opens over the game, such as the Index.

1. **A new player on the island.** Top-left, under Roblox's own buttons, one gold chip says "King" and the King's name, with a small hourglass and the reign time. Nothing else royal is on screen.
2. **They tap the chip.** The Kingdom panel opens in the centre. It looks exactly like the Index: the blue glossy title bar, a "KINGDOM" title image, the red X, the dark body. Four tabs sit on its left edge like the Shop's tabs: Throne, Decrees, Styles, Records. The Throne tab shows the King's avatar on a kit card, their reign, defenses and status. At the bottom is a grey Challenge pill, and under it one red line: "Discover 5 fruits (3/5)".
3. **Later, they are eligible.** A red "!" badge (the kit's alert badge, the same as on Upgrade) appears on the chip. They open the panel and press the red Challenge pill. A green toast at the bottom says "You are #1 in line". The pill turns grey ("In Line #1") and a blue Leave Line pill appears.
4. **The King sees a dialog.** It is the kit's Yes/No dialog, the same art as Clear: "CROWN CHALLENGE" title image, "Ana challenges you for the crown. Accept the duel?", a gold bar draining over 15 seconds, Yes and No. If the King is vulnerable, No is hidden and a red line says "You are vulnerable. You must fight."
5. **The duel.** Both fighters go to the arena. Their garden HUD clears away (Inventory, Index, Shop, Rebirth, Merge All, Clear, the counters, the potion timers). One card at top centre says "Crown Duel", "Ana 0 - 0 Ben", "First to 3". In the middle: 3, 2, 1, then the "FIGHT!" title image. Phone players see six icon buttons around the jump button, each with its word baked in ("Attack", "Block", "Dash", "Slam"). PC players see one row of small plates at the bottom: "LMB  Strike", "F  Block", "Q  Dash", "E  Slam".
6. **A ring-out.** "CROWN POINT" appears in the middle for under a second. The card's score changes. The arena's two big scoreboards show the same score.
7. **The end.** The winner sees "VICTORY" (or the King sees "DEFENDED"); the loser sees "DEFEAT". Everyone else in the server sees one herald line under the event strip for 3 seconds: "Ana took the crown". Then it is gone.
8. **The new King.** Their chip says "You Are King". When a decree is ready, a red "!" sits on the chip. The Decrees tab shows five kit cards (icon, name, one-line rule, reward, time) with green Issue pills. The Styles tab shows six crowns on kit cards, locked ones under a padlock with their requirement in red. The Records tab shows this server's reigns, the Eternal King, the King's own record, and eight achievement medals that light up when earned.
9. **A contest.** Players away from the castle see the event strip change to "Crown Contest, starts in 0:42, 3 joined" with Join and Watch pills inside the card. The garden HUD is never covered.
10. **In the castle.** Every sign uses the kit's dark panel and a title image in the INDEX style ("ROYAL GATE", "THRONE ROOM", "CURRENT RULER"). All other text is white with a thick black outline. The boards inside the hall list reigns and Empire Power in two clean columns.

---

## 3. Rules

**Terms.** *s*: the HUD scale, `Config.HudScaleFor(viewport, touch)` (`GardenConfig.lua:1949-1957`). *Reference*: a 1600x900 screen, where s = 1. *Ladder*: the four fixed text sizes in section 4. *RoyalState*: the object in ReplicatedStorage whose attributes the server writes for the royal HUD (`Crown.lua:62-73`). Rules are proposals until Samarth signs off.

### Look
1. **No emoji.** Every string drawn by RoyalUI, RoyalCombat, the Shop's King tab, every royal toast and every `RoyalBoard` SurfaceGui uses only characters 32 to 126 (plain ASCII). Player display names are exempt (they are user data).
2. **No emoji fallback.** If an icon id in `CrownConfig.Icons` is 0 or missing, that spot shows no icon and its text moves left to fill the gap. It never falls back to an emoji. (This changes [[Decisions Log]] D-001; see Open questions.)
3. **One font.** Every label uses `Font.new("rbxasset://fonts/families/FredokaOne.json")`, as `ShopMenu.lua:47` does.
4. **Three text colours only.** White (255, 255, 255) for all text. Red `C.Colors.Red` (226, 52, 52) for unmet requirements and for timers under 5 s. Green (86, 200, 96, the Shop's Equip green, `ShopMenu.lua:479`) for met requirements. No gold, brown, cream or pale-green text.
5. **Outline on every label.** Black `UIStroke`. Thickness = max(2, 0.12 x TextSize), rounded to the nearest 0.5 px. Same ratio on castle boards, measured in SurfaceGui pixels.
6. **Ladder sizes, no TextScaled.** TextSize is one of Title 30, Heading 24, Body 20, Small 17 (reference px). `TextScaled = false`. Names that might not fit use `TextTruncate = AtEnd`. After scaling, no text renders below 9 px.
7. **Casing.** Title images are UPPER CASE (like INDEX). Buttons, tabs and labels are Title Case ("Challenge", "Leave Line"). Sentences are sentence case and at most 48 characters.
8. **Kit art only.** Every visible panel, card, button and bar is an `ImageLabel` or `ImageButton` using a kit image ([[UI Kit Inventory]]) or a new image from section 7. No visible `Frame` background, no `UICorner`, no `UIGradient` in royal UI. Transparent layout frames are allowed.
9. **Titles are images.** The Kingdom panel title, the challenge dialog title, every stinger and every castle sign title. Until an image id is filled in, the title is Heading-size white text (on the panel it sits on the kit blue title bar, which the inventory allows).
10. **No separator characters.** No middle dot, em dash or multiplication sign. Facts sit in separate labels or separate rows.

### Layout and overlap
11. **Zones.** At reference size (all offsets multiplied by s unless stated):
    - Chip cluster: top-left at (16, 8). At most two things: the chip and one action pill.
    - Event strip: top centre. Its top edge is at **112 px, not scaled**, because the warning toast is not scaled and occupies y 76 to 104 (`GardenToast.lua:54`, 28 px tall). At most one card.
    - Herald: top centre, 8 px under the event strip card, or at 112 px when no card shows. If its bottom edge would come within 8 px of the notice lane (H - 186, rule 14), it takes the card's place at 112 px for its hold time and the card hides, the same way the stinger gives way. On a 667 x 375 phone the herald under the card would span y 173 to 208 and cross the notice toast at y 197 to 225 (found in the [[Integration Review]], conflict C40).
    - Stinger: centred horizontally, vertically between the strip and the notice toast lane (rule 14).
12. **Nothing royal overlaps anything else on the HUD layer.** No royal HUD rectangle intersects a GardenUI HUD cluster, either toast label, or another royal element, on the Phone, Tablet, Desktop and Console emulator presets. A dev-only checker proves it (section 8).
13. **The action pill gives way.** If the action pill's bottom edge would come within 6 px of `GardenUI.InventoryButton`'s top edge (live `AbsolutePosition`), the pill is not shown. Its action appears inside the event strip card instead.
14. **The notice lane stays clear.** No royal element is placed between y = H - 186 and the bottom of the screen in the centre third, except during arena focus (rule 16) when the carry row is hidden anyway. H is the screen height. The notice toast occupies H - 178 to H - 150 (`GardenToast.lua:64`).
15. **One modal at a time.**
    - Opening the Kingdom panel sets `Visible = false` on `ShopMenu`, `IndexMenu`, `StoredFruitsMenu`, `UpgradeMenu`, `RebirthMenu`, and the expansion's panels `PetsMenu` ([[Pets]]), `MineMenu` ([[Underground and Ores]]), `BoatDockMenu` ([[Boats and Swimming]]) and `ArmoryUI` ([[Royal Combat]], browse mode). Each of those already resets its own open flag when hidden from outside (`ShopMenu.lua:640-642`, "same guard the other four carry").
    - When any of those, or `ClearConfirm`, `RebirthConfirm`, `OfflinePopup` or `MaxFruitPopup` ([[Rarity Gating and Rebirth]] U5), becomes visible, the Kingdom panel closes. The full list lives in one place, `UIKit.Modals`, so a new panel is added once (added by the [[Integration Review]]).
    - While any modal is visible, the whole royal HUD layer (chip, strip, herald, stinger) is hidden. A herald line that arrives meanwhile is shown as a green notice toast instead.
16. **Arena focus.** While the local player's `InArena` attribute is true, these GardenUI clusters are hidden: `InventoryButton`, `IndexButton`, `ShopButton`, `PetsButton` ([[Pets]]), `RebirthButton`, `MergeAllButton`, `ClearButton`, `TopButtons`, `CornerHud`, `EffectStack`, `CarryButtons`, `CarryInfo`, `BatSlot`, `BatInfo` ([[Bats]]), `WeaponBar` ([[Weapons and Artillery]]) and `SeaStatus` ([[Boats and Swimming]]), plus the `BatControls` ScreenGui (the touch action slot). Only Mute stays. The royal chip cluster is hidden too. [[Royal Combat]] rule 51 uses this same list (it first kept CornerHud; changed in the [[Integration Review]], conflict C26). Each is restored to its earlier `Visible` value within one frame of `InArena` becoming false, and again on respawn if `InArena` is false.
17. **The challenge dialog and the trial never use the bottom centre.**

### Calm behaviour
18. **Herald.** One line at a time. Hold 3.0 s (2.0 s for `short`). At most 3 lines waiting; when a 4th arrives, the oldest waiting line that is not a crown change is dropped. A line identical to the one showing or any waiting line, within 10 s, is dropped.
19. **Personal news is a toast.** Achievements, style unlocks, new titles, queue position, decree rewards and "nobody answered your trial" go to the green notice toast, never the herald.
20. **Who hears what.** Crown changes and contest announcements go to everyone. Sound plays only for the King, the challenger, and players within 260 studs of the `CrownDisplay` marker (`C.EmphasisRange`, `CrownConfig.lua:360`) or 220 studs of `ArenaCenter` (`RoyalUI.lua:1114`). At most one herald sound per 4 s per player.
21. **No click sounds.** The royal Sparkle on every press (`RoyalUI.lua:284`) is removed. Button feel comes from ButtonAnimator, like every garden button.
22. **Stingers only for fighters and watchers.** Shown when `InArena` is true or the player is within 220 studs of `ArenaCenter` (the existing `nearArena`, `RoyalUI.lua:1107-1115`). "NEW KING" shows only to the new King. Motion: fade in 0.12 s while scaling from 1.12 to 1.0 over 0.18 s; never larger than 1.12.
23. **No camera takeover.** `C.CameraEmphasis` defaults to false (`CrownConfig.lua:359`).
24. **Event strip priority.**
    - In the arena or within 220 studs of `ArenaCenter`: Duel, then Contest, then Trial, then Decree.
    - Anywhere else: Contest while it is in its join window, then Trial (only if `RoyalEligible`), then Decree, then Duel, then Contest in later phases.
    - Other systems may add cards through the shared slot (section 8), each with a stated priority.

### Chip and action pill
25. **Chip states** are exactly the three in section 5. Badge priority: Vulnerable, then Protected (countdown), then reign time.
26. **Alert badge on the chip** shows when:
    - not King: `RoyalEligible` is true, `RoyalBusy` is empty, and `ProtectedUntil` is in the past; or
    - King: `DecreeKey` is empty and `NextDecreeAt` is in the past.
27. **Tapping the chip** opens the Kingdom panel on the Throne tab.
28. **Action pill**, one at a time, in this order: Watch (a duel or contest is on, `GateOpen` is true, not `InArena`), Join (contest join window, not King, not joined), Answer (trial open, eligible, not King). There is no Challenge or Kingdom button on the HUD any more; both live in the panel.

### Kingdom panel
29. **Four tabs**: Throne, Decrees, Styles, Records. It opens on Throne, except: the server's `openMenu` event for the Decree Desk opens Decrees and for the Style Mirror opens Styles (`Crown.lua:1005`, `:1012`); sitting on the throne opens Throne.
30. **Throne tab actions by role** are exactly the table in section 5. Leave Line sends the existing `cancelChallenge` request (`CrownArena.lua:1025-1030`), which no button sends today.
31. **Query cache and limit.** The client caches each `RoyalQuery` answer for 5 s. The server answers at most 6 `RoyalQuery` calls per player in any 2 s window and returns nil above that.
32. **Gamepad.** Every pill, card and tab is `Selectable`. `SelectionImageObject` is the kit's card selection outline (113285622864672). On open with a gamepad, the selected tab gets focus.

### World
33. **Castle boards.** Every part tagged `RoyalBoard` keeps its SurfaceGui named `Board`, a child named `Back`, and its line labels with their current names, so `setLine` (`Crown.lua:576-598`) keeps working. `Back` becomes an `ImageLabel` using the kit's dark panel body (82827190351667), 9-sliced. The title becomes an `ImageLabel` named `TitleImage` (fallback text label `Title`). Lines are white, outlined per rule 5, `TextScaled = false`, TextSize = 0.7 x the line's pixel height.
34. **Arena scoreboards are live.** During a bout: Status = the bout title, Names = "Ana  vs  Ben", Score = "2 - 1", Detail = "First to 3" or "Sudden death". When idle: Names = "Challenge the King at the altar", Score and Detail empty.
35. **King title over the head.** Crown icon plus the title in white, the display name under it, `MaxDistance` 100. Hidden while the King is invisible (existing rule, `RoyalUI.lua:972-975`).
36. **No court line.** The "Royal Court, +X% income" text is removed from the chip, the panel and the Current Ruler board (it described `Config.FriendBonus`, which every player gets).

### Combat controls (visuals only; [[Royal Combat]] owns what the buttons do)
37. **Touch buttons** are icon images with their word baked in (section 7), at the sizes and positions `RoyalCombat.lua:284-315` already computes. Cooldown shows as a dark overlay that shrinks from top to bottom. The charge ring is a ring image, gold, turning orange (255, 120, 40) when full (`RoyalCombat.lua:358`).
38. **PC legend.** One row of kit name plates (`CarryInfo.NameBar`, 123470450245311) reading "LMB  Strike", "Hold  Heavy", "F  Block", "Q  Dash", "E  Slam" (King in a contest: "E  Wave", "R  Guard", "T  Knights"). Bottom centre, 16 px up (plates 30 tall, so 16 to 46 px). It replaces [[Royal Combat]]'s text hint line. Royal Combat's ability bar (cooldown discs, 64 tall) sits above it at 54 px up (54 to 118 px), so the two never touch. Only while `InArena` and not touch.

### Shop King tab and KingClient
39. **Shop King tab.** The aura card's Equip button is the green kit pill; Equipped is the grey pill. The code-drawn pill (`ShopMenu.lua:567-583`) is removed.
40. **KingClient** keeps drawing only the King's aura, unchanged. Its dead floating-crown code is removed, its header is corrected, and it also returns early when `CrownConfig.Enabled` is false.

### Words on screen (the full CrownConfig.Text rewrite)

Proposal. Replaces `CrownConfig.lua:417-477`. All ASCII, Title or sentence case, %s = display name. Handlers stop calling `:upper()` on names (`RoyalUI.lua:1305-1311`, `CrownDecree.lua:153`, `:249`).

| Key | Today | New |
|---|---|---|
| NowKing | crown emoji + "%s IS NOW KING!" | "%s is now King" |
| ClaimedByWealth | crown + "%s HAS CLAIMED THE CROWN!" | "%s claims the crown" |
| ClaimedSub | "The strongest empire in the server" | "Highest income in the server" |
| TookCrown | crown + "%s HAS TAKEN THE CROWN!" | "%s took the crown" |
| Conquered | crown + "%s HAS CONQUERED THE EMPIRE!" | "%s won the Crown Contest" |
| Defended | shield + "THE KING HAS DEFENDED THE THRONE!" | "The King defended the throne" |
| Fallen | crown + "THE KING HAS FALLEN!" | "The King has fallen" |
| ThroneEmpty | "THE THRONE IS EMPTY" | "The throne is empty" |
| ThroneEmptySub | "The strongest empire will claim it" | "The highest income claims it next" |
| KingLeft | crown + "THE KING HAS LEFT THE EMPIRE" | "The King left the server" |
| Challenged | swords + "%s HAS CHALLENGED THE KING!" | "%s challenged the King" |
| ChallengePrompt | "%s has challenged you for the crown." | "%s challenges you for the crown. Accept the duel?" |
| Declined | "The King declined the challenge." | "The King declined the challenge" |
| Refused | warning + "THE KING HAS REFUSED..." | "The King refused a challenge" |
| Vulnerable | "The King is CROWN VULNERABLE" | "The King is vulnerable" |
| MustFight (new) | (inline rich text, `RoyalUI.lua:1353`) | "You are vulnerable. You must fight." |
| RefusedContest | warning + "...A CROWN CONTEST BEGINS!" | "The King refused again. A Crown Contest begins" |
| OpenToChallengers | crown + "THE KINGDOM IS OPEN TO CHALLENGERS" | "The King can be challenged now" |
| TrialCalled (new) | crown + "THE KING CALLS FOR A ROYAL TRIAL!" (`CrownArena.lua:1039`) | "The King calls a Royal Trial" |
| TrialSub (new) | "The first worthy challenger fights for the crown" | "First eligible player to answer fights" |
| DuelTitle | "CROWN CHALLENGE" | "Crown Duel" |
| TrialTitle / SparTitle / FinalTitle | "ROYAL TRIAL" / "PRACTICE BOUT" / "THE FINAL DUEL" | "Royal Trial" / "Practice Bout" / "Final Duel" |
| Fight, CrownPoint, CrownIsYours, YouDefended, YouLost, SuddenDeath | text with emoji | title images (section 7); text fallback "Fight!", "Crown Point", "New King", "Defended", "Defeat", "Sudden Death" |
| ContestSoon | swords + "...IN %d SECONDS!" | "Crown Contest in %d seconds" |
| ContestBegins | swords + "THE CROWN CONTEST BEGINS!" | "The Crown Contest begins" |
| CrownAppears | crown + "THE CROWN APPEARS -- TAKE THE ZONE!" | "The zone is open" |
| LostControl / HoldsEmpire | crown + caps | "The King lost control" / "The King holds the throne" |
| GuardSummoned | shield + caps | "The King called the Royal Guard" |
| ContestOff | "...called off -- no challengers came." | "Contest called off. Nobody joined." |
| ClaimTheThrone | crown + "CLAIM THE THRONE!" | "Claim the Throne" |
| DecreeIssued / DecreeDone / DecreeFailed | scroll + caps | "Royal Decree: %s" / "Decree complete: %s" / "The decree was not completed" |
| NeedDiscovered | "Discover %d fruits to challenge (%d/%d)" | "Discover %d fruits (%d/%d)" |
| NeedTime / OnCooldown | "You can challenge in %s" / "...again in %s" | "Wait %s in this server" / "Challenge again in %s" |
| Queued | "You are #%d in line to challenge the King" | "You are #%d in line" |
| GateClosed / GateOpenSign / GateShutSign | long sentences | "The Royal Gate opens for duels and contests" / "Open: walk through to watch" / "Opens for duels and contests" |
| Underdog | "UNDERDOG MOMENTUM" | "Underdog" |
| (others) | unchanged words, emoji removed, sentence case | |

Also in Title Case: `C.Titles` ("King", "High King", "Emperor", "Sovereign", `CrownConfig.lua:283-288`), achievement titles (`:328-337`), decree titles (`:209-254`, whose `icon` emoji fields are deleted).

---

## 4. Numbers

This note sets **no money or shard amounts**. Its numbers are sizes, times and limits. The stage-tied tables below are the places where what the UI shows depends on the [[Stage Model]].

### 4.1 Stage-tied: how wide money values get (Empire Power column)

*Empire Power* is a player's income per second, read from the new `EmpirePower` attribute ([[Explosives and Fruit Health]] code plan: the same income as `CashPerSecond`, including pets and the fruit-only Ore Bonus, but counting stunned fruit so a nuke cannot swing the throne). Today it is `CashPerSecond` (`Crown.lua:296-298`). It shows on the Empire Power board and the Throne tab.

Formula: `text = "$" .. Economy.Format(P) .. "/s"` (`FruitEconomy.lua:65-84`). Highest likely P in a stage = full-plot $/s x 1.355 (pets 25% plus ores 10.5% at the plateau, D-018, [[Stage Model]] rule 6) x 1.5 (full Index bonus, `economy.py` `INDEX_BONUS_TOTAL = 0.50`).

| Stage | Rarity | Full-plot $/s | Shown at full plot | Highest likely (x2.03) | Characters |
|---|---|---|---|---|---|
| **1** | Common | 707 | $707/s | $1.4K/s | 6 to 7 |
| 2 | Uncommon | 430K | $430K/s | $870K/s | 7 |
| **3** | Rare | 3.7B | $3.7B/s | $7.6B/s | 7 |
| 4 | Epic | 3.6T | $3.6T/s | $7.4T/s | 7 |
| 5 | Legendary | 1.8Qa | $1.8Qa/s | $3.6Qa/s | 8 |
| **6** | Mythic | 2.5Qi | $2.5Qi/s | $5Qi/s | 6 to 8 |
| 7 | Astral | 265Qi | $265Qi/s | $536Qi/s | 8 |
| 8 | Eternal | 506Sx | $506Sx/s | $1Sp/s | 6 to 8 |
| **9** | Transcendent | 24.9Sp | $24.9Sp/s | $50.4Sp/s | 9 |

Rule from this: the value column is as wide as the widest string `Economy.Format` can make, "$99.9Qa/s" (9 characters), measured once with `TextService:GetTextSize` at Body 20 px in Fredoka One. The column never resizes as a player moves through stages.

### 4.2 Stage-tied: royal rewards (owned by [[Royal Combat]])

The decree cards and the herald show shard rewards. Today they are flat (`CrownConfig.lua:261-268`): Defense 500, Victory 1,000 and Contest win 3,000 are 167, 333 and 1,000 frontier merges at stage 1 and 0.3, 0.6 and 1.8 at stage 9. The [[Integration Review]] (conflict C27, decision D-017) settled on [[Royal Combat]]'s units: reward = `Config.ShardReward(units, rebirths)` = units x shards per frontier merge of the receiving player's stage. That also covers decrees (Merge Rush 10, Royal Race 25 / 15 / 8, Crown Hunt 20, Empire Build 10 units), which [[Royal Combat]] now owns.

| Stage | Rarity | Shards per frontier merge | Defense (10) | Victory (25) | Contest win (40) | Merge Rush (10) | Royal Race 1st (25) |
|---|---|---|---|---|---|---|---|
| 1 | Common | 3 | 30 | 75 | 120 | 30 | 75 |
| 3 | Rare | 15 | 150 | 375 | 600 | 150 | 375 |
| 6 | Mythic | 155 | 1,550 | 3,875 | 6,200 | 1,550 | 3,875 |
| 9 | Transcendent | 1,646 | 16,460 | 41,150 | 65,840 | 16,460 | 41,150 |

Each is 2 to 18 minutes of frontier merging at every stage ([[Integration Check]]). What the UI requires: **the server sends the reward number for each viewer**, and the UI never hard-codes an amount (rule in section 8.3). The earlier proposal in this section (k = 33 / 67 / 200, anchored at stage 3) is withdrawn.

### 4.3 Stage-tied: who can challenge (the red requirement line)

Today `worthy` needs 1+ rebirths or `C.MinDiscovered` = 10 Index fruits (`Crown.lua:314-323`, `CrownConfig.lua:38`).

| Stage | Rebirths | Fruits a player can discover | Eligible today? | With MinDiscovered = 5 (proposal) | Red line shown until eligible |
|---|---|---|---|---|---|
| **1** | 0 | 5 (Blueberry to Apple) | Never (5 < 10) | When all 5 are found | "Discover 5 fruits (n/5)" |
| **3** | 2 | 24 | Yes (rebirths) | Yes | none, unless time or cooldown |
| **6** | 5 | 47 | Yes | Yes | same |
| **9** | 8 | 64 | Yes | Yes | same |

[[Rarity Gating and Rebirth]] owns and makes this change (its code plan, decision D-026). The UI only prints `RoyalWhy` (`Crown.lua:1173`).

### 4.4 Layout (reference 1600x900; multiply by s unless stated)

| Element | Where | Size (px) | Notes |
|---|---|---|---|
| Chip | top-left, (16, 8) | 286x62 | new image A6 / A7 |
| Chip icon | inside chip at (5, 5) | 52x52 | `C.Icons.Crown` |
| Chip line 1 (title) | (64, 7) | 140x20, Small | e.g. "High King" |
| Chip line 2 (name) | (64, 28) | 140x26, Body, truncate | display name |
| Chip status badge | right side, anchored (1, 0) at (-8, 8) | 80x22 | icon 20 + Small text |
| Alert badge | centred on the chip's top-right corner | 31x35 (kit native) | kit 79539007098845 |
| Action pill | (16, 78) | 180x44 | new image A1 to A5 |
| Event strip card | top centre, top edge y 112 (not scaled) | 560x96 | kit panel body, 9-sliced |
| Card title strip | top of card | 560x28 | kit blue title bar, 9-sliced, Small text |
| Card pills (inside) | right side of card | 144x36 each, 8 gap | small pill |
| Herald | top centre, 8 under the card | 640x64 | kit blue title bar, Heading + Small |
| Stinger | centre x; y per rule 11 / 14 | 120 tall, width from image | title images |
| Kingdom panel | centre | Scale 0.6 x 0.6, aspect 1.775, max 948x534 | cloned from the Index shell |
| Panel tabs | 8 left of the panel | 100x132 each, 8 gap | kit Shop tab 110917091182913 |
| Challenge dialog | centre | cloned `GardenUI.ClearConfirm` | kit confirm 112127226387884 |
| Touch combat buttons | around the jump button | 128 / 92 / 88 / 92 / 78 / 78 | unchanged layout `RoyalCombat.lua:306-311` |
| PC legend | bottom centre, 16 up | plates 140x30, 8 gap | kit name bar; [[Royal Combat]]'s ability bar sits above it at 54 up |
| King title billboard | head + 3.4 studs | 220x60 | `MaxDistance` 100 |

Worked overlap check for the chip cluster (Inventory button centre is 181 px above screen middle and about 92 tall, [[UI Kit Inventory]]):

| Screen | s | Inventory top edge | Chip cluster bottom (chip + pill) | Gap | Pill shown? |
|---|---|---|---|---|---|
| Desktop 1600x900 | 1.00 | 450 - 181 - 46 = 223 | (8 + 62 + 8 + 44) = 122 | 101 | yes |
| Tablet 1024x768 (touch) | 0.65 | 384 - 118 - 30 = 236 | 79 | 157 | yes |
| Phone 844x390 (touch) | 0.55 | 195 - 100 - 25 = 70 | 67 | 3 | **no** (rule 13), chip bottom 39 |

Phone stinger check: strip 112 to 165, notice lane 212 to 240. Free band 165 to 204 is 39 px, smaller than a 66 px stinger. So on screens where the band is too small, the stinger hides the strip card for its hold time and centres in 112 to 204 (rule 11).

### 4.5 Text ladder

| Step | TextSize (ref) | Outline px (rule 5) | Smallest on a phone (x0.55) | Used for |
|---|---|---|---|---|
| Title | 30 | 3.5 | 16.5 | countdown digits fallback, panel title fallback |
| Heading | 24 | 3 | 13.2 | herald line, King name on Throne tab, card titles in panel |
| Body | 20 | 2.5 | 11 | chip name, stat rows, card text |
| Small | 17 | 2 | 9.35 | chip title line, badges, requirement lines, tab labels |

### 4.6 Timing

| What | Value | Source |
|---|---|---|
| Panel open / close | 0.22 s Back Out from scale 0.85 / 0.14 s Quad In | same as the garden panels, `IndexMenu.lua:456-457` |
| Button hover / press / release | 1.08 / 0.92; 0.15 / 0.07 / 0.28 s | ButtonAnimator, `ButtonAnimator.lua:28-34` |
| Herald in / hold / out | slide 8 px + fade 0.2 s Quad Out / 3.0 s (short 2.0 s) / fade 0.2 s | proposal (today 0.35 s Back, 3.4 s, `RoyalUI.lua:496-499`) |
| Herald queue / dedupe | 3 waiting / 10 s | proposal |
| Herald sound gap | 4 s minimum | proposal |
| Stinger | in 0.12 s, scale 1.12 to 1.0 over 0.18 s; holds: digit 0.6, Fight 0.8, Crown Point 0.9, result 1.6 to 1.8; out 0.2 s | holds as today `RoyalUI.lua:1374-1400` |
| Event card swap | crossfade 0.15 s | proposal |
| Chip / strip refresh | every 0.5 s, text written only if changed | today's loop `RoyalUI.lua:1521-1533` |
| Fighter tag refresh | every 0.1 s | today every frame, `RoyalUI.lua:1535-1536` |
| Query cache / server limit | 5 s / 6 per 2 s | proposal |
| Challenge answer time | 15 s | `C.ResponseTime`, `CrownConfig.lua:55` |

### 4.7 Layers (ScreenGui DisplayOrder)

| DisplayOrder | ScreenGui | Holds |
|---|---|---|
| 0 | GardenUI (today, `LoadingScreen.lua:58` comment) | garden HUD, garden panels, toasts |
| 1 | RoyalHud (was RoyalUI at 6, `RoyalUI.lua:114`) | chip, action pill, event strip, herald, stinger |
| 2 | RoyalCombat (was 7, `RoyalCombat.lua:172`) | touch buttons, charge ring, PC legend, ability bar |
| 3 | BatControls ([[Bats]] UI 3) | the touch action slot: Bat, Swing or weapon Fire; hidden `InArena` |
| 20 | RoyalPanel, ArmoryUI | the Kingdom panel and its blocker; [[Royal Combat]]'s Armory panel (pick and browse modes). One modal at a time (rule 15) |
| 30 | RoyalConfirm | the challenge dialog |
| 100 / 1000 | DevPanel / LoadingScreen (today) | unchanged |

All royal ScreenGuis: `ResetOnSpawn = false`, `IgnoreGuiInset = false`, `ScreenInsets = CoreUISafeInsets`, `ZIndexBehavior = Sibling`.

---

## 5. UI

### 5.1 Every current royal element and its replacement

| # | Today | Where | Replacement (section) |
|---|---|---|---|
| 1 | King chip: code-drawn green box, crown emoji, gold title text | `RoyalUI.lua:311-327`, `:400-431` | Chip image A6/A7, icon, white text (5.2) |
| 2 | CHALLENGE / KINGDOM / WATCH buttons under the chip | `RoyalUI.lua:329-346` | Removed; one action pill (5.2); Challenge and Kingdom move into the panel |
| 3 | Royal Court line | `RoyalUI.lua:392-393`, `:426-428` | Removed (rule 36) |
| 4 | Decree card | `RoyalUI.lua:376-390`, `:434-461` | Event strip, Decree card (5.3) |
| 5 | Duel scoreboard | `RoyalUI.lua:554-592` | Event strip, Duel card (5.3) |
| 6 | Contest card with JOIN / WATCH hanging below | `RoyalUI.lua:596-667` | Event strip, Contest card, pills inside (5.3) |
| 7 | Royal Trial call card (bottom centre) | `RoyalUI.lua:702-709` | Event strip, Trial card (5.3) |
| 8 | Announcement cards (5 colour styles) | `RoyalUI.lua:465-523` | Herald (5.4) |
| 9 | Big centre text | `RoyalUI.lua:527-550` | Stinger title images (5.5) |
| 10 | Challenge card for the King | `RoyalUI.lua:671-699` | Challenge dialog (5.7) |
| 11 | Cream menu window: Kingdom, Status, Styles, Decrees | `RoyalUI.lua:713-963` | Kingdom panel with 4 tabs (5.6) |
| 12 | Title over the King's head | `RoyalUI.lua:977-1002` | Restyled billboard (5.8) |
| 13 | Stagger tag and outline over fighters | `RoyalUI.lua:1006-1049` | Kit bar billboard; content from [[Royal Combat]] (5.8) |
| 14 | PC key hint line | `RoyalCombat.lua:197`, `:335-336` | Legend of name plates (5.9) |
| 15 | Six round touch buttons with emoji | `RoyalCombat.lua:204-264`, `:343` | Icon buttons with baked words (5.9) |
| 16 | Charge ring (stroke on a Frame) | `RoyalCombat.lua:267-281` | Ring image A10 (5.9) |
| 17 | Royal toasts with emoji | `Crown.lua:207`, `:211`, `:228`, `:232`; `CrownDecree.lua:93`, `:130` | Plain text through GardenToast |
| 18 | Shop King tab Equip pill | `ShopMenu.lua:567-583` | Kit pills (5.10) |
| 19 | 28 castle boards and 2 portraits | `Builder.lua` (table in 5.11) | Kit boards (5.11) |
| 20 | KingClient aura | `KingClient.lua` | No UI; tidy only (rule 40) |

### 5.2 HUD: chip and action pill (zone: top-left)

Components: chip image A6 (gold) or A7 (stone), icon from `C.Icons`, kit alert badge 79539007098845, pills A1 to A5.

| State | Chip image | Icon | Line 1 (Small) | Line 2 (Body) | Status badge |
|---|---|---|---|---|---|
| Throne empty | A7 stone | Crown at 50% transparency | "Throne" | "Empty" | none |
| Someone else rules | A6 gold | Crown | their title, e.g. "High King" | their display name | Vulnerable (alert icon, red "Vulnerable"), or Shield + "0:42", or hourglass (`Reign`) + "12:04" |
| You rule | A6 gold | Crown | "You Are " + title | your display name | as above |

- Alert badge: rule 26.
- If an icon id is 0, the badge shows the word only ("Protected 0:42", "Reign 12:04").
- Action pill states: Watch (blue A2, `Watch` icon), Join (red A3, `Swords` icon), Answer (red A3, `Trial` icon). Rule 28. Pressing sends the same requests as today: `watch`, `joinContest`, `joinTrial` (`RoyalUI.lua:346`, `:615`, `:708`).
- Overlap: the chip ends at x 302 at reference. The Lock pill starts at x 592 (800 - 208). The pill obeys rule 13. The cluster hides while a modal is open (rule 15) and in the arena (rule 16).

### 5.3 Event strip (zone: top centre, below the warning toast)

One card, 560x96, built from the kit panel body (9-sliced) with the kit blue title bar (9-sliced) as its 28 px header. Rows: header (Small, white), main row 40 px, sub row 20 px. Pills sit inside the card on the right.

| Card | Header | Main row | Sub row | Pills inside |
|---|---|---|---|---|
| Duel | bout title ("Crown Duel", "Royal Trial", "Practice Bout", "Final Duel") | Name A (crown icon if King), "2 - 1" (Title size), Name B | "First to 3" or "Sudden death"; "Underdog" under the trailing name when `DuelUnderdog` is set | Watch, if not in the arena and the gate is open |
| Duel with health ([[Royal Combat]]) | same | same | two kit bars (A8 track, A9 fill), one per fighter, 196 x 20 each, fill banded like every HP bar: above 50% Uncommon green, 25% to 50% Legendary gold, under 25% Mythic red (D-027) | same |
| Contest, join window | "Crown Contest" | "Starts in 0:42" | "3 joined"; King sees "You defend the zone" | Join (not King, not joined), Watch |
| Contest, brawl | "Crown Contest" | "Brawl: zone opens in 7" | "Defending: " + King name, or "No King: hold the zone alone" | Watch |
| Contest, zone phase | "Kingdom Control" or "Claim the Throne" | kit bar, gold fill (King holds) or red fill (challengers), value text on it ("64%" or "Your claim 40%") | zone state ("King holds the zone", "Challengers take the zone", "Contested", "Zone empty") and "You: 23" | Watch |
| Royal Trial | "Royal Trial" | "First eligible player to answer fights" | kit bar draining to `TrialEnds` | Answer |
| Decree | decree title (icon at the left of the header if it exists) | kit bar, gold fill | "14 / 25 merges" or the decree's text, and time left "1:42" | none |

- Data: the same RoyalState attributes as today (`RoyalUI.lua:435-459`, `:577-591`, `:628-665`).
- Priority: rule 24.
- Overlap: top edge fixed at 112 px (rule 11). The card is hidden while any modal is open (rule 15) and gives way to the stinger on short screens (section 4.4).

### 5.4 Herald (zone: top centre, under the strip)

- The kit blue title bar (95532041481396), 9-sliced to 640x64. Main line Heading, optional sub line Small. Optional icon at the left from the event's `icon` key.
- Server events keep their `style` field (`crown`, `royal`, `alarm`, `decree`, `plain`, server note "change both sides together"). `style` now picks only the sound, never a background colour.
- Scope and queue: rules 18 to 20.
- Overlap: placed under the strip card, or at 112 px. Hidden while any modal is open (the line goes to a toast instead).

### 5.5 Stingers (zone: centre band, only near or in the arena)

- Title images T3 to T9 (section 7). Height 120 at reference.
- Countdown digits are Title-ladder text at 4x size (TextSize 96 inside the stinger frame), white, outline per rule 5 (11.5 px).
- Placement: centred in the band from the strip's bottom + 8 to H - 186. If the band is shorter than the stinger plus 16 px, the strip card hides for the stinger's hold time.

### 5.6 Kingdom panel (zone: centre modal)

Shell: a clone of the Index panel's frame pieces, kept in `ReplicatedStorage.UIKitPrefabs.Panel` (section 8): title frame 110138832714236, blue title bar 95532041481396, title image T1 "KINGDOM" in place of "INDEX", dark body 82827190351667, inline frame 110403403749726, red X (square) 107111219044453 at the right of the title bar. A full-screen transparent blocker one ZIndex below it, the same pattern as `ClearMenu.lua:84-98`.

Tabs: four kit Shop tabs (110917091182913) outside the left edge. Each tab: icon (`Throne`, `Decree`, `Crown`, `Status`) on the top 60%, label (Small) underneath: "Throne", "Decrees", "Styles", "Records". Inactive tabs are tinted with `Config.ShopTabDim` (190, 182, 172), exactly as the Shop dims its tabs.

**Throne tab**
- Left: a kit item card (99120915117089) holding the King's avatar (`rbxthumb://type=AvatarBust&id=<userId>&w=150&h=150`, the same thumbnail kind as `Crown.lua:614-616`), the name (Heading) and the title (Body). Empty throne: the crown icon at 50% and "The throne is empty".
- Right: the kit scroll box (83403190365135) with one fact per row (icon, label left, value right): Reign, Defenses, Crowned (times), Duels won, Status (Protected 0:42 / Vulnerable / Can be challenged), Line ("2 waiting", from `ChallengeQueue`).
- Bottom: an action row of up to two pills, then one Small line.

| You are | Pills | Line under them |
|---|---|---|
| Not King, eligible, not in line | Challenge (red, `Swords`) | none |
| Not King, in line | "In Line #n" (grey, disabled), Leave Line (blue) | none |
| Not King, not eligible | Challenge (grey, disabled) | `RoyalWhy` in red, e.g. "Discover 5 fruits (3/5)" |
| King | Royal Trial (red, `Trial`), Passage (blue, `Passage`) | "Passage: between your garden and the throne room" |
| Anyone, while a duel or contest is on and not in the arena | Watch (blue) added | none |

**Decrees tab**
- Top line (Body): King: "Ready", "Next decree in 4:12" or "A decree is running". Not King: "Only the King can issue decrees".
- Grid of 5 kit item cards, 3 columns, in the kit scroll box. Each card: decree icon (66x66), name (Heading), one-line rule (Small, 2 lines max), "Reward: N shards" or "Top 3: a / b / c shards" (Small; N comes from the server per viewer, section 8.3), duration "3:00" (Small), and an Issue pill (green). When cooling down or not King: grey pill, disabled.

**Styles tab**
- Top: "Royal Reputation 340" and the title, with a kit bar (A8/A9, gold) to the next title (`C.Titles` thresholds 250 / 1000 / 3000).
- Grid of 6 kit item cards (3x2): crown icon (`Golden` ... `Soul`), name (Heading).
  - Locked: padlock icon (`Lock`) over the crown at 60%, the requirement in red ("300 Royal Reputation", "Achievement: Conqueror"), no pill.
  - Unlocked: Wear pill (green).
  - Worn: Worn pill (grey) and the kit card selection outline (113285622864672) around the card.
- Footer: "Royal auras are sold in the Shop" and an Open Shop pill (blue). The pill sets the client-only attribute `OpenShopTab = "King"` on GardenUI (optional ShopMenu hook, section 8).

**Records tab**
- Left, kit scroll box: "This server" reigns (up to 8 rows: rank, name, time, defenses) and the Eternal King row (name, time). From `RoyalQuery("history")` (`Crown.lua:928-934`).
- Right, kit info box (101725900357985): "Your record": crowned, defenses, duels won, contests won, longest reign, total reign, reputation. From `RoyalQuery("me")` (`Crown.lua:935-947`).
- Bottom: 8 achievement cells (96x96 each, 12 gap). Medal icon, full colour when earned and 55% transparent, grey-tinted when not; a kit bar under each showing `progress / need` (the `me` answer already sends `progress`, `Crown.lua:941-943`). Tapping a cell shows its title and one-line description in the info box.

### 5.7 Challenge dialog (zone: centre, confirm layer)

- A clone of `GardenUI.ClearConfirm` (dimmer, kit confirm panel 112127226387884, Title, Body, YesButton 71592791133553, NoButton 105863640938118). The same art the Clear and Rebirth dialogs use.
- Title: image T2 "CROWN CHALLENGE" (fallback: Heading text "Crown Challenge").
- Body (Body size): `C.Text.ChallengePrompt` with the challenger's name. If `canDecline` is false, a red Small line `C.Text.MustFight` and the No button is hidden.
- A kit bar (A8/A9, gold) drains from full to empty over `expires - now` (today's Heartbeat logic, `RoyalUI.lua:1537-1541`).
- Yes sends `respond, true`; No sends `respond, false` (`RoyalUI.lua:693`, `:698`). Opening it closes the Kingdom panel (as today, `RoyalUI.lua:1356`).
- The blade picker is not in this layer: [[Royal Combat]]'s Armory panel (an Index-shell panel, ArmoryUI at DisplayOrder 20) opens during its 10 s pick phase before the title card. This dialog and the Armory never show together: the Armory closes when a challenge prompt arrives.

### 5.8 World billboards

- **King title**: `BillboardGui` 220x60, `StudsOffset` (0, 3.4, 0), `MaxDistance` 100 (today 140), `LightInfluence` 0. Row 1: crown icon 24x24 + title (Heading, white). Row 2: display name (Body). Text-only if the icon id is 0.
- **Fighter tags**: a `BillboardGui` 110x26 with a kit bar (A8 track, A9 fill) and a Small number. [[Royal Combat]] decides whether it shows health or stagger. Outline highlights for Crown Guard and Block stay as today (`RoyalUI.lua:1034-1042`).

### 5.9 Combat controls

- Touch: images F1 to F7 (icon plus baked word) on `ImageButton`s, same diameters and positions as today. The Ability button swaps image between Slam (F4) and Wave (F5) when the King is in a contest (today `RoyalCombat.lua:343` swaps emoji text). Guard (F6) and Knights (F7) show only for the King in a contest (today `RoyalCombat.lua:339`).
- Cooldown: a copy of the button image tinted black at 45% transparency, inside a clipping frame whose height shrinks from 100% to 0 over the cooldown.
- Charge ring: A10, `ImageColor3` gold, transparency 1 - 0.9 t as today (`RoyalCombat.lua:355-361`).
- PC legend: rule 38.
- The touch ScreenGui gets the `KitButtons` attribute so ButtonAnimator gives the same press feel; the manual press shrink at `RoyalCombat.lua:242-253` is removed so the two do not stack.

### 5.10 Shop King tab

- Equip: green pill A1 with "Equip" (Body). Equipped: grey pill A5 with "Equipped". Buy stays the kit buy button (`ShopMenu.lua:549-562`).
- Optional hook: ShopMenu listens for the GardenUI attribute `OpenShopTab`, opens and selects that tab, then clears the attribute.

### 5.11 Castle boards (the "AI-looking" signs)

Every board: the kit dark panel body as `Back` (9-sliced), a title image at the top, white Fredoka lines with outlines per rule 5, fixed text sizes, no emoji. Line names stay, so the server's writes keep working.

| Board (attribute `Board`) | Builder line | Title image | Lines (server writes in bold) | New static text |
|---|---|---|---|---|
| Altar | `Builder.lua:716-722` | W5 CHALLENGE THE KING | Caption, **King**, **Detail**, Hint | Caption "Current King"; Hint "Hold E at the altar to challenge" |
| ReturnSign | `:768-770` | W4 RETURN HOME | Text (hidden when the image exists) | none |
| Signpost x2 | `:778-785` | W2 ROYAL ARENA / W3 THRONE ROOM | Text (hidden when the image exists) | none |
| GatePlaque | `:799-802` | W6 KING'S CASTLE | **King** | none |
| KingPlate / EternalPlate | `:1107-1113` | W13 THE KING / W14 ETERNAL KING | **Name** | none |
| History | `:1135` | W9 HISTORY | **Row1-8**, **Foot** | none |
| Power | `:1136` | W10 EMPIRE POWER | **Row1-8** split into name (left) and **Row1Value-8Value** (right, fixed width, section 4.1), **Foot** | Foot "Income per second" |
| Ruler | `:1147-1154` | W7 CURRENT RULER | **King**, **Rank**, **Reign**, **Defenses**; Court removed | none |
| Records | `:1155-1163` | W8 ROYAL RECORDS | Eternal, **EternalName**, **EternalTime**, **Row1-3** | Eternal "Eternal King" |
| VaultPlate x6 | `:1269` | none | Text | style name in Title Case ("Golden") |
| ReignRecords | `:1294-1301` | W11 YOUR REIGN | **Row1-8**, each one fact, with **RowNValue** | none |
| StatuePlate x4 | `:1453` | none | Text | fruit name in Title Case ("Golden Apple") |
| ArenaSign | `:1528` | W2 ROYAL ARENA | Text (hidden when the image exists) | none |
| Scoreboard x2 | `:1564-1569` | W2 ROYAL ARENA (idle only) | **Status**, **Names**, **Score**, **Detail** (new writer) | rule 34 |
| SparSign | `:1618` | W12 PRACTICE BOUT | Text (hidden when the image exists) | none |
| RoyalGateSign x2 | `:1685-1688` | W1 ROYAL GATE | **Status** (`CrownArena.lua:730`) | none |
| Portraits x2 (`RoyalPortrait`, not `RoyalBoard`) | `:1055-1113` | none | the empty-portrait emoji label `Empty` (`:1079-1085`) becomes an ImageLabel with the crown icon at 55% | none |

Sizes: `PixelsPerStud` stays 40 (`Builder.lua:263`). Each line's TextSize = 0.7 x (line height fraction x board height x 40). Example: the Altar's King line is 0.26 x 8.2 studs x 40 = 85 px tall, so TextSize 60 and outline 7 px (today `TextScaled` with a 4 px outline). `MaxDistance`: 260 for big boards (today), 160 for signposts and plates, 80 for vault and statue plates.

### 5.12 Prompts (the "Hold E" boxes)

Royal `ProximityPrompt`s use Roblox's Default style (`Builder.lua:504`), which draws in Roblox's own font. So do the fruit prompts (no script sets a style for them). Changing only the royal ones would make the game inconsistent, so this note leaves them Default (open question 8).

---

## 6. World and assets

**No new 3D models are needed.** This is a UI and sign pass.

| Need | Source | Notes |
|---|---|---|
| Castle board faces | existing board parts from `Builder.lua` | only their SurfaceGuis change |
| Optional studded plank for the 4 plank signs (2 signposts, Return Gate, Sparring Post) | `Workspace."Stud Sign"`, a 2-part sign with a SurfaceGui at (-155, 2.8, -44.5) ([[Existing VFX and Props]]) | proposal: clone it in place of the flat wood planks (`Builder.lua:781`, `:767`, `:1617`) after checking it in Studio |
| Crown model for style cards with no icon yet | `ReplicatedStorage.KingAssets.RoyalCrown` | the Shop already previews crowns in a ViewportFrame this way (`ShopMenu.lua:495-526`) |
| VFX | none new for UI | crown-flight sparkle rate 60 to 24 (`RoyalUI.lua:1166`) and contest-crown burst 40 to 20 (`:1294`), proposal, to calm it down. Hit, slash and slam effects belong to [[Royal Combat]] (it can use `Anime` Slash-01 and `Big` Big-Crack-01 from [[Existing VFX and Props]]) |
| Sounds | existing `C.Sounds` (`CrownConfig.lua:366-383`) | played less often (rules 20 and 21). No new sounds |

What must be built (code): the `UIKit` module, the `UIKitPrefabs` folder, the RoyalUI split into six modules, the sign reskin tool, the overlap checker, and the server text pass (section 8).

---

## 7. Asset requests for Samarth

No animations to publish, no Robux products, no audio. Only images.

Until each image exists, its spot works without it (rules 2 and 9), so building starts now.

### 7.1 Which route for which image

| Kind | Best route | Why |
|---|---|---|
| Anything with words (titles, combat button words) | **Figma**, from his own components | AI tools are weak at exact text ([[Consistent Image-Based UI Kits in Roblox]] section 8) |
| Recolours of his own buttons and bars (A1 to A9) | **Figma first** (duplicate the component, delete the label and icon layers, recolour), ChatGPT as fallback | exact match to his kit |
| Icons (B, C, D, E, F) | **ChatGPT** (transparent PNG) or **Gemini** (green screen, then key out) | no text, style can be copied from references |

### 7.2 Phases

| Phase | Images | Unblocks |
|---|---|---|
| 1 | A1 to A9, B1 to B12, T1, T2, W1 to W14 | the whole HUD, panel, dialog and castle in his style |
| 2 | C1 to C5, D1 to D8, E1 to E6 | decree, achievement and crown-style cards |
| 3 | F1 to F7 with words, A10, T3 to T9 | combat buttons and stingers (time with [[Royal Combat]]) |

### 7.3 Figma: title images (23)

Instruction:
```
In Figma, duplicate the frame you used to make the INDEX title (IndexMenu.TitleText, asset 103221050515090). Keep the same font, italic, white fill, dark outline and drop shadow. Change only the words. Export each as PNG at 2x with a transparent background, 148 px tall, width = the text plus 24 px of padding. Name each file exactly as in the table.
```

| Id | Words | File | Used on |
|---|---|---|---|
| T1 | KINGDOM | title_kingdom.png | Kingdom panel |
| T2 | CROWN CHALLENGE | title_crown_challenge.png | challenge dialog |
| T3 | FIGHT! | title_fight.png | stinger |
| T4 | CROWN POINT | title_crown_point.png | stinger |
| T5 | VICTORY | title_victory.png | stinger |
| T6 | DEFENDED | title_defended.png | stinger |
| T7 | DEFEAT | title_defeat.png | stinger |
| T8 | NEW KING | title_new_king.png | stinger |
| T9 | SUDDEN DEATH | title_sudden_death.png | stinger |
| W1 | ROYAL GATE | sign_royal_gate.png | 2 gate signs |
| W2 | ROYAL ARENA | sign_royal_arena.png | arena arch, signpost, 2 scoreboards (idle) |
| W3 | THRONE ROOM | sign_throne_room.png | signpost |
| W4 | RETURN HOME | sign_return_home.png | Return Gate |
| W5 | CHALLENGE THE KING | sign_challenge_the_king.png | Altar board |
| W6 | KING'S CASTLE | sign_kings_castle.png | castle gate plaque |
| W7 | CURRENT RULER | sign_current_ruler.png | hall tablet |
| W8 | ROYAL RECORDS | sign_royal_records.png | hall tablet |
| W9 | HISTORY | sign_history.png | hall wall board |
| W10 | EMPIRE POWER | sign_empire_power.png | hall wall board |
| W11 | YOUR REIGN | sign_your_reign.png | chamber board |
| W12 | PRACTICE BOUT | sign_practice_bout.png | Sparring Post |
| W13 | THE KING | sign_the_king.png | portrait plate |
| W14 | ETERNAL KING | sign_eternal_king.png | portrait plate |
| T10 | ARMORY | title_armory.png | [[Royal Combat]] Armory panel (replaces that note's ChatGPT request) |

The same batch also makes the other systems' panel titles, so every INDEX-style title comes out of one Figma frame (merged in the [[Integration Review]]): REBIRTH and MAX FRUIT ([[Rarity Gating and Rebirth]] A1, A2), MINE ([[Underground and Ores]] A1), PETS ([[Pets]] A1) and BOAT DOCK ([[Boats and Swimming]] request 3). Each note keeps its own file name and config key.

### 7.4 ChatGPT / Gemini: the shared first message

Paste this once at the start of each chat, with the reference images for that family attached.

```
I am making UI art for my Roblox game Fruit Empire. I will attach pieces of my existing UI kit. Copy their style exactly:
- glossy, chunky cartoon look, like a Roblox simulator game
- soft gradient shading with a bright white highlight at the top left
- saturated colours
- a thick near-black outline around the whole object, the same thickness as in my attached art
- front view, centred, with empty padding on every side
Rules for every image: one object only, transparent PNG background, no text, no letters, no numbers, no logos, nothing cut off at the edges, no shadow on the background. I will ask for one image per message. Keep the style identical across all of them.
```

On Gemini, replace "transparent PNG background" with: `flat pure green #00FF00 background, and a 3 px white outline around the object` (then key out the green; research [36] in [[Consistent Image-Based UI Kits in Roblox]]).

Reference images to attach:
- Family A: crops of `throne-kingdom-menu.jpg` (the Lock and Upgrade pills, the Friend Boost, Shards and Cash bars) and `ui-reference-index-menu.jpg` (the blue title bar and the green Claim button), or the original PNGs of those components.
- Families B to F: `~/fruit-empire/art/royal-icons/style-reference.png` (his six HUD buttons) and `ui-reference-hud-icons.png`.

### 7.5 Family A: buttons, bars, chip, ring (Figma first; prompts are the fallback)

| Id | File (canvas) | Kit reference | Figma route | ChatGPT prompt (paste after the first message) |
|---|---|---|---|---|
| A1 | kit_pill_green.png (360x88) | TopButtons.UpgradeButton 116083144480280 (shape); RewardsBox.ClaimButton 132804991293392 (green) | duplicate the Upgrade pill, delete arrow and word, fill with the Claim green | Image A1. Canvas 360x88. A blank pill-shaped button that fills the canvas with 6 px padding. Copy the exact shape, rounded ends, glossy top band, darker bottom lip and thick outline of my Upgrade top button (TopButtons.UpgradeButton, asset 116083144480280). Colour it the green of my Claim button (RewardsBox.ClaimButton, asset 132804991293392). The face must be empty: no arrow, no text. |
| A2 | kit_pill_blue.png (360x88) | TopButtons.UpgradeButton 116083144480280; IndexMenu.TitleBlue 95532041481396 (blue) | same, fill with the title-bar blue | Image A2. The same pill as A1, coloured the glossy sky blue of my INDEX title bar (IndexMenu.TitleBlue, asset 95532041481396). Empty face. |
| A3 | kit_pill_red.png (360x88) | TopButtons.LockButton 118392145629817 | duplicate the Lock pill, delete padlock and word | Image A3. The same pill as A1, coloured the red of my Lock top button (TopButtons.LockButton, asset 118392145629817). Empty face, no padlock. |
| A4 | kit_pill_gold.png (360x88) | TopButtons.UpgradeButton 116083144480280 | duplicate the Upgrade pill, delete arrow and word | Image A4. The same pill as A1, in the yellow-gold of my Upgrade top button (asset 116083144480280). Empty face. |
| A5 | kit_pill_grey.png (360x88) | UpgradeMenu Maxed button 76326716689640 | duplicate the Maxed button, delete its word, match the pill shape | Image A5. The same pill as A1, in the grey of my Maxed upgrade button (asset 76326716689640), slightly flatter so it reads as switched off. Empty face. |
| A6 | kit_royalchip_gold.png (572x124) | CornerHud.CashBar 100015543922655, CornerHud.ShardsBar 140489943167779, CornerHud.BoostBar 85577747844362 | duplicate the Boost bar, widen to 572, delete the money icon, add an empty round socket at the left | Image A6. Canvas 572x124. A wide rounded HUD bar in the same style as my Cash and Shards counter bars (CornerHud.CashBar asset 100015543922655, CornerHud.ShardsBar asset 140489943167779): thick outline, glossy bevel, darker inner face. Dark royal green face with a gold rim, like my Friend Boost bar (asset 85577747844362). At the left end, an empty round socket 104 px across (a darker disc with a thin gold ring) where an icon will go. No icon, no text, no money symbol. |
| A7 | kit_royalchip_stone.png (572x124) | same as A6 | recolour A6 grey and silver | Image A7. The same bar as A6, but cool stone grey with a silver rim. Same empty socket. |
| A8 | kit_bar_track.png (336x54) | CarryInfo.RateBar 120297790302429 | duplicate the rate bar, delete its text, darken the inside | Image A8. Canvas 336x54. An empty progress-bar track in the style of my fruit rate bar (CarryInfo.RateBar, asset 120297790302429): rounded ends, thick outline, dark recessed inside. Empty. |
| A9 | kit_bar_fill.png (336x54) | CarryInfo.RateBar 120297790302429 | the inner shape of A8, 6 px smaller, white | Image A9. Canvas 336x54. The fill for A8: the same rounded shape, 6 px smaller on every side, pure white with a soft vertical light-grey gradient and a thin brighter line along the top. No outline. It must be white because the game tints it. |

**A8 and A9 are the one bar pair for the whole expansion** (decision D-027): fruit health ([[Explosives and Fruit Health]]), ore node HP ([[Underground and Ores]]), boat HP and stamina ([[Boats and Swimming]]), fighter HP ([[Royal Combat]]) and the decree and challenge timers here. The four separate bar requests those notes had are withdrawn. Fill colours for every HP bar: above 50% Uncommon green, 25% to 50% Legendary gold, under 25% Mythic red.
| A10 | kit_ring_charge.png (300x300) | Templates.Cell.Selection 113285622864672 | a circle stroke in the selection-outline style | Image A10. Canvas 300x300. A plain circular ring 18 px thick, in the white glossy style of my card selection outline (Templates.Cell.Selection, asset 113285622864672). White only, empty centre, thin dark outline on both edges. |

### 7.6 Family B: royal HUD icons (256x256, object fills about 80%)

Every prompt starts with: `Style: exactly like my HUD button art (InventoryButton asset 108477171933167, IndexButton asset 118941334639047, ShopButton asset 71485100495903), attached. Canvas 256x256, the object fills about 80%.` Then the object line.

| Id | Key in `C.Icons` | File | Object line |
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
| B11 | Lock (new key) | icon_lock.png | Icon B11: a chunky gold padlock with a dark keyhole, the same padlock look as the one on my Lock button (asset 118392145629817). |
| B12 | Reign (new key) | icon_reign.png | Icon B12: a golden hourglass with blue sand, about half run through. |

### 7.7 Family C: decree icons (256x256)

Prefix as Family B, plus: `Also match the sparkles and energy of my Merge All button (MergeAllButton, asset 99980479639840).`

| Id | Key | File | Object line |
|---|---|---|---|
| C1 | MergeRush | icon_mergerush.png | Icon C1: a red apple and an orange bumping together, a big yellow lightning bolt between them, a few small four-point sparkles. |
| C2 | FruitFrenzy | icon_fruitfrenzy.png | Icon C2: a burst of mixed fruit flying outward (cherries, a strawberry, a banana, a bunch of grapes) with short motion lines. |
| C3 | RoyalRace | icon_royalrace.png | Icon C3: a black-and-white checkered race flag waving on a gold pole, a small gold crown on top of the pole. |
| C4 | CrownHunt | icon_crownhunt.png | Icon C4: a golden crown half hidden in a leafy green bush, a magnifying glass in front of it. |
| C5 | EmpireBuild | icon_empirebuild.png | Icon C5: a tall stack of gold coins with a big green arrow beside it pointing up. Match the money green of my Cash chip (RewardsBox chip, asset 82056408182031). |

### 7.8 Family D: achievement medals (256x256)

Prefix as Family B, plus: `All eight medals use the SAME round medal: a thick rim around a darker inner disc, a symbol in the middle. Same outline weight as my reward chips (RewardsBox chips, assets 89282702522748, 82056408182031, 78947574768646). Only the rim colour and the symbol change.`

| Id | Key | File | Object line |
|---|---|---|---|
| D1 | FirstCrown | medal_firstcrown.png | Medal D1: bronze rim; symbol a small gold crown. |
| D2 | CrownKeeper | medal_crownkeeper.png | Medal D2: silver rim; symbol a green shield with a gold crown on it. |
| D3 | Tyrant | medal_tyrant.png | Medal D3: red rim; symbol a gold crown with a small hourglass in front. |
| D4 | Emperor | medal_emperor.png | Medal D4: purple rim; symbol a gold crown inside a green laurel wreath. |
| D5 | Dynasty | medal_dynasty.png | Medal D5: gold rim; symbol three gold crowns stacked. |
| D6 | Conqueror | medal_conqueror.png | Medal D6: dark steel rim; symbol two crossed swords behind a gold crown. |
| D7 | Undisputed | medal_undisputed.png | Medal D7: orange rim; symbol a gold crown surrounded by flames. |
| D8 | Legend | medal_legend.png | Medal D8: rainbow rim; symbol a glowing white-gold crown with small angel wings and tiny stars. |

### 7.9 Family E: crown styles (256x256)

Prefix as Family B, plus: `This is the SAME crown as icon B1: identical shape, size and angle. Only the colours change.` Colours are the style colours in `CrownConfig.lua:297-316`.

| Id | Key | File | Object line |
|---|---|---|---|
| E1 | Golden | crown_golden.png | Crown E1: band gold (RGB 255, 198, 46), gems red (230, 40, 50). |
| E2 | Emerald | crown_emerald.png | Crown E2: band gold (255, 198, 46), gems bright green (40, 220, 110). |
| E3 | Ruby | crown_ruby.png | Crown E3: band warm orange-gold (255, 186, 60), gems pink-red (255, 30, 80). |
| E4 | Shadow | crown_shadow.png | Crown E4: band dark iron grey (60, 58, 70), gems glowing purple (150, 70, 255). |
| E5 | Celestial | crown_celestial.png | Crown E5: band white-silver (235, 245, 255), gems sky blue (90, 200, 255), a faint white glow. |
| E6 | Soul | crown_soul.png | Crown E6: band glowing hot pink (255, 120, 230), gems white, a soft pink glow. |

### 7.10 Family F: combat buttons (256x256, then the word in Figma)

Prefix as Family B, but replace the size line with: `Canvas 256x256. Put the object in the top 70% only and leave the bottom 30% completely empty (I add a word there). Style like my Merge All and Clear buttons (MergeAllButton asset 99980479639840, ClearButton asset 88888506311530).`

| Id | Key in `C.CombatButtons` (new) | File | Object line | Word added in Figma |
|---|---|---|---|---|
| F1 | Attack | btn_attack.png | Icon F1: a silver sword slashing diagonally with a short white swoosh arc. | Attack |
| F2 | Block | btn_block.png | Icon F2: a round wooden shield with a steel rim and a gold centre boss. | Block |
| F3 | Dash | btn_dash.png | Icon F3: a green boot with small white wings and speed lines behind it. | Dash |
| F4 | Slam | btn_slam.png | Icon F4: a gold gauntlet fist hitting cracked stone ground, small rocks flying. | Slam |
| F5 | Shockwave | btn_wave.png | Icon F5: a gold crown sending out two golden rings of wave around it. | Wave |
| F6 | Guard | btn_guard.png | Icon F6: a gold crown in front of a glowing blue shield bubble. | Guard |
| F7 | Summon | btn_knights.png | Icon F7: a steel knight helmet with a green plume, two smaller helmets behind it. | Knights |

Figma step for the words:
```
Use the label style baked into my Clear button (ClearButton, asset 88888506311530): same font, white fill, same thick dark outline. Put the word centred in the empty bottom 30% of each F image. Export 256x256 PNG, transparent.
```

### 7.11 Cleaning, slicing and upload plan

1. **One image per message**, so there are no sheets to cut. If the old 3x3 and 3x2 sheets from `~/fruit-empire/art/royal-icons/PROMPTS.md` (3 Oct) already exist, I cut them on their grid (341 px cells on a 1024 sheet, 512 px cells on a 1536x1024 sheet), then trim each the same way. The per-image prompts here replace those sheet prompts.
2. **Trim and pad** to the exact canvas in the tables. Icons keep 12 px of empty padding.
3. **Remove the background** (Gemini: key out the green, hue within 22 degrees, saturation and value at least 0.3, as in the research note).
4. **Alpha bleed 4 px** (PixelFix or the 2026 automated tool in research [23]), so edges do not get a dark halo.
5. **Check on one sheet**: place each new image next to 3 of his kit images over a light and a dark background. Compare outline thickness, highlight direction and saturation. Redo outliers with the fix-up line: `Redo only image <Id> as a single image. Same style, same canvas, transparent background, no text.`
6. **Upload** in Studio: Window, Asset Manager, Bulk Import (or Creator Hub, Creations, Development Items, Images). Either Samarth uploads, or I do it through the Studio bridge's image upload if that works for this account. Images are Open Use by default unless Asset Privacy is on ([[Roblox Platform Constraints for the Expansion]] section 8).
7. **Wait for moderation** (usually a few hours). Type an id into config only after the image shows in Studio. A pending image renders blank, and a non-zero id switches off the text fallback.
8. **Paste ids**:
   - B, C, D, E keys into `CrownConfig.Icons` (D-001's one-number swap).
   - T and W into the new `CrownConfig.TitleImages`.
   - F into the new `CrownConfig.CombatButtons`.
   - A1 to A10 into `UIKit.Images` (`PillGreen`, `PillBlue`, `PillRed`, `PillGold`, `PillGrey`, `ChipGold`, `ChipStone`, `BarTrack`, `BarFill`, `RingCharge`).
9. **9-slice** in Studio's 9-Slice Editor for A1 to A9 and for the kit panel body, inline frame, blue title bar and info box. Record each image's 4 edge insets in `UIKit.Slice`. Starting guesses to adjust: pills 44 px (half the 2x height), chip 30 px, bar 13 px, panel body 40 px.
10. **Add every new image to [[UI Kit Inventory]]** with its id and native size.

---

## 8. Code plan

Pattern: the server's GardenService loads modules that each `return function(G)`, in a fixed order, sharing state through `G` (`GardenService.lua:512-518`). The client side copies that shape: one LocalScript builds a shared table `R` and loads child ModuleScripts that each `return function(R)`.

### 8.1 New modules and tools

| Name | Kind | Job |
|---|---|---|
| `ReplicatedStorage.UIKit` | ModuleScript (client use) | The kit in code. Owned by this note; other systems use it too. |
| `ReplicatedStorage.UIKitPrefabs` | Folder, made once in Studio | Copies of kit shells: `Panel` (the Index frame pieces with their content removed), `Confirm` (`GardenUI.ClearConfirm`), `Card` (one Shop item card), `Tab` (one Shop tab). Code clones these so sizes, slices and strokes match exactly. |
| `StarterPlayerScripts.RoyalUI` | LocalScript (rewritten, wiring only) | builds `R`, loads the six modules below, routes `RoyalEvent` to `R.handlers` |
| `RoyalUI.Herald` | ModuleScript | herald queue, scope and sound rules (18 to 20), stingers (22) |
| `RoyalUI.Strip` | ModuleScript | the event strip cards and priority (24) |
| `RoyalUI.Hud` | ModuleScript | chip, alert badge, action pill (25 to 28), arena focus (16) |
| `RoyalUI.Confirm` | ModuleScript | challenge dialog; picker shell for [[Royal Combat]] |
| `RoyalUI.Panel` | ModuleScript | Kingdom panel, tabs, one-modal rule (15), query cache (31), gamepad (32) |
| `RoyalUI.World` | ModuleScript | King title, fighter tags, crown flight, spinners. Moved from `RoyalUI.lua:965-1295` with the restyles in 6.8 |
| `StarterPlayerScripts.UIOverlapCheck` | LocalScript, Studio only | every 2 s, collects the rectangles of visible cluster roots in GardenUI, RoyalHud and RoyalCombat (not their children), plus the two toast labels, and warns about any two that intersect (rule 12) |
| `ServerStorage.ZZTools.RoyalSignsReskin` | dev tool, run once from the command bar | reskins every `RoyalBoard` SurfaceGui in place (rule 33), without rebuilding the castle. Idempotent: stamps `KitSkin = 1` on each board part |

`UIKit` API (proposal):

```lua
local Kit = {}
Kit.Font = Font.new("rbxasset://fonts/families/FredokaOne.json")
Kit.Size = { Title = 30, Heading = 24, Body = 20, Small = 17 }
Kit.Images = { PanelBody = 82827190351667, PanelInline = 110403403749726, TitleBar = 95532041481396,
	ScrollBox = 83403190365135, InfoBox = 101725900357985, ItemCard = 99120915117089,
	ShopTab = 110917091182913, CloseSquare = 107111219044453, SelectOutline = 113285622864672,
	AlertBadge = 79539007098845, NameBar = 123470450245311, RateBar = 120297790302429,
	ClaimButton = 132804991293392, Maxed = 76326716689640,
	PillGreen = 0, PillBlue = 0, PillRed = 0, PillGold = 0, PillGrey = 0,   -- new art, 0 until uploaded
	ChipGold = 0, ChipStone = 0, BarTrack = 0, BarFill = 0, RingCharge = 0 }
Kit.Slice = {}                         -- Rect insets per key, measured in the 9-Slice Editor
function Kit.scale() end               -- Config.HudScaleFor(viewport, TouchEnabled and not KeyboardEnabled) or 1
function Kit.text(parent, step, text, props) end   -- TextScaled false, white, stroke = max(2, 0.12 * size)
function Kit.image(parent, key, props) end         -- ImageLabel, Slice when Kit.Slice[key] exists
function Kit.pill(parent, colour, label, iconId, props) end -- ImageButton; while Pill<colour> is 0, ClaimButton tinted
function Kit.bar(parent, props) end                -- returns { set = fn(fraction), tint = fn(Color3) }
function Kit.icon(parent, id, size) end            -- nil when id is 0: never an emoji
function Kit.clone(prefabName) end                 -- from UIKitPrefabs
Kit.Slot = {}                                      -- the shared top-centre status slot: push(id, priority, card), pop(id); royal cards only
Kit.Modals = { "ShopMenu", "IndexMenu", "StoredFruitsMenu", "UpgradeMenu", "RebirthMenu", "PetsMenu", "MineMenu",
	"BoatDockMenu", "ArmoryUI", "KingdomPanel" } -- every panel that hides the others (rule 15); one list for the whole game
return Kit
```

`RoyalUI` LocalScript shape:

```lua
local R = { C = C, Config = Config, Kit = require(ReplicatedStorage.UIKit), player = player,
	state = state, remotes = { request = requestRemote, event = eventRemote, query = queryRemote },
	handlers = {} }
-- R.S(key), R.now(), R.fmt(seconds), R.query(what) (5 s cache) defined here
for _, name in ipairs({ "Herald", "Strip", "Hud", "Confirm", "Panel", "World" }) do
	require(script:WaitForChild(name))(R)   -- each module: return function(R) ... end
end
eventRemote.OnClientEvent:Connect(function(kind, data)
	local h = R.handlers[kind]
	if h then local ok, err = pcall(h, data or {}) if not ok then warn("[RoyalUI] " .. kind .. ": " .. tostring(err)) end end
end)
```

### 8.2 Changes to existing scripts

| File : lines | Today | Change |
|---|---|---|
| `RoyalUI.lua:108-289` | code-drawn kit: `corner`, `border`, `shade`, `frame`, `label`, `button` | deleted; `UIKit` instead |
| `RoyalUI.lua:110-116` | one ScreenGui "RoyalUI", DisplayOrder 6 | three ScreenGuis: RoyalHud (1), RoyalPanel (20), RoyalConfirm (30), each with attribute `KitButtons = true` |
| `RoyalUI.lua:119-125` | own scale curve | `Kit.scale()` |
| `RoyalUI.lua:214-224` | `iconId` / `dropEmoji`: id 0 keeps the emoji | `Kit.icon` returns nil; no emoji strings remain to drop |
| `RoyalUI.lua:284` | Sparkle on every click | deleted |
| `RoyalUI.lua:297-346`, `:392-431` | chip, 3 buttons, court line | `RoyalUI.Hud` |
| `RoyalUI.lua:348-461`, `:552-667`, `:701-709` | three stacked cards, trial card | `RoyalUI.Strip` |
| `RoyalUI.lua:463-550` | announcements, big text | `RoyalUI.Herald` |
| `RoyalUI.lua:669-699` | challenge card | `RoyalUI.Confirm` |
| `RoyalUI.lua:711-963` | cream menu window, 4 menus | `RoyalUI.Panel` |
| `RoyalUI.lua:965-1295` | world visuals | `RoyalUI.World`; fighter tags every 0.1 s, not every frame (`:1535-1536`) |
| `RoyalUI.lua:1299-1490` | handlers | stay in the LocalScript and call the modules; texts from the new `C.Text`; no `:upper()` (`:1305-1311`); `achievement` (`:1465`) and `throne` (`:1473-1478`) no longer use the herald |
| `RoyalCombat.lua:169-264` | code-drawn round buttons with emoji | F1 to F7 images; ScreenGui DisplayOrder 2 and `KitButtons = true`; manual press shrink `:242-253` removed |
| `RoyalCombat.lua:197`, `:335-336` | dotted key hint | name-plate legend (rule 38) |
| `RoyalCombat.lua:267-281` | charge ring as a Frame stroke | ring image A10 |
| `RoyalCombat.lua:343` | emoji ability text | image swap F4 / F5 |
| `KingClient.lua:2-3` | says the crown goes to the richest player | corrected: draws the King's aura only |
| `KingClient.lua:30-33` | gated by `GardenConfig.KingEnabled` only | also returns if `CrownConfig.Enabled` is false |
| `KingClient.lua:87-107`, `:249-256` | dead floating-crown code (early return at `:80`) | deleted |
| `ShopMenu.lua:567-583` | code-drawn Equip pill | `Kit.pill` green / grey |
| `ShopMenu.lua:633-645` (after) | nothing | optional `OpenShopTab` listener (5.10) |
| `ButtonAnimator.lua:26`, `:207-217` | adopts buttons under GardenUI only | also adopts buttons in any PlayerGui ScreenGui with attribute `KitButtons = true`, existing and future |
| `CrownConfig.lua:209-254` | decree `icon` emoji, ALL CAPS titles | `icon` fields deleted, Title Case |
| `CrownConfig.lua:283-288`, `:328-337` | ALL CAPS titles | Title Case |
| `CrownConfig.lua:359` | `CameraEmphasis = true` | false |
| `CrownConfig.lua:390-397` | 29 icon keys | + `Lock`, `Reign`; new tables `TitleImages` (23 keys, all 0), `CombatButtons` (7 keys, all 0) and `UI` (the layout and herald numbers of section 4) |
| `CrownConfig.lua:417-477` | text with emoji | the Words table in section 3 |
| `Crown.lua:207`, `:211`, `:228`, `:232` | toasts with emoji | plain text |
| `Crown.lua:576-598` | `boardsOf`, `setLine` | + `applyBoardTitles()` at load: for each board, set `Back.TitleImage.Image` from `C.TitleImages` and hide `Back.Title` when the id is above 0 |
| `Crown.lua:927-968` | `RoyalQuery` has no limit | 6 calls per player per 2 s, nil above that |
| `Crown.lua:1085-1141` | board strings with emoji, middle dots and the court line | rewritten; Power and ReignRecords rows split into name and value labels; court line removed |
| `CrownArena.lua:678-697` (`publishBout`) | RoyalState only | also writes the two scoreboards (rule 34) through `G.royalSetLine("Scoreboard", ...)` |
| `CrownArena.lua:1039` | emoji announcement | `C.Text.TrialCalled`, `scope = "all"` |
| `CrownDecree.lua:93`, `:130`, `:153`, `:249` | emoji and `:upper()` | plain text |
| `CrownDecree.lua:345-357` (`decrees` query) | sends `icon` | drops `icon`; adds `reward` (number or list, for this player), `time`, `goal` |
| `Builder.lua:257-310` (`board`) | flat Frame, TextScaled, optional no-outline | kit board: `Back` as panel-body ImageLabel, `TitleImage` ImageLabel, fixed sizes, outline per rule 5 |
| `Builder.lua` title lines (5.11 table) | emoji titles | title keys and plain words |
| `Builder.lua:1079-1085` | portrait emoji placeholder | crown icon ImageLabel |

### 8.3 Remotes

No new remotes.

| Remote | Direction | Payload change | Server validation |
|---|---|---|---|
| `RoyalRequest` | client to server | none. The panel now sends `cancelChallenge` (handler already exists, `CrownArena.lua:1025-1030`) | unchanged: action string at most 24 characters, one request per 0.15 s per player, `prompt:` actions refused (`Crown.lua:865-880`); each handler re-checks King, eligibility and busy |
| `RoyalQuery` | client asks server | `decrees` answer: `{ key, title, blurb, reward, time, goal }`, no `icon`. `reward` is the amount this player would get | new: 6 calls per player per 2 s; `what` must be one of history, me, king, power, decrees |
| `RoyalEvent` | server to clients | `announce` gains `scope` ("all" or "near") and optional `icon` (a `C.Icons` key). `style` stays | server only |
| `Notice` / `Notify` | server to client | plain text only (no emoji) | server only |

### 8.4 Save fields

None. This is presentation only. Nothing to migrate. `royal.style` and the other royal fields are untouched (`Crown.lua:252-285`).

### 8.5 Attributes

| Attribute | On | Type | Written by | Read by |
|---|---|---|---|---|
| `KitButtons` (new) | royal ScreenGuis | bool | RoyalUI, RoyalCombat | ButtonAnimator |
| `KitSkin` (new) | each `RoyalBoard` part | number (version) | RoyalSignsReskin | the tool itself (skip if already done) |
| `OpenShopTab` (new, optional) | GardenUI, client only | string | RoyalUI.Panel | ShopMenu |
| `NextDecreeAt`, `DecreeKey`, `TrialOpen`, `TrialEnds`, `ChallengeQueue`, `PendingChallenger`, `DuelUnderdog`, `GateOpen` | RoyalState | existing | Crown modules | chip badge, strip, Throne tab |
| `RoyalEligible`, `RoyalWhy`, `RoyalBusy`, `InArena` | Player | existing | Crown modules | chip, pill, panel, arena focus |

### 8.6 Build order

1. `UIKit` with fallbacks, `UIKitPrefabs` (Studio copy), ButtonAnimator change.
2. RoyalUI split into the six modules, new layout, `C.Text` rewrite, server text pass. This is already emoji-free and overlap-free with no new art.
3. `RoyalSignsReskin` on the castle, `applyBoardTitles`, scoreboard writer.
4. RoyalCombat visuals, Shop King tab pills, KingClient tidy.
5. Type in image ids as each phase of art lands (section 7.2).

---

## 9. Edge cases and abuse

1. **Art not ready.** Icon id 0: no icon (rule 2). Title id 0: plain text title (rule 9). Pill and bar ids 0: the kit Claim button and rate bar, tinted. Nothing waits on art.
2. **Image still in moderation.** It renders blank. Ids are typed in only after the image shows in Studio (7.11 step 7).
3. **Long display names** (up to 20 characters) truncate with an ellipsis on the chip, card and herald. The herald line is at most 48 characters plus the name.
4. **Late join.** RoyalState attributes replicate on join, so the chip and strip are right on the first refresh. Panel tabs query on open.
5. **Leave mid-challenge.** The server voids the pending challenge (`CrownArena.lua:1344-1366`). The dialog also hides itself when its timer runs out (`RoyalUI.lua:1540`).
6. **Leave or die mid-duel.** The server forfeits the bout and clears the `Duel*` attributes (`publishBout` with no bout, `CrownArena.lua:679-684`), so the duel card goes away. Arena focus restores the HUD when `InArena` turns false, and on respawn.
7. **Rebirth mid-panel.** Opening the Rebirth panel closes the Kingdom panel (rule 15). The King keeps the crown through a rebirth ([[Royal System - Server]]).
8. **Two events end in one frame.** The strip re-picks by priority on the next refresh (at most 0.5 s).
9. **Contest final.** `ContestState` "final" hides the contest card (`RoyalUI.lua:626`) and the final bout shows as a Duel card ("Final Duel").
10. **Herald while a modal is open.** Goes to a green toast, so it is neither hidden behind the panel nor lost.
11. **Phones.** Rule 13 drops the action pill when space is short. The stinger can hide the strip briefly. Text never goes under 9 px. Touch combat buttons keep today's layout around the jump button, and arena focus hides the potion timers that would sit near them.
12. **Gamepad.** Tabs, cards and pills are selectable with the kit selection outline (rule 32).
13. **Player Text Size set to Largest.** Fixed text sizes follow the setting (TextScaled does not, research section 5). Labels truncate rather than overflow; tested in section 11.
14. **Exploits.** The client only draws and asks. Every royal action is re-checked by the server. `RoyalQuery` gets a rate limit. The UI cannot give shards, rep or the crown. Nothing here takes another player's fruit, pets or currency.
15. **Streaming.** `Sky Islands` and the Royal Gate are Persistent models (`Builder.lua:1749`, `:1656`), so the boards are always present on clients.
16. **Two `KingUserId` copies.** RoyalUI reads only `RoyalState.KingUserId`. KingClient reads the workspace copy. `publishCrown` writes both (`Crown.lua:691-696`).
17. **Studio vs live.** Studio needs 0 fruits and 5 s in the server to challenge (`CrownConfig.lua:38-41`). The red requirement line must also be checked in a live server.
18. **Roblox's chat window** opens over the top-left, as it does over the garden's left column today. It is Roblox UI and is not counted by the overlap checker.
19. **Invisible King.** The overhead title hides (existing, `RoyalUI.lua:972-975`); the chip still shows who rules.
20. **Font coverage.** All royal strings are ASCII, so no glyph can be missing from Fredoka One.

---

## 10. Performance

Limits used: 60 FPS frame budget 16.67 ms; most images 512x512 or less and minor ones 256x256 or less; avoid changing emitter properties every frame; 50 players per server ([[Roblox Platform Constraints for the Expansion]]).

| Item | Count | Cost |
|---|---|---|
| Royal HUD GuiObjects visible at once | about 30 (chip 7, pill 3, strip card about 14, herald 4, stinger 2) | today about 60 (chip, buttons, three cards, announcement) |
| Kingdom panel | about 165 when every tab has been opened; built on first open, only the shown tab is visible | one-off |
| Castle SurfaceGuis | 28 boards + 2 portraits, about 130 labels and 28 title images | same count as today plus one image each; smaller `MaxDistance` on plates cuts drawing |
| Per-frame work | spinners and flags within 320 studs (existing, `RoyalUI.lua:1253-1269`); charge ring only in the arena | fighter tags drop from every frame to 10 per second |
| Refresh loop | every 0.5 s; text written only when it changed | existing loop |
| Network | no new remotes; up to about 10 scoreboard line writes per bout; `RoyalQuery` capped at 6 per 2 s per player | negligible |
| New images | 48 AI images (icons 256x256, pills 360x88, chip 572x124, bars 336x54, ring 300x300) + 30 Figma images (titles about 600x148, buttons 256x256) | about 25 MB of texture memory if every one were loaded at once (estimate: 256 KB per 256x256 image); most load only when their panel or sign is on screen |

The full-workspace scan in `collect()` (`RoyalUI.lua:1225`) runs at 4 s and again when a spinning model appears. A CollectionService tag would avoid walking every instance; out of scope here, noted for later.

---

## 11. Test plan

Setup for every test: in Studio run `workspace:SetAttribute("StudioNoSave", true)` first (playtests write the real save otherwise, `GardenStore.lua:572`). Use Test, Clients and Servers, 3 players. Server command bar: `game.ServerStorage.RoyalDev:Invoke(cmd, ...)` with `crown`, `vacate`, `rep`, `bout`, `ringout`, `endBout`, `spar`, `contest`, `contestNow`, `join`, `decree` (`Crown.lua:1199-1219`, `CrownArena.lua:1410-1423`, `CrownContest.lua:571-583`, `CrownDecree.lua:367-374`). Device Emulator at Actual Resolution: iPhone landscape (Phone), iPad (Tablet), 1920x1080 (Desktop), Xbox (Console).

| Test | Steps | Pass when | Rules |
|---|---|---|---|
| 1. Emoji scan | Studio-only script walks every TextLabel and TextButton under RoyalHud, RoyalPanel (each tab), RoyalConfirm, RoyalCombat and every `RoyalBoard` SurfaceGui; it also prints every string the client receives on the `Notice` and `Notify` remotes during tests 8 to 17 | no character above 126 except inside player names | 1, 2 |
| 2. Icons at 0 | leave every `C.Icons` id at 0, open each screen | no emoji, no empty icon gaps, text left-aligned | 2, 9 |
| 3. Font and look | open each screen next to the Index panel | Fredoka everywhere, white text, outlines present, no Frame backgrounds, UICorner or UIGradient in royal ScreenGuis (scripted check) | 3 to 8, 10 |
| 4. Ladder | in Studio, print TextSize and TextScaled of every royal label | all TextScaled false, sizes in {30, 24, 20, 17} (or the board formula) | 6 |
| 5. Overlap checker | run `UIOverlapCheck` on all four devices through: idle, duel (fighter and watcher), contest announce, contest zone phase, decree running, King with decree ready, a warning toast and a notice toast fired at the same time | zero warnings | 11, 12, 14, 17 |
| 6. Phone pill | Phone preset, make a player eligible and start a contest | the pill does not show; Join appears inside the strip card | 13 |
| 7. One modal | open Kingdom, then press Index; open Index, then tap the chip; open Kingdom while the offline popup shows | never two panels; the royal HUD hides while any is open | 15 |
| 8. Arena focus | `RoyalDev:Invoke("spar")`, then end it; also die in the arena | the listed clusters hide on entry and come back exactly as before on exit and on respawn | 16 |
| 9. Herald | in the server command bar run `game.ReplicatedStorage.GardenRemotes.RoyalEvent:FireAllClients("announce", { text = "Test " .. i })` for i = 1 to 5 within 1 s, then twice with the same text | one at a time, at most 3 waiting, duplicate dropped, 3.0 s hold | 18 |
| 10. Personal news | `RoyalDev:Invoke("rep", userId, 300)` to unlock Ruby and High King | green toasts only, no herald | 19 |
| 11. Sound scope | crown a player while a second client stands far from the castle | the far client sees the herald with no sound | 20, 21 |
| 12. Stinger scope | `bout` with one client in the arena, one in the stands, one on the island | the island client sees no stinger | 22 |
| 13. Camera | crown a player near the throne | no camera movement | 23 |
| 14. Strip priority | start a decree, then a contest, then a bout; move a client near and far from the arena | the card shown follows rule 24 | 24 |
| 15. Chip states and badge | `vacate`, `crown` another player, `crown` yourself; let protection expire; make a decree ready | each state and badge as in section 5.2 | 25 to 27 |
| 16. Action pill order | open the gate during a duel and call a trial | Watch beats Answer; no Challenge or Kingdom on the HUD | 28 |
| 17. Panel tabs and actions | press Challenge, then Leave Line (check the server queue empties); as King, issue a decree and call a trial; use the Decree Desk and the Style Mirror prompts | correct tab opens; each pill sends its request; Leave Line clears `RoyalBusy` | 29, 30 |
| 18. Query limit | in a client command bar run `for i = 1, 20 do print(i, game.ReplicatedStorage.GardenRemotes.RoyalQuery:InvokeServer("me") ~= nil) end` | calls 7 to 20 print false; the panel still renders from its cache | 31 |
| 19. Gamepad | connect a controller, open the panel | tabs and pills reachable, selection outline is the kit image | 32 |
| 20. Castle boards | run `RoyalSignsReskin`, then crown, duel and vacate | every board shows the kit panel, title image or white fallback, live lines still update, scoreboards show the bout | 33, 34 |
| 21. Overhead title | crown a player, drink Invisibility outside the arena | title shows icon, white title and name; hides while invisible | 35 |
| 22. Court line | crown a player who has a friend in the server | no court text anywhere | 36 |
| 23. Combat visuals | on Phone, `spar`; press each button; hold Attack | images with words, cooldown overlay drains, ring turns orange at 0.45 s | 37 |
| 24. PC legend | Desktop, `spar`, then a contest as King | legend rows as in rule 38 | 38 |
| 25. Shop King tab | buy and equip an aura | kit pills for Equip / Equipped | 39 |
| 26. KingClient | set `CrownConfig.Enabled = false` in a Studio copy | no aura is drawn | 40 |
| 27. Text Size Largest | set Roblox Text Size to Largest, repeat tests 5 and 15 | no overflow outside cards; truncation instead | 6, 12 |

---

## 12. Dependencies and conflicts

| System | How it touches the King UI | Resolution |
|---|---|---|
| [[Rarity Gating and Rebirth]] | `MinDiscovered = 10` can never be met at stage 1 (5 fruits), so stage-1 players see a requirement they cannot finish (section 4.3). Rebirth panel opening must close the Kingdom panel. | Gating sets `MinDiscovered` (recommended 5). The UI prints `RoyalWhy` only. Rule 15 covers the panels. |
| [[Bats]] | Bats must not work in the arena (server note). No royal UI. | Bat HUD uses kit art, avoids the royal zones, and is hidden by arena focus (rule 16). Its touch action slot is ScreenGui `BatControls` at DisplayOrder 3 (section 4.7). |
| [[Explosives and Fruit Health]] | No royal UI. Explosives must not damage castle boards or the gate. | Their server code skips `Sky Islands` and the `Royal Gate` model. |
| [[Weapons and Artillery]] | Artillery must not hit the Royal Capital or the arena (the King never griefs, `CrownConfig.lua:20-21`). | Same skip list. Any weapon picker UI uses `UIKit` cards. |
| [[Underground and Ores]] | Ore entries must not count toward `MinDiscovered` (`Crown.lua:289-292`), or the red line flips after a rejoin. | Underground keeps ores out of the fruit Index count. |
| [[Pets]] | Pets never leave their owner's plot (Pets P35), so none reach the arena. Pet UI must not reuse the royal zones. | `PetsButton` is in the arena focus list (rule 16) and `PetsMenu` in the one-modal list (rule 15). Pets use `UIKit`. |
| [[Boats and Swimming]] | The research suggested a swim fatigue meter at top centre below the pills, the same place as the event strip. | Resolved in the [[Integration Review]] (C23): the stamina and boat HP bar (`SeaStatus`) stays at the bottom centre, above the hands stack, and HudScale keeps it out of the notice toast lane (rule 14). It does not use `Kit.Slot`, so the event strip stays royal-only. |
| [[Potions]] | The potion timer block sits bottom right with a mouse; on touch it moves to the top left, right of the left HUD column and 8 px under the chip cluster ([[Potions]] UI 3). Invisibility hides the King title. | Arena focus hides `EffectStack`. On touch the block reads the chip cluster's live bottom edge, so it never touches the chip or the action pill. |
| [[Plot Overhaul and Environment]] | The Royal Gate stands on the road between Plot0 and Plot11 (`CrownConfig.lua:351-353`), and its two signs are reskinned here. | The plot rebuild keeps the `Royal Gate` model, its `Portal`, `Bars`, `RoyalPrompt` and both `GateSign` boards. |
| [[Royal Combat]] | Shares the duel card, fighter tags, touch buttons, legend, stingers and the blade picker. | Split: Royal Combat owns rules, numbers, animations, hit and slam effects, skin data, the Armory panel's content, the ability bar and the RoyalState attributes for health (`DuelAHP`, `DuelBHP`). King UI owns how every one of those looks and where it sits: the duel board is this note's event strip card (top edge 112 px, 560 x 96, bars in the sub row), touch buttons use F1 to F7 with baked words, the legend replaces Royal Combat's hint line, and every bar is A8 and A9. Royal rewards per stage (section 4.2) are Royal Combat's units (D-017). |
| King UI Overhaul | (this note) | `UIKit` and `Kit.Slot` are offered to every system above, so new screens come out of one kit. |

---

## 13. Open questions for Samarth

Each has a recommended default, so building can start now.

1. **Figma or ChatGPT?** You offered both. Default: Figma for the 23 title images, the 7 combat words and the button and bar recolours (duplicate your own components). ChatGPT for the 31 icons, medals and crowns.
2. **Change D-001.** D-001 says an icon id of 0 keeps the emoji. You said "I dont like the emoji-slop". Default: an id of 0 shows no icon, text only (rule 2). This contradicts D-001, so it needs your yes.
3. **One panel title.** Default: "KINGDOM" for the whole panel, with tabs Throne, Decrees, Styles, Records.
4. **Strip the HUD down to the chip.** Default: no Challenge or Kingdom button on the HUD; the chip opens the panel and one action pill appears only for Watch, Join or Answer.
5. **Camera takeover off.** Default: off (rule 23).
6. **Remove the "Royal Court +income" line.** It shows the friend bonus every player already gets as if it were a King perk. Default: remove.
7. **Castle boards in the dark kit panel.** Default: the same dark panel body as the Index, with your title images. Alternative: a studded wood plate made to match (one more image).
8. **"Hold E" prompts.** They use Roblox's own font everywhere, fruit prompts included. Default: leave them for now; a custom kit prompt later as a game-wide change.
9. **Auras stay in the Shop.** Default: yes, with an Open Shop pill on the Styles tab (a 6-line ShopMenu hook).
10. **Royal rewards are flat** today (worth 333 merges at stage 1 and 0.6 at stage 9). Decided in the [[Integration Review]]: [[Royal Combat]]'s stage-scaled units for duels, contests and decrees (section 4.2, D-017). The UI shows whatever the server sends.
11. **Who can challenge at stage 1.** Default: `MinDiscovered = 5`; [[Rarity Gating and Rebirth]] owns and makes the change (D-026).
12. **Arena focus hides the cash counters too.** Default: yes, the whole garden HUD except Mute.
13. **Server-wide challenge news.** Default: everyone sees "Ana challenged the King", but only the King, the challenger and nearby players hear it.
14. **Which font is in your INDEX title?** Needed only if you want me to make the title images instead of you. Default: you duplicate your INDEX title component in Figma.
15. **Do your Yes / No buttons have the words baked in?** Default: assume yes, and the dialog asks "Accept the duel?" so Yes and No read correctly.
16. **Studded planks.** Default: try the existing `Stud Sign` model for the 4 plank signs; keep the current planks if it looks wrong.
