# Flight controller: three public references

Research note, 2026-10-10. The initiator asked to keep the useful parts of three public repos. This is not a proposal, not an approved design and not a decision. Nothing from the repos is stored here: no code, no measured constants, no bindings, no ship names. Where a number below is ours, it is a placeholder from `content/tuning/ship.json` in the code repo, tagged as such.

Our flight step is `flight_core::axis` (spike 13). Assisted flight asks for `(goal − velocity) × linear_decay`, then clamps that vector to a per-axis thrust box (`Dirs::clamp`), also limited by G-safety. With the placeholders (`linear_decay` 3/s, forward 60 m/s², sideways 24 m/s²) the forward axis saturates at a 20 m/s error and the side axes at 8 m/s. Cruise is 150 m/s in atmosphere and 300 m/s in space, so almost every real direction change spends its time saturated. Near the goal the same law eases in exponentially. Assist off is thrust along the stick; the velocity is kept.

## 1. Coupled thrust direction

Source: `https://github.com/deng0/SimpleFlightComputer` (public domain, read 2026-10-10). A small test of one question: in assisted flight, how should thrust turn the current velocity into the desired one. Three laws, same thrust box:

| Law | What the acceleration does | Path |
|---|---|---|
| Max thrust | Each axis runs at its cap until that component of the error is gone. Axes finish at different times, so the acceleration direction kinks mid-manoeuvre. | Curved in velocity space. One axis can leave the speed cap while another is still correcting. |
| Stable | Thrust stays on `desired − current`, throttled to what the box allows in that exact direction. | Straight in velocity space. Drift across the desired direction lasts the whole burn. |
| Anti-drift | While an axis is saturated and a desired direction exists, bias the thrust so the component across that direction dies in proportion to how much thrust is available that way, then fall back to the stable direction. | Straighter flight path. Braking and steering at the same time stays predictable. |

Our clamp is the first law for as long as the request sticks out of the box, and the second law only on the last few metres per second. The missing piece is the direction choice inside the box we already have (`Dirs::clamp`, `Dirs::support`). It runs only while assisted, a desired direction exists, and at least one axis is saturated. G-safety is unchanged, because the new direction stays inside the same box. Assist off does not use it; the playtest note that the loose mode is hard to recapture is a different problem (no damping, on purpose).

The repo is not a flight model. No rotation, jerk, atmosphere, gravity or mass integration.

## 2. A measured Newtonian flight step

Source: `https://github.com/emcodem/sc_webgl` (no licence file, read 2026-10-10). A browser dogfight trainer whose flight step was fit to frame-counted captures of one light fighter in 2026, then corrected when later captures broke the fit. Read for the shape of the step. Their constants stay in their repo.

What the step actually branches on:

- **One rotational budget.** Pitch, yaw and roll inputs are scaled together so a diagonal does not add the three maxima. Pitch and yaw are a second-order underdamped response: they overshoot the target rate and settle. A reversal (target opposite the current spin) is a constant deceleration, not that same spring. Roll release is its own constant deceleration. Clamping pitch and yaw to the max rate would delete the overshoot, so they leave it unclamped and let the spring converge.
- **Spool, per thruster group.** Main, retro and vertical wait a short time after a fresh press before thrust appears. The timers reset when the input returns to zero. Lateral strafe had no such wait in their captures. Boost skips the wait.
- **Boosted linear thrust has two jobs.** Aligned (pushing further along the axis velocity, or from rest): thrust plus real drag, a curve toward an asymptote above the cap. Countering (pushing against an existing velocity on that axis): a flat deceleration, no drag, around three fifths of the aligned rate, on every axis they measured. One thrust value per axis cannot be both. Unboosted countering needed no extra branch: the input sign already picks the opposing thruster, and their unboosted drag is negligible.
- **Strafe weakens as forward speed climbs.** Keyed on speed, not on whether forward is held. Two shapes in their captures: under boost the authority tapers across the whole run up to the boost cap; unboosted it stays full up to the cruise cap and only collapses while coasting above that cap. Rough fit, noisy captures.
- **The speed cap refuses thrust, it does not cancel it.** Above the cap, drop only the part of the acceleration that points along the current velocity. Strafe and steering across that velocity still work, so the bleed stays steerable. Subtracting a governor equal to the thrust freezes the speed (the two terms annihilate). Coast and the governor must not both bleed on the same tick. Re-engaging boost raises the cap, which is what lets speed climb again.
- **Letting go while assisted brakes to a stop, through the cruise cap, with no plateau there.** Each local axis sheds speed at the opposing thruster's acceleration. That does not preserve the heading: a diagonal slows crookedly because the axes differ. Their capture of a full release was roughly constant rate after a short transient, and much faster when the strong retro thrusters were the ones stopping forward speed.
- **The space brake is a different law.** It decelerates along the actual velocity, so the heading of travel stays put. The ceiling is the weakest axis in that direction: an axis's acceleration divided by how much of the unit velocity it has to cancel. Full brake power only when the velocity lies on one axis. Near zero the rate becomes proportional to speed; a flat rate all the way down stopped about three times too fast in the last stretch. The brake is a hold. Decouple is a toggle. Decouple removes the auto-damping and keeps the speed cap.
- **Author mass, per-axis thrust and the caps.** Boost thrust, countering thrust and drag are derived from those plus a few ratios, so a second ship is not a new curve fit. The same step drives the player and the AI.

