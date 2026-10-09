# Spike 5 report — float limit and origin shift

Date: 2026-10-04. Brief: `archive/SPIKE-5-BRIEF.md`. Code: `~/Work/exo-1`, branch `spike/origin-shift` (throwaway, from `spike/planet`, worked in the worktree `~/Work/exo-1-origin-shift`), options in `spikes/planet/README.md`.

Machine: RTX 5070 Ti, Compatibility renderer (OpenGL), 144 Hz window on workspace 7. Labels: **verified** (checked directly), **measured** (test runs), **calculated** (float32 maths on the CPU, same formula as the GPU), **assumed**.

## Result

- Float32 precision does **not** break physics in the tested range: walking, flying, landing work up to 197 km from the origin (measured, 0 fall-throughs). The limit is visual: the GPU computes `view * model` in float32, so things near the camera are drawn up to about one float step off. That passes 1 px (1080p, 75° FOV, object 2 m from the camera) at about 50-60 km from the origin and reaches about 2.5 px at 100 km and 8.5 px at 200 km (calculated).
- The 16 km "walking wall" from spike 1 was **not** precision: the walker ran into the parked ship (verified).
- An origin shift fixes it: at any tested distance the error stays below 0.15 px (calculated), one shift costs at most 0.4 ms, moves nothing relative to the planet (0 mm) and keeps the ship's velocity (measured). It must run in `_process`, not `_physics_process`.
- Several planets: both "re-centre on the nearest planet" and "continuous shift" work (measured with a second planet at 60 and 200 km). Continuous shift is the simpler rule; re-centre shifts less often.

## Questions from the brief

| # | Question | Answer | Evidence | Confidence |
|---|---|---|---|---|
| 1 | Planet at the origin: where do walking, landing, visuals break? Is 16 km really precision? | Physics does not break up to R = 64 km (and up to 197 km from the origin with an offset planet). Visual error near the camera passes 1 px at about 50-60 km from the origin. 16 km was the parked ship | Radius sweep 5-64 km, offset planets to 98 km, slide-collision log | High (physics measured, visuals calculated) |
| 2 | Flying out to 100 km: when does it jitter? | Ship in the chase camera: below 0.7 px at 100 km. Something 2 m from the camera: about 1.3 px at 60 km, 2.5-3 px at 100 km. Motion gets uneven: at 50-100 km each physics step of a ship drifting at 2 m/s is off by up to 2 mm from velocity × time (6 % of a 33 mm step); with origin shift at most 0.05 mm | Fly-out test 10/25/50/100 km | Medium (calculated, not judged by eye) |
| 3 | Origin shift: does it fix 1 and 2, what breaks, cost? | Fixes both. Broke: shift timing (falls through the floor if done in `_physics_process`) and every place that mixes world and planet positions (sky and fog in this spike). Physics bodies, collision patches, LOD, worker jobs: no problems found. Cost: shift function at most 0.4 ms; no frame over 33 ms in windowed runs with up to 658 shifts | Stress runs with shift thresholds 10 m, 50 m, 1 km | High |
| 4 | Several planets: re-centre per planet or continuous? Which is simpler? | Both work. Continuous is one rule and keeps errors near zero everywhere. Re-centre shifts once per planet change but leaves the ship up to half the planet distance from the origin in between (0.66 px at the ship at 100 km) and still needs the same shift code. Both need "which planet is current" for gravity anyway | Second planet at 60 and 200 km, 3 variants each | Medium-high |
| 5 | Multiplayer | Notes below; spike 4 tests it | — | Assumed |

## How it was measured

- **Planet centre as a variable.** The planet no longer has to sit at the origin; all code converts world positions to planet positions through one function (`to_planet`). Options: `--planet-offset`, `--origin-shift`, `--second-planet`, `--recenter`.
- **Visual error ("jitter probe").** Both renderers compute `modelview = view_matrix * model_matrix` in float32 on the GPU (`drivers/gles3/shaders/scene.glsl:718` and `scene_forward_clustered.glsl:440`, 4.7.2-stable; verified in the source). The probe repeats that product in float32 on the CPU each frame and compares it with the same product in double precision, for a point 2 m in front of the camera (nearest terrain chunk) and for the ship's nose. Reported in pixels at 1080 lines and 75° FOV. Calculated, not read back from the GPU; the worst frame per phase is reported, frame-to-frame jitter is at most about twice that.
- **Physics.** Walk 20 s at 12 m/s away from the ship; counts fall-throughs ("rescues") and frames without a collision patch under the walker. Standing still 5 s; largest frame-to-frame step relative to the planet.
- Headless runs for physics and the calculated error (both independent of the window), windowed runs for frame times. Frame times: 144 Hz vsync, so 6.94 ms is the floor.

