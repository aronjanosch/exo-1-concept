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
| `Skippeh/ScheduleOne_UnityProject` | Modding project: stripped scripts, plugin list and render pipeline settings, no meshes or textures, no licence | Says nothing about modelling or shading. It shows the render stack (see below). For the look itself, game screenshots are the reference |

## What carries the Schedule I look (read from the modding repo, 2026-10-08)

Read from plugin names, the URP settings and avatar field names; nothing copied. The assets themselves are not in the repo, so flat versus smooth shading cannot be read there.

- **Render stack:** Unity URP, Forward+, HDR, soft shadows from the sun to 70 m in 3 cascades. Renderer features: screen-space ambient occlusion, volumetric fog, god rays, decals, an outline feature, grass bending. Plugins add more ambient occlusion (HBAO), screen-space GI (RadiantGI), height fog, volumetric light beams, a sky system, stylized grass and water, and colour grading with LUTs (Beautify).
- **Characters:** a base body tinted by skin colour, shaped by height, weight and gender sliders; faces and clothes are tinted texture layers stacked on the body; eyes, eyelids, eyebrows and hair are separate parts; accessories carry their own colour. Distant characters become impostors (flat pictures).
- **Takeaway for us:** the geometry and textures are simple, but the mood comes from a fairly heavy lighting and post stack (ambient occlusion, fog, GI, grading). Bevy has counterparts (SSAO, distance and volumetric fog, bloom, tonemapping and grading); how much of it we afford is a performance question.

## Sources

- https://github.com/ahujasid/blender-mcp
- https://github.com/raydeStar/reference-asset-compiler
- https://github.com/storytold/artcraft
- https://docs.meshy.ai/en/webapp/pricing
- https://github.com/microsoft/TRELLIS.2
- https://github.com/Skippeh/ScheduleOne_UnityProject
