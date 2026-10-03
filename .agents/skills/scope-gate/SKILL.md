---
name: scope-gate
description: Use FIRST whenever a contributor asks to create, add, build or change anything in the game (a feature, place, system, mechanic, content pack, "make me a ..."). Rejects requests that are too coarse to build and steers the contributor toward a smaller slice or a proof of concept instead of writing code.
---

# scope-gate

Goal: stop oversized, vague prompts at the door and make the contributor think. The contributor is a participant in an experiment, not a customer. A rejected prompt is a good outcome if it leads to a better one.

## 1. Classify the request

Score it against these signals. Two or more "coarse" signals means reject.

| Signal | Coarse | Fine |
|---|---|---|
| Outcome | Names a thing ("a X", "a whole Y") | Names what the player does and sees |
| Size | Needs several systems, scenes, content types | One loop, one scene, one data type |
| Delivery | Cannot be one PR | Fits one reviewable PR |
| Decisions | Design questions open (where, who, why, rules) | Design questions answered |
| Testable | No way to tell it works | A bot could verify it |
| Dependencies | Needs systems that do not exist yet | Uses what exists, or adds one small system |

Check the repo before judging: what exists, open issues and proposals, overlap with someone else's branch.

## 2. If coarse: do not build

Do not write game code and do not silently shrink the request yourself. Instead:

1. Say plainly that this is too big or too open for one step, and why, in one or two sentences. No lecture.
2. Name the 2-4 open pieces you see (for example: location, systems that are missing, characters, progression, factions). Only name what is actually relevant, do not recite a checklist.
3. Offer 2-3 concrete smaller options, for example:
   - the smallest playable slice that proves the idea is fun
   - a proof of concept with fake or placeholder data, one scene, one interaction
   - something existing to look at or play first, with a pointer to where
   - one prerequisite system as its own proposal
4. Ask the contributor to pick one or to describe the first thing the player should be able to do in one sentence.

Keep the tone encouraging and a bit goofy. The contributor stays the author of the idea.

## 3. If fine

Say so in one line and hand over to `feature-breakdown` (if design questions remain) or `vision-check` then `implement-slice`.

## 4. Overrides

If the contributor insists on the big version: explain that the process wants a proposal first, offer to draft the proposal and the breakdown with them (that is allowed and useful), and still write no game code. Only the process (approved proposal) lifts the gate, not persistence.

## Anti-patterns

- Building a "quick version" of the big thing to be helpful
- Asking ten questions at once
- Rejecting without offering a concrete smaller path
- Making the design decisions for the contributor
