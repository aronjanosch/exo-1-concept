# Spike 9b report — agent tooling for Rust and Bevy

Done 2026-10-07. Code: branch `spike/bevy-tooling` (from tag `spike/9-bevy`), worktree `~/Work/exo-1-spike9b`, not pushed. Machine: 16 cores, btrfs, Rust 1.98.1, Bevy 0.19.1, Avian 0.7.0. Baselines are those in the brief (clean dev build 6 min 08 s = 368 s, one-line rebuild with `dynamic` 1.1 s).

## Verdicts

| Tool | Verdict | Number |
|---|---|---|
| sccache (rustc-wrapper) | **Drop** | Cold cache: 449 s (+22 % over the 368 s baseline). Fresh worktree, warm cache: 389–394 s (no gain). Same path, warm cache: 54 s. |
| Shared `CARGO_TARGET_DIR` for all worktrees | **Keep** (new, found while measuring sccache) | Fresh worktree builds in 8.7 s instead of 368 s |
| mold, rebuild with `dynamic` | **Drop** | 0.93–0.95 s against 1.09–1.14 s: saves 0.18 s (limit was 0.3 s) |
| `cargo dev` alias (`run -p exo_app --features dynamic --`) | Keep | trivial, `.cargo/config.toml` |
| rust-analyzer via the LSP plugin | **Keep for navigation, needs a wrapper; no error feedback after edits** | Own LSP client: type error after 1.0–1.1 s (pull diagnostic). Plugin: symbols work, **no diagnostic after an Edit** |
| BRP (`bevy_brp_extras` + `bevy_brp_mcp` 0.22.10) | **Keep as opt-in feature, not in the default setup** | Blind test: 0 of 1 BRP session used it; both found the cause. BRP session 345 s / 21 turns / $0.38, scenario-only 107 s / 20 turns / $0.32 |
| Bevy skills: `bevy-ecs-queries`, `bevy-testing` | **Adopt** (vendored verbatim) | 28 examples checked, 0 wrong |
| Bevy skills: `core-concepts`, `ecs-components`, `ecs-systems`, `rendering`, `cameras`, `migration-0-18-to-0-19`, router `bevy` | **Adopt after fix** (not vendored) | 8 of ~30 wrong in `ecs-systems`, 1–3 each in the others |
| Bevy skills: `diagnostics-profiling`, `capture` | Adopt after trim, low priority (not vendored) | 0 wrong of 25; one runtime hazard (derived from source, not run) |
| Bevy skill `porting` | **Drop** | ~13 of ~60 examples wrong |
| `bevy-physics` (excluded in the brief) | Exclusion confirmed | Rapier only |
| API lookup rule | Proposed line below | Review only, no A/B run |
| cargo-nextest 0.9.146 | **Drop** | `cargo nextest run` 11.9–12.1 s against `cargo test` 12.7 s (the 11.3 s scenario test dominates) |
| bevy_lint | **Blocker** | No release for Bevy 0.19, see step 6 |

## Step 1: build speed

- sccache 0.18.0 and mold 3.0.0 installed with `mise` (project `mise.toml` in `spikes/bevy`, pinned). The first attempt to use the mise **shim** as `build.rustc-wrapper` fails: cargo runs `rustc` in directories outside the mise config, the shim answers "No version is set for shim: sccache" and every crate fails. The real binary directory must be on `PATH` (`mise activate` or `mise env`).
- Clean `cargo build -p exo_app --features dynamic`:

| Run | Wall | Cache hits / misses |
|---|---|---|
| A: empty cache | 449 s | 0 / 378 |
| B: second worktree (other path), warm cache | 394 s | 259 / 119 |
| C, D: other paths, `SCCACHE_BASEDIRS=/home/aron/Work` | 389 s, 389 s | 259 / 119 |
| B again, same path | 54 s | 378 / 0 |

  The 119 misses depend on the worktree path; `SCCACHE_BASEDIRS` did not change them. Cause not found: the proc-macro libraries are byte-identical between worktrees and contain no path. Not pursued further because of the next point. sccache cannot cache proc-macros and the dylib (86 calls, "crate-type") and incremental builds.
