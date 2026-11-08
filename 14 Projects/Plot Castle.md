---
tags: [fruit-empire, project, plot, castle]
status: built
updated: 2026-11-08
code: ~/fruit-empire-plot-upgrades
handoffs: [9, 24]
---
# Plot Castle (plot upgrades)

Back to [[Project Status Board]].

## What players get

Buying **Plot Size** at the sign beside your gate makes your plot bigger and builds more of the **Fruit Castle** round your pen, one step per level (sign levels):

1. **Garden:** bunting, lamp posts, fruit crates.
2. **Stone Keep:** stone walls with battlements, four towers with flags, wall torches, a crest over the gate.
3. **Fort:** tiled roofs, a gatehouse over the gate with a fruit shield, banners.
4. **Castle:** tall cone towers, side turrets.
5. **Fruit Palace:** white and gold, onion domes with giant golden fruit, a spinning crown over the gate, a fountain, a red carpet.

Flags, flames, the golden fruit, the crown and the fountain move on each client. A **LEVEL UP!** show plays on each buy: a big title, the pen rebuilding in a wave from the gate, a gold ring, confetti; the castle then builds up from its lowest piece over 1.6 s. The last level says MAX LEVEL!. Max Capacity plays a shorter CAPACITY UP!. (STATE.md "21 Oct: base-upgrade show" and "24 Oct: the Fruit Castle")

## Samarth's words

- 20 Oct, with a TikTok: "do this for base upgrades ... like it should be super insane when you upgrade", then "use claude design to make anything and everything you need do not limit yourself go all out". (memory `project_fruit_empire.md`, 20-21 Oct)
- 24 Oct: "the current plot designs when you upgrade your plot, are super basic ... let your imagination run wild and make the plot upgrades better". He picked the Castle from three concepts (Castle, Five Worlds, Fruit Factory) and said "make it look super realistic using claude design you know". Realistic = more detail, still studded. (`~/fruit-empire-plot-upgrades/src/PlotPenDecor.luau` header; memory `project_fruit_empire_plot_castle.md`)

## Where it lives

- **In the place:** `ServerStorage.PlotPenDecor` (the castle), `StarterPlayerScripts.PlotDecorLife` (the movement), `ServerStorage.PlotKit.DecorFx` (particles); changes to `PlotPenLayout`, `GardenService.PlotPen`, `GardenConfig.PlotPenThemes`, `GardenConfig.UpgradeShow`, `UpgradeShow`. **The switch:** `GardenConfig.PlotPenThemes` `Themes.Style = "castle"` (nil brings back the 21 Oct themes). Backups `ServerStorage.PlotUpgrades_backup_2026-10-24` and `ServerStorage.UpgradeShow_backup_2026-10-21`.
- **Local:** `~/fruit-empire-plot-upgrades` (git: `before/`, `src/`, `concepts/` with a Lune harness and Blender renderer, `canvas/`, `shots/`).
- **Canvases:** [castle concepts](https://claude.ai/artifact/57uK9K1QhRiS2z7h85AHP7); [LEVEL UP! titles](https://claude.ai/artifact/JksqFPNqBgW27AVdL3KJcU). Colours: [[Colour Palettes]].
- **Handoffs:** [PR #9](https://github.com/svaka2000/fruit-empire-handoffs/pull/9) (the show, 21 Oct themes), [PR #24](https://github.com/svaka2000/fruit-empire-handoffs/pull/24) (the castle).

> [!tip] Change the castle in `PlotPenDecor`
> Never edit a pen by hand: pens rebuild. (memory `project_fruit_empire_plot_castle.md`)

## Open items

- Weilin's OK on the castle replacing the five themes, and on up to 725 parts per level 5 pen (12 castles measured at about +1 ms render time).
- Not checked with two real players or on a phone. Nobody has listened to the show's sounds.
- The Lock dome is shorter than the level 4 and 5 towers.
- The window glow and gold beams only show at night, and the game is always day ([[Light, Day and Sound]]).
- The Sky Keep floor style grows this castle upward ([[Mega Rebirth]]).

Sources: [STATE.md](https://github.com/svaka2000/fruit-empire-handoffs/blob/main/STATE.md); memory `project_fruit_empire_plot_castle.md`.
