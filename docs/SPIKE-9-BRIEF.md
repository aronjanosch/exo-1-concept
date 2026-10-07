# Spike 9 brief — Bevy validation (handoff for a new session)

Status: ready to start. Written 2026-10-07. Follows the decision to go fully Rust with Bevy once a validation spike passes (`DECISIONS.md`, "Engine", 2026-10-07: "einen spike test davor um das zu validieren dass alles genau so gut oder besser klappt").

## Goal

Rebuild what spikes 1, 3, 5, 7 and 8 showed in Godot, now in pure Rust with Bevy, and compare. The answer per area is **as good, better or worse than Godot**, with numbers. The initiator's two criteria: physics on par or better, and the framework fits agentic coding better. The result is a report, not a decision: the initiator commits after reading it.

## Known so far (Godot reference values, all measured on the dev machine unless noted)

| Area | Godot result | Source |
|---|---|---|
| Planet, terrain LOD | Seamless cube-sphere, skirts, chunk 2.2-2.7 ms on workers, worst frame 8-29 ms | `SPIKE-1-REPORT.md` |
| Generator | `planet_core` (pure Rust) drives mesh and collision; mesh vs height 0.55 mm, seams 0.086 mm, collision 2.3 mm with float32 body origin (cloud, no GPU) | `SPIKE-8-REPORT.md` |
| Walker | Radial gravity, 0 fall-throughs; `is_on_floor()` false in 6-37 % of walk frames | `SPIKE-1-REPORT.md`, `SPIKE-8-REPORT.md` |
| Ship | Jolt `RigidBody3D`, board, fly to space and back, land; assisted flight with 51 controller checks | `SPIKE-1-REPORT.md`, `LEARNINGS.md` (assisted-flight) |
| Walking in a flying ship | Drift 6-12 mm at 340-400 m/s while rolling; ramp boarding unreliable | `SPIKE-3-REPORT.md` |
| Large worlds | Physics fine to 197 km; origin shift < 0.4 ms, 0 mm drift, keeps velocity | `SPIKE-5-REPORT.md` |
| Builds | Linux native, Windows cross (`x86_64-pc-windows-gnullvm`, llvm-mingw), CI cold 6 min / warm 2 min 42 s, needs 1.2 GB export templates | `SPIKE-7-REPORT.md` |
| Agent friction | Focus stealing, key release on focus loss, wrapper needed, headless exit core dumps, worker-thread cleanup crash, transform flush quirk, godot-rust rebuild 34 s with LTO | `LEARNINGS.md` |

Networking (spike 4) is **not** in this spike; it is spike 10.

## Read first

1. `LEARNINGS.md` (environment, git, spike 8 section).
2. `SPIKE-8-REPORT.md` and `spikes/planet_gen/REPORT.md` on the spike 8 branch.
3. The Godot ship and walker code on `spike/planet-gen` (`spikes/planet/`), for behaviour to port, not to copy line by line.
4. Code repo `WORKSPACE.md` (windowed runs, no focus stealing).

## Where to work

- Code repo, new branch `spike/bevy` from `origin/spike/planet-gen` (commit `0bfd853`, contains `planet_core` and the assisted-flight code), own worktree `~/Work/exo-1-spike9`. Freeze tag at the end: `spike/9-bevy`.
- New Cargo workspace in `spikes/bevy/`; use `planet_core` by path, do not fork it. Changes to `planet_core` only if Bevy needs them, listed in the report.
- Commit small and often. Push only after the initiator says yes.

## Architecture rule

Simulation logic lives in crates without Bevy types (pattern from faith-runner, `World` trait with `sweep`/`overlaps`): `planet_core` exists; add `flight_core` (ship controller, assisted flight) and, if it fits, `walker_core`. Bevy does rendering, input, camera, HUD, physics glue. Use `f64` in the core crates.

## Steps

1. **Setup.** Latest stable Bevy, pinned. Physics: Avian (with `f64` if available) as the first choice; Rapier only if Avian blocks something, reason in the report (choice is an assumption). Floating origin: try `big_space` first, own origin shift as fallback. Check licences of every dependency (no LGPL or GPL).
2. **Planet.** Terrain chunks with LOD from `planet_core`, collision near the player, water sphere, simple shading. Check seams and mesh-vs-collision agreement like spike 8 T1/T2.
3. **Walker.** Radial gravity, walk anywhere, slopes stop the walker (spike 8 T5 values). Measure fall-throughs and floor contact ratio.
4. **Ship.** Rigid body, board and leave, take off, fly to space and back, land. Port the assisted-flight controller into `flight_core` and port its checks as `cargo test` (no window).
5. **Walking in a flying ship.** Spike 3 test: stand and walk in the cabin at up to 400 m/s while rolling; also the ramp.
6. **Origin shift.** Spike 5 tests: precision and physics at 10, 60, 200 km, cost per shift.
7. **Agent loop.** Headless test suite via `cargo test`; scripted input run that writes PNG screenshots without taking focus (offscreen render target if possible). Measure incremental rebuild time (with and without Bevy `dynamic_linking`).
8. **Builds.** Linux native and Windows cross build with the spike 7 toolchain; Windows build started once under Proton. CI only after the initiator says yes to a push.

Each step that fails stops only that step: write down the blocker and continue with the next.

## What to measure

- Per area: as good / better / worse than the Godot value in the table, with the number.
- Frame time (worst and mean) on the dev machine in a windowed run on the agent workspace, never on a hidden workspace.
- Incremental rebuild time after a one-line change in a core crate and in the Bevy crate; clean build time.
- Binary size per platform.
- Count of workarounds needed, compared with the Godot list above.
- Lines of code for the Bevy glue versus the core crates (rough).

Tag every number as measured, calculated or assumed.

## Not in scope

- Networking (spike 4: client authority, snapshots). That is spike 10, after this one (initiator, 2026-10-07: "Netzwerk als Spike 10").
- Game design: speeds, gravity, look, controls stay the test values from the Godot spikes, labelled as such.
- Any data or files from other games.

## Results

`SPIKE-9-REPORT.md` in this repo, learnings in `LEARNINGS.md`. One line at the top: does Bevy reach Godot's level in every area, and where not.
