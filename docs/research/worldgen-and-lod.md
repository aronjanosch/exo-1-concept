# Worldgen and LOD research for EXO-1 (web, 2026-10-10)

Scope: extends `docs/research/procedural-planet.md` (section 4 Minecraft, section 6 sphere maps), `docs/research/scale-and-early-game.md` (Appendix A sizes, Appendix C radius) and `docs/research/no-mans-sky.md`. Those already cover vanilla pipeline stages, the mod table, the Appendix A size table, NMS record structure, and our `planet_core` numbers; this note adds what they lack.

Tags: [O] official/developer, [C] community wiki/measurement/forum, [P] press. [I] = my inference, not in the source. Where a source is weak it is marked "weak".

Not verified against the original text: the GDC 2017 talk (members-only, only the abstract was readable), the Strugar CDLOD paper, the Ulrich 2002 course notes, and Distant Horizons' README (the GitLab pages returned only nav chrome). Claims that rest on those are flagged.

---

## 1. Minecraft world generation (1.18+) and mods

**Density and noise.** Each block gets a 3D Perlin-based density value; density > 0 is solid, otherwise air. Height bias and base height shift the density vertically. The wiki does not define "noise router" in its text, only in diagram filenames. [C] https://minecraft.wiki/w/World_generation (section "Terrain > 3D noise")

**Biomes.** Six parameters: temperature, humidity, continentalness, erosion, weirdness, depth. Everything except depth depends only on horizontal position. Temperature has 5 bands, humidity 5, erosion 7, continentalness 7; peaks-and-valleys is derived from weirdness as `1 − |(3|weirdness|) − 2|`. Depth rises by 1/128 per block downward. [C] https://minecraft.wiki/w/World_generation ("Biomes > Overworld")

**Shape and biome share the same fields.** Continentalness, erosion and PV go through splines that give a height offset and vertical stretch. Higher continentalness raises average height; higher erosion lowers and flattens. So mountain biomes and mountain shapes co-occur because they read the same noises. [C] https://minecraft.wiki/w/World_generation ("Terrain > Splines")

**Height limits.** The current Overworld chunk spans Y = -64 to 319 (384 blocks, 24 sections of 16). Nether and End are 256 tall. [C] https://minecraft.wiki/w/Chunk . Lower in 1.18 than before (0 to 255). A weak content-farm source says vanilla 1.18 mountain terrain tops out near Y=256 and that a datapack lifts that cap; I could not confirm this on a wiki page. [C, weak] https://sportskeeda.com/minecraft/news-minecraft-1-18-terrain-build-limits-players-revealed

**How tall mountains get.** The 1.18 change is the taller build limit (the ceiling moved from 255 to 319) plus the 3D density that allows overhangs. Not verified: whether a given mod removes a vanilla terrain cap in code.

**Structure placement, in chunks.** `spacing` is the average distance in chunks between generation attempts (the grid cell size under `random_spread`). `separation` is the minimum distance between neighbouring attempts and must be below spacing. `frequency` (default 1.0) is the probability an attempt is made; biome or terrain can still reject it. Defaults: villages spacing 34 / separation 8; pillager outposts 32 / 8; ruined portals 40 / 15; strongholds are `concentric_rings` (distance 32 units of 6 chunks, count 128, spread 3). Max distance between neighbouring attempts is 2×spacing − separation. [O/C] https://minecraft.wiki/w/Structure_set . The Structures section on the world-generation page gives examples: desert pyramids 32/8/100%, buried treasure 1/0/1%, pillager outposts 32/8/20%, ruined portals 40/15/100%. [C] https://minecraft.wiki/w/World_generation ("Structures")

Converted [I]: with 16-block chunks, a village grid of 34 chunks is about 544 blocks between attempts; the minimum gap is 8 chunks (128 blocks). A structure spacing of 32 chunks is about 512 blocks. That is the scale vanilla assumes, and it is very dense relative to a 5 km planet.

**Carvers.** Caves run from Y=-56 to 180 (more likely in -56..47); Nether caves run 0..126; canyons start at levels 10 to 72. [C] https://minecraft.wiki/w/World_generation ("Carvers")

