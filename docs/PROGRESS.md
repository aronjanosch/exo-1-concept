# Progress log

Dated log of what happened, for the video track (B1/B2) and for anyone joining later. Facts only: what was done, where it lives. Decisions stay in `DECISIONS.md`, learnings in `LEARNINGS.md`. Newest at the bottom.

## 2026-10-03 — Day 1: from vision to first playable spikes

**Concept (repo `exo-1-concept`)**

- 11:14 Vision written; same time `community-gate` gets its vision and roadmap.
- 11:36 Renamed to EXO-1 (working title), roadmap added, docs translated to English.
- 12:04 Roadmap split into two tracks: experiment (A) and video (B).
- 14:52 Feasibility research and core loop proposal.
- 16:01–16:44 Feasibility and core loop reworked, `DECISIONS.md` and sources list added, reference notes.
- 17:02 Spike 1 brief (planet prototype).
- 21:46 `AGENTS.md` draft and contributor workflow skills (scope-gate, feature-breakdown, vision-check, implement-slice, exo-review).
- 22:58 Spike 1 report and `LEARNINGS.md`; entries that nobody had decided were corrected back to open.

**Code (private repo `exo-1`)**

- 23:05 Repo scaffold on `main`; spike 1 code on throwaway branch `spike/planet`.
- Spike 1 (planet): seamless small planet in Godot 4.7.2, walk, board, fly to space and back, land. See `SPIKE-1-REPORT.md`. Covers spike 2 (transition) as well.
- Spike 3 (leaving the ship), branch `spike/leave-ship`: walking inside a flying ship works up to about 400 m/s; ramp boarding still unreliable. See `SPIKE-3-REPORT.md`.
- Spike 5 (float limit, origin shift), branch `spike/origin-shift` in a second worktree (`exo-1-origin-shift`): brief in `SPIKE-5-BRIEF.md`, report in `SPIKE-5-REPORT.md`. Physics works up to about 200 km from the origin; the visible limit is GPU float32 (about 1 px at 50-60 km); an origin shift in `_process` fixes it at under 0.4 ms per shift. The 16 km "precision wall" from spike 1 turned out to be the parked ship.

**How we worked (video material)**

- Several AI sessions in parallel on separate branches and worktrees, one human deciding design.
- Repeated pattern: AI drafts turned assumptions into "decisions" (example: 3 km start radius); caught and rolled back. See `LEARNINGS.md`, section "Working with the initiator".

**Raw material for the video**

- 142 screenshots from scripted test runs (17:56 on Oct 3 to 00:15 on Oct 4) in Godot `user://screenshots` (project "Planet Spike"), for example climb to space, landing, flight at 10/25/100 km.
- Measured test output: `user://spike_results.txt`, `user://spike5_results.txt`.
- These live only on the dev machine; copy what the video needs before cleaning up.
- Clip `~/Videos/exo-1-clips/2026-10-04-ship-tumbles-low-cruise.mp4` (20 s): the test bot's low cruise with the new ship inertia rolls the ship upside down; found while merging spikes 3 and 5.
- Clip `~/Videos/exo-1-clips/2026-10-04-ship-lands-into-space.mp4` (33 s): same bug, worse: the ship skims the ground upside down, then "lands" into space because down in ship space now points up.

## 2026-10-04 to 2026-10-07 — spikes 4, 6, 7, 8 and the move to Rust

- 10-04: assisted-flight experiments on `spike/assisted-flight` (findings in `LEARNINGS.md`); spike 4 (client authority, LAN) started on `spike/4-network`.
- 10-06: spike 4 closed after a two-computer LAN test, tag `spike/4-client-authority` (`SPIKE-4-REPORT.md`); client authority decided. Spike 6 benchmarked the terrain generator, GDScript versus Rust (`SPIKE-6-REPORT.md`); Rust generator and no web export decided.
- 10-07: spike 7 built the Rust extension for Linux and Windows locally and in CI (`SPIKE-7-REPORT.md`); Linux ships native. `spike/combined` became the base branch for spikes. Spike 8 built the procedural planet in a cloud run (`SPIKE-8-REPORT.md`); walking anywhere decided.
- 10-07: decision to go fully Rust with Bevy after a validation spike (spike 9, `SPIKE-9-BRIEF.md`), networking as spike 10, EXO-1 re-oriented as a private project (`DECISIONS.md`).
- 10-07: spike 9b brief (agent tooling for Rust and Bevy: build speed, rust-analyzer, BRP, Bevy skills), starts after spike 9 is frozen (`SPIKE-9B-BRIEF.md`).
