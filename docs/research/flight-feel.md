# Flight feel: what makes Star Citizen fly well, and what a low-fidelity game needs

Research of 2026-10-09: web sources, the local Star Citizen records in `research/local/sc-logistics` (structure only, no values taken) and the planet code of the code repo. It answers the initiator's question: "Recherchiere mal im internet nach berichten warum Star Citizen sich gut anfühlt, ob wir das mit low fidelity auch hin bekommen und was uns aus den Game files noch fehlt um das SC Spielgefühl umzusetzen." The plan that came out of it is code repo epic #143.

## Why Star Citizen flies well

Ranked by weight; numbers in brackets point to the sources at the end.

1. **Mass and inertia, with thrust where the thrusters sit.** CIG: "moment of inertia, mass changes and counter thrust are VERY necessary"; IFCS is "just the interface" between pilot and thruster physics [1]. Loadout and cargo change handling through force over mass [2].
2. **Jerk.** The 2.0 IFCS uses a third-order motion target; tuning jerk alone gives ships a range "from highly responsive and jerky, like a high performance sports car, to less responsive but smooth" [2]. Since 3.10 jerk is finite and ships feel heavier [3].
3. **Every ship flies differently,** with strong and weak rotation axes [2], [3].
4. **Speed caps for combat (master modes, 3.23).** Lower speeds bring fights closer [4]. Players are split on it [8]–[10].
5. **Atmosphere is its own regime:** weaker thrusters, lift and drag per surface, stall [3].
6. **Body and cockpit:** G stress with blackout and redout [2], look-ahead [5], planned cockpit sway and jolts [6].
7. **Seamless scale:** mountains visible from orbit, curved horizon, landing anywhere [7].

## What survives low fidelity

Points 1–6 are simulation, camera and sound; none of them needs texture detail. Swink defines game feel as real-time control with polish and a correction loop under 100 ms [12]. Small teams show it works: Project Wingman (three people, praised for weight and the sense of speed low) [14], Tiny Combat Arena [13], House of the Dying Sun [15], Flight of Nova [16]. Scale (point 7) needs a stand-in: silhouettes, haze layers and reference objects. Perceived speed comes from optic flow near the ground, not from detail [18], [19].

## What the local Star Citizen records hold for feel

None of this was read before. Structure only.

| Area | Structure | EXO-1 |
|---|---|---|
| G camera effects | G bands drive FOV and effect strength through curves | FOV from speed only |
| Head bob from G | spring-damper per axis from the G vector, plus "fake G" from speed | none; later (#43) |
| Look-ahead | weighted look points, roll from input, horizon align, velocity offset | simple version |
| Shake | vibration inputs through curves to shake, noise, first/third-person scale | none (#148) |
| G stress | per axis tolerance, stress over time, recovery, pass-out | none (#43) |
| Jerk, boost ramp | jerk profile linear/angular; boost pre-delay, ramp, curve | none (#146) |
| Turbulence | ground turbulence by altitude and speed band, fractal noise on angular acceleration | none (#147) |
| Proximity assist | sphere and ray tests near terrain above a speed | stopping distance only (#151) |
| Thruster audio | one thrust signal into many parameters, each with its own attack and release; spool timers | minimal loops (#150) |
| Thruster effects, backwash | thrust drives particles; dust by speed and density | none (#49) |
| Atmospheric effects | trails by angle of attack, contrails, re-entry heat | streaks only (#163) |
| Landing gear | spring, damping, lengths per gear | camera bump (#162) |
| Aerodynamics | lift and drag per surface over angle of attack, stall | quadratic drag (#164) |

Missing from the local extract: the screen-effect shader parameters (only a list of references), the HUD layout (not in the records), the audio data, the vehicle entity records (wind and hull sounds), and quantum camera effects (empty in this build).

## Worlds worth flying

- **Relief.** Noise peaks about 160 m (3 % of the radius); Hearth's needle is 400–420 m, Cinder's caldera 200–260 m; canyons 30–45 m deep and under 1.2 km long. On a 5 km planet a few hundred metres already towers. The needle's tip (~5,420 m from the centre) is above Hearth's quantum obstruction radius (5,400 m): to check (#153).
- **Horizon** on a 5 km radius: 141 m from 2 m, 993 m from 100 m, 3.15 km from 1,200 m. Curvature cuts the view before haze does.
- **Haze.** The distance fog scales with the density at the camera, so it is not a height fog; Hearth loses ~21 % at 2 km. Stacked ridges need 50–80 % at 1–2 km (#155).
- **Arches and overhangs** on a height shell: a ground stamp plus a mesh with a collider, as Star Citizen places rock sets on its terrain. Sites and scatter have no colliders today; the arch can be flown through (#156).
- **Low flight:** Star Citizen's community race courses (Snake Pit 10.5 km) became official landmarks; one long canyon per planet would give us that (#154).
- **Collision at speed:** the ring covers 100 m around the ship and its point 0.5 s ahead; above ~200 m/s that point is outside the ring (#152, later).

## Sources

[1] https://api.star-citizen.wiki/comm-links/13951
[2] https://robertsspaceindustries.com/en/comm-link/engineering/15031-Design-Notes-Flight-Model-Changes-In-Star-Citizen-Alpha-20
[3] https://api.star-citizen.wiki/comm-links/17647
[4] https://api.star-citizen.wiki/comm-links/20053
[5] https://citizenwiki.cn/Game_Options
[6] https://www.pcgamer.com/au/latest-star-citizen-video-shows-off-future-cockpit-improvements
[7] https://www.vice.com/en/article/star-citizen-planet-landing
[8] https://robertsspaceindustries.com/spectrum/community/SC/forum/3/thread/i-am-a-real-pilot-and-i-think-mm-is-great
[9] https://robertsspaceindustries.com/spectrum/community/SC/forum/4/thread/master-modes-have-we-reached-the-point-yet
[10] https://massivelyop.com/?p=521491
[12] https://en.wikipedia.org/wiki/Game_feel
[13] https://stormbirds.blog/2022/02/27/first-impressions-of-tiny-combat-arena/
[14] https://www.pcgamer.com/project-wingman-review/
[15] https://gameinformer.com/games/house_of_the_dying_sun/b/pc/archive/2016/11/11/short-sweet-and-somewhat-sentimental
[16] https://www.skywardfm.com/post/first-impressions-flight-of-nova
[18] https://human-factors.arc.nasa.gov/groups/HCSL/publications/Foyle_AHS92.pdf
[19] https://trid.trb.org/View/1646045
Camera shake: https://www.gamedeveloper.com/programming/video-sprucing-up-cameras-with-math ; camera mistakes: https://gdcvault.com/play/1020460/50-Camera ; race courses: https://starcitizen.tools/The_Snake_Pit