**Mods that make the world look "breathtaking":**
- Terralith: "over 95 brand new biomes" plus updates to nearly every vanilla biome; new terrain types (canyons, shattered biomes, floating islands, deep ocean trenches); underground biomes (Underground Jungle, Frostfire Caves). It describes itself as taking the 1.18 overhaul and "turning it up to eleven." [O] (developer page on CurseForge) https://www.curseforge.com/minecraft/mc-mods/terralith . The look comes from biome silhouettes and palettes plus a few spectacle landforms (spires, volcanoes, islands), not from a new noise.
- Tectonic: continent-scale landmasses "several thousand to tens of thousands of blocks wide"; mountain ranges "tens of thousands of blocks" that approach the build limit; jungle pillars over 100 blocks; 2–3 tiered plateaus with ramps; large badlands canyons; valleys inside plateaus; underground rivers carved where a range is too tall for a surface river; lava tunnels. Oceans reach into the deepslate layer. [O] https://www.curseforge.com/minecraft/mc-mods/tectonic . It gives no height numbers in its text.
- William Wythers' Overhauled Overworld: rebuilds all vanilla biomes with sub-biomes and smoother transitions, uses only vanilla blocks (vanilla clients can join servers running it). Add-ons: Navigable Rivers, Cliffs and Coves, Towering Tepuis. Its own page does not give biome or structure counts. [O] https://www.curseforge.com/minecraft/mc-mods/william-wythers-overhauled-overworld
- TerraForged and the Dynamic Trees points already live in `procedural-planet.md` section 4.3; not repeated.

**Why it feels alive despite emptiness** [I, from the above]: the first-order cause is that many layers vary on different scales and react to each other (shape, biome, surface, carvers, features), so a single screen contains several kinds of change. Overlap of biome silhouette and surface palette does most of the "look" work; structures are rare on purpose (spacing 30+ chunks) and are placed where the biome allows, so the few that appear are read as events. Tectonic's "range as long as a hike" is the kind of scale that makes flat ground between features read as weather, not void.

### Lessons for EXO-1
- Express POI density as spacing + separation + frequency, the way Minecraft does, but in metres: spacing is the average cell size, separation the minimum gap, frequency the reject rate. Then the knob is per-kind and per-biome, and the numbers are data, not code.
- Shape and climate should read the same few fields (continentalness, erosion, weirdness analogues, as splines). That is already our direction (`hearth.json` landform/macro fields); keep the splines explicit so a designer can read them.
- Spectacle landforms (ranges, plateaus, canyons, a few tall spires) do more for "breathtaking" than many small ones. Count them, do not sprinkle them.
- Treat the height limit as a design value: Minecraft's 384-block column is a hard ceiling set by the engine; for us the ceiling is `atmosphere_height` and the obstruction radius, so pick the ceiling first.

---

## 2. Distant Horizons (Minecraft LOD mod)

**What it is.** A client-side mod that adds LOD. It "renders simplified chunks outside of the normal render distance" and claims this widens render distance without harming performance. [O] https://gitlab.com/distant-horizons-team/distant-horizons (project page; the README was not readable).

**Detail levels, data format, generation, rendering.** Not verified. The README and wiki returned no body text in my fetches. A search summary (weak) says it renders at several detail levels by distance and stores LOD data that persists between sessions; the CurseForge page confirms that LODs persist for a client-only setup and must be regenerated by exploring an area again, and that a server can send LODs to clients who enable "Distant Generation". [O] https://www.curseforge.com/minecraft/mc-mods/distant-horizons

**View distances.** CurseForge: "extreme (256+) chunk render distances", with an example screenshot at 512 chunks. [O] https://www.curseforge.com/minecraft/mc-mods/distant-horizons . A search summary (weak, not the official page) gives a default of 64 and a maximum of 4096, and warns that values above 512 may need a lot of RAM or GPU. [C, weak] https://www.curseforge.com/minecraft/mc-mods/lod-level-of-detail

**Performance and memory.** Official-page figures: 6–8 GB RAM suggested for 4–8 core CPUs, 12–16 GB for 10+ cores; renders on the GPU via OpenGL or Vulkan; FAQ suggests a concurrent garbage collector for stutter; LOD generation can briefly reduce performance while new areas are explored. [O] https://www.curseforge.com/minecraft/mc-mods/distant-horizons . A 1.16.5 alpha changelog claims a large database size and memory drop at high distances (old, weak). [C, weak] same search result as above.

### Lessons for EXO-1
- The idea that matters is the split: full-detail chunks near the player, coarse LOD far away, both on the same world. Our coarse-global plus fine-per-chunk plan has the same shape. Confirm our LOD distance in chunk units, not metres, if we compare.
- Persistence per client and regeneration on exploration is a known cost. Our "coarse global layer cached per client" is the same tradeoff: cache it, and say so in the save format.
- The RAM numbers (6–16 GB) show how much a far-LOD cache costs in a voxel world. Our heightfield per planet is far smaller, which is an argument for the heightfield approach, not against it [I].

---

## 3. No Man's Sky (planet generation tech)

