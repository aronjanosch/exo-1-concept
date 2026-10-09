# Night run brief — milestone D "Foundation of the business"

Written 2026-10-09, moved to the NAS 2026-10-10, for one unattended run through the whole night. The initiator reviews in the morning and decides what gets merged. Same pattern as `RUN-C-NIGHT-BRIEF.md`. Scope: the parts of D that can be tested without a second computer. Nothing here is final design: every placeholder (names, texts, numbers) is marked `TODO(initiator)`.

## Where it runs

The NAS (herdr machine `roost`, container user `agent`), as a `/goal` run in auto mode, in its own worktree and target dir:

```sh
git -C ~/projects/exo-1 fetch --prune origin
git -C ~/projects/exo-1 worktree add ~/projects/exo-1-night-d -b night/d-foundation origin/main
git -C ~/projects/exo-1-concept pull --ff-only
source ~/.cache/exo-buildenv/env.sh
export CARGO_TARGET_DIR=~/.cache/exo-1-target-night-d CARGO_BUILD_JOBS=6
```

The branch starts at `main` (kernel, jobs and the first round are merged, #180). Before every `cargo` command: `source ~/.cache/exo-buildenv/env.sh` and the two variables. One cargo command at a time. The NAS has no GPU: headless only, no performance or frame-time claims; a window only under `xvfb-run`.

## Read first

1. Code repo `AGENTS.md` and `WORKSPACE.md`.
2. Concept repo (`~/projects/exo-1-concept`, read only): `docs/CORE-LOOP.md`; `docs/DECISIONS.md`, rows from "Gameplay crates and build order (D)" to "Milestone order"; `docs/research/loop-feel.md` (beats, feedback, briefings) and `docs/research/crime-empire-design.md` (customers, licences as tutorial).
3. Epic #37 (its spec stays valid for what is built) and the issues below (`gh issue view <n>`).
4. What is built: `crates/gameplay_core`, `crates/jobs_core` (tests in `tests/`), `crates/exo_app/src/gameplay.rs`, `scenario/deliver.rs`. Build on them; do not rewrite them.

## Phase 1: branch `night/d-foundation`

Work these issues in order. Each is done when its "Done when" holds, as far as it can be checked without a second computer, and `cargo t` and `cargo scenario` both exit 0.

| Order | Issue | Note |
|---|---|---|
| 1 | #128 save model | Envelope and sections for kernel, jobs, world. Every system added later in this run brings its own section with a round-trip test. |
| 2 | #165 feedback beats | Notices and pacing queue in the kernel, banners, toasts, sounds, the arrival ritual. `gameplay.rs` stops writing plain note lines. |
| 3 | #167 givers | Record, briefings from parts, standing per giver. Two placeholder givers: a legal courier office and the small odd family (kind `family`, its jobs not yet offered in D). |
| 4 | #170 courier jobs | On foot, between places at walking distance from the start, from the courier giver, with the beats of #165. New places as data, like `drip_rock.json`, **placed in metres relative to Drip Rock** (a new optional place field, e.g. `near: { place, east_m, north_m }`, test-first in `planet_core/tests/places.rs`), so walking distances stay the same when the planet radius changes (#177). |
| 5 | #168 customers | New crate `customers_core`; orders become offers through a domain event. Three placeholder customers at existing places. |
| 6 | #169 flight licence | The exam as a job template; new objective kinds and world events as needed (e.g. took off, pad reached, landed with a touchdown speed), each test-first. The seat refuses without the licence. **The exam checks only these events, never values of the flight model**: the axis flight model is being reworked by the initiator. In the scenario, take-off and landing may be set by test hooks, as `deliver` moves crates. |
| 7 | #166 map | Pin list without Bevy, drawn in the window. The screenshot under `xvfb-run`: save it as `night-shots/map.png` in the worktree, do not commit it, name the path in `NIGHT-LOG.md`. |
| 8 | #135 save file | Write and read `saves/` in the game's own directory, autosave, client id file. The scenario: deliver halfway, save, restart the app state, load, finish. **Not** the join/send part: that is networking (#134) and on the ask-first list. |

Not in this run: #134 (networking), #138 (playtest content, the initiator's), the board #126 #131 #132 and the HUD #136 (phase 2 at most).

- Test first, as the tickets say: write the failing test, then the code.
- Commit after each finished issue with `(#n)` in the message and push the branch. Comment on the issue: what was built, the check numbers, the `TODO(initiator)` values. Do **not** close issues, do **not** tick the epic's checklist.
- Systems talk only through domain events and conditions (epic #37, "Domain events"). A new system is a new `*_core` crate without Bevy; `exo_app` is the glue.
- Every feature that shows up in the game gets a scripted headless scenario driven through `Controls`, like `deliver`.
- Design gaps: pick a starting value, mark it `TODO(initiator)` in the data or code, list it in `NIGHT-LOG.md`. Never stop for a design question.
- Tone for placeholder texts: silly and satirical, invented names and invented goods only (`CORE-LOOP.md`, "Tone"). No real people, brands, drugs or names from other games.

## Guardrails from the research

`DECISIONS.md` row "Borrowed patterns (direction)" and the avoid list in `research/crime-empire-design.md`. What applies to D:

- **Humour in items and mechanics first**, then in short lines; no cutscenes.
- **Never the same line twice in a row**: every text a player sees often (greetings, briefings, notices, customer lines) comes from a pool, picked by seed without immediate repeats.
- **No mistake locks a giver or customer for good**: standing and relationship drop and recover with work.
- **No co-op player waits for another**: without a licence you ride along and carry; banners never block input; the arrival ritual can be skipped.
- **No staring at a timer**: no waits without something to do.
- **The exam is the tutorial**, with a grade (pass, honours).
- **Illegal must not always pay more than legal** later on: keep courier and wholesaler prices in one table (`TODO(initiator)`), so G and H can balance against them.

The four decisions of 2026-10-09 (`DECISIONS.md`: "Where jobs come from (D)", "Customer goods before production (D)", "Flight licence exam (D)", "Courier start (D)") are in the tickets #167, #168, #169, #170 under "Decided".

## Phase 2: extras, branch `night/d-extras`

When phase 1 is done (or an issue is blocked for good), branch `night/d-extras` from the tip of `night/d-foundation` and keep going there. Pick by value, in this order:

1. Polish that serves the playtest question "does a job feel like a job?": the target arrow and money on the HUD (#136), the board with offers by giver (#126 #131 #132), better synthesized sounds for the beats, briefing texts.
2. Groundwork for G that needs no new dependency: a `production_core` sketch (recipe record, a station state machine with tests) — only as tested data and state, no models.
3. Backlog items you judge valuable for the game.

Each extra: commits, tests and a scenario where it makes sense, an entry in `NIGHT-LOG.md` (what, why, checks, open points).

## Rules (both phases)

- `cargo t` and `cargo scenario` exit 0 before every push. Never push red. If something cannot be made green within a few attempts, revert it, note it in `NIGHT-LOG.md` and move on to the next issue.
- **Ask-first list from `AGENTS.md` = skip, do not do**: new dependencies, CI, `.cargo/`, `build.rs`, `unsafe`, networking, spawning processes, file access outside the game's own directories. No edits to `AGENTS.md`, skills or `WORKSPACE.md`.
- No pushes to `main`, no PRs, no merges, no force pushes, no closing issues, no new GitHub issues.
- Do not touch the other worktrees on the NAS (`~/projects/exo-1`, `exo-1-skills`, `exo-1-spike11`, `exo-1-spike12`, `exo-1-warp`) or their target dirs, and leave the other herdr panes and agents alone.
- Invented names only, no personal data.

## NIGHT-LOG.md

At the root of the code repo, committed on the current branch; replace the old C log at the start (it stays in git history). Top: a checklist of the phase 1 issues and the extras with status. Below, per item: what was built, checks with numbers (test count, scenario steps), `TODO(initiator)` values, open questions. It is the morning report; keep it current with every commit. After a context compaction, read it first to know where you are.

## Stop

Each turn starts with `date` and `cat ~/.claude/usage.json` and prints the local time and the 7-day `used_percentage`. Stop when the local time is 08:30 or later, or the 7-day value is at least 90 (if `usage.json` is older than 15 minutes, the time rule alone). Finishing phase 1 is not a stop: go on with phase 2 until then. Then: finish or revert the current change, make sure both branches are pushed and green, update `NIGHT-LOG.md`, and end with a short summary.

Do not stop earlier for any of these: a summary that announces the next step without doing it, offering to continue, listing decisions that do not block, reporting after an issue. Stop earlier only when nothing more can move without the initiator or a protected action; then say which.

## Goal condition (for `/goal`)

> The night run of `docs/RUN-D-NIGHT-BRIEF.md` (concept repo) has ended by its stop rule: the last printed `date` shows 08:30 or later, or the last printed `usage.json` shows a 7-day `used_percentage` of at least 90, or `NIGHT-LOG.md` states that nothing more can move without the initiator and names why. In each case the stop steps are done: the branches `night/d-foundation` (and `night/d-extras`, if started) are pushed, the last printed `cargo t` and `cargo scenario` on each pushed tip exit 0, and `NIGHT-LOG.md` is committed and pushed with a checklist of #128, #165, #167, #170, #168, #169, #166, #135 and the extras. Finishing phase 1 alone does not meet this goal. Never: a push to `main`, a PR, a merge, a closed issue, a new dependency.
