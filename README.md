# CTO Pay

**On-chain usage-based billing and quality-gated agent marketplace for the [CTO](https://github.com/5dlabs/cto) platform, built on Solana with Anchor.**

CTO is an autonomous AI engineering platform by [5D Labs](https://5dlabs.io). CTO Pay moves its billing on chain — every task settlement, every agent payment split, and every receipt is verifiable on Solana. When a customer's task uses a third-party agent package from the marketplace, payment is conditionally split between 5D Labs and the agent author, but *only* if the task passes a quality threshold. If quality isn't met, the customer doesn't pay.

This is not a hackathon toy — it is designed to become the production billing rail that runs alongside (and eventually replaces) traditional payment processing for CTO's usage-based charges.

> **Target:** [Solana Agent Hackathon](https://www.colosseum.org/) — April 2026

[![Listen to Deliberation](https://img.shields.io/badge/🎧_Listen-Architecture_Deliberation-blue?style=for-the-badge&logo=github)](https://5dlabs.github.io/cto-pay/)
[![PRD](https://img.shields.io/badge/📄_Read-PRD-green?style=for-the-badge)](prd.md)

---

## How it works

```
Customer deposits USDC → Escrow PDA
                              │
           CTO runtime runs a task
                              │
                     Task completes
                              │
              ┌───────────────┴───────────────┐
              │                               │
      Default agent only              Public agent used
              │                               │
     100% → Operator treasury     Quality gate check
                                              │
                                    ┌─────────┴─────────┐
                                    │                   │
                               Quality met         Quality NOT met
                                    │                   │
                           Split payment:          No charge —
                        Author gets their cut    customer keeps funds
                        Operator gets remainder
                                    │
                              TaskReceipt written on-chain
                              Receipt blob → Arweave via Irys
```

### Core primitives

| On-chain account | Purpose |
|---|---|
| **Customer Balance (Escrow PDA)** | USDC deposit vault. Only the operator wallet can debit; the customer can withdraw unused balance at any time. |
| **Agent Package PDA** | Registered by any author (permissionless). Stores author wallet, split percentage (basis points), and a reference to the package source (repo URL, Arweave URI). |
| **Task Receipt PDA** | Immutable record of every settlement: task ID, amount, customer, author split, quality verdict, receipt hash. |
| **Operator Config** | Platform-wide settings: operator wallet, treasury, fee schedule, pause state. Multisig-controlled. |

### Settlement cases

1. **No public agent** — 100% of the billable amount goes to the operator treasury.
2. **Public agent, quality met** — payment splits between the agent author and operator per the package's `author_split_bps`.
3. **Public agent, quality NOT met** — zero charge. Customer balance untouched. Receipt still written (with `quality_met: false`) for auditability.

### Design principles

- **Decentralized-first** — Arweave/IPFS over S3, wallet-based identity over OAuth, on-chain receipts over database records.
- **Verifiable by default** — every economic event produces an independently verifiable artifact.
- **Composable over monolithic** — small, well-defined instructions that external programs can compose.

## Architecture

```
┌─────────────────────────────────────────────────────┐
│  CTO Controller (Rust/Axum)                         │
│  ├── Settlement hook → signs settle_task ix          │
│  └── Receipt uploader → Arweave via Irys            │
├─────────────────────────────────────────────────────┤
│  Solana Program (Anchor)                            │
│  ├── initialize_operator        init platform       │
│  ├── deposit / withdraw         customer escrow     │
│  ├── register_agent_package     marketplace entry   │
│  ├── settle_task                billing + splits    │
│  └── update_rates / pause       admin controls      │
├─────────────────────────────────────────────────────┤
│  On-chain state                                     │
│  ├── OperatorConfig PDA                             │
│  ├── CustomerBalance PDA (per customer)             │
│  ├── AgentPackage PDA (per author × package)        │
│  └── TaskReceipt PDA (per settlement)               │
└─────────────────────────────────────────────────────┘
```

## Task breakdown

The [intake pipeline](https://github.com/5dlabs/cto) deliberated the architecture and decomposed the work into 10 tasks with 64 subtasks:

| # | Task | Agent |
|---|------|-------|
| 1 | Bootstrap Solana Dev Infrastructure & Anchor Workspace | Bolt |
| 2 | Anchor Program Core — Operator, Customer & Escrow Instructions | Rex |
| 3 | Anchor Program Marketplace — Agent Packages, Settlement & Receipts | Rex |
| 4 | Arweave/Irys Receipt Upload Module | Nova |
| 5 | Integrate Solana Settlement Hook into CTO Controller | Rex |
| 6 | Hackathon Demo CLI Script — Full E2E Flow | Nova |
| 7 | Web Dashboard for Receipts, Balances & Agent Packages | Blaze |
| 8 | Comprehensive Integration & E2E Test Suite | Tess |
| 9 | Security Review — Authority, Overflow & Escrow Safety | Cipher |
| 10 | CI/CD Pipeline & Devnet Deployment Automation | Atlas |

See [`.tasks/`](.tasks/) for the full artifact set: per-task prompts, acceptance criteria, and lobster workflows.

## 🎧 Architecture deliberation

Listen to the AI-generated architectural deliberation — intake agents debate design trade-offs, account structures, settlement strategies, and marketplace economics.

[▶ **Open the audio player →**](https://5dlabs.github.io/cto-pay/) · [📦 Releases (direct download)](https://github.com/5dlabs/cto-pay/releases)

## Development

```bash
# Prerequisites: Rust 1.79+, Solana CLI 2.x, Anchor 0.31+
anchor build
anchor test
```

## Links

- [**PRD**](prd.md) — full product requirements document
- [**Design Brief**](.tasks/docs/design-brief.md) — detailed architecture decisions from the deliberation process
- [**CTO Platform**](https://github.com/5dlabs/cto) — the parent autonomous engineering platform
- [**5D Labs**](https://5dlabs.io) — the team behind CTO

## License

See [LICENSE](LICENSE).
