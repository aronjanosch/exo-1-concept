# Spike 11 report — two planets and a warp

Date: 2026-10-08. Brief: `SPIKE-11-BRIEF.md`. Code: branch `spike/11-warp` in the code repo (worktree `~/projects/exo-1-spike11`, from `main` at `c953922`), local only, freeze tag `spike/11-warp`. Machine: the roost container (8 cores, no GPU, no display). Tags: **measured**, **calculated**, **assumed**, **open**.

## Answer in one line

**Yes, a warp between two far planets works without a loading screen, in the simulation:** a scripted pilot flies Hearth to Cinder and back, 12,500 km apart, 25 s of real movement at up to 1,000 km/s, the ship ends exactly on the exit point at 2,000 m altitude and 400 m/s, a walker in the cabin keeps 100 % deck contact the whole way. **Not seen yet:** anything on screen (distant planets, tunnel look, terrain on arrival, frame cost). This container has no GPU and no window system, so all of that is written and compiles but has never run; the first look is yours, see "What I could not do".

## What was built

| Step | State | Evidence |
|---|---|---|
| 1 Second planet | done | `warp_core::System` registry (`content/system/system.json`): seed, radius, centre, obstruction, arrival and frame radius per planet, all explicit. Planet B = Cinder, seed 4242, same 5 km radius. The scenario puts the ship and walker on Cinder's ground: walker grounded, ship 0.0 m above ground at 0.00 m/s. |
| 2 Distant planets | code only | One lit sphere per planet at its true direction, at most 100 km from the camera and scaled to keep the angular size, shown when the camera is more than 100 km from the centre (where the terrain is beyond the far plane, now 120 km). Not run, no frame cost. |
| 3 Quantum drive | done | State machine in `warp_core` (below). Scenario `warp`: refused start (too low), refused start (ship on the path), calibration lost, cancel while spooling, Hearth to Cinder and back. Also in `cargo t`. |
| 4 Terrain on arrival | partly | The target planet is generated on a background thread from the ramp-up (**measured** 607–787 ms against 25 s of flight), swapped in 0.06–0.11 ms on the main thread, the terrain rebuild is coded. First frames after exit not seen. |
| 5 Passengers | done | Walker stands in the cabin from the start of the ramp-up: deck contact **100 %** of 1,574 steps, drift **0.0 mm**; walking back and forth at 1,000 km/s: **100 %** of 132 steps. |
| 6 Network | done (report plus a small fix) | See below. |

## The drive as built

Sequence, as in the research note: pick target → **spool up** 4 s → calibration delay 1.5 s → **calibrate** (gauge fills while the nose is within 5° of the course; 5–8° the gauge waits with a warning; beyond 8° the jump is lost) → **pre-ramp** 0.5 s → **ramp-up** (stage one 57,000 m/s² to 200 km/s, stage two 100,000 m/s² to 1,000 km/s) → **cruise** → **ramp-down** (the same two stages backwards, lands exactly on the exit point at 400 m/s) → **post-ramp-down** 1 s → **cooldown** 5 s. Aborts are transitions with a reason: too low (below the atmosphere top, 1,200 m), obstructed (planet or ship on the path, at the start and again every tick until the ramp-up), cancelled, calibration lost, not ready.

- **Path:** a Hermite spline from the ship to an exit point on the target's arrival sphere. It leaves along the planet's tangent when the straight line would run into the planet, and arrives along the tangent of the arrival sphere (the exit point is turned 40° towards the departure side, so the arrival is not head-on). Near-radial directions blend into a stable tangent so the path does not flip from tick to tick. At 12,500 km the path is 0.5 % longer than the straight line. A planet behind the departure planet is passed on the spline (**calculated** in a test). The obstruction check sweeps segments of the curve, never points: at 1,000 km/s one tick is 17 km (**measured**, largest step), more than three planet radii.
- **Speed:** a feedback law, not a timetable: `v = min(v + a·dt, top, brake_speed(remaining))`. It lands exactly, works for any distance (short trips peak below top speed: 300 km peaks at 130 km/s), and the exit point is reached to the last bit (test: error < 1e-6 m; in the game the ship is placed on it).
- **Effects:** the tunnel level is a function of speed (`tunnel_level`: 6.25 to 25 km/s), the same for every distance and duration.
- **Ship on rails:** from the ramp-up the ship is put at the path's pose every tick with zero velocity of its own; the nose turns to the path direction at 0.6 rad/s; at the exit the pilot gets the exit velocity back. Not done from the reference: group jump, the 2 km/s short-hop mode, emergency exit, fuel and heat (listed as open below).

