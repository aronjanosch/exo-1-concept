# Decisions — EXO-1 (working title)

Status: living document. Date of this version: 2026-10-03. Each entry says whether it is decided, open or parked. Details and sources: `FEASIBILITY.md`, `CORE-LOOP.md`.

## Decided

| Topic | Decision |
|---|---|
| Engine | Godot 4, GDScript. Pin the version. No double-precision build, no custom engine build |
| World | Fixed hand-built system. Small but complete, seamless planets (start radius about 3 km, tunable). Procedural terrain, hand-built city and outposts |
| Interiors | Small shops stay in the open world, large or complex interiors (for example a sewer) are instanced |
| Scale | City is small, dense and simple, like Schedule I |
| Players | One player, one ship first. More players after the core works |
| Flight | Arcade, starting values only, tune by feel. Long flights are fine if there is something to do on board |
| Ship tuning | Not planned and out of scope. May come later. The Gummi-Ship reference is dropped entirely |
| Assets | Scripts instead of binaries, flat shading with palette texture, CC0 placeholders (Kenney, Quaternius) marked as such. Models work via script and/or MCP, not native 3D |
| Content format | JSON with JSON Schema, one file per object, never `.tres`. Adopted as proposed in the research, still to be reviewed in detail |
| Third-party code | Build core systems ourselves, use others as inspiration, take small parts only |
| Dev MCP | For development we use the best MCP, not the safest. Automated-test MCP is designed separately later |
| AgentBridge | Simple start: narrow interface with `get_state`, `do_action`, `step`, `reset`. No `eval` |
| Licence | MIT for code, CC0 for assets with a provenance file, DCO instead of CLA (proposal from research) |
| AI stance | Accepted tension with engine communities that ban AI. EXO-1 is an experiment, we document what happens |
| Slop defence | Community plus the `community-gate` framework. Governance details (voting, Sybil protection, quorum) are worked out there |
| Trade balance | Never perfect, and perfection is not the goal |

## Open

- **Name:** "EXO-1" cannot stay. A game named "Exo One" (developer Exbleative, Steam and consoles, since 2021) already exists in the same field. Official trademark registers (EUIPO/TMview, DPMA, USPTO) were not checked. Until a new name is chosen, "EXO-1" is only a working title. Do not use it in videos, thumbnails, domains or store pages. The docs and repo names still use it for now.
- Where votes run. Proposal from research: discussion on GitHub, Discord only for polls.
- Details of the content schema (initiator wants to review).
- Voter builds: desktop binary or web export.
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
