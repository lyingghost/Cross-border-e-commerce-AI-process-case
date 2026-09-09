# 跨境电商 Agent 协同体系

> 阶段：0→1 轻资产试运营 · 团队：2 人 × 5 小时/天 · 模式：AI Copilot（建议制，人工执行）

## 这是什么

一套让 **8 个专职 Agent + 3 个支撑 Agent** 协同运作的跨境电商经营系统。

关键设计：Agent 之间**不靠互相聊天**，而是读写同一块「共享黑板」。
这是它们能真正协同、而不是 8 个孤立 prompt 的核心。

```
         CEO Agent（目标拆解 / 预算分配 / 冲突仲裁）
                        ↓
    Market Agent      Competitor Agent        ← 情报层
                        ↓
              Product Agent（选品 / 利润 / 供应链）   ← 供给层
                        ↓
     Store Agent      Content Agent      Ads Agent   ← 转化层
                        ↓
                 Customer Agent                     ← 留存层
                        ↓
   ┌────────────────────────────────────────────┐
   │  协同底座：共享黑板 · 事件总线 · 审批队列 · 知识库  │
   └────────────────────────────────────────────┘
                        ↑ 数据回流闭环 → CEO
```

## 硬约束：轻资产试运营（所有 Agent 必须遵守）

| 编号 | 约束 | 具体口径 |
|---|---|---|
| C1 | 单 SKU 首批投入上限 | ≤ ¥3,000（样品 + 首批货 + 素材 + 广告测试预算） |
| C2 | 库存优先 | 优先国内直发 / 平台全托管·半托管 / 一件代发；**压货必须人工审批** |
| C3 | 类目禁区 | 不碰带电、液体、食品、儿童用品、医疗器械、有侵权风险的品牌类目 |
| C4 | 现金流优先 | 任何决策先回答「最坏情况亏多少、多久能回本」 |
| C5 | 人力预算 | 2 人 × 5h/天 = **50 人时/周**。任何流程若周占用 > 10 人时，必须简化或交给 Agent |

> C5 是这套系统存在的理由：你们只有 50 人时/周，Agent 必须吃掉重复性工作。

## 当前阶段：Phase 0 — 市场与平台选型

市场、平台、品类都还没定，所以**不一次性上 8 个 Agent**，按阶段滚雪球：

| 阶段 | 上线 Agent | 核心产出 | 预计周期 |
|---|---|---|---|
| **Phase 0** | Market | 《市场-平台-品类选型决策报告》 | 1–2 周 |
| Phase 1 | + Competitor, Product | 候选 SKU 清单 + 利润测算 + 供应商短名单 | 2–4 周 |
| Phase 2 | + Store, Content, Ads | 首批 Listing / 素材 / 投放计划 | 4–8 周 |
| Phase 3 | + Customer, Finance, Compliance | 客服 SOP / 利润看板 / 合规清单 | 持续 |
| Phase 4 | 全量 | 定时编排 + 事件联动闭环 | 8 周后 |

**现在唯一需要跑的流程**：[`workflows/phase0-market-selection.md`](workflows/phase0-market-selection.md)

## 马上开始（3 步）

1. 读 [`ARCHITECTURE.md`](ARCHITECTURE.md) 了解全局（10 分钟）
2. 打开 [`workflows/phase0-market-selection.md`](workflows/phase0-market-selection.md)，按步骤驱动 Market Agent
3. 产出落到 `data/output/phase0/`，选型拍板后再进 Phase 1

## 目录

```
.
├── README.md              本文件
├── ARCHITECTURE.md        架构设计（分层 / 职责 / 协同机制 / 数据流）
├── ROADMAP.md             分阶段路线图与验收标准
├── CONTRIBUTING.md        2 人协作规范（每日节奏 / Git 流程 / 命名）
├── agents/                Agent 定义（每个一个文件）
│   ├── _TEMPLATE.md       新增 Agent 的模板
│   ├── ceo.md  market.md  competitor.md  product.md
│   ├── store.md  content.md  ads.md  customer.md
│   └── finance.md  compliance.md        （支撑 Agent）
├── shared/                协同协议
│   ├── blackboard.md      共享黑板 schema 与读写规则
│   ├── events.md          事件总线：事件 → 订阅 Agent → 动作
│   ├── approval-matrix.md Copilot 模式动作分级
│   └── kpi.md             指标字典与计算口径
├── workflows/             可执行流程
│   ├── phase0-market-selection.md   ← 现在跑这个
│   ├── phase1-product-screening.md
│   ├── daily.md           每日 30 分钟节奏
│   └── weekly.md          每周经营会
└── data/
    ├── README.md          数据分层与接入计划
    └── templates/         CSV 模板（市场扫描 / SKU 候选）
```

## 文件放哪里

| 内容类型 | 存放位置 | 原因 |
|---|---|---|
| Agent 定义、schema、SOP、决策记录 | **GitHub** | 需要版本管理、可 diff、可被程序读取 |
| 业务文档、数据表、会议纪要、素材排期 | **腾讯文档** | 非技术成员友好，支持多人实时协作 |
| 原始导出数据、报表 | `data/` 目录 | 单一事实来源，Agent 直接读 |
