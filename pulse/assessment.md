# Knitweb/Pulse: A Post-Consistency Assessment Against the Crypto Corpus and the Project's Own Papers

*Build: `fix/consistency-pass-02` (PR #49, post-consistency) · Date: 2026-06-18 · ~5,536 src LOC / 50 modules · Runtime dependency: `cryptography` only*

## Abstract

This paper assesses the `knitweb/pulse` repository at its post-consistency checkout (`fix/consistency-pass-02`, PR #49) along four axes: fidelity to the ~190-repo crypto corpus (via its in-repo distillation), fidelity to the project's own papers, Python optimality and script minimality, and architecture/differentiation. Method is a read-only, `file:line`-precise pass over `src/knitweb/`, cross-referenced against `docs/` and validated by a green test suite. Two headline verdicts: the Python is **mostly optimal** — clean, strictly typed, single-dependency, with the only material non-optimality being ~100 LOC of boilerplate duplicated across four domain looms — and it **does work with a minimal set of scripts** (five distinct runnable entry points, one operational pipeline, no redundancy). Against the corpus, the ledger/encoding/feed substrate and settlement timing meet or exceed shipped DePIN practice, while the proof model's verification economics trail. Against the papers, all seven primitives and the Fiber-vs-PLS reconciliation are realized; the aspirational set is bounded and mostly self-flagged.

## Methodology & scope

This assessment is deliberate about what was and was not available on the analysis machine, so the reader can weight each claim accordingly:

- **The ~190-repo corpus is NOT on this machine.** It lives off-machine (on EDS2). The comparison therefore uses the in-repo distilled study, `docs/CRYPTO_CORPUS_STUDY.md`, as the corpus proxy — it is treated as the authoritative distillation of the corpus's top actions, exemplar networks, and backlog markers, and every corpus claim below is anchored to a line in that file.
- **Code was analyzed at the post-consistency branch** `fix/consistency-pass-02` (PR #49), including the `pouw` dispute window shipped via #32 (commit `fa0e511`). All `src/` paths are absolute under `/Users/develuse/repo/pulse`.
- **Tests = 273 green.** The full suite (property + interop + looms) collects 273 test functions across 37 files and is green per the consistency pass. (Section 2 cites an earlier intermediate count of 205 for the primitives sub-run; the authoritative full-suite figure is 273.)
- **Grading vocabulary** is shared across sections: **Implemented/Realized/Shipped** = code exists *and* sits on the proof/settlement path and is tested; **Partial** = a correct standalone primitive exists but a stated guarantee is missing or it is not wired into `job.verify`/`escrow`/`dispute`/`mint`; **Missing/Aspirational** = no code, or explicitly deferred in `ROADMAP.md`/docstrings.

A note carried throughout: where a module is *present and correct* but *not on the live path*, this paper calls it out as a gap between "module present" and "proof model uses it," and distinguishes aspirational claims from shipped ones rather than blending them.

## 1. Knitweb vs the crypto corpus

This section maps every concrete action and lesson in `docs/CRYPTO_CORPUS_STUDY.md` (the authoritative distillation of the ~190-repo corpus, which itself lives off-machine on EDS2) onto the current `fix/consistency-pass-02` checkout (PR #49, post-consistency; `pouw` dispute window shipped via #32, commit `fa0e511`). Status is graded **implemented** (code exists *and* sits on the proof/settlement path), **partial** (code exists as a standalone, correct primitive but is not yet wired into `job.verify`/`escrow`/`dispute`/`mint`, or is a documented placeholder), or **missing** (no code). Evidence is `file:line`-precise against `src/knitweb/`.

### 1.1 The corpus's five "top actions" (`CRYPTO_CORPUS_STUDY.md` §1)

| # | Corpus top action | Status | Evidence |
|---|---|---|---|
| 1 | **Digest determinism / tolerance digests** — never exact-match raw floats; quantize + hash integers, attest when unpinnable | **Partial** | The primitive is fully built and correct: `pouw/digest.py:35` `quantize(value, eps)` snaps to a float-free integer bucket via deterministic round-half-up (`math.floor(value / eps + 0.5)`), and `pouw/digest.py:53-61` `tolerance_digest` hashes the *integer* buckets through canonical CBOR so "two outputs whose elements agree to within `eps` produce the same digest". **But it is not wired into settlement.** `pouw/job.py:69` still gates on exact byte-equality — `if _bc.bundle_digest(proof.bytecode) != proof.digest` and `pouw/job.py:73` `if recompiled != proof.bytecode` — and `digest.py` is imported by nothing on the verify path (`grep` for `tolerance_digest`/`quantize` in `job.py`/`escrow.py`/`mint.py` is empty). For the current useful-work job (deterministic synaptic-compile, `job.py:9-12`) exact-match is *correct*; the existential GPU-noise risk is solved as a library but unconnected to any float-output job. |
| 2 | **Salt every challenge + commit-before-sample** — worker commits a digest over the full output at submit; verifier samples fresh-salted random indices against that fixed commitment | **Partial** | Fully implemented as a standalone protocol in `pouw/challenge.py`: `commit(blocks)` builds a domain-separated Merkle root over the whole output (`challenge.py:115-120`, leaf tag `\x00` / node tag `\x01` at `:48-49` for second-preimage safety), `new_salt` is verifier-chosen *after* the commit (`:110-112`), `sample_indices` derives unpredictable-yet-reproducible indices from the salt via a SHA-256 counter stream (`:123-144`), and `verify_response` rejects reordered indices, stale/precomputed salted digests, and work-swap via Merkle membership (`:189-197`). This defeats precompute + retroactive swap exactly as the corpus prescribes (Filecoin beacon, Arweave SPoRA, Livepeer salts). **But, like #1, it is not invoked by `job.verify`, `escrow.settle_on_verify`, or `dispute`** — the live verify path is the single deterministic re-execution in `job.py:71-75`, with no commitment/salt round. |
| 3 | **Release delay > dispute window; slash pending withdrawals** — `slashable_until = submit_beat + dispute_window`; no withdraw before re-exec | **Implemented (shipped via #32)** | `pouw/dispute.py` is the settlement-timing layer and is the one top-action fully on the path. The safety invariant is enforced in the constructor — `dispute.py:84` `if release_delay <= dispute_window: raise ValueError(...)`, so the release beat lies strictly after the window closes ("EigenLayer's 'withdrawal delay must exceed the dispute window', in miniature", `:20-22`). `slashable_until = submit_beat + dispute_window` (`:131-133`); `release` refuses while pending and `beat < submit_beat + release_delay` (`:179-180`, "within dispute window — escrow still locked"); a detected-mismatch `dispute` within `[submit_beat, slashable_until]` flips the submission to `slashed`, burns collateral and refunds escrow (`:158-162`). Collateral is staked *alongside* escrow and slashed on detection, so "fraud is never net-profitable" (`:14-16, 96`). Defaults `DEFAULT_DISPUTE_WINDOW=10`, `DEFAULT_RELEASE_DELAY=11` (`:43-44`). |
| 4 | **k-of-n verifier quorum** (~55% confirm, tolerate ~33% adversary) | **Missing** | No quorum code exists anywhere in `src/` (`grep -rln "quorum"` matches only the *prose* TODO in `dispute.py`). A dispute is explicitly "a single detected mismatch" and the quorum is named as the next increment: `dispute.py:27-28` — "Declared-vs-detected fault asymmetry and the k-of-n verifier quorum are the next increment (`pouw/verifier-quorum`)". The verify path trusts one verifier (`job.verify`, `escrow.settle_on_verify` call `verify` once, `escrow.py:33`). This matches the open backlog item `CRYPTO_CORPUS_STUDY.md:139`. |
| 5 | **Streaming / probabilistic escrow + declared-fault asymmetry** — settle ~1/N jobs; self-declared incapacity = small fee; size collateral ≥ one window's payout-at-risk | **Missing** | No winning-ticket/streaming/declared-fault code exists (`grep -rln "winning.ticket\|declared.fault\|probabilistic"` matches only `dispute.py` prose). `escrow.settle_on_verify` settles *every* verified job in full (`escrow.py:33-36`); there is no random ~1/N redemption and no cheap self-declared-fault path distinct from the full slash. Collateral is a free integer field with no minimum-sizing rule tying it to payout-at-risk (`dispute.py:101-127` accepts any `collateral >= 0`). |

**Net on the top-5:** one of five is fully shipped on-path (#3, dispute window + slash-pending). Two (#1 tolerance digests, #2 salt/commit-before-sample) exist as correct, tested standalone primitives but are **not connected to the settlement path** — a real gap between "module present" and "proof model uses it". Two (#4 quorum, #5 probabilistic/declared-fault) are entirely absent and remain the open Phase-4 backlog (`CRYPTO_CORPUS_STUDY.md:139`).

### 1.2 Beyond §1 — the encoding / ledger / feed actions also tracked by the corpus

These are not in the §1 top-5 but are concrete corpus actions; the post-consistency build is strongest here.

- **Canonical strict decode (ADR-027 / RLP `ErrCanonInt` baseline)** — **Implemented.** `core/canonical.py:202-207` `decode` rejects trailing bytes; `read_len` (`:124-146`) rejects non-minimal heads (`"non-minimal integer: value fits a shorter head"`, `:145`); map keys are checked for strict ascending encoded-byte order, catching both unsorted maps and duplicate keys in one pass (`:183-186`); floats are forbidden in both encode (`:100-103`) and structurally on decode (no major-7 float branch). This is the `[x]` backlog item `CRYPTO_CORPUS_STUDY.md:135`.
- **Exact-match nonce + chainID-in-signature anti-replay** — **Implemented.** `ledger/loom.py:71-74` enforces exact `knit.from_nonce != prev.nonce → raise`; the `network` id is inside the signed record (`ledger/knit.py:47-56`) and refused on mismatch at `loom.py:49-50`. **Partial gaps vs the action text:** no **EIP-2681 overflow guard** (`grep` for `overflow`/`2681` empty — `nonce + 1` at `loom.py:80` is unbounded), and **no reorg/nonce-revert** (the Braid is strictly append-only with a spent-Knit guard, `ledger/braid.py:51-56`; there is no reorg path, so "revert nonce on reorg" is N/A today rather than done). Mempool pending-vs-queued separation is also absent (no mempool module).
- **Versioned address/key scheme byte for post-quantum soft-fork** — **Implemented (ahead of the backlog).** The corpus lists this as an open `[ ]` item (`CRYPTO_CORPUS_STUDY.md:136`), but `core/crypto.py:57-64` already ships `ADDRESS_HRP="pls1"`, `SCHEME_SECP256K1_ECDSA=0`, and a `scheme || sha256²(pub)[:20]` payload; `address()` rejects unknown schemes (`:190-191`) and `decode_address` returns the scheme even when unknown so callers can reject deliberately (`:201-206`). The actual SPHINCS+/ML-DSA algorithm is not implemented (only secp256k1 is blessed), but the soft-fork hook the action asked for exists.
- **Phase-3 Hypercore-style signed-tree feed + fork counter + `check_conflict`** — **Implemented.** `fabric/feed.py` signs the *head* `{ns, feed, root, length, fork}` not each entry (`:58-62, 131-135`), with an explicit `fork` counter bumped on legitimate truncate (`:145-156`); `check_conflict` proves equivocation as two valid sigs at same `(length, fork)` with different roots (`:186-199`), and `check_prefix_conflict` catches a same-fork prefix rewrite (`:202-224`). `p2p/node.py` quarantines on conflict (`:176-198`, `frozen_feeds`) and runs the two-party Knit handshake over canonical-CBOR frames (`p2p/wire.py:109-134`, strict `canonical.decode`). This is the `[x]` backlog item `CRYPTO_CORPUS_STUDY.md:140`.
- **Partial-range Merkle proofs + DHT/pubsub backend** — **Missing (by design, deferred).** `p2p/node.py:170-171` explicitly rejects partial proofs (`"partial Merkle proofs are not supported by this MVP"`); discovery is PEX, not Kademlia (`p2p/discovery.py:1-13`). Open `[ ]` backlog (`CRYPTO_CORPUS_STUDY.md:141`).

### 1.3 Comparison against the corpus exemplar networks

| Exemplar (corpus) | What they prove / pay | Lesson for Knitweb | Knitweb's current status (file evidence) |
|---|---|---|---|
| **Akash** (`x/escrow`, `x/market`) | No compute proof; per-block streaming escrow on elapsed time | Stream escrow **but gate each tranche on a digest match**, not time | **Trails.** `escrow.py:20-36` gates settlement on a digest/verify match (`if not verify(job, proof): return False`) — so it *does* avoid Akash's no-verification hole — but settlement is **all-or-nothing per job**, not streamed per-tranche. No streaming escrow exists. |
| **Filecoin** (`wdpost_run.go`, `miner.go`) | Beacon-seeded sampled challenges; **declared faults cheap, detected faults expensive** | Copy the **declared-vs-detected asymmetry** | **Beacon/salt lesson met (off-path), asymmetry missing.** `challenge.py:123-144` is precisely beacon-seeded sampling (salt→indices), but unwired. The declared-vs-detected asymmetry is explicitly unbuilt: `dispute.py:28` — only a single "detected mismatch", "Declared-vs-detected fault asymmetry … the next increment". |
| **Livepeer** (`pm/recipient.go`, `verification`) | Sampled recompute + **winning-ticket** probabilistic micropayments; redemption delay | Adopt winning-ticket escrow (settle ~1/N) + a redemption/dispute window | **Mixed.** The redemption/dispute-window half is **done**: `dispute.py:84` (`release_delay > dispute_window`), `:166-185` release-after-window. The **winning-ticket probabilistic settlement is missing** — `escrow.py:33-36` settles every job in full; no `H(sig,rand) < winProb` sampling anywhere. |
| **EigenLayer / EigenDA** (`AllocationManager.sol`, `aggregation.go`) | 14-day withdrawal delay > dispute window; slashing reaches **queued withdrawals**; k-of-n BLS quorum + KZG length-proof | Release delay must strictly exceed dispute window; slash pending; **k-of-n quorum** | **Half exceeds, half missing.** The withdrawal-delay-exceeds-dispute-window invariant is *enforced in code* (`dispute.py:84`, constructor raises otherwise) and slashing reaches the still-pending submission's collateral (`:158-162`) — this meets the EigenLayer lesson the corpus singled out. The **k-of-n verifier quorum is entirely missing** (`dispute.py:27-28` names it as future `pouw/verifier-quorum`); verdicts trust one verifier. |
| **Chutes / Targon** (`cfsv_wrapper.py`, `validator.md`) | Fresh per-challenge **salts** + GraVal device binding; TEE attestation sidesteps GPU non-determinism | **Tolerance/quantized digests** (raw-float equality breaks under GPU non-determinism) or hardware attestation; **salt every challenge** | **Both lessons solved as libraries, neither on-path.** Tolerance digests: `digest.py:35-66` (quantize + integer-bucket hash). Salt-every-challenge: `challenge.py:110-144`. Both exist and are correct, but `job.verify` still uses exact-match `bundle_digest` (`job.py:69,73`) and no attestation/TEE path exists. This is the single biggest "built but unwired" gap. |
| **Arweave** (`ar_poa.erl`) | SPoRA: challenge input unforgeable without the real packed chunk; reward-only, no slash | Borrow "challenge input must be unforgeable without doing the real work"; reward-only is **insufficient** since we hold collateral | **Met where wired.** Unforgeable-without-the-work: `challenge.py:147-148` `_salted_digest(salt ‖ index ‖ block)` forces holding real content computed *after* the salt (off-path, as above). Crucially, Knitweb does **not** repeat Arweave's reward-only mistake: it holds collateral and slashes — `dispute.py:158-162` burns collateral on detected fraud; minting is itself gated and anti-replayed (`token/mint.py:142-143, 167-172`). |

### 1.4 Corpus verdict

Measured against shipped DePIN practice, the post-consistency build is **strongest on the ledger/encoding/feed substrate and on settlement timing, and weakest on the verification economics of the proof model itself.** It *meets or exceeds* the corpus on three fronts: strict canonical decode (`core/canonical.py` matches the ADR-027/RLP baseline the corpus calls the gold standard), the Hypercore-style signed-tree feed with a real equivocation proof (`fabric/feed.py:186-224`), and the EigenLayer-grade invariant that release delay must *strictly* exceed the dispute window with slashing reaching pending collateral — and it enforces that invariant in the constructor (`pouw/dispute.py:84`), which is more rigorous than a configured constant. It even ships the versioned PQ address byte (`core/crypto.py:57-64`) that the corpus still lists as open backlog. It clearly **trails** shipped practice on the proof model's core economics: the two existential float/sampling defenses — tolerance digests (`pouw/digest.py`) and salt + commit-before-sample (`pouw/challenge.py`) — are written, correct, and tested but **not wired into `job.verify`/`escrow`/`dispute`**, which still rely on single-verifier exact-match (`pouw/job.py:69-75`), and there is no k-of-n quorum, no winning-ticket/streaming escrow, and no declared-vs-detected fault asymmetry anywhere in `src/`. In short: the cryptographic and timing scaffolding now matches or beats the corpus, but until the digest/challenge primitives are connected and a verifier quorum lands (the explicit `pouw/verifier-quorum` TODO at `dispute.py:27-28`), Knitweb's *live* proof path remains a deterministic-job special case rather than the GPU-grade, multi-verifier model that Livepeer/EigenLayer/Targon actually run in production.

## 2. Papers vs. implementation (fidelity)

This section maps each major claim in the written papers to the code on `fix/consistency-pass-02`. Status legend: **Realized** = implemented and property-tested; **Partial** = core mechanism present but a stated guarantee/refinement is missing; **Aspirational** = described in a paper but not yet built (or marked pending in `ROADMAP.md`/`PROOF_OF_USEFUL_WORK.md`). The full suite is green (273 tests; the primitives sub-run reported here was 205).

### 2.1 The seven primitives

The concept paper's normative note (`docs/research/08-knitweb.md:13`, restated `:569`) and `CLAUDE.md` fix the seven primitives as `Blob · Fiber · Loom · Knit · Braid · Web · Pulse`. Each exists as a module and matches its described role.

| Paper claim | Where stated | Code status | Evidence |
|---|---|---|---|
| **Blob** = account balance state, integer-only, nonce | 08-knitweb `:573`; CLAUDE.md | **Realized** | `ledger/blob.py:1-56` — int-only `credit`/`debit`/`normalize`, rejects floats/bools (`:21`), overdraft guard (`:53`) |
| **Fiber** = immutable, content-addressed account-state commitment (a `Braid` link), *never itself transferred* | 08-knitweb `:16-18`, `:127`, `:573-579`; SYNAPTIC_WEB `:5-7` | **Realized** | `ledger/fiber.py:22-50` — frozen snapshot `(owner, seq, balances, nonce, prev, knit)`, `cid` over canonical record; no transfer method exists on `Fiber` |
| **Loom** = validation primitive (the only determinism-critical surface) | 08-knitweb `:110`, `:733`; CLAUDE.md | **Realized** | `ledger/loom.py:35-114` — `validate_knit`, dual-sig check, nonce match, overdraft, `conserves_value` |
| **Knit** = two-party dual-signed transfer of a `symbol` balance | 08-knitweb `:18`, `:733`; CLAUDE.md | **Realized** | `ledger/knit.py:33-103` — both `from_sig`/`to_sig` over canonical bytes, `from_nonce` replay-pin |
| **Braid** = a yarn's local append-only history chain | 08-knitweb `:576`, `:737` | **Realized** | `ledger/braid.py:26-80` — seq+1, prev-CID link, spent-Knit double-spend guard (`:51`), full `validate()` |
| **Web** = the woven global graph (content-addressed nodes + typed edges) | 08-knitweb `:489`, `:731`; CLAUDE.md | **Realized** | `fabric/web.py:50-131` — idempotent `weave`, typed `Edge`, deterministic `traverse` |
| **Pulse** = the heartbeat; epochs/beats anchoring state roots | 08-knitweb `:569`, `:734` | **Realized (clock only)** | `core/pulse.py:67-135` — deterministic injected-time epochs, chained `Beat`, `verify_chain`. Note: the doc's own role for Pulse ("drive mint windows") is *not* wired — see 2.3 |
| `Yarn`/`stitch` are **narrative aliases only**, not primitives | 08-knitweb `:14-15` | **Realized (consistent)** | No `yarn.py`/`stitch.py`; identity is a secp256k1 keypair over a `Blob` (`ledger/node.py:21-37`); paper's own crosswalk maps yarn→account, stitch→signed Web item (`:573-577`) |

**Verdict on primitives:** 7/7 exist with names and roles matching the papers. The only gap is that `Pulse` is currently a standalone deterministic clock, not yet the trigger for mint windows it is described as driving.

### 2.2 Token model: Fiber-as-commitment vs PLS-as-pay-token (post-consistency reconciliation)

This is the headline of the consistency pass, and the code now matches the reconciled story.

| Paper claim | Where stated | Code status | Evidence |
|---|---|---|---|
| **PLS is the active pay-token**; "Fiber" is the brand coin / value unit but the `Fiber` *primitive* is never transferred | 08-knitweb `:16-20`; SYNAPTIC_WEB `:5-9`; CLAUDE.md "Tokens" | **Realized** | Transferable value is a `symbol` balance moved by `Knit` (`ledger/knit.py:38`); native symbol `NATIVE = "PLS"` (`token/mint.py:36`); `Fiber` exposes no transfer path |
| **FBR is reserved and not active** | 08-knitweb `:20`, `:735` | **Realized (consistent)** | `FBR` appears nowhere as an active symbol in `src/`; only `"PLS"` is used in the value path |
| **No premine** | 08-knitweb `:567`, `:740`; COLLECTIVE_INTELLIGENCE `:39` | **Realized** | `genesis_fiber` defaults to empty balances, `premine=0` documented (`ledger/fiber.py:52-66`); `Treasury` starts at `total_minted=0` with no admin/ungated mint (`token/mint.py:104-116`) |
| **Bounded, demand-gated mint** (≤ escrow consumed, optional `max_supply`) | PROOF_OF_USEFUL_WORK `:83-91`, `:123-133`; 08-knitweb `:405` | **Realized** | `EmissionPolicy.reward` clamps to escrow and `max_supply` (`token/mint.py:68-76`); `reward_verified_work` gates on `verify` then settles then mints a coinbase (`:117-172`); test `test_token_mint.py` green |
| **Anti-replay on issuance** ("no-infinite-mint" guard) | PROOF_OF_USEFUL_WORK `:52`, `:85` | **Realized** | Rewarded proof digests tracked, duplicate ⇒ `None` (`token/mint.py:142-143`, `:171`); coinbase tagged with issuance CID so Braid's spent-knit guard blocks double-mint (`:174-191`) |
| **Per-Pulse-epoch mint cap** + 1-pulse-per-bundle access payment | SYNAPTIC_WEB `:106`; PROOF_OF_USEFUL_WORK `:132-133`; 08-knitweb `:582` | **Aspirational** | Explicitly deferred to Sprint 3 (`ROADMAP.md:52,83` B9); `core/pulse.py:8-11` docstring states epoch-bound cap is "a future wiring, not yet implemented" |
| **ERC20-like user-issued tokens** ("LoomToken") | `src/knitweb/__init__.py:17` (L6 description) | **Aspirational / stale** | No user-token module exists (`token/` holds only `mint.py`); `ROADMAP.md:99-100` records owner decision *"Maak geen loomtoken"* — the package docstring claim is unbuilt and contradicts the roadmap |

### 2.3 The Synaptic Web USP (the differentiator)

`SYNAPTIC_WEB.md` and `COLLECTIVE_INTELLIGENCE.md` stake the project's USP on the Fiber Synaptic Compiler. Every claimed property of the bytecode is realized and tested.

| Paper claim | Where stated | Code status | Evidence |
|---|---|---|---|
| **Deterministic** bytecode — identical relation sets ⇒ identical bytes (content-addressable) | SYNAPTIC_WEB `:65-67`, `:103-104` | **Realized** | `compile_bundle` sorts dictionary lexicographically + canonical relation order (`synaptic/bytecode.py:135-174`); `test_synaptic.py` proves order-independence |
| **Reversible** — `decode_bundle` reconstructs exact relations | SYNAPTIC_WEB `:68` | **Realized** | `decode_bundle` round-trips with trailing-byte check (`synaptic/bytecode.py:177-215`); round-trip test green |
| **Provenance-bearing** — embeds asset CID + originator; `sign_bundle`/`verify_bundle` so an edge device verifies *before* executing; tampered bundle refused | SYNAPTIC_WEB `:69-71`, `:91-93` | **Realized** | `sign_bundle`/`verify_bundle` via secp256k1 (`synaptic/bytecode.py:227-234`); edge runtime refuses unverified bundles (`edge/runtime.py:1-26`, `EdgeVerifyError`) |
| **Compact** — interning + varints; toy ~58% smaller than `json.dumps` | SYNAPTIC_WEB `:72-75` | **Partial (under-claims, honestly)** | Compression real (LEB128 + interning); but the shipped test asserts only `len(data) < json_size` (`test_synaptic.py:49-54`), not the specific ~58% figure — the strong number is narrative, the weaker proven claim is what ships |
| **Edge bytecode is data, not a general VM** ("relation matrix") | SYNAPTIC_WEB `:54`, `:97-99` | **Realized** | Format is a flat dict+relations table (`synaptic/bytecode.py:40-61`); no opcode/VM execution |
| **Spider resolves OT asset → compiles → serves as useful work, verified by sampled re-execution** | SYNAPTIC_WEB `:84-87` | **Realized (compile path)** | `pouw/job.py:48-75` — `execute`/`verify` re-compile to byte-identical bytecode + check signature; **but** registering synaptic-compile as a formal PoUW *job class* and wiring PLS access payment is Sprint 3 (`ROADMAP.md:50,52`) |

### 2.4 OriginTrail symbiosis (complement, never compete)

| Paper claim | Where stated | Code status | Evidence |
|---|---|---|---|
| Fiber **reads** verified OT Knowledge Assets into relations; "never invents data" | SYNAPTIC_WEB `:39-50`; 08-knitweb `:580`; COLLECTIVE_INTELLIGENCE `:34` | **Realized** | `synaptic/origintrail.py:41-86` — `resolve_asset` parses explicit triples + linked sources, emits only present fields |
| Fabric **writes** its state root back to OT as a Knowledge Asset (UAL receipt) | 08-knitweb `:580` (resolve), anchor narrative | **Realized (in-process)** | `anchor/origintrail.py:47-67` — content-derived UAL, `submit`/`resolve` round-trip; transport is in-process, a live DKG client is a documented swap (`:9-14`); `test_anchor_origintrail.py` green |
| Light/heavy split: stitch never inlines a blob, only a typed UAL reference | 08-knitweb `:185-198`, `:443-455` | **Partial** | The *discipline* holds in code (Web stores small content-addressed records; UAL references are strings). But UAL *resolution/swarm-fetch of heavy bytes* (§10.2 two-phase fetch) is not implemented — there is no DKG byte-fetch client, only the in-process anchor store |
| OT answers "is this true & whose?"; KnitWeb answers "what is the live state?" | 08-knitweb `:443`; SYNAPTIC_WEB `:47`; COLLECTIVE_INTELLIGENCE `:34` | **Realized (as division of labour)** | Read path (`synaptic/origintrail.py`) + write/anchor path (`anchor/origintrail.py`) + mutable `Web` (`fabric/web.py`) jointly realize the split |

### 2.5 Proof-of-Useful-Work

| Paper claim | Where stated | Code status | Evidence |
|---|---|---|---|
| Deterministic sampled re-execution; exact-match on content digest for float-free jobs | PROOF_OF_USEFUL_WORK `:35-42`, `:61-68` | **Realized** | `pouw/job.py:59-75` re-executes + digest-matches; `test_pouw.py` green |
| Tolerance digest (eps-grid, integers only) for honest-noise slash | PROOF_OF_USEFUL_WORK `:48`, `:62-68`; status `✅ #24` | **Realized (primitive); not on verify path** | `pouw/digest.py` (round-half-up bucket, hash integers); `test_pouw_determinism.py` green — but see §1.1 #1: not wired into `job.verify` |
| Commit-before-sample challenge (Merkle commit, fresh salt, domain tags) | PROOF_OF_USEFUL_WORK `:49-51`, `:70-81` | **Realized (primitive); not on verify path** | `pouw/challenge.py` 4-step protocol; `test_pouw_determinism.py` green — but see §1.1 #2: not invoked by `job.verify`/`escrow`/`dispute` |
| Escrow settle-on-verify (conservation-preserving Knit, mints nothing) | PROOF_OF_USEFUL_WORK `:113-121` | **Realized** | `pouw/escrow.py` `settle_on_verify`; `token/mint.py:117-156` keeps escrow (transfer) and mint (coinbase) separate |
| **Dispute window** (`release_delay > dispute_window`, slash on detected mismatch) | PROOF_OF_USEFUL_WORK `:54`, `:97-99` (marked 📐 *designed, Sprint 2*) | **Realized — now ahead of the doc** | `pouw/dispute.py:68-185` enforces `release_delay > dispute_window` (`:84`), slashing + refund; `test_pouw_dispute.py` 18 passed. The PoUW doc still lists this as "designed" — the doc is stale relative to code |
| **k-of-n verifier quorum** (~55% confirm, ~33% adversary) | PROOF_OF_USEFUL_WORK `:55`, `:100-107`, `:149`; 08-knitweb `:387` (quorum-pulse `k=3,m=2`) | **Aspirational** | No `pouw/quorum.py` exists (confirmed absent); `ROADMAP.md:48,75` Sprint 2, status 📐. `dispute.py:27` itself notes quorum "is the next increment" |
| Collateral sizing / winning-ticket probabilistic settlement | PROOF_OF_USEFUL_WORK `:56`, `:108-111` | **Aspirational** | `ROADMAP.md:49,76` Sprint 2; not in `pouw/escrow.py` |
| GPU compute path (Julia/WebGPU producer) | PROOF_OF_USEFUL_WORK `:135-137`; `__init__.py:15` | **Aspirational** | `pouw/scheduler.py` is a pure-stdlib bounded gate with **no GPU/driver deps** (`:9-10`); actual compute is "a later producer plugin"; `DEPENDENCY_READINESS` notes `wgpu`/`juliacall` not installable. The `__init__.py:15` "(Julia + WebGPU)" label is aspirational |

### 2.6 Transport / P2P claims

| Paper claim | Where stated | Code status | Evidence |
|---|---|---|---|
| L2 today = stdlib-`asyncio` signed-feed sync + static peers | 08-knitweb `:21-22`, `:582`; CLAUDE.md | **Realized** | `p2p/node.py:1-9` static peers + signed feed replication over localhost; `ROADMAP.md:16` ✅ |
| **py-libp2p / DHT / pubsub** backend | 08-knitweb `:21-22`, `:692`, `:623`; A.4 transport profile | **Aspirational** | `p2p/node.py:3`, `p2p/discovery.py:13`, `fabric/feed.py:7` all describe DHT/libp2p as "later"; `ROADMAP.md:54` B11 "later, gated on sanctioned install" |
| Bluetooth-LE mesh / QUIC / Nostr bridge transports | 08-knitweb `:261`, `:688-693` | **Aspirational** | No transport modules beyond asyncio localhost; described only as the "reference profile" |
| In-session **hashgraph** + **blockchain** settlement layers (three-consistency stack) | 08-knitweb §9 `:399-437`, `:520-522` | **Aspirational (by design)** | No hashgraph or blockchain module in `src/`; the paper frames these as "narrow, optional services… omitted" for deployments without value/ordering (`:437`). MOLGANG game itself is an L5-plugin concept, not built |

### Fidelity verdict

The papers and the code are in strong, honest alignment on everything load-bearing: all seven primitives exist with matching roles, and the central reconciliation — `Fiber` as a non-transferable state-commitment versus `PLS` as the demand-gated, no-premine pay-token moved by `Knit` — is fully realized and tested, exactly as the consistency pass claims. The differentiating Synaptic Web USP (deterministic, reversible, provenance-signed edge bytecode) and the OriginTrail symbiosis (read via `synaptic.origintrail`, write/anchor via `anchor.origintrail`) are implemented and green, with the compression claim notably *under*-asserted in tests rather than over-sold. The PoUW story is largely realized and in one place (the dispute window) actually runs ahead of its own spec, which still labels it "designed" — `PROOF_OF_USEFUL_WORK.md` should be refreshed. The genuinely aspirational set is clearly bounded and mostly self-flagged in `ROADMAP.md`/docstrings: k-of-n verifier quorum (no `quorum.py`), collateral/winning-ticket economics, per-epoch mint cap and 1-pulse-per-bundle access payment, the GPU/Julia/WebGPU compute producer, libp2p/DHT and the richer transports, and the hashgraph/blockchain tiers of the three-consistency stack. The one stale, unguarded over-claim worth fixing is `src/knitweb/__init__.py:17` advertising "ERC20-like user-issued LoomTokens," which is unbuilt and directly contradicts the owner decision recorded at `ROADMAP.md:99-100` ("Maak geen loomtoken").

## 3. Is the Python code optimal & minimal-scripts?

This section answers the two questions directly: (a) is the runnable-script surface minimal, and (b) is the Python optimal. Findings are from a read-only pass over `fix/consistency-pass-02` (PR #49). All paths absolute under `/Users/develuse/repo/pulse`.

### 3.1 Metrics

| Area | Files | LOC | Role |
|---|---|---|---|
| `src/knitweb` (library) | 50 (44 non-empty) | 5,536 | The whole system: L0 core → L6 token |
| `tests` | 37 | 4,079 | property (fast, deterministic), interop (2-node), loom |
| `examples/` | 2 | 156 | `mvp_demo.py` (102), `synaptic_demo.py` (54) |
| `tools/` | 1 | 130 | `loc_report.py` — generates the gitignored LOC doc |
| `experiments/` | 1 | 122 | `ledger.py` — proofs-ledger helper (library, no `__main__`) |
| Console script | — | — | `knitweb = knitweb.app.cli:main` (`app/cli.py`, 291 LOC) |

Test-to-src ratio ≈ **0.74:1** (4,079 / 5,536), **273 test functions** across 37 files. Runtime dependency surface = **exactly one** package: `cryptography>=42`. Everything else (`py-libp2p`, `wgpu`, `juliacall`, `polars`, `pyarrow`, `mlflow`, `pytest`) is gated behind per-role optional extras (`p2p/compute/data/experiments/dev`). A scan of all non-stdlib imports in `src/` confirms no hidden third-party coupling — the only non-stdlib import in the library is `cryptography`.

### 3.2 Minimal scripts

There are exactly **five** files with a runnable surface (`__main__` or a console entry):

1. `app/cli.py` — the single `knitweb` console script (the one operational pipeline)
2. `examples/mvp_demo.py` — end-to-end acceptance demo (genesis → p2p pay → PoUW → persist → checkpoint)
3. `examples/synaptic_demo.py` — the SDK/edge loop (pay → compile → verify → decode)
4. `tools/loc_report.py` — generates `docs/LOC_BY_LANGUAGE.md` (gitignored; per CLAUDE.md, deliberately not hand-tracked to avoid merge conflicts)
5. `experiments/ledger.py` — has **no** `__main__`; it is a proofs-ledger *library* called from tests/phases, not a standalone script

Assessment: there is **no redundancy and no superseded scripts**. Each entry point has a distinct, non-overlapping job — one CLI, two demos that exercise different layers (full-stack ledger vs. SDK/edge), one generator, one record helper. The "one pipeline, reuse files" mandate is honored: the CLI is the only operational pipeline; the demos and tests import the library rather than re-implementing it (e.g. `tests/interop/test_mvp_demo.py` runs the demo as the acceptance proof rather than duplicating it). The two demos are *not* duplicates — `mvp_demo` drives the raw ledger/p2p/PoUW stack while `synaptic_demo` drives the `sdk` facade and edge-decode path; collapsing them would lose coverage of two distinct public surfaces. This is a clean, minimal script surface.

### 3.3 Optimality

**LOC distribution / hotspots.** The distribution is healthy and flat. Largest modules are `p2p/node.py` (361), `app/cli.py` (291), `core/crypto.py` (246), `synaptic/bytecode.py` (234), `core/canonical.py` (224), `fabric/feed.py` (224), `pouw/dispute.py` (214), `store.py` (210). Nothing is a god-module; the median module is ~100 LOC. The only naturally large file is the asyncio P2P node, which is justified (wire framing + lifecycle + handlers).

**Pythonic quality — strong.** `@dataclass(frozen=True)` is used pervasively (20 files), with `__post_init__` invariant checks on value objects. `from __future__ import annotations` in 39 files. `__all__` discipline is **excellent**: 41 of 44 non-empty modules export an explicit `__all__`; the only three without it are the thin package aggregators (`knitweb/__init__.py`, `app/__init__.py`, `looms/__init__.py`), which is the correct exception. Typed signatures throughout, including modern union syntax (`bytes | None`).

**Canonical / hashing hot path — optimal for its contract.** `core/canonical.py` is a hand-rolled, float-free deterministic CBOR encoder/decoder (zero external surface, as mandated). The encoder is shortest-form per RFC 8949 §4.2; the decoder is *strict* (rejects non-minimal heads, unsorted/duplicate keys, indefinite-length, trailing bytes) — matching the RLP ErrCanonInt / Cosmos ADR-027 guarantee. The one micro-inefficiency is `+=` byte concatenation in `_encode` for arrays/maps (quadratic in the worst case for very large containers); a `bytearray`/`b"".join` accumulator would be the textbook optimization. In practice records are small (ledger entries, attestations), so this is a non-issue at current scale — a note, not a defect. `crypto.sign/verify` reconstruct the EC key from hex on every call (`_load_private`/`_load_public`); for a tight signing loop an `lru_cache` on key objects would help, but key reuse is low and caching private-key material is a deliberate security trade-off, so the current choice is defensible.

**Test coverage — broad and intentional.** Property tests cover canonical (incl. fuzz + strict-decode), crypto, ledger, address scheme, all four looms, p2p discovery/replay, pouw (determinism/dispute/scheduler), token mint, fabric (attest/items/feed/provenance), spatial, sdk, synaptic, store, and CLI. Critically, the property suite is **deterministic** — `hypothesis` is imported in **0** test files and `cbor2` parity is not wired — consistent with the stated intent that pytest/cbor2/hypothesis are not runtime deps. The "fuzz" test is a self-contained deterministic generator, not Hypothesis-driven.

**The one real duplication — the four domain looms.** This is the clearest optimization target. `looms/{chemistry,finance,operational,supplychain}/__init__.py` each independently re-implement the **same loom skeleton**:
- identical constructor: `self._priv = priv; self.actor_pub = crypto.public_from_private(priv); self.address = crypto.address(...)`
- identical `to_record` tail: `canonical.encode(record)  # fail fast on any non-canonical content`
- a near-identical `emit` (compute domain invariant → raise if non-zero → `attest(self.to_record(x), self._priv, author_field=...)`)
- a byte-for-byte-equivalent `weave(x, web) -> tuple[str, Attestation]`

Only the *invariant* (element/charge balance, double-entry sum, mass conservation, capacity feasibility), the `KIND` string, and the record shape genuinely differ. Roughly 25–30 LOC per loom (~100+ LOC total) is mechanical boilerplate. This does not threaten correctness (each loom is well-tested), but it is the spot where "reuse files" is least honored *inside* the library.

### 3.4 Concrete improvements (ranked)

1. **Factor a shared loom base** (highest value). Introduce a small `looms/_base.py` (e.g. `SignedLoom` ABC or a mixin) carrying the priv→pub→address constructor, the `to_record`-fails-fast guard, the generic `emit` (invariant hook + `attest`), and `weave`. Each domain loom would then supply only `KIND`, `_invariant(event) -> error_or_none`, and `_build_record(event) -> dict`. Removes ~100 LOC of duplication and makes the "every loom is a signed conservation gate" thesis explicit in code, not just in four parallel docstrings.
2. **Unify the author-field convention.** Three looms use `actor`, chemistry uses `author`; the base class would normalize this to one named hook, eliminating the `author_field=` divergence.
3. **`bytearray`/`b"".join` accumulator in `canonical._encode`** for arrays/maps — cheap, removes the only algorithmic wart in the hash hot path; do it only if/when large containers appear (currently premature).
4. **Optional `lru_cache` on `_load_public`** for verify-heavy paths (e.g. replaying a braid) — measure first; skip if it complicates the security story.
5. **Naming clarity (docs, not code):** `ledger/loom.py` (the core validation primitive) and `looms/*` (L5 domain plugins) share the "loom" word; a one-line note in the looms package would prevent reader confusion. Not a code change.

**Already optimal / leave alone:** the single-dependency surface and per-role extras (textbook minimal); the strict canonical codec with zero external surface; `__all__`/dataclass/typing discipline; the five-script surface; the test layout and deterministic property suite; the `attest`/`verify_record` envelope (already the shared, non-duplicated signing primitive the looms correctly call into).

### 3.5 Verdicts

- **Optimal? Mostly.** The architecture is clean, strictly typed, well-tested (0.74:1), single-dependency, and the hash-critical path is correct and minimal-surface. The only material non-optimality is ~100 LOC of mechanical boilerplate duplicated across the four domain looms (a shared base class is the obvious fix); the canonical-encoder concatenation and key-reload are minor, scale-dependent, and currently fine. None of these affect correctness.
- **Minimal scripts? Yes.** Five runnable entry points, each with a distinct purpose, no superseded or redundant scripts, one operational pipeline (the `knitweb` CLI), demos and tests reuse the library rather than re-implementing it, and the LOC report is generated-not-tracked exactly as the CLAUDE.md mandate requires.

## 4. Architecture & differentiation

Knitweb (`knitweb` package, repo `pulse`, branch `fix/consistency-pass-02`) is a single-language (Python ≥ 3.12) implementation organized as a strict seven-stratum stack. The layering is asserted in `CLAUDE.md`/`README.md` and is observable in the import graph under `src/knitweb/`.

### 4.1 The L0→L6 layer stack and dependency discipline

| Layer | Module(s) | Role |
|---|---|---|
| **L0 core** | `core/canonical.py`, `core/crypto.py`, `core/pulse.py` | crypto, canonical encoder, CID, the Pulse heartbeat |
| **L1 ledger** | `ledger/{blob,fiber,loom,knit,braid,node}.py` | integer settlement primitives |
| **L2 p2p** | `p2p/{node,wire}.py` | asyncio signed-feed sync, Knit handshakes, peer exchange |
| **L3 fabric** | `fabric/{web,feed,items,attest,spatial,...}.py` | the woven Web graph, signed feeds, attestation |
| **L4 pouw** | `pouw/{job,challenge,digest,escrow,dispute,scheduler}.py`; `synaptic/`; `edge/` | proof-of-useful-work, the Synaptic compiler, edge runtime |
| **L5 looms** | `looms/{finance,operational,supplychain,chemistry}/` | domain plug-ins |
| **L6 token** | `token/mint.py`; `anchor/origintrail.py` | PLS issuance, OriginTrail anchoring |

A mechanical pass over every `from ..X import` / `from ...X import` edge confirms the dependency arrows run **downward** with one well-understood exception. Each layer imports only `core` plus lower layers: `ledger→core`; `fabric→core`; `pouw→{core, ledger, synaptic}`; `looms→{core, fabric}`; `token→{core, ledger, pouw}`; `anchor→{core, fabric}`. The `core` package itself imports nothing from `knitweb` outside its own files (`canonical.py` is dependency-free; `crypto.py` depends only on `cryptography`). The single inversion of the nominal stack is `p2p` (L2) importing `fabric.feed` (L3) in `p2p/node.py` and `p2p/wire.py`. This is deliberate, not accidental coupling: `fabric/feed.py` is documented and coded as a **pure, networking-free data structure** (a Hypercore-style signed append-only log) whose only dependency is `core`, and it has no back-edge to `p2p`. So there is no import cycle; the "feed" is effectively a transport-adjacent primitive that the README placed in `fabric` for narrative reasons. The discipline ("strictly downward, domain looms are L5 plug-ins never in core") therefore holds in the soundness-relevant sense (no cycles, core is a sink), with the caveat that the L2/L3 boundary is a layering label rather than a hard dependency floor.

### 4.2 The four invariants, confirmed in code

- **Single crypto scheme.** `core/crypto.py` hard-codes `_CURVE = ec.SECP256K1()` and signs/verifies ECDSA over a `Prehashed(SHA256())` digest. `KNOWN_SCHEMES = {SCHEME_SECP256K1_ECDSA}` (= 0) is the only blessed scheme; addresses carry a 1-byte scheme version reserving 1/2/3 for SPHINCS+/ML-DSA/hybrid by future soft-fork, but none are active. No Ed25519/BLAKE2b appears in the value path.
- **Single canonical encoder.** `core/canonical.encode` is a hand-rolled deterministic CBOR subset (shortest-form heads, byte-sorted map keys); `decode` is *strict* (rejects non-minimal heads, unsorted/duplicate keys, indefinite lengths, trailing bytes), giving exactly one byte-string per value (the RLP `ErrCanonInt` / Cosmos ADR-027 guarantee). `cid()` produces a real CIDv1 (`0x01` + dag-cbor `0x71` + sha2-256 multihash, base32-lower).
- **Float-free.** `_encode` raises `CanonicalError` on any `float`; even the float-tolerant PoUW path (`pouw/digest.py`) quantizes to integer buckets *before* hashing, so no float ever reaches a digest.
- **Integer-only value path.** `ledger/blob.py` rejects non-`int` (and `bool`) amounts and negative balances on credit/debit; `token/mint.py` computes rewards with integer floor division (`escrow * rate_num // rate_den`) and clamps `min(r, escrow)` and an optional `max_supply`.

### 4.3 Differentiated vs table-stakes

| Capability | Table-stakes or novel | Maturity | Evidence |
|---|---|---|---|
| secp256k1+SHA-256, strict canonical CBOR, CIDv1, integer balances | Table-stakes (parity with Bitcoin/ETH/Cosmos/IPLD) | **Shipped** | `core/crypto.py`, `core/canonical.py`, `ledger/blob.py` |
| Signed append-only feed w/ provable equivocation (fork counter, signed tree head) | Table-stakes (Hypercore-equivalent, re-derived on own primitives) | **Shipped** | `fabric/feed.py` |
| Seven-primitive minimalist surface (Blob/Fiber/Loom/Knit/Braid/Web/Pulse) | Novel framing (deliberately tiny deterministic surface) | **Shipped** | `ledger/*`, `fabric/web.py`, `core/pulse.py` |
| **Synaptic Web edge-bytecode compiler** (verified relations → interned, LEB128, sorted, signed, content-addressed bundle) | **Novel — the USP** | **Shipped** (format v1, deterministic, round-trips) | `synaptic/bytecode.py`, `edge/runtime.py` |
| PoUW with sampled re-execution, commit-before-sample challenge, fresh-salt indices, Merkle membership, **tolerance digests** for GPU non-determinism | **Novel** (the tolerance digest directly addresses the Chutes/Targon raw-float-equality failure) | **Shipped as protocol on a deterministic CPU job; not yet wired to real GPU kernels** | `pouw/{job,challenge,digest}.py`, `pouw/scheduler.py` |
| Demand-gated bounded mint, no premine, no raw `mint()` (`Treasury.reward_verified_work` only), anti-replay on proof digest | Novel-leaning (credible-neutrality as a code invariant) | **Shipped** logic; **Planned** Pulse-epoch/Beat cap binding | `token/mint.py`, `CLAUDE.md` |
| OriginTrail complement (read assets in, anchor checkpoint roots out via content-derived UAL) | Novel positioning (complement-not-compete with the DKG) | **Partial** — in-process transport; real DKG client not wired | `synaptic/origintrail.py`, `anchor/origintrail.py` |
| Four domain looms (finance/operational/supply-chain/chemistry) as L5 plug-ins | Table-stakes extensibility | **Shipped** (property-tested) | `looms/*`, `tests/looms/*` |
| P2P transport | Table-stakes | **Partial** — stdlib `asyncio` over static peers/localhost; py-libp2p/DHT explicitly deferred | `p2p/node.py` docstring |
| CLI node (pay PLS, compile/verify/edge-load bytecode, persistent state) | Table-stakes | **Shipped** | `tests/interop/test_cli*.py` |

The full suite collects **273 tests** (property + interop + looms), green per the consistency pass.

### 4.4 Readiness gaps that bound the claim

The "verifiable GPU compute" thesis is not yet demonstrated on a GPU: the only PoUW job today is `SynapticCompileJob`, whose "useful work" is the deterministic resolve-and-compile of an OriginTrail asset, and `pouw/scheduler.py` is a bounded semaphore with **no GPU/driver dependencies** by design. So sampled re-execution and tolerance digests are proven as *mechanisms* but exercised against deterministic CPU work, not chaotic ML kernels. The OriginTrail interlock is in-process (assertions kept in a dict; UAL is content-derived but not published to a live DKG node). P2P is single-host asyncio, not libp2p/DHT. Post-quantum schemes are reserved but inactive. The `max_supply`/Pulse-epoch binding is documented as planned, not implemented.

### 4.5 Positioning verdict

This is a sound, defensibly-layered architecture: the soundness-critical core is a genuine dependency sink, the four invariants (one curve, one strict encoder, no floats, integer money) are enforced in code rather than convention, and the credible-neutrality story (no premine, no ungated mint, demand-bounded issuance) is structurally true at the type level, not merely asserted. The sharpest, most defensible edge is the **Synaptic Web edge-bytecode compiler paired with the tolerance-digest PoUW**: shipping a deterministic, content-addressed, signed "relation matrix" that a kilobyte-scale edge device can verify-then-execute, while solving the GPU-non-determinism slashing problem that has broken comparable compute-marketplace designs, is a combination not present in the prior-art corpus. The bounding risk is maturity, not design — the differentiation is real at the protocol layer but still awaits a live GPU workload, a real libp2p web, and a production DKG round-trip before the "verifiable GPU DePIN" claim is fully earned.

## Findings: the two questions

### (a) Is the current Python code already optimal?

**Verdict: Mostly — clean, strictly typed, single-dependency, and correct on the hash-critical path; one ~100-LOC duplication is the only material wart.**

- The architecture is flat and disciplined: no god-module, median module ~100 LOC, pervasive `@dataclass(frozen=True)` with `__post_init__` invariants, `__all__` on 41/44 non-empty modules, and a strict float-free canonical codec with zero external surface (§3.3).
- The single material non-optimality is ~100 LOC of mechanical boilerplate duplicated across the four domain looms (`looms/{chemistry,finance,operational,supplychain}`), fixable with a shared `looms/_base.py` carrying the constructor, `to_record` guard, `emit`, and `weave` (§3.3, §3.4 item 1).
- Remaining nits are minor and scale-dependent: `+=` concatenation in `canonical._encode` (premature to fix at current record sizes) and per-call EC key reload in `crypto` (a deliberate security trade-off). None affect correctness (§3.3).

### (b) Does it work with a minimal number of scripts?

**Verdict: Yes — five distinct runnable entry points, one operational pipeline, zero redundancy.**

- Exactly five files have a runnable surface: `app/cli.py` (the sole `knitweb` console pipeline), `examples/mvp_demo.py`, `examples/synaptic_demo.py`, `tools/loc_report.py`, and `experiments/ledger.py` (a library with no `__main__`) (§3.2).
- No superseded or duplicate scripts: the two demos exercise different public surfaces (raw ledger/p2p/PoUW vs. the `sdk`/edge-decode path); collapsing them would lose coverage (§3.2).
- The "one pipeline, reuse files" mandate holds: demos and tests import the library rather than re-implementing it (`tests/interop/test_mvp_demo.py` runs the demo as the acceptance proof), and the LOC report is generated-not-tracked per the CLAUDE.md mandate (§3.2).

## Recommendations

### Before migration to the knitweb org

These are correctness/honesty items that should not ship under a public org banner uncorrected:

1. **Fix the stale over-claim** at `src/knitweb/__init__.py:17` ("ERC20-like user-issued LoomTokens"). It is unbuilt and directly contradicts the owner decision `"Maak geen loomtoken"` (`ROADMAP.md:99-100`). Remove or rephrase to match the roadmap (§2.2, §2 fidelity verdict).
2. **Refresh `PROOF_OF_USEFUL_WORK.md`** where it still marks the dispute window as "designed (📐)" — the code runs ahead of the doc (`pouw/dispute.py:68-185`, `test_pouw_dispute.py` 18 passed). Doc should read "shipped" (§2.5).
3. **Wire the two existential proof-model primitives onto the live path** — tolerance digests (`pouw/digest.py`) and salt + commit-before-sample (`pouw/challenge.py`) — or explicitly document in the papers that the *live* verify path is the deterministic exact-match special case (`pouw/job.py:69-75`) and the GPU-grade primitives are present-but-unwired. This is the single biggest "built but unwired" gap and the corpus's clearest trailing edge (§1.1 #1–#2, §1.4).
4. **State the maturity boundary in the README/claims**: the "verifiable GPU DePIN" thesis is proven as a *mechanism* against deterministic CPU work only; no live GPU workload, no libp2p web, no production DKG round-trip yet (§4.4). Honest scoping before public exposure.

### Post-migration / nice-to-have

5. **Factor `looms/_base.py`** to remove the ~100 LOC of loom boilerplate and unify the `author`/`actor` field convention (§3.4 items 1–2). Highest code-quality value; not correctness-blocking.
6. **Land the k-of-n verifier quorum** (`pouw/verifier-quorum`, the explicit TODO at `dispute.py:27-28`) — the largest standing economic gap vs. Livepeer/EigenLayer/Targon (§1.1 #4, §2.5).
7. **Add winning-ticket / streaming escrow + declared-vs-detected fault asymmetry and collateral minimum-sizing** (§1.1 #5, §1.3 Livepeer/Filecoin rows).
8. **Bind the per-Pulse-epoch mint cap and 1-pulse-per-bundle access payment** (Sprint 3, `ROADMAP.md:52,83`) so `Pulse` becomes the mint-window driver the papers describe, not just a clock (§2.1, §2.2).
9. **Scale-gated micro-optimizations** only if profiling warrants: `bytearray`/`b"".join` in `canonical._encode`; optional `lru_cache` on `_load_public` for braid replay (§3.4 items 3–4).
10. **Deferred-by-design backlog**: libp2p/DHT transport, partial-range Merkle proofs, live DKG byte-fetch client, GPU/Julia/WebGPU compute producer (§1.2, §2.6, §4.4).

## Conclusion

At `fix/consistency-pass-02`, Knitweb/Pulse is a rigorous, honestly-scoped codebase whose cryptographic and timing scaffolding now meets or beats the crypto corpus, and whose papers are in strong alignment with what actually ships. All seven primitives, the Fiber-as-commitment / PLS-as-pay-token reconciliation, the Synaptic Web edge-bytecode USP, and the OriginTrail symbiosis are realized and green across 273 tests. The Python is mostly optimal and runs on a genuinely minimal five-script surface with a single runtime dependency. The real gaps are bounded and mostly self-flagged: the existential proof-model defenses (tolerance digests, salt/commit-before-sample) are built and correct but not yet on the live verify path; the k-of-n quorum, probabilistic escrow, and per-epoch mint cap remain backlog; and the "verifiable GPU DePIN" claim still awaits a live GPU workload, a libp2p web, and a production DKG round-trip. Design is sound; the remaining work is wiring and maturity, not redesign.