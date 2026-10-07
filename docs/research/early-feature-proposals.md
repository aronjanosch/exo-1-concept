# Early feature proposals: LAG as data, boost capacitor, minimal HUD

Proposal, opened 2026-10-08. Three small things worth adding at the current stage, each derived from the Star Citizen records read in `docs/research/star-citizen-vs-exo1-mapping.md`. Status: **proposal, not decided.** The initiator decides; the design gaps are listed per feature so they can be answered before anything is built.

The philosophy here is the opposite of Star Citizen: we take the *idea* and cut it to a stub. Values below are assumptions for a playtest, not design. Nothing is taken from their code, data, names or texts.

## Why these three, now

The code today is a flight/walk/net tech demo. The HUD is debug text (`exo_app/src/view.rs:203`), the cabin has a hard-coded gravity constant (`exo_app/src/walker.rs:193`), and boost is a flat multiplier (`flight_core/src/lib.rs:256`). The three below are the smallest changes that make the existing systems read as a *game*: the ship interior becomes a real place with its own gravity, flying gets a resource to manage, and both become visible. All are self-contained, need no new dependencies, and fit the "every feature gets a scripted scenario" rule.

The other candidates (walker speed ladder, ship power on/off, zero-G suit, and the core loop itself) are recorded at the end so the thinking is not lost.

---

## Feature A — Cabin gravity (LAG) as data, with ramp and on/off

### Now

`exo_app/src/walker.rs:191-198`: inside a cabin the walker is handed `up = DVec3::Y` and `g = 9.81`, a hard-coded branch. There is no ramp, and no link to landed/flying state. `DECISIONS.md` ("Cabin gravity (LAG)") already decided the behaviour: off while landed, up over about 1 s after take-off, on in flight (also upside down), `G` forces it in a landed ship.

### Star Citizen

Gravity is a volume, not a planet property:
- `entities/area/gravityarea.xml` → `GravityAreaParams { active, uniform, fallOffInner, gravityMagnitude, roomBased }` plus a `gravityDirection` vector.
- `entities/area/gravitybox.xml` → same, with a box `size` and `filled`.
- `entities/roomsystem/roomgravity.xml` → a room carrying a room volume, an atmosphere container and an action area.
- `entities/scitem/ships/gravitygenerator/grgn_s00_template.xml` → gravity is produced by a normal item (`Type="GravityGenerator"`), so it can be switched and powered; `itemresourcenetworkglobal.xml` lists `@RN_resource_Gravity`.

### Simplified for us

A ship-local gravity description instead of a branch: a direction (ship space) and a magnitude, plus a strength 0..1 that ramps. The planet field stays the default when not in a cabin. Keep it as one small struct owned by the ship.

Sketch (all assumptions):
- direction: ship-local `-Y` (the cabin floor), so a tilted or upside-down ship just works.
- magnitude: reuse the planet's local gravity magnitude near the surface, or a fixed constant.
- strength: 0 while landed, ramps to 1 over ~1 s after take-off, 1 in flight, `G` toggles it in a landed ship.
- the walker reads `up` and `g` from this instead of the hard-coded pair.

### Payoff and fit

Makes the decided LAG real and data-shaped, supports upside-down flight for free, gives `G` a real function, and becomes the place a future ship power system (Feature E) switches gravity on/off. Directly on the path from `walker.rs:193` to the decided behaviour.

### Open questions (for the initiator)

1. Magnitude in a cabin: a fixed value, or the planet's gravity at the ship?
2. Ramp: ~1 s time constant (assumption), or a fixed ramp per state?
3. Do remote players' ships (proxies) get LAG too, or is it cosmetic on the local ship only for now?

---

## Feature B — Boost as a capacitor (simplified afterburner)

### Now

`flight_core/src/lib.rs:256`: `let mut boost = if input.boost { self.boost_factor } else { 1.0 };` — a flat ×5 while Shift is held, no cost.

### Star Citizen

Boost is an afterburner with a full capacitor model (per-ship `Ifcs` block): `CapacitorMax`, `CapacitorRegenPerSec`, `CapacitorRegenDelayAfterUse`, `CapacitorAfterburnerIdleCost`, ramp-up/down times, per-axis acceleration multipliers, a separate `AfterburnerNew` variant and a `NoFuelParams` fallback. There is also a game-mode damping ramp in `ifcs/ifcsgamemodeparams_default.xml`.

### Simplified for us

