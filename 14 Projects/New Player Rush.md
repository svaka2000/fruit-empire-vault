# New Player Rush (19 Nov 2026)

Samarth, 19 Nov: "make fruit empire more engaging in the beginning. if a ten year old plays fruit empire and it takes really long for fruits to spawn their gonna leave. do deep research on other really popular games and first plan everything and then make it super engaging and dopamaine filled."

## What a new player gets today

Measured in a Studio Play test on a brand-new save, 19 Nov 2026.

- The first-join cutscene runs about 55 seconds. While it plays, the plot fills to its cap of 6 fruit.
- One Blueberry grows every 10 seconds. It appears silently at a random spot, with no pop and no sound.
- A Blueberry earns $1 a second. Cash is paid every 5 seconds.
- Every merge uses up 2 fruit. So after the first 3 merges, a new player gets one merge about every 20 seconds, and spends the rest of the time waiting.
- Blueberries are 1.4 studs wide on a big orange plot, so the plot looks empty.
- Nothing on screen says what to do next, apart from the red "!" badges.
- The first rebirth needs about $1.1K, 50 shards and all 5 Common fruit.

## What the big games do

- **Grow a Garden.** You start with 20 Sheckles. A Carrot seed costs 10 and grows in seconds, then you sell it and buy more. Crops can randomly turn Gold (x20) or Rainbow (x50). (Guides on games.gg, Sportskeeda, VideoGamer.)
- **Steal a Brainrot.** The cash-per-second number is always on screen and climbs as you buy. Rebirth trades your progress for a permanent boost. (allthings.how, Kotaku, Wikipedia.)
- **Pet Simulator 99.** Free gifts on a timer, and daily rewards. (BIG Games update posts.)
- **Roblox's own onboarding guide.** Get to the fun quickly, with low thresholds for the first levels. Give starter currency. Set short, mid and long goals and show them on screen. End the tutorial with a moment of joy. (create.roblox.com/docs/production/game-design/onboarding)
- **RoLearn (an opinion blog, not Roblox).** In a simulator, a player should be collecting and watching numbers rise within the first minute. Keep progress visible, and show the day 7 reward from day 1.
- **The DevForum on daily rewards.** One developer's test found that daily rewards alone did not raise playtime. Others say they work best together with playtime rewards.

## The plan

### Phase 1: the first hour

1. **Sprout Rush.** A new garden grows fruit 4 times as fast until Rebirth 1, then 2 times as fast until Rebirth 2. While it lasts, 15% of new fruit are "lucky sprouts", one Spawn Rarity level higher (never above the spawn ceiling). The rush ends at a milestone, not on a timer, so it never suddenly runs out.
2. **Sprout pop.** Every new fruit pops up out of the soil with a squash-and-stretch bounce, a puff of soil, sparkles and a soft pop sound. A lucky sprout flashes gold and shows LUCKY!. The cutscene already promises this ("New fruit pops up on your plot").
3. **Starter goals.** A goal card on screen holds one short goal at a time, 13 in all, from "Pick up a fruit" to "Rebirth 2". Each pays cash or shards. Finishing one plays a celebration (a DONE! stamp, confetti and a fanfare), and the reward flies into the cash or shards bar. When a goal means pressing a button (Upgrade, Index, Rebirth), an arrow points at that button.
4. **Free gifts.** 12 gifts over the first hour of play each day, at 1, 3, 5, 8, 12, 16, 20, 25, 30, 40, 50 and 60 minutes. They give cash (scaled to your income), shards, potions, and x2-fruit boosts. A gift button counts down to the next one, and the gift panel shows every upcoming reward.
5. **NEW FRUIT!** The first time you make each fruit, its name comes up in big words at the top of the screen.

### Phase 2: next

6. **Daily streak.** Days 1 to 7, with the day 7 reward shown from day 1.
7. **Golden fruit.** Rare golden fruit worth more, a surprise reward like Grow a Garden's mutations.
8. **Merge combo.** Quick merges in a row build a combo, and each merge in it sounds higher.
9. **Garden level.** An XP bar that fills with every merge.

## Status

- **19 Nov:** phase 1 (parts 1 to 5) was built in Studio and Play-tested on a fresh save. Measured: a fruit every 2.55 s at Rebirth 0 and every 5.1 s at Rebirth 1. Every goal and gift path was tested. It is in the handoffs repo as `2026-11-19-samarth-new-player-rush`.
- **19 Nov, later:** phase 2 (parts 6 to 9) was built in Studio and Play-tested on fresh saves: the garden level (an XP bar and LEVEL UP!), the merge combo, golden fruit with Golden Hour, and the 7-day DAILY REWARDS card. It is in the handoffs repo as `2026-11-19-samarth-new-player-rush-phase2`.
  - Learned in the tests: Studio's screenshots left out labels set to AlwaysOnTop, so the combo is drawn on the screen instead. `Kit.watchModals` only watches the panels listed when it starts, so new panels go in UIKit's own lists.
- **20 Nov:** a player said the free gifts were too easy and too short. Samarth picked "over days of play": the 12 gifts now count all play time and never start over, from 3 minutes to 24 hours of play in all, bigger with the wait. The rush's screens also wear the UI kit a friend of Samarth's made with Codex.
- **Not published yet** (phase 1 and phase 2). Not checked on a phone or with two players. Nobody has listened to the sounds.
- **Waiting:** Weilin's OK on the new UI of both phases.

## Rules it keeps

- Only stages 1 and 2 (no rebirths, or 1) get faster. Later stages keep their pacing.
- The goals and gifts use Samarth's kit art, Luckiest Guy and the game's own sounds. No icon is code-drawn.
- New numbers live in their own module (GardenConfig is near its 200,000-character limit).
- Weilin has to OK the new UI.
