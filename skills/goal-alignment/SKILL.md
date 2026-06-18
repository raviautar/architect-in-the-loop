---
name: goal-alignment
description: Align on a plain-english brief (Problem, Implementation, Verification) for a feature or fix, drive all preparation to completion (your understanding, open decisions, live prerequisites), then hand the agent a self-correcting loop. The skill owns the thinking and the preparation; the loop owns the looping. Each run is one round with its own fresh brief file. Use when the user says "align the goal", "build a goal", "plan a goal", "build a loop", "write the brief", "write the piv", "design the proof", "next round". Inherits `pair` and `teach`.
---

# goal-alignment

Design-and-preparation skill, and the heart of this repo. It produces one artifact (the PIV brief, co-written with you in plain english), one handoff (a self-correcting loop), and a prepared runway. It runs no implementation loop itself.

**Core principle: every decision made and every prerequisite proven during preparation is one the loop never stops for.** Preparation is the most important phase, more important than the implementation itself; never do it sparingly. The loop opens with zero decisions left to make. Preparation is complete only when all three hold:

1. You say you fully understand what is going to happen, reached through teach pacing and the Mermaid diagrams that carry the problem and the plan.
2. No open questions or decisions remain; each was walked and resolved into the brief.
3. Every preparation task that CAN be done before the loop HAS been done now, live (mint the token, capture the real fixture, prove the environment is up, prove each verification layer is actually executable).

**Rounds, not one growing goal.** Each loop run is one round with its own fresh brief file. A round is scoped to one functional outcome; its verification layers are numbered from 1. The next round gets a NEW file, carries forward only the still-open gap, and renumbers from 1 again. Never edit-pile onto the last round's brief: an accreting plan becomes too complicated for a human head to follow, and stale, already-passed layers drown the live work.

Inherits `pair` (voice, cut list, drift-recovery) and `teach` (pacing). The brief is co-written, so whenever you need the codebase explained mid-design, switch to teach pacing: one concept per message, anchored to a real snippet or a diagram you extend, checkpoint before the next. You read everything this skill produces; digestible beats complete.

## The engine: a loop with an independent grader

The concrete engine here is Claude Code's `/goal`, but the methodology applies to any loop that re-grades a condition each turn.

- `/goal <condition>` starts a turn immediately with the condition as the directive. After every turn a small fast model judges the condition against the conversation: "no" feeds its reason back and the agent starts another turn; "yes" clears the goal. A real loop with an independent grader, not a single pass.
- The evaluator cannot run tools or read files. It judges only what the worker surfaced in the transcript. Every proof must therefore land in the conversation (run the suite, show the output, render the UI result). The flip side: the evaluator can be satisfied by a claim of success the worker never actually demonstrated, so the brief must force user-visible proof (see Verification).
- Bound every loop with "or stop after {N} turns" inside the condition.
- The command is yours to run; the agent cannot invoke it. The skill ends by handing the condition to you in a code block.
- One condition, no phases. The loop does not orchestrate; the PIV brief does that in plain english. Split into sequential rounds at real boundaries (one functional gap closed, then the next), never to manage context.

## The artifact: the PIV brief — Problem, Implementation, Verification

Plain english, compact enough to read aloud. Fill `piv-template.md`. Mermaid diagrams (`flowchart TD`) carry the structure, and they come AFTER the prose in each section, never opening the brief: Problem gets one or more showing the current flow and exactly where it breaks; Implementation gets one or more showing what is going to happen (where the change lands, the resulting flow). The loop and you read the same pictures.

**Problem.** What is broken or missing, in plain english, and how to observe it so "fixed" is checkable. For a fix-round, name the previous round's brief file and the specific functional gap it left open (e.g. "the second request in a session is served stale data; only the first is correct"). Scope the round to that functional gap only, nothing wider. After the prose, draw one or more Mermaid diagrams of the current flow with the exact point it breaks marked.

**Implementation.** The decisions made (every open decision walked during design with its resolution, one line each, so the loop inherits zero), then how it lands: order and why, commit strategy, what stays unchanged and what is explicitly untouched, and the code-hygiene constraint: the worker re-reads `pair`'s cut list (rule 18) before writing code and adds a comment only where it states a constraint the code cannot show, at the file's existing density (usually almost none), and strips any slop comment it or a prior round left. Keep the change minimal; cut edits the round does not need. After the prose, draw one or more Mermaid diagrams of what is going to happen: where the change lands and the resulting flow.

**Verification.** How the implementation is proven to have landed. Three parts, in order:

