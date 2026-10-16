---
tags: [roadmap, m2, verification]
date: 2026-10-05
---
# M2 UI kit and King UI: verification

M2 is built and live in Studio (not yet published). Spec: [[King UI Overhaul]]. Every changed or new script in Studio matches `~/fruit-empire/src-m2` exactly (24 of 24, by length and hash). Tested in Play with `StudioNoSave` on; Samarth's save was never written (rebirths 0, saveCount 599 before and after).

## What M2 put in
- **UIKit**, one module every new screen uses: the kit's image ids, the four text sizes, white Fredoka with an outline, pills, bars, titles, icons that draw nothing while their id is 0, and the "one panel at a time" list.
- **RoyalUI split into six parts** in three layers:
  - the King chip and its one action pill (top left);
  - the event card (top centre) and the herald line under it;
  - the big fight words (stingers);
  - the Kingdom panel with four tabs: Throne, Decrees, Styles, Records;
  - the King's challenge dialog;
  - the titles and tags in the world.
- **Every royal word rewritten** in CrownConfig: plain text, no emoji, Title Case.
- **Server text pass** on the four Crown scripts, plus the RoyalQuery limit (6 answers per 2 s).
- **The 28 castle signs and 2 portraits** reskinned in the kit. A backup of every sign is in `ServerStorage.RoyalSignsBackup_2026-10-04`.
- **The garden menus** all close each other through one list. The Shop's King tab uses kit pills.

## Checked in Play
| Screen | What it shows |
|---|---|
| ![[m2-chip-hud.jpg]] | **King chip.** "You Are King", the name, and a live badge ("Protected 0:24", then "Reign 1:34"). It hides while any panel or dialog is open. |
| ![[m2-panel-throne.jpg]] | **Kingdom panel, Throne tab.** The King's card with avatar, seven facts, and the King's two actions (Royal Trial, Passage). |
| ![[m2-panel-styles.jpg]] | **Styles tab.** Reputation bar, six crowns (Worn, Wear, or the red need), and Open Shop, which opens the Shop on its King tab. |
| ![[m2-panel-records.jpg]] | **Records tab.** This server's reigns, the Eternal King, your record, and eight medals with progress bars. |
| ![[m2-challenge-dialog.jpg]] | **Challenge dialog.** The garden's own confirm art, the question, Yes and No, and a draining timer. It closes itself when the time runs out. |
| ![[m2-decree-strip-herald.jpg]] | **A decree.** The Merge Rush card at top centre (merges, timer) and the herald line under it. Issued from the Decrees tab, the news became a green toast, because a panel was open. |
| ![[m2-practice-bout-arena-focus.jpg]] | **Practice bout.** Arena focus hides the whole garden HUD except Mute. The bout card and its herald show. |
| ![[m2-stagger-tag.jpg]] | **Stagger tag** over the fighter, with a fill that turns from green to red. |
| ![[m2-castle-altar-board.jpg]] | **A reskinned castle board** (the altar): the kit panel, white words, live King name. |
| ![[m2-shop-king-tab.jpg]] | **Shop King tab.** Kit Equip pills (green) and Equipped (grey). |

**Other checks that passed:**
- **Scripted look check:** 189 royal labels, no emoji, all Fredoka, all on the 4-size ladder, no code-drawn frames. The castle boards have no emoji either.
- **Query limit:** 20 fast calls get 6 answers, then nil. An unknown question gets nil.
- **One panel at a time:** the Kingdom panel's backdrop stops a garden panel opening over it. The royal HUD hides while any panel shows.
- **Console:** no errors from any M2 script.

## Bugs found on screen and fixed
1. **Every kit image size was wrong.** The builder agent guessed them from screenshots. That broke the pill art (dark bands, white blocks), cut the cards short, and made panel centres see-through. All sizes were read from the uploaded images in Studio, and the corner sizes and pill cuts were measured from their pixels. ([[m2-panel-styles-before-fix.jpg|before]] / [[m2-panel-styles.jpg|after]])
2. **Bars had no fill.** The stand-in was the Rate bar art, which is translucent black and can't be tinted. Bars now use the cream Shop-tab art, tinted.
3. **Text was cut off for players with a larger Roblox Text Size.** Samarth's Studio is set to "Larger". It adds about 10 screen pixels to every text size, even inside a scaled panel. Text is now measured with a real hidden label under the same scale as where it goes.
4. **The challenge question lost "Accept the duel?"** at a larger Text Size. The question now gets the room above the buttons.
5. **Medal names broke mid-word.** The medal cells were widened a little.

## Still open
- **Placeholder art (from [[Asset Requests]]):**
  - Titles: plain words until the title images exist.
  - Icons: none drawn while their id is 0.
  - Pills: cut from existing buttons.
  - Bars and chip: kit stand-ins.
- **At the "Larger" Text Size,** a few fixed labels still end in "...": tab words ("Decr..."), crown needs, and long medal names. The spec allows this (test 27). At the normal size they fit.
- **Not testable solo:** the 3-player tests (sound and stinger scope, the strip choice near and far from the arena) and the device sizes.
- **Out of M2 (Royal Combat, M3):** the old combat key hint line at the bottom of a bout still uses middle dots.
- **Garden layout from before M2:** the red warning toast can touch the top buttons on a 1529x736 screen.
- **Spec question from M2's review:** on short phones, the event card at 112 px reaches into the notice lane. It steps aside while a notice shows. The spec still needs a call: let it move up, or make it smaller.

## What Samarth does before players see it
1. Ctrl+S, then Publish. This ships M1 and M2 together.
2. Shut down all servers.
