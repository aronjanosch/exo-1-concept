# Building the city

Research note, 2026-10-08. What AI "build a town" demos use, and how Schedule I builds its town. Decisions that came out of it: `DECISIONS.md`, rows "City look" and "Live Blender (MCP)". Code: code repo `art/city/` (kit, brief, first shops and road tiles).

## What the demos use

Most AI town demos on X are one-shot videos, not maintained pipelines. Three setups recur:

- An agent drives Blender, through `blender-mcp` or by writing Blender Python directly.
- An agent writes three.js scenes of instanced boxes.
- Image-to-3D services for hero props (Hunyuan3D, Trellis, Tripo, Meshy, Rodin).

The quality comes from a structured spec plus checks, not from free modelling.

| Source | What it is | What we take |
|---|---|---|
| [Hunyuan3D-WorldClaw](https://tencent-hunyuan.github.io/Hunyuan3D-WorldClaw/) | Paper only, no code. Claude plans, GPT-Image draws, Hunyuan3D makes meshes, BlenderMCP places them; 4 server GPUs | The render-check-edit loop: diagnostic renders, check scale, floating objects and clipping, fix, render again. The outputs go through Hunyuan3D, whose licence excludes the EU |
| [glb-buildings-skill](https://github.com/hec-ovi/glb-buildings-skill) | MIT, TypeScript CLI plus agent skill. A building is a JSON stack of floor bands, facades are 10 cm cell grids, support, overlap and triangle checks run before export | The checks and the JSON report of what is missing, the facade grid, three passes (massing, facades, roof). The ideas went into `art/city/kit.py`; we do not use the tool itself (Node, modern/cyber look) |
| [duplexity-3d](https://github.com/hec-ovi/duplexity-3d) | three.js adventure, no licence | `city.json` as a list of models and positions; prove reachability and matching doorways before use. Pattern only |
| [stratum](https://github.com/SamG-Coder/stratum) | Procedural ray-marched city in WebGPU, no meshes | Nothing |
| [GPT-6 Astra demos](https://www.mindstudio.ai/blog/gpt6-astra-3d-generation-demos) | Demo roundup, no technical detail | Warning about "AI design smell": the same palette and flat design everywhere unless a brief steers it |
| [zer0-g](https://github.com/witnesstodark/zer0-g) | Finished three.js racer. Meshes come from Tripo, art is CC BY-NC | `blender -b --factory-startup`; "an attractive render is not an engine handoff", so check in the running game |
| [universal-modder](https://github.com/rehan-remade/universal-modder) | Plugin for modding existing games | Field notes written by agents for agents; look before you ship |

## How Schedule I builds its town (local extract)

The main scene did not export (0 bytes). Layout evidence comes from the tutorial town and from loose meshes. Nothing is copied.

- **Composition:** hand-placed prefabs in one scene.
  - Each custom building is one model with a few named parts (main, roof, foundation, windows, facade); windows and units repeat on about a 4 m pitch.
  - Next to them sit bought whole-house meshes with LODs.
  - The main city has some 4 × 4 m wall modules.
- **Roads:** modular 10 m tiles on a 10 m grid, made of a 6 m carriageway and two 2 m sidewalks.
  - The tile set is small: straight, T, X, end, curve wedges, ramp, crossing.
  - Random crack and grime decals give each tile its variety.
  - Splines are used only outside town.
- **Buildings:** most are facades. Their windows are opaque, and NPCs "enter" by vanishing.
  - The few real interiors sit inside the same building model.
  - Doors are separate interactive objects.
  - Typical sizes: door about 1 × 2.1 m, corner shop about 12 × 12 m and 6 m high.
- **Materials:** each hero building has one baked texture set plus 2–4 shared tiling materials, about 4–5 materials per building.
- **Props:** grouped under their building, so they move with it. Lamps, traffic lights and power poles are small scripted prefabs.
- **Scale:**
  - The walkable town is about 400 × 300 m, in six districts.
  - It has roughly 30 unique buildings (estimated from texture sets).
  - The main terrain is 512 × 512 m.
- **Star Citizen records:** nothing on city layout; the object container layouts are missing from the dump. They give a medium landing pad of 56 × 88 m and suggest a 2 m module for shop counters.
