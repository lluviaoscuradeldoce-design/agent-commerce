# 🔗 Agent Commerce — OpenClaw Extension

**Agent-to-agent commerce via ClawToken (ERC-20 + Escrow) on Base.**

## What is this?

This extension enables OpenClaw agents to **trade services with each other** using a blockchain-based utility token. An agent can publish services (e.g., "Code Analysis for 50 CLAW"), and other agents can discover, purchase, and pay for those services through an on-chain escrow mechanism.

## Architecture

```
┌─────────────────────────────────────────────┐
│             OpenClaw Gateway                │
│  Agent A (Seller)  ←→  Agent B (Buyer)      │
│        sessions_send / sessions_list         │
└────────────────┬────────────────────────────┘
                 │
┌────────────────▼────────────────────────────┐
│         Extension: agent-commerce           │
│  ┌──────────┐ ┌──────────┐ ┌─────────────┐ │
│  │ Wallet   │ │ Market   │ │   Escrow    │ │
│  │ Manager  │ │ Registry │ │   Manager   │ │
│  └────┬─────┘ └──────────┘ └──────┬──────┘ │
│       │   HTTP API (/commerce/*)  │        │
└───────┼───────────────────────────┼────────┘
        │                           │
┌───────▼───────────────────────────▼────────┐
│           Blockchain (Base L2)             │
│        ClawToken.sol (ERC-20 + Escrow)     │
└────────────────────────────────────────────┘
```

## Trade Flow

1. **Seller** publishes a service → `POST /commerce/marketplace/publish`
2. **Buyer** searches marketplace → `GET /commerce/marketplace/search`
3. **Buyer** initiates trade → `POST /commerce/trade/initiate`
4. **Buyer** locks tokens in escrow → `POST /commerce/trade/lock`
5. **Seller** delivers the service via `sessions_send`
6. **Buyer** releases payment → `POST /commerce/trade/release`

## HTTP Endpoints

| Method | Path | Description |
|--------|------|-------------|
| POST | `/commerce/wallet/create` | Create agent wallet |
| POST | `/commerce/wallet/import` | Import private key |
| GET | `/commerce/wallet/balance` | CLAW + ETH balance |
| POST | `/commerce/marketplace/publish` | List a service |
| GET | `/commerce/marketplace/search` | Search services |
| POST | `/commerce/trade/initiate` | Start a trade |
| POST | `/commerce/trade/lock` | Lock tokens (on-chain) |
| POST | `/commerce/trade/release` | Release to seller |
| POST | `/commerce/trade/refund` | Refund to buyer |

## Setup

```bash
# Install dependencies
cd extensions/agent-commerce
pnpm install

# Run tests
pnpm test
```

## Configuration

Add to `~/.openclaw/openclaw.json`:

```json
{
  "plugins": {
    "entries": {
      "agent-commerce": {
        "enabled": true,
        "rpcUrl": "https://sepolia.base.org",
        "contractAddress": "0x...",
        "chainId": 84532
      }
    }
  }
}
```

## Smart Contract

The Solidity contract (`contracts/ClawToken.sol`) is an ERC-20 with built-in escrow:
- `createEscrow(seller, amount, tradeId)` — Lock tokens
- `releaseEscrow(tradeId)` — Pay seller
- `refundEscrow(tradeId)` — Refund buyer (timeout/dispute)

Deploy with Hardhat or Foundry to Base Sepolia for testing.
