# Spike 10 brief — network in Bevy (handoff for a new session)

Status: ready to start. Written 2026-10-07. Networking as spike 10 was decided with the move to Bevy (`DECISIONS.md`, "Engine", initiator: "Netzwerk als Spike 10"). Client authority is decided ("Multiplayer authority", after spike 4).

## Goal

Rebuild what spike 4 showed in Godot, now on the spike 9 Bevy code, and give each area a **verdict**: better, same or worse than Godot, with the number behind it. Client authority: every client simulates its own player and ship, a host relays snapshots, the others interpolate. The result is a report; the initiator decides what carries over into the real code.

## Known so far (Godot reference, all from `SPIKE-4-REPORT.md`)

| Area | Godot result |
|---|---|
| Smoothness | 30 Hz, 150 ms buffer: worst P95 position error 4.4 mm over all cases, 0 % underruns |
| Latency and loss | 96 replay cases (2/8 players, 20/30 Hz, 100/150 ms buffer, 0/50/150 ms delay ±20 ms jitter, 0/1/5/10 % loss) all pass |
| Own input | No added wait on the own ship: first physics tick (16.7 ms at 60 Hz) |
| Bandwidth | 144 bytes per player (ship plus walker); 8 players at 30 Hz: about 30 kB/s in per client, about 183 kB/s out at the host |
| CPU | 8 players: process mean about 0.5 ms per frame on the receiver |
| Shared frame | Planet id plus planet-relative f64; 0 mm jump over 48 origin shifts; two clients on planets 200 km apart |
| Walker in a foreign ship | Drift 0.000 m at 350 m/s, deck contact 358/360 ticks |
| Contacts | Both sides detect a head-on contact 217 ms apart, up to 3 m disagreement: **open design question**, not solved |
| Transport | Plain Godot RPCs over ENet, own snapshot buffer (Hermite plus slerp); netfox not needed |

What is different in Bevy: physics already runs in f64 world space and only the render origin shifts (spike 9), so the float32 caveats of spike 4 section 4 should disappear. Measure that, do not assume it.

## Read first

1. `LEARNINGS.md` (spike 9 and 9b sections), `SPIKE-9-REPORT.md`, `SPIKE-4-REPORT.md`.
2. The spike 4 snapshot code on tag `spike/4-client-authority` (`spikes/network/`), as the behaviour to port.
3. Code repo `AGENTS.md` (`main`) and `WORKSPACE.md`. This spike is explicitly about networking, which is on the "ask first" list: every new dependency goes into the report with its licence.

## Where to work

- Code repo, new branch `spike/bevy-network` from tag `spike/9-bevy`, own worktree `~/Work/exo-1-spike10`. Freeze tag at the end: `spike/10-bevy-network`.
- Commit small and often. Push only after the initiator says yes.

## Architecture rule

Snapshot format, interpolation buffer, clock sync and fault injection live in a `net_core` crate without Bevy types, in f64, tested with `cargo test` (replay matrix without sockets). The Bevy crate does transport, proxies and rendering.

## Steps

1. **Transport.** Pick the thinnest option that carries unreliable snapshots and a host relay. Candidates: plain UDP from `std::net`, `renet2`, `bevy_replicon` (server-authoritative by design, so check it fits client authority first), `lightyear` (0.28–0.29 for Bevy 0.19, prediction and rollback we do not need). The choice is an assumption; write the reason and the dependency count into the report. Done when two headless processes exchange snapshots on localhost.
2. **Replay matrix.** Port the 96 cases of spike 4 into `net_core` tests against the recorded spike 9 `full` scenario path. Same metrics: RMS, P95, max position error, P95 frame-step error, underruns. Done when all 96 cases run in `cargo test` with a pass bound of 10 mm (spike 4's fixture bound).
3. **Live processes.** 2 and 8 headless processes on localhost (host plus clients), scripted flight. Measure CPU per frame, bandwidth sent and received, holds. Done when both runs have numbers next to the Godot table.
4. **Shared frame and origin.** Two clients on planets 200 km apart, each with its own render origin, plus a live origin shift during interpolation. Done when the jump at a shift and the position error at 200 km are measured.
5. **Walker in a foreign ship.** Port the spike 4 representation (walker frame kind, owner, pose in that frame) on top of the spike 9 cabin code. Measure drift and deck contact at 350 m/s.
6. **Contacts.** Reproduce the head-on case and measure the disagreement. List the problems for the initiator; invent no ownership rule.
7. **Two computers.** Prepare a Linux and a Windows build (spike 7 toolchain) and the commands for a LAN test with the initiator: flight, passenger carry, independent shift, reconnect. The initiator runs it.

Each step that fails stops only that step: write down the blocker and continue with the next.

## Not in scope

- Contact ownership, player count, who hosts, how players meet: design questions, list them as open.
- Prediction, rollback, lockstep, anti-cheat (initiator, 2026-10-04: "schummeln ist uns egal. Performance ist wichtig!").
- Internet play (NAT, relay servers). LAN and localhost only.

## Results

`SPIKE-10-REPORT.md` in this repo, learnings in `LEARNINGS.md`. The spike is done when every row of the Godot table has a verdict with a number or a named blocker. One line at the top: does the Bevy network reach spike 4's level, and where not.
