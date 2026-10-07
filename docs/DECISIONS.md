# Decisions — EXO-1 (working title)

Status: living document. Date of this version: 2026-10-07 (multiplayer authority added after spike 4; Rust generator and no web export added after spike 6; movement on planets added after spike 8; Rust/Bevy, private project and studying other games' files added 2026-10-07; agent tooling added after spike 9b). Each entry says whether it is decided, open or parked. Details and sources: `FEASIBILITY.md`, `CORE-LOOP.md`.

## Decided

| Topic | Decision |
|---|---|
| Engine | **Fully Rust, Bevy**, replacing Godot 4 and GDScript, once a validation spike shows that everything works as well or better (spike 9, `SPIKE-9-BRIEF.md`). Initiator, 2026-10-07: "Das ist jetzt meine entscheidung wir gehen komplett auf rust. einen spike test davor um das zu validieren dass alles genau so gut oder besser klappt." and "Ja bevy spike und dann commiten wir einfach." Reasoning given: if physics is on par or better and the framework fits agentic coding better, Godot's advantage is gone, since all work so far was done 100 % by agents. Until spike 9 is done the Godot spikes stay the reference. Earlier entry: Godot 4, GDScript, pinned version, no double-precision build, no custom engine build |
| Private project | EXO-1 is re-oriented as a private project, same idea. Initiator, 2026-10-07: "exo-1 ersmal neu ausrichten. gleiche Idee nur als privates Projekt." The community parts (votes, `community-gate`, video series, licence) stay in the docs as a possible later step. Initiator, 2026-10-07: "die community bezüge lassen wir mal drinnen vll machen wir das irgendwann mal noch" |
| Terrain generator in Rust | **Superseded by "Engine" (2026-10-07) once spike 9 passes;** the generator stays Rust, the GDExtension layer goes. Earlier entry: Godot stays the engine. The terrain generator is written in Rust as a GDExtension (godot-rust), starting from `gen_core` of spike 6; other compute-heavy parts may follow when measured. Initiator, 2026-10-06: "wir fangen mal an aber anstatt dann immer alles wieder umzuschreiben in rust weil es doch besser funktioniert für den scale den wir haben". Spike 6: Rust 5-7x faster than GDScript on the test workload, same output, rebuild about 0.5 s with a non-LTO profile. The `AGENTS.md` hard rule on GDExtension still needs a proposal. Source: `SPIKE-6-REPORT.md` |
| No web export | No browser game. Initiator, 2026-10-06: "web ist raus wir machen kein browser game. auf keinen fall" |
| Platforms | Linux and Windows first; development happens on Linux. Initiator, 2026-10-07: "Wir supporten erstmal nur linux und win. lvv sogar nur windows und dann linux mit proton. aber ich etnwickel halt auf linux." Follow-up after spike 7, 2026-10-07: "Können auch linux nativ gehen kein problem", so Linux ships as a native build. macOS is not supported for now |
| Renderer | Godot-specific, superseded by "Engine" once spike 9 passes. Earlier entry: Current choice: Forward+ with Vulkan (initiator, 2026-10-04). Best option we think fits right now, not a permanent requirement; revisit with measured performance and visual correctness. Forward+ passed the distant-surface depth tests; fastest renderer has not been established by an A/B benchmark |
| World | Fixed hand-built system. Small but complete, seamless planets. Procedural terrain, hand-built city and outposts |
| Planet size | Radius 5 km as a first guide value. Larger and smaller planets are possible |
| Movement on planets | Walking anywhere is possible; the player decides whether to walk or take the ship. Natural obstacles (steep slopes, water) may stop a walker, no path around them is guaranteed. Initiator, 2026-10-07: "die möglichkeit gibts auch zu Füß muss er selbst wissen ob er das machen will. warum sollte er das nicht amchen können? wenn es natürliche hindernisse gibt dann kann er halt nicht weiter". Source: `SPIKE-8-REPORT.md` |
| Interiors | Small shops stay in the open world, large or complex interiors (for example a sewer) are instanced |
| Scale | City is small, dense and simple, like Schedule I |
| Players | One player, one ship first. More players after the core works |
| Flight | Arcade, starting values only, tune by feel. Long flights are fine if there is something to do on board |
| Ship tuning | Not planned and out of scope. May come later. The Gummi-Ship reference is dropped entirely |
| Assets | Scripts instead of binaries, simple, readable look (chunky low-detail shapes, smooth shading, flat colours or simple painted textures, light and haze carry the mood, cartoonish characters; Schedule I as a reference), CC0 placeholders (Kenney, Quaternius) marked as such. Models work via script and/or MCP, not native 3D |
| Content format | JSON with JSON Schema, one file per object, never `.tres`. Adopted as proposed in the research, still to be reviewed in detail |
| Inspiration | Learning from other games is welcome and encouraged (mechanics, ideas, public code, write-ups, how others implemented things). Looking into other games' files to understand how something was done is allowed (example: Star Citizen); the best parts are then reimplemented, nothing is taken over 1:1. Initiator, 2026-10-07: "Ich will nichts von Star Ciritizen und 1zu1 in exo-1 packen ich will verstehen wie es gemacht wurde und dann die besten Teile reimplementieren. ZUm verstehen muss man aber rein gucken. So ist es in jedem Feld der Technik". What we do not put into the repo: other games' code, assets, data, names or texts |
| Third-party code | Build core systems ourselves, use others as inspiration, take small parts only |
| Dev MCP | For development we use the best MCP, not the safest. Automated-test MCP is designed separately later |
| Licence | MIT for code, CC0 for assets with a provenance file, DCO instead of CLA (proposal from research) |
| AI stance | Accepted tension with engine communities that ban AI. EXO-1 is an experiment, we document what happens |
| Slop defence | Community plus the `community-gate` framework. Governance details (voting, Sybil protection, quorum) are worked out there |
| Trade balance | Never perfect, and perfection is not the goal |
| Large worlds | Origin shift (the world moves back when the player gets far from the origin). Shared snapshots carry planet id plus planet-relative pose, so each client shifts independently; verified on two computers (spike 4). Threshold still open. Source: `SPIKE-5-REPORT.md`, `SPIKE-4-REPORT.md` |
| Floating origin (Bevy) | Own render origin: physics in f64 world space (Avian `f64`), only what the GPU sees shifts; no big_space. Bevy 0.19, Avian 0.7 f64. Initiator, 2026-10-07: "dann eigene lösung und bevy 19", confirmed with "ja". Source: `SPIKE-9-REPORT.md` |
| Cheating | Not a concern; performance comes first (initiator, 2026-10-04) |
| Multiplayer authority | Client authority: each client simulates its own player and ship, a host relays snapshots (initiator, 2026-10-06, after spike 4 passed on two computers). Ships are meant to affect each other physically (initiator, 2026-10-06); the rule for who owns a contact is open, see below. Source: `SPIKE-4-REPORT.md` |
| Agent tooling (Rust/Bevy) | After spike 9b (`SPIKE-9B-REPORT.md`): of the third-party Bevy skills only the two that checked out without errors stay (`bevy-ecs-queries`, `bevy-testing`); the others are not corrected or maintained, agents look the API up in the Bevy source instead. BRP (`remote` feature) is not carried forward: the blind test showed no gain against the scenario runner, and it adds about 30 dependencies and an HTTP server (red-class). The code stays on branch `spike/bevy-tooling` as reference; revisit only for a bug the scenario runner cannot reproduce. Initiator, 2026-10-07, on this recommendation: "ja passt" |
| Code repo layout and risk classes (Rust/Bevy) | One Cargo workspace at the repo root, all crates in `crates/` (simulation in `*_core` crates without Bevy types, Bevy glue in the game crate), content in `content/` as the Bevy asset root. Chosen by the agent on the initiator's delegation, 2026-10-07: "nimm das beste für unser projekt". Risk classes: green `content/**`, yellow `crates/**`, red CI, dependencies, `.cargo/`, `build.rs`, `unsafe`, networking. Initiator, 2026-10-07: "classes passt". Source: `AGENTS-CODE-DRAFT.md` |

## Open

- **Name:** "EXO-1" cannot stay. A game named "Exo One" (developer Exbleative, Steam and consoles, since 2021) already exists in the same field. Official trademark registers (EUIPO/TMview, DPMA, USPTO) were not checked. Until a new name is chosen, "EXO-1" is only a working title. Do not use it in videos, thumbnails, domains or store pages. The docs and repo names still use it for now.
- Where votes run. Proposal from research: discussion on GitHub, Discord only for polls.
- Details of the content schema (initiator wants to review).
- AgentBridge interface (research proposal: `get_state`, `do_action`, `step`, `reset`, no `eval`; see `FEASIBILITY.md`).
- Travel between planets: direction like No Man's Sky or Star Citizen, to be tried.
- Contact authority between ships: ships should affect each other physically (initiator, 2026-10-06), but with client authority each side computes a contact alone and the two histories can disagree (spike 4 fixture: 217 ms and 3 m apart). Candidates: pair rule (one fixed side computes the contact for both), migrating authority for touching/docked groups, damage-only ramming where each owner decides its own damage. Which interactions (ramming, docking, towing) and which rule: not decided.
- Data from other games' files (decided: look, understand, reimplement): whether values read locally may serve as validation targets in tests, and where such local extracts live (outside the repo, gitignored). The game's terms of use forbid extraction.
- Voter builds: desktop binary or web export. Note: the game itself has no web export (decided 2026-10-06); whether that also rules out web voter builds is not decided.
- Planet generator after spike 8: site spacing (24 sites leave a 3.4 km worst gap; denser or more even?), sea-level rule (70 % on the macro field or on the full height), which biome row is the broken rim. Source: `SPIKE-8-REPORT.md`.
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
3. Spike 9: Bevy validation (`SPIKE-9-BRIEF.md`). Replaces the earlier plan for Godot test-infrastructure spikes 9-11 (headless speed, gdUnit4 versus GUT, lavapipe variance); headless speed is part of spike 9.
4. Spike 10: networking in Bevy (client authority and snapshots as in spike 4), after spike 9. Initiator, 2026-10-07: "Netzwerk als Spike 10". Brief still to write.
5. After spike 9: rewrite the code repo `AGENTS.md` for Rust/Bevy and the private project.
