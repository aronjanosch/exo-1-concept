# Designer brief: the walker figure

Status: open. Date: 2026-10-08. Owner: the initiator, with a designer agent that writes Blender Python scripts.
Related: code repo issue #32 (the plumbing that loads this file), `GLOSSARY.md` in the code repo (Walker versus Character model), `VISION.md`.

## What we need

How other players look in EXO-1: one figure that every player wears. Each slot shows it in its own colour, with a name tag above the head. It is seen from 2 m (in a cabin) to about 60 m (across a landing pad).

The figure is only the look (the **character model**). Movement and collision stay with the walker, a capsule 1.8 m tall and 0.35 m in radius. The figure must sit inside that space, roughly.

## Feel

- **Goofy, chunky, readable.** The setting is a strange galaxy and the goofy tone is a feature. The figure should make a friend laugh a bit when it floats past the cabin window.
- **Simple look, great feel.** Few shapes, flat colours, a strong silhouette. No textures needed, no fine detail that disappears at 30 m.
- **Original.** Build our own figure. It must not resemble a known game, film or brand character, and nothing is copied from a model library unless it is CC0. Invented names only, no personal data.

## The process

1. **Three quick blockouts**, each in its own direction. Show a render from the front, the side and three quarters, plus one at 30 m against a dark sky. Starting ideas, change them freely:
   - **The Thermos**: a round tank body, a dome helmet that is too big, stubby legs, a little antenna.
   - **The Mailbox**: a boxy suit, a visor like a slot, a backpack like a small boiler.
   - **The Pear**: bottom-heavy, tiny arms, a round porthole visor, hover boots instead of feet.
2. **The initiator picks one** (or a mix) and says what to push further.
3. **The final model and the export**, checked against the contract below.

There is no animation yet: remote players slide over the ground in one pose. Pick a pose and shapes that do not look broken when they slide. Hover boots, a floaty stance or a pose that hides the feet all work.

## Technical contract (must match code repo issue #32)

- **File:** glTF 2.0 binary, `content/models/walker.glb` in the code repo. The source is a Blender Python script, `art/walker/walker.py` (not shipped); `blender -b -P art/walker/walker.py` builds the figure and writes the `.glb`. No `.blend` is committed (`DECISIONS.md`, model source). The initiator reviews the result in Blender.
- **Scale and axes:** 1 unit = 1 m. About 1.8 m tall. Origin at the feet, centred. In Blender the figure faces −Y (towards the viewer in the default front view). Exported with "+Y Up" it then faces +Z in glTF, and the game turns it by 180° to Bevy's forward (−Z).
- **Colour per slot:** one material named exactly `Suit`. The game replaces its base colour per slot from an 8-colour palette, so the design has to work in any bright colour. Other materials (visor, boots, trim) keep their own flat colours.
- **Name tag:** an empty named `NameTag` where the tag should float, about 0.3 m above the head.
- **Budget:** about 3,000–5,000 triangles, a guide value (`DECISIONS.md`, triangle budgets); round parts get enough segments for a clean silhouette. Flat colours per material or vertex colours. No textures, no armature, no shape keys. Smooth shading with hard edges (`DECISIONS.md`, shading).
- **Export:** apply modifiers, selected objects only, no cameras or lights.

## Done when

- `walker.glb` loads in the game: the code repo's placeholder is replaced, and the figure shows in its slot colour with its name tag, in the cabin and outside.
- It reads at 60 m and fits the capsule.
- The initiator says it is funny.

## Open

- **Live view while iterating.** Blender 5.2.1 LTS is installed on the initiator's desktop, so the scripts run headless there. A live connection (blender-mcp: the agent drives the open Blender and gets viewport screenshots) would be a new dependency, the initiator's call. Without it, the script renders the review images itself.
