# Spike 6 report — terrain generator: GDScript versus Rust

Date: 2026-10-06. Dev machine only (16 logical cores, Godot 4.7.2 headless, Rust 1.98.1 release build with LTO). Code: branch `spike/6-generator-bench` in worktree `~/Work/exo-1-spike6` (from `spike/4-network`), `spikes/gen_bench/`. Nothing committed or pushed. Benchmark only: the engine decision is the initiator's.

Tags: **measured** (this spike), **calculated** (from measured numbers), **assumed**.

## Answer in one line

On this workload GDScript (variant A2) is fast enough by a wide margin. Rust is 5-7x faster per thread and about 7x faster in throughput, but the measured chunk demand is 15x or more below what A2 can deliver. Path 1 (stay with GDScript) is what the numbers point to. A Rust component is not needed for the generator now.

## Workload

Defined in `spikes/gen_bench/SPEC.md`: 200 chunks over six faces (35x35 grid with skirt ring, 7 noise calls per vertex, bilinear lookup in a baked 512² x 6 macro image, biome pick, normals, scatter of canopy and rocks on a jittered grid). All numbers in the spec are test values, not design decisions (assumed). Variants: A (GDScript, spike-1 style), A2 (GDScript, optimised), B (Rust in Godot via godot-rust), C (plain Rust). All four give the same checksum:

| | height sum (200 chunks, 3 reps counted once) | canopy | rocks | biome rows |
| --- | --- | --- | --- | --- |
| A | 4169609.113 | 3021 | 11549 | 107142 / 89594 / 20880 / 184 |
| A2 | 4169609.153 | 3021 | 11549 | same |
| B | 4169609.072 | 3021 | 11549 | same |
| C | 4169609.072 | 3021 | 11549 | same |

Noise parity (measured): `fastnoise-lite` 1.1.1 against Godot's `FastNoiseLite`, 1000 points, all 10 noise configs: max difference 0 (including ridged and OpenSimplex2S). A against A2 over 60 chunks: max vertex difference 1.2 mm (f32 versus f64 rounding), identical scatter and biome counts.

Licence (checked): the crate's `Cargo.toml` says MIT but the packaged crate has no licence file. Upstream `Auburn/FastNoiseLite` has a `LICENSE`: MIT, Copyright 2020 Jordan Peck and contributors.

## Numbers (measured)

Time per chunk on one thread, 600 samples (200 chunks x 3 reps) after 20 warm-up chunks, wall time:

| Variant | mean ms | P95 ms | max ms | chunks/s, 1 thread |
| --- | --- | --- | --- | --- |
| A | 4.37 | 4.83 | 5.87 | 221 |
| A2 | 3.32 | 3.73 | 4.47 | 288 |
| B (Rust in Godot) | 0.645 | 0.83 | 0.97 | 1465 |
| C (plain Rust) | 0.647 | 0.84 | 1.13 | 1559 |

Throughput with several threads (chunks/s, WorkerThreadPool for A, A2, B; `std::thread` for C):

| Threads | A | A2 | B | C |
| --- | --- | --- | --- | --- |
| 1 | 221 | 288 | 1465 | 1559 |
| 4 | 769 | 1019 | 5593 | 5807 |
| 8 | 1330 | 1753 | 9671 | 10671 |
| 16 | 263 | 369 | 12452 | 13844 |

The GDScript variants collapse above 8 threads: A2 gives 1524/s at 8, 586 at 10, 355 at 16 threads (separate run). Rust keeps scaling (SMT adds about 30 percent from 8 to 16). The cause of the GDScript drop was not investigated (contention in the runtime is likely, assumed). Use at most about 8 worker threads for GDScript generation.

Macro bake (once per seed): A 1.55 s single thread; A2 1.40 s single, 0.43 s with 4 threads (0.8 s with 8, 1.6 s with 16: same collapse); B 0.37 s single, 0.053 s with 16 threads; C 0.38 s single, 0.053 s with 16 threads.

Main-thread cost for B (measured): the call overhead (Dictionary and packed-array conversion) is 3.5 µs per chunk (wall minus time inside Rust). For A and A2 it is 8 µs per chunk, and there the build itself would stay on a worker thread, so the main-thread cost is the same upload as before.

Where A2's time goes (measured, phases of one chunk): vertex loop 2.36 ms, normals and skirt 0.39 ms, scatter 0.71 ms. The 7 noise calls alone are about 0.9 ms per chunk (micro test with similar inputs); the rest is interpreter overhead. This is the part Rust removes. A2 against A: 24 percent faster, same output. Tricks used: per-thread noise sets, scalar noise calls, inlined `cube_to_sphere` and macro lookup, normals only for inner vertices, in-place packed-array writes, threaded bake. `Noise.get_image` does not fit (2D, regular grid; the sphere needs 3D points).

