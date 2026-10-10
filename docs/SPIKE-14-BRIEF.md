# Spike 14 brief — seed-based planet terrain at 30 / 100 / 300 km

Written 2026-10-10 for one long run on the NAS. The spec is code repo issue **#177**; this brief adds where it runs, the order, and when to stop. The initiator reviews the branch and the report, then flies it on the desktop and decides the radius.

## Where it runs

The NAS (herdr machine `roost`, container user `agent`), as a `/goal` run in auto mode, in its own worktree and target dir:

```sh
git -C ~/projects/exo-1 fetch --prune origin
git -C ~/projects/exo-1 worktree add ~/projects/exo-1-spike14 -b spike/planet-seed-lod origin/main
git -C ~/projects/exo-1-concept pull --ff-only
source ~/.cache/exo-buildenv/env.sh
export CARGO_TARGET_DIR=~/.cache/exo-1-target-spike14 CARGO_BUILD_JOBS=6
```

Before every `cargo` command: `source ~/.cache/exo-buildenv/env.sh` and the two variables. One cargo command at a time, under `nice`. The NAS has no GPU: headless only. Frame time, pop-in and screenshots go into the report as an open list for the desktop.

## Read first

1. Code repo `AGENTS.md` and `WORKSPACE.md`.
2. Concept repo (read only): `docs/DECISIONS.md` rows "Flight is the foundation", "Planet scale (direction)", "Planet terrain architecture (direction)"; `docs/research/worldgen-and-lod.md`; `docs/research/scale-and-early-game.md` appendix C (where the radius enters `planet_core`, what breaks); `docs/research/procedural-planet.md`.
3. Issue #177 (`gh issue view 177`): direction, tests first, done criteria.
4. What is built: `crates/planet_core` (recipe, macro fields, landforms, drainage, sites, places) and `crates/exo_app/src/terrain.rs`, `ring.rs`. Carry the tested functions over (noise, landform stamps, carve); replace the fine global bake, do not keep it as a second path.

## Order

Test-first in `planet_core`, one commit per working step, push the branch after each step.

1. Radius and atmosphere as plain parameters end to end (`--radius=`, `atmosphere_height`, ceilings), the fixed 5 km offsets in scenarios and the orbit camera fixed (#177 "Fix what breaks").
2. Coarse global layer: bake, cache on disk under a hash of recipe, seed and code version, load from cache. Determinism and cache tests.
3. Drainage on the coarse layer, rivers as polylines with width; carve per chunk without seams.
4. Fine detail per quadtree chunk on threads, evicted when far; landforms as functions.
5. Sites and landforms by density rules in metres (spacing, separation, frequency), counts growing with area.
6. Skirts on the chunk edges.
7. Headless measurements at 30, 100 and 300 km (memory, first bake, cached load, chunk generation time) and the 1000 km dry run of the coarse layer.

Design gaps: pick a marked placeholder (`TODO(initiator)`), note the assumption as a comment on #177, go on.

## Rules

- `cargo test -p planet_core` and the touched scenarios (`cargo ts scenario_<name>`) green at every pushed step.
- Push only `spike/planet-seed-lod`. Never: a push to `main`, a PR, a merge, a tag, a closed issue, a new dependency (ask-first list of `AGENTS.md` = skip).
- Do not touch the other worktrees on the NAS or their target dirs, and leave the other herdr panes and agents alone.
- Invented names only, no personal data.

## Report

`SPIKE-14-REPORT.md` at the root of the code repo, committed on the branch: a checklist of the seven steps with status; per step what was built and the checks with numbers; the measurement table per radius against today's 5 km; what the full rebuild of `planet_core` costs; the open list for the desktop (frame time, pop-in in a NAV approach from orbit, screenshots, flight minutes between sites); a recommended radius and atmosphere. A short comment on #177 links it.

## Stop

Stop when all seven steps are done and the report is pushed, or when nothing more can move without the initiator, a GPU or a protected action (then the report says which). Do not stop earlier for any of these: a summary that announces the next step without doing it, offering to continue, listing decisions that do not block, reporting after a step.

## Goal condition (for `/goal`)

> The spike run of `docs/SPIKE-14-BRIEF.md` (concept repo) has ended by its stop rule: either all seven steps of its order are checked in `SPIKE-14-REPORT.md`, or the report states that nothing more can move without the initiator, a GPU or a protected action and names why. In each case the branch `spike/planet-seed-lod` is pushed, the last printed `cargo test -p planet_core` on the pushed tip exits 0, `SPIKE-14-REPORT.md` with the measurement table for 30, 100 and 300 km is committed and pushed, and a comment on issue #177 links it. Finishing a single step does not meet this goal. Never: a push to `main`, a PR, a merge, a tag, a closed issue, a new dependency.
