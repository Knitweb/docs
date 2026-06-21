# knitweb/gither — repository instructions

**What it is.** A peer-to-peer, serverless Git layer for Knitweb agentic engineering. Gither lets teams own their source history, collaborate without a central forge, and still plug into familiar IDE workflows (VS Code, Cursor, PyCharm).

## Core idea
- **Git semantics, no central server.** Every peer keeps the full DAG; sync happens over libp2p.
- **CRDT metadata.** Issues, pull requests and review comments merge deterministically without a single source of truth.
- **Optional serverless CI.** `gither.yml` triggers builds, tests and linting as short-lived functions—verifiable by any peer.
- **IDE-native.** Extensions for VS Code/Cursor and JetBrains make Gither feel like a normal SCM provider.

## Relation to Knitweb
Gither is the **code-and-provenance layer** of the Knitweb stack. It stores repository history, signs artifacts, and feeds verifiable build artifacts into Pulse / Fiber / Lens.

## Key documents
- [`launch-plan.md`](launch-plan.md) — how we take Gither from prototype to launched product.
- [`commercial-strategy.md`](commercial-strategy.md) — how Gither can earn revenue without becoming "just another GitHub".
- [Academic paper (source)](https://github.com/Knitweb/gither/issues/18) — the research foundation.
- [Backlog board](https://github.com/orgs/Knitweb/projects/4) — implementation issues derived from the paper.
