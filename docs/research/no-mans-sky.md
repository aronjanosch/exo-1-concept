# No Man's Sky: how a shipped planet generator is structured

Research note, opened 2026-10-09. What the public record says about how Hello Games structures terrain, biomes, planet generation, fauna, weather and streaming in No Man's Sky (NMS), how the shipped data records are unpacked, what those records show, and what of it is worth anything for `planet_core` and the backlog issues #72 to #75. This is not a proposal, not an approved design and not a decision. Nothing here is to be copied into EXO-1 code, content or data (see `docs/VISION.md`: our own code, assets, data, names and texts).

Same rule as `star-citizen-datamining.md`: read the *structure and the knobs*, not their values. Field, type and enum names below are quoted from the records and from public tooling only as evidence of how a system is cut. None of them is a suggestion for an EXO-1 name. Where a number appears it is marked **[from file]** and only explains what a knob does (often as an order of magnitude), never as a value for us.

Sources come in five kinds, marked per claim:

- **[talk]** a Hello Games conference talk. The two GDC 2017 talks have auto-generated transcripts in `research/sources/` (private, see `SOURCES-TO-CHECK.md`); `docs/FEASIBILITY.md` already summarises them for the sphere and LOD questions. The link given is the public GDC Vault page.
- **[HG]** Hello Games' own pages and patch notes, or press quoting the developers.
- **[schema]** the type definitions in MBINCompiler (`libMBIN`), the community tool that decompiles NMS data files. They show which records and fields exist in the current build. `M` below is short for `https://raw.githubusercontent.com/monkeyman192/MBINCompiler/development/libMBIN/Source/NMS/`; every file under it was fetched on 2026-10-09. Names drift between game versions.
- **[wiki]** community modding wikis (STEP).
- **[record]** a record unpacked from the installed game on 2026-10-09 (section 2). Paths are relative to `research/local/nms/exml/` (gitignored, never committed); `B/` is short for `metadata/simulation/solarsystem/biomes/`, `eco/` for `metadata/simulation/ecosystem/`. Readings marked *(inference)* come from field names, not from data.

## The short answer

- NMS is the opposite end of our scale: an unbounded number of large voxel planets, generated near the player only, with each point computed without asking its neighbours. Our 5 km radius planet is finite and baked globally first. That difference decides almost every comparison in section 4.
- The most useful thing in NMS is not a technique but the **shape of its data**: a planet is a handful of small inputs (seed, star, biome, sub-biome, size class), and everything else hangs off the biome record (terrain weights, weather weights, object spawn lists) or off parallel tables keyed by biome (creature archetypes). That is the same "recipe per planet, biome rows as data" direction `planet_core` already has.
- The records confirm and sharpen the public picture: terrain is one "uber noise" function instanced as a fixed set of named layers, plus grid primitives, features and caves, each with its own LOD cutoff; scatter separates *which list*, *where (named patch regions)* and *how much and how far (per quality tier)*; weather is base weather + one storm override + a context × severity hazard grid; fauna is a chain of weighted tables ending in role slots.
- Unpacking works natively on Linux with HGPAKtool and MBINCompiler under `mise` runtimes; the exact commands are in section 2 so the read can be repeated after a game update.

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

### 1g. Additions from the free talk recordings and later notes

GDC posted both 2017 talks free on YouTube; links below jump to the second in the recording. The captions are auto-generated, so treat spoken numbers as approximate.

- The voxel data covers only a thin band around the surface; a cheap noise "elevation" offsets the sphere radius below it, which is how high mountains and deep seas fit into a thin voxel shell. **[talk]** https://www.youtube.com/watch?v=sCRzxEEcO2Y&t=1234s
- A region is a small cube of voxels, meshed with an apron of overlap so neighbouring regions and LODs meet without stitching; lower LODs lower the vertical resolution more than the horizontal. **[talk]** https://www.youtube.com/watch?v=sCRzxEEcO2Y&t=1313s, https://www.youtube.com/watch?v=sCRzxEEcO2Y&t=1458s
- From space each planet is a separate, very cheap voxel sphere a couple of voxels high, built on system entry so every planet is always visible. **[talk]** https://www.youtube.com/watch?v=sCRzxEEcO2Y&t=1593s
- The density function runs a 2D pass per column (elevation, terrain-kind weights) and then a 3D pass (noise layers, density fading with height so nothing floats, thresholded "worms" subtracted for caves). **[talk]** https://www.youtube.com/watch?v=sCRzxEEcO2Y&t=2130s, https://www.youtube.com/watch?v=sCRzxEEcO2Y&t=2243s
- "Uber noise" is one function combining several noise bases with analytical derivatives and domain warping; its controls (slope erosion, altitude erosion, sharpness, per-octave emphasis) are themselves varied by noise across the surface. **[talk]** https://www.youtube.com/watch?v=C9RyEiEzMiU&t=1555s, https://www.youtube.com/watch?v=C9RyEiEzMiU&t=2172s. The same controls appear as fields in the shipped records (section 3a).
- Every generator stage outputs a serialisable data blob that can be dumped, reloaded and compared, for debugging and performance regressions. **[talk]** https://www.youtube.com/watch?v=sCRzxEEcO2Y&t=1904s
- Job order is purely visual: coarse LOD first, regions in front of the camera first, then spreading out. **[talk]** https://www.youtube.com/watch?v=sCRzxEEcO2Y&t=1831s
- Scatter: objects go where a noise field passes a cutoff, a second noise thins them; clustering is recursive by scale (tree, bush, small plants, pebbles). When finer terrain arrives, objects slide down to it and cross-fade from impostor to mesh. **[talk]** https://www.youtube.com/watch?v=sCRzxEEcO2Y&t=2783s, https://www.youtube.com/watch?v=sCRzxEEcO2Y&t=2871s
- Creature paths are generated with the terrain but creatures are only instantiated near the camera; paths are deterministic, so creatures are where you left them. **[talk]** https://www.youtube.com/watch?v=sCRzxEEcO2Y&t=2976s
- An automated smoke test flies to fixed planets on every build, takes screenshots and records performance. **[talk]** https://www.youtube.com/watch?v=sCRzxEEcO2Y&t=3124s
- Worlds Part I (2024) rewrote terrain meshing to "dual marching cubes" for fewer vertices, faster generation and less memory. **[HG]** https://www.nomanssky.com/worlds-part-i-update/
- Biome and terrain kind are separate axes: the community wiki lists terrain types (canyon, floating islands, alpine and so on) independently of biome, and star colour gates which biomes can occur. **[wiki]** https://nms.miraheze.org/wiki/Biome, https://nms.miraheze.org/wiki/Star_system

## 2. Unpacking the shipped data (done 2026-10-09)

The installed Steam copy was read on 2026-10-09. Only data archives with MBIN records were unpacked, no meshes, textures or audio, and the game executable was not touched. Everything lives under `research/local/nms/` in this repo, which `.gitignore` covers (`research/local/`) and which is never committed. The tools come from monkeyman192's GitHub releases; the runtimes come from `mise` and are pinned in a local `mise.toml`, nothing is installed globally.

### What was used

| Item | Version | Source |
|---|---|---|
| Game | Steam build id `25732212` (app 275850, `appmanifest_275850.acf`, last updated 2026-10-09), runs under Proton | `/mnt/games/SteamLibrary/steamapps/common/No Man's Sky/GAMEDATA/PCBANKS` |
| Archive format | `HGPAK` v2 (header bytes `HGPAK`, version 2), the format since game 5.50 | https://github.com/monkeyman192/HGPAKtool |
| HGPAKtool | 1.1.3 (2026-02-16), from PyPI in a venv, MIT | https://github.com/monkeyman192/HGPAKtool/releases/tag/1.1.3, https://pypi.org/project/hgpaktool/ |
| MBINCompiler | v7.07.0-pre1 (2026-10-08, prerelease; files carry `MBINCompiler version (7.07.0.1)`), assets `MBINCompiler-linux-dotnet10` + `libMBIN-linux-dotnet10.so` + `mapping.json`, LGPL-3.0 | https://github.com/monkeyman192/MBINCompiler/releases/tag/v7.07.0-pre1 |
| Python | 3.13.16 via `mise` | `mise install python@3.13` |
| .NET | 10.0.401 via `mise` | `mise install dotnet@10` |
| PSArcTool | not used: only for the old PSARC archives before 5.50 | https://github.com/periander/PSArcTool |

