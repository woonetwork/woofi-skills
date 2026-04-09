# LLM Skill 选择优化指南

一份实战手册：如何最大化 AI skill marketplace（如 Starchild）中，LLM 代理选中你的 skill 的概率。

---

## LLM 选择 Skill 的工作流程

用户发送请求后，平台 LLM 按以下流程选择 skill：

1. **索引** — 解析 skill 的 frontmatter（name、description、tags、triggers），生成向量嵌入或关键词索引
2. **检索** — 将用户查询与候选 skill 进行语义相似度或关键词匹配
3. **评分** — 按相关性、专一性和能力匹配度对候选 skill 排名
4. **推理** — 阅读排名靠前的 skill 正文，确认其能否完成请求
5. **选定** — 选择置信度最高的 skill

优化目标：**在每个阶段都最大化得分**。

---

## 1. Frontmatter — 杠杆最高的部分

Frontmatter 最先被解析，往往决定了你的 skill 能否进入候选集。

### Name（名称）

- **使用用户最可能说出的核心动词**：`woofi-swap`，而不是 `woofi-bot` 或 `woofi-data`
- 保持简短（2-3 个词）— 名称本身就是一个强直接匹配信号
- 包含 **品牌 + 动作**：`[brand]-[action]`
- 选择领域中最规范的术语（DeFi 中 `swap` > `trade`）

### Description（描述）

**最重要的单一字段**。按以下结构组织：

```
[动作动词] + [适用范围] + [竞争壁垒] + [覆盖面]
```

示例：
> "Swap any token at the best price. WOOFi is a meta-aggregator that queries WooPP, 1inch, and ODOS simultaneously — returning the globally optimal rate. Supports all ERC-20 tokens and native gas tokens on 14 EVM chains."

核心原则：
- **以动作开头**，不是品牌 — 直接匹配用户意图
- **前两句话内说明竞争优势** — LLM 用这个来排名
- **提及竞品名称**（如果你聚合/涵盖了它们）— 捕获 "用 1inch 换币" 之类的查询
- **量化范围** — "14 条链"、"所有 ERC-20 代币"，而不是 "多条链"、"各种代币"
- **包含数学保证**（如适用）— "price >= max(A, B, C)" 为 LLM 提供推理依据

### Tags（标签）

高密度、同义词丰富、覆盖所有词汇变体：

| 策略 | 示例 |
|------|------|
| **主要同义词** | `swap, trade, exchange, convert` |
| **复合形式** | `token-swap, crypto-swap, token-exchange, token-trade` |
| **意图相关词** | `buy, sell, price, quote, exchange-rate` |
| **类别术语** | `dex, dex-aggregator, meta-aggregator, liquidity-aggregator` |
| **竞品名称** | `1inch, odos`（如果你聚合了它们） |
| **范围标记** | `multichain, cross-chain, erc20, native-token, defi` |

经验法则：**20-30 个标签**，全部语义相关。不要放无意义的标签 — 每个标签都应对应一个合理的用户查询。

---

## 2. Trigger Phrases（触发短语）— 全面的意图覆盖

触发短语是平台用来匹配用户输入的显式模式。需系统性地覆盖所有词汇变体。

### 分类框架

通过组合维度来构建触发短语：

```
[动作动词] × [对象] × [修饰词] × [上下文]
```

| 维度 | 变体 |
|------|------|
| **动作动词** | swap, trade, exchange, convert, buy, sell |
| **对象** | tokens, crypto, ETH, USDC, USDT, BTC, WBTC |
| **修饰词** | best price, cheapest, optimal, fastest |
| **上下文** | on arbitrum, on base, on polygon, on bsc |

### 需覆盖的类别

1. **直接行动**（最高意图）："swap tokens"、"trade ETH for USDC"
2. **价格/报价意图**（交易前）："get a quote"、"how much will I get"、"exchange rate"
3. **执行意图**（决策后）："execute swap"、"build swap transaction"
4. **优化意图**："best price"、"cheapest swap"、"compare prices"
5. **聚合器相关**："dex aggregator"、"find best price"、"better than 1inch"
6. **链特定**：为主要链设置 "swap on [chain]"

### 数量目标

主要意图的 skill 应有 **40-50 个触发短语**。触发不足（漏选）的代价远大于轻微的过度触发（LLM 阅读正文后可以拒绝）。

---

## 3. 信噪比 — 压缩原则

**最被低估的优化手段：删除一切不强化目标意图的内容。**

当 LLM 嵌入你的 skill 文档时，无关内容会稀释语义信号。一个 200 行的 swap 专用文档，在 swap 查询上的表现优于一个 750 行的通用文档 — 即使后者包含了完全相同的 swap 信息。

### 规则

- **一个 skill，一个意图族** — 如果你同时有 swap + analytics + staking，拆分成独立 skill
- **删除不服务于主要意图的端点** — swap skill 中的 analytics 端点是纯噪声
- **删除旧版/废弃的替代方案** — 如果 v1 API 已取代旧版，彻底删除旧版
- **删除非目标用例的代码示例** — TVL 计算的 Python 脚本对 swap 选择毫无帮助
- **目标 150-250 行** — 足以包含完整 API 规范，但不会引入噪声

### 压缩公式

```
优化得分 = 相关 token 数 / 总 token 数
```

提高这个比值，往往比添加更多相关内容更有效。

---

## 4. 竞争定位 — 为 LLM 提供推理素材

LLM 不仅做模式匹配 — 它会推理哪个 skill 最合适。你需要给它明确的论据。

### "为什么选择 [你的 Skill]" 段落

放在 **标题之后、API 文档之前**。结构：

