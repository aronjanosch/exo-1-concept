# Spike 11 brief — two planets and a warp

Status: draft, waiting for the initiator. Written 2026-10-08. Research: `research/interplanetary-travel.md`. Initiator, 2026-10-08: "der warp zwischen planeten muss funktionieren das muss ja auch nicht umbedingt eine richtige bewegung durch den echten raum sein da können wir ruhig trixen"; "ja spike und dann ggf. den spike übernehmen"; "Wir wollen so nah ran anden quatum drive wie es für einen Spike"; "die abständer kann der agent im spike selbst definieren, vorbild star citizen, no mans sky und andere games".

## Goal

Two planets far apart in one system, and a warp modelled as closely on Star Citizen's quantum drive as a spike allows (see "Quantum drive model" below), that takes a ship with pilot and passengers from orbit around one to an approach point at the other, without a loading screen. The result is a report with numbers and a build the initiator can fly with distance and warp duration as settings. Unlike earlier spikes, the code may be carried over: keep it clean enough to merge.

## Quantum drive model

As close to Star Citizen's quantum travel as a spike allows (research note, verified on the Star Citizen wiki): pick a destination, the drive **spools up**, the pilot **aligns** with the target and the drive **calibrates** while the course is held; then real movement along the line at very high speed, and an exit at an approach point near the target. Losing alignment during calibration aborts; an **obstruction** on the line (planet, ship) blocks the start. Our own names, look and numbers; nothing copied.

## Distances

The spike agent picks the distances between the planets and the warp speeds, with Star Citizen, No Man's Sky and other games as reference (initiator, 2026-10-08). Write the chosen values and the reasoning into the report (sky size of the other planet, travel time, speed); the research note has a starting table. 200 km (spike 4) is too short. Keep them as settings so the initiator can change them after flying.

## Not decided (the spike makes them settings, the initiator picks)

- Warp duration and feel (follows from the distances and the speed curve).
- Where a warp is allowed (orbit only, minimum distance from a planet), what can interrupt it.
- How the second planet differs (seed, radius). Default: different seed, same 5 km radius.
- Fallback: if real movement fails, the tunnel trick (teleport behind a tunnel effect). Build it only then, and say why.

## Read first

1. `research/quantum-drive-reference.md` (Star Citizen's quantum drive records: states, knobs, per-body radii, spline path), `research/interplanetary-travel.md`.
2. `LEARNINGS.md`, `SPIKE-9-REPORT.md` (render origin, f64 physics), `SPIKE-8-REPORT.md` (generation cost), `SPIKE-10-REPORT.md` (frames, hold error at speed).
3. Code repo `AGENTS.md`, `README.md` (Run) and `WORKSPACE.md`.

## Where to work

- Code repo, branch `spike/11-warp` in its own worktree `~/Work/exo-1-spike11`, from `main` after `fix/5-walker-falls-in-space` is merged (it changes walker movement in space). Freeze tag at the end: `spike/11-warp`.
- Commit small and often. Push only after the initiator says yes.

## Architecture

Warp state, speed curve, exit point and planet registry live in `*_core` crates without Bevy types, in f64, tested with `cargo test`. Bevy does the tunnel look, rendering of distant planets, loading and unloading terrain, HUD.

## Steps

1. **Second planet.** Planet registry with centre, radius and seed per planet, two planets at a configurable distance. Done when both render and you can land on each (scenario teleports the ship next to each).
2. **Distant planets.** Draw a planet you are not near as a cheap sphere or impostor. Reference: Kerbal Space Program draws far bodies in a scaled-down second scene rendered behind the normal one ("scaled space"; from memory, not checked). Done when each planet is visible from the other at the chosen distance, with frame cost measured.
3. **Quantum drive.** Spool-up, alignment and calibration, obstruction check, speed curve along the line, exit at the approach point, tunnel look. Collisions off or swept during the warp: read how Avian 0.7 handles continuous collision and the broadphase at 1e5 m/s and above in `~/.cargo/registry/src/*/avian3d-0.7.0/` before choosing. Done when a scenario warps A to B and back, an aborted calibration and a blocked start are covered, and the ship ends at the exit point within a stated tolerance.
4. **Terrain on arrival.** Generate the target planet's terrain during the warp, free the old one. Done when the first frames after exit show terrain, with the time to ready and memory measured.
5. **Passengers.** A walker in the cabin during the warp. Done when drift and deck contact are measured as in spike 10.
6. **Network.** One remote ship warping, seen by another client. Measure the snapshot error during and after the warp. Report only, no fix unless small.

Each step that fails stops only that step: write down the blocker and continue.

## Not in scope

- More than one system and jumps between systems.
- Fuel, cost, travel events, interdiction: design questions, list them as open.
- Places and trade on the second planet.

## Results

`SPIKE-11-REPORT.md` in this repo, learnings in `LEARNINGS.md`. One line at the top: does a warp between two far planets work without a loading screen, and at which distances.
