# Spike 9b brief — agent tooling for Rust and Bevy (handoff for a new session)

Status: ready to start (spike 9 frozen as `spike/9-bevy`, `SPIKE-9-REPORT.md`). Written 2026-10-07. The initiator asked whether the Rust/Bevy agent environment needs more to be efficient, and wants BRP included "wenn es was bringt".

## Goal

Try each tool below on the spike 9 code and give it a **verdict**: keep or drop, with the number behind it (build seconds saved, turns or minutes saved on a real bug, wrong answers avoided). The result is a report with proposed lines for `AGENTS.md` and `WORKSPACE.md`. The initiator decides what goes in.

## Known so far

| Item | State on 2026-10-07 |
|---|---|
| Profiles | `opt-level` 1 for own code, 3 for deps; `fast` profile for measurements |
| Rebuild after a one-line change | 1.1 s with feature `dynamic`, 5.6–6.8 s without; off by default |
| Clean dev build | 6 min 08 s, paid again in every new worktree |
| Linker | Rust default (lld since Rust 1.90); no `.cargo/config.toml` |
| `target/` | 15 GB per worktree; every worktree builds Bevy and Avian from scratch |
| rust-analyzer | Installed via rustup, Claude Code plugin `rust-analyzer-lsp` installed; shares `target/` with cargo |
| Toolchain | rustup from pacman, not mise; no `rust-toolchain.toml` |
| Game state for agents | Own scenario runner (`--headless --scenario=...`, PASS/FAIL lines, exit code); screenshots via `--hidden` without a visible window; no live inspection |

These baseline numbers are from `SPIKE-9-REPORT.md` ("Agent loop"). Reuse them instead of measuring again. With 1.1 s rebuilds, the clean build per worktree is the build cost that matters.

## Where to work

- Code repo, branch `spike/bevy-tooling` from tag `spike/9-bevy`, own worktree `~/Work/exo-1-spike9b`.
- Install tools through mise (`mise use` with the `cargo:` or `aqua:` backend). Pin versions.
- Commit small and often. Push only after the initiator says yes.

## Steps

1. **Build speed.** Add `.cargo/config.toml` with sccache as `build.rustc-wrapper` and an alias `cargo dev` that always passes `--features dynamic`. Measure a clean build in a second fresh worktree with a warm sccache against the 6 min 08 s baseline. Try mold as linker only for the rebuild with `dynamic` (1.1 s baseline); drop it if it saves less than 0.3 s. Done when sccache and mold each have a measured number.
2. **rust-analyzer.** Give it its own target dir (`rust-analyzer.cargo.targetDir = true`) so it does not lock cargo's `target/`. Check that the LSP plugin reports a deliberately introduced type error after an edit, without a `cargo check`. Done when the round trip works and its time is noted.
3. **BRP.** Add `bevy_brp_extras` (with `RemotePlugin`) behind a dev-only feature `remote`, bound to localhost; install `bevy_brp_mcp` (0.22.x for Bevy 0.19) and register it as an MCP server. This is networking, so it is red-class: flag it in the report, keep it out of release builds, and check each dependency's licence. Then run a **blind test** on a real spike 9 bug: the cabin walker that read colliders one physics step late (fixed in `76b6319`, bug state `c4dfb0a`). Export the bug state without git history (`git archive c4dfb0a`), so the fix cannot be looked up. Two fresh sessions get the same symptom description; one has BRP, one only the scenario runner. Compare minutes, turns and rebuilds until the cause is named. Also list which parts of the MCP rule ("every feature reachable by the bot interface") BRP covers: state, input, screenshots, and what is missing. Done when both sessions have finished or hit a 45-minute limit, and the comparison is in the report.
4. **Bevy skills** ([chrisgliddon/bevy-skills](https://github.com/chrisgliddon/bevy-skills), MIT, targets Bevy 0.19). Third-party instructions: read each candidate skill completely before adopting it, and check its code examples against the Bevy 0.19.1 source in `~/.cargo/registry/src/`. Vendor only the adopted skills into the code repo's `.claude/skills/`, pinned to a commit, with the MIT notice. Candidates and exclusions are in the table below. Done when every candidate has a verdict and every wrong example found is listed.
5. **API lookup rule.** Draft one line for `AGENTS.md` that sends agents to the Bevy and Avian sources in the cargo registry and to the Bevy examples, instead of their memory of older APIs. Compare with step 4: does a skill add anything that this lookup does not?
6. **Smaller tools.** Try `cargo-nextest` and `bevy_lint` (from `bevy_cli`) once each on the spike 9 workspace. Verdict with one number each (test wall time; real findings vs noise).

Each step that fails stops only that step: write down the blocker and continue with the next.

## Bevy skill candidates

| Verdict before review | Skills | Reason |
|---|---|---|
| Candidate | `bevy-core-concepts`, `bevy-ecs-components`, `bevy-ecs-queries`, `bevy-ecs-systems`, `bevy-testing`, `bevy-rendering`, `bevy-diagnostics-profiling`, `bevy-cameras`, `bevy-migration-0-18-to-0-19`, `bevy-capture`, `bevy-porting` | Used by the current code, the agent loop (headless, profiling), clip recording, or the Godot port |
| Later, when the feature comes | `bevy-input-actions`, `bevy-ui`, `bevy-assets`, `bevy-custom-assets`, `bevy-save-load`, `bevy-pbr-materials`, `bevy-audio`, `bevy-animation`, `bevy-a11y`, `bevy-cargo-features` | No code for it yet |
| Exclude | `bevy-physics` | Teaches Rapier; the project uses Avian, so it would steer agents wrong |
| Exclude | `bevy-wasm-webgpu` | No web export (decided after spike 6) |
| Exclude | `bevy-voxel-*`, `bevy-vfx`, `bevy-fluent`, `bevy-migration-0-17-to-0-18`, `similarity-rs` | Off the project's path or simple-look rule; old version |
| Decide after vendoring | `bevy` (router) | Indexes all sibling skills; needs editing to the vendored set, or drop it |

## Not in scope

- Hotpatching (`bevy_hotpatching_experiments`): helps humans tuning values more than agents; later.
- Wild linker (pre-alpha) and Cranelift (nightly).
- Changes to game code beyond the `remote` feature.

## Results

`SPIKE-9B-REPORT.md` in this repo, learnings in `LEARNINGS.md`. One table: tool, verdict, number. Below it the proposed `AGENTS.md` and `WORKSPACE.md` lines, marked as proposals. The spike is done when every step has a verdict with a number or a named blocker.
