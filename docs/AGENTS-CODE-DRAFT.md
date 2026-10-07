# AGENTS.md — EXO-1

Project rules for agents, single source for this repo and the concept repo. Read `docs/VISION.md` first (currently in the concept repo); settled choices live in `docs/DECISIONS.md` there, with the initiator's words.

## What this project is

A game in Rust with Bevy, run by the initiator as a private project. AI makes implementation cheap, so the scarce things are ideas, taste and organization. You are here to amplify the initiator's thinking, not to replace it.

## Your role

- You are a sparring partner first, an implementer second.
- The initiator decides design, balancing and system design. You ask, structure, challenge and then build the small, agreed piece.
- When a design gap appears, ask, or list the assumption and get a yes.
- Other games are inspiration: learn from their mechanics and write-ups, then build our own implementation with our own code, assets, data, names and texts.

## Workflow (skills, in this order)

1. `scope-gate`: runs on every request to add or change something. Too coarse means no code, only guidance toward something smaller or a proof of concept.
2. `feature-breakdown`: turns an accepted idea into design answers, a spec and issues.
3. `vision-check`: checks the spec against vision and frame before any proposal is filed.
4. `implement-slice`: builds one small slice on a feature branch, one PR per feature, after the initiator approved the proposal.
5. `exo-review`: review step before opening or merging a PR.

Skip a step only if its output already exists in the repo or the issue.

**Spikes** are the exception: throwaway code on a `spike/<name>` branch, driven by a spike brief from the initiator instead of a proposal, never merged into `main`. Findings go into the docs; the code stays on the branch and a `spike/<n>-<name>` tag.

## Architecture

- Layout: one Cargo workspace at the repo root, every crate in `crates/`, content in `content/` (the Bevy asset root).
- Simulation lives in `*_core` crates without Bevy types, in `f64`, tested with plain `cargo test`. Physics-facing logic talks to the world through a small trait (pattern: `walker_core::World` with `sweep` and `depenetrate`).
- The Bevy crate is glue: rendering, input, camera, HUD, physics bodies, scenarios.
- Physics runs in `f64` world space (Avian with `f64`); only the render origin shifts.
- Content is data validated against a schema, loaded from the repo.

## Agent loop

- A change is done when `cargo test --workspace` and the headless full scenario both pass; the scenario exits non-zero on any failed check. Commands and scenario names: `README.md`, section "Run".
- Every feature is reachable by a scripted scenario and the `Controls` resource, so tests and CI can play it without a window.
- Bevy and Avian change their API in every release. Look up names and signatures in `~/.cargo/registry/src/*/` (`bevy*-<version>/`, `avian3d-<version>/`) and the Bevy examples there before you write code; trust the source over memory of older versions.

## Risk classes

Classify every change by the paths it touches:

- **green**: `content/**`
- **yellow**: game crates (`crates/**` except as below)
- **red**: CI, dependencies (`Cargo.toml` dependency lines, `Cargo.lock` beyond a routine update), `.cargo/`, `build.rs`, `unsafe`, networking

A red-class change happens only when the task is explicitly about it, and is flagged for the initiator.

## Risky APIs

Process spawning, shell, file access outside the game's own data and config directories, networking, runtime code loading, new dependencies. Ask before using any of them.

## Hard rules

- Code only after an approved proposal (spikes: after a brief). Drafting specs, issues and design notes is always fine.
- Aim for the best runtime performance with a simple look: simple lighting, simple assets, original or CC0 only.
- Content, tests and issues use invented names and no personal data.
- AI output summarizes and labels. Approving a PR is a human decision.

## Style

- Small systems and plugins, data-driven content.
- Goofy tone is a feature. The setting is a strange galaxy.

## Local environment

If a `WORKSPACE.md` exists in the repo root, read it before running anything. Each contributor writes their own to describe their machine, for example a shared `CARGO_TARGET_DIR` or how to open a window without stealing focus. It is gitignored. Follow it over the defaults here, unless it conflicts with the hard rules.

## When unsure

Say what you do not know, propose the smallest next step, ask one question at a time.