Not repeated from `docs/research/no-mans-sky.md`, which already covers the GDC pipeline, records, POI grid and uber noise. What the new search adds:

- GDC 2017 abstract (members-only, readable part): the talk covers "the techniques used to generate planets and the supporting structures ... allowing this to happen continuously in real-time", a "step-by-step breakdown of their generation pipeline ... from voxel-based world generation, through polygonization and texturing, to eventual population and simulation." [O] https://www.gdcvault.com/play/1024265/Continuous-World-Generation-in-No
- The developer's stated aim for the procedural layer: "to let our artists to produce more, rather than replacing them with an algorithm." [O, relayed] https://procedural-generation.tumblr.com/post/158637281963/continuous-world-generation-in-no-mans-sky-gdc
- Planet size: Murray on planet scale: "We're dealing with planet-sized planets. Even if a million of us played on one planet, we'd still be really far apart." [O, quoted by press] https://www.killscreen.com/no-mans-sky-virtual-world-so-big-you-may-never-meet-another-player/ . A fan measured half a planet on foot at about 12 h (single run, weak). [C] https://mein-mmo.de/en/no-mans-sky-spieler-wandert-planet,109110 . No official radius found.
- POI density in km: not found in any source I could reach.

### Lessons for EXO-1
- NMS's "planet-sized" framing is marketing, not a measured radius; the measured walk time (fan, 12 h for half a planet) is what players actually feel. Use walk and flight minutes as the target unit, not radius.
- Nothing new on POI spacing; the offset-grid lookup in `no-mans-sky.md` remains the one concrete technique to take.

---

## 4. Star Citizen planet tech (v4, v5)

**v4 (shipped in Alpha 3.8).** Terrain from two climate layers (temperature and humidity) that replace pre-baked colour textures; biome transitions and texture blending; object scattering; on-demand terrain generation; LOD transitions "significantly more fluid with less pop-ins and no global textures"; the new version "overcomes issues when transitioning from ground to space"; less texture memory than v3. [O] https://starcitizen.tools/Planet_Tech_v4 and https://starcitizen.tools/CitizenCon_2019_-_Terra_Firmer (session summary). Atmosphere: improved sunrise/sunset via an ozone-like layer; clouds came later. [O] https://starcitizen.tools/Planet_Tech_v4

**Not in the sources I could read:** tile layout, precision and floating origin, ecosystem rules, planet sizes beyond the existing Appendix A, atmosphere heights. The CitizenCon session video is the place for those, and I could not access it.

**v5.** Two different things. (a) The wiki page "Planet Tech v5 (triggerfish)" is an April Fools' piece (2020) about flat planets with edges and domes; the page itself does not say "joke", only that the category is "April Fool's" and the update "was to be implemented". Do not use it. [O] https://starcitizen.tools/Planet_Tech_v5_(triggerfish) (b) The August 12, 2026 roadmap roundup lists a real Planet Tech v5, marked Tentative, for Alpha 4.11: a rework of how planets are built, populated and rendered, with generation and spawning moved to the GPU for denser environments. [O] https://api.star-citizen.wiki/comm-links/21284 . An October 2026 report says it had an Evocati-only preview and no date was given. [P] https://theimpound.com/blogs/star-citizen-news/this-week-in-star-citizen-5-october-2026 (not fetched in full; from search snippet).

**Sizes and atmosphere (from Appendix A, rechecked for this note):** Hurston diameter 2000 km in-game (one source 2370); the Crusader atmosphere height in the wiki search snippet is 745,000 m, but that is Stanton II, not Hurston. Hurston atmosphere height was not found. [C] https://starcitizen.tools/Hurston . The "about 100 km" figure in `scale-and-early-game.md` could not be re-verified here; treat it as unverified.

### Lessons for EXO-1
- The v4 change (climate fields drive biome and texture; no baked global texture; "pop-in" removed by a fluid LOD transition) is the same direction as our seed + climate fields plan. The explicit "less pop-in" goal is worth a line in our acceptance criteria.
- v5 moving generation to the GPU is a warning for us: a CPU heightfield shared by render and collision is the simpler contract, and the render side can move later. Keep heights on CPU first (that is the plan) and measure before going GPU.
- Do not cite the 2020 April Fools' v5 page as a plan.

---

## 5. Rust and Bevy prior art, LOD for cube-sphere planets

