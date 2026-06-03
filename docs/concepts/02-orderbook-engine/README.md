# 订单簿引擎：深度解析

> Hyperliquid 最核心的技术创新：全链上订单簿（On-chain CLOB）。这个目录拆解它怎么实现、为什么下单免费、一笔订单的完整生命周期。

---

## 阅读顺序

| 文档 | 内容 | 难度 |
|------|------|:---:|
| [01-onchain-clob.md](./01-onchain-clob.md) | 全链上 CLOB：为什么之前不可能，Hyperliquid 怎么做到 | 🟡 进阶 |
| [02-matching-and-gasfree.md](./02-matching-and-gasfree.md) | 撮合引擎 + Gas-free 订单的实现原理 | 🔴 深入 |
| [03-order-lifecycle.md](./03-order-lifecycle.md) | 一笔订单从提交到成交的完整路径 | 🟡 进阶 |

---

## 前置知识

- [basics/03-technical-architecture.md](../../basics/03-technical-architecture.md) —— HyperCore 在架构中的位置
- [basics/04-trading-mechanism.md](../../basics/04-trading-mechanism.md) —— 订单类型、撮合规则
- [01-hyperbft-consensus/](../01-hyperbft-consensus/) —— 确定性执行（订单簿的基础）

## 核心问题

1. 为什么"链上订单簿"被认为不可能？Serum 为什么失败？
2. Hyperliquid 怎么做到全链上 CLOB？
3. 为什么下单/撤单不消耗 Gas？
4. 撮合引擎是怎么工作的？
5. 一笔订单的完整生命周期是什么？

---

> ⚠️ Hyperliquid 核心代码不开源，本目录的实现细节基于官方文档和合理推测。
