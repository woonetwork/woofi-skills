---
name: woofi-api
version: 1.0.0
description: Query WOOFi DEX API for trading stats, earn yields, staking, user data, perps volume, and integration data across 16 blockchain networks.
homepage: https://fi.woo.org
metadata:
  tags: [defi, dex, trading, analytics, multichain]
---

# WOOFi API Skill

WOOFi is a decentralized exchange aggregator offering deep liquidity across 16 blockchain networks. This skill teaches you how to use the WOOFi public REST API to query trading volume, earn vault yields, WOO staking stats, user portfolios, perpetual trading data, and more.

**Base URLs**:
- Main API: `https://api.woofi.com`
- Swap & Quote API (V2): `https://sapi.woofi.com`
**Authentication**: None required
**Rate Limit**: 5 requests/second
**Response Format**: All responses return `{"status": "ok"|"fail", "data": ...}`

---

## Critical Rules

1. **Wei conversion for volumes**: Most volume/TVL values from stat endpoints are strings in wei (18 decimals). Convert: `actual_usd = value / 10^18`

2. **Yield TVL uses token decimals**: The `/yield` endpoint returns TVL scaled by `10^decimals` (NOT always 18). USDC is typically 6 decimals; USDT is 18 on BSC but 6 elsewhere. Always read the `decimals` field from the response.

3. **Never sum `trader_count` across buckets**: Each time bucket contains unique traders for that window. Summing inflates the count. Use the value from a single bucket or a dedicated aggregate endpoint.

4. **Period format differs by endpoint type**:
   - Stat endpoints (`/stat`, `/source_stat`): `1d`, `1w`, `1m`, `3m`, `1y`, `all`
   - User endpoints (`/user_trading_volumes`, `/user_perp_volumes`): `7d`, `14d`, `30d`

5. **`1d` period returns hourly buckets**; all other periods return daily buckets.

6. **Addresses must be EVM checksummed**: The backend calls `Web3.toChecksumAddress()` but you should provide properly checksummed addresses.

7. **Paused networks**: `fantom`, `zksync`, `polygon_zkevm` are excluded from `/earn_summary`. Data may be stale.

8. **Cross-chain queries**: Most endpoints require one request per network. Exceptions: `/multi_total_stat`, `/user_trading_volumes`, `/earn_summary`, `/stakingv2` aggregate automatically.

9. **Native Token Address**: When querying swaps or quotes for native EVM assets (e.g., ETH, BNB, AVAX), you can use the token symbol (e.g., `"ETH"`, `"BNB"`) or the address `0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE`.

10. **V2 API Flexible Input**: The V2 swap/quote endpoints accept chain names (e.g., `"arbitrum"`) or chain IDs (e.g., `42161`), and token symbols (e.g., `"USDC"`) or addresses. No manual address lookup needed.

---

## Supported Networks

| Network | Key | Chain ID | /stat | /yield | /earn_summary | /user_* | /v2/swap |
|---------|-----|----------|-------|--------|---------------|---------|----------|
| Ethereum | `ethereum` | `1` | Yes | Yes | Yes | Yes | Yes |
| BNB Chain | `bsc` | `56` | Yes | Yes | Yes | Yes | Yes |
| Avalanche | `avax` | `43114` | Yes | Yes | Yes | Yes | Yes |
| Polygon | `polygon` | `137` | Yes | Yes | Yes | Yes | Yes |
| Arbitrum | `arbitrum` | `42161` | Yes | Yes | Yes | Yes | Yes |
| Optimism | `optimism` | `10` | Yes | Yes | Yes | Yes | Yes |
| Linea | `linea` | `59144` | Yes | Yes | Yes | Yes | Yes |
| Base | `base` | `8453` | Yes | Yes | Yes | Yes | Yes |
| Mantle | `mantle` | `5000` | Yes | Yes | Yes | Yes | Yes |
| Sonic | `sonic` | `146` | Yes | Yes | Yes | Yes | Yes |
| Berachain | `berachain` | `80094` | Yes | Yes | Yes | Yes | Yes |
| HyperEVM | `hyperevm` | `999` | Yes | No | No | Yes | Yes |
| Monad | `monad` | `143` | Yes | No | No | Yes | Yes |
| Solana | `solana` | (SVM) | Yes | No | No | No | No |
| Fantom | `fantom` | `250` | Yes | Yes | Paused | Yes | Yes |
| zkSync | `zksync` | `324` | Yes | Yes | Paused | Yes | Yes |
| Polygon zkEVM | `polygon_zkevm` | `1101` | Yes | Yes | Paused | Yes | Yes |

