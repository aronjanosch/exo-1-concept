# Core Loop — EXO-1 (working title)

Status: living proposal. The initiator decides the core loop and the pillars (see `VISION.md`). What is decided is in `DECISIONS.md` and summarised under "Milestone D" below; everything else here is a suggestion until confirmed. All numbers are starting values to be tuned in playtests, not rules.

## One sentence

Schedule I in space: a fixed, hand-built world, short loops of take a job, fly, land, do the job, get paid, and unlock more of the world, with simple menus and a minimal HUD.

## Loop

1. Take a contract at the city or an outpost.
2. Fly there, land (seamless, no loading screen).
3. Gather, pick up or deliver cargo.
4. Get paid (shared wallet in co-op).
5. Spend money on unlocks (new places, new contracts, later maybe a better ship) and repeat.

Target for the first prototype cycle: about 4-8 minutes, with visible progress each round and a bit of risk and goofy text. Long flights can be interesting if there is something to do on board; start with shorter ones and tune by feel.

Analogy to Schedule I:

| Schedule I | EXO-1 |
|---|---|
| Fixed town, properties | Fixed system, one planet to start |
| Grow, produce | Pick up, gather, haul |
| Sell, earn money | Contracts, trade, money |
| Expand the empire | Unlock places and contracts, later outposts |
| Unlock regions | New places and new contracts |

## Milestone D: the first loop (decided, provisional)

The decisions of 2026-10-09 are a starting point that the loop playtest may change (`DECISIONS.md`, "Cargo and the loop"). In short:

- A job is a template with objectives; milestone D builds one objective, *deliver*, with two or three modifiers. Jobs come from a seeded generator and sit on a board at a place; one player accepts, the whole crew can carry and deliver.
- Money: a fixed reward per job, graded by delivered share, condition and time. The only sink is buying unlocks. No market, no production in D.
- Progress is shared (crew wallet and unlocks) with a personal layer (freight XP, counted but gating nothing).
- Encounters (route events) are shown on the job up front; a thin pool of harmless ones runs on every flight. No combat.
- Each system is its own `*_core` crate on a small kernel, built in seven slow steps. Details and quotes: `DECISIONS.md`, research: `research/loop-references.md`.

## Pillars (proposal)

1. **Simple and effective:** simple menus, simple assets, minimal HUD; gameplay and systems over graphics.
2. **Flying and landing feel good:** this is the core; if it is not fun, nothing else matters.
3. **Fixed, hand-built world with a dense city:** a small, dense hub like in Schedule I, not an empty open world.
4. **Co-op makes it funnier:** one player and one ship first; more players follow once the core works.
5. **Content is data:** places, goods and contracts are validated data, so the community can add them.

## Say no to (proposal)

- Procedural galaxy
- Ship tuning and ship building in the MVP (not planned; may come later; the community may build it as part of the experiment)
- Full Newtonian simulation (parked); the flight model aims at Star Citizen's feel with limits per axis (`DECISIONS.md`, "Flight model (spike 13)")
- Simulated full economy, fleet command
- Roguelike reset, story with cutscenes, voice acting
- Walkable stations in the MVP (landing leads to the small city)
- Anything copied from other games (names, assets, texts, factions)

## Inspirations to take mechanics from (not names or assets)

- **Freelancer:** simple arcade flight, price differences between places, landing as a menu.
- **Privateer:** job types (delivery, smuggling, escort, patrol, hunt), goofy tone.
- **Schedule I:** short loops, cosy management, visible progress, co-op chaos. Known criticism to avoid: endgame burnout after roughly 20-40 hours ("everything automated, nothing to do") and co-op progression desync.
- **Outer Wilds:** small worlds, travel times of minutes, familiarity makes curiosity possible.
- **Star Citizen (direction only):** goes the full-simulator way. Whether EXO-1 can grow toward that step by step is part of the experiment.

## Flight and ships (starting values)

- Arcade flight with assist and a generous landing aid; no menus or loading screens between space and ground.
- Rough target for the first prototype: start to landing about 60-120 seconds, about 30 seconds without input or an event as a guide, 45 or 55 is fine (`DECISIONS.md`, "Travel time"). Not a rule, tune by feel.
- Later idea (parked): ship classes such as a slow, simple, easy-to-fly ship (like a drone you can control from a phone) versus a fast, heavy FPV-racer-like ship that is harder to fly.
- Events on a flight are *encounters*, a record each with trigger, weight, cooldown and effect (`DECISIONS.md`, "Encounters"). Starting idea: roughly one per minute, 8-12 hand-written ones (distress call, drifting cargo, broken freighter); pirates only once combat exists.

