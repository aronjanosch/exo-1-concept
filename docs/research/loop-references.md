# Loop references: jobs, cargo, money and unlocks in other games

Research note, opened 2026-10-09, prepared for the grilling of milestone D (epic #37 in `exo-1`, `docs/CORE-LOOP.md`). It describes how three shipped games build jobs, cargo, payment and progression, lists public write-ups from further games, and lays out options for each open question of the epic. This is not a proposal, not an approved design and not a decision; the initiator decides.

Same rule as `star-citizen-datamining.md` and `no-mans-sky.md`: read the *structure and the knobs*, not their values. Class, field, enum and file names below are quoted only as evidence of how a system is cut. None of them is a suggestion for an EXO-1 name, value or text. Numbers are left out unless one is needed to explain what a knob does; then it is marked **[from file]**. Readings that come from names rather than data are marked *(inference)*.

Local sources (all under `research/local/`, gitignored, never committed):

- `schedule-i/`: decompiled Unity project. Method bodies are stripped and no ScriptableObject values are in the dump, so only schemas, enums and constants can be read. Code paths below are written `S1/…` for `schedule-i/Scripts/Assembly-CSharp/ScheduleOne/…`.
- `sc-logistics/`: Star Citizen DataCore XML, git HEAD "4.7.2 build 11674325", branch `PU`. The mission, contract, reputation, faction, shop and cargo folders were added to the sparse checkout on 2026-10-09; what the data still lacks is in section 2.0.
- `nms/exml/`: No Man's Sky records as MXML; how they were unpacked is in `no-mans-sky.md` section 2.

## The short answer

- All three games use **one lifecycle for every job** and build variety from a small set of templates plus data pools, not from a full simulation. Schedule I runs contracts on the quest state machine; No Man's Sky builds board missions from hand-authored templates filled from item and reward pools; Star Citizen fills generic templates from generators (one career contract per job, with a standing band and cargo orders).
- **Pay is graded, not binary** wherever it can be read: Schedule I scores what is handed over against what was asked; Star Citizen pays partial delivery in percent bands with separate reputation multipliers and fails on the deadline; No Man's Sky draws board pay from tiered pick lists.
- **Unlocks hang on the unlocked thing** (a rank requirement on the region or shop item) in Schedule I; No Man's Sky adds explicit unlock trees and reputation-gated shops; Star Citizen gates by standing bands per career scope, plus once-only intro contracts and rehire contracts sealed by completion tags; standings themselves carry no unlock payload.
- **Demand is person- or contract-driven in the games that work for short loops**: Schedule I's customers, Sunless Skies' prospects. No Man's Sky has price multipliers per system class but switched its per-sale drift table off.
- For route events the clearest public models are Left 4 Dead's intensity phases, RimWorld's storytellers, and Deep Rock Galactic's one-warning-plus-one-anomaly modifiers with shown hazard pay.

## 1. Schedule I

### 1a. Jobs: one state machine, three layers of contract data

- Contracts are quests: `Contract : Quest` (`S1/Quests/Contract.cs`, `S1/Quests/Quest.cs`). States in `EQuestState` (`S1/Quests/EQuestState.cs`): inactive, active, completed, failed, expired, cancelled. Transitions are virtual methods (`Begin`, `Complete`, `Fail`, `Expire`, `Cancel`, `End`), each with a network flag.
- Expiry is configured per quest (`ConfigureExpiry(bool, GameDateTime)`), with reminder hooks and a visibility enum for the timer (`EExpiryVisibility`: always, only when critical, never). Contracts override UI and journal visibility, so they look different from story quests while sharing the lifecycle.
- Three layers:
  1. **Offer** `ContractInfo` (`S1/Quests/ContractInfo.cs`): payment, product list, delivery location id, delivery window, expires flag and duration, pickup schedule index, counter-offer flag; it also fills the phone message template.
  2. **Live job** `Contract`: adds customer, dealer, accept time, a list of bonus payments (title + amount) and the payout call.
  3. **Save** `ContractData : QuestData` (`S1/Persistence/Datas/ContractData.cs`): customer id plus the offer fields.
- An order line is `ProductList.Entry{ProductID, Quality, Quantity}` (`S1/Product/ProductList.cs`): "N units of X at quality Y or better".
- Time windows: the day is cut into a fixed number of deal windows (`EDealWindow`, `S1/Economy/DealWindowInfo.cs`); the player picks one when accepting. A deal has a soft start, a hard start and an end (`Customer.GetContractTimings`), with an attendance tolerance constant.
- Generation runs per customer on the minute tick (`S1/Economy/Customer.cs`): should-try → pick a weighted product among those the player has listed for sale (`ProductManager.ListedProducts`) → pick a free delivery location in the region (`MapRegionData.RegionDeliveryLocations`) → offer. Unanswered offers expire. If the customer is assigned to an NPC dealer, the offer goes to the dealer instead.
- Every finished contract is logged as a receipt (`S1/Economy/ContractReceipt.cs`): who completed it (player, player's dealer, rival), customer, time, items, amount paid; other systems query receipts by region and age.
- Contracts are spawned prefabs (`S1/Quests/QuestManager.cs`); story quests are about 25 separate classes, i.e. story is code, not data.

### 1b. Customers

- Per-customer data is a ScriptableObject `CustomerData` (`S1/Economy/CustomerData.cs`): affinity per product type, preferred effects, weekly spend range, orders per week range, preferred order day and time, a quality standard (`ECustomerStandard` mapped to `EQuality`), whether they can be approached directly, a mutual-relationship requirement for unlocking, police-call chance, addiction knobs.
- Runtime state (`S1/Persistence/Datas/CustomerData.cs`): affinities that drift, dependence, time since last deal, offered and completed deal counts.
- Enjoyment of a delivery is a weighted mix of three terms (type affinity, liked effects, quality vs. standard); the three max-effect constants on `Customer` sum to one *(inference: weighted sum)*.
- Relationship (`S1/NPCs/Relation/NPCRelationData.cs`) is a bounded value plus a social graph (`Connections`). Unlock paths for new customers: a sample handover (`ESampleFeedback`: wrong product, wrong quality, correct; success from a curve asset, `ProductManager.SampleSuccessCurve`), a recommendation from a satisfied customer, or a direct approach. Each region starts with its own NPCs (`MapRegionData.StartingNPCs`).

### 1c. Deals and prices

- Counter-offer: the player may change product, quantity and price; the server evaluates it against a value proposition (`GetValueProposition`), the UI shows a fair price (`S1/UI/Phone/CounterofferInterface.cs`).
- Handover (`S1/UI/Handover/HandoverScreen.cs`): items are ranked against each requested line (`Contract.GetProductListMatch`); excess goods count partially toward the match **[from file: a half-weight multiplier]**; the result is a satisfaction value that changes the relationship (`CustomerSatisfaction.GetRelationshipChange`).
- Price: `ProductDefinition` (`S1/Product/ProductDefinition.cs`) has a base price and market value; each `Effect` asset (`S1/Effects/Effect.cs`) carries a flat change, a multiplier and a base-value fraction; `ProductManager.CalculateProductValue` combines them. The player sets the asking price per product, clamped to a range.
- Buy side: `StorableItemDefinition` (purchase price, resell multiplier), `ShopListing` (override price, limited stock, restock rate, visibility condition).
- Money sinks: employee daily wage and signing fee (`S1/Employees/Employee.cs`), dealer signing fee and sales cut (`S1/Economy/Dealer.cs`), supplier debt (`S1/Economy/Supplier.cs`), fines (`S1/Law/PenaltyHandler.cs`), hospital bill. Property is a one-time price, no rent.
- Two kinds of money: cash and bank balance; cash moves to the bank through a weekly-capped ATM or laundering businesses (`MoneyManager`, `Business.LaunderCapacity`).

### 1d. Unlocks and progression

- Rank × tier (`ERank`, `FullRank`, tier count constant); XP per tier grows with rank (`LevelManager`, min and max constants). XP sources: quest completion XP, successful counter-offers, small actions.
- Rank also scales order size (`LevelManager.GetOrderLimitMultiplier`).
- Unlock requirements sit on the unlocked thing, not in a central table: `MapRegionData{UnlockedByDefault, RankRequirement, StartingNPCs, RegionDeliveryLocations, AdjacentRegions}` (`S1/Map/MapRegionData.cs`, regions unlocked on rank-up), `StorableItemDefinition.RequiredRank`, a rank gate on the dark market, on cosmetic options.
- The rank-up screen reads a display-only list (`LevelManager.Unlockables`), filled at runtime *(inference)*.
- Supplier services unlock at relationship thresholds (meetups, then deliveries).

### 1e. Progress per round: the day

- `TimeManager` (`S1/GameTime/TimeManager.cs`): events per minute, tick, hour, day, week, sleep start/end and time skip; systems catch up on skip. A game day is a short real-time cycle *(inference from the constant)*.
- Per day: `DailySummary` (`S1/UI/DailySummary.cs`) shows items sold, money by player, money by dealers and XP gained, on sleep. Daily drains: addiction, police intensity, wages.
- Per week: each customer has a weekly budget and order count, spread over order days (`GetOrderDays`, `WeeklyPurchaseRecord`) *(inference)*.

### 1f. Production, briefly

Growing (`S1/Growing/Plant.cs`, additives), cooking (`StationRecipe` asset: ingredients, product, cook time, temperature tolerance, quality method) and mixing (`MixingStation`, `MixerMap`) produce products with a quality and a set of effects. Those two outputs are exactly what the customer evaluation reads, so production choices drive price, enjoyment and relationship.

### 1g. What is data and what is code

- Assets: customer profiles, product and effect definitions, recipes, seeds, additives, packaging, shop items with price and rank lock, NPC presets, dialogue and string databases.
- Serialized on scene components: regions, delivery locations, property prices, shop listings, quest fields.
- Hard-coded: deal windows, expiry times, enjoyment weights, XP curve, rank-to-order multiplier, fines, cooldowns, all enums, every story quest.

## 2. Star Citizen

### 2.0. What is readable

The local checkout is a sparse, partial clone of the public repo. On 2026-10-09 the folders for contracts, missions, reputation, factions, shops and cargo were added to it (`git sparse-checkout add …`, fetched from the public repo, still only under `research/local/`). Readable now: `contracts/`, `missionbroker/`, `missiondata/`, `missiontype/`, `missiongiver/`, `missionfailureconditions/`, `missionscenarios/`, `contractawardconfig/`, `reputation/`, `reputationvaluesettings/`, `factions/`, `Shops/`, `globalshopparams/`, `globalcargoloadingparams/`, `inventorycontainers/`, `cargomanifest/`, `commodityconfiguration/`, `refiningprocess/`, `playertrade/`.

Not in the data at all (presumably in code): the money formula of a calculated reward, per-SCU or distance pay, the shop price curve, refining times and yields, and the per-commodity records (`commoditytypedatabase/` and `resourcetypedatabase/` are only index records whose targets are not in the repo). The shop JSON looks like an older snapshot *(inference: it still names a station that was removed)*.

### 2a. Two contract systems side by side

1. **Legacy broker entries**: one flat record per mission with fixed reward and its own objectives (`missionbroker/pu_missions/cargo/*.xml`, `missionbroker/pu_missions/delivery/*.xml`).
2. **Generator → template**: a template (`contracts/contracttemplates/*.xml`) is a reusable skeleton with properties, objective tokens and a flow state machine; a generator (`contracts/contractgenerator/**`) instantiates templates through *career contract* entries that fill the properties, set availability and set rewards.

The two are bridged: a generator can wrap broker entries directly (`ContractGeneratorHandler_Legacy` → `ContractLegacy missionBrokerEntry=…`, e.g. in `contracts/contractgenerator/interstellartransport_guild/covalex industries/covalex_hauling.xml`). Only about half of the cargo broker entries are referenced by a generator; the rest look unused *(inference)*.

### 2b. Generator: how it is cut

- Folder per guild → company → job file; one file holds a list of handlers, usually one per distance band or per rank.
- **Handler** (`ContractGeneratorHandler_Career`):
  - fixes one faction and one reputation scope for all its contracts;
  - `required_active_scenarios`: runs only while a named event scenario is active;
  - `defaultAvailability` (`ContractAvailability`): max players per instance, once only, can re-accept after abandoning or failing with cooldown and variation, personal cooldown, notify on available, plus prerequisites;
  - `contractParams`: mission type, text keys, the contractor, and comms tag sets for accept, abandon and complete.
- **Career contract** (one job): a template link, a `minStanding`/`maxStanding` band, parameter overrides (all concrete locations, cargo orders and display tokens), sub-contract variants with their own extra prerequisites, `generationParams` (max instances overall and per player, respawn time and variation), `contractLifeTime` (how long the offer stays on the board), and `contractResults` with a `timeToComplete` and a difficulty.
- **Prerequisite types**: locality (which system), completed contract tags (required or excluded, with counts; used to chain ranks), reputation band, location, crime level, location property. Most carry `includePrerequisiteWhenSharing`: whether party members must meet it too.
- **Locations are tag searches** (`DataSetMatchCondition_TagSearch` with positive and negative tags against `TagDatabase.TagDatabase.xml`). Hauling pickups and dropoffs require a tag meaning "can host cargo missions". Other job types filter by distance (`…ExcludeNearbyLocationsDef`, `…ExcludeDistantLocationsDef`, each with a range in km relative to another property); hauling instead gets its distance band from hand-picked location tags per route class. A further condition keeps two stops from resolving to the same place.
- **Random choices** are options with a weighting and dependent properties (choosing one option can set further properties).

### 2c. Template and broker entry fields

- **Template** (`contracts/contracttemplates/haulcargo_atob.xml`):
  - contract class: has complete button, handles abandon, can be shared, show allied markers, only owner can complete; auto-fail if sent to prison, became criminal, left prison;
  - deadline (`MissionDeadline`): completion time, auto end, result after the timer (almost always failed), when to show the timer, end-reason text key;
  - display: mission type, illegal flag, show lifetime, pre-show objectives;
  - modifiers: security clearance, faction hostility, hostile mission;
  - properties: each a variable name for the logic plus a text token for the UI, shipped empty and filled by the generator (contractor, route, grade, max box size, rank, pickup, dropoff, hauling orders);
  - objective tokens, partial-payout bands, the flow state machine and comms on start and end.
- **Broker entry** (`missionbroker/pu_missions/cargo/haulcargo_rounddelivery.xml`) is a flat superset: giver, difficulty, lawful flag, where offered, initially active, request only, invitation chaining, buy-in with refund on withdraw, instance and player limits, can be shared, once only, respawn and lifetime, re-accept and cooldowns, a fixed reward with max, bonuses, currency and a reputation bonus record, reputation per result, deadline, completion tags (also on fail or abandon), required tags, missions, journal entries and area tags, a date schedule.

### 2d. Hauling specifics

- Hauling uses a native objective handler (`ObjectiveHandler_Hauling`) that owns a list of **hauling orders**; one order is one pickup → dropoff leg. Order kinds: a resource with min/max SCU and a max container size, an entity class with min/max count, a property the generator fills, a mission item, alternatives, and "drop off whatever was collected".
- **Two authoring styles**: baked templates (commodity and SCU range inside the template, one template per shape × grade × good, hence hundreds of files), and property-driven templates where the career contract supplies the orders (several goods per contract make a mixed load, each with its own container cap).
- **Shapes are the topology of the order list**: A→B (one order), one-to-N (N orders from one pickup), N-to-one, round (A→B→C→A with rising volume per leg), linear chain (A→B→C→D), open delivery. Round and chain exist only as unreferenced broker entries *(inference: not offered live)*. Combat variants add a second objective token that spawns an encounter beside the cargo token.
- **Grade** is a display token paired with a max box size token; the binding constraint is the max container size on each order, i.e. the biggest box the cargo is packed into. Which ship takes which box is one field on the ship's cargo grid (2g).
- **Return-goods branch** (`contracts/contracttemplates/haulcargo.xml`): if the player abandons while carrying the cargo, a second objective starts that asks to bring it back; finishing it counts as abandoned, failing it counts as failed.

### 2e. Rewards, difficulty, results

- Results are a list of result objects, each with a five-slot mask saying which mission results fire it (slot names are not in the data; slot one is success, one slot carries every negative reputation result *(inference)*). Result kinds include calculated reward, reputation amount, completion tags, item rewards (fixed or weighted, `contracts/contractrewards/iawr_*.xml`), blueprint pools, badges, scenario points, journal entries, buy-in refund.
- **Calculated reward has no parameters in the data.** Its inputs are, by elimination, the contract's `timeToComplete`, buy-in amount and difficulty *(inference: formula in code)*.
- **Difficulty** (`ContractDifficulty`) is four rated axes, each on a seven-step scale: mechanical skill, mental load, risk of loss, game knowledge. A difficulty profile (`contracts/contractdifficultyprofiles/*.xml`) is only four weights on those axes; the logistics profile leans on mental load and risk, boosted variants raise the weights.
- **Partial completion** (`partialRewardPayout`): bands of percent delivered, each with a money multiplier and separate reputation multipliers per scope. Hauling uses four bands, with nothing paid for the lowest **[from file]**. An objective opts in with a percent-of-mission contribution *(inference: share of SCU delivered)*. Lost or damaged cargo therefore shows up only as a lower delivered share; there is no explicit time-bonus or cargo-damage knob in these folders.
- **Failure**: the global failure record is nearly empty. Real wiring is per template: deadline → failed; prison or crime → failed; and the flow state machine (conditions on token state, property set, and/or/not → state change actions).
- **Modules**: `contracts/modulehierarchy/*_mmh.xml` is a tree of scripts a mission may spawn (cargo recovery → ship fight). Hauling needs no module: pickup, dropoff and time are orders, location properties and the deadline.
- **Givers and scenarios**: a giver (`missiongiver/**`) has a faction, headquarters, invitation and visit timeouts, short/medium/long cooldowns, allies and enemies. Event scenarios (`missionscenarios/*.xml`) have a schedule, cycles and progress tracks with tiered rewards; contracts feed them points.

### 2f. Reputation as the unlock system

- Reputation is a number per **faction × scope**. A scope (`reputation/scopes/*.xml`, cut by career: hauling, courier, bounty, security, salvage, racing, smuggling …, many cloned per organisation) has a ceiling, an initial value and an ordered ladder of standings. Ladders are short and roughly geometric (each rank needs several times the previous one); a separate symmetric "affinity" ladder runs from hated to exalted.
- A **standing** (`reputation/standings/**`) is a threshold plus a drift rule (amount per hours) and a `gated` flag. Blacklisted standings drift back up on their own (a timeout); some high ranks decay over time. `gated` sits on entry ranks *(inference: you cannot climb past it by points alone, only by an intro or rehire contract)*.
- **Standings carry no unlock payload.** What they unlock is defined on the consumer: a contract's standing band, a perk list (`reputation/perks/*.xml`, item rewards at a standing, barely used), or a payout bonus record that adds a fraction of the money reward per standing (`reputation/rewards/missionrewards_bonusuec/`). No shop discount coupling was found.
- **Reward amounts** (`reputation/rewards/missionrewards_reputation/`) are a symmetric size ladder, roughly doubling per step, used positive for success and negative for fail and abandon.
- **States** (`reputation/states/`, updated by `reputation/globalmissionresultmodparams.xml`): succeeded/failed/abandoned most recent and previous, has done any mission, success streak, fail streak. Abandon counts as a failure for streaks. No consumers of these states were found in the data.
- **Factions** come in two layers: the AI and law record (`factions/*.xml`: default reaction, lawful/unlawful/security/police, allies, enemies, can arrest) and the reputation-facing organisation (`factions/factionreputation/*.xml`: which scopes it shows, allies and enemies, a hostility threshold on the faction scope, sandbox triggers that cost reputation for harming people).
- **Gating patterns** in the generators:
  - *tiers*: contracts of rising difficulty share a maximum standing and raise the minimum;
  - *intro*: a once-only contract at the entry rank, sealed afterwards by a completion tag it writes;
  - *rehire*: after a failure drops the player below entry, the score drifts back to the gated entry rank and a rehire contract is the way back onto the ladder *(inference from the structure)*.

### 2g. Cargo, loading and shops

- One volume unit everywhere, in three scales (standard, centi, micro) for cargo, ore and fuel alike.
- **Boxes**: one crate entity per box size (`entities/scitem/ships/scu_cargo_template_*scu.xml`, listed in `globalshopparams/globalshopcommoditydata.xml`), each a rigid body, pushable and carryable.
- **Cargo grids** (`inventorycontainers/cargogrid/*.xml`): interior dimensions, cell size, min and max permitted item size, item-type filters. Capacity is geometric (volume ÷ one box cube). The **max permitted item size is the whole "which ship takes which box" rule**. Destruction knobs say how much cargo survives a ship kill. Station freight elevators are grids too (`freightelevator_cargogrid_*.xml`). Per-location storage with lawful and impound flags: `inventorycontainers/landingzoneinventory.xml`.
- **Loading** (`globalcargoloadingparams/globalcargoloadingdata.xml`): manual loading time per unit; auto-loading with a base time plus a time per box size (big boxes take longer, but less per unit); an auto-load fee with a base plus a price per box size (big boxes cheaper per unit, `globalshopcommoditydata.xml`); loading-area discipline (move time limit, revoke delay, forfeit buffer: cargo left too long is lost). Strings: auto-load needs the ship stowed.
- **Cargo damage** (`commodityconfiguration/commoditydamageconfig*.xml`): impact damage above a minimum speed, scaling with speed squared, after a grace period; volatile cargo explodes with radius, force and chain damage to other cargo.
- **Shop entries** (`Shops/shopinventories/*.json`): exactly six fields: item id, buy price, sell price, current stock, max stock, rental offerings. An entry either sells or buys, never both. No base price, restock rate or curve per entry.
- **Supply and demand bands** (`globalshopparams/globalshopcommoditydata.xml`): six thresholds on the stock fill ratio for supply and mirrored ones for demand; the kiosk shows the band, not a number *(inference: price moves with fill, curve in code)*.
- **Global shop knobs**: a resale cut depending on whether the shop stocks that item type, per-type overrides that make most gear nearly worthless to resell, wear and shop-fullness curves (`globalshopsellingdata.xml`); licence-tier price modifiers and a per-hour buy limit (`globalshopbuyingdata.xml`); max quantity per purchase and a list of transaction errors that spells out the model (insufficient funds or stock, buy limit, price mismatch, invalid box size, player in vehicle during cargo transfer) (`globalshopterminaldata.xml`).
- **AI cargo** (`cargomanifest/`): fill mode (random, even, custom), min/max share of the hold, weighted resource list; generic manifests are a small matrix legal/illegal/mixed × value low/medium/high × fill tier; extra sets per faction and per scenario.
- **Player trade** (`playertrade/playertradeglobalparams.xml`): money transfer only, with a service fee rate; two currencies.
- **Money sinks visible**: resale loss (the largest), auto-load fees, cargo forfeiture, cargo destruction, refining cost tier, transfer fee, licence modifiers, impound.

### 2h. Refining (a production reference)

`refiningprocess/processingtype-*.xml`: a process is only speed (fast, normal, slow) × care (careful, normal, wasteful) as enums, nine combinations. Times and yields are not in the data; the localisation describes each as a trade-off between speed, cost and yield (careful = high yield, fast = high cost).

### 2i. Contract text and UI (localisation, `global.ini`)

- Contract text is templated with variables (location, contractor, destination, items 1–5, pickups and dropoffs 1–n, timed, danger, max cargo size, max box size, cargo grade, payment, combat pay, reward, reputation rank). Hauling texts come per company in intro and rehire variants.
- Contract manager tabs: offers, accepted, history, beacons; fields availability, deadline, contracted by, reward, bonuses, distance. Players can create beacons.
- Reputation UI: standing, stances (ally, neutral, hostile), rising/falling velocity; rank ladders per career.
- Rewards land in the player's home inventory.
- Tag tree (`TagDatabase.TagDatabase.xml`, `Missions/…`): end-state tags (exceptional time, partial, time expired; cargo destroyed, out of time; abandon after collection, cargo lost, cargo returned), mission tier, difficulty, legality, location types. No consumer of the exceptional-time or cargo-destroyed tags was found in the contract records.

### 2j. Pads, fuel, harvestables, travel

- `landingpadsize/*.xml`: six size classes, each with an id, a ship bounding box and a ground-vehicle box (zero on some classes, so no vehicles there). The same records serve as docking class for ATC timing overrides (`entities/scitem/entityclassdefinition.atc_datamanager.xml`) and as the largest-ship limit of a jump point (`entities/jumppoints/jumppoint_permanent.xml`).
- Fuel: `fuelparams/evathrusterfuel.xml` is suit fuel only. Ship fuel is a resource network (`itemresourcenetwork/itemresourcenetworkglobal.xml`: power, fuel, coolant, shield, quantum fuel, gas …), burnt per thrust (`fuelBurnRatePer10KNewton` on thrusters), stored in tanks (`entities/scitem/ships/fueltanks/*`), refilled by intakes; quantum travel costs fuel per jump (`quantumdrive/*`: fuel requirement, cooldown, calibration, speed).
- `harvestable/`: four layers linked by id: provider presets per planet or region (groups with probability, elements with relative probability and clustering), harvestable presets (entity class, respawn-in-slot time, polymorphic harvest conditions, despawn with a wait for nearby players), clustering presets, slot presets for caves and facilities with depth scaling and loot constraints. Yields sit on entity classes and loot tables, not in this repo.
- `ssolarsystem/*.xml` (position, default location, landing zone inventory); `starmap/pu/**` marks which objects can host player-created missions (`exposeForPlayerCreatedMissions`), plus jurisdiction and respawn type; `jumppoints/globaljumpdriveparams.xml` has a jump state enum (idle, checks, tuning, requesting, waiting, entering, transiting, exiting, failing).

## 3. No Man's Sky

How the records were unpacked and the path conventions: `no-mans-sky.md` section 2. Paths below are relative to `research/local/nms/exml/`.

### 3a. The hub

`metadata/reality/defaultreality.MXML` points to every table (reward, cost, trading cost, trading class, substance, products, unlock trees, purchasable specials and blueprints, stat rewards) and holds commerce blocks directly: shop profiles (`TradeSettings`), reputation shops, freighter cargo options, faction standing ids, mission name word pools, mission-board reward tiers, never-sellable lists, price bands.

### 3b. Missions

- 28 tables in `metadata/simulation/missions/tables/`; the station board templates are `npcmissiontable.MXML`, planet-local ones `planetprocmissiontable.MXML`, fleet ones `fleetmissiontable.MXML`.
- A mission (`GcGenericMissionSequence`) has identity, class, objective text pools (`GcNumberedTextList`: a format key plus a count, filled into titles and 3-part procedural descriptions), flow flags (auto start, restart on completion, can renounce, recurring), start and cancel conditions with a test mode (any/all true/false), scan events (where), rewards, costs, a shop override, default item pools, and a stage tree.
- Board knobs (`GcMissionBoardOptions`): type (`GcMissionType`, combat dominates, cargo is a small share), difficulty (three), minimum rank, faction list, selection weight, a penalty reward on abandon, guild-shop and planet-local flags. Board globals in `gcgameplayglobals.global.MXML` (at the root of `exml/`): missions per system, max per giver, a difficulty multiplier on required amounts, standing required for station boards.
- **Delivery template** (`DELIVER` in `npcmissiontable.MXML`): the item comes from a pool of trade goods (`DefaultItems.PrimaryProducts`); a cost entry charges that item at the destination (delivering = paying the cost); a start reward hands the item over; the destination is a scan event (`SearchType` station, nearest, local or near system) with optional filters on the system's wealth, trading class, race and conflict level.
- Randomness enters only through: board picking by weight/faction/rank, item pools, scan-event target lookup and seeded rewards. All sequences are hand-authored templates; `IsProceduralAllowed` is never set *(inference)*.
- Stages: a tree of groups with conditions, consequences, repeat logic and notify timers; about 90 sequence types (wait, reward, collect, get to scan event, collect money …) and about 270 condition types (has money, can pay cost, has product, faction rank, trade surge …). Consequences are few; effects flow through reward stages naming a reward id.
- Time: no general deadline field on board missions *(inference: untimed)*. Timers exist as real-time waits, a timed trade-surge condition and calendar schedules (`metadata/simulation/missions/missionschedulestable.MXML`: daily or weekly recurrence with start and end date).
- Board pay is not per mission: `MissionBoardRewardOptions` in defaultreality holds tiers, each a pick list of reward ids from low to mega; repeating an id acts as a weight.

### 3c. Rewards: `metadata/reality/tables/rewardtable.MXML`

- Sections by source: generic, world containers, mission board, interactions, destruction, fleet, settlements, stories, salvage, special, and more; plus two ordered unlock lists (tech and product reward order).
- Entry: id → list with a choice mode (`RewardChoice`: give all, select always, try each, select from success, silent variants) → items, each with a percentage, a label and a polymorphic reward. In select modes the percentage acts as a weight, in give-all/try-each as a roll *(inference)*.
- World containers use rarity × size → item list.
- About 150 reward types: specific product or substance, procedural product, money (min, max, round, currency), standing, faction standing, recipe, tech, ship, frigate, scan event, mission, unlock tree, wanted level. Three currencies (`GcCurrency`). Substance rewards can scale with board difficulty.
- Related: `metadata/gamestate/stats/statrewardstable.MXML` converts stats to currency; `expeditionrewardtable.MXML` for fleet runs.

### 3d. Goods and prices

- Substances (`nms_reality_gcsubstancetable.MXML`) and products (`nms_reality_gcproducttable.MXML`, plus base-part and customisation product files): base value, rarity, legality, stack size, price modifiers (`GcItemPriceModifiers`: station markup, low and high price mod, buy markup), normalised value on and off world, trade category, economy influence multiplier.
- Trade goods are a small set: a few per trade category (`GcTradeCategory`, seven categories), separate from the thousands of crafting items.
- Economy classes (`tradingclassdatatable.MXML`): each class sells one category and needs another, with min/max sell and buy multipliers and a higher surge multiplier; a global min/max clamp.
- System attributes: wealth class, trading class, conflict level.
- Per-sale drift: `tradingcosttable.MXML` holds cost, min, max and change per sale per good, but the globals switch the table and local price changes off *(inference: legacy or disabled)*.
- Shop profiles (`TradeSettings`, about 30): always-present and optional stock, items-for-sale range, stock amounts and price improvements per wealth class, UI colour thresholds for good and bad prices, barter knobs. A mission can override the profile.
- Costs (`costtable.MXML`): substance, product, money, standing and special costs; flags like "remove option if can't afford" make costs double as gates.

### 3e. Unlocks and standing

- `unlockableitemtrees.MXML`: groups of trees, each node an unlockable with children (parent before child); each tree pays in one cost type (a currency, a product or a substance).
- Purchasable specials with shop number and mission tier.
- Standing per faction as a stat with level thresholds (`metadata/gamestate/stats/leveledstatstable.MXML`); standing gates reputation shops (`GcRepShopItem`: product, amount, price multiplier, currency, required level), standing costs, board minimum rank, and gives a tech discount.

### 3f. Freight and the automated haul loop

- `FreighterCargoOptions` in defaultreality: id, min/max amount, chance.
- Fleet expeditions (`gcfleetglobals.global.MXML` (root of `exml/`), `expeditioneventtable.MXML`, `frigatetraittable.MXML`): choose one of several offered expeditions, a duration class, events per time and per distance, difficulty by event number with variance and per-extra-ship increase, a dice cap for event resolution, wear after a number of runs. Events have a stat focus, a difficulty modifier and reward ids per outcome (easy success, success, jackpot, failure). Mid-run intervention events can call the player in. Ship traits raise stats with an offer chance per ship class.

### 3g. Interactions

`metadata/simulation/interactions/` holds only a salvage rule file. NPC trade and reward dialogs are in `metadata/reality/tables/nms_dialog_gcalienpuzzletable.MXML`: each option has a cost id, reward ids, mood, next interaction and disabling conditions; interaction types include NPC, guild envoy, shop, mission giver, fleet navigator.

## 4. Public write-ups of other games

**[V]** fetched and checked; **[S]** known from search snippets only (fetch blocked); *secondary* = fan wiki or outside analysis.

### Jobs and contracts

- **Sunless Skies**, Failbetter blog "Trade in the Skies: Roche Limit" **[V]**: trade built on *prospects* (a port pays a premium for N units of one good, a few held at once) and *bargains* (cheap limited stock at minor ports); plain port prices are flat so the route alone earns nothing; a market simulation was rejected. https://www.failbettergames.com/news/trade-in-the-skies-roche-limit
- **Sea of Thieves**, cargo runs **[V, secondary]**: pick up at A, deliver to B, the timer starts at pickup, pay scales with crate condition, cargo types have handling rules, higher rank adds crates. https://seaofthieves.wiki.gg/wiki/Cargo_Runs
- **Euro/American Truck Simulator**, job market **[V, secondary]**: tiers of the same delivery (company truck, all costs paid, low pay → own truck, own costs, more profit → own trailer, permanent damage); damage cuts pay and XP; better jobs unlock with level and skills. https://trucksimulator.wiki.gg/wiki/Job_Market
- **ETS2**, SCS blog on World of Trucks contracts (2015) **[V]**: a server-side contract layer on top of single player with a fixed time window and a speed limiter for fairness. https://blog.scssoft.com/2015/12/world-of-trucks-contracts-questions.html

### Economy and pricing

- **Sunless Sea** postmortem (Game Developer) **[V]**: supply pressure pulls the player home and the return is part of the pacing; slow travel became grind. https://www.gamedeveloper.com/audio/postmortem-failbetter-games-i-sunless-sea-i-
- **Starsector**, "Economy & Outposts" (2017) **[S]**: supply and demand matched across markets, the player can become a supplier. https://fractalsoftworks.com/2017/09/19/economy-outposts (the earlier Starsector posts are summarised in `CORE-LOOP.md`).
- **Death Stranding**, design discussion (Game Developer) **[V, secondary]**: rewards that change the world or the player's options matter more than a score. https://www.gamedeveloper.com/design/a-design-discussion-on-death-stranding

### Progression and unlocks

- **Sea of Thieves**, Rare interview on progression and quests (Destructoid) **[V]**: reputation per company makes its voyages richer and adds mechanics, not power; crews of mixed rank play together without barriers. https://www.destructoid.com/rare-talks-progression-and-quests-in-sea-of-thieves/
- **Sunless Skies** (Roche Limit post above): affiliation tracks unlock better prospects and bargains.

### Events and pacing

- **Left 4 Dead**, "The AI Systems of Left 4 Dead" (Valve, 2009) **[V]**: per-player intensity rises with damage and decays; the director cycles build-up, sustain peak, peak fade, relax. https://cdn.fastly.steamstatic.com/apps/valve/2009/ai_systems_of_l4d_mike_booth.pdf
- **RimWorld**, AI storytellers **[V, secondary]**: threat size budgeted in points scaled by progress; storytellers differ in cadence; time since last event and recent losses are tracked. https://rimworldwiki.com/wiki/AI_Storytellers. GDC 2017 talk by the designer **[V]**: https://gdcvault.com/play/1024232/-RimWorld-Contrarian-Ridiculous-and
- **FTL**, GDC 2013 postmortem **[V]**: process, not event pacing. https://gdcvault.com/play/1018034/Designing-Without-a-Pitch-FTL. Event lists with min/max counts per sector type, from forum threads **[unverified]**: https://www.subsetgames.com/forum/viewtopic.php?p=47168

### Co-op

- **Deep Rock Galactic**, mutators **[V, secondary]**: at most one negative warning plus one anomaly per mission, each warning shows its hazard pay bonus up front, each rotation keeps a clean mission per type, the whole crew gets the same reward. https://deeprockgalactic.wiki.gg/wiki/Mutator and https://deeprockgalactic.wiki.gg/wiki/Update_15:_Mutation_Warning
- **Overcooked**, Develop postmortem (MCV) **[V]**: "too many tasks for the players you have", routines broken up mid-round, a timer instead of lives, new recipes balanced by easier environments. https://mcvuk.com/the-develop-post-mortem-overcooked/

### Content as data

- **RimWorld**, XML Defs **[V, secondary]**: all content is XML in a data folder; vanilla files are the examples, mods add files in the same format. https://rimworldwiki.com/wiki/Modding_Tutorials/Defs
- **FTL** (events in XML) and **EV Nova** (missions as resources, plugins with own id ranges) **[unverified]**.

No usable public developer write-ups were found for Elite Dangerous, Freelancer, Space Rangers, Highfleet, Kenshi, Port Royale, Mount & Blade or Slipways.

## 5. Options per open question of epic #37

Options only, with trade-offs and which game does what. Nothing is decided here.

### 5a. Contract types

- **A. One delivery template with modifiers.** A pickup → dropoff job; variety from modifiers (fragile, timed, heavy, risky). *For:* one state machine, one scenario, fastest to playtest; matches the A1 baseline of three templates. *Against:* may feel samey after a few rounds. *Who:* Sea of Thieves cargo runs (one shape, handling rules per cargo type); Deep Rock Galactic (modifiers on a fixed mission type).
- **B. A few shapes × parameters.** Shapes such as A→B, one-to-many, many-to-one, chain; parameters distance, load size, time. *For:* lots of variety from little data; distance and size map directly onto flight and the cargo limit. *Against:* multi-stop shapes need more HUD (several targets) and more tests. *Who:* Star Citizen hauling (shapes as topology of the order list, grade as max box size, distance band per route class, section 2d); No Man's Sky board (fixed templates filled from item pools, 3b).
- **C. Person-driven orders.** Named givers with preferences who order regularly; the relationship decides what gets offered. *For:* gives places and characters personality, goofy text has a home; demand without a market simulation. *Against:* more content per giver; relationship systems need tuning. *Who:* Schedule I customers (1b); Sunless Skies prospects.

### 5b. Money

- **A. Fixed pay per contract, flat prices.** Profit only from contracts; goods have one price. *For:* trivial to tune and explain; no spreadsheet play. *Against:* no trading layer at all. *Who:* Sunless Skies (flat port prices, margin in contracts); Sea of Thieves.
- **B. Base price × place multiplier.** Places sell some goods cheap and need others; optional temporary surges from events. *For:* a small trading layer with readable routes. *Against:* dominant routes appear unless there are sinks or rotating surges. *Who:* No Man's Sky trading classes with surge multipliers (3d); Starsector lesson in `CORE-LOOP.md`.
- **C. Graded payout.** Pay = base × result (on time, partial, damaged, bonus lines). Combines with A or B. *For:* rewards skill in flying and handling cargo, ties pay to the flight feel. *Against:* needs clear feedback on why pay was cut. *Who:* Star Citizen partial-payout bands (2e); Schedule I match score and bonus lines (1c); Sea of Thieves crate condition; ETS damage.
- Sinks (any option): running costs (fuel, repairs), one-time purchases, fines. *Who:* Schedule I (wages, fees, fines, 1c); Star Citizen fuel per thrust and per jump (2j), auto-load fees per box, resale loss, cargo forfeiture (2g).

### 5c. Unlocks

- **A. Rank from XP, requirement on the unlocked thing.** Places, contracts and shop items carry a minimum rank. *For:* one number, easy data (a field per entry). *Against:* grind if XP is slow; one global axis. *Who:* Schedule I (`MapRegionData.RankRequirement`, 1d).
- **B. Standing per giver or faction.** Each giver's standing unlocks their better jobs and shop items. *For:* choices about whom to work for; fits person-driven contracts. *Against:* several axes to balance; co-op desync risk if standing is per player. *Who:* No Man's Sky standing and rep shops (3e); Star Citizen faction × scope ladders with drift, intro and rehire contracts (2f); Sunless Skies affiliations.
- **C. Buy unlocks with money.** Places or licences are purchases. *For:* money has a direct use, strong sink. *Against:* money then carries two jobs (progress and running costs). *Who:* Schedule I property prices; No Man's Sky unlock trees paid in a currency (3e).
- Across all: unlocks that add mechanics or larger loads rather than power keep mixed co-op groups working (Sea of Thieves interview, section 4).
- Across all, for co-op: Star Citizen marks each prerequisite with whether party members must meet it too, and contracts with max players and whether only the owner can complete (2b, 2c). That is one way to decide what a shared job asks of the guests.

### 5d. Production yes or no

- **A. No production in the MVP.** Hauling and maybe trading only. *For:* smallest scope, keeps the focus on flight; matches "pick up, gather, haul" in `CORE-LOOP.md`. *Against:* less "build an empire" feel than Schedule I.
- **B. Gathering only.** Collect raw goods at places (pick up, mine) and sell or deliver them. *For:* a second verb without stations or recipes; uses the same cargo code. *Against:* respawn and distribution need tuning. *Who:* Star Citizen harvestables (provider presets, respawn per slot, 2j).
- **C. Light production.** One or two conversion steps (raw → product) at an owned place. *For:* the Schedule I management feel, quality as a lever on pay. *Against:* recipes, stations, UI, more balancing; Schedule I's known burnout comes from automation late on. *Who:* Schedule I growing, cooking, mixing (1f); No Man's Sky fleet expeditions as an automated loop (3f); Star Citizen refining as one speed × care choice trading time, cost and yield (2h).

### 5e. Content schema

- **A. Flat records, one file per object.** Commodity, location, mission template as in the first sketch in `CORE-LOOP.md`; references by id. *For:* simple to validate and mod. *Against:* variety must be written out by hand.
- **B. Templates plus pools.** A mission template references pools (goods, destinations by filter, text fragments, reward tiers); the generator draws from them. *For:* lots of variety from little data; text fragments fit the goofy tone. *Against:* harder to validate and to test deterministically (needs seeds). *Who:* No Man's Sky (item pools, scan-event filters, numbered text lists, board reward tiers, 3b); Star Citizen (templates with empty properties, generators fill them; locations as tag searches; weighted options, 2a–2c).
- **C. Requirements on the unlocked thing vs. a central unlock table.** A sub-choice for either A or B. *Who:* Schedule I puts rank on each region and item (1d); No Man's Sky keeps unlock trees as their own records (3e).
- Across all: the save stores offer fields plus a link to the giver (Schedule I `ContractData`, 1a); a completed-job log lets other systems ask "what was delivered where lately" (Schedule I receipts).

### 5f. Route events

- **A. Timer with cooldown and weights.** About one event per interval, picked by weight per route, with a cooldown per template. *For:* simple, already in `CORE-LOOP.md`, easy to test with a fixed seed. *Against:* can feel mechanical; events may stack with whatever else is happening.
- **B. Intensity director.** Track a tension value (damage, time since last event, cargo value); events build up, peak, then a forced quiet phase. *For:* pacing adapts to the players; avoids stacking. *Against:* more tuning, harder to explain in a test. *Who:* Left 4 Dead director phases; RimWorld storytellers (points budget, time since last event).
- **C. Modifiers fixed at contract time.** The contract shows its hazards up front (one warning, one upside) with a pay bonus; events during flight come from those. *For:* the player chooses risk knowingly; ties events to pay. *Against:* fewer surprises in flight. *Who:* Deep Rock Galactic warnings and anomalies; Star Citizen difficulty rated on four axes with a weight profile feeding pay, combat variants that add an encounter token (2d, 2e); No Man's Sky fleet expedition events with outcome rewards (3f).
