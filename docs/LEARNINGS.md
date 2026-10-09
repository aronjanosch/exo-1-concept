# Learnings

Loose list of what we learned while working, for humans and agents. Source material for future skills. One entry per point: what, why, how to apply. Newest at the bottom of each section. Machine-specific setup lives in each person's `WORKSPACE.md`, not here.

## Working with the initiator

- **The initiator decides game design.** Record only what they explicitly decided, in their words. Brainstorming, my own conclusions and technical findings are not decisions. Why: AI choices rest on assumptions, not on play experience. How: before writing to DECISIONS/CORE-LOOP/VISION, quote the exact decision back.
- **AI guesses turn into "decisions" by copying.** Example: "start radius 3 km" came from an AI research draft, moved into DECISIONS as "decided", then into the spike brief; nobody had chosen it. How: tag every number with its origin (decided, assumption, measured) and keep the tag when copying.
- **Design-relevant defaults in prototypes are assumptions.** Speeds, gravity, flight assists, look: label them as such in the spike README, don't present them as settled.
- **Everything is early and experimental.** A change the initiator tries is an experiment, not a rule. Do not write it into DECISIONS or other docs as settled; at most note it as an idea or direction to try.
- **Be brief.** Say a fact once, no repeated lists, no long recaps.
- **Spikes: progress over premature optimisation.** If it runs well on the initiator's machine, move on; optimise later. Stay on the essentials: when one detail eats time, note it as open and move on.

## Local environment and tests

- **Never steal focus.** Check `WORKSPACE.md` before a windowed run, prefer headless for checks, and never take frame times from a window on a hidden workspace (it is throttled to about 8 FPS).
- **Scripted runs must not grab the mouse.** A grab pulls pointer and focus to the game window even when it was started unfocused on another workspace.
- **Exclude screenshot frames from frame-time stats.** Readback plus PNG save costs 50-140 ms.
- **Bots must fly like players.** A blind bot at boost speed crashed into a hill and broke the run; give test bots simple controllers (altitude hold).
- **Two agent sessions in one checkout collide.** A parallel session switched the shared working tree to its branch mid-edit. How: each session works in its own `git worktree`, and result files and screenshots get their own names.
- **Check the branch before every commit.** Another session can switch a worktree without notice, and a commit meant for one branch landed on the other. How: run `git branch --show-current` and `git worktree list` right before committing, and say which branch the commit went to.
- **Gitignored local files are missing in a new worktree.** `WORKSPACE.md` does not come along. How: symlink it into the new worktree; it stays ignored. Delete the original and the link breaks.
- **Parallel agents spoil timings.** Benchmarks run while subagents compile give noisy numbers; rerun when the machine is quiet.
- **Frame times under vsync measure the display, not the game.** Every phase of the warp read 6.94 ms on a 144 Hz screen; without vsync the same run read 1.4 ms. How: measure with `--no-vsync` (`PresentMode::AutoNoVsync`) and leave screenshot frames out.
- **One cargo target dir per session or lane (initiator, 2026-10-09); never two commits in one.** Registry crates are fingerprinted without the workspace path, so a shared `CARGO_TARGET_DIR` makes a fresh worktree build in 8.7 s instead of 6 min (only the own crates rebuild; sccache gave nothing). But own crates are fingerprinted without the path too, and freshness is by mtime: a worktree or lane on another commit overwrote `warp_core` and `flight_core` or `planet_core` in the shared dir, which gave type errors and 27 compile errors on a branch that was fine, a binary from the other tree, and one lane running another lane's binary. How: every session or lane builds in its own dir (btrfs reflink copy, instant, e.g. `~/.cache/exo-1-target-rivers`; `WORKSPACE.md` names the shared preset); a comparison commit gets its own dir, or `touch` the sources and check the binary afterwards; no two cargo runs at once in one dir.

## Git and spike states

