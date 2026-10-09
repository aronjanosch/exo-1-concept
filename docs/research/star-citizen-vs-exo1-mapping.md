# Star Citizen records vs. EXO-1: where their implementation interests us

Research note, opened 2026-10-08. Reads our crates against the Star Citizen DataCore records (`docs/research/star-citizen-datamining.md`) and says, per area, what they built, what we have, and what is worth learning for a current or future feature.

This is a map, not a decision. Nothing below is a value to copy: numbers appear only to explain what a knob does. Per `docs/VISION.md` and `DECISIONS.md` ("Inspiration"): look, understand, reimplement the best parts, never take code, assets, data, names or texts.

## How to read a record

Almost every Star Citizen system is one XML record with a `__path` like `libs/foundry/records/<system>/<name>.xml`, referenced by GUID from other records. The recurring shape:

- **one global params record** (game-mode wide) plus **per-instance component records** that override it;
- **state filters** select which record applies (`filterByStanceState`, `filterByMotionSpeed`, ...);
- **response curves** live in their own records (`curves/beziercurves`, referenced as `useLUT="1"`).

Keep that shape in mind: it is the same "content is data, one thing per file" direction as our `content/*.json`.

## Mapping at a glance

| Our crate / file | Star Citizen system | Interest |
|---|---|---|
| `flight_core::ShipController` | `ifcs/`, per-ship `Ifcs` block | High |
| `flight_core::Field`, cabin gravity in `walker.rs` | `entities/area/gravity*.xml`, `entities/roomsystem/roomgravity.xml`, `gravitygenerator` | High |
| `walker_core::Walker`, `WalkerConfig` | `actor/stanceinfo/{speeds,dimensions}/*`, `actorstanceconfig`, `actormovementsets`, `actorslidingparam`, `playerspeedthrottle`, `actorjumpfallland*` | High |
| zero-G suit (planned) | `zerogtraversalgraph/`, `actorzerogtraversalparams`, `actor/stanceinfo/*/zerog*` | High |
| ship systems / power (planned) | `itemresourcenetwork/itemresourcenetworkglobal.xml`, per-item `ResourceNetwork` | High |
| damage / armour (future) | `damage/` macros + resistance macros | Medium |
| `planet_core` streaming, biome/day-night | `densityclasses/`, `planetdaynighttemperatureparams/`, `ssolarsystem`, `roomsystem/*atmosphere*` | Medium |
| `net_core` | (not shipped as records) `characterserializationpresets/`, `longtermpersistence/` are the closest | Low |
| content architecture (`content/*.json`) | the whole record system; `curves/`, `capacitorassignment/` | High (structure) |
| quantum travel / interplanetary (planned) | `jumppoints/`, `globalquantumdriveparams/` | Medium |
| instanced interiors (planned) | `instancedinterior/`, `transitsystem/` | Medium |

---

## 1. Assisted flight — `flight_core::ShipController`

**Ours.** One `ShipController` struct (`crates/flight_core/src/lib.rs:117`) with thrust/boost, turn and roll caps, an assist budget with a response time, a forward-speed curve keyed on terrain clearance, drag `k * density * v²`, a landing sink cap, and two booleans (`hover_assist`, `horizon_follow`). It is a single tuned controller; boost is a flat multiplier.

**Theirs.** Split across records:
- Per-ship `Ifcs` (seen in `scunpacked-data` `ship-items.json`, `FlightController` `stdItem`): `ScmSpeed`, `MaxSpeed`, `BoostSpeedForward/Backward`, `LinearAccelDecay`, `AngularAccelDecay`, `TorqueImbalanceMultiplier`, `LiftMultiplier`, `DragMultiplier`, `ScmMaxDragMultiplier`, precision-mode distances, and separate `Pitch`/`Yaw`/`Roll` with `*Boosted` values.
- `ifcs/ifcsgamemodeparams_default.xml`: game-mode flags `enableNewModel`, `enableDecoupledGliding*`, `allowDisablingIFCSCore`, `cruiseModeOnByDefault`, and four `physicsDamping` values with a transition time — i.e. a mode switch and a damping ramp.
- The afterburner is a **capacitor**, not a flat multiplier: `CapacitorMax`, `CapacitorRegenPerSec`, `CapacitorRegenDelayAfterUse`, `CapacitorAfterburnerIdleCost`, ramp-up/down, per-axis acceleration multipliers plus a separate `AfterburnerNew` block and a `NoFuelParams` fallback.
- `actor/inputdeflectiontime/ifcsinputdeflectiontime_default.xml`: `minDeflectionTime`/`maxDeflectionTime` (0.2–1 s here) with a penalty **Bezier curve** — input ramps to full deflection instead of snapping.

