---
tags: [research]
topic: Merge a Nuke explosives, health drain, damage feedback, and how other Roblox games tier and price TNT and nukes
date: 2026-10-04
---

# Merge a Nuke - Explosives and Health Drain

## Summary

- **Merge a Nuke!** is made by the group "Nuke The Game" (place 128784467030899). It was created on 2026-05-21. On 2026-10-03 it had 435.8 million visits, 9,890 people playing, and 2.05 million favorites. Servers hold 8 players [1]. Nukes spawn on your island. You **merge** two nukes of the same tier into one nuke of the next tier. Every nuke earns cash each second, even while you are offline. You launch nukes at other bases to steal cash, and you "lock" your base to block attacks [1][15].
- Nukes do **not** delete things in one go. Every nuke sitting on a base has **HP** (health points). A guide site's baseline rule is: a nuke's max HP = 2 x the base damage of its own tier. Hits needed = HP divided by the attacker's damage, rounded up [7]. So a nuke hit by a nuke of the same tier survives the first hit and dies on the second.
- **Damage grows slowly. Cash grows fast.** Damage = 300 x 1.0905^(tier - 1). That is about +9% per tier, so damage doubles every 8 tiers. Cash per second doubles every single tier (Tier 1 earns 1/s, Tier 30 earns 536.87M/s) [8][14].
- This was probably a rebalance. Videos uploaded on 2026-06-15 and 2026-07-01 show damage = 100 x the tier label, doubling each merge: Lil Boomer 100, Mega Fizzer (Tier 8) 800, Boom Baron (Tier 32) 3,200 [18][19][20]. Videos from 2026-07-10 onward match today's 300 x 1.0905 numbers [21][22]. [inferring the rebalance from the videos; no patch note found]
- The main health-drain content is **PvE** (player versus environment, meaning fighting the game, not other players). **City events** (Shipping Port, Downtown, Oil Rig) have 200,000 HP and heal 250 HP per second. The whole server shoots them. Players get cash for hitting them. After an event there is a 50% chance a **commander** spawns [9]. Commanders are roaming ships and units with 3,500 to 660,000 HP that you shoot down to collect [11][12].
- Feedback that is confirmed: targets show an HP number that drops with each hit. Each nuke shows its name, its damage, and a cooldown timer. Blasts splash onto nearby bases [18][20][21]. **Not confirmed** for Merge a Nuke: health-bar graphics, floating damage numbers, mushroom clouds, screen shake, and damage falloff with distance.
- Other games: in Destruction Simulator, bomb prices rise much faster than bomb power (by my fit, price grows with power^1.7). It runs from a free Grenade (x10) to a Classic Nuke at 15,000,000 coins (x20,000) [27]. The best nuke spectacles are Jailbreak (16,000 Robux, siren, orange screen tint, blur, mushroom cloud, and a cosmetic reward for players caught in it) [29] and Military Tycoon ($25M or 499 Robux, siren, flash, a blast that grows to cover the map, darker sky, 10-minute server cooldown) [28].
- **Fruit Empire already sells TNT.** It costs 2,500 shards, works on any plot including other players', and deletes every fruit within 18 studs at once. Only the 60 s Lock stops it (local code) [39][40][41]. The redesign turns that delete into a health drain.
- The main lesson for Fruit Empire: take health away over several hits, keep explosive damage growing slower than earnings, and point big explosives at shared PvE targets that heal over time.

## How it works

### Terms used below

- **Tier**: the level of a nuke. In Merge a Nuke the tier label is also the cash per second. A "Tier 32" nuke earns 32 cash per second before multipliers [3]. Guide sites also number tiers 1 to 60 in order (T1, T2, T3...). This note uses that numbering, written as "T".
- **Merge**: put two same-tier items together to make one item of the next tier.
- **Raid**: launching a nuke at another player's base.
- **Lock Base**: a timed shield that makes incoming nukes bounce off.
- **Commander**: a collectible unit. You equip commanders for passive bonuses (always-on effects).
- **Mutation**: a random modifier on a nuke. It gives a bonus and usually a penalty.
- **Rebirth**: a reset. You lose nukes, cash, and upgrades. You gain a permanent bonus and a higher tier cap.
- **Regen**: health that comes back over time.
- **Splash damage**: damage that hits an area, not one target.
- **Falloff**: damage getting weaker the farther a target is from the center of the blast.
- **R$**: Robux.

