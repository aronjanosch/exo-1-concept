# Core Loop — EXO-1 (working title)

Status: DRAFT, rewritten 2026-10-09 after the first loop playtest and the identity grilling; it supersedes the earlier milestone-D proposal of the same day, whose still-useful parts are kept below. The initiator decides; every line here traces back to `DECISIONS.md` (the 2026-10-09 identity, loop and progression rows). Numbers are starting values for playtests, not rules.

## One sentence

Build a crime empire in a goofy, brutal galaxy: start broke, make and sell invented contraband with hands-on machines, haul it past the police, outgrow the family you started with, and take over the districts, the planet and more.

## Tone

Silly and satirical with real brutality, in the style of Rick and Morty: absurd world and characters, losses and violence that hit. The goods are invented illegal substances with comical effects, never real drugs.

## The arc (from courier to boss)

1. **Broke in the city.** Legal courier jobs, first on foot, then with the flight licence (a small practical exam). Noticed by a small, odd crime family.
2. **Errand runner for the family.** First illegal deliveries, a first rented room with machines, ranks in the family (the reputation track per giver).
3. **Own shop under the family's protection.** Own goods, own customers, own districts. The family takes a percentage of the profit (protection, contacts, goods); not a fixed burden.
4. **The break.** Break away or take the family over. From here it is your empire and the game opens up: new planets, an own family, war with the rival clan.

## The loop

