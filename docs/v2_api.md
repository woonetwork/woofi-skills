# WOOFi Backend V2 API Reference

## Overview

All endpoints accept `POST` with JSON body. All amount fields are human-readable decimal strings (e.g. `"1.5"`).

All endpoints support **flexible input** for chain and token parameters:

- **Chain**: accepts numeric chain ID (`42161`) or chain name (`"arbitrum"`)
- **Token**: accepts EVM address (`"0xaf88..."`) or token symbol/name (`"USDC"`)

---

## Chain & Token Resolution

### Chain Name Resolution

Case-insensitive. Automatically strips spaces, dashes, underscores, and "chain"/"network" suffixes.

| Chain ID | Accepted Names |
|----------|---------------|
| 1 | ethereum, eth, mainnet |
| 10 | optimism, op |
| 56 | bsc, bnb, binance, bnbchain, binancesmartchain |
| 137 | polygon, matic, pol |
| 143 | monad |
| 146 | sonic |
| 324 | zksync, zksyncera |
| 999 | hyperevm, hyper |
| 5000 | mantle |
| 8453 | base |
| 42161 | arbitrum, arb, arbi, arbitrumone |
| 43114 | avalanche, avax |
| 59144 | linea |
| 80094 | berachain, bera |

Examples: `"Arbitrum One"` -> `42161`, `"BSC Chain"` -> `56`, `42161` -> `42161`

### Token Name Resolution

Three-layer local lookup, plus async external fallback:

1. **Local symbol index** (`CHAIN_ADDRESSES`) - common tokens: WETH, USDC, USDT, WBTC, etc.
2. **Static JSON manifests** (`src/resources/tokens/{chain_id}.json`) - 30+ tokens per chain, indexed by symbol and name
3. **Alias table** - native tokens and wrapped variants (e.g. `eth` -> WETH, `btc` -> WBTC)
4. **External API fallback** (async) - queries Odos + 1inch APIs in parallel, results cached in Redis

**Native tokens**: chain-specific aliases (e.g. `"eth"` on Arbitrum, `"matic"` on Polygon) resolve to `0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE`.

**Cross-chain note**: `src_token` resolves against `src_chain`, `dst_token` resolves against `dst_chain`. The same symbol `"USDC"` maps to different contract addresses on different chains.

---

## Endpoints

### GET `/health`

Health check. Returns API status and per-chain connectivity.

**Response:**

```json
{
  "status": "ok",
  "chains": {
    "42161": "ok",
    "8453": "ok"
  }
}
```

### GET `/metrics`

Prometheus metrics. Requires internal-tier API key.

---

### POST `/v2/quote`

Get a price quote for a single-chain swap.

**Request:**

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `chain` | int \| str | yes | — | Chain ID or name |
| `sell_token` | str | yes | — | Token address or symbol |
| `buy_token` | str | yes | — | Token address or symbol |
| `sell_amount` | str | yes | — | Amount to sell (human-readable, e.g. `"1.5"`) |
| `slippage_pct` | float | no | 0.5 | Slippage tolerance (%) |
| `woofi_only` | bool | no | false | Restrict to WOOFi pools only |

**Response:**

| Field | Type | Description |
|-------|------|-------------|
| `chain` | int | Chain ID |
| `sell_token` | str | Resolved token address |
| `buy_token` | str | Resolved token address |
| `sell_amount` | str | Input amount |
| `buy_amount` | str | Expected output amount |
| `price` | str | Effective price |
| `guaranteed_price` | str | Guaranteed minimum price after slippage |

**Example:**

```json
// Request
{
  "chain": "arbitrum",
  "sell_token": "USDC",
  "buy_token": "WETH",
  "sell_amount": "1000"
}

// Response
{
  "chain": 42161,
  "sell_token": "0xaf88d065e77c8cC2239327C5EDb3A432268e5831",
  "buy_token": "0x82aF49447D8a07e3bd95BD0d56f35241523fBab1",
  "sell_amount": "1000",
  "buy_amount": "0.543",
  "price": "0.000543",
  "guaranteed_price": "0.000540"
}
```

---

### POST `/v2/swap`

Build transaction(s) for a single-chain swap.

**Request:**

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `chain` | int \| str | yes | — | Chain ID or name |
| `sell_token` | str | yes | — | Token address or symbol |
| `buy_token` | str | yes | — | Token address or symbol |
| `sell_amount` | str | yes | — | Amount to sell (human-readable) |
| `slippage_pct` | float | no | 0.5 | Slippage tolerance (%) |
| `to` | str | yes | — | Recipient address |
| `rebate_to` | str | yes | — | Rebate recipient address |
| `signer_address` | str \| null | no | null | Signer address (for approve check) |
| `woofi_only` | bool | no | false | Restrict to WOOFi pools only |

**Response:**

| Field | Type | Description |
|-------|------|-------------|
| `chain` | int | Chain ID |
| `sell_token` | str | Resolved token address |
| `buy_token` | str | Resolved token address |
| `sell_amount` | str | Input amount |
| `buy_amount` | str | Expected output amount |
| `price` | str | Effective price |
| `guaranteed_price` | str | Guaranteed minimum price |
| `needs_approve` | bool | Whether token approval is needed |
| `tx_steps` | TxCall[] | Transaction list: `[approve?, swap]` |

