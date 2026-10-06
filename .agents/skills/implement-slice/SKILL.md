---
name: implement-slice
description: Use to implement one small, already agreed slice of a feature on a feature branch, after the proposal is approved. Not for coarse requests, run scope-gate first.
---

# implement-slice

## Preconditions (verify, do not assume)

- An approved proposal or issue exists and the slice matches it. If not: stop and go back to `scope-gate` or `feature-breakdown`.
- The slice is one PR in size. If it grows, stop and split.
- Which risk class (`AGENTS.md`) do the touched paths fall in? Red means stop and flag for the initiator.

## Steps

1. Read the spec and design note. Restate the slice and its acceptance check in two lines.
2. Branch from main, one feature per branch.
3. Build the smallest thing that plays. Prefer data over code, small scenes over big ones, existing systems over new ones.
4. Expose the behavior through the bot/agent interface so CI can play it. Add or extend a bot playthrough for the acceptance check.
5. Add tests, run lint, format and a headless boot locally if available.
6. Keep placeholder assets simple, original or CC0.
7. Run `vision-check`, then `exo-review` on your own diff and fix what is real.
8. Open one PR: link the proposal, state the acceptance check and how it was verified, list known gaps. No claims you did not verify.

## Do not

- Add scope that was not agreed, even if it seems nice
- Touch unrelated files or red-class paths
- Use risky APIs (list in `AGENTS.md`) without asking
- Embed scripts in `.tscn` or `.tres` content
- Invent design decisions. Ask the contributor.
