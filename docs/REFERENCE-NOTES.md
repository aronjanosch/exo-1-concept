# Reference notes — what we can learn from other games

Status: living document, in our own words. We learn from how others built things; we copy no code, assets, data, names or texts (see `VISION.md` and `DECISIONS.md`).

## Schedule I (from a public modding-aid repo with stripped scripts)

Source: `Skippeh/ScheduleOne_UnityProject`, read on 2026-10-03: class and field names only, all method bodies are removed. The repo has no licence and is not from the developers; nothing from it is stored in this repo. Everything below is **[verified from names and fields]** unless marked otherwise. Behaviour, numbers and units are **[unverified]**.

### Scope map (what a full game of this type contains)

The code is organised into many systems: economy (customers, dealers, suppliers, dead drops), employees, delivery, police and law with a heat map, levelling (XP), money, property, quests, dialogue, persistence, networking, weather, game time, a global variables store, crafting stations, storage, vehicles and more. For EXO-1 this is a map of what we do **not** build in the MVP: employees, police and heat, casino, cartel, weather, XP.

### Contracts are quests

- A contract is a quest subclass, so story quests and contracts share one state machine (begin, active, complete, expire, fail). We can do the same: one mission state machine, with the schema deciding whether it is a story quest or a contract.
- Contract fields: payment, list of required goods, delivery location (referenced by an ID), a delivery time window, an expiry flag and time, a pickup schedule, and a counter-offer flag (the player can negotiate). Bonus payments exist (title plus amount), and over-delivery or quality mismatch is scored by a matching rule. A default expiry constant exists (unit unknown).
- Customers have affinities for product types and a satisfaction value, and standards they expect. So the economy is person-based (who wants what), not a market simulation. Matches the lesson from the Starsector posts: small, event- and person-driven instead of a full simulation.

### Story content is code, not data

About 25 story quests are separate hand-written classes in a chain (each with its own script) plus prefab data. Authoring is expensive and not community-friendly. Our plan (quests and contracts as validated JSON data, a few hand-authored story beats) is the opposite and fits the community experiment.

### Persistence

Every saveable thing has a stable GUID and its own data class plus its own loader (hundreds of small files). Lesson for us: give every persistent object a stable ID from day one, but use one generic save format (a dictionary tree) instead of a class pair per type.

### Networking

The code uses the FishNet networking library, and quest end calls take a flag for whether to replicate over the network. Whether it is host-client or dedicated is not checked. Known co-op issues from player reports (progress desync) are listed in `CORE-LOOP.md`.

### Quest data

A quest has a title, subtitle, description, an ID, a flag to track it on start, an expiry visibility, auto-completion when all entries are done, an XP reward, a list of entries, an icon and a map point of interest. Useful checklist for our `mission_template` schema.

## Starsector, Dead Space, NMS, Gaffer, Godot docs

See `CORE-LOOP.md` and `FEASIBILITY.md`; saved copies in `research/sources/`.
