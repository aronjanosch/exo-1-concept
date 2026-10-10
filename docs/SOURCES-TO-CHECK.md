# Sources to check — EXO-1 (working title)

Status: list for manual checking, cleaned up on 2026-10-09 (the Godot-specific open items and the search log of 2026-10-03 are in `archive/SOURCES-TO-CHECK-GODOT-ERA.md`). Saved copies live in `research/sources/` (private repo, third-party content, never publish). Results go into `CORE-LOOP.md` or a note in `docs/research/` with a **[verified]** tag.

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
| Coupled thrust direction (`deng0/SimpleFlightComputer`), a measured Newtonian flight step (`emcodem/sc_webgl`), Alpha 2.4 action maps (`jllamas/StarCitizenActionMaps`); read 2026-10-10, nothing stored | `docs/research/flight-controller-notes.md` |

## Still open, by priority

### Needs a decision soon

| What | Where | Why |
|---|---|---|
| Trademark check for the new name | TMview (covers EUIPO, DPMA and USPTO) | Registers were not reachable by tool; "EXO-1" is taken in the same field by the game "Exo One" (https://store.steampowered.com/app/773370/Exo_One/). No legal advice |

### Nice to have

| What | Where | Why |
|---|---|---|
| Schedule I developer interview or postmortem | Search; none found so far | Only Wikipedia, a studio blog and Steam threads |
| POI density numbers | Postmortems of Outer Wilds, Valheim, Subnautica | Numbers in the research are guesses; use only as playtest starting values |
| Dead Space diegetic UI, primary source | GDC 2013 talk: http://www.gdcvault.com/play/1017723/Crafting-Destruction-The-Evolution-of | The saved article is secondary |
| Outer Wilds reference frames | Developer talks or interviews on moving bodies | Only forum guesses; not critical while planets do not orbit |
| Elite Dangerous planetary tech | Frontier forum recap https://forums.frontier.co.uk/threads/planetary-tech-with-dr-kay-ross-recap.565755/ , save again (first attempt captured only the cookie banner) | Procedural plus hand-built mix, which is our approach |
| Netcode talks | Overwatch GDC 2017 (https://www.gdcvault.com/play/1024001/-Overwatch-Gameplay-Architecture-and, not opened), Valve source-engine networking docs | Snapshot interpolation reference for `net_core` |
| More Starsector posts | https://fractalsoftworks.com/2014/03/02/on-trade-design/ , https://fractalsoftworks.com/2016/04/20/economy-revamp/ , https://fractalsoftworks.com/2017/09/19/economy-outposts/ , https://fractalsoftworks.com/2018/01/03/revisiting-the-economy/ | Optional; the saved posts already cover the points we need |

### Legal and assets (before publishing)

| What | Where | Why |
|---|---|---|
| Meshy terms of service | Meshy website | Pricing docs (read 2026-10-08): free-plan output is CC BY 4.0 (credit Meshy), paid plans private; not CC0, so it does not fit our asset licence. Full terms text still unread |
| Tripo free-tier output licence | Tripo terms and pricing page | Sources contradict each other (non-commercial versus CC BY 4.0) |
| Hunyuan3D 2.1 licence territories | Licence text | Reported to exclude the EU, UK and South Korea; secondary source |
| Kenney and Quaternius packs actually used | Licence file inside each pack | Per-pack details not checked |
| AI-generated asset copyright | Primary legal sources (US Copyright Office reports, German UrhG commentary) | Current picture rests on secondary sources; no legal advice |