Version match: MBINCompiler is tied to a game build. v7.07.0-pre1 appeared the day before this game update and converted 1095 of 1098 records; the three that failed and the warnings are debug, UI and per-platform user settings (`gcdebugoptions.global.mbin`, `gcuiglobals.global.mbin`, `metadata/enginesettings/**`), none of them in scope. That is the practical test: if a newer game breaks the records you need, take the matching MBINCompiler release (README badges at https://github.com/monkeyman192/MBINCompiler). MBINCompiler 7.x writes `.MXML`, the successor of the older `.EXML` text format; same idea, one XML element per field.

### Which archives

| Archive | Size | Taken | Why |
|---|---|---|---|
| `NMSARC.globals.pak` | <1 MB | `*.mbin` (38 globals) | terrain, environment, sky, creature, placement, solar generation globals |
| `NMSARC.Precache.pak` | 6 MB | `metadata/*` (915 of 14,760 entries) | `metadata/simulation/solarsystem/**` (biomes, weather, voxel generator), `metadata/simulation/ecosystem/**`, `metadata/simulation/environment/**` |
| `NMSARC.MetadataEtc.pak` | 47 MB | `metadata/*` (145 of 49,948 entries) | `metadata/reality/**`, engine settings, game state defaults |
| `NMSARC.misc.pak` | <1 MB | nothing | only input bindings (`.vdf`, `.json`) and a dictionary |
| `NMSARC.EntitySceneMBIN.pak`, the `models/**` part of the others | — | nothing | scene and descriptor records of models; model data is out of scope |
| `NMSARC.Mesh*`, `NMSARC.Tex*`, `NMSARC.audio*`, `NMSARC.Shaders.pak` and the rest | — | nothing | meshes, textures, audio, shaders |

Result: 1098 `.mbin` files (78 MB) in `research/local/nms/mbin/`, 1095 `.MXML` files (537 MB) in `research/local/nms/exml/`, conversion log in `research/local/nms/convert.log`, pak listings in `research/local/nms/lists/`.

### Commands (repeat after a game update)

```sh
NMS="/mnt/games/SteamLibrary/steamapps/common/No Man's Sky/GAMEDATA/PCBANKS"
R=~/Work/exo-1-concept/research/local/nms        # gitignored
mkdir -p $R/tools $R/lists && cd $R/tools

# 0. game build (compare with the table above)
grep buildid "/mnt/games/SteamLibrary/steamapps/appmanifest_275850.acf"

# 1. runtimes via mise, pinned locally (no global activation)
printf '[tools]\npython = "3.13"\ndotnet = "10"\n' > mise.toml
mise trust -q mise.toml && mise install
mise exec -- python -m venv .venv
.venv/bin/pip install hgpaktool==1.1.3

# 2. MBINCompiler matching the game build (check the README badges first)
V=v7.07.0-pre1
gh release download $V -R monkeyman192/MBINCompiler \
  -p MBINCompiler-linux-dotnet10 -p libMBIN-linux-dotnet10.so -p mapping.json -D mbin-$V
chmod +x mbin-$V/MBINCompiler-linux-dotnet10

# 3. list the data archives (writes filenames.txt in the cwd)
cd $R/lists
for p in globals MetadataEtc misc EntitySceneMBIN Precache; do
  ../tools/.venv/bin/hgpaktool -L -p -q "$NMS/NMSARC.$p.pak" && mv filenames.txt $p.txt
done

# 4. unpack only MBIN records (no meshes, textures, audio)
cd $R
H=tools/.venv/bin/hgpaktool
$H -U -q -O mbin -f '*.mbin'     "$NMS/NMSARC.globals.pak"
$H -U -q -O mbin -f 'metadata/*' "$NMS/NMSARC.Precache.pak" "$NMS/NMSARC.MetadataEtc.pak"

# 5. MBIN -> MXML (absolute paths; the tool runs under mise's dotnet)
mise -C tools exec -- $R/tools/mbin-$V/MBINCompiler-linux-dotnet10 \
  convert -y -f -d $R/exml $R/mbin > convert.log 2>&1
tail -3 convert.log      # "N files converted", failures listed above it
```

Checks after a run: `git -C ~/Work/exo-1-concept check-ignore -v research/local/nms/exml` must name `.gitignore`, and `git status` in the concept repo must stay clean. To pin an older game build instead of waiting for a tool release, DepotDownloader can fetch a specific Steam manifest (https://github.com/SteamRE/DepotDownloader; depot and manifest ids not looked up).

Legal note, unchanged from the earlier draft: Hello Games tolerates mods and ships an official mod path (https://www.nomanssky.com/2025/02/worlds-part-ii-5-53); no EULA clause on extraction was found [unverified]. Reading for structure, keeping it local and publishing only structure matches how we handled Star Citizen.

## 3. Structure and knobs of the records

Read from the unpacked records (section 2) on 2026-10-09, with four parallel readers, one per theme, and spot checks. All paths are **[record]**, relative to `research/local/nms/exml/`; `B/` = `metadata/simulation/solarsystem/biomes/`, `eco/` = `metadata/simulation/ecosystem/`. Field names are quoted exactly (some carry a trailing space in the files, e.g. `"MinScale "`). Numbers are **[from file]**, kept to a handful, and only explain what a knob does.

### 3a. Planet selection and terrain

**The chain from star to voxel:**

```
B/biomelistperstartype.MXML      star colour -> weights per biome type (+ a "prime" table), life-level chances
B/biomefilenames.MXML            biome type -> weighted sub-type files (cGcBiomeData)
B/<type>/<type>biome.MXML        biome record: terrain layer weights, texture/tile/palette files,
                                 weather weights, object-list slots, resources, water look
metadata/simulation/solarsystem/voxelgeneratorsettings.MXML
                                 terrain archetypes, each a Min/Max pair of generator data
gcterrainglobals.global.MXML     sea-level presets, water ratios, resource mapping, blending, editing
```

- `B/biomelistperstartype.MXML` (`cGcBiomeListPerStarType`): per star colour a `GcBiomeList` with `BiomeProbability.<Biome>` and `PrimeBiomeProbability.<Biome>`; some biome types exist only in the prime table. `LifeChance.{Dead,Low,Mid,Full}` rolls the life level, `ConvertDeadToWeird` turns some dead rolls into odd biomes.
- `B/biomefilenames.MXML` (`cGcBiomeFileList`): `BiomeFiles.<Biome>.FileOptions[]` with `SubType`, `Filename`, `Weight`, `PurpleSystemWeight`. Also `CommonExternalObjectLists` (object lists every planet gets) and `OptionalExternalObjectLists` (conditional layers, see 3c), and whitelists `ValidStartPlanetBiome`, `ValidGiantPlanetBiome`.
- `metadata/simulation/solarsystem/voxelgeneratorsettings.MXML` (`cTkVoxelGeneratorSettingsArray`): 31 `TerrainSettings` entries, ten terrain kinds in a normal, a "prime" and a "purple" family, each with `Min` and `Max` `TkVoxelGeneratorData`. Min and Max share the structure and differ in shape parameters (widths, heights, remap windows, slope terms, octaves, sea level), so *(inference)* a planet picks or interpolates within the range. The families differ mostly in scale: the prime family has several times the vertical relief of the normal one.
- `gcterrainglobals.global.MXML` (`cGcTerrainGlobals`): `TerrainPrimeIndexStart` and `TerrainPurpleSystemIndexStart` split the archetype list into families; `gcdebugoptions.global.MXML` exposes the same selection points as overrides (`ForceTerrainType`, `ForcePrimeTerrain`, `ForceSeaLevel`, `ForcePlanetsToHaveNoCaves`, `ForceBiomeMaintainsTerrain`).

**One generator record (`TkVoxelGeneratorData`):**

- Globals: `BaseSeed`; water `SeaLevel`, `BeachHeight`, `NoSeaBaseLevel`, `WaterFadeInDistance`; caves `MinimumCaveDepth`, `CaveRoofSmoothingDist`, `MaximumSeaLevelCaveDepth`; building pads `BuildingSmoothingRadius`, `BuildingSmoothingHeight`, `BuildingTextureRadius` (terrain is smoothed and retextured under generated buildings); material channels `BuildingVoxelType`, `ResourceVoxelType`.
- Four layer lists with a fixed set of named slots, identical in every archetype:

| List | Template | Slots | Role |
|---|---|---|---|
| `NoiseLayers` | `TkNoiseUberLayerData` | Base, Hill, Mountain, Rock, UnderWater, Texture, Elevation, Continent | fractal layers from boulder to continent scale (widths span about five orders of magnitude) |
| `GridLayers` | `TkNoiseGridData` | Small, Large, seven resource slots | primitives on a cell grid: boulders, and buried ore bodies |
| `Features` | `TkNoiseFeatureData` | River, Crater, Arches, ArchesSmall, Blobs, BlobsSmall, Substance | tubes and blobs, added or carved |
| `Caves` | `TkNoiseCaveData` | Underground → `Mouth`, `Tunnel` | subtractive tunnels with entrances |

- **One noise layer** (`TkNoiseUberLayerData`): scale `Height`, `Width`; where it applies `RegionRatio`, `RegionScale`, `RegionGain` *(inference: a low-frequency mask over the planet)*; combination `Active`, `Subtract`, `VoxelType`, `Offset.OffsetType` (stack on base, on all, or on zero), `HeightOffset`, `SmoothRadius`, `SeedOffset`; terraces `PlateauStratas`, `PlateauSharpness`, `PlateauRegionSize`; `WaterFade` (only the underwater layer fades below water); `MaximumLOD` (the coarsest LOD that still evaluates the layer); `TileBlendMeters`.
- **The uber noise core** (`NoiseData`, `TkNoiseUberData`): `Octaves`, `Lacunarity`, `Gain`, `SlopeGain`, `SlopeBias`, `SharpToRoundFeatures`, `AmplifyFeatures`, `PerturbFeatures` (domain warp), `AltitudeErosion`, `RidgeErosion`, `SlopeErosion`, and a remap window `Remap From Min/Max` → `Remap To Min/Max`. Every layer configures this same function; these are the controls from the 2017 talk (section 1g).
- **Grid primitives** (`TkNoiseGridData`): `NoiseGridType` (sphere, cube, cone, puck, torus, superformula, random superprimitive), size ranges `MinWidth`/`MaxWidth`, `MinHeight`/`MaxHeight`, `MinHeightOffset`/`MaxHeightOffset`, density `RegionRatio`/`RegionScale`, orientation `Yaw`/`Pitch`/`Roll` plus `Vary*`, a nested `TurbulenceNoiseLayer` for surface roughness, shape parameters `SuperFormula1/2`, `SuperPrimitive`. Resource slots are the same type buried under the surface by a negative offset, with a cell ratio in the order of a thousandth: ore bodies are rare cells in the same voxel field.
- **Features and caves** (`TkNoiseFeatureData`): `FeatureType` (tube or blob), `Width`, `Height`, `Octaves`, `RegionSize` (spacing), `Ratio`, `HeightVarianceAmplitude`/`HeightVarianceFrequency` (how much a tunnel wanders up and down), `Trench` (the river is a trench), `Subtract`, `MaximumLOD`. Which features are `Active` is what makes an archetype read as "arches" or "craters".
- **Material channel** (`TkNoiseVoxelTypeEnum`): every layer writes one of a few material values (base, mountain, rock, random rock, cave, three substances). The biome's tile types and the object lists key on those (3c), so material is the link from shape to surface look and scatter.

**The biome's terrain part** (`B/lush/lushbiome.MXML`, `cGcBiomeData`): `Terrain` (`GcTerrainControls`) has one weight per named slot above (`NoiseLayers.Base`, `Features.River`, `Caves.Underground` and so on) plus `WaterActiveFrequency`, `HighWaterActiveFrequency`, `RockTileFrequency`, `SubstanceTileFrequency`, `ForceContinentalNoise`. The biome does not own noise; it switches and reweights the archetype's layers by name.

**Sea level and water** (`gcterrainglobals.global.MXML`): `SeaLevelMoon`, `SeaLevelStandard`, `SeaLevelHigh`, `SeaLevelWaterWorld`, `SeaLevelGasGiant` presets per planet class, plus `MinWaterRatio`/`MaxWaterRatio` and the high-water family (`MinHighWaterRatio`, `MinHighWaterRegionRatio`, `Min/MaxHighWaterLevel`). Sea level is a preset per class plus a range per archetype, not a free noise.

**Resources** (`gcterrainglobals.global.MXML`, `metadata/simulation/scanning/regionhotspotstable.MXML`): `MiningSubstanceBiome` maps a biome type to a substance id, `MiningSubstanceStar`/`...Extreme`/`...Rare` map a star colour; the ids point into `metadata/reality/tables/nms_reality_gcsubstancetable.MXML`. Hotspots sit on a pole grid (`RegionHotspotsPoleSpacing`, `RegionHotspotsPerPoleMin/Max`) with categories and class weights.

**Planet size** (`metadata/simulation/scanning/planetarymappingtable.MXML`, `cGcPlanetaryMappingTable`): no record sets a radius in metres. Size is an enum (moon to giant) mapped to `SectionPerSide` and `PolesPerSection`, *(inference)* a cube-sphere grid of map sections that grows with size. `gcgalaxyglobals.global.MXML` `PlanetRadii` are galaxy-map display values. `gcsolargenerationglobals.global.MXML` sets system layout: `Solar System Size`, per-planet angle and elevation ranges, `ExtremePlanetChance` per star colour, `PlanetRingProbability`, asteroid fields.

**Surface look** (`metadata/simulation/solarsystem/textures/lush.MXML`, `B/lush/lushtiletypes.MXML`, `metadata/simulation/solarsystem/colours/basecolourpalettes.MXML`): a terrain texture record holds one atlas and a fixed number of tile settings (`TextureConfig`: `Brightness`, `Contrast`, `Specular`); tile-type sets (`cGcTileTypeSets`) give each tile slot a palette channel (`TkPaletteTexture`: `Palette`, `ColourAlt`, `Index`); palette lists (`cGcPaletteList`) hold named channels (grass, rock, sand, cave, sky, water ...) with `NumColours`, and a biome palette overrides only the channels it marks active *(inference)*. `gcsimulationglobals.global.MXML` `PlanetTerrainMaterials` has one terrain material per LOD level.

**Gas giants** (`B/gasgiants/gasgiantstandardbiome.MXML`, `metadata/simulation/solarsystem/atmosphere/gasgiantatmospherelist.MXML`): same biome template with some layers off, own tile types, plus an atmosphere list (`AtmosphereID`, `GradientMapResource`) and flow knobs in `gcsimulationglobals.global.MXML` (`GasGiantFlowSpeed`, `GasGiantFlowStrength`).

### 3b. Biomes

- A biome record (`B/<type>/<type>biome.MXML`, `cGcBiomeData`, 103 files) bundles: linked files (`TextureFile`, `OverlayFile`, `TileTypesFile`, `ColourPaletteFile`), the terrain weights (3a), resources (`MiningSubstance1..3`, `FuelMultiplier`), water look (`Water`: `ColourIndex`, `WaterEmission`, `FoamEmission`, `Murkyness`), weather weights per star colour (`WeatherOptions.<Star>.WeatherWeightings.<Weather>`, 3d), clouds (`CloudSettings`: coverage, variance, rate of change, `TendencyTowardsBeingCloudy`), `DarknessVariation`, weighted screen filters (`FilterOptions`), and the scatter slots (`ExternalObjectLists`, 3c).
- **What varies between biome types** (compared across `B/lush/lushbiome.MXML`, `B/barren/barrenbiome.MXML`, `B/weird/bonespire/bonespirebiome.MXML`, `B/weird/hexagon/hexagonbiome.MXML`, `B/hugeprops/hugelush/hugelushbiome.MXML`): the texture, palette and tile-type files; which terrain slots are on (odd biomes switch off the underwater layer and damp the grid layers); water frequency; cloud coverage; and above all the set of object-list slots. Odd biomes drop life levels and use one bespoke object list.
- **Sub-types instead of new biomes**: variants (`Standard`, quality tiers, `Variant_A..D`, huge-prop and remix types, odd shapes) are rows in `biomefilenames.MXML` with a weight, all pointing at records of the same template. A new look is a new row, not a new system.
- **Caves and sea floor are not biomes.** `B/cave/` and `B/underwater/` hold only object lists; every planet gets them through `CommonExternalObjectLists`, bound to the tile types `Cave` and `Underwater` (`B/biomefilenames.MXML`). "Biome" means the surface; underground and underwater are layers keyed by material.

### 3c. Scatter

```
biome slot (ExternalObjectLists[]: Name, TileType, ChooseUsingLifeLevel, Options[])
  -> object list file  B/<type>/<type>objects{dead,low,mid,full}.MXML, B/objects/**   (cGcExternalObjectList)
       Objects: Landmarks[] / Objects[] / DetailObjects[] / DistantObjects[] / Creatures[]
         -> GcObjectSpawnData  (one placement rule)
              Placement name -> B/placementvalues/spawndensitylist.MXML  (patch noise)
              QualityVariants[_id] -> metadata/simulation/environment/biomequalitytable.MXML (LOW/STANDARD/ULTRA)
+ biomefilenames.MXML  CommonExternalObjectLists / OptionalExternalObjectLists
+ gcplacementglobals.MXML  clamps and default LOD ladders
```

- **Slots** (`GcExternalObjectListOptions` in the biome): `Name`, `Probability`, `TileType` (which material the slot spawns on), `ChooseUsingLifeLevel` (pick the option by the planet's life level, else one at random), `Options[]` (object-list paths), `Order`, `SuppressSpawn`, `ResourceHint`. Life levels are separate files (`lushobjectsdead/low/mid/full`), not a field.
- **Conditional layers** (`OptionalExternalObjectLists` in `B/biomefilenames.MXML`): `ProbabilityOfBeingActive`, `MinFilesToChoose`/`MaxFilesToChoose`, and gates such as `OnlyOnExtremeWeather`, `OnlyOnDeepWater`, `NotOnDeadPlanets`, `NotOnGasGiant`, `OnlyOnBiome`, `SubBiomeProbability.<SubType>`. Per-biome flora is built by stacking such layers on top of the base slots.
- **Spawn tiers** (`GcEnvironmentSpawnData`): `Landmarks`, `Objects`, `DetailObjects`, `DistantObjects` are separate arrays with separate LOD ladders; detail objects mostly do not collide, landmarks almost always do (counted over `B/**`).
- **One placement rule** (`GcObjectSpawnData`, same field set on all 4091 rules under `B/`):
  - what: `Resource` (model path, `Seed`, `ProceduralTexture` samplers that tie model layers to palette channels), `Type` (instanced or single);
  - where: `Placement` (name of a patch region), `PlacementPriority`, `ExtraTileTypes`, slope band `MinAngle`/`MaxAngle` (bands above 90° appear only in cave and underwater lists: walls and ceilings *(inference)*), height band `MinHeight`/`MaxHeight` with `RelativeToSeaLevel`, `LargeObjectCoverage` (keep out of, or only around, big objects' footprints), `OverlapStyle`;
  - how it sits: `AlignToNormal`, `"MinScale "`/`MaxScale`, `MinScaleY`/`MaxScaleY`, `SlopeScaling`, `PatchEdgeScaling` (smaller towards patch edges), `MaxXZRotation`, `MaxYRotation`, `MaxRaise`/`MaxLower`, `MoveToGroundOnUpgrade` (re-snap when finer terrain arrives *(inference)*);
  - colour: `MatchGroundColour`, `GroundColourIndex`, `SwapPrimaryForSecondaryColour`;
  - interaction: `CollideWithPlayer`, `CollideWithPlayerVehicle`, `DestroyedByPlayerShip`, `DestroyedByTerrainEdit`, `CreaturesCanEat` (which plants fauna may graze), `ShearWindStrength` (wind sway);
  - impostor: `ImposterType`, `ImposterActivation`;
  - per quality tier (`QualityVariants[]`, `GcObjectSpawnDataVariant`): `Coverage`, `FlatDensity`, `SlopeDensity`, `SlopeMultiplier`, `LodDistances` (five steps), `MaxRegionRadius`, `MaxImposterRadius`, `FadeOutStartDistance`/`FadeOutEndDistance`. The highest tier mostly reaches further and is a bit denser; it does not swap assets (counted over `B/**`).
- **Patch regions** (`B/placementvalues/spawndensitylist.MXML`, `cGcSpawnDensityList`): only five fields per entry, `Name`, `Active`, `CoverageType` (`GridPatch` for rare, spaced finds; `SmoothPatch` for organic clumps; `Total` for everywhere), `PatchSize`, `RegionScale`. Clustering is a two-scale mask: patch size sets clump size (from a few metres to hundreds), region scale a coarser on/off pattern *(inference)*. Some rules name regions that are not in the list (typo or suffixed twins); what the engine does with them is not visible in the data.
- **Global clamps** (`gcplacementglobals.MXML`): `MinFrequency`/`MaxFrequency`, `MinDensity`/`MaxDensity`, `MinPatchSize`/`MaxPatchSize`, `MinPatchVariance`/`MaxPatchVariance`, interpolation thresholds (`High/Mid/LowInterpValue`), and four default LOD ladders (`LodDistancesDetail`, `...Object`, `...Landmark`, `...Distant`) with `MultiplyLodDistance`/`AddToLodDistance`.

### 3d. Weather

```
biome  WeatherOptions.<StarColour>.WeatherWeightings.<Weather> = weight
  -> metadata/simulation/solarsystem/weather/weatherlist.MXML   key -> weather file
       -> weather/<name>.MXML (cGcWeatherProperties)
            fog sets, one storm override, hazard grid, effect ids, hazard ids
            -> weather/weathereffects.MXML, weather/weatherhazards.MXML (cGcWeatherEffectTable)
                 -> metadata/effects/planeteffects.MXML  (impact visuals)
  hazard value -> metadata/simulation/environment/hazardtable.MXML -> protection drain, damage
storm timing: gcskyglobals.globals.MXML (global); chances: per weather file
```

- **Weather list** (`weather/weatherlist.MXML`, `cGcWeatherTable`): a fixed key → file table, plus `DefaultTemperature`, `DefaultToxicity`, `DefaultRadiation`, `DefaultSpookLevel` used when a weather does not set `Override<Hazard>` *(inference from the flags)*. Two weather files in the folder are not referenced by the table.
- **One weather** (`weather/snowweather.MXML` and siblings, all the same shape):
  - fog per context: `Fog`, `StormFog`, `ExtremeFog`, `FlightFog` (each `GcFogProperties`: `FogStrength`, `FogMax`, `HeightFogStrength`, `HeightFogOffset`, `FogHeight`, `HeavyAir` particle settings, `IsRaining`, `RainWetness` ...);
  - `Storms[]` (`GcStormProperties`): `Weighting`, its own `Fog`, `ColourModifiers`, `HazardModifiers` (temperature, toxicity, radiation, life-support drain, gravity, spook). Every shipped weather has exactly one storm; one storm type lowers gravity through its modifier (`weather/gravitystormweather.MXML`);
  - storm and extreme chances: `LowStormsChance`, `HighStormsChance`, `ExtremeWeatherChance`; clear weathers set storms to zero;
  - looks: `ExtremeColourModifiers` (per sky, horizon, sun, fog, light and cloud colour: force, offset or multiply saturation and value), `RainbowChance`, `HeavyAir[]` particle scenes, `StormFilterOptions[]`, optional own `Sky` and light-shaft blocks (`UseWeatherSky`, `UseLightShaftProperties`);
  - **hazard grid**: `Temperature`, `Toxicity`, `Radiation`, `SpookLevel`, `LifeSupportDrain`, each with six contexts `Ambient`, `Water`, `Cave`, `Storm`, `Night`, `DeepWater`, and each context a `Normal`/`Extreme` pair. "Colder at night", "worse in storms", "milder in caves", "worse in deep water" and "extreme planet" are all cells of one table;
  - links: `WeatherEffectsIds[]` (distant visuals) and `WeatherHazardsIds[]` (things that hurt or move the player).
- **Effects and hazards** (`weather/weathereffects.MXML`, `weather/weatherhazards.MXML`, both `cGcWeatherEffectTable` of `GcWeatherEffect`): when `SpawnConditions` (any time, during storm, not in storm, at night, at night and not in storm); how often `SpawnAttemptsPerRegion`, `SpawnChancePerSecondPerAttempt`, `SpawnChancePerSecondExtreme`, `ChanceOfPlanetBeingExtreme`, `MaxHazardsOfThisTypeActive`; interplay `ForcedOnByHazard` (a visual follows its hazard), `ExclusivePrimaryHazard`; placement and motion `WeatherEffectSpawnType` (single, cluster, patch), `WeatherEffectBehaviour` (static, wander), spawn distance and scale ranges, `MoveSpeed`, wander radius and arc; lifetime and fades; a reward `ImpactGift` + `ImpactGiftChance` (a strike leaves something to pick up); and typed `EffectData`: a telegraphed strike (`IndicatorDecal`, `NumFlashes`, `EarliestImpact`, `MinStrikes`/`MaxStrikes`, `FullDamageRadius`, `DamageRadius`), or a pull (`SuckInRadius`, `SuckInStrength`, `SuckUpHeight`).
- **Player hazard rules** (`metadata/simulation/environment/hazardtable.MXML`, `cGcPlayerHazardTable`): per hazard type `TriggerValue`, `CriticalValue`, `CapValue`, `ProtectionTime` (a range from trigger to cap), `DamageRate`, `RechargeTime`, `Damage` id, output scaling. The weather supplies a number, this table turns it into protection drain and damage; difficulty multipliers sit in `gcplayerglobals.global.MXML` (`NormalModeHazardTimeMultiplier`, `...RechargeUnderground`, `HardMode*`).
- **Storm timing and day/night** (`gcskyglobals.globals.MXML`, `cGcSkyGlobals`): gap between storms for a "low" and a "high" regime (`MinTimeBetweenStormsLow`/`...High` and `Max...`; minutes apart vs tens of minutes), storm length per regime, `StormWarningTime`, `StormTransitionTime`, `InFlightStormStrength` (weaker in the ship), `CreatureStormThreshold`, `TakeoffStormThreshold`; the day cycle `DayLength`, `NightThreshold`, sunset, night and fog fade windows, day/dusk/night light colours; per planet class sky and fog defaults (`PlanetProperties`, `PlanetPrimeProperties`, `PlanetGasGiantProperties`).
- **Sky colour sets** (`weather/skysettings/dayskycolours.MXML`, `duskskycolours.MXML`, `nightskycolours.MXML`, `cGcWeatherColourSettings`): `GenericSettings`, `DarkSettings` and `PerBiomeSettings`, each a weighted list of whole palettes (`SkyColour`, `HorizonColour`, `SunColour`, `FogColour`, `LightColour`, `LightColourUnderground`, `CloudColour1/2` ...); storm types can have their own day file. A planet picks a palette by weight *(inference)*.
- **Sea state follows storms** (`metadata/effects/water/waterdata.MXML`, `cTkWaterData`): named `WaterConditions` (wave sets and foam), chosen by `WaterConditionUsage.NoStorm` / `.Storm` weights.
- **Wind** (`gcenvironmentglobals.global.MXML`): `EnableWind`, `ShearWindSettings` (named wind presets incl. one for under water), cloud wind offsets in `CloudProperties`; plants read it through `ShearWindStrength` (3c).

### 3e. Fauna

```
eco/creaturegenerationdata.MXML        biome / sub-biome / system kind -> archetype ids per domain, density math
  -> eco/creaturegenerationarchetypes.MXML   archetype -> role tables (+ optional extra tables)
       -> eco/{ground,air,cave,underwater}/*.MXML   role table: slots (role, size, group, density, active time)
            -> eco/creaturedatatable.MXML           creature id -> move area, scale, rarity, role bias, tags, behaviour
                 -> eco/creaturefilenametable.MXML  creature id -> model
gccreatureglobals.MXML                  spawn radius, caps, perception, group make-up, health, steering
```

- **Entry** (`eco/creaturegenerationdata.MXML`, `cGcCreatureGenerationData`): `"BiomeSpecific "` and `"SubBiomeSpecific "` weighted archetype lists per domain (`Ground`, `Air`, `Cave`, `Water`), system-level overrides (`AbandonedSystemSpecific`, `EmptySystemSpecific`, `PurpleSystemSpecific`), a `Generic` fallback, `AirArchetypesForEmptyGround`. Density is enum math: `LifeLevelDensityModifiers` (by planet life level) × `GroundGroupsPerKm`/`AirGroupsPerKm`/`WaterGroupsPerKm`/`CaveGroupsPerKm` (groups per km for each of four density steps) × `DensityModifiers`; `HerdCreaturePenalty`. Weighting: `RoleFrequencyModifiers` (never/low/normal/high → weight) and `RarityFrequencyModifiers` (common to super rare → weight).
- **Archetypes** (`eco/creaturegenerationarchetypes.MXML`): per domain a list keyed by `Id`; each has `Tables[]` (`DensityModifier`, `File`) that are combined, and `AdditionalTables[]` (`MaxTablesToAdd`, `ChanceOfHemisphereLimit` *(inference: confine to one hemisphere)*). An ecosystem is a mix of niche tables.
- **Role table slot** (`GcCreatureRoleDescription` in `eco/ground/*.MXML` etc.): `Role` (predator, player predator, prey, passive, bird, butterfly, fish prey, fish predator), which creature (`ForceType`, `ForceID`, `RequireTag`, `Filter` on model parts; empty means procedural pick), `MinSize`/`MaxSize` (size classes), `MinGroupSize`/`MaxGroupSize`, `Density` (one of four steps), `ActiveTime` (any time, only/mostly day, only/mostly night; any time dominates), `ProbabilityOfBeingEnabled` (does this slot exist on this planet), `IncreasedSpawnDistance` (big things spawn further out). Table level: `TileType` (surface, underwater, cave), `MinScaleVariance`/`MaxScaleVariance`.
- **Creature definition** (`eco/creaturedatatable.MXML`, 85 entries): `MoveArea` (ground, air, water, space), `MinScale`/`MaxScale`, `Rarity`, `PredatorProbabilityModifier` and `HerbivoreProbabilityModifier` (how likely this body fills a predator or herbivore slot), `Tags[]` with `RarityOverride`, `OnlySpawnWhenIdIsForced`, `EcoSystemCreature`; behaviour as a polymorphic `Data[]` list of components: movement speed scales, swarm (`MinCount`/`MaxCount`, `SwarmMovementType`, `SwarmMovementRadius`), flock, hover, riding, pet, health, vocal. Role and body are separate axes: the slot says what is needed, the creature says how likely it fits.
- **Boids as data** (`eco/swarmdatatable.MXML`): per swarm `CountMin`/`CountMax`, `Coherence`, `Alignment`, `Separation`, `Spacing`, `Momentum`, `MinSpeed`/`MaxSpeed`, `StartLanded`.
- **Behaviour trees** (`eco/creaturebehaviourtrees.MXML`): a handful of trees (idle, herbivore, flying, melee, ranged ...) built from `Nodes` with `Children`, `BlackboardKey`, `SucceedWhen`/`FailWhen`.
- **Globals** (`gccreatureglobals.MXML`): population caps `MaxEcosystemCreaturesNormal`/`...Low`; spawn radius from size (`SpawnDistAtMinSize` → `SpawnDistAtMaxSize`, tens to hundreds of metres) and `DespawnDistFactor`; off-screen spawning `SpawnCameraAngleCos`, `SpawnOnscreenDist`; size-class thresholds `CreatureMedMinSize`, `CreatureLargeMinSize`, `CreatureHugeMinSize`; group make-up `GroupFemaleProportion`, `GroupBabyProportion`, `HerdGroupSizeMultiplier`; perception and flight `CreatureSightRange`, `CreatureHearingRange`, `*FleePlayerDistance`, `PredatorChargeDist`, `PredatorBoredomDistance`; steering weights `FlowFieldWeight`, `AvoidCreaturesWeight`, `FollowLeaderAlignWeight`; health per size and role. Diets are UI labels here; no record shows diet driving behaviour.
- **Biomes barely name fauna.** The `Creatures` array in object lists is empty in all but one list (`B/rocky/rockobjectsfull.MXML`), whose single hand-placed entry shows the flat runtime shape a creature resolves to: groups per km², `CreatureSpawnDistance`/`CreatureDespawnDistance`, `CreatureActiveInDayChance`/`CreatureActiveInNightChance`, `HemiSphere`, group size, rarity, `Herd`. The link from plants to fauna is `CreaturesCanEat` on placement rules (3c).

### 3f. Streaming and LOD

- **Region rings per quality** (`gcenvironmentglobals.global.MXML` `LODSettings.{Low,Medium,High,Ultra}`, `TkLODSettingsData`): six-entry arrays `RegionLODRadius` (how many regions at each LOD ring), `RegionLODHiddenRanges`, `ImposterOverrideRange`, `MaxObjectDistanceOverride`, plus `LODAdjust`, `ViewImpostersFromSpace`, `NumberOfImposterViews`.
- **Altitude stages** (same file, `EnvironmentProperties`, `EnvironmentPrimeProperties`, `EnvironmentGasGiantProperties`): `PlanetObjectSwitch`, `PlanetLodSwitch0..3`, `PlanetLodSwitch0Elevation`, atmosphere bands (`AtmosphereStartHeight`, `AtmosphereEndHeight`, `StratosphereHeight`, `CloudHeightMin/Max`), one block per planet class; fades `TerrainFadeTime`, `CreatureFadeTime`, `FloraFadeTimeMin/Max`.
- **Per-layer cutoff**: every noise, grid, feature and cave layer carries `MaximumLOD` (3a); small grids stop early, caves before surface features.
- **Budgets**: `gcgraphicsglobals.global.MXML` `TerrainBlocksPerFrameLow/Med/Hi/Ult` (terrain blocks meshed per frame per quality), terrain cache switches, `TessSettings`, `TerrainMipDistance*`, `UseImposters`; `gcterrainglobals.global.MXML` `NumGeneratorCalls`, `NumPolygoniseCalls`, `NumPostPolygoniseCalls` *(inference: jobs per stage)*.
- **Objects**: per rule `LodDistances`, `MaxRegionRadius`, `MaxImposterRadius`, fade distances, per quality tier (3c); defaults per spawn tier in `gcplacementglobals.MXML`.
- **User-facing quality** (`metadata/enginesettings/tkenginesettingsmapping.MXML`): which settings (`PlanetQuality`, `TerrainTessellation`, `WaterQuality`, `TextureStreaming`) the player can change; they select the tables above.

### 3g. Patterns in the records (structure only)

1. **Fixed slots, named once.** The generator has the same named layer slots in every archetype; biomes, debug options and LOD all address layers by those names.
2. **Ranges, not values.** Archetypes are Min/Max pairs; storms, creatures, sizes, scale and spawn distances are ranges or enum steps mapped to numbers in one global table.
3. **Enum steps mapped centrally.** Density, role frequency, rarity, size class and life level are small enums; one global record maps each step to a number, so retuning is one edit.
4. **Material as the join.** Terrain layers write a material value; tile types, object-list slots, caves and underwater key on it.
5. **Context × severity grids** for hazards instead of rules in code.
6. **Quality tiers inside a rule.** Low/standard/ultra are variants of one placement rule, not separate content.
7. **Conditional layers with gates.** Extra object lists switch on by planet properties (extreme weather, deep water, not dead, not gas giant) instead of new biomes.

## 4. Compared with `planet_core` and #72 to #75

What we have (code repo, `crates/planet_core/src/`, read on `main` 2026-10-09): a recipe per planet as data (`recipe.rs` `Recipe`, `content/planet/<id>.json`, unknown fields rejected); a global bake of the macro image (five channels: elevation, temperature, moisture, landform, weirdness; `planet.rs` `CHANNELS`), sea level, landform stamps and sites before any chunk (`planet.rs` `bake`, `landform.rs`, `site.rs`); one height function for mesh, collision, scatter and sites (`planet.rs` `height_at`: shape offset + stamps + noise bands, then site edits); one heightfield shell on a cube sphere, no voxels, no digging (decision in #62, https://github.com/aronjanosch/exo-1/issues/62); biome rows picked by nearest point in a parameter space of the shared macro fields (`planet.rs` `biome_for`, `recipe.rs` `BiomeRow`, `Climate`, #68); scatter per quadtree cell and storey as a pure function of seed and cell, with masks, clusters and a rare roll (`scatter.rs` `build_scatter`, `recipe.rs` `ScatterSpec`, `ScatterEntry`, #65); chunks built on Bevy's async pool with a capped upload per frame (`crates/exo_app/src/terrain.rs` `SPLIT_FACTOR`, `MERGE_FACTOR`, `MAX_UPLOADS_PER_FRAME`); a sky block and a water block per planet (`recipe.rs` `Sky`, `Water`, #67). The backlog issues: #72 rivers and lakes (https://github.com/aronjanosch/exo-1/issues/72), #73 fauna per biome (https://github.com/aronjanosch/exo-1/issues/73), #74 weather per planet (https://github.com/aronjanosch/exo-1/issues/74), #75 caves as instanced interiors (https://github.com/aronjanosch/exo-1/issues/75). All four are backlog stubs with a goal and a "revisit when", no acceptance criteria yet.

Verdicts are suggestions for discussion; the initiator decides.

### Terrain

- **Not worth it: voxels and dual contouring.** They are the price NMS pays for caves and overhangs (https://gdcvault.com/play/1024514/Building-Worlds-Using). We decided the planet stays a shell (#62) and caves are interiors (#75), so a heightfield keeps physics, LOD and scatter simple.
- **Already the same shape: one noise function, many named layers.** NMS instances one uber noise as fixed, named slots (3a, `voxelgeneratorsettings.MXML`); our `Band` list is the same idea with `name`, `amplitude`, `noise`, `warp` and a `stretch`/`roughness` hook (`recipe.rs` `Band`). Nothing to adopt; it confirms the cut.
- **Worth it: noise derivatives as a local slope signal.** NMS uses analytical derivatives to make detail depend on slope without neighbour queries (https://www.youtube.com/watch?v=C9RyEiEzMiU&t=1869s); the records expose it as `SlopeErosion`, `SlopeGain`, `SlopeBias` per layer (3a). Our bands are scaled by macro fields only (`recipe.rs` `Shape`); a slope term per band would be the same idea, and FastNoiseLite does not return derivatives, so it would cost a finite difference. Small, optional, only if the look harness (#63) shows uniform roughness on slopes.
- **Worth it: altitude and ridge erosion as per-band knobs.** Same family: `AltitudeErosion`, `RidgeErosion` let a layer soften with height or along ridges (3a). Our `Shape.stretch`/`roughness` splines already scale bands by a macro field; an "erode by own height" option on a band is the cheap variant. Only if a planet's peaks read too uniform.
- **Worth it, check it: a maximum LOD per layer.** NMS skips fine layers at coarse LODs (`MaximumLOD` on every layer, 3a, 3f). Worth checking whether our chunk build evaluates every band at every quadtree depth; a per-band cutoff by chunk size is a cheap win if the perf harness (#19) points there.
- **Worth noting: terraces as a band option.** `PlateauStratas`, `PlateauSharpness` (3a) quantise a layer into steps. Our mesa and escarpment stamps cover the big cases; a terrace option on a band would add the small ones. Not needed now.
- **Not worth it: terrain edit buffers and editing knobs.** NMS stores player digging as deltas (`M` + `GameComponents/GcTerrainEditsBuffer.cs`, `TerrainEditing` in `gcterrainglobals.global.MXML`). No digging here (#62); our ground edits are site-authored and part of the height function (`site.rs` `apply_edits`).
- **Different on purpose: sea level.** NMS uses a preset per planet class plus a range per archetype (3a). Ours is a land fraction turned into a height by an area-weighted percentile (`recipe.rs` `SeaLevel`, `planet.rs`), which guarantees the land share; keep it.

### Biomes

- **Different on purpose: biome per place, not per planet.** NMS gives a planet one biome and varies inside it with tile types and object lists (3b). Our planets are small enough to walk across, so several biome rows per planet are the point (#68, `procedural-planet.md` section 2). Keep ours.
- **Worth it: one record per biome bundles everything that follows from it.** In NMS the biome record points at terrain weights, weather weights, object lists, palette and water look (3b). Our `BiomeRow` already carries `palette`, `tints` and `scatter` multipliers; when #73 and #74 land, their per-biome parts should hang off the same row (a fauna multiplier map like `scatter`, a weather or haze tweak) rather than build a parallel table.
- **Worth it: a biome may switch terrain layers by name.** NMS biomes reweight the generator's named layers (`GcTerrainControls`, 3a). Our bands are global per planet and scaled by macro fields; a per-row multiplier over band names, like the existing per-row scatter multipliers, would let a rim or highland row get its own roughness. Only if a biome needs a different ground shape that the splines cannot give.
- **Worth it: variants as weighted rows of the same record.** NMS adds looks as sub-type rows pointing at records of one template (3b). For us: a new look is a new biome row or a new recipe, never a new code path; already our rule ("differ through recipe data only", milestone P).
- **Worth noting: the star gates the biome table.** Not needed while we author each recipe by hand; interesting only if planets are ever generated from a system seed.
- **Worth it as a rule: new content as rolls on top, not reseeding.** NMS added biome variants without regenerating old planets (https://www.nomanssky.com/origins-update/). For us: adding a recipe field should leave existing planets the same when the field is absent. Our scatter is a pure function of seed and cell (`scatter.rs`); the same discipline matters for new layers.

### Scatter

- **Already the same: rules as data, masks, storeys, rare roll.** NMS's placement rule (3c) and our `ScatterEntry` overlap closely: slope band (`slope_deg`), height band relative to sea (`height_above_sea_m`), scale range (`scale`), align to normal (`align`), clear zones (`clear_sites` vs `LargeObjectCoverage`), patch masks (`mask` vs `Placement`), and storeys with their own ranges (`Storey` vs the landmark/object/detail/distant tiers). The research behind #65 already drew on NMS; the records confirm it.
- **Worth it: named patch regions shared by many rules.** NMS rules point at a named region with only five fields (3c); many rules reuse one region, so a forest and its undergrowth share a patch layout. Our `masks` are already named and referenced by key (`ScatterSpec.masks`); sharing one mask between a tree entry and its undergrowth gives the same effect today. Worth writing down as a recipe convention.
- **Worth it: shrink towards the patch edge.** `PatchEdgeScaling` makes instances smaller at the edge of a patch (3c). Our mask has an `edge` ramp for weight; using the same ramp for scale is a few lines in `build_scatter` and gives forest edges for free. Candidate if forests read as cut out.
- **Worth it: separate flat and slope density.** `FlatDensity` and `SlopeDensity` per rule (3c) instead of only a slope cut-off. We filter by `slope_deg` only; a density falloff with slope would soften the hard line. Small; only if the look harness shows it.
- **Not now: quality tiers inside a rule.** NMS keeps low/standard/ultra variants per rule (3c). We have one quality; if a settings menu comes, a per-storey range multiplier is enough.
- **Worth noting: a "graze" flag.** `CreaturesCanEat` marks which plants fauna may approach (3c). For #73 a tag on scatter entries that critters are drawn to is the cheapest link from flora to fauna.

### Planet generation

- **Worth it, we have it: small inputs, everything derived.** NMS derives a planet from seed, star, biome, size (`M` + `GameComponents/GcPlanetGenerationInputData.cs`). Ours derives from the recipe and a seed; our recipe is bigger because it is authored, not rolled.
- **Ours is better at this scale: the global pass.** NMS cannot place budgets ("about one caldera") because each point is local; it uses an offset grid for points of interest (https://gdcvault.com/play/1024265/Continuous-World-Generation-in-No). We bake stamps and sites globally with quotas and retries (`landform.rs`, `site.rs`). Keep it.
- **Worth it: enum steps mapped centrally.** NMS maps density, rarity, role frequency and life level through small enums and one global table (3e, 3g). Our recipe uses raw numbers per entry; for a few recurring scales (scatter density, fauna density) a named step table in the recipe would make planets easier to compare and retune. Only when the number of entries grows.
- **Worth it: generated and authored treated alike.** Same as NMS (https://gdcvault.com/play/1024265/Continuous-World-Generation-in-No): our sites are kits placed by the same bake.
- **Worth it: dump every stage.** NMS serialises each generator stage for debugging and regression tests (https://www.youtube.com/watch?v=sCRzxEEcO2Y&t=1904s); our `BakeStats` and the look atlas (`look.rs`) are the same habit. Keep extending them rather than adding ad-hoc logs.

### Fauna (#73)

- **Worth it: role and body as separate axes.** NMS separates what a slot needs (role, size class, group, active time) from what a creature is (body, move area, scale, role bias) (3e). For #73: a "behaviour preset" (flock, ground critter, flee distance) separate from "mesh from a Blender script", so one goofy body can flock on one planet and hop on another.
- **Worth it: spawn entries look like our scatter entries.** Density per area, group size range, spawn and despawn distance, day/night activity, biome weight (3e; `M` + `GameComponents/GcCreatureSpawnData.cs`). #73 already says "entries per biome like the scatter (#65)"; the records confirm that shape and add two knobs our scatter lacks: a despawn distance (creatures move, props do not) and an active time (relevant once #48 is in).
- **Worth it: boid weights as data.** NMS keeps `Coherence`, `Alignment`, `Separation`, `Spacing`, `Momentum`, speed range and `StartLanded` per swarm in a table (`eco/swarmdatatable.MXML`). #73 plans boids; those weights belong in the recipe or a fauna file per flock type, not in code.
- **Worth it: spawn radius from size.** Big things spawn further out, small things close (`SpawnDistAtMinSize` → `SpawnDistAtMaxSize`, `DespawnDistFactor`, 3e). For critters that only exist near the walker, one rule from size beats a range per entry.
- **Not now: archetype → niche-table chain.** Three levels of weighted tables (3e) serve a rolled universe. With two authored planets, entries per biome row are enough; revisit if planets are rolled.
- **Not now: one base model with all parts, part swapping by descriptor.** Needs a rig system and many parts (https://stepmodifications.org/wiki/NoMansSky:Reference_Guides/DESCRIPTOR_Files). Boids with a few authored bodies (#73) are far cheaper.
- **Not needed: domain split (ground, air, water, cave).** #73 has fly and land states in one critter; a domain field is only worth it when water creatures exist.

### Weather (#74)

- **Worth it: base weather per planet plus a storm as an override.** In NMS a storm is a weight, a fog and modifiers on top of the base weather, and every shipped weather has exactly one (3d). For #74: the planet's `sky` and haze stay the base, and a storm is one block of overrides blended in over time, not a second sky model.
- **Worth it: fog per context.** NMS keeps separate fog for calm, storm, extreme and flight (3d). Our haze is one density per planet (`Sky.haze_density`); on foot and in flight want different amounts, which may show up in the look harness (#63) before weather does.
- **Worth it: storm timing as two regimes.** A planet rolls a calm or stormy regime; global ranges set gap and length per regime, plus a warning and a transition time (`gcskyglobals.globals.MXML`, 3d). For #74 that is four numbers per planet (gap and length ranges) plus warning and blend time, enough for gusts and dust.
- **Worth it: one wind for everything.** NMS drives trees, smoke, particles, fog and waves from one planet wind (https://www.nomanssky.com/2024/07/no-mans-sky-worlds-part-i/), and plants read it through a per-rule sway (`ShearWindStrength`, 3c). #74 puts wind first; one wind resource that the foliage shader, the water ripple (`Water.ripple_speed`) and later flight read is the structure to keep.
- **Worth noting: sea state follows storms.** NMS swaps wave sets by storm/no storm (`metadata/effects/water/waterdata.MXML`). Our water has `ripple_m` and `ripple_speed`; a storm override can scale them. No new system needed.
- **Worth noting: telegraphed events with a reward.** Strikes with a growing marker, a delay and something left behind (3d) are cheap and fit the goofy tone. Not part of #74's look-first scope; note for later.
- **Not now: hazards.** The context × severity grid feeds survival meters (3d). We have no survival loop; #74 says look first. If a suit resource ever comes, the grid shape (ambient, night, storm, cave, water) is the structure to copy, not per-weather code.
- **Not now: weather picked by biome weights.** With two authored planets, weather is written into the recipe, as #74 proposes.

### Caves (#75)

- **Different on purpose.** NMS caves are subtractive tunnel layers in the voxel field (`Caves.Underground` with `Mouth` and `Tunnel`, 3a), and cave content is just another object list keyed on the cave material (3b). We decided caves are instanced interiors with a room graph (#75, #62). The closest NMS structure for a room graph is the L-system building layout, where locators chain into the next piece (https://stepmodifications.org/wiki/NoMansSky:Reference_Guides/LSYSTEM_Files).
- **Worth noting: interior dressing as an object list.** NMS dresses caves with the same placement rules as the surface, keyed on the cave material, including rules for walls and ceilings (slope bands above 90°, 3c). For #75 the room tiles come from kits (`KitPiece`), and loose props inside rooms can reuse our scatter entries with a wider slope band instead of a new system.

### Rivers and lakes (#72)

- **NMS cannot, we can.** The talk names rivers as one of the things a local generator cannot do, "because one feature needs to flow into another" (https://gdcvault.com/play/1024514/Building-Worlds-Using). The records show the workaround: a `River` feature slot that is a noise "trench" (3a), not a network. Our bake is global, so mapgen4-style drainage (#72) is possible here. Correction to #72: the bake does **not** reserve a drainage channel yet; `CHANNELS = 5` holds elevation, temperature, moisture, landform and weirdness (`planet.rs`), and `procedural-planet.md` section 6 only plans the slots. NMS is not a reference for #72; mapgen4 stays the one.

### Streaming and LOD

- **Worth it: object tiers with their own ranges.** NMS streams distant objects, landmarks, objects and detail objects as separate tiers with separate LOD ladders (3c, 3f). Our scatter storeys already have their own ranges (`Storey.range_m`, `elevated_range_m`, `crates/exo_app/src/scatter.rs`). A landmark tier visible from far away is the one we lack; it matters if the playtest (#71) asks for things to walk towards.
- **Worth it: fade by distance.** NMS rules carry fade start and end distances, and the talk found distance fades less visible than time fades (https://gdcvault.com/play/1024265/Continuous-World-Generation-in-No). Our chunks swap on split and merge (`terrain.rs`); if popping shows in the look harness, this is the order to try.
- **Worth it, partly have it: staged space-to-ground switches as data.** NMS keeps LOD switch heights, atmosphere bounds and fade times as one record per planet class (3f). Ours are constants in `terrain.rs`; moving them into `content/tuning/` would match how we handle other tunables, once someone needs to tune them.
- **Already the same: generation off the main thread with a budget.** NMS caps terrain blocks per frame per quality (`TerrainBlocksPerFrame*`, 3f); `terrain.rs` builds chunks on the async compute pool and uploads at most `MAX_UPLOADS_PER_FRAME`.
- **Watch: physics and nav meshes cost more than terrain.** NMS's own finding (same talk). Our collision comes from a ring of patches (`planet.rs` `patch_heights`), not from chunks; nav meshes would arrive with #73 only if critters walk, which boids avoid.

## What EXO-1 takes from this

- **Structure ideas** above may inform discussions on #72 to #75 and on the recipe: per-biome rows as the hub, named layers and named masks reused by key, storms as one override block, fog per context, one wind, role separate from body, spawn entries shaped like scatter entries with a despawn distance and an active time, tiered streaming, distance fades, enum steps mapped centrally.
- **No values, names, texts, files or assets** from NMS enter our code or `content/`. The enum and field names quoted here are evidence, not vocabulary; our biome, weather and creature names stay with the initiator (#62 "Open").
- The unpacked records stay in `research/local/nms/` (gitignored). Nothing from there is committed or published; this note carries only structure and paths.

## Open questions / unverified

- The GDC transcripts are auto-generated and from 2017; the records are from the current build (2026-10-09). Where they disagree (for example the mesher: dual contouring in 2017, dual marching cubes since 2024), the newer source wins, but records show fields, not behaviour.
- Meanings marked *(inference)* in section 3 come from field names: Min/Max interpolation of archetypes, region masks, hemisphere limits, palette fallback, what happens to placement names missing from the density list, the unit of `MaxRegionRadius`.
- Planet radius in metres is not in the records we read; planet sizes in NMS remain player estimates (`procedural-planet.md` section 3).
- MBINCompiler v7.07.0-pre1 is a prerelease; three out-of-scope records failed to convert. Re-check after the next game update with the commands in section 2.
- The fandom NMS wiki refused access (HTTP 402) on 2026-10-09; the Miraheze mirror was used instead.
- Not verified: a Hello Games EULA clause on data extraction; the Steam depot and manifest ids.
