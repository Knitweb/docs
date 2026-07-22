# Security findings and remediation — 2026-07-23

## Executive summary

The active local repositories were audited for secrets, authentication, request
boundaries, filesystem access, logging, dependency risk, and deployment defaults.
The application-level findings below were remediated and verified with builds,
tests, and dependency audits. Deployment hardening remains required before
exposing Paperclip services outside a private development network.

## Remediated findings

### SEC-001 — Unauthenticated Paperclip API

- **Severity:** High
- **Location:** `custom-paperclip/src/index.ts:20-64, 149-286`
- **Evidence:** `/api` routes were mounted without authentication.
- **Impact:** Any network client could read shared memory, write decisions/facts,
  route model tasks, and inspect Kafka state.
- **Fix:** Added constant-time bearer-token authentication, fail-closed API
  behavior when no token is configured, per-client rate limiting, security
  headers, and generic client-facing errors.

### SEC-002 — Unbounded and weakly validated Paperclip request bodies

- **Severity:** High
- **Location:** `custom-paperclip/src/index.ts:65-76, 149-264`
- **Evidence:** `express.json()` had no size limit and route fields were passed
  directly to integrations.
- **Impact:** Memory exhaustion and malformed-input propagation into model and
  graph integrations.
- **Fix:** Added a 1 MiB JSON limit, strict parsing, bounded string validation,
  threshold validation, and generic error responses.

### SEC-003 — Default database credential

- **Severity:** Critical
- **Location:** `custom-paperclip/src/index.ts:95-97`, `.env.example`
- **Evidence:** Neo4j previously defaulted to `password` and Compose configured
  `NEO4J_AUTH: none`.
- **Impact:** Unauthorized graph access if the service or database is reachable.
- **Fix:** Application startup now requires `NEO4J_PASSWORD`; the example uses a
  replacement marker. Compose still requires deployment hardening (see open findings).

### SEC-004 — Cross-agent cache key collision and sensitive logging

- **Severity:** Medium
- **Location:** `custom-paperclip/src/integrations/lightrag/agent-api.ts:81-108`
- **Evidence:** Cache keys used only the topic, and logs included query/decision
  content.
- **Impact:** Results could be served across agent scopes and sensitive user
  content could enter logs.
- **Fix:** Cache keys now include encoded agent and topic identifiers; raw query
  and decision content was removed from logs; rate-limit state is bounded.

### SEC-005 — Vulnerable development dependencies

- **Severity:** High
- **Location:** `claudeclaw/package.json`, `custom-paperclip/package.json`
- **Evidence:** Audit reported vulnerable Vitest/Vite and Axios/Express/uuid/
  minimatch chains.
- **Fix:** Upgraded ClaudeClaw to Vitest 4.1.10 and Node 20, and upgraded
  Paperclip's dependency tree including Axios-compatible audit fixes, uuid 14,
  and TypeScript ESLint 8. Both repositories now report zero high-severity audit
  findings.

### SEC-006 — Arbitrary state export path and unsafe config permissions

- **Severity:** Medium
- **Location:** `claudeclaw/src/cli.ts:17-27, 115-119`, `src/config.ts:87-94`
- **Evidence:** CLI accepted any output path and config files used default file
  permissions; exported config could include the Codex API key.
- **Fix:** State exports are restricted to the current working directory and
  written `0600`; config directories are `0700`, config files `0600`, and config
  export redacts `codex.apiKey`.

## Open deployment findings

### SEC-007 — Paperclip Compose exposes unauthenticated infrastructure

- **Severity:** Critical for non-local deployment
- **Location:** `custom-paperclip/docker-compose.yml:7-14, 33-55, 65-75`
- **Evidence:** Neo4j uses `NEO4J_AUTH: none`; database, Kafka, ZooKeeper, and
  Redis ports are published; Kafka uses plaintext listeners.
- **Impact:** Network attackers could access or alter shared memory and message
  infrastructure.
- **Required fix:** Replace `latest` tags with pinned digests, enable Neo4j
  authentication, remove host port publication except explicitly required local
  development ports, enable Kafka/ZooKeeper/Redis authentication or keep them on
  an isolated internal network, and add TLS where traffic crosses hosts.

### SEC-008 — In-process rate limits are not multi-instance protection

- **Severity:** Medium
- **Location:** `custom-paperclip/src/index.ts:22-46`,
  `control-plane/src/security.ts`
- **Required fix:** Put rate limiting at the authenticated edge or use a shared
  store before horizontal scaling.

### SEC-009 — Connector permissions and secret inventory

- **Severity:** High pending verification
- **Location:** GitHub/App deployment configuration, not visible in local source
- **Required fix:** Verify the `febuz` GitHub App installation is scoped only to
  required repositories and rotate any credentials used by local setup scripts.
  No credential values were copied into this report.

## Verification

- `custom-paperclip`: `npm run build`, `npm audit --audit-level=high` — pass.
- `claudeclaw`: `npm run build`, `npm run type-check`, tests, and
  `npm audit --audit-level=high` — pass.
- `control-plane`: tests and `npm audit --audit-level=high` — pass.
