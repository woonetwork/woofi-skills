# woofi-skills

A Claude Code plugin providing skills and commands for querying the [WOOFi Swap API](https://api.woofi.com).

## Overview

WOOFi Swap is a decentralized exchange aggregator offering deep liquidity and competitive pricing across 13 blockchain networks. This plugin gives Claude the ability to query WOOFi's public API to answer questions about trading volume, integrator sources, earn vault yields, and WOO staking.

**API Base URL**: `https://api.woofi.com`
**Authentication**: None required
**Rate Limit**: 120 requests/minute

---

## Installation

```bash
/plugin install woofi/woofi-skills
```

Or from a local path during development:

```bash
cc --plugin-dir /path/to/woofi-skills
```

---

## Skills

Skills are invoked automatically when Claude detects relevant trigger phrases in your conversation.

### `trading-stats`
Query trading volume, trader counts, and transaction counts by time period and network.

**Trigger phrases**: "trading stats", "trading volume", "woofi stats", "how much volume", "trader count"

**Endpoint**: `GET /stat?period=<period>&network=<network>`

---

### `volume-by-source`
Break down trading volume by integrator or aggregator source.

**Trigger phrases**: "volume by source", "top integrators", "who is sending volume", "traffic source"

**Endpoint**: `GET /source_stat?period=<period>&network=<network>`

---

### `earn-tvl`
Query WOOFi Earn vault TVL, APY, and yield composition.

**Trigger phrases**: "earn tvl", "earn vault", "yield farming", "woofi earn", "tvl", "apy"

**Endpoint**: `GET /yield?network=<network>`

---

### `woo-staking`
Get global WOO token staking statistics including base APR and Multiplier Point boost.

**Trigger phrases**: "woo staking", "staking apr", "staked woo", "multiplier points"

**Endpoint**: `GET /stakingv2`

---

## Commands

### `/woofi-query <query>`

A general-purpose command for querying any WOOFi API endpoint. Describe what you want to know and Claude will select the correct endpoint, fetch the data, and present results with proper unit conversions.

**Examples**:
```
/woofi-query what is the trading volume on arbitrum for the past month?
/woofi-query show me the top 5 integrators by volume on BSC last week
/woofi-query what is the current APY for earn vaults on base?
/woofi-query how much WOO is staked and what is the current APR?
```

---

## Supported Networks

| Network | /stat | /source_stat | /yield | /stakingv2 |
|---------|-------|--------------|--------|------------|
| BSC | Yes | Yes | Yes | — |
| Avalanche | Yes | Yes | Yes | — |
| Polygon | Yes | Yes | Yes | — |
| Arbitrum | Yes | Yes | Yes | — |
| Optimism | Yes | Yes | Yes | — |
| Linea | Yes | Yes | Yes | — |
| Base | Yes | Yes | Yes | — |
| Mantle | Yes | Yes | Yes | — |
| Sonic | Yes | Yes | Yes | — |
| Berachain | Yes | Yes | Yes | — |
| HyperEVM | Yes | Yes | No | — |
| Monad | Yes | Yes | No | — |
| Solana | Yes | Yes | No | — |
| (global) | — | — | — | Yes |

---

## Key Notes

- **Wei conversion**: All volume and TVL values are returned in wei. Divide by `10^18` (or `10^decimals`) to get human-readable amounts.
- **Trader counts**: Do NOT sum `trader_count` across time buckets — each bucket is independently unique-per-period.
- **Period `1d`**: Returns hourly buckets. All other periods return daily buckets.
- **Cross-chain queries**: Require separate requests per network, then manual aggregation.

---

## License

MIT — Copyright 2026 WOOFi
