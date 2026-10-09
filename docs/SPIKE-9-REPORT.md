# Spike 9 report — Bevy validation

Date: 2026-10-07. Brief: `archive/SPIKE-9-BRIEF.md`. Code: branch `spike/bevy` in worktree `~/Work/exo-1-spike9` (from `origin/spike/planet-gen`, `0bfd853`), `spikes/bevy/`, local only, freeze tag `spike/9-bevy`. Raw outputs: `spikes/bevy/results/`. Machine: Ryzen 7 5800X (16 threads), RTX 5070 Ti, 144 Hz monitor, Hyprland. Tags: **measured**, **calculated**, **assumed**.

## Answer in one line

Bevy reaches or beats Godot's level in every area of the table; the only area where it is not clearly ahead is builds (larger, slower-to-compile binaries in exchange for no export templates). Physics, walking in the ship, large worlds and the agent loop are better, with one Avian quirk worked around and one open point (single 50–97 ms frames when the collision ring is rebuilt in a burst, partly inside Avian).

## Stack (assumptions where marked)

- Bevy **0.19.1** (latest stable; 0.20 is a release candidate), pinned with `=`.
- Physics: **Avian 0.7.0 with `f64`/`parry-f64`** (first choice of the brief, an assumption; Rapier not needed, nothing blocked).
- Floating origin: **own render origin**. `big_space` 0.12 (latest) supports only Bevy 0.18, Avian 0.7 needs 0.19, so the brief's fallback applies. Physics runs in f64 world space and never shifts; only the f32 `Transform`s the GPU sees are relative to a render origin that jumps in whole metres (83 lines).
- Architecture as in the brief: `planet_core` (unchanged, **no change was needed**), `flight_core` (277 lines, port of `ship.gd`), `walker_core` (179 lines, own move-and-slide over a `World` trait with `sweep` and `depenetrate`), all f64 and without Bevy types. Bevy glue `exo_app`: 1282 lines plus a 700-line scenario bot. The ported Godot files were about 2070 lines of GDScript (measured, non-blank, non-comment).
- Licences: 425 crates, all permissive (MIT/Apache-2.0 in most cases; also BSD, Zlib, ISC, MIT-0, CC0, Unlicense, 0BSD and Unicode-3.0 for the ICU data). Nothing copyleft. Bevy, Avian, glam: MIT OR Apache-2.0; fastnoise-lite: MIT.

## Verdicts against the Godot table

| Area | Godot | Bevy (measured unless noted) | Verdict |
|---|---|---|---|
| Planet, terrain LOD | Chunk 2.2–2.7 ms on workers, worst frame 8–29 ms | Same `planet_core` chunks, build max 1.2–1.5 ms on Bevy's async pool. Release build, scripted full run at 144 Hz: every phase mean 6.94 ms (vsync floor), p99 7.3–7.5 ms, max ≤ 13 ms, except one 85 ms frame while landing (see open) | **Same or better** in flight and walking; ring bursts still spike, as in Godot (60–140 ms) |
| Generator | Mesh vs height 0.55 mm, seams 0.086 mm, collision 2.3 mm with float32 body origin | `planet_core` tests unchanged and green (T1 core, T2 seams 0.086 mm, T3). Collision vs `height_at` at all 201,598 interior patch samples: **0.238 mm**, the body origin is f64. 2 rays of 201,600 slipped through at a vertex | **Better** (the 2.3 mm engine limit is gone) |
| Walker | 0 fall-throughs; `is_on_floor()` false in 6–37 % of walk frames | 0 rescues in every run (all scenarios, 10/60/200 km, Proton). On walkable ground grounded **100 %** (run and 1.8 m/s walks); standing 0.0000 mm drift. Spike 8 T5: escarpment stops the walker after 317 m (Godot 313 m), plateau after 372 m (Godot 354 m); while it pushes against a too-steep slope it counts as grounded in 59–68 % of ticks | **Better** |
| Ship | Jolt body, board, to space and back, land; 51 controller checks | Avian dynamic body; ramp boarding, take-off, to 7000 m (field 0), back, landing (0.005 m/s, 0.12 m above ground), walk out and back in: all PASS. The 51 checks pass as `cargo test` without a window in 0.24 s after an edit. Firm stop from 350 m/s 2.77 s (Godot 2.75 s), gentle stop 7.28 s (7.15 s) | **Better** (same behaviour, much faster check loop) |
| Walking in a flying ship | Drift 6–12 mm at 340–400 m/s rolling; ramp unreliable | Stand and walk in the cabin, 400 m/s, rolling, assist off: drift **0.0000 m**, height in cabin constant (0.3100 m), never left the ship. Same at 280 m/s in atmosphere. Ramp up to the seat: PASS in every full run (twice per run) | **Better**, one workaround (below) |
| Large worlds | Physics fine to 197 km; shift < 0.4 ms, 0 mm drift, keeps velocity | Physics in f64: 0 rescues, 0.0000 mm standing drift at 8, 57 and 197 km from the origin. Shift ≤ 0.010 ms (stress threshold 10 m at 197 km), 0 mm jump and no velocity change by construction (bodies never move). GPU error 2 m ahead (calculated): without shift 0.21 / 1.73 / 4.06 px at 8 / 57 / 197 km, with shift 0.005 px | **Better** (no timing rule, no shift of physics at all) |
| Builds | Linux native, Windows cross with llvm-mingw, CI 6 min / 2 min 42 s, 1.2 GB export templates | Same llvm-mingw toolchain, **no changes needed** for Windows; only `libunwind.dll` next to the exe. Release Linux 297 s, Windows 286 s. Sizes: Linux 128 MB (87 MB stripped), Windows 117 MB (all Bevy default features, nothing trimmed; Godot export 74/109 MB). Windows build under Proton Experimental: headless run identical results, windowed run 6.94 ms mean, p99 17.3 ms, 0 rescues. CI not built (comes with the first approved push) | **Same**: no templates, but longer release builds and larger binaries |
| Agent friction | Focus stealing (wrapper), key release on focus loss, headless exit core dumps, worker-thread cleanup crash, transform flush quirk, godot-rust rebuild 34 s | See next section | **Better** |

