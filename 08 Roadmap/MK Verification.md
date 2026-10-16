---
tags: [roadmap, mk, verification]
date: 2026-10-09
---
# King removal, crown cosmetic, Settings, Index tabs: verification

Samarth's asks from 8 Oct (his words are in `~/fruit-empire/docs/mk/requests-2026-10-08.md`):
- Remove the King's Throne from the cloud island. Keep the island.
- The crown stays, bought with shards like an aura. It shows only on the richest player.
- Auras show all the time.
- A Settings panel that can hide the crown and has everything a player needs, simple but high quality.
- Rarities listed on the left of the Index.

Everything is deployed to Studio and checked in a solo Play test. It is not published yet. Code: `~/fruit-empire/src-m2` tag `mk-deployed` (8af160d).

## The King's Throne is gone
- `KingRemoval` moved the Royal Gate, the Royal Capital (hall, towers, arena, courtyard, and their fire and fountain sound loops) and the throne's version of the cloud into `ServerStorage.KingSystem_backup_2026-10-09`. Nothing was deleted. `KingRemoval.undo()` puts it all back.
- The cloud is back to its shape before the throne (no arena hole). The castle has its back wall, its 20 yard flowers and its stone colours again.
- The 18 King scripts are in `ServerStorage.Scripts_backup_2026-10-09`.
- The "Royal Procession" and "Sword Fight" music is gone. No sound is left on the cloud island.
- Nothing royal is left in the running game: no royal tags, no royal screens, no royal music.

![[mk-before-cloud-garden.jpg]]
![[mk-after-cloud-garden.jpg]]
*Before and after: the Royal Capital, the arena and the light beam are gone. The cherry trees, the castle and the beanstalk stay.*

![[mk-after-cloud-garden-close.jpg]]
*Where the arena was: whole cloud, no hole.*

## The crown and the auras
- The shop's King tab is now called **Style**.
- **The crown** costs 15,000 shards. You never "equip" it. Once you own it, it floats over your head whenever you are the richest player on the server.
- **The auras** (Fire, Water, Slash, Rift) are worn one at a time and show all the time. Press the one you wear to take it off.

| Checked | Result |
|---|---|
| Buy the crown | 15K shards taken. It appears over your head because you are the richest. |
| Show my crown: Off | The crown disappears; On brings it back. |
| Buy the Fire Aura | 4K taken; it goes on straight away (flames on the arms, legs and body). |
| Take off | The aura goes away; the card says Owned with a green Equip. |

![[mk-style-tab.jpg]]
![[mk-style-tab-owned.jpg]]
![[mk-fire-aura-and-crown.jpg]]

## Settings
A gear beside the speaker opens it. Seven settings in three groups:
- **Sound:** Music, Sound effects (sliders).
- **Look:** Show my crown, Show my aura, Other players' auras (and their crowns).
- **Game:** Money popups, Camera shake.

Reset puts every setting back. Choices are saved in your profile, so they follow you to every server.

| Checked | Result |
|---|---|
| Each toggle | Reaches the server on the tap. |
| Music at 50% | The music volume halves (0.126 = 0.35 x 0.5 x the track's own level). |
| Sound effects at 50% | Every non-music sound in one group at 0.5, the bat's too. |
| Reset | All seven back to default. |

![[mk-settings-panel-first-try.jpg]]
*First try: Camera shake was cut off at the bottom, and the plot sign showed through the words.*

![[mk-settings-panel.jpg]]
*After the fix: all seven fit, the body is darker, and each slider shows its whole track.*

## Index rarity tabs
Done earlier on 8 Oct; see `mk-index-rarity-list.jpg`. Tap a rarity on the left and the grid glides to it. Locked rarities are grey "?" tabs.

## Cloud Garden: Sky Stars (9 Oct)
Your ask: "add more satisfaction to the cloud islands." The design is in `docs/mk/SKY-GARDEN.md`.

- **12 gold stars** hidden around the Cloud Garden: 9 on the cloud, 3 on floating cloud puffs. Walk into one and it pops, sparkles, rings a chime (higher for each star in a row) and pays 3 shards (more after rebirths). All 12 pay a 24-shard bonus. They come back every day at 00:00 UTC, and each player has their own.
- **A counter** under Lock and Upgrade shows a star and "found/12" while you are on the island. It turns gold at 12/12 and hides under every menu.
- **Five bounce mushrooms.** The three beside the puffs throw you on an arc that lands on top of the puff, right through its star. The other two bounce you straight up for fun.

| Checked in Play | Result |
|---|---|
| Arc onto each puff, from the cap's centre, near edge, far edge and side | Every one landed on the top, within 0.4 studs of the centre. |
| All 12 stars | +60 shards in all, the bonus notice, the counter turns gold. |
| Arrival notice | "Welcome to the Cloud Garden! Find the 12 Sky Stars." |
| Counter | Hides under the Shop and on the ground. |
| Console and save | No warnings or errors. Your save unchanged. |

![[mk-sky-puff-and-mushroom.jpg]]
*A bounce mushroom and the floating puff it throws you onto.*

![[mk-sky-star-and-counter-first-try.jpg]]
*A star on the cloud and the counter (first try: the star and "3/12" sat to the left of the plate).*

![[mk-sky-standing-on-puff.jpg]]
*Standing on a puff after the arc. The counter is gold at 12/12.*

![[mk-sky-counter-centred.jpg]]
*After the fix: the counter's star and count are centred. A puff with its star, top right; mushrooms by the trees.*

What changed after the first Play test: the first version bounced you straight up and you had to steer onto the puff. Steering on the way up hit the puff's underside and fell short. So the puffs are now wide at the bottom and small on top, and the puff mushrooms aim for you.

## Left for Samarth
- Your "go" before Publish (M6, M7, M9 and this).
