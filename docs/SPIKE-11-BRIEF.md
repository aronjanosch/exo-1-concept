# Spike 11 brief — two planets and a warp

Status: draft, waiting for the initiator. Written 2026-10-08. Research: `research/interplanetary-travel.md`. Initiator, 2026-10-08: "der warp zwischen planeten muss funktionieren das muss ja auch nicht umbedingt eine richtige bewegung durch den echten raum sein da können wir ruhig trixen"; "ja spike und dann ggf. den spike übernehmen".

## Goal

Two planets far apart in one system, and a warp that takes a ship with pilot and passengers from orbit around one to an approach point at the other, without a loading screen. The result is a report with numbers and a build the initiator can fly with distance and warp duration as settings. Unlike earlier spikes, the code may be carried over: keep it clean enough to merge.

## Not decided (the spike makes them settings, the initiator picks)

- Distance between the planets. Measure 200 km, 20,000 km and 1,000,000 km.
- Warp duration and feel: a short transition (5–10 s) or a trip (20–40 s).
- Real movement along the line versus the tunnel trick (teleport behind a tunnel effect). Start with real movement (research note: one world frame, co-op stays simple); build the tunnel trick only if real movement fails, and say why.
- What starts a warp (button, spool-up, course to hold), where it is allowed (orbit only?), what can interrupt it.
- How the second planet differs (seed, radius). Default: different seed, same 5 km radius.

## Read first

1. `LEARNINGS.md`, `SPIKE-9-REPORT.md` (render origin, f64 physics), `SPIKE-8-REPORT.md` (generation cost), `SPIKE-10-REPORT.md` (frames, hold error at speed).
2. Code repo `AGENTS.md`, `README.md` (Run) and `WORKSPACE.md`.

## Where to work

- Code repo, branch `spike/11-warp` in its own worktree `~/Work/exo-1-spike11`, from `main` after `fix/5-walker-falls-in-space` is merged (it changes walker movement in space). Freeze tag at the end: `spike/11-warp`.
- Commit small and often. Push only after the initiator says yes.

## Architecture

Warp state, speed curve, exit point and planet registry live in `*_core` crates without Bevy types, in f64, tested with `cargo test`. Bevy does the tunnel look, rendering of distant planets, loading and unloading terrain, HUD.

## Steps

1. **Second planet.** Planet registry with centre, radius and seed per planet, two planets at a configurable distance. Done when both render and you can land on each (scenario teleports the ship next to each).
2. **Distant planets.** Draw a planet you are not near as a cheap sphere or impostor. Done when a planet is visible from the other at all three distances, with frame cost measured.
3. **Warp.** Speed curve along the line between the orbits, duration as a setting, tunnel look, collisions off or swept during the warp. Done when a scenario warps A to B and back at each distance and the ship ends at the exit point within a stated tolerance.
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
