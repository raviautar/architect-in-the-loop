---
name: chief
description: The operating model for running many Claude Code sessions at once. One session you talk to, the chief, routes work between persistent tmux-hosted "boxes" and short-lived "runners", keeps a registry of what is running, and coordinates through an append-only ledger. Use when the user mentions the chief, boxes, runners, the tmux setup, "put X in a box", "spin up a box", "reopen the box", "route this", "who's working on what", or when deciding whether a request should run as a background job or go to a long-lived session. Inherits `plain-english`.
---

# chief

One session orchestrates. Every other session is a box or a runner. You talk to the chief; the chief keeps everything else moving and brings you one thing at a time.

Inherits `plain-english` for voice. Needs tmux and Claude Code.

## Three terms

- **Box**: a persistent, named workstream in its own Claude Code session, hosted in its own tmux window. You can resume it, and you can jump in and type into it at any time. "Put X in the box" hands work to a named box; "reopen the box" resumes one. Keep one ad-hoc box as the standing home for work that has no box of its own yet.
- **Runner**: a short-lived background worker (a subagent). Spawned for one job, reports back, gone. Nothing to resume or type into.
- **Chief**: the session you talk to. It routes work, drives boxes through tmux, spawns runners, and is the only session you must keep open.

A box fits a workstream: a product area, a service, a recurring lane, a long run of tickets. A runner fits a task: a sweep, a review, an extraction, one research question.

## Topology

- One tmux session named `chief`. Window 0 is the chief itself; every box gets its own window.
- A box's tmux window name and its Claude session name are the same word.
- Turn on mouse mode (`set -g mouse on` in `~/.tmux.conf`). Move between boxes with the status bar, `ctrl-b <n>`, `ctrl-b n` / `ctrl-b p`, or `ctrl-b w` for the list.
- Closing the terminal kills nothing. `tmux attach -t chief` brings the view back from any terminal.

## Two files

The chief keeps its state in two plain files in one folder at your workspace root. This skill uses `~/workspace/.agents/`. Keep both out of any public repo.

**The registry** (`.agents/chief/registry.md`) says what exists right now: one row per box, with its tmux window, its Claude session name and full id, its working directory, and a one-line status. Start from [registry-template.md](registry-template.md). Update it whenever a box is created, renamed or retired. After a reboot, it is all you need to bring every box back.

**The ledger** (`.agents/ledger/<yyyy-mm>.md`) says what happened. It is append-only: one entry per decision, change, incident or hand-off, newest at the bottom.

```
### 2026-03-02 · change · search
**Search reads from the new index; the full-table scan is gone.**
Why it changed, what changed, and how to undo it, in two to four sentences.
for: api
refs: <commit sha> · <PR link or file path>
```

`for: <box>` addresses an entry to a box. A box reads the entries addressed to it before it starts a task. The chief reads its own (`for: chief`) at every turn. Boxes coordinate through the ledger, not through each other's windows.

## Workflow

1. **Read the ledger at every turn.** The chief's inbox is the entries addressed to it, plus whatever other boxes logged since its last turn. Work that list one item at a time.
2. **Route incoming work.** Task-shaped work goes to a runner in the background while you keep talking. Workstream-shaped work goes to its box. When no box fits, propose a new one instead of quietly doing the work in the chief. "Put X in the box" with no box named goes to the ad-hoc box.
3. **Drive a box through tmux.** Read the pane, type the message, read the pane again, then press Enter:

   ```sh
   tmux capture-pane -p -t chief:<n> | tail -40
   tmux send-keys -t chief:<n> -l "<message>"
   tmux send-keys -t chief:<n> Enter
   ```

   Never sit in a loop waiting for the reply. Arm a watcher (R9) and end the turn.
4. **Start and resume boxes the same way every time.** A new box gets an id you choose up front, so the registry row can be written before the box says a word:

   ```sh
   id=$(uuidgen | tr A-Z a-z); echo "$id"
   tmux new-window -d -t chief: -n <name> -c ~/workspace
   tmux send-keys -t chief:<name> "claude --name <name> --session-id $id" Enter
   ```

   Resume an existing box by its full id, `claude --resume <full-session-id>`, from the directory in its registry row. Resuming by name can open a picker with several matches.
5. **Keep the registry current** on every create, rename and retire.
6. **Present serially** (R1).

## Rules

Each rule is here because breaking it cost something.

**R1. Parallel execution, serial presentation.** Your attention is serial; the chief's is not. Run everything that can run in parallel, but bring you one item, one decision, one question per message, in a few short lines. When two things wait on you, the message carries the more important one and holds the other. A status line about background work may ride along; a second question may not.
*Why:* ten boxes asking ten questions at once is ten context switches, and every answer gets worse.

