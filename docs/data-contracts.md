# 数据契约示例

## 为什么需要标准化契约

不同工具对“账户”“持仓”“事件”和“结论”的字段定义并不一致。如果把原始返回直接交给模型，模型会承担字段猜测、单位换算和缺失值解释，容易产生不一致结论。

AUREVAN 在工具适配层统一为以下语义：

| 对象 | 必要字段 | 关键约束 |
| --- | --- | --- |
| 工具快照 | `tool_id`、`status`、`captured_at` | 时间缺失时不能视为有效快照 |
| 账户 | `source`、`account_key`、`currency` | 对外发送时对标识脱敏 |
| 持仓 | `symbol`、`quantity`、`cost_basis`、`valuation` | 缺失数量不能解释为 0 |
| 事件 | `event_key`、`severity`、`occurred_at` | 同一事件更新而不是重复创建 |
| 结论 | `direction`、`summary`、`evidence` | 必须保留来源和证据时间 |

## 时效规则

```text
fresh      数据时间完整，且在工具定义的有效窗口内
stale      数据曾经有效，但已经超出有效窗口
unknown    缺少时间或关键字段，无法确认状态
offline    工具明确返回不可用
```

只有 `fresh` 数据可以作为实时持仓和报告结论依据。`stale`、`unknown` 和 `offline` 必须显式降级，不能被转换成“零持仓”“无风险”或“市场没有变化”。

