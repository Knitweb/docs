# Active febuz backlog improvements and Knitweb control plane

**Status:** implementation in progress
**Canonical backlog:** GitHub Issues and Projects  
**Documentation home:** `Knitweb/docs`  
**Control-plane repository:** `Knitweb/control-plane`

**Execution project:** [Nexus Control Plane](https://github.com/orgs/Knitweb/projects/8)  
**Initial security issue:** [control-plane#8](https://github.com/Knitweb/control-plane/issues/8)  
**Molgang source-of-truth issue:** [control-plane#11](https://github.com/Knitweb/control-plane/issues/11)  
**Numerai backlog-mapping issue:** [control-plane#12](https://github.com/Knitweb/control-plane/issues/12)
**Security audit report:** [security-findings-2026-07-23.md](./security-findings-2026-07-23.md)

## Objective

Turn the active febuz repositories into one traceable delivery system without merging their codebases or making any repository depend on another repository's internals.

The resulting developer control plane will show repository health, backlog state, dependencies, agent work, evidence, security findings, and release readiness in one place.

## Active scope

| Repository or surface | Role | Immediate backlog focus |
|---|---|---|
| `febuz/agent-mesh-whitepapers` | specifications and vocabulary | Keep normative claims, citations, and implementation status aligned |
| `febuz/claudeclaw` | TypeScript agent orchestration | Replace MCP, skills, and Codex stubs with authenticated adapters and verification |
| local `custom-paperclip` | orchestration prototype | Reuse useful Kafka/LightRAG/cost ideas through interfaces; repair TypeScript test configuration |
| `febuz/molgang-web` | web game and evidence surface | Make the 0.1 reset baseline authoritative and reconcile sync/review blockers |
| `febuz/molgang-roblox` | Roblox runtime | Resolve the canonical source branch and keep game data/manifests synchronized |
| `febuz/molgang-godot` | open engine target | Preserve the shared arena and chemistry contracts |
| `febuz/molgang-unity` | native AR target | Keep pure chemistry tests portable; isolate Unity Editor-only verification |
| `febuz/numerai-crypto-signals` | current data/model pipeline | Ingest freshness, proof, experiment, and submission evidence without changing live submission behavior |
| `febuz/numerai_signal` | legacy Signals backlog | Map issues to the current pipeline and close only with evidence |
| Knitweb Pulse/Lens/Monitor/docs | substrate, reasoning, observability, and documentation | Read-only adapters and provenance links |

Archived `virtualpc` remains historical context and is not revived by this pass.

## Backlog normalization

Create one GitHub Project named **Nexus Control Plane** with views for `Now`, `Blocked`, `Waiting for Evidence`, `Security`, and `Ready for Release`.

Every active issue must contain:

- one owner and one priority;
- one repository-local acceptance test;
- explicit dependencies using `blocks` or `blocked-by`;
- an evidence link to a commit, test run, report, or reproducible state;
- a security impact field where the change crosses an auth, webhook, upload, execution, data, or deployment boundary.

Use labels consistently: `area:*`, `priority:P0` through `priority:P3`, `kind:bug`, `kind:feature`, `kind:integration`, `kind:security`, `evidence:required`, and `status:blocked`.

Do not recreate existing issues. Preserve issue numbers and add canonical successor links.

### Required issue reconciliation

- Merge the three Molgang synchronization conversations into one canonical cross-repository dependency while preserving the original issue links.
- Link Roblox café/supply-chain issues 8–12 to the Web baseline and the Godot/Unity parity contracts.
- Cross-check legacy `numerai_signal` issues against `numerai-crypto-signals`; do not close anything based only on age or a newer document.
- Create implementation issues for ClaudeClaw MCP transport, Researcher/LightRAG adapters, Codex patch verification, and Skills MCP handlers.
- Create a Paperclip test-harness issue covering `ts-jest`/ESM configuration, integration-test isolation, and service-free test doubles.
- Keep whitepaper citation PRs separate from runtime implementation issues.

## Control-plane implementation

`Knitweb/control-plane` is a public Node 20 + TypeScript service. It contains no private backlog data or credentials.

The first vertical slice is:

```text
GitHub webhook
  -> HMAC verification and bounded JSON parsing
  -> normalized control event
  -> durable owner-only JSONL outbox
  -> authenticated event query
  -> later adapters and dashboard views
```

The versioned event envelope is `knitweb.control.event/v1` and carries source, subject, correlation, idempotency, payload, and provenance fields.

Initial adapters are read-only toward product repositories:

- GitHub issues, pull requests, projects, labels, and workflow runs;
- ClaudeClaw and Paperclip task status over authenticated HTTP;
- Alexander GitNexus and LightRAG structural/semantic evidence;
- Molgang Web, Roblox, Godot, and Unity manifests and review artifacts;
- Numerai freshness, proof, experiment, and submission reports;
- Pulse, Lens, and Monitor health/evidence surfaces.

Kafka is deferred until measured event volume requires it. Knowledge systems remain sovereign and are connected by adapters rather than merged.

## Security workstream

Security is a first-class backlog lane, not a final checklist.

### Implemented in the first control-plane slice

- HMAC-SHA256 verification for GitHub webhooks with constant-time comparison;
- rejection of invalid signatures, non-JSON payloads, malformed JSON, and bodies over 1 MiB;
- authenticated, bounded event inspection;
- no arbitrary outbound URL fetching, shell execution, or dynamic evaluation;
- owner-only outbox file permissions and no outbox/secrets in Git;
- no stack traces or secret values in HTTP responses;
- cache, framing, MIME-sniffing, and referrer-leakage protections on responses.
- trusted-host validation, bounded GitHub delivery identifiers, and fixed-window webhook rate limiting ([control-plane PR #13](https://github.com/Knitweb/control-plane/pull/13)).

### Required security backlog

- Install the GitHub App for the `febuz` account so private repositories can be managed through the connector; until then use a scoped CI credential.
- Extend the current idempotent outbox with replay-window timestamps and a durable delivery ledger.
- Replace the current in-process rate limiter with edge-backed limits for multi-instance deployments; add TLS deployment checks and authenticated operator sessions.
- Add strict schemas for every adapter payload and reject unsafe unknown fields.
- Add dependency scanning, secret scanning, CodeQL, and signed release provenance.
- Audit FastAPI and Next.js surfaces for debug mode, public docs, permissive CORS, missing auth dependencies, unsafe uploads, SSRF, and exposed client secrets.
- Audit ClaudeClaw/Paperclip execution paths for command injection, path traversal, patch authorization, and verification bypasses.
- Apply the deployment-specific Compose hardening recorded in [SEC-007](./security-findings-2026-07-23.md#sec-007--paperclip-compose-exposes-unauthenticated-infrastructure).

## Acceptance criteria

- The control plane accepts a valid GitHub webhook exactly once and rejects invalid/replayed input.
- `/health` is public and minimal; event inspection is authenticated.
- All active backlog items have ownership, priority, dependency, acceptance, evidence, and security classification.
- A Molgang sync blocker and a Numerai proof report can be represented as linked events without changing either product's runtime.
- Existing ClaudeClaw and control-plane builds/tests pass with no secrets in artifacts.
- A failed or stale adapter produces `Blocked` or `Waiting for Evidence`, never false readiness.

## Delivery sequence

1. Merge this documentation update through a PR in `Knitweb/docs`.
2. Merge the initial control-plane scaffold and security baseline.
3. Create the Nexus Control Plane Project and normalized issue templates.
4. Add GitHub, agent, Molgang, Numerai, and knowledge adapters in separate issues/PRs.
5. Add write automation only after authenticated review gates and replay-safe event handling are proven.
