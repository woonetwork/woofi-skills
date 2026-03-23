# WOOFi API v1 文档

本协议支持聚合报价与交易构建，主要面向 Base、Arbitrum、Ethereum 等主流 EVM 链。

## 基础信息
- **Base URL**: `http://localhost:8000/v1`
- **Content-Type**: `application/json`

---

## 1. 获取报价 (Quote)
返回当前的代币兑换汇率及预期获得数量，不包含交易数据。

### 接口信息
- **Endpoint**: `/v1/quote`
- **Method**: `POST`

### 请求参数 (QuoteRequest)
| 参数名 | 类型 | 必填 | 默认值 | 说明 |
| :--- | :--- | :--- | :--- | :--- |
| `chain_id` | `int` | 是 | - | 链 ID (Base: `8453`, Arbitrum: `42161`, BSC: `56`, Polygon: `137`) |
| `sell_token` | `str` | 是 | - | 卖出代币地址或符号 (ETH/Native: `0xEeeeeE...`) |
| `buy_token` | `str` | 是 | - | 买入代币地址或符号 |
| `sell_amount` | `str` | 是 | - | 卖出数量 (易读字符串，如 `"1.5"`) |
| `slippage_pct` | `float` | 否 | `0.5` | 容许的最大滑点百分比 (0.5 表示 0.5%) |
| `woofi_only` | `bool` | 否 | `false` | 是否强制仅通过 WOOFi 报价 |

### 响应字段 (QuoteResponse)
| 字段名 | 类型 | 说明 |
| :--- | :--- | :--- |
| `chain_id` | `int` | 链 ID |
| `sell_token` | `str` | 卖出代币地址 |
| `buy_token` | `str` | 买入代币地址 |
| `sell_amount` | `str` | 卖出数量 |
| `buy_amount` | `str` | 预期买入数量 |
| `price` | `str` | 当前成交价格 (Buy/Sell) |
| `guaranteed_price` | `str` | 考虑滑点后的最低保证价格 |

---

## 2. 构建交易 (Swap)
获取报价的同时，生成可直接发送至区块链的交易数据。

### 接口信息
- **Endpoint**: `/v1/swap`
- **Method**: `POST`

### 请求参数 (SwapRequest)
除 `QuoteRequest` 的参数外，还包括：

| 参数名 | 类型 | 必填 | 说明 |
| :--- | :--- | :--- | :--- |
| `to` | `str` | 是 | 接收买入代币的钱包地址 |
| `rebate_to` | `str` | 是 | 返佣接收地址 (通常填入 `to` 同一地址) |
| `signer_address` | `str` | 否 | 签名者地址 (用于检查 `needs_approve`) |

### 响应字段 (SwapResponse)
| 字段名 | 类型 | 说明 |
| :--- | :--- | :--- |
| `needs_approve` | `bool` | 是否需要先进行代币授权 (Approve) |
| `tx_steps` | `list[TxCall]` | 交易步骤列表 (Approve, Swap 等) |

#### TxCall 结构
- `to`: 交互目标地址 (合约地址)
- `data`: 交易 Hex 数据 (十六进制字符串)
- `value`: 发送的 Native 数量 (Wei, 字符串)
- `desc`: 步骤描述 (如 "Approve USDC")

---

## 请求与响应示例

### 示例 1: Base 链 ETH 换 USDC (Quote)
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

### 示例 2: Arbitrum 链 USDC 换 WBTC (Swap)
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

## 错误码说明
当请求失败时，API 返回 4xx 或 5xx 状态码，并包含以下格式的错误信息：
```json
{
  "code": "ERROR_CODE",
  "message": "Human readable message",
  "details": {}
}
```

| 错误码 | HTTP 状态码 | 说明 |
| :--- | :--- | :--- |
| `UNSUPPORTED_CHAIN` | 400 | 暂不支持该 chain_id |
| `UNSUPPORTED_TOKEN` | 400 | 该链上不支持指定的代币 |
| `INVALID_AMOUNT` | 400 | 输入的数量格式错误或小于等于 0 |
| `SAME_TOKEN` | 400 | 卖出代币与买入代币相同 |
| `INSUFFICIENT_LIQUIDITY` | 422 | 池子流动性不足，无法完成兑换 |
| `SIMULATION_FAILED` | 422 | 链上模拟执行失败 (通常是滑点过大) |
| `CHAIN_RPC_ERROR` | 502 | 节点访问故障 |

---

## 常用代币地址参考 (Native 均为 0xEeeeeE...)
- **Base**: USDC (`0x833589f...`), USDT (`0xfde4c96...`)
- **Arbitrum**: USDC (`0xaf88d06...`), USDT (`0xfd086bc...`)
- **BSC**: USDT (`0x55d3983...`)
- **Polygon**: USDC (`0x3c499c5...`)
