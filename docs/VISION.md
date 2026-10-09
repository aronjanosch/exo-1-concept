# Vision — EXO-1

Status: living. This repo is the concept (`exo-1-concept`); code lives in a separate private repo (`exo-1`).

Name: "EXO-1" is a working title only. A game called "Exo One" already exists, so the name will change (see `DECISIONS.md`).
Related: `CORE-LOOP.md` (loop, pillars), `ROADMAP.md` (milestones), `DECISIONS.md` (decided, open, parked).

## Idea

A goofy co-op space game in a strange galaxy, built by agents under the direction of one person: how far does a game get when implementation is no longer the bottleneck, and people provide direction, ideas and taste? Since 2026-10-07 it is a private project (the initiator and friends); the community idea below stays a possible later step.

## Spirit

- Ideas, inspiration and taste are the bottleneck, not implementation. Structure only where it must be.
- AI can produce junk, but it can also boost good ideas and creativity. That is the part we use.
- The initiator decides game design (`DECISIONS.md`). AI is weak at creative ideas, balancing and system design.
- Learning from other games is welcome: their mechanics, ideas, published code and write-ups, and how they solved problems. We write our own code and make our own assets, data, names and texts.
- Scope starts deliberately small before anything expands.

## Frame

- Engine: Rust with Bevy, physics in `f64` (Avian). See `DECISIONS.md`.
- License: MIT for code, CC0 for assets (`DECISIONS.md`). Original or CC0 assets only.
- Setting: a strange galaxy, deliberately goofy. Graphics don't matter: simple lighting, simple assets.
- Aim for the best runtime performance; the simple look helps keep rendering costs down.
- Gameplay and systems over graphics.

## Multiplayer

- Small co-op groups (about 2-5 players). Client authority: each client simulates its own player and ship, one player hosts and relays snapshots (`DECISIONS.md`, 2026-10-06).
- No peer-to-peer mesh, no central MMO.
- Players can run their own persistent server: their own galaxy.
- Shared markets across servers: not planned, maybe later if the community votes for it.

## World

- Start small: one star system, similar to starting a No Man's Sky save.
- Pure procedural generation tends to feel empty. Quests, points of interest and NPCs can be authored, including AI-generated.

## Content

- No editor. The engine plus AI coding agents is the toolchain.
- Content is data (places, goods, jobs, encounters), validated against a schema (`DECISIONS.md`, "Loop content schema").

## Community step (possible later)

The experiment version of EXO-1 had a community decide features by vote through the `community-gate` framework, with risk classes by path, a merge train, proof-of-play, a video series and a stop right for the initiator. None of it applies today. The full text is in `archive/VISION-COMMUNITY-ERA.md`, the plan in `archive/ROADMAP-GODOT-ERA.md`.

## Inspiration (no assets, no names)

- Star Citizen: the game that never gets finished
- Schedule 1: simple, goofy, simple design and assets, good gameplay, good systems
- Valheim: gameplay over graphics
- No Man's Sky: starting in a system

## Non-goals

- No guaranteed good game.
- No foreign trademarks or assets.
- No runtime-shared executable code.
