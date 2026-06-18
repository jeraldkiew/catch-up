# catch-up

A [Claude Code](https://claude.com/claude-code) skill that helps a human engineer rebuild their mental model of code that was just written or changed — usually by Claude — so they stay in control of code they didn't type.

As more code gets generated than you can absorb, you take on *cognitive debt* — a widening gap between what the code does and what you actually understand. This skill helps you pay it down: it turns "I lost track of what you did" into a guided, interactive walkthrough.

## What it does

When you ask Claude to catch you up, it:

1. **Finds the real change set** — reads the actual `git diff` (working tree, staged, or a commit/branch range) instead of trusting conversation memory, and reads the surrounding code for context.
2. **Leads with the *what and why*** — a ~30-second briefing in terms of behavior and intent, plus the blast radius and any risks — then stops and lets you steer how deep to go.
3. **Draws the map** — Mermaid or ASCII diagrams that mark what's *new* vs. *pre-existing*, when structure or data flow changed.
4. **Gives an annotated code tour** — a deliberate reading order with clickable `file:line` references, focused on the non-obvious: decisions, assumptions, tradeoffs, and places you'd likely change later.

It's built to keep you in control: it explains *why* (the part that doesn't survive in the code), is honest about uncertainty instead of inventing rationale, and treats you as a peer rather than a beginner.

## Install

Clone into your personal Claude skills directory:

```bash
git clone https://github.com/jeraldkiew/catch-up.git ~/.claude/skills/catch-up
```

Then reload skills (`/reload-plugins`) or restart your Claude Code session.

## Usage

Say any of:

- "Catch me up on what changed"
- "Walk me through what you did"
- "Explain these changes"
- "I lost track — what's going on in this branch?"

Or invoke it directly with `/catch-up`.

A natural flow: after Claude makes a batch of edits, ask it to catch you up before you commit, review, or build on the work.

## License

MIT — see [LICENSE](LICENSE).
