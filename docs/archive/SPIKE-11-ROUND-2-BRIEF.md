# Spike 11, round 2 brief — make the warp mergeable

Status: ready to start. Written 2026-10-08, after the review of round 1 (`SPIKE-11-REPORT.md`). Initiator, 2026-10-08: "Generell bedeutet das wir bauchen noch eine Runde." Design decisions below are in `DECISIONS.md` with the initiator's words; everything else was delegated ("den rest kannst du entscheiden basierend auf deiner expertise").

## Goal

Round 1 showed the warp works in the simulation. Round 2 fixes what the review found, builds the design changes below, and makes the code ready to merge into `main`. The result: an updated `SPIKE-11-REPORT.md` (section "Round 2") and a branch that passes `cargo t`, `cargo scenario` and the `warp` scenario **in a window**, with screenshots of every phase including arrival.

## Read first

1. `SPIKE-11-REPORT.md` and `SPIKE-11-BRIEF.md` (round 1).
2. `DECISIONS.md`: "World values and scale", "Travel time", "Quantum drive: where it may start", "Quantum drive: emergency exit", "Quantum drive: arrival", "Planets in spike 11".
3. `research/quantum-drive-reference.md`. The Star Citizen records themselves are in `research/local/sc-logistics` (gitignored, sparse clone; read, never copy files, names or texts).
4. Code repo `AGENTS.md`, `README.md`, `WORKSPACE.md`.

## Where to work

Same branch `spike/11-warp`, worktree `~/Work/exo-1-spike11` (now at `2c9c947`). This machine has a GPU and a window: run the windowed scenario on workspace 7 (`WORKSPACE.md`). Commit small and often. Push only after the initiator says yes. Move the tag `spike/11-warp` to the last commit at the end.

## Design changes (decided)

1. **Target visible.** Today Cinder is about one pixel from orbit at 12,500 km. Star Citizen solves this with HUD markers for every destination (icon by body type, `navIcon`; icons scale up to `maxIconScaleRange` 7e7 m). Do the same: a HUD marker on the selected target with name and distance, plus a marker on the other planets. Keep the 12,500 km default.
2. **Travel time is a guide value.** 30 s without input is where we start; 45 or 55 s is fine. No hard limit anywhere in code or checks. No speed scaling for long trips this round.
3. **Where a jump may start:** above **1.5 × the planet's atmosphere height** (Hearth and Cinder: 1,800 m). The atmosphere height and this factor live in the planet's entry in `system.json`, not drive-wide.
4. **Emergency exit**, as in Star Citizen: during the flight the pilot holds a key and the ship drops out early, stops at a normal speed in open space, then cooldown. Star Citizen's records hold only the status message, so the behaviour is ours: pick a short drop (a few seconds, not the full ramp-down), check the drop point for obstruction, and state the values in the report. Scenario: exit at mid-flight, ship ends in open space, no planet within its obstruction radius.
5. **Arrival facing the planet.** The ship arrives on the line to the target's centre, nose at the centre, outside the atmosphere with the planet clearly ahead. Start value: arrival 12 km from the centre (the planet fills about 49° of the view, 5.8 km above the atmosphere top; well above the 1,800 m jump limit). The last part of the path comes in radially, not along a tangent; drop the 40° turn of the exit point. Keep the tangent departure where the straight line would hit the departure planet.
6. **Planets stay practically identical** (same radius and atmosphere, different seed). Different radii and atmospheres are a later feature, but every value stays per planet in the file.
7. **Cabin view:** the tunnel streaks are visible through the cabin window. Nothing more; window and ship design come later.
8. Not this round: group jump (parked), fuel, heat, interdiction, events on the way.

## Fixes from the review (round 1)

| # | Problem | Fix |
|---|---|---|
| F1 | `Snapshot::to_frame_of` indexes `centres[self.planet]` unchecked; `decode` accepts ids up to 7, there are 2 planets: one bad packet panics | Reject unknown planet ids on receive |
| F2 | `MAX_SHIP_POSITION` 1e8 m and `MAX_SHIP_SPEED` 2e6 m/s are constants; at 187,500 km a warping ship is rejected mid-flight | Derive both from the loaded system (largest planet distance plus margin, top speed times 2) |
| F3 | `net_core::PLANET_CENTRES` (0 and 200 km) still used by `to_world` for remote ships; correct today only because everything is converted to planet 0 at the origin | Remove the constant; centres come from the system |
| F4 | "arrived at the exit point" passes on the `Arrived` event alone; the distance is printed, not checked. `exit_pos` is set once in `begin` while the path is rebuilt every tick until ramp-up | Keep the exit point in step with the path; check the distance against a stated tolerance |
| F5 | Hearth's radius in `system.json` is always overwritten by `--radius` (default 5000) | The file wins; `--radius` only when given |
| F6 | `min_altitude` and `obstruction_margin` are drive-wide | Per planet (see design change 3) |
| F7 | `--distance` puts all planets at one spot and leaves the 1,000 km frame zones overlapping at short distances | Check that frame zones stay below half the distance; refuse or clamp with a message |
| F8 | The cabin window shows a flat blue area during the warp; one cruise shot is blue all over (`shot-04-a2b-rampdown.png`) | Find the cause; streaks visible through the window (design change 7) |
| F9 | HUD shows nonsense during the warp (`ground 6171801 m`), and can name one target while showing another's distance | One target rule (see C1); ground and altitude only near a planet |
| F10 | No screenshot after arrival; "terrain on arrival" never seen | Screenshots at exit and 2 s after; check terrain is drawn |

The `/proc/self/statm` memory read is allowed (initiator, 2026-10-08: "Ist okay, habe ich erlaubt").

## Cleanups (judgement calls from the review, delegated)

- **C1** One function in `warp_core` picks the effective target (skip the planet you are at); drive and HUD both use it.
- **C2** One method on `Phase` says whether the ship is held on the path; `ship.rs` uses it instead of its own list.
- **C3** A small `PlanetId` type instead of bare `usize`, `u32` and `u8` for planets.
- **C4** Split `warp_step`: drive control, planet swap and telemetry apart. Telemetry fields belong to the scenario, not to the game resource.
- **C5** Remove the dead defaults (`PendingPlanet.id`, `PlanetRes.id` default 0) and the load path without a planet entry if nothing needs it.
- Keep the network fixes, the course ring, `FrameLog` and the planet colour from round 1.

## Done when

- `cargo t` and `cargo scenario` pass; the `warp` scenario passes in a window with screenshots: spooling, ramp-up, cruise (outside and from the cabin), ramp-down, exit, 2 s after exit, emergency exit.
- Checks for F1 (bad planet id rejected, no panic), F2 (snapshot accepted at the longest trip), F4 (exit within tolerance), design change 3 (refused below 1.5 × atmosphere), 4 (emergency exit) and 5 (nose within a stated angle of the target's centre at exit).
- Frame time per phase from the windowed run in the report.
- Report section "Round 2": what changed, values chosen and why, what is still open.
