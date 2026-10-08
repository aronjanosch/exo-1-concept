# Designer brief: the walker figure

Status: open. Date: 2026-10-08. Owner: the initiator, with a designer agent that writes Blender Python scripts.
Related: code repo issue #32 (the plumbing that loads this file), `GLOSSARY.md` in the code repo (Walker versus Character model), `VISION.md`.

## What we need

How other players look in EXO-1: for milestone B, one figure that every player wears. Each slot shows it in its own colour, with a name tag above the head. It is seen from 2 m (in a cabin) to about 60 m (across a landing pad). Choosing a species per player comes later (code repo #59, see `DECISIONS.md`, figure style).

The figure is only the look (the **character model**). Movement and collision stay with the walker, a capsule 1.8 m tall and 0.35 m in radius. The figure must sit inside that space, roughly.

## Feel

- **Creatures, not objects.** Humans and aliens in the spirit of adult cartoon sci-fi (Rick and Morty as the style reference, nothing taken over 1:1). Initiator, 2026-10-08: "lass mal mehr richtung rick and morty aliens und menschen gehen. keine objekte".
- **One body plan, one absurd twist.** Mostly a human build (head, torso, two arms, two legs, everyday clothes), plus one strange deviation: a single big eye, eye stalks, a lump for a head, tentacles for legs, extra arms, no neck.
- **Extreme proportions.** Stilt legs under a heavy body, a big head on a thin body, a creature that is mostly head and legs.
- **Details with character, not surface noise.** Lids, brows, hands, clothes with collars, cords and belts add life; warts and bumps do not. Initiator, 2026-10-08: "weniger warzen auf den aliens. das sind keine interessanten details."
- **The eyes carry the joke.** Big white round eyes with small dot pupils, often a bit cross-eyed or uneven.
- **Everyday clothes in space.** T-shirt, jacket, bathrobe, coat. The clothes are the `Suit` material and take the slot colour; skin keeps its own colour.
- **Colours.** Saturated but slightly dirty: mustard, salmon, poison green, purple. Flat colours, no textures.
- **Readable.** A strong silhouette that still reads at 60 m; no fine detail that disappears at 30 m.
- **Original.** Build our own creatures. They must not resemble a known character from the show or anywhere else, and nothing is copied from a model library unless it is CC0. Invented names only, no personal data.

## The process

1. **Blockouts** (done 2026-10-08): four figures from one shared body-plan script, `art/walker/walker.py` on the code repo branch `feat/walker-blockouts`: Norb (human), Glibbo, Zorp, Wobbel. The initiator picked **Norb**.
2. **Norb, built like Schedule I** (in progress): one seamless base body from a joint skeleton (Blender skin modifier and subdivision), no face. On top: eyes with upper and lower lids, brows, nose and ears in 3D; mouth, blush, freckles and the clothes (collar, seams, print, belt, pockets, laces) painted on a texture the script generates. The Suit zone is painted in grey so the slot colour multiplies on top. Hairstyles from a small library (mop, mullet, side part, spikes): a short cap plus chunky locks, melted into one smooth surface.
3. **Character builder**: base body and parts split from a recipe (proportions, colours, eyes, hairstyle, painted layers), Norb as the first recipe; aliens get their own base bodies and parts.
4. **The final model and the export**, checked against the contract below.

Still open on Norb: the head flows into the neck like a cone; nose and ears show seams against the body.

There is no animation yet: remote players slide over the ground in one pose. Pick a pose and shapes that do not look broken when they slide.

## Technical contract (must match code repo issue #32)

- **File:** glTF 2.0 binary, `content/models/walker.glb` in the code repo. The source is a Blender Python script, `art/walker/walker.py` (not shipped); `blender -b -P art/walker/walker.py` builds the figure and writes the `.glb`. No `.blend` is committed (`DECISIONS.md`, model source). The initiator reviews the result in Blender.
- **Scale and axes:** 1 unit = 1 m. About 1.8 m tall. Origin at the feet, centred. In Blender the figure faces −Y (towards the viewer in the default front view). Exported with "+Y Up" it then faces +Z in glTF, and the game turns it by 180° to Bevy's forward (−Z).
- **Colour per slot:** one material named exactly `Suit`. The game replaces its base colour per slot from an 8-colour palette, so the design has to work in any bright colour. Other materials (visor, boots, trim) keep their own flat colours.
- **Name tag:** an empty named `NameTag` where the tag should float, about 0.3 m above the head.
- **Budget:** about 3,000–5,000 triangles, a guide value (`DECISIONS.md`, triangle budgets); Norb is at about 12,000 for now. Flat colours plus one painted texture for the body (`DECISIONS.md`, character build). No armature, no shape keys yet. Smooth shading with hard edges (`DECISIONS.md`, shading).
- **Export:** apply modifiers, selected objects only, no cameras or lights.

## Done when

- `walker.glb` loads in the game: the code repo's placeholder is replaced, and the figure shows in its slot colour with its name tag, in the cabin and outside.
- It reads at 60 m and fits the capsule.
- The initiator says it is funny.

## Open

- **Outlines.** Dark outlines carry much of the cartoon look (for example an inverted hull). They cost some performance and are a separate decision.
- **Live view while iterating.** Blender 5.2.1 LTS is installed on the initiator's desktop, so the scripts run headless there. A live connection (blender-mcp: the agent drives the open Blender and gets viewport screenshots) would be a new dependency, the initiator's call. Without it, the script renders the review images itself.