### Nuke tiers, damage, and cash

- The Fandom wiki lists 100 nuke types. The highest listed is Mass Extinction (Tier 633.83 Octillion) at 1.59M damage. Cash is multiplied by rebirths [3].
- A guide site lists T1 to T60 [8][14]. Sample rows:

| T | Nuke | Damage | Cash per second |
|---|---|---|---|
| 1 | Lil Boomer | 300 | 1 |
| 10 | Galaxy Gobbler | 654 | 512 |
| 20 | Dimension Demolisher | 1.56K | 524.29K |
| 30 | Eternity Ender | 3.7K | 536.87M |
| 40 | The Sundering Choir | 8.8K | 549.76B |
| 50 | The Last True Word | 20.93K | 562.95T |
| 60 | Void Eater | 49.78K | 576.46Qa |

- The damage formula is `baseDamage = 300 * 1.0905^(tier - 1)`. The game rounds the number down for display [8].
- My math from that formula: 1.0905^8 = 2.0. So damage doubles every 8 tiers. Over the same 8 tiers, cash goes up 256 times.
- Cross-check: a 2026-07-11 video reads Glitch nukes (Tier 590 Quintillion) as "118,000 damage" [22]. Fandom lists Glitch at 118.38K [3]. They match.

### Health, and how damage drains it

- Every nuke on a plot has HP. The guide's baseline is max HP = 2 x that tier's base damage. The guide calls this an observed baseline, not an exact value [7].
- Example HP values: T10 = 1,309. T30 = 7,401. T50 = 41,861. T60 = 99,556 [7].
- Damage lands instantly on each hit. Hits to destroy = ceiling(defender HP / attacker damage) [7][8].
- My math: a same-tier attack needs 2 hits. An attacker about 8 tiers higher kills in 1 hit, because its damage is 2x.
- Things that change health:
  - **x2 Nuke Health** gamepass, 199 R$ [2].
  - Mutations [16][17]. Dense Core: +25% cash, +25% HP, 40% longer cooldown. Reformed Plating: +50% HP, -25% cash. Enriched Core: +40% cash, -25% HP. Overclocked Reactor: 2x cash, +75% damage, -40% HP, 35% longer cooldown.
  - Health-reaction mutations [16]. Leeching Warhead heals 25% HP after a successful hit. Regenerative Alloy slowly heals after it avoids damage. Critical Mass deals +50% damage while below half HP.
  - Commanders. The Interceptor card was read aloud as a "50% chance to block enemy nuclear bombs, reducing [damage]" [21]. A Sergeant "deploys sandbags, multiplying health" when your base is attacked [22].
- **Commander ships** are drain targets that move. Listed HP: Salvager 3,500, Foreman 5,000, Broker 6,500, Engineer 7,000, Gunner 8,500, Barrier 26,000, Artillery and Spotter 28,000, Looter 31,000, Siegebreaker 83,700, Overclocker 92,000, Carrier, Raider and Sovereign 125,500, Apache 575,000, Interceptor 660,000. They move by land, by sea, through the air, or stay still [11][12]. In one video the player watches a Sovereign's HP fall "to 12,000", then "to 6,000", then fires all his nukes at once to finish it and collect it. Its passive gives +15% nuke damage and a bigger city event bonus [21].
- **Splash damage** exists. One player says "It's the splash damage" after his nuke hurt a neighbor. Later he aims a 500-million-tier nuke between bases and says "It's going to hit all of them" [20]. At very high tiers, another player says "I literally think the whole map just got hit" [19]. Blast radius numbers are **not confirmed**.
- **Distance falloff is not confirmed.** The guide's hit formula has no distance term [8].

### Cities: the shared boss that heals

- Current city events (guide site): Shipping Port, Downtown, and Oil Rig. Each has 200,000 HP and heals 250 HP per second. After the event there is a 50% chance a commander spawns [9].
  - Shipping Port: Foreman 70%, Barrier 30%.
  - Downtown: Broker 70%, Artillery 30%.
  - Oil Rig: Looter 70%, Overclocker 30%.
