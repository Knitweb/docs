# knitweb/molgang — repository instructions

**What it is.** A peer-to-peer **chemistry knowledge game** on knitweb. Players walk into a bar, take a seat, **knit chemistry terms** (spend silk), peers **vote with a pulse**; a **quorum weaves** the term into the shared fabric as a **Fiber** (a real knowledge-graph node/edge), anchored to OriginTrail. Economy = **PLS + silk + reputation, no NFTs**.

## Run the bar (Python)
```bash
git clone https://github.com/knitweb/molgang && cd molgang
git clone https://github.com/knitweb/pulse ../pulse          # the engine, as a sibling
python3 -m venv .venv && . .venv/bin/activate && pip install -e . -e ../pulse
molgang serve --port 8765 --world ~/.molgang/world.json --db ~/.molgang/registry.db
# explorer over the woven web:
molgang.explorer --web ~/.molgang/world.json --port 8990
```

## The machine API (same one bots/agents drive)
`GET /api/state?sid=` · `POST /api/join {name,avatar,device}` → `{sid,address}` · `POST /api/sit {sid,table}` · `POST /api/propose {sid,term}` · `POST /api/vote {sid,pid,verdict}` · `GET /api/web` · `GET /api/graph` · `POST /api/certificate {sid}` (public PoUW PDF — **always redacted**; shows PLS balance + staked votes; bearer/private-key export is local-CLI-only via `--private --confirm-private-key-export`) · `GET /api/certificates` (tracked list of issued certificates incl. PDF sha256) · `GET /api/web/jsonld` (provenance-linked JSON-LD: OriginTrail UAL + state_root + term lang/direction) · `GET /metrics` (Prometheus) · `POST /api/spiral/*`.

A stale `sid` (e.g. after a server restart) answers `GET /api/state` with `200 { you: null }` — clients detect that and transparently re-join with the same `device` wallet; balances persist in the registry DB across restarts.

The browser client is **path-prefix-safe** — works at `/` or under `/molgang/` — and is an **installable PWA**: web app manifest + icons (#115), an offline-first service worker (cached app shell, network-first `/api/*` with cached fallback, versioned caches — #116), and **EN/NL i18n chrome** (`web/i18n.js` + `web/locales/*.json`, `data-i18n` scan, header language switcher; canonical vocabulary Web/Knitweb/Knit/Pulse/Fiber/silk/spiders/PLS is never translated — #117).

## Three deploy shapes (pick one canonical — #58)
1. **Python `molgang serve`** — the authentic engine (needs an always-on box).
2. **PHP/MySQL dapp** (`php/`) — request-driven, runs on **shared hosting** (live at 5mart.ml/molgang). See the 5mart.ml runbook.
3. **Static UI + containerized backend** (#35/#47).

## Open priorities (July 2026)
- **Must** #88–#96 p2p hardening (Kademlia routing, hole-punch, equivocation quarantine, Merkle range proofs, multi-relay) · #108/#109 graded chemistry ground truth + reaction conditions · #118 mobile-responsive bar · #121 Prometheus RED on the /api contract · launch wave: #128–#133/#138/#139.
- **Should** #54 e2e+agentplay · #57 faucet redeploy · #106 TOKENOMICS.md · #30/#32/#33 multilingual chemistry KG (**gated on pulse CID-stability sign-off**).
- **Recently shipped:** certificate shows live PLS balance + tracked list `GET /api/certificates` (#224) · restart-recovery contract tests (#226) · provenance JSON-LD (#227→#107) · installable PWA (#228→#115) · offline service worker (#229→#116) · EN/NL i18n chrome (#230→#117) · lab-3d device-wallet persistence (#225, lands with PR #223).

## Test
```bash
PYTHONPATH=.:src:../pulse/src python3 -m pytest -q   # Python (pulse sibling required; 338 tests)
php php/tests/smoke.php                       # PHP dapp (SQLite, no server)
```
