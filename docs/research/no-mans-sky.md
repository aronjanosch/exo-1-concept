# No Man's Sky: how a shipped planet generator is structured

Research note, opened 2026-10-09. What the public record says about how Hello Games structures terrain, biomes, planet generation, fauna, weather and streaming in No Man's Sky (NMS), how one would read the shipped data later, and what of it is worth anything for `planet_core` and the backlog issues #72 to #75. This is not a proposal, not an approved design and not a decision. Nothing here is to be copied into EXO-1 code, content or data (see `docs/VISION.md`: our own code, assets, data, names and texts).

Same rule as `star-citizen-datamining.md`: read the *structure and the knobs*, not their values. Field, type and enum names below are quoted from public tooling only as evidence of how a system is cut. None of them is a suggestion for an EXO-1 name, and no NMS value appears here.

Sources come in three kinds, marked per claim:

- **[talk]** a Hello Games conference talk. The two GDC 2017 talks have auto-generated transcripts in `research/sources/` (private, see `SOURCES-TO-CHECK.md`); `docs/FEASIBILITY.md` already summarises them for the sphere and LOD questions. The link given is the public GDC Vault page.
- **[HG]** Hello Games' own pages and patch notes, or press quoting the developers.
- **[schema]** the type definitions in MBINCompiler (`libMBIN`), the community tool that decompiles NMS data files. They show which records and fields exist in the current build. `M` below is short for `https://raw.githubusercontent.com/monkeyman192/MBINCompiler/development/libMBIN/Source/NMS/`; every file under it was fetched on 2026-10-09. Names drift between game versions.
- **[wiki]** community modding wikis (STEP).

## The short answer

- NMS is the opposite end of our scale: an unbounded number of large voxel planets, generated near the player only, with each point computed without asking its neighbours. Our 6 km planet is finite and baked globally first. That difference decides almost every comparison in section 3.
- The most useful thing in NMS is not a technique but the **shape of its data**: a planet is a handful of small inputs (seed, star, biome, sub-biome, size class), and everything else hangs off the biome record (terrain weights, weather weights, object spawn lists, creature archetypes). That is the same "recipe per planet, biome rows as data" direction `planet_core` already has.
- For the later unpack step: since game version 5.50 (early 2025) the archives need **HGPAKtool**, the binary records need **MBINCompiler** matched exactly to the game build, and both run natively on Linux; the game itself runs under Proton. Nothing was downloaded for this note.

## 1. What the public record says

### 1a. Terrain

