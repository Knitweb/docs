> **Provenance:** synthesized 2026-06-20 from an 8-part research dossier (molgang current
> codebase read + target architecture, three dapp-landscape slices, deep design extractions of
> Red Alert and The Settlers, and the broader epic-strategy canon). 9-agent research workflow
> `wf_5e627ea6-fba`. Owner: Edwin Hauwert / knitweb.

# molgang — Best Codebase & Game-Design Playbook

*Synthesis of an 8-part research dossier (current codebase, target architecture, three dapp-landscape slices, Red Alert + The Settlers extractions, and the epic-strategy canon) into one actionable plan for **knitweb/molgang** on the **knitweb/pulse** substrate.*

Vocabulary note for everyone who picks this up: it is **knitweb / knitwebs** throughout — never "loom." A faction/domain-plugin is a **knitweb**; the substrate primitive is the **Knitweb**.

---

## 1. Executive Summary — the highest-leverage moves

molgang's primitives are unusually principled (integer Fiber-Tension, byte-exact relay signing, one canonicalization rule, real `pouw.quorum` settlement). The problem is **three drifting implementations of one ruleset** and a **single-process, in-memory, wall-clock-touching authority**. The whole plan collapses to one arc: *collapse the three state models into one CID-addressed, integer-deterministic, event-sourced core, then build the game's economy↔conflict loop natively on top of it.*

1. **Kill the three-implementation drift.** Make `game.py` + `tension.py` + the quorum binding the **single** rules core; demote PHP/Lua/PHP-MySQL to thin clients/light-clients. Today a knit "woven" on 5mart.ml gets a *different fiber CID* than the same knit in Python (PHP rolls its own `bafyrei…` base32) and a *different quorum threshold* — they can never converge to one state_root. This silently defeats "one shared web." **This is the precondition for everything else.**

2. **Event-source the state, keyed by CID.** Replace the in-memory `Bar` + unlocked `world.json` with an append-only log of signed, content-addressed actions; current state is a fold over the log. This makes restarts lossless, two instances reconcilable, and removes the file-truncation race. `merge.py`'s union-and-sum-confirmations logic is already ~80% of the CRDT you need.

3. **Finish the determinism job `tension.py` started.** Remove `time.time()` from `world.py`'s weave path (3 call sites feed `anchor_ts` → tautness → SNAP/routing — a determinism leak on a *decision* path). Drive everything from an injected logical beat (`Pulse.beat`/`Round._tick`) and a CID-seeded counter-PRNG. Then Erlay reconcile can actually converge two molgang worlds to one UAL.

4. **Adopt lockstep over a knitweb-ordered action log as the netcode.** molgang is strategy/economy at tick cadence, not twitch. Lockstep's hard requirement (bitwise determinism) you *already mostly have*; its weakness (input latency) is irrelevant. Order actions by `(tick, action_CID)` for a leaderless total order; gossip carries signed actions, Erlay reconciles the per-tick action set (the lockstep barrier). **Reject** rollback and server-authority.

5. **Carve a pure `sim/` package with a compiler-enforced no-`float`/no-`time`/no-`random`/no-unsorted-iteration rule, and stand up a replay-determinism CI gate** (replay one action log on two nodes with randomized `PYTHONHASHSEED`, assert identical `world_CID` every tick). This single test catches set-iteration, float-leak, and cross-runtime divergence at once — it is the crown jewel.

6. **Make PLS a medium-and-stake, never the prize.** The Axie/SLP death-spiral is the dossier's loudest warning. PLS is **sunk to act** (tension upkeep, traversal fees, claims, certificate mints), **minted only against verified PoUW**, and progression is **event/graph-driven, never elapsed-time-driven** (which your no-wall-clock invariant already forces — treat that as an anti-inflation *feature*). Publish a burn ≥ mint invariant in integer µPLS.

7. **Build the core loop as Red Alert × Settlers on knitweb primitives:** PoUW yield = the harvester/spice economy (discrete trips, settled at a "dock"); **Fiber Tension = the power grid** (graceful global tempo tax, never touches settled value); the `has/is-a/contains` graph = the tech-tree DAG and the Settlers production pyramid; **knitwebs = asymmetric factions** (distinct verb-sets + one capstone each). The moment-to-moment skill is **real-time four-way µPLS allocation: yield / contest / tech / defense.**

8. **Promote the `:8990` explorer from debug tool to the play surface, and animate flow.** The dossier's single highest satisfaction-per-effort change: render in-flight CIDs as moving objects on edges, edge-thickness = throughput, color = tension/congestion, plus a one-line "most-taut fiber = current bottleneck" HUD. This is what makes Factorio/Settlers compulsive, and you already have the graph.

---

## 2. Best Codebase / Architecture for molgang

### 2.1 Where molgang is today (grounding)

A multi-runtime implementation of one game on `knitweb/pulse`: a canonical **Python engine** (`src/molgang/`, ~10.7k LOC, stdlib-only HTTP, 3 deps: `knitweb`, `networkx`, `fpdf2`), a thin **Django/DRF** wrapper sharing one `Bar` singleton, a faithful **PHP/MySQL** port at 5mart.ml, and a **Lua/Roblox** counterpart with a two-way bridge.

**The strengths to preserve and build on:**
- **Determinism discipline where it matters.** `tension.py` (integer `S=1000` fixed-point NARS confidence, documented rounding + denominator guards + hysteresis), `faucet_micropulses` (exact two-phase integer decay curve), the spiral economy, and the relay `signed_preimage` (secp256k1 over a byte-exact preimage) are all genuinely float-free.
- **The game *is* the protocol.** Votes are real `AccountNode.transfer_to` Knits into escrow tallied by the real `pouw.quorum` — not a game-only shortcut.
- **One canonicalization rule** (`casefold(clean(term))`) reused identically across `world.py`, `merge.py`, `relay_sync.py` — the foundation any CID-stable sync needs.
- **Clean engine/framework separation** (`molgang_web/bar/engine.py` enforces "Django imports the engine, never the reverse").
- **Good test density on the hard parts** (`test_tension`, `test_relay_sync` with injectable transport, `test_certificate` asserting the PDF text layer, `test_faucet_decay`, `test_chemistry_mirror_parity` keeping Python↔Lua in lockstep), with a headless Playwright e2e in CI.

