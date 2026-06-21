# Gither Launch Plan

This plan describes how to move Gither from a research-backed prototype to a launched product. It is intentionally separated from funding or subsidy discussions; those live outside this repository.

## 1. Product Positioning

**Gither is not "GitHub without servers."** GitHub and GitLab are unmistakably *company-shaped*: they host your code, control your namespace, enforce their policies, and bill per seat. Gither is *protocol-shaped*:

- Your repository lives on your machine first.
- Collaboration is peer-to-peer; hosting is optional, not mandatory.
- Verifiability replaces trust in a platform.
- Serverless functions replace always-on CI infrastructure.

**One-line pitch:** *Gither is the sovereign forge for teams that want GitHub-like workflows without surrendering their code to a platform.*

## 2. Target Audiences

| Segment | Pain | Why Gither fits |
|---|---|---|
| **Privacy-heavy teams** | Cannot put source on public clouds | Local-first, encrypted P2P sync |
| **Open-source collectives** | Fear takedown, account bans, or policy shifts | Censorship-resistant replication |
| **AI-agent teams** | Need verifiable provenance for generated code | Signed commits + CRDT history |
| **Remote / low-connectivity teams** | Intermittent internet, high latency | Offline-first, eventual sync |
| **Compliance-heavy enterprises** | Must prove custody and audit history | Content-addressable, signed artifacts |
| **Independent developers** | Want to self-host without ops overhead | Optional managed relays, no mandatory server |

## 3. Launch Milestones

### Phase 0 — Foundation (0–3 months)
- [ ] Reference CLI works: `init`, `status`, `commit`, `log`, `clone`, `push`, `pull`.
- [ ] SHA-256 object store validated against Git semantics.
- [ ] libp2p transport and mDNS/LAN discovery functional.
- [ ] Public alpha repo: `Knitweb/gither` with install instructions.

### Phase 1 — Early Adopters (3–6 months)
- [ ] VS Code extension in the marketplace (manual install side-load first).
- [ ] CRDT-based issues and lightweight pull requests.
- [ ] Public beta with 5–10 pilot teams.
- [ ] First case study: open-source collective or AI-agent team.

### Phase 2 — Product-Market Fit (6–12 months)
- [ ] Cursor and JetBrains plugins.
- [ ] Managed relay service for NAT traversal and always-on seeds.
- [ ] `gither.yml` serverless CI with local and cloud runners.
- [ ] Public pricing page for managed services.

### Phase 3 — Scale (12–24 months)
- [ ] Marketplace for verified serverless functions.
- [ ] Enterprise SSO, audit logs, compliance reports.
- [ ] Migration wizard from GitHub/GitLab.
- [ ] Gither Cloud: optional hosted namespace for public projects.

## 4. Packaging and Pricing (high-level)

| Tier | What you get | Commercial hook |
|---|---|---|
| **Gither Core** | CLI, P2P sync, IDE extensions, CRDT metadata | Free, open source — drives adoption |
| **Gither Relay** | Managed bootstrap/relay nodes, NAT traversal, uptime SLA | SaaS subscription per team |
| **Gither Build** | Managed serverless CI runners, caching, build attestation | Usage-based + seat bundle |
| **Gither Enterprise** | SSO, audit logs, compliance reports, on-prem relay option | Annual contract |

## 5. Go-to-Market Channels

1. **Developer communities.** Hacker News, Lobsters, relevant subreddits, FOSDEM, local meetups.
2. **Open-source collectives.** Radicle, Fossil, and IPFS communities already understand the problem.
3. **AI/ML toolchains.** Position as the verifiable provenance layer for AI-generated code.
4. **Privacy/compliance consultancies.** Sell through channels that already advise on data sovereignty.
5. **IDE marketplaces.** VS Code, Cursor, JetBrains plugin directories.
6. **Content.** Whitepapers, benchmark reports, migration guides, YouTube demos.

## 6. Risks and Mitigations

| Risk | Mitigation |
|---|---|
| Users expect GitHub UX parity | Ship IDE integrations first; never ask users to abandon their editor |
| P2P is harder to explain than SaaS | Lead with "offline-first Git" and "own your code" |
| Large repo performance | Packfiles, partial replication, optional IPFS for big blobs |
| Sybil/spam in open networks | Reputation + signed refs; public relays can require invites/deposits |
| Enterprise sales cycle is long | Land with free Core, expand with Relay/Build/Enterprise |

## 7. Success Metrics

- **Adoption:** GitHub stars, CLI installs, active peers per week, IDE extension installs.
- **Engagement:** Repositories synced, commits signed, CRDT issues created.
- **Revenue:** Relay subscriptions, Build minutes, Enterprise contracts.
- **Trust:** Security audits passed, migration success rate, NPS from pilot teams.

## 8. Launch Checklist

- [ ] Core CLI installable from `pip` / `cargo` / Homebrew.
- [ ] Public docs site at `knitweb.github.io/gither`.
- [ ] Landing page with clear positioning vs. GitHub/GitLab.
- [ ] Demo video: 3-minute "clone without a server" walkthrough.
- [ ] Beta signup form for managed Relay/Build tiers.
- [ ] Issue board groomed and milestones visible.
- [ ] First security review published.
