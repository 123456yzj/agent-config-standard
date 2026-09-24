# Memory 结构与生命周期

这是目标项目的结构规范。集成时复用等效目录；本模板不携带事件或经验数据。所有写入由主代理完成。

```text
memory/
├── events/
├── candidates/
└── lessons/
    ├── engineering/
    ├── architecture/
    ├── business/
    ├── ui-design/
    └── user-preference/
```

## 记录契约

建议每条记录使用一个 Markdown 文件，字段采用明确标题或元数据；目标项目已有可靠格式时可保留，语义必须等效。事件和候选 ID 必须唯一且稳定，不含个人信息；冲突时使用序号区分。

| 位置 | 必须保存的内容 | 使用限制 |
| --- | --- | --- |
| events | Event Id、五项 Skill 输入、完成状态、审查结论与原因、候选引用 | 仅供追溯，不作为决策原则 |
| candidates | Skill 的 12 项候选字段、Correction Type、Reuse Assessment、Status、Validation、History | 验证前不得作为有效经验 |
| lessons/对应类型 | 原候选全部字段、验证证据、激活时间及历史 | 只允许符合范围的 Active 条目参与决策 |

12 项候选字段为 Id、Type、Background、Agent Mistake、User Correction、Root Cause、Extracted Principle、Applicable Scope、Not Applicable Scope、Confidence、Source。

Validation 记录待检验假设、方法、预期条件、实际结果、证据定位、执行时间和验证者。History 逐项记录前后状态、时间、执行者及原因，不编造执行者身份或时间。只保存与纠正必要相关的脱敏摘要和定位，避免复制整段会话及敏感原始数据。

## 保存与去重

- 输入：完整事件和 Reviewer 输出。
- 处理：保存事件；对候选按原则含义、前提和适用范围检索现有候选及经验，而非仅按标题匹配。
- 判断：同义且范围一致时追加来源，不复制条目；新证据不自动扩展既有原则的范围。范围或原则发生变化时建立新候选并引用旧条目。与当前规则冲突时先调查，未解决前不激活。
- 输出：唯一候选或既有条目的新证据引用；DISCARD 仅保存事件结论。

## 生命周期

```text
Candidate → Validated → Active Memory → Deprecated
    ├─ 验证失败 → Discarded
    └─ 证据不足 → 保持 Candidate
```

持久化 Status 使用 `Candidate`、`Validated`、`Active`、`Deprecated`、`Discarded`。审查输出 DISCARD 与记录状态 Discarded 分属不同契约。

### Candidate → Validated

- 输入：候选、来源、验证证据、相关现有规则。
- 处理：核对事实支持根因、四项复用判断、至少一个不适用场景、原则与证据的直接关系，以及重复和冲突。
- 判断：以上全部通过才 Validated；有明确反证则 Discarded；尚未验证、证据不足或冲突待解时保持 Candidate。Confidence 不可代替验证。
- 输出：Validation 和 History，必要时更新事件中的引用与结论。

验证证据必须与原则匹配：

| 类型 | 证据示例及通过条件 |
| --- | --- |
| engineering / architecture | 针对根因的复现与回归结果、约束规范或独立案例，能够支持原则而不仅是本次改动成功 |
| business | 权威业务契约或明确业务确认，支持规则在声明范围内稳定成立 |
| ui-design | 与用户任务相关的操作、可访问性或实际效果验证，不能以单次审美选择代替 |
| user-preference | 用户明确的长期偏好声明，或多个独立事件的一致证据，且无未解决的相反表述 |

lint、构建或一次测试通过只能证明相应检查结果，不能单独证明所有抽象原则。测试不适用时可用有出处的规则审查或行为证据；不得虚报测试。

### Validated → Active

- 输入：已验证候选及完整 Validation。
- 处理：主代理复核当前规则没有新增冲突，记录激活，然后将记录移至 lessons 对应类型目录，保留 Id 和完整 History，并更新事件引用。
- 判断：验证通过且没有未解决冲突时自动激活；目标项目已有更严格确认规则时遵循该规则并停留 Validated。
- 输出：Status 为 Active 的唯一记录。不得同时保留一个仍标为 Candidate 的有效副本。

### 失败与废弃

- 输入：候选验证失败，或 Active/Validated 条目出现反证、过时约束或被替代。
- 处理：记录证据与原因；失败候选留在 candidates 并设 Discarded，已激活经验留在原位置并设 Deprecated，写明替代 Id（若有）。
- 判断：废弃条目不得继续参与决策；需要恢复时用新增证据建立关联候选并重新完整验证，不直接改回 Active。
- 输出：状态历史、来源与替代关系，保留追溯能力。

## 按需检索

- 输入：当前任务目标和范围。
- 处理：先按类型、Status 和 Applicable Scope 筛选，再读取匹配经验。
- 判断：仅采用 Active 且未命中 Not Applicable Scope 的条目；与当前明确要求或更高优先级规则冲突时以当前规则为准并调查。
- 输出：少量与决策相关的原则。不得全量把事件、候选、废弃条目注入会话。