## Measurements

### 1. Planet at the origin, radius sweep (headless, walk-only)

Every radius from 5 to 64 km: walked 216-246 m, 0 rescues, 0 frames without a collision patch, 0.0000 mm steps standing still.

Planet with R = 5 km moved away from the origin (diagonal offset, all three coordinates large):

| Distance of the walker from the origin | Walk | Error, point 2 m ahead (calculated) |
|---|---|---|
| 5 km (planet at the origin) | ok | 0.35 px (0.98 mm) |
| 13 km | ok | 0.21 px |
| 16 km (R = 16 km at the origin) | ok | 0.67 px (1.7 mm) |
| 29 km | ok | 0.54 px |
| 60 km | ok | 1.29 px (3.6 mm) |
| 64 km (R = 64 km at the origin) | ok | 1.77 px (6.6 mm) |
| 98 km | ok | 2.51 px (7.0 mm) |
| 197 km (second planet, no shift) | ok | 8.48 px (26 mm) |

These are the worst frames of a 20 s walk. The ship's hull seen from the chase camera (12 m) stays smaller: 0.27 px at 50 km, 0.64 px at 100 km.

Landing worked at every tested distance (0 rescues); see the note on the landing phase below.

### 2. Flying out (fly-out test, ship climbs to 10/25/50/100 km from the origin)

| Distance | Ship nose, chase camera | Step error at 2 m/s, no shift (shift 1 km) | Windowed frames |
|---|---|---|---|
| 10 km | 0.07 px | 0.49 mm (0.04 mm) | max 10.6 ms |
| 25 km | 0.12 px | 1.0 mm (0.05 mm) | max 9.8 ms |
| 50 km | 0.27 px | 2.0 mm (0.03 mm) | max 10.0 ms |
| 100 km | 0.64 px | 2.0 mm (0.004 mm) | max 19.4 ms |

Step error: largest difference between a physics step and velocity × time step, in world coordinates, measured on the combined branch (see below). A first version measured steps relative to the planet centre; that difference itself only has the float resolution of the distance (7.8 mm at 100 km) and wrongly showed 25/33 mm steps.

0 frames over 33 ms. The planet disappears beyond 50 km because the camera's far plane is 50 km (spike 1 value).

### 3. Origin shift