---

## Section 1: Swap Configuration

### GET /swap_support

Returns supported networks, DEXs, and token lists for WOOFi swaps.

| Parameter | Required | Description |
|-----------|----------|-------------|
| (none) | — | — |

**Cache**: 1 hour

**Response**: Dictionary keyed by network name. Each value contains:
- `dexs` — list of supported DEXs
- `network_infos` — chain ID, RPC URL, explorer URL, etc.
- `token_infos` — list of tokens with `address`, `symbol`, `decimals`, `swap_enable`

```
GET https://api.woofi.com/swap_support
```

---

## Section 2: Aggregated Stats

### GET /multi_total_stat

Returns cross-chain aggregated trading statistics. No parameters needed.

**Cache**: 5 seconds

```bash
curl -s "https://api.woofi.com/multi_total_stat"
```

**Response Example**:
```json
{
  "status": "ok",
  "data": {
    "past_24h_volume": "45000000000000000000000000",
    "total_volume_usd": "120000000000000000000000000000",
    "yesterday_volume_usd": "42000000000000000000000000",
    "last_week_volume_usd": "300000000000000000000000000",
    "updated_at": 1711000000
  }
}
```

**Conversion**: `past_24h_volume / 10^18 = 45,000,000 USD`

All volume fields are in wei (18 decimals). Divide by `10^18` to get USD.

---

### GET /total_stat

Returns lifetime totals (volume, traders, txns) for a single network.

| Parameter | Required | Description |
|-----------|----------|-------------|
| `network` | Yes | Network key (e.g., `arbitrum`) |

```
GET https://api.woofi.com/total_stat?network=arbitrum
```

**Response fields**: `total_volume_usd` (wei), `total_traders`, `total_txns`

---

### GET /cumulate_stat

Returns 24-hour cumulative stats including turnover rate.

| Parameter | Required | Description |
|-----------|----------|-------------|
| `network` | Yes | Network key |

```
GET https://api.woofi.com/cumulate_stat?network=arbitrum
```

**Response fields**: `24h_volume_usd` (wei), `24h_traders`, `24h_txns`, `turnover_rate_percentage`

---

## Section 3: Historical Stats

### GET /stat

Returns historical volume, traders, and txns as time-series data points.

| Parameter | Required | Values |
|-----------|----------|--------|
| `network` | Yes | Any supported network |
| `period` | No | `1d`, `1w`, `1m` (default), `3m`, `1y`, `all` |

**Cache**: `1d`=10min, `1w`=30min, `1m`/`3m`/`1y`/`all`=2h

```bash
curl -s "https://api.woofi.com/stat?period=1m&network=arbitrum"
```

**Response Example**:
```json
{
  "status": "ok",
  "data": [
    {
      "timestamp": "2025-03-01T00:00:00Z",
      "volume_usd": "2500000000000000000000000",
      "trader_count": 3200,
      "tx_count": 8500,
      "turnover_rate_percentage": 12.5
    }
  ]
}
```

**Conversion**: `volume_usd / 10^18 = 2,500,000 USD`

Notes:
- `1d` returns hourly buckets; other periods return daily buckets
- `turnover_rate_percentage` is included for daily data only
- Prefer `period=1m` with timestamp filtering over `period=1w` for complete weekly data

---

### GET /source_stat

Returns volume and txns grouped by integrator source (e.g., 1inch, DODO, OKX).

| Parameter | Required | Values |
|-----------|----------|--------|
| `network` | Yes | Any supported network |
| `period` | No | `1d`, `1w`, `1m` (default), `3m`, `1y`, `all` |

```bash
curl -s "https://api.woofi.com/source_stat?period=1m&network=arbitrum"
```

**Response**: Dictionary keyed by source name (e.g., `WOOFi`, `1inch`, `DODO`, `OpenOcean`, `ODOS`, `OKX`). Each contains volume and txn data per time bucket.