**Worth learning.** (a) A boost *budget with regen and an idle cost* is more expressive than `boost_factor` and would give us the afterburner feel without new state elsewhere. (b) Layering `global game-mode params` (ours: assist mode) above `per-ship numbers` (ours: controller constants) matches where we are heading with `content/`. (c) The input-deflection *curve* is the same idea as our decided "walking feel" step-off, generalised: a min/max time and a mapping curve per control axis. (d) Splitting accel decay, torque imbalance and lift as named knobs is a cleaner vocabulary than our single `assisted_accel`.

## 2. Gravity: planet field and cabin LAG — `flight_core::Field`, `walker.rs`

**Ours.** `Field` (`flight_core/src/lib.rs:35`) blurs surface gravity to zero between `atmosphere_height` and `gravity_end_height`; `PlanetEnv::gravity_at` is radial. Cabin gravity is hard-coded: in the cabin the walker is given `up = DVec3::Y` and `g = 9.81` (`crates/exo_app/src/walker.rs:193`). The decided LAG behaviour (off while landed, ~1 s ramp after take-off, always on in flight) is not modelled as data yet.

**Theirs.** Gravity is a **volume component**, not a property of the planet:
- `entities/area/gravityarea.xml` → `GravityAreaParams { active, uniform, fallOffInner, gravityMagnitude, roomBased }` plus a `gravityDirection` vector, paired with a `GravityShapeComponentParams`.
- `entities/area/gravitybox.xml` → the same with a box `size` and `filled`.
- `entities/roomsystem/roomgravity.xml` → a room entity carrying a room volume, an **atmosphere container** and an action area: gravity, air and events belong to the room, and rooms are the ship interiors.
- `entities/scitem/ships/gravitygenerator/grgn_s00_template.xml` → the generator is a normal ship item (`Type="GravityGenerator"`) on the resource network, so it can be switched and powered.
- `itemresourcenetworkglobal.xml` lists `@RN_resource_Gravity` as a first-class network resource.

**Worth learning.** Model gravity (and later atmosphere) as an **optional volume attached to a frame** — magnitude *and direction* — with the planet field as the default and cabin LAG as an override. That makes our decided "off while landed / on in flight / G to force" a small data change instead of a branch in `walker.rs`, supports upside-down ships for free, and gives the future "ship on/off" a home. `fallOffInner`/`roomBased`/`uniform` are exactly the knobs our LAG ramp needs.

## 3. Walking — `walker_core::Walker`

**Ours.** `WalkerConfig` (`crates/walker_core/src/lib.rs:48`) is walk/run speed, jump speed, floor angle, snap length, skin, slides. One capsule, one heading, move-and-slide over a `World` trait.

**Theirs.** Movement is a matrix of data records:
- `actor/stanceinfo/speeds/stand.xml`: one record with `defaultSpeed`, `walkSlowSpeed`, `walkMidSpeed`, `walkFastSpeed`, `runSlowSpeed`, `runFastSpeed`, `sprintSpeed`, `greenZoneWalkSpeed/SprintSpeed`, `aimDownSightSpeed`, `leanSpeed`, `conversationSpeed`, plus accel/rotation defaults. Separate files for crouch, prone, seated, hurt, drunk, AI, Vanduul.
- `actor/stanceinfo/dimensions/stand.xml`: collider height, ground epsilon, trace spread, pivot, **viewOffset**, weaponOffset, **headStabilization** — per stance.
- `actor/actorstanceconfig.xml` references the whole speed/dimension set; `actormovementsets.xml` transitions movement sets on status (`Drunk`, `Hurt`, `ForceStumble`) with start/end delays; `actorslidingparam.xml` is a whole slide model (max time, start/stop speed, deceleration, a slide-speed curve).
- `playerspeedthrottle/...`: analog speed selection (`defaultSpeed`, `defaultSpeedWithWeapon`, `mouseWheelSpeedStep`, `durationAccelerateToFastRun`) — a throttle between walk and sprint.
- `actorjumpfalllanddefaultparams.xml` / `actorjumpfalllandplayerparams.xml`: jump as `jumpHeight` in a tiny record.