- Shared target dir: `CARGO_TARGET_DIR=<target of an existing worktree> cargo build -p exo_app --features dynamic` in a fresh worktree took **8.7 s** (only the workspace crates rebuild). Fingerprints of registry crates do not depend on the path. Costs: switching worktrees rebuilds the own crates (about 8 s), two simultaneous cargo runs wait for each other's lock, and one 15 GB target instead of one per worktree.
- mold: needs `-C link-arg=-fuse-ld=mold` with mold on `PATH` (an absolute path fails with gcc 16). It changes `RUSTFLAGS`, so the first build is a full rebuild. One-line rebuild (touch `exo_app/src/lib.rs`), interleaved, 3 pairs: default linker 1.14 / 1.09 / 1.11 s, mold 0.94 / 0.93 / 0.93 s. Below the 0.3 s limit, dropped.

## Step 2: rust-analyzer

- **Blocker found and fixed:** pacman's `rustup` installs no `rust-analyzer` proxy (`/usr/bin` has cargo, rustc, rustfmt, rustup, but no rust-analyzer), so the plugin `rust-analyzer-lsp` (command `rust-analyzer`, no options) cannot start it. Fix outside the repo: `~/.local/bin/rust-analyzer` runs `rustup run stable rust-analyzer`.
- `rust-analyzer.cargo.targetDir` is a global key: neither `rust-analyzer.toml` in the repo nor the plugin (it has no `initializationOptions`) can set it. The wrapper therefore sets `CARGO_TARGET_DIR=~/.cache/rust-analyzer-target/<cwd>`. Checked: rust-analyzer's `cargo check` now writes there and no longer into `target/` (the `target/flycheck0` directory stopped appearing). Cost: 583 MB for the check build of the workspace, a cold start with the fresh dir took 41.6 s until quiescent (9 s warm).
- Round trip with my own LSP client (Python, same protocol): index 9 s warm, then a deliberate `let x: u32 = "text";` sent by `didChange` was reported 1.0–1.1 s later as "expected u32, found &'static str". It comes as a **pull** diagnostic (`textDocument/diagnostic`), native, with no `cargo check`.
- **Plugin round trip (tested later in a session with the LSP tool):** the server starts through the plugin, `documentSymbol` works and sees the edit (the new function appeared). After an Edit that introduced `let y: u32 = "text again";` **no diagnostic was shown**, so the plugin does not replace `cargo check` for errors. Likely cause (inferred from the behaviour, plugin not read): rust-analyzer only answers pull diagnostics, which my own client had to request explicitly. `hover` on the local variable returned nothing (position may have been off, not investigated). The test line was reverted.

## Step 3: BRP

- `bevy_brp_extras = "=0.22.10"` (MIT OR Apache-2.0, Bevy 0.19), optional dependency behind feature `remote` of `exo_app`, added only when not `--headless`. **Red-class (networking):** off by default, `remote` must never be in a release build; the HTTP server binds `127.0.0.1:15702` (`bevy_remote` `DEFAULT_ADDR`, checked in source and with `ss -ltn`). About 30 packages added or newly enabled with the feature (`cargo tree` against the base tree), all MIT or Apache-2.0 or both: bevy_remote, hyper, http, tokio, schemars, strum, tempfile, ref-cast, and so on.
- Game types are not reflected, so BRP could only see Bevy and Avian components. I added `Reflect` behind the feature to `Player`, `Ship` (non-reflectable fields ignored) and `WalkStats`. Cost per type: three attributes.
- `bevy_brp_mcp` 0.22.10 installed with `mise use cargo:bevy_brp_mcp@0.22.10` (compiles from source, 557 s). Registered in `.mcp.json` at the repo root; stdio handshake works, 52 tools. Running `--hidden --scenario=full` windowed: `world.query` returns the player, `brp_extras/screenshot` writes a PNG with the window invisible.
- **Blind test.** Bug state: the tree of `76b6319` with only the fix removed (walker frame taken from the ship body pose again, the `CabinFloor` marker removed), exported without history. Deviation from the brief: `c4dfb0a` itself cannot be used, its `full` scenario fails earlier for other reasons; this state shows exactly the symptom (`standing drift 0.3144 m ... left ship true`, grounded 72.3 %). Two fresh sessions (`claude -p`, Sonnet 5.5, same prompt with symptom, no repo hints, own copy of the build cache), one with the `bevy_brp` MCP server and the windowed run explained, one with the scenario runner only. Run at the same time, so wall times carry noise.

