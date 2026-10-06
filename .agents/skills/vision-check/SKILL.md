---
name: vision-check
description: Use on a spec, proposal, issue or diff to check whether it fits the EXO-1 vision, frame and non-goals. Use before filing a proposal and as part of review. Labels and explains, never approves.
---

# vision-check

Read `docs/VISION.md` (and `docs/ROADMAP.md` if relevant) fresh each time. Do not rely on memory of it.

## Check

| Area | Question |
|---|---|
| Spirit | Does it add ideas, creativity or needed structure, or is it just slop or filler? |
| Originality | Anything taken from another game: names, assets, mechanics 1:1? |
| Frame | Godot, MIT-compatible, original or CC0 assets, aims for the best runtime performance with a simple look, gameplay over graphics? |
| Tone | Strange, goofy galaxy? |
| Scope | Small and within the current core loop and pillars? Does it expand scope before the baseline is solid? |
| Multiplayer | Fits small host-authoritative co-op? No cross-server assumptions? |
| Content | Data validated by schema, no executable code in content, no runtime-loaded code? |
| Core | Does it change the core loop or pillars? Then it is the initiator's call, flag it. |
| Risk class | Which paths does it touch: green, yellow or red? |
| Testability | Reachable via the bot/agent interface? |

## Output

```
Fit: fits | fits with changes | does not fit
Risk class: green | yellow | red
Findings: short bullets, each with the vision line it refers to
Suggested smaller or different shape: only if it does not fit
```

You label and advise. The community vote and human review decide. If something is a judgment call about taste, say so and leave it to the humans.
