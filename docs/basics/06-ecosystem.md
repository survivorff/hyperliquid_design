# Hyperliquid 生态系统：HyperEVM、Builder Codes 与 DeFi 协议

> Hyperliquid 不再只是一个永续合约交易所。HyperEVM 让它从"产品"进化成"平台"，Builder Codes 让任何前端都能分润，HIP-3 让任何人都能上市场。这篇拆解 Hyperliquid 的整个生态。

---

## 1. 从"产品"到"平台"的进化

### 1.1 三个阶段

Hyperliquid 的生态演进可以分为三个清晰的阶段：

| 阶段 | 时间 | 定位 | 标志 |
|------|------|------|------|
| **阶段 1：永续 DEX** | 2023-2024 | 单一产品 | 主网上线、Genesis 空投 |
| **阶段 2：交易平台** | 2024-2025 | 永续 + 现货 + Vault | 现货上线、HLP 普及 |
| **阶段 3：金融基础设施** | 2025-至今 | 通用金融链 | HyperEVM 上线、HIP-3 |

### 1.2 为什么进化是必然的

任何成功的金融产品最终都会演化成平台：
- **Binance**：从交易所 → 链（BSC）→ 全栈金融服务
- **Coinbase**：从交易所 → L2（Base）→ 钱包 + 借贷 + 卡
- **Hyperliquid**：从永续 DEX → HyperEVM → 通用金融基础设施

**这不是路径依赖，是商业逻辑**：
- 用户聚集 → 流动性聚集 → 开发者聚集 → 更多产品 → 更多用户（飞轮）
- 不进化的平台会被进化的平台吃掉

---

## 2. HyperEVM：通用智能合约层

### 2.1 什么是 HyperEVM

HyperEVM 是 Hyperliquid 的**完全兼容 EVM 的智能合约层**。它和 HyperCore（订单簿）共享同一个验证者网络和共识层，但是独立的执行环境。

**关键特性**：
- 完全兼容 EVM（Solidity / Vyper 合约直接部署）
- 工具链兼容（Foundry、Hardhat、Remix 都能用）
- Gas 用 HYPE 支付
- 和 HyperCore **原子组合**（这是最独特的能力）

### 2.2 双区块节奏

HyperEVM 有一个独特的设计：**双区块（Dual-Block）**模式。

| 区块类型 | 出块时间 | Gas Limit | 用途 |
|---------|---------|-----------|------|
| Fast Block（快块） | ~2 秒 | ~2M Gas | 延迟敏感操作（清算、小额交互） |
| Slow Block（慢块） | ~60 秒 | ~30M Gas | 吞吐重活（批量结算、复杂计算） |

**为什么需要双区块**：

传统 EVM 链（以太坊、Arbitrum）有一个矛盾：
- 区块时间短 → 响应快，但单块容量小（不能跑复杂计算）
- 区块时间长 → 单块容量大，但响应慢（用户体验差）

HyperEVM 的解法：**两种区块交替出**，应用按需选择。
- 一个 keeper 监控清算 → 用 Fast Block（响应快）
- 一个 vault 月底重新平衡 → 用 Slow Block（容量大）

### 2.3 Precompiles：和 HyperCore 的桥

HyperEVM 通过特殊的预编译合约（Precompiles）和 HyperCore 通信。

**读 Precompile（同步）**：

```solidity
// 伪代码：从 Solidity 合约读 HyperCore 状态
uint256 midPrice = IHyperCoreReader.getMidPrice(BTC_PERP);
uint256 myMargin = IHyperCoreReader.getAccountMargin(msg.sender);
uint256 fundingRate = IHyperCoreReader.getFundingRate(ETH_PERP);
```

合约可以直接读：
- 任何市场的中间价、最优买卖价
- 任何账户的保证金、持仓
- 资金费率、Oracle 价格
- **没有外部 Oracle 依赖**

**写 Precompile（异步）—— CoreWriter**：

