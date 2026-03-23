# WOOFi Backend API Developer Documentation

This document provides a detailed overview of the WOOFi backend APIs, including their parameters, return values, logic, and other important considerations.

**Base URL**: All endpoints are served under blueprint-specific prefixes (e.g., `/api/swap/`, `/api/stat/`, `/api/user/`, `/api/staking/`, `/api/integration/`, `/api/woofi_dex/`).

**Common Response Format**:
```json
{
  "status": "ok" | "fail",
  "data": ...
}
```

---

## 1. Swap Support API

### **GET /swap_support**
- **Blueprint**: `/api/swap/`
- **Description**: Returns supported networks, DEXs, and token information for swaps.
- **Rate Limit**: 5 requests/second.
- **Caching**: 3600 seconds (1 hour).
- **Parameters**: None.
- **Return Value**:
    - `status`: `"ok"` or `"fail"`.
    - `data`: A dictionary where keys are network names (e.g., `bsc`, `avax`) and values contain:
        - `dexs`: List of supported DEXs on the network.
        - `network_infos`: General network metadata (name, chain ID, RPC URL, explorer URL, etc.).
        - `token_infos`: List of tokens supported for swaps on this network, including address, symbol, decimals, and `swap_enable` status.
- **Logic**: Aggregates configuration from internal `NETWORK_INFOS`, `NETWORK_DEXS`, and `NETWORK_WOOFI_INFOS`. It also includes a hardcoded configuration for the `Sei` network (chain ID `1329`, tokens: SEI, USDC).

---

## 2. Statistics APIs

### **GET /multi_total_stat**
- **Blueprint**: `/api/stat/`
- **Description**: Returns aggregated statistics across all supported chains.
- **Rate Limit**: 5 requests/second.
- **Caching**: 5 seconds.
- **Parameters**: None.
- **Return Value**:
    - `past_24h_volume`: 24-hour trading volume in USD (18 decimals).
    - `total_volume_usd`: Total cumulative trading volume.
    - `yesterday_volume_usd`: Trading volume from the previous UTC day.
    - `last_week_volume_usd`: Trading volume for the last 7 days.
    - `updated_at`: Unix timestamp of the last update.
- **Logic**: Reads from Redis (`woofi:multi_total_stat`) and adds real-time Solana statistics for `1d`, `7d`, and `total` periods.

### **GET /total_stat**
- **Blueprint**: `/api/stat/`
- **Description**: Returns total volume, traders, and transactions for a specific network.
- **Rate Limit**: 5 requests/second.
- **Caching**: 60 seconds.
- **Parameters**:
    - `network` (string, required): The network name (e.g., `bsc`, `solana`).
- **Return Value**:
    - `total_volume_usd`: Total trading volume in USD.
    - `total_traders`: Total unique traders.
    - `total_txns`: Total transactions.
- **Logic**:
    - For EVM: Queries from `SQLSubgraphApi` via `query_global_variable`.
    - For Solana: Aggregates data via `fetch_solana_stats()` helper function.

### **GET /cumulate_stat**
- **Blueprint**: `/api/stat/`
- **Description**: Returns 24-hour cumulative statistics (volume, traders, transactions, turnover rate).
- **Rate Limit**: 5 requests/second.
- **Caching**: 5 seconds.
- **Parameters**:
    - `network` (string, required): The network name.
- **Return Value**:
    - `24h_volume_usd`: 24-hour volume.
    - `24h_traders`: 24-hour unique traders.
    - `24h_txns`: 24-hour transactions.
    - `turnover_rate_percentage`: Turnover rate based on TVL.
- **Logic**:
    - For EVM: Queries 24 hours of hourly data from SQL via `SQLSubgraphApi(network).query_hour_data()`.
    - For Solana: Loads hourly aggregates from Redis via `load_solana_hourly_agg()` and calculates turnover based on TVL.

### **GET /stat**
- **Blueprint**: `/api/stat/`
- **Description**: Returns historical statistics (volume, traders, txns) for the specified period.
- **Rate Limit**: 5 requests/second.
- **Caching**: Varies by period:
    - `1d`: 600 seconds (10 minutes).
    - `1w`: 1800 seconds (30 minutes).
    - `1m`, `3m`, `1y`, `all`: 7200 seconds (2 hours).
