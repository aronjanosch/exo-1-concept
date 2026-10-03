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
- **Scripted runs must not capture the mouse.** `Input.mouse_mode = CAPTURED` pulls pointer and focus to the game window even when the wrapper starts it unfocused on another workspace. Spike 5: no capture with `--auto-test`/`--auto-shot`.
- **Two agent sessions in one checkout collide.** A parallel session switched the shared working tree to its branch mid-edit. How: each session works in its own `git worktree`. Projects with the same name also share `user://`, so give result files and screenshots their own names.

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

## Ships and walking inside (spike 3)

- Walker as a child of the ship, velocity relative to the ship, ship ignoring the walker's layer: holds up to about 400 m/s with roll (mm drift).
- Do not reparent bodies from `Area3D` signal handlers: reparenting re-fires the signals and a deferred call from there crashed Godot 4.7.2. Use a box test instead.
- Thin slabs (ramps) over uneven terrain let a capsule slip underneath; boarding on slopes needs its own design and test session.
- Stay on the essentials of a spike; when one detail (here the ramp) eats time, note it as open and move on.

## Float precision and origin shift (spike 5)

- **Check the obvious collider before blaming precision.** The 16 km "precision wall" from spike 1 was the parked ship standing in the walk path. How: log slide collisions (collider, normal, contact height) before drawing conclusions.
- **A safety net can hide the test.** The CPU height fallback kept the walker on the ground while the collision ring had silently built no patches. How: count frames where the net is the only thing holding the body.
- **Quadtree bounds on a cube-sphere: use the farthest corner.** `0.75 x edge` is too small for large cells on the curved face and pruned whole quadrants.
- **Godot sends moved bodies to physics only at a transform flush.** There is no flush between two `_physics_process` calls of the same tick. An origin shift in `_physics_process` let `move_and_slide` run against stale collision (falls through the floor). How: shift in `_process`.
- **Origin shift breaks silently wherever world and planet coordinates mix.** Example: `density_at` got a planet position and subtracted the centre again; the sky turned black only once the planet left the origin. How: one conversion function, type or name every position as world or planet-relative, and test with the planet away from the origin.
- **Physics survives far from the origin, the picture does not.** Jolt walking and landing worked up to 197 km from the origin; the error is in the float32 `view * model` on the GPU (about 2.5 px at 100 km for something 2 m from the camera). Measure in pixels, not only in physics.
- **Shift by whole metres and track the total in GDScript floats (64 bit).** Subtracting whole metres is exact here (0 mm jump in all runs), and the double total keeps the true position.

