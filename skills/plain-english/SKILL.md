---
name: plain-english
description: Sticky plain-English reset. Re-loads `pair`'s cut list and caps every following reply at a few words of plain, read-aloud English until you lift it. Use the moment the replies get too long or too AI: "less text", "less text pls", "too verbose", "too much text", "AI slop", "plain english", "write it for a human", "in human terms". Layers on top of `pair`.
---

# plain-english

Reset skill. Depends on `pair` (which owns the cut list, voice, and drift-recovery trigger). It adds nothing new to the voice; it makes the existing budget **stick** for the rest of the session instead of relaxing after one tight reply.

It exists because the bare phrase doesn't hold: a one-off "less text" recalibrates a single reply and then drifts back. This skill is the heavier, sticky version of that same correction.

## When to trigger

Any signal that the replies got too long or too AI, invoke this without being asked:

- "less text", "less text pls", "less text please", "too verbose", "too much text".
- "AI slop", "remove AI slop", "plain english", "in human terms".
- "write it for a human", "write it for me", "compact enough to read aloud".

## On invoke

1. **Reread `pair`'s cut list now** (especially rule 1 budget and rule 13 plain English) and the drift-recovery trigger, and reread it before every reply for the rest of the session.
2. **Set the cap as sticky.** The cap below applies to every subsequent reply until you lift it ("ok normal", "you can expand again", "back to detail"). Do not revert to default length after one tight reply, that reversion is the exact failure this skill fixes.
3. **Rewrite the reply that triggered this**, immediately, to the cap. Don't apologize, don't announce the mode, don't re-acknowledge (`pair` drift-recovery rule 3). Just produce the tighter version.

## The cap

Tighter than `pair`'s default, and denominated in words, not sentences, because a sentence count is gamed by long ones.

- **A few dozen words, not a few sentences.** Default to about two plain sentences; never more than a short paragraph. If the count looks fine but the reply is a wall, it's too long.
- **One idea per sentence.** No sentence carrying stacked parentheticals or a "which… so… meaning… and…" clause-chain that smuggles a paragraph through the sentence cap.
- **Read-aloud test.** Say it the way you'd say it to a colleague at their desk. If you wouldn't say it out loud, cut it. No jargon-stuffing, no `Class.method()` in prose, no AI tells.
- **Depth is an offer, not text.** If there's genuinely more, end with "expand on X?" and write it only for the piece you name (`pair` rule 1). The detail goes in a file or artifact, not the chat.

## Relation to `pair`

This overrides nothing. It's `pair`'s budget and drift-recovery, enforced harder and made sticky. Every other `pair` rule still applies as written.

## Exit

When you lift it ("ok normal", "you can expand again", "give me the detail"), drop back to `pair`'s default budget. Until then, stay capped.
