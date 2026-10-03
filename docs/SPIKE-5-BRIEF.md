# Spike 5 brief — float limit and origin shift (handoff for a new session)

Status: ready to start. Written 2026-10-03 after spike 1. Spike 4 (network) follows in the same session afterwards; write its brief when spike 5 is done.

## Goal

Find where float32 precision breaks for our game and whether an origin shift fixes it cheaply. This decides how large planets can be and how several planets in one system work.

## Known from spike 1 (`SPIKE-1-REPORT.md`)

- Planet centred at the origin: no physics jitter standing still up to 16 km from the centre; walking works at 8 km, fails at 16 km (catches on an invisible edge, likely offset collision patches; not verified).
- Depth: Forward+ has no z-fighting to 40 km; Compatibility has z-fighting from about 500 m at cm gaps (renderer choice is open).

## Read first

1. `LEARNINGS.md` (how to work with the initiator, local environment, tech learnings).
2. `SPIKE-1-REPORT.md`, then `spikes/planet/README.md` in the code repo.
3. `FEASIBILITY.md` (precision notes, spike list), `DECISIONS.md`.
4. Code repo `AGENTS.md` and `WORKSPACE.md` (how to run Godot without stealing focus).

## Where to work

- Code repo `~/Work/exo-1` (private GitHub `aronjanosch/exo-1`). New throwaway branch `spike/origin-shift` from `spike/planet`; reuse the spike 1 code.
- Another session may work on spike 3 (leaving the ship) on its own branch at the same time. Do not edit `spike/planet`.
- Commit and push only when the initiator asks.

## Questions to answer

1. With the planet centred at the origin: where exactly do walking, landing and visuals break (radius sweep, for example 8, 10, 12, 16 km)? Is the 16 km walking failure really precision?
2. Flying out to 100 km: when does the ship or camera visibly jitter?
3. Origin shift: when the player moves far from the origin, shift the world so the player is near the origin again. Does it fix 1 and 2? What breaks (physics bodies, collision patches, LOD, threads in flight, the sky)? Cost of one shift (longest frame)?
4. Several planets: can each planet be re-centred on approach instead of a continuous shift? Which approach is simpler?
5. What does an origin shift mean for multiplayer later (note only, spike 4 tests it)?

## Rules for this spike

- Technical findings only. No game-design decisions (planet sizes, travel between planets, speeds): list them as open questions for the initiator.
- Report numbers, not feelings; mark verified, measured and assumed.
- Results go into `SPIKE-5-REPORT.md`, learnings into `LEARNINGS.md`.