### Avian at 1e6 m/s (read in `avian3d-0.7.0/src/collider_tree/update.rs`, `dynamics/ccd`)

The AABB of every body is stretched by its velocity for one step (speculative margin is infinite by default; `SweptCcd` makes it the full swept box), so a ship with a real velocity of 1e6 m/s would own a 17 km box and pair with every terrain patch inside it. Choice: **no velocity on rails** (position written each tick, velocity zero, `SweptCcd` removed and put back at the exit); the AABB then follows the position like a teleport. Nothing else collides on the way: the ring only builds patches near the surface, and the start check covers other ships. Cabin colliders sit one tick behind the body (17 km at top speed); the walker works in the frame of the cabin collider as before, so deck contact held at 100 % (**measured**). Not tested: another ship's proxy flying past at warp speed (its AABB is long), see open points.

## Distances and speeds (settings in `content/system/system.json`, `--distance=<m>`)

I took the research note's start table, which maps Star Citizen's three trip buckets to our 5 km planets with factor 1/160 (distances, speeds and accelerations scale, times stay). **Calculated** by running the drive (test `trip_times_by_distance`):

| Distance | Target in the sky from orbit | Spool+calibration (J to engage) | Flight | Top speed reached | Cruise |
|---|---|---|---|---|---|
| 300 km | 114.6 arcmin (3.7× the full moon) | 11.1 s | 4.5 s | 130 km/s | 0 s |
| 2,000 km | 17.2 arcmin (0.55 moon) | 11.1 s | 11.2 s | 411 km/s | 0 s |
| **12,500 km (default)** | **2.75 arcmin (9 % of the moon, a small disc)** | 11.4 s | **25.2 s** | 1,000 km/s | 2.2 s |
| 62,500 km | 0.55 arcmin (a dot) | 12.7 s | 75.5 s | 1,000 km/s | 52.5 s |
| 187,500 km | 0.18 arcmin (a dot) | 16.0 s | 201.1 s | 1,000 km/s | 178.1 s |

Reasoning: at 12,500 km the other planet is a visible disc (about 2.5 times the size of Venus at its brightest, from memory, **assumed**), so you see where you are going. It is Star Citizen's shortest bucket: 12 s cruise there and 25 s here, because our ramps are not shortened. 200 km (spike 4) would be a moon. The 62,500 and 187,500 km cases keep Star Citizen's ratios, but 75 s and 200 s of flight are far past the core loop's "no more than about 30 s without input or an event" (`CORE-LOOP.md`): either raise the top speed for long trips, or put events on the way. That is your call. The jump from J to engaging takes 11–16 s on top of the flight, as in Star Citizen. About No Man's Sky: I did not look anything up and give no numbers from it.

Other start values, all in the file: arrival radius 7,000 m (2,000 m above the ground, above the 1,200 m atmosphere), obstruction radius 5,400 m (radius + 400 m; the highest terrain is 159 m on Hearth and 154 m on Cinder above the radius, **measured** by a scenario check, so 400 m leaves a margin of 240 m), frame zone 1,000 km (less than half the distance), exit speed 400 m/s, minimum altitude 1,200 m.

## Terrain on arrival (step 4)

- Target planet bake (background thread): **607 / 673 / 733 / 787 ms**, in four runs, against 25.2 s of flight. Planet B and A each need about 32 MB of memory (process: 148 MB with one planet, 180 MB while both are resident during the last seconds of a flight).
- The swap itself: 0.06–0.11 ms on the main thread (insert the new planet, drop the old patches).
- The six root chunks of the new planet: **15 ms** in one go (**measured** in the scenario, equals what the terrain does synchronously on the frame of the swap), refinement chunks 2.4 ms each, built on the pool as before. So the first frame after the swap draws a coarse planet and has a main-thread cost of about 15 ms plus mesh upload, which I could not measure (**open**: a visible hitch is likely on that frame; moving the six roots to the background before the swap would remove it).
- The old terrain is freed on the swap (despawn of everything tagged `PlanetScene`).

## Network (step 6)

A warping ship was replayed through the real snapshot path (wire format, interpolation buffer, 150 ms playout, 30 Hz): test `crates/warp_core/tests/net.rs`.

