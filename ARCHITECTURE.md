# 架构设计

## 1. 设计原则

| 编号 | 原则 | 为什么 |
|---|---|---|
| P1 | **黑板优于对话** | Agent 两两对话会产生 N² 条链路且无法审计。所有 Agent 读写同一块结构化状态，链路变成 N 条且可追溯 |
| P2 | **契约优于自由文本** | 每个 Agent 的输入/输出都有 schema，下游才能稳定消费。自由文本只能给人看，不能给程序用 |
| P3 | **Copilot 优先** | 0→1 阶段放权 = 自杀。Phase 0–2 Agent 只出方案，人摁按钮。验证有效后再逐步放权 |
| P4 | **单一事实来源** | 黑板上每个字段只有一个 owner Agent 可写，其他 Agent 只读。避免两个 Agent 给出矛盾的价格 |
| P5 | **决策留痕** | 每个决策记录 `假设 / 证据 / 结论 / 复盘日期`。错了能追溯是哪一步的判断出问题 |

## 2. 分层与职责卡

图例：**P** = 上线阶段

### 决策层

| Agent | 一句话职责 | 主要输入 | 主要输出 | 核心指标 | P |
|---|---|---|---|---|---|
| **CEO** | 把月度目标拆成各 Agent 的 OKR，分配预算与人力，仲裁冲突 | 黑板全局状态、各 Agent 汇报、资金水位 | 目标分解表、预算分配、优先级排序、风险预警 | 净利润、现金周转天数 | 0 |

### 情报层（回答「卖什么、卖给谁」）

| Agent | 一句话职责 | 主要输入 | 主要输出 | 核心指标 | P |
|---|---|---|---|---|---|
| **Market** | 扫描市场/平台/品类机会，输出选型建议 | 平台公开数据、趋势数据、类目规模、政策 | 《市场-平台-品类选型报告》、机会清单 | 机会评分、TAM、增速 | **0** |
| **Competitor** | 持续监控竞品的价格、排名、评论、上新、促销 | 竞品 ASIN/店铺清单、搜索词 | 竞品拆解卡、价格带地图、差评机会点 | 监控覆盖率、异动响应时长 | 1 |

### 供给层（回答「从哪来、赚多少」）

| Agent | 一句话职责 | 主要输入 | 主要输出 | 核心指标 | P |
|---|---|---|---|---|---|
| **Product** | 选品打分、利润测算、供应商筛选与谈判要点 | 机会清单、1688/供应链数据、物流报价、关税 | SKU 候选清单、单品利润表、供应商短名单 | 毛利率、ROI、打样通过率 | 1 |

### 转化层（回答「怎么卖出去」）

| Agent | 一句话职责 | 主要输入 | 主要输出 | 核心指标 | P |
|---|---|---|---|---|---|
| **Store** | Listing 撰写、关键词布局、定价、SEO | SKU 定稿信息、关键词库、竞品 Listing | 标题/五点/描述/A+、关键词表、定价建议 | 曝光量、CTR、转化率 | 2 |
| **Content** | 短视频脚本、图文素材、达人建联、素材排期 | 卖点、竞品素材、平台热榜 | 脚本、分镜、素材清单、达人名单 | 播放完成率、素材产出量 | 2 |
| **Ads** | 站内/站外投放、预算分配、出价与素材 AB | Listing、素材、目标 ACOS、日预算 | 广告结构、词库、出价方案、日报 | ACOS、TACOS、ROAS | 2 |

### 留存层（回答「怎么留住、怎么复购」）

| Agent | 一句话职责 | 主要输入 | 主要输出 | 核心指标 | P |
|---|---|---|---|---|---|
| **Customer** | 售前售后话术、评价管理、退货归因、复购触达 | 订单、评价、客服会话、退货原因 | 话术库、差评应对、退货归因报告、复购方案 | 差评率、退货率、复购率 | 3 |

### 支撑层（横切）

| Agent | 一句话职责 | 主要输出 | P |
|---|---|---|---|
| **Finance** | 单品利润、现金流、汇率、税费测算 | 利润看板、回本周期、现金流预警 | 3 |
| **Compliance** | 平台政策、认证要求、知识产权、侵权排查 | 合规清单、风险预警 | 3 |
| **Ops（由 CEO 兼任）** | 任务编排、SLA、日报周报汇总 | 日报、周报、待办队列 | 0 |

## 3. 协同机制

### 3.1 共享黑板（Blackboard）

所有 Agent 唯一的读写对象。物理形态：`shared/blackboard.md` 定义的 JSON 结构，存于仓库 `data/blackboard.json`。

