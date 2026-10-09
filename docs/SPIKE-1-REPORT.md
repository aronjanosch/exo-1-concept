# Spike 1 report — planet

Date: 2026-10-03. Brief: `archive/SPIKE-1-BRIEF.md`. Code: `~/Work/exo-1`, branch `spike/planet` (throwaway, not committed), details and controls in `spikes/planet/README.md`.

## Result

A small, seamless planet works in Godot 4.7.2 with GDScript and threads: walk, board a ship, fly to space and back, land, no loading screens. No reason for the flat-zone fallback so far.

Initiator's feedback: still very simple, flying and walking need strong tuning and improvement, but a good first step for a spike.

Measurements are from the dev machine (RTX 5070 Ti).

## What was built

Cube-sphere terrain with quadtree LOD and skirts, chunks on worker threads; CPU height function; collision ring of `HeightMapShape3D` patches around the active body; radial gravity walker; arcade ship (`RigidBody3D`, Jolt) with altitude-blended gravity, drag and a planet-aware sky; debug overlay; scripted test run (`--auto-test`) with frame-time and precision stats; depth-buffer test scene.

## Questions from the brief

| # | Question | Answer | Evidence | Confidence |
|---|---|---|---|---|
| 1 | GDScript with threads fast enough? | Yes on the dev machine | Chunk build 2.2-2.7 ms avg (6 ms max) on workers; main thread at most about 5 ms per frame; 0 frames over 33 ms in all clean test runs | High for the measured dev machine |
| 2 | Seams at LOD borders, which fix? | Skirts work | Screenshot checks; two shader bugs found and fixed (skirt normals, NaN from derivative normals) | High |
| 3 | "Hairy ball" friction with triplanar? | None so far | World triplanar colour needs no tangents; normal maps not tested | Medium |
| 4 | Reverse-Z, near 0.05 / far 50 km? | Forward+: no z-fighting. Compatibility: z-fighting from about 500 m at 1-10 cm gaps | Depth test scene, both renderers; Compatibility matches a classic 24-bit buffer | High |
| 5 | First-person precision at the surface? | Fine up to 8 km radius | Walker and landed ship standing still: no measurable jitter up to 16 km from the centre; at 16 km walking catches on an invisible edge (likely patch offsets, not verified). **Corrected by spike 5:** the edge was the parked ship in the walk path, not precision; walking works up to R = 64 km (`SPIKE-5-REPORT.md`) | High to 8 km |
| 6 | Which radius feels right? | 5 km as first guide value (initiator, see `DECISIONS.md`) | 1.5, 3, 5, 8 km all run without performance difference | — |

## Success criteria

- Stable frame rate: yes on the dev machine (worst frames 8-29 ms in clean runs).
- Walking a full circumference: not done.
- Ascent to orbit and back without pops, seams, holes: no holes or seams found in screenshots; popping not judged yet (no LOD fade).
- No precision jitter near the surface: yes (see question 5).
- No falling through collision patches: 0 rescues in every run.
- Chunk build time and longest frame: see question 1.

## Open

- Renderer choice resolved for now (initiator, 2026-10-04): Forward+ with Vulkan; revisitable based on measured performance and visual correctness (see `DECISIONS.md`).
- Larger planets than about 8 km and several planets need an origin shift (spike 5). Spike 5: physics is fine much farther out; the visible limit is about 50-60 km from the origin, and an origin shift fixes it (`SPIKE-5-REPORT.md`).
- All design-relevant values and behaviours in the prototype (speeds, gravity, hover assist, landing aid, boarding, terrain shape, look) are assumptions for testing, not designed. The faceted terrain look does not match the look in `DECISIONS.md`.
- No LOD fade or geomorphing yet.

Spike 2 (transition: Jolt ship without engine gravity, altitude blending of gravity, drag and atmosphere, reverse-Z test) is covered by this spike.

Learnings from this spike are in `LEARNINGS.md`.