## HUD (starting values)

Decided: at most four permanent elements (mode, speed, altitude near a planet, boost); everything else only when relevant (`DECISIONS.md`, "HUD"). The target arrow and co-op markers come with the loop (build step 7). Test it; it may change.

Ideas from Dead Space's diegetic UI **[verified, secondary article about a GDC 2013 talk, `research/sources/dead-space-ui-medium.md`]**: a holographic locator line pointing to the destination instead of a map (fits our target arrow); status shown on the ship itself (Dead Space shows health as a light bar on the suit); loading and travel wrapped into a believable element (a tram ride) instead of a loading screen; and "usability trumps aesthetics" where the two clash. Diegetic is a style option, not a requirement.

## Trade and contracts (starting values)

- About 6 goods with different prices per place, cargo limit, shared wallet.
- Saturation per sale, rotating boom goods, money sinks to blunt dominant routes. It will never be perfect and does not need to be.
- Lessons from Starsector's designer **[verified, own blog posts from 2014 and 2018, `research/sources/starsector-*.md`]**:
  - In a supply-and-demand simulation prices drift to equilibrium where trade is not profitable; with a 30% tariff on both ends a plain A-to-B run is unprofitable unless something disturbs the balance. Profitable runs come from events (a food shortage raises one price and destabilises the others).
  - Avoid "spreadsheet hell": do not make the player compare every price at every market. Prices are reported as intel and the game only surfaces "interesting" ones (extremely high or low, cheap compared to other known prices, goods you carry a lot of), picked by weighted random. Price information is time-sensitive, so wandering around to note all prices is less worthwhile than using a known route.
  - The economy was rewritten about five times. The last version dropped colony-to-colony relationships for one accessibility rating per market and a global market value split by market share, because playtesting showed the old system was too complicated ("if I find myself being confused by the system, that's Not Good"). Lesson: playtest the loop early and keep trade simple.
  - Smuggling (black market without tariff, banned goods with higher margins, reputation loss, customs inspections) is a ready-made risk layer, parked for later.
  - For us: few goods, events as the source of opportunities, a small curated price overview, no full simulation.
- Contracts from templates (verb x place x modifier): delivery, hunt, escort or smuggling.
- Schedule I's contract fields as a checklist **[verified from field names, see `REFERENCE-NOTES.md`]**: payment, required goods, delivery place, delivery time window, expiry, optional counter-offer, bonus payments. Contracts and story quests share one state machine (begin, active, complete, expire, fail).

## Co-op (simple start)

- Milestone B brings friends in: host and join, a roster, one figure for all players (`ROADMAP.md`).
- Host save must survive reconnects without resetting progress.
- Later ideas: "warp to friend", parallel tasks, contract quantities that scale with player count.

## MVP scope (first idea, the milestones in `ROADMAP.md` replace the baseline)

- 1 star system, 1 planet (seamless), 1 small ship
- 1 dense city (maybe 2-3 districts), 2-3 outposts
- Arcade flight, landing by button press, seamless atmosphere transition
- About 6 goods, cargo limit, shared wallet
- 3 contract templates (milestone D starts with one objective, *deliver*)
- Minimal HUD
- Data schema from day one, even with few entries
- One player first; 2-5 later

Interiors: small shops stay in the open world; large or complex interiors (for example a sewer) are instanced.

## Content schema

One file per object, validated. The records for milestone D are decided: `commodity`, `site`, `job_template`, `unlock`, `progress_track`, `encounter`, with templates plus pools and strings in localisation tables (`DECISIONS.md`, "Loop content schema"). Planets carry their values per body (`DECISIONS.md`, "World values and scale"). Factions, ship parts and hulls are not part of it yet.

## Later (after the MVP, by community vote)

Second faction with reputation and black market, ranks, employees or autopilot freighters (the Schedule I idea), ship classes, ship tuning, more planets.

## Playtest plan

Playtests start at greybox with 3-5 people, in this order: flight and landing; the loop; then co-op with 4-5 once it exists. Sessions up to about one hour with focused questions instead of "is it fun". Measure cycle duration, voluntary repeats, waiting times and desync cases (target: zero).

## Open design questions

- Planet radius: see `DECISIONS.md`.
- How much combat, and of what kind? Not before the loop playtest; later vision in `DECISIONS.md`, "No combat before the loop".
- ~~Is production (making goods) part of the MVP?~~ Not in milestone D; each of farming and mining gets its own milestone after the loop playtest (`DECISIONS.md`, "Production (loop D)").
- Tone and name of the "strange galaxy" (and the game; the working title is not final, see `DECISIONS.md`).