**Worth learning.** (a) Per-stance **speed sets** and **dimensions** as two data groups is a clean expansion path for crouch/sprint/ADS without adding fields to the hot loop. (b) `viewOffset` and `headStabilization` as data answers "camera stays put on enter/leave / on tilts" — directly relevant to our decided look continuity. (c) The **speed throttle** is a concrete model for "tap W = slow step, hold = full run" that may fit better than a pure accel curve. (d) A slide as its own small record is a cheap future feature.

## 4. Zero-G suit (planned) — `zerogtraversalgraph/`, `actorzerogtraversalparams`

**Ours.** Decided but not built: in zero gravity the walker keeps its velocity and moves with suit thrusters on the ship keys (thrust 2 m/s², boost ×3, brake, roll; no fuel), free orientation.

**Theirs.**
- `actor/actorzerogtraversalparams.xml`: `zeroGLaunchParams { maxLaunchSpeed, launchRotationDuration, launchEdgeCheckRadius, launchEdgeCheckDistance, ... }`, with optional variants selected by an activation tag (different max launch speed per suit/context).
- `actor/stanceinfo/speeds/zerogtraversal.xml`: its own speed set (`defaultSpeed`, `sprintSpeed`) — zero-G is a **stance**, not a special case in the walker.
- `actor/stanceinfo/dimensions/zerogtraversal.xml`: its own collider size, view offset and a *trail-sphere* extra (a sphere that follows the pivot and expands on launch).
- `zerogtraversalgraph/playerzerogtraversalgraph.xml`: a small state graph (`Attach` / `Detach` / `Launch`) with connections, delays and view-reset flags — i.e. grabbing a surface, detaching, launching off it.

**Worth learning.** Model zero-G as a **stance with its own speeds, dimensions and a tiny state graph** rather than branching on gravity. The "launch off an edge" params and the graph are the exact shape our suit needs, and it keeps the ship-key mapping uniform. The activation-tag variants show per-context values (suit vs. EVA) without code.

## 5. Ship systems and power (planned) — `itemresourcenetwork`

**Ours.** Decided later: "ship power and other ship systems come later"; cabins already have LAG and the `ShipController` has a boost budget, but there is no resource model.

**Theirs.**
- `itemresourcenetwork/itemresourcenetworkglobal.xml`: a named set of network resources — `Power`, `Fuel`, `Coolant`, `Shield`, `Gravity`, `QuantumFuel`, `Gas` — each with `affectsItemFunctionality`, `isShared`, `itemMustBeOnline` and a composition map. Globals like `maxPowerToCoolantRatio`, `thrusterCoolantMultiplier`, `powerBaseConversionRate`.
- Per item `stdItem` carries `ResourceNetwork` with **states** (e.g. `Online`, `Offline`) and per-state **deltas** (`Consumption`/`Generation` of a resource at a rate, with `MinimumFraction`), plus a power priority.

**Worth learning.** This is the blueprint for the decided "ship on/off / more ship systems": items are **nodes** (producers/consumers) on typed networks, and a system's on/off is just its node's state. Gravity, thrusters, shields, coolers all read the same way. We can start with one network (Power) and a couple of nodes; the shape scales without new code.

## 6. Damage and armour (future) — `damage/`

**Ours.** None yet.

**Theirs.** Orthogonal data: a weapon names a **damage macro** with six channels — `DamagePhysical`, `DamageEnergy`, `DamageDistortion`, `DamageThermal`, `DamageBiochemical`, `DamageStun` (e.g. `damagemacro.damagelaser.xml`, `damageballistic.xml`); a target names a **resistance macro** with per-channel `Multiplier`/`Threshold`/`DamageCap` (`heavyarmor.xml`, `lightarmor.xml`, `undersuitarmor.xml`) plus `impactForceResistance`.

