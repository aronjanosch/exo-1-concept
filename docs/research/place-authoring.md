# Authoring places (outposts, small city)

Research note, 2026-10-08. Not a decision: tool choice and new dependencies stay with the initiator. Target look: simple, like Schedule I (few flat colours, low poly, hardly any textures).

## What Bevy 0.19 offers

- **No official editor yet.** Bevy 0.19 brings the new scene system with BSN, but only as the `bsn!` macro in code; BSN asset files and the visual editor come in a later release ([Bevy 0.19 release notes](https://bevy.org/news/bevy-0-19/), [BSN overview](https://taintedcoders.com/bevy/bsn)). Not worth waiting for.
- **Blender plus glTF.** Blender custom properties arrive as `GltfExtras` in `bevy_gltf` (in our Bevy already, no new dependency). Objects can be tagged, for example `spawn = "trader"` or `pad = "landing"`, and a small system turns tags into components.
- **[Skein](https://github.com/rust-adventure/skein)**: a Bevy plugin plus a Blender add-on. It reads the reflected Bevy components from the running game and lets you attach real components in Blender; they come back through glTF extras. Latest version 0.3.0-rc.1. Bevy 0.19 support not checked. New dependency, so ask first.
- The older Blender workflow (`Blender_bevy_components_workflow`, later Blenvy) is no longer maintained.
- **Preview**: Bevy's `file_watcher` feature reloads changed assets at runtime. The game runs next to Blender; export, and the place updates live. Close to an editor for our needs.

## The catch: the planet is procedural

Blender does not know the terrain. Proposed split:

| What | Where | Format |
|---|---|---|
| How a place looks (landing pad, 3–5 low-poly buildings, trader) | Blender, local coordinates | glTF in `content/` |
| Where it sits (planet, latitude/longitude, heading, which glTF) | In the game: a debug key "place here" writes the file | RON in `content/` |
| Flattening the ground under it | Generator, derived from the place position | Code |

Both files hot-reload, so moving a place or rebuilding a building shows up without a restart.

## Open

- Skein or plain `GltfExtras` with our own tags.
- How the generator flattens the ground (radius, blend) and whether `planet_core` reads places as input.
- Where places live in the content schema (`location` in `CORE-LOOP.md`) once several planets exist; depends on spike 11.