- Pipeline per region: generate voxels, polygonise, map onto the sphere, build the physics mesh, build the creature nav mesh, then populate with plants, creatures and buildings. **[talk]** https://gdcvault.com/play/1024265/Continuous-World-Generation-in-No (transcript in `research/sources/`)
- The mesher is dual contouring; they started with marching cubes. **[talk]** same transcript, https://gdcvault.com/play/1024265/Continuous-World-Generation-in-No
- Voxels, not a heightfield, because they wanted caves and overhangs, at the cost of generating far more data. **[talk]** https://gdcvault.com/play/1024514/Building-Worlds-Using (transcript in `research/sources/`)
- Every voxel is computed without querying its neighbours, otherwise generation cascades. Hence no real erosion, caves or **rivers**, "because one feature needs to flow into another and it needs context". Erosion-like looks come from analytical noise derivatives (slope known locally), domain warping and per-octave emphasis ("uber noise"). **[talk]** https://gdcvault.com/play/1024514/Building-Worlds-Using
- Voxel regions are octrees on a cube projected onto a sphere; storage on cube faces, evaluation on the sphere. **[talk]** nucl.ai 2015 write-up: https://procedural-generation.isaackarth.com/2016/06/22/building-a-galaxy-procedural-generation-in-no.html and `docs/FEASIBILITY.md`
- Terrain is data: `VoxelGeneratorSettings` holds one entry per terrain archetype, each a min and a max generator record that the planet interpolates between. **[wiki]** https://stepmodifications.org/wiki/NoMansSky:Tutorials/Terrain_Generation
- The generator record holds grid layers, a list of noise layers, features, caves, a base seed, sea level, beach height, minimum cave depth and a smoothing radius around buildings. **[schema]** `M` + `Toolkit/TkVoxelGeneratorData.cs`
- Noise layers come in fixed roles (base, hills, mountains, rock, underwater, texture, elevation, continent). **[schema]** `M` + `Toolkit/TkNoiseLayersEnum.cs`
- Each layer carries its own fractal knobs (octaves, gain, lacunarity, slope and ridge and altitude erosion terms, feature sharpening, remap) plus height, width, region scale, plateau terms, a seed offset and a **maximum LOD** above which the layer is skipped. **[schema]** `M` + `Toolkit/TkNoiseUberData.cs`, `M` + `Toolkit/TkNoiseUberLayerData.cs`
- Terrain archetypes are an enum of whole-planet looks (canyons, arches, craters, floating islands and so on), each with variants. **[schema]** `M` + `Toolkit/TkVoxelGeneratorSettingsTypes.cs`
- Player terrain edits are stored per planet as a buffer of small deltas (position plus a byte) under anchors, with age and protection flags, applied on top of the regenerated ground. **[schema]** `M` + `GameComponents/GcTerrainEditsBuffer.cs` (the "applied on top" reading is an inference)

### 1b. Biomes and planet types

- A planet has one biome type from a short enum (lush, frozen, toxic and so on, plus gas giants), and one sub-biome that varies it (quality tiers, odd structure sets, remixes). **[schema]** `M` + `GameComponents/GcBiomeType.cs`, `M` + `GameComponents/GcBiomeSubType.cs`
- The star's class gates which biomes and how much life can appear, through a table of biome lists per star type with a life chance. **[schema]** `M` + `GameComponents/GcBiomeListPerStarType.cs`; **[wiki]** https://stepmodifications.org/wiki/NoMansSky:Game_Structure/METADATA/SIMULATION
- One biome record bundles everything for that biome: weather option weights, terrain controls (weights on the voxel noise, grid, feature and cave layers), water, clouds, palette, texture and tile-type files, external object lists, resources and a weather change time. **[schema]** `M` + `GameComponents/GcBiomeData.cs`, `M` + `GameComponents/GcTerrainControls.cs`
- So in NMS the biome is per planet, not per place: variation inside a planet comes from tile types and object lists, not from biome borders. **[schema]** same files; consistent with **[wiki]** https://stepmodifications.org/wiki/NoMansSky:Game_Structure/METADATA/SIMULATION
- New biomes arrive as rolls on top of old ones (an existing type can turn into a new variant) so existing planets stay mostly put. **[HG]** https://www.nomanssky.com/origins-update/
- Worlds Part II added new planet kinds "without regenerating existing planets". **[HG, via press]** https://www.shacknews.com/article/142889/no-mans-sky-update-55-patch-notes

### 1c. Planet generation overall

- One seed generates everything; seeds moved from 32 to 64 bits; no load times, everything generated on the fly. **[HG, via press]** https://www.playstationlifestyle.net/2014/08/27/mans-sky-wont-feature-load-times/ and https://www.engadget.com/2014-08-18-it-takes-billions-of-years-to-see-all-of-no-mans-sky.html
- Hierarchy: galaxy region (a voxel of the galaxy), then system index, then planet index. **[community]** https://glyphs.had.sh/docs/FAQ
- A solar system record holds a seed, the star type, the planets, their positions and orbits, and one generation input per planet. **[schema]** `M` + `GameComponents/GcSolarSystemData.cs`
- The per-planet input is small: seed, star, class, size, biome, sub-biome, rings, a few system flags, index. Terrain and everything else are derived from it later. **[schema]** `M` + `GameComponents/GcPlanetGenerationInputData.cs`
- The full planet record (terrain, resources, weather) is filled by one function inside system generation. **[wiki]** https://stepmodifications.org/wiki/NoMansSky:Reference_Guides/GcPlanetData-RE
- The engine treats generated and hand-authored content the same way. **[talk]** https://gdcvault.com/play/1024265/Continuous-World-Generation-in-No; write-up https://procedural-generation.tumblr.com/post/158637281963/continuous-world-generation-in-no-mans-sky-gdc
- Points of interest sit on an offset grid so the nearest one can be found from anywhere without generating terrain. **[talk]** https://gdcvault.com/play/1024265/Continuous-World-Generation-in-No (see `docs/FEASIBILITY.md`)

