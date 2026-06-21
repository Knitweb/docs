# molgang — Development Plan

### A peer-to-peer chemistry knowledge game on the Knitweb Pulse substrate

**Subtitle:** Building a top-10, fully serverless, agentic-LLM dapp that ships its own source code over the Web it runs on

| | |
|---|---|
| **Document** | Development Plan & Architecture Programme |
| **Flagship knitweb** | molgang — chemistry (*scheikunde*) bar game |
| **Substrate** | Knitweb Pulse (`knitweb/pulse`) — pure-Python, stdlib-only P2P crypto Web |
| **Components** | PULSE (engine) · LENS (reasoning lobe) · MONITOR (`knitweb_monitor`) · VBANK (governance/treasury) |
| **Token** | PLS — PoUW-minted activity-accounting unit (no premine) |
| **Status of substrate** | ~20.7k LOC pure-Python · 102 property suites · 1183/1184 tests passing |
| **Plan horizon** | P0 Foundation → P1 Serverless MVP → P2 Agentic + Economy → P3 Scale & top-10 push |
| **Date** | June 2026 |

---

## Executive Summary

### The vision

molgang is a chemistry bar game in which players — humans and agentic LLMs alike — *knit* chemistry concepts into a shared, content-addressed knowledge graph and earn **PLS** for contributions that other players verify. Its slogan is exact and load-bearing: **the game *is* the protocol.** Every move is a real protocol operation, never a re-skinned database write. Forming a chemical bond is a real **Knit** (a dual-signed, two-party ledger transfer); a molecule's growing chain is a real **Fiber** (an immutable account-state commitment); classmates voting on a bond stake real pulses and settle a real BFT **quorum**; the confirmed-chemistry graph is anchored to OriginTrail as a verifiable UAL. The game is not a wrapper around a blockchain — it is a domain plugin (an L5 *knitweb*) that imports one engine and re-implements nothing.

The ambition of this plan is to take molgang from where it is today — a ~470-node multilingual explorer graph running partly as PHP and static assets on shared hosting (5mart.ml), plus a working pure-Python playable client with live P2P peers — to a genuinely **top-10-calibre dapp** distinguished by three properties that almost no shipping application combines:

1. **Fully serverless / P2P.** No central backend. The chemistry Web replicates over the Pulse gossip mesh (Erlay set-reconciliation, a gossipsub mesh, an anti-entropy convergence backstop), reaches into the browser, and survives NAT — there is no database an operator could read, no status page to trust, no upgrade key to surrender.
2. **Self-hosting / self-distributing.** The dapp **ships its own source code over the Web it runs on.** Application code and assets become content-addressed (CIDv1, dag-cbor/sha2-256) records, fetched and verified peer-to-peer by hash. molgang is not deployed *to* a Web; it *is* records on the Web, and so is its own client.
3. **Agentic LLMs as first-class players.** NPCs are not chatbots bolted onto a UI. An agentic NPC knits the same canonical-CBOR record, signs with the same secp256k1 key, settles the same PLS Knit, passes the same Sybil-resistant **personhood** gate, and is held to the same verification quorum as a human at the bar. The reasoning lobe — **LENS** — produces *provenance-cited* answers grounded in the converged graph, and its distilled relation bundles re-enter the economy as verifiable PoUW jobs.

### The thesis

The structural claim beneath the product is this: **an autonomous agent that earns, reasons, and acts on behalf of others is, mechanically, a tiny economy — and an economy needs one substrate that gives it verifiable compute, durable provenance, unique identity, and exact micro-settlement as facets of a single content-addressed Web.** The mainstream "agentic web" of 2026 assembles those four properties from four unrelated systems glued together by trust. Knitweb Pulse provides them as one coherent stack: the *fabric* (the Web) is provenance; **PoUW** is verifiable compute; **personhood** is unique identity; the integer-only **Braid/Knit/Fiber** ledger is exact micro-settlement; and **PLS**, minted *only* as a demand-gated reward for verified-useful-work and now being bound to the **Pulse** heartbeat, is the unit that ties activity to money. molgang is the proof that this substrate carries a real consumer product, not just a protocol test harness — the first end-to-end knitweb.

### What is built vs. what is to-build

We are deliberately honest about maturity. The substrate's *spine* is real and well-tested; its *body* — the parts a top-10 consumer dapp needs — is the work this plan funds.

| Already production-grade (ACTIVE) | Adapter-only / scaffold | Stubbed / to-build |
|---|---|---|
| `core` (canonical/crypto/pulse), full integer ledger (Braid/Knit/Fiber), p2p base/transport/wire/reconcile/mesh/kademlia, fabric Web + FabricNode, all PoUW verify/quorum/committee/dispute/collateral/escrow, token mint, **VBANK** (poll/ranked/liquid/tally), crowdfunding, **personhood** gate | `personhood.verifier` (VC/ZK backend injected), `pouw.scheduler`/`marketplace` (not in a live epoch loop), synaptic/anchor/OriginTrail (deterministic, minimal), gateway/CLI/sdk (thin), `interpret`/LENS (deterministic retrieve + gated distill, float-quarantined) | epoch-bound issuance (Pulse↔mint seam, **open**), `knitwebs/chemistry` plugin molgang needs (`__init__.py` only), the agentic LENS player loop, MONITOR-as-product, the browser-P2P transport, the self-distributing code channel |

The single most important open seam is architectural, not cosmetic: **Pulse is not yet bound to issuance.** The heartbeat chains and verifies, but `token.mint` caps emission by escrow and an optional `max_supply`, *not* by epoch. Closing that seam — an `epoch_mint_cap` carried on the Beat, gating `Treasury.reward_verified_work` to per-epoch escrowed demand — turns Pulse from a decorative clock into the monetary governor and completes the activity→money loop the entire stack exists to express. It is buildable entirely within `core/pulse.py`, `token/mint.py`, and `pouw`, with no p2p surface to race.

### The ask

This plan requests resourcing for a four-phase programme (**P0 Foundation → P1 Serverless MVP → P2 Agentic + Economy → P3 Scale & top-10 push**) whose guiding principle is **continuity, not rewrite.** Concretely, the funded work is: (1) close the Pulse↔mint epoch seam and fix the one RED Erlay byte-budget interop test; (2) build the ~10 new packages catalogued in the master table below — the `chemistry` knitweb, the browser-P2P transport and self-distributing code channel, the LENS agentic player loop, MONITOR-as-product, and the supporting glue — each as its own versioned package with its own test suite; (3) ship a serverless MVP, then the agentic economy, then a measured top-10 push. The honest framing on the token is maintained throughout: **PLS is an activity-accounting unit, not a speculative instrument**, and every surface is designed to keep it that way. The remainder of this document substantiates the market case, the architecture, the role of each component, the gap analysis, the tokenomics, and the milestone-by-milestone path to a top-10 dapp.

---

## Table of Contents

| # | Chapter | Focus |
|---|---|---|
| 1 | Introduction & vision | The molgang bet; reader's map; what "the game *is* the protocol" means |
| 2 | Market & Landscape: the path to a top-10 dapp | Defining "top-10"; fully-on-chain/serverless, agentic-LLM payments, decentralized knowledge; competitor comparison |
| 3 | The Knitweb Thesis: a core P2P substrate for agentic-LLM dapps | Why one substrate for verifiable compute + provenance + identity + settlement |
| 4 | Molgang Product Deep-Dive | The game as it exists today; target loops, progression, collectibles |
| 5 | The serverless / P2P architecture: a dapp with no server that ships its own code | Browser gossip, content-addressed assets, NAT traversal, source-code-included |
| 6 | Role of PULSE (the engine) | The substrate seam molgang binds to; what PULSE must still add |
| 7 | Role of LENS (the agentic reasoning layer) | The reasoning lobe; deterministic retrieve + gated distill; provenance-cited answers |
| 8 | Role of MONITOR (observability & trust) | Manufacturing trust from the protocol; locally verifiable evidence |
| 9 | Role of VBANK (governance, treasury & the PLS economy) | The constitutional layer; deterministic, auditable community decisions |
| 10 | Agentic LLM integration: NPCs, verifiable inference & the agent economy | NPCs as first-class peers; verifiable inference; the agent economy |
| 11 | Gap analysis: the missing modules that need their own package | From spine to body; package-by-package gap inventory |
| 12 | Tokenomics, incentives & go-to-market | The PLS unit; PoUW minting; Sybil resistance; staged GTM |
| 13 | Technical implementation plan & source-code structure | Repo/package layout; browser P2P stack; CI discipline; API seams |
| 14 | Roadmap & milestones to top-10 | P0→P3 phases with entry/exit criteria and owners |
| 15 | Risks, mitigations & conclusion | Seven risk classes, architected mitigations, candid unknowns |

---

## New Packages / Missing Modules — Master Table

This is the authoritative, consolidated catalogue of every new package this plan funds. It synthesises the Chapter 11 gap analysis with packages implied across Chapters 5–10 and 13. The discipline is deliberate: each item earns **its own versioned package** with its own test suite and its own semantic-version line, because each one (a) introduces a distinct trust boundary, (b) has a different release cadence than the engine spine, or (c) must be independently auditable to keep the substrate's invariants intact. Packages are listed in dependency order; "Layer" uses the L0–L6 stack plus the cross-cutting lobes (Lens, PH = personhood). "Owner deliverable" priority is **P0** (foundation, blocks everything) → **P3** (scale).

| # | Package | Role (what it is) | Layer | Priority | Why it needs its own package |
|---|---|---|---|---|---|
| 1 | `knitweb.pulse_epoch` *(or in-tree `core.pulse` + `token.mint` seam)* | Bind issuance to the Pulse heartbeat: `epoch_mint_cap` on the Beat, gate `Treasury.reward_verified_work` to per-epoch escrowed demand, settle mints atomically at epoch boundary. Closes the activity→money loop (Gap **G1**). | L0+L6 | **P0** | It is the single load-bearing monetary seam; isolating it keeps the consensus-critical mint logic small, audited, and versioned independently of game churn. Ships solo (no p2p surface). |
| 2 | `knitweb.p2p.webtransport` (browser-P2P transport) | A browser-reachable carrier under the existing `BaseNode._dispatch` seam: WebRTC/WebTransport datachannels, WSS bootstrap, the gossip mesh reaching into the tab. The thing that makes "no server" literally true in a browser. | L2 | **P0** | Distinct runtime (browser vs. CPython), distinct security surface (untrusted JS peers), distinct release cadence; must not be coupled to the stdlib-only engine core. |
| 3 | `knitweb.fabric.codechannel` (self-distributing source/asset channel) | Content-addresses application code + assets into CIDv1 dag-cbor records and serves them verbatim over inv→getdata→inv-data; verify-by-hash bootstrap so the dapp ships *itself* over the Web. | L3 | **P0** | "Source-code-included" is a defining product property and a supply-chain trust boundary (reproducible, signed, hash-pinned) that warrants its own audit and version line. |
| 4 | `knitwebs.chemistry` (the chemistry knitweb) | Replace the `__init__.py`-only stub with the real L5 plugin molgang binds to: chemistry node/relation schemas, the bond→Knit / molecule→Fiber mapping, validation rules, the ~470-node seed graph importer. | L5 | **P0** | It is the domain model; it changes at product speed (concepts, i18n, curricula) far faster than the engine and must be packaged and released on its own cadence. |
| 5 | `molgang` (the game client/server package) | The playable knitweb: faucet, propose→vote→woven-Fiber loop, collection/XP/levels/leaderboard, bar + explorer surfaces, the bridge. Already partly real in `src/molgang/`; harden to product grade. | L5 | **P0** | The flagship application; consumer-facing release cadence, its own UX/i18n surface, distinct from protocol packages. |
| 6 | `knitweb.lens.agent` (agentic player/NPC loop) | The agentic LENS loop: deterministic retrieve + gated distill → propose Knits, vote, teach, curate as a first-class signed peer; provenance-cited generation; distilled bundles registered as SPLIT-verified PoUW jobs. | Lens (L4 jobs) | **P1** | LLM/runtime dependencies and prompt/policy logic must be quarantined from the deterministic core; its non-determinism is gated *into* PoUW, so it needs its own boundary and tests. |
| 7 | `knitweb.lens.inference` (verifiable inference adapter) | Pluggable inference backend + the determinism/replay harness that lets an LLM output become a verifiable PoUW artifact (re-checkable distill, signed manifest). Extends `interpret/backends` + `inference/control.py`. | Lens / L4 | **P1** | Third-party model adapters and the verifiability contract are a swappable trust boundary; must be isolated so a backend swap can't touch consensus code. |
| 8 | `knitweb_monitor` → MONITOR-as-product | Promote the zero-dependency dashboard from liveness viewer to a *trust instrument*: locally verifiable state roots, reproducible evidence, graph + session views — trust without trusting MONITOR. | cross-cut (obs) | **P1** | Separate repo already; needs its own release line as a user-facing trust surface, decoupled from engine internals it only *reads*. |
| 9 | `knitweb.lens.weight` (integer-only relation-weight type) | An integer-only weight type / `to_canonical()` boundary that makes "no floats near canonical" a *type* boundary, not a convention (Gap **G3**); tighten `Web._validate_metadata_value`. | Lens + L3 | **P1** | Protects the single most sacred invariant at the exact seam (interpret→fabric) where floats legitimately originate; small, audited, versioned guard. |
| 10 | `knitweb.personhood.proofkit` (real VC/ZK backend) | A concrete `PresentationVerifier` implementation behind the existing adapter seam: revocable proof, pairwise DIDs, no PII on-fabric — the Sybil gate NPCs and humans both pass. | PH (cross-cut) | **P1** | Cryptographic/privacy trust boundary with its own audit, key-management, and compliance surface; must be swappable without touching the gate. |
| 11 | `knitweb.core.hardening` *(in-tree, tracked as a unit)* | Per-container length budgets in `canonical._decode` (`MAX_STRING_LEN`/`MAX_ARRAY_LEN`, Gap **G4**); `crypto.validate_pubkey()` EC-point check called in `Fiber`/`Knit` (Gap **G7**). | L0 | **P0** | Anti-amplification + curve-membership hardening of the replication surface; cheap, pure-core, high-value, must land before browser peers widen the attack surface. |
| 12 | `knitweb.ledger.equivocation` (structural fork witness) | Optional `fork_height` + `equivocation_witness` on `Fiber`, a `Braid.weave` quarantine path + local detection (Gap **G5**); structure now so p2p can gossip fork proofs later without a schema break. | L1 (→L2) | **P2** | A schema-level change consumed across layers; isolating it lets the L0 structure ship solo while the gossip half is handed to the p2p lane. |
| 13 | `knitweb.token.economy` (VBANK treasury/emission policy) | Bind VBANK governance to emission: what counts as useful work, treasury spend, NPC sanctioning, PLS entry policy — the constitutional economic layer on top of epoch-bound mint. | L6+L5 | **P2** | Governance-controlled monetary policy is its own auditable surface, released on a deliberate, slow cadence distinct from the engine. |
| 14 | `knitweb.pouw.epochloop` (live scheduler/marketplace wiring) | Promote `pouw.scheduler`/`marketplace` from tested-but-idle adapters into a live epoch loop driving job demand, settlement cadence, and the agent economy's order flow. | L4 | **P2** | The runtime orchestration layer has operational concerns (liveness, backpressure) separate from the pure verification primitives it schedules. |
| 15 | `molgang.bridge` (Roblox ⇄ Knitweb + onboarding) | Harden the existing two-way bridge (`bridge/`) and faucet onboarding into a packaged adapter: external-world ingress/egress, free-silk faucet, identity hand-off. | L5 | **P3** | An external-integration trust boundary (untrusted off-fabric input) with its own cadence and its own security review, kept out of the core game package. |

**Reading the table.** Items 1, 2, 3, 4, 5, and 11 are the **P0 critical path** — without epoch-bound issuance, a browser transport, the self-distributing code channel, the chemistry plugin, the hardened game package, and the core hardening, there is no serverless MVP. Items 6–10 turn the MVP into the *agentic economy* that makes molgang unique. Items 12–15 are the scale-and-govern layer for the top-10 push. Every package preserves the non-negotiable invariants — integers everywhere a hash/signature/balance is touched, canonical CBOR as the sole identity, and the sacred vocabulary (Web / Knit / Pulse / Fiber / knitweb — never "network", never "loom").

---

## Methodology Note

This plan was compiled through a multi-agent research workflow run over the three live repositories — `knitweb/pulse` (the engine, `src/knitweb`, ~20.7k LOC, 102 property suites, 1183/1184 tests passing at synthesis), `molgang` (the game/web/PHP and pure-Python client), and `knitweb-monitor` (the dashboard) — cross-referenced against a verified architecture state-of-the-union (the maturity heatmap, value-flow trace, and ranked gap analysis G1–G8) and against public web sources on the 2025–2026 dapp, agentic-payments, and decentralized-knowledge landscape, which are cited inline where market or competitive claims are made. Each chapter was drafted by a dedicated agent against the actual module names, file paths, and invariants in the trees rather than from memory, then consolidated and reconciled by the lead author for consistency of vocabulary, layering (L0–L6 plus the cross-cutting personhood/Lens lobes), and the built-vs-to-build distinction. All architectural assertions reflect code present in the repositories as of June 2026; all quantitative figures — LOC, node counts, test totals, effort estimates, timelines, and market sizing — are **indicative**, intended to support engineering and resourcing decisions rather than to serve as audited measurements, and should be re-validated against the repositories and live metrics before any external commitment is made.


---



## 2. Market & Landscape: the path to a top-10 dapp

This chapter situates molgang — and the Pulse substrate beneath it — inside the 2025–2026 decentralized-application market. The goal is not to flatter the project but to find the one defensible niche where a serverless, P2P, agentic knowledge-graph game with verifiable useful work can plausibly reach a top-10 position. We do that by (a) defining what "top-10" actually measures, (b) reading where the market's attention and money sit, (c) examining the three structural movements molgang rides — fully-on-chain/serverless, agentic-LLM payments, and decentralized knowledge/inference — and (d) placing molgang against named competitors in a comparison table. Throughout, we are honest that Pulse today is a hardened pure-Python substrate (≈20.7k LOC, 1,180+ passing property suites) and molgang is a ~470-node multilingual explorer still partly served as PHP/static on shared hosting. The market case is about where that substrate can credibly go, not where it is.

### 2.1 What "top-10" actually means

