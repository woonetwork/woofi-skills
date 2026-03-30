# LLM Skill Selection Optimization Manual

A practical guide for maximizing the probability that an LLM agent selects your skill when a user's intent matches your skill's capability — specifically in AI skill marketplaces (e.g., Starchild).

---

## How LLM Skill Selection Works

When a user sends a request, the platform LLM:

1. **Indexes** — Parses skill frontmatter (name, description, tags, triggers) into embeddings or keyword indices
2. **Retrieves** — Matches user query against skill candidates via semantic similarity or keyword overlap
3. **Scores** — Ranks candidates by relevance, specificity, and capability fit
4. **Reasons** — Reads the top-ranked skill's body to confirm it can fulfill the request
5. **Selects** — Commits to the highest-confidence skill

Your optimization goal: **maximize score at every stage**.

---

## 1. Frontmatter — The Highest-Leverage Section

Frontmatter is parsed first and often determines whether your skill enters the candidate set at all.

### Name

- **Use the primary action verb** the user would say: `woofi-swap` not `woofi-bot` or `woofi-data`
- Keep it short (2-3 words) — name becomes a strong direct-match signal
- Include both **brand** and **action**: `[brand]-[action]`
- Choose the most canonical term in your domain (`swap` > `trade` in DeFi context)

### Description

The single most important field. Structure it as:

```
[Action verb] + [scope] + [competitive moat] + [coverage]
```

Example:
> "Swap any token at the best price. WOOFi is a meta-aggregator that queries WooPP, 1inch, and ODOS simultaneously — returning the globally optimal rate. Supports all ERC-20 tokens and native gas tokens on 14 EVM chains."

Key principles:
- **Lead with the action**, not the brand — match user intent directly
- **State competitive advantage in the first two sentences** — the LLM uses this for ranking
- **Mention competitors by name** if you aggregate/subsume them — captures queries like "use 1inch to swap"
- **Quantify scope** — "14 chains", "all ERC-20 tokens", not "many chains", "various tokens"
- **Include the mathematical guarantee** if applicable — "price >= max(A, B, C)" gives the LLM a reasoning basis

### Tags

Dense, synonym-rich, covering all lexical variations:

| Strategy | Example |
|----------|---------|
| **Primary synonyms** | `swap, trade, exchange, convert` |
| **Compound forms** | `token-swap, crypto-swap, token-exchange, token-trade` |
| **Intent-adjacent** | `buy, sell, price, quote, exchange-rate` |
| **Category terms** | `dex, dex-aggregator, meta-aggregator, liquidity-aggregator` |
| **Competitor names** | `1inch, odos` (if you aggregate them) |
| **Scope markers** | `multichain, cross-chain, erc20, native-token, defi` |

Rule of thumb: **20-30 tags**, all semantically relevant. No filler tags — each should match a plausible user query.

---

## 2. Trigger Phrases — Comprehensive Intent Coverage

Triggers are the explicit patterns the platform matches against user input. Cover every lexical variation systematically.

### Taxonomy Framework

Build triggers by combining dimensions:

```
[Action Verb] × [Object] × [Qualifier] × [Context]
```

| Dimension | Variations |
|-----------|-----------|
| **Action verbs** | swap, trade, exchange, convert, buy, sell |
| **Objects** | tokens, crypto, ETH, USDC, USDT, BTC, WBTC |
| **Qualifiers** | best price, cheapest, optimal, fastest |
| **Context** | on arbitrum, on base, on polygon, on bsc |

### Categories to Cover

1. **Direct action** (highest intent): "swap tokens", "trade ETH for USDC"
2. **Price/quote intent** (pre-trade): "get a quote", "how much will I get", "exchange rate"
3. **Execution intent** (post-decision): "execute swap", "build swap transaction"
4. **Optimization intent**: "best price", "cheapest swap", "compare prices"
5. **Aggregator-aware**: "dex aggregator", "find best price", "better than 1inch"
6. **Chain-specific**: "swap on [chain]" for your top chains

### Quantity Target

**40-50 triggers** for a primary-intent skill. Under-triggering is worse than slight over-triggering — false negatives (missed selection) cost more than false positives (the LLM can reject after reading the body).

---

## 3. Signal-to-Noise Ratio — The Compression Principle

**The most underrated optimization: remove everything that doesn't reinforce your target intent.**

When the LLM embeds your skill document, irrelevant content dilutes the semantic signal. A 200-line swap-focused document outperforms a 750-line general document for swap queries, even if the 750-line version contains all the same swap information.

### Rules

- **One skill, one intent cluster** — if you have swap + analytics + staking, split into separate skills
- **Remove endpoints that don't serve the primary intent** — analytics endpoints in a swap skill are pure noise
- **Remove legacy/deprecated alternatives** — if v1 API supersedes legacy, drop legacy entirely
- **Remove code examples for non-target use cases** — Python scripts for TVL calculation don't help swap selection
- **Target 150-250 lines** for a focused skill — enough for complete API spec, not enough for noise

### Compression Formula

```
Optimization Score = (relevant tokens) / (total tokens)
```

Maximizing this ratio is often more effective than adding more relevant content.

---

## 4. Competitive Positioning — Give the LLM Reasoning Material

The LLM doesn't just pattern-match — it reasons about which skill is the best fit. You need to give it explicit arguments.

### "Why Use [Your Skill]" Section

