# Spike 13 report — an axis-limited flight model next to the current one

**In one line:** the axis model flies in the game next to the classic one (F7), lands and holds like it, and with its placeholder values it feels very different: one speed cap for every direction makes strafing and climbing four to five times faster, and G-safety slows a full turn at cruise speed to 17 °/s, where the classic model spins at 143 °/s and loses its speed. Which parts to keep is the initiator's call.

Branch `spike/ifcs` in the code repo (based on `main` 1c295ee, `origin/main` merged in at 54accef), last state f988387 with the freeze tag `spike/13-ifcs` (2026-10-09, after the second playtest; the first numbers below are from 9e74a73). Brief: `SPIKE-13-BRIEF.md`. Measured headless at 60 Hz with `cargo dev --headless --scenario=flight-models`, values as shipped in `content/tuning/ship_axis.json` (all placeholders). No frame timings, no windowed run: F7 is checked through the bindings and the HUD readout headless; the HUD text element itself was not looked at in a window.

## What the spike built

- `flight_core::axis` (own names and code; structure after Star Citizen's flight control records, no values taken): a second model behind `ShipController::step`, switched by `ShipController::model`. It reuses the classic model's inputs, input ramp, boost capacitor, H (assist) and C (decoupled) with its 4 s blend, drag and the ground hold.
  - Coupled: velocity goal = stick in a ball × cruise speed; boost raises the forward and backward caps.
  - Commanded acceleration = linear decay × velocity error + what keeps the goal turning with the ship. Gravity and drag are compensated.
  - Thrust is clamped per axis and direction, then to the pilot's G tolerance per direction. A rolled ship holds only if its side thrust is at least 1 g.
  - Decoupled: thrust along the stick plus compensation. Coupled and decoupled are each clamped, then blended.
  - Rotation: target rate per axis, with pitch and yaw sharing an ellipse. Angular decay and an angular acceleration cap per axis. G-safety caps pitch and yaw at speed so the turn stays inside the tolerance.
  - Precision mode: inside a clearance band, the speed along the ground and the descent are capped; the climb is not. The clearance counts less the stopping distance of the current descent, so a fast descent enters the band early (like their "velocity test").
- F7 in the game (`flight_model` in `bindings.json`, with a fallback to F7 for old files). The HUD names the model (`CLASSIC`, `AXIS`, `AXIS PRECISION` near the ground); F3 adds felt G, precision share and saturation. `ship_axis.json` hot-reloads.
- Scenario `flight-models`: every manoeuvre starts from the same pose (400 m above the start point, level, at rest, full charge), once per model, numbers side by side. 14 core tests in `crates/flight_core/tests/axis.rs`. `cargo t` and `cargo scenario` pass (exit 0) at the spike's last commit.

## Results (first version, 9e74a73)

Same pose for every manoeuvre (400 m above the start point; landing from 100 m), placeholder values. From `flight-models.txt`:

| Measure | Classic | Axis |
|---|---:|---:|
| W: time to 50 m/s (s) | 1.75 | 1.70 |
| W: speed after 15 s (m/s) | 98.5 | 150.0 |
| W: time to 90 % of it (s) | 3.12 | 4.92 |
| W: highest felt g | 3.40 | 3.22 |
| release: time to < 1 m/s (s) | 6.72 | 7.28 |
| release: distance (m) | 351 | 468 |
| X: speed at the brake (m/s) | 108.0 | 150.0 |
| X: time to < 0.5 m/s (s) | 2.65 | 5.77 |
| X: distance (m) | 142 | 332 |
| D: side speed after 6 s (m/s) | 20.0 | 68.3 |
| D: time to 90 % of it (s) | 0.85 | 5.35 |
| Space: climb after 5 s (m/s) | 15.0 | 73.1 |
| Ctrl: sink after 5 s (m/s) | 15.0 | 57.5 |
| turn: speed at the start (m/s) | 102.2 | 150.0 |
| turn: heading change in 5 s (deg) | 694 | 77 |
| turn: yaw rate at the end (deg/s) | 143.2 | 17.1 |
| turn: largest slip nose/velocity (deg) | 179.8 | 41.3 |
| turn: speed at the end (m/s) | 14.4 | 172.4 |
| turn: highest felt g | 4.73 | 2.99 |
| boost: speed after 3 s (m/s) | 125.9 | 127.7 |
| boost: top speed (m/s) | 133.7 | 159.8 |
| boost: speed after 10 s (m/s) | 105.3 | 150.5 |
| C: speed kept after the blend + 8 s (%) | 73.8 | 65.6 |
| rolled: height lost in 5 s (m) | 0.00 | 0.00 |
| land: time to touchdown (s) | 10.05 | 12.75 |
| land: sink at touchdown (m/s) | 2.00 | 2.00 |
| land: slide after touchdown (m) | 0.20 | 0.21 |
| land: time to rest (s) | 12.4 | 14.3 |

| Question | What the numbers say |
|---|---|
| 1. Acceleration and top speed | Same start (1.7 s to 50 m/s). The classic model stops at its clearance curve (98.5 m/s at 400 m); the axis model goes on to its cruise cap (150 m/s) because its cap does not depend on height. Sideways and up the difference is large: the axis model's ball cap allows 150 m/s in every direction, so D reaches 68 m/s after 6 s and is still accelerating (12 m/s² side thrust), against the classic model's 20 m/s in 0.85 s. Climb is 73 m/s against 15 m/s. |
| 2. Stopping | Coasting is similar (6.7 s and 7.3 s). The brake is twice as strong in the classic model (48 m/s² budget against 30 m/s² backward with boost): 2.7 s / 142 m against 5.8 s / 332 m, from 108 and 150 m/s. |
| 3. Turning at cruise speed | Classic: the nose turns at the full 143 °/s (694° in 5 s), the velocity cannot follow, slip reaches 180° and the assist brakes the ship to 14 m/s. Axis: G-safety caps the yaw at 4 g over the forward speed, 17 °/s at the end (77° in 5 s). Its side thrust (12 m/s²) is below what that turn needs (about 45 m/s²), so it still slips 41°, and the speed grows to 172 m/s because forward thrust keeps adding. Highest felt G: 4.7 against 3.0. With the placeholder values every thrust limit is below its G tolerance, so G-safety acts only through the turn cap, never by clipping thrust; the slip comes from the side thrust. |
| 4. Boost | Same for the first 3 s (126 and 128 m/s). Top speed 134 against 160 m/s; after the capacitor is empty, the classic model falls back to 105 m/s and the axis model to its cruise cap of 150 m/s. |
| 5. Decoupled | Speed kept after the 4 s blend plus 8 s: 74 % (classic) against 66 % (axis). Both glide on afterwards. |
| 6. Rolled on its side | Both hold their height (axis: 12 m/s² side thrust is above 1 g). The core test shows the axis ship sinking at 1.8 m/s² with 8 m/s² side thrust. |
| 7. Landing from 100 m | Both touch down at 2.0 m/s (their landing caps) and rest without sliding (0.20 m and 0.21 m). The axis model takes 2.7 s longer to touch down (12.8 s against 10.1 s). |

## Bugs found on the way (fixed in the spike)

- **Blending before limiting brakes through the whole decoupling blend.** Coupled and decoupled requests mixed first, then clamped: the coupled part (300 m/s² asked at 150 m/s) stayed saturated almost to the end of the 4 s blend, so only 41 % of the speed was left. Clamping each first, then blending, makes the damping fade linearly (66 %).
- **A push along a tilted hull slides the landed ship.** With Ctrl held on a slope, "down" is the ship's tilted down axis; its share along the ground went into the precision speed cap and slid the ship 63 m. The classic model's ground hold (settle straight down, then ask for nothing) fixed it: 0.07 m.
- **A clearance band alone does not stop a fast descent.** 150 m/s down with 15 m/s² to brake needs about 740 m; an 80 m band starts far too late. Counting the clearance less the stopping distance slows the descent early enough (core test from 400 m). The braking thrust has to be taken in the ship's attitude: rolled on its side it is the side thrust (2.2 m/s² left after gravity, not 15).

Found by the review subagent and fixed:

- **The G-safety turn cap by the sign of the rate only is wrong in reverse.** Nose up while flying backwards pulls the velocity down, so the down tolerance applies, not the up one; and the 1 g the thrusters already hold against gravity counts. Now the cap scales pitch and yaw so that rate × velocity plus the gravity hold stays inside the tolerance box.
- **A boost cap below the cruise cap slows the ship.** Backward boost 80 m/s under a cruise of 150 m/s made Shift+S brake. Boost caps are now at least the cruise cap (200 m/s backward, placeholder).
- **The scenario read the touchdown speed too late.** The contact flag comes a step after the contact, and the solver has cut the approach by then (1.56 m/s shown for a 2.0 m/s touchdown). It now takes the largest sink of the last 0.25 s before the flag.
- **Decoupled on the ground the axis ship glided.** The ground hold now always damps and strips sideways speed, as in the classic model.

## First playtest (initiator, 2026-10-09)

"habe das neue flugmodell etwas getest. es ist viel besser als das alte. Presicion ist aber viel zu langsam." An impression, not yet a decision to replace the classic model.

Precision mode after that (placeholders, commit 47fedf8 on `spike/ifcs`): full below 5 m instead of 15, off above 40 m instead of 80, 15 m/s along the ground instead of 4, touchdown 3 m/s instead of 2, full turn rates. Landing from 100 m: 6.1 s to touchdown instead of 12.8 s (classic 10.1 s), sink at touchdown 3.2 m/s.

Star Citizen switches its landing mode by hand (a toggle, which also limits the turn rate), ours comes on by itself near the ground. That difference is a likely reason it felt slow; see TODO d.

## Second playtest and tuning (2026-10-09, f988387)

Initiator: "Also mit langsam habe ich träge gemeint. Der detach modus wird deutlich schneller und ist dann aber schwer wieder zu korrigieren. Vorallem im All müssen die Düsen irgendwie Leistungsfähiger sein." "Das flugmodell in SC ist echt super und da wollen wir ran." "Dann landemodus manuell" (decided for the spike). Further work in a new session; this spike counts as successful.

What the records say (structure and ratios only; read by two subagents):
- Star Citizen: all thrusters share one curve that lowers thrust with air density (vacuum about 1.8x full atmosphere); the flight computer's caps do not depend on density. One speed limiter per ship, no coupled/decoupled split (so decoupled is likely capped too); drift is corrected with the space brake or by coupling again. Turn rates have a curve over speed with a peak at mid speed; nothing ties the nose to the velocity. Angular decay is about 3-4x the linear one on fighters. Landing mode is engaged by the player (landing gear); the distance band only shapes the caps inside it. Gravity compensation is a pilot toggle, default on; hover is coupled flight with it on (hover bikes have their own spring model).
- No Man's Sky: separate parameter sets for space and planet instead of a blend: cruise about 1.5x, thrust 2x, boost top speed about 7.7x in space. Boosting cuts turning hard, the velocity follows the nose quickly, a descent limiter and look-ahead rays protect the ground; normal ships cannot hover (a minimum speed on planets), landing is a scripted assist.

What changed (all values still placeholders in `ship_axis.json`):
- Landing mode by hand (K, binding with fallback). Without it, low flight is not capped; the descent is always held to what the upward thrust can stop above the ground, with the braking the limit's curve asks for (it touched down at 6 m/s while it only capped the goal).
- Thrust per direction is now the vacuum value; at air density 1 it is half (`atmosphere_thrust`), so near the ground it is as before and in space twice as strong.
- Speed caps in space (`space`: cruise 300, boost 600/400 m/s), blended by density with the atmosphere caps (150, 350/200).
- Decoupled thrust stops at the same caps per axis; X still stops a decoupled ship.
- Livelier: linear decay 3/s (was 2), angular decay 12/s (was 8), angular acceleration 8/8/14 rad/s² (was 6/6/10), turn rates over speed (0.85 at rest, 1 at half the cruise cap, 0.8 at the cap).
- G-safety turn cap switchable (`cap_turns`); on by default, as in the playtested version. Backward tolerance 6 g (4 g held the brake in space to 39 m/s²).

Numbers (axis model; classic unchanged): W from rest at 400 m 1.4 s to 50 m/s, 186 m/s after 15 s (the cap at that height). In space 300 m/s, 90 % after 4.5 s (classic 350 m/s, 3.3 s). Decoupled W for 15 s in space: 301 m/s (classic runs away to 456 m/s); X then stops it in 6.0 s / 769 m (classic 3.2 s / 694 m). Landing from 100 m in landing mode: 6.6 s, 3.0 m/s at touchdown.

Turn at cruise speed (400 m up, full stick, 5 s), turn cap on and off:

| | Classic | Axis, cap on | Axis, cap off |
|---|---:|---:|---:|
| heading change (deg) | 694 | 62 | 391 |
| yaw rate at the end (deg/s) | 143 | 12.5 | 84 |
| largest slip (deg) | 180 | 29 | 179 |
| speed at the end (m/s) | 14 | 205 | 136 |

With the cap off the axis model turns like the classic one, only slower: the side thrust cannot turn the velocity at that rate, the ship spins around it. With the cap on it stays on its line but turns slowly at speed.

## Open for the initiator

- TODO(initiator) a) Every value in `ship_axis.json` (caps, accelerations, decays, band, G tolerances).
- TODO(initiator) b) One speed cap for every direction (their structure: strafe and climb as fast as forward, only slower to reach) or a cap per axis (ours today).
- TODO(initiator) c) Turning: G-safety caps the nose (`cap_turns: true`, playtested) or the nose turns at its rate and the velocity lags behind (`false`, closer to Star Citizen's records, spins around at speed). A middle way would be more side thrust or a softer cap.
- TODO(initiator) d) Landing mode (now by hand, K): the key, the band, the touchdown speed.
- TODO(initiator) g) Space: blend by air density (now) or separate sets as in No Man's Sky; how much faster; a NAV mode as in Star Citizen instead?
- TODO(initiator) e) The model's name and its HUD word.
- TODO(initiator) f) If not the whole model: which parts to carry into the classic one. Candidates: the angular acceleration cap, the G-safety turn cap, acceleration per direction, the stopping-distance rule.
- Not built: jerk limits, proximity sensing beyond the stopping distance, turn-rate changes over speed, nav mode.