**Worth learning.** If/when we add damage, separating "what a hit is" (a typed macro) from "what a target resists" (another macro) keeps both as content and avoids per-combination code. Even without combat, environmental damage (heat, fall, distortion) fits the same two-record shape.

## 7. Planet streaming and environment — `planet_core`

**Ours.** `planet_core` bakes a recipe (noise, stamps, biomes, scatter, sites) into a `Planet`; `exo_app::terrain` streams chunk LODs and a heightfield collision ring. `recipe.json` is the data.

**Theirs.**
- `densityclasses/*.xml`: per-category streaming budgets — `clusterDetectionRadius`, `clusterUpperObjectCountDGS`, `...Persistence`, `clusterPersistenceTimeout`, `resetLifetimeOnMove`, `entityIdleBuryOnly`. One record per category (carryable, debris, NPC, spaceship, decoration, ...).
- `planetdaynighttemperatureparams/{stanton,pyro,nyx,templates}`: per-system day/night temperature, i.e. environment keyed by planet.
- `ssolarsystem/` and `starmap/`: system layout as data.
- `roomsystem/` atmosphere behaviours/states/gas composition: air as room data.
- `landingpadsize/*.xml`: pad sizes as records (`shipSize`, `groundVehicleSize`).

**Worth learning.** (a) A **density/budget class per content category** is a clean way to bound what we stream and spawn; it maps onto our scatter rules and future POIs. (b) Day/night temperature and atmosphere as **per-planet data** fits our recipe direction and our "climate is a bias, not stripe" finding. (c) Pad sizes as data are worth copying in spirit when we do authored landing sites.

### Surface records, read 2026-10-08

Sparse checkout of `harvestable/`, `densityclasses/`, `entities/environment/`, `creatures/`, `procedurallayout/` (local, gitignored). The records hold **no** terrain, biome, ecosystem or weather data; surfaces are authored in level data. What they show is the layer around the terrain. Specs built on it: code repo #65 (scatter), #70 (sites), #73 (fauna), #75 (caves).

- **Scatter is a two-level weighted pick.** `HarvestableProviderPreset` per body: `harvestableGroups` (`groupName`, `groupProbability`), each with elements (`harvestable`, `relativeProbability`, `clustering`, `geometries` tag for the visual variant). Biomes do not repeat lists: `areas` hold per-element `modifiers` (multipliers, 0 = off) over the one planet-wide list.
- **Clusters are their own preset**: `probabilityOfClustering` plus weighted shapes of `minSize/maxSize` (count) and `minProximity/maxProximity` (spacing). No radius field; extent follows from count and spacing.
- **Placement filters live on the item** (`transformParams`: `minSlope/maxSlope`, `minElevation/maxElevation`, `terrainNormalAlignment`, scale, z offset), though the shipped records leave slope and elevation at defaults.
- **Authored slots carry tags**; a tag picks a table, with fill probability and a "deepest" override as a depth/reward gradient.
- **Density classes are clutter caps**, not streaming: at most N entities of a class inside radius R, plus lifetimes, with per-location overrides.
- **Terrain-edit primitives** sorted by `sortOrder`: smoothing (`size`, `rollOff`, `strength`), push/pull (`pull`, `steepness`, `rimRadius`), rectangle (`rollOff`, `dishEffect`), a noise wrapper with seed. "Flatten under a building" is a rectangle with roll-off.
- **Planet root is a component bag** (`proceduralentity.xml`): atmosphere (pressure, temperature, humidity), weather (`maximumWindSpeed`, gusts, drop-off with elevation), harvestable provider, audio biome switch.
- **Fauna**: boids with states (`maxLinearSpeed`, rules: alignment, cohesion, separation, terrain/ocean/actor repel) and transitions (random, proximity, alerted). Biome keying by duplicated classes, no table.
- **Caves**: `ProceduralLayoutGraph`, a tag-filtered room graph (`Start`, element nodes with `MinElementsToGenerate/MaxElementsToGenerate`, `ChanceOfGeneration`, `Mandatory`, `outputLinks`), no geometry in the record.

