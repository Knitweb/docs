# Ecosystem backlog cleanup — 2026-07-22

Tracking document for a cross-organization backlog triage across all active `febuz`
repositories: `5mart-ml`, `Knitweb`, `ChemField`, `fieldintelligence`, `ledgerfield`,
`GithField`, `FinField`, `molgang`, `virtuanalytica`, plus personal `febuz/*` repos.

## Why

An inventory via `gh` across all 9 organizations found:

- **184 open issues**, **18 open PRs**, spread over ~50 non-archived repositories.
- Knitweb alone carries 119 open issues, a large share of which are multi-week epics
  (gither P2P federation G2-01..23 + CG-01..15, molgang's 1,000,000-concurrent-peer
  load-test cluster #122-#144, ledgerfield's 17-country tax modules #29/#38,
  ChemField's Slag Run production chain #14/#18-24).
- A recurring template issue, **"Role-driven feature backlog — from the 100-professions
  field guide,"** appears with an identical title in 6 repos (docs#1, gither#1,
  knitweb.github.io#1, molgang#198, monitor#4, vank#1) — needs checking for literal
  duplication vs. repo-specific content.
- The older `numerai_signal` (#1-18) and `Numer_crypto` (#17-35) issues (opened
  2026-06-20) likely overlap with later work in `numerai-crypto-signals`/`5mart_numerai`,
  where the feature frontier is already documented as **exhausted** (17/17 formulations
  tested).

## Scope for this pass

1. **Depth: triage + quick wins.** Merge PRs where safe, close duplicate/stale/template
   issues with evidence, and land a handful of small concrete bugfixes. Large epics stay
   open but get recognized/labeled rather than attempted — out of scope for a single pass.
2. **Priority: Numerai submission flow stays untouched.** Per standing project policy,
   Numerai submission quality is top priority; the daily/deadline submission scripts are
   not modified. Numerai-repo backlog work is limited to issue triage (close with
   evidence), no new features or experiments.
3. **Numerai legacy issues: cross-checked, not blanket-closed.** Each of the 35 older
   `numerai_signal`/`Numer_crypto` issues is checked against current evidence before
   closing; only issues that are genuinely superseded get closed, real gaps stay open.

## Phases

- **Phase 0 — Numerai issue cross-check** (35 issues): verify each against
  `numerai-crypto-signals`/`5mart_numerai` state and process-learning evidence; close
  with a reference to the evidence where superseded/rejected/deployed, keep real gaps
  open. No changes to submission code.
- **Phase 1 — Open PR triage** (18 PRs): merge CI-green low-risk PRs (docs/citations/small
  fixes); evaluate larger PRs for merge-readiness; claim project leases
  (`coord.py claim <project>[/lane]`) before touching molgang/numerai/virtualpc.
- **Phase 2 — Duplicate/template issue cleanup**: resolve whether the six "Role-driven
  feature backlog" issues are literal duplicates; keep one canonical tracker, close or
  re-scope the rest.
- **Phase 3 — Concrete quick-win bugfixes**: `molgang-web`#15 (steel_grades_produced
  overcounting + missing rollback), `FinField/scrapers`#7 (stooq anti-bot wall),
  `FinField/web`#3 (field-isolation leak), plus the three "sync routine blocked" issues
  (`molgang-roblox`#7, `molgang-web`#1/#2) — checked for staleness first.
- **Phase 4 — Epic recognition, no implementation**: confirm large epics (gither
  G2-*/CG-*, molgang scale-test cluster, ChemField Slag Run, ledgerfield tax modules)
  are clearly labeled as multi-week so a future pass doesn't mistake them for quick
  closes.

## Verification

- Re-run the same `gh search/issues` counts per org before/after each phase and report
  the delta.
- Spot-check 5-10 closed issues for a substantiated closing comment (evidence link or
  reason).
- Confirm the Numerai daily submission cron is unchanged and still runs.
- Final report: per-org table of (issues open→open, PRs open→merged/open) plus a list of
  remaining epics with rough size, so a follow-up pass can pick up cleanly.

## Results (2026-07-23)

| Org | Issues open (before → after) | PRs open (before → after) |
|---|---|---|
| febuz | 47 → 17 | 6 → 6 |
| Knitweb | 119 → 124* | 11 → 12* |
| ChemField | 10 → 10 | 0 → 0 |
| ledgerfield | 2 → 2 | 0 → 0 |
| FinField | 6 → 6 | 1 → 2** |

\* Knitweb/gither and Knitweb/molgang carry active scheduled backlog-generation crons
(see e.g. lease note "knitweb: backlog #123 Grafana/Prometheus") that opened new issues
during this pass, offsetting the closes/merges below — the raw before/after delta is not
a clean measure of this pass's impact.
\*\* net +1 because this pass opened FinField/web#7 (the field-isolation fix).

**Closed (30 issues, all with an evidence-based comment):**
- 28 of the 37 legacy `numerai_signal`/`Numer_crypto` issues — superseded by
  production infrastructure (round_dashboard, promotion gate, deadline-safe flow,
  MLflow tracking, crypto_arms.json) or by investigated-and-rejected findings (regime
  classifiers, Yiedl on-chain features, sector/peer-cluster features). 9 kept open as
  real gaps (signals #7/#13/#15/#16/#17, crypto #30/#32/#33/#34).
- `molgang-web`#15 — described bugs (`steel_grades_produced` overcount, unrolled-back
  atom deduction) belong to abandoned PR #10; current `steel.py` uses a different,
  already-atomic data model. Confirmed via code search before closing.
- `molgang-web`#2 — duplicate of `molgang-web`#1 / `molgang-roblox`#7 (same root cause).

**Merged (5 PRs):** `gither`#76, `weave-node`#1, `weave-core`#1, `molgang`#287,
`pulse`#365.

**Opened (1 PR):** `FinField/web`#7 — fixes the field-isolation bug in `web/studio/fin-engine.js`
described in `FinField/web`#3 (Studio leaked cross-field facts into the vote list).

**Confirmed not duplicates:** the six "Role-driven feature backlog" issues (docs#1,
gither#1, knitweb.github.io#1, molgang#198, monitor#4, vank#1) each carry distinct,
repo-specific content deduplicated from the same 490-request source survey — no
action needed.

**Labeled as epics (14 issues, not implemented):** `molgang`#122/124/126/127/129/130/
132/133/134/136/138/144 (the load/scale-test cluster) and `ledgerfield`#29/#38 (Tier-2
tax modules, Knitweb P2P integration) now carry a new `epic` label. `gither`'s
G2-*/CG-* federation issues and `openchem`'s Slag Run issues already had adequate
`epic:*`/`slag-run` labeling.

## Follow-up pass (2026-07-23): the 5 conflicting PRs

All 5 resolved:
- `agent-mesh-whitepapers`#28/#29 — checked individually: the `03_MeshLattice.md` fixes
  had already landed on `main` via a separate run; only `05_KnitNet_KnowledgeGraph.md`'s
  citation-style cleanup was still outstanding. Re-applied directly against current
  `main` as `agent-mesh-whitepapers`#33, merged. #28/#29 closed as superseded.
- `FinField/facts`#8 — diffed content directly: all 30 net-new records this PR would add
  already exist verbatim in `main` (landed independently via a later batch). Closed as
  redundant, nothing to merge.
- `pulse`#364 and `pulse`#357 — both were pure-addition PRs (new files only) whose sole
  conflict was an adjacent-line collision in `docs/FEATURES.md` against the
  already-merged GeoWeave/PAR bridge (#365) — three PRs had each appended a bullet at
  the same list position. Merged `main` forward into each branch, kept all three bullets,
  verified the new Python files still byte-compile, opened `pulse`#370 / `pulse`#371 as
  direct replacements (original branches closed as superseded, not force-pushed — avoids
  touching another agent's branch ref), waited out full CI (CodeQL + lint analyzers +
  property-test suite, all green on both), and **merged both** — #370 first, then
  re-merged `main` into #371 once more (it had picked up #370's own FEATURES.md line in
  the interim) before its final green CI and merge.

All 5 previously-conflicting PRs are now closed, either merged directly or replaced by a
merged equivalent. Nothing left blocking in this pass.

## Follow-up (2026-07-23): `FinField/scrapers`#7 data-source decision

Verified live that stooq's `/q/d/l/` CSV endpoint still serves a JS SHA-256 PoW
verification page (not solving that programmatically, per the issue). Implemented
Option 1 from the issue: `FinField/scrapers`#10 adds `alphavantage-eod`, a second
`FactSource` under Alpha Vantage's documented API terms. Deliberately scoped down from
stooq — US-listed tickers only (non-US free-tier coverage is inconsistent, not guessed
at), unadjusted daily close (`_ADJUSTED` moved to Alpha Vantage's premium tier in 2024)
— and opt-in: `covers()` returns `False` with no `ALPHA_VANTAGE_API_KEY` configured, so
existing pipelines don't break for anyone who hasn't set one up. `stooq-eod` stays
registered as-is; it resumes working automatically if stooq lifts the block. 55/55 tests
pass (5 new, offline fixture-based, matching Alpha Vantage's documented response
schema — no live key spent from this pass). Issue #7 left open pending review/merge of
#10, and pending a decision on whether to also pursue the issue's Option 2 (ask stooq
for API access).

## Follow-up (2026-07-23): `molgang-roblox`#7 / `molgang-web`#1 branch fix

Investigated before touching anything: the local checkout the coordination config
pointed at no longer existed on this host, and no sync script/cron could be found here
(runs elsewhere or was decommissioned). Diffing `master` against `main` turned up
something worse than ordinary drift — `master`'s 110 unique commits were not
molgang-roblox content at all. They were **VirtualPC/OpenClaw project history**
(dashboards, Kafka, auth, Docker/K8s docs), almost certainly pushed to the wrong remote
at some point. Verified `master`'s exact tip commit (`ca41f3b9`) already exists
byte-identical in `virtuanalytica/virtualpc` before touching anything, confirming
nothing would be lost.

Owner explicitly authorized a one-off exception to the standing "never force-push
main/master" guardrail for this specific case. Force-pushed `main` onto `master`
(`git push origin origin/main:refs/heads/master --force-with-lease`); both branches now
point at the identical commit `164ffdb5`. Verified via the GitHub API. Both issues
closed with the root-cause explanation.

**Left open, needs manual attention:**
- All multi-week epics (gither federation, molgang scale-test cluster, ChemField Slag
  Run, ledgerfield tax modules) — recognized/labeled, not attempted, per this pass's
  scope.

*Generated by Claude Code, 2026-07-22/23, as the execution plan and results log for a
user-requested ecosystem-wide backlog triage.*
