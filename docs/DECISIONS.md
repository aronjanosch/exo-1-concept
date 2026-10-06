# Decisions — EXO-1 (working title)

Status: living document. Date of this version: 2026-10-06 (multiplayer authority added after spike 4; Rust generator and no web export added after spike 6). Each entry says whether it is decided, open or parked. Details and sources: `FEASIBILITY.md`, `CORE-LOOP.md`.

## Decided

| Topic | Decision |
|---|---|
| Engine | Godot 4, GDScript. Pin the version. No double-precision build, no custom engine build |
| Terrain generator in Rust | Godot stays the engine. The terrain generator is written in Rust as a GDExtension (godot-rust), starting from `gen_core` of spike 6; other compute-heavy parts may follow when measured. Initiator, 2026-10-06: "wir fangen mal an aber anstatt dann immer alles wieder umzuschreiben in rust weil es doch besser funktioniert für den scale den wir haben". Spike 6: Rust 5-7x faster than GDScript on the test workload, same output, rebuild about 0.5 s with a non-LTO profile. The `AGENTS.md` hard rule on GDExtension still needs a proposal. Source: `SPIKE-6-REPORT.md` |
| No web export | No browser game. Initiator, 2026-10-06: "web ist raus wir machen kein browser game. auf keinen fall" |
| Renderer | Current choice: Forward+ with Vulkan (initiator, 2026-10-04). Best option we think fits right now, not a permanent requirement; revisit with measured performance and visual correctness. Forward+ passed the distant-surface depth tests; fastest renderer has not been established by an A/B benchmark |
| World | Fixed hand-built system. Small but complete, seamless planets. Procedural terrain, hand-built city and outposts |
| Planet size | Radius 5 km as a first guide value. Larger and smaller planets are possible |
| Interiors | Small shops stay in the open world, large or complex interiors (for example a sewer) are instanced |
| Scale | City is small, dense and simple, like Schedule I |
| Players | One player, one ship first. More players after the core works |
| Flight | Arcade, starting values only, tune by feel. Long flights are fine if there is something to do on board |
| Ship tuning | Not planned and out of scope. May come later. The Gummi-Ship reference is dropped entirely |
| Assets | Scripts instead of binaries, simple, readable look (chunky low-detail shapes, smooth shading, flat colours or simple painted textures, light and haze carry the mood, cartoonish characters; Schedule I as a reference), CC0 placeholders (Kenney, Quaternius) marked as such. Models work via script and/or MCP, not native 3D |
| Content format | JSON with JSON Schema, one file per object, never `.tres`. Adopted as proposed in the research, still to be reviewed in detail |
| Inspiration | Learning from other games is welcome and encouraged (mechanics, ideas, public code, write-ups, how others implemented things). What we do not copy: code, assets, data, names or texts. Own implementation keeps the MIT and CC0 licences clean |
| Third-party code | Build core systems ourselves, use others as inspiration, take small parts only |
| Dev MCP | For development we use the best MCP, not the safest. Automated-test MCP is designed separately later |
| Licence | MIT for code, CC0 for assets with a provenance file, DCO instead of CLA (proposal from research) |
| AI stance | Accepted tension with engine communities that ban AI. EXO-1 is an experiment, we document what happens |
| Slop defence | Community plus the `community-gate` framework. Governance details (voting, Sybil protection, quorum) are worked out there |
| Trade balance | Never perfect, and perfection is not the goal |
| Large worlds | Origin shift (the world moves back when the player gets far from the origin). Shared snapshots carry planet id plus planet-relative pose, so each client shifts independently; verified on two computers (spike 4). Threshold still open. Source: `SPIKE-5-REPORT.md`, `SPIKE-4-REPORT.md` |
| Cheating | Not a concern; performance comes first (initiator, 2026-10-04) |
| Multiplayer authority | Client authority: each client simulates its own player and ship, a host relays snapshots (initiator, 2026-10-06, after spike 4 passed on two computers). Ships are meant to affect each other physically (initiator, 2026-10-06); the rule for who owns a contact is open, see below. Source: `SPIKE-4-REPORT.md` |

## Open

- **Name:** "EXO-1" cannot stay. A game named "Exo One" (developer Exbleative, Steam and consoles, since 2021) already exists in the same field. Official trademark registers (EUIPO/TMview, DPMA, USPTO) were not checked. Until a new name is chosen, "EXO-1" is only a working title. Do not use it in videos, thumbnails, domains or store pages. The docs and repo names still use it for now.
- Where votes run. Proposal from research: discussion on GitHub, Discord only for polls.
- Details of the content schema (initiator wants to review).
- AgentBridge interface (research proposal: `get_state`, `do_action`, `step`, `reset`, no `eval`; see `FEASIBILITY.md`).
- Travel between planets: direction like No Man's Sky or Star Citizen, to be tried.
- Contact authority between ships: ships should affect each other physically (initiator, 2026-10-06), but with client authority each side computes a contact alone and the two histories can disagree (spike 4 fixture: 217 ms and 3 m apart). Candidates: pair rule (one fixed side computes the contact for both), migrating authority for touching/docked groups, damage-only ramming where each owner decides its own damage. Which interactions (ramming, docking, towing) and which rule: not decided.
- Voter builds: desktop binary or web export. Note: the game itself has no web export (decided 2026-10-06); whether that also rules out web voter builds is not decided.
- Open questions in `ROADMAP.md` that this research did not touch.

## Parked (not now, maybe later by community vote)

- Ship tuning and building
- Ship classes (slow simple drone-like ship versus fast heavy FPV-racer-like ship)
- Second faction, reputation, black market, employees and autopilot freighters
- Newtonian simulator direction (Star Citizen style)
- More planets and own-galaxy servers

## Next steps

1. Check the open sources in `SOURCES-TO-CHECK.md` (initiator, with help from transcripts).
2. Pick a new name and do a proper trademark check.
3. Spike 1: planet (see `FEASIBILITY.md`), in a throwaway prototype branch in the code repo.
4. Test-infrastructure spikes in parallel (headless speed, gdUnit4 versus GUT, lavapipe variance).