```solidity
// 伪代码：从 Solidity 合约下单
ICoreWriter(0x3333...).placeOrder(
    BTC_PERP,
    size,
    limitPrice,
    OrderType.Limit
);
```

合约可以提交：
- 下单（限价、市价、Post-Only、IOC）
- 撤单
- 转账（HyperEVM ↔ HyperCore）
- 开/关子账户

**关键**：写是**异步**的。你的交易里发出指令，HyperCore 在**下一个区块**真正撮合。

### 2.4 同步读 + 异步写的设计哲学

这是 Hyperliquid 整个架构里**最巧妙**的工程决策。

**为什么读是同步的**：
- 读不改变状态，没有竞态条件
- 同步读让合约逻辑可以基于实时价格做决策
- 不需要 Oracle，避免延迟和操纵风险

**为什么写是异步的**：
- 撮合不能被绑进 EVM 的 Gas metering（否则一个慢撮合卡住整条链）
- 避免跨层 reentrancy（EVM 调撮合 → 撮合回调 EVM → 死循环）
- 撮合引擎可以独立优化，不被 EVM 语义绑架

**对开发者的影响**：
- 你不能"假装撮合在你这笔交易里完成"
- UI 层面要做"pending"状态（订单提交后等 1-2 个区块确认）
- 任何依赖订单立即成交的逻辑要做 fallback

这个设计在系统工程里有个通用名字：**Clock Boundary Crossing**。CPU 和 PCIe 设备之间也是这种关系：CPU 通过 MMIO 同步读设备状态，通过 DMA 队列异步下发命令。

---

## 3. HyperEVM 上的 DeFi 协议

### 3.1 协议分类

HyperEVM 在 2025 年 Q2 上线，到 2026 年 Q1 已经形成了相对完整的 DeFi 生态。

| 类别 | 代表项目 | TVL（估计） | 状态 |
|------|---------|------------|------|
| **借贷** | HyperLend, Felix | $200-500M | 成熟 |
| **稳定币** | USDH, hyUSD | $100-300M | 早期 |
| **DEX (现货)** | HyperSwap, KittenSwap | $50-150M | 早期 |
| **期权** | HyperOptions, Lyra | $20-50M | 实验阶段 |
| **收益聚合** | HyperYield, Beefy-HL | $50-100M | 早期 |
| **跟单/策略** | Vault 生态（数百个） | $500M+ | 成熟 |
| **预测市场** | Hyperpredict | $5-20M | 实验阶段 |
| **NFT** | HyperNFT marketplace | 极小 | 起步 |

### 3.2 借贷协议

**HyperLend** 是 HyperEVM 上的旗舰借贷协议：

**核心创新**：用 Hyperliquid 的**永续仓位**作为抵押品借贷。

```
传统借贷（Aave）：
  抵押 ETH → 借 USDC
  风险：ETH 价格波动

HyperLend 创新：
  抵押 BTC 永续多头持仓 → 借 USDC
  → 同时获得永续做多的杠杆 + 借出的 USDC 流动性
  → 资本效率极高
```

**这只有在 Hyperliquid 上才能做到**：
- 借贷协议需要实时知道抵押品价值（永续持仓的 PnL）
- 清算时需要平掉抵押的永续持仓
- 这两个操作都依赖 HyperEVM 和 HyperCore 的原子组合

### 3.3 稳定币

**USDH**（推测名称）是 Hyperliquid 原生稳定币：

**机制类型**：
- CDP（抵押债务头寸）模式，类似 MakerDAO 的 DAI
- 用户抵押加密资产 → 铸造 USDH
- 抵押率要求 150%+
- 清算机制和 Hyperliquid 订单簿原生集成

**为什么需要原生稳定币**：
- 减少对 USDC 的单点依赖（USDC 受 Circle 监管）
- 给 HyperEVM 上的 DeFi 提供"原生"基础货币
- 抵押 HYPE 铸造稳定币 → 创造 HYPE 需求

### 3.4 现货 DEX

虽然 Hyperliquid 自己有现货订单簿，但 HyperEVM 上的 AMM-style DEX 仍有需求：

