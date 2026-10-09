# Sources to check — EXO-1 (Godot-era version, archived 2026-10-09)

Archived copy. The current list is `../SOURCES-TO-CHECK.md`; this one keeps the Godot-specific open items and the search log of 2026-10-03.

Status: list for manual checking, cleaned up on 2026-10-03. Saved copies live in `research/sources/` (private repo, third-party content, never publish). Results go into `FEASIBILITY.md` or `CORE-LOOP.md` with a **[verified]** tag.

## Done (saved, read, entered in the docs)

| Source | Used in |
|---|---|
| NMS GDC 2017 talks (two transcripts, Hello Games) | `FEASIBILITY.md`, world design. Caveat: 9 years old, auto-generated transcripts |
| Starsector blog posts: trade and smuggling (2014), once more with feeling (2018), tag page `economy` | `CORE-LOOP.md`, trade |
| Dead Space UI lessons for VR (secondary article) | `CORE-LOOP.md`, HUD |
| GUT in CI (single-author blog post) | `FEASIBILITY.md`, CI |
| Schedule I modding-aid repo (names and fields only, nothing stored) | `REFERENCE-NOTES.md`, `CORE-LOOP.md` |
| Star Citizen datamined mechanics: `gitlab.com/painlabs/SCLogistics` (raw DataCore, no assets) and `github.com/StarCitizenWiki/scunpacked-data` (JSON); read 2026-10-08, nothing stored | `docs/research/star-citizen-datamining.md` |
| Star Citizen records mapped to our crates (gravity volumes/LAG, flight capacitor, stance sets, resource network); read 2026-10-08 | `docs/research/star-citizen-vs-exo1-mapping.md` |
| Early feature proposals derived from that (LAG as data, boost capacitor, minimal HUD); proposal 2026-10-08 | `docs/research/early-feature-proposals.md` |
| Godot docs on large world coordinates, Terrain3D double-precision notes, Gaffer on Games snapshot interpolation | `FEASIBILITY.md`, precision and network sync |
| Godot issues #122707, #112976, proposals #4925, #1281 (read via `gh`) | `FEASIBILITY.md`; corrected the "headless stall" claim |

## Newer sources search (2026-10-03): result

The search tool is weak (US-only, rate-limited) and found **no new GDC or GodotCon talks since 2020** on procedural planets or seamless transitions. That may be a search limitation. Worth searching by hand: GDC Vault, GodotCon playlists, SIGGRAPH "Advances in Real-Time Rendering". Findings (details in the scratchpad report, key points in `FEASIBILITY.md`):

