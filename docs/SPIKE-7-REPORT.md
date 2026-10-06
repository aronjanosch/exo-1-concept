# Spike 7 report — Rust extension in builds and CI

Date: 2026-10-07. Status: steps 1-3 done locally, step 4 (CI) written but not run (needs a push, waiting for the initiator). Code: branch `spike/rust-builds` in worktree `~/Work/exo-1-spike7`, `spikes/rust_builds/` and `.github/workflows/spike7-rust-builds.yml`. Tags: **measured**, **assumed**.

## Answer in one line

The Rust extension builds for Linux and Windows on the Linux dev machine, both exports load it and reproduce the spike-6 checksum, and the Windows build runs under Proton a few percent slower than native Linux.

## How it works

- `spikes/rust_builds/build_export.py`: cargo release build for Linux, cross build for Windows (`x86_64-pc-windows-gnullvm`, linker from llvm-mingw 20260922 through mise, no root, no Microsoft SDK), then a small check project (`check.gd`: load `ExoGen`, build the 200 spike-6 chunks, compare the checksum, write `spike7_result.json` next to the binary, exit 0 or 1) exported for both platforms with the official 4.7.2 templates.
- The Windows DLL imports `libunwind.dll` from llvm-mingw. The `.gdextension` lists it under `[dependencies]`, so the export copies it next to the `.exe`.
- `cargo-xwin` (MSVC target) was not used: it downloads the Microsoft CRT and SDK and accepts their licence on the user's behalf. Not needed while llvm-mingw works.

## Numbers (measured, dev machine)

| | Linux native | Windows build under Proton Experimental |
| --- | --- | --- |
| Checksum (3 runs) | ok | ok |
| Chunk, single thread | 0.58-0.60 ms | 0.62-0.63 ms |
| Macro bake, all threads | 42-52 ms | 53-62 ms |

- Cold Windows cross build: 92 s. Export per platform: 2.6 s.
- Sizes: Linux export 74 MB + `libgen_godot.so` 3.3 MB; Windows export 109 MB + `gen_godot.dll` 4.0 MB + `libunwind.dll` 0.19 MB. Most of it is the Godot template.

## Problems found

- The first `godot --headless --import` of a fresh project aborts on exit (SIGABRT, exit -6) although the extension is registered. The script checks `.godot/extension_list.cfg` and continues. Cause not investigated.

## CI (written, not run)

Linux runner: cache cargo and tools, download Godot 4.7.2, templates and llvm-mingw, build and export both platforms, run the Linux check, upload both exports. Windows runner: download the Windows export and run the check on real Windows. Actions pinned to commit SHAs, read-only token, no secrets, spike branch only. The repo is private, so runs use the account's Actions minutes (Windows minutes count double). First run expected around 10-15 min because of the 1.2 GB templates (assumed).

## Not covered

- Rendering, input and feel under Proton; this was a headless compute check.
- Native Windows on real hardware (the Windows runner would be a VM).
- Debug builds of the extension, editor hot reload, export of the full game project.