- **Parameters**:
    - `network` (string, required): The network name.
    - `period` (string, optional): Time period. One of `1d`, `1w`, `1m`, `3m`, `1y`, `all`. Default: `1m`.
- **Return Value**: A list of data points, each containing:
    - `volume_usd`: Trading volume.
    - `traders`: Unique traders.
    - `txns`: Transaction count.
    - `turnover_rate_percentage`: Turnover rate (included for daily data only, not for `1d` hourly data).
    - `timestamp`: Unix timestamp of the data point.
- **Logic**: Returns daily data points for most periods, or hourly data points for `1d`.

### **GET /source_stat**
- **Blueprint**: `/api/stat/`
- **Description**: Returns trading volume and transactions grouped by source (e.g., specific integrators or frontend).
- **Rate Limit**: 5 requests/second.
- **Caching**: Varies by period (same as `/stat`).
- **Parameters**:
    - `network` (string, required): The network name.
    - `period` (string, optional): Time period. Default: `1m`.
- **Return Value**: A dictionary keyed by source name, each containing volume and transaction data.
- **Logic**: Maps internal source IDs to names via `SOURCE_ID_TO_SOURCE_NAME` (e.g., `0` → `WOOFi`, `1` → `1inch`, `2` → `DODO`, `3` → `OpenOcean`, `12` → `ODOS`, `15` → `OKX`, etc.). For Solana, it merges long-address "rebate" sources into an `"Other"` category via `_merge_long_names_into_other()`.

### **GET /token_stat**
- **Blueprint**: `/api/stat/`
- **Description**: Returns 24-hour statistics for tokens supported by WOOFi on the network.
- **Rate Limit**: 5 requests/second.
- **Caching**: 5 seconds.
- **Parameters**:
    - `network` (string, required): The network name.
- **Return Value**: List of token objects, each containing:
    - `logo_url`: URL to token logo image.
    - `symbol`: Token symbol.
    - `decimals`: Token decimal places.
    - `tvl`: Total Value Locked (string, 18 decimals).
    - `24h_volume_usd`: 24-hour volume in USD (string, 18 decimals).
    - `24h_txns`: 24-hour transaction count.
    - `turnover_rate_percentage`: Turnover rate calculated from volume / TVL.

### **GET /solana_stat**
- **Blueprint**: `/api/stat/`
- **Description**: Specialized endpoint for raw Solana pool statistics from Redis.
- **Rate Limit**: 5 requests/second.
- **Caching**: 5 seconds.
- **Parameters**:
    - `filter_days` (int, optional): Number of days to look back. Default: `14`.

---

## 3. Earning & Staking APIs

### **GET /earn_summary**
- **Blueprint**: `/api/staking/`
- **Description**: Returns a summary of active WOOFi Supercharger vaults across all networks, sorted by APR.
- **Rate Limit**: 5 requests/second.
- **Caching**: 5 seconds.
- **Parameters**: None.
- **Return Value**: List of vault objects with APR, network, and vault details.
- **Logic**:
    - Iterates through `NETWORK_UI_SPECIFIC_VAULTS`.
    - Excludes networks in `PAUSED_NETWORKS` (currently: `fantom`, `zksync`, `polygon_zkevm`).
    - Calculates weighted average APR based on loan vault and reserve vault balances, interest rates, and debt from Redis.
    - Filters out deprecated vaults and those with 0% APR.

---

## 4. User-Specific APIs

### **GET /user_balances**
- **Blueprint**: `/api/user/`
- **Description**: Fetches the user's native and ERC20 token balances for all tokens supported by WOOFi on that network.
- **Rate Limit**: 5 requests/second.
- **Caching**: 5 seconds.
- **Parameters**:
    - `network` (string, required): The network name.
    - `user_address` (string, required): The EVM address of the user.
- **Return Value**: List of token balance objects with address, symbol, decimals, and balance.
- **Logic**: Uses a `Multicall3` contract to fetch native balance and all ERC20 `balanceOf` calls in a single batched RPC call.

