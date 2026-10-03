# Spike 3 report — leaving the ship

Date: 2026-10-03. Code: `~/Work/exo-1`, branch `spike/leave-ship` (from `spike/planet`). Agreed test assumptions (not designed): walkable greybox cabin with a ramp at the back, gravity inside towards the cabin floor, F at the seat to sit/stand, hover assist holds the ship when nobody sits at the controls.

## Result

The walker as a child of the ship works: the parent transform carries it, velocity is relative to the ship, and it is moved in or out of the ship's frame by a box test on the cabin.

| Test (`--auto-test`) | Result |
|---|---|
| Sit, fly, stand up in flight | Works; ship holds position |
| Stand in a ship boosting to 340-400 m/s while rolling | Walker drift in cabin 6-12 mm, on floor 100 %, stays inside |
| Walk in a ship moving at 70-95 m/s | Height in cabin 0.298-0.302 m, stays inside |
| Walk out of the landed ship | Works |
| Walk up the ramp into the ship | Unreliable: works in some runs, fails in others (see open) |

## Ship physics (changed after the initiator's feedback, 2026-10-04)

The initiator wants fairly realistic physics fundamentals: the ship braked far too fast and should glide in space. Changes: hover assist off by default (H toggles it), inertia and gravity always, quadratic air drag only in the atmosphere, Godot's default rigid-body damping (0.1/s) switched off. Measured: gliding in space 500 -> 485.5 m/s in 5 s while climbing 2391 m, which matches energy conservation (485.3 m/s expected). The walker walks in the gliding ship and stays inside. Rotation is still rate-controlled (assumption). `DECISIONS.md` still says "Flight: Arcade" and parks the Newtonian direction; the initiator decides whether that changes.

Also seen: the ship never went above exactly 500 m/s, likely Jolt's default maximum linear velocity (not verified). The test bot's altitude hold cannot handle the new momentum and crashes in the low cruise, so the later auto-test phases are not valid until the bot is improved.

## Open

- **Ramp and terrain:** the walker often cannot get up the ramp (ship tilted on a slope, ramp end above ground, or walking under the ramp where the terrain dips). Last state: the ramp is its own walker-only body (solid wedge), but the walker still passes through it; cause not found. To be done together with the initiator.
- **Design question:** should a landed ship stay level (landing legs, levelling aid) or may it stand tilted (landed at 20-30 degrees on slopes in tests)?
- Idea (initiator), to try later: run the ship interior as its own instance (separate physics space), so a walker cannot glitch out into space. The current child-of-ship approach works well for now.
- Walking out of the open back of a flying ship drops the walker into the air. Correct for the current setup; whether it is wanted is a design question.

## Notes

- Reparenting a body inside an `Area3D` re-fires the area's signals; a `call_deferred` from that handler crashed Godot 4.7.2 (segfault in `CallQueue`). A plain box test replaced the area.
- Ship and walker are on separate collision layers and the ship ignores the walker, so walking inside never pushes the ship. `platform_floor_layers = 0` avoids adding platform velocity on top of the parent transform.
