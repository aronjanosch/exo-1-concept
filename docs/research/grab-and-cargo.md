# Grab and cargo: how other games pick up, carry and throw things

Research note, opened 2026-10-09, for milestone C "Things you can touch" (code repo epic #35, question #36). Goal: a short grilling session with the initiator. Status: **decided 2026-10-09**, see `DECISIONS.md` "Grab and cargo (milestone C)". Values are assumptions for a playtest.

Settled already (`DECISIONS.md`): cargo is physical (crates in a few standard sizes, carried by hand, limit by volume, no counter); the grab tool is the first tool; planets are static; client authority over plain UDP. We learn from the games below and build our own implementation. Nothing is copied: no code, data, names or texts.

## Per game

**Half-Life 2, gravity gun and +use pickup.** The held object is driven by a controller towards a hold point in front of the eyes (position error plus damping), with a cap on linear and angular speed. Heavy objects get a lower cap, so they lag. When the object touches something heavy, its turn rate drops sharply, so a light box does not shove the world around. It drops when the error stays too big for about a second, or when you stand on it. Bare-hand pickup slows you and blocks jumping. Feels good: weight you can read from the lag, objects that bump into walls instead of clipping. Bad: stiff springs make light things too strong, soft springs feel dead.
[Source SDK `weapon_physcannon.cpp`](https://github.com/ValveSoftware/source-sdk-2013), [Godot thread on HL2-style holding](https://forum.godotengine.org/t/hl2-amnesia-style-physics-object-manipulation/133886)

**Star Citizen, tractor beam and boxes.** Boxes come in standard sizes. Small items are carried in one or two hands (two-handed items turn to a fixed carry pose), bigger ones need a tractor beam (hand tool capped by mass, later also by box size) or ship beams. In a cargo hold boxes snap to a locking grid; only boxes fully on the grid are held. Loose boxes are a known source of chaos and bugs. Mass and its placement were meant to change ship handling.
[Design Notes: Cargo Interaction](https://starcitizen.tools/Comm-Link:Design_Notes_-_Cargo_Interaction), [Alpha 3.24 notes (tractor limits)](https://starcitizen.tools/Update%3AStar_Citizen_Alpha_3.24.0)

**Space Engineers.** Loose items ("floating objects") inside a moving grid are a long-standing bug source: items stuck in a grid make the ship shake or spin. Lesson: free bodies inside a moving body need care, or they fight it.
[Keen support: subgrids and floating objects](https://support.keenswh.com/spaceengineers/pc/topic/23555-subgrids-freak-out-and-start-floating)

**Lethal Company.** Each item has a weight; the sum slows you and drains stamina. Two-handed items block the other slots, so you carry one big thing and nothing else. The trade-off (big loot vs. running from danger) is the whole tension. Simple, readable, no physics needed for the hold itself.
[Scrap and loot guide](https://dungeonpath.com/posts/lethal-company/scrap-and-loot-guide/)

**Deep Rock Galactic.** Heavy objectives (crystals, eggs) are carried in both hands: about 25 % slower, no sprint, no weapon. You can throw them to a teammate (from play; the link covers speed only). Carrying is a short, clear state with one job, and teammates cover you.
[Steam discussion on carry speed](https://steamcommunity.com/app/548430/discussions/1/3758850346180729337)

**Hydroneer.** No inventory: one item in the hands, drop it to do anything else. To move many things you put them in a bucket or cart and carry that. The tactile loop is the fun.
[Beginner guide](https://gamepretty.com/hydroneer-starting-guide-for-beginners-how-to-play-placing-items/)

**R.E.P.O.** Everything valuable is moved with a physics beam. Heavy things need two or three players on the beam at once (or a strength upgrade). Impacts lower the value, so careful carrying is the game and dropping is the comedy. Co-op physics over Photon, with ownership of grabbed objects.
[Photon blog on R.E.P.O.](https://blog.photonengine.com/r-e-p-o-multiplayer-success-powered-by-photon/), [Strength upgrade](https://sportskeeda.com/esports/the-strength-upgrade-repo-explained)

**Gang Beasts / Human Fall Flat.** Each hand grabs on its own button; two hands give leverage to lift overhead. Friction and flailing are the joke. Too floppy for a game about flying a ship, but the "two hands, two buttons" idea and the goofy failure are worth noting.
[Gang Beasts review](https://waytoomany.games/2017/12/20/review-gang-beasts-ps4)

**Outer Wilds.** Small items are held in front of you, snapped, not simulated; the ship interior is calm. Shows that "snap to hand" is fine when the item is not the point. (From play; no write-up found.)

**Networked physics (Gaffer On Games, for Oculus).** Two ideas per object: *authority* (who simulates it now; goes to whoever touched it last, spreads to objects it hits, returns to default when it comes to rest) and *ownership* (held in hand; nobody else can take it until released). Ownership beats authority. Sequence numbers per object, the host arbitrates. Corrections were rare even with lag.
[Networked Physics in VR](https://gafferongames.com/post/networked_physics_in_virtual_reality/)

## Patterns worth learning

- **Hold point plus capped regulator.** The object is pulled to a point in front of the eyes with a spring and damper, with a maximum force and a maximum speed. Mass lowers the caps, so weight shows as lag. Ours: `grab_core` computes the force from (hold point, body state, mass), test-first, f64.
- **Soften on contact.** When the held object touches something, cut its turn rate and force. Stops light boxes from launching the ship's furniture.
- **Break on sustained error.** Drop when the object lags too far for too long (about 1 s), not on the first snag. Also drop when the holder stands on it.
- **Carry is a state with costs.** Slower walk, no sprint or jump, other hand busy. Clear and readable beats a weight formula.
- **One big thing at a time.** Two-handed means nothing else. Containers (bucket, cart, crate) are how you move many.
- **Snap targets in the ship.** A grid or rack that locks a crate; loose crates are physics, locked crates are part of the ship.
- **Authority and ownership as state, not events.** Owner on grab; last toucher simulates after a throw; the default owner takes it back at rest.
- **Shared carry for big objects.** Several holders each add force; the object goes where the sum points. Weight needs friends.

## Risks for us

- **Moving cabin, one-tick lag.** Avian moves child colliders to the body pose only at the start of the next step (`walker.rs:138`, `cabin_frame`). At 400 m/s the cabin floor sits 6.7 m behind the body between steps. A free crate that collides with that floor in world space sees a jumping floor. The walker avoids this by living in ship-local coordinates (`walker_core::Frame`, `change_frame`).
- **Warp teleports the ship.** On rails the ship gets its pose from the drive each tick with zero velocity (`warp.rs`, `warp_drive`), up to 1 000 000 m/s, about 16.7 km per 60 Hz tick. A dynamic crate in world space stays behind instantly. Crates in a cabin must live in the ship's frame (local simulation, or locked to the ship) at least during warp. The issue's test crate "through take-off, flight and warp" is exactly this.
- **Crate mass on the ship.** A 2000 kg ship (`ship.rs`) with explicit mass and inertia. Crates pushing on the hull from inside push the ship; loose crates sliding at high G could shake it (the Space Engineers bug class). Locked crates should either add mass on purpose or not at all.
- **Frame hand-over.** A crate leaving the cabin (ramp, thrown out) needs the same hand-over as the walker: keep world position, add the ship's velocity. Entering needs the reverse.
- **Network.** A held crate is owned by the holder, so it follows the holder's hand without lag on their screen and jitters on others'. Two grabbers at once (#36) need one rule. A crate in a remote player's ship lives in that ship's frame, so the owner question and the frame question meet.
- **Planet swap.** `swap_planet` replaces the planet during warp. Loose crates left on a planet must be despawned or saved (the epic's object budget: persistence cap, timeout).
- **Perf of many bodies.** Each loose body costs in the solver and in snapshots. Sleeping bodies (the ship uses `SleepingDisabled`, crates should not), a cap per category, and locked crates as ship children (no own body) keep it cheap.

## Where grab hooks in today

- `walker_core` (`lib.rs`): add a `grab_core` crate next to it. Inputs: hold point, look, frame; body position, velocity, mass; outputs: force and torque on the object, reaction on the holder, a break flag.
- `exo_app/src/walker.rs`, `walker_step`: F is hard-wired to the seat (`Tap::Seat`, distance to `SEAT_POS` < 1.8 m). The interaction system replaces this: one verb, nearest target in a cone, a prompt.
- `exo_app/src/controls.rs`: `Tap::Seat` and the `Controls` resource; a scenario drives grab through `Controls` like the rest.
- `exo_app/src/ship.rs`, `add_hull`: the place for a crate rack or lock grid as cabin children.

## Decisions for the initiator

Ordered by importance. Each has the agent's recommendation.

1. **How does a crate live in a flying cabin?**
   a) Free physics body in world space, always. b) Free body in the ship's frame (simulated ship-local, like the walker), locked to the ship during warp. c) Loose only while carried; set down in a cabin it snaps to a rack and becomes part of the ship.
   *Recommendation: b plus racks from c later; a breaks at 400 m/s and in warp.*

2. **Feel of the hold.**
   a) Physical: spring to a hold point with force cap, heavy things lag and bump. b) Snap to the hand, no lag. c) Physical for big crates, snap for small items.
   *Recommendation: a; the playtest question is "does moving cargo feel good", and lag is where weight lives.*

3. **Hands and weight on the walker.**
   a) Small crate one hand, big crate two hands; two hands means slower, no sprint, no jump. b) Everything two-handed, one fixed slow-down. c) Weight scales speed continuously.
   *Recommendation: a with two or three fixed steps; readable like Deep Rock, no formula to learn.*

4. **How many crate sizes, and does the biggest need two players?**
   a) Two sizes (small, big). b) Three sizes, the largest needs two holders. c) Three sizes, the largest needs a tool (tractor) later.
   *Recommendation: b; shared carry is the cheapest co-op moment we can get, and it is goofy.*

5. **Who owns a crate on the network (#36)?**
   a) The holder owns it; last toucher simulates it after release until it rests, then the ship's owner (in a cabin) or the host (outside). b) The host owns all loose crates. c) A crate in a ship always follows the ship's owner.
   *Recommendation: a, with "first grabber owns, the second holder adds force through the owner" for shared carry.*

6. **Crates in a cabin during flight.**
   a) They slide and tumble with the ship's acceleration (cabin gravity on, inertia felt). b) Mag-locked on a floor grid once set down; only unlocked ones slide. c) Never slide.
   *Recommendation: b; sliding on purpose is funny, but only for crates someone forgot to lock.*

7. **Throw.**
   a) Release with the hand's velocity only. b) Plus a throw button with a fixed impulse (less for heavy). c) No throw.
   *Recommendation: b; passing a crate to a friend across the cabin is part of the fun.*

8. **Reach: hands or tool?**
   a) Hands only, short reach (about 2 m). b) A grab tool with longer reach (about 5-8 m) and falloff, hands for the rest. c) Tool only.
   *Recommendation: b as the epic says (force with distance falloff, cone); start with the tool at short reach and stretch it in the playtest.*