Known source mappings: `0`=WOOFi, `1`=1inch, `2`=DODO, `3`=OpenOcean, `12`=ODOS, `15`=OKX. On Solana, long-address sources are merged into `"Other"`.

```
GET https://api.woofi.com/source_stat?period=1m&network=bsc
```

---

### GET /token_stat

Returns 24h stats per token on a network (TVL, volume, turnover).

| Parameter | Required | Description |
|-----------|----------|-------------|
| `network` | Yes | Network key |

```
GET https://api.woofi.com/token_stat?network=arbitrum
```

**Response fields per token**: `symbol`, `decimals`, `logo_url`, `tvl` (wei 18 decimals), `24h_volume_usd` (wei 18 decimals), `24h_txns`, `turnover_rate_percentage`

---

### GET /solana_stat

Returns raw Solana pool statistics from Redis.

| Parameter | Required | Description |
|-----------|----------|-------------|
| `filter_days` | No | Days to look back (default: 14) |

```
GET https://api.woofi.com/solana_stat?filter_days=14
```

---

## Section 4: Earn & Staking

### GET /yield

Returns earn vault TVL, APY, and yield composition for a network.

| Parameter | Required | Description |
|-----------|----------|-------------|
| `network` | Yes | 10 networks only: `bsc`, `avax`, `polygon`, `arbitrum`, `optimism`, `linea`, `base`, `mantle`, `sonic`, `berachain` |

**Not supported**: `solana`, `hyperevm`, `monad`

```bash
curl -s "https://api.woofi.com/yield?network=arbitrum"
```

**Response Example**:
```json
{
  "status": "ok",
  "data": [
    {
      "token": "USDC",
      "apy": 8.45,
      "tvl": "12500000000000",
      "decimals": 6,
      "price": 1.0,
      "loan_percentage": 60.0,
      "reserve_percentage": 40.0,
      "lending_apr": 5.07,
      "reserve_apr": 3.38
    }
  ]
}
```

**TVL Conversion** (CRITICAL — uses `decimals`, not always 18):
```
usd_tvl = tvl / 10^decimals
= 12500000000000 / 10^6 = $12,500,000
```

Decimals vary: USDC=6, USDT=18 on BSC but 6 elsewhere. Always use the `decimals` field from the response.

For global TVL: query all 10 networks, convert each vault's TVL, then sum.

---

### GET /earn_summary

Returns Supercharger vault APR summary across all networks, sorted by APR.

| Parameter | Required | Description |
|-----------|----------|-------------|
| (none) | — | — |

Excludes paused networks (`fantom`, `zksync`, `polygon_zkevm`). Filters out deprecated vaults and 0% APR.

```
GET https://api.woofi.com/earn_summary
```

**Response**: List of vault objects with APR, network, and vault details.

---

### GET /stakingv2

Returns global WOO token staking stats. Chain-agnostic — no parameters needed.

```bash
curl -s "https://api.woofi.com/stakingv2"
```

**Response Example**:
```json
{
  "status": "ok",
  "data": {
    "avg_apr": 18.7,
    "base_apr": 12.4,
    "mp_boosted_apr": 6.3,
    "total_woo_staked": "850000000000000000000000000",
    "decimals": 18
  }
}
```

**APR Formula**: `avg_apr = base_apr + mp_boosted_apr`

The Multiplier Point (MP) system rewards long-term stakers with additional APR.

**Conversion**: `total_woo_staked / 10^18 = 850,000,000 WOO`

---

## Section 5: User Data

All user endpoints require an EVM checksummed address. Not available for Solana.

### GET /user_balances

Returns user's native and ERC20 token balances for all WOOFi-supported tokens on a network.

| Parameter | Required | Description |
|-----------|----------|-------------|
| `network` | Yes | EVM network key |
| `user_address` | Yes | Checksummed EVM address |

```bash
curl -s "https://api.woofi.com/user_balances?network=arbitrum&user_address=0xYourAddress"
```

**Response Example**:
```json
{
  "status": "ok",
  "data": [
    {
      "address": "0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE",
      "symbol": "ETH",
      "decimals": 18,
      "balance": "1500000000000000000"
    }
  ]
}
```

