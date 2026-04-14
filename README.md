# CTO Pay

On-chain usage-based payment and settlement engine for the CTO platform, built on Solana with Anchor.

See [`prd.md`](prd.md) for the full product requirements.

## 🎧 Architecture Deliberation

> Listen to the AI-generated architectural deliberation — the intake agents debate design trade-offs, account structures, and settlement strategies for CTO Pay.

[![Listen to Deliberation](https://img.shields.io/badge/🎧_Listen-Deliberation_Audio-blue?style=for-the-badge&logo=github)](https://5dlabs.github.io/cto-pay/)

[▶ **Open the audio player →**](https://5dlabs.github.io/cto-pay/) · [📦 Releases (direct download)](https://github.com/5dlabs/cto-pay/releases)

## Overview

CTO Pay handles:

- **Customer balance accounts** — USDC deposit vaults with on-chain tracking
- **Usage metering** — Agent call recording with per-model token pricing
- **Settlement engine** — Periodic batch settlement (debit customer → credit platform treasury)
- **Admin controls** — Rate updates, pause/resume, account management via multisig

## Development

```bash
# Prerequisites: Rust, Solana CLI, Anchor
anchor build
anchor test
```

## License

See [LICENSE](LICENSE).
