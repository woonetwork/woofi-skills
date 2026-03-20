# woofi-skills

A Claude Code plugin providing skills and commands for querying the [WOOFi Swap API](https://api.woofi.com).

## Overview

WOOFi Swap is a decentralized exchange aggregator offering deep liquidity and competitive pricing across 13 blockchain networks. This plugin gives Claude the ability to query WOOFi's public API to answer questions about trading volume, integrator sources, earn vault yields, WOO staking, user portfolios, perpetual trading, and more.

**API Base URL**: `https://api.woofi.com`
**Authentication**: None required
**Rate Limit**: 5 requests/second

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

### `swap-support`
Query supported networks, DEXs, and tokens for WOOFi swaps.

**Trigger phrases**: "swap support", "supported networks", "supported tokens", "which chains"

**Endpoint**: `GET /swap_support`

---

### `aggregated-stats`
Query cross-chain aggregated trading statistics including total volume, 24h stats, and per-network totals.

**Trigger phrases**: "total volume", "total stats", "cross chain stats", "24h stats", "overall volume"

**Endpoints**: `GET /multi_total_stat`, `GET /total_stat`, `GET /cumulate_stat`

---

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

### `token-stats`
Query per-token 24-hour trading statistics including TVL, volume, and turnover rate.

**Trigger phrases**: "token stats", "token volume", "token tvl", "top tokens"

**Endpoint**: `GET /token_stat?network=<network>`

---

### `solana-stats`
Query Solana-specific raw pool statistics.

**Trigger phrases**: "solana stats", "solana pool", "solana trading"

**Endpoint**: `GET /solana_stat`

---

### `earn-tvl`
Query WOOFi Earn vault TVL, APY, and yield composition.

**Trigger phrases**: "earn tvl", "earn vault", "yield farming", "woofi earn", "tvl", "apy"

**Endpoint**: `GET /yield?network=<network>`

---

### `earn-summary`
Query Supercharger vault APR summary across all networks, sorted by APR.

**Trigger phrases**: "earn summary", "supercharger", "earn apr", "best earn vault", "vault ranking"

**Endpoint**: `GET /earn_summary`

---

### `woo-staking`
Get global WOO token staking statistics including base APR and Multiplier Point boost.

**Trigger phrases**: "woo staking", "staking apr", "staked woo", "multiplier points"

**Endpoint**: `GET /stakingv2`

---

### `user-portfolio`
Query user portfolio data including token balances, Supercharger positions, staking info, and boosted APR status.

**Trigger phrases**: "user balance", "user portfolio", "my position", "user staking", "boosted apr"

**Endpoints**: `GET /user_balances`, `GET /user_supercharger_infos`, `GET /user_stakingv2_infos`, `GET /boosted_apr_info`

---

### `user-trading`
Query user swap and perpetual trading volume history.

**Trigger phrases**: "user trading volume", "my volume", "user perp volume", "trading history"

**Endpoints**: `GET /user_trading_volumes`, `GET /user_perp_volumes`

---

### `perps-volume`
Query WOOFi Pro daily perpetual trading volume.

**Trigger phrases**: "perps volume", "woofi pro", "woofi dex", "perpetual volume"

**Endpoint**: `GET /woofi_pro/perps_volume`

---

### `integration-data`
Query trading pairs, 24-hour ticker data, and pool states for third-party integrations.

**Trigger phrases**: "trading pairs", "ticker data", "pool state", "integration pairs", "fee rate"

**Endpoints**: `GET /integration/pairs`, `GET /integration/tickers`, `GET /integration/pool_states`

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
/woofi-query what are the supported tokens on arbitrum?
/woofi-query show me the cross-chain total volume
/woofi-query what are the pool states on arbitrum?
/woofi-query show me WOOFi Pro perps volume trends
```

---

## Supported Networks

| Network | /stat | /source_stat | /token_stat | /yield | /earn_summary | /stakingv2 | /user_* | /integration |
|---------|-------|--------------|-------------|--------|---------------|------------|---------|-------------|
| BSC | Yes | Yes | Yes | Yes | Yes | — | Yes | Yes |
| Avalanche | Yes | Yes | Yes | Yes | Yes | — | Yes | Yes |
| Polygon | Yes | Yes | Yes | Yes | Yes | — | Yes | Yes |
| Arbitrum | Yes | Yes | Yes | Yes | Yes | — | Yes | Yes |
| Optimism | Yes | Yes | Yes | Yes | Yes | — | Yes | Yes |
| Linea | Yes | Yes | Yes | Yes | Yes | — | Yes | Yes |
| Base | Yes | Yes | Yes | Yes | Yes | — | Yes | Yes |
| Mantle | Yes | Yes | Yes | Yes | Yes | — | Yes | Yes |
| Sonic | Yes | Yes | Yes | Yes | Yes | — | Yes | Yes |
| Berachain | Yes | Yes | Yes | Yes | Yes | — | Yes | Yes |
| HyperEVM | Yes | Yes | Yes | No | — | — | Yes | Yes |
| Monad | Yes | Yes | Yes | No | — | — | Yes | Yes |
| Solana | Yes | Yes | Yes | No | — | — | No | — |
| Fantom | Yes | Yes | Yes | Yes | Paused | — | Yes | Yes |
| zkSync | Yes | Yes | Yes | Yes | Paused | — | Yes | Yes |
| Polygon zkEVM | Yes | Yes | Yes | Yes | Paused | — | Yes | Yes |
| (global) | — | — | — | — | — | Yes | — | — |

---

## Key Notes

- **Wei conversion**: All volume and TVL values are returned in wei. Divide by `10^18` (or `10^decimals`) to get human-readable amounts.
- **Trader counts**: Do NOT sum `trader_count` across time buckets — each bucket is independently unique-per-period.
- **Period `1d`**: Returns hourly buckets. All other periods return daily buckets.
- **User endpoints**: Period values are `7d`, `14d`, `30d` (different from stat endpoints).
- **Cross-chain queries**: Require separate requests per network, then manual aggregation. Exception: `/multi_total_stat` and `/user_trading_volumes` aggregate automatically.
- **Address format**: All EVM addresses should be checksummed.
- **Paused networks**: `fantom`, `zksync`, `polygon_zkevm` are excluded from `/earn_summary`.

---

## License

MIT — Copyright 2026 WOOFi
