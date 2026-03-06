---
description: Query WOOFi Swap API for trading stats, volume by source, earn TVL, or WOO staking data
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
| Trading volume, trader counts, transaction counts | `GET /stat` | `period`, `network` |
| Volume breakdown by integrator/aggregator | `GET /source_stat` | `period`, `network` |
| Earn vault TVL, APY, yield farming metrics | `GET /yield` | `network` |
| WOO token staking APR and total staked | `GET /stakingv2` | none |

If the request is ambiguous, ask the user to clarify before making API calls.

---

## Step 2: Validate Parameters

### Period values (for `/stat` and `/source_stat`)
`1d` (hourly buckets), `1w`, `1m`, `3m`, `1y`, `all` (daily buckets)

### Network values
`bsc`, `avax`, `polygon`, `arbitrum`, `optimism`, `linea`, `base`, `mantle`, `sonic`, `berachain`, `hyperevm`, `monad`, `solana`

Note: `/yield` only supports 10 networks — excludes `solana`, `hyperevm`, `monad`.

If the user did not specify a network or period, ask before proceeding.

---

## Step 3: Fetch Data

Construct the URL and fetch using WebFetch. Examples:

```
# Trading stats
https://api.woofi.com/stat?period=1m&network=arbitrum

# Volume by source
https://api.woofi.com/source_stat?period=1m&network=arbitrum

# Earn TVL
https://api.woofi.com/yield?network=arbitrum

# WOO staking
https://api.woofi.com/stakingv2
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
- Rate limit is 120 requests/minute — space out multiple requests if needed