**Veloren (voxel RPG, Rust).** Default world: x_lg = y_lg = 10, so 1024 × 1024 chunks. Each chunk is 32 × 32 blocks, 16 high in the voxel terrain (the chunk-format note). [O] https://book.veloren.net/players/world-generation.html and https://docs.veloren.net/src/veloren_common/terrain/map.rs.html . Derived [I]: 1024 × 32 = 32,768 blocks per side, about 32.8 km at the ~1 block/m scale the dev docs use (the docs say 1024 blocks per km). Generation: "10 minutes on a good CPU is expected, for standard-sized worlds"; each doubling of a dimension "roughly doubles world generation time and RAM consumption." [C] https://book.veloren.net/players/world-generation.html . This is the closest Rust reference for "how big can a generated world be before bake cost bites": about 1M chunks, minutes of bake, and the cost is linear in chunk count. A code comment says ~5 hours on a good computer; that is older and conflicts with the wiki (weak) https://docs.veloren.net/veloren_world/sim/index.html .

**big_space (Bevy).** Nestable integer grids (i8 to i128) with Bevy's `Transform`, "absolute coordinates without drift". Latest pairing listed: Bevy 0.19 with big_space 0.13; MIT/Apache; 407 stars, 120 commits on main, 3 open issues. [O] https://github.com/aevyrie/big_space . Relevance: our plan (f64 world space with render origin shift) is an alternative to this; adopting it is a new dependency and falls under "Ask first" in AGENTS.md.

**bevy_terrain (Kurt Kühnert).** Chunked clipmap data structure and the UDLOD geometry algorithm; spherical terrain is listed as planned, not done in the dev notes. Maturity: experimental [C] https://github.com/kurtkuehnert/bevy_terrain/blob/main/docs/development.md . Not usable as a planet crate now.

**loddy.** A small chunking/LOD helper for flat grids, last seen on Bevy 0.15. Not for spheres. [C] https://zff.dev/Azorlogh/loddy

**No maintained Bevy planet-LOD crate found** in these searches (checked crates.io-style results only; not exhaustively).

**Non-Rust references, same family:** Cuberact's GDScript cube-sphere quadtree (already in `procedural-planet.md`), Hoimar's Planet-Generator; the cube-sphere quadtree idea (six face quadtrees, neighbour handling across cube edges) is described in an acko.net write-up. [C] https://acko.net/blog/making-worlds-1-of-spheres-and-cubes/

