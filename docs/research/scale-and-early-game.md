# Planet scale and the early game

Research note, 2026-10-09. Not a decision. Structure only; numbers from other games are evidence, not proposals. Follows the playtest of the first round (`loop-feel.md`) and the initiator's question, 2026-10-09: "Ein problem auch bei unserem kleinen Planeten es wirkt recht klein alles also man fliegt einfach weg ins all. Aber vom gefühl her sollte man erst bisschen was am boden machen, kleinere Aufträge vll eine schiffslizen besorge, dann ein tutorial und einen Art prüfung, dann kann man auf dem planeten ein paar leiferungen fahren, mit einem noch recht langsamen schiff und dann skaliert das immer weiter. Sollen wir den durchmesser doch direkt erhöhen was hat SC?"

Appendices: A planet sizes in other games (web), B onboarding and ground-to-space progression (web), C what a larger radius costs in our code (read, not measured).

## Appendix A. Planet sizes

Source tags: [C] community wiki/forum/press, [O] official dev statement (relayed). Numbers are evidence, not proposals. Gaps are marked "not found". Many values come via search/fetch summaries of wiki pages; verify before relying on one.

#### 1. Sizes

| Game / body | Radius (km) | Note | Source |
|---|---|---|---|
| EXO-1 (ours) | 5 | | |
| Star Citizen Hurston | 1000 (diam. 2000; one source says 2370) | "1/6 scale"; Roberts: 2000 km in-game = 12,000 km lore [C, quoting O] | https://starcitizen.tools/Hurston , https://www.dualshockers.com/star-citizen-hurston/ |
| SC microTech | 1000 | [C] | https://starcitizen.tools/MicroTech_(planet) |
| SC ArcCorp | 800 | [C] | https://starcitizen.tools/ArcCorp_(planet) |
| SC Daymar | 295 | moon [C] | https://starcitizen.tools/Daymar |
| SC Cellin | 260 | moon [C] | https://starcitizen.tools/Cellin |
| SC Yela | 313 | moon [C] | https://starcitizen.tools/Yela |
| SC Aberdeen | 274 | moon [C] | https://starcitizen.tools/Aberdeen |
| SC Arial | 345 | moon [C] | https://starcitizen.tools/Arial |
| No Man's Sky | ~30 (largest, diam. ~60) | player estimate, not official; ~12 h to walk half way round [C] | https://steamcommunity.com/app/275850/discussions/0/2952595757891767014 |
| Space Engineers | 30-60 (diam. 60-120) | moons 9.5 (diam. 19); gravity 0.25-1.2 g; fixed, non-rotating [C] | https://spaceengineers.wiki.gg/wiki/Planets |
| Kerbal Space Program Kerbin | 600 | Earth 6,353 -> "a bit less than 1/10"; 1 g kept; atmosphere 70 km not scaled [C] | https://wiki.kerbalspaceprogram.com/wiki/Kerbin (search summary), https://forum.kerbalspaceprogram.com/topic/164573-on-the-physical-properties-of-kerbin-in-stock-122-ksp |
| Outer Wilds | ~0.3-1.7 (Brittle Hollow 308 m, Hourglass Twins ~1740 m) | astrophysicist's measurement [C]; hand-made, pole to pole in minutes | https://thephysicsmill.com/2024/09/20/an-astrophysicist-attempts-to-measure-the-physics-of-outer-wilds/ |
| Starfield | n/a | landing area is a ~1 km tile; edge = invisible wall; "kilometer-sized tiles wrapped around the planet" (Todd Howard, IGN) [O via press] | https://www.tweaktown.com/news/93080/starfield-leaks-show-planetary-exploration-isnt-as-seamless-bethesda-said/ |
| Elite Dangerous Odyssey | real size (~Earth-like, 1:1 galaxy) | no per-planet km found; settlements are "human scale" on full-size planets [C/press] | https://gamespot.com/articles/new-elite-dangerous-odyssey-dev-diary-reveals-more/1100-6481068/ |
| Empyrion | not spherical; playfields 8x4 up to 64x32 km (32-2048 km2); big ones "60+ min to circle on foot" | [C] | https://empyriongame.com/faq/ , https://www.moddb.com/news/alpha-80-out-now |

Not found: official NMS radius; Outer Wilds developer quote on why small; Squad quote on the 1/10 choice (only a community-reported rationale: keep 1 g so the beginner planet feels normal).

#### 2. How scale affects feel