**TxCall object:**

| Field | Type | Description |
|-------|------|-------------|
| `to` | str | Contract address |
| `data` | str | Calldata (hex) |
| `value` | str | Native token value (wei) |
| `chain` | int | Chain ID |
| `desc` | str | Description (e.g. `"approve"`, `"swap"`) |

---

### POST `/v2/check_approval`

Check if a token approval is needed before swapping.

**Request:**

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `chain` | int \| str | yes | — | Chain ID or name |
| `sell_token` | str | yes | — | Token address or symbol |
| `owner` | str | yes | — | User wallet address |
| `sell_amount` | str | yes | — | Amount to check (human-readable) |

**Response:**

| Field | Type | Description |
|-------|------|-------------|
| `chain` | int | Chain ID |
| `sell_token` | str | Resolved token address |
| `owner` | str | User wallet address |
| `spender` | str | Router address (approval target) |
| `current_allowance` | str | Current allowance (human-readable) |
| `needs_approval` | bool | Whether approval is needed |

---

### POST `/v2/cross_chain/quote`

Get a price quote for a cross-chain swap (via Stargate bridge).

**Request:**

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `src_chain` | int \| str | yes | — | Source chain ID or name |
| `dst_chain` | int \| str | yes | — | Destination chain ID or name |
| `src_token` | str | yes | — | Source token address or symbol (resolved against `src_chain`) |
| `dst_token` | str | yes | — | Destination token address or symbol (resolved against `dst_chain`) |
| `src_amount` | str | yes | — | Amount to sell (human-readable) |
| `slippage_pct` | float | no | 1.0 | Slippage tolerance (%) |
| `extra_fee_pct` | float | no | 0.0 | Additional fee (%) |

**Response:**

| Field | Type | Description |
|-------|------|-------------|
| `src_chain` | int | Source chain ID |
| `dst_chain` | int | Destination chain ID |
| `src_token` | str | Resolved source token address |
| `dst_token` | str | Resolved destination token address |
| `src_bridge_token` | str | Bridge token address on source chain |
| `dst_bridge_token` | str | Bridge token address on destination chain |
| `src_amount` | str | Input amount |
| `net_src_amount` | str | Amount after fees |
| `bridge_amount_in` | str | Amount entering the bridge |
| `bridge_amount_out` | str | Amount exiting the bridge |
| `dst_amount` | str | Final output amount on destination chain |
| `price_f` | str | Effective cross-chain price |

**Example:**

```json
// Request
{
  "src_chain": "arbitrum",
  "dst_chain": "base",
  "src_token": "USDC",
  "dst_token": "USDC",
  "src_amount": "100"
}

// Response
{
  "src_chain": 42161,
  "dst_chain": 8453,
  "src_token": "0xaf88d065e77c8cC2239327C5EDb3A432268e5831",
  "dst_token": "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913",
  "src_bridge_token": "0xaf88d065e77c8cC2239327C5EDb3A432268e5831",
  "dst_bridge_token": "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913",
  "src_amount": "100",
  "net_src_amount": "100",
  "bridge_amount_in": "100",
  "bridge_amount_out": "99.95",
  "dst_amount": "99.95",
  "price_f": "0.9995"
}
```

---

### POST `/v2/cross_chain/swap`

Build transaction(s) for a cross-chain swap.

**Request:**

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `src_chain` | int \| str | yes | — | Source chain ID or name |
| `dst_chain` | int \| str | yes | — | Destination chain ID or name |
| `src_token` | str | yes | — | Source token address or symbol (resolved against `src_chain`) |
| `dst_token` | str | yes | — | Destination token address or symbol (resolved against `dst_chain`) |
| `src_amount` | str | yes | — | Amount to sell (human-readable) |
| `to` | str | yes | — | Recipient address on destination chain |
| `slippage_pct` | float | no | 1.0 | Slippage tolerance (%) |
| `extra_fee_pct` | float | no | 0.0 | Additional fee (%) |
| `signer_address` | str \| null | no | null | Signer address (for approve check) |
| `airdrop_native_amount` | int | no | 0 | Native token airdrop amount on destination chain (uint128) |

**Response:**

| Field | Type | Description |
|-------|------|-------------|
| `src_chain` | int | Source chain ID |
| `dst_chain` | int | Destination chain ID |
| `src_token` | str | Resolved source token address |
| `dst_token` | str | Resolved destination token address |
| `src_bridge_token` | str | Bridge token on source chain |
| `dst_bridge_token` | str | Bridge token on destination chain |
| `src_amount` | str | Input amount |
| `net_src_amount` | str | Amount after fees |
| `bridge_amount_in` | str | Amount entering the bridge |
| `bridge_amount_out` | str | Amount exiting the bridge |
| `dst_amount` | str | Final output amount |
| `price_f` | str | Effective cross-chain price |
| `native_fee` | str | LayerZero native fee (human-readable, in source chain native token) |
| `needs_approve` | bool | Whether token approval is needed |
| `tx_steps` | TxCall[] | Transaction list: `[approve?, crossSwap]` |
