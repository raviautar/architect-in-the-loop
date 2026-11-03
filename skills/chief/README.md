# chief

**The operating model.** Layers on `plain-english`. How to run many Claude Code sessions at once without turning into their switchboard.

One idea: you talk to one session, the chief. Every other session is a box (a long-lived workstream in its own tmux window) or a runner (a background worker for one job). The chief routes work, drives the boxes through tmux, keeps a registry of what is running, and reads an append-only ledger the boxes write to. It brings you one thing at a time.

The rules are the valuable part. Each one is in the skill because breaking it cost something: a box typing into the chief, two processes writing one transcript, two chiefs routing the same work, a box started inside a repo that could not be found again.

Needs tmux. Start your registry from [registry-template.md](registry-template.md). See [SKILL.md](SKILL.md).
