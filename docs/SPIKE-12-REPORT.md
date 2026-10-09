# Spike 12 report — crates as Avian bodies

**In one line:** the three-state model works for holding, carrying and waking. Two things are open: stacks of three are not stable yet, and the ramp. For the ramp, the simple answer is to keep it out of Avian: a crate on the ramp stays a `CrateBody`, as today.

Branch `spike/avian-crates` (ef24e65, pushed; based on `night/extras` 531fd02). Brief: `archive/SPIKE-12-BRIEF.md`. Measured headless on the NAS: behaviour only, no timings. The switch `EXO_AVIAN_CRATES=0` runs the old model for comparison.

## What the spike built

`crates/exo_app/src/avian_crates.rs`:
- A crate in the planet frame within 40 m of the walker, and over a collision patch, becomes an Avian rigid body. Its density is set so the box's own volume gives the crate's mass. Friction 0.8, restitution 0, SweptCcd.
- Before the step, gravity and the hold (`grab_core::hold_force`, unchanged) go in as accelerations. A held crate is turned upright through its angular velocity.
- After the step, the pose is copied back into `CrateBody`, so grab, interaction and rendering stay unchanged.
- When its bottom enters the cabin, the crate becomes a `CrateBody` again and is set upright.
- Far away and at rest, it goes back to a sleeping `CrateBody` (frozen).
- The walker sweeps against crate bodies, except the held one.

## Results

| Question | Verdict | Numbers |
|---|---|---|
| 1. Hold servo through Avian forces | **Same feel** | Hold error RMS over 5 holds (`crate-carry`) within +0.015 m of the old model; max +0.02 to 0.06 m; tool pull 8 m to 3.00 m in both. A thrown crate flies 5.08 m (old 4.77 m). One check fails: the medium crate lay tipped 18° on the slope, so the hold target sat below its centre. That is the check's measure, not the servo. |
| 2. Handover at the ramp edge while carrying | **Works** | `crate-unload`: first tick as a body 1.6 mm / 0.16 m/s off the prediction; worst value elsewhere in the run 54 mm / 3.0 m/s. Tilt set upright at the cabin edge 0.05°. No hold dropped. Set-down crates lie flat on the slope (0.01 and 0.03 m up; the old upright crate sat at 0.10 and 0.23 m). |
| 3. Crate on the ramp at takeoff | **Simplest: ramp counts as ship** | As now, the ramp collider (filter `NONE`) and the hull (filter World/Ship) do not stop bodies: a body falls through the ramp at once. If they are allowed to hit crates, the crate rests on the ramp and is carried along at takeoff, 1.4 m above floor level at 40 m altitude: a planet-frame body in dynamic contact with the ship, exactly what state 3 is meant to avoid. The old `CrateBody` rests on the ramp and stays on the ground at takeoff. |
| 4. Waking on freshly streamed patches | **Works with the patch guard** | Patch 0.17 s after the walker arrives. With the guard (body only over a patch) the crates wake at 0.18 s with no drop. Without it they free-fall 0.139 m below ground until the patch catches them, so a slower stream would let them fall through. |
| 5. Stacks of three, walker | **Stack open** | Three small crates stacked: the bottom one bounces on the heightfield with a growing amplitude (0.02 to 0.13 m), and the stack falls at about 1.1 s. After that all three sleep. Cause not found (suspects: SweptCcd, substeps, the box against the heightfield); not chased further. The walker is stopped by bodies but does not push them. |

## Open for the initiator

- TODO(initiator) a) Cabin: does a crate set down become part of the ship at once, or does the lock grid stay the condition? (from the brief)
- TODO(initiator) b) Friction 0.8 plus impact friction (from the brief).
- TODO(initiator) c) Ramp: recommended to treat it as ship (`CrateBody`, sweeps against the ramp as today), with Avian only from the ramp's end on. No crate-to-ship physics contact at all.
- TODO(initiator) d) Stacks: needed? If yes, a small follow-up on the bouncing bottom crate.
- TODO(initiator) e) Should the walker push crates? Today it is only blocked.