```jsonc
{
  "meta":      { "updated_at": "", "updated_by": "", "version": 1 },
  "strategy":  { "target_market": null, "platform": null, "category": null,
                 "monthly_gmv_goal": null, "cash_available": null },   // owner: CEO
  "market":    { "opportunities": [], "trend_signals": [] },           // owner: Market
  "competitor":{ "tracked": [], "price_bands": [], "alerts": [] },     // owner: Competitor
  "product":   { "candidates": [], "shortlist": [], "costs": {} },     // owner: Product
  "store":     { "listings": [], "keywords": {}, "prices": {} },       // owner: Store
  "content":   { "scripts": [], "assets": [], "creators": [] },        // owner: Content
  "ads":       { "campaigns": [], "acos": null, "budget": {} },        // owner: Ads
  "customer":  { "reviews": [], "return_reasons": [], "tickets": [] }, // owner: Customer
  "events":    [],   // 事件总线，见 3.2
  "approvals": [],   // 待人工审批队列，见 3.3
  "decisions": []    // 决策留痕，见 P5
}
```

**读写规则**
- 每个 Agent 只写自己的 namespace，可读全部
- 写入必须带 `updated_at` / `updated_by` / `evidence`（数据来源）
- 冲突时以 owner 字段为准，不服的写进 `events` 由 CEO 仲裁

### 3.2 事件总线

Agent 不主动催别的 Agent，而是往 `events` 里丢事件，订阅方自行响应。

| 事件 | 触发条件 | 订阅 Agent | 动作 |
|---|---|---|---|
| `market.selected` | 市场/平台拍板 | Competitor, Product | 开始建竞品清单、类目扫描 |
| `sku.shortlisted` | SKU 进入短名单 | Store, Content, Finance | 起草 Listing、准备素材、算利润 |
| `listing.live` | Listing 上线 | Ads, Content | 开广告、排素材 |
| `acos.spike` | ACOS 超过阈值 X% | CEO, Ads, Store | 降预算 / 查词 / 查转化率 |
| `review.negative` | 出现 1–2 星评价 | Customer, Product, Store | 归因、改 Listing、改包装 |
| `stock.low` | 可售天数 < N | CEO, Ads, Product | 降广告、触发补货 |
| `price.war` | 竞品降价 > X% | CEO, Store | 重新测算价格底线 |

完整定义见 [`shared/events.md`](shared/events.md)。

### 3.3 审批队列（Copilot 模式）

Phase 0–2 全部动作**默认需人工确认**。分级见 [`shared/approval-matrix.md`](shared/approval-matrix.md)。

三档：**A 自动执行**（几乎无风险） / **B 建议 + 一键确认**（默认） / **C 必须人工决策**（花钱、压货、改价、上新）。

### 3.4 知识库（RAG）

沉淀四类内容，供所有 Agent 检索：
1. 品牌与产品知识（卖点、参数、禁忌表述）
2. 平台规则与类目合规
3. 历史决策与复盘
4. SOP 与话术库

存放：GitHub `docs/` + 腾讯文档知识空间。

## 4. 数据流

```
外部数据 ──▶ [ 采集/整理 ] ──▶ data/raw ──▶ Agent 消费 ──▶ 写入黑板
                                                              │
                                                              ▼
   人工决策 ◀── 审批队列 ◀── Agent 产出方案 ◀──── 事件触发
        │
        └──▶ 执行（平台后台操作）──▶ 结果数据回流 ──▶ 复盘 ──▶ 知识库
```

**Phase 0 简化版**（现在适用）：

```
公开市场数据 ──▶ Market Agent ──▶ 选型报告 ──▶ 你拍板 ──▶ strategy 字段落黑板
```

## 5. 技术选型

| 层 | 选型 | 理由 |
|---|---|---|
| 代码 / Agent 定义 | GitHub | 版本管理、可 diff、Agent 可被程序读取 |
| 业务文档 / 数据表 | 腾讯文档 | 非技术成员友好，实时协作 |
| 运行时（Phase 0–1） | 人工在 WorkBuddy 内触发 | 2 人小团队不需要编排引擎，先验证价值 |
| 运行时（Phase 2+） | 定时自动化 + 事件编排 | 流程稳定后再自动化 |
| 数据源（0→1） | 公开数据 + 手工 CSV | **不采购付费工具**，验证出单后再接 |
| 模型 | WorkBuddy 当前模型 | 先不定预算，跑两周看实际消耗再设护栏 |

> 刻意不做的事：不上向量数据库、不上编排框架、不自建后端、不买数据工具。
> 0→1 阶段这些全是负资产。

## 6. 成本护栏

- Agent 产出必须先过 C1（单 SKU ≤ ¥3,000）和 C4（最坏情况亏损）检验
- 每月复盘一次 token/时间成本 vs 节省的人时，若净收益为负则精简 Agent
- 任何"要不要买 XX 工具"的决策，走 `decisions` 留痕，并设定验证期

## 7. 演进路径

1. **Phase 0–1：人肉编排**。Agent 是高级分析师，人做所有决策。
2. **Phase 2：流程固化**。把跑通的流程写成 `workflows/*.md`，开始可复用。
3. **Phase 3：半自动**。低风险动作（改词、小幅调价）放权给 Agent。
4. **Phase 4：事件驱动闭环**。定时任务 + 事件总线自动运转，人只看异常和审批。
