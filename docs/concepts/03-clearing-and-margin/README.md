# 清算与保证金：深度解析

> 永续合约的核心是杠杆，杠杆的核心是保证金和清算。这个目录拆解 Hyperliquid 的保证金模型、清算机制、Insurance Fund 和 ADL。

---

## 阅读顺序

| 文档 | 内容 | 难度 |
|------|------|:---:|
| [01-margin-model.md](./01-margin-model.md) | 保证金模型：Cross/Isolated、初始/维持保证金 | 🟡 进阶 |
| [02-liquidation.md](./02-liquidation.md) | 清算机制：触发、标记价格、清算流程 | 🔴 深入 |
| [03-insurance-fund-and-adl.md](./03-insurance-fund-and-adl.md) | Insurance Fund、ADL、HLP 的兜底角色 | 🔴 深入 |

---

## 前置知识

- [basics/04-trading-mechanism.md](../../basics/04-trading-mechanism.md) —— 杠杆、保证金、清算基础
- [basics/08-risks-and-events.md](../../basics/08-risks-and-events.md) —— JELLY 事件（清算的极端案例）

## 核心问题

1. Cross 和 Isolated 保证金的本质区别？
2. 清算什么时候触发？怎么计算清算价格？
3. 为什么用标记价格而非最新价？
4. Insurance Fund 怎么运作？什么时候触发 ADL？
5. HLP 在清算中扮演什么角色？为什么是风险点？

---

> ⚠️ 具体的保证金参数和清算阈值随市场和时间变化，本目录的数字是示例，以官方实时数据为准。
