# 事件总线

Agent 不主动催别的 Agent。它们往 `events` 里丢事件，订阅方在下次运行时自行消费。

## 事件清单

| 事件 | 触发条件 | 发布者 | 订阅 Agent | 期望动作 |
|---|---|---|---|---|
| `market.selected` | `strategy.platform` 与 `category` 被填满 | CEO / 人 | Competitor, Product | 建立竞品监控清单；开始类目 SKU 扫描 |
| `competitor.price_drop` | 竞品降价 > 10% | Competitor | CEO, Store | 重算价格底线，判断跟不跟 |
| `competitor.new_launch` | 监控竞品上新 | Competitor | Product, Market | 评估是否为新趋势信号 |
| `sku.shortlisted` | SKU 进入短名单 | Product | Store, Content, Finance | 起草 Listing；准备素材；算单品利润 |
| `sample.approved` | 样品验收通过 | 人 / Product | Store, Ads | 开 Listing；准备冷启动预算 |
| `listing.live` | Listing 上架成功 | Store | Ads, Content | 建广告结构；排素材计划 |
| `acos.spike` | ACOS > 目标值 × 1.5，持续 3 天 | Ads | CEO, Ads, Store | 降预算 / 清洗词 / 查落地页转化 |
| `acos.excellent` | ACOS < 目标值 × 0.6，持续 3 天 | Ads | CEO, Ads | 建议加预算放大 |
| `conversion.drop` | 转化率环比下降 > 30% | Store | Ads, Content, Customer | 查 Listing、素材、近期差评 |
| `review.negative` | 出现 1–2 星评价 | Customer | Product, Store, Content | 归因（产品/描述/物流），改对应环节 |
| `return.spike` | 退货率 > 类目均值 × 1.5 | Customer | Product, Store | 查品质与描述一致性 |
| `stock.low` | 可售天数 < 14 天 | Product / 人 | CEO, Ads | 降广告避免断流；触发补货决策 |
| `stock.out` | 库存为 0 | Product / 人 | Ads, Store | 暂停广告；Listing 改预售或下架 |
| `cash.low` | 可用资金 < 下月固定支出 × 1.5 | Finance | CEO | 砍广告预算、暂缓备货 |
| `compliance.risk` | 发现政策/认证/侵权风险 | Compliance | CEO, Product, Store | 停售评估、改 Listing |
| `phase.advance` | 某阶段验收标准全部打勾 | CEO | 全体 | 下一阶段 Agent 上线 |

## 事件生命周期

```
pending ──(订阅 Agent 认领)──▶ claimed ──(处理完)──▶ done
   │                              │
   └──(超过 SLA 未处理)──▶ escalated（升级到 CEO / 人）
```

- **SLA**：`acos.spike`、`stock.out`、`review.negative`、`compliance.risk` = 24 小时内响应；其余 = 每周经营会处理。
- Phase 0–2 阶段，所有事件由**人**查看并决定要不要响应（Copilot 模式）。

## Phase 0 当前只启用这些事件

- `market.selected` —— 选型拍板后触发，进入 Phase 1
- `compliance.risk` —— 任何时候发现禁区信号立即触发
- `phase.advance` —— 验收标准打满后触发

其余事件在对应 Agent 上线后再启用。不要提前建一堆没人消费的事件，那是噪音。