### **GET /user_trading_volumes**
- **Blueprint**: `/api/user/`
- **Description**: Returns the user's trading volume history over the specified period across all supported networks.
- **Rate Limit**: 5 requests/second.
- **Caching**: 5 seconds.
- **Parameters**:
    - `period` (string, required): One of `7d`, `14d`, `30d`.
    - `user_address` (string, required): User EVM address.
- **Logic**: Calculates UTC day boundaries based on the period, then queries `get_network_trading_volumes()` across all supported networks.

### **GET /user_supercharger_infos**
- **Blueprint**: `/api/user/`
- **Description**: Returns the user's positions (deposits, pending withdrawals, etc.) in WOOFi Supercharger vaults.
- **Rate Limit**: 5 requests/second.
- **Caching**: 5 seconds.
- **Parameters**:
    - `user_address` (string, required): User EVM address.

### **GET /user_stakingv2_infos**
- **Blueprint**: `/api/user/`
- **Description**: Returns user Staking V2 info, including claimed rewards (USDC/ARB), compounded WOO, and expected yearly earnings.
- **Rate Limit**: 5 requests/second.
- **Caching**: 5 seconds.
- **Parameters**:
    - `user_address` (string, required): User EVM address.
    - `network` (string, optional): Default `arbitrum`.
    - `days` (int, optional): Number of days for earnings calculation. Default `30`.

### **GET /user/boosted_apr_info**
- **Blueprint**: `/api/user/`
- **Description**: Checks if a user is eligible for a "Boosted APR" (for new stakers within 60 days of first stake).
- **Rate Limit**: 5 requests/second.
- **Caching**: 5 seconds.
- **Parameters**:
    - `user_address` (string, required): User EVM address.
    - `network` (string, optional): Network for staking lookup.
- **Return Value**:
    - `is_boosted`: Boolean indicating if the user is currently eligible for boosted APR.
    - `deadline`: Unix timestamp (`first_stake_time + 60 days`).
- **Logic**: Looks up `user_first_stake_info` and checks if current time is within `BOOSTED_APR_PERIOD` (86400 * 60 = 60 days) of the first stake timestamp.

---

## 5. WOOFi Pro / DEX APIs

### **GET /woofi_pro/perps_volume**
- **Blueprint**: `/api/woofi_dex/`
- **Description**: Returns the daily trading volume for the WOOFi Pro broker.
- **Rate Limit**: 5 requests/second.
- **Caching**: 5 seconds.
- **Parameters**: None.
- **Return Value**: List of daily volume data points.
- **Logic**: Fetches data from the Orderly API (`OrderlyApi.fetch_volume_broker_daily()`) for the last 80 days.

### **GET /woofi_dex/user_perp_volumes**
- **Blueprint**: `/api/woofi_dex/`
- **Description**: Returns the user's perpetual trading volume on WOOFi DEX for the given period.
- **Rate Limit**: 5 requests/second.
- **Caching**: 5 seconds.
- **Parameters**:
    - `period` (string, required): One of `7d`, `14d`, `30d`.
    - `user_address` (string, required): User EVM address.
- **Logic**: Reads from Redis key `woofi:woofi_pro_user_perp_volumes:{period}` and looks up the user's volume by address.

---

## 6. Integration APIs

### **GET /integration/pairs**
- **Blueprint**: `/api/integration/`
- **Description**: Returns a list of supported trading pairs for third-party integrations (e.g., CoinMarketCap, CoinGecko).
- **Rate Limit**: 5 requests/second.
- **Caching**: 3600 seconds (1 hour).
- **Parameters**: None.
- **Return Value**: List of objects with `ticker_id`, `base`, and `target` (e.g., `{"ticker_id": "ETH_USDC", "base": "ETH", "target": "USDC"}`).

### **GET /integration/tickers**
- **Blueprint**: `/api/integration/`
- **Description**: Returns 24-hour market data (last price, volume) for all integration pairs.
- **Rate Limit**: 5 requests/second.
- **Caching**: 5 seconds.
- **Parameters**: None.
- **Return Value**: List of ticker objects with `ticker_id`, `last_price`, `base_volume`, `target_volume`.
- **Logic**: Aggregates data from volume subgraphs across all supported chains.