### 1d. Fauna and flora

- Few real-world skeleton types, so few rigs; an animation system that adapts to body size; a tagging system links a creature's look to its sound and behaviour. **[HG, interview]** https://www.gameinformer.com/b/features/archive/2014/12/24/a-look-at-no-mans-skys-creature-features.aspx
- One base model holds all possible parts; code picks which are shown. **[press]** https://kotaku.com/a-look-at-how-no-mans-skys-procedural-generation-works-1787928446
- Part swapping is data: a descriptor file mirrors the scene hierarchy, and one entry per list is picked by chance. **[wiki]** https://stepmodifications.org/wiki/NoMansSky:Reference_Guides/DESCRIPTOR_Files
- Creatures have a role (predator, prey, passive, bird, fish and so on) and a body type, as separate enums. **[schema]** `M` + `GameComponents/GcCreatureRoles.cs`, `M` + `GameComponents/GcCreatureTypes.cs`
- Archetypes are split by domain (ground, air, water, cave) and weighted per biome and sub-biome, with a groups-per-area density, a life chance and role frequency modifiers. **[schema]** `M` + `GameComponents/GcCreatureGenerationArchetypes.cs`, `M` + `GameComponents/GcCreatureGenerationData.cs`
- A spawn entry carries role, type, groups per area, group size range, spawn and despawn distance, day and night activity chances, tile type, herd flag and rarity. **[schema]** `M` + `GameComponents/GcCreatureSpawnData.cs`
- Flora and objects use placement rules: angle range, height range, scale range, coverage, quality variants, seed. **[schema]** `M` + `GameComponents/GcObjectSpawnData.cs`
- Flora comes in storeys (ground cover, shrubs, trees), shares trunk rigs and varies branches and leaves; each specimen has slope and altitude preferences. Already used in `procedural-planet.md` section 5. **[talk]** https://gdcvault.com/play/1024265/Continuous-World-Generation-in-No
- Buildings use L-system files whose locators chain into one another. **[wiki]** https://stepmodifications.org/wiki/NoMansSky:Reference_Guides/LSYSTEM_Files

### 1e. Weather

- Per planet: one weather type, a storm frequency (four steps), an intensity (normal or extreme), an atmosphere type and colour indices for day, dusk and night. **[schema]** `M` + `GameComponents/GcPlanetWeatherData.cs`
- The biome supplies weights over the weather types; the planet rolls one. **[schema]** `M` + `GameComponents/GcWeatherOptions.cs`, `M` + `GameComponents/GcBiomeData.cs`
- Per weather type: several fog settings (normal, storm, in flight, extreme), a list of storms with chances, an extreme chance, and **hazard arrays** (temperature, toxicity, radiation, life-support drain) plus effect and hazard ids. **[schema]** `M` + `GameComponents/GcWeatherProperties.cs`
- A storm is a weight, a fog, colour modifiers and hazard modifiers: a storm is a temporary override of the base weather, not its own system. **[schema]** `M` + `GameComponents/GcStormProperties.cs`
- Weather files (list, fog, storms, particles, filters, hazards) live together under the simulation metadata. **[wiki]** https://stepmodifications.org/wiki/NoMansSky:Game_Structure/METADATA/SIMULATION
- Later additions came as events on top: lightning, rare tornadoes, meteor showers, firestorms. **[HG]** https://www.nomanssky.com/origins-update/
- One planet-wide wind drives trees, smoke, rain, fog, snow and waves together; volumetric clouds came with it. **[HG]** https://www.nomanssky.com/2024/07/no-mans-sky-worlds-part-i/

