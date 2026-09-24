# Memory 存储结构

这是目标项目的存储规范。集成时复用等效目录；本模板不携带事件或经验数据。所有写入由主代理完成，状态转换需遵循 Memory Manager 决策。状态的唯一规范见 [lifecycle.md](lifecycle.md)，索引字段与同步见 [index.md](index.md)，任务加载见 [retrieval.md](retrieval.md)。

```text
memory/
├── index.md
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
| index.md | Index 规定的元数据和条目摘要 | 所有任务检索的首个入口，不存经验全文 |
| events | Event Id、五项 Skill 输入、完成状态、审查结论与原因、候选引用 | 仅供追溯，不作为决策原则 |
| candidates | Skill 的 12 项候选字段、Correction Type、Reuse Assessment、Status、Validation、History | 验证前不得作为有效经验 |
| lessons/对应类型 | 原候选全部字段、验证证据、激活时间及历史 | 只允许符合范围的 Active 条目参与决策 |

12 项候选字段为 Id、Type、Background、Agent Mistake、User Correction、Root Cause、Extracted Principle、Applicable Scope、Not Applicable Scope、Confidence、Source。

记录另存 Keywords、Created Time、Revision，与 Index 一致；Revision 从 1 开始，内容或状态修改时递增。Validation 记录待检验假设、方法、预期条件、实际结果、证据定位、执行时间和验证者。History 逐项记录前后状态、时间、执行者、Manager 决策及原因，不编造执行者身份或时间。只保存与纠正必要相关的脱敏摘要和定位，避免复制整段会话及敏感原始数据。

## 保存与去重

- 输入：完整事件和 Reviewer 输出。
- 处理：保存事件；主代理先查询 Index，向 Manager 提供相关候选/经验及 Reviewer 结果，按原则含义、前提和适用范围去重，而非仅按标题匹配。
- 判断：同义且范围一致时追加来源，不复制条目；新证据不自动扩展既有原则的范围。范围或原则发生变化时建立新候选并引用旧条目。与当前规则冲突时先调查，未解决前不激活。
- 输出：唯一候选或既有条目的新证据引用；DISCARD 仅保存事件结论。

## 存储位置与状态

- Correction Event 保存在 events；候选及其 Validated、Discarded 状态保存在 candidates。
- Manager 批准激活后移到 lessons 对应类型，保留 Id、创建时间和完整历史，递增 Revision 并同步 Index、事件引用，不保留重复的候选副本。
- Deprecated 保留在原 lessons 位置，但索引状态必须同步为 Deprecated；位置不代表有效性。
- 状态变更及同步中断的恢复按 Lifecycle 与 Index 执行，不在本文件另设激活条件。

## 检索交接

- 输入：当前任务和项目上下文。
- 处理：主代理执行 Retrieval，先查 Index，再读取匹配正文。
- 判断：只有正文与 Index 一致的 Active 条目可参与任务。治理访问候选时也不得把其作为任务原则。
- 输出：匹配经验、加载原因、不匹配原因及待修复问题。
