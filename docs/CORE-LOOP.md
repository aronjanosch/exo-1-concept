# Core Loop — EXO-1 (working title)

Status: DRAFT, rewritten 2026-10-09 after the first loop playtest and the identity grilling. The initiator decides; every line here traces back to `DECISIONS.md` (rows from "Depth and progression (direction)" to "The arc from courier to boss"). Numbers are starting values for playtests, not rules.

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

## Say no to (for the first version)

- An open sandbox without a goal (the empire is the goal; opening up and user-generated content come later)
- Real drugs or real-world brands
- Fixed burdens that feel bad (debts)
- A procedural galaxy
- A simulated full economy
- Cutscenes and voice acting
- Anything copied from other games (names, assets, texts, factions, machines)

## Inspirations (mechanics, not names or assets)

- **Schedule I:** hands-on production, customers with taste, staff, districts, a small dense town.
- **Gangland:** rising inside a crime family, territory, rival families.
- **Star Citizen:** licensed careers, hauling, physical cargo, ships as tools.
- **No Man's Sky:** varied side jobs (mining missions, hits), discovery.
- **Rick and Morty:** tone.
- Earlier references stay valid: Freelancer and Privateer (job types, goofy tone), Outer Wilds (small worlds, licence-like first launch), Starsector's lessons on keeping trade simple and smuggling as a risk layer.

## Research behind it

`research/loop-references.md` (job structures), `research/loop-feel.md` (beats, feedback, framing), `research/scale-and-early-game.md` (planet scale, onboarding, licences).

## Playtest plan

Greybox playtests with 3-5 people: flight and landing; one production chain end to end; the business session (orders, police, rival); co-op with 4-5. Measure what players do voluntarily, how long a business day takes, and where they wait.

## Open design questions

- Names: the game, the family, the rival, the districts, the goods (working title stays; placeholders by agents are marked `TODO(initiator)`).
- The first goods and machines: which invented substances, which effects, which machines.
- Planet scale: a short spike on Hearth (`research/scale-and-early-game.md`).
- How staff works and how much is automated (Schedule I's late-game burnout warning).
- How the family's share is set and when the break becomes possible.