### **GET /integration/pool_states**
- **Blueprint**: `/api/integration/`
- **Description**: Returns the internal state of WOOFi Liquidity Pools (WooPP), including reserves, fee rates, prices, and coefficients.
- **Rate Limit**: 5 requests/second.
- **Caching**: 5 seconds.
- **Parameters**:
    - `network` (string, optional): The network name.
- **Return Value**: Pool state objects containing:
    - `reserve`: Token reserve amount.
    - `fee_rate`: Trading fee rate.
    - `max_gamma`: Maximum gamma parameter.
    - `max_notional_swap`: Maximum notional swap size.
    - `cap_bal`: Balance cap.
    - `price`: Current oracle price.
    - `spread`: Price spread.
    - `coeff`: Price coefficient.
    - `wo_feasible`: Whether the oracle state is feasible.
- **Logic**: Uses `Multicall3` to batch-call `tokenInfos` on `WooPP` and `state` on `Wooracle` contracts.

---

## Important Notes

1. **Rate Limiting**: Most APIs have a limit of 5 requests/second. Rate-limited requests receive a standardized error message.
2. **Precision**: Statistics values are typically represented as strings with 18 decimals (standard ETH/USD precision) regardless of the underlying token's decimals.
3. **Address Handling**: All EVM addresses should be checksummed. The backend converts inputs to checksummed format via `Web3.toChecksumAddress()` before processing.
4. **Network Names**: Use lowercase internal keys. Supported networks include:
    - **EVM**: `bsc`, `avax`, `fantom`, `polygon`, `arbitrum`, `optimism`, `zksync`, `polygon_zkevm`, `linea`, `base`, `mantle`, `sonic`, `berachain`, `hyperevm`, `monad`.
    - **SVM**: `solana`.
    - **Other**: `sei` (hardcoded in swap_support only).
5. **Paused Networks**: Some networks may be paused (currently: `fantom`, `zksync`, `polygon_zkevm`) and excluded from certain endpoints like `earn_summary`.
6. **Caching**: Cache times vary per endpoint. Most user-specific endpoints cache for 5 seconds; static configuration like `swap_support` and `integration/pairs` caches for 1 hour; statistics endpoints vary by period.

---

## 7. WOOFi v1 Swap & Quote APIs

This protocol supports aggregated quotes and transaction building, primarily for major EVM chains such as Base, Arbitrum, Ethereum, etc.

### Basic Information
- **Base URL**: `https://sapi.woofi.com/v1`
- **Content-Type**: `application/json`

---

### 7.1. Get Quote (Quote)
Returns the current token exchange rate and expected received amount, excluding transaction data.

#### Interface Information
- **Endpoint**: `/v1/quote`
- **Method**: `POST`

#### Request Parameters (QuoteRequest)
| Parameter | Type | Required | Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| `chain_id` | `int` | Yes | - | Chain ID (Base: `8453`, Arbitrum: `42161`, BSC: `56`, Polygon: `137`) |
| `sell_token` | `str` | Yes | - | Sell token address or symbol (ETH/Native: `0xEeeeeE...`) |
| `buy_token` | `str` | Yes | - | Buy token address or symbol |
| `sell_amount` | `str` | Yes | - | Sell amount (human-readable string, e.g., `"1.5"`) |
| `slippage_pct` | `float` | No | `0.5` | Maximum allowed slippage percentage (0.5 means 0.5%) |
| `woofi_only` | `bool` | No | `false` | Whether to force routing through WOOFi only |

#### Response Fields (QuoteResponse)
| Field | Type | Description |
| :--- | :--- | :--- |
| `chain_id` | `int` | Chain ID |
| `sell_token` | `str` | Sell token address |
| `buy_token` | `str` | Buy token address |
| `sell_amount` | `str` | Sell amount |
| `buy_amount` | `str` | Expected buy amount |
| `price` | `str` | Current execution price (Buy/Sell) |
| `guaranteed_price` | `str` | Minimum guaranteed price after considering slippage |

---

### 7.2. Build Transaction (Swap)
Gets the quote while generating transaction data that can be sent directly to the blockchain.

#### Interface Information
- **Endpoint**: `/v1/swap`
- **Method**: `POST`