## Demand from the current LOD (measured, `spikes/planet/terrain.gd`, R = 6000, split factor 1.5, 60 s each, headless at 60 fps)

Chunks per second built and uploaded, steady state (seconds 10-60), finest level in brackets; finest chunk edge about 54 m at depth 7, about 27 m at depth 8. One run per configuration, height above the base sphere, terrain amplitude ignored.

| Speed | depth 7 mean (peak) | depth 8 mean (peak) |
| --- | --- | --- |
| 5 m/s | 1.6 (16) [1.0] | 4.5 (16) [2.9] |
| 45 m/s | 16.9 (44) [9.4] | 43.0 (72) [26.1] |
| 350 m/s | 113.6 (144) [61.3] | 237.1 (244) [123.9] |

Start-up burst (first 10 s): up to 240 chunks/s. At 350 m/s with depth 8 the run is saturated by the uploads (4 per frame x 60 fps = 240/s), the queue grows to 356 and 16946 chunks were built against 14218 uploaded, so the true demand there is higher and was not determined. That limit is main-thread upload, not generation, and does not depend on the language.

## Comparison (calculated)

- Worst measured sustained demand that is not upload-bound: 114 chunks/s (depth 7, 350 m/s), peak 144. A2 on one thread does 288/s, on 4 threads about 1000/s: 9x the sustained demand with a quarter of the machine. Even A (221/s on one thread) covers walking and ground cruise.
- Depth 8 at 350 m/s needs more than 240/s. A2 with 8 threads (1750/s) still covers it 7x over. Rust would give 40x.
- One chunk takes 3.3 ms (A2), far under one frame (16.7 ms). Late or popping terrain from generation latency is not expected at this workload. Frame time is not part of this measurement (generation runs on workers); the main-thread LOD traversal and uploads are the same in every variant.
- Break-even (calculated, simple): with all other costs equal, A2 on 4 threads keeps up with the peak 144/s as long as the real generator costs less than about 7x this workload per chunk; with 8 threads about 12x. Cores shared with physics, collision patches and rendering lower this; a factor of 3 for that gives 2-4x. The real generator is not specified yet, so this is the number to watch (assumed margin).

## Effort (measured or counted)

| | Lines | Notes |
| --- | --- | --- |
| A | 277 | GDScript, closest to spike 1. |
| A2 | 367 | same output, more code for scalar maths and per-thread noise. |
| C | 489 (gen_core) + 175 (CLI) | `fastnoise-lite` port, bit-equal to Godot. Recompile after a change: 3.7 s. |
| B | 75 (binding) on top of gen_core | godot-rust 0.5.5, `experimental-threads` feature needed for parallel `&self` calls from worker tasks. Recompile after a change: 33.8 s (LTO plus one codegen unit over the godot crates). One compile error over `Dictionary::set` taking `&Variant`, otherwise no binding trouble. |

Rebuild time later in the session (measured): a separate Cargo profile `fast` (inherits release, no LTO, 16 codegen units, incremental) rebuilds C and B in about 0.3 s after a change, with the same speed as the release profile (C: 626 µs per chunk, 14060 chunks/s with 16 threads). The 34 s above is the full release profile; use `fast` for development and the LTO build for releases. First full build without cache: about 40-60 s.

Cost not measured: shipping GDExtension builds for every platform contributors use (the spike 4 export notes show macOS universal and Windows exports already take care), CI with a Rust toolchain, a second language for contributors. GDExtension is a red-class path in `AGENTS.md`; the approval here covers the spike branch only.

## What this does not show

- The real generator will be heavier (more bands, rivers, erosion, caves, sites). The numbers are for the test workload only.
- One machine, one run per configuration in the demand test, three reps in the generator tests, no spread beyond P95 and max.
- No rendering, no collision builds, no frame-time measurement. The demand counts uploaded chunks, not requested ones.
- Bevy was not benchmarked. Rust without an engine (C) is only the upper bound for the generator; it says nothing about the rest of an engine. Path 3 would need a reason beyond this generator.

## Open questions for the initiator

1. Is a margin of 7-12x over the measured demand enough to stay with GDScript for the generator, or should a heavier real workload (more bands, rivers) be benchmarked first?
2. If the generator grows, the cheapest steps before Rust: lower the amplitude of expensive bands (fewer octaves), bake low-frequency bands into the macro image, limit generation to about 8 threads.
3. Should the upload limit (4 meshes per frame) be revisited? At depth 8 and high speed it is the first bottleneck, regardless of language.

Files: `spikes/gen_bench/{SPEC.md, gen_a.gd, gen_a2.gd, bench.gd, parity.gd, profile_*.gd, demand_test.gd, noise_probe.gd, exo_gen.gdextension, rust/}`. Run: `godot --headless --path . -s res://spikes/gen_bench/bench.gd -- --variant=A|A2|B`; C: `rust/target/release/gen_bench_cli bench out.json`.
