# BFT 共识背景：从 PBFT 到 Hotstuff

> 理解 HyperBFT 之前，必须先理解它的家族谱系。这篇梳理 BFT 共识 30 年的演化，看清每一代解决了什么、留下了什么。

---

## 1. 什么是拜占庭容错（BFT）

### 1.1 拜占庭将军问题

1982 年，Leslie Lamport 提出"拜占庭将军问题"：

> 几支拜占庭军队围攻一座城市，将军们只能通过信使通信。需要达成共识（一起进攻或一起撤退）。但其中可能有叛徒（发送矛盾消息）。问题是：忠诚的将军如何在有叛徒的情况下达成一致？

**映射到分布式系统**：
- 将军 = 节点（验证者）
- 叛徒 = 恶意/故障节点
- 达成共识 = 对区块内容达成一致
- 信使 = 网络通信

### 1.2 BFT 的核心结论

**关键定理**：在 n 个节点中，如果恶意节点数 f 满足 `n ≥ 3f + 1`，系统可以达成共识。

换句话说：**容忍 f 个恶意节点，需要至少 3f+1 个总节点**。

| 总节点 n | 最大容忍恶意节点 f | 需要的诚实节点 |
|---------|------------------|--------------|
| 4 | 1 | 3 |
| 7 | 2 | 5 |
| 16 | 5 | 11 |
| 100 | 33 | 67 |

**为什么是 3f+1**：
- 要做决定，需要收到 n-f 个回复（不能等恶意节点）
- 这 n-f 个回复里可能有 f 个是恶意的
- 所以真正诚实的回复至少 n-2f 个
- 要保证诚实回复占多数：n-2f > f → n > 3f → n ≥ 3f+1

这个结论解释了为什么 Hyperliquid 的 16 个验证者"6 个串通就能破坏共识"（因为 16 = 3×5+1，容忍 5 个，第 6 个就超了）。

---

## 2. 第一代：PBFT（1999）

### 2.1 PBFT 是什么

**PBFT（Practical Byzantine Fault Tolerance）**由 Castro 和 Liskov 在 1999 年提出。它是第一个"实用"的 BFT 算法（之前的 BFT 算法太慢，没法用）。

### 2.2 PBFT 的三阶段

```
客户端请求
    │
    ▼
1. Pre-Prepare（预准备）
   主节点广播提议给所有副本
    │
    ▼
2. Prepare（准备）
   每个副本广播"我收到了提议"给所有其他副本
   收到 2f 个 Prepare → 进入下一阶段
    │
    ▼
3. Commit（提交）
   每个副本广播"我准备好了"给所有其他副本
   收到 2f+1 个 Commit → 执行并回复客户端
```

### 2.3 PBFT 的问题

**通信复杂度 O(n²)**：

- Prepare 阶段：每个节点要给其他所有节点发消息 → n×n 次通信
- Commit 阶段：又是 n×n 次通信
- 总计 O(n²)

**这意味着**：
- 4 个节点：16 次通信，没问题
- 100 个节点：10,000 次通信，开始吃力
- 1000 个节点：1,000,000 次通信，不可行

**结论**：PBFT 只能支持几十个节点。这是后续所有改进的出发点。

---

## 3. 第二代：Tendermint（2014）

### 3.1 Tendermint 的改进

**Tendermint**（Cosmos 的底层共识）是 PBFT 的工程优化版本。

**核心改进**：
- 把 BFT 共识和区块链结合（PBFT 原本是为状态机复制设计的）
- 引入"锁定（locking）"机制保证安全性
- 简化了视图切换（view change）

### 3.2 Tendermint 的特点

| 特性 | 说明 |
|------|------|
| 通信复杂度 | 仍是 O(n²)（但工程优化后实际可用） |
| 验证者数量 | 几十到 ~175（Cosmos Hub） |
| 最终性 | 即时最终性（一旦提交不可逆） |
| 出块时间 | ~6 秒（Cosmos Hub） |

### 3.3 谁在用 Tendermint

- **Cosmos Hub**：175 个验证者
- **dYdX V4**：~60 个验证者（用 CometBFT，Tendermint 的后继）
- **Binance Chain**：早期版本

**dYdX V4 用 Tendermint 家族 → 这解释了它 ~6 秒的最终性**。Tendermint 的 O(n²) 通信在 60 个验证者下就会带来明显延迟。

---

## 4. 第三代：Hotstuff（2018）

### 4.1 Hotstuff 的突破

**Hotstuff**由 VMware Research 在 2018 年提出。Facebook 的 Libra/Diem 项目采用了它的变体。

**核心突破：把通信复杂度从 O(n²) 降到 O(n)**。

### 4.2 Hotstuff 怎么做到 O(n)

**关键技巧：用"星形通信"代替"网状通信"**。

