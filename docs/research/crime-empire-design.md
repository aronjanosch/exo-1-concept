# Crime empire design: what to borrow from other games

Research note, 2026-10-09, after the identity grilling (`CORE-LOOP.md`, `DECISIONS.md` from "Identity: a gangster empire"). Not a decision. Structure and lessons only; nothing is copied. Source quality is mixed: most entries rest on reviews, wikis and search summaries, few developer postmortems were reachable; each appendix tags its sources and marks claims from memory.

## The short answer

| Our system | Borrow | Avoid |
|---|---|---|
| Production | Station verbs you do with your hands and readouts on the machine itself; a recipe space with hazards and saved recipes (Potion Craft); more tasks than hands and one new complexity axis at a time (Overcooked); visible progress at once (PowerWash); mix effects to discover (Schedule I) | Rules that say "you can't do that"; dead waits on timers; number lists that are solved once |
| Staff and automation | Automate only what the player just did by hand (Factorio); staff after hand-work, assigned in one action; production that runs while you are away, collected at one hub (GTA Online nightclub) | Forced early automation; a late game of pure management (Schedule I's criticism); slow, fragile staff |
| Territory and family | Hand districts or rackets to lieutenants, each with a perk path, as a real choice (Mafia III); named ranks (The Godfather); crew with personalities who may refuse (Empire of Sin) | The same takeover mission in every district; a strategy layer without threat (Empire of Sin, Omerta); XP grinds to open the next region |
| Fronts and money | Fronts each with their own goofy activity plus passive income (Saints Row 2022); clean and dirty cash, dirty cash costs when spent openly (Cartel Tycoon) | Illegal work that always out-pays legal work |
| Police and heat | A visible suspicion meter and escalating scans with choices: comply, flee, fight, talk or bribe (Starsector, X4, EV Nova); smuggling holds that lower detection; free ports; forged papers (No Man's Sky); suspicion from how you carry and move (Drug Dealer Simulator); escape as an activity, a zone to leave (GTA); tiers with different police behaviour (Cyberpunk 2.0); heat as a deliberate risk choice (NFS Heat); decay plus a way to pay it off (Star Citizen, Elite) | Silent scans; heat that never drops or drops only by timer; one mistake locking a whole faction; too many stacked meters; frequent forced raids (GTA Online) |
| Licences | Graded exams that are the tutorial (Gran Turismo); certificates that unlock cargo classes independently (ETS2 ADR); a rank bar per company (Sea of Thieves) for legal front and crime | Training harder than the early game |
| Jobs | A board of reused verbs (deliver, smuggle, gather, eliminate) with variants (Rebel Galaxy Outlaw); short sneak runs where a partner distracts (Everspace 2) | Delivery jobs that pay badly; sell runs others can grief |
| Heists and FPS instances | Commit a run to stealth, loud or mixed; randomise relevant variants; 2-minute sub-goals, non-linear, so drop-in and role swaps work (Payday 2); setup missions then a finale with roles (GTA heists); police in waves with pauses (Payday 3); few enemy archetypes, authored cover markers, an attacker cap, scaling by crew size; simpler AI felt smarter (a small FPS postmortem) | Loot locked behind one playstyle; fully generated levels; full cover analysis in a first version; unresolved payout disputes |
| Tone | Humour in items, loot and mechanics first, barks second, cutscenes last (Borderlands); a tone reference reel before writing (Saints Row 3) | Talky, low-density humour; the same bark again and again; losing the satire |
| Co-op | Rival-clan and police meters kept apart; crew roles in production and in runs | Scan farming as an exploit; one player waiting for another |

Gaps worth a later look: developer talks on Mafia III's rackets, Empire of Sin, Chinatown Wars' trading, Payday 2's level design, GTA Online heists.

## Appendix A. Crime and business empires

Research 2026-10-09. Source tags: [DEV] developer interview/talk/statement, [REV] review, [COM] community/forum, [WIKI] encyclopedic. Almost everything here is secondary (reviews/community); developer sources found are few and flagged. Items marked (bg) come from general knowledge of the game, not from a fetched page, and should be verified before relying on them. Structure and lessons only.

#### Per-game blocks

##### Gangland (2004, MediaMobsters)
- Loop: city-builder-like rackets plus real-time tactical fights (stick-ups, hits, battles) directed RTS style, in one city of districts.
- Worth borrowing: racket placement as a city-builder layer; events in real time on top of a management layer.
- Pitfalls: repetitive gameplay, bad checkpoint saves, mechanics that punish using certain units later (hidden penalties). Metacritic 63.
- Sources: https://en.wikipedia.org/wiki/Gangland_(video_game) [WIKI]; https://www.gamespot.com/reviews/gangland-review/1900-6091479/ [REV]

##### Empire of Sin (2020, Romero Games)
- Loop: 1920s Chicago, choose one of 14 bosses, buy/take over venues (speakeasies, casinos, brothels), hire a crew (up to 16) with personalities and relationships, turn-based combat to take rival territory; goal is kill all rivals within 13 in-game years.
- Worth borrowing: crew members with personalities/relationships who may refuse orders (cheap source of goofy events); designing the game via a board-game prototype first [DEV].
- Pitfalls: bugs, headline features (economy, management) felt unimportant next to combat, single "kill all rivals" win condition, repeated NPC models.
- Sources: https://shacknews.com/article/114574/romero-games-interview-building-the-empire-of-sin [DEV]; https://wccftech.com/review/empire-of-sin-disorganized-crime/amp/ [REV]; https://en.wikipedia.org/wiki/Empire_of_Sin_(video_game) [WIKI]

##### Omerta: City of Gangsters (2013, Haemimont)
- Loop: Prohibition Atlantic City; strategic layer (buy businesses, run rackets, launder profits through fronts, grow territory) plus turn-based tactics.
- Worth borrowing: fronts as the explicit laundering step in the economy layer.
- Pitfalls: "two games in one" where neither is excellent; no challenge or threat in the strategy layer; shallow combat.
- Sources: https://www.denofgeek.com/games/omerta-city-of-gangsters-pc-review [REV]; https://www.gameinformer.com/games/omerta_city_of_gangsters/b/pc/archive/2013/01/31/mashing-up-two-styles-into-a-mess.aspx [REV]

##### Mafia III (2016, Hangar 13)
- Loop: take a district by hurting its rackets (interrogate, wreck, kill goons), kill the district boss, then assign the racket/district to one of three underbosses. Each underboss has a perk path (weapons, police-bribe, backup); income "earn" grows with assignments. Sit-downs are procedural-dialogue scenes where underbosses argue over splits (GDC 2017 talk).
- Worth borrowing: the allocation decision as the real choice (who gets what, who gets jealous) and perks tied to the people you feed.
- Pitfalls: district loop quickly becomes the same rhythm and slows the story; the allocation consequences felt thin to many reviewers (bg).
- Sources: https://www.pcgamer.com/mafia-3-review/ [REV]; https://www.thesixthaxis.com/?p=276288 [REV]; https://gamesbeat.com/?p=131270 (postmortem) [DEV]; https://gamedeveloper.com/design/video-designing-i-mafia-iii-i-s-system-driven-dialogue [DEV]

##### Cartel Tycoon (Early Access 2021, 1.0 2024)
- Loop: Colombian cartel management sim; grow, legal vs dirty cash, laundering via legal businesses (taxi, casino, amusement park); pressure from authorities (bribe demands, police/DEA, buildings seized if you spend dirty cash too loudly).
- Worth borrowing: two money types and a visible exposure meter that punishes spending dirty cash; bribe-or-fight events.
- Pitfalls: early access reviewers found it inconsistent and early; the pressure can only be avoided, not fought (bg for depth).
- Sources: https://www.gameskinny.com/5qpfn/cartel-tycoon-early-access-review-this-is-bat-country [REV]; https://www.gamesasylum.com/2024/03/13/cartel-tycoon-review/ [REV]

##### GTA Online businesses (bunker, MC businesses, nightclub; 2015-)
- Loop: buy a property, stock it via supply missions (or buy supplies), staff produce over real time, then run a sell mission to deliver the stock. Raids/defend missions happen on MC businesses and bunker; nightclub passively collects from your other operations and rewards owning several.
- Worth borrowing: production that continues while you do other things; a hub business that ties other businesses together (nightclub); upgrades that reduce raid frequency (security).
- Pitfalls: MC business criticised as poor pay for time, glitches, frequent raids, bad delivery missions; public-lobby sell missions invite griefing because a global signal goes out; time limits when selling in bulk; a bunker bug once blocked use of bunkers. Players prefer other activities that pay better per time.
- Sources: https://game-wisdom.com/general/gta-online-businesses_-need-know [REV/analysis]; https://sportskeeda.com/gta/why-griefing-will-always-huge-part-gta-online-experience [COM]; https://www.dexerto.com/gaming/terrible-gta-online-bug-makes-it-impossible-to-use-bunkers-1461087/ [REV]; https://www.gamepressure.com/gtaonline/faq/z6a04a [guide]

##### GTA: Chinatown Wars (2009, Rockstar Leeds)
- Loop: buy/sell six drugs between dealers; prices swing with the day and with cameras (higher when cameras are present), a stand-alone trading puzzle on top of the main game.
- Worth borrowing: a tiny, understandable trading layer where the map (not a menu) shapes risk; "one more piece of the puzzle to keep you motivated" [DEV, the studio via Edge].
- Pitfalls: not enough sources found on criticism; trading minigames can become spreadsheet play if prices have no map reason.
- Sources: https://www.pocketgamer.com/articles/009095/r/ [DEV/preview]; https://destructoid.com/?p=31004 [REV]; https://www.shacknews.com/article/54888/grand-theft-auto-ds-sports [REV]

##### Red Dead Online: Moonshiner (2019)
- Loop: buy a shack (25 gold bars), run a distillation mini-process with timed steps, deliver a limited-quantity product to buyers, with ambushes en route; a "role" with its own progression.
- Worth borrowing: a small, hands-on production step rather than a menu timer; delivery with ambush risk.
- Pitfalls: RDO grind and currency complaints overall; shack selling bug with very long loading screens; payoff low early so role needs a large entry fee.
- Sources: https://www.thegamer.com/moonshiner-role-red-dead-online-best-money-maker-students/ [REV]; https://steamcommunity.com/app/1404210/discussions/0/4342111331712219296 [COM]

##### Saints Row (2022) Criminal Ventures, and Saints Row 2/3 territory
- Loop 2022: set up front businesses (clinic, toxic-waste disposal etc.) in districts, each with its own activity type that raises passive income; Saints Row 2/3 use gang territory takeover and optional "diversions" (Insurance Fraud, Mayhem) that earn money and respect.
- Worth borrowing: each venture has its own goofy activity (the comedy lives in the activity, not in a menu); passive income that accrues; comic fronts as the identity of the business. Criminal Ventures was one of the few praised parts of the 2022 game [REV].
- Pitfalls: the 2022 game lost the satire and mission variety; diversions repeat; the tone that worked was a place where absurd things happen with consequences, not random wackiness.
- Sources: https://www.trustedreviews.com/reviews/saints-row-2022 [REV]; https://checkpointgaming.net/reviews/2022/08/saints-row-review-its-boss-time [REV]; https://gamesradar.com/saints-row-review-2022 [REV]; https://www.wikipedia.com/wiki/Saints_Row_III [WIKI]

##### Drug Dealer Simulator 1/2 (Byte Barrel / Movie Games)
- Loop: open-world dealer; sell to clients, turn customers into addicts then into dealers who work for you; police patrol more aggressively if you carry a big rucksack and rush between drops; if caught you lose stock plus a fine.
- Worth borrowing: behaviour-based police suspicion (how you move draws attention); customers becoming staff.
- Pitfalls: reviews mixed, feels dated, slow; the sequel's reputation was also hit by a review-bombing episode around Schedule I. Policy: ignore that drama for design.
- Sources: https://cogconnected.com/review/drug-dealer-simulator-2-review/ [REV]; https://www.keengamer.com/articles/reviews/pc-reviews/drug-dealer-simulator-2-review-a-complex-narcos-operation/ [REV]; https://www.thexboxhub.com/drug-dealer-simulator-2-review/ [REV]

##### Schedule I (2025, TVGS, one dev; co-op up to 4)
- Loop: make products by combining mix-ins (6 products, 16 reagents), deal to customers in regions that unlock via XP and customer relationships, buy properties (restaurants, car wash as legal fronts for laundering), hire dealers/cooks/cleaners/botanists, avoid police frisks and roadblocks. 97 percent positive on Steam at about 298k reviews (secondary summary).
- Worth borrowing: mixing discovery is the core fun and creates stories; legal fronts with a laundering cap; staff roles; co-op; the game sold on a small clear loop.
- Pitfalls (community): late game is tedious bulk ordering and XP grinding to unlock the next region; employees are slow, expensive, buggy, and assigning/reassigning is awkward because stations share the same name; automation is not viable at larger scale; missing logistics roles.
- Sources: https://wnhub.io/news/stores-and-publishing/item-47411 [REV/news]; https://vaporlens.app/app/3164500/schedule_i [aggregate of Steam reviews]; https://steamcommunity.com/app/3164500/discussions/0/599651305036763457 [COM]; https://steamcommunity.com/app/3164500/discussions/1/599652266084976897 [COM]; https://gamegeeker.com/games/schedule-i-3164500/review [aggregate]

##### Sleeping Dogs (2012, United Front)
- Loop: undercover cop in a triad; three XP tracks: Cop (lawful), Triad (violent), Face (reputation from favours for citizens), unlocking different perks.
- Worth borrowing: split reputation tracks so one deed can raise one meter and lower another; Face from favours for locals unlocks discounts/cars.
- Pitfalls: little empire/business play; the tracks mostly unlock cosmetics and fighting upgrades (bg).
- Sources: https://en.wikipedia.org/wiki/Sleeping_Dogs_(video_game) [WIKI]; https://gameinformer.com/games/sleeping_dogs/b/xbox360/archive/2012/07/23/mixed-loyalties-and-massage-parlors.aspx [REV]; https://newatlas.com/sleeping-dogs-game-review/23836/?amp=true [REV]

##### The Godfather (2006, EA Redwood Shores)
- Loop: extort businesses, take over rackets/warehouses/hubs/rival compounds, and rise Outsider, Enforcer, Associate, Soldier, Caporegime, UnderBoss, Don, Don of New York; held territory must be guarded or rivals take it back (compared to GTA San Andreas gang turf).
- Worth borrowing: a named rank ladder (the names are generic, ours must be invented); extortion by intimidation with an interactive pressure step (bg); defend-or-lose pressure.
- Pitfalls: turf defence loops tied to the open world get chore-like (bg); the game is a GTA-imitator, strongest in its mob fantasy.
- Sources: https://en.wikipedia.org/wiki/The_Godfather_(2006_video_game) [WIKI]; https://strategywiki.org/wiki/The_Godfather:_The_Game [guide]; https://worthplaying.com/article/2006/5/6/reviews/33121-xbox-review-the-godfather/ [REV]

##### Yakuza: Like a Dragon business minigame (2020, RGG)
- Loop: after chapter 5 an optional company-management game (stores, restaurants, nightlife): invest, hire staff with stats, hold shareholder meetings, rank up; at top rank a big lump sum for little work.
- Worth borrowing: a business sim wrapped in character and comedy with its own rank ladder; staff as characters with traits; periodic "meeting" payouts instead of constant collection.
- Pitfalls: optional and mostly detached from the main game; money becomes trivial once understood (bg).
- Sources: https://gamespot.com/articles/yakuza-like-a-dragon-how-to-make-money-fast-and-manage-businesses/1100-6484448/ [guide]; https://www.pcgamer.com/uk/yakuza-like-a-dragon-management-mode-set [guide]; https://en.wikipedia.org/wiki/Yakuza:_Like_a_Dragon [WIKI]

#### Borrow (mapped to EXO-1 systems)

1. Arc: start with a tiny, clear loop (Schedule I lesson) and unlock one system per rank step; new districts unlock by relationship/rank, not by a pure XP grind (counter to Schedule I late game).
2. Arc: make the break-away/take-over moment a real branching beat; Mafia III's sit-down idea (a short scene with a hard choice) fits it.
3. Family/ranks: one named ladder (invented names) where each rank grants one new capability (errands, own shop, hire staff, run a district), as in The Godfather and Like a Dragon.
4. Family/ranks: allocation as choice (Mafia III): you assign districts/rackets to lieutenants who each want something; the favoured one grows, the other plots. Keep to 2 or 3 lieutenants.
5. Family/ranks: split reputation meters (Sleeping Dogs): family standing, street face, police heat, so deeds trade one off against another.
6. Territory: district control as a share of customers (your own rule) shown on the map; make it a dial that moves daily, so there is no "capture and forget".
7. Territory: income accrues over time and is collected at hubs; a nightclub-style hub that pays out from other businesses (GTA Online) is a good reason to build more than one.
8. Operation/staff: each venture has its own goofy hands-on activity (Saints Row, Moonshiner) and a passive background mode; staff are characters with traits and quirks (Empire of Sin, Like a Dragon) who can refuse orders.
9. Operation/staff: design the staff UI first (Schedule I's biggest complaint): clear names for stations, bulk assign, saved roles, and logistics roles from day one.
10. Police: exposure from behaviour (Drug Dealer Simulator: carrying and hurrying looks suspicious) plus a wanted level with scans and raids; Cartel Tycoon's seized buildings give a concrete loss.
11. Rival: customers drift to the rival through visible actions (price, quality, ads), and are answered by your own actions; the rival must be a threat in the management layer (Omerta lacked this).
12. Laundering: two money types (clean/dirty), fronts with a cap per day tied to legal footfall, and a wanted-level cost when the numbers do not match (Cartel Tycoon, Schedule I).
13. Laundering: legal courier jobs double as front income; the more you launder through them the more audit pressure.
14. Contraband: a mixing/combination discovery loop (Schedule I) is the strongest creative hook found; invented effects make co-op stories.
15. Co-op: roles for 2 to 5 players (cook, courier, enforcer, accountant) so each has a distinct job on a shared business, and pace by shared time rather than solo grind.

#### Avoid

1. Do not repeat the same district-takeover mission shape for every district (Mafia III, Gangland).
2. Do not make raid/defend missions frequent, forced, or tied to time you do not control (GTA Online MC businesses); give defence as a choice with a preparation step, and let preparation lower the frequency.
3. Do not expose sell/delivery runs to griefing by other players; in co-op the opposition should be AI (rival clan, police).
4. Avoid a strategy layer without threat (Omerta) and a "management" layer overshadowed by combat (Empire of Sin).
5. Avoid two half-games glued together; if combat exists it supports the economy.
6. Avoid XP/bulk-order grinds for the next region (Schedule I late game); gate by relationships/story beats.
7. Avoid expensive, slow, buggy staff and weak automation (Schedule I); either automate well or keep staff small.
8. Avoid hidden penalties that punish an option later (Gangland); tell the player the cost.
9. Avoid pay that is poor per hour of tedious delivery (MC businesses): if a business is dull it must also be a gamble for a story.
10. Avoid wackiness for its own sake; Saints Row 2022 lost praise when the satire went; our brutal-but-goofy tone needs consequences.

#### Gaps
- Few developer postmortems were reachable via search snippets; Mafia III (GDC), Empire of Sin, Chinatown Wars are the only dev-side leads. Fetch those pages directly if depth is needed.
- Sleeping Dogs, Godfather, Like a Dragon, Moonshiner details marked (bg) are unverified.

## Appendix B. Hands-on production

Research for EXO-1 (2026-10-09). Structure and lessons only.

Source tags:
- [P] = primary (developer text/talk), fetched and read.
- [S] = secondary (review, guide, wiki, press), fetched or seen in search results.
- [K] = from general knowledge of the game, no source fetched this session. Treat as unverified and check before leaning on it.

Honest gap: few dev postmortems for these games are freely readable. Most game blocks below are [S] or [K]. Several fetches (Astroneer UE interview) returned 403.

#### Per game

##### Schedule I
- Works: stations as distinct physical places (pots, drying, mixing, packaging); quality tiers; mixing effects give a discoverable recipe space [K]. Staff split by job (Botanist, Chemist, Handler, Cleaner); the player assigns each worker to stations, recipes and output shelves. [S]
- Pitfall: automating one product needs many workers (community counts about 6 for one chain); setup is clipboard/assignment admin; players ask for better routing (assign to shelves, pick outputs). Late game becomes management chores, not hands-on. [S]
- Source: https://gameshorizon.com/guides/how-to-automate-mixing-in-schedule-1 [S]; https://steamcommunity.com/app/3164500/discussions/1/599650563430310893 [S]

##### Drug Dealer Simulator 2
- Works (from memory): mixing/production at a physical bench, with customer-driven demand. [K]
- Pitfall: nothing verified. No source fetched; do not cite.
- Source: none.

##### Potion Craft
- Works: an alchemy map as a 2D recipe space. Each ingredient moves the brew along its own path; raw vs crushed changes path length; water pulls back; bad zones (skulls) punish. A solved recipe can be saved and replayed, so the puzzle is paid once. Depth comes from routing, not from numbers. [S]
- Pitfall: daily ingredient supply vs customer requests creates optimisation pressure; once the map is learned, repeat brewing is a chore unless the shortcut (saved recipes) exists. [S]
- Source: https://www.destructoid.com/potion-craft-alchemist-simulator-relaxing-alchemy-shop-sim-impressions/ [S]

##### Potionomics
- Works: brewing is a ratio puzzle (5 essence types, tiers Minor to Masterwork, star quality from amounts). Hidden "sweet spot" quantities reward experimentation and community sharing. [S]
- Pitfall: community datamines the sweet spots; the discovery is solved externally. Hidden numeric sweet spots feel like spreadsheet play, not hands-on. [S/K]
- Source: https://www.pcgamer.com/potionomics-review/ [S]; https://gamepretty.com/potionomics-how-to-make-super-potions/ [S]

##### Moonlighter / Potion Permit
- Works: split loop, gather in the dungeon/world and sell at the counter, so each half gives the other a reason. [K]
- Pitfall: not verified. No source fetched.
- Source: none.

##### Overcooked / PlateUp!
- Overcooked works: "too many tasks for the players you have" gives controlled panic; several timers per level keep situations unravelling so players cannot settle into a routine; shared stations force communication; limited ingredient types and simple interactions kept it readable. Each new recipe was paired with a simpler environment, so complexity was added one axis at a time. [P]
- Overcooked pitfall: early prototypes with strict recipe rules ("you can't do that") were dropped; the fun moved to environment puzzles and teamwork. [P]
- PlateUp!: roguelite restaurant with conveyor/automation as a logistics puzzle layered on the co-op kitchen. [S] Details of how automation is introduced are not verified.
- Source: https://mcvuk.com/the-develop-post-mortem-overcooked/ [P]; https://www.gamereactor.eu/every-single-person-has-an-integral-role-in-overcooked/ [S]; https://en.wikipedia.org/wiki/PlateUp! [S, not fetched]

##### Factorio / Satisfactory
- Works: progression order is the point: manual mining, automated mining, automated logistics, automated production and science. Each step automates something the player just did by hand, so they feel the upgrade. Some items cannot be hand-crafted, which pushes toward automation. [P]
- Pitfall: Factorio's 2017 tutorial forced assemblers in the first five minutes, which broke that order; the team called it a "local maximum" and reverted it in 0.18. Manual crafting gets slow as the game grows by design. [P]
- Satisfactory: first-person 3D factory-building with hand-crafting early. No design source fetched. [S]
- Source: https://factorio.com/blog/post/fff-327 [P]; https://wiki.factorio.com/Crafting [S]; https://www.godisageek.com/2024/02/we-want-to-make-the-game-we-feel-our-community-deserves-satisfactory-interview-with-snutt-treptow-part-3 [S, interview, not design-focused]

##### Stardew Valley
- Works: crop timers that fit a day/season rhythm; quality stars; artisan goods as a second processing step that turns one crop into a higher-value product with its own wait time. [K]
- Pitfall: not verified. No source fetched.
- Source: none.

##### Lethal Company / Content Warning
- Works: loot is a physical item you carry; weight/hands-full and shared risk create co-op moments; goofy tone survives horror. 4-player crews. [S/K]
- Pitfall: not verified beyond overview. Content Warning not covered.
- Source: https://www.giantbomb.com/lethal-company/3030-90069/ [S, not fetched]

##### Job Simulator / House Flipper / PowerWash Simulator
- PowerWash works: instant, high-contrast visible progress on every input; almost constant positive feedback; no timers or enemies; complexity added by tools (nozzles, detergents, dirt durability) rather than pressure. Devs distilled what was satisfying, not what was realistic. [P]
- Pitfall: tool/pose friction (e.g. top of wheels) needed patching; tedium must be removed on purpose. [P]
- Source: https://kotaku.com/powerwash-sim-devs-on-making-cleaning-fun-advanced-cro-1847319502 [P]; https://www.shacknews.com/article/131441/powerwash-simulator-developer-interview [P]
- Job Simulator / House Flipper: not covered by any source.

##### Hardspace: Shipbreaker
- Works: tools as physical interactions in a diegetic UI (immersion first, usability and accessibility balanced); sound rules tied to physics. [S]
- Pitfall: not found in what I read.
- Source: https://www.gdconf.com/news/go-behind-fully-diegetic-ui-hardspace-shipbreaker-gdc-2021 [S, abstract only]; https://gdcvault.com/play/1027227/Breaking-the-Silence-The-Sound [S]

##### Astroneer / No Man's Sky / Deep Rock Galactic / Raft / Subnautica
- Astroneer: power and resources are physical blocks moved by hand between backpack, smelter and printer; modular components snap together; "things clicking into place" is the feel. [S]
- DRG: four distinct classes each with a mining/co-op job. [S]
- NMS refiners, Raft, Subnautica fabrication: not researched; no claims.
- Pitfall: none verified.
- Source: https://www.unrealengine.com/developer-interviews/inspired-space-inside-the-development-of-astroneer [P, 403 on fetch, unread]; https://astroneer.wiki.gg/wiki/System_Era [S]; https://deeprockgalactic.wiki.gg/wiki/Developer [S]

#### Borrow (10-15)

1. Machines: one verb per station, physical input (drop crate/item into a hopper, pull lever, hold item under nozzle), visible output at the other end. Astroneer-style snapping and "click into place" feel. [S]
2. Machines: add depth by tools and variants (PowerWash nozzles/detergents), not by timers or threats. [P]
3. Machines: a diegetic readout on the machine itself, not a menu (Shipbreaker abstract). [S]
4. Recipes/quality: a 2D "recipe space" the player moves through by adding inputs (Potion Craft map), with our own invented axes and hazard zones; discovery by routing, not by lookup table. [S]
5. Recipes/quality: save a solved recipe so repeat batches are cheap (Potion Craft), keeping the puzzle at discovery time. [S]
6. Recipes/quality: if hidden sweet spots exist, make them depend on several inputs and the machine's physical state so datamining a number list does not solve them (Potionomics lesson). [S]
7. Growing: crop timers sized to the work loop (plant, go mine/deliver, return), plus a second artisan step with its own wait (Stardew pattern). [K]
8. Mining: keep it a physical haul (carry the ore, carry the crate), so mining feeds the hands-on chain instead of a number. Lethal Company/Astroneer style. [S/K]
9. Packaging: crates are already physical; make packing a visible fill/seal action with a clear before/after contrast and a satisfying sound/pop (PowerWash "dirty to clean" idea applied to "loose to boxed"). [P]
10. Staff/automation: follow Factorio order; automate a step only after the player did it by hand, and only the step they now find tedious. Never force it early. [P]
11. Staff/automation: staff take a role at a station (Schedule I roles) but need a short assignment UI; keep setup to one action per worker (point at station, done). [S]
12. Staff/automation: automation moves the hands-on work up a level (new bottleneck, new rare machine or recipe) rather than removing it. Inference from Factorio order; Schedule I late game shows the failure case. [P/S]
13. Co-op: more tasks than hands (Overcooked), several independent timers so crews cannot idle in a routine. [P]
14. Co-op: roles by station and by carrying, not by locked classes at first (Overcooked), maybe light specialisations later (DRG). [P/S]
15. Co-op: introduce new recipes or machines in simpler layouts, one complexity axis at a time. [P]

#### Avoid (6-10)

1. Forcing automation before the player has felt the manual pain (Factorio tutorial revert). [P]
2. Late game as pure management: worker counts, clipboard assignments, shelf routing (Schedule I complaints). [S]
3. Strict recipe rules that tell the player "you can't do that"; let wrong combos produce funny bad results instead (Overcooked prototype lesson). [P]
4. Recipe depth that is only a hidden number list; the community will solve and post it. [S]
5. Timer waits with nothing to do; every long timer needs a parallel task. Inference from Overcooked multi-timer design. [P]
6. Pile of menus on machines; keep the UI diegetic or on the machine. [S]
7. Tool/pose friction on the physical interaction (PowerWash patch case): test every reach and angle. [P]
8. Staff count as the only automation lever (6 workers for one chain); give scale through machine upgrades too. [S]
9. Adding new complexity on several axes at once (new recipe plus busier room plus more players). [P]
10. Relying on unverified claims above; verify [K] items before they go into DECISIONS.md.

#### Open follow-ups (not researched)
Drug Dealer Simulator 2, Moonlighter/Potion Permit, Stardew, Lethal Company/Content Warning, Job Simulator/House Flipper, NMS refiners, Raft/Subnautica, PlateUp! automation details, Astroneer UE interview (403).

## Appendix C. Smuggling, licences and job variety in space

Date: 2026-10-09. Scope: structure and lessons only. Sources marked [D] developer/official, [W] community wiki (maintained by players, mostly accurate), [S] secondary/press/forum, [M] from memory, not re-verified this session (check before relying on it).

#### Per game

##### Rebel Galaxy Outlaw (Double Damage)
- Prequel; ex-smuggler heroine pulled back into the life. Job board offers delivery, smuggling, raw-material gathering and "eliminate target" jobs; side activities (billiards, dice poker, radio) carry the goofy-grungy tone. [S] https://en.wikipedia.org/wiki/Rebel_Galaxy_Outlaw
- Tone reference: Privateer; economy-driven progression (credits buy ship upgrades instead of skill points); random missions generated from the same tech as story missions. [D, series dev interview] https://www.gamedeveloper.com/audio/the-making-of-rebel-galaxy-part-2
- Lesson: the job board is a small set of verbs reused with flavour text; tone does half the work. Detailed scan rules not verified.

##### Starsector (Fractal Softworks)
- Transponder on: identity broadcast, detected further. Off: patrols in most places investigate; first encounter = demand to turn on and cargo scan, second = combat. Free ports and independent/pirate patrols ignore it. Fighting with transponder off gives only incremental rep loss; turning it on afterwards links you and applies the full penalty. [W] https://starsector.wiki.gg/wiki/Transponder
- Black market trade builds "suspicion" (visible as a tooltip). High suspicion makes patrols ask for a cargo scan. Clean scan: released (chance raised by Shielded Cargo Holds, i.e. hiding compartments). Contraband found: confiscated. Options: allow, fight (hostile), or talk them down (costs a rare resource). Free Ports ignore smuggling. Legit trades and docking with transponder off lower suspicion. [W] https://starsector.wiki.gg/wiki/Black_market
- Dev blog on trade/smuggling: https://fractalsoftworks.com/tag/trade [D, not read in full]
- Lesson: one visible "suspicion" meter, one cheap item that skews the odds, and a clear three-way choice at the scan.

##### Elite Dangerous (Frontier)
- Cargo scan reveals illegal goods; consequence historically a fine, not instant death. Black market sells stolen/illicit cargo, only in some stations. Lawless systems: attacks give no bounty. [S forums/guides] https://primagames.com/tips/elite-dangerous-piracy-guide-nav-beacon-no-fire-zone
- Fines are paid at station security; bounties only at Interstellar Factors, which needs zero Notoriety and adds a 25% fee. Notoriety decays 1 unit per 2 h of play. [S/W] https://elite-dangerous.fandom.com/wiki/Crime_%26_Punishment
- Known exploit: friends scan an illicit-cargo ship to farm bounties. [S] https://steamcommunity.com/app/359320/discussions/0/595136747144234133
- Lesson: separate "fine" (money, small) from "bounty" (hunted) from "notoriety" (cooldown that gates paying off). Timer-based decay plus a fee is an easy, readable loop. Watch for co-op abuse of scans (relevant: EXO-1 is co-op).

##### Star Citizen (CIG)
- CrimeStat: 5 tiers; earned by assault, murder, station damage, illegal goods trafficking. Consequences: lawful NPCs hostile, landing denied, bounty hunters, prison. Decays over time; low tiers payable as fines; clear at a guarded outlaw post; offences registered by comm-arrays, which can be disabled. [W] https://starcitizen.tools/CrimeStat
- Mission pages show reputation-ranked cargo runs with graded cargo (e.g. a drug-cargo faction mission). [W] https://api.star-citizen.wiki/missions/dead-saints-reputation-rank-rank-cargo-grade-token-scale-cargo-run-2
- Drug labs are fought-over locations with redesigned interiors (fewer doors, sightlines). [D comm-link] https://api.star-citizen.wiki/comm-links/19156
- Lesson: "crime is registered by infrastructure" (comm-arrays) gives a physical, sabotageable heat source. Prison as a place is memorable but costly to build.

##### No Man's Sky (Hello Games)
- Outlaws update: pirate-controlled systems; Bounty Master at outlaw stations hands out procedural piracy missions; regulated stations periodically scan arrivals for illegal goods and may send Sentinel interceptors on non-compliance; "Forged Passports" earned from pirates reset/raise standing with authorities. [S] https://www.dsogaming.com/patches/no-mans-sky-outlaws-update-3-85-released-full-patch-notes/ and https://www.wepc.com/news/no-mans-sky-outlaws-update/
- Lesson: forged paperwork as a reward item that converts crime income into a clean slate; scans at the arrival point (pad/dock) fit EXO-1 pads directly.

##### Escape Velocity Nova (Ambrosia)
- Each system owned by a faction with its own legal status for you; criminals may need to bribe port authorities to land; low status = ports closed, attack on sight. Missions for a government and killing its enemies raise status; piracy lowers it. [S] https://evn.fandom.com/wiki/Escape_Velocity_Nova
- Lesson: per-faction legal status plus bribe-to-land is the cheapest working model of "law differs per place".

##### Freelancer (Digital Anvil) [M, not verified]
- Zones of faction space, police/patrol ships that hail and ask for cargo, smuggler-friendly outlaw bases, a job board with trade/combat missions. Not researched here; use only as a reminder of a readable "police hail, you jettison or fight" loop.

##### X4: Foundations (Egosoft)
- Police ships leave traffic lanes to scan random nearby ships for wares illegal to the sector's owner. Pre-set captain responses to an interdiction: Attack, Comply (drop the wares, default), Escape, Wait. Refusal turns the police hostile, possibly the whole station. [W, official wiki] https://wiki.egosoft.com/X4%20Foundations%20Wiki/Manual%20and%20Guides/X4:%20Foundations%20Manual/NPC%20Behaviours/
- Active scans need roughly 1 km proximity; info depth depends on scanner type. [W, same wiki family]
- Forum: at +20 reputation a faction ignores illegal wares in its space; police anger spreads to the whole faction. [S] https://steamcommunity.com/app/392160/discussions/0/7539652761277771150
- Station empire as the long-term goal (Egosoft product page, [D] https://www.egosoft.com/games/x4/info_en.php).
- Lesson: "what is illegal" belongs to the sector owner; good rep can buy leniency, so legal work shields illegal work.

##### Space Rangers 2 [M, not verified]
- Many hybrid careers (trader, pirate, warrior, peaceful) in one world with ranks and per-race relations. Use only as proof that mixed careers can share one board. Not researched here.

##### Everspace 2 (Rockfish)
- Contracts/Hinterlands update added smuggling: sneak across several locations while a companion distracts enemies; a wanted level in the Union system shows how much the local government considers you a threat. [S] https://gamingbolt.com/everspace-2-contracts-hinterlands-update-adds-smuggling-bomber-and-much-more
- Lesson: smuggling can be a short stealth run, not only a cargo scan lottery. Companion/partner distraction fits co-op.

##### Sea of Thieves (Rare)
- Six trading companies, each a playstyle: Merchant Alliance (deliver goods between outposts), Gold Hoarders (treasure), Reaper's Bones (take down rival crews, deliver loot to the haven). Rank 50 in three companies = "Pirate Legend" with a hideout. Crime-ish play emerges from players, not from a law system. [S/W] https://seaofthieves.wiki.gg/wiki/Trading_Companies
- Lesson: ranks per company give a legal front and an illegal front their own progression bars; player-vs-player theft is the emergent heat in co-op.

##### MudRunner / Euro Truck Simulator 2 (SCS)
- ETS2 uses XP-bought skills; ADR certificates (classes 1,2,3,4,6,8) are bought independently (not linear) and unlock more profitable dangerous cargo. [S/W] https://truck-simulator.fandom.com/wiki/Skills?oldid=4457 and https://eurotrucksimulator2.com/about.php [D]
- MudRunner: no licences found; gating is by vehicle/terrain. [M]
- Lesson: a licence = "this cargo class pays more and is unlocked"; independent unlocks, not one ladder.

##### Grand Theft Auto (contrast)
- No licences; stars rise with chaos, escape when out of sight long enough; changing clothes or vehicle reduces what police know (new GTA 6 reports; secondary, unverified press). [S] https://beebom.com/gta-6-wanted-system-explained/
- Lesson: "out of sight timer + disguise" is the most readable evasion; star bands map to escalating response.

##### Gran Turismo licence tests
- Licences (B, A, International A) each a set of eight short tests of increasing difficulty that teach driving techniques and gate harder races. [S] https://en.wikipedia.org/wiki/Gran_Turismo_(1997_video_game)
- Lesson: an exam is a tutorial with a pass/fail result and a reward; it teaches before the game punishes.

##### City/truck games with certifications [M]
- Not researched; ETS2 ADR above is the representative example.

#### Borrow (mapped)

Police / heat
1. One visible "suspicion" or heat meter that rises from illegal actions and decays with time (Starsector tooltip, Elite 2 h decay, CrimeStat decay). Show it in the HUD.
2. Scans happen at defined places: pads, checkpoints, a patrol that deviates to scan nearby (X4, NMS). Warn first (hail, "scan in progress" bar), never scan silently.
3. Scan outcome is a choice: comply (drop/confiscate), flee, fight, bribe (X4 Comply/Escape/Attack, Starsector three options, EV bribe-to-land).
4. Heat is local to a faction/district; free ports and lawless zones ignore it (Starsector, Elite lawless systems, EV).
5. Second offence escalates: first stop = demand and scan, second = combat (Starsector transponder).
6. Crime registered by infrastructure (comm-arrays, cameras) that players can sabotage (CrimeStat).

Smuggling
7. Cheap odds-shifter: shielded/hidden hold lowers detection chance (Starsector); sneaking runs with a partner distraction (Everspace 2).
8. Contraband invented per faction; "what is illegal" depends on who owns the place (X4).
9. Confiscation, not death, as the default penalty: lose the cargo plus a fine, keep playing.

Legal vs illegal
10. Legal jobs reduce suspicion and build rep that buys leniency (Starsector legit trade, X4 +20 rep). Courier work is the cover and the cooldown.
11. Ranks per front (Sea of Thieves companies): a legal rank bar and a crime rank bar.
12. Fine vs bounty vs notoriety split with a fee to clear (Elite).

Licences
13. Licences as independent unlocks for cargo/ship classes (ETS2 ADR); flight exam as GT-style short graded tests that double as the tutorial.
14. Forged/stolen papers as a loot item that resets standing (NMS Forged Passport); fake licence is an illegal job reward.

Job kinds
15. Small reusable job verbs on a board: delivery, smuggle, gather, eliminate, escort (Rebel Galaxy Outlaw, Everspace 2, NMS Bounty Master).

Ship theft
16. No researched model for theft/fencing in the sources above (Elite sells stolen cargo at black markets, Star Citizen has ship-theft scanning in Pyro per search snippet, unverified). Suggest: stolen ship carries a "hot" flag that decays; fence at a free-port pad with a cut and a wait; scans read the hot flag.

#### Avoid

1. Silent or instant lethal scans; no warning before attack (frustration; X4 and Starsector both warn).
2. Scan-farming exploits in co-op: bounties/heat awarded to scanners (Elite exploit). Do not pay players for scanning each other.
3. Heat with no way down: always offer time decay plus a paid clear.
4. Prison/grind-based clearing that takes the player out of the game for long (CrimeStat travel to a guarded post).
5. Whole-faction hostility from one mistake that locks players out of all content (X4 spreads anger to the faction; keep it local, per pad/district).
6. Too many stacked systems (reputation x notoriety x licence x faction at once); start with one meter.
7. Consequences hidden in tooltips; make suspicion and legality of cargo readable before the player commits.
8. GTA-style chaos escalation as the main police feel; EXO-1's pitch is tension from hauling, not shootouts.
9. Hand-built heavy locations per crime (drug-lab interiors, prisons) before the loop works.
10. Reward parity problem: if illegal always pays more with only a fine as risk, legal jobs die; tune so legal is the shield, not the grind.

#### Gaps

Thin sources: Freelancer, Space Rangers 2, MudRunner, ship theft/fencing, Rebel Galaxy Outlaw scan details, Elite/X4 pages that returned errors. Verify before decisions.

## Appendix D. Co-op crime, heat, tone and FPS

Date: 2026-10-09. Structure and lessons only, nothing to copy.

Legend: [P] = primary (developer talk, postmortem, developer-written article or interview as reported by the page fetched); [S] = secondary (press, wiki, fan site, summary); [M] = from general knowledge, NOT verified in this research run (treat as hypothesis, check before relying).
Honest caveat: no GDC video was watched; GDC Vault pages give only abstracts. Several developer-interview pages returned 403 (Unreal Engine interviews). Gaps are named per section.

#### 1. Co-op crime and heists

##### Payday 2 / Payday 3
- [P] Level design write-up "Designing highly replayable stealth levels for Payday 2" (80.lv, Murky Station): https://80.lv/articles/designing-highly-replayable-stealth-levels-for-payday-2/
  - Decide early per level: stealth-only, loud-only or mixed. The commitment shapes space (stealth allows tight vents) and patrol balance.
  - Replayability comes from randomisation that changes gameplay (vault type, key location, layout variant), not cosmetics; the article cites hundreds of combinations from modular prefabs scripted once and reused.
  - Objectives are non-linear and cut into roughly 2-minute sub-goals; kept simple so drop-in/drop-out works.
  - Environment teaches by visual vocabulary (tape, colored lights); several routes; "onion layers" toward the goal.
  - Guard patrols split into short segments so one alarm does not chain into a total collapse.
  - Roles emerge from tasks (overwatch, infiltration, loot securing), four people busy at once.
- [S] Replayability summary, plan changes when stealth fails: https://players.com.ua/en/news/13-years-of-payday-2-how-a-co-op-heist-shooter-became-an-immortal-phenomenon/?amp=1
- [S] Payday 3 launch/stealth/assault-wave notes: https://www.videogameschronicle.com/news/embracer-says-payday-3-underperformed-following-launch-issues/ , https://mp1st.com/reviews/payday-3-review-rusty-heists , https://insider-gaming.com/payday-3-felt-unfinished-developers-admit-a-year-after-disastrous-launch
  - Police arrive in waves from different directions with breaks; hostages buy time between waves.
  - Criticism: some heists lock the full loot behind stealth, forcing one playstyle. Launch failure was mostly online infrastructure, not heist design.
- [M] Physical loot bags that slow the carrier and must be thrown or hauled to an exit make the "loud" phase a logistics puzzle. Verify before use.

##### GTA Online heists
- [S] Structure summary: https://www.gtaboom.com/GTA_Online:_Heist_Payout_Guide , https://kotaku.com/heres-what-gta-onlines-heists-will-look-like-1671685562
  - Setup missions then a finale; each crew slot has a distinct job in the finale.
  - Leader sets role assignment and the percentage cut per member; crew gets small pay per setup, the leader is paid at the finale.
  - Lesson for payout splitting: leader-set cuts create social drama (good for a crime family) but also griefing and arguments; the split is a negotiation surface.
- Gap: no developer postmortem found.

##### GTFO
- [S] Interview/feature: https://primagames.com/featured/gtfo-developers-warn-their-hardcore-co-op-shooter-isnt-for-everyone , https://rog.asus.com/us/articles/gaming/scarcity-leads-to-brilliance-in-the-smash-hit-action-horror-game-gtfo/
  - Hand-built instanced expeditions, scarcity of ammo/tools, detect-prepare-control rather than run and gun; deliberately hostile to a niche.
  - For EXO-1: instanced missions as authored "sites" with scripted-plus-light-random objectives are within small-team scope; GTFO's difficulty tone is not.
- Gap: the GDC talk the query hoped for was not found.

##### Deep Rock Galactic
- [S] Developer interview page (403 on fetch, snippet only): https://www.unrealengine.com/developer-interviews/guns-gold-and-glory-in-the-caverns-of-deep-rock-galactic
  - Recipe described as Left 4 Dead pacing plus procedural levels; "procedural" means hand-built pieces assembled by code, which keeps control of variation.
  - Four distinct classes that need each other; loot doubles as ammo/progress.
- [S] AI-director pacing context: https://www.gamedeveloper.com/game-platforms/q-a-valve-s-swift-on-i-left-4-dead-2-i-s-production-ai-boost

##### Lethal Company
- [S] https://en.wikipedia.org/wiki/Lethal_Company , https://gameindustrylibrary.com/documents/pushtotalk-how-lethal-company-sold-10-million-copies
  - Physical scrap carried by hand, rising quota as a run clock, proximity voice makes failure funny; a tiny scope (one developer) sold on social comedy.
  - Gap: no postmortem by the developer found.

##### Sea of Thieves
- [S] https://www.gameinformer.com/b/features/archive/2017/11/17/how-rare-cast-away-its-developmental-process-for-sea-of-thieves.aspx , https://www.windowscentral.com/sea-thieves-anniversary-update-interview
  - Original pitch "players creating stories together": give tools and a shared world; the stories are the other players. Later added authored "tall tales" because pure emergence needed structure.
  - For EXO-1: a rival clan and police sharing the same space can create stories; keep a few authored arcs for the family storyline.

#### 2. Police and wanted-level design

- [S] GTA: the search area as a visible zone you must leave, not a timer; swapping vehicle or breaking line of sight helps; newer designs let police use gathered information (description, outfit, camera): https://www.grandtheftwiki.com/Wanted_Level_in_GTA_IV_Era , https://gtaintel.com/news/gta-6-police-system-overhaul-explained (the GTA 6 text is pre-release reporting, treat as unconfirmed).
- [S] Need for Speed Heat: pursuit enters a cooldown state you must survive; day escape resets heat, night keeps it, so heat is a risk/reward choice tied to a time-of-day mode: https://nfs.fandom.com/wiki/Need_for_Speed:_Heat
- [S] Cyberpunk 2077 2.0: police patrol and react to crimes in the world, pursuit units escalate by wanted level, special strike teams at top level, scans of crime scenes: https://pcgamer.com/cyberpunk-2077-2-0-patch-breakdown , https://www.shacknews.com/article/137150/cyberpunk-2077-update-2-patch-notes
  - Gap: no developer commentary on why; patch notes only.
- Gap: Watch Dogs and Saints Row notoriety not researched with sources. [M] Notoriety in Saints Row ties heat to factions (police vs gangs), which maps directly to EXO-1's police plus one rival clan.

Lessons (mixed: sources above plus reasoning):
- Fun heat is legible (zone, level icon, counter), escalates in distinct tiers with new behaviours (not just more units), and offers several escapes (break line of sight, hide, bribe, lay low, finish the job).
- Annoying heat is instant, undodgeable, spawns on top of the player, or punishes minor accidents; or when decay is a pure timer with nothing to do.
- Decay: make it an activity (leave search area, lay low in a safe house, pay off) so waiting is not dead time. Day/night-style tradeoffs (NFS Heat) show heat can be a deliberate choice.
- Co-op: one shared level or per-player? Decide on shared crew heat plus individual scan exposure; hard to verify from sources, mark as open design question.
- Payday 3 waves with breaks (S) is a model for raids: telegraphed assault, pause, next wave, so players can plan.

#### 3. Comedic brutality and tone

- [S] High on Life: every gun has its own full set of reactions to each NPC and situation, so humour is reactive to what the player holds; improv-style rambling that continues as long as the player stays; Roiland on games: "like writing a TV show that people can reach in and knock things around". https://techraptor.net/gaming/features/high-on-life-justin-roiland-interview , https://www.gameinformer.com/2022/12/13/justin-roiland-almost-didnt-voice-the-main-gun-in-high-on-life
  - Pitfall evidence: a review criticised the humour as low density and talkative, https://gameinformer.com/review/high-on-life/low-on-laughs (talking-heavy comedy wears thin when jokes are not hit by gameplay).
- [P-via-summary] Saints Row The Third, GDC 2012 (Scott Phillips): define tone first (a "tone video" of reference clips), consistency fixed an incoherent predecessor, team ownership of ideas revealed identity, and they cut features by the thousands of man-days, including traditional cover and parkour. https://gamedeveloper.com/design/gdc-2012-what-makes-a-i-saints-row-i-game-anyway-
- [S] Borderlands 2, Anthony Burch GDC talk "Designing Humor in Borderlands 2": humour through mechanics and loot (an item that is the punchline), jokes can be "debugged", pattern-breaking as dark comedy, a quest can be funnier by removing gameplay. https://www.gameskinny.com/?p=9967 , https://gamedeveloper.com/design/hard-edge-creativity-defining-i-borderlands-2-i-  (talk title/themes via press; fetch of the GDC page failed)
- [M] Trover Saves the Universe and Virtual Rick-ality: absurd interaction props and short gags in a simple loop; not sourced in this run.

Lessons:
- Put comedy in verbs, items and consequences (a loot item with a stupid side effect, a weapon with a personality, an enemy that reacts absurdly to being hit) rather than lines of dialogue.
- Item descriptions and barks cost little and scale for a small team; long cutscenes do not.
- Brutality needs a cartoon logic (consistent gore style, absurd cause) so it stays funny; real brutality plus goofy tone works only if the stakes in the family storyline are played straight.
- Bark rules: cooldowns, per-situation pools, never in the middle of important callouts, one-liners that are short.
- Playtest jokes like mechanics: log which barks players hear repeatedly.

#### 4. Small-scale FPS with cover in co-op

- [P-abstract] Shatterline AI postmortem, GDC AI Summit (co-op shooter): prototype early; "dumb AI gets smarter when simplified"; high-intensity encounters felt chaotic until simplified; adaptive difficulty. https://gdcvault.com/play/1028932/AI-Summit-Shatterline-Practical-AI
- [P-abstract] The Last of Us human enemy AI (GDC 2014, Naughty Dog): real-time level analysis for cover/navigation, search behaviour, mistakes listed. Large team, use only as a vocabulary. https://gdcvault.com/play/1020338/The-Last-of-Us-Human
- [S] F.E.A.R.: one AI programmer, planning AI made soldiers read as smart; reported as the standard for small-team AI. (Jeff Orkin GDC 2006 talk, found via https://hardforum.com/posts/1042921450/ , not the original) 
- [P-via-summary] Saints Row: cut traditional cover (see section 3). Cover is expensive; consider what level of "cover" you actually need.
- [S] Left 4 Dead director pacing: https://www.gamedeveloper.com/game-platforms/q-a-valve-s-swift-on-i-left-4-dead-2-i-s-production-ai-boost

Lessons:
- Cover points as authored markers in the arena (hand-placed nodes with a facing and height), not generated analysis; enemies pick the best free node by distance from players and exposure.
- Few enemy archetypes that differ by behaviour (rusher, shooter-from-cover, flanker, heavy), each readable by silhouette and sound; complexity comes from combinations.
- Intensity director: limit concurrent attackers, token system, periods of quiet (L4D-style) so fights stay legible with 2-5 players.
- Arenas: instance = few rooms with 2-3 routes, clear sightlines, authored cover density; reuse prefabs, randomise which objective/variant spawns (Payday 2).
- Co-op: scale enemy health/count by crew size, not AI sophistication.
- Test AI headless against the scenario harness (matches the repo's agent loop).

#### Borrow (13)
1. Commit per mission to stealth, loud or mixed; design the space for that mode (Payday 2 write-up).
2. Randomise gameplay-relevant variants of authored prefabs (vault type, key location, entry route), not cosmetic ones.
3. Cut objectives into roughly 2-minute sub-goals, non-linear, so drop-in/drop-out and role swapping work.
4. Setup mission leads to a finale; setup gives small pay and shapes the finale (GTA heists), which also teaches the crime-family structure.
5. Leader or boss sets the cut of the payout; the cut is the story lever for the family and the later break-away.
6. Physical loot carried by hand/bag, slowing the carrier, with a quota or target as the run clock (Lethal Company, Payday).
7. Role = a distinct task plus one signature gadget, not a class with a separate progression (DRG classes, heist roles).
8. Police as visible tiers with different behaviours; assaults come in telegraphed waves with pauses (Payday 3, Cyberpunk 2.0).
9. Heat decays through activity: leave the search area, lay low, pay a fixer, finish a job; make the escape verbs explicit (GTA search zone).
10. Heat tied to factions (police vs rival clan) as separate meters, so the family and the rival give different pressures.
11. Humour in items, loot and mechanics first, barks second, cutscenes last (Borderlands 2 talk).
12. Define tone with a reference reel and a one-page rule list before writing content (Saints Row 3).
13. Simplify enemy AI on purpose; intensity director with an attacker cap; authored cover markers (Shatterline talk, Saints Row cutting cover).

#### Avoid (8)
1. Locking the full loot behind stealth so players are forced into one mode (Payday 3 criticism).
2. Instant, unavoidable heat from small accidents or spawns on top of the player.
3. Heat decay that is only a timer with no action available.
4. Joke density that depends on spoken lines alone; repeating the same bark in short windows (High on Life review criticism).
5. Hardcore, high-scarcity difficulty tuning for a casual crew (GTFO openly niche).
6. Procedural levels that are fully generated; use hand-built pieces assembled by code (DRG).
7. A full dynamic cover-analysis system and complex planning AI in v1; also too many enemy types before the first works.
8. Payout rules with no way to resolve disputes; one player waiting idle while others do setup (GTA heist leader gets paid late).
