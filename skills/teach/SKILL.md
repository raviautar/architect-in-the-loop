---
name: teach
description: Learning-mode pacing for walkthroughs and explanations. Use when the user says "walk me through", "explain", "help me understand", "beat by beat", "I'm new to X", or otherwise signals they want to learn a concept rather than execute a task. Layers on top of `plain-english`.
---

# teach

Learning-mode skill. Depends on `plain-english` (the voice and its rules). Adds pacing rules specific to teaching one concept at a time.

## When to trigger

User signals learning intent:

- Explicit verbs: "walk me through", "explain", "help me understand", "show me how X works", "beat by beat".
- "I'm new to X" / "I don't get how Y works".
- The user pushes back on a wall of explanation with "slow down", "you went too fast", "I don't understand".

## Pacing rules

One concept per message. Anchor each concept two ways at once: the actual snippet inline, AND a clickable `path:LINE` pointer to it (see "Anchor with real code"). After landing the concept, ask one short checkpoint question:

> with me? / want to keep going? / next?

Wait for "yes" / "keep going" / "next" before pre-loading the next concept. Do not stack three concepts in one message because they feel related.

## Anchor with real code, and a clickable pointer

Every time you reference code or explain an implementation, do BOTH, never one without the other:

1. Render the actual snippet inline (read the file first; never fabricate line numbers).
2. Point to it with a clickable `path:LINE` pointer the editor can resolve, e.g. `src/api/handler.py:142` (a range when it helps: `src/api/handler.py:142-160`). An absolute path is the most reliably clickable; a bare filename or "see line 123 of foo.py" in prose is not clickable and does not count.

The snippet shows the code; the `path:LINE` pointer is how the reader jumps straight to it.

## One running diagram, extended

For anything structural (workflows, pipelines, parent/child topologies), draw one small ASCII diagram early and EXTEND that same diagram as the explanation deepens, instead of re-describing structure in fresh prose. When a wall of prose leaves the reader lost, extending the existing diagram in place is what un-loses them. Never coin shorthand or a metaphor ("hops") without grounding the term against the code or the diagram in the same sentence.

## Backing up

If the user says "you went too fast", "I don't understand", "wait, what's X?", that means the previous beat didn't land. Back up to the last beat they confirmed and restart from there. Don't keep pushing forward and add new context, go back, re-anchor, slow down.

## Where this loosens `plain-english`

A short bullet sequence (2 or 3 items) is allowed when the concept really is a list of related primitives that need to land together. Use sparingly; prose plus one code reference is still the default.

Explanations can run longer than a normal reply, but still one concept per message.

The code anchor is a local clickable `path:LINE` pointer instead of a GitHub link, because clickable navigation into the editor is the whole point of teaching from real code.

Every other `plain-english` rule still applies.

## When to exit teach mode

When the user signals they understand and want to act: "got it, now let's…", "ok, do it", "ready, make the change". From that point, drop the checkpoint questions and return to `plain-english`'s default.
