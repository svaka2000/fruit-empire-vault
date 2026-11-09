---
tags: [fruit-empire, pets, eggs, bats, ui, plan, steal-an-egg]
status: current
updated: 2026-11-09
---
# Reaching Steal an Egg's level

What makes Steal an Egg's pets, eggs, bats and GUI look better than ours, and exactly what we change. Written 9 Nov 2026 from Samarth's references ([[Steal an Egg References]]), the eldorado.gg list of all 243 pets, and our own play-test shots. Back to [[Knowledge Base - Start Here]].

> [!important] Tweak ours, don't redo
> Samarth: "instead of redoing everything just make tweaks to ours to make it on this level." We keep our animals, their colours, their rig parts and the game code. We change how the pieces are shaped, lit and animated.

## 1. Their pets, looked at closely

- **Their cheap pets look like ours.** Chicken, Dog, Frog, Toucan and the rest of the low tiers are simple blocky voxel animals with studs.
- **Their best pets are sculpted.** The Eternal and Divine pets, and every pet from the newer worlds (Cherry Blossom, Titan Temple, Angels & Demons, Enchanted Forest, the events), are built from chunky angled pieces: bevelled blocks, tapered horns and spikes, rounded shells, wedge-shaped armour plates. There are no stair-step edges.
- **Studs everywhere.** Every face has studs, the angled ones too.
- **Few colours, used well.** Two or three main colours per pet, a darker trim along plate edges, and one bright accent (gold, a gem).
- **Glow in small places.** Eyes, inlays, cracks, runes and gems glow. Big faces don't.
- **Particles and an outline.** Leaves, sparkles or embers drift off the pet. The 602 kg pet has a thin white outline around it.
- **A prop with a story.** A tree grows on the Royal Skywhale, Jitterbug plays a banjo, the ArchAngel holds a spear, the Petal Beetle wears flowers.
- **Big.** Weight sets size. A 602 kg pet stands taller than a player.
- **The ladder.** Each rarity step adds pieces, glow and particles.

## 2. Ours now

- Each animal is a list of coloured boxes on a block grid (`~/fruit-empire-pets/forest-animals/.../<name>.json`). `tools/rig_build.py` cuts it into rig parts (Head, LegFL, Tail...) and writes one mesh per part.
- In the game: MeshParts with the place's "Studs" MaterialVariant, colour in the vertex colours, Motor6D joints, procedural animation (`PetAnimator`). See [[Art Style - Studded Voxels]] and [[Pets, Eggs and Fuse Machine]].
- So our pets sit at their common tier: small and pixel-stepped.

## 3. Exactly how we reach their level

**Geometry: sculpt each rig part instead of voxelising it.**
- Add shaped pieces to the spec: bevelled box, tapered box, rounded box, spike or horn (a cone with 4-6 sides), shell (a low sphere), plate (a thin bevelled wedge), each with a rotation. Today's boxes stay valid.
- A new builder (Blender, through the Blender MCP) meshes each piece low-poly and flat-shaded, with a one or two segment bevel. It paints the bevel a shade darker (the trim) and the top faces a shade lighter. It writes the same part names and pivots, so `PetAnimator`, the rigs and the game code don't change.
- Budget: about 1,500 to 6,000 triangles per pet, one MeshPart per rig part.