**适用场景**：
- HIP-1 标准之外的长尾 Token（订单簿不一定上架）
- LP 提供流动性赚取手续费（不同于做市）
- 跨链桥过来的 ERC-20 Token 交易

**代表项目**：
- **HyperSwap**：Uniswap V2 风格的 AMM
- **KittenSwap**：Uniswap V3 风格的集中流动性

### 3.5 Vault 生态

Vault 是 Hyperliquid 生态最活跃的产品形态。

**什么是 Vault**：
- 用户存入资金 → Vault 自动执行某种策略
- 类似"链上量化基金"或"跟单"
- 收益归存款人，策略管理者抽佣

**主要类型**：

| 类型 | 策略 | 风险 | 收益（年化） |
|------|------|------|-------------|
| **HLP（官方）** | 全市场做市 + 清算接盘 | 中 | 10-30% |
| **Trader Vault** | 跟单某个交易员 | 高 | 极不稳定 |
| **Market Maker Vault** | 单市场做市 | 中 | 5-20% |
| **Funding Arb Vault** | 资金费率套利 | 低-中 | 10-25% |
| **Delta-Neutral Vault** | 多空对冲 + 收益增强 | 低 | 5-15% |

**Vault 的繁荣是 Hyperliquid 独特优势**：
- 链上策略可以原子组合（HLP 的清算接盘是一个例子）
- 用户可以选择信任谁的策略
- 透明可查（所有持仓和交易链上可见）
- 类似 eToro 的跟单交易，但完全去中心化

### 3.6 期权

期权是 Hyperliquid 生态相对薄弱的一块：

**挑战**：
- 期权定价复杂（需要 Black-Scholes 等模型）
- 流动性碎片化（每个行权价 + 到期日是独立市场）
- HyperCore 目前只支持永续，不支持固定期限

**HIP-4 的意义**：
- 计划引入"固定期限合约"
- 为期权提供基础设施
- 可能是下一个生态爆发点

---

## 4. Builder Codes：分润系统

### 4.1 什么是 Builder Codes

Builder Codes 是 Hyperliquid 的**前端聚合器分润系统**。

**机制**：
1. 任何人可以注册一个 Builder Code
2. 在自己的前端（网站、机器人、App）集成 Builder Code
3. 用户通过该前端交易 → 系统记录 Builder Code
4. 该用户产生的手续费的一部分 → 自动分给 Builder

**这本质上是"链上推荐返佣"**，但实现得非常优雅。

### 4.2 分润比例

| Builder 抽成 | 用户折扣 | 平台收入 |
|------------|---------|---------|
| 0-50% Builder Fee | 可选给用户折扣 | 剩余 |

Builder 可以自定义：
- 抽多少（比如 30% 给自己，20% 折扣给用户，剩下 50% 给平台）
- 是否给用户折扣（吸引用户）
- 不同用户不同费率（VIP 客户更低费率）

### 4.3 谁在用 Builder Codes

**前端聚合器**：
- 第三方做的 Hyperliquid 交易界面
- 提供更好的 UI、更多功能
- 通过 Builder Codes 赚钱

**交易机器人**：
- 量化策略机器人
- 跟单机器人
- 套利机器人

**KOL / 教育者**：
- 推荐自己的粉丝用某个前端
- 分享专属链接获得分润

**机构服务**：
- 为大客户定制服务
- 通过低费率 + 私人服务吸引客户

### 4.4 Builder Codes 的战略意义

**对 Hyperliquid**：
- 用户增长不依赖自己（前端开发者帮你拉新）
- 多样化前端 → 更多创新
- 机构客户接入 → 大体量交易

**对生态**：
- 创业者有商业模式（不用自己做撮合，做前端就行）
- 类似"Shopify 模式"：Hyperliquid 是后端基础设施，无数前端品牌共生

**对比传统 CEX**：
- CEX 不允许第三方前端（怕用户流失）
- DEX 必须支持（用户自己接钱包就能交易）
- Builder Codes 是 DEX 时代的"分销系统"

