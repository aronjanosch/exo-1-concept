# Design tools for models

Research note, 2026-10-08. The decision that came out of it lives in `DECISIONS.md` (model source). Everything else here is not decided; new tools and dependencies stay with the initiator.

Target look: simple, like Schedule I (chunky shapes, flat colours, light and haze carry the mood). Free and open-source project, assets original or CC0.

## Chosen path

- **Blender with Python scripts.** Blender 5.2.1 LTS (GPL) is installed on the initiator's desktop. The agent writes one script per model; `blender -b -P <script>.py` builds it, exports the `.glb` into `content/` and can render review images. The script is the only source in the repo. The initiator opens the result in Blender to review it.
- **Placeholders:** Kenney and Quaternius (CC0), marked as placeholders.

## Looked at, not used

| Tool | What it is | Why not (for now) |
|---|---|---|
| blender-mcp (ahujasid, MIT, v1.8) | MCP server plus Blender add-on: the agent drives an open Blender, runs Python, gets viewport screenshots | Useful for live iteration, but a new dependency that runs arbitrary code; initiator's call. If used, switch off its Sketchfab, Hyper3D and Hunyuan integrations (foreign licences) |
| reference-asset-compiler (raydeStar, MIT) | Pipeline from one concept image to a rigged UE5 character | Windows and UE 5.8 only, 24 GB VRAM, 20k triangle budget, built on Hunyuan3D 2.1 (licence excludes the EU) |
| ArtCraft (storytold, MIT) | AI image and video crafting app | Mostly 2D and video; its 3D goes through cloud services (Meshy, Tripo, Rodin, Hunyuan), so foreign licences and costs |
| Meshy, Tripo, Rodin | Text and image to 3D services | Dense meshes with PBR textures, the opposite of our look. Meshy free plan: CC BY 4.0 (credit required), not CC0 |
| TRELLIS.2 (Microsoft, MIT for code and weights) | Local image-to-3D model | Needs at least 24 GB VRAM (the initiator's GPU has 16 GB); high-resolution PBR output needs cleanup. Maybe later as an idea generator |
| Blockbench (GPL) | Free box and low-poly modeller with glTF export | Good for boxy figures, but a second tool next to Blender without a clear gain for script-driven work |
| `Skippeh/ScheduleOne_UnityProject` | Modding project with the game's prefabs, meshes and shaders, no licence | Not a source of anything. For the Schedule I look, game screenshots are the better reference |

## Sources

- https://github.com/ahujasid/blender-mcp
- https://github.com/raydeStar/reference-asset-compiler
- https://github.com/storytold/artcraft
- https://docs.meshy.ai/en/webapp/pricing
- https://github.com/microsoft/TRELLIS.2
- https://github.com/Skippeh/ScheduleOne_UnityProject
