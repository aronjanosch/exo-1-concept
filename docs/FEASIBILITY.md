# Feasibility — EXO-1 (working title)

Status: DRAFT. Result of research passes on 2026-10-03. Facts are marked **[verified]** (checked in a primary source), **[calculated]**, or **[unverified]** (general knowledge, secondary sources or guess). Unverified points must be checked in a spike or in the primary source before we rely on them. Links that could not be fetched are collected in `SOURCES-TO-CHECK.md`.

Decisions taken from this research live in `DECISIONS.md`; where they differ, `DECISIONS.md` wins. Since then: client authority instead of host authority (2026-10-06), fully Rust with Bevy once spike 9 passes (2026-10-07), so the Godot sections below are the reasoning of 2026-10-03.

## Verdict

A small 3D space game with host-authoritative co-op is feasible in Godot 4 if the scope stays small. We start with **one player, one ship**. The hardest part is not the engine but a seamless small planet (terrain, collision, atmosphere transition). Network sync for a single ship is much easier than the "players moving inside moving ships" problem, which we avoid for now.

## Engine: Godot 4 stays

| Criterion | Result |
|---|---|
| Governance | MIT, foundation-run; no licence account needed for contributors or CI |
| Scenes | text-based `.tscn`; merge conflicts are a known weakness (see below) |
| CI / bots | `--headless`, gdUnit4 or GUT, small exports |
| Runtime performance | aim for the best performance with a simple look; compare renderers using measurements |
| Multiplayer | built in (ENet, `MultiplayerSynchronizer`); the synchronizer has no interpolation **[verified]** |
| Physics | Jolt is the default for new projects since Godot 4.6 (official release page), experimental in 4.4 **[verified]** |
| Version | Latest stable is 4.7.2 (confirmed by the initiator; released 2026-08-18; 4.7 on 2026-06-18, 4.6.3 on 2026-05-20) **[verified via `gh release list`]**. Pin one version for the project; 4.5 added a shader baker and 3D physics interpolation in the scene tree, 4.4 async GPU readback **[verified from release pages, per research agent]** |

Alternatives considered:
- **Unity:** good engine (Schedule I uses it), but closed source, licence activation in CI, past licence changes, scenes/prefabs are YAML with `fileID`/GUID references that merge badly. C# code itself merges fine.
- **Unreal:** binary assets, heavy, C++ barrier.
- **Bevy:** only serious alternative; Rust barrier and breaking API changes every release hurt casual and AI-assisted contributors.
- **Stride:** community too small.

Escape hatch: Godot is MIT, so extending or forking the engine for our own needs is possible without upstreaming. It would force custom builds for everyone, so only as a last resort.

Note: since July 2026 Godot bans autonomous AI agents and substantial AI code for contributions to the *engine* repo. Our game is a separate repo and not affected, but engine bugs need workarounds or a human upstream PR. The tension with our AI-as-amplifier stance is accepted and documented as part of the experiment (see `DECISIONS.md`).

### Known Godot weaknesses and countermeasures

1. **`.tscn` merge conflicts:** keep scenes small, logic in GDScript, data in data files, CODEOWNERS per scene, agents prefer scripts over scene edits. `gdmerge` (MIT) exists but is very young: test first.
2. **Agents and scenes:** CI loads every scene headless and fails on broken UIDs/paths.
3. **Large worlds:** no double-precision build (needs custom engine and templates; host and clients must match).
4. **Network physics:** default sync is not ready for rigid-body ships. Host-authoritative ship with snapshots and interpolation (own buffer or netfox).
5. **Custom engine builds:** avoid, so standard export templates keep working.
6. **Async compute readback:** `buffer_get_data_async` was added in 4.4, but issue #105256 ("Compute shaders with `buffer_get_data_async` not working", opened 2025-04-10) is still open **[verified via `gh`]**. Do not plan terrain collision on GPU readback; keep a CPU height function.
7. **Headless speed:** Godot issue #122707 (closed as "not planned") was titled as a stall after 25-55 s, but the discussion shows it is unproven: the report was AI-written, and the reporter themselves said it may be expected behaviour. A commenter found that `--headless` does not run with uncapped FPS even with vsync off, while a normal run with `--quit-after` does **[verified in the issue thread]**. So "run faster than real time" must be checked against our Godot version, but no real hang is confirmed.

## World design