---

## 5. HIP 标准（Hyperliquid Improvement Proposal）

### 5.1 什么是 HIP

HIP 是 Hyperliquid 的协议改进提案，类似以太坊的 EIP。每个 HIP 定义一个标准或机制。

| HIP | 名称 | 状态 | 说明 |
|-----|------|------|------|
| **HIP-1** | 原生 Token 标准 | 已上线 | HyperCore 原生 Token |
| **HIP-2** | 流动性激励 | 已上线 | 现货市场做市激励 |
| **HIP-3** | 无许可市场创建 | 已上线 | Builder 创建永续市场 |
| **HIP-4** | 固定期限合约 | 测试中 | 期权基础设施 |

### 5.2 HIP-1：原生 Token 标准

不是 ERC-20，是 HyperCore 的原生 Token 标准。

**优势**：
- 转账、铸造、销毁都是原生操作（不烧 EVM Gas）
- 性能远高于 ERC-20
- 和 HyperCore 订单簿原生集成

**和 ERC-20 的桥**：
- HyperEVM 上的 ERC-20 ↔ HyperCore 上的 HIP-1
- 双向桥，可以无缝转换
- 部分 Token 同时存在两种形式

### 5.3 HIP-2：流动性激励

**目标**：引导新现货市场的初始流动性。

**机制**：
- 项目方拿出一部分 Token 作为激励
- 做市商在该市场提供流动性 → 获得 Token 奖励
- 类似 Uniswap V2 的"流动性挖矿"，但针对订单簿

### 5.4 HIP-3：无许可市场创建

这是 Hyperliquid 从"产品"到"平台"的关键一步。

**机制**：
- 任何人质押 100 万 HYPE → 成为 Builder
- Builder 可以创建新的永续合约市场
- Builder 提供 Oracle 价格源
- 如果市场出问题（操纵、Oracle 失效）→ Builder 质押被 slash

**已经创建的市场类型**：
- Tokenized 股票（特斯拉、英伟达、苹果）
- 大宗商品（原油、黄金）
- 外汇（EUR/USD、USD/JPY）
- 选举概率
- AI 模型性能
- 天气

**HIP-3 的革命性**：
- 第一次"任何人可以上市场"
- 长尾衍生品市场终于有地方落地
- 监管套利空间（不需要任何监管批准）

**风险**：
- 每个新市场都是新的尾部风险（HLP 可能要兜底）
- 监管可能盯上"permissionless perp"
- Token 化股票可能违反证券法

### 5.5 HIP-4：固定期限合约

**目标**：为期权和结构化产品提供基础设施。

**和永续的区别**：
- 有到期日
- 价格在到期时收敛到某个值
- 适合期权、远期合约、预测市场

**预期上线**：2026 年某个时间点（具体未公布）

---

## 6. 跨链生态：桥与互操作

### 6.1 当前的桥

| 桥 | 链 | 用途 |
|---|---|------|
| **官方 Arbitrum 桥** | Arbitrum ↔ Hyperliquid | USDC 主入金通道 |
| **LayerZero** | 多链 ↔ Hyperliquid | 任意 Token 跨链 |
| **deBridge** | 多链 ↔ Hyperliquid | 主要面向交易者 |
| **Stargate** | 多链 ↔ Hyperliquid | LayerZero 生态 |

### 6.2 LayerZero 的 Composer 模式

LayerZero 和 Hyperliquid 的集成是跨链领域的一个亮点：

**Composer 模式**：
- 用户在源链（比如以太坊）发起一笔交易
- LayerZero 跨链消息 → Hyperliquid
- 在 Hyperliquid 上**自动**完成下单 / 开仓 / 转账
- 一笔交易跨链 + 链上操作

**典型场景**：
```
用户在以太坊上：
  存 USDC → LayerZero → Hyperliquid → 自动开 BTC 多头

全程一笔交易，不需要手动桥过来再开仓。
```