1. **Prerequisites** — the gate before the loop starts; NOT a verification layer (a prerequisite proves the stack is ready, not that the feature works). The exact health command per touched service with its green signal, PLUS a client-UI reachability check: load the actual UI a user touches, sign in, reach the exact surface the feature lives on. Backend-green alone does not prove the user path works. Run this as a QA engineer: validate the stack AND propose any extra feature-readiness checks the feature needs. Everything provable now is proven live during design (fixtures captured, tokens minted, the workaround recorded for any step automation cannot drive). The stack decays, so prerequisites re-run right before arming and gate the loop's start; a red service is brought up or the loop does not start.
2. **Layers** — renumbered from 1 each round, ordered cheapest-feedback-first. Each layer names the seam it covers and the failure class it catches, gives the exact command and the visible evidence, and is driven end-to-end through the client where the feature is user-facing: assert the rendered result in the UI AND the backend signal, never the backend signal alone (a replay test that never loaded the client can pass while the user-facing path is still broken). When the bug is about state across turns or requests, the layer must assert the specific occurrence that was broken (the second turn), not the happy-path first one, asserting only the first is how a loop "passes" while still broken. Each layer must be proven executable during prep, a layer you cannot actually run gets faked or skipped under loop pressure (a gesture automation can't drive an OS file picker, but setting the input directly may reach the same path; find that in prep). **A layer passes only by being run for real with its evidence in the transcript.** A static proxy or logical argument (a byte-identity diff, "structurally equivalent", a prior round's result) is a separate check, never a substitute for executing the layer the round names; disclosing the swap does not make it pass.
3. **Critic gate** (mandatory final layer) — a fresh context window that did not write the code reviews the diff and the verification that took place, judging whether the agent took shortcuts to make its work easier, whether any layer bypassed what it claimed to cover and must be redone, plus a code review and the laziness attack list: hardcoded values where config belongs, swallowed or blanket-caught errors, tests weakened/skipped/special-cased to go green, a verification layer marked passed by a proxy or argument instead of an actual run (byte-identity standing in for a runtime check, a reused prior-round result), copy-paste past an existing abstraction, leftover stubs/TODOs, AI-slop comments (scan every added comment; strip next-line narration and change-markers, keep only constraint comments), local workarounds leaking into the diff. The loop is graded by an evaluator that only reads the transcript, so it has structural pressure to cut corners; this is the counterweight. The critic must POST its finding list and each finding's disposition (the diff or line for a fix, or the rebuttal) into the conversation; "critic gate done" as a bare claim is not a proof the grader can check. Every finding fixed or explicitly rebutted.

**Binary completion.** One line: all layers green with the feature demonstrated user-visibly (not merely grader-satisfied) + critic findings dispositioned + any human follow-up.

Where it lives: each round is its own file (e.g. `briefs/{name}-round{N}.md`), never an edit pile-up on the last round's file. Save it even for a same-session loop: long loops auto-compact and the file is what survives the summary; the worker re-reads it after a compact.

## The handoff: Problem and Implementation steer the worker, Verification becomes the condition

Do NOT paste the whole brief into the loop condition. The condition is re-graded every turn, and Problem/Implementation prose is not yes/no checkable; the grader needs the proof checklist, nothing else.

Every condition opens with the prerequisite as a one-time turn-1 entry gate, then a `done means all of` clause that is ONLY the verification layers plus the critic gate. Keeping the gate out of the done-clause is load-bearing: the grader re-judges the whole string every turn, so if "stack green" were a completion proof the loop could never clear once the worker stopped re-printing health output. So the worker confirms prerequisites once (self-healing a dead dependency, or reporting the blocked prerequisite and stopping so an unachieved status reads as a halt, not a pass), and thereafter the grader judges only the layer proofs and the critic.

Match the condition's length to where the detail lives. When it references the brief file, the condition stays short and human-readable: point at the file for the layer specifics instead of restating each command + UI result + backend signal. Spell the proofs out in full only in the same-session form, where there is no file to lean on.

Same session (brief co-written there, Problem and Implementation already in context, no file referenced):

```
/goal Before implementing, confirm prerequisites once: {service health commands} and the
feature's UI entry point loads ({client surface}); if anything is red, restart it and
re-confirm, or report the blocked prerequisite and stop. Then implement. Done means all of:
{layers as binary proofs, each naming the command, the rendered UI result for the broken
turn, and the backend signal}; a fresh-context critic review posted in-transcript (including
a scan of every added comment, with AI-slop comments stripped) with each finding fixed (diff
shown) or rebutted; {constraint: what must not change}; or stop after {MAX_TURNS} turns.
```

Fresh session or headless (brief saved to a file; keep this short, the file carries the detail):

