# 共享黑板（Blackboard）

所有 Agent 唯一的读写对象。文件位置：`data/blackboard.json`。

## Schema

```jsonc
{
  "meta": {
    "updated_at": "2026-09-09T13:00:00+08:00",
    "updated_by": "market",
    "version": 1
  },

  "strategy": {              // owner: CEO（Phase 0 由人代填）
    "target_market": null,   // 例："美国"
    "platform": null,        // 例："TikTok Shop"
    "category": null,        // 例："家居收纳"
    "monthly_gmv_goal": null,
    "cash_available": null,  // 可用于试运营的资金
    "risk_appetite": "light" // 固定为 light（轻资产）
  },

  "market": {                // owner: Market
    "candidates": [],        // 候选「市场×平台×品类」组合
    "scores": {},            // 组合 → 评分明细
    "selected": null,        // 拍板后的组合
    "trend_signals": []
  },

  "competitor": {            // owner: Competitor
    "tracked": [],           // 监控中的竞品
    "price_bands": [],
    "alerts": []
  },

  "product": {               // owner: Product
    "candidates": [],        // 候选 SKU
    "shortlist": [],         // 短名单
    "costs": {},             // SKU → 成本结构
    "suppliers": []
  },

  "store": {                 // owner: Store
    "listings": [],
    "keywords": { "positive": [], "negative": [] },
    "prices": {}
  },

  "content": {               // owner: Content
    "scripts": [],
    "assets": [],
    "creators": []
  },

  "ads": {                   // owner: Ads
    "campaigns": [],
    "daily_budget": null,
    "acos": null,
    "tacos": null
  },

  "customer": {              // owner: Customer
    "reviews": [],
    "return_reasons": [],
    "tickets": [],
    "repeat_rate": null
  },

  "finance": {               // owner: Finance
    "unit_profit": {},       // SKU → 单件净利
    "cash_flow": [],
    "payback_days": {}
  },

  "events":    [],           // 事件总线
  "approvals": [],           // 待人工审批队列
  "decisions": []            // 决策留痕
}
```

## 读写规则

1. **只写自己的 namespace**。例如 Market Agent 只能写 `market.*`，不能改 `product.*`。
2. **只读其他 namespace**。需要别人改数据时，往 `events` 丢事件，而不是自己动手。
3. **每次写入必须带三个字段**：
   ```jsonc
   { "updated_at": "<ISO8601>", "updated_by": "<agent 名>", "evidence": "<数据来源>" }
   ```
   `evidence` 不能为空。写"我估计的"也是有效 evidence，但必须写明，不能假装是数据。
4. **冲突以 owner 为准**。两个 Agent 对同一事实给出不同值时，以 owner Agent 的为准；异议写进 `events`。
5. **Phase 0–2 黑板由人维护**。Agent 输出 Markdown，人负责搬进 JSON。Phase 3 起再让 Agent 直接写。

## 事件对象

```jsonc
{
  "id": "evt-20260909-001",
  "type": "sku.shortlisted",
  "emitted_by": "product",
  "emitted_at": "2026-09-09T13:00:00+08:00",
  "payload": { "sku_ids": ["SKU-003"], "reason": "毛利率 42%，供应商可 30 件起订" },
  "subscribers": ["store", "content", "finance"],
  "status": "pending"        // pending → claimed → done
}
```

## 审批对象

```jsonc
{
  "id": "apr-20260909-001",
  "level": "C",              // A / B / C，见 approval-matrix.md
  "proposed_by": "product",
  "action": "下单首批 50 件",
  "amount": 2800,
  "rationale": "样品已验收，供应商账期可接受",
  "worst_case": "全部滞销，亏损 ¥2,800",
  "status": "pending",       // pending → approved / rejected
  "decided_by": null,
  "decided_at": null
}
```

## 决策留痕对象

```jsonc
{
  "id": "dec-20260909-001",
  "topic": "选择 TikTok Shop 美区作为首发平台",
  "assumption": "内容电商冷启动门槛低于 Amazon，广告成本低",
  "evidence": "market-scan.csv 中该组合评分 78/100，竞品密度中等",
  "conclusion": "首发 TikTok Shop 美区，家居收纳类目",
  "worst_case": "3 个月无单，损失约 ¥5,000（样品+素材+广告）",
  "review_date": "2026-12-09",
  "outcome": null            // 复盘时填
}
```
