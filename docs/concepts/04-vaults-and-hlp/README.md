# Vault 系统与 HLP：深度解析

> Vault 是 Hyperliquid 最活跃的产品形态，HLP 是其中最核心的官方 Vault。这个目录拆解 Vault 系统的设计、HLP 的运作、以及各种策略和风险。

---

## 阅读顺序

| 文档 | 内容 | 难度 |
|------|------|:---:|
| [01-vault-system.md](./01-vault-system.md) | Vault 系统：什么是 Vault、如何运作、vs GLP | 🟡 进阶 |
| [02-hlp-deep-dive.md](./02-hlp-deep-dive.md) | HLP 深度拆解：三重角色、收益来源、风险 | 🔴 深入 |
| [03-vault-strategies-and-risks.md](./03-vault-strategies-and-risks.md) | Vault 策略类型、收益分析、风险评估 | 🟡 进阶 |

---

## 前置知识

- [basics/01-platform-overview.md](../../basics/01-platform-overview.md) 第 4.3 节 —— Vault 概览
- [03-clearing-and-margin/03-insurance-fund-and-adl.md](../03-clearing-and-margin/03-insurance-fund-and-adl.md) —— HLP 的兜底角色

## 核心问题

1. 什么是 Vault？和 GMX 的 GLP 有什么区别？
2. HLP 怎么运作？它的收益和风险从哪来？
3. 为什么 HLP 是"中央对手方"？这意味着什么？
4. 有哪些 Vault 策略？收益和风险如何？
5. 存钱到 Vault 前应该理解什么？

---

> ⚠️ Vault 的历史收益不代表未来，本目录的收益数字是历史参考，不构成投资建议。