- Early videos show a bigger city in the middle of the map. Players call out its HP as it falls: "10 million health", then 7 million, then 3 million and 2 million. They complain: "Why does it keep healing?" One calls it "like a boss" [20]. Another video also reads "the HP is 10 million" [18].
- The city opens on a timer. One video shows "the city can be attacked in 11 seconds", then "Event start to destroy the city to earn cash" (auto-caption wording). Later the player says "just two more minutes for us to attack the city". When the team is too weak he says "it's gaining HP. We're not doing enough damage" [19].
- **Payout.** Hitting the city pays cash: "Whenever you hit stuff, you get a lot of money" [20]. One player guesses "the more damage we do to the city, the more money we make" [19]. The exact formula is **not confirmed**.
- One odd case: a nuke rated at 1.7 quadrillion damage showed only "a million damage" on the city, yet paid "six quintillion dollars" [19]. So city damage looked capped, and pay seemed tied to something else. This is from one video only. **Not confirmed** as a rule.

### Raids, Lock Base, and cooldowns

- How to launch: pick up a nuke, enter targeting, choose a base or a cluster of nukes, confirm. After you launch, your own base can become unlocked [7].
- **Lock Base** durations: level 1 = 1:00, then +0:30 per level. Caps: 5:30 at Rebirth 0 (level 10), 10:30 at Rebirth 1, 15:30 at Rebirth 2, 20:30 at Rebirth 3, and a hard cap of 25:30 from Rebirth 4 on [7]. An early guide says there is a 30-second cooldown between locks [15].
- **Launch cooldown** in early videos: 5 minutes [20] and 10 minutes [19]. The timer is shown on the nuke itself [20].
- What a raid does: the game description says "Launch nukes at enemy bases and STEAL their cash" [1]. The Raider commander "will go there and blockade them, steal their income, and then leave the loot at your plot". Later the same player reads a card as "For 10 seconds, it takes 15% of their income" (the Raider, by context) [21]. An early guide says launching can "reset their progress" [15]. One player paid 19 Robux to get his nukes back after his base was wiped [18].

### Upgrades and prices

- The main upgrades are **Spawn Tier** (the tier new nukes spawn at), **Max Spawn** (how many nukes your base holds), and **Lock Base** [15].
- Spawn Tier costs (Fandom): level 2 = 10K, level 3 = 25K, level 4 = 58.75K, level 5 = 138.06K ... level 25 = $3.64T, level 27 = $20.2T, level 28 = $47.28T, level 29 = $111.1T, level 30 = $261.08T [4]. My math: each level costs 2.35x the last. Each level doubles the spawn tier. So each upgrade is a little harder to afford than the one before.
- Gamepasses (official Roblox API) [2]:
  - Auto Merge, 199 R$
  - x2 Spawn Speed, 129 R$
  - x2 Nuke Health, 199 R$
  - Quick Nuke Cooldown, 99 R$
  - Quick Lock Cooldown, 79 R$
  - Off sale: Launch All Nukes!, Quick Launch Cooldown, Skip Cooldown [PERM], x2 Cash
- Cash multiplier products go up to 256x [13]. In June one player paid "63 Robux for a 32 times boost" [19]. Commander pack odds: Salvager, Engineer, and Gunner 25% each. Interceptor 20%. Raider 4.5%. Carrier 0.5% [13].

### Rebirths

- What is checked: the highest nuke you actually own in your base. Finding it in the index is not enough [10]. Rebirths also need cash [16].
- Tier gates: T30 (536.87M), then T35, T40, T45, T50, T55 [10]. Each rebirth raises the merge cap by 5 tiers and adds +25% cash (1.25x after the first, up to 150% total after the sixth listed) [10]. A player confirms: "If I rebirth, I get 25% more cash" [21].
- A rebirth resets nukes, cash, and upgrades [15]. "We're all the way back to zero upgrades" [19].
- The rebirth count has grown over time. There were 3 at launch [15]. The Fandom Rebirths page says 12 [6]. The Fandom Nukes page has sections up to Rebirth 14 [3]. The 2026-10-03 update notes say "new rebirths" [38].
- Unlocks tied to rebirths: mutations at Rebirth 3, Legendary mutations at Rebirth 5, Admiral mutations at Rebirth 10 [16][17].

### Explosion effects and cosmetics

