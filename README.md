# Hyperliquid 知识库

> 从 dYdX 到 GMX 到 Hyperliquid —— 链上永续合约的终极形态是什么？本仓库是对 Hyperliquid 的系统性技术拆解，覆盖架构、交易机制、生态、竞争格局和未来方向。

> 📖 **Featured in:** [《为什么 Hyperliquid 赢了链上 perp：一个交易所工程师的架构拆解》](https://blog.frankfu.cloud/posts/hyperliquid-architecture-breakdown/) — 基于本仓库内容写的中文长文，约 8000 字。本仓库是它的延伸阅读资源。

---

## 为什么做这个

Hyperliquid 是 2024-2026 年 DeFi 领域最重要的产品之一：

- **$4.15 万亿**累计永续合约交易量
- **70%+** 链上永续合约市场份额（2026 年峰值）
- **200,000 TPS**，0.2 秒最终性
- **零 VC 融资**，31% Token 空投给用户
- 自建 L1（HyperBFT），不是"在别人链上做 DEX"
- HyperEVM 让它从"永续交易所"进化成"通用金融链"

它代表了一种新范式：**不是在通用链上做应用，而是为应用定制一条链**。

这个仓库拆解它的一切：技术架构、交易机制、Token 经济、生态系统、竞争格局、风险事件、以及它对整个 DeFi 的启示。

---

## 适合谁看

| 你是谁 | 从哪开始 | 你会得到什么 |
|--------|----------|-------------|
| 🟢 **好奇的交易者** | basics/01 + 02 | 理解 Hyperliquid 是什么、为什么火 |
| 🟡 **DeFi 研究员** | basics/03 + 04 + 07 | 技术架构 + 竞争格局的深度分析 |
| 🔵 **想在上面开发的工程师** | concepts/ 全部 | HyperBFT、订单簿、HyperEVM 的工程细节 |
| 🔴 **做类似产品的团队** | basics/02 + 07 + concepts/01-03 | 设计决策的 why + 竞品对比 |

---

## 目录结构

```
docs/
├── basics/                         # 基础介绍（按功能模块，从浅到深）
│   ├── 01-platform-overview.md        # 什么是 Hyperliquid、核心数据、定位
│   ├── 02-history-and-context.md      # 前世今生：dYdX → GMX → Hyperliquid
│   ├── 03-technical-architecture.md   # L1 + HyperBFT + HyperCore + HyperEVM
│   ├── 04-trading-mechanism.md        # 永续合约机制、订单类型、资金费率、杠杆
│   ├── 05-tokenomics.md              # HYPE Token、空投、Assistance Fund、质押
│   ├── 06-ecosystem.md               # HyperEVM 生态、Builder Codes、DeFi 协议
│   ├── 07-competition.md             # vs dYdX vs GMX vs Vertex vs Jupiter Perp
│   └── 08-risks-and-events.md        # JELLY 事件、Vault 风险、中心化争议、监管
│
└── concepts/                       # 核心概念深度解析（按知识点拆分）
    ├── 01-hyperbft-consensus/         # HyperBFT 共识：Hotstuff 变体、验证者、最终性
    ├── 02-orderbook-engine/           # 全链上订单簿：撮合、Gas-free、确定性执行
    ├── 03-clearing-and-margin/        # 保证金模型、清算机制、Insurance Fund、ADL
    ├── 04-vaults-and-hlp/             # Vault 系统、HLP（Hyperliquidity Provider）
    ├── 05-bridge-and-deposits/        # Arbitrum 桥、USDC 入金、跨链安全
    ├── 06-api-and-sdk/                # REST + WebSocket + Python SDK + Info API
    ├── 07-hyperEVM/                   # EVM 兼容层、原子组合性、precompiles
    └── 08-market-making/              # 做市策略、Builder Codes、Vault 策略
```

---

## 快速导航

### 基础介绍

| 文档 | 内容 | 推荐阅读顺序 |
|------|------|:---:|
| [平台概览](docs/basics/01-platform-overview.md) | 什么是 Hyperliquid、核心数据、为什么重要 | 1 |
| [前世今生](docs/basics/02-history-and-context.md) | 链上永续的进化史：dYdX → GMX → Hyperliquid | 2 |
| [技术架构](docs/basics/03-technical-architecture.md) | HyperBFT + HyperCore + HyperEVM 三层架构 | 3 |
| [交易机制](docs/basics/04-trading-mechanism.md) | 永续合约、订单类型、资金费率、杠杆、清算 | 4 |
| [Token 经济](docs/basics/05-tokenomics.md) | HYPE 分配、空投、Assistance Fund、质押 | 5 |
| [生态系统](docs/basics/06-ecosystem.md) | HyperEVM 上的 DeFi、Builder Codes、生态项目 | 6 |
| [竞争格局](docs/basics/07-competition.md) | vs dYdX V4 vs GMX V2 vs Vertex vs Jupiter Perp | 7 |
| [风险与事件](docs/basics/08-risks-and-events.md) | JELLY 事件、Vault 风险、中心化争议 | 8 |

### 核心概念（按需查阅）

概念文档按知识点拆分为独立小文件，适合工程师和研究员深入某个具体模块。

---

## 关键数据速览（2026 年 Q1）

| 指标 | 数据 |
|------|------|
| 累计永续交易量 | $4.15 万亿 |
| 链上永续市场份额 | 60-70%（峰值 80%） |
| 日均交易量 | $5-15B |
| 链上 TPS | 200,000 |
| 最终性 | 0.2 秒（单区块确认） |
| 支持交易对 | 150+ 永续 + 现货 |
| 最大杠杆 | 40x（BTC/ETH），20x（其他） |
| 验证者数量 | 16（计划扩展） |
| HYPE Token 市值 | ~$5-8B（波动大） |
| 空投比例 | 31%（Genesis Distribution） |
| VC 融资 | $0（完全自资金） |
| 团队 | ~10 人核心（Jeff Yan 创始人） |

---

## 和其他知识库的关系

| 仓库 | 关系 |
|------|------|
| [polymarket_design](https://github.com/survivorff/polymarket_design) | 预测市场拆解（不同赛道，但架构思路可对比） |
| [meme-trade-wiki](https://github.com/survivorff/meme-trade-wiki) | Meme 交易生态（Hyperliquid 的现货 + Hyperps 和 Meme 有交集） |
| [blog.frankfu.cloud](https://blog.frankfu.cloud) | 基于本仓库的观点文章会发到博客 |
| [web3-insider](https://github.com/survivorff/web3-insider) | 英文版深度文章 |

---

## 写作原则

1. **先讲 Why 再讲 How**：每个设计决策先解释"为什么这么做"，再讲"怎么实现的"
2. **和竞品对比**：每个模块都和 dYdX / GMX / CEX 做对比，让读者有参照系
3. **标注数据来源**：所有数据标注时间和来源，避免过时信息误导
4. **承认不确定性**：Hyperliquid 很多细节未公开（代码不开源），推测的地方明确标注
5. **工程师视角**：不是营销文，是技术拆解。有观点但基于事实。

---

## 更新计划

- **Phase 1** ✅：basics/ 全部 8 篇 + README
- **Phase 2** ✅：concepts/ 前 4 个目录（HyperBFT、订单簿、清算、Vault）
- **Phase 3**：concepts/ 后 4 个目录（桥、API、HyperEVM、做市）
- **Phase 4**：基于仓库内容写 blog + web3-insider 文章

---

## 关于作者

交易所工程师，做过 Solana + EVM 的交易基建。对 Hyperliquid 的兴趣来自：它用一条自建链解决了我们在通用链上花了 2 年都没解决好的问题（延迟、确定性、订单簿性能）。

- 📝 [blog.frankfu.cloud](https://blog.frankfu.cloud)
- 𝕏 [@FrankFu2262](https://x.com/FrankFu2262)
- 🐙 [@survivorff](https://github.com/survivorff)

## 许可证

[CC BY-SA 4.0](./LICENSE) — 自由分享和改编，需注明出处。
