---
tags: [fruit-empire, project, monetization, robux]
status: waiting
updated: 2026-11-08
code: ~/fruit-empire/robux-shop
handoffs: [29]
---
# Robux shop

Back to [[Project Status Board]].

## What players get

The shop's Robux page shows five pass cards and six shard pack cards with their price and icon from Roblox, "Owned" or "Soon", and opens Roblox's purchase prompt. (STATE.md "25 Oct: the Robux shop")

| Item | What it does |
|---|---|
| x2 Spawn (pass) | Doubles the spawn rate |
| Auto Merge (pass) | Merges one pair every 2 s |
| Merge All (pass) | Unchanged; the only pass that exists on Roblox (id 1982540690) |
| Less Lock Cooldown (pass) | Halves the Lock cooldown |
| Less Explosive Cooldown (pass) | Halves the explosive cooldown |
| Shard packs: Handful, Pouch, Bag, Chest, Vault, Mountain | 100 / 220 / 480 / 1,000 / 2,200 / 4,500 units times one merge's shard drop at your stage |
| Skip Rebirth (product) | In `GardenService.Purchases` since 10 Oct; its button is back on the Rebirth panel |

The page header says SHARDS (it said GEMS), and the Gift buttons are hidden.

## Samarth's ask

25 Oct: "help me setup ALL gamepasses.", after Weilin asked for gamepasses and a way to buy shards. (DECISIONS.md)

> [!important] Nothing new can be bought yet
> The four new passes and seven products are not made on Roblox, so their ids are 0 and the cards say "Soon". They must be made on **Weilin's account**, which owns the game (like Merge All). When Samarth says they exist, a Studio turn puts the ids into `GardenConfig.Shop` (`Passes[].id`, `Packs[].id`) and `Config.RebirthSkipProductId`, then tests real prompts. New passes stay off sale until publish. (memory `project_fruit_empire_robux_shop.md`; STATE.md "Next up")

Suggested prices given to Samarth: passes 129 / 199 / 79 / 99, Skip 79, packs 49 / 99 / 199 / 399 / 799 / 1,499 Robux. (memory `project_fruit_empire_robux_shop.md`)

## Where it lives

- **In the place:** `ReplicatedStorage.GardenConfig.Shop`, `ServerScriptService.GardenService.Passes`, `StarterPlayerScripts.RobuxShop`, `ServerStorage.ZZTools.RobuxPageBuilder`; small changes to `GardenConfig`, `GardenService`, `Merging`, `Purchases`, `Ambient`, `RebirthLock`, `Explosives`. Backup `ServerStorage.RobuxShop_backup_2026-10-25`.
- **Icons:** Samarth's ChatGPT versions in `~/Downloads/fruit-empire-shop-icons/chatgpt/`; Claude's originals on the [shop icons canvas](https://claude.ai/artifact/LiXjHcL2S5qAukYwypaPnn).
- **Test doors:** `DevPass_<key>` attribute, `RoyalDev:Invoke("passPerks")`, `RoyalDev:Invoke("packReceipt", key)`.
- **Handoff:** [PR #29](https://github.com/svaka2000/fruit-empire-handoffs/pull/29).

## Open items

- Not tested: pressing a Buy button (MCP clicks did not reach the buttons), a real purchase prompt, two players, a phone, the five screen sizes.
- Merge All's icon on Roblox is the Rebirth art.
- Weilin's OK on the perks, packs, prices and page changes.
- The Troll Potion's Robux version is parked ([[Troll Potion]]). Robux hatching is ruled out in the pets design ([[Pets, Eggs and Fuse Machine]]).
- On Roblox, the game's name still starts with "[STEAL!]" and its description still talks about stealing. Only the owner's account can change that on the Creator Dashboard. (STATE.md "Open items")

Sources: [STATE.md](https://github.com/svaka2000/fruit-empire-handoffs/blob/main/STATE.md); memory `project_fruit_empire_robux_shop.md`.
