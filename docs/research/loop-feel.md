# Loop feel: what makes a delivery more than A to B

Research note, 2026-10-09, after the first playable round of milestone D (code repo #133). Not a decision. Structure only: no names, numbers or texts from other games are proposed for EXO-1.

The initiator's playtest verdict, 2026-10-09: "Macht so keinen Spaß aber das liegt natürlich daran dass es sich komplett wie ein test anfühlt. Es geht sehr schnell es hat null tiefe. Es ist einfahc und man halt kaum feedback für eine Belohnung. Wir haben weder Sotry noch sonst irgendwas." and "zu einer Lieferung gehört dann auch eine art von navigation, die man erledigen muss, wir haben keine Karte, Die Flug mechaniks müssen sich super anfühlen damit man auch spürt was man macht. Es braucht lebendigkeit, sound, Feedback, Die Distanzen müssen etwas länger sein. Einfach nur Fracht von A nach B ist super langweilig."

Sources: Star Citizen data (`research/local/sc-logistics`), No Man's Sky data (`research/local/nms`), Schedule I files (`research/local/schedule-i`), public write-ups. The four raw notes are the appendices A to D; the structural data model is in `loop-references.md`.

## The short answer

All three games run a bare state machine much like ours. What makes it feel like a job is the layer around it:

- **Beats instead of one event.** A delivery is 8 to 13 small staged moments (a message arrives, accept, briefing, marker appears, loading, take-off, a twist en route, arrival, hand-over, payout counted up, standing moves, the next thing unlocks), each with its own text, sound or animation and short pauses between them. Ours has three (accept, crates appear, money changes).
- **Someone wants it.** Jobs come from a few named givers with a voice, taste and memory (first time, after a failure, last chance), text assembled from authored parts with a reason why the cargo moves.
- **Finding the place is part of the task.** Markers come from place queries; some targets have to be found (scan, clue, landmark) instead of shown.
- **Variety on independent axes**, not more of the same: route class, cargo kind, fragility, time pressure, hazard, multi-stop, second stop at the giver, optional greed objective.
- **The trip is the content.** Weight changes handling, terrain and weather resist, distance takes minutes, not seconds; arrival is a ritual.
- **Rewards on several tracks and announced one by one**: money, standing with a giver or faction, unlocks queued for calm moments.

## By the initiator's points

### Navigation and a map
- SC: places are tag-resolved tokens, one marker label per objective, one tracked contract, multi-stop in free order; no starmap hook in contract data (the map is a separate system).
- NMS: every pointer is a place query (target kind, scope, filters with fallback, marker label, icon, discovery sound); some targets are found by scan pulses or clue chains, with GPS fuzz and near/far text.
- Schedule I: every place is a POI; a job creates and moves its own pin; phone map; a compass that shows distance only when near; track/untrack.
- Write-ups: landmarks and partial maps (Breath of the Wild, Outer Wilds, SnowRunner); a marker-only HUD removes the activity.
- For EXO-1: a planet map (from the baked macro data we already have) with tracked-job pins; place queries in job objectives; pads with a visual identity and a few landmarks; later a scan that reveals a hidden target.

### Feedback and reward
- SC: fixed notification kinds per beat (available, accepted, updated, complete, failed, reward), a money toast that says where the money went, loading notifications, an out-of-time warning.
- NMS: rewards are their own stages; each reward announced; music stings at start and end; waits of 0.5 to 2.5 s between beats.
- Schedule I: payment counted up, satisfaction and relationship shown, bonuses itemised with a jingle, rank-up and region unlock held until a calm moment; each currency has a colour and a sound.
- Write-ups: many small layers, intensity scaled to importance (a crate ticks, a finished job gets a fanfare).
- For EXO-1: a beat/notification system (banner, toast, sound id per beat), an arrival ritual at the pad, payout counted up and itemised (base, condition, bonus), sound on pickup, lock, landing, payout.

### Story and framing
- SC: briefings from parts (title formula, giver voice, per-shape paragraph variants, optional reason keyed to cargo or route); persona givers react to history.
- NMS: one template layer for generated and story missions; text pools in parts plus a named client; arcs as chained missions with a progress counter.
- Schedule I: many named NPCs with taste, voice and relationship; story as a quest chain with a mentor and an antagonist.
- Write-ups: a small cast of recurring contacts beats a big anonymous board; generate the situation from small world state.
- For EXO-1: givers as records (name key, voice pool, mood by relationship), text fragments in three parts plus a reason, a short authored intro chain using `once_only` and `follow_up` that already exist.

### Variety instead of A to B
- SC: ~300 presets from independent axes; hazards; "all or nothing" rule; free-order multi-stop.
- NMS: same skeleton with different middles (ambush en route, pickup a person, repair, photo, scan, clue chain); a second stop at the giver.
- Schedule I: one template, many knobs (customer taste, quality, quantity, price haggle, time window, region, risk overlay).
- Write-ups: vary the situation (complication, twist), not only the numbers; optional greed objective (Deep Rock Galactic).
- For EXO-1: modifiers that change play (fragile, heavy, timed, hazardous route, multi-stop, return to giver), encounters en route (step 5 already planned), an optional extra objective.

### Distances, pacing, liveliness, flight feel
- SC: expected times per route class from several minutes for local to an hour for interstellar.
- NMS: length comes from a second stop and a target that must be found; waits bracket each beat.
- Schedule I: liveliness from NPC schedules, voice barks, time-of-day music, ambient loops, weather, small interactive props.
- Write-ups: the trip itself must be pleasant or tense (sound, weight, weather, terrain); calm and tension alternate.
- For EXO-1: longer routes (other side of the planet, the second planet by warp), the city as a living destination (`art/city`), ambient and ship sound, the flight model of spike 13 as the base of "feeling what you do".

### Co-op
- Write-ups: interdependent roles with an information gap (pilot, loader, spotter), shared risk, optional greed.
- For EXO-1: cargo that needs two (large crates exist), a spotter role with the map or scanner, shared payout and its own feedback.

## Appendix A. Star Citizen data

Source: `research/local/sc-logistics` (paths below relative to it). Structure only. Evidence tags: [file] = read in data, [inference] = my reading.
Key finding up front: the hauling data itself is thin on feel. The core template (`contracts/contracttemplates/haulcargo_atob.xml`) has one objective token, one handler, a deadline, partial-payout bands, three flow triggers, and **empty comms lists** (`commsNotifications`, `contractStartCommsNotification`, `contractEndCommsNotifications` are all empty) and **empty journal lists** (`journalEntriesToAddOnComplete` empty in `missionbroker/pu_missions/cargo/haulcargo_rounddelivery.xml`; journal entries are only filled in tutorial/bounty-certification/event contracts). The richness comes from layers *around* the state machine: text assembly, generated variety, standing, global notification strings, the freight UI.

#### 1. Phases and beats, offer to payout

Beats and what is attached to each [file unless marked]:

1. **Offer (board)**: contract appears as "Contract Available: %s" (`global.ini` `mobiGlas_ui_MissionEvent_Available`); optional `notifyOnAvailable`. Card shows type (hauling subtype icon+name, `missiontype/pu/hauling_*.xml`), contractor ("Contracted By", `contract_from`), reward as range "Up to"/"min to max"/"+ Bonuses" (`ui_mission_reward_*`), deadline label (`ui_mission_deadline`), `contract_timed` variant of the contractor line for timed jobs. Objectives are shown before accepting (`preShowObjectives="1"` in template). Offers expire (`contractLifeTime`), have max instances and respawn timers (`generationParams`).
2. **Accept**: toast "Contract Accepted: %s" (`mobiGlas_ui_MissionEvent_Activated`); handler carries accept comms tag set (`contractParams`, see 3). Optional buy-in (`contractBuyInAmount`), refunded on withdraw.
3. **Collect phase**: objective "Collect Shipment" (`HaulCargo_obj_short_01`), long text "Collect ~item from ~location" (`hauling_collect_objective`), progress readout per resource "amount/total SCU" (`hauling_deliver_resource_objective`). The pickup is physically a freight-elevator interaction; the elevator UI shows contract payout and time left (`FreightElevator_ContractOrderPayout`, `FreightElevator_ContractOrderTimeLeft`, "This contract will be completed after you deliver the goods to the warehouse") and rejects wrong goods ("Your current selection includes contracted resources that can't be accepted"). Loading itself has beats: transfer start/stop/interrupt/resumed strings (`cargo_notify_*`, `communicationname/cargotransferstart|stop|end.xml`), loading-area assignment and revoke ("Failure to Report"), forfeit warning ("Cargo Will Be Forfeit If You Leave the Area").
4. **Transit**: combined objective "Collect/Deliver Shipment" (`_short_03`); timer shown only in the last N seconds before expiry (`remainingTimeToShowTimer="15"`), comms lines for running out of time (`communicationname/aligncargooutoftimesoon.xml`, `entercargooutoftimesoon.xml`, `...outoftime.xml`). Optional encounter objective spawned beside cargo or at a dropoff (`SpawnDescription_ShipGroup "DropOff1EnemyShips"` in `contracts/contractgenerator/interstellartransport_guild/redwind/redwind_hauling.xml`: several options, each with concurrent count and weight).
5. **Per-stop deliver**: "Deliver Shipment" (`_short_02`), per-stop counters; multi-stop lists render as "DROP OFF LOCATIONS (ANY ORDER)" with a generated bullet list (`HaulCargo_N_SingleToMultiToken` etc.).
6. **Branch: abandon while carrying** -> second objective "Return Shipment" (`HaulCargo_obj_short_06`, `contracts/contracttemplates/haulcargo.xml`). Fail by timer: end-reason "Delivery Window Expired" (`HaulCargo_Fail_TimeOut`), auto-fail on prison/crime flags.
7. **Complete/payout**: toast "Contract Complete: %s"; "Reward Updated: %s" exists as a live-update toast (`mobiGlas_ui_MissionRewardUpdated`); money toast "You've Earned: ~reward ... Access It at ..." (`Mission_Reward_Notification`); status list Active/Completed/Failed/Abandoned/Withdrawn (`ui_missionstatus_*`). Partial delivery -> reduced pay via four percent bands (known from earlier summary, not repeated).
8. **Aftermath**: reputation result per outcome (symmetric ladder); giver-side reactions are in text (see 4); rank changes are delivered as journal/message text per organisation (`*_ReputationJournal_*_Promotion_BodyText` / `_Demotion_BodyText`).

Where the state machine is: `missionFlow` has just 3 triggers (token completed -> mission completed; token failed -> failed; abandon -> token abandoned). Everything above hangs on the token's `displayInfo` (short, long, marker label, category "Primary", `hideOnHUD`) plus global UI strings, not on extra states.

Implication: A contract should be a thin state machine with a fixed set of named beats (offer, accept, collect, transit, deliver, return branch, payout, aftermath), each beat owning its own label, toast and optional sound hook; the feel is a table of beat -> presentation, not more logic.

#### 2. Navigation

- **Locations are data tokens, never coordinates**: each pickup/dropoff is a mission property (`MissionPropertyValue_Location`, variable names `PickupLocation`, `DropoffLocation1..n`, UI token `Location` / `Destination`), picked at generation time by a tag search (hand-picked tags per route class). Text renders them as a place name with address form (`~mission(Location|Address)`) or a list (`|ListAll`).
- **Objective markers**: every objective has `objectiveMarkerLabel` (short label drawn at the marker, e.g. "Collect Shipment" / "Deliver Shipment") and `hideOnHUD`; template has `hideMarkers` switch on the hauling handler and `displayAlliedMarkers` (party members see each other's markers). Separate short/long/marker/HUD-title text variants exist (`ObjectiveSetupShort/Long/HUD/Marker`, `titleHUD` param on 46 contracts).
- **Tracking**: Contract Manager has "Current Objectives" and a tracked flag ("Tracked", "No Mission Tracked", `ui_Track`); `mobiglasDisplayLayout` is an (empty) per-objective layout hook. [inference: only one mission is tracked at a time and drives HUD markers.]
- **Route/quantum targets**: no quantum waypoint or route object in mission data. The only route-ish thing is a **route class label** (`CargoRouteToken`: Local / Planetary / Stellar / Interstellar) used in title and difficulty; plus quantum-travel music stings in `jumppoints/globaljumpdriveparams.xml` (`MUS_PU_Shared_Scripted_QT_ShortJourney` / `ZeroJourney`) and a general "Journal_General_StarmapQuantumTutorial" text. Starmap integration is not visible in contract records. [inference: the marker feeds the existing starmap/HUD systems in code.]
- Players are never told "go to coordinates"; they get a name + a marker + an address string. For multi-stop jobs, order is explicitly free ("ANY ORDER") so the player plans the route.

Implication: Navigation for us = every stop is a named place resolved at generation time, with a marker label per objective and one tracked contract driving HUD markers; free-order multi-stop makes route planning part of the fun without extra systems.

#### 3. Feedback and reward presentation

- **Notification family** (`global.ini` `mobiGlas_ui_MissionEvent_*`): Available, Activated(Accepted), Complete, Fail, Deactivate(Withdrawn), Shared, plus RewardUpdated and `JournalEntry_Added/Updated`. One string template each, with the contract title substituted. Status enum separate (`ui_missionstatus_*`).
- **Journal entry types** (`journalentrytype/*.xml`): exactly four: AudioLog, DialogueLog, TextLog (shown as "Message"), VideoLog. Each has DisplayName, `NotificationName` (own toast text per type, e.g. "Journal Audio Entry Added") and `IconName`. Contracts can attach entries on start and on complete (`journalEntriesToAdd`, `journalEntriesToAddOnComplete`, ...Remove...), but the hauling ones leave them empty. Used for lore/tutorial/aftermath, not for hauling.
- **Reward feedback**: money toast with where to collect ("Access It at Your Primary Residence's Inventory" for item rewards; money via account); sell order line "Sell Order Completed - deposited to your account"; reward display "X + Bonuses". Reputation change is a numeric result (see earlier summary) and surfaces as rank text in a journal message, not as a per-contract toast (no string found for "reputation increased"). [inference]
- **Comms layer**: `commsNotification` / `commsTiming` / `commsChannelName` fields exist on objectives and contracts (146 notifications, 2,584 channel-name fields, nearly all `@LOC_UNINITIALIZED`). Handlers pick accept/abandon/complete comms by **tag set** (`communicationname/acceptjob.xml` and the `missioncommunications` folder), so a giver's voice line is chosen by tags, not written per contract.
- **Sound/music**: mission data has one hook type, `musicWwiseEvent audioTrigger` (a Wwise trigger name) on reward notifications and scripted encounters; 284 of 286 are empty. Real ones: scripted fight start/defeat stings, a "Collect_Reward" sting on an item reward, jump-point fail/quantum stings. No sound hook on the hauling template. Hence audio exists as a hook but is sparsely used.
- **Failure presentation**: global failure record = one entry with `warningLevel`, `displayText`, `useAutomaticFailureScreen="1"` (`missionfailureconditions/globalmissionfailureconditions.xml`): a generic fail screen; specific reason via `missionEndReason` text on the deadline.
- **Warnings during play**: loading-area warnings, "out of time soon" comms, forfeit warning.

Implication: Give every beat a notification type and a short string from a fixed set (available, accepted, updated, completed, failed, abandoned, reward, standing), plus an optional audio-trigger name and a journal-entry slot; reward feel needs its own presentation event (amount + where it went + standing change), not just a balance change.

#### 4. Framing and story

- **Text is assembled from parts** (`contracts/contractgenerator/**`, `ContractStringParam`): `Title`, `Description`, `Contractor`, optional `TitleHUD` (counts: Title 2393, Description 2373, Contractor 338, TitleHUD 46). Contract UI strings are only wrappers: `contract_title=~mission(Title)`, `contract_desc=~mission(Description)`, `contract_from=~mission(Contractor)`.
- **Tokens substituted into text** (`extendedTextToken`): Location, Destination, Contractor, MissionMaxSCUSize, ReputationRank, CargoRouteToken, CargoGradeToken, Item, plus shape tokens that expand to bullet lists (`SingleToMultiToken`, `MultiToSingleToken`, `RoundDeliveryToken`, `LinearChainToken`), plus Danger, ShipStory, ClaimNumber, approvalcode (flavour tokens on other types).
- **Title is a formula**, not a sentence: `~mission(ReputationRank) Rank - ~mission(CargoGradeToken) Cargo Haul` (`Covalex_HaulCargo_*_title`), or `~mission(ReputationRank) Hauler Needed for ~mission(CargoGradeToken) Shipment` (`RedWind_HaulCargo_SingleToMulti_title_01`). Title tells rank, size grade, shape at a glance.
- **Description has three layers**: (a) a per-company letterhead/voice (one company opens with a slogan banner, another is a chatty dispatcher), (b) a per-shape paragraph that is picked from 3 variants (`..._desc_01/02/03`) that all contain the same tokens, (c) optional **situation flavour keyed to a cargo/route** (`desc_RawOre_Stanton1`, `desc_ToCFP_Danger`, `desc_Valakkar_Timed`, `FromRuinStation`, `ToLevski`, `DeliverAll`). Flavour variants carry the *reason* for the job (a client threatening to leave, a hauler who backed out, a settlement under threat, a mine that struck a vein), then the logistics.
- **Recurring gag/hint line**: "we strongly encourage contractors to bring a handheld tractor beam": the briefing doubles as tutorial/requirements hint (box size, tool).
- **Givers** (`missiongiver/**`): record = faction, display name, description, headquarters, `invitationTimeout`, `visitTimeout`, short/medium/long cooldown, allies/enemies. Persona-driven givers (named people) have large line pools keyed by context in text: `..._MG_Intro_1stTime_`, `..._MG_Greeting_`, `..._MG_MissionBriefs_<type>_`, `..._MG_LastMissionComment_{Fail,FailStreak,LastChance,NoLonger}_`, `..._MG_Bar_`/`ConvoCont`. So the giver reacts to the *state* (first time, last result, fail streak, last chance, fired). These map to the reputation states (success/fail streak, last result) [inference].
- **Chains/series**: rank-gated ladders (Trainee..Master, `RepStanding_TransportGuild_Rank0..6`), **Intro** contract (once only, sets a completion tag, text "Opportunity for Independent Cargo Hauler"), **Rehire** contract ("Re-evaluation": easy qualifying delivery after demotion), special one-offs (`..._Rehire_`, `..._DeliverAll`). Tutorial generator (`contracts/contractgenerator/tutorial/tutorial_generator_stanton1..4.xml`) is a 3-part chain with journal entries and hint strings (`Tut03_Part02_Hint01...`, final "Congratulations" journal message). Event scenarios (`missionscenarios/*.xml`) give a seasonal/campaign layer with progress tracks.
- **Lore carriers**: journal entries (text/audio/video), station-specific datapads, the giver's flavour text, cargo names and places themselves. Hauling itself carries lore only through briefing text.

Implication: Compose a briefing from {title formula, giver voice, per-shape paragraph variants, optional cargo/route flavour with a reason}; one giver identity per job family plus a few context-keyed lines (first time, after fail, after streak) gives story without authored chains; an intro and a rehire contract bracket the progression.

#### 5. Variety and depth in hauling

What makes two hauls differ, as axes visible in generator debug names, e.g. `RedWind_Pyro_SupplyGrade_Solar_CFP_StationToRuin_Carbon_CargoHauling_Multi3ToSingle` (`.../redwind/redwind_hauling.xml`):

1. **Route class**: Local / Planetary / Stellar / Interstellar (own handler and mission subtype each).
2. **Grade** (box size cap and volume): ExtraSmall/Small/Supply(Medium)/Bulk(Large); biggest box = ship-fit gate.
3. **Shape**: A->B, 1->N (N=2..4), N->1, round, linear chain; free order for multi-stop.
4. **Client faction** (e.g. two rival customers inside one company's board) -> different reputation/hostility effects.
5. **Location-type pair**: Station/Outpost/Tradepost/Ruin ... "StationToOutpost" (tag-search per type), plus region letters.
6. **Cargo**: one good or **mixed load** (several goods, each its own container cap and min SCU); goods carry different handling (damageable, volatile, restricted/illegal flag, waste/scrap as low-prestige).
7. **Hazard**: optional ambush ships at a dropoff (weighted options, concurrent counts); "Danger" and "Timed" flavour variants; unlawful flag (`Illegal` param, 429 contracts), crime-stat prerequisite.
8. **Time pressure**: deadline on the template (`missionCompletionTime`, result after timer almost always Failed); "Timed" briefing variants; time shown only near the end.
9. **Special conditions**: "deliver 100% or no pay" brief (`DeliverAll`), recover-cargo (take back stolen/lost cargo, `*_recovercargo.xml`), return-goods branch.
10. **Prerequisites/gates**: standing band, completed-contract tags, crime level, system locality, location.
11. **Risk knobs on payout**: partial bands, buy-in, difficulty rating on four axes (mechanical, mental load, risk of loss, game knowledge; `ContractDifficulty` with each at seven steps) that set the reward profile, bonuses ("+ Bonuses").
12. **Scarcity of offers**: instance caps, per-player cooldowns, lifetime, respawn with variation.

Typical volume: ~300 hauling career contracts across 3 companies, each = (route x grade x shape x client x place-pair x good) preset; 3 description variants per shape; so variety is combinatorial presets + weighted random picks, with a small set of hand-written special briefs.

Implication: Treat a job as a record of independent axes (distance class, load grade, shape, client, place-type pair, cargo mix, hazard, time limit, special rule), generate by weighted picks, and attach a briefing flavour line per notable axis value; two jobs then differ in kind, not only in number.

#### 6. Pacing and length

- **Expected duration is explicit**: each career contract has `contractResults timeToComplete="N"` (minutes [inference from magnitude/scale]) that feeds pay and difficulty. Measured over 303 hauling contracts in `contracts/contractgenerator`: Local/small grade ~7-20, Planetary 10-30, Stellar/Solar 20-100 (median 60 for the solar-class), Interstellar 27-74 (median ~34 for A->B, 42-74 for multi-stop). More stops and bigger grade -> larger value (e.g. Small A->B 15, Supply A->B 35, Bulk A->B 44; Multi3ToSingle Supply 70, Bulk 80). So tiers are time budgets by route class, grade and stop count.
- **Template deadline** (`missionCompletionTime`): 254 templates carry a generic 300, 180 carry 0 (no timer); the real expiry is set per contract; a few special types use 60/90/1800/7200. Timer UI appears only near the end (15 s window in the hauling template).
- **Distance bands**: not computed from km for hauling; picked by hand-chosen location tags per route class (other mission types do use min/max km exclusion). Loading time per box size (big boxes are slower in total but faster per unit) is a separate global (`globalcargoloadingparams`).
- **Tiers/progression**: seven standings (Trainee..Master) and grade/route unlocked by standing band; longer interstellar/bulk jobs sit at higher bands; chain items inside a job (shape) lengthen it without new content.
- Mission type record has a `DisplayTime` field (all -1 for hauling), i.e. no pre-computed duration is shown on the card; the player sees route class, grade and stops instead.

Implication: Length should come from an explicit per-job time budget derived from distance class, load grade and stop count, rising with tier; stacking stops (multi, round, chain) is the cheap lever to turn a short A->B into a 10 to 20 minute run.

## Appendix B. No Man's Sky data

Base: `research/local/nms/exml/metadata/` (short: `m/`). Mission tables `m/simulation/missions/tables/*.MXML`, scan events `m/simulation/scanning/`, rewards `m/reality/tables/rewardtable.MXML`, hub `m/reality/defaultreality.MXML`. Localisation text is NOT in the dump (only keys like `NPC_MISSION_DELIVERY_PROC_A_%d`), so what the lines say is unknown; only how they are assembled. Not repeated from earlier notes: template list, reward choice modes, cost-as-delivery.

Method: I walked the full stage tree of DELIVER, HIDE_SEEK_*, MISSING_PERSON, REPAIR_1, FACTORY_RAID, PIRATES1, SO_POSTMAN, SO_COLLECT_H1, ACT1_STEP3 and counted stage types over all tables (`/tmp/.../scratchpad/skel.py`).

#### 1. Stages and beats

A bare board delivery (`npcmissiontable.MXML`, id `DELIVER`) is NOT "pick up, drop off". Sequence, top level, each a `GcGenericMissionStage`:
1. Group "wrapper for main objective" (`ObjectiveID`, `ObjectiveTipID` = HUD line plus a hint line). Inside: `Wait 1s` -> `ShowMissionUpdateMessage{Start, SetMissionSelected, WaitForMessageOver, PlayMusicSting=Start}` -> reward stages (hand item over) -> `Wait` -> `StartScanEvent` (puts the target marker on the map/HUD, with a delay `Time`) -> branch groups on conditions (is the target local, seeded percentage chance) -> `WaitForConditions` / `WaitForScanEvent` with a `Message` (the objective sentence) -> `Wait 1s` -> `ShowMissionUpdateMessage{End, PlayMusicSting=End}`.
2. Group "return" with its own `ObjectiveID` : `StartScanEvent` (hand-in marker) -> `WaitForCompletionMessage` -> `EndScanEvent`.
3. Two silent reward stages (money tier, then standing).

Beat vocabulary (counts over all tables, dominant): Group (~1000), StartScanEvent, Wait (many short 0.5 to 2 s breathers), Reward, ShowMissionUpdateMessage (Start/End), EndScanEvent, WaitForConditions, WaitForScanEvent, GetToScanEvent, ShowMessage (timed OSD), Communicator (comms hail with portrait/holo and VO dialog id), AudioEvent (raw sound/music cue), ModifyStat, SetCurrentMission, StartMission (chain), Scan (do a scan), GetInShip, Pirates, Kill, Collect*, WaitForPhoto, DetailMessage (a popup page with title, image, bullet lines), BroadcastMessage/WaitForMessage (stage-to-stage signalling).

What fires per beat (all declared in the stage, not hard-coded):
- Start: `ShowMissionUpdateMessage` (big banner) + `PlayMusicSting` (enum None/Start/End/Corrupted; Start and End stings used ~235 and ~281 times) + `SetMissionSelected` (makes it the tracked mission).
- Each step: `Message` (objective line), `ObjectiveID` (log title), `TipID`, optional `AudioEvent` and `OSDMessage`.
- Target marker: scan event `MarkerLabel`, `MarkerIcon`, `MessageAudio` (`UI_NEW_DISCOVERY` on ~180 events, VO cues on others), `MessageDisplayTime`, `IconTime`, `TooltipTime`, `ShowEndTooltip`, highlight style (Diamond almost always, Circle/Hexagon for special).
- End: `ShowMissionUpdateMessage{End}` banner + End sting, then reward stages (see 3).
- Mission level flags: `MessageStart`/`MessageComplete` (Never / Default / Always; "Default" for roughly half, "Never" when the stages show their own banner), `AutoStart`, `RestartOnCompletion`, `CancelSetsComplete`.
- Repeating reminder: group `CustomNotifyTimers{NotifyDisplayTime, NotifyPauseTime}` re-shows the objective on a cycle, and `Survey*Hint` strings tell the player what the scanner/survey key does for this objective (4 variants: inactive, swap, on foot, vehicle).

A second, invisible layer: `MissionClass=Guide` is ~545 of ~1900 sequences (core 48, pirates 13, space POI 87, tutorial 107, ...). Guides have no objective; they are looping watchers (`RepeatLogic=Loop`, `WaitForConditions` on something like HasFuel or "inside a POI") that fire `ShowHintMessage`/`ShowMessage` once when the player first meets a situation (`AutoStart=AllModes`). That is the "game explains itself in-world" layer, authored as tiny missions.

Implication: model a mission as ordered objective-groups, each a list of cheap typed beats (wait, banner, marker on, wait-for, marker off, reward, sound cue). The "feel" is the beat list around the one real condition, and a separate always-on set of tiny context watchers adds hints and flavour without touching mission code.

#### 2. Navigation: how a mission points to a place

Scan event (`GcScanEventData`) is the only way a mission points anywhere; stages merely `StartScanEvent` / `WaitForScanEvent` / `GetToScanEvent` / `EndScanEvent` by name. Fields:
- Target kind: `SearchType` (FindBuildingClass 613, Nexus, SpaceStation, PlanetBaseTerminal, AnyNPC, Any, Atlas, SpacePoi, SpaceAnomaly, FriendlyDrone, Freighter, SpaceMarker, AnyRobotSite, NPC_HideOut, PlanetPostBoxes) plus `SpecificBuildingClass` (e.g. Outpost) and `ForceInteractionType` (MissionGiver, Shop ...) so the arrival target is a specific interaction, not just a place.
- Where: `SolarSystemLocation` (LocalOrNear ~1370, Local, Near, FromList, NearSpecificPartyIndex), `BuildingLocation` (Nearest ~980, AllNearest, PlanetSearch, Random, RandomOnNearPlanet, RandomOnFarPlanet, PlayerSettlement), `SpacePoiLocation`. Filters: `SolarSystemAttributes` (race, conflict, star type, wealth/trading class, biome) with a `...Fallback` set tried when no match, `MinPlanets`, `ExcludePlanetsWithEvents`, `PreferPlanetWhereStatIsZero`.
- Start/end: `EventStartType` (Special = mission starts it; Timer; ObjectScan; Discovered), `EventEndType` (Interact ~1400, Proximity 20, None 134), `TriggerActions{Range}`.
- Presentation: `MarkerLabel`, `MarkerIcon`, `SurveyHUDName`, `SurveyDiscoveryOSDMessage`, `MessageAudio`, `UseGPSInText` (GcScanEventGPSHint: offset-mid / offset-narrow gives a fuzzy "somewhere around here" instead of an exact marker).
- Stage-side: `GetToScanEvent{Distance, Message, GalaxyMapMessage, TimeoutOSD}` has a pre-message (far) and a near message at a smaller `Distance` (core ACT1_STEP2: far 40, near 15), so text changes as you close in. `WaitForConditions` with `IsScanEventLocal` / `OnCurrentPlanet` / `NearScanEvent` switches the hint from "go to the map" to "look around here".

Discovery IS the task in several kinds: clue chains (PIRATES1: crash site marker -> space marker -> radar marker, one new marker per objective, each with its own label `..._CLUE1/2/3_LABEL`); hide-and-seek (`NPC_HideOut` target the player must find); missing person (`Scan` stage: "do a scan" before the target even appears); photo missions (a different scan event per biome; the chosen one is picked by a `RequestedPhoto` condition group). The scanner itself (`m/simulation/scanning/scandatatable.MXML`) is a pulse: `PulseRange`, `PulseTime`, `ChargeTime` (cooldown), `AddMarkers`, `PlayAudioOnMarkers`, `ScanRevealDelay`, separate entries for tool/ship/vehicle (ship pulse range is ~150x the hand tool); results are markers with an audio ping. `ScanEvents` also have tables per context (space, planet, vehicle, tutorial, npc planet site, missions).

Implication: give missions one abstract "place query" object (kind + scope + filters + fallback) that the world resolves lazily to a concrete target, and keep marker presentation (label, icon, discovery sound, fuzzy vs exact, near/far text) as data on that object. Making the target need a scan pulse or a clue step turns "navigation" into a small activity.

#### 3. Feedback and reward presentation

- Rewards are always a stage, never a side effect: `GcMissionSequenceReward{Reward=<id>, Message}`; the DELIVER ends with TWO reward stages (a money/item draw `R_MB_*` then a standing step `MB_STAND_*`), plus a start-of-mission reward stage that hands over the item (also a reward-table entry, `R_DELMESSAGE` / `GcRewardMissionMessage` fires a named message to other missions). Two separate popups, not one lump.
- Reward item fields (`rewardtable.MXML`): `ForceSpecialMessage` (1840 occurrences) and `HideAmountInMessage` (1690) = per item choice of how loudly to announce; `Silent`; `LabelID`; `RewardMessageSubstanceForIcon`; `CentreMessage`; `OSDMessage` (127). So the announce style is data per reward, ranging from silent to special full-screen.
- Reward type palette (~150 types): money (min/max range, currency), specific product (1690 uses, by far the most), multi-item bundles (686), tech, recipe, ship, pet egg, "teach word" (a lore/language unlock, 70), wiki/guide topic (44, opens a guide page and centres it), "open page" (opens a shop page right after), standing, stat modify/increment, scan event (reward = start a new marker/mission, 160), mission (chain, 61).
- Board payout shape (`R_MB_LOW` etc.): one `SelectAlways` weighted draw whose weight is dominated by two money-like outcomes, with a long tail (~50 entries, many with low weight) of small odd items: food, tokens, crafting goods, fuel, blueprints, pet-ish oddities, ship-upgrade chits. Tiers (`MissionBoardRewardOptions` in defaultreality: ~11 pick lists from LOW up to MEGA, repeating an id biases it) keep the same shape but higher amounts; the first-ever board mission gets its own table (`R_MB_FIRST`). Abandon has an explicit penalty reward (`RewardPenaltyOnAbandon`, negative standing).
- Sound: `AudioEvent` on stages/options, `MessageAudio` on markers (`UI_NEW_DISCOVERY`, `UI_RECORD_UPLOADED`, `UI_SCAN_PORTAL`), `PlayMusicSting` Start/End, story music loops started and stopped by explicit `AudioEvent` stages (`MUS_STORYMODE_MUSICCUE_xx_LP` / `_LP_STOP`). Audio is referenced by id only; the audio dir holds no mission data. HUD message categories (Mission, Urgent, Info, Danger) tint and style the OSD lines. Mission and reward screens are driven off this, no UI code per mission.
- Small rewards feel good through: (a) never a single number, a draw with surprises in the tail; (b) a second parallel track (standing) with its own popup; (c) reward messages are separate stages after a short `Wait` and the End banner, so they land one after another; (d) reward can open the next thing (shop page, next mission, a new marker); (e) stats/lore points tick up silently during the mission (`ModifyStat`) so progress is felt before the payout.

Implication: make "reward" a typed, announceable step with presentation flags (silent, loud, with sound), give every mission two or three small payouts on different tracks, and draw loot from weighted tables with a long tail of oddities instead of fixed pay. Sound and banner ids belong on the beat, not in code.

#### 4. Framing and story

- Giver: `ForceInteractionType=MissionGiver` on the hand-in scan event plus `Participant` (Secondary1...) names who owns the thread; `MissionBoardOptions.Faction` list says which factions offer the template; `FactionClients` (defaultreality; pool of ~25 numbered client names per faction) fills "who is asking"; `FactionNames`, `FactionStandingIDs` per faction; globals cap how many missions one giver offers (`MaxNumMissionsFromMissionGiver`) and `NumMissionsPerSolarSystem`.
- Procedural text assembly (`GcNumberedTextList{Format key %d, Count}`, nothing else): per template `MissionTitles` (Count 10 to 20 variants), `MissionSubtitles`, `MissionDescriptions` (whole-text variants), AND `MissionProcDescriptionHeader` + `A` + `B` + `C` (3-part procedural description, each a pool of ~5 to 8 sentences, so 5x5x5 combinations per template). On top, `MissionNameFormats` + `MissionNameAdjectives` (~15) + `MissionNameNouns`, one set per mission type (name = format applied to adjective+noun). `PrefixTitle`, `UseScanEventDetailsInLogInfo` mixes the resolved target name into the log. Text variants are therefore shared pools per kind, filled slots from the resolved target and item.
- NPC dialogue at the board is an `AlienPuzzleTable` per mission: entries with `Text` (what the giver says), `TextAlien`, options (`Name`, response `Text`, `Cost`, `Rewards`, `Mood`, `AudioEvent`, `DisablingConditions`, `MarkInteractionComplete`, `SelectedOnBackOut`). Even delivery has a branch "you don't have the item" (`DELIVERY_NOITEM1`) with a mood and a different text, i.e. the failure path is authored too.
- Story arcs are the same machinery, hand-authored and chained: `MissionClass` Primary/ACT*_STEP1..13 chained by `NextMissionHint`/`StartMission`; `ChainedSecondary` (PIRATES1->2, OVERSEER1..10, EXOTUT, SCIENTIST, FARMER, WEAPGUY: a named character with 7 to 13 linked missions); `Milestone`, `Atlas`, `Seasonal` (~520 with log text overrides per season), `Settlement`. Story progress is stored as a numeric stat per arc (`CORE_LORE`, `PIRATES_LORE`, ... `statstoriesmissiontable.MXML`) set by `ModifyStat` inside stages, used as the arc's save pointer and as a gate for other missions.
- Authored feel inside generated missions comes from: the same beat vocabulary as story missions (banner, sting, comms, hint), per-faction client and standing, 3-part text, race flavour via `RequireInteractionRace`, a Communicator stage inside tier-2 board jobs (HIDE_SEEK_BRIBE: comms hail after boarding), and board missions that spawn a story ping (reward type scan event/mission). All board missions are hand-authored templates (the earlier note already infers procedural generation is never switched on).

Implication: keep the mission-structure data the same for generated and authored content, and make the "generated" ones feel authored through three data-only things: a named client with a faction, text from pooled parts filled with the resolved target, and a one-line giver reaction (including the failure branch). Story arcs are chains plus a per-arc progress counter, nothing more.

#### 5. Variety and depth

Kinds built on "go there and bring X" (all `npcmissiontable.MXML` unless noted, same state skeleton, different middle):
- Deliver / deliver hard (item from pool; seeded high chance of an ambush spawn en route: `Pirates{PirateSpawnType=CargoAttackStart}`, i.e. a mid-mission twist that is a condition-gated group inside the carry phase).
- Collect (substance/product by amount), `COLLECT1..4`.
- Pickup person or object from a hidden place (`MISSING_PERSON`, `HIDE_SEEK_BRIBE`, `HIDE_SEEK_TIMED`: GetInShip, then find-by-scan, then carry back; the "timed" one has `TimeoutOSD`).
- Repair (go to a broken thing, wait for `IsScanEventRepaired`, return), photo (per biome), feed, dig, fish, scan creatures/minerals/trees, raid factory/depot (3 difficulties), bounty (3 tiers with escalating combat), kill robots/predators/fiends.
- Space outpost postman (`spaceoutpostmissiontable.MXML` SO_POSTMAN, the nearest to our crates): a physical carry with live conditions: `GrabbingRecyclableOfType`, `Location`, `NumPhysicsObjectsInPhysicsWorld` (the objects were dropped), `NearScanEvent`, a "message" stage when done, and one reward stage that opens another target (the planet or station box) via `R_POST_DELIVERY` (a scan-event reward), then the board reward, a `SP_POI_MISSIONS` stat tick, and standing.
- Space outpost collect (SO_COLLECT_H1): looping nested watcher groups while carrying: inside POI, has substance, `MeltdownStatus` (a hazard timer), each condition swaps the message text, and a return-to-collector step; so the "carry" phase is reactive (different line when you are inside the zone, when the hazard is up, when you lose the cargo).
- Chains: PIRATES1->2->3 (clue per objective, space marker, forced `CreateSpecificPulseEncounter`, comms, three lore ticks), `BOUNTY_NEW1..3`.

Twists and optionals: seeded `PercentageChance` groups (twist only sometimes, seeded so reproducible), `Pirates`/`CreateSpecificPulseEncounter`/`KillEncounter` inside a carry, conditions that fork the text or markers (`HasProduct` false -> "no item" scan event instead of the normal one), `DoConsequencesIfNeverActivated` groups, side missions auto-offered via reward type mission, `Seasonal` and recurring ones (`recurringmissiontable.MXML`: daily/weekly via `missionschedulestable.MXML`, ~99 entries incl. D_/W_ chains and "DM_" dispatch missions). I found no explicit "optional objective" flag; optionality is done by branch groups plus extra rewards.

Depth by size: board missions have 2 objective-groups (do, return) median, max 4, about 13 leaf beats; space POI secondary 1.5 objectives, ~10 beats; chained clue missions 3 objectives, ~34 beats; core story primary ~2 objectives with ~15 beats per step and 13 steps per act; tutorial primaries up to 9 objectives.

Implication: add depth by adding a typed middle (carry with hazard/ambush, find-by-scan, interact-with-thing-then-return) to the same skeleton, not by new systems, and make the carry phase reactive (condition-switched messages, optional seeded ambush, physical-object checks). Chains of 3 to 13 small missions plus a progress counter give "story".

#### 6. Pacing and length

- No deadline field on board missions; timers exist as a few `Time` waits, `TimeoutOSD`, timed variants, real-time waits (`WaitRealTime`), schedules.
- Breathers: nearly every beat is bracketed by `Wait 0.5 to 2.5 s` (216 Wait stages in the core table alone, 100+ WaitForConditions), and `StartScanEvent` has its own delay `Time` (1 to 6 s) so the marker appears after the banner. Banners block (`WaitForMessageOver`). The delay between "task finished" and "payout" is on purpose.
- Length is mostly travel plus one nested step: board job = go there (search scoped Local/LocalOrNear, mostly same system), do a thing, return to the giver; hand-in scan event is another stop (the giver, not the pickup place). Mid-length missions chain 2 to 3 stops.
- Rhythm across missions: reminders by `Notify` timers, 4 slots of hint messages, `NextMissionHint` auto-offers the next chain step, rewards that start a follow-up (scan event, mission, open shop page) keep the player going; stage-level `RestartOnCompletion`/`Recurring` for repeatable daily and weekly.
- Difficulty gates length: the same template exists in easy/normal/hard variants (BOUNTY_EASY/MED/HARD, KILL_ROBOTS/MED/HARD, FACTORY_RAID/MED/HARD, DEPOT_RAID, DELIVER/DELIVER_HARD) with a higher reward tier and more squads/steps; board difficulty is a multiplier on required amounts (`gcgameplayglobals.global.MXML`).

Implication: pacing = the pause beats and delayed reveals between steps plus two stops minimum; a mission is long because the hand-in is a second place, the target must be found, and something can interrupt the carry. Scale by difficulty variants of one template, not by new content.

#### Short list of data-only pieces to copy structurally
(a) objective-group list with typed beats; (b) place-query object with fallback; (c) per-beat banner/sting/sound ids; (d) reward as stages, multi-track, announce flags, long-tail draw; (e) pooled 3-part text + named client/faction; (f) carry-phase watcher groups with seeded twists; (g) guide-watcher mini missions for hints; (h) chain via next-id + per-arc progress counter.

## Appendix C. Schedule I files

Source: `research/local/schedule-i`. Paths `S1/` = `Scripts/Assembly-CSharp/ScheduleOne/`. Method bodies, string literals and ScriptableObject values are stripped, so evidence is fields, enums, constants, class cuts and audio clip names (`Assets/AudioClip`). Prefabs are mesh-only `.glb`; no scene hierarchy or UI text is readable. Where a claim is inferred from names it says so.

#### 1. Beats from request to payment

Sequence, with what each beat touches (fields/classes):

1. **Offer arrives as a phone text.** `Customer.NotifyPlayerOfContract(ContractInfo, MessageChain offerMessage, canAccept, canReject, canCounterOffer)`. Message = `MessageChain` (list of strings, sent with `initialDelay` so it types in like a chat) via `MessagingManager.SendMessageChain(..., notify)`. Phone UI: `MSGConversation` has an unread dot (`unreadDot`), `HUD.UnreadMessagesPrompt`, `Phone vibrate single` / `Mobile_Cell_Phone_Ringing` clips. Player replies are canned `Response(text, label, callback)` buttons, not free text (`MessagingManager.ShowResponses`).
2. **Decision is a small mini-game, not a button.** Three responses: accept, reject, counter-offer (`AcceptContractClicked`, `CounterOfferClicked`, `CounterofferInterface` with product/quantity/price and a "fair price" hint via `GetValueProposition`). Reject has a relationship cost (`DEAL_REJECTED_RELATIONSHIP_CHANGE`). Accept/reject post a random canned player line (`Customer.PlayerAcceptMessages/PlayerRejectMessages`) and the customer plays a reaction (`PlayContractAcceptedReaction/RejectedReaction`).
3. **Pick when.** `DealWindowSelector` (clock face with `CurrentTimeArm`, four buttons Morning/Afternoon/Night/LateNight, `WINDOW_CUTOFF_MINS`). The player chooses a slot in a day that is cut in four 6-hour windows (`DealWindowInfo`). Contract becomes a tracked quest with `Expiry`, HUD entry (`QuestHUDUI`, `BopHUDUI`), journal entry, and a time label turning red under `CriticalExpiryThreshold`.
4. **Wait/travel is physical.** The customer NPC walks to the drop on their own (`CustomerAttendDealBehaviour`, `WalkSpeedMultiplier`, `CheckWarp`), so the player sees them arrive. `DeliveryLocation` has `CustomerStandPoint`, `LocationName`, `LocationDescription`. Late customers: soft start / hard start / end (`Customer.GetContractTimings`, `DEAL_ATTENDANCE_TOLERANCE`).
5. **Handover is its own screen** (`HandoverScreen`, modes Contract/Sample/Offer): expectation entries (`ExpectationEntries`), 4 customer slots you drag items into, `SuccessLabel`/`ErrorLabel`/`WarningLabel`, a colour gradient (`SuccessColorMap`) for match quality, a success chance for offers. Outcome enum Cancelled/Finalize. Player must put the right things in slots, so the handover is a hands-on act with a live quality read-out.
6. **Payment.** `Contract.SubmitPayment(bonusTotal)`; payment is cash that physically appears (`MoneyManager.cashChangePrefab`, `CashSound`, `CashSlot`, `$10 Pickup` / `CashChange` prefabs), HUD cash slot shows a `changeDisplay` delta. Bonuses are named line items (`Contract.BonusPayment{Title, Amount}`).
7. **Deal completion popup** (`DealCompletionPopup.PlayPopup(customer, satisfaction, relationshipDelta, basePayment, bonuses)`): coroutine lerps a payment counter (`paymentLerpTime`), then a satisfaction value coloured by `SatisfactionGradient`, then a `RelationCircle` (portrait + notch) animating the relationship change, `BonusLabels[]` listed one by one, `SoundEffect` = clip `DealCompleted_SFX_*`. `Quest.PlayQuestCompleteSound`/`QuestComplete_SFX` is separate.
8. **Follow-up.** Satisfaction changes relationship (`CustomerSatisfaction.GetRelationshipChange`), may trigger `ContractWellReceived(npcToRecommend)` which recommends another customer/dealer/supplier (`RecommendCustomer/Dealer/Supplier`), and feeds `ContractReceipt` and the end-of-day `DailySummary` (items sold, money by player/dealers, XP). Next offer is gated by `DEAL_COOLDOWN`, per-customer weekly order count and order days.

Implication: Treat a delivery as 6-8 short, separately-staged beats (text in, choose reply, choose time, watch the other party arrive, hand-on handover with live quality read-out, paced payment tally, relationship/next-hook), each with its own sound and animation; our current "take, carry, drop, paid" collapses all of them into one.

#### 2. Navigation

- Everything that matters is an `POI` (`Map/POI.cs`: `MainText`, `TextShowMode`, `AutoUpdatePosition`, UI icon in `PoIContainer`); NPCs get `NPCPoI`, drops get `DeadDrop.PoI`, quests get `QuestEntry.PoILocation + AutoCreatePoI + AutoUpdatePoILocation`. Contract updates its pin via `Contract.UpdatePoI()` so the active job's drop shows up on the map by itself.
- Phone `MapApp`: zoom/scroll (`FocusPosition`, `LabelScrollMin/Max` fade labels by zoom), main map sprite plus separate tutorial map. Region labels and rank-locked regions (`MapRegionData`).
- First-person HUD: `CompassManager` (24 notches, `Element` per tracked target with `DistanceLabel` shown only inside `DISTANCE_LABEL_THRESHOLD`; `QuestEntry.compassElement`). Quests can be tracked/untracked (`SetIsTracked`, `TrackOnBegin`), HUD shows tracked ones only.
- Drop identification is human: `DeliveryLocation.LocationName` + `LocationDescription` ("where exactly") plus `CustomerStandPoint`, shown in the message/quest text; a handful of named spots per region (`RegionDeliveryLocations`, `GetRandomUnscheduledDeliveryLocation` so two contracts never share a spot).
- Worldspace help: `WorldspacePopup` (icon over a thing, `Range`, `ScaleWithDistance`, `DisplayOnHUD` so off-screen ones sit on the HUD edge), `InteractableObject` prompts with `EInteractableState` (default/invalid/disabled/label) and `HintDisplay` timed hint strip.
- `NPCEnterableBuilding`, `ParkingLot` etc. give places names; region names are on-screen labels, and entering a new region shows `RegionUnlockedCanvas` (name, description, image).

Implication: Give every job target a pin that is created and moved by the job itself, reachable three ways (map, compass strip with distance only when near, a named place with a one-line description), and make places named and few rather than coordinates.

#### 3. Reward feedback

Layers, each with its own surface and sound:

- **Money**: green cash colour constants (`MONEY_TEXT_COLOR`), cash physically spawned, cash slot in HUD with floating delta, online balance in a second colour; sound `cash-register-kaching-sound-effect`, `coin_drop_01..03`; weekly/ledger (`Transaction`, `lifetimeEarnings`).
- **Deal result**: `DealCompletionPopup` (see 1.7) with sequential count-up, satisfaction gradient, relationship wheel, itemised bonuses, jingle `DealCompleted_SFX_mixnmaster1`.
- **Relationship**: bounded 0..5 (`NPCRelationData`), shown as `RelationCircle` (portrait coloured by dependence, notch needle); relationship thresholds unlock things (recommend, dealer/supplier meetups at `MeetupRelationshipRequirement`).
- **New customer**: `NewCustomerPopup` (title + `Entries[]` + `SoundEffect` `CustomerUnlocked_SFX_*`), queued so it does not collide with the deal popup.
- **XP/rank**: `XPAmounts` (small flat constants per action; sample, deal, escape, discovery, graffiti all give XP), `Rank x 5 tiers`, `XP_PER_TIER_MIN/MAX`. XP gain is silent-ish; rank-up is a full-screen event (`RankUpCanvas`: `OldRankLabel`->`NewRankLabel`, `RankUpAnim`, `UnlockedItems[]` list from `LevelManager.Unlockables` (Rank/Title/Icon), `ProgressSlider`, `BlipSound`, `ClickSound`, `RankUp_SFX`), queued until after sleep (`QueuePostSleepEvent`) so it lands at a calm moment. `RegionUnlockedCanvas` does the same for a new area.
- **Toasts**: `NotificationsManager.SendNotification(title, subtitle, icon, duration)`, max 6 stacked, shared sound.
- **Day end**: `DailySummary` (product entries, player earnings, dealer earnings, XP) on sleep.
- UI sounds are plentiful: `glossy_click_*`, `glossy_success_13`, `success_elegant_19`, `general_click_03`, `Tick 1..3`, `computer_data_blip_*`.

Implication: Present one payout as a short staged sequence (counter ticks up, bonus lines appear one by one, a relationship/standing meter moves, a jingle) and queue bigger moments (rank-up, region/new-client unlock) to a calm point; every currency (cash, relationship, XP) gets its own colour, surface and sound.

#### 4. Framing and story

- **Cast is large and named, each a class in `NPCs/CharacterClasses`** (about 80 entries); ~7 named in `NPCs/` root. Roles: customers (many), dealers, suppliers, a fixer, arms dealer, estate agent, vehicle salesman, skateboard seller, cop, cartel goons, plus oddities (`SewerGoblin`, `SewerKing`, `SchizoGoblin`). Per-role dialogue controllers (`DialogueController_*`, e.g. `_Dealer`, `_Supplier`, `_Fixer`, `_Police`) and a handful of unique ones named per character.
- **Personality is data**: `CustomerData` (preferences, `Standards` VeryLow..VeryHigh, order times, `PreferredOrderDay`, weekly spend, `DependenceMultiplier`, `CallPoliceChance`, `CanBeDirectlyApproached`), plus voice: `VODatabase` / `VODatabaseEntry`, `EVOLineType` (Greeting, Thanks, Annoyed, Angry, Scared, Question, Think ...) and voice sets by archetype (clips "Crackhead Question", "Hippie Question", "Timid", "Monotone", "Female1/2"). `VocalReactionDatabase` = key -> random reactions.
- **Conversation shape**: `DialogueController` has `Choices` (each with priority, enable/valid checks that return an `invalidReason` string), `GreetingOverrides` (offer waiting, request pending), weather-aware greetings (`RainyGreetingThreshold/Chance`), `GreetingCooldown`. Same NPC says different things by state.
- **Story is code**: about 25 `Quest_*` classes forming a tutorial/progression chain (names read like chapters: first deal, grow, mix, gear up, employees, warehouse, laundering, cartel), each tracked quest with entries (`QuestEntry`, `AutoComplete`, `PoI`), `CompletionXP`, a mentor NPC who talks through the phone (`UncleNelson` referenced in `Quest_GettingStarted`, `Quest_WelcomeToHylandPoint` has a scripted RV explosion with camera-look timing), and a `Quest_DealForCartel/DefeatCartel` antagonist line. Cutscenes: `IntroManager`, `EndCutscene`.
- **Messages carry tone**: customers' offers are authored chains; the player's replies are canned lines with random variants; a `Supplier` sends debt reminders (`_repaymentReminderSent`), dealers send cash reminders (`CASH_REMINDER_THRESHOLD`); contract expiry sends reminder/expired notifications (`ShouldSendExpiryReminder`).
- Dead drops (`DeadDrop`: name + description + region, `DeaddropQuest`) are a second delivery flavour: leave/collect in a hidden spot, with a quest and a light that marks it.

Implication: Put a few named, voiced characters behind the jobs (one-line trait set, a handful of reactive greeting variants, a standing relationship) and a short mentor-led quest chain that teaches the loop; the contracts themselves can stay data, the intro chain is where story lives.

#### 5. Variety and depth

What differs between two deals:

- **Who**: customer with taste (`CustomerAffinityData` per product type, `PreferredProperties`, `Standards`, order day/time, weekly spend), plus a social graph (`Connections`) so new clients come via people.
- **What and how much**: `ProductList.Entry{ProductID, Quality, Quantity}`; rank scales order size (`GetOrderLimitMultiplier`); `MaxOrderQuantityPerProduct`.
- **Quality**: `EQuality` with tolerance (`QualityTierTolerance`, dealer `Negative/PositiveQualityTolerance`), excess goods count half (`ExcessProductsMatchSumMultiplier`); enjoyment mixes affinity, effects, quality (`AFFINITY/PROPERTY/QUALITY_MAX_EFFECT`).
- **Price negotiation**: asking price per product, counter-offer with fair-price hint, success chance (`GetOfferSuccessChance`).
- **When**: four windows, `OrderTime`, `PreferredOrderDay`, `OFFER_EXPIRY_TIME_MINS`, `DefaultExpiryTime`, travel time min/max, curfew (`CurfewManager`, `ScheduleGroupPair` Normal/Curfew).
- **Where**: six named regions (`EMapRegion`), per-region delivery spots, regions locked by rank.
- **Risk**: law intensity per weekday (`LawActivitySettings` Monday..Sunday, `LE_Intensity`, daily drain), patrols (`FootPatrol`, `VehiclePatrol`), checkpoints (`RoadCheckpoint`, `CheckpointManager`), crimes list (drug trafficking, possession by severity, evading ...), investigation -> wanted -> arrest (`CrimeStatusUI` masks Investigating/UnderArrest/Wanted), body search, customer can call police (`CallPoliceChance`), `Heatmap`, cartel ambush/steal-drop/rob-dealer (`CartelActivities`, `InfluenceRequirement`), escape XP.
- **Escalation of the loop around the delivery**: you can delegate (dealers `AssignDealer`, cut and cash reminders), samples to unlock (`ESampleFeedback`), direct approach, product requests in the street (`RequestProductBehaviour`: NPC approaches, follows you, asks again after `TicksBeforeAskAgain`).

Implication: Cheapest variety comes from a few orthogonal knobs on one template (who asks, what and how good, by when, from which place, how watched), with at least one quality-style scoring term and one risk term that can bend the plan; time window choice and a counter-offer are the two decisions that make a trip a choice.

#### 6. Liveliness

- **NPCs live on schedules**: `NPCScheduleManager`, schedule events (`NPCEvent_Conversate`, `_Sit`, `_StayInBuilding`, `_LocationDialogue`, `NPCSignal_UseATM`, `_UseVendingMachine`, `_WalkToLocation`, `_DriveToCarPark`), behaviours like `SmokeBreakBehaviour`, `ConsumeProductBehaviour`, `GraffitiBehaviour`, `BagTrashCanBehaviour`, `PickUpTrashBehaviour`; they enter buildings, use umbrellas (`UseUmbrella`) when wet (`_wetness`, `NPC_WET_RATE`), headlights on at night.
- **Reactions**: `NPCResponses_Civilian` / `_Police` / `_CartelGoon` (flinch, cower, flee, panic, call police), `NPCAwareness`, `VOEmitter` plays `EVOLineType` lines (Alerted, Scared, Annoyed, Question, Thanks, Snore); a pool of Alerted1..9 / Question1..7 clips; idle conversation clips "X & Y Arguing" (paired NPCs, with a muffled variant behind glass).
- **Audio layers**: music by time of day (`MorningAmbientTrack`, `MiddayAmbientTrack1/2`, `AfternoonAmbientTrack1/2`, `NightAmbientTrack1/2`, `SneakyAmbientTrack`), police music in two intensities each with start + loop (`Police_Light*`, `Police_Loud*`, `PursuitMusicTrack`), `MusicManager.SetMusicDistorted` (state effect), world radios (`Chill Wave RADIO`, `Radio static`), shops with ambience (`coffee_shop_ambience`, `Ambience_Laundromat`), `AudioZone`/`AudioZoneTrack`/`AudioZoneModifierVolume` (area-driven loops), `AmbientOneShot` (random one-shots), `TimeOfDayVolumeController`, `FoliageRustleSound`, `HeartbeatSoundController`, `SpottedTremolo` (cue when seen).
- **Weather and light**: `Weather/` rain, clouds, thunder, puddles, day-night phases; NPC greetings change in rain.
- **Small physical toys**: skateboard, graffiti spray surfaces, TV, vehicles, vending machines, trash that is picked up; everything is interactive via `InteractableObject` labels.
- **Rhythm anchors**: sleep ends the day with `DailySummary` then queued rank-up/region events; `TimeManager` events per minute/hour/day.

Implication: A small town feels alive when there is time-of-day music, a handful of schedule behaviours per NPC type, voiced one-liners from a pool and a place-based ambience loop; most of this is cheap layering of short clips on a clock, not simulation.

#### Cross-cutting (to carry)

- Staging over mechanics: the same underlying pay is surfaced through message, reply choice, wait, handover, popup, relationship wheel, XP and day summary.
- Decisions inside the flow (reply, counter-offer, time window, what goes in the slots) so the player is never only a courier.
- Few named people with taste and voice, rather than anonymous clients.
- Unlock events are queued and given a full-screen moment at calm points.
- Risk is an overlay with its own UI/music, switched on by simple state, so deliveries differ in pressure.

Caveats: no UI text, numbers or scene layout were readable (stripped); popup order/timing and "sequential" are inferred from field names and coroutine locals (`paymentLerpTime`, `satisfactionLerpTime`, `BonusLabels[]`).

## Appendix D. Public write-ups

Method note: found via web search; summaries of the pages I could fetch (marked [fetched]) are verified, the others rest on search snippets and prior knowledge of the talk/article. Two fetches (Starsector blog, Shadows of Doubt devblog) failed (402/403), so those entries are about where to look, not verified text. Primary sources are preferred; forum threads are marked as such.

#### 1. What makes delivery / hauling engaging

1. Death Stranding, design discussion (Game Developer) [fetched]
   https://www.gamedeveloper.com/design/a-design-discussion-on-death-stranding
   Delivery gets depth from four coupled systems: package weight/arrangement, terrain traversal, load balance (bad stacking makes you fall, damages cargo) and shared structures. The route itself is the puzzle; the cargo constrains it. Lesson: make the carried thing change how you move.

2. The director, "Death Stranding's Design Philosophy" (GDC 2020 session page)
   Frames the whole game as "ropes not sticks": delivery exists to connect places and people. Walking is made interesting (balance, stamina) rather than skipped. Lesson: give the haul a meaning (who is waiting) and make the in-between physical.

3. Death Stranding, community structures/likes (see source 1 and the GDC AI postmortem page)
   https://gdconf.com/news/see-death-strandings-ai-deconstructed-gdc-2020
   Other players' ladders/bridges soften the trek and "likes" give a small constant thank-you. Caveat noted in source 1: help can cheapen personal achievement. Lesson for co-op: let one player's prep (marked route, placed ramp, tow cable) pay off for the other.

4. SnowRunner, lead designer interview (The Sixth Axis)
   https://www.thesixthaxis.com/2020/05/05/snowrunner-interview-trucks-toughness-sequel-vitaliy-yaruta/
   Terrain is the antagonist; progression gives more tools (winch, trucks, upgrades) to solve it instead of making the world easier. Large maps had to be filled with objectives and playtested end to end, no scenery-only areas. Lesson: distance is only interesting if the ground resists, and if you can fail and recover.

5. SnowRunner, devs on what is new (GameSpot)
   https://www.gamespot.com/articles/snowrunner-devs-explain-whats-new-in-the-open-worl/1100-6476014
   Contracts are a thin wrapper; the fun is route reading, getting stuck, improvising with tools, then the relief of arrival. Lesson: planned risk plus recovery tools make a slow trip dramatic.

6. Euro Truck Simulator 2, why people play (reviews, Bonus Stage)
   https://www.bonusstage.co.uk/archives/130032
   Appeal is a weighty driving model, scenery, radio, weather and a slow growth loop (own truck, hire drivers); "a screensaver for the brain". Lesson: if you choose the calm genre, the trip itself must be pleasant (sound, weather, vehicle weight), and money is mostly a growth ladder.

7. Elite Dangerous trading, why it is called boring (Frontier forum threads, player voice, not a primary source)
   https://forums.frontier.co.uk/threads/improving-trading-through-contracts-player-influence-and-automation.111892
   Recurrent complaint: best profit is the same back-and-forth route, the player has no effect on the economy, no story around cargo, trading is a grind to afford the "fun" career. Lesson: A-to-B with only a number as reward collapses into route optimisation; add players' influence, variation and consequences.

8. Star Citizen 4.0 economy overhaul feedback (RSI Spectrum, community thread)
   https://robertsspaceindustries.com/spectrum/community/SC/forum/4/thread/4-0-preview-feedback-economy-overhaul
   CIG wants hauling to be physical (load, unload, protect) but players report a list of ~35 near-identical contracts, rewards decoupled from risk. Lesson: physical handling of cargo is not enough; variety in contract kind, risk and reward fit matters more than count.

9. Hardspace: Shipbreaker, Early Access lessons (Game Developer) [fetched]
   https://www.gamedeveloper.com/game-platforms/hardspace-shipbreaker-is-taking-flight-here-s-what-its-devs-learned-in-early-access
   Stripped-down onboarding (start with small pieces, learn zero-g tools step by step) and iteration on player feedback; mood alternates between calm work and tense hazards. Lesson: a job is a sequence of small skill moments with a clear "done" state, not one action.

10. Lethal Company, risk/reward overview (Giant Bomb wiki, secondary)
    https://www.giantbomb.com/lethal-company/3030-90069/
    Scrap value sits in dangerous places, inventory is tiny, a quota and clock force greed decisions, crew has to decide together when to leave. Lesson: hauling is fun when carrying capacity, time and danger make you choose what to leave behind. (Weak source; a developer talk would be better, none found.)

Not found: a good primary write-up on Schedule I design. Skipped rather than guess.

#### 2. Reward feedback and "juice"

11. "Juice It or Lose It", GDC Europe 2012
    https://www.youtube.com/watch?v=Fy0aCDmgnxg (talk; background summary: https://www.psu.com/news/why-winning-feels-so-good-reward-design-from-arcades-to-balatro/)
    Same rules, much more feedback: every action gets animation, sound, particles, tiny delays. Lesson: feedback is layered on moments (pick up, load, take-off, delivery), not added in one summary screen.

12. "The Art of Screenshake", a Vlambeer developer
    https://www.youtube.com/watch?v=AJdEqssNZ-U (overview: https://infovore.org/?p=5275)
    Step by step list of cheap tricks (camera kick, hit pause, sound layering, anticipation, permanence). Lesson: add feedback in many small independent layers and judge each by feel; for a ship, thrust, landing gear contact and cargo thump are the "gun shots".

13. Juice overload warning (Wayline)
    https://www.wayline.io/blog/juice-overload-sensory-feedback-hurts-gameplay
    Counter-view: too much effect hides the information. Lesson: tie intensity to importance (small crate = tick, finished job = fanfare) so the scale carries meaning.

#### 3. Navigation as gameplay

14. Breath of the Wild, "Change and Constant" (GDC 2017) and analysis (GMTK)
    https://www.gdcvault.com/play/1024562/ and https://gmtk.substack.com/p/how-nintendo-solved-zeldas-open-world
    "Triangle rule": silhouettes hide the next thing, big landmarks give orientation, you choose your own waypoint. Lesson: for small planets, place a few readable landmarks and give each pad a visual identity, so a player can say "past the red tower".

15. Outer Wilds, curiosity-driven exploration (GDC 2020 session)
    https://www.gdconf.com/news/attend-gdc-and-learn-how-outer-wilds-nailed-curiosity-driven-game-design
    No missions; information gives conscious choices, the world is the clue. Lesson: navigation becomes gameplay when finding the place is a small discovery (a radio signal, a landmark, a bearing) rather than a HUD arrow.

16. SnowRunner (source 4) for route choice
    Route choice has teeth when alternatives differ in cost and risk (short and steep vs long and flat), and the map shows terrain only after scouting. Lesson: let the map be partial, revealed by flying or scanning.

#### 4. Making procedural missions feel authored

17. Starsector, blog on writing and missions (Fractal Softworks), plus wiki on Contacts
    https://fractalsoftworks.com/?p=4994 and https://starsector.wiki.gg/wiki/Contact (fetch failed; from search snippets)
    Mission givers are persistent contacts you meet in a bar and who return with jobs; hand-written text shells with procedural slots. Lesson: a small cast of repeat characters with a face and a voice beats a large anonymous board.

18. No Man's Sky Mission Board structure (fan wiki, secondary)
    https://nomanssky.fandom.com/wiki/Mission_Board
    Board is a fixed pattern: slots per faction, low rank first, higher rank unlocks as standing rises, standing is the reward. Lesson: reputation with a named faction gives a number its own story and unlocks better jobs.

19. Shadows of Doubt devblogs (ColePowered)
    https://colepowered.itch.io/shadows/devlog/78044/shadows-of-doubt-devblog-15-moving-in-the-citizens (fetch failed; snippet only)
    Cases are built from a simulated world: citizens with jobs, homes, habits; clues are real facts. Side jobs are one-offs with few details, you may drop them without penalty. Lesson: generate the story from a small simulated state (who ordered, why, who is the rival), not from random text strings.

20. Mount & Blade quests, critique (RPG Codex review, secondary)
    https://rpgcodex.net/article.php?id=6589
    Randomly generated quests of a few fixed types read as filler and the most common one repeats. Lesson: fixed templates with random numbers feel like errands; vary the situation (complication, twist), not only the names.

#### 5. Co-op specifics

21. Sea of Thieves, cooperation beyond doing the same thing (Road to VR analysis)
    https://www.roadtovr.com/7-lessons-sea-of-thieves-can-teach-us-about-great-vr-game-design/2
    Ship has more jobs than players (steer, sails, anchor, leaks, lookout); the helmsman cannot see, so someone calls the way. Lesson: split the haul into interdependent jobs (pilot, loader, spotter) with an information gap, and communication becomes the fun.

22. Sea of Thieves, how Rare's co-op works (Kotaku)
    https://kotaku.com/how-rares-chaotic-co-op-pirate-game-sea-of-thieves-work-1782033340
    Emergent stories come from shared vessel, shared loot at risk and a loose structure. Lesson: let cargo be shared at risk (a crate falling out is everyone's problem) and leave room for improvisation.

23. Deep Rock Galactic, developer interview (Unreal Engine)
    https://www.unrealengine.com/developer-interviews/guns-gold-and-glory-in-the-caverns-of-deep-rock-galactic
    Four classes with distinct tools, one main and one optional side objective per mission, the extraction/haul as shared final phase. Lesson: role-differing tools plus an optional greed objective create natural roles and arguments, even in a short mission.

#### Patterns that recur

- The trip must be the content: physical constraints on movement (weight, balance, terrain, fuel) turn the same A-to-B into a puzzle; a flat path with a number at the end is "a test".
- Carried cargo should change the vehicle or body (handling, stability, noise), so each crate makes a difference.
- Tools that solve problems (winch, ramps, tow cable) beat difficulty sliders; progression is more options, not an easier world.
- Risk with recovery: failure should be recoverable and funny (crate lost, ship stuck), not just punitive.
- Feedback lives at many small moments and scales with importance; the delivery needs an arrival ritual (counter ticking, sound, a person reacting), not only a balance change.
- Money alone is a weak reward; reputation with a named faction or person, unlocks (new ship parts, harder jobs) and visible world change carry more.
- Navigation works through landmarks, partial maps and discovered bearings; a marker-only HUD removes the activity.
- Few recurring named characters with a voice and memory beat a large board of anonymous jobs; hand-written shells plus procedural slots and one twist per job.
- Generate jobs from small world state (who needs it, why, rival, deadline) so text and gameplay agree, and vary the situation, not only numbers.
- Calm genres (ETS2, Shipbreaker) rely on sound, weather, weight and a rhythm of calm and tension; the dull bits must be pleasant by themselves.
- In co-op, give interdependent roles with information gaps (pilot cannot see, loader can) and shared risk; extra optional greed objectives create disagreement and stories.
- Variety of contract type (rush, fragile, secret, passenger-ish, rescue) matters more than quantity; many same-looking contracts read as filler.
