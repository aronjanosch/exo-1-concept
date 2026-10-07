# Spike 8 report — the procedural planet

Date: 2026-10-07. Status: done. Frozen as tag `spike/8-planet-gen`, `spike/combined` fast-forwarded to it (2026-10-07). The local Godot check was dropped: the engine moves to Bevy once spike 9 passes, and spike 9 judges the planet's look again. Code and full report: branch `spike/planet-gen` (commit `0bfd853`) in the code repo, `spikes/planet_gen/REPORT.md`, raw outputs in `spikes/planet_gen/results/`, 9 screenshots in `spikes/planet_gen/shots/`. Brief: `spikes/planet_gen/BRIEF.md` on the same branch. Tags: **measured**, **calculated**, **assumed**.

## Answer in one line

A Rust generator (macro shell, three stamps, crust bands, scatter, sites) drives the planet scene through one height function; mesh, collision and safety net agree within about 1 mm, there are no seams, and a walker never falls through. Whether the planet reads as a place is for the initiator to judge locally.

## What was built

- `planet_core` (plain Rust): recipe from `recipe.json`, macro bake, basin, escarpment and plateau stamps, sea level, the one height function, chunks, scatter, sites, statistics.
- `planet_godot`: GDExtension class `PlanetGen`, thin conversion layer. Each planet body has its own instance and seed.
- Scene: terrain, collision ring, safety net and F3 overlay use the Rust height function; water sphere at sea level; MultiMesh canopy and rocks on the finest chunks; 24 site pillars; biome-tinted shader.
- Debug: orbit camera on **O**, overlay ground line, `--planet-walk`, `--planet-shots`.
- `project.godot` unchanged against the branch start.

## Numbers (measured, cloud container, 8 cores, no GPU)

| Test | Result |
| --- | --- |
| T1 one height function | Mesh against `height_at`: 0.55 mm (core), 0.59 mm (scene, 1.14 M vertices). Collision patches 0.30 mm; 2.3 mm with the float32 body origin the physics server sees (engine limit at 5 km) |
| T2 seams | All twelve cube edges, worst 0.086 mm, no unmatched vertex |
| T3 macro | Sea level -7.49 m; land 70 % on the macro field, **79.7 %** on the full height; basin below sea for about 1.68 km (target > 1 km); plateau +110 m, escarpment step +60 m; bake 486-547 ms on 8 threads |
| T4 sites | 24 placed, smallest distance 727 m, mean nearest 2.04 km, largest nearest **3.41 km** (about 32 min on foot) |
| T5 walk | 0 rescues, 0 frames without collision patch in 8 runs. 5 min at 1.8 m/s is only 540 m. Brief wavelengths: one biome change per about 926 m; tuned (moisture 0.0008, landform 0.001): one per about 489 m, 0-2 changes per walk. Two walks stopped at the escarpment and plateau slopes (about 48° plus noise against a 50° floor limit) |
| T6 cost | Chunk 1.7-2.3 ms mean (spike 6: 0.65 ms), about half a core at 240 chunks/s; scatter 9 instances per finest chunk on average, max 36 |
| T7 regressions | `--auto-test` exit 0, 0 rescues; build script builds and exports Linux and Windows for spike 7 and spike 8 |

## Changed test values

The cloud session raised two region frequencies so a five-minute walk crosses biomes: moisture 0.0003 → 0.0008, landform 0.0004 → 0.001. Allowed by the brief (test values), stated in the report.

## Assumptions (main ones; full list in the code repo report)

- Stamps are evaluated analytically in the height function, not baked into the macro images.
- Macro images are vertex-centred, 513 × 513 per face.
- Escarpment runs east-west as a 1 km deep shelf with 300 m tapers.
- Biome rules are an ordered data list, first match wins (cold high, rim, wet, rest).

## Not verified

- Look: screenshots were made with OpenGL on Mesa llvmpipe under Xvfb, not Forward+ on Vulkan.
- Windows export was built but never started (no Windows or Wine in the container).
- Frame rate with the canopy (no GPU in the container).
- `is_on_floor()` is false in 6-37 % of walk frames although the walker follows the ground; cause not investigated (assumption: floor snapping on the 1 m height-field steps).

## Open after this spike

- Site spacing: 3.4 km worst gap. Walking that far is allowed (see `DECISIONS.md`, "Movement on planets"); whether sites should be denser or more evenly spread (best-candidate sampling) is a design question.
- Sea level rule: aim the 70 % at the full height function instead of the macro field? One change in the rule.
- Which biome row is the broken rim: today a landform id from noise plus the escarpment.
- `is_on_floor()` gaps: small issue for when jumping, footsteps or animation depend on it.

## Local check (dropped 2026-10-07, kept for reference)

1. Orbit (O): can you point at basin, rim and plateau? The rim is only 2 km long and may be hard to see from 15 km.
2. On foot: escarpment and plateau slope stop the walker; walk to them.
3. Forest edge and canopy density, site pillars from far away.
4. Frame rate with the canopy on the dev machine.
5. Start the Windows export once (Proton is enough).