**R2. Delegate by default; the chief's turns stay seconds-short.** Anything beyond routing, relaying, reading a file or a one-line command goes to a box or a runner, even when the chief could do it itself. What the chief keeps: read its inbox, put things in order, present.
*Why:* a chief busy doing work is a chief you are waiting on. Its whole value is that it answers in seconds.

**R3. Nothing types into the chief's pane.** Boxes report through the ledger, never by typing into the chief's window. Box-to-box nudges over tmux are fine; a nudge only points at a ledger entry and never carries the work. The one exception is your explicit word, in that box, for that one message: the box then types one short nudge that says it is relayed with your OK and points at the ledger entry. The permission is per message, not standing.
*Why:* text typed into the chief arrives looking like your own message, in the middle of your own thought, and the chief acts on it as if you had said it.

**R4. One live process per session.** Before resuming a session, check that nothing already runs it: `pgrep -fl "claude.*<session-id>"`.
*Why:* two processes on one session are two writers on one transcript, and the history forks under you.

**R5. One chief at a time.** Before acting, the chief checks that no other chief runs. A second one (an old window, a background fork) gets its open threads written to the ledger and is stopped.
*Why:* two chiefs route the same work twice, run duplicate watchers, and split your inbox between them.

**R6. Every session starts at the workspace root.** The chief and every box start in `~/workspace`, never inside a repo, worktree or subfolder. A box may `cd` into a repo to run a command; its process and its transcript stay at the root.
*Why:* Claude Code stores a transcript under the folder the session started in, and resume looks for it there. A box started inside one repo is hard to find, hard to move, and stuck once its work spans two repos.

**R7. A box's window name and session name change together.** On create, rename or repurpose, set both, `tmux rename-window -t chief:<n> <name>` and `/rename <name>` typed into the box, then check that they match.
*Why:* nothing keeps them in step. A repurposed session can keep its old name for weeks, and then the name you see says the wrong thing about what the box does.

**R8. A relay says "from you" only for your literal words.** When the chief passes your words to a box, it labels them as yours. Its own steps and additions are labeled as the chief's.
*Why:* a box treats "the operator said" as a decision. It must never act on a decision you did not make.

**R9. Watchers are scripts, never a model on a timer.** To wait for something (a box going idle, a new ledger entry, a review on a pull request), the chief starts a background script that exits the moment the thing changes. Claude Code wakes the chief when a background command exits. The chief reads what the script printed, routes it, and arms the watcher again.

```sh
#!/bin/bash
# wait-idle.sh <window>: exits once the box has looked idle for three checks in a row, or after 90 minutes.
w=$1; n=0
for i in $(seq 1 540); do
  if tmux capture-pane -p -t "chief:$w" | tail -3 | grep -q "esc to interrupt"; then n=0; else n=$((n+1)); fi
  [ $n -ge 3 ] && { tmux capture-pane -p -t "chief:$w" | grep -v '^\s*$' | tail -30; exit 0; }
  sleep 10
done
echo "still busy after 90 minutes"
```

The grep looks for the line Claude Code shows while it works; check yours, it can change between versions.
*Why:* a script costs nothing while it waits and fires exactly when something changes. A model polling on a timer spends tokens on every tick and still misses the moment.

**R10. A wake word means a live sweep.** Pick a short phrase, such as "catch me up". When you say it, the chief runs every check live before answering: the ledger, each box's pane, the reviews and comments waiting on you. It routes what it finds, then answers with the one item that matters most.
*Why:* the chief only knows what it last read. Without a sweep it answers from an old picture.

**R11. Write files with the file tool, never a shell heredoc.**
*Why:* a heredoc inside a shell command can trigger a permission prompt, and a box nobody is watching sits blocked on it until you happen to look.

**R12. Box names stay out of commits, pull requests and anything a teammate reads.** A commit says what changed and why.
*Why:* boxes are your private scaffolding. A teammate reading "the search box did X" learns nothing about the change and a lot about your setup.

## Guidelines

Steering, not exhaustive.

- Never rename, move or delete something a box is using in the middle of a step (a branch, a file, a stored credential). Tell the box, wait until its step is done, then change it.
- Two boxes in one checkout step on each other's uncommitted work. A box that commits gets its own `git worktree`.
- Ledger writes are appends, never a rewrite of the file. Before creating a new month's file, list the folder: another box may have created it already.
- One canonical session per box. Retire duplicates once their history is in the ledger.
- Script the rebuild once. After a reboot, a dozen lines of shell can read the registry, recreate the `chief` session, open one window per row, and resume each box by id from its directory. Doing it by hand, box by box, is the step that makes people give up on the setup.
