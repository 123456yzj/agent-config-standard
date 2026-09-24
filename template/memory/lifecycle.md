# Memory Lifecycle

本文件是状态与转换条件的唯一规范。存储格式见 [structure.md](structure.md)，治理决策见 [Memory Manager](../agents/memory-manager.md)，任务加载见 [retrieval.md](retrieval.md)。

```text
Correction Event → Candidate → Validated → Active → Deprecated
       └─ DISCARD       └─ 验证失败 → Discarded
```

Correction Event 是事件阶段，事件记录的 Status 为 `Correction Event`；它不属于经验索引。Memory 的 Status 为 `Candidate`、`Validated`、`Active`、`Deprecated` 或 `Discarded`。保留既有 Discarded 终止分支。Reviewer 的 DISCARD 是审查结论，不是 Active 经验的废弃操作。

**仅 Active Memory 可以参与任务执行。Candidate 不允许自动进入 Agent 任务上下文；Validated 也不可提前使用。**治理时按 Id 检查候选属于审查数据访问，不得把其原则注入业务任务规划。任务检索不把非 Active 条目的正文带入上下文。

## 状态契约

| 状态 | 状态定义 | 进入条件 | 转换条件 | 允许操作 | 禁止操作 |
| --- | --- | --- | --- | --- | --- |
| Correction Event | 对用户纠正的事实记录，尚无经验资格 | 主代理记录任务、原决策及原因、纠正内容 | 最终结果与证据材料齐备后送 Reviewer；CREATE_CANDIDATE 才可创建 Candidate；DISCARD 只记录原因 | 补充结果、证据、审查结论及引用 | 直接激活，推测原始动机，作为未来任务经验 |
| Candidate | 已通过 Reviewer 候选判断，尚未通过验证 | 有可追溯 CREATE_CANDIDATE、完整字段和四项 PASS，去重后保存 | 验证门槛全通过且 Manager 批准后到 Validated；明确反证到 Discarded；证据不足保持原态 | 补充证据、安排验证、去重、审查范围 | 自动加载进任务上下文，跳过验证到 Active，绕过 Reviewer 改写原则 |
| Validated | 原则与范围已验证，但尚未激活 | Manager 核验验证证据后以 PROMOTE 批准 Candidate → Validated | 激活条件全部满足后到 Active；证据失效或出现未解决冲突退回 Candidate；明确反证到 Discarded | 最终冲突检查、完成宿主要求的确认、准备索引 | 参与任务规划，以高置信度代替激活条件 |
| Active | 已验证并完成激活和索引同步的有效经验 | Manager 以 PROMOTE 批准 Validated → Active，记录及索引一致 | 反证、过时、被替代、不适用或有证据的未解决实质冲突，由 DEPRECATE 转 Deprecated | 通过 Index 按范围加载、追加不改变原则的来源 | 绕过 Index 全量加载，原地扩展原则或范围，覆盖当前更高优先级要求 |
| Deprecated | 曾激活但已停止生效，保留追溯 | Manager 核实废弃依据并输出 DEPRECATE | 终止态；恢复必须创建关联候选、重新 Reviewer 审查和验证 | 查看历史、补充原因及替代 Id、修复索引 | 参与任务检索或直接恢复 Active |
| Discarded | 验证失败或被证据否定的未激活候选 | Candidate/Validated 经 Manager 输出 DEPRECATE，明确 Next Status 为 Discarded | 终止态；新证据可触发新的关联审查 | 保留失败原因、来源及历史 | 作为 lessons 生效，直接恢复或自动激活 |

## 验证门槛

- 输入：Reviewer 结果、候选、来源、主代理提供的 Validation、既有规则及冲突检查。
- 处理：Memory Manager 核查六项：来源支持根因；四项复用判断成立；原则和证据直接相关；适用边界及至少一个不适用场景已检查；验证结果可追溯；不存在未解决的实质冲突。
- 判断：六项全部通过才批准 Validated；证据缺失保持 Candidate；明确反证关闭为 Discarded。Manager 不运行测试，主代理提供真实结果，Confidence 不能替代证据。
- 输出：PROMOTE、KEEP_CANDIDATE 或 DEPRECATE，附状态转换、证据及理由。

| 经验类型 | 可支持验证的证据 |
| --- | --- |
| engineering / architecture | 根因复现及回归、约束规范或独立案例，能支持声明的原则与范围 |
| business | 权威业务契约或明确业务确认，支持范围内稳定规则 |
| ui-design | 与任务相关的操作、可访问性或实际效果验证 |
| user-preference | 明确长期表述或多个独立事件的一致证据，并排除未解决的相反表述 |

lint、构建、单次修复成功不能自动证明抽象原则；不适用测试时提供有出处的规则审查或行为证据。缺证据不是验证通过。

## 激活与持久化

- 输入：Validated 记录、Manager 的 PROMOTE、当前索引与项目激活规则。
- 处理：主代理复核决策对应的 Id 和 Revision 未变、验证仍有效、当前无新冲突，完成必要确认，按 [Index 写入协议](index.md) 更新正文、位置、历史和索引。
- 判断：各项满足后自动激活；目标已有更严格确认要求时停留 Validated。Candidate → Validated → Active 可在一次治理中连续执行，但必须有两条明确转换记录，不能合并成跳过 Validated 的直接激活。
- 输出：正文及索引一致的 Active 记录。更新中断或状态不一致时不得加载，先修复一致性。

## 修改、废弃与恢复

- 输入：变更证据、现有记录及治理请求。
- 处理：不改变原则/范围的来源可追加并更新 Revision；改变原则或适用范围必须新建关联候选，重新送 Reviewer。Manager 核查失效原因与替代关系。
- 判断：Active 被确认失效则 Deprecated；有证据的未解决实质冲突由 Manager 标记 Deprecated 并注明适用性待调查，不声称已证明原则错误。单个任务前提不匹配仅由 Retrieval 排除，不自动废弃。Deprecated/Discarded 不直接复活。
- 输出：状态历史、索引更新和替代 Id；新原则必须走完整生命周期。
