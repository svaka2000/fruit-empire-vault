---
tags: [fruit-empire, project, rebirth, ui]
status: built
updated: 2026-11-08
code: ~/fruit-empire/rebirth-panel
handoffs: [28, 30]
---
# Rebirth panel (and harder rebirths)

Back to [[Project Status Board]].

## What players get

- **Layout A (25 Oct):** the fruit cards sit in two rows of four. Each need has a bar under its numbers: gold while short, green with white numbers once met. Rewards are bigger. REBIRTH sits under them with Skip Rebirth beside it, and moves to the middle while Skip is hidden. The title bar says "Rebirths: N". (STATE.md "25 Oct: the Rebirth panel's layout A")
- **Harder rebirths (25 Oct):** from rebirth 2 on, the shards (spent) and cash (checked) needed are the old needs times 2 to the power of (stage - 1). Rebirth 2 needs 500 shards, rebirth 8 needs 30M. Rebirth 1 stays 50 shards. About 100 hours of focused play from the first rebirth to the last (an estimate, not measured). (STATE.md "25 Oct: harder rebirths")
- At Rebirth 8 the panel shows the Mega Rebirth's needs and rewards, still with the REBIRTH button art. ([[Mega Rebirth]])

## Samarth's words

- 25 Oct ~17:00: "please use claude design for the GUI, it doesnt look that good right now", with a WhatsApp note: "Send the entire frame and have it re arrange stuff to make it look good". After seeing layouts A, B and C: "A is fine".
- On rebirths: "first, make it exponentially harder and harder to rebirth. like extremeley hard. this will make players incentivised to go out and hunt for eggs."

Sources: memory `project_fruit_empire_rebirth_panel.md`; DECISIONS.md (25 Oct rows).

## Where it lives

- **In the place:** `ServerStorage.ZZTools.RebirthPanelBuilder` (moved the panel's own pieces, saving old values as `Was<Property>` attributes; `undo()` puts them back) and `StarterPlayerScripts.RebirthPanelLook` (bars, count, slices). `RebirthMenu`, `GardenConfig` and `RobuxShop` were deliberately left alone. Rebirth needs: `Config.RebirthShards` and `Config.RebirthCashSeconds`. Backup `ServerStorage.RebirthPanel_backup_2026-10-25`.
- **Local:** `~/fruit-empire/rebirth-panel` (`README.md`, the real kit PNGs in `kit/`, `tools/unkit.py`, `render.sh`).
- **Canvas:** [layouts](https://claude.ai/artifact/T4w3TH8QntJtAcmViUnKuP). **Handoffs:** [PR #28](https://github.com/svaka2000/fruit-empire-handoffs/pull/28) (harder rebirths), [PR #30](https://github.com/svaka2000/fruit-empire-handoffs/pull/30) (layout A).
- Layout facts: cards are 4 x 2 because `RebirthMenu`'s fitCards picks the fewest columns that fit; sliced boxes use `SliceScale = base x panelWidth / 984`. (memory `project_fruit_empire_rebirth_panel.md`)

## Next

> [!todo] The Mega look (next Studio turn in the queue)
> Build the canvas's Mega Rebirth state: tiles, a green pill, "Mega Rebirths: N", and a **Floor style** button that fires the `OpenFloorStyleMenu` BindableEvent (`Fire()` opens switch mode; `Fire("mega")` is the first Mega Rebirth's picker). The queue note gives this turn to the Robux shop chat. (STATE.md "Next up"; `studio-turn-rule.md`)

## Open items

- Weilin's OK on the restyle.
- Not checked on a phone or tablet (sizes only worked out from fitCards' sums), with two players, or with a real REBIRTH press through the new layout.
- Delete the backup once both people accept.

Related: [[Mega Rebirth]], [[Robux Shop]] (Skip Rebirth), [[UI Kit and Text Rules]].
