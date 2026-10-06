# Spike 7 brief — Rust extension in builds and CI (handoff for a new session)

Status: ready to start. Written 2026-10-07. Follows the decision to write the terrain generator in Rust (`DECISIONS.md`, 2026-10-06). Platforms: Linux and Windows, development on Linux; Windows-only with Linux through Proton is an option (`DECISIONS.md`, 2026-10-07).

## Goal

Show that the Rust GDExtension from spike 6 builds, exports and runs on both target platforms, from the Linux dev machine and from CI, before any game code depends on it. The answer is a working pipeline or a list of what blocks it.

## Known so far

- `spikes/gen_bench/rust/` on tag `spike/6-generator` (branch `spike/6-generator-bench`): `gen_core` (generator), `gen_godot` (class `ExoGen`, godot-rust 0.5.5, feature `experimental-threads`), `gen_bench_cli`, `exo_gen.gdextension` (Linux entries only). Checksum for 200 chunks, seed 1337: height sum 4169609.072, canopy 3021, rocks 11549.
- Cargo profile `fast` rebuilds in about 0.5 s; full `release` (LTO) about 35 s.
- Spike 4 export notes in `LEARNINGS.md`: official 4.7.2 export templates are needed (1.2 GB), presets live outside the root `project.godot`.
- Dev machine: Linux, Rust 1.98.1 with only the `x86_64-unknown-linux-gnu` target, no mingw, no Wine. Steam's "Proton - Experimental" is installed.

## Read first

1. `LEARNINGS.md` (environment, git, spike 4 export section).
2. `SPIKE-6-REPORT.md` and `spikes/gen_bench/rust/NOTES.md` on the spike branch.
3. Code repo `AGENTS.md` (risk classes: CI and native code are red; this spike is explicitly about both) and `WORKSPACE.md`.

## Where to work

- Code repo, new branch `spike/rust-builds` from tag `spike/6-generator`, own worktree `~/Work/exo-1-spike7`. Freeze tag at the end: `spike/7-rust-builds`.
- Commit small and often on the branch. Push only after the initiator says yes; GitHub Actions only run after a push.

## Steps

1. **Linux, local.** Release `.so`, Linux export with the extension, run the exported binary headless: load `ExoGen`, build the 200 chunks, compare the checksum.
2. **Windows from Linux.** Pick a cross toolchain (`x86_64-pc-windows-gnu` with mingw-w64, or MSVC through `cargo-xwin`); install through rustup and the system package manager only as far as needed, and list what was installed. Build the `.dll`, add Windows entries to `exo_gen.gdextension`, export for Windows from Linux.
3. **Windows build under Proton.** Run the Windows export through Proton on the dev machine: does the extension load, does the checksum match, how long does a chunk take compared with native Linux? This tests the Windows-only option.
4. **CI.** One GitHub Actions workflow on the spike branch: build the extension for Linux and Windows (cross-compiled on a Linux runner, or a Windows runner if cross-compiling fails), cache cargo, run the headless checksum test on Linux, upload both exports as artifacts. Pin actions to commit SHAs, read-only token, no secrets (`FEASIBILITY.md`, PR security).
5. If time is left: a native Windows check on real hardware is out of scope unless the initiator offers a machine; say so in the report.

## What to measure

- Works or not per step, with the exact blocker if not.
- Build times: cold and warm (cached) CI run, local cross build.
- Size of the `.so`/`.dll` and of each export.
- Chunk time and checksum: native Linux, Windows build under Proton.
- Effort notes: what had to be installed, which `.gdextension` and export-preset details mattered.

Report each number as measured, calculated or assumed.

## Rules for this spike

- Build and pipeline only. No game design, no generator changes beyond what building requires.
- The workflow runs on the spike branch only; nothing goes into `main`.
- Results go into `SPIKE-7-REPORT.md`, learnings into `LEARNINGS.md`.