- Fixed star system, hand-placed. Planets are data (seed, radius, list of places with coordinates).
- **Planets are small but complete spheres**, seamless between space, atmosphere and ground. Radius: see `DECISIONS.md`.
- Numbers for a 3 km research example **[calculated]**: circumference about 19 km; horizon about 110 m at eye height (first-person feels flat); orbital speed about 171 m/s; orbital period about 110 s.
- **Precision [verified, Godot docs page saved in `research/sources/godot-large-world-coordinates.md`]:** float32 step size is about 0.0002 at 2048-4096 m from the origin and about 0.0005 at 4096-8192 m. The docs call 2048-4096 the maximum recommended range for a first-person 3D game and 4096-8192 for third-person; 32768-65536 (step about 0.0039) is the maximum for any 3D game, past which double precision is usually required. Open-world games with a playable on-foot area up to 8192 x 8192 m centred on the origin stay acceptable even in first person. The docs also say: most modern AAA open-world titles do not use large world coordinates; games split into levels with loading can centre each level on the origin. A 3 km planet (research example) centred at the origin needs neither double precision nor floating origin **[calculated]**, and because we stand on the surface (about 3000 m from the centre) we are at the edge of the first-person recommendation, so planet centre at the origin and keeping the player-relevant region near the surface are worth checking in the planet spike. Floating origin only matters for space flight beyond roughly 30-60 km or several distant bodies.
- **Double-precision build [verified, same page]:** needs `precision=double`, recompiled editor and export templates, GDExtensions rebuilt, shaders do not use double (emulated through a different engine path), server and all clients must use the same build type, and it costs performance and memory (aimed at mid-range and high-end desktops). The docs warn that origin shifting adds complexity, especially in multiplayer. Terrain3D's own double-precision page calls its support experimental with one positive report, and its maximum world size is about 65.5 km **[verified, `research/sources/terrain3d-double-precision.md`]**.
- Terrain: deterministic heightmap from a seed on a cube-sphere with chunk LOD. No voxels. A flat heightmap patch deviates from the sphere by 0.42 m at 100 m width **[calculated]**, so collision patches should be at most about 32 m wide.
- Collision only in a ring around the player and ship (about 100-300 m **[unverified]**), `HeightMapShape3D` per patch, skirts against seams. Compute-shader readback in Godot is slow and blocking **[verified]**, so terrain height for collision should be available on the CPU.
- Hand-built places (city, outposts) sit at fixed coordinates in a local tangent frame; flattening the terrain under them is an idea to try. They are normal scenes.
- Interiors: small shops stay in the open world; large or complex interiors (for example a sewer) are instanced.
- Gravity: radial ("up" is away from the planet centre), `CharacterBody3D.up_direction` aligned; gravity, drag and atmosphere blended by altitude.
- Depth buffer: reverse-Z exists since Godot 4.3 **[verified]**; which renderers use it is open. Test near 0.05 / far 50 km in the spike.
- Between planets: open, see `DECISIONS.md`. Distant bodies shown as scaled proxies (skybox or separate camera).
- Multiplayer: terrain is deterministic from the seed, so only seed and later edits go over the network.
- MVP content: 1 planet, 1 small dense city (maybe 2-3 districts), 2-3 quest outposts. The space between places must not feel empty.

### What the No Man's Sky talks confirm (GDC 2017, 9 years old)

Source: transcripts of two Hello Games talks in `research/sources/` **[verified, auto-generated transcripts]**. The techniques are from 2017; newer developments are still being collected (see `SOURCES-TO-CHECK.md`).

