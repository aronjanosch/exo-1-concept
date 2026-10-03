# Feasibility — EXO-1 (working title)

Status: DRAFT. Result of research passes on 2026-10-03. Facts are marked **[verified]** (checked in a primary source), **[calculated]**, or **[unverified]** (general knowledge, secondary sources or guess). Unverified points must be checked in a spike or in the primary source before we rely on them. Links that could not be fetched are collected in `SOURCES-TO-CHECK.md`.

Decisions taken from this research live in `DECISIONS.md`.

## Verdict

A small 3D space game with host-authoritative co-op is feasible in Godot 4 if the scope stays small. We start with **one player, one ship**. The hardest part is not the engine but a seamless small planet (terrain, collision, atmosphere transition). Network sync for a single ship is much easier than the "players moving inside moving ships" problem, which we avoid for now.

## Engine: Godot 4 stays

| Criterion | Result |
|---|---|
| Governance | MIT, foundation-run; no licence account needed for contributors or CI |
| Scenes | text-based `.tscn`; merge conflicts are a known weakness (see below) |
| CI / bots | `--headless`, gdUnit4 or GUT, small exports |
| Low-spec hardware | good (Compatibility renderer, small binaries) |
| Multiplayer | built in (ENet, `MultiplayerSynchronizer`); the synchronizer has no interpolation **[verified]** |
| Physics | Jolt is the default for new projects since Godot 4.6 (official release page), experimental in 4.4 **[verified]** |

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
6. **Headless speed:** Godot issue #122707 (closed as "not planned") was titled as a stall after 25-55 s, but the discussion shows it is unproven: the report was AI-written, and the reporter themselves said it may be expected behaviour. A commenter found that `--headless` does not run with uncapped FPS even with vsync off, while a normal run with `--quit-after` does **[verified in the issue thread]**. So "run faster than real time" must be checked against our Godot version, but no real hang is confirmed.

## World design

- Fixed star system, hand-placed. Planets are data (seed, radius, list of places with coordinates).
- **Planets are small but complete spheres**, seamless between space, atmosphere and ground. Starting radius about 3 km (a tunable parameter).
- Numbers **[calculated]**: circumference about 19 km; horizon about 110 m at eye height (first-person feels flat); orbital speed about 171 m/s; orbital period about 110 s.
- **Precision [verified from Godot docs]:** float32 step is about 0.24 mm at 2-4 km and 3.9 mm at 32-65 km from the origin. A 3 km planet centred at the origin needs neither double precision nor floating origin **[calculated]**. Floating origin only matters for space flight beyond roughly 30-60 km or several distant bodies. There is no origin-shift API for particles (open proposal), so keep particles local.
- Terrain: deterministic heightmap from a seed on a cube-sphere with chunk LOD. No voxels. A flat heightmap patch deviates from the sphere by 0.42 m at 100 m width **[calculated]**, so collision patches should be at most about 32 m wide.
- Collision only in a ring around the player and ship (about 100-300 m **[unverified]**), `HeightMapShape3D` per patch, skirts against seams. Compute-shader readback in Godot is slow and blocking **[verified]**, so terrain height for collision should be available on the CPU.
- Hand-built places (city, outposts) sit at fixed coordinates in a local tangent frame; terrain is flattened under them. They are normal scenes.
- Interiors: small shops stay in the open world; large or complex interiors (for example a sewer) are instanced.
- Gravity: radial ("up" is away from the planet centre), `CharacterBody3D.up_direction` aligned; gravity, drag and atmosphere blended by altitude.
- Depth buffer: reverse-Z exists since Godot 4.3 **[verified]**; which renderers use it is open. Test near 0.05 / far 50 km in the spike.
- Between planets: short jump or proxy-scaled travel. Distant bodies shown as scaled proxies (skybox or separate camera).
- Multiplayer: terrain is deterministic from the seed, so only seed and later edits go over the network.
- MVP content: 1 planet, 1 small dense city (maybe 2-3 districts), 2-3 quest outposts. The space between places must not feel empty.

### Fallback

If the planet spike fails: flat, bounded landing zones per planet (planet as a sphere only seen from space). City, outposts and quests are unaffected because they are local scenes anyway.