- Fandom lists 8 features: Nukes, Commanders, Explosion Effects, Mutations, Base Skins, Cities, Upgrades, and Player Plots/Bases [5]. So explosion looks are a separate cosmetic you can collect.
- The expired code MAGMA gave "Molten Magma" [23][24]. The Magma mutation gives +50% cash and extra damage, and "the nukes' impact creates a fire" [16]. Whether that fire does damage over time is **not confirmed**.
- The second currency is **Nuclear Waste**. The code DELAY gives 300 [24]. What it buys is **not confirmed** in the sources found.

### Other Roblox games with tiered explosives

**Destruction Simulator** (bombs bought with coins; "Explosion" = how many blocks it can break) [27]:

| Bomb | Explosion | Cost |
|---|---|---|
| Grenade | x10 | Free |
| Dynamite | x20 | 30 |
| C4 | x150 | 6,500 |
| Candy Bomb | x300 | 30,000 |
| Evil Robot | x1,000 | 255,000 |
| Satellite | x4,500 | 1,250,000 |
| Mini Nuke | x7,500 | 2,000,000 |
| Nuke | x10,000 | 3,500,000 |
| Huge Nuke | x15,000 | 5,000,000 |
| Classic Nuke | x20,000 | 15,000,000 |
| Nuclear Reactor | x30,000 | 1,400 R$ |
| Black Hole Bomb | x100,000 | 5,000 R$ |

- My fit over the 20 coin bombs: cost grows with power^1.7. Cost per unit of power climbs from 1.5 (Dynamite) to 750 (Classic Nuke). Stronger bombs are worse value per coin. You buy them to save time.

**Nuke Tycoon Nuclear** (missiles fired with "Power", gated by rebirths):
- Light Missile Launcher: Missile 1 (0 rebirths) has Strength 150, Speed 350, Power 300, Radius 256. Missile 5 (2 rebirths) has Strength 485, Speed 650, Power 2,000, Radius 458 [31].
- Heavy Missile Launcher: Missile 1 (1 rebirth) has Strength 500, Power 2,300, Radius 512. Missile 5 (6 rebirths) has Strength 2,500, Power 5,000, Radius 1,024 [32].
- Nuclear Ballistic Missile: unlocks at 9 rebirths. It needs 80,000 Power and all 8 launch stages done. It ignores spawn protection and has fixed damage [33].
- Interceptors have a 65% chance to stop a missile, shell, or nuke. They cost 15 gems and need 2 rebirths [34].
- Artillery shells: Low Explosive costs 10 gems, radius 256, damage 100 [35]. High Explosive costs 65 gems, radius 814, damage 200 [36].
- Update logs: stronger missiles have a higher chance to break spawn protection. Updates added "Screen shaking", new launch effects, and new explosion effects [37].

**Military Tycoon** nuke: costs $25,000,000 or 499 R$. It flies over the island for tens of seconds. A server message and siren tell everyone to get in a bunker. Then a bright flash, then a blast that grows until it covers the map. A kill list shows on screen. Afterward the sky clouds over, colors shift (ColorCorrectionEffect), and palm trees look burned. Only one per 10 minutes per server [28].

**Jailbreak** nuke: 16,000 R$, or one free per season at level 10. A chat warning ("launched a nuke! Take cover!") and a siren play. The nuke rises out of a silo, then flies in an arc. An orange blast covers most of the map. The screen turns orange and blurs for a few seconds, then a gray mushroom cloud shows and fades. Players caught in the blast get an exclusive cosmetic [29].

**Super Bomb Survival** nuke: it explodes several times. The first blast deals 50 damage. Then more blasts at random spots near the impact deal 10-35 each. A red glow on the ground shows the blast radius before it lands, and a siren plays. The Mega Nuke deals 80, then 20-50 per blast [30].

### Engine tools for "feel"

- Roblox's `Explosion` object by default breaks joints and kills characters. To do your own damage, set `BlastPressure = 0` and `DestroyJointRadiusPercent = 0`, then use the `Hit(part, distance)` event [25].
- `Hit` fires once per part, so a model can be hit many times. Track which models were already hit [25].
- Walls do **not** block explosions. Parts behind cover still get hit [25].
- The official code sample uses linear falloff: `damage = maxDamage * (1 - distance / BlastRadius)` [25].
- Camera shake: the RbxCameraShaker "Explosion" preset uses magnitude 5, roughness 10, 0 s fade-in, and 1.5 s fade-out ("An intense and rough shake. Should happen once"). "Bump" uses 2.5, 4, 0.1 s, and 0.75 s [26].

