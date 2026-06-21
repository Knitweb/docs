# 5mart.ml/molgang — QR node-onboarding "backdoor" (p2p registration + sending knitwork)

**Purpose.** A low-friction path to bring a **new peer node** into the knitweb via a **scannable QR code**: scan → the device's deterministic wallet **signs a registration** → it is recorded in the **5mart.ml MySQL node registry** → the node can **send/receive knitweb knitwork** (woven knowledge: concepts + links) through the always-on 5mart.ml relay. It's a "backdoor" only in the sense that it **bypasses the bar walk-in UI** — it is still **wallet-signed and operator-gated**, not an auth bypass. (Backlog: #63 onboarding, #61/#62 5mart.ml live node + HTTP relay.)

## Flow
1. **QR contents** — a deep link, e.g. `https://5mart.ml/molgang/node?reg=<b64url(payload)>` where payload = `{device, pub, ts, sig}`; `sig = secp256k1_sign(priv, "knitweb:node-reg:"+device+":"+ts)`. The device wallet is the deterministic `from_seed(device)` (same wallet across scans).
2. **Scan → register** — the device POSTs the payload to `POST /api/node/register`. The server **verifies the signature** against `pub`, checks `ts` freshness (anti-replay), and upserts a row in the **node registry**.
3. **Send knitwork** — once registered, the node `POST /api/web/push {device, links[], sig}` to contribute woven concepts/links (each link signed); the relay folds verified links into the shared fabric (subject to peer quorum) and re-anchors the `state_root` to OriginTrail.
4. **Pull** — `GET /api/web` / `GET /api/graph` returns the current woven knitweb for the node to mirror (the explorer reads the same).

## MySQL node registry (suggested)
```sql
CREATE TABLE node_registry (
  device   VARCHAR(96) PRIMARY KEY,
  pub      VARCHAR(72) NOT NULL,
  address  VARCHAR(64) NOT NULL,
  first_seen DOUBLE, last_seen DOUBLE,
  status   VARCHAR(12) DEFAULT 'active',   -- active | suspended | royeed
  woven    INT DEFAULT 0
);
```

## Operator control (head node)
The operator keeps oversight for the first months: every registration/contribution is **signature-verified and logged**, and the operator can **suspend / *royeren*** a node (set `status`) to stop spam or push **corrective transactions**. This mirrors the email-subscribe custody intent (#76) — control is **explicit, signed, and time-bounded to the beta**, not a hidden bypass.

## Security notes
- It's wallet-**signed** end to end (no shared secret in the QR); freshness + per-link signatures stop replay/forgery.
- Rate-limit `register`/`push` per device; size-cap pushes; require quorum before woven links become canonical.
- Don't put any private key in the QR — only `pub` + a signature. (The bearer-key PoUW certificate, #55, is a *separate*, operator-only artifact.)