Their boost meter, on that one ship, drained and refilled at one rate each. A two-rate "red zone" did not show up in the capture. The code still allows a per-ship kink.

Mouse virtual stick, from their side-by-side notes: the on-screen stick marker matches the value the flight step uses (it is not a smoothed cosmetic). Deadzone and gain are one absolute threshold, so changing the gain moves the deadzone even when the displayed percentage stays. The marker's full-scale mark is not the flight step's full-scale mark; matching the pictures can detune the rate. We already have a virtual stick (issues #24, #27). Aim assist that pulls the nose is out (`DECISIONS.md`, mouse flight).

## 3. Action maps, Alpha 2.4

Source: `https://github.com/jllamas/StarCitizenActionMaps` (no licence, last push 2016, read 2026-10-10). Personal joystick layouts for an input mode where the mouse aims and the stick flies. Not a flight model, and ten years older than the captures in section 2. Do not copy the files or the button assignments.

The shape that is still worth remembering:

- Actions live in named groups (movement, view, targeting, weapons, missiles, defence). A layout rebinds a group; it does not hard-code keys in the flight step. We already store bindings as data (`bindings.json`).
- Boost and afterburner are two actions. In that era a tap raised acceleration inside the current cap, a second tap raised the cap. One physical button can send both. Our boost is one speed stage (`DECISIONS.md`, "erstmal nms weil simpel"). The split is a later option, not a change.
- The flight-mode cycle, the decouple toggle, the landing-system toggle and the landing-mode cycle are separate actions. We already collapsed flight modes to assist on/off, with landing mode on its own key.
- A hat can be strafe, and a side direction can add a forward component. That is a binding, not a flight law.
- The mouse-aim mode has to be turned back on every time the pilot sits down. Mouse-aim stays out of our bindings.

## 4. A block ship in Bevy

Source: `https://github.com/AnthonyTornetta/Cosmos` (GPL-3.0, Rust, Bevy, read 2026-10-10). A multiplayer game where the ship is the blocks you place. GPL: ideas only, no code and no constants. It is not a flight computer. Translation is one impulse along the camera axes, rotation is written straight into angular velocity, and a single speed clamp stops the ship. No per-axis box, no assist, no atmosphere.

What is still worth keeping:

- **Thrust is the sum of the thruster blocks**, plus a little from the ship core so a bare core can still move. Power scales that sum down, and an empty store means no translation. Torque does not come from where the blocks sit. It comes from the mouse offset, and a larger footprint turns more slowly. That size rule is a stand-in for inertia. Our #145 already asks for a real force and torque on the mass; ship power stays with the ship sprints (#174).
- **The pilot sends a request, the server applies the impulse.** No pilot, the request is cleared. Brake and match-speed are impulses along a velocity error, scaled by mass so the acceleration does not depend on mass, and capped by the thrust sum. The brake therefore keeps the velocity heading. Match-speed chases a focused body inside a distance. The 2016 action maps already name that action. We do not have it; it is a later idea, not this slice.
- **Gravity is an emitter** (force per kilogram, radius). Close to one of their cube faces it pulls along the face; farther out it pulls toward the body. A gravity-well block is a placed emitter. Our planets stay spheres. Cabin gravity is already decided.
- **A warp drive can be too small for the ship.** Charge comes from the drive blocks; the jump costs more as the structure gets heavier; past that the state is "too big", not a slower jump. A later upgrade reason (#174), not flight feel.

## Where this lands on our step

| Idea | Today | Open |
|---|---|---|
| Direction inside the thrust box while saturated | Per-axis clamp of `(goal − v) × decay` | Anti-drift bias, assisted only |
| Stop along the velocity, heading kept | X drives the assisted goal to zero, so a saturated stop still kinks | Brake as one deceleration along velocity, weakest-axis ceiling, ease near zero |
| Cap refuses further thrust along velocity | Coupled goal is already inside the cap; decoupled zeroes the stick on an axis that is at its cap | Clip only the along-velocity part, leave steering |
| Idle assisted braking | `(0 − v) × decay`, then the box | Per-axis opposing thruster, through the cap, no plateau |
| Rotation | Pitch/yaw ellipse, first-order rate error, angular accel cap. Roll stacks on top | Shared budget with roll; second-order spool; reversal as its own deceleration |
| Boost | One multiplier per direction, same for extending and braking. Capacitor exists, default is a speed stage | Aligned versus countering, derived from thrust and caps |
| Strafe versus forward speed | No link | Authority taper, two shapes |
| Engine spool | Stick ramp (0.3 s placeholder), not a dead time | Per-group delay, reset on release, skipped by boost |
| New ship | Every field authored | Mass, thrust, caps, a few ratios |
| Brake along the velocity | X is the assisted goal at zero | Cosmos does this as an impulse along the velocity error, capped by total thrust. Same candidate as the row above |

## Smallest next step

Code repo issue #183, sub-issue of epic #143. A numbers-only comparison, same thrust box, the live step unchanged: sideways at the cap then stick forward; a full stop; a stop while a new direction is held. Columns: time, path length, whether an axis leaves the cap, whether the velocity heading stays put. The initiator decides afterwards whether any open row becomes a slice. The milestone could not be set from this session.

#145 (force and mass), #146 (jerk and the boost ramp), #141 (assist on/off) and #178 (a slow first ship) stay as they are. Feel work these repos do not cover stays in `docs/research/flight-feel.md`.
