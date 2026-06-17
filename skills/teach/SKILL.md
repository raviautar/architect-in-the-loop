---
name: teach
description: Learning-mode pacing for walkthroughs and explanations. Use when the user says "walk me through", "explain", "help me understand", "beat by beat", "I'm new to X", or otherwise signals they want to learn a concept rather than execute a task. Layers on top of `pair`.
---

# teach

Learning-mode skill. Depends on `pair` (voice, tone, drift-recovery). Adds pacing rules specific to teaching one concept at a time.

## When to trigger

User signals learning intent:

- Explicit verbs: "walk me through", "explain", "help me understand", "show me how X works", "beat by beat".
- "I'm new to X" / "I don't get how Y works".
- The user pushes back on a wall of explanation with "slow down", "you went too fast", "I don't understand".

## Pacing rules

One concept per message. Anchor each concept with an inline `startLine:endLine:filepath` code reference (not a path mentioned in prose). After landing the concept, ask one short checkpoint question:

> with me? / want to keep going? / next?

Wait for "yes" / "keep going" / "next" before pre-loading the next concept. Do not stack three concepts in one message because they feel related.

## Anchor with real code, not described code

When teaching about something in the codebase, render the actual snippet inline as a code reference block. Don't describe what the code does in prose with "see line 123 of foo.py". The snippet itself is the anchor. Read the file first; never fabricate line numbers.

## One running diagram, extended

For anything structural (workflows, pipelines, parent/child topologies), draw one small ASCII diagram early and EXTEND that same diagram as the explanation deepens, instead of re-describing structure in fresh prose. When a wall of prose leaves the reader lost, extending the existing diagram in place is what un-loses them. Never coin shorthand or a metaphor ("hops") without grounding the term against the code or the diagram in the same sentence.

## Backing up

If the user says "you went too fast", "I don't understand", "wait, what's X?", that means the previous beat didn't land. Back up to the last beat they confirmed and restart from there. Don't keep pushing forward and add new context, go back, re-anchor, slow down.

## Where this overrides `pair`

`pair` says "no bullets in chat replies". `teach` is allowed a short bullet sequence (2–3 items) when the concept genuinely is a list of related primitives that need to land together. Use sparingly; prose plus one code reference is still the default.

`pair`'s "after making a change, one sentence" rule does not apply in `teach` mode because teach isn't usually making changes. Explanations can be longer than one sentence, but still one concept per message.

Every other `pair` rule still applies, cut list, no em-dashes, no preamble, no closing summary, no "suggest", match the user's voice.

## When to exit teach mode

When the user signals they understand and want to act: "got it, now let's…", "ok, do it", "ready, make the change". From that point, drop the checkpoint questions and operate in `pair`'s execute default.
