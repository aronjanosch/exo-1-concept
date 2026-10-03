# Learnings

Loose list of what we learned while working, for humans and agents. Source material for future skills. One entry per point: what, why, how to apply. Newest at the bottom of each section.

## Working with the initiator

- **The initiator decides game design.** Record only what they explicitly decided, in their words. Brainstorming, my own conclusions and technical findings are not decisions. Why: AI choices rest on assumptions, not on play experience. How: before writing to DECISIONS/CORE-LOOP/VISION, quote the exact decision back.
- **AI guesses turn into "decisions" by copying.** Example: "start radius 3 km" came from an AI research draft, moved into DECISIONS as "decided", then into the spike brief; nobody had chosen it. How: tag every number with its origin (decided, assumption, measured) and keep the tag when copying.
- **Design-relevant defaults in prototypes are assumptions.** Speeds, gravity, flight assists, look: label them as such in the spike README, don't present them as settled.
- **Everything is early and experimental.** A change the initiator tries is an experiment, not a rule. Do not write it into DECISIONS or other docs as settled; at most note it as an idea or direction to try.
- **Be brief.** Say a fact once, no repeated lists, no long recaps.
- **Spikes: progress over low-spec work.** If it runs well on the initiator's machine, move on; optimise later.

## Local environment and tests

- **Never steal focus.** Check `WORKSPACE.md` in the code repo before running Godot. Prefer `--headless` for checks; for windowed runs use the contributor's wrapper (here `GODOT_AGENT_WORKSPACE=7 godot-agent`).
- **A window on a hidden workspace is throttled** (about 8 FPS). Frame times from such runs are worthless.
- **Godot releases all pressed keys when its window loses focus.** Scripted input via `Input.parse_input_event` then stops silently. Tests hold keys through their own input layer (`SpikeInput`), which is also closer to the proposed AgentBridge.
- **Screenshot readback plus PNG save costs 50-140 ms.** Exclude those frames from frame-time stats.
- **Bots must fly like players.** A blind bot at boost speed crashed into a hill and broke the run; give test bots simple controllers (altitude hold).

## Godot and planet tech (spike 1)

- Derivative flat normals (`cross(dFdx, dFdy)`) are zero on sub-pixel triangles; `normalize()` gives NaN and it survives `mix(..., 0)`. Guard the length.
- Skirts must use the smooth normal, or they show as dark lines.
- Per-chunk origins cause hairline cracks between chunks; skirts fill them.
- Jolt `HeightMapShape3D`: square maps (at least 4 samples per side) use the native height field; non-square ones fall back to a mesh (4.7.2 source).
- Tangent-frame collision patches with curvature baked into the heights avoid the flat-plane error; let them overlap.
- Distance tests for the collision ring must ignore terrain amplitude (compare on the base sphere), or the ring grows about 5x.
- Depth: Forward+ uses reverse-Z with a float buffer (no z-fighting to 40 km); Compatibility behaves like a classic 24-bit buffer (z-fighting from about 500 m at cm gaps).
- Precision: no physics jitter standing still up to 16 km from the origin; walking breaks at 16 km (likely patch offsets), fine at 8 km.
- On a small planet "straight" flight leaves the planet in seconds. Horizon follow: add angular rate `up x v / r`. A levelling force fights intended climbs.
- Ring/LOD bursts on teleport cause 60-140 ms frames; continuous movement does not.
