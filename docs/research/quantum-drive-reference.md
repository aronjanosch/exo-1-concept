# Quantum drive: what Star Citizen's records show (for spike 11)

Research note, 2026-10-08. Read from `gitlab.com/painlabs/SCLogistics` (branch `PU`, snapshot about 2026-04), cloned into scratch space and not stored; how to get it: `research/star-citizen-datamining.md`. Only what spike 11 needs. Other systems are covered in a separate note.

Rule: we take the **structure** (states, knobs, ratios). Numbers are quoted **[verified from the file]** to show scale, not as values to reuse. Whether read values may serve as targets is still open in `DECISIONS.md`. Where a field's meaning is not obvious from the record, it is marked **[meaning guessed]**.

## Files

| Record | What it holds |
|---|---|
| `globalquantumdriveparams/quantumdriveglobalparams.xml` | Rules for every drive: where QT is allowed, path shape, party linking, notifications, music by trip length |
| `entities/scitem/ships/quantumdrive/*.xml` (62 drives) | Per drive: `SCItemQuantumDriveParams` with a normal `params` block and a `splineJumpParams` block, heat, fuel |
| `starmap/pu/system/<system>/<body>/starmapobject.*.xml` | Per planet or moon: `StarMapQuantumTravelDataParams` (obstruction, arrival, adoption radius) |
| `squantumdriveeffecttagstemplate/*.xml`, `vfx/quantumdriveeffectsettings.xml` | Effect tags for align, spool, pinch, travel, entry and exit flash |
| `communicationname/ship_quantumdrive*.xml` | Status messages: engaged, disengaged, arrived, cooldown, disabled, emergency exit, insufficient fuel or power, interdiction warning |

## Sequence (from the state and audio names)

Pick destination → **spool up** → **align** and **calibrate** (course held within an angle) → **pre-ramp-up** → **ramp-up** (two acceleration stages) → **flight in progress** → **ramp-down** → **post-ramp-down** → **cooldown**. Abort points: spool cancel, spool fail, calibration interrupt or fail, emergency exit, interdiction.

## Knobs per drive

| Field | Range over the 62 drives | What it does |
|---|---|---|
| `driveSpeed` | 1.38e8 to 2.31e8 m/s (0.46c to 0.77c) | Top speed |
| `stageOneAccelRate` / `stageTwoAccelRate` | 1.5e6 to 9.1e6 / 9.7e6 to 2.34e7 m/s² | Two-stage ramp; at these rates top speed comes after roughly 7 to 25 s **[calculated]** |
| `spoolUpTime` | 4 s in the sampled drive | Wait before a jump can start |
| `calibrationRate`, `minCalibrationRequirement`, `maxCalibrationRequirement` | 1000; 5000; 10000 | Calibration fills at a rate up to a requirement between min and max, so about 5 to 10 s; probably longer for longer trips **[meaning guessed]** |
| `calibrationDelayInSeconds` | 1.5 | Pause before calibration starts |
| `calibrationProcessAngleLimit` / `calibrationWarningAngleLimit` | 5° / 8° | Aim within 5° to calibrate, warning beyond 8° |
| `cooldownTime` | 5.5 to 41.4 s | Lockout after a jump |
| `engageSpeed` | 1500 | Speed at the hand-over from ship flight to the drive **[meaning guessed]** |
| `disconnectRange` | 20,001 m | Party members further apart drop out **[meaning guessed]** |
| VFX thresholds (`VFXSpoolEndVelocity`, `VFXPinchMaxVelocity`, `VFXEntryFlashVelocity`, `VFXTrailStartVelocity`, `VFXTravelEffectStart/EndVelocity`, `VFXExitEffectVelocity`) | 1e6 to 4e6 m/s | **Effects are keyed to speed, not time**: the tunnel look starts and ends at velocity thresholds |
| `splineJumpParams` | top 5e5 m/s, accel 250 / 5e4 m/s², cooldown 10 s | A second, slow mode with the same fields for short hops **[meaning guessed]** |

## Global rules

| Field | Value | What it does |
|---|---|---|
| `minimumAltitudeForQuantum` | 2000 m | No jump close to the ground |
| `maximumAtmosphericPressureForQuantum` | 0.4 | No jump in thick atmosphere |
| `maxLinkingRange` | 10,000 m | Range to link ships for a group jump |
| Party audio cues | member aligned, spooled up, ready, alignment lost, dropped; all ready | **Group jump**: each ship aligns and spools, the jump goes when all are ready |
| `splineTraversalParams` | tension near/mid/far for origin and target, `tangentPlanetScalar`, `maxAlignmentToUseTangentDirection`, `rollUnderFullRotationDistance`, `rollbackParams` | **The path is a spline, not a straight line.** It leaves and arrives along the planet's tangent and rolls the ship during alignment |
| `arrivalRadiusScalar` | 1.1 | Arrival point scales with the body's arrival radius |
| Music `tripCategory` | short ≥ 2e9 m, medium ≥ 1e10 m, long ≥ 3e10 m; pre-arrival cue 5, 10, 21, 50 s | Trips are bucketed by length, and a cue warns before arrival. At about 1.7e8 m/s a 2e9 m trip cruises about 12 s, a 3e10 m trip about 3 min **[calculated]** |

## Per body

