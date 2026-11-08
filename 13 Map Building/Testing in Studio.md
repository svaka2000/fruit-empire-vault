---
tags: [fruit-empire, testing, studio]
status: current
updated: 2026-11-08
---
# Testing in Studio

How to Play-test without hurting anyone's save, and the test shortcuts. Back to [[Knowledge Base - Start Here]].

> [!danger] Before every Play test
> ```lua
> workspace:SetAttribute("StudioNoSave", true)
> ```
> When Studio's "Enable Studio Access to API Services" is on, a Play test loads and writes the tester's **real** save (autosave every 60 s, and a last save on Stop). StudioNoSave turns normal save writes into no-ops. Say in the handoff whether you left it on. Read it with `print(workspace:GetAttribute("StudioNoSave"))`.
> Source: [PLACE.md](https://github.com/svaka2000/fruit-empire-handoffs/blob/main/PLACE.md) "Saves".

## Save traps

- StudioNoSave does **not** stop the DevPanel's "fresh" and "seed" buttons. In September a test emptied a real save that way. (PLACE.md)
- If your own account is in a live server, a Play test on that account can't load your save; it waits about 2.5 minutes, then kicks you. Leave the live game first. (PLACE.md)
- Test fruit tagged to Samarth's plot once got saved as real when Play stopped mid-test. The fix is the "guard lock": right after the profile loads, write a lock into the save record so the session can't write, and finish inside the 60-120 s before its first autosave. (memory `project_fruit_empire.md`, 3 Oct entry)
- The save store keeps old versions about 30 days, so a bad save can be put back. (STATE.md "Samarth's save")

## Test shortcuts (dev doors)

All work only in Studio Play tests. The full list is in PLACE.md "Test shortcuts".

- **Player attributes, set on the server as numbers:** `DevSetCash`, `DevSetShards`, `DevSetRebirths` (0 to 8), `DevFillIndex`, `DevGiveTools`, `DevSetPlotSize`, `DevExplosivesSelf`.
- **`ServerStorage.GardenDev:Invoke(action, userId, ...)`:** place, wipe, handmerge, mergeall, clear, rebirth, plot, drink, effects, capture, explosives and more.
- **`ServerStorage.RoyalDev:Invoke(cmd, ...)`:** bats, crate, swimming, Sky Stars (`skyStars "reset"` / `"almost"`), and doors added by later projects:
  - mine: `mine enter`, `ore near`, `ore hit`; pickaxes: `pick near <userId> Gus`, `pick buy|equip <userId> <tier>`, `pick ... profile` ([[Mine]])
  - digging: `dig state|cycle|egg|near|wall|hit|take|reset` ([[Mine Digging and Cave-in]])
  - shop: `passPerks`, `packReceipt <key>`, and the `DevPass_<key>` player attribute ([[Robux Shop]])
- **DevPanel (F4):** only the game owner's account can open it, so Samarth's test account does not see it. (PLACE.md)

## Real input

- Use the MCP's `user_keyboard_input` (`keyPress` E fires the prompt the client shows, once) and `user_mouse_input` with `instance_path`. (memory `reference_roblox_studio_mcp.md`, 22 Oct)
- In the Robux shop test (25 Oct), MCP clicks did **not** reach the GUI buttons, so no Buy press was tested. In the pickaxe test (22 Oct), real clicks did work. Why they differed is not recorded. `[unclear]` (memory `project_fruit_empire_robux_shop.md`, `project_fruit_empire_mine.md`)

## Recipes that worked

- **A full egg race:** StudioNoSave on; set `DevSetRebirths 8` and `DevSetShards 1000000`; `pick near Gus`; `pick buy 9` (Halo Pick); step toward the egg 3.5 studs at a time with `dig hit` until `dig state` says open; `dig take` from about 6 studs. Close the join popup first. (memory `project_fruit_empire_mine.md`, 24 Oct)
- **Force an egg now:** `game.ServerStorage.RoyalDev:Invoke("dig", "egg")` on the Server side. (memory `project_fruit_empire_mine.md`, 23 Oct)
- **Rebirth flow:** dev door attributes plus `GardenDev "handmerge"`, then click through with `user_mouse_input`. (memory `project_fruit_empire.md`, M1 entry)

## What is almost never tested yet

Across nearly every project, STATE.md "Open items" lists the same gaps:

- **Two or more real players** (only stand-ins so far).
- **A real phone or tablet**, and the frame rate there (the mine's lights, a stage 9 bat, a dug shaft).
- **The five screen sizes** with `UIOverlapCheck` (often only worked out by calculation).
- **A real save and load** of new save fields (`eggs`, `crystal`, `floors`).
- **Listening to the sounds.**

See [[Project Status Board]] for each project's list.

Related: [[Studio MCP Traps]], [[Studio Turn Rules]], [[Hard Rules]].