**The blocking gaps:**
- **Single-process in-memory authority** (`Bar` holds all live state in dicts; Fly pinned to `min_machines_running=1`). Two instances = two divergent bars; a restart drops every open round, vote, and seat.
- **No file locking on `world.json`** (zero `fcntl`/`flock`/atomic-rename; `_save()` is a plain `open(...,"w")` + `json.dump`). Concurrent writers interleave/truncate — the most concrete correctness bug blocking p2p.
- **Three drifting implementations of one ruleset** — divergent BFT quorum thresholds and **divergent fiber CID derivation** (PHP's bespoke base32 ≠ knitweb CID). A 5mart.ml knit and a Python knit of the same fact cannot converge to one state_root/UAL.
- **Wall-clock on the weave path** (`world.py` stamps `anchor_ts=int(time.time())` → feeds `tension.age_steps()` → tautness → SNAP/routing). A determinism leak on a *decision* path.
- **XP/level recomputed ad hoc** (`woven_count * XP_PER_WOVEN` in ~5 places, re-derived in PHP) — no single source of truth.
- **Identity insecure-by-design** (`sha256("molgang:device:"+id)` keys derivable from a localStorage UUID; the certificate *used to* print the private key in cleartext — since fixed: the public `/api/certificate` path is always redacted and bearer export is an explicit local-CLI-only flow). Still a hard blocker for anything value-bearing or adversarial until client-held keys land.
- **No type-checking/linting in CI; the only fully-persistent runtime (PHP) is the least tested; no concurrency/load tests.**

### 2.2 Target architecture (concrete)

A **pure, single-threaded, INTEGER-only ECS simulation** advances `world_CID(T) → world_CID(T+1)` from a **canonically-ordered log of signed actions**. Actions and the per-tick state-root propagate over knitweb gossip; Erlay reconcile (a) converges the per-tick action set (the lockstep barrier) and (b) fast-syncs state-component CIDs to joiners. **The world IS the content-addressed knowledge graph.** Every node is authoritative by re-derivation, so cheating reduces to forging signed actions (cryptographically blocked) or self-desyncing (detected via divergent `world_CID` and dropped). PoUW certificates gate expensive economic ops and provide replay-audit attestations for light clients. Async-actor networking and Django/Channels + Three.js rendering wrap the pure core but **never enter the sim path**; 5mart.ml is frozen as a thin read/relay light client.

**Determinism (extend `tension.py`'s discipline to the *whole* sim):**
- INTEGER / fixed-point everywhere on the sim path. Reuse the µPLS mental model: one micro-scale constant `FIX = 1_000_000` for non-token fractional quantities (tension decay, positions). Ban single-slash `/`; use an `idiv()` helper with a documented round-toward-zero rule (beware negatives) and `fix_mul` (multiply-then-shift).
- **Ordered application:** sort each tick's actions by `(tick, action_CID)`. CID ordering is deterministic, content-derived, tamper-evident — a leaderless total order, native to your CID layer.
- **Seedless RNG:** counter-based PRNG seeded only from on-graph data: `seed = H(epoch_CID || tick || entity_id || purpose)` (keyed BLAKE2 truncated to int). No shared mutable RNG state.
- **The set-iteration landmine:** `set` iteration is hash-randomized across processes (`PYTHONHASHSEED`); dicts keyed in network-receipt order are equally hazardous. **Rule: the sim never iterates an unsorted collection** — always `sorted(...)` by CID or integer id first. Lint for it.

**ECS, data-oriented (the on-chain-game industry converged here — MUD on EVM, Dojo on Starknet, both ECS):**
- Components are integer-keyed, sorted-by-entity-id arrays of plain ints → they serialize byte-identically → directly feed a byte-identity-sacred encoder → directly a CID. State diffs = "which component cells changed" = exactly what Erlay set-reconciliation wants.
- Systems are pure functions `system(component_views, ordered_actions) -> mutations` — no I/O, no clock, no RNG except the injected counter-PRNG.
- **Actors belong only at the node boundary** (one per peer connection, the gossip pump, the action mempool). Hard wall: actors do networking; the ECS sim is a single-threaded pure step.
- **NetworkX is a read-side projection, not the model.** Object-graph + float-friendly + non-deterministic iteration in places — keep it strictly in `view/`, built *from* the ECS component tables for the explorer UI. The authoritative graph lives in `has`/`is-a`/`contains` edge-component tables keyed by `(src_id, dst_id)`.

**State model on the knowledge graph (molgang's genuine differentiator):**
- Entities = graph nodes; relationships = typed edges. `is-a` → unit/building type classification (selects which systems apply). `contains` → spatial/hierarchical containment (map, inventory). `has` → ownership/composition (drives the `Owner` check for signed actions).
- Resources are INTEGER component balances (a resource = `N micro-units`); on-graph transfers are the *same* integer-conservation logic as token transfers — supply chains are typed edges with integer flow per tick. **One no-float invariant unifies economy and token.**
- **Fiber Tension is the supply-chain/logistics layer** (the killer mechanic): each `contains`/supply edge carries an integer tension `0..FIX` with thresholds → taut/slack/snapped. Flow-demanded-vs-capacity raises tension; overload → snapped → downstream starves. **Tension-weighted traversal = integer-weight Dijkstra** for goods/reinforcements — deterministic, replayable, float-free. The graph explorer becomes the actual play surface.
- **State commitment:** sorted component tables → per-table CID → Merkle root = `world_CID`. Per tick `world_CID(T) → world_CID(T+1)`. This *is* your save-game, sync unit, and anti-cheat checksum in one object.

**Language / runtime (honest assessment):**
- **Sim core: Python now, int-only.** Python ints are arbitrary-precision and exact → *more* deterministic than C int (no overflow UB, no width surprises). At tick cadence with thousands (not millions) of entities, pure-int Python is genuinely fine. Don't pre-optimize.
- **When measured to be too slow:** port the *specific* hot system (tension propagation, integer supply-graph Dijkstra) to (1) `numpy` int64 (easiest — but **watch int64 overflow vs Python bigint**, keep magnitudes bounded or it diverges from the reference) or (2) a small **Rust `pyo3`** module (best determinism + speed; explicit wrapping). Any second implementation is a hard gate on the dual-implementation determinism test.
- **Networking:** Python (existing FabricNode) — I/O-bound, determinism doesn't live here.
- **5mart.ml PHP/MySQL:** **freeze its scope.** Repurpose as a read-only explorer + signed-action relay; it forwards signed actions into gossip and renders `world_CID` projections. Don't grow game logic in PHP — that's a determinism dead-end. (It is also, today, the *only* fully-persistent runtime, which is why event-sourcing the Python core is what lets you safely demote it.)

**Front-end / 3D:**
- **Three.js (WebGL) primary**, for the graph-world (`3d-force-graph` sits on it; tension-colored fibers render naturally). **Phaser/canvas secondary** for 2D HUD/economy overlays. **Avoid Godot-web** for now (heavy WASM, pulls you toward a float/non-deterministic sim in the renderer).
- **The rule that matters more than the engine:** the renderer never runs game logic. It consumes `world_CID` projections/diffs over WebSocket and interpolates for visual smoothness only. **Visual interpolation is the one place floats are fine** — they never feed back into state. Optional client-side prediction (render the local player's likely action, reconcile on tick finalize) gives responsiveness without touching determinism. Django/Channels is the host + WebSocket bridge to the FabricNode, not in the sim path.

**Testing / CI (priority order):**
1. **Replay-determinism gate (crown jewel):** record a signed-action log; replay on two freshly-built nodes (and both impls if a hot-path fork exists) with randomized `PYTHONHASHSEED`; assert identical `world_CID` every tick.
2. **Float/clock/random lint:** AST check rejecting `float(`, single-slash `/` on sim types, `import time`/`random`, and unsorted `set`/`dict` iteration inside `sim/`. Make the existing no-floats rule *compiler-enforced* for the whole package via import-linter / custom AST check.
3. **Property tests (Hypothesis):** PLS conservation per tick (no mint/burn outside faucet + reward bank), settle idempotency, tautness monotonic under corroboration, CID stability under re-encode (encode→CID→decode→encode = same bytes — guards the byte-identity-sacred encoders).
4. **Fuzz the action validator** against malformed/malicious signed actions; assert deterministic rejection.
5. **Soak/divergence test:** N nodes gossiping K ticks under simulated loss/reorder; assert `world_CID` convergence and that an injected cheater is detected + dropped.
6. **Cross-runtime golden vectors:** feed the same proposal+votes to every surviving runtime; assert identical fiber CID, quorum verdict, state_root. Extend `test_chemistry_mirror_parity.py`'s exact idea from the chemistry table to the *settlement and CID* paths.

Plus operational hardening the repo lacks today: **ruff + mypy + a coverage gate**, PHPUnit wired into CI, structured relay metrics (pull lag, reject rate, fiber convergence) reusing pulse's metrics surface, and an indexed query layer to replace PHP's N+1 `state()` and Python's full-floor rebuild on every poll.

**Target package boundaries:**
```
molgang/
  sim/    # PURE: ECS components (int tables), systems (pure fns), tick step.
          # imports nothing from net/web/view; never imports time/random; no float.
  state/  # CID commitment: byte-identity-sacred encoders, Merkle world_CID,
          # FIX/idiv/fix_mul helpers.
  net/    # FabricNode glue: gossip pump, Erlay reconcile (action-set + state),
          # mempool, signed-action validation, tick-finalization rule.
  rng/    # counter-based PRNG seeded from CIDs only.
  view/   # read-only projections: NetworkX explorer feed, WebSocket diffs.
  web/    # Django + Channels host (no game logic).
  pouw/   # certificate engine bindings (replay-audit + action gating).
```

### 2.3 Migration path (each step ships independently, game stays playable; 1–3 are pure refactors that de-risk everything after)

1. **Carve the pure boundary (no behavior change).** Move tick logic into `sim/`; add the import-linter forbidding `net/web/networkx/time/random/float`. Demote NetworkX to `view/`. *Milestone: `sim/` runs headless from an action log.*
2. **Lock determinism.** Convert remaining float/clock sim fields to fixed-point; add `idiv`/`fix_mul`; replace `time.time()` in `world.py`'s weave path with the injected logical beat; swap `random` for the CID-seeded PRNG. Stand up the replay-determinism gate + float/clock lint. *Milestone: two nodes replay one log to identical `world_CID`.*
3. **CID-commit the state.** Component encoders on the byte-identity-sacred path; Merkle `world_CID`; `action_CID` canonical ordering. *Milestone: `world_CID` is the save/sync/checksum object.* **Fix the `world.json` race here** (atomic write + flock, or skip straight to the append-only event log).
4. **Wire the lockstep loop onto knitweb.** Gossip signed actions; Erlay-reconcile the per-tick action set; rounds-based (not wall-clock) tick-finalization; broadcast `tick_state_CID` for divergence detection. *Milestone: 2-node deterministic multiplayer over real gossip.*
5. **Cheat-resistance + PoUW.** Signed-action validation in `net/`; PoUW gating on economic ops; replay-audit certificates. *Milestone: injected cheater detected + dropped in the soak test.*
6. **Front-end split.** Three.js graph-world fed by WebSocket `world_CID` diffs via Channels; optional prediction; **freeze 5mart.ml as light client.** *Milestone: browser renders live multiplayer from projections only.*
7. **Scale honestly.** Profile the tick budget; *only if* exceeded, port the hottest system to numpy-int64 or Rust `pyo3`, guarded by the dual-impl determinism gate.
8. **Observability.** Add `world_CID`-agreement ratio (your single best production canary), reconcile-rounds-to-close, PoUW replay-cert coverage, and a Fiber-Tension heatmap to knitweb/monitor.

---

## 3. Dapp-Landscape Lessons

### 3.1 The single line that separates winners from losers (every category agrees)

**Emission-bootstrapped supply dies; verified, fee-backed, real-demand systems survive.** molgang's rare advantage: *the game manufactures genuine demand for PoUW*, and *integer-deterministic execution makes verification cheaper and more trustworthy than any floating-point DePIN can achieve.* Build the entire economy around those two facts, with a **published burn/mint ratio** as the public health metric.

### 3.2 Patterns to ADOPT

| Pattern | Source category | molgang application |
|---|---|---|
| **Burn-and-Mint Equilibrium (BME)** | DePIN (Helium) | Consumers **burn µPLS** to buy PoUW/certs; network **mints µPLS** to reward verified providers. Integers make the accounting *exact* (no rounding drift) — a genuine trust differentiator. Publish burn/mint; GEODNET's ~70% is the bar, below ~1.0 long-term is the death-spiral warning. |
| **Deterministic re-execution as the verification challenge** | DePIN (Filecoin PoRep/PoSt, Hyperbolic PoSP) | Your PoUW Certificate engine + byte-identity-sacred encoders let you *re-run the wire-encoded job and byte-compare the CID* — stronger than DePINs fighting float/timing nondeterminism. Sample a fraction of jobs, re-run, slash on mismatch. |
| **Restaking + slashing** | DeFi (EigenLayer) | Providers **stake PLS to back certificates**, slashable on a failed re-execution check. Makes verification *economic*, not just cryptographic; one stake can secure multiple knitwebs (your game modes are the first "AVSs"). |
| **Real-yield over emission-yield** | DeFi (GMX, Hyperliquid) | Route actual PoUW-consumption fees + molgang sinks to PLS stakers. **Never let the thing securing the system be reflexively the same as the thing speculated on** (the Terra/UST lesson). |
| **Receipt-token composability ("money-legos")** | DeFi (stETH, aTokens) | A completed **PoUW Certificate is your receipt token** — usable as in-game collateral, stakeable, tradeable as a claim on future compute. Turns one-shot work into a compounding economy; integer accounting = exact µPLS divisibility, no dust. |
| **Sub-token / per-subnetwork reward pools** | DePIN (HNT↔IOT/MOBILE) | Each **knitweb** gets its own reward-weighting settling back to base PLS — fund a new knitweb's cold-start without inflating the whole graph. |
| **Isolated risk, shared core** | DeFi (Aave V3, Morpho) | Per-knitweb reward pools + caps = blast-radius containment; a buggy/malicious knitweb can't drain base PLS or other knitwebs. |
| **EAS-style attestation record shape** | Identity (EAS) | Make every signed record canonical `(issuer_pubkey, schema_id, subject_CID, claim_bytes, revocation_ref)`. Schema-ize reputation, PoUW completions, and `has/is-a/contains` claims so they're one verifiable, *composable* object (encoders byte-identity-sacred per schema_id). |
| **Reputation = graph algorithm, not a stored counter** | Social/Identity (OpenRank, EigenTrust) | Run integer power-iteration over **taut** edges for centrality; snapped edges = trust revocation. Near-free — the substrate already holds the tension-weighted graph. |
| **Composite Sybil gate with a threshold** | Identity (Gitcoin Passport) | Integer-point score from stamps you can already prove (device-balance age, validated-cert count, anchored contributions, distinct attesters). Gate high-value actions behind a threshold. No external service needed. |
| **PoUW certs as POAP/Soulbound provenance badges** | Identity (POAP) | Non-transferable attestations that *feed* reputation — a peer's cert history is its on-graph résumé. |
| **Seaport-style offer/consideration orders + trait-level bidding** | NFT marketplace (OpenSea/Blur) | Replace fixed-price PoUW jobs: an order = "I give N µPLS + input CIDs, I want this result CID meeting these conditions." Add "any worker with reputation ≥ T and cert-class C" bids — decouples demand from a specific peer, deepens the market. One integer-clean order struct. |
| **Protocol-level reward splits on settlement** | NFT marketplace (Zora) | Auto-split each PoUW settlement among worker + referrer + the relay/gossip peer that carried it. Every job becomes a referral; rewards the gossip layer. Integer division with deterministic remainder handling. |
| **Inline executable affordances ("Frames")** | Social (Farcaster) | **Highest-leverage UX borrow.** A node carries a button a viewer can trigger inline: accept a PoUW job, tension/slack a fiber, propose an edge, tip µPLS. Under the hood: a pre-authorized signed action. |
| **Hybrid governance from day one** | DAO (Optimism, Sky) | Emissions schedule, burn/mint targets, slashing params, knitweb reward weighting *will* need governance. **Pure token-weight reliably becomes plutocracy at <2% turnout.** Off-chain signal + integer on-chain execution + a **block/epoch-counted (never wall-clock) timelock + guardian council.** |
| **RetroPGF** | DAO (Optimism) | "Reward *past, measured* impact, not promises" *is* proof-of-useful-work philosophy. Fund knitweb-builders/providers retroactively for verified contribution — the same primitive you already meter. |
| **Quadratic funding/voting** | DAO (Gitcoin) | Dilute whales in graph-priority/community decisions — integer square-root checks, no floats. |
| **Session keys + sponsored first action** | Onboarding (AA/4337/7702, Cartridge) | Extend your existing **wallet-signed QR onboarding**: after QR pairing issue a scoped, time-boxed session key (no per-move prompt — essential for a real-time Spiral engine), and **sponsor the first PoUW/first edge** so step one costs zero µPLS. |

### 3.3 Patterns to AVOID

- **Uncapped/weakly-sunk reward token** (SLP) — structural inflation, the canonical death spiral.
- **Time-based / presence-based yield** ("X PLS per hour") — your no-wall-clock invariant *already forbids it*; treat that as anti-inflation armor.
- **"Projected earnings" UI** — the instant you show it, you've reintroduced the ponzi.
- **Scholarship/guild rental economies** optimized for extraction (YGG-style) — zero retention.
- **"Buy PLS to start playing" wall** — the #1 P2E conversion killer.
- **Pure friend.tech tokenomics** — the bonding-curve access *mechanic* is a fine onboarding hook, but pair it with utility retention (unlock tension boosts, cert privileges, traversal priority), never pure speculation.
- **Speculative-land trap** (Sandbox/Decentraland — huge land valuations, near-zero players) — if molgang ever sells nodes/regions, they must be **used in a return loop**, not held for appreciation.
- **Delegation monopolies & treasury-in-native-token concentration** — cap/decay delegation; hold treasury in something not purely reflexive to PLS price.
- **Pure fully-on-chain real-time** — Dark Forest/Sky Strife/Eternum all learned it's painful; keep rendering and per-frame logic off the value path.

### 3.4 Named leaders per category (for reference)

| Category | Named leaders | The one lesson |
|---|---|---|
| DeFi | Lido, EigenLayer, Aave, Sky/Maker, Pendle, Uniswap, Curve, GMX/Hyperliquid | Restaking + real-yield; isolate risk; oracle is the trust boundary |
| DePIN | Filecoin, Render, Helium, Akash, io.net, Grass, Hyperbolic, GEODNET | Bootstrap *demand*, not just supply; verification is the whole ballgame |
| DAO | Uniswap, Optimism, Arbitrum, Sky, Nouns, Gitcoin, ENS | Hybrid > pure token-weight; RetroPGF; quadratic vs whales |
| GameFi | Axie (the warning), Splinterlands, Gods Unchained, Big Time, Pixels | Sell *content/utility*, not yield; play-and-own; fun-without-token |
| FOCG / autonomous worlds | Dark Forest, MUD/Sky Strife, Dojo/Realms:Eternum, Loot | ECS-as-protocol; hidden-info-on-transparent-substrate; ship a game, not just a primitive |
| Social | Farcaster, Lens, Nostr, friend.tech (the warning) | Identity scarce / content cheap+gossiped; edges as objects; Frames |
| NFT marketplace | OpenSea/Seaport, Blur, Magic Eden, Zora, Sudoswap | Order = offer/consideration bundle; trait bids; reward splits |
| Identity/Reputation | ENS, World ID, Gitcoin Passport, POAP, EAS, OriginTrail | Attestation is the universal record; reputation = graph traversal; anchor hash, store data off-chain |

**Closest analog:** knitweb *is* a compute-DePIN with a game-driven demand engine bolted on, and your `:8990` knowledge graph is the **moat** — composable knowledge (`has/is-a/contains` = "money-legos for knowledge") that can't be cheaply forked away. Optimize so *adding verified structure to the graph* is the most-rewarded action.

---

## 4. Game-Design Playbook — the core economy↔conflict loop

### 4.1 The one core loop

molgang's destiny (the canon's verdict) is to be the **Factorio/Settlers of knowledge-graphs**: a transparent, deterministic, emergent logistics game where **Fiber Tension is the belt/power-grid, the `:8990` graph is the board, PoUW is the spice, and content-addressed knitwebs are the artifacts players own and show off.**

The moment-to-moment skill is **real-time four-way µPLS allocation** (Red Alert's eco/army/tech/defense, in graph terms):

```
        ┌──────────────── YIELD (harvest) ────────────────┐
        │  PoUW worker trips over graph regions → dock      │
        │  → integer µPLS settlement (the "refinery")       │
        └──────────────┬───────────────────────────────────┘
                       │ µPLS allocated across:
   ┌───────────────────┼───────────────────┬──────────────────┐
   ▼                   ▼                   ▼                  ▼
 CONTEST            TECH                DEFENSE            (re-invest)
 reconcile races   prove has/is-a/      defensive certs   more workers /
 & claim           contains motifs →    + reputation      sub-root CIDs
 challenges on     unlock higher-yield  moats; cost       near rich regions
 shared graph      regions/capstones    tension slack
   │                   │                   │                  │
   └─── all spend µPLS that was SUNK by someone who got value ─┘
                       │
              FIBER TENSION couples it all:
   demand-vs-capacity raises tension → taut fibers slow YIELD/
   query/gossip (graceful tempo tax) → snapped fibers cut supply
   downstream (Settlers starvation) & burn staked PLS (the sink)
```

**Economy↔conflict, restated in the Settlers spirit:** don't bolt on a separate "combat" system. *War is the economy's P&L statement.* The apex competitive good — **ratification/consensus weight over contested CIDs** — is a *manufactured top-of-pyramid item* (the molgang knight = settler + sword + shield + gold → deep-derivation cert + the scarce universal input + staked PLS). Whoever ran the deeper, healthier production pyramid simply *has more* ratification inventory to deploy in a dispute. "Attacking" = committing that inventory to contest a rival's claimed graph-region. molgang stays a *pure economy game* where conflict is decided by who built the better supply chain.

### 4.2 Mechanic-by-mechanic mapping onto knitweb primitives

| # | Source mechanic | knitweb primitive | Concrete molgang design |
|---|---|---|---|
| 1 | **Harvester loop** (collect → dock → settle) — RA/Dune II | PoUW compute yield → µPLS | Model yield as **discrete trips** with explicit fill-capacity and an explicit **settlement event** ("dock"), not a passive per-tick trickle. Discrete trips give you harassability + the dock bottleneck + clean integer settlement. Passive income tempts wall-clock dependence — avoid it. |
| 2 | **Ore (renewable) vs gems (depleting premium)** — RA | Graph topic clusters vs scarce certificate jobs | Two yield grades, **one currency (µPLS)**: a region yields replenishing work as the graph grows; high-value "gem" certs (first-to-prove a relationship, a contested CID resolution) pay a multiplier but deplete once claimed. Texture without a second UI. |
| 3 | **Per-refinery dock bottleneck** — RA | Per-node settlement-rate cap | Structural cap forces **geographic spread** across graph regions — doubles as a Sybil/centralization brake. |
| 4 | **Power as graceful global scalar** — RA | **Fiber Tension** (taut/slack/snapped) | **The cleanest cross-over.** One **tension bar per player-subgraph** in the UI: green (slack) → red (taut) → black (snapped). Taut → *systemic, graceful* slowdown of yield-rate / query-latency / gossip — never a hard stop (preserves comeback potential). **Tension drives tempo, NEVER settled value** — settlement stays integer-exact regardless of tension. |
| 5 | **Tech gated by placed structures** — RA | `has/is-a/contains` DAG | Capability unlocked by **proven graph relationships (CIDs)**, not a research bar — so tech is *in-graph, visible, and adversarially destroyable* (challenge a relationship → it snaps → unlock revoked). Keep it **shallow, 4–5 tiers** for legibility. |
| 6 | **Production pyramid** (raw→intermediate→finished, fixed ratios; apex touches 12–15 tiers) — Settlers | Certified-derivation depth over CIDs | A finished-good CID is produced by a **PoUW cert certifying it was derived from input CIDs** (`contains` = bill-of-materials, output `is-a` recipe type). Designate **one scarce universal input** (a "verification credit" almost every recipe consumes) = the deliberate central chokepoint, importing Settlers' "read which bottleneck binds" tension for free. The apex good *transitively means a whole working sub-economy.* |
| 7 | **Road/flag/carrier logistics** (bucket-brigade, finite flag slots, backpressure) — Settlers | gossip/reconcile links + mempool buffers | A reconcile link = a road segment; a CID hop = a carrier; a node's reconcile/mempool buffer = a flag. **Bounded per-link outboxes**: when full, upstream stalls and backpressure **propagates backward visibly** (a gameplay feature, not just ops). **Tiered carriers**: thin gossip for cold CIDs, **batched Erlay reconcile auto-promoted** (integer item-count trigger, never wall-clock) on hot arteries = the "donkey road." |
| 8 | **Warhead × armor counter matrix** — RA | Integer (operation × claim-type) effectiveness matrix | "Combat" = contested graph operations (reconcile races, challenges, policing). Define an **integer effectiveness matrix** (cheap claim-workers ↔ heavy reconcile/re-anchor ↔ fast gossip ops ↔ defensive certs), **version it as a CID and gossip it** so every node computes outcomes byte-identically. The triangle is the tutorial story; the matrix is the balance lever. |
| 9 | **RPS-breaking signature units** (Tanya/Engineer/Spy) — RA | Signature knitweb plugin abilities | Ship **2–3** memorable verbs: a "Spy" that forces-reveals a rival's recently-changed subgraph; an "Engineer" that captures an under-defended node by re-rooting its CID; a "Thief" that diverts a settlement. Rare, high-skill, RPS-breaking. |
| 10 | **Asymmetric factions on a shared skeleton** — RA/StarCraft | **knitwebs as factions** | The single highest-leverage RA import. Identical pulse substrate (CIDs/PoUW/µPLS/tension/gossip = the byte-identity-sacred core); each knitweb exposes a **distinct verb-set + one capstone**. Launch with **two well-differentiated archetypes** (a "force/economy" heavy-reconcile knitweb vs an "information/maneuver" fast-gossip+intel knitweb), not six samey ones. Asymmetry = *different verbs*, not stat reskins. |
| 11 | **Superweapon capstone** (cooldown) — RA | Top-tier capstone certificate | One **cooldown-gated, game-altering action** per knitweb (re-anchor authority over a CID neighborhood; a challenge-immunity window). Cooldown counted in **proof-counts/graph-events, not seconds.** |
| 12 | **MCV deploy + expansion bases** — RA | Genesis CID anchor + sub-root CIDs | A player's **genesis CID** seeds their subgraph (identity anchor; device-balance persistence hangs off it). Make **"found an expansion sub-root near a rich cluster"** an explicit verb (shorten worker trips toward yield). Re-anchoring is powerful but exposes you mid-move — gate it with CID-stability sign-off. |
| 13 | **Territory ratchet** (border tower → claim radius → unlock resources) — Settlers | Graph-region claiming via PoUW + PLS stake | Spend **PoUW cert + PLS stake** to occupy a CID neighborhood, unlocking the right to certify/derive over it. Regional yield funds the next claim — an irreversible production→stake→claim→more-production ratchet, giving PLS a clean integer **sink** (claiming) and **source** (regional yield). |
| 14 | **Fog of war + Gap Generator** — RA | p2p partial-view (discovery/anti-entropy/reconcile) | A node's local view *is already* fog — un-reconciled subgraph = the shroud. Surface "reconciled vs not" as an explicit fog layer; a **tech+tension-gated "graph radar"** strategic overview; a knitweb whose signature is **selective non-disclosure** (Gap Generator) — visibility manipulation only, never wire-encoding tampering. Borrow Dark Forest's *real* lesson — **proof-gated relationships** (prove a fiber is taut / a path exists *without revealing the whole subgraph*) for asymmetric-information depth. |
| 15 | **Build queue + tempo** — RA | Per-node operation queue | Metered µPLS→ops conversion, but **progress gated by PoUW completion, not wall-clock** (the invariant fix). Parallelism is purchasable (more worker capacity = a second War Factory). |
| 16 | **Few knobs, big leverage** (priority list + tool dial) — Settlers | Two control dials | Resist dashboard sprawl. Exactly two: a **CID transport-priority list** (which CID classes carry first under contention) and a **production-mix dial** (bias the cert engine toward the currently-starved recipe to self-heal a snapped chain). |
| 17 | **Campaign missions vs skirmish/MP** — RA | 5mart.ml scenarios vs pulse sandbox | **Ship both.** Authored, scripted single-player scenarios over curated subgraphs on 5mart.ml (each isolating one mechanic — a harvester-only economy, a tension-management drill, a claim-defense) for onboarding; the open p2p sandbox on pulse for retention. Don't launch multiplayer-only. |
| 18 | **Moddability → longevity** (WC3 editor → MOBAs) | knitweb-authoring API/editor | The plugin model *is* the mod system. Make **knitweb-authoring a first-class, documented path** — your CnCNet / Workshop / World-Editor moment. Build it *after* the core loop is fun. |

### 4.3 Anti-inflation economic spine (tie it together)

- **PLS enters only against a proof of useful work**, amount bounded by the work's verified value. **Faucets tied to verifiable work, never time/presence.**
- **Tension upkeep is a sink:** holding a fiber taut costs µPLS/staked PLS; let go → slack (cheaper, less useful); over-stress → **snapped, burning the stake.** A risk/reward economy with a built-in burn.
- **Tension-weighted traversal is a market:** charge µPLS to traverse high-value (high-tension) paths; route the fee to whoever pays upkeep to keep that fiber taut. Players earn by **providing a useful service to other players**, not by minting — every PLS earned was a PLS someone spent for value (the Big Time model in graph form).
- **A healthy molgang economy:** snaps + traversal fees + claim costs + certificate costs **≥** certificate emissions, tracked as a running integer invariant and shown publicly. **The Spiral engine is your emissions-decay curve** — tie reward emission to Spiral progression so it front-loads then tapers as a *game arc* (difficulty progression), not a jarring tokenomics cliff.
- **Acid test:** *if PLS went to zero, would building, tensioning, and traversing the graph still be fun?* Design so the answer is yes.

---

## 5. Risks & Anti-Patterns

**A. Unsustainable token loops (the GameFi graveyard).**
- *Risk:* recreating the Axie/SLP death spiral — value that requires perpetual new entrants. *Tells:* uncapped/weakly-sunk reward token, earnings denominated in a volatile token, yield-as-the-core-loop (attracts bots/mercenaries with zero retention), faucets >> sinks, success measured by token price.
- *Mitigation (mostly already structural):* PLS sunk-to-act and minted-only-against-verified-PoUW; **no time-based yield** (your no-wall-clock invariant enforces this — lean on it); published burn ≥ mint integer invariant; **never ship a "projected earnings" UI**; no scholarship/rental extraction layer; no "buy PLS to start" wall.

**B. Determinism pitfalls (lockstep desyncs).**
- *Risk:* the wall-clock leak on `world.py`'s weave path is *live today* and feeds SNAP/band decisions — two nodes already diverge. Beyond it: `set` hash-randomization across `PYTHONHASHSEED`, dict iteration in network-receipt order, float creep, and **two language implementations of "the same" integer math diverging** (the #1 lockstep-desync cause; especially numpy int64 overflow vs Python bigint).
- *Mitigation:* the `sim/` purity wall + AST lint + the replay-determinism CI gate as a *hard merge gate*; never iterate an unsorted collection; canonical `(tick, action_CID)` ordering; one canonical implementation, fork to a second language *only* behind the dual-impl gate.

**C. The three-implementation drift (the silent killer).**
- *Risk:* PHP's bespoke fiber CID + divergent quorum threshold mean cross-runtime "one shared web" is *already broken*. Every new feature added in three places widens the gap.
- *Mitigation:* collapse to one rules core *first* (§2.3 step 1–3); cross-runtime golden-vector test on the settlement+CID path; freeze PHP/Lua to light clients. This is a **design lock-in risk on a young repo**, not legacy rot — cheap to fix now, expensive later.

**D. Scope creep.**
- *Risk:* the canon warns explicitly — defer asymmetry (content layer, not foundation), moddability (only pays off *after* you have players worth modding), and 3D spectacle polish (minimally watchable now, invest heavily only once matches are competitive). Building six knitwebs, a full skill tree, or a fancy renderer before the core loop is fun is the classic Star Atlas "perpetual coming-soon" trap.
- *Mitigation:* ship the pure refactors (§2.3 steps 1–3) and **two** faction knitwebs first; everything else is P2. Respect the "strict minimum first, rest optional" principle.

**E. Identity & Sybil.**
- *Risk:* today's `sha256(device_id)` keys are fully derivable (the certificate cleartext-key leak is since fixed — public path always redacts; bearer is CLI-only) — still a hard blocker for any value-bearing or adversarial multiplayer; Sybil farming ("one operator, 100 nodes") is the universal DePIN tax.
- *Mitigation:* client-held keypairs (WebCrypto/passkeys), server sees only pubkeys; certificate proves work by *signing a challenge*, never printing the key; PoUW re-execution + stake-and-slash makes faking expensive; composite Sybil-gate score for high-value actions; per-node settlement-rate cap as a structural brake.

**F. Liveness/governance.**
- *Risk:* a node falling behind stalls the lockstep barrier; pure token-weight governance → plutocracy at <2% turnout; flash-borrow governance attacks.
- *Mitigation:* rounds-based (not wall-clock) tick-finalization — close a tick when the action-set CID is stable across a quorum *or* after N gossip rounds, late actions roll forward; hybrid governance with block/epoch-counted timelocks + a guardian council; quadratic mechanisms vs whale capture.

---

## 6. Prioritized Roadmap (hand-ready for Codex/agents)

Each P0 item ships independently and leaves the game playable; P0 is almost entirely pure refactors that de-risk everything after. Ties to existing modules and the open issues noted in the dossier.

### P0 — Foundation (must land before any real multiplayer; collapses the drift + locks determinism)

| ID | Task | Touches | Done-when |
|---|---|---|---|
| P0-1 | **Carve the pure `sim/` package**; move tick/settlement logic out of `bar.py`/`game.py`; add import-linter forbidding `net/web/networkx/time/random/float` in `sim/`; demote NetworkX (`explorer.py`/`graphx.py`) to `view/`. | `bar.py`, `game.py`, `explorer.py`, new `sim/` | `sim/` runs headless from an action log; CI import-linter green. |
| P0-2 | **Fix the `world.json` data-loss race** — atomic write + `flock` (or jump straight to the append-only event log in P0-4). | `world.py` `_save()` | Concurrent-writer test no longer truncates. |
| P0-3 | **Remove wall-clock from the weave/decision path** — replace `anchor_ts=int(time.time())` (3 sites) with the injected logical beat (`Pulse.beat`/`Round._tick`); make the World's weave timestamp injectable like the Bar's `clock=`. | `world.py`, `tension.py` consumers | Two nodes weaving the same fiber get identical tautness/SNAP. |
| P0-4 | **Event-source the state** — append-only log of signed, content-addressed actions; current state = fold over the log; reuse `merge.py` union-and-sum-confirmations as the CRDT merge. | new `state/`, `merge.py`, `bar.py` | Restart is lossless; two logs reconcile by CID. |
| P0-5 | **CID-commit state** — byte-identity-sacred component encoders, Merkle `world_CID`, `action_CID` canonical `(tick, action_CID)` ordering; fixed-point `FIX`/`idiv`/`fix_mul` helpers. | `state/`, `tension.py` | `world_CID` is the save/sync/checksum object. |
| P0-6 | **Replay-determinism CI gate + float/clock/random lint** (AST). Extend `test_chemistry_mirror_parity.py`'s idea to the settlement+CID path (cross-runtime golden vectors). | `tests/`, `.github/workflows/ci.yml` | Two nodes replay one log to identical `world_CID` under randomized `PYTHONHASHSEED`; merge-blocking. |
| P0-7 | **Unify the rules core / kill drift** — make Python the single source of truth; freeze PHP (`php/src/Bar.php`) + Lua to light clients; replace PHP's bespoke `bafyrei…` CID and `max(2, 2*committee/3+1)` quorum with the canonical path (or have PHP relay signed actions instead of computing settlement). | `php/src/Bar.php`, `roblox/`, `bridge/` | A 5mart.ml knit and a Python knit of the same fact share one fiber CID + one verdict. |
| P0-8 | **Cryptographic identity** — client-held keypairs (WebCrypto/passkeys), server sees only pubkeys; certificate signs a challenge instead of printing the private key. | `certificate.py`, `game.py` `from_device`, `webserver.py` | No private key in the PDF; keys not derivable from a localStorage UUID. |

### P1 — Multiplayer + the core game loop

| ID | Task | Touches | Done-when |
|---|---|---|---|
| P1-1 | **Lockstep loop on knitweb** — gossip signed actions; Erlay-reconcile the per-tick action set; **rounds-based (not wall-clock) tick-finalization**; broadcast `tick_state_CID` for divergence detection. | `relay_sync.py`, `net/`, FabricNode glue | 2-node deterministic multiplayer over real gossip. |
| P1-2 | **Cheat-resistance + PoUW gating** — signed-action validation; PoUW gating on economic ops; **replay-audit certificates** ("I re-simulated tick T → `world_CID = X`"). | `net/`, `certificate.py` (`pouw/`) | Injected cheater detected + dropped in the soak test. |
| P1-3 | **Core economy loop** — PoUW yield as **discrete trips → integer "dock" settlement**; per-node settlement-rate cap; the **four-way µPLS allocation** (yield/contest/tech/defense). | `game.py`, `sim/` | A solo player (with NPC bots for quorum) plays the full loop. |
| P1-4 | **Fiber Tension as the power grid** — one **tension bar per subgraph**; taut → graceful systemic slowdown (tempo only, value untouched); snapped → downstream starvation + staked-PLS burn; tension wired to *flow* (integer demand/throughput over an item-count window). | `tension.py`, `world.py`, `view/` | Most-taut-fiber "current bottleneck" HUD live; tension never changes settled µPLS. |
| P1-5 | **Tech DAG + production pyramid** — capability gated by proven `has/is-a/contains` motifs (shallow 4–5 tiers); certified-derivation depth; **one scarce universal input** as the chokepoint. | `world.py`, `game.py`, `explorer.py` | Unlock DAG renders in `:8990` with greyed-out prerequisites. |
| P1-6 | **Two faction knitwebs** — a "force/economy" heavy-reconcile archetype + an "information/maneuver" fast-gossip+intel archetype, each with one capstone; **2–3 signature plugin verbs** (Spy/Engineer/Thief). | new knitweb plugins | Two knitwebs play with distinct verbs, not stat reskins. |
| P1-7 | **Graph-region claiming ratchet** — PoUW cert + PLS stake to occupy a CID neighborhood; regional yield funds the next claim (clean integer sink/source). | `game.py`, `world.py` | Claim → unlock → yield → re-claim loop closes. |
| P1-8 | **Onboarding** — session keys + sponsored first action on top of the existing wallet-signed QR; **no "acquire PLS first" wall**; first-session fun in <60s. | `webserver.py`, `web/`, `php/public/` | New player does something fun with no seed phrase / no token purchase. |
| P1-9 | **Burn/mint invariant + published health metric**; emission tied to Spiral progression (front-load→taper as a game arc). | `game.py` (`faucet_micropulses`), monitor | Running integer `sinks ≥ faucets` invariant visible. |
| P1-10 | **Animate the `:8990` explorer** — in-flight CIDs as moving objects on edges, thickness = throughput, color = tension; bounded per-link queues with visible backpressure. | `explorer.py`, `graphx.py`, `web/` | The explorer is the play surface, not a debug view. |

### P2 — Polish, scale, longevity (only after the core loop is fun)

| ID | Task | Notes |
|---|---|---|
| P2-1 | **Three.js graph-world front-end** fed by WebSocket `world_CID` diffs via Django/Channels; optional client-side prediction; renderer runs zero game logic. | README defers Channels — this is the increment. |
| P2-2 | **Marketplace upgrade** — Seaport-style offer/consideration PoUW orders, trait-level bidding ("any worker with rep ≥ T, cert-class C"), Zora-style reward splits (worker/referrer/relay). | Integer order struct, deterministic remainder handling. |
| P2-3 | **Reputation as graph algorithm** — integer power-iteration (EigenTrust) over taut edges; snapped = revocation; composite Sybil-gate score for high-value actions. | Near-free; substrate already holds the weighted graph. |
| P2-4 | **EAS-style attestation record shape** for all signed records; PoUW certs as POAP/Soulbound provenance badges feeding reputation. | Schema-ize reputation + relationship claims. |
| P2-5 | **Inline "Frames" affordances** on nodes (accept job / tension fiber / propose edge / tip µPLS as pre-authorized signed actions). | Highest-leverage UX borrow. |
| P2-6 | **Authored scenarios on 5mart.ml** (one mechanic each) for onboarding; pulse sandbox for retention. | Don't launch MP-only. |
| P2-7 | **Hybrid governance** — off-chain signal + integer on-chain execution + **block/epoch-counted** timelock + guardian council; RetroPGF for knitweb-builders; quadratic funding for graph-priority votes. | Never pure token-weight. |
| P2-8 | **Scale honestly** — profile tick budget; *only if* exceeded, port the hottest system (tension propagation / integer supply-graph Dijkstra) to numpy-int64 or Rust `pyo3`, **guarded by the dual-impl determinism gate** (watch int64 overflow vs bigint). | Don't pre-optimize. |
| P2-9 | **knitweb-authoring API/editor** (the mod system / CnCNet moment) — only once there are players worth modding. | Decade-defining multiplier; deliberately last. |
| P2-10 | **Observability** — `world_CID`-agreement ratio (best canary), reconcile-rounds-to-close, PoUW replay-cert coverage, Fiber-Tension heatmap, relay metrics (pull lag, reject rate) on knitweb/monitor; ruff+mypy+coverage gate; PHPUnit in CI. | Operational hardening the repo lacks today. |

**The one-line strategy:** collapse molgang's three drifting state models into one CID-addressed, integer-deterministic, event-sourced core (P0); build the Red-Alert-×-Settlers economy↔conflict loop natively on knitweb primitives with **Fiber Tension as the belt, the graph as the board, PoUW as the spice, and knitwebs as factions** (P1); then layer asymmetry, marketplace depth, governance, scale, and mods on top (P2). PLS is always a medium-and-stake, never the prize — and the no-wall-clock invariant you already enforce is your strongest anti-inflation guarantee.
