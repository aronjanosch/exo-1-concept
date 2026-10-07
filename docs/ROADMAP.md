# Roadmap — EXO-1

Status: DRAFT. Milestones are coarse on purpose. Details get worked out when a milestone starts.

**Re-orientation (2026-10-07):** private project, Rust with Bevy once spike 9 passes (`DECISIONS.md`). Track A milestones and Track B are not updated yet.
Related: `CORE-LOOP.md` (loop, pillars, MVP scope), `FEASIBILITY.md` (research results, spikes), `DECISIONS.md` (decided, open, parked), `SOURCES-TO-CHECK.md`.
Each topic is a decision plus research task, not one document per topic.

Two tracks: the **experiment** (ideas, creativity, structure where needed) and the **video** (story lines and hooks). The experiment does not depend on the story.

---

# Track A — Experiment

## Milestones

- **A0 — Concept:** vision, governance, decisions below settled enough to start
- **A1 — Baseline (v0.0.1):** minimal playable Godot game in the public code repo, CI gates running, `community-gate` enabled
- **A2 — Dry run:** small group (about 5 people, then the existing ~50-person community) walks through one full proposal-to-merge cycle; fix the process
- **A3 — Public:** open Discord and GitHub, first real community cycles
- **Later:** expansion by community vote (more systems, own-galaxy servers, tier-2 content, sponsorship)

## To define, research and think through

- **Core design:** gameplay loop and pillars (initiator decides), "what we say no to" to fend off scope creep
- **Minimal rules:** deliberately no long rulebook; as open as possible. What does not fit EXO-1 (for example a rewrite of another game) is simply not accepted, by vote and review
- **AI as amplifier:** use AI to boost ideas and creativity and keep what is worth playing; ideas, balancing and system design stay human, implementation is cheap
- **Self-organization:** how sub-groups, branch leads and community initiatives form without the initiator as bottleneck
- **Networking:** host-authoritative co-op, own-galaxy servers, how a galaxy is stored and shared, cross-server markets deliberately out for now
- **Content format:** how quests, POIs, NPCs and whole systems are described as validated data so AI-generated worlds can be contributed safely; schema design is arguably the real product
- **World generation:** procedural plus authored content, avoiding the "empty" feeling
- **Large-world limits in Godot:** precision, floating origin, why scope stays small
- **Bot- and AI-testable game:** the game must expose an interface so AI agents and bots can play and test it. Essential for a decentralised project, since reviewers cannot play every PR by hand
  - **AgentBridge (to validate):** a small autoload with `get_state`, `do_action` (fixed action list), `step` and `reset(seed)`, no `eval`. Start simple; whether it is viable is decided in the first spikes (see `FEASIBILITY.md`)
- **CI gates:** lint, format, tests, security lint, headless boot, bot playthrough through that interface, performance regression checks
- **Security:**
  - risky Godot APIs (`OS.execute`, shell, file access outside `user://`, networking, GDExtension, addons) and `.tscn`/`.tres` embedding scripts
  - PR builds run on voters' machines: build only after review, signed CI artifacts, maybe web export as sandbox
  - AI reviewer can be prompt-injected, so it only labels
  - malicious content and moderation, like mods and workshops
- **Proof-of-play:** server-measured playtime via session nonce and heartbeats, tied to PR commit SHA and Discord ID; PR-specific "proof event"; deterministic replay via bot; friction, not security
- **Voting rules for the game:** feature PRs need play-verification, pure discussion votes stay free; weighting, quorum, Sybil limits
- **Modularity:** many small scenes and data-driven content to limit `.tscn` merge conflicts; AI resolves the rest
- **Balancing:** telemetry so votes about "too easy" have evidence
- **Licensing:** MIT, original or CC0 assets, AI-generated assets and trademarks, DCO or CLA question, name and trademark check for "EXO-1"
- **Community setup:** Discord roles, GitHub linking, moderator promotion, `CONTRIBUTING.md`, `GOVERNANCE.md`, AI-use policy, trolls (spam PRs, vote brigading, malware attempts)
- **Automation:** the initiator has limited time, so deterministic CI, bots and the merge train carry the load; initiator handles red-class PRs and vetoes
- **Seeding:** playable baseline and good-first issues before opening up

## Open questions

- New name and trademark check: "EXO-1" collides with the existing game "Exo One" (see `DECISIONS.md`)
- Concurrent feature branches at the start
- Where voting runs: Discord, GitHub, own site
- Telemetry details and privacy
- Hosted builds for voters: desktop binary or web export
- Do cross-server markets ever happen
- Does runtime-shared content (data only, never code) ever happen
- What counts as a release: a vote threshold, a date, or both
- What happens to the repo when the experiment ends or is stopped

## Ideas parked

- Web of trust between this and other communities
- Moderators promoted by rule

---

# Track B — Video

## Milestones

- **B1 — Video 1:** slop problem and `community-gate` (see that repo's roadmap), ends with a pixelated teaser of the experiment
- **B2 — Video 2:** the experiment, first full proposal-to-merge run; should follow video 1 within 2-4 weeks
- **Ongoing:** recurring episodes around merge days

## Story lines

- **The slop problem:** AI makes contributions free, maintainers drown; AI slop is also what the viral mod mashups are (spectacle for attention, not something you want to play)
- **The turn:** the same AI can boost ideas and creativity; what was missing is a structure that keeps the humans in charge
- **The experiment:** a community builds a game together under that structure, failure included
- **Pioneer project:** the difficulty is organization, and we try to solve it live

## Hooks

- **Star Citizen meme:** "the game that never ships, we build it ourselves." Only works if it ships: public countdown or release vote, small scope. Name and assets stay original, referencing it in a title is fine
- **The modding wave:** passthrough mods (Minecraft inside Elden Ring and Skyrim, Tarkov mechanics in Skyrim) as the "why now", with the criticism acknowledged
- **Pixelated teaser** in the thumbnail and clips at the end of video 1
- **Drama as content:** best and worst PR, a caught malicious PR, a rejected mashup, the vote result
- Honest line: never perfect, but keeps out most slop

## Promotion

- Small channel (15 subs) plus ~50 community members and friends
- Hacker News (fits the governance angle best), r/godot, r/gamedev, Godot Discord, short clips for TikTok
- Sponsors (Cursor, OpenAI) only after weeks of public numbers

## Open questions

- Exact title and thumbnail
- How much of video 1 can be shown before the framework runs on a real repo (it needs proof, not a concept)