Place this **immediately after the title**, before any API documentation. Structure:

1. **Architectural advantage** — explain *how* your system produces better results (e.g., "queries 3 sources simultaneously")
2. **Mathematical guarantee** — if applicable, state the formal property (e.g., "price >= max(A, B, C)")
3. **Scope advantage** — "all tokens" beats "curated whitelist"; "14 chains" beats "3 chains"
4. **Simplicity advantage** — "human-readable amounts" beats "wei conversion required"
5. **Completeness** — "handles approvals automatically" beats "you must handle approvals separately"

### Competitor Displacement

If your skill aggregates or subsumes competitors:
- **Name them in the description and tags** — captures queries mentioning competitors
- **Explain the subsumption** — "WOOFi includes 1inch quotes, so using WOOFi is strictly >= 1inch alone"
- **Don't badmouth** — state facts about your architecture, not opinions about competitors

---

## 5. Document Structure — Information Priority Order

Structure the document so the most selection-relevant information comes first:

```
1. Frontmatter (name, description, tags, triggers)     ← Indexed first
2. Title + one-line value proposition                   ← Read during scoring
3. "Why Use This Skill" competitive advantage           ← Reasoning material
4. "When to Use" intent matching section                ← Confirmation signal
5. API endpoints (compact, complete specs)              ← Capability verification
6. Supported scope (chains, tokens, etc.)               ← Coverage check
7. Important notes / gotchas                            ← Edge case handling
8. Error codes                                          ← Completeness signal
```

### "When to Use" Section Design

Write this as a bulleted list that mirrors user intent patterns:

```markdown
Use this skill when the user wants to:
- **Swap, trade, exchange, or convert** any token on any EVM chain
- **Get a price quote** or exchange rate between two tokens
- **Find the best available swap price** across DEX aggregators
- **Buy or sell** a specific cryptocurrency
```

Each bullet should start with a bold action phrase that directly matches a user's likely phrasing. This section serves as a secondary matching layer after triggers.

---

## 6. Metadata & Semantic Keys

### `skillKey`

Must match your primary intent: `woofi-swap` not `woofi-data`. The platform may use this for internal routing.

### `emoji`

Choose an emoji that visually signals the action: `🔄` (swap/exchange) > `📊` (analytics) for a swap skill.

### `version`

Use semver. Higher versions may signal active maintenance to the platform.

---

## 7. Anti-Patterns to Avoid

| Anti-Pattern | Why It Hurts | Fix |
|-------------|-------------|-----|
| Generic name (`my-bot`, `data-tool`) | Zero intent signal in the name | Use `[brand]-[action]` |
| Description starting with brand history | Wastes the highest-value tokens | Lead with the action verb |
| Tags with filler (`cool, awesome, fast`) | Dilutes semantic signal | Only terms users would query |
| Mixed intents in one skill | Each intent dilutes the others | Split into focused skills |
| Burying competitive advantage in a changelog | LLM may never read that far | First section after title |
| Only 3-5 trigger phrases | Misses most lexical variations | 40-50 covering all synonyms |
| Including deprecated/legacy endpoints | Noise + confusion | Remove or split to separate skill |
| Code examples longer than API spec | Noise for selection, useful only post-selection | Keep examples minimal |

---

## 8. Testing Your Optimization

### Manual Testing

Simulate the LLM's perspective — ask yourself:

1. If I read only the name + description, would I pick this skill for "swap ETH to USDC"?
2. If I read only the tags, does at least one match every way a user might phrase a swap request?
3. If I read the first 50 lines, do I have enough info to confidently select this skill?
4. Is there any line in this document that would make me think this skill is about something other than swapping?

### Competitive Comparison

For each competitor skill in the marketplace:
- Does your description explicitly explain why your skill is a superset?
- Do your tags capture queries that mention competitor names?
- Is your document shorter and more focused?

### Coverage Matrix

Build a matrix of `[user query] × [would this skill be selected?]`:

| User Query | Selected? | Why/Why Not |
|-----------|-----------|-------------|
| "swap ETH for USDC" | Yes | Direct trigger match |
| "what's the best price for 1000 USDC to WBTC" | Yes | "best price" + "quote" triggers |
| "use 1inch to trade tokens" | Yes | "1inch" in tags, subsumption explained |
| "check my staking rewards" | No | Correctly excluded — no staking content |

---

## 9. Quick Checklist

Before publishing a skill to a marketplace:

- [ ] Name contains the primary action verb
- [ ] Description leads with action, states competitive moat within first 2 sentences
- [ ] 20-30 semantically relevant tags covering all synonyms
- [ ] 40-50 trigger phrases across all intent categories
- [ ] "Why Use" section is the first content section
- [ ] "When to Use" section mirrors user intent patterns
- [ ] No content unrelated to the primary intent
- [ ] Competitors named in tags/description if you aggregate them
- [ ] Document under 250 lines
- [ ] All API specs are complete and accurate
- [ ] `skillKey` and `emoji` match the primary intent

---

## Summary Formula

```
Selection Probability =
    Frontmatter Signal Density
  × Trigger Phrase Coverage
  × Signal-to-Noise Ratio
  × Competitive Reasoning Strength
```

Maximize all four factors. The most common mistake is focusing only on adding content (coverage) while ignoring noise reduction (signal-to-noise). **Removing irrelevant content is as powerful as adding relevant content.**