- **Frozen spike states get annotated tags `spike/<n>-<name>`** (e.g. `spike/1-planet`), releases will use `v*`, so the two never mix. Why: old states must stay checkable for videos after branches are deleted; a tag also survives squash and rebase. Tag names differ from branch names (`spike/1-planet` vs `spike/planet`) because equal names make `git checkout` ambiguous.
- **A finished milestone gets an annotated tag `milestone/<letter>-<name>` on its merge commit into `main`** (first: `milestone/p-planets-with-character` on `7e92b3d`, 2026-10-09; the earlier sprint state is `sprint/1-4-foundation`). Why: the end of a milestone is a state to come back to (playtest, video) after its branch is gone. How: merge, tag the merge commit, push the tag, then delete the branch.
- **Name spike branches without the number so the freeze tag can carry it.** Branch `spike/4-network` blocked the usual tag name `spike/4-network`; the freeze became `spike/4-client-authority`. Pattern: branch `spike/<name>`, tag `spike/<n>-<name>`.
- **Merge spikes normally, never squash,** or the single commits are gone. Spike code stays off `main` (throwaway, no approved proposal); an archive branch plus tags keep it reachable. Only findings go to the docs.
- **A branch that exists only locally is not safe.** `spike/combined` with all video commits was local only until pushed. Tags protect against deleting a branch, not against losing the disk.
- **`spike/combined` is the base for new spikes** (since 2026-10-07, at spike 7). Each spike branches from it; after review it is fast-forwarded to the finished spike and the spike's tag goes on that commit. Why: one name to branch from instead of "whichever spike was last", and the chain stays linear.
- **Merging spikes: git conflicts are the small part.** Spike 3 + 5 had 2 text conflicts but 4 real interactions (double shift of a reparented child, origin assumptions, a hidden ramp bug, a changed default). How: rerun every test of both spikes on the merge and look for assumptions the other spike broke.

- **Parallel sessions need one base and one way in (2026-10-09 retro).** Branches cut from in-between states (`fix/x` on `feat/y` on an unmerged `night/extras`) while `main` lagged made every playtest merge conflict, and a file split running beside feature work collided with all of them. How: lanes start from `origin/main`, finish into one round branch, one PR per round; splits and moves run alone. Rules: code repo skill `exo-orchestrate`.
- **Killing a background shell does not kill its cargo.** `pkill` on the shell left the test run going, and the next run wrote into the same log and target dir: a mixed log with failures from neither. How: stop cargo itself, check `pgrep -af cargo` is clear before the rerun.
- **Reports belong on the issue, not in the chat.** Copy-pasting session reports between terminals made the initiator the message bus. How: each lane ends with one comment on its issue; the coordinator reads it with `gh`.

## Planet and terrain

- Derivative flat normals (`cross(dFdx, dFdy)`) are zero on sub-pixel triangles; `normalize()` gives NaN and it survives `mix(..., 0)`. Guard the length.
- Skirts must use the smooth normal, or they show as dark lines. Per-chunk origins cause hairline cracks between chunks; skirts fill them.
- Tangent-frame collision patches with curvature baked into the heights avoid the flat-plane error; let them overlap.
- Distance tests for the collision ring must ignore terrain amplitude (compare on the base sphere), or the ring grows about 5x.
- A reverse-Z float depth buffer shows no z-fighting to 40 km; a classic 24-bit buffer z-fights from about 500 m at cm gaps.
- On a small planet "straight" flight leaves the planet in seconds. Horizon follow: add angular rate `up x v / r`. A levelling force fights intended climbs.
- Ring/LOD bursts on teleport cause 60-140 ms frames; continuous movement does not.
- **Quadtree bounds on a cube-sphere: use the farthest corner.** `0.75 x edge` is too small for large cells on the curved face and pruned whole quadrants.
- **Vertex-centred cube-face images make seams impossible by construction.** Why: both faces interpolate the same samples on a shared edge. How: any baked per-face field on a cube sphere (heights, biomes, moisture) gets the edge samples (N+1 per side), not texel centres.
- **A one-millimetre agreement test at 5 km is limited by 32-bit numbers, not by the generator.** Why: a float32 position at 5000 m has 0.5 mm resolution and a 32-bit direction 0.3 mm of lateral position; times slope that is 1-2 mm. How: do geometry in f64, make chunk centres f32-exact, give tests a double-precision query (`height_at_xyz`).
- **Size biome wavelengths by walking distance, not by planet size.** Why: five minutes at 1.8 m/s is 540 m; the research wavelength (2.5 km) gave one biome change per 926 m. How: choose region frequencies so the median stretch is about half the intended walk (0.0008 to 0.001 gave 271 m).
- **Headless settle loops must not count frames under a fixed frame rate.** Why: frames run faster than the worker threads. How: wait until pending jobs are empty, with a wall-clock limit.
- **Aim the sea-level percentile at the height the player sees.** Why: the rule ran on the macro field (70 % land), but the crust bands have a positive mean, so the full height gave 79.7 % land. How: compute the percentile over the full height function, or report both.
- **Terrain upload, not generation, was the first limit.** At depth 8 and 350 m/s the 4-uploads-per-frame cap (240/s) saturated before any worker did.
- **Measure the demand before choosing the language.** Test workload (7 noise calls per vertex, scatter, 35x35 grid): the former scripting language 3.3 ms per chunk, Rust 0.65 ms, but the LOD asked for at most 114 chunks/s while 4 script threads gave about 1000/s. A 5-7x language gap did not matter at 9x headroom. Numbers and caveats: `SPIKE-6-REPORT.md`.
- The `fastnoise-lite` crate 1.1.1 ships no licence file; upstream Auburn/FastNoiseLite is MIT.

