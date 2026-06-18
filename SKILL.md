---
name: catch-up
description: Help a human engineer understand code that was recently written or changed (often by Claude) so they stay in control of code they didn't type. Use when the user asks to "catch up", "walk me through what changed", "explain these changes", "I lost track of what you did", or wants to review/understand recent edits, a diff, or a branch before committing, reviewing, or building on it.
license: MIT
---

# Catch Up

Help a human rebuild their mental model of code that was just written or changed — usually by Claude — so they understand it well enough to own it, review it, and extend it. The user is a capable engineer who has fallen behind the rate of generated code. Treat them as a peer catching up, not a beginner being taught.

**Core principle:** The goal is *their understanding*, not your summary. Optimize for the human leaving with an accurate mental model and the confidence to modify the code themselves — not for producing an impressive document. Bias toward honesty (including "I'm not sure why this is here") over a tidy narrative.

## Step 1 — Find what changed

Establish the exact change set before explaining anything. Don't rely on memory of the conversation; read the real diff. Check sources in this order and use whichever fits:

```bash
git status                          # what's modified/untracked right now
git diff                            # unstaged working-tree changes
git diff --staged                   # staged changes
git log --oneline -15               # recent commits, to scope a range
git diff <base>...HEAD              # a whole branch vs its base
```

- If the user names a scope ("the auth refactor", "today's work", "this PR"), map it to a concrete diff or commit range.
- If there's no git repo or no diff, fall back to what was edited in this session — but say so, since it's less reliable.
- If the change set is large, **scope it with the user before diving in** rather than explaining everything shallowly.

Read enough of the surrounding code (not just the diff hunks) to explain how the changes fit the existing system. A diff alone hides context the human needs.

## Step 2 — Lead with the "what and why" (not the code)

Open with a short, plain-language briefing the user can absorb in ~30 seconds:

- **What changed** — in terms of behavior and intent, not file names. ("Login now rate-limits by IP" beats "edited `auth.ts` and `middleware.ts`".)
- **Why** — the problem it solves or the goal it serves. If you genuinely don't know the intent, say so and offer your best inference, flagged as such.
- **Blast radius** — what this touches and what could break. Call out anything surprising, risky, or that deviates from existing patterns.

Then **stop and let them steer.** Do not dump the full tour unprompted. Offer the next layers:

> "Want the architecture map, an annotated code tour, or should I zoom in on one part?"

This is an *interactive walkthrough* — progressive disclosure. Go one layer deeper only where the user wants it. Continuously check understanding ("does that match what you expected?") rather than monologuing.

## Step 3 — Draw the map (when structure matters)

When the change spans multiple files or alters control/data flow, show structure visually. Prefer a diagram over prose for "how do these pieces connect".

- Use **Mermaid** for relationships and flows when the user can render it:

  ```mermaid
  flowchart LR
    Request --> Middleware[rate-limit middleware *new*]
    Middleware --> Handler
    Middleware --> Store[(IP counter store *new*)]
  ```

- Use **ASCII** for quick sketches in a plain terminal.
- **Mark what's new vs. pre-existing** in the diagram so the change stands out from the existing system.
- Keep diagrams small and focused on the changed area. A 40-node diagram of the whole app helps no one.

## Step 4 — Give an annotated code tour

When the user wants to read the actual code, guide them through it in a deliberate **reading order** — usually entry point → core logic → edges — not file-alphabetical.

For each stop:
- Cite the precise location as `path/to/file.ts:42` so it's clickable.
- Say *why this spot matters* and what to notice, not what the code literally says (they can read).
- Flag the non-obvious: a subtle decision, an assumption, a tradeoff, a place they'd likely want to change later, or something you're unsure about.

Example:

> 1. `src/middleware/rateLimit.ts:18` — the entry point; every request passes through here. Note the fail-open behavior on store errors (line 31) — a deliberate choice so an outage doesn't lock everyone out. Worth confirming that matches your security stance.
> 2. `src/store/ipCounter.ts:7` — in-memory store. **This won't survive a restart or scale across instances** — fine for now, a gap if you deploy multiple replicas.

## When you're done

You've succeeded when the human could now modify or review this code themselves — not when you've produced a tidy summary. Close by confirming that, and offer to go deeper on whatever's still fuzzy.

Keep these front of mind throughout:
- **Explain why, not just what** — intent and tradeoffs are the parts that don't survive in the code.
- **Admit uncertainty** — "here's my best guess" beats a confident fabrication. Never invent a rationale.
- **Surface risks unasked** — untested paths, scaling limits, half-done work, security-relevant choices.
- **End with a real check** — confirm they could own the code now; don't assume.
