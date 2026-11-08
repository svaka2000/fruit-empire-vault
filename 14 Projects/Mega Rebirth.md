---
tags: [fruit-empire, project, rebirth, mega-rebirth]
status: built
updated: 2026-11-08
code: ~/fruit-empire-mega-rebirth
handoffs: [31]
---
# Mega Rebirth (and the Aether cloud plot, on hold)

Back to [[Project Status Board]].

## What players get

At Rebirth 8 with all 64 fruit found, the Rebirth panel offers a **MEGA REBIRTH**. It needs the full Index, 60M shards (spent) and cash (checked); each later one needs twice as much. It resets you to Rebirth 0, keeps the Index, Plot Size, pickaxes, eggs, potions, Sky Stars and the rest of your shards, and gives cash x2 and merge shards x1.5 each time. (STATE.md "25 to 29 Oct: Mega Rebirth")

- **Floors stack:** floors = 1 + Mega Rebirths, 16 studs apart, up to 14. Fruit, merges, Merge All, TNT, the Lock dome, bats and E prompts work per floor.
- **Floor styles:** the first Mega Rebirth asks for **Sky Keep**, **Hanging Gardens** or **Sky Platform**. Switch from floor 1's lift pad, every 5 minutes at most. (Looks described in [[Colour Palettes]].)
- **A lift pad** on each floor shows Floor buttons to ride between them.
- **The show:** a flash and title, a high camera view, the new storey building in, and you land on the top floor.
- **Plot limits:** the beanstalk lets Plot 1 hold 3 floors, Plot 3 hold 6, Plot 2 hold 13; the rest 14. A Mega Rebirth past your plot's limit moves you to a free plot with room.

## Samarth's words

- 25 Oct, 16:17: "add mega rebirths. once you get to the max fruit then instead of rebirthing you get the option to mega rebirth. this vertically scales your plot and adds a second floor. use claude design to make this look EXTREMELY COOL and seamless. it needs to be perfect."
- 25 Oct, 19:00, on floors: "maybe make an option" / "that lets them chose" / "we need this to be repeatable tho so that multiple rebirths stack it. players can always switch with a cooldown." / "so im saying that if a player has 2 mega rebirths he will get 3 floors" / "shouldnt exceed the sky island height for now".
- The same day: rebirths should be "exponentially harder and harder ... extremeley hard" so players hunt eggs. That became PR #28 ([[Rebirth Panel]]).

Source: memory `project_fruit_empire_mega_rebirth.md`; DECISIONS.md.

> [!note] On hold: the Aether cloud plot
> Samarth, 25 Oct: "once you mega rebirth enough you can start scaling your plot in the aether as well which will be on a floating cloud. but we will hold off on this so just keep it in your memory." Not designed, not built. Don't start it until he asks. The floor count is data, so a plot could later be copied onto a cloud.

## Where it lives

- **In the place:** 8 new scripts (`GardenConfig.MegaRebirth`, `ServerStorage.PlotFloorDecor` with `Keep`, `Gardens`, `Sky`, `GardenService.FloorLift`, `StarterPlayerScripts.MegaShow`, `FloorStyleMenu`) and 25 changed; 15 light presets in `ServerStorage.PlotKit.DecorFx`. Save fields `megaRebirths`, `floors`, `floorStyle`, `floorStyleAt`; a save with a Mega Rebirth is version 2. Backup `ServerStorage.Backup_2026-10-25_before_mega_rebirth`. (STATE.md)
- **Local:** `~/fruit-empire-mega-rebirth` (`PLAN.md`, `SPEC.md`, `SPEC2.md`, `BUILD-NOTES.md`; `src/`, `before/`, `after/`, `deploy/`, `deploy-updates/` u1 to u5, `tests/run.sh`). Change installed scripts only through this repo: patch or `src/`, run the tests, then `tools/make_update.py`. (memory `project_fruit_empire_mega_rebirth.md`)
- **Canvas:** [floor styles](https://claude.ai/artifact/Uu3rtKkKe4gCPbqHf9PQ2K). **Handoff:** [PR #31](https://github.com/svaka2000/fruit-empire-handoffs/pull/31).

> [!danger] Publishing
> Publish with "Migrate to latest update" and never roll back below this version: an older server turns a Mega player away until servers update. **Before a publish: the rejoin test** (leave and rejoin with floors, StudioNoSave off, on a test place). (STATE.md)

## Open items

- Not tested: rejoin with floors, two players, a phone, the five screen sizes, Hanging Gardens and Sky Platform at 14 floors in Play, the Lock dome at 14 floors, bats and TNT on upper floors.
- No style pictures yet (`Config.Mega.Styles[*].Image` is 0).
- Weilin's OK on the new UI (Mega panel words, FLOOR STYLE panel, lift buttons, show titles), the three floor looks and the moved trees.
- Next turn in the queue: the Rebirth panel's Mega look and a "Floor style" button ([[Rebirth Panel]]).

Related: [[Plot Castle]] (Sky Keep grows the castle), [[Rebirth Panel]], [[Workspace Map]] (heights).