**Conversion**: `balance / 10^decimals = 1.5 ETH`

Uses `Multicall3` for efficient batched RPC calls.

---

### GET /user_trading_volumes

Returns user's swap trading volume across all networks for the given period.

| Parameter | Required | Values |
|-----------|----------|--------|
| `period` | Yes | `7d`, `14d`, `30d` |
| `user_address` | Yes | Checksummed EVM address |

```
GET https://api.woofi.com/user_trading_volumes?period=30d&user_address=0xYourAddress
```

Aggregates automatically across all supported networks.

---

### GET /user_supercharger_infos

Returns user's Supercharger vault positions (deposits, pending withdrawals).

| Parameter | Required | Description |
|-----------|----------|-------------|
| `user_address` | Yes | Checksummed EVM address |

```
GET https://api.woofi.com/user_supercharger_infos?user_address=0xYourAddress
```

---

### GET /user_stakingv2_infos

Returns user's staking info including claimed rewards and expected yearly earnings.

| Parameter | Required | Description |
|-----------|----------|-------------|
| `user_address` | Yes | Checksummed EVM address |
| `network` | No | Default: `arbitrum` |
| `days` | No | Days for earnings calculation (default: 30) |

```
GET https://api.woofi.com/user_stakingv2_infos?user_address=0xYourAddress&network=arbitrum&days=30
```

---

### GET /boosted_apr_info

Checks if user is eligible for boosted APR (new stakers within 60 days of first stake).

| Parameter | Required | Description |
|-----------|----------|-------------|
| `user_address` | Yes | Checksummed EVM address |
| `network` | No | Network for staking lookup |

```
GET https://api.woofi.com/boosted_apr_info?user_address=0xYourAddress
```

**Response fields**: `is_boosted` (boolean), `deadline` (unix timestamp = first_stake_time + 60 days)

---

## Section 6: Perps

### GET /woofi_pro/perps_volume

Returns daily trading volume for the WOOFi Pro broker (last 80 days).

| Parameter | Required | Description |
|-----------|----------|-------------|
| (none) | — | — |

```
GET https://api.woofi.com/woofi_pro/perps_volume
```

Data sourced from Orderly Network API.

---

### GET /user_perp_volumes

Returns user's perpetual trading volume on WOOFi DEX.

| Parameter | Required | Values |
|-----------|----------|--------|
| `period` | Yes | `7d`, `14d`, `30d` |
| `user_address` | Yes | Checksummed EVM address |

```
GET https://api.woofi.com/user_perp_volumes?period=7d&user_address=0xYourAddress
```

---

## Section 7: Integration

These endpoints are designed for third-party integrations (CoinMarketCap, CoinGecko, etc.).

### GET /integration/pairs

Returns supported trading pairs.

| Parameter | Required | Description |
|-----------|----------|-------------|
| (none) | — | — |

**Cache**: 1 hour

```
GET https://api.woofi.com/integration/pairs
```

**Response**: List of `{"ticker_id": "ETH_USDC", "base": "ETH", "target": "USDC"}`

---

### GET /integration/tickers

Returns 24h market data (last price, volume) for all integration pairs.

| Parameter | Required | Description |
|-----------|----------|-------------|
| (none) | — | — |

```
GET https://api.woofi.com/integration/tickers
```

**Response fields per ticker**: `ticker_id`, `last_price`, `base_volume`, `target_volume`

---

### GET /integration/pool_states

Returns internal WooPP pool states including reserves, fees, oracle prices, and coefficients.

| Parameter | Required | Description |
|-----------|----------|-------------|
| `network` | No | Network key (returns all if omitted) |

```bash
curl -s "https://api.woofi.com/integration/pool_states?network=arbitrum"
```

**Response Example**:
```json
{
  "status": "ok",
  "data": {
    "ETH": {
      "reserve": "1000000000000000000000",
      "fee_rate": "0.00025",
      "max_gamma": "0.01",
      "max_notional_swap": "1000000",
      "cap_bal": "5000000000000000000000",
      "price": "3500.50",
      "spread": "0.001",
      "coeff": "0.00000012",
      "wo_feasible": true
    }
  }
}
```