### What Fruit Empire's TNT does today (local code, 2026-10-04 export)

This matters for every proposal below, because TNT is not new. It is already in the shop.
- **Price:** 2,500 shards. Tools are bought with shards, not cash. [39]
- **Where it works:** on any plot the player is standing on, their own **or another player's**. [40]
- **What it does:** it deletes every fruit within 18 studs at once. Fruit that is mid-merge or mid-drop is skipped. The victim gets the message "<name> blew up N of your fruit!". [39][40]
- **The blast is only a look.** It spawns an `Explosion` with `BlastPressure = 0`, `DestroyJointRadiusPercent = 0` and no craters, so it pushes and breaks nothing. [40]
- **The defense:** the Lock button. A locked plot refuses TNT for 60 s, then the lock needs 30 s to recharge. [41]
- So the owner's ask ("tnt shouldnt instantly get rid of your fruits ... it should drain the health") changes an existing cross-plot weapon. Whether TNT should keep hitting other players' plots is still the owner's call (see Open questions).

## What Fruit Empire should take

These are proposals adapted to Fruit Empire. They are my design suggestions, not facts about Merge a Nuke.

1. **Give fruit HP tied to explosive damage.** Copy the baseline: fruit max HP = 2 x the damage of the matching-tier explosive [7]. Then a matching TNT needs 2 hits, which is a visible drain, not a delete. Match explosive tiers to the rarity stages (Common to Transcendent), not to all 64 fruits. That keeps the TNT list short.
2. **Keep damage growth slow and earnings growth fast.** Use `damage(t) = D1 * g^(t - 1)`. With g = 1.09, damage doubles every 8 tiers. Fruit cash keeps its own steeper curve. This stops one top-tier nuke from wiping a whole plot. Early Merge a Nuke had doubling damage, and players asked "How do new people win this?" [20].
3. **Sell each TNT tier at a rising price, and let TNT merge as a bonus.** The owner's brief says "stronger tnt cost more" [[Brain Dump 2026-10-04]], so every tier should be buyable, with stronger tiers priced higher. Merging two TNT into the next tier (like nukes) can still be an extra route, since players already know the merge loop. **Currency is an open decision.** Today TNT costs 2,500 shards [39]. A full Fruit Index pays only 26,528 shards in total, and just 31 for the Common page (worked out from the local `IndexRewards` code) [42]. So a shard ladder of several tiers would be out of reach for most players. A cash price set as "N seconds of your current income" would follow the player's stage on its own, which the earnings rule asks for.
4. **Make the big boom PvE first.** Add a shared target with lots of HP and regen, like the city: it opens on a countdown, stays open for a few minutes, the whole server shoots it, and pay is based on each player's damage share [9][19][20]. Put it in the existing world. For example, put it across the sea so boats and artillery matter. Add a chance for a bonus drop at the end, like the 50% commander chance [9]. Regen makes players focus fire. Tune it so a full server can win (see "What to avoid").
5. **Respect the no-stealing rule.** Merge a Nuke raids take cash and wipe nukes [1][18][21]. Fruit Empire's live TNT already hits other players' plots and deletes their fruit [40]. If that stays, a fruit at 0 HP should **not** be deleted or moved. One option: the fruit is "stunned". It stops earning for a few seconds, then heals back. Another option: explosives only hit PvE targets, boats, and the duel arena. This is a decision for Samarth (see Open questions).
6. **If plot hits stay, grow the existing Lock.** Fruit Empire's Lock is a flat 60 s with a 30 s recharge [41]. Merge a Nuke's shield starts at 1:00, grows +0:30 per level, and has a cap that rises with each rebirth [7]. Attacking drops your own shield [7]. Copying the "grows with rebirths" part fits the rebirth plan.
7. **Show the drain clearly.** Use a floating damage number on every hit, plus an HP bar that shrinks. Merge a Nuke shows the HP number; the bar is an addition. Label each TNT with its tier, damage, and cooldown, like nukes do [20][18]. Show a red circle on the ground before impact, the size of the blast [30]. Shake the camera with the Explosion preset, weaker with distance and stronger with tier [26]. Save the mushroom cloud, the orange tint and blur, and the server-wide siren and chat warning for nuke tiers only [28][29].
8. **Use a matching retro look.** The classic Roblox `Explosion` sphere looks like 2006-2008 Roblox. You can keep `Visible = true` with `BlastPressure = 0` and `DestroyJointRadiusPercent = 0`. Then it looks classic but does not knock apart the studded fruit models [25]. The live TNT already uses exactly this setup [40]. [inferring that the default visual fits the art style]
9. **Use falloff from the docs.** Use `1 - distance / BlastRadius` [25], and count each fruit model only once per blast [25]. Decide on purpose whether walls should block blasts, because by default they do not [25].
10. **Make nukes drain over time with follow-up blasts.** Nuke tiers can do one big first hit, then several smaller hits nearby, like Super Bomb Survival (50, then 10-35 each) [30]. This makes health tick down on screen.
11. **Gate the rebirth on owned items, not the index alone.** Merge a Nuke checks the highest nuke you own, raises the cap by 5 tiers per rebirth, and adds +25% cash [10]. Fruit Empire's plan already says 0 rebirths = max fruit Apple, plus full index, shards, and money. Add the "must own it now" check to stop index-only rushing. Unlock modifiers (like mutations) at rebirth milestones such as 3, 5, and 10 [16].
12. **Add tradeoff modifiers.** Mutations that give +cash but -HP, or +damage but a longer cooldown, create choices [16]. A modifier can carry over through a merge with a chance, so a low-tier mutated item is still worth keeping [17].
13. **Price-tested gamepass ideas.** x2 health at 199 R$, and cooldown cuts at 79-99 R$ [2]. These only matter if explosives affect your own fruit.
14. **Reward the victims of big blasts.** Jailbreak gives every player caught in the nuke a cosmetic [29]. For Fruit Empire, a small shard bonus for being inside a nuke blast turns a bad moment into a good one.

