# HyperBFT 共识：深度解析

> 这个目录把 Hyperliquid 的共识层拆成 4 个知识点。从 BFT 共识的历史背景，到 HyperBFT 的具体设计，再到验证者经济和确定性执行。
>
> 适合：想理解"为什么 Hyperliquid 能做到 0.2 秒最终性"的工程师和研究员。

---

## 阅读顺序

| 文档 | 内容 | 难度 |
|------|------|:---:|
| [01-bft-background.md](./01-bft-background.md) | BFT 共识谱系：PBFT → Tendermint → Hotstuff | 🟢 入门 |
| [02-hyperbft-design.md](./02-hyperbft-design.md) | HyperBFT 的具体设计：流水线、解耦、乐观执行 | 🟡 进阶 |
| [03-validators-and-staking.md](./03-validators-and-staking.md) | 验证者集合、质押、委托、slashing | 🟡 进阶 |
| [04-finality-and-determinism.md](./04-finality-and-determinism.md) | 单区块最终性、确定性执行、无 MEV | 🔴 深入 |

---

## 前置知识

阅读这个目录前，建议先读：
- [basics/03-technical-architecture.md](../../basics/03-technical-architecture.md) —— 三层架构总览

## 核心问题

这个目录回答：
1. BFT 共识是什么？为什么 Hyperliquid 选 Hotstuff 家族？
2. HyperBFT 在 Hotstuff 基础上做了什么改进？
3. 为什么只有 16 个验证者？这是技术限制还是运营选择？
4. "确定性执行"和"单区块最终性"到底意味着什么？
5. 为什么 Hyperliquid 上没有 MEV？

---

> ⚠️ **关于信息来源**：Hyperliquid 的核心代码不开源，本目录的很多技术细节基于官方文档、白皮书、第三方分析和合理推测。推测的地方会明确标注。
