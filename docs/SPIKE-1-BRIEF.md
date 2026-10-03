# Spike 1 brief — planet (handoff for a new session)

Status: ready to start. Written 2026-10-03 at the end of the concept session. Read this first, then the files listed below.

## Goal

Prove or disprove: a small, complete, seamless planet (radius about 3 km) is feasible in Godot 4.7.2 on low-spec hardware. One player, one ship. Walk, fly, land, no loading screen, no visible seams or pops.

If it fails, we fall back to flat, bounded landing zones (see `FEASIBILITY.md`, "Fallback"). Spikes are throwaway prototypes: no final structure needed, but results must be written down.

## Read first (in the concept repo `~/Work/exo-1-concept`)

1. `docs/FEASIBILITY.md`: world design, numbers, the NMS findings, spike list, weaknesses and countermeasures. Facts are tagged **[verified]**, **[calculated]**, **[unverified]**.
2. `docs/CORE-LOOP.md`: what the game is, MVP scope, what we say no to.
3. `docs/DECISIONS.md`: decided, open and parked. Notably: no ship tuning, one player and one ship first, no double-precision build, learning from others is welcome but we copy no code, assets, data, names or texts.
4. `docs/SOURCES-TO-CHECK.md` and `research/sources/` (saved copies, private) if a claim needs checking.

## Where to work

- Code repo: `~/Work/exo-1` (public later). Scaffold only, **no commits yet**, `project.godot` belongs in the repo root. Read its `AGENTS.md` and `README.md`.
- Godot: `/usr/bin/godot`, version 4.7.2 stable (pin this version). Jolt is the default physics engine.
- Do the spike on a throwaway branch. Do not commit unless the initiator asks. Do not touch the concept repo's docs except to record spike results (ask first).
- The name "EXO-1" is only a working title (a game "Exo One" exists). Do not put it in new user-facing strings or file names beyond what already exists.

## Scope of spike 1

In:
- Cube-sphere planet, radius as a tunable parameter (start 3000 m), heightmap per cube face, noise-based terrain from a seed.
- Chunked LOD (quadtree per face), skirts or overlap against seams, chunks built off the main thread.
- A CPU height function (collision must not depend on GPU readback; issue #105256 about async readback is still open).
- Collision only in a ring around the player (about 100-300 m, unverified), `HeightMapShape3D` patches, patch width at most about 32 m (a 100 m flat patch deviates 0.42 m from the sphere).
- Radial gravity; `CharacterBody3D` with an `up_direction` aligned to the planet; first-person walking.
- A simple flying ship (`RigidBody3D` in Jolt) that can leave the ground, go to space and come back; altitude-based blending of gravity, drag and atmosphere (sky shader).
- Flat shaded look with smooth normals for texture blending, face normals for lighting (the NMS trick); triplanar texturing over the whole planet.
- A debug overlay: FPS, frame time, chunk counts, memory, altitude, height of the player above the surface.

Out (later spikes): network sync, getting out of a ship with reparenting details, cities, content schema, AgentBridge, floating origin (only test the limit in spike 5).

## Success criteria (initiator decides, these are starting values)

- Runs at stable frame rate on a low-spec profile (Compatibility renderer first, then Forward+); report numbers, not feelings, plus the initiator's feel feedback.
- Walking one full circumference (about 19 km at R = 3 km) feels fine; the horizon curvature does not annoy in first person.
- Ship ascent from the ground to orbit and back without visible pops, seams or holes.
- No precision jitter near the surface; test with the planet centre at the origin (the docs call 2048-4096 m the maximum recommended range for first-person) and note what you see.
- Collision patches appear and disappear without the player falling through.
- Time per chunk build and the longest frame spike are measured and written down.

## Questions the spike must answer

1. Is chunk generation fast enough in GDScript with threads, or do we need a different approach (compute shader for visuals only, native code)? Do not plan on GPU readback for collision.
2. Do seams show at LOD borders, and which fix (skirts, overlap) works?
3. How big is the "hairy ball" friction (tangents, normal mapping) with triplanar texturing in Godot?
4. Does reverse-Z and near/far (try near 0.05, far 50 km) cause z-fighting or artifacts; which renderers use reverse-Z?
5. Is the first-person precision at 3000 m from the centre acceptable (visual jitter, physics glitches)? If not, what is the smallest fix (shift planet centre, shift world, smaller radius)?
6. What radius feels right (try 1.5 km, 3 km, 5 km)?

## References (inspiration, read, do not copy)

- `research/sources/nms-gdc2017-*-transcript.txt`: how No Man's Sky stores terrain on cube faces and simulates on the sphere, regions, LOD, seams, fade-in. Voxels and caves are not our plan.
- `github.com/cuberact/godot-cuberact-planet-chunked-lod` (MIT, Godot 4.6+): cube-sphere, quadtree LOD, origin shifting, atmosphere. Learning project. Read it, build our own.
- `research/sources/godot-large-world-coordinates.md`: precision table.
- `github.com/Ivorforce/Godot4-Custom-Gravity` (MIT), `github.com/xen-42/godot-4-mini-planet-tutorial` (MIT, archived): gravity ideas.
- Licence rule: before reusing anything, check its licence file. Several repos found in the research have no licence; reading is fine, copying is not.

## Working agreements (from the concept session)

- Language: chat in German, documents and code comments in English. Keep proper German umlauts in chat.
- Roles: the initiator is designer and idea giver, gives feedback on how it feels and breaks problems down. The AI is the Godot and code expert and researches when needed. "We do it together."
- Simple start. Do not fix too many values; start with a first value, then tune by feel.
- Use `gh` for anything on GitHub (it is logged in); web fetches for GitHub pages are unreliable.
- Be honest about what is verified and what is a guess; mark it.
- When something is unclear, ask one question at a time.
- No personal data or customer data anywhere (organisation rule).
- Sources behind logins or cookie banners: the initiator saves pages as Markdown into `~/Downloads` and asks you to copy them into `research/sources/` in the concept repo.

## Suggested order of work

1. Minimal Godot 4.7.2 project in `~/Work/exo-1` on a throwaway branch (Compatibility renderer), one scene with a sphere, a first-person walker and the debug overlay.
2. Terrain on one cube face with LOD; then all six faces; then seams.
3. Collision ring and walking; radial gravity.
4. Ship: take off, leave the atmosphere, return; blending of gravity, drag and sky.
5. Measure and tune; try the radius variants.
6. Write the results into a short report (numbers, screenshots, what failed, what to do next) and give it to the initiator.

## Report back

For each question above: answer, evidence (numbers or a screenshot), confidence. List what would make you choose the fallback.