- They tried "flat plane wrapped onto a sphere" first and dropped it: places that cannot be mapped consistently (points of interest differed depending on where you took off), distortion at the poles, precision issues. They then simulated directly on the sphere with continuous 3D coordinates. This supports our sphere approach. Our fallback (bounded flat zones) differs: zones are fixed places, so the consistency problem does not apply.
- On a sphere "up" is simply the normalised position relative to the centre, cheap and simple. The costs are everywhere else: gameplay code, shaders and third-party libraries that assume a fixed up axis, and the "hairy ball problem" (no continuous tangent field), which hurts normal mapping and lighting. Their answer: triplanar projection, applied over the whole planet without ugly blend zones. We should expect the same friction in Godot.
- Storage on cube faces, simulation on the sphere: terrain is generated as if on the sphere, then mapped onto cube faces for storage. Elevation is a noise-varied radius (about 600 m to 1 km) plus a fixed local band (128 m). Matches our "heightmap per cube face" plan.
- LOD: 32 m regions (36 voxels with overlap so seams do not open), six LOD levels each twice the size, stored in an octree. A cheap low-resolution planet (six cube faces) is shown from far away. Generation order is driven by visual benefit (near first, coarse first).
- Pipeline per region: generate, polygonise, map onto the sphere, build the physics mesh, build the nav mesh, populate. Everything except a few sync points runs as jobs off the main thread. Physics and nav meshes cost more than the terrain itself. On PS4 some jobs ran in compute shaders. Relevant for Godot: threads and the readback problem.
- Look: they keep two normals, a smooth one for texturing and a face normal for lighting. That gives the low-poly faceted look with smooth texture blends. Directly relevant to our flat-shaded style.
- Popping: dithered fade-in, trees fade between impostor and real model while being re-seated on the finer terrain; distance-based fading is less noticeable than time-based but needs terrain generated far enough ahead.
- Placement: buildings and resources use an offset grid so the nearest one can be found from any point without generating terrain. Hand-placed places in EXO-1 simply have fixed coordinates.
- Generation must be data-local (a point cannot ask its neighbours), otherwise it cascades. Erosion-like looks are faked with noise derivatives and domain warping (Sean Murray's "uber noise": ridged and billow noise, analytical derivatives, domain warping, per-octave emphasis, exponentially distributed slopes). Since we use a 2D heightfield (no caves or overhangs), these techniques are cheap for us. Plain Perlin noise looks repetitive; real elevation data is realistic but boring to walk on.
- A curved world complicates small things too: a marker over a distant building must be projected to the horizon, with other planets possibly in between. A target arrow across a small planet needs the same care.
- Testing: a smoke-test tool flew drones over fixed planets after every build and recorded screenshots and performance; they also review many generated planets at once, not one. Useful pattern for our CI: fixed seeds, screenshots and performance counters per build, a contact sheet of many seeds.
- Their philosophy: procedural generation augments artists, it does not replace them; the engine does not care whether content is generated or authored. Matches our "procedural terrain, hand-built places".
- Caveats for us: NMS planets are huge by design (distances should feel weighty), voxels and caves are not part of our plan, and the talks do not cover multiplayer, Godot or our runtime performance. They also say nothing about how walking in moving ships works.

### Fallback

If the planet spike fails: flat, bounded landing zones per planet (planet as a sphere only seen from space). City, outposts and quests are unaffected because they are local scenes anyway.

## Spikes (each 1-2 days, in this order)

Game spikes (one player, one ship first):
1. **Planet:** rebuild the cuberact approach at R = 3 km; height from face heightmap plus detail noise; CPU height query; `HeightMapShape3D` only near the player; skirts. Aim for the best runtime performance without visible pops or seams.
2. **Transition:** `RigidBody3D` ship in Jolt with zero gravity; blend gravity, drag and atmosphere by altitude; reverse-Z test.
3. **Exit the ship:** `CharacterBody3D` with radial `up_direction`; player is a child of the ship while inside and reparents on exit.
4. **Network:** host-authoritative rigid ship, snapshots with interpolation (own buffer or netfox), test with latency and packet loss.
5. **Float limit:** fly the ship out to 100 km and measure jitter. Build origin shifting only if needed.

Planned Godot test-infrastructure spikes (headless speed against issue #122707, gdUnit4 versus GUT, lavapipe variance) were dropped on 2026-10-07; spike 9 is now the Bevy validation, spike 10 networking in Bevy (`DECISIONS.md`).

## Prior art: what we know and what we do not

- **Star Citizen:** 64-bit world coordinates, camera-relative rendering, hierarchy of local coordinate systems **[verified, secondary sources]**. The local physics grid per ship is roughly described as an own moving physics frame for the interior **[verified, coarse]**.
- **No Man's Sky:** deterministic generation from a seed, generated near the player only, simulated on the sphere with cube-face storage (see the section above) **[verified, GDC 2017 transcripts]**. Normal ships have no walkable interior; only freighters, frigates and the corvette do **[verified]**. How bases and multiplayer positions work is still **[unverified]**; the talks do not cover it.
- **Outer Wilds:** tiny planets (radii about 308 m to 2620 m, measured by a third party) **[verified]**. No developer talk on reference frames was found; only forum guesses. Not critical for us as long as planets do not orbit.
- **Starfield:** zone and cell loading, not seamless **[unverified]**.
- **KSP:** floating origin, shifts the world from roughly 2 km distance **[verified, forum sources]**; scaled-space proxies for distant bodies.

## Network sync for the ship (starting rules)

From Gaffer on Games, "Snapshot Interpolation" (2014, still the standard reference) **[verified, `research/sources/gaffer-snapshot-interpolation.md`]**:
- Deterministic lockstep needs a deterministic simulation and is recommended for 2-4 players at most; physics in Godot is not deterministic, so we use **snapshots**: the host simulates, clients only render interpolated state.
- Send snapshots over an unreliable channel, never over TCP; a lost snapshot is skipped, not resent. (Godot's ENet is UDP-based; use an unreliable channel for ship state.)
- Clients keep an interpolation buffer. Rule of thumb: delay about 3x the send interval so two lost packets in a row still leave something to interpolate towards (at 10 snapshots per second about 300 ms plus about 50 ms jitter margin, at 30 per second about 150 ms, at 60 per second about 85 ms). A higher send rate is the way to cut delay.
- Position: Hermite interpolation using the velocity in each snapshot looks much smoother than linear at the same rate; orientation: slerp is enough, angular velocity does not need to be sent.
- Extrapolation works badly for rigid bodies that collide; for mostly linear motion such as spaceships it may work for short stretches (50-250 ms), but not once objects hit others.
- Consequence for us: with one player and one ship the host-side state is small, so 20-30 snapshots per second with a buffer of 100-150 ms is a sensible first value. The player's own ship is controlled locally with host confirmation (client-side prediction); check netfox only if plain interpolation is not enough.

## Assets

Update 2026-10-08: the model source is a Blender Python script only (`DECISIONS.md`), and the tool research of that day (`research/design-tools.md`) replaces the text-to-3D notes below.

- Assets are scripts (Blender Python via `blender -b -P script.py`, or GDScript/CSG), CI builds GLB and a preview image. No MCP is needed for the PR workflow.
- Style: see `DECISIONS.md`. Animation is code-driven (tweens, rigid parts).
- Text-to-3D services (Meshy, Tripo, Rodin) are poorly suited to flat low-poly with a palette (topology, textures); some free-tier outputs are not licensed for commercial use. Hunyuan3D 2.1 excludes the EU, UK and South Korea by licence **[verified, secondary]**.
- Placeholders: Kenney and Quaternius (CC0, no attribution required, checked in primary sources) marked `placeholder: true`. Every asset carries origin and licence metadata.
- Pure AI output is probably not protected by copyright in the US and Germany (human authorship required) **[secondary sources; not legal advice]**.
- CI checks: metadata, triangle budget, palette, size, Khronos glTF validator.
- Models such as GPT-6 Astra/Sol exist, but native 3D capability is not verified. The working approach is that models produce assets through scripts and/or MCP.

## Reusable code: inspiration, not foundation

Rule: build the core systems ourselves (flight, landing, trade, content schema). Read others, copy little. Licence and activity must be checked in the repo before any reuse; data below comes from GitHub overview pages.

| Area | Project | Licence | Use |
|---|---|---|---|
| Network sync | foxssake/netfox | MIT | Option if plain interpolation is not enough |
| Planet LOD | cuberact/godot-cuberact-planet-chunked-lod | MIT, 147 stars, last push 2026-03-22 **[verified via `gh`]** | Read as reference (cube-sphere, quadtree LOD, origin shifting, atmosphere); explicit demo, not production |
| Planet demo | Zylann/solar_system_demo | MIT (audio: check) | Reference only; needs custom engine build and strong hardware |
| Gravity | Ivorforce/Godot4-Custom-Gravity | MIT | Point-gravity idea |
| Mini planets | xen-42/godot-4-mini-planet-tutorial | MIT | Reference, archived |
| Flat terrain (fallback) | TokisanGames/Terrain3D | MIT, about 4.3k stars, pushed 2026-10-03 **[verified via `gh`]** | Editable heightmap terrain for Godot 4 (C++ GDExtension). Not spherical, so only relevant for the flat-zone fallback; check the build and export requirements first |
| Planet renderer (reference) | kurtkuehnert/planetary_terrain_renderer | Apache-2.0, Rust, 28 stars, thesis project **[verified via `gh`]** | Idea reference for GPU ellipsoidal terrain; not Godot code |

Do not copy: projects without a visible licence (arthifact spaceship controller, TizioMaurizio Godot_Space_Program) or with an unclear one (CelestialSim, VR00D controller). Reading is fine, copying is not.

Gaps: no Godot 4 spaceship controller with a clear licence, no Godot 4 floating-origin addon, no co-op space template, no space HUD addon. All of these we build ourselves.

## Content schema and security

- Format: JSON, one file per object, validated with JSON Schema 2020-12. Path `content/<namespace>/<type>/<name>.json`, with `$schema` and `schema_version`. Translations separate (CSV or PO, both native in Godot). TOML and YAML have no native Godot parser. `.tres`/`.tscn` are not allowed for community content.
- Why not `.tres`: Godot resources can embed scripts that run on load (godot-proposals #4925). Real cases: GodLoader (malicious GDScript in a `.pck`, over 17,000 systems) and trojaned Schedule I mods. No case in the Godot asset library was found (a gap in the search, not an all-clear).
- Content holds only namespaced IDs (`ns:name`), never paths or class names.
- CI: path allowlist (PRs only touch `content/**` and `i18n/**`; no symlinks, binaries or scripts), `additionalProperties: false`, size limits, CODEOWNERS on code, schemas and workflows, custom script for forbidden APIs (`gdlint` is a style linter, not a security scanner; Semgrep does not support GDScript).
- Balance linter as a second step (price and reward formulas, reference integrity, arbitrage check): JSON Schema cannot do this. Findings are `error` or `warn`.
- Migration through `schema_version`, support N and N-1. No patch system in v1, only new IDs.
- Content PR automation: stage 0 allowlist, format, schema, lint; stage 1 headless smoke test with game code from `main` and only the PR data; stage 2 AI agent advisory only, never auto-merge.
- Not verified: the schema drafts in the research report were not run through a validator; claims about `ext_resource`/`sub_resource` embedding rest on format knowledge (the docs page returned 404); `str_to_var` and `ConfigFile` behaviour with embedded objects is unchecked and must not be used in the content path.

## CI and agent interface

- Detail is deferred to its own work item (see `DECISIONS.md`). Findings so far:
- gdUnit4 and GUT are both MIT with JUnit XML output. Bot playthroughs use a state JSON with a fixed action list, fixed seed and invariant checks. Godot maintainers state that physics is not deterministic in general, not even for repeated runs in the same build (issue #112976, a 2D case; the statement was general) **[verified in the issue thread]**, so game logic must be separated from physics and tests check invariants, not exact positions.
- GUT in GitHub Actions, from a single-author blog post (Godot 4.5.1, CC BY 4.0, `research/sources/gut-ci-medium-kpicaza.md`) **[verified, one source]**: run a headless import first (`godot --headless --path . --import --quit`) because `.godot/` is not committed and `class_name` scripts are not registered otherwise; then run GUT with `--headless --display-driver headless --audio-driver Dummy --disable-render-loop -gdir=res://tests -ginclude_subdirs -gexit`. Godot can print leak errors on exit and return non-zero although all tests passed, so the author sets `GODOT_DISABLE_LEAK_CHECKS=1`. Caveat: that also hides real leaks; a separate, deliberate leak check may be worth having. The workflow uses `chickensoft-games/setup-godot@v2`.
- Performance gate: counters, object and memory growth and relative A/B measurements in the same job are meaningful; absolute FPS on shared runners or software rendering are not.
- PR security: only `pull_request` with a read-only token, no `pull_request_target` with checkout, pin actions to SHAs, approval for external contributors. The "Comment and Control" attack made several AI review actions leak secrets in PR comments, so AI reviewers only label, never approve, with no secrets and no shell. Builds for voters only after a maintainer label, first as web export, with artifact attestations.
- Dev-time MCP servers (Godot MCP) are used for development as the best tool, not the safest. Most can run arbitrary code (`run_script`, `game_eval`), which is why they are not suitable for the PR gate. The automated-test MCP is designed separately.

### AgentBridge (proposal, open)

An autoload that exposes the game to bots, tests and an MCP wrapper through one narrow interface:
- `get_state()` returns a JSON snapshot (ship position/velocity, fuel/hull, money, cargo, current contract, nearby places).
- `do_action(name, args)` takes one action from a fixed list (for example `throttle`, `turn`, `land`, `takeoff`, `accept_contract`, `trade`). No `eval`, no scripts.
- `step(ticks)` advances the game deterministically, and `reset(seed)` restarts with a fixed seed.

It is deliberately small. The same interface serves bot playthrough tests, headless smoke tests and (later) AI agents playing the game.
