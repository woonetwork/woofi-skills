---
description: Query WOOFi Swap API for trading stats, volume by source, earn TVL, WOO staking, user portfolio, perps volume, integration data, and more
argument-hint: <query description>
allowed-tools: [WebFetch, Bash]
---

# WOOFi Query Command

You are helping a user query the WOOFi Swap API at `https://api.woofi.com`. Based on the user's request, identify which endpoint(s) to call, fetch the data, and present the results clearly.

## User Request

$ARGUMENTS

---

## Step 1: Identify the Endpoint

Determine which WOOFi API endpoint to use based on the request:

| Request Type | Endpoint | Required Parameters |
|---|---|---|
| Supported networks, DEXs, tokens | `GET /swap_support` | none |
| Cross-chain total volume & stats | `GET /multi_total_stat` | none |
| Per-network lifetime stats | `GET /total_stat` | `network` |
| 24h cumulative stats & turnover | `GET /cumulate_stat` | `network` |
| Historical trading stats by period | `GET /stat` | `period`, `network` |
| Volume breakdown by integrator | `GET /source_stat` | `period`, `network` |
| Per-token 24h stats | `GET /token_stat` | `network` |
| Solana raw pool stats | `GET /solana_stat` | (optional: `filter_days`) |
| Earn vault TVL, APY, yield | `GET /yield` | `network` |
| Supercharger vault APR summary | `GET /earn_summary` | none |
| WOO token staking APR & total staked | `GET /stakingv2` | none |
| User token balances | `GET /user_balances` | `network`, `user_address` |
| User swap trading volumes | `GET /user_trading_volumes` | `period`, `user_address` |
| User Supercharger positions | `GET /user_supercharger_infos` | `user_address` |
| User staking info & rewards | `GET /user_stakingv2_infos` | `user_address`, (optional: `network`, `days`) |
| User boosted APR eligibility | `GET /boosted_apr_info` | `user_address`, (optional: `network`) |
| WOOFi Pro daily perps volume | `GET /woofi_pro/perps_volume` | none |
| User perp trading volumes | `GET /user_perp_volumes` | `period`, `user_address` |
| Trading pairs list | `GET /integration/pairs` | none |
| 24h ticker data (prices, volume) | `GET /integration/tickers` | none |
| Pool states (reserves, fees, oracle) | `GET /integration/pool_states` | (optional: `network`) |

If the request is ambiguous, ask the user to clarify before making API calls.

---

## Step 2: Validate Parameters

### Period values
- For `/stat` and `/source_stat`: `1d` (hourly), `1w`, `1m`, `3m`, `1y`, `all` (daily)
- For `/user_trading_volumes` and `/user_perp_volumes`: `7d`, `14d`, `30d`

### Network values
`bsc`, `avax`, `fantom`, `polygon`, `arbitrum`, `optimism`, `zksync`, `polygon_zkevm`, `linea`, `base`, `mantle`, `sonic`, `berachain`, `hyperevm`, `monad`, `solana`

Notes:
- `/yield` only supports 10 networks — excludes `solana`, `hyperevm`, `monad`
- User endpoints are EVM only — excludes `solana`
- Paused networks (`fantom`, `zksync`, `polygon_zkevm`) are excluded from `/earn_summary`

If the user did not specify a required parameter, ask before proceeding.

---

## Step 3: Fetch Data

Construct the URL and fetch using WebFetch. Examples:

```
# Swap support
https://api.woofi.com/swap_support

# Cross-chain totals
https://api.woofi.com/multi_total_stat

# Per-network totals
https://api.woofi.com/total_stat?network=arbitrum

# 24h cumulative
https://api.woofi.com/cumulate_stat?network=arbitrum

# Historical trading stats
https://api.woofi.com/stat?period=1m&network=arbitrum

# Volume by source
https://api.woofi.com/source_stat?period=1m&network=arbitrum

# Per-token stats
https://api.woofi.com/token_stat?network=arbitrum

# Earn TVL
https://api.woofi.com/yield?network=arbitrum

# Earn summary
https://api.woofi.com/earn_summary

# WOO staking
https://api.woofi.com/stakingv2

# User balances
https://api.woofi.com/user_balances?network=arbitrum&user_address=0x...

# User swap volumes
https://api.woofi.com/user_trading_volumes?period=30d&user_address=0x...

# User staking info
https://api.woofi.com/user_stakingv2_infos?user_address=0x...&network=arbitrum&days=30

# Boosted APR
https://api.woofi.com/boosted_apr_info?user_address=0x...

# Perps volume
https://api.woofi.com/woofi_pro/perps_volume

# User perp volumes
https://api.woofi.com/user_perp_volumes?period=7d&user_address=0x...

# Integration pairs
https://api.woofi.com/integration/pairs

# Tickers
https://api.woofi.com/integration/tickers

# Pool states
https://api.woofi.com/integration/pool_states?network=arbitrum
```

For cross-chain queries, fetch each network in sequence and aggregate results.

---

## Step 4: Process and Convert Values

Apply necessary conversions before presenting results:

- **Volume/TVL in wei**: divide by `10^18` (or `10^decimals` for non-18-decimal tokens)
- **`trader_count`**: do NOT sum across time buckets — each bucket is a unique-per-window count
- **APR formula**: `avg_apr = base_apr + mp_boosted_apr`

---

## Step 5: Present Results

Format the output clearly:

1. State which endpoint(s) were queried and with what parameters
2. Show converted values (not raw wei) with appropriate units (USD, WOO, %, etc.)
3. Highlight key insights relevant to the user's question
4. Note any limitations or caveats (e.g., incomplete `1w` data, no cross-bucket trader summation)

If the user asked for a comparison or trend, present data in a table or ranked list.

---

## Error Handling

- If an API call fails, report the error and suggest retrying or using a different period/network
- If the user requests a network not supported by an endpoint (e.g., Solana for `/yield`), inform them and offer alternatives
- Rate limit is 5 requests/second — space out multiple requests if needed
