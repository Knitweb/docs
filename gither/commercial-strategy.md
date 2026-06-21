# Gither Commercial Strategy

GitHub and GitLab are *company-shaped*: central hosting, seat-based billing, platform policies, and a single namespace. That shape is their commercial engine, but also their weakness. Gither's commercial opportunity is to be *protocol-shaped* and monetize around convenience, verification, and enterprise assurance—not around owning the code.

## 1. The Core Principle: Free the Protocol, Sell the Service

Gither Core is free and open source. It must be trivial to run without paying anyone. Revenue comes from *optional* services that make the protocol more convenient, verifiable, or enterprise-ready.

This is the same shape that made Git successful: Linus open-sourced the protocol; GitHub later monetized convenience. Gither short-circuits that history by designing the monetization layer into the protocol from the start.

## 2. Revenue Models

### 2.1 Managed Relays (SaaS)

**What it is.** Optional, always-on libp2p relay and bootstrap nodes that help peers find each other and traverse NATs.

**Why teams pay.** You do not *need* a relay, but without one you must keep a machine online or rely on random peers. A managed relay is the easiest way to guarantee availability.

**Pricing signal.** Per-team subscription, tiered by number of repositories and peers.

### 2.2 Serverless Build/CI (Usage-Based)

**What it is.** Managed runners that execute `gither.yml` functions: build, test, lint, sign, attest.

**Why teams pay.** Local runners are free, but teams want reproducible builds, caching, and attestation without maintaining infrastructure.

**Pricing signal.** Per-build-minute plus storage for attestation artifacts.

### 2.3 Gither Cloud Namespace (Freemium Hosting)

**What it is.** Optional, centralized *view* of public repositories, similar to a read-only forge index. The actual repositories remain P2P; Cloud only provides search, discovery, and a web UI.

**Why teams pay.** Public projects want visibility and searchability; private projects want a branded namespace without surrendering the repository.

**Pricing signal.** Free for public repos; paid for private namespaces and custom domains.

### 2.4 Enterprise Assurance

**What it is.** SSO, audit logs, compliance reports, on-prem relay appliances, signed SBOMs, and attestation chains.

**Why teams pay.** Regulated industries need evidence of custody, not just a tool.

**Pricing signal.** Annual contracts, priced by organization size.

### 2.5 Verified Marketplace for Functions

**What it is.** A curated marketplace of reusable `gither.yml` functions: security scanners, license checkers, deployment hooks, AI review agents.

**Why teams pay.** Verified, maintained functions save time and reduce supply-chain risk.

**Pricing signal.** Revenue share with function authors; Gither takes a percentage.

### 2.6 Training and Certification

**What it is.** Courses and certifications for Gither administration, CRDT-based collaboration, and secure P2P development.

**Why teams pay.** Enterprises prefer certified tooling and trained staff.

**Pricing signal.** Per-seat course fees and certification exams.

## 3. Why This Avoids the "GitHub Trap"

| GitHub/GitLab shape | Gither shape |
|---|---|
| Code lives on their servers | Code lives on your machine |
| Namespace is theirs | Identity is cryptographic |
| Billing is per seat | Billing is per convenience/service |
| Policies decide who can publish | Protocol decides what is valid |
| Lock-in through network effects | Lock-in through workflow value, not data hostage |

By making the central platform *optional*, Gither keeps the moral high ground: users pay because they want the service, not because they have no alternative.

## 4. Open-Core Boundary

| Free (Core) | Paid (Commercial) |
|---|---|
| CLI, P2P sync, CRDT metadata, IDE extensions | Managed relays |
| Local serverless execution | Managed CI runners |
| Signed refs and verification primitives | Compliance dashboards and attestation APIs |
| Community functions | Verified marketplace functions |
| Public documentation | Training and certification |

## 5. Competitive Positioning

| Competitor | Their strength | Gither counter-position |
|---|---|---|
| **GitHub** | Social coding, network effects | Sovereignty, offline-first, no platform risk |
| **GitLab** | All-in-one DevOps | Lightweight, serverless CI, no server sprawl |
| **Radicle** | Fully P2P, crypto-native | IDE-native, serverless CI, enterprise-friendly |
| **Fossil** | Self-contained, simple | Modern P2P stack, libp2p, agentic provenance |
| **SourceHut** | Minimalist, no JS | Local-first, CRDT metadata, optional cloud |

## 6. Go-to-Market Sequence

1. **Credibility first.** Publish the academic paper, security model, and benchmarks.
2. **CLI and VS Code.** Capture developers where they already work.
3. **Open-source collectives.** These users feel the platform-risk pain most acutely.
4. **AI-agent teams.** Sell verifiable provenance as a feature for compliance-minded AI workflows.
5. **Enterprise pilots.** Use compliance and audit requirements as the wedge.

## 7. Anti-Patterns to Avoid

- **Do not** make the free version artificially limited in repository size or peer count.
- **Do not** require an account to use the CLI.
- **Do not** route all traffic through Gither-owned infrastructure.
- **Do not** mint a token unless it solves a real coordination problem.
- **Do not** imitate GitHub's UI; imitate the developer's workflow.

## 8. Summary

Gither's commercial model is: **open protocol, paid convenience, enterprise assurance.** The product is free to use; the business earns money by making the protocol easier, faster, and safer for teams that outgrow DIY operations. This keeps Gither aligned with its users: it wins when sovereignty is practical, not when lock-in is inescapable.
