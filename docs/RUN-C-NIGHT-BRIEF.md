# Night run brief — milestone C "Things you can touch"

Written 2026-10-09 for one unattended run on the NAS (roost container). The initiator reviews in the morning. Initiator, 2026-10-09: "falls er fertig ist und noch budget hat kann er ja für den milestone sachen implementieren, aber gerne auch für andere Teile des spiels. was er am sinnvollsten hält, die extras kommen alle gemeinsam in einen neuen branch [...] keinen PR zum schluss das passiert erst nach meiner kontrolle am nächsten morgen."

## Read first

1. Code repo `AGENTS.md` and `WORKSPACE.md` (this machine: no GPU, no perf numbers, `source ~/.cache/exo-buildenv/env.sh` before every `cargo`).
2. `DECISIONS.md`, entry "Grab and cargo (milestone C)": the eight decisions. Research: `research/grab-and-cargo.md` (risks: warp sets the ship pose each tick at up to 1e6 m/s; Avian moves child colliders one step late).
3. The issues below (`gh issue view <n>`), and epic #35.

## Phase 1: milestone C, branch `feat/milestone-c-grab`

Branch from `main`. Work these issues in order; each is done when its "Done when" holds and `cargo t` and `cargo scenario` both exit 0.

| Order | Issue |
|---|---|
| 1 | #81 `grab_core` (pure Rust, test-first; quick and safe start) |
| 2 | #80 crates as data, living in the ship's frame |
| 3 | #82 interaction: one verb, one prompt |
| 4 | #83 grab in the game: hands and tool |
| 5 | #84 lock grid in the cabin |
| 6 | #85 object budget |

Not in this run: #86 (network, two computers).

- Commit after each finished issue with `(#n)` in the message; push the branch. Comment on the issue what was built and the check numbers; do **not** close issues and do **not** open a PR.
- Design gaps: pick a starting value, mark it `TODO(initiator)` in the data or code, list it in `NIGHT-LOG.md`. Never stop for a design question.
- Every feature gets a scripted scenario driven through `Controls` (headless).

## Phase 2: extras, branch `night/extras`

When phase 1 is done (or an issue is blocked for good), branch `night/extras` from the tip of `feat/milestone-c-grab` and keep going there. All extras go on this one branch, even if they do not fit together.

Pick what you think adds most, in this order of preference:
1. Milestone C polish that serves the playtest question "does moving cargo feel good?": synthesized grab and lock sounds, a visible tool beam, lit lock plates, crate stacking, carrying crates down the ramp, HUD hints.
2. Other open roadmap work that needs no GPU, no second computer and no new dependency: milestone D groundwork (content loader with schema validation, the job state machine in a `*_core` crate), backlog items like #52 (split `scenario.rs`).
3. Anything else you judge valuable for the game.

Each extra: one or more commits, tests and a scenario where it makes sense, an entry in `NIGHT-LOG.md` (what, why, checks, open points). Before starting, pick by value; do not polish the same thing twice.

## Rules (both phases)

- `cargo t` and `cargo scenario` exit 0 before every push. Never push red. If something cannot be made green within a few attempts, revert it, note it in `NIGHT-LOG.md` and move on.
- **Ask-first list from `AGENTS.md` = skip, do not do**: new dependencies, CI, `.cargo/`, `build.rs`, `unsafe`, networking, spawning processes, file access outside the game's own directories. No edits to `AGENTS.md`, skills or `WORKSPACE.md`.
- No pushes to `main`, no PRs, no merges, no force pushes, no closing issues, no new GitHub issues.
- No perf or frame-time claims from this machine. Windowed runs only under `xvfb-run` for screenshots.
- Invented names only, no personal data.
- One cargo command at a time (shared target dir lock).

## NIGHT-LOG.md

A file at the root of the code repo, committed on the current branch. Top: a checklist of phase 1 issues and extras with status. Below: per item what was built, checks with numbers, `TODO(initiator)` values, open questions. It is the morning report; keep it current with every commit. After a context compaction, read it first to know where you are.

## Stop

Each turn starts with `cat ~/.claude/usage.json` and `date`, and prints the 7-day `used_percentage` and the local time. Stop when the 7-day value is at least 90, or the local time is 08:30 or later. Then: finish or revert the current change, make sure both branches are pushed and green, update `NIGHT-LOG.md`, and end with a short summary. If `usage.json` is older than 15 minutes, use the time rule alone.
