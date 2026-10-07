# Star Citizen: reading the shipped mechanics data

Research note, opened 2026-10-08. What is publicly available if we want to study how a shipped space game structures its flight, mining, damage and capacitor systems. This is not a proposal, not an approved design and not a decision. Nothing here is to be copied into EXO-1 code, content or data (see `docs/VISION.md`: our own code, assets, data, names and texts).

The point: read the *structure and the knobs* a professional team exposes, not their numbers. Where a number is quoted below it is **[verified from the file]**, but it is shown only to explain what the knob does, never as a value to reuse.

## The short answer

We do **not** need to unpack Star Citizen ourselves (no StarBreaker, no `unp4k`). Two public repos already hold the extracted mechanics data, both current, and one of them is exactly the "no models, only systems" view asked for:

| Repo | What it is | Licence | State |
|---|---|---|---|
| `https://gitlab.com/painlabs/SCLogistics` | Raw DataCore XML records from `Data.p4k`, **all assets stripped**, branch `PU` (also `PTU`, `TECH-PREVIEW`) | none listed (public) | active, last activity 2026-04 |
| `https://github.com/StarCitizenWiki/scunpacked-data` | Pre-parsed JSON of ships/items/components via `ScDataDumper` | no licence file | active, pushed 2026-09 |

Extraction tools exist, but are only needed if we ever want our own dump: `dolkensp/unp4k` (317★, unpacker), `diogotr7/StarBreaker` (Rust, 134★, p4k/DataCore/CHF), `octfx/ScDataDumper` (drives scunpacked-data), `StarCitizenWiki/scunpacked` (C# loader, GPL-3.0, archived).

Everything below is datamined proprietary CIG data. It is evidence about **how a team shapes a system**, nothing more.

## 1. SCLogistics — raw mechanics records

Clone (large, assets removed, so read-only reference outside our repo, e.g. `/tmp/opencode/sc-logistics`):

```
git clone https://gitlab.com/painlabs/SCLogistics.git
```

The default branch is `PU`. Top level is one directory per game system, names taken straight from CIG's record paths. The interesting ones for us:

| Directory | Records in it |
|---|---|
| `ifcs/` | flight controller and ESP: `esp_spaceships.xml`, `esp_turrets.xml`, `globalespparams.xml`, `ifcs_esp_default.xml`, `ifcsgamemodeparams_default.xml` |
| `mining/` | global params split by context: `miningglobalparamsship.xml`, `...fps.xml`, `...groundvehicle.xml`, `...ship.xml`; controller params; `mininglaserglobalparams.xml` |
| `damage/` | one macro per damage type (`damagemacro.damagelaser.xml`, `damageballistic.xml`, ..., thermal, distortion, stun, biochemical), plus armour classes (`lightarmor.xml`, `mediumarmor.xml`, `heavyarmor.xml`) and resistance macros |
| `capacitorassignment/` | ~17 input→output curves used to route power to engines/weapons/shields (e.g. `capacitorassignmentinputoutput_engines_usage_buff.xml`) |
| `actor/` | movement: stances, movement sets, `gforce/`, `externalforceresponse/`, `actorjumpfallland*.xml`, `actorslidingparam.xml`, recoil, head-tracking limits |
| `sglobaltractorbeamparams/`, `sglobalsalvagerepairbeamparams/` | tractor and salvage/repair beam globals |
| `refiningprocess/`, `harvestable/`, `resource` records | economy/industry chains |
| `vehicle/`, `vehiclecombat/`, `turret/`, `radarsystem/`, `transponder/` | ship-side systems |
| `factions/`, `reputation/`, `contracts/`, `missiondata/` | the mission and rep graph |
| `curves/beziercurves/` | reusable easing/response curves shared by the systems above |

### What the records actually teach

These are the concrete knobs a shipped flight/combat model exposes. Field names below are quoted exactly; this is the useful part — the *shape* of the model:

- **IFCS (flight controller),** in `ships.json`/`ship-items.json` and `ifcs/`:
  `ScmSpeed`, `MaxSpeed`, `BoostSpeedForward/Backward`, `LinearAccelDecay`, `AngularAccelDecay`, `TorqueImbalanceMultiplier`, `LiftMultiplier`, `DragMultiplier`, `ScmMaxDragMultiplier`, `PrecisionMinDistance`/`PrecisionMaxDistance` and `PrecisionLandingMultiplier`, plus separate `Pitch`/`Yaw`/`Roll` rates with `*Boosted` variants. A whole `Afterburner` block is a capacitor model: `CapacitorMax`, `CapacitorRegenPerSec`, `CapacitorRegenDelayAfterUse`, `CapacitorAfterburnerIdleCost`, ramp-up/down times and per-axis acceleration multipliers.
- **ESP (aim assist),** `ifcs/esp_spaceships.xml`: `trackingStrength`, `distanceFalloffStart`/`End`, `outerZoneDeg`, `innerZoneRatio`, `adsZoneMinSizeDeg`, `alignmentAngleCurve`, `dampeningMin`/`dampeningMax`, `allowPulling`. All the tuning of a soft aim-assist in one record.
- **Mining, ship context** (`miningglobalparamsship.xml`): `powerCapacityPerMass`, `decayPerMass`, `optimalWindowSize`/`optimalWindowFactor`/`optimalWindowMaxSize`, `resistanceCurveFactor`, `controlledBreakingFillRate` vs `dangerBreakingFillRate` (+ exponent), `absorbableVolumeThreshold`, `cSCUPerVolume`, `childRock*` fields for rock splitting, `gadget*` thresholds, `defaultMass`. Shows a mining loop as charge-vs-mass against a shrinking optimal window.
- **Damage:** each type is a macro with a six-channel base — `DamagePhysical`, `DamageEnergy`, `DamageDistortion`, `DamageThermal`, `DamageBiochemical`, `DamageStun`. Armour records then layer resistance multipliers/thresholds. Clean example of "damage types as data, armour as data".
- **Capacitor assignment:** the mapping is one `BezierCurve` (`inputOutputMapping`, `useLUT`) in its own record, so power routing is tuned with curves, not code.
- **Actor locomotion:** jump/fall/land params are tiny records (`jumpHeight`, `jumpDistance`, `useJump`), stances and movement sets are separate files — same "one system, one record" pattern.
- **Tractor / salvage beams:** global params with tag GUIDs plus visual/feedback params (`hitsPerSecond`, `hitDuration` for salvage). Shows gameplay rate and presentation co-located.

### Patterns worth stealing (the ideas, not the files)

1. **Every tunable is a named record, referenced by GUID.** Systems read GUID-referenced records; content never hard-codes. This is the same direction as our "content is validated data" rule.
2. **Curves are first-class shared assets.** Response/easing lives in `curves/beziercurves/` and is referenced, not re-implemented.
3. **One system = one global-params record + per-instance component records.** Global tuning and per-ship overrides are separate files.
4. **Resources are typed networks.** Item `stdItem` blocks carry `ResourceNetwork` with `Consumption`/`Generation` deltas per resource (Power, Fuel, Coolant); powerplants and coolers are just nodes on it.
5. **Damage and armour are orthogonal data.** A weapon names a damage macro; the target names a resistance macro.

## 2. scunpacked-data — pre-parsed JSON

`https://github.com/StarCitizenWiki/scunpacked-data` (~54★, updated 2026-09). No cloning needed for a quick look; raw files are fetchable, e.g.:

```
https://raw.githubusercontent.com/StarCitizenWiki/scunpacked-data/master/ship-items.json
```

Layout: `ships.json` (321 vehicles / 270 spaceships), `ship-items.json` (5,412 components), `items.json`, `blueprints.json`, `contracts/`, `factions/`, `resources/`, `starmap.json`, `labels.json` (English strings). A component's `stdItem` carries the full systems block; a ship's top-level object carries `Mass`/`MassTotal`, `Propulsion` (`ThrustCapacity` per Main/Retro/Vtol/Maneuvering in G, and `FuelUsage` per mode) and a `Systems` summary keyed by subsystem (`Shields`, `QuantumDrives`, `FlightControllers`, `Thrusters`, `PowerPlants`, `Coolers`, `Weapons`, `Mining`, `TractorBeams`, ...).

Use it for the numbers and the schema shape; use SCLogistics for the raw record structure and curves.

## 3. What EXO-1 takes from this

- **Structure/architecture ideas** in the list above may inform our design discussions (GUID-referenced tunables, shared curves, resource networks, damage×armour as data).
- **No numbers, names, texts, files or assets** from these repos enter our code or `content/`. Per `VISION.md` and `AGENTS.md`, nothing from other games is copied 1:1.
- Keep any clone in scratch space (`/tmp/opencode`), not inside the repo. If a raw source ever needs saving, it goes in `research/sources/` (private, third-party) with an origin entry in `SOURCES-TO-CHECK.md`, never published.

## Open questions / unverified

- SCLogistics lists no licence; treat it as all-rights-reserved CIG-derived data — read, do not redistribute.
- The `PU` branch snapshot is from 2026-04; Live/PTU tuning drifts every patch, so any number read is a point-in-time value.
- We have not checked whether all DataCore systems are present or whether curves are fully resolved; treat coverage as partial until sampled in the areas we care about.
- The JSON (scunpacked-data) is derived by a third-party loader, so its field names are the loader author's, not CIG's. Prefer SCLogistics when the exact record layout matters.