| | Scenario runner only | With BRP |
|---|---|---|
| Wall time to named cause | 107 s | 345 s |
| Turns | 20 | 21 |
| Cost | $0.32 | $0.38 |
| Cargo runs | 5 (4 full scenario runs) | 4 (4 scenario runs) |
| Cause named | yes: collider tree one tick behind the body pose | yes, same |
| Fix shown to work | yes (shifted frame: drift 0.0000 m, left ship false) | yes, same |
| BRP used | – | **no**: "scenario runs were faster" |

  n = 1 per arm. What it supports: for a deterministic bug that the scenario runner reproduces in 15 s, BRP adds nothing, and the extra 52 tool descriptions cost time and money. What it does not support: any claim about bugs that only show in a running window (visuals, input feel, timing under load). Both sessions inferred the one-tick lag from evidence and said so; neither read Avian's source.
- **Coverage of the rule "every feature reachable by the bot interface":**
  - State: reflected components and resources, read and write (`world.*`, `world_get_resources`, `world_mutate_*`), watches. Only for types with `Reflect`.
  - Input: keys, text, mouse, gamepad (`brp_extras/send_keys` and others). Our scenarios feed `Controls` directly, which BRP cannot reach.
  - Screenshots: yes, also with an invisible window (tested).
  - Missing: headless mode (BRP needs the windowed app, so CI needs a GPU), stepping or pausing physics ticks, assertions (PASS/FAIL, exit code), determinism (real-time frame rate instead of fixed ticks), reflection of game types is manual work.

## Step 4: Bevy skills

Source `chrisgliddon/bevy-skills` @ `b1b4da5744ebbd5c526342b2351967411cd5ca61` (MIT). A subagent read every candidate completely (including `references/` and scripts) and checked examples against the sources in `~/.cargo/registry/src/` (Bevy 0.19.1). No prompt injection or off-topic instructions. Off-topic only: `sudo apt install` and `cargo install` hints in docs. When vendoring, copy only `skills/<name>/`, not the source repo's `.dex/config.toml` (GitHub sync with a token). Full notes (German, model output): `docs/SPIKE-9B-SKILLS-REVIEW.md`.

| Skill | Checked / wrong | Verdict |
|---|---|---|
| bevy-ecs-queries | 12 / 0 | adopt, vendored |
| bevy-testing | 16 / 0 (matches `TimeUpdateStrategy`, `Screenshot` use in our code) | adopt, vendored |
| bevy-diagnostics-profiling | 14 / 0, 1 hazard | adopt after trim |
| bevy-capture | 11 / 0 (partly via docs.rs, crate not in registry) | adopt, low priority |
| bevy-core-concepts | 9 / 1 | after fix |
| bevy-ecs-components | 10 / 2 | after fix |
| bevy-ecs-systems | ~30 / 8 | after fix |
| bevy-rendering | 14 / 1 | after fix |
| bevy-cameras | 14 / 1 | after fix |
| bevy-migration-0-18-to-0-19 | ~35 / 3 | after fix |
| bevy (router) | 16 / 1 | edit to the vendored set (lines listed in the review) or drop |
| bevy-porting | ~60 / 13 | drop |
| bevy-physics | – | excluded, confirmed: Rapier only |

Wrong examples found (all verified in source):
- `ecs-systems`: run conditions are system functions and are passed without parentheses (`resource_exists::<R>`, not `resource_exists::<R>()`; SKILL.md:3, :109, run-conditions.md:7-14, :23, :29, :59; `condition.rs:730`). `OnTransition` has fields `exited` and `entered`, not `from` and `to` (`transitions.rs:34`). Missing `LogLevel` import in ordering.md. "Freeable states" do not exist.
- `ecs-components`: `SetEntityEventTarget` is in `bevy::ecs::event`, not `...::entity`.
- `rendering`, `migration`: `PrepareViewAttachments` does not exist (`CreateViews`, `Specialize`, `PrepareViews`). `migration`: `FontSource::family(..)` does not exist, it is `FontSource::Family("..".into())` (the error is in the official guide too).
- Router, `components`, `migration` (prose): inserting an existing resource type on another entity does not move ownership; the new value is dropped (`resource.rs`, `IsResource::on_insert`).
- `core-concepts`: `SimpleExecutor` removal and `State::set` behaviour are 0.18 changes listed under "0.19 gotchas".
- `diagnostics-profiling` (hazard, from source): `bevy/trace_tracy` already makes `RenderPlugin` add `RenderDiagnosticsPlugin` (`bevy_render/lib.rs:381`); the skill also tells you to add it, which should panic.
- `porting`: `Volume::new`, `trigger_targets`, `&AudioSink` with `set_volume`, feature `"zstd"`, `LayoutAlgorithm::Flex`, `vel.linvel`.
- Conflicts with the project: Rapier instead of Avian (`rendering`, `cameras`, See-also links, router rule 10), WebGPU/WASM/Steam Deck passages, a deferred-rendering recommendation against the simple look, dead See-also links after a partial vendor.

