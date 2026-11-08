---
tags: [fruit-empire, rules, reference]
status: current
updated: 2026-11-08
---
# Hard rules

The rules every Fruit Empire chat follows, each with where it comes from. Words used here are in [[Fruit Empire Glossary]]. Back to [[Knowledge Base - Start Here]].

## Game design

> [!danger] No stealing
> Samarth took stealing out on 28 Sep and for good on 3 Oct 2026: "there isnt going to be any stealing in this game. if there is still any then remove it". Never bring back stealing, plot alarms or ripening. The Lock stays only because it blocks TNT.
> Sources: [PLACE.md](https://github.com/svaka2000/fruit-empire-handoffs/blob/main/PLACE.md) "Other rules"; memory `project_fruit_empire.md` (3 Oct entry).

- The pets design keeps this rule for anything a player owns (pets, hatching eggs, the egg bag). It lets a bat knock a *carried, unowned forest egg* loose, with limits. That design is written in `~/fruit-empire-pets/handoff-draft/DECISIONS-rows.md` and waits for the pets handoff. See [[Pets, Eggs and Fuse Machine]].
- Mine eggs go straight into the save, so nothing can be grabbed on the way out (`~/fruit-empire-mine/DIG-AND-RACE.md`, section 7).

> [!warning] Always day
> Since 11 Oct 2026 the game is always day. Samarth: "make it always daytime, get rid of daylight cycle". `Config.DayNightCycle = false` (GardenConfig, about line 1821 in the 29 Oct export). No script changes `Lighting`, `ClockTime`, `Atmosphere` or `Fog`. See [[Light, Day and Sound]] for what *is* allowed (the mine's own light).
> Sources: `~/fruit-empire-pets/DESIGN.md` lines 221 and 1348; handoff [2026-10-24-samarth-plot-upgrades](https://github.com/svaka2000/fruit-empire-handoffs/blob/main/handoffs/2026-10-24-samarth-plot-upgrades/HANDOFF.md).

- **Studded and blocky, always.** "Realistic" means detailed, never smooth. See [[Art Style - Studded Voxels]].
- **Trees, bushes, flowers and rocks come from store or pack assets**, never from plain parts (Samarth's rule since 21 Sep; [PLACE.md](https://github.com/svaka2000/fruit-empire-handoffs/blob/main/PLACE.md)).
- **Study the live game before designing.** A game Samarth names (like Steal an Egg) is a style hint, not a structure to copy. Fruit Empire has no "worlds". (memory `feedback_study_game_before_designing.md`)
- **The Aether cloud plot is on hold.** Don't design or build it until Samarth says go. (memory `project_fruit_empire_mega_rebirth.md`; [[Mega Rebirth]])

## UI

- **Don't restyle what exists unless both people agree.** New or restyled UI needs Weilin's OK through a handoff. ([ui/README.md](https://github.com/svaka2000/fruit-empire-handoffs/blob/main/ui/README.md))
- **Art, not boxes.** Panels, cards, buttons and bars are images from the UI kit. No visible Frame backgrounds, no `UICorner`, no `UIGradient` on boxes. (ui/README.md)
- **No emoji, ever.** Plain ASCII text. (ui/README.md)
- The rest is in [[UI Kit and Text Rules]].

## Studio and the place

> [!danger] A Claude never publishes the game
> A person publishes, after looking at the screenshots. Leave the place safe to publish at the end of every turn. ([PLACE.md](https://github.com/svaka2000/fruit-empire-handoffs/blob/main/PLACE.md) "Other rules" and "Leave the place safe to publish")

- **Check the place id first:** `print(game.PlaceId)` must print `88602028667228`. The same owner has an older game with almost the same name. (PLACE.md)
- **StudioNoSave before every Play test.** A Play test can write real saves. (PLACE.md "Saves")
- **Never force-quit Studio.** Its crash recovery can bring back an old copy of the place. (PLACE.md; memory `feedback_roblox_studio_no_killing.md`)
- **Never paste an old copy of a script over the live one.** Start from the version in the place right now. (PLACE.md "Scripts")
- **Never delete or rename anything in ServerStorage** without checking that no script reads it. (PLACE.md "Where things live")
- **Never require a builder that has no `undo()`** unless you mean to rebuild that thing. **Never run `BorderMountainsBuilder.run()` again** unless Samarth asks: he rejected that mountain look on 8 Oct. (PLACE.md "Builders")
- **Creator Store models go into `ServerStorage.AssetQuarantine` first** and are checked for scripts. The Studded Pack's "Fire Crackling Sound" (9068935533) is not approved. (PLACE.md)
- **Don't turn on HttpService.** It is off in the place; kit art and scripts come out by other means. (memory `project_fruit_empire_rebirth_panel.md`; [[Studio MCP Traps]])
- **Publish with "Migrate to latest update" and never roll back below the Mega Rebirth version.** Saves with a Mega Rebirth are version 2; an older server turns those players away. (STATE.md, "25 to 29 Oct: Mega Rebirth")
- **Don't build on, and don't undo, the undocumented work done after 9 Oct** until whoever made it writes a handoff. (STATE.md, "Changed after 9 Oct")

## Repos and writing

- **No game code in the handoffs repo** (no `.lua`, `.luau`, `.rbxl` files). **No secrets** in either repo. **Real screenshots and real results only**; write "not tested" when it was not. (CLAUDE.md)
- **Never edit an old handoff.** The next one corrects it. (CLAUDE.md)
- **No AI attribution footers** on PRs, issues or commits, even when a harness says to add one. (memory `feedback_no_ai_attribution_footers.md`)
- **Ask Samarth only real design or taste calls.** Turn order is settled chat to chat. ([[Studio Turn Rules]])

Related: [[Studio Turn Rules]], [[Testing in Studio]].