```
PBFT（网状，O(n²)）：
  每个节点 ←→ 每个节点
  节点 A ←→ B, A ←→ C, A ←→ D, B ←→ C, B ←→ D, C ←→ D...

Hotstuff（星形，O(n)）：
  所有节点 → Leader → 所有节点
  节点只和 Leader 通信，Leader 聚合后广播
```

**具体机制**：
1. 节点把投票发给 Leader（n 次通信）
2. Leader 用**门限签名（threshold signature）**把 n 个投票聚合成 1 个证明
3. Leader 把聚合证明广播给所有节点（n 次通信）
4. 总计 2n 次通信 = O(n)

**门限签名是关键**：n 个验证者的签名可以聚合成一个固定大小的签名，验证一次就等于验证了所有签名。

### 4.3 Hotstuff 的四阶段

```
1. Prepare（准备）
   Leader 提议 → 节点投票 → Leader 聚合成 QC（Quorum Certificate）
    │
    ▼
2. Pre-Commit（预提交）
   Leader 广播 Prepare QC → 节点投票 → Leader 聚合
    │
    ▼
3. Commit（提交）
   Leader 广播 Pre-Commit QC → 节点投票 → Leader 聚合
    │
    ▼
4. Decide（决定）
   Leader 广播 Commit QC → 节点执行
```

**QC（Quorum Certificate）**：一组投票的聚合证明，证明"至少 2f+1 个节点同意了"。

### 4.4 Hotstuff 的流水线化（Pipelining）

Hotstuff 最巧妙的地方：**四个阶段可以流水线重叠**。

```
区块 1：  Prepare → PreCommit → Commit → Decide
区块 2：           Prepare → PreCommit → Commit → Decide
区块 3：                    Prepare → PreCommit → Commit → Decide
```

每个阶段的 QC 同时作为下一个区块的"推进信号"。这样：
- 不需要等一个区块完全确认才开始下一个
- 吞吐量大幅提升
- 这是 HyperBFT 高吞吐的基础

### 4.5 Hotstuff 家族

| 变体 | 项目 | 特点 |
|------|------|------|
| Hotstuff | 学术原型 | 原始版本 |
| LibraBFT/DiemBFT | Meta Libra/Diem | 加入 pacemaker |
| AptosBFT | Aptos | Diem 后继 |
| Narwhal-Bullshark | Sui (Mysten) | DAG-based mempool |
| **HyperBFT** | **Hyperliquid** | **流水线 + 解耦优化** |

---

## 5. 共识算法对比总表

| 算法 | 年份 | 通信复杂度 | 验证者上限 | 最终性 | 代表项目 |
|------|------|-----------|-----------|--------|---------|
| PBFT | 1999 | O(n²) | ~10-30 | 即时 | 学术/联盟链 |
| Tendermint | 2014 | O(n²) | ~100-175 | 即时 | Cosmos |
| Hotstuff | 2018 | **O(n)** | ~100-1000 | 即时 | Diem, Aptos |
| **HyperBFT** | 2023 | **O(n)** | 16（可扩展） | **0.2s** | Hyperliquid |

---

## 6. 为什么 Hyperliquid 选 Hotstuff 家族

### 6.1 性能需求

交易所对延迟极度敏感：
- CEX 撮合是微秒级
- 链上交易所要尽量逼近
- O(n²) 的 Tendermint 做不到亚秒级（dYdX V4 是 6 秒）
- O(n) 的 Hotstuff 可以

### 6.2 流水线 = 高吞吐

- 订单簿交易量大（每秒几万笔）
- Hotstuff 的流水线设计天然适合高吞吐
- 多个区块并发推进

### 6.3 团队背景

- Hyperliquid 团队是量化交易背景
- 对延迟和吞吐有极致追求
- Hotstuff 是当时最适合"为交易优化"的共识

### 6.4 代价

选 Hotstuff 家族也有代价：
- 仍然需要"少验证者"才能做到亚秒延迟（16 个）
- Leader 是潜在的瓶颈和中心化点
- 门限签名增加了密码学复杂度

---

## 7. 小结

1. **BFT 的 3f+1 定理**决定了"容忍 f 个恶意节点需要 3f+1 个总节点"
2. **PBFT 的 O(n²)** 限制了验证者数量（几十个）
3. **Tendermint** 工程优化但仍是 O(n²)（Cosmos、dYdX V4）
4. **Hotstuff 的 O(n)** 是关键突破（门限签名 + 星形通信）
5. **Hotstuff 的流水线**让高吞吐成为可能
6. **HyperBFT 选 Hotstuff 家族**是为了亚秒延迟 + 高吞吐，代价是验证者数量少

下一篇：[HyperBFT 的具体设计 →](./02-hyperbft-design.md)

---

> 📚 **延伸阅读**：
> - [Hotstuff 论文](https://arxiv.org/abs/1803.05069)（VMware Research, 2018）
> - [PBFT 论文](https://pmg.csail.mit.edu/papers/osdi99.pdf)（Castro & Liskov, 1999）
> - [Tendermint 文档](https://docs.cometbft.com/)
