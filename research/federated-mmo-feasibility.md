# Research: a federated "MMO" without our own server infrastructure

Status: research pass, 2026-10-08. Question from the initiator: could EXO-1 grow into a shared galaxy with markets later, without Star Citizen-style infrastructure, while keeping client authority and keeping cheaters from breaking the economy? Facts are marked **[verified]** (checked against a source today), **[calculated]**, or **[unverified]** (reasoning, needs a spike before we rely on it). This document is the basis for the parked entries in `DECISIONS.md`; nothing here is decided.

## Short verdict

Viable, but not as "one decentralized MMO". As a **federation of self-hosted star systems** (the email/Mastodon model, not the single-shard model). Each host is fully authoritative inside its own system; what crosses system borders is low-rate and discrete (identity, inventory transfers, market offers, reputation), never live physics. Blockchain is the wrong tool for almost all of it; the one problem it could solve (double-spend between mutually distrustful hosts) has two far cheaper designs (escrow gate, burn-and-sign receipts).

This matches what is already parked in `VISION.md` ("players can run their own persistent server: their own galaxy"; "shared markets across servers: not planned, maybe later").

## The cost model, verified against precedent

- Star Citizen sells one seamless authoritative shard. Over US$1 billion in funding by May 2026, most expensive game projects ever, and the Persistent Universe still fights server-side limits after a decade of object-container-streaming work **[verified, Wikipedia, fetched 2026-10-08]**. That is what "shared everything, low latency for everyone" costs. We must not buy this.
- Dual Universe bet on exactly the tech we are told to want: CSSC, one continuous single-shard cluster, no instances, no loading screens, fully editable voxel world, player-run economy **[verified, Wikipedia, fetched 2026-10-08]**. Servers shut down on 2025-08-27; the shutdown statement says player-hosted servers are being investigated **[verified]**. A single-shard player-economy MMO with venture funding died; the player-hosted variant is the piece that survives.
- OpenSimulator (open-source Second Life server) has run the exact pattern we propose since 2007: standalone or grid mode, anyone can host, and **Hypergrid** lets avatars teleport between independently operated grids over a hyperlinked map; around 400 active Hypergrid-enabled grids existed as of February 2023 **[verified, Wikipedia, fetched 2026-10-08]**. Federation of self-hosted worlds with cross-world travel is a solved, 15+-year-old pattern. Its known weak points (malicious grid admins, uneven maintenance quality) are the same weak points we would inherit, managed socially (defederation, allowlists).
- Mastodon confirms the social layer of the same pattern: independently run servers, ActivityPub federation, per-instance rules, admins can defederate other instances; instances run by volunteers with uneven security skills are the documented risk **[verified, Wikipedia, fetched 2026-10-08]**. The game equivalent of a server list plus allow/defederate lists is the whole "anti-cheat infrastructure" a federation needs at its boundary.

Why federation is nearly free for us specifically **[calculated]**: our host already relays snapshots (`net_core`, UDP, `--net-host`); the simulation already lives in Bevy-free `*_core` crates. A host that is authoritative for economy adds a handful of rule checks per second, not physics. The server list starts as a static JSON file in a repo.

## Cheating: three separate problems

### 1. Client cheats inside one host's system (speedhack, teleport, "I mined 100 million ore")

- Industry standard: "never trust the client"; the better the server enforces rules, the less cheating matters **[verified, Wikipedia "Cheating in online games", fetched 2026-10-08]**. With client authority, a cheating client can mainly lie about itself.
- Defence that costs nothing: the receiving host sanity-checks incoming snapshots and requests (max speed, max acceleration, teleport distance, rate limits, "is the player actually at the node"). A few comparisons per packet, a few ifs per request **[verified as the standard approach; our exact rules are design work]**.
- The economy rule that closes the "100 million items" hole: **anything with value is decided by the host**. Node inventory, wallet, cargo, trade, crafting results live in host state; the client sends requests, the host mutates and answers. Client-side prediction may display optimistic results, but the host's answer is canonical. **[unverified as a design decision — this is the one early decision to take, because retrofitting client-authoritative economy into host-authoritative is painful, the reverse is not]**. Our existing decision "Cargo is physical, limited by volume" already caps what any character can carry at all.
- What remains: a cheating client moves too fast *towards* a node. Economically irrelevant.

### 2. A malicious host (the "cheat server" the initiator is sad about)

