---
name: feature-breakdown
description: Use after scope-gate when a contributor wants to work out an idea. Guides a design conversation that breaks the idea down, then produces a spec, a design note and small issues in the contributor's fork or area. Works for any feature, content or system.
---

# feature-breakdown

Goal: the contributor thinks the idea through with you, and leaves with a spec and issues they own. You interview, structure and challenge. You do not decide.

## Process

Ask one or two questions at a time. Wait for answers. Reflect back in short form before moving on. Skip questions that are already answered or irrelevant.

### 1. Player experience
- What does the player do, see, and feel in the first minute?
- Why is this fun, or goofy, in a way that fits the strange galaxy?
- What is the smallest version that already proves it?

### 2. Design questions (ask only the relevant ones)
- Where does it live in the world, and how does the player get there?
- Which existing systems does it use? Which missing systems does it need? Is each of those its own proposal?
- Who is in it: NPCs, factions, other players? What do they want?
- Quests, goals, progression, rewards, failure?
- Co-op: what changes with 2-5 players? What does the host decide?
- What is authored, what is generated?
- Which parts are data (content) and which need code?

### 3. Cuts
- What do we explicitly not do in the first version?
- What is the proof-of-concept slice, and what comes after?

### 4. Testability
- How could a bot play it through the agent interface and show it works?
- Runtime performance risks?

## Outputs

Write into the contributor's fork or area, not into shared core files. Suggested layout (adapt to repo conventions):

- `proposals/<slug>/SPEC.md`: problem, player experience, scope, non-goals, open questions, acceptance (bot-checkable)
- `proposals/<slug>/DESIGN.md`: answers from the conversation, decisions marked as the contributor's, assumptions marked as such
- Issues: small, each one PR-sized, each with a one-line acceptance check. Order them, the first is the proof-of-concept slice.

Use the contributor's wording for design decisions. List unresolved questions explicitly instead of resolving them yourself.

## Finish

Offer to run `vision-check`, then help prepare the proposal for the `community-gate` flow. Do not start implementing.