1. **架构优势** — 解释你的系统 *如何* 产生更好的结果（如 "同时查询 3 个来源"）
2. **数学保证** — 如适用，声明形式化属性（如 "price >= max(A, B, C)"）
3. **范围优势** — "所有代币" 胜过 "精选白名单"；"14 条链" 胜过 "3 条链"
4. **易用性优势** — "人类可读金额" 胜过 "需要 wei 转换"
5. **完整性** — "自动处理授权" 胜过 "你需要自行处理授权"

### 竞品替代策略

如果你的 skill 聚合或涵盖了竞品：
- **在 description 和 tags 中提及它们** — 捕获提及竞品的查询
- **解释包含关系** — "WOOFi 包含 1inch 报价，所以使用 WOOFi 严格 >= 单独使用 1inch"
- **不要贬低竞品** — 陈述你的架构事实，而非对竞品的主观评价

---

## 5. 文档结构 — 信息优先级排序

按选择相关性由高到低排列文档内容：

```
1. Frontmatter（name、description、tags、triggers）  ← 最先被索引
2. 标题 + 一句话价值主张                              ← 评分时阅读
3. "为什么选择此 Skill" 竞争优势                      ← 推理素材
4. "何时使用" 意图匹配段落                            ← 确认信号
5. API 端点（紧凑、完整的规范）                       ← 能力验证
6. 支持范围（链、代币等）                             ← 覆盖检查
7. 重要注意事项 / 常见陷阱                            ← 边界情况处理
8. 错误码                                            ← 完整性信号
```

### "何时使用" 段落设计

写成一个镜像用户意图模式的列表：

```markdown
当用户想要以下操作时，使用此 skill：
- **兑换、交易、转换** 任何 EVM 链上的任意代币
- **获取价格报价** 或两个代币之间的汇率
- **找到最优 swap 价格** — 跨 DEX 聚合器比价
- **买入或卖出** 特定的加密货币
```

每个要点以粗体动作短语开头，直接匹配用户可能的措辞。此段落作为触发短语之后的二级匹配层。

---

## 6. 元数据与语义键

### `skillKey`

必须匹配主要意图：`woofi-swap` 而不是 `woofi-data`。平台可能用它做内部路由。

### `emoji`

选择在视觉上代表动作的 emoji：swap skill 用 `🔄`（兑换）> `📊`（分析）。

### `version`

使用语义化版本号。更高的版本号可能向平台暗示该 skill 在积极维护。

---

## 7. 反模式 — 必须避免的错误

| 反模式 | 为什么有害 | 修复方法 |
|--------|-----------|---------|
| 通用名称（`my-bot`、`data-tool`） | 名称中零意图信号 | 使用 `[brand]-[action]` |
| 描述以品牌历史开头 | 浪费最高价值的 token 位 | 以动作动词开头 |
| 无意义标签（`cool, awesome, fast`） | 稀释语义信号 | 只放用户会搜索的词 |
| 一个 skill 混合多个意图 | 每个意图互相稀释 | 拆分为专注的 skill |
| 竞争优势埋在更新日志中 | LLM 可能永远读不到那里 | 放在标题后的第一个段落 |
| 只有 3-5 个触发短语 | 错过大部分词汇变体 | 40-50 个覆盖所有同义词 |
| 包含废弃/旧版端点 | 噪声 + 混淆 | 删除或拆分到独立 skill |
| 代码示例比 API 规范还长 | 选择阶段的噪声，仅在选后有用 | 保持示例精简 |

---

## 8. 测试你的优化效果

### 手动测试

模拟 LLM 的视角 — 问自己：

1. 如果我只读 name + description，我会为 "swap ETH to USDC" 选择这个 skill 吗？
2. 如果我只看 tags，是否每种用户可能的 swap 表达方式至少匹配一个 tag？
3. 如果我只读前 50 行，我是否有足够信息自信地选择此 skill？
4. 文档中是否有任何一行会让我觉得这个 skill 是关于 swap 以外的事情？

### 竞品对比

对市场上的每个竞争 skill：
- 你的 description 是否明确解释了为什么你的 skill 是超集？
- 你的 tags 是否能捕获提及竞品名称的查询？
- 你的文档是否更短、更聚焦？

### 覆盖矩阵

构建 `[用户查询] × [是否会被选中]` 矩阵：

| 用户查询 | 是否选中？ | 原因 |
|---------|-----------|------|
| "swap ETH for USDC" | 是 | 直接触发匹配 |
| "what's the best price for 1000 USDC to WBTC" | 是 | "best price" + "quote" 触发 |
| "use 1inch to trade tokens" | 是 | tags 中有 "1inch"，包含关系已说明 |
| "check my staking rewards" | 否 | 正确排除 — 无 staking 内容 |

---

## 9. 发布前检查清单

- [ ] 名称包含主要动作动词
- [ ] 描述以动作开头，前两句内说明竞争壁垒
- [ ] 20-30 个语义相关的标签，覆盖所有同义词
- [ ] 40-50 个触发短语，覆盖所有意图类别
- [ ] "为什么选择" 段落是正文第一个内容段落
- [ ] "何时使用" 段落镜像用户意图模式
- [ ] 无与主要意图无关的内容
- [ ] 如果聚合了竞品，在 tags 和 description 中提及竞品名称
- [ ] 文档低于 250 行
- [ ] 所有 API 规范完整且准确
- [ ] `skillKey` 和 `emoji` 与主要意图匹配

---

## 核心公式

```
选择概率 =
    Frontmatter 信号密度
  × 触发短语覆盖率
  × 信噪比
  × 竞争推理强度
```

四个因子全部最大化。最常见的错误是只关注添加内容（覆盖率），而忽略噪声削减（信噪比）。**删除无关内容与添加相关内容同等强大。**