```
/goal Implement round {N} per the brief at {file path}. Confirm its prerequisites once
before implementing; if anything is red, restart and re-confirm, or report it and stop.
Done means the brief's verification layers all pass (each shown in-transcript with the
rendered UI result + backend signal) and its critic gate is posted (including a scan of
every added comment, with AI-slop comments stripped) with every finding fixed or rebutted;
or stop after {MAX_TURNS} turns.
```

The first line makes the agent read the file on turn 1, so the full PIV enters the transcript for both worker and grader.

Choosing the session: same session is the default for PR-sized work; the worker keeps everything it learned exploring the codebase during planning, a filling window auto-compacts mid-loop without stopping it, and the saved file anchors the plan through compaction. Go fresh when the work spans days or planning was long and exploratory: clean window, with the brief carrying the load-bearing discoveries.

Right-sizing: the gate preamble + end state + how it is proven + one constraint line + a turn cap. Outcomes only, never process: no promise tokens, no "one slice per turn", no "use a subagent for X". The model manages its own slicing and context.

## Five moves

```
- [ ] 1. Frame the round. New feature or fix-round? If a fix-round, the Problem points at
        the prior brief and the gap the between-rounds audit found. If the task arrives with
        no context, gather it first. Co-write Problem and Implementation in pair voice;
        explain unfamiliar code in teach pacing (one concept, one anchor, checkpoint). Draw
        the Problem diagram(s) here, after the prose.
- [ ] 2. Walk the open decisions, one at a time. Enumerate every decision the implementation
        would otherwise hit, resolve each with you in teach pacing while building up the
        Implementation diagram(s); each resolution folds into Implementation's "decisions made". A decision
        settled now never ships to the loop.
- [ ] 3. Prove the prerequisites live, as a QA engineer. Validate the stack through the exact
        path each layer will use, including loading the client UI and reaching the feature's
        surface, not backend probes alone, and propose any extra feature-readiness checks.
        Execute every prerequisite that can be done now: mint the token, capture the real
        fixture, and PROVE each planned layer is actually executable. A layer you cannot
        execute is not a layer.
- [ ] 4. Design the verification. Renumber layers from 1 for this round, cheapest-feedback-
        first; for each: the seam it covers, the failure class it catches, the exact command,
        and the dual assertion (rendered UI result + backend signal). Then the mandatory
        fresh-context critic gate. V is written now, with the Problem, never after
        implementation.
- [ ] 5. Re-verify and hand off, only when you confirm full understanding and no open
        decisions remain. Re-run the prerequisite checks right before arming (the design-time
        smoke goes stale); a red service is brought up and confirmed with you, not left for
        the loop. Then emit the compiled loop condition in a code block (ALWAYS include it).
        Stop; do not start implementing.
```

## Between rounds

- The loop's "done" means the evaluator signed off on the transcript, not that the feature works.
- Audit with fresh context: spawn an agent that did not write the code to walk the implementation and each verification layer and find what is missing or faked. Fresh context beats self-critique.
- The gaps become the next round's Problem. Start a NEW brief file numbered from 1, carry forward only the open gap, strike through (`~~layer~~`) anything already proven so you see remaining work at a glance. Never grow one ever-expanding goal.

## Anti-patterns

- ❌ One ever-growing goal/brief. Each run is a fresh round file, layers renumbered from 1; accreting makes the plan too complicated for a human head.
- ❌ Treating a prerequisite as a verification layer (or vice versa). The stack/UI-reachability gate gates the loop's start; it is not a proof the feature works.
- ❌ "Done" on backend-green alone. A layer asserts the rendered result in the client UI plus the backend signal; a replay "UI test" that never loaded the client hides user-facing gaps.
- ❌ Committing a verification layer not proven executable in prep. A layer that cannot be driven gets faked or skipped under loop pressure.
- ❌ Pasting the whole PIV brief as the condition. The grader needs binary proofs; Problem and Implementation steer from the conversation or the file.
- ❌ Process rules in the condition (promise tokens, per-milestone slicing, subagent instructions).
- ❌ Proofs the evaluator cannot see: a passing suite never run in-transcript, a UI result never rendered.
- ❌ No turn cap in the condition.
- ❌ Skipping the critic gate, or letting the worker self-review instead of a fresh context.
- ❌ Compiling while open decisions remain or you have not confirmed understanding. The loop implements; it does not decide.
- ❌ Deferring to the loop a prerequisite that could be proven live today. A precondition discovered at turn 40 is the failure move 3 prevents.
- ❌ Rushing preparation to get to the loop. Preparation is the product; the loop is the cheap part.