- Opened and useful: Godot docs on large world coordinates (https://docs.godotengine.org/en/stable/tutorials/physics/large_world_coordinates.html), Godot 4.5 and 4.6 release pages (https://godotengine.org/releases/4.5/ , https://godotengine.org/releases/4.6/), Terrain3D double-precision notes (https://terrain3d.readthedocs.io/en/stable/docs/double_precision.html), Gaffer on Games on snapshot interpolation (https://gafferongames.com/post/snapshot_interpolation/, 2014, still valid), Kitten Space Agency changelog (https://kittenspaceagency.wiki.gg/wiki/Version_2026.6.9.4750).
- Verified by us with `gh`: current Godot is 4.7.2; issue #105256 (async readback) is open; cuberact (MIT) and Terrain3D (MIT) are real and active; planetary_terrain_renderer is Rust, Apache-2.0, a thesis project.
- Saved by the initiator but not usable: the Frontier forum recap (the saved file contains only the cookie banner, no thread text; try saving again with reader mode or by selecting the posts) and the Kitten Space Agency changelog (a plain developer changelog, nothing relevant, removed).
- Hits only (not opened, treat as unverified): Overwatch gameplay architecture GDC 2017 (https://www.gdcvault.com/play/1024001/-Overwatch-Gameplay-Architecture-and), Godot 4.7 RC announcement (https://godotengine.org/article/release-candidate-godot-4-7-rc-1/; 4.7 itself is verified).
- Nothing usable found for: Schedule I postmortem, newer trade design, minimal space HUD (only secondary literature), Star Citizen/Starfield/Space Engineers/Elite Odyssey technical talks (only wiki, press, forums).
- Judgement from the research (unverified): the big changes since 2017 (Nanite-style virtualised geometry, mesh shaders, server meshing) are not relevant for a Godot project with a simple look and 3 km planets.

## Still open, by priority

### Needs a decision soon

| What | Where | Why |
|---|---|---|
| Trademark check for the new name | TMview (covers EUIPO, DPMA and USPTO) | Registers were not reachable by tool; "EXO-1" is taken in the same field by the game "Exo One" (https://store.steampowered.com/app/773370/Exo_One/). No legal advice |
| `.tscn`/`.tres` format and embedded scripts | Current Godot docs page on the TSCN file format (the old URL returned 404, find the new location) | Our "no `.tres` for community content" rule rests on format knowledge; proposal #4925 supports it |
| `str_to_var` and `ConfigFile` with embedded objects | Godot class reference | Behaviour not checked; must not be used in the content path |

### Verify in spikes (no reading needed)

- Headless speed and stability on our Godot version (issue #122707 is unproven, see `FEASIBILITY.md`)
- Physics non-determinism in 3D/Jolt (maintainers say it is non-deterministic in general; the issue is about 2D)
- Lavapipe frame rate under CI (the 5-7 FPS claim came from search snippets)
- Which renderers use reverse-Z (it exists since Godot 4.3)
- Scene merging: test `gdmerge` (very young); see also https://github.com/godotengine/godot-proposals/issues/1281

### Nice to have

| What | Where | Why |
|---|---|---|
| Schedule I developer interview or postmortem | Search; none found so far | Only Wikipedia, a studio blog and Steam threads |
| POI density numbers | Postmortems of Outer Wilds, Valheim, Subnautica | Numbers in the research are guesses; use only as playtest starting values |
| Dead Space diegetic UI, primary source | GDC 2013 talk: http://www.gdcvault.com/play/1017723/Crafting-Destruction-The-Evolution-of | The saved article is secondary |
| Outer Wilds reference frames | Developer talks or interviews on moving bodies | Only forum guesses; not critical while planets do not orbit |
| Elite Dangerous planetary tech | Frontier forum recap https://forums.frontier.co.uk/threads/planetary-tech-with-dr-kay-ross-recap.565755/ , save again (first attempt captured only the cookie banner) | Procedural plus hand-built mix, which is our approach |
| Netcode talks | Overwatch GDC 2017 (link above), Valve source-engine networking docs | Snapshot interpolation reference for the ship-sync spike |
| More Starsector posts | https://fractalsoftworks.com/2014/03/02/on-trade-design/ , https://fractalsoftworks.com/2016/04/20/economy-revamp/ , https://fractalsoftworks.com/2017/09/19/economy-outposts/ , https://fractalsoftworks.com/2018/01/03/revisiting-the-economy/ | Optional; the saved posts already cover the points we need |

### Legal and assets (before publishing)

| What | Where | Why |
|---|---|---|
| Meshy terms of service | Meshy website | Pricing docs (read 2026-10-08): free-plan output is CC BY 4.0 (credit Meshy), paid plans private; not CC0, so it does not fit our asset licence. Full terms text still unread |
| Tripo free-tier output licence | Tripo terms and pricing page | Sources contradict each other (non-commercial versus CC BY 4.0) |
| Hunyuan3D 2.1 licence territories | Licence text | Reported to exclude the EU, UK and South Korea; secondary source |
| Kenney and Quaternius packs actually used | Licence file inside each pack | Per-pack details not checked |
| AI-generated asset copyright | Primary legal sources (US Copyright Office reports, German UrhG commentary) | Current picture rests on secondary sources; no legal advice |
| Godot asset library malicious cases | Search | No case found: a gap in the search, not an all-clear |
| Godot AI policy | https://contributing.godotengine.org/en/latest/pull_requests/pull_request_guidelines.html | Relevant for upstream bug reports only |