## What to avoid

- **Doubling damage per tier.** Early Merge a Nuke did this [19][20]. High tiers one-shot everything. A player says "How do new people win this?" [20], and another overkills a player who "has level one nukes" [19].
- **Hidden damage rules.** A player says "I don't understand how they decide how much damage to do" [21]. A 1.7-quadrillion nuke showed "a million damage" on the city [19]. If there is a cap, show it.
- **Regen faster than players can deal damage.** "Why does it keep healing?" [20]. "I also don't know if it's even possible to take this city down. We are doing so much damage, but it just it keeps getting it back" [19]. Players feel useless.
- **Shield spam that kills PvP.** "Everyone just locking their bases" [18]. But no shield means griefing. If you have PvP plot hits, you have to tune this.
- **Paying to recover after being wiped.** One player paid 19 Robux to get his nukes back [18]. That feels bad, and it goes against why stealing was removed.
- **Too many screen-wide effects.** Big nukes are rare on purpose: 16,000 R$ or one per season in Jailbreak [29], and one per 10 minutes per server in Military Tycoon [28].
- **Default explosion physics.** A plain `Explosion` breaks joints and kills characters within its radius [25]. That would break studded fruit models and kill players.
- **Trusting SEO fan wikis for numbers.** Many "Merge a Nuke wiki" sites exist, and their numbers clash (some still show the old doubling damage). Even the best guide site marks HP values as "estimated" [7]. Check numbers in-game before copying them.

## Open questions

- Does Merge a Nuke use damage falloff with distance? Not confirmed [8].
- What is the blast radius for each tier? Not confirmed. Splash is confirmed [20].
- How much cash does one raid hit steal? Only the Raider commander's "15% of their income for 10 seconds" was found [21].
- How is city pay calculated, and is city damage capped per hit? One video suggests a cap [19]. Not confirmed.
- Does Merge a Nuke draw HP as a bar, show floating damage numbers, or play a mushroom cloud and screen shake by default? Transcripts confirm HP numbers only [18][20][21]. Someone should check this in-game.
- What does Nuclear Waste buy? Not confirmed [24].
- **For Samarth:** live TNT already deletes other players' fruit [40]. With health drain, should it keep hitting other players' plots at all, given stealing was removed on purpose? If yes, does a fruit at 0 HP get stunned, or something else?
- **For Samarth:** the brief says stronger TNT costs more, so each tier is sold. Should merging TNT also exist? And should TNT be priced in shards (today: 2,500) or in cash scaled to the player's stage?
- **For tuning:** what are D1 and g for each rarity stage, and how much HP and regen should the shared target have? The Roblox API lists Fruit Empire's server size as 50 players, but the map has only 12 plots [43][44]. So a "full server" could mean 12 or 50 players.

