# Spike 12 brief — crates as Avian bodies

Status: ready to start. Written 2026-10-09, after the milestone C playtest. Initiator, 2026-10-09 (playtest feedback): "Sie drehen sich kaum, die Physik fühlt sich nicht gut an." "Sie rutschen viel zu weit." New direction: "Eine Kiste hat drei Zustände" (physics, frozen, part of the ship), "Erst ein Spike auf spike/avian-crates".

## Goal

Find out whether crates outside, near a player, can be Avian rigid bodies (tipping, rolling, stacking, pushed by the walker) next to the existing `grab_core::CrateBody` for the cabin, and how the handovers between the three states behave. The result is a report with a verdict and numbers per question; the initiator decides the model.

## The three states (initiator, 2026-10-09)

1. **Physics:** outside and near a player, an Avian rigid body. It tips, rolls and stacks, and the walker bumps it.
2. **Frozen:** outside and resting, beyond the collision patches or on a planet nobody is on. A fixed object with no body; only the pose is kept. It wakes when a player comes close. Moving crates never freeze. Farther out, the existing height-function net and the budget's distance rule apply.
3. **Part of the ship:** in the cabin. No Avian there, because of the child-collider lag and warp speeds. The tested `CrateBody` stays for the short flight up to an impact (a throw or a drop in the cabin).

## Known so far

- `CrateBody` (`grab_core`, milestone C, #81 to #85): an upright box (yaw only) moved by sweeps, Coulomb friction along the floor normal, sleep. Since 531fd02 it has impact friction and friction 0.8 (it used to keep the full tangential velocity at an impact).
- Night extra E2 (reverted, 56f3686; code in a local stash and in commit 7e83b52): one collider per crate on its own layer, synced every tick, a child of the ship in the cabin. Lesson: `tests/session.rs` (menu: host, back, join) failed with "Encountered an error in command: Entity despawned" until the sync skipped crates whose ship was gone and used `try_despawn`. Any per-crate entity that hangs on a ship must survive the ship's despawn.
- Hold: a velocity servo with a force cap per holder (`grab_core::hold_force`); today its output is an acceleration on `CrateBody`.

## Questions (each gets a verdict and a number)

1. **Hold servo through Avian forces.** Does holding, carrying and throwing feel the same as now? Measure: lag and overshoot on the `crate-carry` paths, small/medium/large, hands and tool; compare with the current numbers.
2. **Handover at the ramp edge while carrying.** A held crate crosses from cabin (ship state) to outside (Avian body) and back (as in the `crate-unload` scenario). Measure: pose jump and velocity jump at the handover, and dropped holds.
3. **A crate on the ramp when the ship takes off.** Does it slide off, come along, or tunnel through the ramp? Which state is it in? Measure the outcome over several takeoffs.
4. **Waking on freshly streamed terrain patches.** A frozen crate wakes as a player approaches; does it have ground under it yet? Measure: drop, sinking, or launch at waking, with the patch streamed in the same frame, one frame late, and several frames late.
5. Also measure: step cost per Avian crate (on this NAS no perf claims, so count bodies and contacts; timings on a desktop only), and whether stacks of 3 sleep.

## Open design questions (list them, do not decide them)

- TODO(initiator) a) Cabin: does every crate set down become part of the ship at once (the lock grid then only helps keep order, and sliding under acceleration goes away)? Or does the lock grid stay the condition, with loose crates keeping the current model?
- TODO(initiator) b) Friction: start value 0.8 plus impact friction (in `content/tuning/grab.json`), to be tuned by feel; the Avian friction for the physics state has to be picked to match.

## Read first

1. Code repo `AGENTS.md`, `WORKSPACE.md` (build environment, no perf claims on the NAS), `NIGHT-LOG.md` on `night/extras`.
2. `crates/grab_core` (`CrateBody`, `hold_force`), `crates/exo_app/src/cargo.rs`, `grab.rs`, `cargo_scenario.rs`.
3. Avian source for the version in `Cargo.lock` (`~/.cargo/registry/src/*/avian3d-*`): rigid bodies in f64, sleeping, collision layers.

## Where to work

- Code repo, branch `spike/avian-crates` from `night/extras` (531fd02), not from `spike/combined`: the crate code exists only there. Freeze tag at the end: `spike/12-avian-crates`.
- Each question gets a scripted scenario through `Controls`, so it runs headless.
- Commit small and often. Push only after the initiator says yes.

## Not in scope

- Deciding a) and b), crate sizes, the object budget numbers.
- Networking crates.
- Merging into `main` or `night/extras`.

## Results

`SPIKE-12-REPORT.md` in this repo, learnings in `LEARNINGS.md`. One line at the top: can the three-state model work, and which question blocks it if not.