### 6.3 跨链衍生品的未来

Hyperliquid 正在变成**跨链衍生品的结算中心**：
- 其他链上的 DeFi 协议想做永续 → 用 Hyperliquid 做后端
- 用户在熟悉的链上操作 → 实际交易在 Hyperliquid
- Hyperliquid 收手续费，其他链做前端

这是"金融乐高"的下一阶段：**跨链可组合性**。

---

## 7. 生态参与者

### 7.1 用户类型

| 类型 | 数量（估计） | 平均交易量 | 主要诉求 |
|------|------------|-----------|---------|
| **散户交易者** | 200K+ | $10K-100K/月 | 杠杆交易、低费率 |
| **专业交易员** | 5K+ | $1M-100M/月 | 流动性、API、低延迟 |
| **机构** | 50-200 | $10M-1B/月 | 合规、托管、定制服务 |
| **做市商** | 20-50 | $100M-10B/月 | 低费率、API、稳定性 |
| **Vault 存款人** | 50K+ | 取决于 Vault | 被动收益 |
| **Vault 管理者** | 1000+ | 取决于策略 | 用别人的钱赚钱 |

### 7.2 开发者生态

| 角色 | 工作内容 | 收入模式 |
|------|---------|---------|
| **协议开发者** | 在 HyperEVM 上构建 DeFi | Token、TVL 费用 |
| **前端开发者** | 做交易界面 | Builder Codes |
| **机器人开发者** | 交易策略机器人 | 卖订阅、抽佣 |
| **基础设施提供者** | API 节点、数据服务 | 订阅费 |
| **集成方** | 钱包、聚合器集成 Hyperliquid | 流量分润 |

### 7.3 验证者

- 16 个验证者（计划扩展）
- 包括机构验证者（Unit Labs、Figment 等）
- 长期目标：50-100 个验证者

---

## 8. 和其他生态的对比

### 8.1 vs 以太坊 / L2

| 维度 | 以太坊生态 | Hyperliquid 生态 |
|------|-----------|----------------|
| 协议数量 | 数千个 | 数十个 |
| TVL | $100B+ | $1-3B |
| 成熟度 | 高 | 早期 |
| 创新方向 | 全方位 | 围绕交易 |
| Composability | 单链强 / 跨 L2 弱 | 内部强 / 跨链弱 |

### 8.2 vs Solana 生态

| 维度 | Solana | Hyperliquid |
|------|-------|-------------|
| 通用性 | 通用 L1 | 交易专用 |
| DEX 模式 | AMM 主导 | CLOB 原生 |
| Meme 文化 | 强 | 弱 |
| 衍生品 | Drift, Jupiter | 原生集成 |
| 开发者数量 | 数千 | 数百 |

### 8.3 vs dYdX 生态

| 维度 | dYdX | Hyperliquid |
|------|------|-------------|
| 链类型 | Cosmos appchain | 自建 L1 |
| 智能合约 | 无（Cosmos 模块） | 有（HyperEVM） |
| 生态广度 | 窄（只有 dYdX） | 广（DeFi 全栈） |
| 开发者吸引力 | 低 | 中-高 |
| 跨链互操作 | Cosmos IBC | LayerZero |

### 8.4 Hyperliquid 生态的独特优势

1. **订单簿原生集成**：其他链上的 DEX 是合约，Hyperliquid 上是原生功能
2. **原子组合性**：HyperEVM + HyperCore 的组合在其他链上做不到
3. **零外部 Oracle**：合约可以直接读盘口价格
4. **Gas-free 订单**：做市商成本接近零

### 8.5 Hyperliquid 生态的劣势

1. **生态规模小**：协议数量、TVL 都远不如以太坊 / Solana
2. **桥依赖**：资金主要通过 Arbitrum 桥进入，是单点故障
3. **代码不开源**：核心代码不开源，开发者难以独立审计
4. **学习曲线**：HyperEVM 的 precompiles 和异步写需要适应

---

## 9. 生态的未来方向

### 9.1 短期（2026 年）

