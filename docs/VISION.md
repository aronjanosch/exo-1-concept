# Vision — EXO-1

Status: DRAFT. This repo is the concept (`exo-1-concept`); code lives in a separate public repo.

## Idea

An open-source game built by a community. An experiment: how far does a game get when implementation is no longer the bottleneck (AI), and people provide direction, ideas and taste?

## The experiment

- Documented as a video series. Failure is a valid result.
- Decisions by community vote, built via the `community-gate` framework (proposals, votes, feature branches, one PR per feature).
- The community decides features, not security. Code gates and review are not up for a vote.
- Released when the community votes for it. The initiator can stop the experiment at any time.

## Frame

- Engine: Godot
- License: MIT, fully open. Original or CC0 assets only.
- Setting: a strange galaxy, deliberately goofy. Graphics don't matter: simple lighting, simple assets.
- Must run on weak hardware.
- Gameplay and systems over graphics.

## Multiplayer

- Small co-op groups (about 2-5 players), host-authoritative: one player hosts.
- No peer-to-peer mesh, no central MMO.
- Players can run their own persistent server: their own galaxy.
- Shared markets across servers: not planned, maybe later if the community votes for it.

## World

- Start small: one star system, similar to starting a No Man's Sky save.
- Pure procedural generation tends to feel empty. Quests, points of interest and NPCs can be authored, including AI-generated.

## User-generated content

- No editor. The engine plus AI coding agents is the toolchain.
- Content is data (quests, POIs, NPCs, systems), validated against a schema.
- Scripts only enter through reviewed PRs. No executable code is loaded from the network at runtime.
- Moderated by the community, like mods and workshops. Malicious content has happened there before, so the gates matter.

## Core design

- The initiator decides the core gameplay loop and pillars. AI is weak at creative ideas, balancing and system design.
- The community adds ideas, features and content within that frame, and votes on them.
- Scope starts deliberately small before anything expands.

## Review and merge

- Contribution rules: see `community-gate` (proposal approved, then branch, then one PR).
- Hard cap on concurrent feature branches and open PRs.
- Risk classes by path:

| Class | Paths | Gate | Merges |
|---|---|---|---|
| green: content | `content/**` | schema, no scripts, size limits, bot playthrough | bot, after vote |
| yellow: gameplay code | `game/**` | all CI gates, security lint, AI review, 1 moderator | moderator |
| red: core | CI, `project.godot`, autoloads, networking, addons | everything above | initiator only |

- CI (deterministic, carries the load): lint, format, tests, security lint, headless boot, bot playthrough through the agent interface (MCP), performance budget on a low-spec profile.
- AI summarizes, labels risk and checks vision fit. It never approves.
- Merge train every 1-2 weeks. Trusted contributors can become moderators.
- Voters should have played the build under review (proof-of-play, friction rather than security).

## Inspiration (no assets, no names)

- Star Citizen: the game that never gets finished
- Schedule 1: simple, goofy, simple design and assets, good gameplay, good systems
- Valheim: gameplay over graphics
- No Man's Sky: starting in a system

## Non-goals

- No guaranteed good game.
- No foreign trademarks, assets, or reverse-engineering of other games.
- No runtime-shared executable code.