### 1f. Streaming and LOD

- Generation runs continuously and in real time; that is what makes the space-to-ground transition seamless. **[talk]** https://gdcvault.com/play/1024265/Continuous-World-Generation-in-No
- Regions with overlap, six LOD levels each twice the size, an octree; a cheap low-resolution planet from far away; generation ordered by visual benefit (near and coarse first); jobs off the main thread; physics and nav meshes cost more than the terrain. **[talk]** same; summarised in `docs/FEASIBILITY.md`
- Popping is hidden by dithered fades; distance-based fades work better than time-based but need terrain generated ahead. **[talk]** same
- Region LOD is data: radii per LOD, hidden ranges, object distance overrides, impostor ranges, impostors visible from space. **[schema]** `M` + `Toolkit/TkLODSettingsData.cs`
- Space to ground is staged: several planet LOD switch heights, an object switch, atmosphere start and end, plus fade times for terrain, flora and creatures and two object spawn radii. **[schema]** `M` + `GameComponents/GcEnvironmentProperties.cs`
- Objects stream in tiers: distant objects, landmarks, objects, detail objects, creatures. **[schema]** `M` + `GameComponents/GcEnvironmentSpawnData.cs`

## 2. Unpacking the shipped data (a later step, not done)

Nothing was downloaded or installed for this note. This is the procedure for when someone decides to read the actual records, for example a biome or weather file next to our recipe. Same storage rule as for Star Citizen: work in scratch space outside both repos, keep nothing, and if a raw file ever has to be kept it goes to `research/sources/` with an entry in `SOURCES-TO-CHECK.md`, never published.

### The tools

| Tool | What it does | Linux | Licence | State 2026-10-09 | Source |
|---|---|---|---|---|---|
| HGPAKtool | Reads the `.pak` archives of game version 5.50 (Worlds Part II) and later, every platform | native binary or `pip install hgpaktool` (Python 3.9+, `zstandard`, `lz4`) | MIT | 1.1.3, 2026-02-16 | https://github.com/monkeyman192/HGPAKtool, releases https://github.com/monkeyman192/HGPAKtool/releases |
| MBINCompiler | Binary `.MBIN` to text `.MXML` and back (EXML output dropped since Worlds Part II) | native `MBINCompiler-linux`, needs the .NET 8 runtime (a .NET 10 build exists) | LGPL-3.0 | v7.07.0-pre1, 2026-10-08 (prerelease) | https://github.com/monkeyman192/MBINCompiler, https://github.com/monkeyman192/MBINCompiler/blob/master/LICENSE.md, https://github.com/monkeyman192/MBINCompiler/releases |
| PSArcTool | Old PSARC `.pak` archives before 5.50 | Windows only, Wine needed [unverified] | none listed | last push 2016 | https://github.com/periander/PSArcTool |
| DepotDownloader | Fetch a specific older game build from Steam (needs an owned copy and login) | native | GPL-2.0 | 3.4.0, 2025-05-09 | https://github.com/SteamRE/DepotDownloader |

