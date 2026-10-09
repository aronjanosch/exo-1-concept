# Spike 13 brief — an axis-limited flight model next to the current one

Status: ready to start. Written 2026-10-09 on the initiator's request: an alternative flight model built after the structure of Star Citizen's flight control (IFCS), switchable in the game, compared with the current one on the same manoeuvres.

## Goal

Find out how a flight model with acceleration limits per axis and direction, decay, a precision mode near the ground and a G-safety limit flies compared with today's `ShipController`, on the same scripted manoeuvres. The result is a report with numbers side by side; the initiator decides which model (or which parts) to keep.

## Known so far

- **Ours** (`flight_core::ShipController`, "classic" below): the assist asks for a velocity (stick times per-axis speeds, forward speed from a curve over terrain clearance) and closes the gap with one shared acceleration budget, smoothed over a response time. Gravity is cancelled for free while the assist is on. Rotation: target rate (turn rate, roll rate) reached by a fixed lerp, no angular acceleration limit. Boost capacitor (#90), input ramp (#25), assist on/off (H) and decoupled (C) blended over 4 s (#26), landing sink cap near the ground.
- **Theirs, structure only** (records in `research/local/sc-logistics/ifcs/` and `entities/scitem/ships/controller/`, read 2026-10-09; no values, names or texts taken):
  - Speed caps are scalars: one cruise cap for stick input (the stick vector normalised into a ball), separate boost caps forward and backward.
  - Acceleration is limited per axis and direction (forward is stronger than backward), with per-direction boost multipliers.
  - Rotation: a rate cap per axis (pitch and yaw share an ellipse, roll is the fastest), an angular acceleration cap per axis, boost multipliers per axis.
  - One linear and one angular decay. Inferred: it works like a gain, saturated far from the target and exponential close to it.
  - Precision mode: a band of ground distances, a lower speed cap inside it.
  - G limits are pilot tolerances per axis and direction (one direction of an axis much more tolerant than the other).
  - Coupled and decoupled have no per-ship numbers; they are a mode with a damping blend.
  - Not built here: jerk times, proximity sensing, rotation-rate modifiers over speed, thruster damage, nav mode.

## What the spike builds

- A second model in `flight_core`, selected per ship, using the same inputs, boost capacitor, input ramp and H/C switches as the classic one:
  - coupled: velocity goal = stick (in a ball) times the cruise cap; boost raises the forward and backward caps;
  - commanded acceleration = linear decay times the velocity error, plus what keeps the goal turning with the ship, plus gravity and drag compensation;
  - thrust clamped per axis and direction, then to the pilot's G tolerance per direction; what the thrusters cannot give, the ship does not get (a ship rolled on its side sinks if its side thrust is below 1 g);
  - rotation: target rate per axis (pitch/yaw ellipse), angular decay, angular acceleration cap per axis; with G-safety the pitch and yaw rate at speed are capped so the turn stays within the tolerance;
  - precision mode: inside a clearance band the speed cap drops (climbing away from the ground stays free);
  - decoupled: thrust along the stick plus gravity compensation, no damping; assist off: no compensation.
- Values in `content/tuning/ship_axis.json`, all `TODO(initiator)`.
- F7 switches the model in the game; the HUD names the active one. The binding gets a fallback so old `bindings.json` files load with F7.
- Scenario `flight-models`: the same manoeuvres with both models from the same start pose, numbers side by side.

## Questions (each gets a number)

1. Acceleration and top speed forward, sideways, up: time to 90 % and speed reached.
2. Stopping: coasting to rest after release, and the brake (X): time and distance.
3. Turning at cruise speed with full stick: heading rate, slip between nose and velocity, highest felt G.
4. Boost: top speed and time while the capacitor lasts.
5. Decoupled: speed kept after the blend.
6. Rolled on its side with no input: height lost.
7. Landing from 100 m with Ctrl held: time, touchdown speed, slide.

## Open design questions (list them, do not decide them)

- TODO(initiator) every value in `ship_axis.json` (caps, accelerations, decays, band, G tolerances).
- TODO(initiator) the model's name and its HUD word.
- TODO(initiator) should precision mode cap climbing too, or be switchable?
- TODO(initiator) should G-safety cap the turn rate (nose follows the velocity) or let the ship slip?

## Where to work

- Code repo, branch `spike/ifcs` (from `main` at 1c295ee). Freeze tag after the initiator's review: `spike/13-ifcs`.
- `cargo t` and `cargo scenario` must pass; one cargo command at a time.
- Commit small and often. No push, no merge into `main`.

## Not in scope

- Deciding values or which model wins.
- Network, warp and the walker (they see only the ship's velocities, as now).
- Jerk limits, proximity sensing, nav mode.

## Results

`SPIKE-13-REPORT.md` in this repo, learnings in `LEARNINGS.md`.