## Spikes (each 1-2 days, in this order)

Game spikes (one player, one ship first):
1. **Planet:** rebuild the cuberact approach at R = 3 km; height from face heightmap plus detail noise; CPU height query; `HeightMapShape3D` only near the player; skirts. Must run on low-spec hardware without visible pops or seams.
2. **Transition:** `RigidBody3D` ship in Jolt with zero gravity; blend gravity, drag and atmosphere by altitude; reverse-Z test.
3. **Exit the ship:** `CharacterBody3D` with radial `up_direction`; player is a child of the ship while inside and reparents on exit.
4. **Network:** host-authoritative rigid ship, snapshots with interpolation (own buffer or netfox), test with latency and packet loss.
5. **Float limit:** fly the ship out to 100 km and measure jitter. Build origin shifting only if needed.

Test-infrastructure spikes (can run in parallel):
6. Headless speed and stability (issue #122707) against our Godot version.
7. gdUnit4 versus GUT.
8. Software-rendering (lavapipe) variance for the performance gate.

## Prior art: what we know and what we do not

- **Star Citizen:** 64-bit world coordinates, camera-relative rendering, hierarchy of local coordinate systems **[verified, secondary sources]**. The local physics grid per ship is roughly described as an own moving physics frame for the interior **[verified, coarse]**.
- **No Man's Sky:** deterministic generation from a 64-bit seed, generated near the player only **[verified]**. Normal ships have no walkable interior; only freighters, frigates and the corvette do **[verified]**. How coordinates and bases work is **[unverified]**: no slides or transcript found for the GDC 2017 talk.
- **Outer Wilds:** tiny planets (radii about 308 m to 2620 m, measured by a third party) **[verified]**. No developer talk on reference frames was found; only forum guesses. Not critical for us as long as planets do not orbit.
- **Starfield:** zone and cell loading, not seamless **[unverified]**.
- **KSP:** floating origin, shifts the world from roughly 2 km distance **[verified, forum sources]**; scaled-space proxies for distant bodies.

## Assets

- Assets are scripts (Blender Python via `blender -b -P script.py`, or GDScript/CSG), CI builds GLB and a preview image. No MCP is needed for the PR workflow.
- Style: flat shading with one palette texture in the Compatibility renderer. Animation is code-driven (tweens, rigid parts).
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
| Planet LOD | cuberact/godot-cuberact-planet-chunked-lod | MIT | Read as reference (cube-sphere, quadtree LOD, origin shifting, atmosphere); explicit demo, not production |
| Planet demo | Zylann/solar_system_demo | MIT (audio: check) | Reference only; needs custom engine build and strong hardware |
| Gravity | Ivorforce/Godot4-Custom-Gravity | MIT | Point-gravity idea |
| Mini planets | xen-42/godot-4-mini-planet-tutorial | MIT | Reference, archived |

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
- Performance gate: counters, object and memory growth and relative A/B measurements in the same job are meaningful; absolute FPS on shared runners or software rendering are not.
- PR security: only `pull_request` with a read-only token, no `pull_request_target` with checkout, pin actions to SHAs, approval for external contributors. The "Comment and Control" attack made several AI review actions leak secrets in PR comments, so AI reviewers only label, never approve, with no secrets and no shell. Builds for voters only after a maintainer label, first as web export, with artifact attestations.
- Dev-time MCP servers (Godot MCP) are used for development as the best tool, not the safest. Most can run arbitrary code (`run_script`, `game_eval`), which is why they are not suitable for the PR gate. The automated-test MCP is designed separately.

### AgentBridge (simple start)

An autoload that exposes the game to bots, tests and an MCP wrapper through one narrow interface:
- `get_state()` returns a JSON snapshot (ship position/velocity, fuel/hull, money, cargo, current contract, nearby places).
- `do_action(name, args)` takes one action from a fixed list (for example `throttle`, `turn`, `land`, `takeoff`, `accept_contract`, `trade`). No `eval`, no scripts.
- `step(ticks)` advances the game deterministically, and `reset(seed)` restarts with a fixed seed.

It is deliberately small. The same interface serves bot playthrough tests, headless smoke tests and (later) AI agents playing the game.