| Body | `size` (radius) | `obstructionRadius` | `arrivalRadius` | `arrivalPointDetectionOffset` | `adoptionRadius` |
|---|---|---|---|---|---|
| Planet (stanton1) | 1,000 km | 1,008.9 km | 1,050 km | 10 km | 1.28e8 m |
| Planet (stanton3) | 800 km | 807.1 km | 880 km | 20 km | 2.83e8 m |
| Moon (stanton2b) | 295 km | 299 km | 460 km | 10 km | 1.18e6 m |

Ratios **[calculated]**: obstruction is the radius plus about 1 % (terrain and atmosphere), arrival 1.05 to 1.56 radii, adoption (the zone where a ship joins the body's frame) about 130 to 350 radii for planets and 4 for this moon.

## What spike 11 takes from this

1. **State machine**: the sequence above as an enum in a `*_core` crate, each abort as its own transition, cooldown at the end.
2. **Calibration as a gauge**: it fills while the aim is within a limit, shows a warning band beyond, and resets or fails outside it.
3. **Two-stage ramp to a top speed**, then ramp-down to an exit speed at the arrival point.
4. **Spline path** leaving and arriving along the planet's tangent, not straight through. This also lets the path go around a body that would block the straight line.
5. **Per-body data**: obstruction, arrival and adoption radius in the planet registry, as ratios to the radius.
6. **Effects keyed to speed**, so the same effect works for every distance and duration.
7. **Allowed only above a minimum altitude** and outside thick atmosphere.
8. **Group jump** (linking range, all ready), which fits co-op. Probably too big for the spike: list it as a next step unless it comes cheap.
9. **Scale**: SC trips cruise about 12 s to 3 min at about 0.5c to 0.8c. With our 5 km planets (SC planets are 160 to 200 times larger) distances and speeds can shrink by the same factor and keep the same trip times. The spike agent picks our values.

Not for the spike: fuel, heat, interdiction (a separate device with charge, pulse and jammer), misfires.

## Mapped to our scale (start values for spike 11)

Scale factor `k` = our radius / SC radius = 5 km / 800 km = **1/160** (stanton3; stanton1 would give 1/200). Rule per kind of value **[calculated]**:

- **World lengths** scale with `k`. Then **speeds and accelerations scale with `k` too**, so every time and every angle stays the same. Because all lengths shrink together, the sky looks the same as in SC: a planet seen from a given number of radii away has the same angular size.
- **Times and angles** stay (spool, calibration, cooldown, pre-arrival cue, 5° and 8°).
- **Ship-scale values** stay, because our ships are not smaller (linking and disconnect range).
- **Altitude rules** follow our own atmosphere, not the ratio: 2000 m scaled would be 12.5 m, which means nothing for us.

Our current values (code repo `flight_core::Field`, `content/planet/recipe.json`): radius 5000 m, atmosphere top 1200 m, planet gravity ends at 6000 m altitude; the ship flies about 400 m/s.

| Value | SC | Kind | Ours (start value) |
|---|---|---|---|
| Distance between planets (trip buckets short / medium / long) | 2e9 / 1e10 / 3e10 m | world | **12,500 / 62,500 / 187,500 km** |
| Top speed `driveSpeed` | 1.38e8 to 2.31e8 m/s | world | **860 to 1,440 km/s** (about 1,000 km/s) |
| Stage one acceleration | 1.5e6 to 9.1e6 m/s² | world | 9,400 to 57,000 m/s² |
| Stage two acceleration | 9.7e6 to 2.34e7 m/s² | world | 61,000 to 146,000 m/s² |
| Tunnel effect thresholds (VFX velocities) | 1e6 to 4e6 m/s | world | 6,250 to 25,000 m/s |
| Spline roll distance / rollback rotation distance | 250 km / 2,450 km | world | 1.6 km / 15.3 km |
| Short-hop mode (`splineJumpParams`) top speed | 5e5 m/s | world | 3,125 m/s |
| Obstruction radius | radius + about 1 % | world, but terrain-bound | **radius + highest terrain** (our relief is a larger share of the radius than SC's) |
| Arrival radius | 1.05 to 1.56 radii | world, but atmosphere-bound | **above the atmosphere: 5000 + 1200 + margin, about 7 km (1.4 radii)**, inside SC's range |
| Adoption radius (planet frame zone) | 130 to 350 radii (planets) | world | 650 to 1,750 km; must stay below half the distance to the next planet |
| Minimum altitude, maximum pressure | 2000 m, 0.4 | atmosphere | **one rule: above the atmosphere top (1200 m)** |
| `engageSpeed` | 1500 m/s | ship | about our ship's top speed (meaning guessed) |
| Spool up / calibration delay / calibration | 4 s / 1.5 s / 5 to 10 s | time | same |
| Cooldown | 5.5 to 41 s | time | same, start low (about 5 s) for testing |
| Calibration angle / warning angle | 5° / 8° | angle | same |
| Pre-arrival cue | 5 / 10 / 21 / 50 s | time | same |
| Linking range / disconnect range | 10 km / 20 km | ship | same (2 to 4 of our radii, fine in orbit) |

What stays the same as in SC **[calculated]**: cruise about 12 s (short) to 3 min (long), ramp-up about 7 to 25 s, braking distance about 1,000 radii.

What changes for us **[calculated]**: at 1,000 km/s and 60 Hz a ship moves about 17 km per physics tick, more than three of our planet radii. Obstruction and arrival checks have to sweep along the path, never test single points. Terrain at the target has to be ready from the start of the ramp-down: about 10 s before arrival at these values.