#### Request Parameters (SwapRequest)
In addition to `QuoteRequest` parameters, includes:

| Parameter | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `to` | `str` | Yes | Wallet address to receive the bought tokens |
| `rebate_to` | `str` | Yes | Rebate receiving address (usually the same as `to`) |
| `signer_address` | `str` | No | Signer address (used to check `needs_approve`) |

#### Response Fields (SwapResponse)
| Field | Type | Description |
| :--- | :--- | :--- |
| `needs_approve` | `bool` | Whether token approval is needed first |
| `tx_steps` | `list[TxCall]` | List of transaction steps (Approve, Swap, etc.) |

##### TxCall Structure
- `to`: Interaction target address (contract address)
- `data`: Transaction Hex data
- `value`: Amount of Native token to send (Wei, string)
- `desc`: Step description (e.g., "Approve USDC")

---

### Request and Response Examples

#### Example 1: Base Chain ETH to USDC (Quote)
**Request:**
```json
{
  "chain_id": 8453,
  "sell_token": "0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE",
  "buy_token": "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913",
  "sell_amount": "0.1"
}
```
**Response:**
```json
{
  "chain_id": 8453,
  "sell_token": "0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE",
  "buy_token": "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913",
  "sell_amount": "0.1",
  "buy_amount": "250.452130",
  "price": "2504.5213",
  "guaranteed_price": "2492.0000"
}
```

#### Example 2: Arbitrum Chain USDC to WBTC (Swap)
**Request:**
```json
{
  "chain_id": 42161,
  "sell_token": "0xaf88d065e77c8cC2239327C5EDb3A432268e5831",
  "buy_token": "0x2f2a2543b76a4166549f7aab2e75bef0aefc5b0f",
  "sell_amount": "1000",
  "to": "0xAbc...123",
  "rebate_to": "0xAbc...123",
  "signer_address": "0xAbc...123"
}
```
**Response:**
```json
{
  "chain_id": 42161,
  "sell_token": "0xaf88d065e77c8cC2239327C5EDb3A432268e5831",
  "buy_token": "0x2f2a2543b76a4166549f7aab2e75bef0aefc5b0f",
  "sell_amount": "1000",
  "buy_amount": "0.01524300",
  "price": "0.000015243",
  "guaranteed_price": "0.000015167",
  "needs_approve": true,
  "tx_steps": [
    {
      "to": "0xaf88d065e77c8cC2239327C5EDb3A432268e5831",
      "data": "0x095ea7b3...",
      "value": "0",
      "desc": "Approve USDC for WOOFi Router"
    },
    {
      "to": "0x4c4AF8DBc524681930a27b2F1Af5bcC8062E6fB7",
      "data": "0x...",
      "value": "0",
      "desc": "Swap 1000 USDC for WBTC via WOOFi"
    }
  ]
}
```

---

### Error Codes
When a request fails, the API returns a 4xx or 5xx status code with the following error format:
```json
{
  "code": "ERROR_CODE",
  "message": "Human readable message",
  "details": {}
}
```

| Error Code | HTTP Status | Description |
| :--- | :--- | :--- |
| `UNSUPPORTED_CHAIN` | 400 | The chain_id is not supported |
| `UNSUPPORTED_TOKEN` | 400 | The specified token is not supported on this chain |
| `INVALID_AMOUNT` | 400 | The amount format is invalid or <= 0 |
| `SAME_TOKEN` | 400 | Sell token and buy token are the same |
| `INSUFFICIENT_LIQUIDITY` | 422 | Insufficient pool liquidity to complete the swap |
| `SIMULATION_FAILED` | 422 | On-chain simulation failed (usually due to excessive slippage) |
| `CHAIN_RPC_ERROR` | 502 | Node access failure |

---

### Common Token Addresses Reference (Native is always 0xEeeeeE...)
- **Base**: USDC (`0x833589f...`), USDT (`0xfde4c96...`)
- **Arbitrum**: USDC (`0xaf88d06...`), USDT (`0xfd086bc...`)
- **BSC**: USDT (`0x55d3983...`)
- **Polygon**: USDC (`0x3c499c5...`)