Vendored in `.claude/skills/` of the code repo: `bevy-ecs-queries`, `bevy-testing`, the licence file and `VENDORED.md` with the commit. The "after fix" skills are not vendored: fixing eight wrong examples in a skill that agents would trust is a decision for the initiator.

Side finding (not verified by running): the comment at `exo_app/src/lib.rs:92` says each update with `ManualDuration` is exactly one physics tick; `bevy_time/real.rs:99` suggests the first update has delta 0, so there may be an off-by-one in tick counts.

## Step 5: API lookup rule

Proposed line for `AGENTS.md` (see below). Comparison with step 4 is by review only, no A/B run: for signatures and names the lookup is enough, since every wrong skill example above was found by exactly that lookup. A skill adds only things the source does not say: usage patterns (the `TimeUpdateStrategy` replay idiom in `bevy-testing` is already what our code does) and a shortlist of gotchas. The two vendored skills are cheap but not needed to be correct.

## Step 6: smaller tools

- **cargo-nextest 0.9.146:** 9 tests, 11.9–12.1 s against 12.7 s for `cargo test`; the 11.3 s `scenario_full` test is the whole cost. Drop. (A first `cargo test --workspace` needs its own 294 s build of the test profile, same for both.)
- **bevy_lint:** the compatibility table lists 0.6.0 for Bevy 0.18 and `0.7.0-dev` (nightly-2026-04-16, still Bevy 0.18) on main; there is no version for Bevy 0.19, and the docs say other versions "may crash". Installing needs a pinned nightly with `rustc-dev` (about a gigabyte) plus a source build. Not tried: no real findings could be compared. Blocker: wait for a Bevy 0.19 release of `bevy_lint`.

## Proposed lines (initiator decides)

`AGENTS.md`:
- `Bevy and Avian APIs change in every release. Look up names and signatures in ~/.cargo/registry/src/*/bevy-<version>/ (and bevy_*, avian3d-<version>) and the Bevy examples there before you write code; do not trust memory of older versions.`
- (only if BRP is wanted) `The "remote" feature (BRP, localhost) is dev-only and red-class; never enable it in release builds or CI artefacts.`

`WORKSPACE.md` (Rust/Bevy part):
- `Use one target dir for all worktrees of the Bevy spike: export CARGO_TARGET_DIR=~/Work/cargo-target/exo-bevy. A fresh worktree then builds in about 8 s instead of 6 min. Do not run two cargo builds at the same time (lock).`
- `Run with cargo dev -- <args> (dynamic linking, 1.1 s rebuilds). Never ship a dynamic build.`
- `rust-analyzer: pacman's rustup has no proxy. ~/.local/bin/rust-analyzer should be: exec rustup run stable rust-analyzer, with CARGO_TARGET_DIR set to ~/.cache/rust-analyzer-target/<dir> so it does not lock target/.`
- `The shell of an agent session may lack WAYLAND_DISPLAY; set WAYLAND_DISPLAY=wayland-1 for windowed and --hidden runs.`
- `Tools through mise (pinned in spikes/bevy/mise.toml); a mise shim cannot serve as build.rustc-wrapper.`

## What was not done or not certain

- Why the plugin shows no diagnostics (only the behaviour was observed).
- BRP blind test: one run per arm, run concurrently, BRP not used; Sonnet 5.5 as the debugging model.
- sccache path-dependent misses: cause open.
- `bevy_lint`: not installed (blocker).
- The two sessions of the blind test and the skill review are model output; I read both reports in full, and the cause they name matches the fix commit `76b6319`.