**Key fields**:
- `reserve` — token reserve in pool
- `fee_rate` — trading fee (e.g., 0.025%)
- `price` — current oracle price in USD
- `spread` — bid-ask spread
- `wo_feasible` — whether oracle state is valid for trading

Uses `Multicall3` to batch-call `WooPP.tokenInfos` and `Wooracle.state` contracts.

---

## Section 8: Swap & Quote (V2)

These endpoints provide aggregated token exchange rates and build transactions. **Note: These use the `https://sapi.woofi.com` base URL.**

All V2 endpoints support **flexible input**:
- **Chain**: accepts numeric chain ID (`42161`) or chain name (`"arbitrum"`)
- **Token**: accepts EVM address (`"0xaf88..."`) or token symbol/name (`"USDC"`)

### POST /v2/quote

Returns the current exchange rate and expected receive amount.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `chain` | int \| str | Yes | — | Chain ID or name |
| `sell_token` | str | Yes | — | Token address or symbol |
| `buy_token` | str | Yes | — | Token address or symbol |
| `sell_amount` | str | Yes | — | Amount to sell (human-readable, e.g., `"1.5"`) |
| `slippage_pct` | float | No | 0.5 | Slippage tolerance (%) |
| `woofi_only` | bool | No | false | Restrict to WOOFi pools only |

```bash
curl -X POST "https://sapi.woofi.com/v2/quote" -H "Content-Type: application/json" -d '{"chain": "arbitrum", "sell_token": "USDC", "buy_token": "WETH", "sell_amount": "1000"}'
```

**Response**: `chain`, `sell_token`, `buy_token`, `sell_amount`, `buy_amount`, `price`, `guaranteed_price`

### POST /v2/swap

Gets the quote and generates blockchain-ready transaction data.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `chain` | int \| str | Yes | — | Chain ID or name |
| `sell_token` | str | Yes | — | Token address or symbol |
| `buy_token` | str | Yes | — | Token address or symbol |
| `sell_amount` | str | Yes | — | Amount to sell (human-readable) |
| `to` | str | Yes | — | Recipient address |
| `rebate_to` | str | Yes | — | Rebate recipient address |
| `slippage_pct` | float | No | 0.5 | Slippage tolerance (%) |
| `signer_address` | str \| null | No | null | Signer address (for approve check) |
| `woofi_only` | bool | No | false | Restrict to WOOFi pools only |

**Response**: Includes `needs_approve` (boolean) and `tx_steps` (list of transactions to execute sequentially, such as Approve then Swap). Each tx_step has `to`, `data`, `value`, `chain`, `desc`.

```bash
curl -X POST "https://sapi.woofi.com/v2/swap" -H "Content-Type: application/json" -d '{"chain": "arbitrum", "sell_token": "USDC", "buy_token": "WETH", "sell_amount": "1000", "to": "0xYourWallet", "rebate_to": "0xYourWallet", "signer_address": "0xYourWallet"}'
```

### POST /v2/check_approval

Check if a token approval is needed before swapping.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `chain` | int \| str | Yes | Chain ID or name |
| `sell_token` | str | Yes | Token address or symbol |
| `owner` | str | Yes | User wallet address |
| `sell_amount` | str | Yes | Amount to check (human-readable) |

**Response**: `chain`, `sell_token`, `owner`, `spender`, `current_allowance`, `needs_approval`

---

## Section 9: Cross-Chain Swap (V2)

These endpoints handle cross-chain swaps via Stargate bridge (LayerZero). **Base URL: `https://sapi.woofi.com`**

### POST /v2/cross_chain/quote

Get a price quote for a cross-chain swap.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `src_chain` | int \| str | Yes | — | Source chain ID or name |
| `dst_chain` | int \| str | Yes | — | Destination chain ID or name |
| `src_token` | str | Yes | — | Source token (resolved against `src_chain`) |
| `dst_token` | str | Yes | — | Destination token (resolved against `dst_chain`) |
| `src_amount` | str | Yes | — | Amount to sell (human-readable) |
| `slippage_pct` | float | No | 1.0 | Slippage tolerance (%) |
| `extra_fee_pct` | float | No | 0.0 | Additional fee (%) |