**Minutes (the hands):** grow and mine the raw materials, process them at interactive machines (our own machines, not Schedule I's), package them into crates, carry and fly them to customers, get paid. Licences gate what you may fly and use.

**A session (the business):** customers order by their taste; orders become delivery jobs. The police wanted level rises with deliveries and goods aboard; scans at pads, checkpoints, raids on the room. The rival clan wins customers away from you when you do not serve them well. Legal jobs (courier, hauling, later others like Star Citizen's) earn clean money and need a front; side jobs like mining missions and hit jobs (No Man's Sky's variety) mix in.

**Over many sessions (the empire):** territory (a district is yours when most of its customers buy from you, shown on the map) and the operation (more machines, bigger rooms, staff who grow and deliver) are the main axes; fleet, reach, underworld standing and the wanted level grow along. Always show the next goal.

The earlier proposal framed the same loop as "Schedule I in space"; the comparison is kept as reference:

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

## Systems and where they come from

| System | Role in the empire | Status |
|---|---|---|
| Kernel (`gameplay_core`) | Domain events, conditions, tracks, unlocks, content loading, save envelope | Built (D) |
| Jobs (`jobs_core`) | Orders, deliveries, legal and illegal jobs, givers, ranks | Deliver objective built (D) |
| Production (new) | Growing, mining, interactive machines, recipes, quality, packaging | Next |
| Customers (new) | Named buyers with taste and relationship, districts, rival competition | Next |
| Police and heat (new) | Wanted level, scans, confiscation and fines, raids | First version: no chases |
| Licences | Per player, earned by exam: flight first, then cargo and ship classes, weapons with combat | Next |
| Places | Hand-placed: city, pads, outposts | Pads built (D); city models in `art/city` |
| Ships | Systems you switch on and off, cockpit displays, upgrade paths, heavy jobs, battles, pirates, stealing ships | Own sprints |
| FPS combat | Combat with cover; instances you fight through | Own strand |
| Flight and terrain | The core feel; planet scale | Parallel tracks |

## Pillars

1. **Depth at the right places.** Complexity the player enjoys working through (making, choosing jobs, outwitting the police), progression that is felt and aims at goals.
2. **Hands-on.** Machines, crates and ships are handled, not menus.
3. **Flying and landing feel good.** The core; if it is not fun, nothing else matters.
4. **A small, dense, hand-built world.** The city as the hub, districts as territory, not an empty open world.
5. **Co-op is a crew.** One grows, one mixes, one flies, one keeps watch. Unlocks and goals are shared; licences and personal tracks are per player.
6. **Content is data.** Goods, places, givers, customers, jobs and recipes are validated data.

## First version (small)

One city with two or three districts, one family, one rival clan, one growing chain and one small mining chain with a few machines, a rented room, the flight licence with an exam, legal courier jobs, customers with taste, the wanted level with scans and raids, the save. Ship sprints, FPS combat and more planets follow as their own strands.

The earlier proposal's MVP baseline (replaced by the milestones in `ROADMAP.md`): 1 star system, 1 planet (seamless), 1 small ship; 1 dense city (maybe 2-3 districts), 2-3 outposts; arcade flight, landing by button press, seamless atmosphere transition; about 6 goods, cargo limit, shared wallet; 3 contract templates (milestone D starts with one objective, *deliver*); minimal HUD; data schema from day one; one player first, 2-5 later.

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

## Content schema

One file per object, validated. The records for milestone D are decided: `commodity`, `site`, `job_template`, `unlock`, `progress_track`, `encounter`, with templates plus pools and strings in localisation tables (`DECISIONS.md`, "Loop content schema"). Planets carry their values per body (`DECISIONS.md`, "World values and scale"). Factions, ship parts and hulls are not part of it yet.

## Later (after the MVP, by community vote)

Second faction with reputation and black market, ranks, employees or autopilot freighters (the Schedule I idea), ship classes, ship tuning, more planets.

## Say no to (for the first version)

- An open sandbox without a goal (the empire is the goal; opening up and user-generated content come later)
- Real drugs or real-world brands
- Fixed burdens that feel bad (debts)
- A procedural galaxy
- A simulated full economy, fleet command
- Cutscenes and voice acting; a roguelike reset
- Ship tuning and ship building in the MVP (not planned; may come later; the community may build it as part of the experiment)
- Full Newtonian simulation (parked); the flight model aims at Star Citizen's feel with limits per axis (`DECISIONS.md`, "Flight model (spike 13)")
- Walkable stations in the MVP (landing leads to the small city)
- Anything copied from other games (names, assets, texts, factions, machines)

## Inspirations (mechanics, not names or assets)

- **Schedule I:** hands-on production, customers with taste, staff, districts, a small dense town; short loops, cosy management, visible progress, co-op chaos. Known criticism to avoid: endgame burnout after roughly 20-40 hours ("everything automated, nothing to do") and co-op progression desync.
- **Gangland:** rising inside a crime family, territory, rival families.
- **Star Citizen:** licensed careers, hauling, physical cargo, ships as tools; direction only, it goes the full-simulator way, and whether EXO-1 can grow toward that step by step is part of the experiment.
- **No Man's Sky:** varied side jobs (mining missions, hits), discovery.
- **Rick and Morty:** tone.
- **Freelancer:** simple arcade flight, price differences between places, landing as a menu.
- **Privateer:** job types (delivery, smuggling, escort, patrol, hunt), goofy tone.
- **Outer Wilds:** small worlds, travel times of minutes, licence-like first launch; familiarity makes curiosity possible.
- **Starsector:** lessons on keeping trade simple and smuggling as a risk layer.

## Research behind it

`research/loop-references.md` (job structures), `research/loop-feel.md` (beats, feedback, framing), `research/scale-and-early-game.md` (planet scale, onboarding, licences).

## Playtest plan

Playtests start at greybox with 3-5 people, in this order: flight and landing; one production chain end to end; the business session (orders, police, rival); then co-op with 4-5 once it exists. Sessions up to about one hour with focused questions instead of "is it fun". Measure what players do voluntarily, cycle duration, how long a business day takes, waiting times and desync cases (target: zero).

## Open design questions

- Names: the game, the family, the rival, the districts, the goods (working title stays; placeholders by agents are marked `TODO(initiator)`).
- The first goods and machines: which invented substances, which effects, which machines.
- Planet scale: a short spike on Hearth (`research/scale-and-early-game.md`); planet radius in `DECISIONS.md`.
- How staff works and how much is automated (Schedule I's late-game burnout warning).
- How the family's share is set and when the break becomes possible.
- How much combat, and of what kind? Not before the loop playtest; later vision in `DECISIONS.md`, "No combat before the loop".
- ~~Is production (making goods) part of the MVP?~~ Not in milestone D; each of farming and mining gets its own milestone after the loop playtest (`DECISIONS.md`, "Production (loop D)").
- Tone and name of the "strange galaxy" (and the game; the working title is not final, see `DECISIONS.md`).