The de-facto scoreboard for dapps is [DappRadar](https://dappradar.com/rankings), which tracks over 18,000 dapps across 90+ chains and ranks them on three primary signals: **UAW (Unique Active Wallets)** — distinct wallets interacting with a dapp's contracts in a window; **Volume** — token value flowing through tracked contracts; and **Balance/TVL** — value held in those contracts. Industry trackers add **DAU** and **retention** (the share of wallets that return week-over-week), which separate genuine products from airdrop-farming spikes.

Two facts about the denominator matter for strategy. First, the market contracted in 2025: average daily UAW fell from **24.3 million in Q2 2025** to **18.7 million in Q3 2025**, a 22.4% quarter-over-quarter drop, per DappRadar's [State of the Dapp Industry Q3 2025](https://dappradar.com/blog/state-of-the-dapp-industry-q3-2025). A top-10 *category* position therefore requires a far smaller absolute audience than the headline numbers suggest — tens of thousands of genuinely-returning wallets in a focused category, not millions. Second, UAW is wallet-denominated and trivially Sybil-inflated; rankings reward whoever games it. This is precisely where Pulse's **personhood gate** (one-person-one-scope nullifier, epoch-pinned non-revocation, `personhood.gate.require_personhood`) is not a compliance afterthought but a *ranking-integrity advantage*: molgang can report Sybil-resistant **proof-of-personhood DAU**, a metric most competitors structurally cannot produce. Our north-star target is consequently not raw UAW but **verified-human returning players executing PoUW jobs** — a harder number to fake and a more honest one to publish.

### 2.2 Where the attention and money sit

Category dominance in 2025 settled into a clear order. Per DappRadar's Q3 2025 report, **Games led at ~25%** of industry activity, **NFTs second at 18.5%**, with **DeFi and AI** close behind and **Social collapsing from 15.9% to 8.4%** in a single quarter. Earlier in the year AI had surged — DappRadar's [Q2 2025 report](https://dappradar.com/blog/state-of-the-dapp-industry-q2-2025) put gaming at 20.1% and AI at 18.6% — before AI cooled to ~16.8% in Q3, as documented across the [Q2 market recaps](https://www.chaincatcher.com/en/article/2190504). DeFi remained the value sink: TVL hit a record **$237 billion** even as active wallets fell, and NFT trading volume nearly doubled to **$1.58 billion** quarterly.

The strategic read is threefold. **Gaming is the largest and most durable activity category** — it is where returning, non-financial users live, and it survived the 2025 contraction better than Social or AI. **Social is a cautionary tale**: a category can lose half its share in a quarter when it has no economic loop binding users in. **AI is volatile but structurally ascendant** — it spiked, corrected, and is consolidating around real infrastructure rather than meme tokens. molgang sits deliberately at the *intersection of the strongest durable category (Games) and the most ascendant infrastructure trend (agentic AI)*, with a built-in economic loop (PoUW → PLS) that the failed Social category lacked. That intersection — gaming-plus-AI with a verifiable-work economy — is thinly populated, which is exactly where a top-10-in-niche claim is winnable.

### 2.3 Movement one — the serverless / fully-on-chain dapp

The "autonomous worlds" thesis — games whose entire state and logic live on a shared, permissionless substrate that *replaces the central server* — matured from Dark Forest's 2020 zk experiment into named engines: Lattice's [MUD](https://lattice.xyz/blog/mud-an-engine-for-autonomous-worlds), the Dojo suite on Starknet, and Argus's World Engine. The core promise, as MUD frames it, is worlds that are "infinitely extendable by default," where the chain "serves as a substitute for a centralized game server."

molgang's differentiation here is sharp and important: the autonomous-worlds field is almost entirely **EVM/Starknet-on-a-public-blockchain**, which means high per-action gas, sequencer dependence, and state that is on-chain but still *centrally ordered*. molgang's target is genuinely **serverless P2P** — content-addressed CIDv1 dag-cbor records replicated over a gossip mesh (Erlay reconcile, gossipsub, anti-entropy backstop), with **no blockchain sequencer in the hot path at all**. A parallel movement validates the demand for exactly this substrate: [Plebbit](https://github.com/ipfs/devgrants/issues/289), a fully-P2P Reddit using IPFS/IPNS over gossipsub, and the broader 2025 wave of self-hostable IPFS (Kubo's DHT Provide Sweep, Helia trustless browser retrieval). The lesson from these projects is that P2P social/content apps are buildable but struggle to *monetize and to resist Sybil attacks*. molgang's two answers — a PoUW-minted token (PLS) and a personhood gate — are precisely the two things the serverless-P2P cohort is missing. A further, near-unique commitment is that **molgang ships its own source code as content-addressed records on the same Web it runs on**: the dapp is self-hosting and self-distributing, with no app-store or CDN dependency. Among autonomous-world engines, none today distribute their own client over the same substrate that holds game state.

### 2.4 Movement two — agentic LLMs and machine-native payments

The defining infrastructure story of 2025–2026 is machines paying machines. Coinbase's **x402** revived the dormant HTTP 402 status into a stablecoin payment handshake "with no human in the loop"; since its May 2025 launch it has processed [over 100 million payments](https://aurpay.net/aurspace/x402-protocol-ai-agents-pay-apis-crypto-2026/), and the [x402 Foundation](https://www.chainup.com/blog/x402-erc8004-ai-agent-payments-agentic-web/) launched under the Linux Foundation in April 2026 with 20+ founding members including Google, Visa, Stripe, Mastercard and Circle. The companion **ERC-8004** standard defines agent identity and reputation on-chain. Crucially, the early demand clusters around *per-request AI inference and per-task agent work* — high-frequency, low-value, fully-automated settlement. The cautionary counterpoint is volatility: per [BlockEden analysis](https://stablecoininsider.org/ai-agents-for-stablecoins-in-2026/), daily x402 transactions fell ~92% from a December 2025 peak of 731k to 57k by February 2026 — proving the rails work but that *durable demand, not hype, is the scarce resource*.

The tokenized-agent market tells the same boom-correction story. The AI-agents sector reached a [~$15.3 billion market cap](https://www.altrady.com/blog/cryptocurrency/ai-agent-crypto-tokens-guide), with Virtuals Protocol and ai16z holding 56.8% of it; yet Virtuals' own token fell from a $5B+ valuation to ~$485M (rank 104) by mid-2026, and ai16z collapsed ~99% from its January 2025 all-time high. Virtuals launched ~14,000 agent tokens; most failed to retain liquidity. The market has spoken: **launchpads for speculative agent tokens are oversupplied; venues with real, verifiable agent *work* are not.**

This is molgang's sharpest positioning. Its NPCs are **agentic LLMs as first-class players** — they curate, contribute, and dispute chemistry-knowledge records — and every such action is a **PoUW job that mints PLS only on quorum-verified usefulness** (`⌊2n/3⌋+1` committee verification, sampled re-execution, dispute window). Where x402 settles a *claim* that work happened, Pulse settles *proof* that work was verified by a Byzantine-resistant committee before any token is minted. The interpret/**Lens** layer makes this concrete: its deterministic-retrieve + gated-distill pipeline emits signed, provenance-cited relation bundles that *themselves re-enter the PoUW flow as SPLIT-verified jobs*. In x402/ERC-8004 terms, molgang is a native venue where agent payments are gated on verifiable knowledge work rather than self-asserted API calls.

### 2.5 Movement three — decentralized knowledge & verifiable inference

Two adjacent ecosystems prove demand for molgang's core primitive. [OriginTrail's Decentralized Knowledge Graph](https://origintrail.io/technology/decentralized-knowledge-graph) runs "knowledge mining," rewarding contributors with $NEURO for enriching a verifiable, AI-grade knowledge base — *Wikipedia-with-incentives* — and Pulse already integrates OriginTrail resolve+compile via its `synaptic`/`anchor` modules. On the compute side, [Bittensor](https://www.cryptotimes.io/learn/bittensor-tao-guide/) runs 128 active subnets paying TAO for machine-intelligence services, with [verifiable-inference subnets](https://github.com/inference-labs-inc/subnet-2) using zk-ML fingerprints, while Gensyn pursues verifiable training proofs. These are billion-dollar validations that "pay for verified knowledge/inference" is a real market — but they are heavyweight L1s requiring node operators and staking. molgang occupies the **lightweight, human-playable, single-domain** corner of the same thesis: chemistry knowledge, knit by humans and agents, verified by PoUW, on a stdlib-only P2P substrate a laptop can run.

### 2.6 Competitive positioning

| Project / category | Substrate | Economic loop | Agentic LLMs | Sybil resistance | molgang's edge |
|---|---|---|---|---|---|
| **Dark Forest / MUD / Dojo** (autonomous worlds) | Public EVM / Starknet, central sequencer | Gas + in-game assets | Rare, bolt-on | Wallet-only | True serverless P2P; no sequencer; self-distributing client |
| **Pixels, Axie** (web3 games) | Ronin / app-chain | Token + NFT economy | None | Wallet-only | Knowledge-as-play + PoUW-verified rewards, not token farming |
| **OriginTrail DKG** | Multi-chain L1 + node network | $TRAC/$NEURO knowledge mining | Consumer of graph | Stake-based | Playable, human+agent authored; integrated, not competing |
| **Bittensor / Gensyn** (decentralized AI) | Dedicated L1 / subnets | TAO for inference/training | Producers | Stake-based | Lightweight, single-domain, human-in-the-loop, no staking floor |
| **Virtuals / ai16z** (agent launchpads) | EVM/Solana | Speculative agent tokens | Token avatars | Wallet-only | Verified *work*, not speculative supply; demand-gated mint |
| **x402 / ERC-8004** (agent payments) | HTTP + stablecoins | Pay-per-call (asserted) | Payers | Off-chain identity | Payment gated on PoUW-*verified* usefulness |
| **molgang on Pulse** | **stdlib-only P2P Web (CIDv1)** | **PLS minted only on PoUW** | **First-class players/NPCs** | **Proof-of-personhood gate** | — |

### 2.7 The defensible niche

No single competitor occupies all five columns at once. Autonomous worlds have serverless ambition but ride central sequencers and lack Sybil-resistant identity; web3 games have economies but no verifiable-work loop; OriginTrail and Bittensor have verifiable-knowledge economies but are heavyweight and unplayable; agent launchpads have agents but no verified work and a discredited token model. **molgang's niche is the empty intersection: a genuinely serverless P2P game where verified humans and agentic LLMs jointly knit a content-addressed knowledge graph, and PLS is minted only as demand-gated reward for committee-verified useful work.** Reaching top-10 does not require beating Uniswap's TVL or PancakeSwap's UAW; it requires owning the small, nameable category of *agentic, serverless, verifiable-knowledge games* — and being the reference implementation that ships its own source over the Web it runs on. The honest caveat, carried into the roadmap chapters, is the load-bearing gap from the architecture review: Pulse's heartbeat is not yet bound to issuance (epoch-bound minting, gap G1), so the activity→money loop the whole pitch depends on is structurally present but not yet closed. Closing it is the precondition for any credible market claim.



## 3. The Knitweb Thesis: a Core P2P Substrate for Agentic-LLM Dapps

The central claim of this plan is structural, not aspirational: **an autonomous agent that earns, reasons, and acts on behalf of others is, mechanically, a tiny economy — and an economy needs a substrate that gives it verifiable compute, durable provenance, unique identity, and exact micro-settlement in one coherent stack.** The mainstream "agentic web" of 2026 assembles those four properties from four unrelated systems glued together by trust. Knitweb/pulse provides them as four facets of a single content-addressed Web. This chapter argues why that consolidation is the right foundation, maps each agentic requirement onto a concrete knitweb layer, and is honest about which seams are production-grade and which are still adapters.

### 3.1 What an agentic dapp actually requires

Strip the marketing from "AI agent dapp" and four hard requirements remain. An agent that does paid, autonomous work must be able to:

1. **Prove its work was done correctly** — *verifiable compute*. If an agent claims it summarised a document, ranked a proposal, or distilled a relationship from data, a counterparty must be able to check that claim cheaply and without re-running the whole task themselves.
2. **Cite where every fact came from** — *provenance*. An agent answer that cannot be traced to source records is unaccountable; it cannot be audited, disputed, or trusted in a financial or governance context.
3. **Be a unique, non-forgeable participant** — *identity / Sybil-resistance*. The cheapest attack on any agent economy is to spin up ten thousand agents and capture rewards, votes, or reputation. One person fielding an army of bots must not be able to count as ten thousand humans.
4. **Pay and be paid in exact, tiny amounts** — *micro-settlement*. Agent-to-agent commerce is high-frequency and low-value: a fraction of a cent per API call, per inference, per curated edge. The settlement primitive must be integer-exact (no rounding drift across millions of transactions) and cheap enough to run per-request.

These are not optional features layered on top of a chain — they are the *definition* of an honest agent economy. The thesis of this plan is that knitweb already expresses all four as native primitives, because it was designed bottom-up around the same invariant: **canonical bytes are identity, and every value that touches a hash, a signature, or a balance is an integer.**

### 3.2 The usual stack, and why it fragments

The dominant 2026 pattern for "crypto AI agents" composes roughly this stack: an **EVM chain** (Base, Solana) for settlement and an agent registry such as **ERC-8004**; **IPFS** for blob storage; a **centralized API** (the actual model endpoint plus an orchestration backend); and **OpenAI/Anthropic** behind that endpoint for the reasoning itself. Payments are increasingly routed through **x402**, the HTTP-402 "pay-per-request" protocol that, across supported chains, processed on the order of [$600M annualised with 119M+ cumulative transactions on Base alone by March 2026](https://www.chainup.com/blog/x402-erc8004-ai-agent-payments-agentic-web/). Identity is bolted on by linking agents to a verified human via systems like [World's AgentKit, which uses World ID zero-knowledge proofs to bind multiple agents to one unique person](https://www.coindesk.com/tech/2026/03/17/sam-altman-s-world-teams-up-with-coinbase-to-prove-there-is-a-real-person-behind-every-ai-transaction).

This stack works, and its momentum is real. But it fragments the four requirements across four trust domains, and the seams between them are exactly where trust silently re-enters:

| Requirement | Usual stack answer | Where trust leaks back in |
|---|---|---|
| Verifiable compute | None native; "the API said so" | The model endpoint is a trusted black box — no one re-checks the work |
| Provenance | IPFS CID of an output blob | The CID proves *a blob existed*, not *which sources produced it* |
| Identity | ERC-8004 registry + external PoP | Registry entry is self-asserted; Sybil-resistance is a separate oracle |
| Micro-settlement | x402 over EVM/stablecoin | Gas + float-denominated token amounts; rounding and fee overhead at true micro scale |

The deeper problem is **semantic discontinuity**. The chain knows about balances but nothing about the *meaning* of the work. IPFS knows about bytes but nothing about *who attested to them*. The model API knows the reasoning but emits it into a system that cannot verify it. Each layer speaks a different addressing scheme — EVM addresses, IPFS CIDs, API keys, model tokens — and the dapp developer is left hand-stitching them, re-introducing a centralized orchestrator as the only component that sees all four. That orchestrator is the single point of failure the whole "decentralized agent" story was supposed to eliminate. As [proof-of-personhood analyses for 2026 note](https://academy.exmon.pro/proof-of-personhood-2026-crypto-vs-deepfakes-ai-agents), the rise of agent armies makes the Sybil surface worse, not better, precisely because identity is the one layer most often left external and self-asserted.

### 3.3 The knitweb answer: one Web, four facets

Knitweb collapses the four trust domains into one content-addressed graph — *the Web* — where a single addressing scheme (CIDv1, dag-cbor/sha2-256 over canonical RFC-8949 bytes) is shared by records, balances, identities, and reasoning artefacts alike. Every requirement maps to a specific, already-built layer:

**Verifiable compute → L4 PoUW.** This is the facet the usual stack simply lacks. In knitweb, work is not asserted, it is *re-executed by a committee*. A job (e.g. `SynapticCompileJob`) is escrowed; a worker executes; verifiers are selected by `committee.select_committee` (worker excluded), a sample size `k` is computed, and verdicts are aggregated to a Byzantine `⌊2n/3⌋+1` quorum (`pouw/quorum.py`). The uniform path re-computes byte-for-byte and checks digest plus signature; the split path additionally requires a closed dispute window with no upheld dispute (`pouw/dispute.py`, with the enforced `release_delay > dispute_window` safety invariant). This subsystem is rated production-hardened (maturity 5) in the architecture review. **An agentic dapp on knitweb gets cryptographically-checked compute as a primitive, not a promise from a black-box endpoint.**

**Provenance → L3 fabric.** Every record is content-addressed into the in-memory `Web` graph (`fabric/web.py`), woven idempotently with typed `Edge`s, deterministic `traverse`, and — critically — `provenance` with enforced acyclicity, `attest`, `feed` proofs and `multiproof`. Provenance here is not "a CID of an output"; it is a checkable ancestry: the Lens distiller *gates every emitted relation* on attestation plus acyclic provenance (`interpret/distill._gate_relation`). An answer cannot be minted into the Web unless its sources resolve. **Provenance is a precondition of authoring, not a metadata afterthought.**

**Identity → personhood (cross-cutting Lens/PH).** The personhood gate enforces *one-person-one-scope* via a nullifier, with epoch-pinned non-revocation and a ticket decoupled from the content signature — an explicit zero-knowledge seam (`personhood/gate.py`, maturity 4). The verifier is an injected `PresentationVerifier` interface: a real VC/ZK backend (World ID-style, ICH-style) plugs in without touching the gate logic. Pairwise DIDs and revocation keep PII off-fabric. This is the same architectural conclusion the broader market reached — bind agents to a unique human via ZK — but expressed as an *orthogonal gate every knitweb (vBank, crowdfunding) calls before accepting a vote or pledge*, rather than a bolt-on registry.

**Micro-settlement → L1 ledger.** Settlement is an integer-only, two-party `Knit`: dual-signed over `[from, to, symbol, amount, from_nonce, timestamp, network]`, validated for positive-int amount, distinct parties, nonce match, no overdraft, and *exact* value conservation (`ledger/knitweb.validate_knit`). Knits append into `Fiber`s woven onto each party's `Braid` with seq+1, prev-CID linkage, and a spent-knit guard. Because every amount is an integer, there is **no rounding drift across millions of micro-transactions** — the property that float-denominated, gas-priced EVM settlement cannot structurally guarantee. The full ledger is rated maturity 5. This is the knitweb analogue of x402's "pay-per-request," but exact, local-first, and free of gas.

**Money itself → L4→L6, gated by the Pulse.** PLS is minted *only* as a demand-gated reward for verified-useful-work: `Treasury.reward_verified_work` issues a coinbase `Fiber`, anti-replayed by proof digest, with no premine and no admin mint (`token/mint.py`). Money supply is therefore a direct function of *checked* agent work — the inverse of a chain where tokens exist first and utility is sought afterwards.

### 3.4 The Lens reasoning lobe — the facet no other stack has

The most distinctive piece of the thesis is **Lens** (`interpret/`): the reasoning lobe that turns the converged Web into new knowledge. `interpret.retrieve` does a deterministic, subscription-gated graph walk (`Web.traverse`/`neighbors`, optional spatial union, provenance ancestry). `interpret.distill` runs a bounded, attestation-gated loop and emits a signed `Selection → DistillManifest` — **which itself re-enters the PoUW flow as a SPLIT-verified job.** This is the closing of the loop that defines the whole substrate:

> Web records → distilled into signed relation bundles → become PoUW jobs → settle escrow → mint PLS → transacted as Knits woven back into the Web.

In the usual stack, an LLM's output is an opaque API response that dead-ends in a database. In knitweb, an agent's reasoning is a *first-class, verifiable, paid, provenance-cited artefact* on the same graph as the money and the identity. An NPC in a knitweb dapp does not "call OpenAI and trust the result" — it authors a distillation that a committee re-checks and that carries its sources with it. That is the difference between an agent that *talks* and an agent that *contributes verifiable, settled work*.

### 3.5 Honest boundaries

Credibility requires naming what is not yet closed. The architecture review flags one **critical** open seam: the Pulse heartbeat exists and chains (`core/pulse.py`) but is **not yet bound to issuance** — `token/mint` caps emission by escrow, not by epoch (Gap G1). Until epoch-bound issuance lands (roadmap item R1), Pulse is a clock the money does not yet obey. There is also a known determinism cliff where float values in the Lens quantizer are quarantined off the hash path *by convention rather than a type boundary* (G3), and an open anti-amplification test on the Erlay reconcile path (G2). The L5 domain knitwebs for chemistry, finance and supply-chain are namespace stubs today; the personhood verifier and PoUW scheduler are real seams awaiting their backends and a live epoch loop.

None of these undermine the thesis — they sharpen it. The four agentic requirements are each met by a **maturity-5 or -4 ACTIVE** subsystem (ledger, PoUW verification, fabric web, personhood gate); what remains is wiring the *governor* (Pulse→mint) that turns activity into money, and hardening the seams Lens touches. The foundation an agentic-LLM dapp needs — verifiable compute, provenance, unique identity, integer micro-settlement, all on one canonical Web — is built. The next chapters take that foundation and stand a real, self-hosting agentic dapp on top of it.



## 4. Molgang Product Deep-Dive

Molgang is the first end-to-end *knitweb* — the proof that the Pulse substrate carries a real consumer product, not just a protocol test harness. It is a chemistry (*scheikunde*) bar game in which players knit chemistry concepts into a shared, content-addressed knowledge graph and earn PLS for contributions that other players verify. The slogan in the repository README is exact and load-bearing: **"the game *is* the protocol."** Every move a player makes is a real protocol operation, not a re-skinned database write. This chapter describes Molgang as it actually exists in `/Users/develuse/repo/molgang` today, then defines the target product: the loops, progression, the *knit-to-earn verified knowledge* mechanic, agentic NPCs, the social layer, and the unique hook designed to make it sticky enough — and strange enough — to break into the top tier of on-chain games.

### 4.1 The game today

Molgang's core insight is a one-to-one mapping between chemistry actions and Knitweb primitives, codified in `src/molgang/game.py`:

| Chemistry / game action | Knitweb primitive | Module |
|---|---|---|
| Forming a **bond** | a **Knit** — two-party ledger transfer | `knitweb.ledger.knit` |
| A molecule's growing chain | a **Fiber** — immutable account-state commitment | `knitweb.ledger.fiber` |
| Peers **voting** on a bond | **PLS pulses** staked + a `pouw.quorum` BFT verdict | `knitweb.pouw.quorum` |
| A bond is accepted | a **confirm quorum** (k-of-n) woven into the player's Braid | `knitweb.pouw.quorum.tally` |
| Free starter material | **silk** + **pulses** from the faucet (`Player.join`) | `src/molgang/game.py` |

Because a vote is literally a Knit settled on a real account, the act of playing weaves a player's first Fibers for real. There is no game-only shortcut around consensus: a bond is only "true" once a Byzantine-fault-tolerant quorum of peers who know their chemistry confirms it. This is the property that makes Molgang categorically different from a quiz app with a token bolted on.

**The current client surface.** A player runs `molgang serve` to open the browser bar at `http://localhost:8765`: a lobby of three tables (`Periodic Bar`, `Organic Lounge`, `Noble Corner`), six seats each (`SEATS_PER_TABLE = 6`), each with a cosmetic avatar. The faucet (`FAUCET_PULSES = 50`, `FAUCET_SILK = 10`) seeds a fresh account. A seated player brainstorms a term and *knits* it for 1 silk; the table votes with pulses (`VOTE_COST = 1`); the real `pouw.quorum.tally` decides; on a confirm quorum the term is woven into a Fiber and appended to the shared `World` graph, with its `fiber_cid` and confirmation count recorded. The header always shows live PLS, silk and knit counts; **📒 My knits** lists each knit with its votes and woven Fiber CID; **🔭 Explorer** shows competing knits for a topic side by side. The same `/api/*` endpoints (`join / sit / propose / vote`, `GET /api/state`) drive both the DOM UI and machine players — a single authoritative `Bar` state with two thin clients.

**The shared knowledge graph.** Confirmed knits accrete into a persistent `knitweb.fabric.web.Web` (`src/molgang/world.py`). A *term* knit ensures a term node; a *link* knit (`A = B`, `A → B`) weaves two nodes joined by a weighted edge (weight = confirmation count, an integer — never a float, respecting the canonical-encoding invariant). Terms are canonicalised (`clean()` folds `CH₄ → CH4`) and de-duplicated case-insensitively, so every spelling of a concept hashes to one CID. The graph is file-shared across processes, so two `molgang serve` instances extend and see the same growing web, and the whole state is anchored to **OriginTrail** as a verifiable UAL with a notary receipt (`World.anchor`). The live explorer (`molgang explore`, port 8990) is a NetworkX lens built for the **523-concept, four-language (EN / RU / ZH / AR)** chemistry graph — roughly **~2,600 nodes** across all label languages — with hub/centrality views, a path finder, and per-concept multilingual cards (water / вода / 水 / ماء). The earlier ~470-node multilingual explorer on shared PHP hosting (5mart.ml) is the lineage this graph grew from.

**The spiral mechanic.** Beyond single bonds, `game.py` implements **spirals**: a leader lays an *auxiliary* (draft) spiral of 2–7 links at escalating silk cost (`spiral_silk_cost`), backers stake one pulse per link, and once `MIN_SPIRAL_VOTERS = 3` confirm via quorum it becomes a sticky *capture* spiral — one settle weaves every link into the web at once, so the graph grows in connected sub-structures rather than isolated strings. This is the chemistry-native analogue of building a reaction pathway.

**Economy and progression.** On a confirmed knit, `game.settle()` pays the staked vote-pot back to the proposer, restores their silk, and mints a capped, *exponential usefulness bonus* (`usefulness_bonus`, base 2, `MAX_USEFULNESS_BONUS = 64`) scaled to confirming votes; confirming voters get their stake back plus a reward, so validation is net-positive. Failed quorum refunds every voter — integer-exact, conservation-preserving. `src/molgang/progression.py` layers XP (100 per woven bond), eight levels (Apprentice → Laureate) and a leaderboard over the Fibers, plus a reputation-scaled quorum threshold so a high-level table demands a stricter supermajority (only ever raising k while `2k > n` holds — `pouw.quorum` itself is untouched).

**NPCs today.** `Bar._seed_bots()` seats three NPC table-mates per table so a solo human can still reach quorum. Today these are deliberately simple: a seeded NPC votes "confirm" on open knits via an honest chemistry verdict (`honest_verdict` / `honest_spiral_verdict`, grounded in the `chemistry.py` knowledge base). They are scaffolding for fairness, not yet players in their own right. There is also a **two-way Roblox bridge** (`bridge/sync.py`) alternating every 30 minutes, mapping each Roblox wallet ID to a stable Knitweb account and replaying votes as real Knits.

### 4.2 The target product: a top-tier, fully on-chain knit-to-earn game

The target is a **top-tier, genuinely fun and sticky dapp that lives entirely on P2P** — no central server, shipping its own source code, with agentic LLMs as first-class players. The 2026 web3-gaming landscape has hardened around "play-and-own" and "learn-and-earn" over pure speculation, with serious operators now treating retention as the gating metric ([Antier](https://www.antiersolutions.com/blogs/from-play-to-earn-to-play-and-own-the-new-blueprint-for-web3-game-development-in-2026/), [cryptostart](https://www.cryptostart.app/play-to-earn-play-to-learn-the-rise-of-web3-trading-games-and-blockchain-simulators-in-2026/)). Molgang's differentiator is that its "ownership" is not a tradeable NFT — it is **non-transferable woven knowledge anchored to a UAL** — and its earning is **demand-gated reward for verified-useful-work**, exactly the PoUW thesis of the Pulse substrate. That makes the economy structurally resistant to the inflation-to-worthlessness failure mode that has killed most learn-to-earn tokens.

**The core loop (90 seconds).** A server-owned phase clock per table drives every browser off the same countdown: `WAITING → BRAINSTORM → VOTE → WOVEN/REJECTED → RESULTS/XP → WAITING`. A seated player spends silk to knit a chemistry claim (a typed ConceptNet-style edge `(subject, relation, object)` with relation from a fixed set: `IsA, PartOf, UsedFor, ReactsWith, RelatedTo`); the table stakes pulses; `pouw.quorum.tally` settles; on a confirm the edge is woven, the proposer earns the pot plus the capped usefulness bonus and XP, and the graph visibly grows. The loop is satisfying in the Jackbox sense — fast, social, shared-screen — but every round leaves a permanent, verifiable artifact behind.

**The unique hook — *knit-to-earn verified knowledge*.** The thing no competitor has is that the prize *is the graph*, and the graph is **provably true by Byzantine consensus, not by a backend's say-so**. A player's portfolio is a publicly queryable subgraph of concepts they originated and that survived peer verification, each backed by a Fiber CID and an OriginTrail UAL. You cannot buy your way to a high reputation (vote weight is earned XP, non-transferable, soulbound-style), you cannot fake a confirmation (it takes an honest BFT quorum), and you cannot farm trivial links forever (**taboo terms** retire over-woven concepts; **slashing and decay** burn stakes on failed or stale proposals so sinks outpace the faucet). The emotional hook is *legacy*: "I am the person who first knit this true thing into the web, and the web will carry it forever." That is a fundamentally stickier reward than a depreciating asset.

**Agentic NPCs as first-class players.** The current confirm-only bots become **agentic LLM players** — the largest single uplift. Built on the LENS reasoning lobe (`interpret/retrieve` + gated `distill`), an NPC retrieves the converged Web subgraph around a proposed edge, reasons about plausibility (MeTTa/Hyperon-style, provenance-cited), and emits a *justified* verdict that itself becomes a PoUW job. Because Molgang exposes a single Crab-Games-style action-manifest endpoint (`GET /api/heartbeat?sid=` returning everything currently legal), an agent is a stateless caller of the same `Bar` methods humans use — the headless-core / thin-client pattern. This rides directly on 2026's maturation of LLM game agents capable of social reasoning, planning and self-play ([git-disl survey](https://github.com/git-disl/awesome-LLM-game-agent-papers), [EITT](https://eitt.academy/knowledge-base/ai-agents-2026-guide-from-llm-to-multi-agent-systems/)). The NPCs do three jobs: they keep tables alive (always enough seats for quorum), they raise the bar (a confident, citing AI opponent makes humans propose harder, non-obvious links), and they *generate the training corpus* from the event-sourced log of every knit and verdict — a flywheel where playing the game improves the agents that make the game better.

**Player journey and progression.** A newcomer arrives at the faucet (gated for Sybil resistance: a newcomer may knit and vote, but their pulse does not count toward quorum until they have earned N confirmations) and walks into a table. Early sessions teach the school-level molecule set (`H2O`, `CO2`, `NaCl`, `CH4`…) — fast, legible wins. The Apprentice → Laureate ladder unlocks: spirals (multi-link pathways), expert tables, the reputation-weighted quorum where their verdict starts to matter, and eventually a **PoUW Certificate** (a signed PDF documenting their pulses spent, knits woven, votes cast and the shared UAL). The endgame is permissionless: agents and curators register expert tables against the same API — the existing Roblox bridge already proves the cross-platform appetite.

**Retention design.** Molgang targets the sustainable-economics band the 2026 market now demands — roughly **35–45% D1, 15–25% D7, 5–10% D30** ([cryptostart](https://www.cryptostart.app/play-to-earn-play-to-learn-the-rise-of-web3-trading-games-and-blockchain-simulators-in-2026/)) — through four reinforcing layers: (1) the **fast social loop** with a shared countdown; (2) **durable progression** (XP, levels, soulbound reputation, growing portfolio); (3) **economic discipline** (capped PoUW minting, taboo/slash/decay sinks, an EVE-style minted-vs-burned economic report shipped from day one); and (4) **agentic liveness** so a player is never alone and never bored. The retention is honest because the reward is honest: you keep coming back not to extract a token, but because the web you are weaving is real, verifiable, multilingual, and yours by consensus — a learn-and-earn product whose moat is that its "earn" can never be separated from genuine, peer-proven learning ([BeyondGames / Proof of Learn](https://www.beyondgames.biz/21543/web3-teaching-platform-proof-of-learn-introduces-learn-to-earn/)).



## 5. The serverless / P2P architecture: a dapp with no server that ships its own code

This is the crux of the whole programme. Everything in the preceding chapters — the content-addressed Web, the PoUW economy, the Lens reasoning lobe — exists to make one claim defensible: that MOLGANG can run as a *true* dapp, with **no central server**, where the application even **distributes its own source code over the Web it runs on**. This chapter describes, concretely, how the chemistry bar game lives without a backend: how the Pulse gossip mesh reaches into the browser, how assets and code become content-addressed records, how peers find each other and survive NAT, and how "source-code-included" stops being a slogan and becomes a verifiable property of a CIDv1.

### 5.1 Where MOLGANG is today, and what "no server" must replace

Today MOLGANG is honestly a *hosted* game. The repository at `/Users/develuse/repo/molgang` ships a `molgang_web/` static explorer (the ~470-node multilingual chemistry graph), a `php/` tier on shared hosting (`5mart.ml`), plus `bridge/` and `roblox/` connectors. That PHP tier already does one thing that matters enormously: it is a **store-and-forward relay**. The Pulse engine talks to it through `knitweb.p2p.relay.RelayTransport`, which `POST`s opaque length-prefixed CBOR frames to `api/relay/send` and drains them with `api/relay/fetch`. The relay "never decodes the payload" — it is a *dumb pipe*.

That distinction is the seam the entire serverless story hangs on. A dumb pipe is not a server in the dapp sense: it holds no authoritative state, validates nothing, and can be swapped or duplicated freely because every byte that matters is signed and content-addressed elsewhere. The migration to a true dapp is therefore not "delete the backend" — it is "demote every backend responsibility to either (a) the client mesh or (b) an interchangeable dumb relay, until nothing authoritative remains centralised." The table below tracks that demotion.

| Backend responsibility (today) | Serverless replacement | Pulse component | Status |
|---|---|---|---|
| Serve game state / graph | Content-addressed `Web` replicated peer-to-peer | `fabric.web.Web`, `fabric.node.FabricNode` | Built (Python); needs browser carrier |
| Authoritative writes | Dual-signed Knits + signed Web envelopes, validated by every peer | `ledger.knit`, `fabric.node._ingest_signed` | Built |
| Message transport | WebRTC/WebTransport datachannels + relay fallback | new `Transport` impls behind `p2p.transport.Transport` | Seam built, browser carriers to build |
| Peer discovery | PEX + Kademlia over signed `peer-exchange` records | `p2p.discovery`, `p2p.kademlia` | Built (Python) |
| Asset/code hosting | CIDv1 dag-cbor records fetched by inv→getdata | `core.canonical.cid`, `fabric` inv flow | Built |
| Mint / economy authority | Demand-gated PoUW + epoch Pulse, no admin mint | `pouw`, `token.mint`, `core.pulse` | Built; epoch binding in progress |

The reassuring part of this audit is how little is *missing*. The substrate is built and tested (≈20.7k LOC, ~1180 passing property suites). What is genuinely new work is one thing: **carrying the existing, transport-agnostic frames over browser-native transports.**

### 5.2 The transport seam: why the browser slots in cleanly

The single most important design decision already made in Pulse is that the **wire is split from the carrier**. `p2p/transport.py` defines a `Transport` protocol with exactly two verbs — `dial` (one-shot request→response) and `listen` (accept inbound, hand to a handler) — and it "never inspects the payload — frames stay byte-identical canonical CBOR." A `PeerAddress` carries a `transport` tag (`"tcp"`, `"relay"`, and soon `"webrtc"`) plus an opaque `params` map, and a `Dialer` routes each dial to the transport that owns that tag. A single node can therefore hold a heterogeneous peer set — some TCP, some relay-mailbox, some WebRTC — at once.

This means bringing MOLGANG into the browser is **not** a rewrite of the protocol; it is implementing two new `Transport` classes behind an interface that already has TCP and relay siblings:

- **`WebRtcTransport`** — opens an `RTCDataChannel` to a peer and uses it as the one-shot dial channel. The browser's `RTCDataChannel` in *reliable, ordered* mode gives exactly the in-order byte stream that `wire.read_frame`/`write_frame` already expect, so the length-prefixed CBOR framing is reused verbatim. No signed-record bytes change crossing this carrier, so a Knit's or a chemistry node's CID stays byte-identical browser-to-browser — the invariant the whole Web depends on.
- **`WebTransportTransport`** — the same shape over HTTP/3 WebTransport streams, used where a node *can* reach an addressable endpoint (a community-run super-peer, or a desktop node with a public address). WebTransport gives unidirectional and bidirectional QUIC streams that map onto dial/listen directly and avoid head-of-line blocking.

Crucially, the `transport.py` docstring already reserves a **"HOLE-PUNCH SEAM"** on `Transport.listen` for "a future STUN-assisted rendezvous then a direct UDP/TCP session." WebRTC *is* that hole-punch path, productised: ICE does STUN/TURN candidate gathering, and the rendezvous (SDP/ICE exchange) rides the existing relay mailbox. We are filling a seam the architects deliberately left open, not bolting on an alien subsystem.

### 5.3 Running the Pulse engine *inside* the browser

A browser peer must do more than relay bytes — it must run real Pulse logic: verify signatures, validate Knits, ingest signed Web records, and walk the graph. Pulse is **pure-Python, stdlib-only**, which gives two viable in-browser execution strategies:

1. **Pyodide / WASM (fidelity-first).** Because the engine imports nothing outside the standard library, the same `knitweb.fabric`, `knitweb.ledger`, and `knitweb.core` modules load under Pyodide unchanged. The browser peer runs *byte-for-byte the same* canonical encoder and the same `validate_knit` as a desktop node — zero protocol drift between client and "server," because there is no server. The cost is a ~6–10 MB WASM payload and slower secp256k1; both are acceptable for a turn-based chemistry game and the payload is itself content-addressed and cached (see §5.5).
2. **A focused JS verifier core (latency-first).** For the hot path — verify an envelope signature, check a CID, gate a relation — a small audited JavaScript/WASM module (noble-secp256k1 + a canonical-CBOR encoder that mirrors `core.canonical`) runs the per-frame checks at native-ish speed, while the heavier Pyodide image handles distill/retrieve and anything off the latency path.

The intended deployment is a **hybrid**: a JS verifier core for per-frame hot checks, Pyodide for the full engine and Lens reasoning, behind one `BrowserNode` facade that presents the same surface as `AsyncioP2PNode`. A non-negotiable conformance requirement falls out of "no floats near canonical": the JS encoder must reproduce `core.canonical.encode` exactly — RFC-8949 §4.2 minimal heads, sorted keys, integer-only, `MAX_DEPTH=64` — and we gate it with a cross-implementation differential test (the same record encoded by Python and JS must yield identical bytes and identical CIDs) before any browser peer is allowed to author.

### 5.4 The hard problems, addressed honestly

A serverless dapp lives or dies on four classic problems. None is hand-waved here.

**NAT traversal.** Most peers are behind NAT; inbound connections are dropped. WebRTC ICE handles the common case via STUN-discovered server-reflexive candidates and direct hole-punching. The residual ~8–12% of peer pairs that cannot hole-punch (symmetric NAT on both ends) fall back to **TURN relaying** *and* to the existing `RelayTransport` store-and-forward mailbox. The relay is the safety net that guarantees liveness even when ICE fails — and because it is a dumb pipe carrying opaque signed frames, using it costs no trust.

**Bootstrap (the cold-start problem).** A brand-new browser tab knows no peers. Pulse already solves the *logic*: `p2p.discovery` implements signed `peer-exchange` (PEX) — "a node tells a peer the addresses it knows… over a few rounds the whole component converges" — backed by a Kademlia DHT (`p2p.kademlia`) for scale. What a browser still needs is a *first contact*. We provide a small, replaceable set of **bootstrap rendezvous points**: the `5mart.ml` relay's signaling mailbox, plus community-run super-peers (any desktop node with `listen=` and a public WebTransport address). These are bootstrap hints, not authorities — a peer that learns *one* live address PEX-converges onto the rest of the mesh and never needs the bootstrap again. The bootstrap list itself is a signed Web record, so it can be updated through the Web rather than re-shipped.

**Persistence.** Browsers forget. A peer that closes its tab must not lose the woven graph, and the Web must survive even if *every* peer is briefly offline. Three layers handle this: (1) **local durability** — each browser peer persists received signed frames verbatim to IndexedDB keyed by CID, exactly mirroring how `FabricNode` "stores the verbatim signed frame under the CID," so a returning tab reloads instantly and re-seeds others; (2) **mesh redundancy** — the gossipsub mesh plus Erlay set-reconciliation (`reconcile_with`, moving O(diff) not O(total)) and the anti-entropy backstop (`sync_from`) re-converge any peer against the rest; (3) **always-on anchors** — a handful of volunteer pinning nodes (desktop Pulse processes, or the relay host running a headless `FabricNode`) keep a full copy so the cold mesh can rehydrate. Note none of these is *authoritative*: they hold the same content-addressed records every peer can verify, so a malicious pin can withhold but never forge.

**Browser limits.** Memory, connection count, and CPU are bounded. The defenses already in the engine carry straight over: per-peer **byte/probe/recon budgets** via `ServeBudget` on the single `BaseNode._dispatch` seam throttle any peer trying to flood a tab; the lazy inv→getdata→inv-data flow means a browser only downloads CIDs it actually lacks; `MAX_DEPTH`/`MAX_ITEMS` caps in the canonical decoder bound post-decode heap explosion; and a browser peer holds a *small* gossipsub mesh degree (e.g. 6–8 datachannels) rather than a full connection table, leaning on PEX/DHT for reachability beyond its immediate neighbours.

### 5.5 Source-code-included: the dapp distributes itself

The final, defining property: **the dapp ships its own verifiable code over the Web it runs on.** This is mechanical, not magical, once everything is a CIDv1.

The MOLGANG client bundle — the JS verifier core, the Pyodide image manifest, the WASM blobs, the game UI, the bootstrap hints — is chunked, each chunk is encoded as a canonical-CBOR record, and the bundle is described by a **release manifest** record: a content-addressed list of `{path, cid, len}` entries plus the author signature, exactly the structure `synaptic`/`edge` already use to ship signed relation bytecode that is "verified-before-trust." That manifest's CID *is* the application version. A user joins MOLGANG by resolving one manifest CID (from a bootstrap hint or a friend's link), then fetching each chunk **through the same inv→getdata flow the game uses for chemistry nodes** — the code travels the identical gossip path as the content. Because identity is the hash, the browser can verify every byte it received reduces to the CID it asked for, and reject a single flipped bit: the dapp is **self-verifying**, not merely self-hosting.

This closes a tidy loop. A new release is *woven* into the Web as a signed manifest; it propagates by gossip and Erlay reconcile like any record; peers fetch and **cryptographically verify** the code before running it; and — because publishing a useful, verified artifact is exactly the shape of a PoUW job — a release can itself be a job that mints PLS for its verifiers. The game, its source, and its economy all live on one content-addressed substrate with no privileged host. There is no server to seize, censor, or bill: there is only the Web, the Pulse, and peers who can prove to each other that the code they are running is the code that was signed.



## 6. Role of PULSE (the engine)

PULSE is the substrate. Everything molgang is — a chemistry knowledge graph that lives on a content-addressed Web, a PLS economy that pays players for verified-useful-work, a set of NPC reasoners that cite their provenance — is, at the protocol layer, a *consumer* of one pure-Python, stdlib-only engine living under `/Users/develuse/repo/pulse/src/knitweb`. molgang is a knitweb (an L5 domain plugin), not a fork; it imports PULSE and never re-implements transport, gossip, the ledger, or issuance. This chapter sets out exactly what the engine provides, the precise API seam molgang binds to today, and what PULSE must still add to carry a browser-grade, self-distributing dapp. The grounding is honest: most of the stack below is ACTIVE and property-tested (the verified architecture reference records 1183/1184 suites green), the issuance-to-heartbeat binding and the browser transport are the two load-bearing gaps.

### 6.1 What the engine is, layer by layer

PULSE is a strict seven-layer stack, and molgang draws a different guarantee from each.

| Layer | Module(s) under `src/knitweb` | What it gives molgang |
|---|---|---|
| L0 core | `core/canonical.py`, `core/crypto.py`, `core/pulse.py` | The sacred bytes: RFC-8949 §4.2 canonical CBOR, CIDv1 dag-cbor/sha2-256, secp256k1+SHA-256 signatures, the `Pulse`/`Beat` heartbeat. Every molecule CID and every signed vote is identical on every peer because it passes through `canonical.encode`. |
| L1 ledger | `ledger/{blob,fiber,braid,knit,knitweb,node}.py` | The integer-only two-party settlement primitives. A chemistry *bond* is a dual-signed `Knit`; a molecule's growing chain is a `Fiber`; a player's history is a `Braid`; `AccountNode` is the wallet. |
| L2 p2p | `p2p/{transport,wire,base_node,kademlia,discovery,reconcile,anti_entropy,mesh,inventory,reputation,relay,…}.py` | The Byzantine-resistant mesh: pluggable carrier, Kademlia discovery, gossipsub mesh, Erlay set-reconciliation, anti-entropy backstop, per-peer byte/probe budgets on one `BaseNode._dispatch` seam. |
| L3 fabric (the Web) | `fabric/{web,node,items,feed,provenance,spatial,attest}.py` | The content-addressed knowledge graph itself — `Web.weave`/`link`/`traverse` plus the live `FabricNode` gossip peer that replicates it. This *is* molgang's shared chemistry graph. |
| L4 pouw | `pouw/{job,verify,quorum,committee,sampling,dispute,collateral,escrow}.py` | Proof-of-useful-work: a player's curation/contribution becomes a verifiable job; peers re-execute; a BFT `⌊2n/3⌋+1` quorum settles; escrow + dispute window guard payout. |
| L5 knitwebs | `knitwebs/*` (chemistry is a stub here; molgang lives in its own repo) | The plugin seam molgang occupies. |
| L6 token | `token/mint.py` | `Treasury` mints PLS — no premine, no admin mint — only as a coinbase `Fiber` rewarding a *verified* job, anti-replayed by proof digest. |

Cross-cutting, two subsystems matter to molgang directly: **personhood** (`personhood/gate.py`, the Sybil gate every vote/pledge passes through — one-person-one-scope nullifier, pairwise DIDs, no PII on-fabric) and **interpret/LENS** (`interpret/{retrieve,distill}.py`, the reasoning lobe the NPCs ride). Both are detailed in their own chapters; PULSE's role here is to make their outputs first-class records and PoUW jobs, not bolt-ons.

### 6.2 Transport, discovery, gossip — how a bond reaches every peer

When a molgang player forms a bond, the engine moves it across the Web through a deliberately layered path, every hop of which preserves byte-identity so the CID never changes in flight.

**Transport / wire.** `p2p/transport.py` separates the *carrier* from the *frame*. A `Transport` knows only how to dial (one-shot request/response) and listen; it never inspects the payload, so the canonical CBOR frame (`p2p/wire.py`, `read_frame`/`write_frame`) crosses unchanged. Two carriers ship today: `TcpTransport` (`asyncio.open_connection`/`start_server`) and a `RelayTransport` for NAT'd peers that registers a store-and-forward mailbox and polls it — the realistic case, since most players are behind home routers. A `Dialer` routes each `PeerAddress` to the carrier owning its `transport` tag (`"tcp"`/`"relay"`), so one node holds a mix of directly-reachable and relayed peers at once. A hole-punch carrier slots in behind the same `Transport` protocol without touching anything above it.

**Discovery.** `p2p/kademlia.py` (a 634-line DHT overlay) plus `p2p/discovery.py` let a fresh molgang client find peers without a directory server — exactly what a serverless dapp needs. `p2p/peer_identity_gate.py` and `p2p/reputation.py` gate and score peers so a flood of fake players cannot drown a classroom.

**Gossip and convergence.** Propagation is lazy (the `inv → getdata → inv-data` flow): a node announces a CID only, the peer requests only the CIDs it lacks, and the announcer serves the *verbatim* stored frame from its `_frames: dict[str, bytes]` cache (`fabric/node.py:178`). Target selection is the bounded gossipsub mesh (`_eager_targets`). On reconnect, `FabricNode.reconcile_with` runs **Erlay** set-reconciliation (`p2p/reconcile.py`, `ReconcileSession`) — moving `O(diff)` not `O(total)`, so a returning player syncs the deltas of a 470-node graph, not the whole thing. The unconditional convergence backstop is **anti-entropy**: `FabricNode.sync_from(peer)` (`fabric/node.py:899`) pulls everything a peer has that this node lacks. The convergence witness is a single value — identical `web_state_root` (`fabric/items.py`) across peers — which is also what molgang anchors to OriginTrail.

Crucially, ingest is hardened. `FabricNode._ingest_signed` verifies the author signature over the signed bytes and routes node-vs-edge **off the signed record itself** (`_is_edge_record`), so a relayer cannot flip a "term" into a "link" and partition the chemistry graph's state root. A per-peer ingest budget throttles even validly-signed floods before they exhaust memory.

### 6.3 The ledger, epoch-bound PLS, and PoUW

molgang's economy is the engine's economy, unmodified. A vote stakes real pulses; a confirmed bond is a conservation-preserving `Knit` woven into the voter's and author's `Braid`; `ledger/knitweb.validate_knit` enforces positive-integer amounts, distinct parties, dual signatures, nonce match, no overdraft, and exact value conservation. Curation work flows through `pouw`: the contribution is escrowed (`pouw/escrow.py`), re-executed by a committee (`pouw/committee.py`, worker excluded), sample-sized (`pouw/sampling.py`), and tallied to a BFT `⌊2n/3⌋+1` outcome (`pouw/quorum.py`). Only then does `Treasury.reward_verified_work` (`token/mint.py`) mint the PLS reward as a coinbase Fiber, anti-replayed by proof digest. This is the spine of molgang's claim to be a *real* economy: PLS cannot be conjured, only earned for chemistry that peers verified.

Honesty demands the open seam be named. The `Pulse` heartbeat exists and chains (`core/pulse.py`, integer-only `verify_chain()`), but `token.mint` currently caps emission by escrow and an optional `max_supply` — **not** by epoch. There is no per-epoch supply ceiling and no `Beat → state-root` checkpoint driving issuance cadence. For molgang this is the difference between "PLS is minted whenever work clears" and "PLS issuance has a heartbeat-governed schedule a classroom can reason about." Closing it (carry an `epoch_mint_cap` on the `Beat`, gate `reward_verified_work` to per-epoch escrowed demand, settle at the epoch boundary) is buildable entirely within `core/pulse.py` + `token/mint.py` + `pouw`, with no p2p surface — the single highest-leverage engine increment, and a prerequisite molgang depends on for any predictable season/round economy.

### 6.4 The API seam: what molgang actually imports

molgang binds to PULSE through three deliberately narrow seams, and the import graph in `/Users/develuse/repo/molgang/src/molgang` confirms the discipline — it touches the engine's facades, never its internals.

- **`gateway.App` / `serve(app)`** (`gateway.py`) is the turnkey app layer, born from molgang as the proving ground. It bundles five things every builder re-solves: stable identity from an external id (`App.actor(external_id)` over `AccountNode.from_seed`), a *persistent, shared* Web (the raw `fabric.Web` is in-memory; `App` persists every woven record and can back the Web with a live `FabricNode` via `listen=`/`peers=`/`sync_from`), a turnkey economy (`actor`, `balance`, `transfer`), BFT validation (`App.validate(verdicts)` → `pouw.quorum.tally`), and OriginTrail anchoring (`App.anchor()` → UAL + notary receipt). `serve(app)` exposes the whole object over plain HTTP/JSON (`/actor`, `/transfer`, `/attest`, `/link`, `/validate`, `/web`, `/provenance`) so *any* runtime — molgang's own `webserver.py`, a Roblox `HttpService` bridge, a Colyseus server — drives the engine without a line of Python.
- **`sdk.Wallet`** (`sdk/__init__.py`) is the friendly facade over the value path: `Wallet.create`, `.address`, `.balance`, `.pay(to, pulses, ts)`, plus `compile_asset` / `distill_bundle` / `verify_bundle` for the synaptic + LENS read path.
- **`FabricNode`** (`fabric/node.py`) is the raw async peer for processes that want live gossip rather than the synchronous `App` wrapper: `start()`, `weave(record)`, `link(src, dst, rel, weight)`, `reconcile_with(peer)`, `sync_from(peer)`, `add_peer(name, peer)`, and `web_state_root` as the convergence witness.

molgang's modules map cleanly onto these: `game.py` and `bar.py` import `ledger.node.AccountNode` and `pouw.quorum`; `world.py` and `anchor.py` import `core.pulse.Pulse`, `fabric.items.checkpoint`/`web_state_root`, and the OriginTrail `Notary`; `webserver.py` re-exports a `gateway.App` store as the Monitor tab's local-knitweb graph. The seam is small enough to teach and stable enough to ship a public source-distributing client against.

### 6.5 What PULSE must add to serve a browser-grade dapp

PULSE today is a serverless engine for *Python* peers. To make molgang a top-10, entirely-P2P dapp that ships its own source and runs in a browser, the engine — not molgang — must grow four capabilities, in priority order.

1. **A browser-reachable carrier.** The `Transport`/`Dialer` abstraction is exactly the right seam; what is missing is a WebRTC/WebTransport carrier and a libp2p-style WebSocket-secure bridge so a browser tab is a first-class `PeerAddress` (`transport="webrtc"`), discoverable via the existing Kademlia overlay and relayed through the existing mailbox path for tabs that cannot accept inbound. This is the one true blocker for "no central server."
2. **Epoch-bound issuance (§6.3).** Without a heartbeat-governed mint, a browser classroom has no predictable season economy and no `Beat → state-root` checkpoint to light-verify against.
3. **A light-client / state-root proof path.** A browser cannot hold the whole Web; it needs `fabric/feed.py`'s feed/multiproof surface exposed as a verifiable light read so a tab can confirm a molecule against `web_state_root` without full replication, and an epoch checkpoint to anchor that proof.
4. **A hardened, typed canonical boundary at the wire.** Per-container length budgets in `canonical._decode` (today only `MAX_DEPTH`/`MAX_ITEMS`) and an integer-only relation-weight type closing the float quarantine in `interpret`, so untrusted browser peers cannot trigger heap-amplification or a determinism cliff.

The strategic point is that none of this is molgang's to build. molgang's job is chemistry, NPCs, and play; PULSE's job is to be the engine that makes that play a real crypto Web. The seam is already drawn — `App`, `Wallet`, `FabricNode` — and three of the four additions extend existing abstractions rather than introduce new ones. That is what makes the plan credible: the substrate is built, tested, and import-stable; the remaining work is reach (the browser carrier) and rigor (epoch binding, light proofs, the typed canonical boundary), not a rewrite.



## 7. The Role of LENS — the Agentic Reasoning Layer

### 7.1 Why a reasoning lobe at all

A content-addressed knowledge Web is, on its own, inert. The molgang chemistry Web today holds roughly 470 multilingual concept nodes and their typed relations; replicated over the Pulse gossip mesh it can hold orders of magnitude more. But a graph of canonical-CBOR records does not *answer questions*, does not *curate itself*, and does not *teach a new player which concepts to knit next*. That gap — between a verifiable store of facts and usable intelligence — is exactly what large language models normally fill, and exactly where they normally fail: ungrounded generation produces fluent, confident, and unfalsifiable claims. Recent surveys are blunt that retrieval-augmented generation reduces but does not eliminate this, because a model "can still misread, over-generalize, or fabricate claims despite being anchored to external documents," and because current training "incentivize[s] confident guessing over admitting uncertainty" ([MDPI review, 2025](https://www.mdpi.com/2227-7390/13/5/856); [arXiv 2505.21072](https://arxiv.org/pdf/2505.21072)).

LENS (the `interpret` package at `src/knitweb/interpret`) is Knitweb's answer. It is the *reasoning lobe* of the web — a Reasoning Language Machine (RLM) seam that turns the knowledge Web into agent intelligence without ever letting an agent's free generation touch the value path. Its design thesis is unusual and deliberate: **the reasoning is deterministic and verifiable; the language model is advisory and gated.** Where a conventional RAG stack lets the model both retrieve *and* assert, LENS splits those concerns into two pure, replayable stages — `retrieve` and `distill` — that any two spiders run to a byte-identical result, and then submits that result as a unit of proof-of-useful-work (PoUW). The output is not prose; it is a signed bundle of typed relations, each carrying its provenance. This is what makes LENS a credible "brain" for agentic NPCs rather than a hallucination engine wearing a chemistry costume.

### 7.2 Stage one — deterministic retrieve

`interpret.retrieve.retrieve()` is a read-path planner with no model logic in it whatsoever. Given a query (a seed CID, free text, or a structured mapping), a `subscription` scope, and the shared `Web`, it performs a constrained graph walk: it derives seeds (`_query_seeds`), expands them via `Web.traverse(depth, rels=...)` and `Web.neighbors`, optionally unions a spatial hit-set from `SpatialIndex.near`, and attaches cheap provenance roots via `provenance.ancestry` for ranking. Two properties make this load-bearing for the rest of the architecture.

First, **epoch safety**: the caller may pass an expected `web_state_cid`, and retrieval fails fast if `web_state_root(web)` disagrees — "so a stale snapshot cannot silently cross web epochs." A reasoning result is therefore always pinned to a specific, hash-identified state of the knowledge graph; there is no ambiguity about *which* Web was reasoned over.

Second, **total determinism**: every collection is sorted, every dedupe is stable, and ranking is a pure integer comparison — candidates are scored by the maximum integer `reputation` found on their incident edges (`_candidate_reputation`) and ordered `(-reputation, cid)`. The result is a frozen `CandidateSet` carrying `cids`, per-candidate `source_cids`, and `source_ancestries`. Two spiders on two continents, given the same query and the same `web_state_cid`, return the identical `CandidateSet`. That reproducibility is the precondition for verifiability in stage two.

Retrieval is pluggable but never at the cost of truth. The `backends` package defines a `RetrieveBackend` protocol with three concrete planners: `InMemoryBackend` (the default no-op frontier), `DASBackend` (a deterministic Distributed-Atomspace-style `kind`-filter, an explicit nod to Hyperon's addressing model), and `VectorBackend` (deterministic lexical token-overlap ranking — a stand-in seam for a future embedding index). Crucially, **whatever a backend proposes is re-validated against the durable Web**: `retrieve` keeps only `cid for cid in selected if cid in web.nodes`. A clever or adversarial planner cannot inject a CID that was never woven. The semantic index is allowed to *rank*; it is never allowed to *invent*.

### 7.3 Stage two — gated distill, the anti-hallucination gate

`interpret.distill.distill()` consumes the `CandidateSet` and emits a `Selection` of typed `synaptic.bytecode.Relation` tuples — `(subject, predicate, object, source_type, weight)`. It runs a **strictly bounded** loop (`max_iters`, with `mode in {"reflect","recurse"}` and a `max_prompt_bytes` ceiling), and it explicitly *refuses to concatenate relation content into one giant prompt* — the module docstring calls this out, because monolithic-prompt stuffing is precisely how context-overflow hallucinations arise. Instead it walks candidates one at a time, deterministically deriving a relation from each, deduplicating by relation key, and unioning provenance sources for reproducibility.

The decisive function is `_gate_relation`. Before any relation is admitted to the `Selection`, it must pass three independent deterministic checks:

| Gate | Check | What it prevents |
|---|---|---|
| **Membership** | `subject`, `predicate`, `object` each exist in `web.nodes` | Fabricated entities — "every triple must point only at things that were actually woven" |
| **Acyclicity** | `provenance.is_acyclic(web, x)` for all three CIDs | Circular justification / provenance loops |
| **Attestation** | `attest.node_is_attested(web, x)` for all three | Unsigned or unattested claims entering the curated layer |

This is the architectural inversion that makes LENS honest. In ordinary RAG the model emits text and a citation layer *tries to retrofit* support — Citation-Enhanced Generation, for instance, runs iterative NLI cycles "until each statement is backed by entailing citations" ([Faithfulness-Aware UQ, 2025](https://arxiv.org/pdf/2505.21072)). LENS reverses the burden of proof: **a relation that cannot be grounded in attested, acyclic, woven nodes is simply never emitted.** Provenance is not a post-hoc decoration; it is a precondition of existence. An agentic NPC physically cannot publish a fact about benzene that does not trace to attested benzene nodes already in the Web.

The relation `weight` — the only place where soft signals like reputation, recency, and PoUW-quality are blended — is itself float-quarantined into determinism by `quantize.quantize_weight`. The float signals are collapsed to integer thousandths exactly once at the edge (`int(x*1000)`), then blended in pure integer fixed point: `(6000*rep + 600*recency_milli + 7*pouw_milli) // 10000`, clamped to `0..255`. This keeps weights byte-stable for compact signatures and, per the architecture audit (Gap G3), keeps floats off the canonical hash path. Honest disclosure: this invariant is today *defended by convention* at the `quantize` boundary rather than enforced by a hard type, and tightening it into a non-float relation-weight type is a tracked next increment (roadmap R3). It is the one sharp edge in an otherwise clean layer.

### 7.4 Where the language model actually lives — and where it does not

It is worth being precise, because the project's vocabulary and its integrity demand it: **today's LENS modules are pure-Python, stdlib-only, and contain no model inference.** The MeTTa/Hyperon influence is structural, not a dependency. Hyperon represents knowledge in a Distributed Atomspace hypergraph and reasons over it with the self-modifying MeTTa language ([SingularityNET](https://singularitynet.io/research/opencog-hyperon/)); LENS mirrors that separation by keeping all graph truth in `fabric.web.Web` (the metagraph) and all selection logic in deterministic, replayable stages (the reasoning), with `DASBackend` as the explicit addressing analogue. The difference — and the advance — is that LENS's reasoning steps are *content-addressed and economically verifiable*, which Atomspace reasoning is not.

So where does the agentic LLM enter? As an **advisory planner inside a backend, never as an asserter.** A future model-driven `RetrieveBackend` may propose which candidate CIDs look most relevant to a player's question; it may suggest which `(subject, predicate, object)` triple to *attempt*. But its suggestions are funnelled through exactly the same `revalidated`/`_gate_relation` gauntlet as any other backend. The model's creativity is harnessed for *ranking and proposal*; the Web's attestation graph remains the sole arbiter of *truth*. This is metadata-aware abstention taken to its logical conclusion: rather than prompting a model to "abstain when uncertain" ([Lakera, 2026](https://www.lakera.ai/blog/guide-to-hallucinations-in-large-language-models)), LENS makes abstention the *default* — an ungroundable claim is dropped by construction, and an empty `Selection` is a valid, honest answer. Calibrated confidence is expressed not as a model's self-reported probability but as the integer `weight` derived from attestable reputation, recency, and verified-work signals — a number two parties can recompute and agree on.

### 7.5 Reliability through reversible feedback

LENS does not freeze its judgments. `interpret.feedback` closes a low-cost, replayable signal loop (issue #111): a confirmer signs a `distill-feedback` record (`record_feedback`) attesting that a bundle's relations were `upheld` or `overturned`, weaving it into the Web with `confirms-relation` edges. `collect_feedback_records` verifies every signature (`crypto.verify` over canonical bytes, with `address(author_pub) == confirmer`) before accepting it, and `replay_feedback` deterministically replays the events — `+1` for upheld, `-1` for overturned — to adjust each relation's `weight` (floored at zero). Because the whole loop is signed, content-addressed, and order-stable (events replayed sorted by CID), the Web evolves a durable, *auditable* quality signal over time. A relation that survives challenge gains weight; one that the community overturns decays. This is reliability as an emergent, verifiable property of the fabric — not a number a model asserts about itself.

### 7.6 Verifiable compute: LENS as proof-of-useful-work

The final move is what distinguishes LENS from every RAG pipeline in the landscape: **a reasoning result is a unit of mined value.** `pouw/job.py` registers distillation as a first-class job. `execute_distill` runs `retrieve → distill` and returns a signed bundle plus a deterministic `manifest_digest` (`distill_manifest_digest`); `verify_distill` *re-runs the identical pipeline against the same Web* and asserts the recompiled digest matches. The job is registered as **SPLIT-verified** (`job.py:129` rejects any non-split distill verification): settlement requires deterministic re-execution by a committee, a closed dispute window, and no upheld dispute, before escrow releases and bounded PLS is minted as a coinbase Fiber.

The consequence is profound. When an agentic NPC reasons over the molgang Web and publishes a curated relation bundle, a committee of other spiders independently re-derives that exact bundle from the same hash-pinned Web state. If they get the same digest, the work is *proven useful* and the agent earns PLS — newly minted only as demand-gated reward for verified-useful-work, now epoch-bound to the Pulse heartbeat. If they cannot reproduce it, the bundle is rejected and collateral is at risk. Hallucination is not merely discouraged; it is **economically unprofitable and physically unmintable**. LENS is therefore the layer that converts a passive knowledge Web into active, self-curating, monetised agent intelligence — deterministic where the money is, advisory where the creativity is, and provenance-bound everywhere in between.

### 7.7 Build status (honest)

| Component | Status |
|---|---|
| `retrieve` (deterministic walk, epoch-pinned, backend seam) | **Active**, tested |
| `distill` (bounded loop, `_gate_relation` membership+acyclicity+attestation) | **Active**, tested |
| `feedback` (signed, replayable up/down-weighting) | **Active**, tested |
| `quantize` (integer-fixed-point weight) | **Active**; float-boundary hardening (R3) pending |
| PoUW wiring (`execute_distill`/`verify_distill`, SPLIT, manifest digest) | **Active**, tested |
| Model-driven `RetrieveBackend` (agentic LLM as advisory planner) | **Seam present** (`VectorBackend`/`DASBackend`); LLM planner to-build |

LENS today is a working deterministic reasoning lobe with real anti-hallucination gates and real PoUW backing; the agentic LLM that rides on top is the named, well-isolated extension point — the next thing to build, on a substrate already designed to keep it honest.

Sources: [MDPI hallucination-mitigation review](https://www.mdpi.com/2227-7390/13/5/856); [Faithfulness-Aware Uncertainty for RAG (arXiv 2505.21072)](https://arxiv.org/pdf/2505.21072); [Lakera LLM hallucinations guide](https://www.lakera.ai/blog/guide-to-hallucinations-in-large-language-models); [OpenCog Hyperon (SingularityNET)](https://singularitynet.io/research/opencog-hyperon/).



## 8. Role of MONITOR (observability & trust)

A serverless dapp removes the one thing every user of a centralised service silently relies on: an operator who can tell you the truth about the system. When there is no company behind a status page, no admin who can read the database, and no support desk to confirm "yes, your transaction settled", trust has to be manufactured from the protocol itself — out of signed records, reproducible state roots, and locally verifiable evidence. **MONITOR** (the `knitweb_monitor` package at `/Users/develuse/repo/knitweb-monitor`) is the component whose job is to turn the raw machinery of Pulse into something a human can see and believe *without trusting MONITOR either*. It is the eyes of the Web, and — crucially — eyes that any participant can run themselves.

### 8.1 What MONITOR is today

MONITOR is deliberately the most humble package in the stack, and that is its design thesis. It is a **single Python file** (`src/knitweb_monitor/__init__.py`, ~380 LOC), **stdlib-only**, **zero dependencies**, cross-platform (Windows/macOS/Linux), and **strictly read-only**. The README states the safety contract plainly: "It polls HTTP endpoints and reads wallet snapshots — it never writes state, moves funds, or speaks a node's wire protocol", and it binds to `127.0.0.1` by default. There is no build step, no framework, no native code; it is `pip install knitweb-monitor` or `python -m knitweb_monitor` straight from a checkout. This matters enormously for a trust tool: a dashboard you cannot audit in an afternoon is itself something you have to trust, which defeats the purpose. A ~380-line file that imports only `http.server`, `urllib`, `socket`, and `json` is a dashboard whose own correctness a moderately technical user can verify.

Architecturally it is a tiny `ThreadingHTTPServer` exposing four JSON endpoints and one HTML page:

| Endpoint | Source | What it surfaces |
|---|---|---|
| `/api/nodes` | local wallet snapshot via optional `knitweb.store` + a TCP `port_live()` probe | PLS balance, `pls1` address, `seq`/`nonce`, last 25 transfers, daemon liveness |
| `/api/graph` | ledger transfers + MOLGANG `/api/web` links | a two-layer force-directed graph (ledger edges blue, knowledge edges purple, ⚓ = OriginTrail-anchored) |
| `/api/molgang` | each session's `/api/state` + `/api/web` | tables, seated players (level, woven count), woven fabric terms with confirmation counts |
| `/api/health` | self | version, whether `networkx` is present |

Three properties of the current build are worth drawing out because they are the seeds of the larger role:

1. **Graceful degradation.** Both heavy capabilities are *optional and auto-detected, never required*. Without the `knitweb` package the node panel still shows liveness via a raw TCP connect (`port_live`, a 0.4 s `socket.create_connection`); the MOLGANG and graph views work fully over HTTP. Without `networkx` it falls back to a **deterministic, pure-Python Fruchterman–Reingold layout** (`_layout`, seeded `random.Random(7)`, 120 iterations). Determinism here is not cosmetic — a seeded layout means two operators watching the same Web see the *same* picture, which is the difference between a dashboard and a shared reference.

2. **It already reads the sacred data, not a summary of it.** The node panel walks `node.braid.fibers` directly and computes per-`Fiber` balance deltas keyed by truncated `knit` CID. It is reading the L1 ledger primitives described in the architecture reference — `Braid`, `Fiber`, `Knit` — rather than a server's opinion about them.

3. **It already understands the two-layer model.** `build_graph()` weaves the **ledger layer** (PLS transfers between watched nodes, reconstructed by pairing negative/positive deltas under a shared `knit`) and the **knowledge layer** (MOLGANG `subject —relation→ object` triples) into one force-directed view. This is conceptually exactly right: the economic Web and the knowledge Web are the same content-addressed fabric seen from two angles.

### 8.2 Why observability *is* trust in a serverless dapp

In a custodial app, the dashboard and the authority are the same party, so the dashboard is believed by fiat. Remove the authority and a dashboard becomes an *oracle problem*: why should I believe what this UI tells me about my balance, about whether my contribution was rewarded, about whether the chemistry fact I knit was accepted? Decentralised systems answer this with verifiability rather than authority — the broad pattern Vitalik Buterin describes as moving from "don't be evil" to "can't be evil" ([vitalik.eth.limo](https://vitalik.eth.limo/general/2022/05/25/stable.html)), and the same instinct behind public chain explorers like [Etherscan](https://etherscan.io/) and node-health maps such as [Ethernodes](https://www.ethernodes.org/) or Bitcoin's [Bitnodes](https://bitnodes.io/). MONITOR's role is to be that explorer *for the knitweb*, but with three differences that the Pulse architecture makes possible and necessary:

- **It runs on the operator's own machine, reading their own node.** There is no `monitor.knitweb.io` to trust. The trust root is local: MONITOR reads the wallet file you control and probes the port you run. This is the only honest design for a system whose whole point is the absence of a central server.
- **The data it shows is independently re-derivable.** Because every record is canonical-CBOR with a CIDv1 identity, and convergence is witnessed by an identical `web_state_root`, MONITOR's claims are checkable: two MONITORs on two machines that have synced should compute byte-identical graph structure and the same state root. A dashboard whose output is reproducible is a dashboard that does not need to be trusted.
- **It must make the economic loop legible.** The architecture's defining thesis is the loop *Web records → distilled relation bundles → PoUW jobs → escrow settlement → minted PLS → Knits woven back into the Web*. A user only believes "useful work mints money, fairly" if they can *watch* a unit of work travel that loop. Today MONITOR shows the endpoints of that loop (transfers, woven terms, confirmations); its strategic job is to show the **whole arc**.

This is the honest line between built and to-build. What exists is a clean, auditable, read-only window onto liveness, balances, transfers, and the two-layer graph. What is *not yet built* is visibility into the layers that most need a trust story: PoUW verification, dispute/settlement, epoch-bound issuance, personhood, the agentic players, and the p2p mesh's actual shape and health.

### 8.3 What MONITOR must add

The gap is best framed against the architecture's own layer map. MONITOR today sees L1 (ledger) and L5 (the MOLGANG knitweb). To deliver trust for a serverless, agent-populated dapp it must grow visibility into L2 (p2p), L4 (PoUW), L6 (token/epoch), and the cross-cutting personhood and interpret/LENS lobes.

**1. PoUW & settlement visibility (L4).** This is the single most important addition, because it is where "money is minted fairly" is either provable or merely asserted. MONITOR needs a panel that follows a job through its lifecycle: escrow opened → worker `execute` → committee selected (`committee.select_committee`, worker excluded) → `k` samples (`sampling.required_samples`) → `Verdict`s → BFT `⌊2n/3⌋+1` `quorum.tally` → dispute window → `release`/slash → coinbase Fiber mint (`Treasury.reward_verified_work`). Showing the quorum margin, the sample size, the open dispute windows, and the anti-replay digest of the settling proof turns issuance from a black box into an auditable event. For MOLGANG specifically, this is what lets a player see *why* their curation earned PLS.

**2. Epoch / Pulse & emission dashboard (L0+L6).** The architecture flags as its CRITICAL gap (G1) that Pulse is not yet bound to issuance. As that seam closes (roadmap R1: per-epoch `epoch_mint_cap`), MONITOR should render the heartbeat directly — current Beat/epoch, `verify_chain()` status, per-epoch escrowed demand vs. emitted PLS, and the supply curve. A visible heartbeat with a visible, capped mint per beat is the most powerful single artefact of monetary honesty the dapp can offer; an invisible one invites exactly the "where do the coins come from?" suspicion that kills token credibility.

**3. Agent activity (LENS / NPC players).** MOLGANG's NPCs are agentic LLMs acting as first-class players, and LENS distill jobs are themselves SPLIT-verified PoUW. Users must be able to *tell agents from humans and watch what agents do*, or an agent-populated web feels like a bot farm. MONITOR should surface, per agent: which `DistillManifest`s it produced, the retrieve→distill provenance chain (every relation gated on attestation + acyclic provenance via `distill._gate_relation`), its acceptance/dispute rate, and its PLS earned. Agent transparency is a trust primitive, not a feature.

**4. Reputation & personhood overlay.** The p2p layer already maintains per-peer reputation with per-round decay, and the personhood gate enforces one-person-one-scope nullifiers with epoch-pinned non-revocation. MONITOR should overlay reputation onto the node/peer view and show personhood status as a *badge* — verified-unique vs. unverified — **without ever rendering PII**, honouring the ICH-style privacy model (revocable proof, pairwise DIDs, no PII on-fabric). The right display is "this peer is a distinct verified person and has reputation R", never an identity.

**5. Network maps (L2 mesh health).** Today MONITOR probes a single TCP port for liveness. A real trust tool for the Web needs the *shape* of the mesh: the gossipsub eager/lazy peer sets, Erlay reconcile sessions and their byte budgets (directly relevant to the architecture's RED interop test G2 and `ServeBudget` throttling), Kademlia routing-table occupancy, anti-entropy convergence lag, and ban/policing events. A peer-graph map that shows convergence (identical `web_state_root` across peers) is the visual proof that a leaderless mesh actually agrees.

### 8.4 How it composes as the `knitweb_monitor` package

MONITOR's composition principle should remain exactly what it is now and resist the gravity toward a framework: **stdlib-only, single-process, read-only, locally-run, gracefully degrading.** The clean way to add the L2/L4/L6 panels is to follow the pattern the code already establishes — each new capability is an *optional, auto-detected* data source feeding one more JSON endpoint and one more tab, never a hard dependency. Concretely:

- New read-only endpoints (`/api/pouw`, `/api/epoch`, `/api/agents`, `/api/mesh`) mirror the existing `/api/nodes` / `/api/graph` / `/api/molgang` handlers, each wrapped so that an absent source degrades to "panel shows what it can" rather than failing — the same discipline as the optional `knitweb` and `networkx` imports.
- The data must come from **read-only seams**: signed records, exported state roots, and a node's own health export — never by MONITOR gaining write or wire-protocol capability. The package's safety contract ("never writes state, moves funds, or speaks a node's wire protocol") is the load-bearing reason it can be trusted, and it must not be traded away for richer telemetry.
- Determinism stays sacred: seeded layouts and integer-derived displays so two operators see one picture.

Because the MOLGANG target is a dapp that **ships its own source code** and lives entirely P2P, MONITOR completes that story: a < 400-line, dependency-free, auditable observability tool that any player can run on their laptop to verify the health, economy, and honesty of a web with no server and no operator. It is the component that lets a serverless system say, credibly, *don't trust us — there is no us; run the monitor and check.*



## 9. Role of VBANK (governance, treasury & the PLS economy)

A serverless dapp has no operator to phone, no admin console, no "trust me" upgrade key. If MOLGANG is to live entirely on the P2P web, the community that plays it must also be the community that *steers* it — deciding what counts as useful work, how the treasury is spent, which NPCs are sanctioned, and how new PLS enters circulation. That self-steering is the job of **VBANK**, the L5 governance knitweb. VBANK is not a bolt-on voting widget; it is the constitutional layer of the dapp, the place where a leaderless community converts opinions into deterministic, content-addressed, independently-auditable decisions — and then binds money to those decisions through the Pulse-gated treasury.

This chapter describes VBANK as it stands in the tree (`src/knitweb/knitwebs/vbank/{poll,ranked,liquid,tally}.py` and `src/knitweb/knitwebs/crowdfunding/`), the PLS economy it governs (`token/mint.py`, now epoch-bound), and the design that makes on-chain-less, plutocracy-resistant governance credible for a game that ships its own source code.

### 9.1 The governance problem VBANK answers

The open landscape gives us a sharp, well-measured problem. Token-weighted DAO governance has visibly failed on its own terms: by 2025 the top 10% of holders controlled **76.2%** of voting power, **73%** of contentious votes were swung by holders of >30% of supply, and average turnout sat near a dismal **17%**, with many proposals drawing under 2% participation ([How DAOs Failed to Deliver](https://lopetaku.medium.com/dao-governance-failures-whales-low-turnout-attacks-d1375c556384), [Frontiers](https://www.frontiersin.org/journals/blockchain/articles/10.3389/fbloc.2025.1538227/full)). One-token-one-vote turns governance into a function of wealth, and wealth into a function of governance — a closed plutocratic loop. The 2026 reaction has been a decisive move *away* from pure token voting toward hybrid, reputation- and personhood-weighted models ([DAO Governance 2026: The End of Token Voting](https://pen-caforr.org/2026/04/15/dao-governance-2026-hybrid-models-legal-wrappers-and-the-end-of-token-voting/)), with quadratic and delegated schemes lifting turnout from ~2.8% to ~11.4% where adopted ([Voting Strategies](https://www.chainscorelabs.com/en/glossary/web3-social-and-creator-economy-models/creator-dao-governance/voting-strategies)).

VBANK takes the radical-but-honest position implied by Knitweb's whole architecture: **the unit of governance is the person, not the coin.** A vote is admitted because a unique, verified human cast it, and counted exactly once — not because a balance backs it. This is only credible because Knitweb already carries a Sybil-resistant **personhood** foundation. VBANK is described in its own doc as *"the first consumer of the personhood foundation"* (`docs/VBANK.md`), and every ballot it admits is gated by `personhood.gate.require_personhood`, which issues a `PersonhoodTicket` carrying a `scope_nullifier` and a pairwise address but **no PII** — proof of unique-personhood without identity ever touching the fabric. This is the same architectural bet that World ID and Human Passport (formerly Gitcoin Passport) have validated at scale: Passport-style proof-of-personhood cut attacker influence in Gitcoin grant rounds by **over 80%**, and by March 2026 protected 120+ projects and >$512M in capital flow ([human.tech](https://human.tech/blog/human-passport-proof-of-personhood-and-sybil-resistance-for-web3), [Digitap](https://digitap.app/news/guide/proof-of-personhood-solving-sybil-attacks)). Knitweb's differentiator is that the personhood gate is *native and privacy-first* — revocable proof, pairwise DIDs, one-person-one-scope nullifiers — rather than an external oracle bolted onto a token contract.

### 9.2 What VBANK is, mechanically

VBANK is a family of pure-Python, integer-only, signature-and-CBOR record kinds woven into the Web. There is no smart contract and no chain; a "vote" is just a signed `vbank-ballot` record that any peer can read, and a "result" is a *deterministic re-computation* that any peer can reproduce byte-for-byte. The guardrail VBANK upholds is explicit in the doc: *deterministic tally and public audit trail*. The pieces:

| Module | Record kinds | Role |
|---|---|---|
| `tally.py` | `vbank-ballot`, `vbank-tally` | One-person-one-vote dedup by `scope_nullifier` (highest `seq` wins, CID tie-break); integer counts; Merkle `ballot_root` over included ballots. |
| `poll.py` | `vbank-poll`, `vbank-result` | Authority-signed poll definition (option count, window, `quorum`) and the certified, attributable outcome linked by `poll_cid`. |
| `ranked.py` | `vbank-ranked-ballot`, `vbank-ranked-result` | Instant-runoff (IRV): deterministic elimination rounds with smallest-id tie-break, exhausted-ballot handling. |
| `liquid.py` | `vbank-delegation`, `vbank-liquid-result` | Liquid/delegated voting: weight flows along signed delegation chains; direct vote overrides delegation; cycles and dead-ends abstain. |

The design is deliberately layered so that the *gate* (admitting a ballot) and the *tally* (counting it) are separate concerns. `tally.tally(...)` operates only on records already admitted to the fabric and decides which of them count — it does **not** verify signatures or personhood tickets, because that is the gate's job at emit time (`VbankKnitweb.emit` → `require_personhood`). This separation is what makes the result trustless: the audit step (`audit_result`, `audit_ranked_result`, `audit_liquid_result`) re-collects the ballots from the Web and recomputes the outcome, so a malicious "authority" cannot publish a result that does not match the ballots. The authority's signature certifies *attribution and timing*, not *correctness* — correctness is reproducible by anyone.

Three properties make this credible for a serverless game. First, **order-independence**: the tally is computed from a *set* of ballots with no reliance on arrival order or timestamps, so two peers that received ballots in different orders still derive an identical `vbank-result` CID. Second, **re-vote tolerance**: a voter may change their mind (highest `seq` ballot wins), which matters in a long-running game where opinions shift. Third, **weighted-but-auditable**: `tally` accepts an optional `{scope_nullifier: weight}` map of *non-negative fixed-point integers* (no floats on the canonical path, per the project non-negotiable) and commits to a `weight_root`, so a community can choose reputation- or contribution-weighted voting — e.g. weight by curated PLS-earning PoUW contributions — without abandoning auditability. This is exactly the hybrid personhood-plus-reputation direction the 2026 landscape is converging on, but expressed natively in canonical CBOR rather than Solidity.

### 9.3 Governing MOLGANG: the concrete decisions

What does the MOLGANG community actually decide through VBANK? The dapp is a chemistry-knowledge game where players knit concepts into a shared content-addressed graph and earn PLS via proof-of-useful-work (curation, contribution, distilled relation bundles from the Lens layer). The governable surface is large and real:

- **Curation policy** — which `vbank-ballot` / ranked polls promote or retire knowledge nodes; whether a disputed reaction edge stays in the canonical graph. Because every knowledge node is already a signed PoUW artifact, governance over *what counts as good chemistry* is a poll over CIDs.
- **PoUW reward parameters** — the `EmissionPolicy` knobs (`rate_num/rate_den` work subsidy, `max_supply`, and the new `epoch_cap`) are policy, not constants. A community vote can ratify a change to the subsidy rate or the per-epoch ceiling.
- **NPC sanctioning** — MOLGANG's NPCs are agentic LLMs that play as first-class participants. Each NPC is a personhood-gated actor under a declared scope; VBANK polls decide which model-signed agents are admitted as players and on what terms, giving the community a kill-switch for misbehaving agents that needs no central operator.
- **Treasury allocation** — ranked-choice polls over how pooled PLS is spent (bounties, translation work for the multilingual explorer, tournament prizes).
- **Self-amendment** — because the dapp *ships its own source code* over the Web, a poll can ratify which source CID is the canonical client, turning upgrades into governed, content-addressed events rather than operator pushes.

Liquid delegation is the turnout answer. Most players will not vote on every curation question; with `liquid.py` they delegate their weight to a trusted curator (transitively, A→B→C), retaining the power to override by voting directly. This is the mechanism shown to lift participation in the open landscape, implemented here as signed `vbank-delegation` records rather than a delegation registry contract.

### 9.4 Treasury, crowdfunding, and the PLS economy

Governance without a treasury is theatre. VBANK's economic half is the **crowdfunding** knitweb plus the **token** treasury, and the two are mirror images. A `CrowdfundingCampaign` (`crowdfunding/campaign.py`) is an authority-signed `crowdfunding-campaign` record — a funding `goal` in PLS-wei, a window, and a `beneficiary` address — that collects gated, signed `crowdfunding-pledge` records. Unlike votes, pledges are *not* deduped on the nullifier (a person may pledge repeatedly), but the nullifier is retained so the campaign can prove every pledge came from a *distinct verified person* and count them — Sybil-resistant fundraising with no identity on the fabric. The certified `crowdfunding-outcome` records how much was raised in-window and whether the goal was met, with a `pledge_root` for independent audit. Crucially, settlement is *real value movement, not an instruction*: `crowdfunding/settlement.py` turns a certified settlement into actual dual-signed L1 Knit transfers from the campaign **escrow** account to the beneficiary (on success) or back to pledgers (on refund), with conservation enforced by the ledger itself. The doc is honest about scope: this models donation/reward fundraising; investment and lending flows are deferred pending EU Reg. 2020/1503 review.

The supply side is `token/mint.py`. The economic non-negotiables are strict and exactly the anti-plutocracy posture the project needs: **no premine, no admin mint, no privileged genesis allocation** — native PLS exists *only* as a bounded, demand-gated reward for verified useful work (`Treasury.reward_verified_work`). Every mint is a coinbase `Fiber` tagged with its issuance CID, so the Braid's spent-knit guard makes double-minting impossible, and an anti-replay digest set ensures the same proof is rewarded at most once. The reward is bounded three ways: never more than the consumer's escrow (mint ≤ proven demand), never past an optional `max_supply`, and — the key 2026 increment on `feat/epoch-bound-issuance` — never past the remaining `epoch_cap` for the **Pulse** epoch the mint falls in. The `Treasury` now takes a `Pulse` and derives each mint's epoch from its injected timestamp via `pulse.epoch_at`, so *activity (Beats) gates the issuance rate*. This is what makes the PLS economy governable: VBANK does not vote on a balance sheet it cannot constrain — it votes on `EmissionPolicy` parameters (`rate_num/rate_den`, `epoch_cap`) that the Pulse heartbeat then mechanically enforces, closing the activity→money loop under community policy rather than operator discretion.

### 9.5 On-chain-less governance, honestly assessed

What VBANK delivers today is substantial and tested end-to-end: poll, ranked, liquid, and weighted tallies; personhood-gated, privacy-preserving, deterministic, independently-auditable results; crowdfunding with real escrow settlement on L1; and a no-premine, demand-gated, now epoch-bound PLS supply. That is a genuine on-chain-less governance stack — decisions are content-addressed records, not contract state, and they converge across the gossip mesh exactly like any other Web record.

What remains to-build is equally clear and worth stating plainly. Authority-certification for ranked and liquid results is described in-code as *"a thin follow-up"* — the pure algorithms and read-models exist; wiring them into `certify_result` so an authority signs a delegated result is pending. Per-ballot voting-window enforcement at tally time awaits a cast timestamp on the ballot (the window lives in the poll definition now). And the deepest open question is *who is the "authority"* in a leaderless game: today it is a signing key that certifies (but cannot falsify) outcomes, which is acceptable because audit is trustless — but the natural endpoint is to make the authority itself a VBANK-elected, rotating, personhood-gated role, so even certification is governed. Those are increments, not redesigns: the load-bearing properties — one-person-one-vote, no PII on-fabric, deterministic reproducible tallies, conservation-preserving settlement, no privileged mint — are already enforced in the code. VBANK is the layer that lets a serverless MOLGANG community own its rules, its money, and its own source — and steer all three without ever asking permission from a server that does not exist.

Sources: [How DAOs Failed to Deliver](https://lopetaku.medium.com/dao-governance-failures-whales-low-turnout-attacks-d1375c556384) · [DAO Governance 2026](https://pen-caforr.org/2026/04/15/dao-governance-2026-hybrid-models-legal-wrappers-and-the-end-of-token-voting/) · [Voting Strategies](https://www.chainscorelabs.com/en/glossary/web3-social-and-creator-economy-models/creator-dao-governance/voting-strategies) · [Frontiers: Decentralizing governance](https://www.frontiersin.org/journals/blockchain/articles/10.3389/fbloc.2025.1538227/full) · [human.tech: Human Passport](https://human.tech/blog/human-passport-proof-of-personhood-and-sybil-resistance-for-web3) · [Digitap: Proof-of-Personhood](https://digitap.app/news/guide/proof-of-personhood-solving-sybil-attacks)



## 10. Agentic LLM integration: NPCs, verifiable inference, and the agent economy

molgang is not a game that *has* an LLM bolted on. It is a knitweb whose first-class citizens are agentic LLMs that play, teach, curate, and earn, sitting on exactly the same footing as a human at the bar. A human "knits" a chemistry concept into the shared content-addressed Web; an agentic NPC knits the same way, signs the same canonical-CBOR record, settles the same PLS Knit, and is held to the same verification quorum. The thesis of this chapter is that the Pulse substrate already built — content-addressed records, PoUW verification, the Lens reliability gate, integer-only settlement, and the personhood gate — is precisely the machinery you need to make autonomous agents *safe and economically legible* participants rather than a hallucination firehose pointed at your canonical knowledge graph. The hard problems of the 2026 agent landscape (where does inference run, can you trust its output, how does an agent pay and get paid) map cleanly onto subsystems Pulse already ships.

### 10.1 NPCs as first-class knitters, not chatbots

A molgang NPC is an agent with its own secp256k1 keypair, its own `pls1` address, and its own PLS balance — indistinguishable at the protocol layer from a human player except for one fact recorded on-fabric: it carries an **agent credential** rather than a personhood proof. Each NPC has a role expressed as a behavioural policy over the Web:

| NPC archetype | What it does in-game | Pulse subsystem it drives |
|---|---|---|
| **Tutor** | Explains a concept node to a new player, cites neighbouring nodes | `interpret.retrieve` (deterministic graph walk) |
| **Curator** | Proposes new edges (`is-a`, `reacts-with`, `catalyses`), flags weak ones | `interpret.distill` → SPLIT-verified PoUW job |
| **Challenger** | Quizzes players, disputes dubious contributions | `pouw.challenge` / `pouw.dispute` |
| **Translator** | Mirrors the ~470-node multilingual explorer across languages | `fabric.web.weave` (idempotent, content-addressed) |
| **Referee** | Sits on verification committees, votes on contribution validity | `pouw.committee` / `pouw.quorum` |

Crucially, an NPC's "turn" is never a free-text blurt into shared state. It is a *proposed record* that flows through the same pipeline as any other: author → canonical CID → Lens gate → PoUW verification → settlement. A Tutor answering a player is cheap and local (a retrieve over already-converged Web state, no write). A Curator proposing a new edge is *expensive and adversarial* — it must survive a committee. This asymmetry is the whole design: agents talk freely, but they cannot *change canonical truth* without paying the verification cost and winning a quorum. That is what keeps an agentic NPC from being a liability.

### 10.2 Where inference runs: edge, browser, hosted, and donated-GPU spiders

molgang's target is a top-10 dapp that lives *entirely* on P2P and ships its own source — so "where does the model run" cannot have a single centralised answer. The architecture is deliberately a **tiered inference market** with graceful degradation:

1. **Edge / in-browser.** For Tutor-class NPCs and lightweight curation, a small quantised model (3–8B class) runs locally via WebGPU/WASM in the player's own browser tab, or natively on a player's laptop. Zero marginal cost, zero server, full offline play. This is the default and it is the only tier required for the game to be *playable* with no infrastructure at all.
2. **Hosted API (Z.AI and peers).** For heavier reasoning — multi-hop curation, dispute adjudication, NPC dialogue with personality — the agent calls a hosted endpoint. This is convenient but trust-asymmetric: you cannot tell whether the returned tokens actually came from the model claimed. That is exactly why hosted inference output is never trusted *as such* (see §10.3).
3. **Donated-GPU spiders via PoUW.** The most interesting tier. Volunteers and small operators run "spiders" — Pulse peers that advertise GPU inference capacity into the `pouw.marketplace`. An agent that needs an inference escrows PLS as a job; a spider executes it; a committee of *other* spiders re-checks (or, for non-deterministic generation, scores it — see below); `pouw.quorum` tallies a BFT `⌊2n/3⌋+1` outcome; `escrow.settle_on_verify` pays the worker in a conservation-preserving Knit; `Treasury.reward_verified_work` mints the bounded PLS reward. This is the same "demand-gated reward for verified-useful-work" loop that mints all PLS — inference is just another flavour of useful work.

This tiered model mirrors where the wider field has landed by 2026. Bittensor's subnet model already routes inference across 8,000+ donated GPU nodes under a "Proof of Intelligence" reward where validators score miner outputs and Yuma Consensus distributes emissions by quality ([Bittensor 2026 guide](https://www.cryptotimes.io/learn/bittensor-tao-guide/); [CoinDesk on decentralized AI tapping idle GPUs](https://www.coindesk.com/opinion/2026/02/22/how-decentralized-ai-is-leveling-the-playing-field)). molgang's spider tier is a special case of that pattern, but settled on Pulse's own integer-only ledger rather than an external chain, and scoped to a single domain (chemistry knowledge) where verification is tractable.

### 10.3 Verifiable inference: making "trust me" unnecessary

The deepest risk of agents-as-players is that LLM output is *unverifiable by construction* — you cannot, from the tokens alone, prove they were produced honestly by the claimed model. The 2026 state of the art splits into two camps, and molgang uses **both** at different value tiers.

**Cryptographic verification (zkML) for high-value decisions.** Zero-knowledge proofs of inference have crossed a real threshold: zkLLM proves GPT-2-class transformer inference with ~287s proof time, a 35.7× improvement over earlier ZKML ([zkLLM](https://www.emergentmind.com/topics/zkllm)), and Lagrange's DeepProve-1 generated the first zk proof of a *full* LLM inference, with LLAMA support on the roadmap ([Lagrange DeepProve-1](https://lagrange.dev/blog/deepprove-1)). The consensus for 2026 is that full-trace zkML is too expensive to run on every token, so the field is converging on **"High-Value Inference"** — prove only the critical decisions (authorising a payment, finalising a canonical edge) — and **"Optimistic zkML"** — generate a proof only when a result is *challenged* ([Extropy zkML 2025 analysis](https://academy.extropy.io/pages/articles/zkml-singularity.html)). Lightweight publicly-verifiable schemes like [VeriLLM](https://arxiv.org/html/2509.24257) and [SVIP](https://arxiv.org/pdf/2410.22307) target exactly this "cheap audit, escalate on dispute" regime.

This maps with almost suspicious neatness onto Pulse's existing `pouw.dispute` machinery. The **optimistic path is the default**: a spider's inference result is accepted, an escrow window opens (`DisputeWindowLedger.submit` with staked collateral), and payment releases only at `release_beat`, strictly after the dispute window (`dispute.py:97-102`). If a Challenger NPC or a human disputes the result inside the window, the worker must produce a verifying re-execution (deterministic re-check for the `verify` uniform path) or a proof; a quorum slashes the collateral on `DETECTED_FAULT`. molgang does not need to invent any of this — it already exists for the OriginTrail/synaptic compile jobs, and inference jobs slot into the same `pouw.job` SPLIT-verified flow.

**Non-deterministic generation: score, don't re-execute.** Pure re-execution only works for deterministic outputs. Free-form NPC generation isn't byte-reproducible, so verification shifts from *equality* to *committee scoring* — the Bittensor-validator pattern, the "runtime judge scores the draft before it ships" pattern that the hallucination literature has settled on ([RAG/agentic hallucination survey](https://arxiv.org/html/2510.24476v1)). A committee (`committee.select_committee`, worker excluded) of other agents scores the generation against the retrieved Web evidence; `quorum.tally` aggregates. The reward is gated on the *score*, not on reproducing the exact tokens.

### 10.4 The Lens reliability gate: agents cannot hallucinate into canonical truth

This is the single most important integration point, because it is where Pulse's existing architecture directly solves the field's worst problem. Studies in 2026 put citation-hallucination rates between **11% and 57%** across deployed models, and crucially note that the citations deep-research agents emit "cannot be reliably verified" ([Cited but Not Verified](https://arxiv.org/html/2605.06635v1); [reference-hallucination detection](https://arxiv.org/pdf/2604.03173)). An agentic NPC let loose on a shared knowledge graph is, untreated, a machine for laundering plausible falsehoods into canonical state.

Pulse's `interpret`/Lens lobe was built to make this structurally impossible. An NPC does not write free relations into the Web. It runs `interpret.retrieve` — a deterministic, subscription-gated walk over the *already-converged* Web (`Web.traverse`/`neighbors`, optional spatial union, provenance ancestry) — and then `interpret.distill`, which runs a bounded loop and **gates every single proposed relation** on (a) an attestation and (b) acyclic provenance (`distill._gate_relation`). Only gated relations survive into a signed `Selection` → `DistillManifest`, which then re-enters PoUW as a SPLIT-verified job (`job.py:201-290`). In other words: **a relation an NPC cannot cite to existing attested Web nodes never becomes canonical.** This is "StrictCitations" / "refuse-without-evidence" ([citation-enforced prompting](https://www.mdpi.com/2076-3417/16/6/3013)) enforced not by a prompt the model can ignore, but by a deterministic gate at the fabric seam the model cannot bypass.

The signed-feedback loop closes the reliability circuit over time. `interpret.feedback` records confirmations and reversals of distill outcomes as weaveable `distill-feedback` records (`feedback.py`), so a Curator NPC whose edges are repeatedly upheld earns a durable positive weight, while one that gets reversed is down-weighted. Reliability becomes a *replayable on-fabric signal*, not a vendor's opaity. One hardening note kept honest from the architecture state: the interpret value path still carries quarantined floats (`quantize.quantize_weight` truncates `float*1000` to int), so the integer-only "no floats near canonical" invariant at the distill→fabric boundary is presently defended by convention and is the one seam to tighten before agent-generated weights flow at volume.

### 10.5 The agent economy: PLS for useful work, agent-to-agent and human-to-agent

Because every NPC holds a `pls1` address and a balance, molgang has a genuine **agent economy**, not a points system. Agents *earn* PLS for verified-useful-work — a Curator whose edge survives quorum is paid via `escrow.settle_on_verify` and the coinbase mint; a spider paid for honest inference; a Referee paid a committee fee. Agents *spend* PLS too: a Tutor that wants a heavier hosted reasoning call escrows PLS into an inference job; an agent buys data access or a hint from another agent. All of it is integer-only two-party Knits woven back into the Web — the loop closes exactly as it does for humans.

This is the same machine-to-machine economy the broader market is racing toward, but settled natively rather than on an external rail. Coinbase's x402 (HTTP 402 + stablecoin) had processed **119M+ transactions on Base and 35M on Solana by March 2026, ~$600M annualised, zero protocol fee**, with Nous Research already billing per-inference for Hermes 4 and World's AgentKit gating agent payments behind human verification ([Coinbase x402](https://www.coinbase.com/developer-platform/discover/launches/x402); [x402 2026 micropayments guide](https://www.autheo.com/blog/x402-gasless-stablecoins-ai-agent-micropayments-batch-settlement-2026)). molgang can and should **bridge** to x402 at the edge — let an external agent pay PLS-priced jobs in USDC via an HTTP-402 gateway, the synaptic/gateway adapter being the natural seam — but the *internal* settlement stays on Pulse Knits, because that is what keeps the game serverless and keeps the verification quorum, the dispute window, and the mint atomically coupled to the same ledger.

Two safeguards make this economy non-toxic. First, **personhood vs agenthood are distinct credentials**: the `personhood.gate` (one-person-one-scope nullifier, pairwise DIDs, no PII on-fabric) gates *human* governance acts — voting in vBank, claiming the human reward tier — so a swarm of agents cannot Sybil the human economy even while trading freely in the agent economy. Second, **emission is demand-gated, soon to be epoch-bound**: PLS is minted only against escrowed demand for verified work, and the roadmap's R1 increment binds issuance to the Pulse heartbeat (`epoch_mint_cap` per Beat). An agent that spins up a thousand sub-NPCs to farm rewards still faces a per-epoch supply ceiling and a committee that must *pay attention to* and *uphold* its work — useless-work mints nothing. The agent economy is therefore self-limiting: agents are first-class earners, but the only thing that prints money is work the Web verifies as useful.

### 10.6 Summary

molgang treats agentic LLMs as keyholding, paying, earning peers whose every canonical action is gated by deterministic retrieval, attested provenance, BFT verification quorums, and dispute-windowed escrow. Inference runs wherever it is cheapest — browser by default, hosted for heft, donated-GPU spiders under PoUW for scale — and is trusted nowhere on faith: optimistic-by-default with zkML/committee escalation on challenge, exactly the regime the 2026 verifiable-inference field has converged on. The reliability gate is the keystone: an NPC simply cannot write an uncited relation into the canonical Web. The result is the rare combination the dapp landscape has been missing — autonomous LLM agents as genuine economic actors, on a substrate that makes their contributions verifiable and their hallucinations harmless.



## 11. Gap analysis: the missing modules that need their own package

The preceding chapters described what Knitweb Pulse *is*: a pure-Python, stdlib-only content-addressed Web (L3), settled by an integer ledger (L1), gossiped over a Byzantine-resistant mesh (L2), with money (PLS) minted only as demand-gated reward for verified-useful-work (L4–L6). That spine is real and well-tested — the architecture reference records 1183 passing property/interop suites across `core`, `ledger`, `p2p`, `fabric`, `pouw`, `token`, `vbank`, `crowdfunding` and `personhood`. But a spine is not a body. The target — MOLGANG as a *top-10, fully serverless P2P dapp that ships its own source and runs agentic LLMs as first-class players* — exposes a precise set of capabilities that the spine deliberately does **not** provide, and that today live as thin scaffolds (`gateway.py`, `sdk/__init__.py` at 168 lines, `store.py` at 210 lines, `edge/`, `inference/`) or not at all.

This chapter does the honest accounting. For each candidate it asks four questions: (a) which concrete gap does it close, (b) where does it sit in the L0–L6 stack or cross-cutting plane, (c) how much is there to build, and (d) **why can it not just live inside an existing module** — the test that separates a genuine new package from feature-creep on an existing one. The vocabulary rule holds throughout: these are knitweb packages, not "network services".

### 11.1 The structural reason new packages are needed

The current code observes a strict invariant: every layer below L5 is *determinism-pure* — integer-only, canonical-CBOR-addressed, no clocks, no I/O on the hash path (`core.canonical` rejects floats and indefinite-length; `interpret.quantize` quarantines its only `int(x*1000)` float touch off the canonical path). That purity is the project's crown jewel and the reason its CIDs are reproducible byte-for-byte. The consequence is that **every capability that is inherently non-deterministic, I/O-bound, or environment-specific must be quarantined into its own package** so it cannot contaminate the pure core. A browser transport, a fiat on-ramp, an LLM call, a blob store, a push channel — each introduces wall-clock time, network non-determinism, or untyped external bytes. Folding any of them into `p2p`, `fabric` or `token` would force a non-deterministic dependency into a layer whose entire value proposition is determinism. That is the architectural test, and most candidates below pass it cleanly.

### 11.2 The proposed packages, evaluated

**`pulse-wasm` — browser & WASM transport (L2 adapter).** Closes the single largest gap between "P2P engine" and "dapp anyone can open in a tab". Today the only transports are `p2p/transport.py` (asyncio TCP) and the `gateway.serve` HTTP shim binding `127.0.0.1`. A browser cannot open raw TCP; a serverless dapp must reach peers over WebRTC data-channels and WebTransport, with WebSocket signalling for NAT traversal. This cannot live in `p2p` because `p2p.transport` is asyncio/socket-bound and Python-runtime-bound, whereas this package must target Pyodide/WASM and a JS data-channel API — a different runtime entirely. It is the prerequisite for "ships its own source": the dapp can only be self-distributing if a fresh browser can bootstrap a `FabricNode` with zero install. **Maturity-to-build: high** (new transport + WASM packaging of the stdlib-only core; the determinism purity actually *helps* — pure-Python compiles to Pyodide cleanly).

**`pulse-blob` — content-distribution / large-asset store (L1.5, between ledger and fabric).** The fabric `Web` is an *in-memory* graph of small canonical records; `store.py` snapshots ledger Braids and feeds to local disk but is not a chunked, replicated blob layer. MOLGANG needs to distribute the ~470-node multilingual explorer graph, avatar art, audio, and — critically — *its own source bundle*. Large binaries must be chunked, Merkle-addressed (CIDv1 already gives the naming), erasure-coded and served by-CID over the mesh with bandwidth accounting. This cannot live in `fabric.web` because the Web is intentionally an in-RAM small-record graph with idempotent `weave`; bolting multi-megabyte chunk transfer onto it would break that invariant and its memory model. It sits *below* fabric as a CID-addressed bytes layer that fabric records *reference* by CID. **Maturity-to-build: high.**

**`pulse-index` — indexer / query layer (L3.5, over fabric).** `Web.traverse`/`neighbors` and `interpret.retrieve` give deterministic graph walks, and `fabric.spatial` has a geohash index, but there is no general secondary-index/query engine: no "all nodes of type X in language Y", no full-text, no aggregation. A serverless dapp has no central database, so the index must itself be a replicated, content-addressed, verifiable structure (think a gossiped inverted index whose root is a CID). This deserves its own package because indexing is a *materialised view* with its own consistency model and update cadence — folding it into `fabric.web` would couple the canonical graph to a mutable derived structure and violate the "Web records are immutable" rule. It pairs naturally with the LENS retrieve path. **Maturity-to-build: medium–high.**

**`pulse-wallet` — identity UX & key-management (cross-cutting / L1-adjacent).** `core.crypto` does secp256k1 + `pls1` addresses; `store.save_node` admits in its own docstring that it "writes the account private key in clear text (file mode 0600)… fine for dev MVP." A real dapp needs encrypted keystores, seed-phrase backup/recovery, hardware/passkey (WebAuthn) signing, multi-account switching, and a session-grant model so an agent can act with scoped authority. This must be its own package because key custody is a security boundary with its own threat model and platform-specific backends (browser WebCrypto, OS keychain, passkey) that have no business inside the deterministic `core.crypto` primitive. It is the human-facing twin of `personhood` (which proves *uniqueness*, not custody). **Maturity-to-build: medium.**

**`pulse-agent` — agent-orchestration / LLM runtime (L4.5, between pouw and knitwebs).** The grounding mandates *agentic LLMs as first-class players/NPCs*. Today `inference/control.py` (652 lines) is a deterministic, signed commit/reveal *inference control plane* — it decides actions from signed atoms, but it is **not** an LLM runtime, prompt orchestrator, or tool-execution loop. LENS (`interpret`) does deterministic retrieve + gated distill, not generative reasoning. An agent package must own: prompt assembly from LENS-cited context, tool/function calling, a turn loop, rate/cost budgets, and — the knitweb-specific part — wrapping every agent action as a *signed record* that re-enters PoUW. It cannot live in `interpret` (which is determinism-pure and float-quarantined) precisely because an LLM call is the canonical non-deterministic operation; it must be quarantined so its outputs only enter the Web *after* attestation. **Maturity-to-build: high.** This is the headline differentiator for the "top-10 unique dapp" goal.

**`pulse-attest` — deterministic-LLM-attestation (L4 extension of pouw.verify).** The hardest and most novel package. If agents are first-class economic actors, their generative output must be *verifiable* to be PoUW-rewardable — but LLM inference is non-deterministic, so the existing `pouw.verify` "re-execute byte-for-byte" path does not apply. This package builds the bridge: model+weights pinning by CID, seed/temperature=0 reproducibility envelopes, sampled re-execution with semantic-equivalence quorum, and signed inference receipts. It is distinct from both `pouw` (which assumes deterministic re-execution) and `pulse-agent` (which *produces* the output) — it is the verification discipline that lets generative work be minted without breaking the "no unverified mint" invariant (G1's downstream concern). **Maturity-to-build: very high; research-adjacent.** Mark as ambitious-but-credible: the v1 can require deterministic decoding so the existing committee/quorum machinery applies almost unchanged.

**`pulse-sdk` & `pulse-docs` — developer SDK and documentation (cross-cutting).** `sdk/__init__.py` is 168 lines and the reference calls the SDK "near-empty"; `gateway.py` is the de-facto app surface because builders hit the same gaps MOLGANG hit. A self-distributing dapp ecosystem needs a real, versioned, typed SDK (JS + Python parity, since browsers run JS) plus generated docs and quickstarts. This is its own package because it is the *stable public contract* over a deliberately-minimal core; letting it leak into `gateway.py` (already a 19k-line grab-bag) is exactly the anti-pattern to reverse. **Maturity-to-build: medium.**

**`pulse-onramp` — fiat/crypto on-ramp & payments (L6 adapter).** PLS is minted by PoUW only; there is no premine and no admin mint. For real users this is a feature, but it means there is *no* path from fiat or external crypto into the economy, and vBank's treasury/crowdfunding (L5) currently settle in PLS only. An on-ramp package brokers external value to PLS via escrowed, attested settlement. It must be its own quarantined adapter because it touches regulated external rails (KYC, custodians) that must never sit inside the trust-minimised `token`/`ledger` core. **Maturity-to-build: high (mostly integration + compliance).** Priority is lower for the MVP dapp than for the broader economy.

**`pulse-presence` — notifications, presence & real-time signalling (L2.5).** A multiplayer bar game needs "who's online", live turn notifications, and ephemeral presence — explicitly *non-persistent*, *non-canonical* state that must NOT be woven into the immutable Web or the ledger. That alone justifies a separate package: presence is soft-state with a TTL, the opposite of everything `fabric` and `ledger` guarantee. It rides on `pulse-wasm`'s data-channels. **Maturity-to-build: medium.**

**`pulse-guard` — moderation / anti-abuse (cross-cutting, L3–L5).** `personhood` blocks Sybils and `p2p.policing`/`reputation` ban byte-floods, but there is no *content* moderation (toxic contributions, spam knowledge-nodes, agent-generated abuse). In a serverless dapp this must be decentralised: community flagging that itself becomes a PoUW job, reputation-weighted. It deserves its own package because it composes personhood + pouw + vbank into a content-governance policy that none of them owns alone. **Maturity-to-build: medium–high.**

### 11.3 Pruned / deferred candidates

- **Mobile/edge runtime** — *partially absorbed.* `edge/runtime.py` and `edge/arglass.py` already give a device-side verify-then-project consumer for AR/humanoid bundles. A full mobile runtime is mostly a packaging concern over `pulse-wasm`; do **not** spin a separate package yet.
- **Persistence/archival** — *fold into `pulse-blob` + harden `store.py`.* `store.py` already does atomic, canonical-CBOR, CID-reproducible snapshots. Archival is the durable tier of the blob store, not a new concept; the only genuinely new need (encrypted key-at-rest) belongs to `pulse-wallet`.
- **Marketplace** — *already L4.* `pouw.marketplace`/`scheduler` exist (maturity 3, "not wired into a live epoch loop"). The gap is *wiring*, not a new package. Mint epoch-binding (reference gap G1) is the real blocker; a knitwebs-level storefront can later be an L5 plugin.

### 11.4 Master table of new packages

| # | Package | Layer | Gap closed | Why not in an existing module | Build | Priority |
|---|---|---|---|---|---|---|
| 1 | **pulse-wasm** | L2 adapter | No browser/WebRTC transport → no zero-install serverless reach | `p2p.transport` is asyncio/TCP/Python-bound; WASM is a different runtime | High | **P0** |
| 2 | **pulse-agent** | L4.5 | No LLM runtime; agents can't be first-class players | `interpret`/`inference` are determinism-pure; LLM calls are non-deterministic and must be quarantined | High | **P0** |
| 3 | **pulse-blob** | L1.5 | No chunked replicated large-asset / self-source distribution | `fabric.web` is in-RAM small-record graph; multi-MB chunking breaks its model | High | **P0** |
| 4 | **pulse-wallet** | cross-cutting | Keys stored cleartext; no recovery/passkey/session UX | Custody is a platform-specific security boundary, not a `core.crypto` primitive | Medium | **P0** |
| 5 | **pulse-sdk / pulse-docs** | cross-cutting | SDK near-empty; builders re-solve gateway gaps | Stable public contract must not leak into the 19k-line `gateway.py` | Medium | **P1** |
| 6 | **pulse-index** | L3.5 | No general/full-text/aggregate query over the Web | Materialised mutable view; coupling it to immutable `fabric.web` violates the rule | Med–High | **P1** |
| 7 | **pulse-attest** | L4 ext. | Generative LLM work isn't verifiable → can't be minted | `pouw.verify` assumes deterministic re-execution; semantic-equivalence quorum is new | Very High | **P1** |
| 8 | **pulse-presence** | L2.5 | No real-time presence/notifications for multiplayer | Soft-state w/ TTL is the opposite of `fabric`/`ledger` immutability | Medium | **P2** |
| 9 | **pulse-guard** | L3–L5 | No decentralised content moderation/anti-abuse | Composes personhood+pouw+vbank; owned by none of them | Med–High | **P2** |
| 10 | **pulse-onramp** | L6 adapter | No fiat/external-crypto path into the PLS economy | Regulated rails must stay outside trust-minimised `token`/`ledger` | High | **P3** |

The four **P0** packages (`pulse-wasm`, `pulse-agent`, `pulse-blob`, `pulse-wallet`) are the minimal set that turns the proven spine into a thing a stranger can open in a browser, play with an LLM, and own — the literal definition of the target dapp. Everything else hardens, scales, or monetises that core experience. Critically, every one of these is buildable *without* relaxing the determinism purity of L0–L4: each is a quarantined adapter at the edge, which is precisely why each earns its own package rather than a patch to the sacred core.



## 12. Tokenomics, Incentives & Go-to-Market

This chapter sets out the economic engine of the Web — the PLS unit, how it comes into existence, how it is earned, how Sybil resistance is enforced through personhood, where durable value accrues — and a credible, staged go-to-market that takes the chemistry-and-AI flagship knitweb (MOLGANG) from a ~470-node explorer on shared hosting to a top-tier, fully serverless dapp. We are deliberately honest throughout about what is built versus to-build, and about regulatory framing: PLS is an **activity-accounting unit**, not a speculative instrument, and we design every surface to keep it that way.

### 12.1 The PLS unit: no premine, PoUW-minted, epoch-bound

PLS ("pulses") is the activity unit of the Web. It meters compute, relay, storage and curation, and is the integer (wei-style) reward for *verified useful work*. The non-negotiable monetary rules are already encoded, not aspirational. In `token/mint.py`, a freshly constructed `Treasury` has minted exactly zero PLS, and there is *intentionally no raw, ungated `mint` method*: the only path to native issuance is `Treasury.reward_verified_work`, which proves the work first. Four invariants govern emission, all integer-only and provable:

| Rule | Mechanism (code) | Effect |
|---|---|---|
| No premine, no admin mint | `Treasury` starts at `total_minted = 0`; no privileged genesis allocation | Founders earn like any spider; no launch float to dump |
| Demand-gated | Mint fires only on a verified `WorkProof` (`pouw.job.verify`, sampled re-execution) | Fraudulent work mints nothing and is slashable |
| Bounded by demand | `EmissionPolicy.reward`: `escrow * rate_num // rate_den`, clamped to `min(r, escrow)` | Mint ≤ the pulses a consumer actually spent |
| Conserved + auditable | Each mint is a *coinbase* `Fiber` on the worker's `Braid`, anti-replayed by `job_digest` | No double-mint; cumulative supply audited as an `Issuance` log |

The default `EmissionPolicy` is a **1/2 work subsidy** (`rate_num=1, rate_den=2`) with an optional hard `max_supply` cap. Economically this means new money is a matching grant on *proven* demand: for every escrow a consumer burns on useful work, the protocol mints at most that much again to the worker, never more. There is no inflation knob disconnected from activity.

The one **load-bearing gap** we name openly (architecture gap G1): emission is bounded by escrow and an optional `max_supply`, but **not yet by the Pulse epoch**. `core/pulse.py` chains `Beat`/`Pulse` deterministically but is not yet wired to a per-epoch mint window. Roadmap increment **R1 (epoch-bound issuance)** closes this: carry an `epoch_mint_cap` on the `Beat`, gate `reward_verified_work` to per-epoch escrowed demand, and settle mints atomically at the epoch boundary. Until R1 lands, PLS is honestly described as *demand-gated but not yet epoch-rate-limited*. This is the single most important tokenomics deliverable before any public liquidity, because it turns the heartbeat from a decorative clock into the monetary governor and gives the supply curve a defensible, auditable cadence.

### 12.2 Knit-to-earn: the incentive surface

Spiders do not mine PLS by burning electricity on arbitrary hashes; they earn it by doing work the Web actually consumed and other peers re-verified. The canonical earning loop (from the architecture's end-to-end value flow) is:

1. A consumer **escrows** pulses for a job (e.g. a `SynapticCompileJob` — compile an OriginTrail asset to signed bytecode; or an `interpret`/Lens distill job).
2. A worker **executes**; a **committee** (`committee.select_committee`, worker excluded) of size `k` (`sampling.required_samples`) **re-executes** and emits `Verdict`s.
3. `quorum.tally` aggregates to a BFT outcome at the `⌊2n/3⌋+1` threshold (`quorum.py`).
4. On a clean verdict and a closed dispute window (`DisputeWindowLedger`, with the `release_delay > dispute_window` safety invariant), escrow settles to the worker as a conservation-preserving `Knit`, and the **bounded reward mints** as a coinbase Fiber.

For MOLGANG specifically, "knit-to-earn" maps onto concrete, gradeable knowledge work rather than gambling: weaving a verified chemistry concept node, adding a correct multilingual translation edge, curating/de-duplicating the shared content-addressed graph, or producing a *signed relation bundle* via the Lens reasoning lobe (`interpret.distill`) that itself re-enters the PoUW flow as a SPLIT-verified job. The reward is proportional to verified contribution, and **provenance is mandatory** — `distill` gates every relation on attestation plus acyclic provenance before it can be canonicalised, so unsourced "knowledge" earns nothing. This is the structural difference from extractive play-to-earn: the only way to be paid is to enlarge or sharpen a graph that other humans and agents then reuse, and reuse is what the verifier samples.

A note on AI agents as first-class earners: NPCs in MOLGANG are agentic LLMs that play by the same rules — they escrow, they distill, they get re-verified. Because verdicts come from re-execution and not from reputation alone, an agent has no privileged mint path; it competes on the verifiable usefulness of its output exactly like a human spider. This is the same "intelligence is rewarded by measurable output" thesis that [Bittensor](https://btcusa.com/decentralized-ai-compute-networks-how-bittensor-render-and-akash-are-building-the-ai-infrastructure-layer/) popularised, but settled on a pure-Python, content-addressed substrate rather than a chain-specific subnet.

### 12.3 Anti-Sybil via personhood

Knit-to-earn collapses the instant one person can be a thousand wallets. The Web's defence is the **personhood gate** (maturity 4, ACTIVE), an orthogonal Sybil layer that vBank governance and crowdfunding call *before* accepting a vote, pledge, or — for our purposes — a one-person-rate-limited reward claim. Its design is privacy-first and ICH-style:

- **One-person-one-scope nullifier** with epoch-pinned non-revocation (`gate.require_personhood`), so a human counts once per scope per epoch.
- **Pairwise DIDs** and a **revocable proof** model — no PII is ever placed on the fabric; the personhood ticket is *decoupled from the content signature* (the ZK seam), so the credential and the work it authorises cannot be linked back to an identity.
- A pluggable `PresentationVerifier` adapter so the actual credential backend (a VC/ZK scheme, or an aggregator like [Human Passport, formerly Gitcoin Passport](https://human.tech/blog/human-passport-proof-of-personhood-and-sybil-resistance-for-web3)) is injected rather than hard-wired.

This matters commercially as much as technically. Sybil-resistant distribution is now table stakes: aggregate proof-of-personhood providers report **Sybil resistance covering 120+ projects and 150+ campaigns, securing over $512M in capital flow** as of early 2026, with grant rounds seeing attacker influence cut by [over 80%](https://www.bydfi.com/en/cointalk/gitcoin-passport-identity-guide-2026). Our position is to consume that ecosystem through the `PresentationVerifier` seam — not to ship our own Orb-style hardware like [Worldcoin](https://cryptodaily.co.uk/2026/05/worldcoin-wld-proof-of-human-identity-ai) — keeping personhood revocable, pairwise, and off-fabric. Crucially, the gate *throttles reward-earning and governance weight*, not read/learn access: anyone can explore and play MOLGANG; only the paid and the enfranchised must prove unique personhood.

### 12.4 Value capture: where worth accrues

We separate three things that are too often conflated:

- **PLS = activity meter.** It captures the *flow* of verified work. Its sink is real consumption — escrow burned on compile/distill/curation jobs — so demand for PLS is demand for useful work, not for holding. This is the [BME/IDE "supply-and-demand tied to real business volume"](https://btcusa.com/decentralized-ai-compute-networks-how-bittensor-render-and-akash-are-building-the-ai-infrastructure-layer/) discipline that has become the de-facto standard for credible DePIN-compute tokenomics.
- **Fiber = account-state value unit.** A `Fiber` is an immutable, content-addressed snapshot of one account state; *Fibers are never transferred*. Value-unit slang is "a fiber". This is where settled worth lives, conserved by the dual-signed `Knit` two-party ledger.
- **The Web itself = the durable asset.** The deepest moat is the content-addressed knowledge graph plus the corpus of signed relation bundles the Lens produces. Each distilled bundle is reusable, provenance-cited, and itself a verifiable work artifact. The graph compounds; the token merely meters access to enlarging it.

Treasury value capture is governed through **vBank** (poll/ranked/liquid voting + tally, end-to-end tested) and the **crowdfunding** knitweb (campaign + settlement, tested). A protocol fee on settled jobs (a small fraction of escrow routed to a vBank-controlled treasury Knit) is the natural, non-speculative revenue line: it scales with *verified work volume*, is set by liquid-democracy governance, and funds the personhood-gated reward pool, education partnerships, and OriginTrail anchoring costs. No part of value capture depends on token price.

### 12.5 Go-to-market: the community/creator flywheel to a top-tier dapp

The landscape is favourable and crowded. Decentralised-compute demand is now a survival mechanism for AI startups facing record GPU costs, with [Akash compute spend breaking $5M in Q1 2026 and AKT up 72% YTD](https://www.weex.com/news/detail/what-are-the-top-ai-crypto-coins-render-vs-akash-5-gems-solving-the-2026-gpu-crisis-692795), and Render at [~$38M monthly revenue](https://www.weex.com/news/detail/what-are-the-top-ai-crypto-coins-render-vs-akash-5-gems-solving-the-2026-gpu-crisis-692795). We do **not** compete head-on for raw H100 supply. Our wedge is a *self-hosting, serverless, agent-native knowledge dapp* — the thing none of the compute incumbents are: a dapp that ships its own source code over the Web and uses agentic LLMs as first-class players.

**The flywheel:** humans and NPC agents knit chemistry knowledge → verified contributions mint PLS and grow the graph → a richer graph makes the Lens answer better and cite provenance → better answers attract more creators and learners → more contribution. Each turn is gated by personhood (no fake spiders) and metered by PLS (work is real).

| Phase | Window | Move | Top-tier metric |
|---|---|---|---|
| 0. Self-hosting cut-over | now–Q3 2026 | Migrate MOLGANG off PHP/`5mart.ml` onto the P2P web (`knitweb/pulse`); ship the dapp source *over* the fabric; land R1 epoch-bound issuance | Game runs with zero central server; CID-stable |
| 1. Creator seeding | Q4 2026 | Onboard chemistry educators/translators as spiders; multilingual ~470-node graph as the genesis corpus; personhood-gated knit-to-earn | First 1k verified human spiders |
| 2. Agent-native loop | Q1 2027 | NPC LLM players escrow + distill as first-class earners; Lens provenance-cited answers as a public good | Agents and humans earning on one verdict path |
| 3. Distribution + partners | 2027 | OriginTrail DKG anchoring; education channels; AI-tooling integrations | Self-sustaining treasury via job fees |

**Distribution & partnerships.** Three credible, non-speculative channels: (1) **OriginTrail** — the `anchor/origintrail.py` resolve+compile path already lets us anchor heavy artifact provenance to the Decentralised Knowledge Graph; MOLGANG's chemistry graph is a natural DKG knowledge asset, and `SynapticCompileJob` makes anchoring itself a paid PoUW job. (2) **Education** — chemistry curricula, university outreach, and bar-game venues (MOLGANG's origin) give a non-crypto-native top-of-funnel; learners need no wallet to explore. (3) **AI** — the Lens/RLM layer is MeTTa/Hyperon-inspired and produces signed, provenance-cited relation bundles, which is a directly marketable artifact to the agent-tooling ecosystem riding the [AI+crypto convergence](https://www.kucoin.com/blog/deep-dive-to-AI-crypto-future).

### 12.6 Regulatory & utility framing — honest by construction

We frame PLS as **activity accounting, not a speculative asset**, and the architecture backs the claim rather than merely asserting it. There is no premine and no privileged genesis allocation (enforced as a non-negotiable, with `tools/check_owner_direction.py` guarding product prose). Issuance is demand-gated and bounded by escrow actually spent on verified work, so supply cannot be conjured ahead of utility. The unit's documented role is to *meter compute, relay, storage and curation* — its sinks are consumption, not holding. Governance over fees and treasury runs through transparent vBank tallies, and personhood gives one-human-one-weight without on-fabric PII. Honest caveats remain: epoch-rate-limiting (R1) is not yet wired, one Erlay anti-amplification test is RED (G2), and the `interpret` float boundary is defended by convention pending R3. We will not list PLS on any speculative venue before R1 closes the activity→money loop and before the personhood gate throttles reward claims in production — because only then is the "activity unit, not asset" framing true in code as well as in prose.



## 13. Technical implementation plan & source-code structure

This chapter turns the vision of the preceding chapters into an engineering blueprint: the concrete repository and package layout, the browser P2P stack that lets a knitweb run with no central server, how source code itself ships peer-to-peer over the Web, the testing and CI discipline that protects the sacred invariants, the API seams between Pulse, Lens, Monitor, vBank and Molgang, and a phased technical sequence that respects what is already built. The guiding principle is continuity, not rewrite. Today's `knitweb/pulse` repository is ~20.7k LOC of pure-Python, stdlib-first code with 102 property suites and 1183 passing tests; the plan extends that spine rather than replacing it.

### 13.1 Repository and package layout

The monorepo `knitweb/pulse` already hosts the layered substrate under `src/knitweb`. The existing tree is the foundation, and the new packages from Chapter 11 slot into it as siblings, never as forks:

```
src/knitweb/
  core/        canonical.py  crypto.py  pulse.py          # L0 — sacred bytes
  ledger/      blob fiber braid knit knitweb node          # L1 — settlement
  p2p/         base_node node transport wire kademlia      # L2 — gossip mesh
               reconcile anti_entropy mesh inventory ...
  fabric/      web node items feed provenance spatial ...  # L3 — the Web
  pouw/        job verify quorum committee dispute ...      # L4 — useful work
  knitwebs/    vbank/  crowdfunding/  chemistry/ (stub) ...  # L5 — plugins
  token/       mint.py                                      # L6 — PLS
  interpret/   retrieve distill quantize backends/          # Lens reasoning
  inference/   control.py                                   # Lens agent control
  edge/        runtime.py arglass.py                        # edge runtime seam
  app/         cli.py                                       # knitweb / pulse CLI
  sdk/  synaptic/  anchor/                                  # adapters
```

Chapter 11's new work lands as the following additions, each a thin, dependency-respecting package:

| New package | Path | Role | Depends on |
|---|---|---|---|
| `pulse.epoch` | `core/epoch.py` | epoch-bound mint governor (closes Gap G1) | `core.pulse`, `token.mint`, `pouw` |
| `lens` (promotion) | `interpret/` + `inference/` re-exported as `knitweb.lens` | agentic RLM, provenance-cited answers | `fabric`, `pouw`, `interpret` |
| `monitor` | `knitweb-monitor` (sibling repo) | observability of the Web/Pulse | read-only `p2p`, `fabric` metrics |
| `vbank` (govern) | `knitwebs/vbank/treasury.py` | epoch-cadenced treasury + PLS policy | `token`, `pouw`, `pulse.epoch` |
| `molgang` | `knitwebs/chemistry/` + `molgang` repo | chemistry knit-game knitweb | `fabric`, `pouw`, `personhood`, `lens` |
| `webnode-js` | `clients/webnode/` | browser P2P node (WASM core) | transpiled `core`, `p2p`, `fabric` |
| `seed` | `tools/seed/` | source-as-content publisher | `core.canonical`, `fabric` |

The non-negotiable rule is the dependency direction: arrows always point **down** the layer stack (L6→L0) and **outward** from cross-cutting Lens/personhood into the substrate, never upward. `core` imports nothing from `knitweb`; `knitwebs/*` may import `fabric`, `pouw`, `personhood`, `lens`, `token` but never each other. Molgang must not reach into vBank's internals — it consumes vBank only through the L5 plugin contract. CI enforces this with an import-linter contract (§13.4) so a careless `from knitweb.knitwebs.vbank import ...` inside Molgang fails the build, not review.

### 13.2 The browser P2P stack (serverless dapp runtime)

Molgang's target — a top-10 dapp that lives entirely on P2P with no central server — requires the substrate to run inside a browser tab. The strategy is **one canonical engine, two runtimes**. The hash-critical, integer-only core (`core.canonical`, `core.crypto`, `core.pulse`, plus `ledger` and the `fabric.web` graph) is the single source of truth in Python; for the browser it is compiled to WebAssembly so byte-identical CIDs are produced on both sides. We evaluated three paths and chose a layered combination.

The **core** ships via Pyodide-class CPython-to-WASM ([Pyodide](https://pyodide.org/)) for an exact behavioural match during the bootstrap phase, then is progressively replaced module-by-module by a hand-audited Rust/AssemblyScript reimplementation of `canonical` and `crypto` compiled to WASM for size and speed. The discriminating test is differential: the WASM `canonical.encode` must produce the same minimal RFC-8949 §4.2 bytes as the Python one for every fuzz vector, or the build is rejected. Because the canonical encoder is hand-rolled and small (`MAX_DEPTH=64`, sorted-key, no-float, no-indefinite), this reimplementation is bounded and auditable rather than open-ended.

The **transport** cannot use raw TCP/UDP in a browser, so the `p2p.transport` seam — already abstracted behind `BaseNode._dispatch` — gains a WebRTC/WebTransport adapter. Peers discover each other through a small set of community-run, stateless **signaling relays** ([WebRTC](https://webrtc.org/) data channels over [libp2p-webrtc](https://github.com/libp2p/js-libp2p)) plus WebTransport to any node that has a public certificate; the relays carry only SDP handshakes, never content, so they are not a trust or availability bottleneck. Once a data channel is open, the existing Erlay reconcile, gossipsub mesh and anti-entropy backstop run unmodified over it because they speak only `wire` frames, not sockets. A browser tab is therefore a first-class `FabricNode`: it weaves records, gossips inventory, reconciles, and serves `inv-data` verbatim to other tabs.

Persistence in the browser uses IndexedDB as the blob store behind the `fabric.items` interface; a service worker keeps the node alive across navigations and lets the dapp work offline, syncing on reconnect via `sync_from`. The diagram in prose: **tab opens → WASM core boots → service worker restores IndexedDB blobs → WebRTC dials 2–3 signaling-discovered peers → Erlay reconcile pulls the diff → gossipsub keeps it live → the player knits, settles Knits, and earns PLS, all with no origin server in the path.**

### 13.3 Build, distribution, and shipping source over P2P

The headline property — the dapp **ships its own source code over the Web** — is implemented by the `seed` tool and is conceptually simple given content addressing. The build pipeline produces a deterministic bundle: `seed publish ./clients/webnode` walks the build output, canonicalises each file into a `blob` record, weaves a directory `Edge` tree into the `fabric.web`, and produces a single root CID that is the immutable identity of that release. Because every byte is content-addressed with CIDv1/dag-cbor, the bundle is self-verifying: a peer fetching it recomputes each CID and rejects any tampered byte.

Bootstrapping is the only piece that touches the legacy web. A tiny (<10 KB) static loader — hostable on any IPFS gateway, a GitHub Pages mirror, or the existing 5mart.ml host — contains nothing but the WASM core hash and a list of bootstrap CIDs. It fetches the WASM core, then asks the P2P mesh for the application bundle **by root CID**, verifying as it goes. After first load, the service worker caches the bundle and the node itself can re-serve it to other browsers: the dapp becomes a distributor of its own source. Updates are new root CIDs announced as signed `fabric` records; clients pin the latest CID their governance (vBank) or their own update policy accepts. This makes the application censorship-resistant and reproducible — anyone can re-derive the published CID from the source and confirm the running code matches.

For Python nodes, distribution stays conventional and dependency-light: a [hatchling](https://hatch.pypa.io/) wheel (`knitweb`), `requires-python >=3.12`, with `cryptography>=42` the only hard runtime dependency and heavier roles (`p2p`, `compute`, `data`) behind optional extras so a pure ledger node stays tiny. The same `seed`-published CIDs let a Python node also fetch and verify a release over the Web, keeping a single distribution truth across runtimes.

### 13.4 Testing and CI

The test pyramid mirrors the existing markers: `property` (deterministic, fast, the bulk — fuzzing `canonical`, parity of crypto, ledger conservation invariants), `interop` (two-or-more-node networked convergence — Erlay byte-budget, mesh propagation, identical `web_state_root`), and `knitweb` (domain plugin end-to-end — vBank tally, crowdfunding settlement, Molgang knit-and-reward). New work obeys three hard gates before merge:

- **Differential WASM gate.** Every `canonical`/`crypto` fuzz vector must yield byte-identical output in Python and WASM. This is the load-bearing test for the browser runtime.
- **Import-contract gate.** An import-linter layered contract enforces §13.1's dependency direction; a Molgang→vBank-internals import fails CI.
- **No-float-near-canonical gate.** A static check plus a runtime property test asserts no float reaches `canonical.encode`, hardening Gap G3 from convention into enforcement.

CI runs on GitHub Actions across CPython 3.12 (GIL) and a 3.14 free-threaded lane (the core is I/O-bound and runs unchanged), executes the full property suite on every push, and gates interop tests on a label to keep PRs fast. The currently-RED `test_p2p_erlay_reconcile.py::test_live_byte_budget_throttles_a_hammering_peer` is a release-blocker: no reconcile/mesh change lands until it is green, and per the cross-session coordination it is reviewed in the p2p lane, not raced.

### 13.5 API seams between components

The components integrate through narrow, signed, content-addressed contracts rather than shared mutable state:

| Producer → Consumer | Seam | Contract |
|---|---|---|
| Pulse → token/vBank | `Beat.epoch_mint_cap` | per-epoch supply ceiling gates `Treasury.reward_verified_work` |
| Lens → PoUW | `DistillManifest` | signed relation bundle re-enters as a SPLIT-verified job |
| Molgang → PoUW | `pouw.job` registration | curation/contribution as verifiable work earning PLS |
| personhood → vBank/Molgang | `require_personhood` | one-person-one-scope Sybil gate before vote/pledge/reward |
| fabric → Monitor | read-only metrics | `web_state_root`, mesh/reconcile counters, no write path |
| seed → fabric | root CID | self-verifying source bundle |

Monitor is deliberately read-only: it subscribes to `p2p.metrics` and `fabric` state roots and never holds a write capability, so observability can never become an attack surface.

### 13.6 Phased technical sequence

**Phase A (close the loop):** ship `pulse.epoch` binding issuance to the heartbeat (Gap G1), fix the RED Erlay byte-budget test (G2), and turn the no-float rule into a type boundary (G3). All solo-lane, no p2p races. **Phase B (browser core):** Pyodide-WASM core + WebRTC transport adapter, differential gate green, a two-tab interop convergence demo. **Phase C (self-shipping):** `seed publish`, the static loader, service-worker re-serving — the dapp distributes its own source. **Phase D (Molgang knitweb):** promote `knitwebs/chemistry` from stub to a live knit-game wired to `lens` NPCs and `personhood`. **Phase E (hardening + WASM rewrite):** per-container canonical limits, EC-point validation, equivocation witness, and the audited Rust/AssemblyScript core replacing Pyodide. Each phase ends with a `seed`-published, CID-pinned release, so progress is itself woven into the Web it builds.



## 14. Roadmap & milestones to top-10

This chapter converts the architecture and product theses of the preceding chapters into a phased, time-boxed plan. It is honest about what is already production-grade in the `knitweb` tree (full ledger, PoUW verification/dispute/collateral, fabric Web, p2p reconcile/mesh, vBank, crowdfunding, personhood gate) and what remains adapter-only or stubbed (epoch-bound issuance, the `knitwebs/chemistry` plugin Molgang needs, the agentic LENS player loop, the MONITOR-as-product surface). The plan is organised into four phases — **P0 Foundation**, **P1 Serverless MVP**, **P2 Agentic + Economy**, **P3 Scale & top-10 push** — each with entry/exit criteria, owning components, KPIs, and dependencies. A dedicated critical-path section names the single chain of work that, if it slips, slips everything.

The strategic backdrop is that the bar for a "top-10" P2P dapp in 2026 is set by what users already experience on chains with rich indexing and account abstraction. [DappRadar's industry reports](https://dappradar.com/blog) consistently rank dapps by Unique Active Wallets (UAW) and on-chain volume, and the leading games sustain tens of thousands of daily UAW. Our differentiator is not raw throughput but the combination no incumbent has: a **serverless, self-distributing** dapp whose **agentic LLM NPCs are first-class economic actors** minting demand-gated PLS for verified-useful-work. The roadmap is therefore organised to land that unique combination early (P1–P2) rather than chasing throughput parity we do not need.

### 14.1 Phase overview and time-boxing

| Phase | Window (2026–2027) | Theme | Exit gate (one-line) |
|---|---|---|---|
| **P0** | Jun–Aug 2026 (Q3'26) | Foundation: close activity→money loop | Epoch-bound mint live; CID-stability sign-off; Erlay throttle green |
| **P1** | Sep–Dec 2026 (Q4'26) | Serverless MVP | Molgang runs entirely on the Web with no central server; first external peers knit chemistry nodes |
| **P2** | Jan–Apr 2027 (Q1–Q2'27) | Agentic + economy | LENS NPCs play as PoUW workers; PLS earned/spent in-game; vBank treasury governs emission |
| **P3** | May–Oct 2027 (Q3–Q4'27) | Scale & top-10 push | 1k+ concurrent peers, public listing, sustained DAU growth into the DappRadar/Web3 game rankings |

The phases are deliberately back-loaded on novelty: P0 and P1 are mostly *closing* and *wiring* work on subsystems already at maturity 4–5, which de-risks the genuinely new P2/P3 work (agentic economy, scale) by giving it a hardened substrate to stand on.

### 14.2 P0 — Foundation (Q3 2026)

P0 finishes the load-bearing seams identified in the architecture state-of-union. Nothing here is user-visible, but every later KPI depends on it.

**Milestones**

- **P0-M1 — Epoch-bound issuance (R1).** Bind `EmissionPolicy` (`token/mint.py`) to the `Beat`/`Pulse` heartbeat (`core/pulse.py`): carry an `epoch_mint_cap` on the Beat, gate `Treasury.reward_verified_work` to per-epoch escrowed demand, settle mints atomically at the epoch boundary. **Owner: PULSE + token.** This is the single highest-leverage increment — it turns the heartbeat from a decorative clock into the monetary governor and unblocks every downstream economic story.
- **P0-M2 — Erlay live byte-budget throttle green (R2).** Triage and fix `test_p2p_erlay_reconcile.py::test_live_byte_budget_throttles_a_hammering_peer` against the `ServeBudget`/`_recon_budget` debit path. **Owner: PULSE (p2p), review-not-race with the molgang session.**
- **P0-M3 — Float type boundary (R3) + canonical hardening (R4).** Make "no floats near canonical" a type boundary at the interpret→fabric seam; add `MAX_STRING_LEN`/`MAX_ARRAY_LEN` to `canonical._decode` and `crypto.validate_pubkey()`. **Owner: PULSE core + LENS.**
- **P0-M4 — CID-stability sign-off.** The hard cross-session gate from the coordination plan: molgang's `chemistry` knitweb may not land schema until pulse signs off that record CIDs are byte-stable across the relay hop. **Owner: PULSE, blocking molgang.**

**Exit KPIs (P0):** test suite 1184/1184 green (currently 1183/1; the RED Erlay test is the gate); epoch mint reconciles to escrowed demand within ±0 PLS over a 100-epoch simulation; zero float reaches `canonical.encode` under fuzz.

### 14.3 P1 — Serverless MVP (Q4 2026)

P1 migrates Molgang off PHP/5mart.ml and onto the Web as a self-distributing dapp. The ~470-node multilingual explorer graph becomes the seed corpus for the `knitwebs/chemistry` plugin (today a stub `__init__.py`).

**Milestones**

- **P1-M1 — `knitwebs/chemistry` plugin.** Promote the stub to an active L5 knitweb: chemistry concepts modelled as fabric `Edge`/node records, weave + traverse over `Web`, contribution/curation framed as PoUW jobs. **Owner: MOLGANG + new module, depends on P0-M4.**
- **P1-M2 — Self-hosting distribution.** The dapp ships its own source as content-addressed records: a peer that joins the Web can fetch the client bytes by CID and verify the author signature — no app store, no central origin. **Owner: MOLGANG + PULSE fabric.**
- **P1-M3 — Seed migration.** Import the 470-node explorer graph as signed chemistry records; preserve multilingual labels as metadata (integer-only after R3). **Owner: MOLGANG.**
- **P1-M4 — MONITOR public surface.** Promote `knitweb-monitor` from internal observability to a player-facing health view (peers, knit volume, mesh health, web state-root convergence). **Owner: MONITOR.**

**Exit KPIs (P1):** Molgang runs with **zero central server** (verified by killing 5mart.ml and confirming a fresh peer can bootstrap, fetch client, and knit a node); ≥10 external peers; ≥1,000 chemistry nodes on-fabric; convergent `web_state_root` across all peers; first non-team contributor earns PLS.

### 14.4 P2 — Agentic + economy (Q1–Q2 2027)

P2 lands the differentiator: agentic LLM NPCs as first-class players, and a real in-game PLS economy governed by vBank.

**Milestones**

- **P2-M1 — LENS agentic player loop.** The interpret/LENS lobe (`retrieve` + gated `distill`) drives NPCs that play Molgang: they retrieve from the converged Web, distill candidate chemistry relations, and submit them as SPLIT-verified PoUW jobs. Agentic NPCs earn PLS exactly as human players do — same `quorum.tally` BFT verification, same dispute window. Provenance-cited answers are non-negotiable (every NPC claim carries its `DistillManifest`). **Owner: LENS, depends on P0-M1 + P1-M1.**
- **P2-M2 — In-game economy.** PLS earned for verified contributions is spendable in-game (unlock concepts, sponsor bounties via crowdfunding knitweb). **Owner: MOLGANG + token + crowdfunding.**
- **P2-M3 — vBank governance of emission.** vBank polls/ranked/liquid voting govern the `epoch_mint_cap` and treasury policy from P0-M1. **Owner: VBANK.**
- **P2-M4 — Personhood gate live.** `personhood.gate.require_personhood` gates votes/pledges and human-vs-agent labelling so Sybil farms cannot drain emission. **Owner: PULSE personhood.**

**Exit KPIs (P2):** ≥30% of knit volume produced or verified by agentic NPCs; ≥100 DAU/UAW; D7 retention ≥20%; first vBank proposal that *changes* an emission parameter passes and takes effect at the next epoch; zero successful Sybil mint (audited).

### 14.5 P3 — Scale & top-10 push (Q3–Q4 2027)

P3 hardens for scale and runs the growth push toward a ranking position. Top-10 in a credible category (P2P / Web3 social-knowledge games) is the explicit goal; [DappRadar's ranking methodology](https://dappradar.com/rankings) weights UAW and volume, so the KPIs below target those metrics directly.

**Milestones**

- **P3-M1 — Mesh scale.** Erlay reconcile + anti-entropy validated at 1k+ concurrent peers; per-container budgets (R4) prove out against adversarial frames. **Owner: PULSE.**
- **P3-M2 — Equivocation witness (R5).** Structural `fork_height`/`equivocation_witness` on `Fiber`, gossiped fork proofs. **Owner: PULSE ledger + p2p.**
- **P3-M3 — Growth push.** Public listing, the launch-kit (target devs Joost Hoeks + Fillslava), tournaments with PLS prize pools funded via crowdfunding knitweb. **Owner: MOLGANG + VBANK.**

**Exit KPIs (P3):** 1,000+ concurrent peers sustained; 1,000+ weekly UAW; agentic activity ≥40% of knit volume; D30 retention ≥10%; a top-10 placement in the target category on a public tracker.

### 14.6 Component-to-phase map

| Component | P0 | P1 | P2 | P3 |
|---|---|---|---|---|
| **PULSE** (core/p2p/fabric) | Epoch mint, Erlay fix, canonical hardening, CID sign-off | Self-host distribution, fabric ingest | Personhood gate live | Mesh scale, equivocation witness |
| **LENS** (interpret) | Float type boundary | — | Agentic NPC player loop | Distill at scale |
| **MONITOR** | — | Public health surface | Agent/human activity split | Scale dashboards, alerting |
| **VBANK** | — | — | Emission governance | Tournaments, treasury at scale |
| **MOLGANG** (knitwebs/chemistry) | — | Plugin + seed migration | In-game economy | Growth/listing |
| **New modules** | epoch_mint_cap | chemistry plugin | NPC harness | fork-proof gossip |

### 14.7 Dependencies and critical path

The dependency graph has one dominant chain. **P0-M1 (epoch-bound issuance)** is the root: without it, "useful work" mints PLS but the heartbeat exerts no control over supply, so neither the P2 in-game economy nor vBank emission governance can be trusted. **P0-M4 (CID-stability sign-off)** is the second root — it is the hard gate from the coordination plan that blocks molgang's `chemistry` schema from landing; the molgang #32 work explicitly may not merge before this sign-off.

The critical path is therefore:

> **P0-M1 (epoch mint) + P0-M4 (CID sign-off) → P1-M1 (chemistry plugin) → P2-M1 (LENS agentic players) → P2-M2 (in-game economy) → P3-M3 (growth push).**

Off-critical-path but enabling: P0-M2 (Erlay fix) and P0-M3 (float/canonical hardening) gate *quality* and the test exit, and P3-M1 (mesh scale) gates the final KPI, but they do not block the central activity→money→agentic loop. The single biggest schedule risk is P2-M1: the agentic NPC loop is the most novel work and the only milestone with no existing maturity-4+ substrate to lean on. It is sequenced after a fully hardened P0/P1 precisely so that, when we build it, the Web, the ledger, the PoUW verification, and the mint are all already proven — leaving the genuinely new risk (agent reasoning quality and its economic incentives) as the only variable in play.

### 14.8 KPI dashboard (targets by phase)

| KPI | P0 | P1 | P2 | P3 |
|---|---|---|---|---|
| Concurrent peers | — | ≥10 | ≥100 | ≥1,000 |
| DAU / UAW | — | — | ≥100 | ≥1,000/wk |
| D7 / D30 retention | — | — | D7 ≥20% | D30 ≥10% |
| Knit volume (records/day) | — | seed 1k | growing | high |
| Agentic share of knits | — | — | ≥30% | ≥40% |
| Test suite green | 1184/1184 | maintained | maintained | maintained |

These targets are deliberately conservative through P2 and ambitious only at P3, where the growth push concentrates. The honest read is that P0–P1 is *de-risking already-built infrastructure*, P2 is *the bet*, and P3 is *the campaign*. Hitting top-10 is a function of executing P2-M1 well; everything before it exists to make that bet safe, and everything after it exists to scale a bet that has already paid off.



## 15. Risks, Mitigations & Conclusion

No development plan is honest unless it names the ways it can fail. Molgang-on-knitweb is an ambitious bet: a top-10-calibre dapp that lives entirely on a peer-to-peer Web, ships its own source, and treats agentic LLMs as first-class players. Each of those properties is also a risk surface. This chapter enumerates the seven material risk classes — browser-P2P maturity, bootstrap/cold-start, Sybil and economic attacks, LLM hallucination into canonical state, regulatory exposure, team/scope discipline, and the small set of genuinely unsolved problems — pairs each with a concrete, already-architected mitigation, and is deliberately candid about what we cannot yet fully solve. It closes with the affirmative case for why this is worth building.

### 15.1 Browser-P2P maturity

**The risk.** Knitweb Pulse is a pure-Python, stdlib-only P2P engine: Erlay set-reconciliation, a gossipsub mesh, Kademlia DHT, and anti-entropy backstop all run today as native processes. But a top-10 *dapp* needs reach into the browser, and browser-to-browser P2P remains the weakest link in the entire decentralized stack. WebRTC connectivity is "riddled with challenges related to routers, NAT layers, VPNs, and firewalls," typically forced to fall back on signaling, STUN, and TURN servers — infrastructure that quietly re-centralizes a system whose whole premise is the opposite ([libp2p WebRTC docs](https://docs.libp2p.io/concepts/transports/webrtc/)). Browsers further constrain us: TLS-certificate requirements signed by trusted CAs are why WebSocket "never saw widespread adoption in the libp2p network," and Firefox and Safari still lag on parts of the `RTCDTLSTransport` API ([libp2p blog](https://blog.libp2p.io/libp2p-webrtc-browser-to-server/)). JavaScript's lack of full threading also caps how much reconcile/verify work a browser peer can shoulder.

**The mitigation.** We do *not* require browser peers to be full validators. The architecture already separates the carrier seam (`p2p/base_node.py`) from peer roles, which lets us run a **tiered topology**: durable native "spine" nodes (the existing AsyncioP2PNode stack) carry reconcile, mesh relay, and PoUW verification; browser participants run a thin **WebRTC-direct + browser-to-server** transport (the path libp2p has now stabilized "in a decentralized way across a broad spectrum of browsers", [libp2p WebRTC](https://libp2p.io/docs/webrtc/)) and act as light clients that author records, fetch via the lazy inv→getdata→inv-data flow, and verify CIDs locally. Because identity is canonical-CBOR bytes (CIDv1, dag-cbor/sha2-256), a light client can *cryptographically check* everything it receives without trusting the spine node that served it. Signaling is the one residual centralization point; we treat it as a commodity relay (multiple independent, swappable signaling endpoints, with bootstrap addresses themselves gossiped) rather than a trusted authority. Honest caveat: until browser-to-browser WebRTC with decentralized signaling is universally robust, a small set of always-on spine nodes is load-bearing. We own that and budget for it.

### 15.2 Bootstrap and cold-start

**The risk.** A content-addressed Web and a demand-gated currency both have the same chicken-and-egg problem. PLS is minted *only* as reward for verified-useful-work, and useful work only exists if there are consumers escrowing pulses for jobs. Molgang's value compounds with graph density — today's ~470-node multilingual explorer is charming but thin. With too few peers, gossip has nothing to reconcile, the PoUW committee (which needs a BFT `⌊2n/3⌋+1` quorum) cannot be safely sampled, and personhood-gated governance has no quorum either. An empty Web is indistinguishable from a dead one.

**The mitigation.** Three levers, all already in the design. First, **genesis content**: the existing molgang chemistry graph is migrated in as signed, content-addressed nodes, so the Web is non-empty on day one and the chemistry knitweb (currently an `__init__.py` stub at L5) is the first populated domain plugin. Second, **agentic NPCs as bootstrap workers**: the interpret/LENS lobe runs deterministic retrieve + gated distill jobs that emit signed relation bundles which *themselves become SPLIT-verified PoUW jobs* (`interpret/job.py`). LLM NPCs therefore generate genuine, verifiable useful work from the first session, seeding both graph growth and the mint loop without a human crowd. Third, **epoch-gated emission as a cold-start governor**: roadmap item R1 (epoch-bound issuance, the highest-leverage open seam per the architecture state) lets us bind a per-epoch mint cap to the Pulse heartbeat, so early emission is generous-but-bounded and cannot be drained before the network thickens. Caveat: NPC-seeded work must be *demand-gated*, not free minting — see 15.4.

### 15.3 Sybil and economic attacks

**The risk.** Any open mint is a target. A Sybil attacker spins up thousands of identities to capture PoUW rewards, dominate vBank governance (poll/ranked/liquid voting), or out-vote honest curators on what counts as "useful." Economic attacks include grinding low-value distill jobs for reward, collusive committees rubber-stamping a confederate's bad work, and griefing via spam transfers (the receiver-nonce gap, gap G6, means an account can be flooded with incoming Knits with no L0–L1 rate limit).

**The mitigation.** The Sybil gate is a first-class subsystem, not an afterthought: `personhood` enforces **one-person-one-scope** via nullifiers with epoch-pinned non-revocation (`personhood/gate.py`), pairwise DIDs, and revocable proofs with no PII on-fabric — an ICH-style privacy posture where the presentation verifier is a pluggable VC/ZK backend. vBank and crowdfunding both call `require_personhood` before accepting a vote or pledge. On the economic side, PoUW is defended in depth: committee selection excludes the worker, sample size is computed (`pouw/sampling.py`), verdicts aggregate to a BFT quorum, and crucially every job carries **staked collateral** with a dispute window whose `release_delay > dispute_window` safety invariant is enforced in the constructor (`pouw/dispute.py`) — collusion that produces a `DETECTED_FAULT` is *slashed*, making attacks negative-EV. Roadmap R4 (EC-point validation + per-container canonical length budgets) and R6 (receiver-side flow control) close the spam vectors. Honest caveat: personhood ultimately rests on the strength of the chosen proof-of-personhood backend; if that backend is weak, every gate above it weakens proportionally. Choosing and hardening that backend is a top-tier dependency.

### 15.4 LLM hallucination into canonical state

**The risk.** This is the most novel and arguably the hardest risk, because it is intrinsic to the "agentic LLMs as first-class players" thesis. LLMs "are prone to hallucinating outputs and fabricating execution traces," and in 2026 hallucination rates still range widely — best systems near 0.7% but many real configurations far higher ([Zylos Research](https://zylos.ai/research/2026-01-27-llm-hallucination-detection-mitigation)). The agentic case is worse: in "multi-agent systems sharing memory, a single hallucinated entry can spread to every downstream agent that queries it" ([SQ Magazine](https://sqmagazine.co.uk/llm-hallucination-statistics/)). A shared content-addressed knowledge graph *is* shared agent memory. If an NPC distills a plausible-but-false chemistry relation and it is canonicalized, the falsehood is now permanently, immutably addressable — and worse, it can poison every later distill and every cited LENS answer.

**The mitigation.** The architecture refuses to let an LLM write directly to canonical state. Distillation is **gated and verified, never trusted**: `interpret/distill._gate_relation` requires attestation plus acyclic provenance for every relation before emission, and the resulting `DistillManifest` re-enters the world as a SPLIT-verified PoUW job that demands deterministic re-check *and* a closed dispute window *and* no upheld dispute before it settles. This is the cryptographic version of the "tool receipts, not just confidence" posture emerging in the literature — distinguishing genuine tool-grounded claims from confabulation by *re-execution*, not by asking the model how sure it is ([arxiv 2603.10060](https://arxiv.org/pdf/2603.10060)). LENS answers are provenance-cited by construction, so a human (or another agent) can trace any claim back to its grounding CIDs. The honest, unsolved core: deterministic re-check catches *fabricated or non-reproducible* work, but it does not by itself adjudicate *subtle factual wrongness* in a reproducible-but-incorrect chemistry relation. That last mile depends on the human-plus-committee curation economy doing its job — which is precisely why the dispute/slash incentives in 15.3 and the personhood gate matter so much here. We are not claiming to have solved truth; we are claiming to have made *cheating* expensive and *every claim* auditable.

### 15.5 Regulatory exposure

**The risk.** PLS is money. A token minted as reward, traded peer-to-peer, funding crowdfunding campaigns and a treasury, raises questions across securities, money-transmission, and AML/KYC regimes that vary sharply by jurisdiction — and a serverless, self-distributing dapp has no obvious legal entity to absorb that exposure. Proof-of-personhood, if mishandled, also collides with privacy law (GDPR and successors).

**The mitigation.** Two design choices reduce, though they cannot erase, this exposure. First, PLS is **utility-shaped, not investment-shaped**: it is demand-gated reward for verified work, with no premine and no admin mint (`token/mint.py` has no privileged issuance path), which is the strongest available posture against a securities characterization. Second, personhood is built **privacy-first**: revocable proofs, pairwise DIDs, nullifiers, and *no PII on-fabric* mean the system is designed to be compliant-by-construction rather than retrofitted. Caveat, stated plainly: regulatory treatment of decentralized tokens remains unsettled in June 2026, and "the protocol has no central operator" is a technical fact, not a legal shield. Launch sequencing should favour jurisdictions with clear digital-asset frameworks, keep the treasury and crowdfunding features behind explicit governance gates, and obtain qualified counsel before PLS has secondary-market liquidity.

### 15.6 Team and scope discipline

**The risk.** The stack is large — ~20.7k LOC across seven layers plus cross-cutting personhood, interpret, and anchor subsystems — and the plan is being executed by a small, coordinated set of sessions (the pulse/molgang two-track split). The classic failure mode is scope sprawl: chasing PQ crypto, four domain knitwebs, and a perfect browser stack simultaneously, and shipping none. There is also a live coordination hazard: the hard gate that molgang's CID-stability-dependent work must not land before pulse signs off, and a currently-RED Erlay interop test (G2) sitting on the parallel session's surface.

**The mitigation.** The continuation roadmap is explicitly **ranked and laned** (`solo` vs `review-not-race`), with R1 (epoch-bound issuance) named as the single highest-leverage increment because it is the only one that closes the activity→money loop the whole stack exists to express. We ship the closing of that loop first; chemistry-knitweb population second; browser reach third; and treat PQ migration, the other three domain stubs, and distributed fork proofs (R5) as deliberately deferred. The two-session coordination is governed by pinned issues and a documented hard gate, which keeps the p2p/fabric surface review-only for the molgang track. Discipline here is a feature, not a constraint.

### 15.7 The honestly-unsolved problems

To be candid, three problems are not yet solved and should not be claimed as such: (1) **truth adjudication** — re-execution proves reproducibility, not correctness, so subtly-wrong-but-reproducible knowledge ultimately leans on the curation economy; (2) **browser-native full participation** without any spine dependency, which awaits further maturation of decentralized WebRTC signaling; and (3) **distributed fork/equivocation proofs** — the ledger today is fork-aware only locally (G5), with the gossipable witness structure still on the roadmap. Naming these is part of being credible about the rest.

### 15.8 Conclusion — why this can be a genuinely unique top-10 dapp

Strip away the hype that surrounds most "AI x crypto" projects and ask what would actually be *new*. Molgang-on-knitweb is unique on three axes at once, and it is the *conjunction* that has no peer. It is **fully serverless and self-distributing**: the dapp ships its own source over the same content-addressed Web that carries its data, so there is no app store, no API host, no kill switch. Its money is **earned, not printed**: PLS exists only as the demand-gated, epoch-governed reward for work that a BFT committee re-executed and a dispute window cleared — a closed activity→money loop, not a speculative emission curve. And its **agents are first-class but never trusted**: LLM NPCs play, contribute, and reason, yet nothing they assert touches canonical state until it survives gated distillation, deterministic re-verification, personhood-gated curation, and economically-staked dispute. Where the industry's hardest current problem is that "a single hallucinated entry can spread to every downstream agent," molgang answers with a cryptographic discipline that makes every claim auditable and every cheat expensive. That is a defensible, demonstrable, and genuinely differentiated position — a knowledge game whose graph is its ledger, whose players include reasoning machines, and whose entire existence is independent of any server we, or anyone, could be compelled to switch off. The hard problems are real and we have named them. The opportunity is that solving even most of them yields something the top-10 does not currently contain.