Shift rule in the spike: when the active body is farther than a threshold from the origin, subtract its position rounded to whole metres from every root node (planets' terrain and collision ring, walker, ship) and add it to a double-precision total.

| Run | Shifts | Rescues | Jump relative to planet | Ship speed change | Shift function | Error near camera |
|---|---|---|---|---|---|---|
| R = 5 km, threshold 1 km, full test | 8 | 0 | 0 mm | 0 | ≤ 0.11 ms | ≤ 0.03 px |
| R = 5 km, threshold 10 m (stress), full test | 658-763 | 0 | 0 mm | 0 | ≤ 0.4 ms | ≤ 0.01 px |
| R = 16 km, threshold 10 m | 654 | 0 | 0 mm | 0 | ≤ 0.29 ms | ≤ 0.03 px |
| Planet 98 km away, threshold 1 km | 8 | 0 | 0 mm | 0 | ≤ 0.16 ms | ≤ 0.12 px (2.5-4.4 px without) |
| Fly-out to 100 km, threshold 1 km | 95 | 0 | 0 mm | 0 | ≤ 0.08 ms | ship ≤ 0.00 px |

Windowed frame times (worst frame per run): R = 5 km without shift 18.8 ms, threshold 1 km 15.3 ms, threshold 10 m (658 shifts) 16.7 ms; planet 98 km away with threshold 1 km 16.7 ms; fly-out to 100 km with threshold 1 km (95 shifts) 10.9 ms; re-centre on a planet 60 km away (one 60 km shift) 6.94 ms in the transfer phase. No frame over 33 ms in these runs. The shift itself does not show in the frame times.

One later check run (planet 98 km away, threshold 1 km) had 3 frames of 33-39 ms in the landing phase. No shift happened in that phase and terrain and ring stayed below 3.1 ms per frame, so they did not come from the shift; the cause is unknown (another session was running Godot tests on the same machine at the time; not verified). An earlier fly-out run had one 42 ms frame at 63 km while the game still captured the mouse; it did not come back in the repeat.

What broke:

- **Timing (verified).** Shifting in `_physics_process` caused 1-2 rescues in 3 of 4 stress runs and short stalls of the walker. Cause: Godot sends moved bodies (the collision patches) to the physics server only when it flushes transform notifications, and there is no flush between two `_physics_process` calls of the same tick. The walker's `move_and_slide` then ran against the old patch positions. Shifting in `_process` (between ticks): 0 rescues in all 4 runs.
- **Mixed coordinates (verified).** Sky, fog and ambient light used a planet position where a world position was expected; the sky turned black on the ground as soon as the planet was away from the origin (screenshot). Fixed and checked by screenshot (blue sky on the ground with shift). This class of bug is the main cost of an origin shift: it appears only after a shift and nothing crashes.
- Not broken (measured): physics bodies (Jolt keeps the velocity of a moved rigid body), collision patches (moved with their parent), terrain LOD and chunk jobs on worker threads (both work in planet coordinates, so a job that finishes after a shift is still correct), the safety-net and ring logic.

### 4. Several planets

Second planet (R = 3 km) at 60 km and 200 km from the first. The ship takes off to 2000 m, a test autopilot flies it to 500 m above the near side of planet 2 (direct velocity, up to 3000 m/s; not a flight model), it lands with the normal controls, the walker gets out and walks 10 s. "Current planet" = nearest surface with 500 m hysteresis; gravity and atmosphere come only from it (test assumption, not designed).

| Variant | Distance | Shifts | On planet 2: error near camera | Ship in transfer | Rescues |
|---|---|---|---|---|---|
| No shift | 60 km | 0 | 1.9-2.0 px | 0.33 px | 0 |
| Re-centre | 60 km | 1 | 0.13 px | 0.15 px | 0 |
| Continuous 1 km | 60 km | 60 | ≤ 0.01 px | 0.01 px | 0 |
| No shift | 200 km | 0 | 6.7-8.5 px | 1.32 px | 0 |
| Re-centre | 200 km | 1 | 0.13 px | 0.66 px | 0 |
| Continuous 1 km | 200 km | 198 | ≤ 0.03 px | 0.01 px | 0 |

Code size of the two rules in the spike: continuous shift is 3 lines on top of the shared `shift_origin`; re-centre is 1 line on top of the "current planet" switch, which is about 40 lines but needed for gravity with several planets anyway.

### Checked by eye (initiator, 2026-10-04)

- Walker 98 km from the origin (`--planet-offset=-58000,-58000,-58000`), no shift: "der Jitter ist schon deutlich zu erkennen". With `--origin-shift=1000`: nothing visible any more, same as at the origin.
- Large planet (`--radius=64000`) with shift: works.

This matches the calculated errors (about 2.5 px without, below 0.15 px with shift).

## Combined with spike 3 (branch `spike/combined`)

Spike 3 (`spike/leave-ship`, walkable cabin, ramp, ship inertia) and spike 5 merged on a new branch `spike/combined`; both original branches are unchanged. Git reported two text conflicts (`main.gd`, `auto_test.gd`); the real interactions were elsewhere:

- **Double shift (fixed).** In the cabin the walker is a child of the ship. The shift moved ship and walker, so a seated walker would have moved twice. Now only nodes directly under the root are moved.
- **Planet at the origin assumed (fixed).** Spike 3's parking code and test code used world positions as planet positions in five places.
- **Ramp collision at the planet centre (spike 3 bug, fixed on `spike/combined` only).** The ramp is an `AnimatableBody3D`; with `sync_to_physics` on (default) it did not follow the ship and stayed where the ship was created, at the planet centre (verified on the unchanged spike 3 branch: ramp 4999.6 m below the ship). Boarding over the ramp failed in every spike 3 test run for this reason; the test fell back to placing the walker at the seat. With `sync_to_physics = false` boarding works in every run. It only became visible with the origin shift, because then the walker stands near the planet centre's old place and hits the lost ramp.
- **Hover assist off by default (spike 3).** The spike 5 test runs switch it on.

Results on `spike/combined` (headless): all spike 3 phases pass with origin shift (threshold 10 m and 1 km, planet at the origin and 98 km away): walk into the parked ship, walk out and back in, stand and walk in a ship flying 133-500 m/s; 0 rescues, walker drift in the cabin 0.000-0.002 m with shift (0.03-0.30 m without, planet at the origin or 98 km away). Spike 5 tests (fly-out, second planet) pass as before.

Video clip of the tumble: `~/Videos/exo-1-clips/2026-10-04-ship-tumbles-low-cruise.mp4` (20 s, movie maker at 30 fps, `spike/combined` at `d8efca0`, `--auto-test`, frames 2300-2900). In this run the ship rolls upside down in the low cruise and then lands. Second clip `~/Videos/exo-1-clips/2026-10-04-ship-lands-into-space.mp4` (33 s, 60 fps, `spike/combined` at `b905998`, `--auto-test --start-at=climb`, frames 2600-4600): the ship skims the ground upside down, then the landing phase holds "down" in ship space, which now points up, and it flies off into space. At fixed 60 fps this happens 3 of 3 times; starting the cruise from rest (`--start-at=cruise`) it never happens (12 headings, 30 and 60 fps): the roll needs the fast approach from the glide phase.

Not caused by the merge, seen on the unchanged spike 3 branch too: the low cruise bot gets down to 2 m above ground with the new inertia physics and sometimes tumbles; then the landing phase ends in space (1 of 2 runs on spike 3, also in some combined runs). Two headless runs crashed when quitting, after the results were written (core dump, not analysed).

## Multiplayer (note for spike 4, assumed)

- With a continuous shift every client has its own origin. Positions on the network must be in a shared frame, for example the true position in doubles (shifted total plus local position) or planet-relative positions (planet id plus local position). Godot's `MultiplayerSynchronizer` syncs raw node properties, so it would send per-client local coordinates; it needs a conversion layer.
- Physics: if the host simulates all ships, two ships near different planets cannot both be near the host's origin. Options: physics per planet in planet coordinates, or clients simulate their own ship (client authority). This touches the ship-sync rules in `FEASIBILITY.md`.
- Re-centre per planet is easier to share (the origin is a planet's centre, the same on every client), as long as players near the same planet use the same one.

## Notes

- Spike 1 code bug found and fixed: the collision ring's quadtree used `0.75 x edge` as the cell bound, too small for large cells on the curved cube face; whole quadrants were skipped and the walker walked only on the CPU safety net (0 patches) at some radii. Now the farthest corner. The terrain LOD has the same bound (`terrain.gd:252`), with 150 m of padding; it may split a little late near face centres. Not changed.
- The landing phase of the auto-test always ends by its time limit (40-60 s) with the ship resting 0.7-0.9 m above the ground (half the hull height), with and without shift, also in spike 1. The end condition "speed < 0.05 m/s while Ctrl is held" is never met. Test weakness, not precision.
- The probe and the shift jump use planet coordinates; on a 16 km planet those have about 1-2 mm resolution, so "max step 0.98 mm" for a landed ship in some shift runs is measurement resolution.
- Scripted runs captured the mouse (from spike 1) and pulled focus to the game window; now off in `--auto-test` and `--auto-shot`.
- `SPIKE-1-REPORT.md` still says the 16 km limit is likely precision. Not changed here; the correction is in this report.

## Open (for the initiator)

- Planet sizes and distances between planets: technically, physics allows much more than 8 km; with an origin shift the picture does too. Which sizes and distances the game wants is a design question.
- Travel between planets (speeds, how long, what happens on the way): open in `DECISIONS.md`; the transfer here is only a test autopilot.
- Gravity with several planets (nearest only, blended, spheres of influence) is a design question; the spike used "nearest only".
- Renderer choice (from spike 1) does not change this result: Forward+ in a single-precision build computes `view * model` the same way.
- Camera far plane (50 km) hides planets farther away; distant planets need another way to be drawn (for example a simple proxy), not tested.