## Agent loop (measured)

- Clean dev build of Bevy + Avian: 6 min 08 s (once per machine/cache).
- One-line change, rebuild of the game binary: 5.6–6.8 s statically linked (core crate or Bevy crate alike), **1.1 s** with Bevy `dynamic_linking` (feature `dynamic`). Godot-rust: 34 s.
- Core crate edit plus its tests: 0.24 s. Whole workspace test suite (51 flight checks, 6 walker tests, T1 collision, full scenario headless): 11 s after a build. The full scenario (249 s of game time) runs headless in about 10 s.
- Headless mode: one physics tick per update, no GPU, no window; runs exit with code 0, or non-zero when a check fails. **No exit crashes in any run.**
- Screenshots: `--hidden` opens an invisible window that still renders; screenshots work without any visible window and without focus. Frame-time measurements still need a visible window (wrapper `run-agent.sh`, workspace 7, no focus).
- Input: scripts drive the same `Controls` resource as the keyboard; no key-release problem by design.

Workarounds (compare with the Godot list):

1. **Avian moves child colliders only at the start of the next physics step** (`update_child_collider_position` in `PhysicsStepSystems::First`). Between steps, spatial queries see the cabin floor one tick behind the ship body, 6.7 m at 400 m/s; the walker fell through the floor at speed. Fix: the walker queries in the frame derived from the floor collider's own pose. Same class as Godot's transform flush.
2. **Avian's own `Position` ↔ `Transform` sync switched off** for the render origin; bodies are drawn from interpolated f64 poses (own code, not a workaround for a bug).
3. Wrapper for visible-window runs (as with Godot), not needed for screenshots.

Godot problems that did not appear: focus stealing in scripted runs, key release, headless exit crashes, worker cleanup crashes, 34 s rebuilds.

## Bugs found on the way (fixed, in the commits)

- `walker_core`: the floor-snap test compared a velocity with exactly 0, and the curvature left about 1e-17 m/s: grounded only 70–75 % while walking. Tolerance 1e-6.
- `walker_core`: in the air against a steep slope, sliding along the surface lifted the walker 3 cm per tick; it climbed 57° slopes. Steep surfaces now never slide upwards (unit test added).
- Avian depenetration with the same skin as the walker's own gap pushed it along slope normals every tick: 35 mm drift in 5 s while standing. Depenetration skin 2 mm, walker gap 10 mm: 0.0000 mm.
- Commit `227fc4f` does not compile (a debug line left over), fixed in the next commit.

## Assumptions

- Physics engine choice (Avian), own walker controller instead of Avian's built-in `MoveAndSlide` (the brief's architecture rule; only its depenetration is used), ship mass 2000 kg with box inertia and centre of mass (0, 1.4, −0.3), Avian default friction, physics interpolation for rendering, depenetration skin 2 mm, heightfield patches 32 × 32 at 1 m (as in Godot).
- Scenario bot: pointing the nose with a proportional mouse controller plus Q/E roll levelling; the cabin test stops boosting at 400 m/s; descent at −80° from 10 km (at −45° the bot missed the planet: from 15 km out it only covers ±19° around straight down).
- All game values (speeds, gravity, assists, look, controls) are the Godot spike test values, not designed.
- Look: simple `StandardMaterial` with biome vertex colours, one sun, no shadows, distance fog. Judged from 4 screenshots only.

## Open

- **Long single frames during collision-ring bursts** (50–97 ms, at startup and when the ship comes within 100 m of the ground; one per landing in every run). Ring system ≤ 0.15 ms and terrain ≤ 0.66 ms per frame; Avian's step took up to 76 ms in one startup frame and 20 ms in one landing frame while many heightfields were added. The rest of some of those frames is not located (rendering suspected, not checked).
- Manual play with keyboard and mouse was not tested by me (only scripted input); feel is for the initiator.
- Binary size: nothing trimmed (Bevy features, `strip`, `panic = "abort"`).
- CI with the first approved push; spike 10 (networking) and spike 9b (agent tooling) follow.
- No video clip: Bevy has no built-in movie maker (spike 9b could look at that).

## Local check (initiator)

1. `cargo run -p exo_app` in `spikes/bevy` (dev build), walk, board over the ramp, fly (H, L, X), orbit (O).
2. `cargo test --workspace` (about 11 s after the build).
3. Start the Windows build once yourself if wanted (Proton is enough).