One meter in `ShipController` that drains while boosting and regenerates when not, with a short delay before regen starts. Boost scales with available charge instead of a constant. This is the same shape as the *decided* zero-G suit values (thrust 2 m/s², boost ×3, brake, roll; no fuel), so the two can share one vocabulary later.

Sketch (all assumptions):
- `boost_charge: f64` in `0..1`, starting full.
- drain while boosting, e.g. to empty in ~3 s.
- regen after a short pause, e.g. full in ~6 s.
- effective boost multiplier = `1 + (boost_factor - 1) * charge` (or a floor below which boost is off).
- expose `boost_charge` for the HUD (Feature D).

### Payoff and fit

Big change to flight feel (pillar 2) for one field and a few lines; replaces two constant branches in `step`; gives the first real HUD gauge. Keeps `boost_factor` as the ceiling, so it is a softening, not a redesign.

### Open questions

1. Drain/regen times and the regen-delay (assumptions above).
2. Does a low charge just weaken boost, or cut it off under a threshold?
3. Should boosting also be gated by a future power/fuel system (Feature E)?

---

## Feature D — A minimal HUD (not debug text)

### Now

`exo_app/src/view.rs:167-214`: a single Bevy `Text` node showing a debug line (mode, frame ms, chunk/patch counts, rescues, and a net line). Usable for development, not for play.

### Star Citizen

A full HUD/MFD suite (`hudparams/`, `ui/buildingblocks/.../ifcs/`), with aim gimbal markers, lead/lag pips and fading curves. We explicitly want the opposite: `CORE-LOOP.md` says at most about 5 permanent elements (hull and shield, money, cargo, target arrow, speed/altitude) and cites Dead Space's "usability trumps aesthetics".

### Simplified for us

A tiny set of readouts, and move the current debug line behind a flag. Keep it plain text/low-detail to match the greybox look; no diegetic work yet.

Proposed elements (assumptions, to be cut to ≤5):
- **Boost** — from Feature B, so it is visible immediately.
- **Speed + altitude** — already computed in the HUD today.
- **Mode** — walk / cabin / ship / zero-G (already effectively there).
- later: **money** and **cargo** once the core loop exists.
- debug line only with a key or a `--debug` flag.

### Payoff and fit

The largest "this feels like a game" change for a greybox, and it forces us to name the first values we actually track. It also makes Features A and B observable, which is what a playtest needs.

### Open questions

1. Which elements make the first cut, and where on screen (top-left text now)?
2. Keep the frame/chunk debug line on a toggle, or move it to the scenario report only?
3. A waypoint/target arrow now, or after landing sites exist (needs a destination)?

---

## Recorded but not proposed now

- **C. Walker speed ladder + throttle.** SC: per-stance speed *sets* (`actor/stanceinfo/speeds/stand.xml`: slow/mid/fast walk, slow/fast run, sprint, ADS, ...) and an analog throttle (`playerspeedthrottle/`). Simplified: one ordered speed ladder with a shift key. Fits the decided "tap W = slow step, hold = full" and crouch/sprint later.
- **E. Ship power on/off (one resource).** SC: a typed resource network with producer/consumer nodes and per-state deltas (`itemresourcenetwork/itemresourcenetworkglobal.xml`). Simplified: one `powered` flag + spin-up time gating thrusters, LAG (Feature A) and lights. Gives the decided "ship on/off / more ship systems" its first form.
- **F. Zero-G suit movement.** SC: zero-G as a stance with its own speeds/dimensions plus a small Attach/Detach/Launch graph (`zerogtraversalgraph/`, `actorzerogtraversalparams`). Simplified: the already-decided suit keys in weightlessness, modelled as a stance. Closes a decided feature.
- **Tier 3 — the core loop.** Landing sites → interaction verb → pick up/deliver cargo → shared wallet, per `CORE-LOOP.md`. This is the real MVP and the next milestone; it depends on the content schema and is bigger than a single add.

## Suggested batch

A + B + D together: LAG makes the interior a place, the boost capacitor makes flying better, the HUD makes both visible — and the HUD needs B anyway. C, E and F slot in after; Tier 3 is the next milestone.

## References

- Mapping and file list: `docs/research/star-citizen-vs-exo1-mapping.md`.
- Data sources and hygiene: `docs/research/star-citizen-datamining.md`.
- Our current code: `crates/flight_core/src/lib.rs:117` (`ShipController`), `:256` (boost), `crates/exo_app/src/walker.rs:191-198` (cabin gravity), `crates/exo_app/src/view.rs:167` (HUD).