- Not preventable by cryptography: simulation nobody else ran cannot be verified from outside. No ledger checks physics. zk-verifiable games exist (Dark Forest runs on-chain with zk proofs for a slow, discrete 4X rule set **[verified, zkga.me and write-ups; fetch of the site returned only an app shell, so treated as background, not a blueprint]**), and that approach does not transfer to a real-time f64 physics sim **[calculated: proof generation for continuous physics is far outside a hobby project, and Dark Forest's tick is a discrete turn, not 60 Hz]**.
- The precedent says: contain, don't prevent. Mastodon's answer to bad instances is defederation; OpenSim grids answer bad grids by dropping them off the hyperlinked map **[verified, above]**. The economics of cheat servers collapse if their goods are worthless outside:
  - **Provenance:** receipts signed by the sending host; items carry origin. A host caught cheating has its receipts refused at every border (tainted goods).
  - **Import allowlists:** each host decides whose receipts it accepts. Start: the initiator and friends. Community servers join by reputation, exactly like Minecraft server lists or the Fediverse.
  - **Import quotas:** max assets per player per border crossing per day, so even a broken trust has bounded blast radius.
  - **Recipe-bound caps [calculated, unique to us]:** our planets are deterministic from a signed recipe (`planet_core`, bake), and the generator is open source. An importing host can re-evaluate the recipe and check: does node N on planet P even exist, and what is its capacity? A receipt claiming more ore than the world contains is rejected without trusting anyone. This caps a cheat server at "what the universe actually offers", which is also the balance boundary for the market design. (Caveat: what a *player* legitimately earned and when cannot be verified this way; the cap is on the world's total supply, not on playtime.)

### 3. Economy collapse through legitimate-looking trade

- The market board holds no assets: offers are text, fulfilment happens on the offering host. A fake offer can waste a trip, nothing more. Ratings/flags in the directory handle the rest **[unverified as design; standard marketplace mechanics]**.
- Cross-server deals between semi-trusted hosts: a tiny escrow service ("space customs": burn on A, hold, release on B) — a mail relay, not a game server **[calculated]**.
- Warning example of an economy bound to a real-money ledger: Axie Infinity's SLP lost over 99% from peak after the 2021-23 crypto crash, DAU fell from 2.7m to about 250k, and the Ronin bridge hack (US$620m, attributed to Lazarus Group) compounded it **[verified, Wikipedia, fetched 2026-10-08]**. A game economy's value should come from the game, not from speculation on a chain.

## Blockchain verdict

Solves exactly one problem here: double-spend between *mutually distrustful* hosts with *no central operator at all*. Costs: wallets/key management for every player, fees, irreversibility (a bug permanently destroys assets), permanent coupling to external infrastructure, and the wrong crowd for an open-source hobby project. Axie is the cautionary tale for economy-on-chain **[verified, above]**; Dark Forest shows zk games are real but slow and turn-based, not real-time sim verification **[verified/limited, above]**.

Two cheaper designs cover the realistic cases first:

1. **Transfer gate (escrow):** one small central relay, burn-and-hold-and-release. Central, but it is a mail relay, not an MMO. Fits "friends plus a community".
2. **Burn-and-sign receipts, no centre:** host A burns the ship/cargo, signs a receipt; host B imports only receipts from its allowlist. Fits the trust graph. No double-spend *inside* the trust graph; a rogue host is expelled.

Revisit blockchain only if the federation grows to hundreds of mutually distrustful hosts that all refuse a gate service. Until then: parked.

## What "shared galaxy" would actually mean

Not one continuous space. A directory of systems, market boards of offers, and travel with signed saves. Jumping is a disconnect/reconnect with a signed character file; the initiator accepts loading screens between systems (chat, 2026-10-08). Live cross-server physics or combat stays out of scope forever under this model **[calculated: cross-server real-time sync is precisely the Star Citizen/Dual Universe cost we are avoiding]**.

## Costs and risks for us

- Runtime: none today; later a host-side economy authority (plain Rust, no physics) and a key pair per host for signing receipts.
- Design risk: economy rules (node capacity, yields, prices) become cross-server rules; a bad balance leaks across borders via trade. Import quotas and recipe caps bound it.
- Social risk: we inherit the Fediverse's moderation problem (bad admins, uneven ops). Allowlists and provenance are the mitigation, same as the platforms we copy.
- Complexity risk: signing keys, receipt format, directory schema are new surface area. Keep all three minimal and versioned.
- The one thing to decide *early* (cheap now, expensive later): economy authority (problem 1) sits with the host, not the client.

## Relation to the roadmap

This is Milestone E ("The world grows") material. Nothing before it changes. The early decision worth taking now is the authority split: client stays authoritative for movement (latency), host becomes authoritative for anything with value.