# Core Loop — EXO-1

Status: DRAFT, proposal. The initiator decides the core loop and the pillars (see `VISION.md`). Everything here is a suggestion until confirmed.

## One sentence

Schedule I in space: a fixed, hand-built world, short loops of take a job, fly, produce or haul, sell, and expand, with a ship you tune like a Gummi Ship.

## Loop

1. Take a contract at a city or outpost.
2. Fly there, land (seamless, no loading screen).
3. Gather, produce or pick up cargo.
4. Deliver, get paid (shared wallet).
5. Spend on ship upgrades or an outpost, which unlocks new places and contracts.

Target: a round of 3-5 minutes with visible progress each time (a new part, more cargo space, a new place) and a bit of risk and goofy text.

Analogy to Schedule I:

| Schedule I | EXO-1 |
|---|---|
| Fixed town, properties | Fixed system, one planet to start |
| Grow, produce | Mine, make, load cargo |
| Sell, earn money | Contracts, trade, money |
| Expand the empire | Outposts, production chains, ship upgrades |
| Unlock regions | New places and new contracts |

## Pillars (proposal)

1. **Simple and effective:** simple menus, simple assets, minimal HUD; gameplay and systems over graphics.
2. **Flying and landing feel good:** this is the core; if it is not fun, nothing else matters.
3. **Fixed, hand-built world with a dense city:** a small, dense hub like in Schedule I, not an empty open world.
4. **A ship you tune:** slot or budget based parts, visible on the ship, stats in the menu.
5. **Co-op makes it funnier:** 2-5 players, shared wallet, roles on the ship (pilot, gunner, cargo).
6. **Content is data:** places, goods, parts and contracts are validated data, so the community can add them.

## Say no to (proposal)

- Procedural galaxy
- Free-form voxel ship building and Newtonian flight physics
- Simulated full economy, fleet command
- Roguelike reset, story with cutscenes, voice acting
- Walkable stations in the MVP (landing is a menu or a small city)
- Anything copied from other games (names, assets, texts, factions)

## Inspirations to take mechanics from (not names or assets)

- **Freelancer:** landing as a menu (market, bar and contracts, workshop, hangar), simple arcade flight, price differences between places.
- **Privateer:** job types (delivery, smuggling, escort, patrol, hunt), goofy tone.
- **Kingdom Hearts Gummi Ship:** block or slot ship with a budget, stats from parts.
- **Starsector:** factions that ban different goods, black market and risk (later).
- **Schedule I:** short loops, cosy management, visible progress, co-op chaos.

Source note: the design research was shallow (search was limited); Elite, X, Space Engineers and parts of Schedule I are general-knowledge assessments, not researched.

## MVP scope (A1 baseline)

- 1 star system, 1 planet (seamless), 1 small ship
- 1 dense city (maybe 2-3 districts), 2-3 outposts
- Arcade flight, landing by button press, seamless atmosphere transition
- About 6 goods with different prices per place, cargo limit, shared wallet
- 3 contract templates (delivery, hunt, escort or smuggling)
- Slot or budget ship tuning: about 5 slot types, 3-4 parts each
- 2 players first; 5 later
- Minimal HUD (hull and shield, money, cargo, target arrow)
- Data schema from day one, even with few entries

Interiors: small shops stay in the open world; large or complex interiors (for example a sewer) are instanced.

## Content schema (first sketch)

One file per object, validated:

- `commodity`: id, name, base price, volume, tags, illegal-per-faction
- `location`: name, faction, produces and demands (multipliers), menu tabs, flavour text, coordinates on the planet
- `planet`: seed, radius, list of locations
- `faction`: relations, banned goods, penalties
- `mission_template`: type, text with placeholders, reward formula, conditions
- `part`: slot type, cost, budget load, stat modifiers, mesh reference
- `hull`: slots, base stats, budget
- `event`: trigger, effect on prices or spawns
- Strings in separate localisation tables

## Later (after the MVP)

Second faction with reputation and black market, ranks, employees or autopilot freighters (the Schedule I idea), events, more planets (community vote).

## Open design questions

- Roles on one ship (pilot and gunner) or one ship per player?
- How much combat, and of what kind?
- Is production (making goods yourself) part of the MVP or only hauling and trading?
- Planet radius feel (start 3 km): tune in the planet spike.
- Tone and name of the "strange galaxy".