**LOD techniques.**
- Chunked LOD (Ulrich, SIGGRAPH 2002 course notes): quadtree of chunks, each chunk a fixed-resolution mesh; cracks between levels are hidden with skirts, a strip of triangles hanging down from each chunk edge. [O/C] Ulrich's demo and source at https://tulrich.com (reference via https://www.cs.cit.tum.de/fileadmin/w00cfj/cg/Research/Tutorials/Terrain.pdf). Skirts are described in a Leadwerks overview as "a small, angled skirt around every patch" [C] https://www.leadwerks.com/community/blogs/entry/1163-large-scale-terrain-algorithms/ . Original paper text not read.
- CDLOD (Strugar): continuous morphing per vertex based on distance; handles seams between LODs without skirts. [C] https://github.com/tschie/terrain-cdlod and forum discussion https://gamedev.net/forums/topic/620084-a-good-heightmap-lod-technique-no-vtf-please/4913948/ . The forum thread notes it is interpolation between discrete LODs, and the morph must sample the heightmap with bilinear filtering.
- Clipmaps (nested rings around the camera, used by bevy_terrain's Chunked Clipmap) suit flat terrain, not a cube-sphere with six faces; not recommended here [I].

**Recommendation for a low-poly, flat-shaded look [I, not from a source]:** a per-face quadtree of fixed-size chunks (Ulrich style), with skirts rather than per-vertex morphing. Reasons: flat shading makes a morphed vertex show as a shading pop anyway; skirts are simpler to get right on CPU-built meshes; and the chunk size is fixed in metres, which our finest-chunk rule (about 37 m in `scale-and-early-game.md` Appendix C) already uses. If pop-in is visible in the first playtest, add a per-chunk height morph (geomorph between two grid resolutions) instead of per-vertex CDLOD.

### Lessons for EXO-1
- Use Veloren's bake-cost pattern as the benchmark: report bake time and RAM per planet at 1 doubling of linear resolution (it doubles time and RAM), and measure that before choosing a macro resolution.
- Stay with our own heightfield and chunk code; big_space and bevy_terrain do not fit now (dependency rules, maturity).
- Chunked quadtree plus skirts is the lowest-risk LOD for a flat-shaded planet; keep geomorph as a fallback.

---

## 6. Scale and fun (target radius)

**Quotes and numbers on radius and traversal (all from the existing Appendix A, rechecked where noted):**
- SC: Hurston's size was set by "the balance between scale and traversal times" [P relaying CitizenCon 2018] https://www.dualshockers.com/star-citizen-hurston/ ; a quantum hop between planets takes about 8 min [C] (existing doc).
- KSP: Kerbin radius 600 km, atmosphere about 70 km (wiki: "70,000 meters"; a Swedish page gives 69,078 m), highest peak 6,767 m. [O] https://wiki.kerbalspaceprogram.com/wiki/Kerbin . Peak/atmosphere ratio about 0.1; peak is about 1.1 % of the radius.
- Space Engineers: diameters 120 km (Earth-like, Mars, Alien), 80 km (Triton), 60 km (Pertam), 19 km moons at 0.25 g; atmosphere heights and max terrain heights are not on the wiki page. [C] https://spaceengineers.wiki.gg/wiki/Planets (radius = half of these).
- Dual Universe: planets are voxel-based down to 5 km depth; each planet dominated by one biome; moons are airless. Radius and atmosphere height not found. [C] https://dualuniverse.fandom.com/wiki/Moon
- Veloren (derived): about 32.8 km per side (see section 5).
- NMS: "planet-sized"; walkable half-circumference about 12 h (one fan run). Murray quote in section 3.
- Minecraft (not a planet, but the only measured "tall" reference): a column of 384 blocks; Tectonic's ranges are "tens of thousands of blocks" long and "approach the build limit" tall. [O] https://www.curseforge.com/minecraft/mc-mods/tectonic

**Mountains against the atmosphere.** KSP: peak 6.8 km under 70 km atmosphere (about 10 : 1). SC Hurston: atmosphere height not verified. Space Engineers: not stated. Minecraft: height ceiling is the build limit, not an atmosphere. For a 5 km EXO-1 planet with an atmosphere of 1.2 km (existing `system.json`), a 1 km mountain would reach the atmosphere top [I]. That is the most important check for the target radius and it is absent from every source I could read.

**POI density.** Not found as a km figure in any game above. The only sourced density method is Minecraft's spacing/separation in chunks (section 1), which is a grid rule, not a km target. Flight minutes between POIs is what the existing appendix computed for our code (1–5 minutes at 15–30 km radius, from `scale-and-early-game.md` Appendix C); no game source gives an equivalent.

### Lessons for EXO-1
- Pick the radius from the two measured quantities players feel: minutes of walking or flight between POIs, and the mountain-to-atmosphere ratio. Take KSP's 1 : 10 as a rough reference, not a rule.
- Set POI density as a spacing rule in metres (Minecraft's spacing/separation/frequency) and derive the per-planet count from radius; avoid fixed counts, which the appendix already flagged as the failure at larger radii.
- Check the ceiling: keep the tallest terrain well below `atmosphere_height` on the chosen radius, and state the ratio in the target.

---

## Patterns

1. Two-scale generation is the consensus: coarse, global, cheap-to-cache fields (biome, climate, landform) plus fine detail generated locally (Minecraft noise and features; Distant Horizons LOD; SC v4 on-demand terrain; NMS voxels).
2. Shared fields drive several systems: Minecraft's splines tie height to biome; SC v4 tie texture and biome to temperature and humidity. This favours one heightfield plus climate fields feeding both render and collision.
3. Density is a rule, not a count: Minecraft uses spacing, separation and frequency in chunks; NMS uses a grid for lookup; Veloren and Minecraft both scale cost linearly with area.
4. LOD near the camera is full detail; far is coarse; the boundary is the risk (pop-in). SC v4 and CDLOD both target it; skirts avoid it at the cost of hidden triangles.
5. Ceilings are data: the build limit (Minecraft 319), the atmosphere (KSP 70 km) and the planet radius set how tall anything can be; games set these first.
6. Bake cost is linear in resolution and area (Veloren doubling; the existing Appendix C numbers). Measure before adding radius.

## Not found

- Minecraft: the exact "noise router" definition on a wiki page; a verified vanilla 256 terrain cap for 1.18 (only weak secondary sources); Tectonic/Terralith mountain heights in metres.
- Distant Horizons: README text (detail levels, database format, LOD tiers, chunk-to-LOD mapping, the official default and max render distance; only search summaries, weak).
- NMS: official planet radius; POI spacing in km; the GDC talk's content beyond the abstract (members-only).
- Star Citizen: v4 tile layout, precision/floating origin method, ecosystem rules, Hurston atmosphere height and mountain height; a date for Planet Tech v5.
- Dual Universe: planet radius and atmosphere height.
- Space Engineers: atmosphere heights and max terrain heights on the wiki page.
- Rust: a maintained Bevy planet LOD crate (none found in the searches); a Strugar CDLOD paper text; the Ulrich 2002 notes text.
- Any game that states "flight minutes between POIs" as a design target.