- **Planet look, milestone P (2026-10-08):**
  - Earth's scattering per metre over our 1.2 km atmosphere gives no visible sky. Size it to Earth's vertical optical depth instead (Rayleigh blue about 0.26, so about 0.64 per km here); four times that already made the planet pastel from orbit.
  - A UV-sphere sea at 5 km radius sags up to 1.5 m below sea level between its vertices, so the coastline misses the terrain. Build the sea per terrain chunk at the same LOD. A dipped skirt on transparent water shows as a dark line along every chunk edge.
  - The largest nearest-neighbour gap between sites grows when sites are spread evenly (best-candidate sampling), so it is the wrong number for "a short hike finds the next". Measure coverage: distance from random land points to the nearest site, worst and median.
  - Rock-by-slope thresholds from other games (25-40 degrees) showed no rock at all: our noise terrain has almost no slopes that steep. Look at the slope distribution before choosing thresholds.
  - The 1 mm collision test mixed the generator's error (0.3 mm) with the scene's f32-rounded frame origin (up to 1.3 mm on new terrain). Assert the two separately.
  - Frame times on a shared machine: parallel builds or a Blender run raised ground frame times 1.5-2x. Check the load average and other game windows before trusting a run.
  - WGSL: `patch` is a reserved word; the shader fails at pipeline build with only a log line, and the material silently draws nothing different.