## 8. Networking — `net_core`

**Honest note:** Star Citizen's netcode is **not** shipped as DataCore records; there is nothing here to read in the way there is for flight or mining. The closest records are `characterserializationpresets/` (what character state is serialized) and `longtermpersistence/` (what persists), plus the density classes above for streaming. Our snapshot/interpolation/clock work (`net_core`) has **no meaningful Star Citizen match** and should keep drawing on the Gaffer and Overwatch sources already in `SOURCES-TO-CHECK.md`. The one transferable idea: `characterserializationpresets` treats "what is worth sending" as an explicit, named set — a good prompt to keep our snapshot fields intentional rather than incidental.

## 9. Content and data architecture — everything

**Ours.** `content/planet/recipe.json` and future `content/*.json` against JSON Schema.

**Theirs.** One record per thing, GUID-referenced; a global params record plus per-instance overrides; state filters choosing which record applies; response curves and LUTs as their own referenced records; a tag dictionary (`tagdatabase`, `itemporttagsdictionary`); manufacturers and factions as references.

**Worth learning.** This validates our direction and offers concrete patterns to adopt where they fit: **global + instance layering**, **named response curves as shared content**, and **state filters** to pick variants. Our `Recipe` already does the global/instance split for planets; the same pattern would tidy ship and walker tuning once it becomes data.

## 10. Travel and interiors (planned) — quantum, jump points, transit

**Ours.** Decided open: "travel between planets: direction like No Man's Sky or Star Citizen, to be tried." Interiors: "large or complex interiors are instanced."

**Theirs.** `globalquantumdriveparams/quantumdriveglobalparams.xml` (`minimumAltitudeForQuantum`, `maximumAtmosphericPressureForQuantum`, spline traversal and rollback params), `jumppoints/globaljumppointparams.xml`, `transitsystem/*` (carriage, destination, gateway, manager), `instancedinterior/`.

**Worth learning.** When we pick a travel direction, the interesting parts are the **gating rules** (altitude, pressure, interdiction) and the **spline traversal with rollback**, not the visuals. `transitsystem` is a template for a managed, scheduled movement (elevators, shuttles) that could also drive large instanced interiors.

---

## What I would read first (if we do one of these next)

1. **Gravity volume + LAG** — smallest, closest to a decided feature (`DECISIONS.md`, "Cabin gravity (LAG)"). Files: `entities/area/gravityarea.xml`, `entities/area/gravitybox.xml`, `entities/roomsystem/roomgravity.xml`, `entities/scitem/ships/gravitygenerator/grgn_s00_template.xml`.
2. **Boost as a capacitor + input-deflection curve** — fits `ShipController` and our walking-feel work. Files: the `Afterburner` block in `scunpacked-data` `FlightController`, `actor/inputdeflectiontime/ifcsinputdeflectiontime_default.xml`.
3. **Stance speed/dimension sets** — the clean expansion path for crouch/sprint/zero-G. Files: `actor/stanceinfo/speeds/stand.xml`, `actor/stanceinfo/dimensions/stand.xml`, `actor/actorstanceconfig.xml`, `playerzerogtraversalgraph.xml`.
4. **Resource network** — before any "ship on/off / power" work. File: `itemresourcenetwork/itemresourcenetworkglobal.xml` plus one `stdItem` with a `ResourceNetwork`.

Follow-up proposals built on this mapping: `docs/research/early-feature-proposals.md` (LAG as data, boost capacitor, minimal HUD).

## Local copy and hygiene

Cloned for this read in scratch space only: `git clone --filter=blob:none --sparse https://gitlab.com/painlabs/SCLogistics.git` under `/tmp/opencode/SCLogistics` (assets already stripped upstream). It is not in our repo and must not be. Re-check the `PU` branch (`git sparse-checkout set <dir>`), since Live/PTU tuning moves every patch.

## Unverified / caveats

- Field and record names are CIG's or a loader author's; the JSON field names (`scunpacked-data`) are not necessarily CIG's internal names.
- The `PU` snapshot is 2026-04; any behaviour read is point-in-time.
- We have not diffed our controller against theirs quantitatively, and we should not: numbers are theirs, the structure is what we want.