**1. HyperEVM 成熟化**
- 更多基础设施（索引服务、开发工具、文档）
- 更多 DeFi 协议部署（衍生品、固收、保险）
- TVL 突破 $5-10B

**2. HIP-4 上线**
- 固定期限合约
- 期权产品爆发
- 结构化产品（保险连结票据等）

**3. 机构接入**
- 更多机构验证者
- 合规交易接口（KYC、报告）
- 托管方案

### 9.2 中期（2027-2028）

**1. 跨链衍生品中心**
- Hyperliquid 成为多链 DeFi 的"结算层"
- 类似传统金融的"清算所"角色

**2. AI Agent 经济**
- AI 自动交易在 Hyperliquid 上爆发
- 因为有低成本订单 + 链上可组合 + 可信价格源

**3. RWA（现实世界资产）**
- 通过 HIP-3 上架更多传统资产永续
- 监管沟通和合规化

### 9.3 长期（2028+）

**1. "金融操作系统"**
- Hyperliquid 不再只是交易，而是所有金融操作的底层
- 借贷、支付、保险、衍生品都在上面

**2. 监管对接**
- 可能出现"合规版 Hyperliquid"
- 或者通过技术（zkKYC）实现链上合规

**3. 全球化**
- 更多语言、更多法币入金通道
- 类似 Binance 的全球覆盖，但去中心化

---

## 10. 生态参与的实操建议

### 10.1 如果你是用户

**新手**：
- 从 Hyperliquid 官方 UI 开始
- 小资金试水永续 + 现货
- 先理解保证金和清算机制再加杠杆

**进阶**：
- 探索 Builder Code 前端（可能有更好的 UX 或费率）
- 试试 Vault（HLP 是最稳的入门）
- 关注新上的 HIP-3 市场

### 10.2 如果你是开发者

**前端开发者**：
- 注册 Builder Code
- 选一个细分场景（比如机构、跟单、套利）做差异化
- 集成 Hyperliquid API 比集成 CEX 简单

**协议开发者**：
- HyperEVM 上"借贷 + 永续"组合是蓝海
- 期权 / 结构化产品赛道还没成熟玩家
- 重点学习 precompiles 和异步写模式

**基础设施开发者**：
- 索引、数据、Oracle、跨链桥都有需求
- 工具链（SDK、IDE 插件、调试器）相对薄弱

### 10.3 如果你是 Builder

**HIP-3 Builder**：
- 100 万 HYPE 质押门槛较高（~$50M+）
- 适合机构或大规模做市商
- 选择对的市场（流动性 + 监管 + 长期需求）很关键

**社区 Builder**：
- 教育内容、研究报告、数据看板都有需求
- 通过 Builder Codes 变现
- 长期价值大于短期分润

---

## 11. 总结

Hyperliquid 的生态正在从"一个永续 DEX"演化成"一个金融基础设施层"。

**核心论点**：

1. **HyperEVM 是关键转折点**：让 Hyperliquid 从产品变成平台
2. **原子组合性是独特优势**：其他链上做不到"合约 + 订单簿"的原子交互
3. **HIP-3 是革命性创新**：第一次"任何人可以上市场"
4. **Builder Codes 是分销利器**：让前端创业者有商业模式
5. **跨链是下一个战场**：Hyperliquid 想成为多链 DeFi 的结算中心

**风险**：
- 生态规模仍远小于以太坊 / Solana
- 协议数量和 TVL 都需要继续增长
- 监管可能打击 HIP-3 这种 permissionless 模式

**判断**：
Hyperliquid 的生态战略是**正确的**，但**执行需要时间**。2026-2028 年是关键观察窗口 —— 如果 HyperEVM 上能跑出几个杀手级应用，Hyperliquid 就真正确立了"通用金融链"的地位。如果跑不出，它会退回到"一个很好的永续 DEX"。

---

*上一篇：[Token 经济 ←](./05-tokenomics.md) · 下一篇：[竞争格局 →](./07-competition.md)*