- **Found and fixed:** the receiver rejected the snapshots as absurd (`decode` limit 1,000 km position and speed). A warping remote ship would have frozen or vanished. The limits for the ship are now 1e8 m and 2e6 m/s (the walker keeps 1e6).
- **Found and fixed:** the game's sender used planet id 0 and the spike 4 centres 200 km apart; with planets 12,500 km apart a ship on Cinder would be shown at the wrong place. The sender now uses the active planet's id. The receiver converts every snapshot to one frame (`Snapshot::to_frame_of`); without that, the buffer holds the older sample across a change of the sender's planet and the shown ship was **13.8 km** off at that moment.
- **Measured** (error of the shown position against the truth at the shown time): ramp-up max 18.3 m (mean 2.8 m), cruise 3.6 m, ramp-down 18.7 m, after arrival 11.2 m; with one packet in ten lost and 100 ms extrapolation 22.9 m at most, no holds. The error comes from interpolating an accelerating, curved path between snapshots 33 km apart. It is small and I did not touch the interval.
- **Inherent:** the shown ship is 150 ms behind the truth, at top speed **150 km**. Not an error for someone watching from far away; it matters only if someone shares a spot with the warping ship.
- **Open:** a remote warping ship near the receiver's walker or ship: the proxy carries a velocity of 1e6 m/s, which gives it the 17 km box above and a huge relative velocity in the walker's moving-collider sweeps. Never run, I did not drive the `foreign` scenario with a warp.

## What I could not do (and why)

- **Anything with a window.** The container has no GPU, no `libXcursor` and no Vulkan driver file. The windowed code for distant planets, tunnel streaks, the course ring and the HUD line compiles and nothing more; I checked the arithmetic of the impostor (position relative to the render origin) by reading, not by looking. The scenario has the hooks: run `cargo run -p exo_app -- --scenario=warp` in a window and `target/scenario/` gets screenshots per phase (`shot-*-a2b-*.png`) and the report gets the frame time per phase (`FrameLog`: orbit, ramp-up, cruise, ramp-down, arrival). **Step 2's "frame cost measured" and step 4's "first frames after exit show terrain" are open until someone runs it.** Interactive flight: `cargo dev`, fly above 1,200 m, aim the nose at the ring, J.
- **The Bevy app does not build in this container with the default features** (no `pkg-config`, no alsa, udev or wayland headers; system packages come from the roost image). I built and tested with a trimmed feature list for Bevy (`default-features = false` plus `x11`, `bevy_winit`, 3d and ui) **in the working tree only**; the commits keep the default features and the lockfile was regenerated for them (the diff to `main` only adds `warp_core` and one edge). `cargo t` and `cargo scenario` passed with the trimmed features; CI has not run.
- Frame zone swap and the walker standing outside during a warp, seats for several players, a second client: not tried.

## Checks

`cargo t`: all tests pass (warp_core 10 tests plus trip times plus the network replay, net_core with its new tests, existing suites), `cargo scenario` (`full`): 0 failures, depenetrations 910 as on `main`. `warp` scenario: 18 checks, 0 failures.

## Open (for you)

1. **Flight length for long trips** (75 s at 62,500 km, 200 s at 187,500 km): speed or events on the way.
2. **Where a warp is allowed:** now one rule, above the atmosphere top (1,200 m) of the nearest planet, anywhere, also from the surface up to 1,200 m refused. A minimum distance from a planet is not in.
3. **Group jump** (linking range, all ready): needs one seat per player and a rule for who pilots. Not built.
4. **What interrupts a warp:** only the player before the ramp-up. During the flight nothing can (no emergency exit, no interdiction).
5. **How the planets differ:** only seed (and colour of the distant sphere). Everything else is the same recipe.
6. Fuel, cost, heat, events: design questions, untouched.
7. The 15 ms root-chunk hitch on arrival and whether the target should be refined during the flight.

## Files (code repo, branch `spike/11-warp`)

`crates/warp_core/` (registry, drive, path, tests, network replay), `content/system/system.json` (all values), `crates/exo_app/src/warp.rs` (the drive in the game, planet swap), `scenario.rs` (`warp`), `terrain.rs` (rebuild for another planet), `view.rs` (distant planets, tunnel, course ring, HUD, frame log), `crates/net_core/src/snapshot.rs` (limits, frame change).
