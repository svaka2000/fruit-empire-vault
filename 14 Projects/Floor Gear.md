# Floor Gear: power-ups and weapons on every floor (21 Nov 2026)

Samarth, 21 Nov 2026, 21:44 PST: "make power ups and weopons you can gain access too on each floor (you will need to buy them like you walk up to the power up which will be on a pedestal and then you buy it with money do it based on game progression calculate accurately.) they get more and more powerful each floor. on floor one there can be the bat which you get for free (basically instead of it already being in their inventory they just get the bat from their plot.) when they rebirth and their plot resets they can equip the new bat. make sure you dont break any functions i know its really complex with all the things the rebirths mega rebirths etc. be super organized and be really smart study other games to get good ideas. and use claude design."

Design board (Claude Design, private to Samarth): the "Fruit Empire Floor Gear" canvas, seven artboards (the pedestals in 3D, floor 1's bat, the buy card, the ladder, the hands, the math, the plan). Source, models and tools: `~/fruit-empire-floor-gear` on Samarth's computer.

## What the game has today (read in the place, 21 Nov)

- **The bat.** Every loaded player gets one Bat tool from `GardenService.Bats`. Its tier is the stage (Rebirths + 1), one of nine bats (`GardenConfig.BatTiers`), swapped on every rebirth. Power 24 + 8T, Lift 12 + 2T, 57 + 3T swings a minute, stun 0.25 + 0.05T s; reach 7.5 studs (the server allows 4 more for lag), a 120 degree arc, 7 studs of height. Hotbar slot 1, the Inventory's Gear tab, and the big bat button on phones.
- **Every hit is the server's.** The attacker must be loaded, alive, bat in hand, hands empty, visible and on foot. The victim's protections: the join and respawn shields, 1.2 s of immunity per kind of push, seats, and the safe zones (plots, judged a moment either side of the edge, and all of a Troll morph's costume). Then reach, height, arc, at most 4 hits on one player per 20 s, and a line of sight. The push is `G.knockPlayer`, and hooks run after it: the Golden Crate drops, a carried egg comes loose under its own rules (worth-your-time, the rookie shell, the carrier grace). A carrier is pushed no harder than a bat at their own stage + 2. A hit never takes anything.
- **Hooks the bat already has for weapons.** The `WeaponDrawn` player attribute puts the bat away; `G.holsterWeapon` is called when the bat is drawn; `G.knockScale` scales pushes (the pets' Steady trait uses it); `G.walkScaleHooks` slow walking (forest pets use it).
- **Floors.** Floors = 1 + Mega Rebirths, up to 22 (fewer on the beanstalk plots). Each floor is 16 studs over the last, a slab with the soil's footprint, with a lift pad 3.5 studs in from the soil's left and front edges on every floor. Fruit never spawns or rests on a part named or tagged `PetObstacle` in the pen, on any floor (the Nest Shelf uses it).
- **Rebirth** clears the plot and stored fruit, cash and cash upgrades (Plot Size stays), and keeps tools, the Index, pickaxes, eggs and potions. **Mega Rebirth** does the same, goes back to Rebirth 0, adds a floor, and gives cash x2 and merge shards x1.5 for good.
- **Saves.** One document per player. A new field needs `Store.Default` and `sanitise` in `GardenStore`, and `captureProfile` and `applyProfile` in `GardenService.Persistence`. A document is written with the lowest version that stops an older server from dropping something (2 Mega, 3 pets, 4 Aether pets, 5 cave creatures, 6 floors over 14). The self-test's worst document must fit 20,480 bytes.
- **The economy.** Rebirth cash needs: $1.1K, $2.6M, $48.1B, $156T, $491Qa, $4.4Sx, $9.7Sp, $27Oc. The first Mega Rebirth needs $2.73No, and each later one 4 times the last.

## What other games do

- **Steal a Brainrot** sells about 40 gears for cash, each unlocked by a rebirth, from $500 to $10B (about x1.5 a step). A slap line (Iron, Gold, Diamond, Emerald, Ruby ... Galaxy, Glitched) gets stronger every rebirth. Utility gear works while held: the Speed Coil (+50% speed), the Gravity Coil (2x jump), the Coil Combo (both). Ranged and stun gear (Taser, Web Slinger, Paintball Gun) have cooldowns of 3 to 30 s, and the Grief Shield absorbs one hit. (bloxodes.com's gear wiki, ofzenandcomputing.com's gear guide, beebom.com's rebirth guide.)
- **Bee Swarm Simulator** gives everyone a free starter tool (the Scooper). Better tools stand in shops higher up the map, from 1K to trillions of honey; you walk up to a stand to see its stats and buy. (The Bee Swarm Simulator wiki: Tools, Pro Shop.)
- **Tycoon buy pads**: the price is on the pad, the server checks and spends it, only the owner can buy, and prices grow by a share per step to keep up with income. (Roblox DevForum tycoon pricing threads; creation.dev's tycoon guide.)
- **Grow a Garden** splits gear into held, placed and one-use, sold in a shop that restocks. (pockettactics.com, beebom.com.)

Taken: the free starter tool on your own land (our bat); held gear, one thing at a time, each with one clear job; a weapon line where every step is visibly stronger; prices that climb with income. Left out: stealing (gone from the game for good), traps and turrets on plots (plots are safe zones), and teleports or grapples (they would skip the beanstalk climb and the thin air).

## Samarth's calls (21 Nov, on the board)

1. **Power-ups work only while held in your hand**, like Steal a Brainrot's coils.
2. **A rebirth takes the old bat back with the plot.** The new bat waits on the pedestal. A Mega Rebirth works the same way.
3. **Floor 1 holds just the bat.** Its card shows floor 2's gear as the next goal.
4. **The ladder, names, looks and prices as on the board.**

## The design

### What a player sees

1. **A pedestal on every floor of your tower,** in the same spot on each floor: the front corner opposite the lift pad. Studded light stone with gold trim and a glowing ring in the floor's tier colour, with the gear floating and turning over it. A plaque on its front says the price, FREE or OWNED. Everyone sees the gear turning, so a tall tower shows off what its owner has.
2. **Walking up** (the plot's owner, on that floor, within about 10 studs): a card rises over the pedestal with the floor and its tier, the gear's name, two stat chips compared with what you hold, one line saying what it does, the price (green if you can pay; red with "You need $X more" if not) and the E prompt (hold half a second). Floor 1 says "TAKE IT: FREE".
3. **Buying:** the gear lifts off, spins and flies to you; NEW GEAR! at the top, the game's real confetti, a fanfare. It goes into your hand and its hotbar slot. The pedestal keeps showing it with a gold ring and OWNED.
4. **The bat:** a new player has no bat until they take it from floor 1's pedestal. Every rebirth and Mega Rebirth takes the bat back with the plot, and the pedestal then holds the new stage's bat with a NEW! ribbon, with the notice "Your new bat is on your plot!".
5. **Your hands:** slot 1 is your weapon (the bat or a floor weapon you picked), slot 2 your power-up outside the mine (the pickaxe in the mine, as today). The Gear tab lists the bat, your floor gear and your pickaxes; Equip picks which weapon and which power-up sit in the slots. On phones the bat button swings your weapon and a second button holds your power-up.
6. **The ladder panel** (from the Gear tab and the pedestal card): all 22 floors, what you own, what comes next and the prices.

### Rules

- Gear is bought with cash, checked and spent by the server, and kept for good: rebirths and Mega Rebirths never take it.
- Only the plot's owner buys at its pedestals, and only gear for floors they have earned (1 + Mega Rebirths, at most 22). A Mega player on a plot with fewer floors (the beanstalk plots) buys the missing floors' gear from the ladder panel.
- One thing in the hands. Drawing gear puts the bat away and drawing the bat puts gear away. Lifting fruit, an egg, the crate or TNT, going into the mine, going invisible or driving puts gear away.
- Weapon hits use the bat's rules (above), including the egg-carrier push cap and the crate and egg hooks. The pair cap is the bat's own and is shared with it: one player can land 4 hits on another in 20 s, bat and gear together. Each victim also gets the bat's 1.2 s immunity after any hit, so nobody is juggled, whatever the range. Gear never damages fruit or plots and never takes anything.
- **Nobody uses gear out of a safe zone,** the bat's own rule ("nobody bats out of one"): not from a plot, a tower floor, a tagged safe box or the beanstalk. The same `zoned()` check the bat makes on its swinger runs on every gear user, so a long-range weapon is never a turret on a plot. The bat itself is not changed.
- **The Royal Arena and the mine put all gear away,** the same way they put the bat away today, and power-ups do nothing there, like the potions and feather fall in the arena.
- **On the beanstalk** nobody is hit (it is a safe zone). The movement power-ups work there the way the Speed and Jump potions already do, and none is stronger than a potion: speed at most x1.6 (the Speed potion's own), jump power at most 76 (the Jump potion gives 85). Air jumps, the dash and gliding make the climb easier but never skip the thin air: only the Potion of Aether gets past it, as today. Gliding only stops fall damage, like feather fall.
- **Bubble Guard and Aurora Guard stop hits from bats and floor gear only,** the "bat" kind of push. They never stop a pet's catch, a blast or the nuke, so nothing bought with cash cancels anything else in the game. A bubble ends the moment its power-up is put away, so it never covers an egg or crate run.
- Power-ups work only while held, on dry land, with empty hands, and never in the mine. Speed and jump take the larger of the power-up and the potion, never both multiplied.

### The ladder

Bat tier 9 (the Soul Fruit Spear) for scale: Power 96, Lift 30, 84 swings a minute, reach 7.5, stun 0.7 s.

| Floor | Tier | Kind | Gear | What it does | Numbers | Price |
|---|---|---|---|---|---|---|
| 1 | - | Bat | Your stage's bat | Free; Applewood at Rebirth 0 up to the Soul Fruit Spear at Rebirth 8 | Power 32 to 96 | FREE |
| 2 | Tower | Weapon | Coconut Club | Hits harder than the best bat; stars spin round their head | Power 104, Lift 32, 78/min, reach 8, stun 0.8 | $192B |
| 3 | Tower | Power-up | Sprint Sprout | Run 30% faster while held | walk x1.3 | $577B |
| 4 | Grand | Weapon | Frost Melon Mace | Whoever it hits walks at half speed for 2 s | Power 110, Lift 32, 76/min, reach 8, stun 0.8, half speed 2 s | $1.5T |
| 5 | Grand | Power-up | Bounce Bean | Jump twice as high, land softly | jump power 72, fall cap 60 | $3.9T |
| 6 | Grand | Weapon | Vine Whip | Reaches twice as far and pulls them to you | Power 92 (a pull), Lift 18, 70/min, reach 16, stun 0.6 | $9.2T |
| 7 | Gilded | Power-up | Bubble Guard | Click: nothing can push you for 6 s | shield 6 s, cooldown 25 s | $21.5T |
| 8 | Gilded | Weapon | Thunder Banana | Lightning jumps to one more player nearby | Power 118, Lift 34, 78/min, reach 8.5, stun 0.85, chain 1 within 12 studs at 70% | $49.3T |
| 9 | Gilded | Power-up | Leaf Glider | Hold jump in the air to glide down and drift far | fall 9 studs/s, air speed x1.4 | $111T |
| 10 | Crystal | Weapon | Seed Blaster | Shoots seeds; knocks back from 70 studs | Power 88, Lift 26, 75 shots/min, range 70, stun 0.6 | $246T |
| 11 | Crystal | Power-up | Root Boots | Pushes on you are half as strong; a bit faster | knock x0.5, walk x1.15 | $542T |
| 12 | Crystal | Weapon | Pineapple Quake Hammer | Slams the ground: everyone within 12 studs flies | Power 124, Lift 40, 40/min, radius 12, stun 0.9 | $1.2Qa |
| 13 | Crystal | Power-up | Twin Coil | Faster and higher at once | walk x1.4, jump power 72 | $2.6Qa |
| 14 | Sky | Weapon | Comet Mango Cannon | A flaming mango that bursts where it lands | Power 132, Lift 36, 30 shots/min, range 90, splash 10, stun 0.9 | $5.5Qa |
| 15 | Sky | Power-up | Dash Pepper | Click to dash forward in a burst of fire | dash 26 studs, cooldown 4 s, walk x1.3 | $11.8Qa |
| 16 | Sky | Weapon | Storm Kiwi Staff | Lightning from far that jumps to two more players | Power 128, Lift 34, 40/min, range 90, chain 2 within 14 studs at 70%, stun 0.9 | $25.2Qa |
| 17 | Sky | Power-up | Cloud Hopper | Jump again in mid-air | one air jump, jump power 76 | $53.6Qa |
| 18 | Star | Weapon | Starfruit Shuriken | Throws three spinning stars at once | Power 116 each, Lift 30, 45 throws/min, 3 stars 18 degrees apart, range 80, stun 0.8 | $113Qa |
| 19 | Star | Power-up | Aurora Guard | A 10 s bubble, and you run faster | shield 10 s, cooldown 18 s, walk x1.25 | $240Qa |
| 20 | Star | Weapon | Nova Durian Blade | Hits everyone in front of you | Power 150, Lift 38, 66/min, reach 11, a 150 degree arc, stun 1.0 | $504Qa |
| 21 | Star | Power-up | Halo Sneakers | Fast, double jump and glide | walk x1.6, air jump, fall 9 studs/s | $1.1Qi |
| 22 | Spire | Weapon | Soul Spire Scepter | A beam that sends everyone in its path flying, frozen | Power 170, Lift 42, 26 beams/min, range 120, stun 1.1, half speed 2 s | $2.2Qi |

### Prices

**Floor k's gear costs $48.1B (the cash Rebirth 3 needs) x 2 for every Mega Rebirth before that floor x k (that tower's floors, each a full plot of fruit).** It is worked out from the game's own formulas (`Config.RebirthCashNeed(3)`, `Config.MegaCashMultiplier(k - 1)`) and rounded like every other price, so a retuned economy carries it.

- **Why:** a Mega Rebirth sends income back to Rebirth 0 and adds a floor. Its gear becomes the first goal of the new lap: buyable around Rebirth 3 or 4 of that lap (the first lap's stages take about 10 min, 20 min and 1.4 h to Rebirth 3's gate, by the 25 Oct pacing estimate, not measured; later laps are faster as merge shards grow x1.5 a Mega Rebirth). The stage after Rebirth 3 earns about 800 times as much a second, so it is never out of reach for long, and it never slows the next Mega Rebirth (floor 2's gear is less than a billionth of what that Mega Rebirth needed).
- **Something found on the way:** each Mega Rebirth's cash alone takes a full Soul Fruit tower about 10, 10, 14, 20, 33, 54 and 93 hours for floors 2 to 8, then 163, 289, 520 and 946 hours for floors 9 to 12, before shards. Most players will meet the first 5 to 8 floors' gear. Not changed by this work.

### Saves

One new field, `gear`:

- `bat`: "m,r", the Mega Rebirths and Rebirths at which the bat was taken ("" = not taken yet). The bat is held while this matches the player's own; any rebirth or Mega Rebirth makes it stale, which is what puts the bat back on the pedestal.
- `own`: the floors whose gear is bought, as "2,3,5".
- `weapon`: the floor of the weapon in slot 1 (0 = the bat); `power`: the floor of the power-up in slot 2 (0 = none).

A document with bought gear is written as version 7, so an older server turns the player away instead of dropping their gear. One with only the bat stays at its version: losing it costs nothing (an older server hands out the bat as today). A document from before the field counts the bat as taken, so nobody loses their bat at the update. The self-test's worst document carries the field at its longest.

### What could break, and how each is guarded

- **The bat.** `GardenService.Bats` only holds a bat back while the player attribute `BatTaken` is exactly false, which only the new module sets. If that module fails to load, every player gets their bat as today. Admin bat picks still work. A player with no plot gets the bat in the bag, as today.
- **Rebirth, Skip Rebirth, Mega Rebirth, plot moves.** Nothing new is spent or kept there; the bat follows the Rebirths and MegaRebirths attributes, and gear lives in the save. Each is Play-tested.
- **Fruit.** The pedestal's footprint is a `PetObstacle` piece in the pen, so fruit never spawns or rests there on any floor.
- **The pen.** The pedestal's stone is built by `GardenService.PlotPen` through one guarded hook, as pen pieces (so new floors build it in), with no change to the three floor styles. If the hook fails, the pen builds as today.
- **Hits.** Gear hits use `G.knockPlayer` and the bat's own checks, the crate and egg hooks, and the egg-carrier push cap; Root Boots chain onto `G.knockScale` without replacing the pets' Steady trait. The two bubbles go through one new guarded hook in the bat's victim rules, asked only about the bat kind of push; if it fails, hits work as today. Tested with two players.
- **Speed, jump and falls.** One guarded hook each in `applyWalkSpeed` (speed and jump take the larger of gear and potion) and in `GardenService.Ambient` (no fall damage while gliding). Neither touches the thin air, the crate's or an egg's carry speed, or the Royal Arena.
- **The hotbar, the Gear tab, the bat button, the mine, Troll morphs, the lift, the nuke and tower tiers.** Checked in Play.
- **Saves.** Version 7 only for bought gear; the self-test's worst document; a leave-and-rejoin test before any publish.

### Left to the builder

The exact pedestal spot on each floor style, checked in Studio; the models' final polish; effects and sounds (the game's own sounds first; Creator Store ones go through the quarantine); the phone buttons' place; ChatGPT prompts for the gear's menu pictures (until they come, menus show the gear's 3D model, the way the bat card does).

## Builder check (22 Nov)

A fresh reviewer read this page cold and asked four things. The game's own rules answered all four, so nothing went back to Samarth. The answers are in the rules above:

1. **Gear from a safe zone?** No: the bat already refuses a swing out of a safe zone, and gear uses the same check.
2. **How fast do the later weapons fire?** Every weapon's rate, lift and stun was already in the ladder on the board; the table now shows them all.
3. **Where does gear turn off?** Where the bat does (the Royal Arena and the mine). The beanstalk follows the potions: power-ups work, hits never land, the thin air still needs Aether.
4. **What do the bubbles stop?** Bat and gear hits only.

My calls in there, for Samarth to check while reading: the shared pair cap, movement power-ups working on the beanstalk, and the bubbles stopping only player hits.

## Status

- **21 Nov, 22:40 PST:** researched, designed, pitched on the board; Samarth picked all four calls. Next: the build plan, the build offline, then a Studio turn (queued behind three other chats).
- **21 Nov, 23:00 PST:** builder check done (above). Build plan written; the gear's settings module is built and tested offline. Building the rest offline now.
- **22 Nov, 00:57 PST:** built and installed in Studio, Play-tested by one player (the bat from the plot, rebirths, Mega Rebirth floors, buying at exactly the price, the panel, the hands, the hotbar), saved to Roblox, not published. Two-player hits were checked only in multi-player tests, which froze the computer; never again (one player and a stand-in from now on). Handoff PR #112. Left: Weilin's OK on the new UI, the gear pictures from ChatGPT, the Inventory's Gear tab, a crate and egg carrier test.
- **22 Nov, 02:30 PST:** round 2, for the forest fights (asked by the inventory and pets chat): every floor weapon can now hit forest animals, the way the bat bonks one. A melee weapon names the animal in front; the quake, the sweep, the shots, the beam and the mango's splash find them on the server. The forest's brain gets the weapon's power as the damage. Frost, the whip's pull and the lightning chains stay player-only, and one use meets at most 8 animals. The bat's script gets three export lines so the weapons measure an animal's body exactly as the bat does. Also the whip's pull speed, which overshot by about 5 studs, is now one number to measure in the turn. Built and tested offline (142 tests); waiting for a Studio turn, sixth in the queue.