**Response**: `src_chain`, `dst_chain`, `src_token`, `dst_token`, `src_bridge_token`, `dst_bridge_token`, `src_amount`, `net_src_amount`, `bridge_amount_in`, `bridge_amount_out`, `dst_amount`, `price_f`

```bash
curl -X POST "https://sapi.woofi.com/v2/cross_chain/quote" -H "Content-Type: application/json" -d '{"src_chain": "arbitrum", "dst_chain": "base", "src_token": "USDC", "dst_token": "USDC", "src_amount": "100"}'
```

### POST /v2/cross_chain/swap

Build transaction(s) for a cross-chain swap.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `src_chain` | int \| str | Yes | — | Source chain ID or name |
| `dst_chain` | int \| str | Yes | — | Destination chain ID or name |
| `src_token` | str | Yes | — | Source token (resolved against `src_chain`) |
| `dst_token` | str | Yes | — | Destination token (resolved against `dst_chain`) |
| `src_amount` | str | Yes | — | Amount to sell (human-readable) |
| `to` | str | Yes | — | Recipient address on destination chain |
| `slippage_pct` | float | No | 1.0 | Slippage tolerance (%) |
| `extra_fee_pct` | float | No | 0.0 | Additional fee (%) |
| `signer_address` | str \| null | No | null | Signer address (for approve check) |
| `airdrop_native_amount` | int | No | 0 | Native token airdrop on destination (uint128) |

**Response**: All fields from cross_chain/quote, plus `native_fee` (LayerZero fee in source chain native token), `needs_approve`, `tx_steps` (`[approve?, crossSwap]`).

---

## Quick Reference

| # | Endpoint | Method | Key Params | Auto Cross-Chain |
|---|----------|--------|------------|-----------------|
| 1 | `/swap_support` | GET | — | Yes |
| 2 | `/multi_total_stat` | GET | — | Yes |
| 3 | `/total_stat` | GET | `network` | No |
| 4 | `/cumulate_stat` | GET | `network` | No |
| 5 | `/stat` | GET | `network`, `period` | No |
| 6 | `/source_stat` | GET | `network`, `period` | No |
| 7 | `/token_stat` | GET | `network` | No |
| 8 | `/solana_stat` | GET | `filter_days` | N/A |
| 9 | `/yield` | GET | `network` (10 only) | No |
| 10 | `/earn_summary` | GET | — | Yes |
| 11 | `/stakingv2` | GET | — | Yes |
| 12 | `/user_balances` | GET | `network`, `user_address` | No |
| 13 | `/user_trading_volumes` | GET | `period`, `user_address` | Yes |
| 14 | `/user_supercharger_infos` | GET | `user_address` | Yes |
| 15 | `/user_stakingv2_infos` | GET | `user_address`, `network?`, `days?` | No |
| 16 | `/boosted_apr_info` | GET | `user_address`, `network?` | No |
| 17 | `/woofi_pro/perps_volume` | GET | — | N/A |
| 18 | `/user_perp_volumes` | GET | `period`, `user_address` | No |
| 19 | `/integration/pairs` | GET | — | Yes |
| 20 | `/integration/tickers` | GET | — | Yes |
| 21 | `/integration/pool_states` | GET | `network?` | Optional |
| 22 | `/v2/quote` | POST | `chain`, `sell_token`, `buy_token`, `sell_amount` | No |
| 23 | `/v2/swap` | POST | Quote params + `to`, `rebate_to` | No |
| 24 | `/v2/check_approval` | POST | `chain`, `sell_token`, `owner`, `sell_amount` | No |
| 25 | `/v2/cross_chain/quote` | POST | `src_chain`, `dst_chain`, `src_token`, `dst_token`, `src_amount` | Yes |
| 26 | `/v2/cross_chain/swap` | POST | Cross-chain quote params + `to` | Yes |

---

## Error Handling

- **Rate limited**: 5 req/s. Exceeded requests return an error message. Space out batch queries.
- **Invalid network**: Returns `{"status": "fail"}`. Check the supported networks table.
- **Invalid address**: Backend attempts checksum conversion but may fail on malformed addresses.
- **Paused networks**: Endpoints may return stale or empty data for `fantom`, `zksync`, `polygon_zkevm`.
- **Empty data**: Some new networks may have sparse data. Always check array length before processing.
