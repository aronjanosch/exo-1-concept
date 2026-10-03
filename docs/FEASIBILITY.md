# Feasibility — EXO-1

Status: DRAFT. Result of a first research pass. Facts marked *(unverified)* come from general knowledge or secondary sources and must be checked in a spike or in the primary source before we rely on them.

## Verdict

A small 3D space game with 2-5 player host-authoritative co-op is feasible in Godot 4, if the scope stays small. The hard parts are not the engine but (1) players moving inside moving ships over the network and (2) a seamless small planet. Both get a spike before anything else is built.

## Engine: Godot 4 stays

| Criterion | Result |
|---|---|
| Governance | MIT, foundation-run; no licence account needed for contributors or CI |
| Scenes | text-based `.tscn`; merge conflicts are a known weakness (see below) |
| CI / bots | `--headless`, gdUnit4 or GUT, small exports |
| Low-spec hardware | good (Compatibility renderer, small binaries) |
| Multiplayer | built in (ENet, `MultiplayerSynchronizer`) |
| Physics | Jolt integrated since 4.4 *(default since 4.6 is from one news source, unverified)* |

Alternatives considered:
- **Unity:** good engine (Schedule I uses it), but closed source, licence activation in CI, past licence changes, scenes/prefabs are YAML with `fileID`/GUID references that merge badly. C# code itself merges fine.
- **Unreal:** binary assets, heavy, C++ barrier.
- **Bevy:** only serious alternative; Rust barrier and breaking API changes every release hurt casual and AI-assisted contributors.
- **Stride:** community too small.

Escape hatch: Godot is MIT, so extending or forking the engine for our own needs is possible without upstreaming. It would force custom builds for everyone, so only as a last resort.

Note: since July 2026 Godot bans autonomous AI agents and substantial AI code for contributions to the *engine* repo (sources: The Register, Godot PR guidelines). Our game is a separate repo and not affected, but engine bugs need workarounds or a human upstream PR.

### Known Godot weaknesses and countermeasures

1. **`.tscn` merge conflicts:** keep scenes small, put logic in GDScript and data in `.tres`/data files, CODEOWNERS per scene, agents prefer scripts over scene edits, try a merge driver.
2. **Agents and scenes:** CI loads every scene headless and fails on broken UIDs/paths.
3. **Large worlds:** no double-precision build (needs custom engine and templates, and host and clients must match). Small planets plus origin shifting in GDScript.
4. **Network physics:** the default sync is not ready for rigid-body ships. Host authority with interpolation, ships kinematic where possible; netfox as an option.
5. **Custom engine builds:** avoid, so standard export templates keep working.

## World design

- Fixed star system, hand-placed. Planets are data (seed, radius, list of places with coordinates).
- **Planets are small but complete spheres**, seamless between space, atmosphere and ground. Starting radius about 3 km (a tunable parameter; circumference about 19 km, horizon about 110 m at eye height, so first-person feels flat, no "Mario Galaxy" curvature).
- Terrain: deterministic heightmap from a seed on a cube-sphere with chunk LOD. No voxels (no caves, no digging).
- Hand-built places (city, outposts) sit at fixed coordinates in a local tangent frame; the terrain is flattened under them with a stamp. They are normal scenes.
- Interiors: small shops stay in the open world; large or complex interiors (for example a sewer) are instanced.
- Gravity: radial ("up" is away from the planet centre), blended by altitude with atmosphere drag.
- Collision is only generated in a ring around players and ships.
- Float32 precision is fine at this radius. Floating origin is only needed for travel in space.
- Between planets: short jump or proxy-scaled travel, not full-scale simulation. Distant bodies are shown as scaled proxies (skybox or separate camera).
- Multiplayer: terrain is deterministic from the seed, so only seed and later edits go over the network.
- MVP content: 1 planet, 1 small, dense city (maybe 2-3 districts spread over the planet), 2-3 quest outposts. The space between places must not feel empty: procedural scatter, small finds, flight targets, a fast ship.

### Fallback

If the planet spike fails: flat, bounded landing zones per planet (planet as a sphere only seen from space). City, outposts and quests are unaffected because they are local scenes anyway.

## Spikes (each 1-2 days, in this order)

1. **Ship sync with players aboard:** 2-5 players, host-authoritative, ship physics with a reference frame (velocity matching or parenting). Biggest unproven point.
2. **Small planet:** 3 km radius, walk, fly, land, seamless atmosphere. Must run on low-spec hardware without visible pops or seams.
3. **Local reference frames:** player in or at the ship, leaving and entering.
4. **Grid ship builder:** stats derived from parts (slots or budget).
5. **First schema:** one planet and one contract as validated data.

## Prior art: what we know and what we do not

- **Star Citizen:** 64-bit world coordinates, camera-relative rendering, hierarchy of local coordinate systems (planet, ship, station). Per-ship physics grids are from general knowledge, not confirmed in this pass.
- **No Man's Sky:** deterministic generation from a 64-bit seed, generated near the player only (GDC 2017 talk exists, not read in full). Local coordinates instead of global double precision is a guess *(unverified)*. How walking in moving ships works there: *(unverified)*.
- **Outer Wilds:** tiny planets (radii about 308 m to 2620 m, measured by a third party). Reference-frame matching is plausible but not confirmed here.
- **Starfield:** zone and cell loading, not seamless *(unverified)*.
- **KSP:** floating origin, scaled-space proxies for distant bodies.

Open research: NMS talk slides, Outer Wilds frames, KSP/Elite scaling, vehicle network sync in Godot.

## Reusable code: inspiration, not foundation

Rule: build the core systems ourselves (flight, landing, trade, tuning, schema). Read others, copy little. Licence and activity must be checked in the repo before any reuse; the data below comes from GitHub overview pages.

| Area | Project | Licence | Use |
|---|---|---|---|
| Network sync | foxssake/netfox | MIT | Option if plain interpolation is not enough (prediction, rollback) |
| Planet LOD | cuberact/godot-cuberact-planet-chunked-lod | MIT | Read as reference for cube-sphere, quadtree LOD, origin shifting; learning project, not production |
| Planet demo | Zylann/solar_system_demo | MIT (audio: check) | Reference only; needs a custom engine build and strong hardware |
| Gravity | Ivorforce/Godot4-Custom-Gravity | MIT | Point-gravity idea |
| Mini planets | xen-42/godot-4-mini-planet-tutorial | MIT | Reference, archived |
| Inventory | peter-kish/gloot | MIT | Probably not needed; a simple inventory is built in-house |

Do not copy: projects without a visible licence (arthifact spaceship controller, TizioMaurizio Godot_Space_Program) or with an unclear one (CelestialSim, VR00D controller). Reading is fine, copying is not.

Gaps: no Godot 4 spaceship controller with a clear licence, no Godot 4 floating-origin addon, no co-op space template, no space HUD addon. All of these we build ourselves.