- **Rivers and lakes from drainage (#72, 2026-10-09):**
  - Size the river threshold by the landmass, not by Earth. Hearth is an archipelago with many pits below sea level; the largest rain-weighted catchment of any land vertex is 3.3 km². A 1.5 km² threshold gave 1.2 km of river, 0.25 km² gave about 37 km. How: read `largest_catchment_km2` in the bake stats before choosing.
  - Filling every sink turns a noise terrain into a lake district: Hearth first got 199 lakes, Cinder (almost no sea) 195 lakes on 11 % of its surface. A dry planet needs an evaporation budget (lake area at most inflow / evaporation) and lakes that never spill; the water around them must then route into them, not to their spill point (a second flood with them as outlets).
  - The drainage cost is the algorithm, not the optimisation level: 1.58 M macro vertices took 1.1 s in the dev profile and 0.93 s in release, about 0.3 s per priority flood. Packing heap keys into a u64, a FIFO for pit cells (Barnes 2014), a fast neighbour path and the steepest descent inside the flood loop took it from 5.3 s to about 1 s.
  - Tests at macro vertices miss what bilinear blending does between them. A river one vertex wide goes dry at every diagonal step (the cell's other two corners keep the bank height), and a bank vertex with a water level above its own ground ends a water sheet in the air. A read-only review subagent found both.
  - Every height change moves the sites (their candidates are filtered by height and slope). The ruin moved to a coast and the `site-walk` scenario started in the sea. How: scenarios pick their path from the data (here the first dry, walkable heading), not a fixed direction.
  - A leak check that compares world counts across swaps to alternating planets breaks once the planets differ (lake meshes). Compare with the first swap to the same planet.
  - Long rivers need one sea, not a sea level. Hearth's area below the sea level is 1055 separate pockets (the largest 8 km²); every pocket ended a river, so the longest river was 0.82 km whatever the lake values. Counting only seas from 0.5 km² as sea gave 7.5 km. How: count the connected areas below the sea level before tuning river values.
  - Crossing a sink: cutting the sill down to the sink's floor made 40-77 m gorges; filling the sink with sediment to its spill point kept the deepest cut at 8 m. A fill right at the sea level freckles with sea between the vertices; fill at least 2 m above it.

## Client authority and network (spikes 4 and 10)

- **Shared snapshots must outlive local origin shifts.** Keep planet id plus planet-relative pose in the buffer; convert to the receiver's world only when displaying. Subtract double centres and origins before creating a float32 vector. Float64 encoding preserves the source precision; it does not recover float32 precision already lost.
- **Separate injected loss from the library's own throttle.** Eight localhost processes triggered adaptive unreliable throttling and extra drops. Disable the throttle in the fixture and report library bytes separately from snapshot payload. This isolates the experiment, not a WAN congestion-control recommendation.
- **Label buffer latency as total or extra.** A 150 ms buffer on top of 150 ms simulated one-way delay means peers show roughly 300 ms-old states.
- **Client authority does not assign contact authority.** Two independent physics spaces with delayed kinematic peer copies hit 13 ticks apart and disagreed by up to 3 m. Mutual ship impulses and docking need a human ownership decision; none is invented.
- **A rejoining owner restarts its snapshot sequence.** A newer shared timestamp plus a lower sequence must reset the display history and reject queued packets from the old lifetime, or the old history rejects the new packets as duplicates or interpolates a teleport.
- **`std::net::UdpSocket` is enough for client authority.** Hello retried until accepted, ping/pong, host relay of the snapshot bytes, 2 s expiry, takeover of a silent slot: about 350 lines, no dependency. Frameworks (replicon, lightyear) are built for server authority or prediction, which this design does not use.
- **Put the protocol in a crate without engine types.** `net_core` (snapshot, buffer, link, clock, wire, replay) runs the 96-case matrix in 4 s of `cargo test`; the live processes only add sockets.
- **Record a real scripted run as the replay path.** The spike 9 `full` scenario (up to 400 m/s, braking, rolling) is harder than spike 4's 25 s hop, and its P95 was still 0.17 mm at 30 Hz/150 ms. Look at per-phase numbers: the brake from 400 m/s is the worst phase (8 mm).
- **Another RNG gives other loss patterns.** "0 holds" in spike 4 does not reproduce exactly; assert a small bound, not zero.
- **A hold is expensive at speed; display-only extrapolation fixes it.** Holding 15 ms at 400 m/s is 6 m. Carrying the last velocity on for 50-100 ms (display only, not collision) cut the worst case from metres to 36-58 mm in the replay. Needs the initiator's OK because spike 4 said no extrapolation.
- **A kinematic Avian proxy needs Position and Velocity.** Avian integrates kinematic bodies; set both each tick or contacts see a standing wall.
- **Clock sync on a tick loop is only good to a few ms; a receive thread fixes it.** Ping and pong waited for the next 60 Hz tick on both ends: error 2-7 ms against the true offset. A thread that stamps arrival time and lets the host answer pings itself: 0.013 ms (8 players). Compare the estimate with the real offset, do not trust the RTT. Sockets cloned with `try_clone` share the non-blocking flag, so the thread blocks and the game thread sends.
- **Whole-metre origin shifts keep far f32 positions exact.** At 200 km the f32 grid is 2^-6 m, which divides 1 m: shifting by whole metres changes the rounding error by zero. The remaining render error at 200 km is one f32 step, 7.8 mm.
- **Count holds only when the stream resumes.** The last 0.15 s before an owner leaves would otherwise look like an underrun.
- **`proton run` hides stdout.** Write result files with relative paths; `Z:` paths made a run exit 1 without a message.

## Bevy and Avian (spike 9)

- **Avian moves child colliders one physics step late.** `update_child_collider_position` runs at the start of the next step, so between steps spatial queries see a ship's cabin colliders one tick behind the body (6.7 m at 400 m/s); the walker fell through the floor at speed. How: query in the frame derived from a child collider's own `Position`/`Rotation` and its `ColliderTransform`; keep walker coordinates ship-relative.
- **With f64 physics the origin shift is a render-only matter.** Avian `f64` keeps bodies in world doubles; switch off its `Position` ↔ `Transform` sync and write `Transform = pose - render_origin`. Nothing physical moves on a shift (0 mm, 0 m/s by construction, ≤ 0.010 ms per shift). `big_space` lagged one Bevy release behind (0.12 = Bevy 0.18), so check plugin versions against Bevy before choosing.
- **A self-written move-and-slide needs tests on a heightfield, not only on planes.** Two bugs passed the plane tests: a floor-snap check against exactly 0 (curvature leaves 1e-17) and sliding upwards along steep slopes while airborne. How: run the spike 8 T5 walk and log the steepest ground under the walker.
- **Depenetration skin must be smaller than the controller's own gap.** Equal values (1 cm) pushed the walker along the slope normal every tick: 35 mm drift standing. 2 mm against a 10 mm gap: 0.0000 mm.
- **`dynamic_linking` turns a 6 s rebuild into 1 s** for a Bevy game binary on this machine; the core crates' own test loop is 0.24 s. Use it for agent loops, never for release builds.
- **An invisible Bevy window still renders**, so screenshots need neither a visible window nor focus (`Window { visible: false }`).
- **Edition 2024 reserves `gen`**; a field named `gen` from older code needs renaming.

## Agent tooling for Rust and Bevy (spike 9b)

- **A mise shim cannot be `build.rustc-wrapper`.** Cargo runs rustc in directories without the mise config, and the shim fails with "No version is set for shim". Put the real tool directory on `PATH` (`mise activate`).
- **rust-analyzer for the Claude Code plugin:** pacman's rustup has no `rust-analyzer` proxy, so install it with `mise use -g rust-analyzer@latest`. `cargo.targetDir` is a workspace-scope key and works from the user config `~/.config/rust-analyzer/rust-analyzer.toml` (`[cargo] targetDir = true`): flycheck and build scripts go to `target/rust-analyzer/` and never lock the agents' target dir. rust-analyzer looks for `Cargo.toml` only in the session root and one level below; the Cargo workspace sits at the repo root, so start the session there.
- **mold saves 0.18 s on a 1.1 s dynamic-link rebuild**, not worth a setting. `-fuse-ld=mold` with an absolute path fails with gcc 16; use the name with mold on `PATH`.
- **BRP bought nothing on a bug the scenario runner reproduces in 15 s.** The session with 52 BRP tools was slower (345 s against 107 s) and did not call one. Keep it for bugs that only show in a running window. It needs the windowed app, `Reflect` on every type you want to see, and it is red-class (HTTP server on localhost).
- **Third-party skills still contain wrong API examples**: 8 of about 30 in `bevy-ecs-systems`, 13 of about 60 in `bevy-porting`; two skills were clean (28 of 28). Check examples against the registry source before vendoring.
- **bevy_lint trails Bevy by a release** (latest tested: 0.18 while Bevy 0.19 is out) and needs a pinned nightly with `rustc-dev`.
- **Nested `claude -p` sessions are a workable blind-test harness**: `--setting-sources project --strict-mcp-config`, tools allowed by pattern, prompt on stdin (a variadic `--allowedTools` swallows a positional prompt), JSON output gives turns, duration and cost.
- **`pkill -f <pattern>` can kill the agent's own shell** when the pattern appears in the command line; use `pkill -x <name>`.

## Warp between planets (spike 11)

- **At warp speed the physics engine must not see a velocity.** Avian stretches every body's AABB by its velocity for a step (infinite speculative margin by default), so 1e6 m/s means a 17 km box that pairs with everything in it. How: put the ship on the path every tick with zero velocity, drop `SweptCcd` for the flight, give the velocity back at the exit; sweep the obstruction check along the curve, since a tick is three planet radii.
- **A speed law that depends on the distance left lands exactly and needs no timetable.** `v = min(v + a·dt, top, brake_speed(remaining))` handles every distance (short trips peak lower) and the exit point is hit exactly; the state machine around it stays small. Effects keyed to speed (not phase or time) work for every distance.
- **A tangent that depends on "into the planet" is unstable near radial.** The projection onto the tangent plane shrinks to nothing and its side is noise, so the path flipped between ticks. How: blend into a fixed tangent on the same side as the projection; turn the exit point 40° away from the line so the arrival is along the sphere.
- **Snapshot sanity limits are design values.** `decode` threw away everything beyond 1,000 km and 1,000 km/s; a warping ship would have vanished for everybody else. Check every "absurd value" limit when a game gets faster or bigger. And planet-relative snapshots need a common frame in the buffer: across a change of the sender's planet the buffer holds, 13.8 km off at that moment.
- **Planet registry plus one "active planet" resource is a cheap way to the second planet.** The sim keeps one `PlanetRes`; entering a planet's frame zone swaps it (0.1 ms), the next planet is generated on a pool thread from the start of the ramp-up (0.6–0.8 s against 25 s of flight), the old one is dropped. About 32 MB per planet bake.
- **The roost container cannot build Bevy with its default features** (no `pkg-config`, alsa, udev, wayland headers), cannot open a window (no GPU, no `libXcursor`), so only headless runs here. Trimmed Bevy features (`default-features = false`, `x11`, `bevy_winit`, 3d, ui) build in 19 min cold; keep them out of commits (`git update-index --assume-unchanged Cargo.toml Cargo.lock`, regenerate the lockfile with the default features before committing). Anything visual needs a run on a machine with a GPU.
- **`c.v` in a scenario context outlives its step.** Keys left by one step (a tap flag) silently skip the same code in the next step; clear your keys at `t == 0`.
- **Anything placed around the camera must use this frame's render origin.** The origin moves in `PostUpdate`, after the systems that set streaks, impostors and the course ring; at 1,000 km/s that is 7 km a frame, and the streaks simply vanished. How: give such entities a world pose (`WorldPose`) that is converted after the shift, never a `Transform` computed from the origin in `Update`.
- **Code that never ran in a window hides look bugs no check finds.** Round 1's cabin "window" was an opaque plate in front of a solid wall, and the tunnel streaks sat inside the closed cabin; both compiled and passed every check. How: the first windowed run is part of the work, with a screenshot of every phase and a look at each.
- **Scripted actions must not depend on a phase being long enough.** A 2.2 s walk fit a 2.2 s cruise by luck; when it did not, the key stayed held and the walker pressed into the wall (422 depenetrations). How: let a started action finish on its own clock and release every key at the end of the step.
- **A drop point on a path that was checked at the start is free of planets**, so an emergency exit only has to check other ships there; moving the point on along the path until it is clear keeps it on the checked curve.

## Crates as Avian bodies (spike 12, 2026-10-09)

- **Keep the game's own state in one struct and copy the Avian pose into it after each step.** Why: grab, interaction, rendering and budget kept reading `CrateBody` unchanged; only the step moved to Avian. How: apply forces before the step, copy pose and velocity back after it, and pass velocity the game set itself (a throw) into `LinearVelocity` when it differs.
- **Collision filters decide more than geometry.** The ramp collider was walker-only (filter `NONE`), so a crate body fell through it at once. Check both sides' masks before blaming the solver.
- **Wake bodies only over a collision patch.** The patch came 0.17 s after the walker; waking earlier, crates fell 0.14 m before it caught them. A slower stream would lose them.
- **Do not let a body rest on a moving ship.** With the ramp hitting crates, a crate rode along at takeoff as a planet-frame body in contact with the ship. Anything on the ship, ramp included, stays in the ship's own model.
- **Box stacks on a heightfield are not free.** Three stacked crates bounced and fell after about 1 s with Avian defaults; cause not found.

## Flight models (spike 13, 2026-10-09)

- **Limit each mode's request, then blend the modes.** Blending coupled and decoupled requests first and clamping the sum kept the coupled braking saturated almost to the end of the 4 s blend (41 % of the speed left instead of 66 %). Why: a large error stays above the limit until its weight is tiny. How: clamp each mode's thrust to what the thrusters give, then blend.
- **A ground band needs the stopping distance, in the ship's attitude.** 150 m/s down with 15 m/s² to brake needs about 740 m; an 80 m precision band started far too late. How: compare the band with the clearance less sink² / (2 × braking thrust), and take the braking thrust along the planet's up in ship space (rolled on its side it is the side thrust).
- **A push along the hull's own axis slides a landed ship.** Ctrl held on a slope pushed along the tilted ship's down; its share along the ground slid the ship 63 m. How: on the ground with no sideways input, settle along the planet's up and strip sideways speed.
- **A turn cap from the rate's sign alone is wrong in reverse.** Which tolerance a turn loads depends on rate × velocity (nose up while flying backwards pulls the velocity down), plus what the thrusters already hold against gravity. How: build the needed acceleration as a vector and scale the rates to keep it inside the limit box.
- **Read a touchdown speed before the contact flag.** The flag comes a step after the contact, and Avian's speculative contacts have cut the approach by then (1.56 m/s shown for 2.0 m/s). How: keep the last 0.25 s of sink and take the largest.
- **One speed cap for every direction changes the feel more than any limit.** With a single cruise cap (stick in a ball), strafing and climbing reach four to five times the classic model's per-axis speeds; only the lower acceleration makes them slower to reach.

## Figures in Blender by script (walker blockouts, 2026-10-08)

- **Primitives joined into one mesh still read as parts.** For people, build one seamless body: a joint skeleton with radii through Blender's skin modifier plus subdivision gives clean quads and a body that can be rigged later.
- **Colour zones need cut edges.** Colouring whole faces by nearest bone gives zigzag hems; bisect the faces of that bone at the hem plane first, and pick the nearest bone by distance minus its radius, or torso faces near the arm turn into sleeve.
- **Painting by surface position works without an artist.** Unwrap (scale the head up before unwrapping to give the face more texels), rasterise every UV triangle to learn each texel's 3D position, then paint with masks over those positions; bleed the colours into empty texels. Write sRGB, the material colours are linear.
- **Hair from loose locks shows every seam.** A short cap cut off the scalp plus chunky locks, then a voxel remesh, a smooth and a decimate, gives one surface that reads as a sculpted haircut. Only for parts without a texture: the remesh drops the UVs.
- **A broad text replace can hit a second, identical block.** Replacing `bm.to_mesh; bm.free()` also changed the eyelid helper, which then overwrote the body's ray-cast data; the symptom (the second eye missing) was far from the cause. Check every match of a replace.
- **`intersect_ray_tri(..., clip=False)` hits the infinite plane**, not the triangle; face parts landed on the crown. Use `clip=True`.
- **Two-point NURBS tubes vanish.** A NURBS spline of order 2 with two points makes no geometry under a bevel; use a POLY spline for straight tubes. A bevel ring has 4 + 2 × resolution sides; at 8 sides the 45° steps cross a 40° sharp-edge angle and the tube looks faceted.

## Character model polish (2026-10-08)

- **Simplify before cutting clothing colour boundaries.** Decimating Norb after assigning materials produced visibly jagged sleeves and belt borders even with the material delimiter enabled. Simplify the plain body first, then bisect clothing boundaries and unwrap. The resulting human models have 5,430 triangles including the hairstyle and face parts.
- **Angle thresholds can mark simplification edges as hard.** Voxel-remeshed, decimated organic surfaces developed visible lighting facets under smooth-by-angle. Clear sharp edges between organic faces (skin, eyes, pupils), while retaining the angle rule on clothing and accessories.
- **Joining does not normalise the origin.** A joined collection inherits the active primitive's origin. Reset it to the foot coordinate frame, then translate mesh vertices so their lowest point is z=0 before positioning a review lineup.

## City kit in Blender (2026-10-08)

- **Blender 5.2's glTF export hides vertex colours in `COLOR_1`.** With materials on the mesh, the defaults (`export_vertex_color="MATERIAL"` or `"ACTIVE"`, `export_all_vertex_colors=True`) write a white `COLOR_0` and move the paint to `COLOR_1`; Bevy only reads `COLOR_0`, so the model turns white. Export with `export_vertex_color="NAME"`, `export_vertex_color_name=<layer>`, `export_all_vertex_colors=False`, then check the accessor. Blender's own glTF import ignores `COLOR_0` as well, so a re-import looks white even when the file is right.
- **Re-import the exports into one review scene.** A street of the exported `.glb` files showed the colour bug that the per-model renders, made from the live scene, could not show.
- **A support check catches real gaps.** "Island touches the ground or a supported island" (bounding boxes, 2 cm tolerance) found a sign floating 10 cm in front of its wall. Edge split cuts each face into its own island, which the bounding-box test still handles.
- **Live Blender over the MCP:** `read_factory_settings` in a live Blender would also reset the add-ons, the MCP add-on among them (expected, not tried). City scripts therefore clear only their own collection when they run live.

## Live MCP character refinement (2026-10-09)

- **Compare the live study with a fresh script build.** Norb's jaw and side-part smoothing, then the lower mop crown, were explored in live Blender through MCP and transferred to `refine_norb` in the source script. Both reproduction checks compared 2,799 live-study vertex positions with the rebuilt mesh and measured a maximum nearest-vertex distance of 0 m. The human variants remain at 5,430 triangles.
- **Strong smoothing can erase the haircut.** Five passes flattened the side part; two passes at factor 0.4 kept its broad swept shape. The mop needed a separate crown-height adjustment relative to the scalp.
- **A file load invalidates the executing context's screen.** After `open_mainfile` inside an MCP command, `bpy.context.screen` was `None`; get the new screen from `bpy.context.window_manager.windows[0].screen` before setting viewport angles.

## Mesh orientation checks (2026-10-09)

- **Check closed islands separately.** A small inverted TV screen or limb can hide inside a positive whole-model signed volume. Check consistent edge winding, nonmanifold edges and signed volume per island before edge splitting; allow intentional open shells. Verify exported triangle normals against corner normals as well.
- **Rounded footprints need a radius limit.** A 10 cm radius in a 4 cm deep City TV screen crossed its own outline. Require radius no greater than half the smaller footprint dimension; the repaired screens use 1.5 cm.
- **Skin can fold acute branch junctions.** Norb's wrist/thumb branch produced folded and detached hands. Recalculating normals alone passed the volume checks while a live face-orientation view still showed the fold. Keep the arm and palm as one connected Skin surface and use rounded thumb primitives at the skeleton positions. The repaired human variants have 5,648 triangles (spiked hair 5,646).
- **Decimation can leave a collapsed face pair.** The spiked hair contained two opposite faces sharing the same vertices, enclosing no volume. Remove only that exact remnant, then validate the final mesh.
- **Make Blender errors fail the command.** Use `--python-exit-code 1` for generators and Blender regression tests so a Python validation failure cannot look like a successful export.

## Godot era (historic)

Findings from the Godot spikes 1 to 8. The engine is gone (see `DECISIONS.md`, "Engine"); keep what transfers (precision, origin shift, flight feel), do not copy Godot specifics.

### Ships, walking inside, origin shift (spikes 3 and 5, Godot)

- Walker as a child of the ship, velocity relative to the ship, ship ignoring the walker's layer: holds up to about 400 m/s with roll (mm drift).
- Thin slabs (ramps) over uneven terrain let a capsule slip underneath; boarding on slopes needs its own design and test session.
- **Check the obvious collider before blaming precision.** The 16 km "precision wall" was the parked ship standing in the walk path. How: log slide collisions (collider, normal, contact height) before drawing conclusions.
- **A safety net can hide the test.** The CPU height fallback kept the walker on the ground while the collision ring had silently built no patches. How: count frames where the net is the only thing holding the body.
- **Origin shift breaks silently wherever world and planet coordinates mix.** Example: a density function got a planet position and subtracted the centre again; the sky turned black only once the planet left the origin. How: one conversion function, type or name every position as world or planet-relative, and test with the planet away from the origin.
- **Physics survives far from the origin, the picture does not.** Walking and landing worked up to 197 km from the origin; the error is in the float32 `view * model` on the GPU (about 2.5 px at 100 km for something 2 m from the camera). Measure in pixels, not only in physics.
- **A difference of two far positions has the float resolution of the far distance.** Measuring steps relative to a planet 100 km away showed fake 7.8 mm stairs. How: measure in the frame where the body lives, or in doubles.
- **To reproduce a bug on film, search headless with the same fixed frame rate first.** The test bot reacts per frame, so the frame rate changes what happens: the ship tumble ended in space at 60 fps (3 of 3) but landed at 30 fps.

### Flight feel (Godot prototypes)

Prototype findings; the values were candidates, never accepted tuning.

- **Flight assistance and throttle retention are separate choices.** Continuing to move can mean commanded cruise or inertial drift. Ask which release behavior the initiator wants before implementing.
- **Keep transcript numbers as research notes.** The supplied video summaries warn of ASR errors and omit original links and dates. They support exploring control principles, not adopting exact speeds, ratios or keybindings.
- **Curvature needs a force budget, not just a turning nose.** During acceleration a large target-velocity error consumed all thrust and the ship climbed in a nominally level run. Reserve curvature and drag support first, then spend the rest on velocity correction.
- **Use terrain clearance for the local speed envelope.** Preview terrain over braking time and reduce clearance by the descent stopping distance; sparse samples still cannot guarantee obstacle avoidance.
- **New speed targets change existing bot timeouts.** A slower vertical target silently turned a phase into a different test. Extend phase budgets and check attained altitude in the report.
- **Passing motion checks does not establish flight feel.** Raising cruise speed while keeping the braking force lengthens the stopping time proportionally; the initiator felt abrupt starts and heavy steering. Responsive steering and quick release stops are different preferences.
- **Nose, aim and trajectory are distinct feedback.** Measure and display ship attitude separately from camera attitude: a chase camera looking 10 degrees below the nose made level camera flight a climb and read as "automatic climbing".
- **Finite gravity needs finite planet follow too.** If gravity fades out with altitude, scale the follow force, the angular transport and the terrain preview with the same envelope. A partial follow must integrate a turning velocity in the preview.
- **Small-planet orbit speeds collide with ordinary cruise values.** On a 5 km planet with 9.81 m/s² the circular speed is about 221 m/s and escape about 313 m/s. Model calculation, not a decision to simulate orbits.

### Builds and Windows (spike 7, Godot extension)

- **Cross-build Windows from Linux with llvm-mingw.** Target `x86_64-pc-windows-gnullvm`, linker `x86_64-w64-mingw32-clang` from `mise install github:mstorsjo/llvm-mingw`; no root, no Microsoft SDK licence.
- **Proton costs little for compute.** The Windows build under Proton Experimental reproduced the checksum and took 0.62 ms per chunk against 0.59 ms native. Headless only; rendering under Proton is untested.
