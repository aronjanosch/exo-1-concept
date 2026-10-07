# Spike 10 report — network in Bevy

Date: 2026-10-07. Brief: `SPIKE-10-BRIEF.md`. Code: branch `spike/bevy-network` (worktree `~/Work/exo-1-spike10`, from tag `spike/9-bevy`), `spikes/bevy/`, local only, freeze tag `spike/10-bevy-network`. Details and commands: `spikes/bevy/NETWORK.md`. Raw results: `spikes/bevy/results/` (`net-matrix.json`, `net/`, `contacts.json`, `foreign/`). Machine: Ryzen 7 5800X, localhost only. Tags: **measured**, **assumed**, **open**.

## Answer in one line

The Bevy network reaches spike 4's level in every area, with no new dependency; it is better in smoothness, the shared frame and the walker, and about the same elsewhere. Clock sync (2–7 ms in the first version) was fixed with a receive thread (0.013 ms). What is not solved: the contact problem (open design question) and the hold rule (extrapolation exists as an option, off by default, needs your decision).

## Verdicts against the Godot table

| Area | Godot (spike 4) | Bevy (measured) | Verdict |
|---|---|---|---|
| Smoothness | 30 Hz, 150 ms buffer: worst P95 4.4 mm, 0 % underruns | Same 24 cases on the recorded spike 9 `full` path (249 s, up to 400 m/s, rolling, braking from 400 m/s): worst P95 **0.17 mm**, max 35.7 mm (no loss), underruns 0 % in 22 of 24 cases and at most 0.003 % otherwise. Worst phase P95: firm brake in space 8.1 mm, cabin at 400 m/s 4.7–5.2 mm | **Better** (f64 positions; path is harder) |
| Latency and loss | 96 cases pass | All 96 cases run in `cargo test` (4 s). P95 below 10 mm in **all 96** (worst 3.1 mm, the 20 Hz cases at 400 m/s without any loss). Holds in 38 cases (up to 1.47 %, 20 Hz, 100 ms, 10 % loss); a hold at 400 m/s costs metres (max 5.8 m even at 30 Hz/150 ms, 8 players, 50 ms, 10 % loss, 0.003 % of samples) | **Same** (30 Hz/150 ms is again the default; zero holds as in spike 4 is seed luck, see below) |
| Own input | First physics tick, 16.7 ms | `input_response_ticks` = **1** in every live run (also with 150 ms delay and 10 % loss): the controller never waits for the net | **Same** |
| Bandwidth | 144 B/player; 8 players, 30 Hz: ~30 kB/s in per client, ~183 kB/s out at host (ENet) | 144 B payload (146 B datagram, 174 B with IPv4/UDP). 8 players: **29.6 kB/s** payload in per client (35.9 kB/s on the wire), host **180 kB/s** payload out (218 kB/s on the wire) | **Same** (plain UDP has less framing than ENet, I count the UDP/IP headers) |
| CPU | 8 players: process mean ~0.5 ms/frame on the receiver | 8 players, receiver: network systems (receive, decode, buffers, 7 proxies, walker markers) **0.035 ms** mean, 0.05 ms P95; send 0.014 ms; host 0.145 ms (receive plus relay). Whole process (all threads, physics, terrain ring) 3.2–3.6 ms CPU per tick, 2 players 2.6 ms. Avian step 0.54–0.80 ms | **Better** for the network part (about 15x), the rest is the game itself |
| Shared frame | Planet id plus f64; 0 mm jump over 48 shifts; planets 200 km apart | Same wire format. Live: clients on planets 0 and 1 (200 km), render-origin shifts every 200 m plus one forced 10 km shift during interpolation: **jump ≤ 0.016 mm** (0 for whole-metre shifts at 200 km), physics side exactly 0 (bodies never move). The only error is what the GPU gets: f32 transform of a proxy 200 km from the render origin, up to **7.8 mm** (one f32 step), about 0 px at that distance; near proxies ≤ 0.034 mm | **Better** (the float32 caveat of spike 4 section 4 is gone; the 7.8 mm is render-only) |
| Walker in a foreign ship | Drift 0.000 m at 350 m/s, deck contact 358/360 | Real walker on the kinematic proxy, 350 m/s translating and yawing, interpolated through the snapshot path, 180 render-origin shifts: drift **0.0000 m**, deck contact **360/360**, lowest feet 0.310 m, proxy against exact path 0.0000 mm. Walking inside works; walking beside a parked foreign ship 24/24 grounded; wire round trip of the pose passes | **Same or better** (360/360 against 358/360) |
| Contacts | 217 ms apart, up to 3 m disagreement, open | Head-on 15 m/s, two independent Avian worlds, delayed kinematic proxies, same lags (150 and 50 ms): contact **3 ticks (50 ms) apart**, max proxy/authority disagreement **3.83 m**, final states differ (A is at x −19.7 in its own world, B at +29.6 in its own world; A's proxy in the other world is 1.2 m from A). With equal lags the contact ticks agree but the outcomes still differ (3 ticks: final vx −4.5 / −0.1). Problems listed below | **Same** (open, not solved; numbers not comparable 1:1, different engine and contact response) |
| Transport | Godot RPC over ENet, own buffer, no netfox | **Plain UDP from `std::net`**, 0 new crates. `net_core` has only `glam`. Own handshake (hello retried until accepted), ping/pong, host relay, takeover of a silent slot after 1 s, 2 s expiry. 2 hosts/clients on Windows (Proton) and Linux interoperate | **Same** |

## Steps

1. **Transport (done).** Plain UDP. Reason: the brief asks for the thinnest option for unreliable snapshots and a host relay; `std::net` needs no dependency, no async runtime, no plugin, nothing replicates by itself (so client authority is not in conflict with a framework). **Assumption**: I did not install or compare `renet2`, `bevy_replicon` (server-authoritative by design) or `lightyear` (prediction/rollback not needed); the choice rests on their documentation and on the fact that UDP was enough. New dependencies: **0** (licence list unchanged from spike 9). What UDP does not give: connection state beyond my own hello/expiry, congestion control, NAT traversal, encryption (all out of scope). Two headless processes exchange snapshots on localhost; so do 8.
2. **Replay matrix (done).** `net_core` tests: 12 unit checks (146-byte datagram, 144-byte snapshot, rejection of truncated/non-finite/absurd/unknown-owner/parentless-frame, Hermite, reorder, duplicate, hold without extrapolation, frame and planet transitions, bounded history, rejoin reset, f64 at 200 km, link determinism, clock) and the 96-case matrix on `results/full-path.bin` (the recorded `full` scenario, ship and walker, 14,946 states). Assertion: 30 Hz/150 ms cases P95 < 10 mm and holds < 0.05 %. Holds: spike 4 asserted exactly 0; with my RNG two of the 24 cases have 0.003 % (four lost snapshots in a row at 10 % loss), so the bound is small, not zero. The RNG differs from Godot's, so loss patterns differ; case-by-case comparison with spike 4 is not possible, the statistics are.
3. **Live processes (done).** `tools/net_run.py`: host plus clients, headless, 60 ticks/s paced, scripted flight (take off, cruise at boost, turn, brake, descend, land), 40 s. All exit codes 0, 0 invalid packets.

   | Run | holds (client) | net systems mean/P95 ms | payload in/out kB/s (client) | host out kB/s |
   |---|---|---|---|---|
   | 2 players | 0 % | 0.015 / 0.021 | 4.3 / 4.3 | 3.7 |
   | 2 players, 150 ms ±20, 5 % loss | 0 % | 0.012 / 0.018 | 4.3 / 4.3 | 3.7 |
   | 8 players | 0–0.037 % | 0.035 / 0.052 | 29.6 / 4.3 | 180 |
   | 8 players, 150 ms ±20, 10 % loss | 0.02–0.08 % | 0.035 / 0.050 | 29.6 / 4.3 | 180 |

   Godot: 8 players 0–0.092 % holds; those include a host drain, mine are counted only when a stream resumes. Process CPU on this machine includes terrain and collision threads and cannot be compared with Godot's script time.
4. **Shared frame and origin (done).** See the table row. 2 and 8 processes, planets 0/1 alternating, `--force-shift` during flight.
5. **Walker in a foreign ship (done).** Representation as in spike 4: frame kind (planet or ship), frame id (owner of the ship), pose and velocity in that frame, repeated in every snapshot. Observers compose with the ship sampled at the same playout time. On top of the spike 9 cabin code the walker now works in any cabin (own ship or proxy); enter by box test, **B** puts it into the nearest remote ship (test placement). The proxy colliders are moved by Avian one step late like the own ship's; the walker uses the floor collider's own pose (spike 9 workaround), no new workaround.
6. **Contacts (done).** `exo_app/tests/contacts.rs`, lag table in `results/contacts.json`. **Problems for the initiator, no rule invented:**
   - Each side detects the contact at a different tick and simulates a different response; the kinematic proxy pushes the local ship back to the proxy's velocity and takes no impulse, so momentum is not conserved across the two worlds.
   - The final states diverge (up to several m and 10+ m/s) and nothing brings them together again; with 300 ms lag the proxy disagreement is 5.1 m.
   - Equal lags do not help: same detection tick, different outcome.
   - Who owns a contact, a docking, a passenger on a pushed ship, and what a client does when the owner says otherwise, are all undecided.
7. **Two computers (prepared).** Linux and Windows release builds (`build/spike10/exo-spike10-{linux,windows}.zip`, about 31 MB each zipped; exe 129/118 MB), commands and acceptance steps in `NETWORK.md`. A Windows client under Proton joined a Linux host (20 s, 59.4 ticks/s, 0 invalid). Reconnect checked locally with three processes: same slot rejoins after 1 s, the host accepts the new address, the observer shows no ghost. The initiator runs flight, passenger carry, independent shift and reconnect on two computers.

## Findings

- **Clock sync was the weakest part, now fixed.** First version: offset error against the true wall-clock offset 2.3 ms (2 players) to 7.4 ms (8 players), because ping and pong were handled once per 60 Hz tick on both ends (2.8 m of relative position at 400 m/s). Fix: a receive thread (`std::thread`, no dependency) stamps the arrival time at once and the host answers pings itself. Measured: **0.013 ms** (8 players), **0.001 ms** (2 players, 150 ms delay, 5 % loss); holds 0 %, net systems 0.028 ms. Windows binary rebuilt with it.
- **Display-only extrapolation fixes the hold error (option, default off).** `--extrapolate=<ms>`: during an underrun the last state moves on with its velocity for at most that long (rotation held, never for collision). Replay on the recorded path: the 5.8 m maximum of 30 Hz/150 ms/8 players/10 % loss becomes **35.7 mm** with 50 ms; at 20 Hz/100 ms/150 ms delay/10 % loss, 8 players, 35 m / 52 m maximum becomes 58 mm / 436 mm with 100–200 ms (50 ms is not enough there); P95 unchanged. Live run, 8 players, 150 ms/10 % loss, 100 ms: 0 invalid, holds 0.01–0.06 %. Spike 4's rule is "hold, no extrapolation": this is an exception for display only; **decision for the initiator**. Data: `results/extrapolation.json`.
- **Hold costs metres at speed.** An underrun is held, not extrapolated (spike 4 rule); at 400 m/s a 15 ms hold is 6 m. Spike 4's paths were slower.
- **Avian: a kinematic body also integrates its velocity.** The proxy gets Position and Velocity from the sample every tick; without velocity a contact would see a standing wall.
- **Proton:** `proton run` swallows stdout and `Z:` output paths made the run fail silently; use relative paths and result files.

## Assumptions

Transport choice (above); 30 Hz, 150 ms buffer, 144-byte format, slots 1–8, 2 s expiry, planets at 0 and 200 km (spike 4 values); the recorded path is the spike 9 scripted bot; bot flight for live runs (the same kind, different from the matrix path); proxies collide as kinematic bodies; fault injection is receiver side after the real UDP; live runs use `--origin-shift=200` to get shifts.

## Open

- Contact, docking and passenger ownership (initiator).
- Clock sync accuracy (above).
- Manual play on two computers (not run by me); feel of 150 ms-old remote ships.
- Process CPU is read from `/proc` and is 0 on Windows.
- Windows rendering untested; only the headless client under Proton ran.
- Remote walker capsule and proxy hull are shown in the window, but I did not look at a windowed multi-process run.
- `bevy_replicon`, `renet2`, `lightyear` not tried (assumption above).
