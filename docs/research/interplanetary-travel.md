# Travel between planets

Research note, 2026-10-08, for spike 11. This is not a decision. Distances, travel time and the feel of the warp stay with the initiator. Open point in `DECISIONS.md`: "Travel between planets: direction like No Man's Sky or Star Citizen, to be tried."

Tags: **verified** (source read), **from memory** (agent knowledge, not checked against a source), **calculated**.

## How other games do it

| Game | Inside a system | Between systems |
|---|---|---|
| Freelancer (2003) | Cruise engine (faster, weapons off); trade lanes accelerate the ship to near light speed along fixed routes **[verified, [Wikipedia](https://en.wikipedia.org/wiki/Freelancer_(video_game))]** | Jump gates and jump holes **[verified, same]**; a tunnel animation covers the system load **[from memory]** |
| Elite Dangerous | Supercruise from 29.9 km/s up to 2001c; speed depends on the distance to large masses, slow near a planet, fast far away **[verified, [Elite wiki](https://elite-dangerous.fandom.com/wiki/Supercruise)]** | Hyperspace jump, exit near the star **[verified, same]**; the jump tunnel covers the load **[from memory]**; other players can pull you out of supercruise (interdiction) **[from memory]** |
| No Man's Sky | Pulse drive: seconds instead of minutes, not usable close to a planet, a station or in combat **[verified, [Twinfinite guide](https://twinfinite.net/guides/no-mans-sky-how-to-fast-travel/)]**; systems are scaled down **[from memory]** | Hyperdrive with fuel, limited range per jump **[verified, [Twinfinite guide](https://twinfinite.net/guides/no-mans-sky-how-to-use-hyperdrive/)]**; warp tunnel during the load **[from memory]** |
| Star Citizen | Quantum travel: real movement at 53.6 to 283 Mm/s (0.18c to 0.94c) depending on the drive; spool-up and calibration, you must hold the course; an obstruction on the line breaks it **[verified, [Star Citizen wiki](https://starcitizen.tools/Quantum_drive)]** | Jump points (not looked at) |
| Outer Wilds | No warp. A tiny, hand-made system, real-time orbits, Newtonian flight; travel times of minutes **[verified, [Destructoid preview](https://www.destructoid.com/?p=247228)]** | One system only |
| Starfield | Menu plus loading screen **[from memory]**, often criticised as the counterexample | Same |

The pattern: **inside a system the movement is real but very fast, between systems a tunnel hides the load.** Two details are worth borrowing: speed that depends on the distance to planets (Elite), and a short spool-up with a course to hold (Star Citizen), which turns the start into a small action instead of a button press.

## What this means for us

Physics runs in f64 world space and only the render origin shifts (spike 9: 0 mm drift at 197 km, shift ≤ 0.010 ms). f64 spacing **[calculated]**: 1.5e-8 m at 1e8 m (100,000 km), 1.2e-7 m at 1e9 m, 1.2e-4 m at 1e12 m. A whole system fits in one world frame, so a warp can be **real movement** with a tunnel look. Benefits: in co-op nobody is "elsewhere", passengers ride along in the ship frame, and events on the way are possible (`CORE-LOOP.md`: no more than about 30 s without input or an event).

The alternative is the **tunnel trick**: the ship enters a local tunnel scene, the target is loaded, the ship comes out at an approach point. Distances become free, but co-op needs a "in transit" state and the snapshot frame has to know it.

## Is 200 km enough?

The 200 km between the two planet centres is a spike 4 test value, not a design. With a 5 km radius (`DECISIONS.md`) **[calculated]**:

| Distance | Second planet as seen from the first | Mean speed for a 30 s warp |
|---|---|---|
| 200 km (40 radii, about Earth-Moon) | 2.9° wide, about 6 times the full Moon | 6.7 km/s |
| 2,000 km | 0.29°, about half the full Moon | 67 km/s |
| 20,000 km | 0.03°, a bright point | 670 km/s |
| 1,000,000 km | a point | 33,000 km/s (0.11c) |

200 km is a planet and its moon. For "huge distances" between planets, 10,000 km and more fit better. With a warp the distance barely costs anything technically; it changes what you see in the sky and the speed curve. Proposal: the spike takes distance and warp duration as parameters and measures several values; the initiator picks by feel.

## Risks to measure

- Generating the target planet in time and freeing the old one (spike 8: macro bake 486–547 ms on 8 threads, chunk 1.7–2.3 ms).
- Drawing distant planets: a full mesh at 200 km, a low-detail sphere or impostor further away.
- Collisions and contacts at warp speed (Avian on a ship at 1e5 to 1e7 m/s): switch off or sweep along the line.
- Network: snapshot error when a remote ship warps (spike 10 hold error grows with speed: 15 ms at 400 m/s is 6 m).
- Arrival: exit point, speed at exit, never inside terrain.
