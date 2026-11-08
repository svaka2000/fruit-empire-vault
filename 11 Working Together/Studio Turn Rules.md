---
tags: [fruit-empire, process, studio, rules]
status: current
updated: 2026-11-08
---
# Studio turn rules

How the Fruit Empire chats share one Roblox place without breaking it. Words used here are in [[Fruit Empire Glossary]]. Back to [[Knowledge Base - Start Here]].

## Why turns exist

Both partners, and many Claude chats, change the **same** place through Team Create. A change by one is in the place for everyone at once, and anyone can publish at any moment. So only one writer works at a time, and every turn ends with the place safe to publish and a handoff written. (Source: [PLACE.md](https://github.com/svaka2000/fruit-empire-handoffs/blob/main/PLACE.md), "Which place, and who is who" and "Leave the place safe to publish".)

## The turn, step by step

All commands run from `~/fruit-empire-handoffs`. The full steps are in [CLAUDE.md](https://github.com/svaka2000/fruit-empire-handoffs/blob/main/CLAUDE.md); this is the short version.

1. **Catch up.** `git checkout main && git pull --ff-only`. Read STATE.md, PLACE.md, DECISIONS.md, ui/README.md, then every handoff since your person's last one. Run `gh pr list`.
2. **Take the turn.** `bash scripts/handoff begin samarth`. It marks the turn as yours in STATE.md and pushes that one line. If it says someone else is working, stop.
3. **Open your handoff.** `bash scripts/handoff new samarth <short-topic>`. It makes the branch and the folder `handoffs/<date>-samarth-<topic>/` with a blank `HANDOFF.md`.
4. **Check Studio.** `list_roblox_studios`, then `print(game.PlaceId)` must print `88602028667228`.
5. **Work.** Keep a list of every script changed (full path, new length and fingerprint), every non-script change, every builder run, and every test with its result. Save screenshots in the handoff folder.
6. **End.** Fill every section of `HANDOFF.md`, update STATE.md (only what changed), add decisions to DECISIONS.md, then `bash scripts/handoff check` and `bash scripts/handoff publish`. Publish commits, pushes, opens the PR and merges it.
7. **Stuck turn.** `bash scripts/handoff begin samarth --take-over`, only after the holder confirms nothing of its own is unsaved in Studio.

## Who decides the turn order

> [!important] The chats settle it among themselves
> Samarth, 24 Oct 2026, 15:52 PDT: "dont ask my for permissions simply decide between eachother what takes priority and work non stop. i dont want to keep clicking yes even with bypass permissions enabled"

- A chat takes the free turn itself when STATE.md on main says nobody is working, nobody is in a Play test, and it is next in the queue the chats agreed.
- It tells the other open chats when it takes the turn and again when it publishes, and hands the turn to the next chat by message.
- If two chats are ready at once, the one that asked first goes first.
- Only real design or taste calls go to Samarth. New UI still needs Weilin's OK through the handoff.

Sources: memory `feedback_fruit_empire_chats_self_coordinate.md` and `feedback_fruit_empire_no_turn_permissions.md`; the queue note below.

## Where the live queue is

The queue note: `~/Library/Application Support/Claude/side-session-notes/local_13ce427d-f956-4fc4-9afc-874a9359fd95/studio-turn-rule.md`. It belongs to the Pet and Egg design chat's group of sessions. Its last update (30 Oct 2026, 11:35 PDT) says that chat took the turn for the pets install, and that the Robux shop chat is next. It also lists known links between chats' scripts (the map's `MineDigSync` reader, and where the dig scripts' source lives).

> [!warning] Status on 8 Nov 2026 is not fully recorded
> STATE.md on main still says "Who is working now: Samarth (started 2026-10-30)", and no pets handoff has been published. On 8 Nov the main chat was building the mine's safe pockets in Studio. Whether that work runs under the 30 Oct turn or a new one is not written in any source read for this vault. Check STATE.md and the queue note before touching Studio.

## Safety rules that never change

> [!danger] Always
> - One Studio writer at a time.
> - `workspace:SetAttribute("StudioNoSave", true)` before every Play test.
> - Leave the place safe to publish: finished and tested, or undone.
> - A handoff every turn, even an unfinished one.
> - A Claude never publishes the game. A person publishes, after looking at the screenshots.
> - No AI attribution footers on handoffs, PRs or commits (memory `feedback_no_ai_attribution_footers.md`).

Sources: [CLAUDE.md](https://github.com/svaka2000/fruit-empire-handoffs/blob/main/CLAUDE.md), [PLACE.md](https://github.com/svaka2000/fruit-empire-handoffs/blob/main/PLACE.md), the queue note.

## Shared Studio traps

- Another chat's `execute_luau` can run inside your Play test and change state under you (rebirths, the held tool) or stop Play. An error from a script called `AssistantCommand` that you did not cause is the sign. Re-read the state before each step. (memory `reference_roblox_studio_mcp.md`, 22 Oct)
- A chat that says its Studio work is done may still be running checks, so ask before starting a Play test. (memory `project_fruit_empire_bats.md`)
- On 22 Oct a chat changed `ReplicatedStorage.UIKit` with no turn and no handoff (`Kit.Panel`, `Kit.panelFrame`). STATE.md still asks whoever made it to publish a handoff.

Related: [[Hard Rules]], [[Testing in Studio]], [[Deploy Method]], [[Handoffs Repo Links]].
