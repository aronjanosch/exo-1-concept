# Spike 4 brief — network (handoff for a new session)

Status: done, result in `SPIKE-4-REPORT.md`. Kept as the record of what was asked.

## Goal

Find out whether client authority works: every client simulates its own player and ship near its own origin and sends their state; the others show it smoothly with plain snapshots and interpolation, under latency and packet loss. This decides the network approach before more players are added.

Direction from the initiator (2026-10-04): "schummeln ist uns egal. Performance ist wichtig! Denke Client-Autorität." Client authority is the direction to try, not a settled decision. `FEASIBILITY.md` still proposes host authority (research proposal, written before this).

## Known so far

- Starting rules in `FEASIBILITY.md`, section "Network sync for the ship": snapshots over an unreliable channel, interpolation buffer of about 3 send intervals, Hermite interpolation with velocity, slerp for rotation, 20-30 snapshots per second and 100-150 ms buffer as first values. Godot physics is not deterministic, so no lockstep.
- Spike 1: ship is a `RigidBody3D` on Jolt with engine gravity off; gravity, drag and hover assist in `_integrate_forces`.
- Spike 3: walking inside a flying ship works up to about 400 m/s (`SPIKE-3-REPORT.md`).
- Spike 5: physics works far from the origin, but the picture needs an origin shift beyond about 50 km. With a continuous shift every client has its own origin, so network positions must be in a shared frame (planet id plus planet-relative position, or a double-precision true position). Shifts must run in `_process`. See `SPIKE-5-REPORT.md`, section "Multiplayer".

## Read first

1. `LEARNINGS.md`, then `SPIKE-1-REPORT.md`, `SPIKE-3-REPORT.md`, `SPIKE-5-REPORT.md`.
2. `FEASIBILITY.md` (network rules), `DECISIONS.md` ("one player, one ship first; more players after the core works").
3. Code repo `AGENTS.md` and `WORKSPACE.md`. Networking is a red-class path: this spike is explicitly about it, keep it inside the spike folder.

## Where to work

- Code repo `~/Work/exo-1`, own `git worktree` (several sessions run in parallel; never switch the branch of a checkout another session uses).
- New throwaway branch from `spike/combined` (spike 3 cabin and ramp plus spike 5 origin shift, committed locally, not pushed).
- Two instances on one machine (host and client over ENet on localhost). Headless where possible; windowed runs only through the wrapper in `WORKSPACE.md`. Scripted runs must not capture the mouse.
- Commit and push only when the initiator asks.

## Questions to answer

1. Client authority: each instance flies its own ship (scripted, as in the auto-test) and sends its state; the other shows it with snapshots at 20 and 30 per second and a 100-150 ms buffer. Is the motion smooth? Measure position error against the sender's true path and frame-to-frame jerk, for cruise, turns, take-off and landing. The own ship has no input delay by design; confirm.
2. Latency and packet loss: how does 1 with 50/150 ms delay, 20 ms jitter and 1/5/10 % loss behave? Simulate inside the snapshot layer (delay queue, random drop), not with `tc netem` (needs root; ask first if it seems necessary).
3. Performance: CPU time per frame and bandwidth for sending and receiving, with 2 and with 8 simulated players (extra headless clients or fake senders). Compare with host authority for one ship (host simulates, client shows it) as a reference.
4. Origin shift: send positions in a shared frame (planet id plus planet-relative position). Does a client's own shift interfere with interpolation (buffer entries before and after a shift)? Two clients on different planets, each near its own origin: both see the other correctly?
5. Contacts between players' bodies: what happens when two client-simulated ships touch or a player stands on another player's ship (spike 3)? Who owns the contact? List the problems, do not solve them all.
6. A walking player next to and inside another player's ship (spike 3): what is the minimum to keep them in sync?
7. Plain Godot (`MultiplayerAPI`, `MultiplayerSynchronizer`, own RPCs) versus netfox: only look at netfox if plain interpolation is not enough. Adding an addon is a red-class change and needs the initiator's go.

## Rules for this spike

- Technical findings only. Player count, who hosts, server or peer-to-peer, and how players meet are design questions: list them as open for the initiator. Cheating does not matter (initiator); performance does.
- Report numbers, not feelings; mark verified, measured and assumed.
- No personal data in logs, screenshots or test names.
- Results go into `SPIKE-4-REPORT.md`, learnings into `LEARNINGS.md`.