## Review (critic pass, 2026-10-04)

- **Verified:** the damage formula against the table (T10 = 654, T30 = 3.7K, T60 = 49.8K), HP = 2 × damage (T10 = 1,309), cash = 2^(T−1), and the Spawn Tier cost step of about 2.35x. The Explosion docs sample does use linear falloff: `distanceFactor = 1 - distance / BlastRadius`, then `TakeDamage(maxDamage * distanceFactor)` [25]. The docs also say obstacles do not block an explosion [25].
- **Added:** what Fruit Empire's TNT does today, from the local code [39][40][41]. The note had treated plot-hitting explosives as a new idea.
- **Changed:** proposal 3. It said "shop TNT only at the lowest tier", which goes against the brief's "stronger tnt cost more".
- **Still not confirmed:** Merge a Nuke's falloff, blast radius per tier, raid cash amount, city pay formula, HP bar and floating numbers, and what Nuclear Waste buys.

## Sources

1. Roblox Games API, Merge a Nuke! details (official description and stats, read 2026-10-03): https://games.roblox.com/v1/games?universeIds=10199301628 (game page: https://www.roblox.com/games/128784467030899/Merge-a-Nuke)
2. Roblox Game Passes API, Merge a Nuke! (official names and prices): https://apis.roblox.com/game-passes/v1/universes/10199301628/game-passes?passView=Full&pageSize=100
3. Merge a Nuke Wiki (Fandom), "Nukes": https://merge-a-nuke.fandom.com/wiki/Nukes
4. Merge a Nuke Wiki (Fandom), "Spawn Tier Upgrade": https://merge-a-nuke.fandom.com/wiki/Spawn_Tier_Upgrade
5. Merge a Nuke Wiki (Fandom), main page: https://merge-a-nuke.fandom.com/wiki/Merge_a_Nuke_Wiki
6. Merge a Nuke Wiki (Fandom), "Rebirths": https://merge-a-nuke.fandom.com/wiki/Rebirths
7. gcz.gg, Merge a Nuke PvP Raid Guide (Lock Base, cooldowns, nuke HP): https://gcz.gg/en/games/merge-a-nuke/pvp-raid
8. gcz.gg, Merge a Nuke Attack Damage Calculator: https://gcz.gg/en/games/merge-a-nuke/attack-damage
9. gcz.gg, Merge a Nuke City Events: https://gcz.gg/en/games/merge-a-nuke/city-events
10. gcz.gg, Merge a Nuke Rebirth Guide: https://gcz.gg/en/games/merge-a-nuke/rebirth
11. gcz.gg, Merge a Nuke Commanders Guide: https://gcz.gg/en/games/merge-a-nuke/commanders
12. gcz.gg, Merge a Nuke Commander Hit Calculator: https://gcz.gg/en/games/merge-a-nuke/commander-hits
13. gcz.gg, Merge a Nuke Shop Value Guide: https://gcz.gg/en/games/merge-a-nuke/shop-value
14. gcz.gg, Merge a Nuke Tier List (all 60 nukes): https://gcz.gg/en/games/merge-a-nuke/nukes
15. Sportskeeda, "Merge a Nuke: A beginner's guide": https://www.sportskeeda.com/roblox-news/merge-nuke-a-beginner-s-guide
16. Sportskeeda, "Merge a Nuke mutations guide" (2026-08-05): https://www.sportskeeda.com/roblox-news/merge-a-nuke-mutations-guide
17. Radex Tips, "ALL MUTATIONS GUIDE ... in MERGE A NUKE Roblox" (YouTube, 2026-08-06, auto-captions): https://www.youtube.com/watch?v=8pWocOUNCrQ
18. Skullss, "MERGE NUKE To Destroy Players Bases! in ROBLOX" (YouTube, 2026-06-15, auto-captions): https://www.youtube.com/watch?v=L85rzXjE89U
19. Quacks, "I Merged MAX LEVEL Bomb in Roblox!" (YouTube, 2026-06-15, auto-captions): https://www.youtube.com/watch?v=76kZ01-kEUU
20. SSundee, "MERGING The BIGGEST NUKE in Roblox" (YouTube, 2026-07-01, auto-captions): https://www.youtube.com/watch?v=vElboau8b1g
21. Kream Blox, "NOOB to PRO in Merge Nukes on Roblox using ALL GAMEPASSES!" (YouTube, 2026-07-10, auto-captions): https://www.youtube.com/watch?v=j9DPrpPnkLw
22. Kream Blox, "Unlocking EVERYTHING in Merge a Nuke Update!" (YouTube, 2026-07-11, auto-captions): https://www.youtube.com/watch?v=g-Tc884OqL4
23. Beebom, "Merge a Nuke Codes": https://beebom.com/merge-a-nuke-codes/
24. Pocket Tactics, "Merge a Nuke codes": https://www.pockettactics.com/merge-a-nuke-codes
25. Roblox Creator Docs, Explosion class (incl. "Custom Explosion" sample): https://create.roblox.com/docs/reference/engine/classes/Explosion
26. Sleitnick, RbxCameraShaker, CameraShakePresets.lua: https://github.com/Sleitnick/RbxCameraShaker/blob/master/src/CameraShaker/CameraShakePresets.lua
27. Destruction Simulator Wiki (Fandom), "Bombs": https://destruction-simulator.fandom.com/wiki/Bombs
28. Military Tycoon Wiki (Fandom), "Nuke": https://military-tycoon.fandom.com/wiki/Nuke
29. Jailbreak Wiki (Fandom), "Nukes": https://jailbreak.fandom.com/wiki/Nukes
30. Super Bomb Survival Wiki (Fandom), "Nuke": https://roblox-super-bomb-survival.fandom.com/wiki/Nuke
31. Nuke Tycoon Nuclear Wiki (Fandom), "Light Missile Launcher": https://nuke-tycoon-nuclear.fandom.com/wiki/Light_Missile_Launcher
32. Nuke Tycoon Nuclear Wiki (Fandom), "Heavy Missile Launcher": https://nuke-tycoon-nuclear.fandom.com/wiki/Heavy_Missile_Launcher
33. Nuke Tycoon Nuclear Wiki (Fandom), "Nuke launcher": https://nuke-tycoon-nuclear.fandom.com/wiki/Nuke_launcher
34. Nuke Tycoon Nuclear Wiki (Fandom), "Interceptors": https://nuke-tycoon-nuclear.fandom.com/wiki/Interceptors
35. Nuke Tycoon Nuclear Wiki (Fandom), "Low Explosive artillery Shell": https://nuke-tycoon-nuclear.fandom.com/wiki/Low_Explosive_artillery_Shell
36. Nuke Tycoon Nuclear Wiki (Fandom), "High Explosive artillery Shell": https://nuke-tycoon-nuclear.fandom.com/wiki/High_Explosive_artillery_Shell
37. Roblox DevForum, "Update Logs - Nuke Tycoon Nuclear": https://devforum.roblox.com/t/update-logs-nuke-tycoon-nuclear/3186396
38. Roblox Virtual Events API, Merge a Nuke! events ("BRAINROTS!??!?!", 2026-10-03): https://apis.roblox.com/virtual-events/v1/universes/10199301628/virtual-events
39. Local code: `~/fruit-empire/src-export-2026-10-04/ReplicatedStorage.GardenConfig.lua` (`Config.Tools.TNT`: price 2500 shards, radius 18; messages `TNTDetonated`, `TNTVictim`)
40. Local code: `~/fruit-empire/src-export-2026-10-04/ServerScriptService.GardenService.Tools.lua` (`useTool`, about lines 244–310)
41. Local code: `~/fruit-empire/src-export-2026-10-04/ReplicatedStorage.GardenConfig.lua` (`LockDuration = 60`, `LockCooldown = 30`, `LockTNTBlocked`)
42. Local code: `~/fruit-empire/src-export-2026-10-04/ReplicatedStorage.IndexRewards.lua` (shards 5 to 2,500 per fruit on a geometric curve; summed over all 64 fruits = 26,528)
43. Roblox Games API, Fruit Empire (read 2026-10-04; `maxPlayers` 50): https://games.roblox.com/v1/games?universeIds=10764283455
44. Local code: `~/fruit-empire/src-export-2026-10-04/ReplicatedStorage.GardenConfig.lua` (comment "measured on all twelve plots", line 107)