Other helpers seen but not needed for reading: AMUMSS (Lua mod builder on top of MBINCompiler, https://github.com/MetaIdea/AMUMSS-nms-auto-modbuilder-updater-modscript-system) and NMS Mod Builder (Windows only, https://github.com/cmkushnir/NMSModBuilder). Tool overview: https://stepmodifications.org/wiki/NoMansSky:Modding_Tools and https://stepmodifications.org/wiki/NoMansSky:HGPakTool.

### Version dependency

- The archive format changed with 5.50 (out by 2025-01-29, https://www.gematsu.com/2025/01/no-mans-sky-worlds-part-ii-update-now-available): PSArcTool for older builds, HGPAKtool for 5.50 and later. https://github.com/monkeyman192/HGPAKtool, https://stepmodifications.org/wiki/NoMansSky:The_Modding_Basics
- MBINCompiler is tied to one game build: "every update to the game breaks any number of MBIN formats", "each version of MBINCompiler is tied to a specific version of NMS". The README badges say which release matches the public and the experimental build; `MBINCompiler version <file>` tells which version wrote a file. https://github.com/monkeyman192/MBINCompiler
- The current public game version is listed at https://www.nomanssky.com/release-log/ (Cosmos 7.06 at the time of writing). Which build the newest MBINCompiler prerelease targets was not visible [unverified]; check the badges first.
- So: read the game version, then pick the matching MBINCompiler. If the installed game is newer than any MBINCompiler release, wait, or pin an older build with DepotDownloader.

### Linux with Proton

- NMS has no native Linux build on Steam; it runs through Proton. https://store.steampowered.com/app/275850/No_Mans_Sky/
- The archives sit in `steamapps/common/No Man's Sky/GAMEDATA/PCBANKS` inside the Steam library (by default under `~/.local/share/Steam/` [unverified]). https://stepmodifications.org/wiki/NoMansSky:The_Modding_Basics
- Both HGPAKtool and MBINCompiler run natively, so Proton is only needed for the game, not the tools. A full extract is about 45 GB (as of 5.12), so filter. https://stepmodifications.org/wiki/NoMansSky:The_Modding_Basics
- Hello Games tolerates mods: since patch 1.12 the game shows a warning when it detects them and has an official install path (https://www.kitguru.net/?p=314091; https://www.nomanssky.com/2025/02/worlds-part-ii-5-53). No EULA clause on extraction was found [unverified]. Reading for structure, keeping nothing, matches how we handled Star Citizen.

### Procedure (commands as documented by the tools)

1. Note the installed game version (in game or https://www.nomanssky.com/release-log/).
2. Optional, to pin a build: `DepotDownloader -app 275850 -depot <id> -manifest <id> -os windows -filelist <file with regex:GAMEDATA/PCBANKS/.*>`; depot and manifest ids not looked up yet. https://github.com/SteamRE/DepotDownloader
3. In a scratch venv: `python -m pip install hgpaktool`, or the Linux binary from the releases. https://github.com/monkeyman192/HGPAKtool
4. List first: `hgpaktool -L "<library>/steamapps/common/No Man's Sky/GAMEDATA/PCBANKS"`. https://github.com/monkeyman192/HGPAKtool/blob/master/hgpaktool/cli.py
5. Extract only what the question needs: `hgpaktool -U -f="<pattern>" -O <scratch> <PCBANKS>`; take the patterns from the listing (the simulation metadata folder per the wiki). https://github.com/monkeyman192/HGPAKtool
6. Get the MBINCompiler release that matches step 1 (`MBINCompiler-linux`) and the .NET 8 runtime. **This is the version-sensitive step.** https://github.com/monkeyman192/MBINCompiler
7. Convert: `MBINCompiler <file>.MBIN` gives `.MXML`. On failure run `MBINCompiler version <file>` and switch releases.
8. Write findings into this note as structure; delete the scratch folder.

Open before doing it: who owns a copy, and whether reading the files adds anything over the `libMBIN` type definitions already used in section 1. For structure alone, the schema files are enough; the unpacked files would only add which entries exist per biome.

## 3. Compared with `planet_core` and #72 to #75

What we have (code repo, `crates/planet_core/src/`): a recipe per planet as data (`recipe.rs`, `content/planet/<id>.json`, unknown fields rejected); a global bake of the macro shell, sea level, landform stamps and sites before any chunk (`planet.rs`, `landform.rs`, `site.rs`); one heightfield shell on a cube sphere, no voxels, no digging (decision in #62, https://github.com/aronjanosch/exo-1/issues/62); biome rows picked by nearest point in a parameter space of shared macro fields (`planet.rs` `biome_for`, #68); scatter per quadtree cell and storey as a pure function of seed and cell (`scatter.rs`, #65); chunks built on Bevy's async pool with a capped upload per frame (`crates/exo_app/src/terrain.rs`); a sky block per planet (`content/planet/*.json`, #67). The backlog issues: #72 rivers and lakes (https://github.com/aronjanosch/exo-1/issues/72), #73 fauna per biome (https://github.com/aronjanosch/exo-1/issues/73), #74 weather per planet (https://github.com/aronjanosch/exo-1/issues/74), #75 caves as instanced interiors (https://github.com/aronjanosch/exo-1/issues/75).

Verdicts are suggestions for discussion; the initiator decides.

### Terrain

- **Not worth it: voxels and dual contouring.** They are the price NMS pays for caves and overhangs (https://gdcvault.com/play/1024514/Building-Worlds-Using). We decided the planet stays a shell (#62) and caves are interiors (#75), so a heightfield keeps physics, LOD and scatter simple.
- **Worth it: noise derivatives as a local slope signal.** NMS uses analytical derivatives to make detail depend on slope without neighbour queries (https://gdcvault.com/play/1024514/Building-Worlds-Using). Our bands already have `stretch` and `roughness` driven by macro fields (`recipe.rs` `Shape`, `Band`); a slope term per band would be the same idea, and FastNoiseLite does not return derivatives, so it would cost a finite difference. Small, optional, only if the look harness (#63) shows uniform roughness on slopes.
- **Worth it, already have it: a maximum LOD per noise layer.** NMS skips fine layers at coarse LODs (`M` + `Toolkit/TkNoiseUberLayerData.cs`). Worth checking whether our chunk build evaluates every band at every quadtree depth; a per-band cutoff by chunk size is a cheap win if the perf harness (#19) points there.
- **Not worth it: terrain edit buffers.** NMS stores player digging as deltas (`M` + `GameComponents/GcTerrainEditsBuffer.cs`). No digging here (#62); our ground edits are site-authored and part of the height function (`site.rs`).

### Biomes

- **Different on purpose: biome per place, not per planet.** NMS gives a planet one biome and varies inside it with tile types and object lists (`M` + `GameComponents/GcBiomeData.cs`). Our planets are small enough to walk across, so several biome rows per planet are the point (#68, `procedural-planet.md` section 2). Keep ours.
- **Worth it: one record per biome bundles everything that follows from it.** In NMS the biome record points at terrain weights, weather weights, object lists and creature archetypes (`M` + `GameComponents/GcBiomeData.cs`). Our biome row already carries palette, tints and scatter multipliers (`content/planet/hearth.json` `biomes`); when #73 and #74 land, their per-biome parts should hang off the same row rather than build a parallel table.
- **Worth noting: the star gates the biome table.** NMS filters biome lists by star class (`M` + `GameComponents/GcBiomeListPerStarType.cs`). We author each planet's recipe by hand, so not needed now; it becomes interesting only if planets are ever generated from a system seed (`warp_core`).
- **Worth it as a rule: new content as rolls on top, not reseeding.** NMS added biome variants without regenerating old planets (https://www.nomanssky.com/origins-update/, https://www.shacknews.com/article/142889/no-mans-sky-update-55-patch-notes). For us that means: adding a recipe field should leave existing planets the same when the field is absent. Our scatter is already a pure function of seed and cell (`scatter.rs`); the same discipline matters for new layers.

### Planet generation

- **Worth it, we have it: small inputs, everything derived.** NMS derives a planet from seed, star, biome, size (`M` + `GameComponents/GcPlanetGenerationInputData.cs`). Ours derives from the recipe and a seed; our recipe is bigger because it is authored, not rolled.
- **Ours is better at this scale: the global pass.** NMS cannot place budgets ("about one caldera") because each point is local; it uses an offset grid for points of interest (https://gdcvault.com/play/1024265/Continuous-World-Generation-in-No). We bake stamps and sites globally with quotas and retries (`landform.rs`, `site.rs`), which is the advantage `procedural-planet.md` section 2 argues for. Keep it.
- **Worth it: generated and authored treated alike.** Same as NMS (https://gdcvault.com/play/1024265/Continuous-World-Generation-in-No): our sites are kits placed by the same bake, and hand-placed places can be site kinds with a fixed spot.

### Fauna (#73)

- **Worth it: role and body as separate axes.** NMS separates what a creature does (role) from what it is (body type) (`M` + `GameComponents/GcCreatureRoles.cs`, `M` + `GameComponents/GcCreatureTypes.cs`). For #73 that maps onto "behaviour preset" (flock, ground critter, flee distance) separate from "mesh from a Blender script", so one goofy body can flock on one planet and hop on another.
- **Worth it: spawn entries look like our scatter entries.** Density per area, group size range, spawn and despawn distance, day and night activity, biome weight (`M` + `GameComponents/GcCreatureSpawnData.cs`, `M` + `GameComponents/GcCreatureGenerationData.cs`). #73 already says "entries per biome like the scatter (#65)"; the NMS record confirms that shape, and adds two knobs our scatter lacks: a despawn distance (creatures move, props do not) and day/night activity (relevant once #48 is in).
- **Not worth it now: one base model with all parts, part swapping by descriptor.** It needs a rig system and many parts (https://stepmodifications.org/wiki/NoMansSky:Reference_Guides/DESCRIPTOR_Files, https://kotaku.com/a-look-at-how-no-mans-skys-procedural-generation-works-1787928446). Boids with a few authored bodies (#73) are far cheaper; part swapping could come later through the same Blender scripts.
- **Not needed: domain split (ground, air, water, cave).** NMS splits archetypes by domain (`M` + `GameComponents/GcCreatureGenerationArchetypes.cs`). #73 has fly and land states in one critter; a domain field is only worth it when water creatures exist.

### Weather (#74)

- **Worth it: base weather per planet plus storms as temporary overrides.** In NMS a storm is a weight, a fog and modifiers on top of the base weather (`M` + `GameComponents/GcWeatherProperties.cs`, `M` + `GameComponents/GcStormProperties.cs`). For #74 that means: the planet's sky and haze block stays the base (`content/planet/*.json` `sky`), and a storm is a list of overrides blended in over time, not a second sky model.
- **Worth it: one wind for everything.** NMS drives trees, smoke, particles, fog and waves from one planet wind (https://www.nomanssky.com/2024/07/no-mans-sky-worlds-part-i/). #74 already puts wind first; one wind resource that the foliage shader, the water ripple (`water.ripple_speed`) and later flight read is the structure to keep.
- **Worth it: fog per context.** NMS keeps separate fog for ground, storm and flight (`M` + `GameComponents/GcWeatherProperties.cs`). Our haze is one density per planet; from orbit and on foot want different amounts, which may show up in the look harness (#63) before weather does.
- **Not now: hazards.** Temperature, toxicity and radiation arrays feed survival meters (`M` + `GameComponents/GcWeatherProperties.cs`). We have no survival loop; #74 says look first. Revisit only if the core loop gets a suit resource.
- **Not now: weather picked by biome weights.** NMS rolls a planet's weather from its biome (`M` + `GameComponents/GcWeatherOptions.cs`). With two hand-authored planets, weather is simply written into the recipe, as #74 proposes.

### Caves (#75)

- **Different on purpose.** NMS caves are part of the voxel field and of the per-layer cave settings (`M` + `Toolkit/TkVoxelGeneratorData.cs`, https://gdcvault.com/play/1024514/Building-Worlds-Using). We decided caves are instanced interiors with a room graph (#75, #62). The closest NMS structure for a room graph is the L-system building layout, where locators chain into the next piece (https://stepmodifications.org/wiki/NoMansSky:Reference_Guides/LSYSTEM_Files); #75 already names Deep Rock Galactic and Star Citizen as the references, and NMS adds nothing beyond that chaining idea.

### Rivers and lakes (#72)

- **NMS cannot, we can.** The talk names rivers as one of the things a local generator cannot do, "because one feature needs to flow into another" (https://gdcvault.com/play/1024514/Building-Worlds-Using). Our bake is global and already reserves a channel for drainage (#72). This is the clearest case where our small finite planet wins; NMS is not a reference for #72, mapgen4 stays the one.

### Streaming and LOD

- **Worth it: object tiers with their own ranges.** NMS streams distant objects, landmarks, objects, detail objects and creatures as separate tiers (`M` + `GameComponents/GcEnvironmentSpawnData.cs`). Our scatter storeys already have their own quadtree depth and ranges (`crates/exo_app/src/scatter.rs`); sites are few enough to need no streaming (`crates/exo_app/src/sites.rs`). A landmark tier visible from far away is the one we do not have as such; it matters if the playtest (#71) asks for things to walk towards.
- **Worth it: fade by distance, not time.** NMS found distance fades less visible than time fades (https://gdcvault.com/play/1024265/Continuous-World-Generation-in-No). Our chunks swap on split and merge (`terrain.rs` `SPLIT_FACTOR`, `MERGE_FACTOR`); if popping shows in the look harness, this is the order to try.
- **Worth it, partly have it: staged space-to-ground switches as data.** NMS keeps the LOD switch heights, atmosphere bounds and fade times as one record (`M` + `GameComponents/GcEnvironmentProperties.cs`, `M` + `Toolkit/TkLODSettingsData.cs`). Ours are constants in `terrain.rs`; moving them into `content/tuning/` would match how we handle other tunables, once someone needs to tune them.
- **Already the same: generation off the main thread with a capped upload.** NMS runs region jobs off the main thread (https://gdcvault.com/play/1024265/Continuous-World-Generation-in-No); `terrain.rs` builds chunks on the async compute pool and uploads at most a few per frame.
- **Watch: physics and nav meshes cost more than terrain.** NMS's own finding (same talk). Our collision comes from a ring of patches, not from chunks (`terrain.rs`: "view only: collision comes from the ring"); nav meshes would arrive with #73 only if critters walk, which boids avoid.

## What EXO-1 takes from this

- **Structure ideas** above may inform discussions on #72 to #75 and on the recipe: per-biome rows as the hub, storms as overrides, one wind, role separate from body, spawn entries shaped like scatter entries, tiered streaming, distance fades.
- **No values, names, texts, files or assets** from NMS enter our code or `content/`. The enum and field names quoted here are evidence, not vocabulary; our biome, weather and creature names stay with the initiator (#62 "Open").
- Any later unpack happens in scratch space and keeps nothing (section 2).

## Open questions / unverified

- The GDC transcripts are auto-generated and from 2017; the schema files are from the current build. Where they disagree (for example how caves are generated today), the schema is the newer evidence, but it shows fields, not behaviour.
- Planet sizes in NMS are player estimates only (`procedural-planet.md` section 3).
- The fandom NMS wiki refused access (HTTP 402) on 2026-10-09; biome lists came from the enums instead.
- Not verified: the "uber noise" credit claimed in a third-party README (https://github.com/gistya/NoMansTerrain); a Hello Games EULA clause on data extraction; the Steam depot and manifest ids; which game build the newest MBINCompiler prerelease targets.