**Studs on sculpted faces.**
- Keep the "Studs" MaterialVariant. DevForum reports say MaterialVariants ignore a mesh's UVs and project the texture onto each face themselves, so angled faces get studs too. Check faces near 45 degrees in Studio. If studs smear there, use a SurfaceAppearance with baked studs for that pet only (it needs texture uploads).
- Sources: [How to UV map for MaterialVariants](https://devforum.roblox.com/t/how-to-uv-map-for-materialvariants/2164215), [MaterialVariant obey MeshPart UV maps](https://devforum.roblox.com/t/materialvariant-obey-meshpart-uv-maps-property/2205123), [MeshPart material variant not even](https://devforum.roblox.com/t/meshpart-using-a-material-variant-is-not-even-around-the-mesh-how-to-fix/2743462).

**Detail by rarity** (our rarities: Common, Uncommon, Rare, Epic, Legendary, Mythic, Astral, Eternal, Transcendent):
| Rarity | What it gets |
|---|---|
| Common to Rare | The sculpt pass only: today's shapes, bevelled and rounded, no stair-steps |
| Epic, Legendary | Plus one or two accessory pieces (horns, a flower, a gem) and glowing eyes |
| Mythic, Astral | Plus armour plates with trim, small glowing inlays (Neon at about 62% of its colour, the bloom rule), themed particles |
| Eternal, Transcendent | Plus a story prop, an aura, idle effects and the white outline |

**Size.** Scale with weight like theirs, with a cap so a pet fits its plot.

**Eggs.** Same pass. Their eggs are shells of rounded petals or plates around a glowing core, with sparkles. Ours keep the zigzag Top and Bottom split, built from petals and plates, with a glowing core and sparkles that step up by rarity.

**Cost limits.** A client draws at most 31 Highlights, so the white outline is for the player's own Buddy and the top rarities only. Particle rates stay low.

## 4. Pets that act

Samarth: "make the pets super animated as well they need to walk around sit down talk emojis etc like you should have full control of them."
- A behaviour brain for owned pets, on the client (we already draw rigs there): follow, wander the plot, sit, lie down and sleep, sniff and eat fruit, play (hop, spin, chase its tail), watch the player, celebrate (money, a hatch, a rebirth), get scared (a bat hit), greet other pets.
- Emote bubbles over the head: heart, laugh, music, sleep, "!", "?", sweat, angry, star eyes, money eyes, fruit. Our rule is no emoji, so these are our own small pictures in a speech bubble.
- Player control: click a pet for Sit, Stay, Follow and a Trick.
- All poses are procedural Motor6D in `PetAnimator`, so no animation uploads.

## 5. Bats, better than theirs

- **Theirs:** each world has a bat with its own material and particles (bubbles, sand swirls, fire, glowing ice, spinning cosmic rings). Late ones are swords, axes, spears and hammers. Event bats are jokes (a banjo, a scythe). They unlock from the Index ("Equip Bat!").
- **Ours:** nine fruit bats from Applewood to Soul Fruit ([[Nine Bats]]).
- **To beat theirs:**
  - Sculpt them with the same pass as the pets.
  - Give every bat three effect layers: an idle aura (particles and a light), a swing trail in its colour, and the hit burst it already has.
  - Turn the late tiers into fruit weapons: a pineapple mace, a flaming pepper axe, a crescent cherry scythe-spear, a halo spear with the Soul Fruit.
  - Give it an unlock moment: an "Equip Bat!" reveal with rays.
  - Show the bat in the Inventory's Gear tab and in hotbar slot 1 (see 6).

## 6. GUI, better than theirs

- **Fonts.** FredokaOne for everything, as [[UI Kit and Text Rules]] says (theirs is the same font). Big headlines get a much thicker black outline, about a sixth of the text size.
- **Colours.** Colour only means something: money green, a rarity's own colour for its name, gold for bar fills. Title bar colour by panel family: green for selling and the shop, blue for info and the Index, purple for fusing, peach for hatching, gold for rewards.
- **Inventory.** Tabs for Pets, Eggs and Gear (the bat, the pickaxe). A ten-slot hotbar along the bottom replaces Roblox's default one: the bat in slot 1, the pickaxe in slot 2 in the mine, eggs stacked.
- **Dopamine.** Panels pop with a little overshoot. Cards pop in one after another. Numbers count up. A shine sweeps across title bars and rare cards. Rare items sparkle, and the top rarity's border cycles colour. Buttons squash when pressed. Coins fly to the counter, and big moments get confetti and a sound.
- **Top notifications.** Huge white words with a thick outline at the top centre, the key part coloured, several stacking. Icons are pictures, not emoji.
- **Chat messages.** Server lines in colour: events in one colour, egg names bold or in their rarity colour, places in theirs. Chat can't show pictures, so chat lines are words only.
- **New art:** from ChatGPT with [[ChatGPT Art Prompts]], with words added in game or in Figma. Not Claude Design: Samarth doesn't like its fonts and colours (9 Nov).
- New or restyled UI still needs Samarth's and Weilin's OK through a handoff ([ui/README.md](https://github.com/svaka2000/fruit-empire-handoffs/blob/main/ui/README.md)).

Related: [[Steal an Egg References]], [[ChatGPT Art Prompts]], [[Pets, Eggs and Fuse Machine]], [[Nine Bats]], [[UI Kit and Text Rules]], [[Art Style - Studded Voxels]].
