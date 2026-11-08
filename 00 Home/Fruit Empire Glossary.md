---
tags: [fruit-empire, glossary, reference]
status: current
updated: 2026-11-08
---
# Fruit Empire glossary

The words the other notes use, in plain terms. Back to [[Knowledge Base - Start Here]].

## The game

- **Fruit Empire:** the Roblox game "[STEAL!] Fruit Empire" that Samarth and his partner Weilin make. The "[STEAL!]" in the name is old: stealing was taken out for good on 3 Oct 2026 (see [[Hard Rules]]).
- **Plot:** one of the 12 gardens on the Main Island. Each player gets one.
- **Merge:** carry a fruit onto a matching fruit; the pair becomes the next fruit up the **fruit ladder** (64 fruit, Blueberry to Soul Fruit).
- **Rarity:** the 9 tiers of fruit, Common to Transcendent. Each has its own colour (see [[Colour Palettes]]).
- **Stage:** your rebirth count plus 1. Stage 1 has Common fruit only; stage 9 reaches Transcendent.
- **Rebirth:** a reset that unlocks the next rarity. It needs cash, shards and a full Index.
- **Mega Rebirth:** the reset after Rebirth 8. It stacks one more **floor** (a storey) on your plot. See [[Mega Rebirth]].
- **Shards:** the second money. They buy potions, explosives, pickaxes and cosmetics.
- **Index:** the book of every fruit you have found.
- **Aether / Cloud Garden:** the cloud island at the top of the beanstalk. It holds the 12 daily Sky Stars and the Fuse Machine.

Source: the "Words used here" part of [STATE.md](https://github.com/svaka2000/fruit-empire-handoffs/blob/main/STATE.md).

## Studio and the chats

- **Studio / the place:** Roblox Studio, and the one Roblox place both partners edit (place id 88602028667228). The place is the source of truth for the game's code.
- **Team Create:** Studio's shared editing. A change by one person is in the place for both at once.
- **MCP / the Roblox_Studio MCP:** the bridge that lets a Claude chat run code in Studio. Its main tool is `execute_luau` (runs Luau like Studio's command bar).
- **Edit mode / Play test:** Edit mode is Studio's building view. A Play test runs the game inside Studio.
- **Publish:** pushing the place to the live game. Only a person publishes, never a Claude.
- **Handoffs repo:** the private GitHub repo `svaka2000/fruit-empire-handoffs`. It is the shared log between Samarth, Weilin and both Claudes. See [[Handoffs Repo Links]].
- **Handoff:** one page in that repo per turn: what changed, where, how it was tested, what is next.
- **Studio turn:** a stretch of time when one chat (one writer) may change the place. See [[Studio Turn Rules]].
- **StudioNoSave:** an attribute on `workspace`. When it is `true`, a Play test cannot write anyone's real save.
- **Builder:** an Edit-mode tool script that builds part of the world. Most have `run()`, `undo()` and `status()`. See [[Builders and ZZTools]].
- **ZZTools:** the folder `ServerStorage.ZZTools` that holds the builders in the place.
- **Backup folder:** a dated folder in ServerStorage, such as `Backup_2026-10-22_before_bat_tiers`, holding what a change replaced, so it can be undone.
- **Fingerprint (h32):** a number computed from a script's text. Same length and same fingerprint means the exact same script. See [[Deploy Method]].
- **Dev door:** a test shortcut that only works in Studio Play tests, such as the `DevSetShards` attribute or `ServerStorage.RoyalDev:Invoke(...)`. See [[Testing in Studio]].

## Look and UI

- **UI kit:** the ModuleScript `ReplicatedStorage.UIKit`. It knows every piece of Samarth's UI art and places it. See [[UI Kit and Text Rules]].
- **9-slice:** a way to stretch an image: the 4 corners keep their size, the edges stretch one way and the middle stretches both ways. Roblox sets it with `SliceCenter` (the middle box, in image pixels) and `SliceScale` (how big the corners draw).
- **FredokaOne:** the one font the UI uses, white with a black outline (`UIStroke`).
- **"Larger" text size:** a Roblox setting (`GuiService.PreferredTextSize`). Samarth's Studio uses it. It makes every label about 10 screen pixels taller.
- **Words pass:** measuring every label's words with `Kit.textWidth` before a panel is laid out, so names fit even at Larger text.
- **Studs / MaterialVariant "Studs":** the place's material variant that draws Lego-like studs on Plastic. The whole game is studded.
- **Voxel:** a small cube. Models are built from voxels about 0.2 studs wide.
- **Neon / Bloom:** Neon is a glowing material. Bloom is the light effect that makes bright things glow. Strong Bloom makes big Neon areas wash out to white.
- **Claude Design canvas:** a private claude.ai artifact page where design ideas are drawn before building. See [[Claude Design Canvases]].
- **GLB / Import 3D:** GLB is a 3D model file. Import 3D is Studio's importer for it.