- SC: Hurston's size "determined by the balance between scale and traversal times" [C/press relay of CitizenCon 2018 statement]: https://www.dualshockers.com/star-citizen-hurston/. A planet-to-planet quantum hop takes about 8 min [C], so even a 1/6 world is a trip, not a hop.
- KSP: scaling radius by 1/10 while keeping gravity at 1 g makes orbit cheap (orbital speed about 2.3 km/s). Atmosphere height was not scaled (scale height at 1/10 would leave 1% atmosphere at a few km, "not a natural experience") [C]. Lesson: you can shrink the sphere but the vertical dimension (atmosphere, sky) is what makes leaving feel like an event.
- Outer Wilds: small is the design. Every location hand-made and "fully explorable"; the game is a counterpoint to "worlds that get bigger but harder to fill with meaningful content" [C review]: https://adamnfinecup.com/2019/10/23/they-create-worlds-outer-wilds/ , https://en.wikipedia.org/wiki/Outer_Wilds . Intimate scale is also a content-density choice.
- Starfield: the opposite fix. A fixed 1 km area per landing (10-15 min run across) keeps content dense, at the cost of seamlessness and a player backlash [C]. Source above.
- NMS: Murray frames "planet-sized planets" as walkable for "days and weeks" [O interview] (https://newshub.co.nz/entertainment/no-mans-sky-director-sean-murray-e3-2015-interview-2015070211), yet measured ones are small (~60 km diam.) [C]. The feel of size comes from density of POIs and fast atmosphere exit, not km.
- Empyrion: >60 min to circle the biggest playfield on foot [C]. Content is placed per playfield, not per planet.
- Not found: explicit developer quotes on "flying off into space too easily" or on POI spacing in km. SC moons (radius 260-345 km) are the closest analogue to a "region-sized" planet.

#### 3. Keeping players on the ground / leaving as a milestone

- SC: quantum drives do not work inside atmospheres; only slow "quantum boosting" does. Quantum Enforcement zones around landing areas/stations; players must clear a few km first [C]: https://starcitizen.tools/Quantum_travel . Atmosphere is ~100 km high on Hurston; "quick transport" altitude 8.9 km [C]: https://starcitizen.tools/Hurston . Hydrogen fuel burns faster in atmosphere/hovering [C, game docs relay].
- SC player report: ~3 min to leave Hurston atmosphere with some ships, 30-60 min with large ones [C, forum, low confidence]. Leaving is a trip.
- KSP: leaving Kerbin takes a real rocket and a launch (~3.4 km/s delta-v to orbit [community figure, not fetched]); the 70 km atmosphere makes orbit a milestone.
- Space Engineers: planet gravity 0.9-1.2 g and 0.25 g moons; atmospheric thrust penalties; gravity cut-off at ~42.9 km on Earthlike [C]: https://spaceengineers.wiki.gg/wiki/Earth_Planet .
- Starfield / Empyrion: loading or playfield boundaries gate leaving (technical, not design) [C].
- Landing licences: not found in any of these games. Gating there is by ship capability, fuel, QT restriction and region.

#### 4. Delivery distances and atmospheric speed

- Not found with sources: typical SC planetary hauling distances in km or minutes, and SC atmospheric speed caps in m/s. SC pages only say speed depends on altitude/air density, SCM top speed = acceleration x mass-dependent time [C]: https://api.star-citizen.wiki/comm-links/15031 (flight model notes, official).
- Anchors available: Daymar Rally route 510 km [C, https://starcitizen.tools/Daymar]; Daymar atmosphere entry ~30 km altitude vs 3 km on a small moon [C press]; Hurston circumference ~12,500 km (radius 1000).
- NMS low flight: no sourced figure found.
- Starfield: 1 km tile = 10-15 min on foot.
- Suggested follow-up if needed: SC community hauling guides and flight-speed tests (Reddit, Spectrum) for m/s numbers.

#### Patterns

- Size ladder in games that let you land and walk: 0.3-1.7 km (Outer Wilds), 30-60 km (NMS, Space Engineers), 600 km (KSP), 260-345 km moons and 800-1000 km planets (SC). Our 5 km sits between Outer Wilds and the voxel/NMS planets, far below every other seamless ground game.
- Nobody scales gravity with radius: KSP keeps 1 g on a 1/10 planet, SE uses 0.25-1.2 g. Small radius is fine for ground feel; the horizon curves faster (this is the "small" cue).
- The deliberate trade-off is traversal time vs. content: SC picks 1/6 for "scale vs traversal"; Outer Wilds picks tiny and hand-fills; Starfield picks 1 km tiles.
- Atmosphere height is not scaled with the planet (KSP 70 km on 600 km radius; SC 100 km on 1000 km). Vertical distance to space is the main "leaving is an event" lever.
- SC makes leaving a trip by disabling quantum travel in atmosphere and near landing zones; the player must fly out manually first.
- Ship class sets the cost: big ships need 30-60 min (player report) to exit SC atmospheres, small ones about 3 min.
- Content density per km2, not planet radius, drives perceived size. The largest "walkable" planets (Empyrion 2048 km2) still take ~1 h to cross.
- Gaps in this research: no sourced SC delivery times or atmospheric m/s caps, no NMS official radius, no landing-licence analogue.

## Appendix B. Onboarding and progression

Legend: [P] = primary (developer talk, blog, official post), [S] = secondary (press, wiki, forum, guide). Source quality note: search returned mostly secondary material; several GDC talks are only linked, not excerpted. Where I only have a summary, it says so. Nothing here was verified by watching the talks.

#### 1. Onboarding and opening hours

- **Outer Wilds** [P] Mobius blog "Filling out the toolbox": https://www.mobiusdigitalgames.com/news/filling-out-the-toolbox
  One overt goal (get launch codes), optional sandbox-style training (model rocket teaches ship controls implicitly), posters as reminders instead of long tutorials, prompts show what you can do rather than what you should. Lesson: tutorial trains the open-ended nature of the game, not just buttons.
- **Outer Wilds** [P, talk, listing only] GDC 2020 "Curiosity-Driven Exploration" and "4D Level Design": https://www.gdconf.com/news/attend-gdc-and-learn-how-outer-wilds-nailed-curiosity-driven-game-design and https://gdconf.com/news/see-4d-level-design-outer-wilds-deconstructed-gdc-2020
  Only the abstracts were read. Worth watching on the GDC YouTube channel for the launch-codes structure.
- **Subnautica** [P, talk, listing only] GDC 2019 "Inside the design of Subnautica" (Charlie Cleveland): https://www.gamedeveloper.com/design/video-inside-the-design-of-i-subnautica-i-
  Abstract mentions radio signals as structure added to a sandbox and mysterious tooltips. Article does not excerpt content; vehicle chain (Seaglide, Seamoth, Prawn Suit, Cyclops) is from secondary guides, e.g. https://consolepulse.com/multiplatform/subnautica/guides/subnautica-vehicles-guide-seamoth-prawn-suit-cyclops [S].
- **Designer Notes 74, Charlie Cleveland** [P, interview, not read in full]: https://www.designer-notes.com/designer-notes-74-charlie-cleveland
- **No Man's Sky** [S] PlayStation Blog "7 things in your first 5 hours": https://blog.playstation.com/archive/2016/08/09/7-things-to-do-in-your-first-5-hours-with-no-mans-sky ; overview https://en.wikipedia.org/wiki/No_Man%27s_Sky
  Start beside a broken ship; repair, refuel, first launch is job one. Later updates made the opening a more guided path (Wikipedia, S).
- **Elite Dangerous** [S] forum/Steam threads, e.g. https://steamcommunity.com/app/359320/discussions/0/1743355067101072302 and https://forums.frontier.co.uk/goto/post?id=8008683
  Pilot training with an instructor, then a fragile starter Sidewinder. Complaints: advanced combat tutorial far harder than the game afterward, early missions need rank, one mistake loses the ship, slow economy. Lesson: exam difficulty not matched to the actual early game.
- **Star Citizen** [S] TechRaptor on Alpha 3.19 New Player Experience: https://Techraptor.net/gaming/news/star-citizen-video-shows-new-player-experience-tutorial-coming-in-alpha-319 ; wiki https://starcitizen.tools/Update%3AStar_Citizen_Alpha_3.19.0
  Original tutorial was outgrown and scrapped; 3.19 added an opt-in mission chain for the first 30 minutes in one city, plus Arena Commander flight instructor. Lesson: tutorial needs maintenance as systems change, and keep it opt-in.
- **Kerbal Space Program** [S] Quarter to Three on First Contract update: https://www.quartertothree.com/fp/2014/07/18/kerbal-space-program-finally-becomes-game/
  Squad's stated purpose of the tech tree: introduce the game gradually, avoid the "wall of content". Contracts give structured goals on top of the sandbox.
- **Space Engineers** [S] wiki "Learning to Survive" scenario: https://spaceengineers.wiki.gg/wiki/Learning_to_Survive_Scenario ; Keen support thread https://support.keenswh.com/spaceengineers2/pc/topic/51935-campaigns-vs-tutorial-scenarios
  Staged scenarios (First Jump, then Learning to Survive) instead of one tutorial; players still report gaps for core building skills.
- **Starfield** [S] Destructoid first hours: https://www.destructoid.com/starfield-opening-hours-first-impressions/ ; 80.lv on "smallness" https://origin.80.lv/articles/starfield-s-procedural-planets-are-empty-boring-by-design
  Many forced tutorials early, fun arrives late per some reviewers, while others praise action within 20 minutes. Lesson: forced stops for mining-style tutorials before the hook hurt.
- **Death Stranding** [S] Game Developer design discussion: https://www.gamedeveloper.com/design/a-design-discussion-on-death-stranding (opinion piece)
  Long cutscene-heavy opening, upgrades and fast travel story-gated around hours 7-10, vehicles useless where no roads exist. Positive side from reviews: on-foot hauling makes later vehicles and roads feel earned (review summaries, S: https://gamecritics.com/mike-suskie/death-stranding-second-opinion/).
- **Schedule I** [S] rank guides: https://dotesports.com/indies/news/schedule-1-ranks-in-order-all-ranks-listed , https://primagames.com/tips/what-does-each-level-unlock-in-schedule-1
  Rank ladder (Street Rat upward, sub-levels) unlocks items, regions and vehicles; XP counted at sleep. No developer commentary found. Useful as a structure reference for "ranks unlock the next scale".

#### 2. Licences, exams, certification

- **Gran Turismo** [S] Wikipedia: https://en.wikipedia.org/wiki/Gran_Turismo_(1997_video_game)
  Licences B/A/IA, eight tests each, increasing difficulty, gate championships; doubled as the tutorial. Yamauchi acknowledged veterans find explanatory content redundant (https://www.topgear.com/car-news/gaming/gran-turismo-boss-talks-vision-gt-cars, S).
- **Euro Truck Simulator 2 Driving Academy** [P] SCS blog: https://blog.scssoft.com/2025/03/driving-academy-truck-driving.html and https://blog.scssoft.com/2024/10/driving-academy-release.html
  Optional, free, scenario-based; deliberately no pre-recorded video tutorials, player solves each scenario unaided; five chapters of rising difficulty. SCS blog on a satisfaction/feedback display notes too much info clutters, too little loses feedback. Lesson: exam as optional skill challenge, not a gate.
- **Elite Dangerous training** (see above) shows the failure mode: exam harder than reality.

#### 3. Progression that changes the world's scale

- **Metroidvania as capability gating** [S] Game Developer "Making Sense of Metroidvania Game Design": https://www.gamedeveloper.com/design/making-sense-of-metroidvania-game-design
  Utility-gated progression: an ability unlocks areas and also has general use; traversal of old areas gets faster (shortcuts). Soft gating by difficulty or hazard rather than literal doors; never trap a player.
- **Subnautica vehicles**: depth (and so range) is the gate; each vehicle lets you reach a new band. Soft gating by pressure and hazard rather than a licence.
- **Death Stranding** counterpoint: capability gated by terrain and infrastructure, but unlocks tied to story chapters felt arbitrary (see above).
- **No Man's Sky / Outer Wilds**: first launch is the milestone; the scale jump (ground to space) is the moment players remember. Outer Wilds makes it the sole mandatory goal.

#### 4. Goal display and felt progression

- **Game Developer "The 3 ways power curves impact progression"** [S]: https://gamedeveloper.com/design/the-3-ways-power-curves-impact-progression
  Enemy power, scaling numbers, hard upgrades. Numbers alone feel hollow; hard upgrades change how you play; "once the player feels like nothing is happening" they leave. Maps well to ship upgrades that change what you can do (range, cargo, speed).
- **KSP tech tree** (above): visible tree as the gradual-introduction tool.
- **Subnautica radio signals** (above): external goals layered on a sandbox.
- **Schedule I ranks** (above): visible ladder, each rank names what it unlocks.
- Gap: no GDC talk excerpt found that directly addresses "felt progression"; worth a follow-up search of GDC Vault.

#### 5. Pitfalls

- **Tutorials**: Game Developer "No one reads text in a tutorial" https://www.gamedeveloper.com/business/no-one-reads-text-in-a-tutorial ; "6 tips for designing a game tutorial" https://www.gamedeveloper.com/game-platforms/6-tips-for-designing-a-game-tutorial [S, practitioner blogs]. Teach by doing, minimal text; high skip rates reported.
- **Gating frustration**: Death Stranding story-gated unlocks; Elite early missions behind rank; Starfield slow rollout of abilities (all above, S).
- **Co-op progression**: Subnautica 2 shares blueprints and base progress across players, per-player inventory (https://mp1st.com/news/subnautica-2-how-co-op-progression-works, S). Valheim: personal skills, but gear and biome access tied to the shared world (https://www.twoaveragegamers.com/is-valheim-actually-a-good-2-player-game-or-just-co-op-shaped/, S). Deep Rock Galactic thread (https://steamcommunity.com/app/548430/discussions/1/4932019356825840342/, S) reports friction from progression gaps. Lesson: share unlocks, keep licences personal, let a licensed player carry others.

#### Patterns that recur

- Start on the smallest scale and make the first scale jump (first launch) a named milestone; Outer Wilds, No Man's Sky and Subnautica all do this.
- One overt goal beats a checklist; extra learning is optional and sandbox-like (Outer Wilds).
- Teach by doing, minimal text; show what can be done, not what must be done.
- Exams work as optional skill challenges or as the tutorial itself (Gran Turismo, ETS2 academy); they fail when harder than the game that follows (Elite).
- Gate by capability that is useful beyond the gate (range, cargo, speed), and soft-gate by hazard or difficulty before hard locks.
- Each new capability should re-contextualise old space: old routes get faster or easier (Metroidvania, Death Stranding roads).
- Progression must change how you play, not just numbers (hard upgrades vs scaling).
- Show the next goal and what it unlocks (KSP tree, Schedule I ranks, Subnautica radio signals).
- Story-timed or arbitrary unlocks feel bad; performance- or choice-driven unlocks feel earned.
- Forced tutorial stops before the hook, and tutorials that rot as the game changes, are the common onboarding failures (Starfield, Star Citizen).
- Early fragility plus slow economy punishes beginners (Elite Sidewinder); a slow first ship needs to be fun slow.
- In co-op, share unlocks and goals, keep skill/licence per person, and never make one player wait for another to catch up.

## Appendix C. A larger radius in our code

Read-only. Branch `feat/first-round` (worktree `exo-1-kernel`, HEAD 134432d). Nothing was built or run, so every timing below is derived from code, not measured. Paths are relative to the worktree unless marked concept.

#### 1. Where the radius enters planet_core

Radius is one value per planet: `system.json` `radius` -> `Recipe::for_planet(text, seed, radius)` (`crates/planet_core/src/recipe.rs:726-729`) -> `Planet.radius` (`planet.rs:238`, `:273`). `PlanetDef.radius` is per planet (`crates/warp_core/src/system.rs:36`), so **radius can already differ per planet**; spike 11 only kept the two equal (`DECISIONS.md:30` in concept: "andere Radien ... ein anderes Feature"). `--radius=` on the CLI overrides planet 0 only (`crates/exo_app/src/lib.rs:239-241`), a quick way to try a value.

**Feature sizes are absolute metres, not relative to R.** Consequence: a bigger planet is a bigger world of the same texture, not a scaled-up Hearth.
- Noise is sampled in metres: `p32(dir) = dir * radius` as f32 (`planet.rs:296-298`), frequencies in 1/m (`hearth.json` macro: elevation 0.0002, moisture 0.0008, landform 0.0009, weirdness 0.0011; bands 0.0004..0.0067). Elevation wavelength is about 5 km, so at R=5 km the planet has a few continents; at R=60 km it has about 12 wavelengths across the radius, i.e. roughly 144x more, and smaller, land/sea patches (archipelago look). The sea level is an area percentile (`planet.rs:597-610`, `land_fraction` 0.66), so land fraction stays but the continent size does not grow. Everything (shape splines, band amplitudes, biome patch size) is per area, so per-area density is unchanged and the whole thing is just more of it. Changing R also changes the planet for the same seed (noise coordinates move).
- Landforms (stamps): fixed counts and absolute sizes (`hearth.json` landforms: needle 1, basin 1, escarpment 1, plateau 1, crater 2-4, canyon 1-2, spire 1-3, mesas 0-1, so about 9-14 in total; basin radius 1300-1600 m, escarpment 1600-2200 m, `min_separation_m` 900-2600). Placement: `landform.rs:225-260` (random direction, `where` filters on macro fields, separation in metres via `radius * acos`, `retry_limit` 12, `candidates` 4000). They are about 3 % of the great circle at 5 km but 0.4 % at 60 km: the planet's big set pieces become rare specks. A 60 km planet needs counts x(R/5)^2 (about 144x) or the signature features are 100+ km apart.
- Sites: fixed counts per kind (`hearth.json` sites: arch 1, huge_tree 1, lookout 4-6, crash 6-8, ruin 7-9, grove 5-7, about 24-32 total), separations 700-3000 m, budgets 1000 m / 3000 m (`site.rs:147-190`). Fixed count is the problem: the DECISIONS note "24 sites leave a 3.4 km worst gap" (`DECISIONS.md:100`) scales linearly with R (about 3.4 / 10 / 20 / 40 km at 5 / 15 / 30 / 60 km, at 350 m/s: 10 s to 2 min between sites). Counts must scale with R^2 to keep density (24 -> 216 / 864 / 3456 sites). The placement loop itself is count-bound, not area-bound (`candidates` 30000 tries per kind, `best_of` 24, `site.rs:150-190`), so bake cost grows only with the count. `candidates` may need raising: each try is one random direction and the farthest-point heuristic needs enough tries per site.
- Hand-placed places (`content/place/*.json`) sit by lat/lon (`bent_spoon.json`: lat 70, lon 315; its comment says "about 1.9 km from the pickup"). Angular placement means pad-to-pad distances scale with R: 1.9 km becomes 5.7 / 11 / 23 km. The `_comment` and the first job would need new numbers or new coordinates. Pad and flatten sizes stay absolute (fine).
- Scatter: local per quadtree cell, spacing and range absolute (`hearth.json` scatter storeys: ground 1.6 m / 40 m, shrub 5 m / 250 m, tree 9 m / 650-1000 m; `scatter.rs:190-240`). Density per area and cost per cell do not depend on R. Nothing sparse at ground level.
- Water/drainage thresholds are absolute metres and km^2 (`hearth.json` drainage: `river_min_catchment_km2` 2.0, widths 8-40 m, `sea_min_area_km2` 0.5, `lake_min_area_m2` 250000). Catchments scale with R^2 but the noise terrain wavelength does not, so river and lake counts grow about ~R^2 (more, not longer rivers). `river_nodes`/`lakes` lists are Vecs so no hard cap. There is a statistic with 200 m / 10 m absolute steps (`planet.rs:896`, `:937`) and fixed `WALKS = 200` (`planet.rs:~910`): fine.
- Sky/water look: `rayleigh_per_km`, `atmosphere_height` 1200 m are absolute (`sky.rs:68-93`, `:110`). The ground sky looks identical; seen from orbit the shell is 24 % of R at 5 km, 8 % at 15 km, 2 % at 60 km (Earth-like). That may be a feature (a thin atmosphere skin) or something to re-tune.

**The one hard resolution problem: the macro grid.** `recipe.macro_.resolution = 512` per cube face (`hearth.json`). Fields (elev, temp, moist, land, weird) and the drainage cut/water channel are baked there and read by bilinear lookup (`planet.rs:418-422`, `macro_cell` / `bil`). Spacing = (pi/2)*R/n (same formula as `planet.rs:784`):

| R | n=512 | n=1024 | n=2048 |
|---|---|---|---|
| 5 km | 15 m | 7.7 m | 3.8 m |
| 15 km | 46 m | 23 m | 11.5 m |
| 30 km | 92 m | 46 m | 23 m |
| 60 km | 184 m | 92 m | 46 m |

Rivers are 8-40 m wide with a bilinear carve (`CARVE` channel, `drainage.rs`), so on a 92-184 m grid a river becomes a 100-200 m wide trough or disappears; the highest macro noise (0.0011 with 2 octaves = 450-900 m wavelengths) is 2.5-5 cells wide at 184 m, i.e. aliased. To keep Hearth's 15 m you need n = 1536 / 3072 / 6144 at 15 / 30 / 60 km (see section 2).

#### 2. Bake time and memory

- No Rust `bake_ms` is recorded anywhere in the docs or tests. The tests print it (`crates/planet_core/tests/drainage.rs:30-52`, `env.rs:47`, the `planet-look` harness prints `bake_ms` at `crates/exo_app/src/look.rs:117`, `lib.rs:245`), but nothing is stored; there is no `target/` look output. Measure with `cargo run -p exo_app -- --radius=15000 --headless ...` or `planet-look` (not done, build dirs in use).
- Only Godot-era numbers exist (concept): macro bake 486-547 ms on 8 threads (`SPIKE-8-REPORT.md:23`, `research/interplanetary-travel.md:41`), target planet bake 607-787 ms in a pool thread during a 25 s flight, about 32 MB per planet (`SPIKE-11-REPORT.md:51`, `LEARNINGS.md:158`). Those predate drainage and CHANNELS 5 -> 7, so treat them as lower bounds.
- Scaling: bake cost is about linear in node count, 6*(n+1)^2. Phase 1 (macro fields + stamps + area weight) and the stats pass are threaded (`planet.rs:557-590`, `par_rows` for stats at `:620`). The drainage phase calls `drainage::drain(...)` with no thread argument (`planet.rs:742`, `drainage.rs:322`): route/erosion/lakes/river routing (sort of all nodes, priority flood, `drainage.rs:362`) are serial, with only the grid build and the ground/rain sampling threaded (`planet.rs:724-745`). Expect the drainage share to dominate and to scale about n^2 (x4 per doubling of n) with no extra cores helping. Also the sea-level percentile sorts 6*(n+1)^2 pairs (`planet.rs:595-596`).
- Memory: `macro_img` is 7 f32 channels (`planet.rs:13`): 6*513^2*28 B = 44 MB at n=512, plus the `ew` pairs (2 f32), the drainage temporaries (heights, rain, `dirs` 12 B/node, `area`, `canon` 4 B/node, `h0`), say 2-3x the image at peak. Rough image sizes: n=1024 about 177 MB, n=1536 about 400 MB, n=2048 about 706 MB, n=3072 about 1.6 GB, n=6144 about 6.3 GB. Peak during a warp is double (old and new planet resident, `LEARNINGS.md:158`).
- Practical window: n=1024 at 15 km (23 m, about 4x the bake and memory of today) is affordable; 30 km needs n=2048 for 23 m (about 16x, plausibly 10+ s serial drainage) or accept 46 m at n=1024; 60 km at any useful resolution is out of reach without changing the method (tiled or chunked macro bake, drainage only on a coarse grid with rivers as a vector overlay, or river width scaled up). The warp scenario bounds target generation by flight time (`scenario/warp.rs:308`, `gen_ms < dur*1000`) and the target planet is baked on a pool thread (`warp.rs:91-94`), so a 10-20 s bake is acceptable in game if the flight is 25 s, but not for first-load of the active planet (`env.rs:63`, blocking `bake_checked(0)`).

#### 3. Terrain, LOD, collision, camera, atmosphere

The scheme scales; the hard-coded absolutes are the issue.
- Terrain quadtree (`crates/exo_app/src/terrain.rs`): 6 root chunks of 32x32 quads built synchronously (`:257-290`), split when `dist < edge_m * 1.5` (`SPLIT_FACTOR`, `:19`, `:197`), `max_depth = round(log2(face_edge / 37))` (`:272`), so the finest chunk is about 37 m (1.15 m vertex spacing) at every R. Depth is 8 at 5 km, about 9.7 / 10.7 / 11.7 at 15 / 30 / 60 km. Chunks per level near the camera are roughly constant, so the live chunk count grows with log2(R) only (about +3-4 levels x about a dozen chunks). Angular resolution is R-independent. Each chunk costs the same (the `M*M` height_ab samples, `chunk.rs:46-70`). Chunk centres are rounded to f32 (`chunk.rs:47-50`), exact at 60 km (ulp 4 mm), with the render origin shift (threshold 1000 m, `lib.rs:104`, `origin.rs`). Concept `SPIKE-5-REPORT.md:9-11,18` measured physics and render without trouble up to R=64 km and 197 km from the origin, with the shift. Cost on the first frame: root chunks at 60 km are 94 km quads on a 32 grid (2.9 km vertex spacing), which hides the whole terrain until the lower levels arrive (async, `MAX_UPLOADS_PER_FRAME` 4, `:21`); expect more pop-in on arrival from orbit as depth increases by 3-4 levels, and there is no morphing.
- No horizon culling in the quadtree (frustum only). Fine.
- Collision ring (`ring.rs`): patch depth `ceil(log2(face_edge / 20))` (`:46`), `ring_radius` 100 m, 32x32 samples at 1 m; absolute and R-independent, cost per patch constant (3 `height_at` per sample, `planet.rs:530-537`). Scales.
- Camera: `far: 120_000.0` hard-coded (`view.rs:120`), near 0.05. Fine on the surface. From orbit at distance d the visible planet reaches sqrt(d^2 - R^2)... a 60 km planet seen from 134 km exceeds the far plane and gets clipped (the far side only; the disc stays). The orbit debug camera is `planet.centre + d * 15_000.0` (`view.rs:726`): **inside the planet for R >= 15 km**. Distance fog is `density 0.00025` exponential (`view.rs:123`, absolute), `aerial_view_lut_max_distance` 20 km (`sky.rs:136`).
- Horizon on the ground grows with sqrt(R): 2 m eye height: 141 m at 5 km, 245 m / 346 m / 490 m at 15 / 30 / 60 km; from a 100 m hill 1 km / 1.7 / 2.4 / 3.5 km. Real benefit: flat-planet feeling, less curvature, ship `horizon_follow` less needed.
- system.json values (all absolute metres, deliberately not derived, `DECISIONS.md:23`; the Hearth entry: radius 5000, `atmosphere_height` 1200, `obstruction_radius` 5400, `arrival_radius` 12000, `frame_radius` 1e6, centre distance 12.5e6): **`obstruction_radius` 5400 and `arrival_radius` 12000 must be set per planet again** (R + highest terrain + margin; R + 7 km arrival). Warp guard scenario already checks `high < obstruction_radius` and `radius + min_jump_altitude < arrival_radius` (`scenario/warp.rs:386-398`), so a wrong value fails loudly. `frame_radius` 1e6 is far above any of these radii, and the check wants `frame_radius < d/2` (fine). `jump_altitude_factor` 1.5 x `atmosphere_height` 1200 is a fixed 1.8 km above the radius (`system.rs:66-67`), unchanged by R. Angular size at 12.5 Mm: 60 km radius is 0.55 deg (one full moon), 15 km 0.14 deg: visually a good argument for larger planets.
- `Field` (gravity and atmosphere fade) is `{9.81, atmosphere_height from json, gravity_end_height 6000}` (`flight_core/src/lib.rs:44-56`, `env.rs:71`): altitude based, so unchanged. Planet gravity is the same on every radius (no R dependence).

#### 4. Flight

- Ship tuning (`content/tuning/ship.json`): `thrust_accel` 20, `boost_factor` 5, `drag_k` 0.0005, `assisted_*`, `forward_speed_curve` (clearance m -> m/s): 30 m -> 45, 150 -> 60, 600 -> 150, 1200 -> 350. In the assisted mode (hover assist, `flight_core/src/lib.rs:703-728`) speed is capped by this clearance curve, last point 350 m/s; boost scales the limit by 2.5 but caps at the curve's top (`:725-728`), so **350 m/s is the cap above 1200 m clearance and about 150 m/s at 600 m**; atmosphere density fades to 0 at `atmosphere_height` (`lib.rs:78-80`) and the drag `0.0005 * density * v^2` (`:715`) is irrelevant at those speeds. With hover assist off, above the atmosphere there is no drag and no cap (20 m/s^2, 100 with boost), the quantum drive (`engage_speed` 400 m/s, `system.json`) covers distance.
- Crossing times at assisted top speed 350 m/s (cruise 150 m/s at 600 m in parentheses):

| Distance | 350 m/s | 150 m/s |
|---|---|---|
| 10 km | 29 s | 67 s |
| 15 km (R=5 km, half circumference 15.7 km) | 45 s | 105 s |
| 50 km | 143 s | 5.6 min |
| half circumference R=15 km (47 km) | 2.2 min | 5.2 min |
| R=30 km (94 km) | 4.5 min | 10.5 min |
| R=60 km (188 km) | 9 min | 21 min |

  Add accel/brake times (3.5 s, 2.25 s). At 15-30 km radius travel between points of interest is 1-5 minutes of flying: a real "trip". At 60 km a single crossing is 9+ min unless a faster in-atmosphere cap (curve points, last y 350) or a plane-speed above the atmosphere is added; the ground is 5 m/s walk / 12 run (`walker.json`), so walking across 5 km already takes 7-17 min and across anything bigger is a vehicle-only thing.
- Landing and clearance preview use altitude and local terrain only (`lib.rs:659-684`); no R in them. The `horizon_follow` rate is `v / |p|` so it is smaller on big planets, also fine.

#### 5. Tests and scenarios that assume 5 km

Content-driven tests follow system.json; the following hard-code it:
- planet_core tests construct the recipe with a literal `5000.0`: `tests/recipe.rs:10-11` (asserts radius == 5000), `tests/look.rs:5`, `core.rs:9`, `places.rs:15`, `scatter.rs:8`, `landforms.rs:9,16-18,30,63`, `drainage.rs:13`, `sites.rs:8`, `biomes.rs:7`. They bake Hearth/Cinder at 5 km: they will keep passing (they test the 5 km planet), but they do not test a larger one, and their asserted values (for example `drainage.rs` `river_length_km > 20`, landform counts, quota misses, site gaps, `biomes.rs` quota shares) are tuned to 5 km. If the content radius changes, these tests do not follow it. They need a shared radius constant or reading `system.json`.
- flight_core tests use their own `TestPlanet`/`5000.0` model (`tests/flight.rs:39-106`, `feel.rs:14`, `boost.rs:141`, `ground_hold.rs:6`) and `warp_core/src/tests.rs:158-161`: independent of content, no change needed (they are model tests, not planet tests). `net_core` tests use 5000 as a plain value.
- exo_app scenarios that break at R >= 15 km (position given as centre + fixed offset, not as radius + altitude): `scenario/warp.rs:38` (orbit pose `centre + Y*7000`, inside the planet at R >= 7 km) and `:409` (snapshot at y 7000), `view.rs:726` (orbit camera at 15 km). Other scenarios use `radius + altitude` (`warp.rs:352`, `net.rs:413`, `space.rs:229`, `look.rs:150`) and the full scenario's "fly to space 7000 m" is altitude-based (`scenario/mod.rs:532`), so they only get slower. `scenario/warp.rs:386-398` fails by design until obstruction/arrival radii are updated.
- Time-bound scenarios: anything that flies a fixed 240 s or walks a path along the surface; the first-job distance in `content/place/bent_spoon.json` (1.9 km) and `gameplay` job line text are tied to R through lat/lon. Perf baseline (`--perf`, tolerance 50 %) and `gen_ms < flight duration` would be re-baselined with the bigger bake.
- Catch: every `cargo t` includes a headless run and bakes both planets; at n=1024+ and 15 km+ the test suite gets slower in proportion to bake time (two planets per scenario, several scenarios in parallel, memory x N processes).

#### 6. Estimate

Config-only (`content/system/system.json`, a few minutes, works today for any R, per planet):
- `radius`, `obstruction_radius` (R + terrain + margin), `arrival_radius` (about R + 7 km or more), optionally `atmosphere_height`. `--radius=` gives a quick CLI trial. Fixed scenario offsets (7000 m, 15 km orbit cam) need R <= about 6 km, so 15+ km needs the three code edits below.

Recipe config (per planet recipe, `content/planet/*.json`, no code):
- `macro.resolution` 1024-1536 (the hard cost), landform and site `count` ranges and `min_separation_m` rescaled (about x(R/5)^2 for counts, for a "denser" planet, or keep and accept sparse), noise frequencies if the larger continents are wanted (divide by R/5 to keep Hearth's shape), drainage `river_min_catchment_km2`/widths if rivers should stay visible, place coordinates and the first job's text.

Small code work (hours, each):
1. `scenario/warp.rs:38,409`, `view.rs:726`: use `radius + altitude` / `radius * k`.
2. `view.rs:120` far plane from the largest planet radius (`far >= ~2.5 R`).
3. Test radius: planet_core tests read radius from one constant or `system.json`; add one large-planet bake test.
4. (Optional) cap `gravity_end_height`, fog and aerial LUT as per-planet values if the orbit look is wrong.

Real work (days): bake for R >= 30 km. Options: make `drain()` use `threads`, drop the macro image to fewer channels or f16, bake the macro grid in tiles, or run the drainage on a coarser mesh with rivers carved at chunk time, or accept wide rivers. A 60 km planet at 23 m spacing (n=4096) is 6.3 GB for the image alone and 10-30x today's serial bake: not viable as is. Travel: in-atmosphere speed curve above 1200 m and the assisted cap (350 m/s) if 100+ km crossings should be under 3-4 min.

Risks:
- Macro resolution versus river and landform size is the main quality risk (rivers vanish or get 100+ m wide); the second is time and memory of the first bake (blocking at start, doubled during a warp).
- Content density: fixed counts make bigger planets empty (sites 40 km apart at 60 km; the signature landforms 100+ km apart), and the look changes completely because noise is absolute (same seed, different planet).
- Scatter, chunks, collision and the origin shift need nothing; far-side visibility from orbit and the 120 km far plane need attention at R >= 30 km.
- Per-planet radius: already supported end to end (system.json -> `PlanetDef` -> recipe -> `PlanetRes`), including the warp swap (planet loads on a pool thread with its own radius). Mixed radii (for example a 5 km moon and a 15-30 km main planet) only need the per-planet values above; the angular-size of a planet in the sky is already computed from the radius.
- Suggested sweet spot to try first: R = 15 km, n = 1024, sites x9 (to about 220) or fewer with accepted gaps, landform counts x3-9, obstruction 15.4 km, arrival 22 km. Flight crossing 45 s to 2 min, bake about 4x today (measure first with `--radius=15000 --headless` and `planet-look`).
