# Spike 6 brief — terrain generator: GDScript versus Rust (handoff for a new session)

Status: done, result in `SPIKE-6-REPORT.md`. Kept as the record of what was asked.

## Goal

Measure how much faster the planned terrain generator is in Rust than in GDScript, and whether the difference matters in play. The answer decides one of three paths, and the initiator picks:

1. GDScript is fast enough: stay with Godot and GDScript.
2. GDScript is too slow, Rust is enough: keep Godot, write the generator in Rust (GDExtension).
3. Rust wins by a large margin and that margin matters everywhere: reopen the Bevy question.

The rest of the engine (renderer, Jolt physics, `FastNoiseLite`) is C++ in Godot, so a language switch changes little there. The open risk is game code that runs per vertex in GDScript loops. The generator is the largest planned piece of such code.

## Known so far

- Spike 1 (`SPIKE-1-REPORT.md`): one chunk (32x32 quads, simple noise) builds in 2.2-2.7 ms on a worker thread, 6 ms max. Main thread at most about 5 ms for terrain per frame (mostly LOD traversal).
- `docs/research/procedural-planet.md` describes the planned pipeline at R = 6 km: macro shell, crust mesh, dressing. That is much more work per vertex than spike 1.
- Chunks are built on the `WorkerThreadPool`. A slow generator first shows as late or popping terrain, not as a lower frame rate.

## Read first

1. `LEARNINGS.md` (how to work with the initiator, local environment).
2. `docs/research/procedural-planet.md`, sections 7 and 8 (the pipeline to benchmark).
3. `SPIKE-1-REPORT.md`, then `spikes/planet/terrain.gd` (`_build_job`, `_make_noise`, `cube_to_sphere`) on `spike/combined` in the code repo.
4. Code repo `AGENTS.md` and `WORKSPACE.md`.

## Where to work

- Code repo `~/Work/exo-1`. New throwaway branch `spike/6-generator-bench` from `spike/combined`, in its own worktree (`~/Work/exo-1-spike6`). The main checkout is on another spike branch; do not switch it.
- Rust 1.98.1 and cargo are installed (`/usr/bin/cargo`).
- Commit and push only when the initiator asks.

## The workload (identical in every variant)

One fine chunk at R = 6000 m, about 64 m across, 32x32 quads (33x33 vertices plus skirts), fixed seed, fixed list of chunk ids (for example 200 chunks spread over all six faces).

Per vertex:

1. `cube_to_sphere` (same formula as spike 1).
2. Macro field lookup: bilinear sample from a baked cube-face image (start with 512² per face) holding elevation, temperature, moisture and a landform id. Bake the images once before timing; the bake is reported separately.
3. Noise stack from section 7 of the research note: region, face and foot bands (3D simplex, fractal, one ridged band, one domain-warped band).
4. Pick one of four biome rows from the fields; write a vertex colour.
5. Normal from the height neighbours.

Per chunk, after the surface exists (the "dressing"):

6. Scatter candidates on a jittered grid (canopy about 10 m, rocks about 6 m), filtered by slope, altitude, a forest mask noise and one site clear-radius. Output transforms only; nothing is rendered.

Output: vertex, normal and colour arrays plus the scatter transform list, plus a checksum (sum of heights, scatter count) so the variants can be compared.

The numbers above are test values for the benchmark, not design decisions.

## Variants

- **A: GDScript, straightforward.** Same style as `terrain.gd`: loops in GDScript, `FastNoiseLite.get_noise_3d` per sample.
- **A2: GDScript, optimised.** Same output, but every trick that stays in GDScript: packed arrays, fewer calls, caching, batch noise through `FastNoiseLite.get_image`/`get_seamless_image` where it fits. Shows how much is left in GDScript before changing language.
- **B: Rust inside Godot (GDExtension via godot-rust).** Same workload as a Rust function called from a Godot worker thread, returning packed arrays. GDExtension is a red-class addon under `AGENTS.md`; the initiator approved it for this spike (2026-10-06). Spike branch only, never merged into `main` without a separate decision.
- **C: plain Rust, outside any engine.** `cargo` binary or `cargo bench`, release profile. The generator code a Bevy game would run, without Godot. This is the upper bound for Rust.

Noise must match between variants so the checksums agree. Check whether a maintained Rust port of FastNoiseLite exists and what its licence is (MIT required; check the licence file, do not trust search snippets). If it does not match Godot's `FastNoiseLite` bit for bit, accept a small height difference and report it.

## What to measure

1. Time per chunk on one thread: mean, P95, max (A, A2, B, C). Warm up first; exclude the macro bake.
2. Throughput with all worker threads: chunks per second (A, A2, B in Godot's `WorkerThreadPool`; C with a thread pool of the same size).
3. Main-thread cost per chunk for B (call overhead, array conversion), compared with A.
4. Macro bake time per seed (once, at load) for A2 and C.
5. Demand: how many fine chunks per second the current LOD needs when walking (5 m/s), at ground cruise (45 m/s) and at a low fly-over at 350 m/s (values from `LEARNINGS.md`). Measure it from the LOD code, do not estimate. Compare with the throughput from 2.
6. Effort, as notes: lines of code per variant, how long the agent needed, problems with APIs or bindings, incremental compile time for B and C.

Report each number as measured, calculated or assumed. Dev machine only (low-spec hardware is no longer a target).

## Rules for this spike

- Benchmark only. No game-design decisions (biome names, sizes, densities, speeds); list them as open questions.
- No rendering, no LOD changes, no network, no Bevy app. C is plain Rust on purpose.
- Do not turn the result into an engine decision. The report lists the numbers and which of the three paths they point to; the initiator decides.
- Results go into `SPIKE-6-REPORT.md`, learnings into `LEARNINGS.md`.
