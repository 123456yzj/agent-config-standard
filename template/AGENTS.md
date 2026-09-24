# 目标项目 Agent 行为规范模板

本文件是待适配的能力定义。集成者先执行源仓库 CONFIG.md，将以下规则增量合并至目标项目入口，并把引用改为实际安装路径。

## 职责

Main Agent 负责理解需求、检索实现、修改代码、执行验证、完成任务及 Memory 持久化。Experience Review 负责候选判断，遵循 [Skill 契约](skills/experience-review/SKILL.md) 和 [Reviewer 定义](agents/experience-reviewer.md)。[Memory Manager](agents/memory-manager.md) 负责生命周期决策；[Retrieval](memory/retrieval.md) 负责未来任务加载。两种子 Agent 均不修改业务代码。

## Self Improvement Loop

用户纠正不等于经验。正常需求演进、一次性选择和临时偏好不自动形成长期规则。

```text
Correction Event → Experience Review → Candidate Memory → Validation
                 → Memory Manager → Active Memory → Future Retrieval
```

Validation 是证据收集与验证过程；Validated 是 Manager 核验通过后记录的状态。完整状态契约见 [Lifecycle](memory/lifecycle.md)。

### 1. 创建 Correction Event

- 输入：任务背景、AI 原始方案及采用原因、用户纠正、最终采用方案、相关差异与测试结果。
- 处理：主代理在纠正发生时记录事实摘要，继续完成任务，再补充最终结果。原始理由无记录则标为未知，不事后编造。
- 判断：结果未确定时仅保存待完成事件；五项审查材料齐备后才送审。没有代码差异时明确“不适用”及原因；未执行测试标为未验证。
- 输出：按 [Memory 结构](memory/structure.md) 保存的事件，以及五项输入组成的精简审查包。

### 2. 错误归因

- 输入：审查包及 Skill 契约。
- 处理：主代理创建独立 Reviewer 任务；Reviewer 区分事实和推断，分析原始决策为何与要求不符，按六种纠正类型分类。
- 判断：缺少关键证据时不能凭用户最终选择反推原始错误；正常变更不得伪装成错误。
- 输出：Root Cause Analysis；无法支持归因时输出 DISCARD 及原因。

### 3. 判断经验复用价值

- 输入：事件、错误归因和候选原则。
- 处理：分别判断是否脱离本任务仍成立、是否描述稳定原则、是否可能再次发生、是否具有明确适用范围，每项写出依据。
- 判断：四项全部通过才继续；任意失败或无法判断时 DISCARD。
- 输出：Reuse Assessment，以及 DISCARD 或 CREATE_CANDIDATE 结论。

### 4. 生成 Candidate Memory

- 输入：CREATE_CANDIDATE 的结构化结果。
- 处理：主代理核对 Reviewer 输出，先查询 Index 并向 Memory Manager 提供重复/冲突材料，按其治理结果保存候选或追加来源，同步 Index。实质冲突暂缓激活并调查。
- 判断：缺字段或来源不可追溯时退回补充，不写成有效经验；候选不得直接影响未来任务决策。
- 输出：包含 Skill 要求的全部字段、状态和验证计划的 Candidate，或附于既有条目的新来源记录。

### 5. Validation

- 输入：候选、来源、相关验证证据及既有规则。
- 处理：主代理按 Lifecycle 的验证门槛检验根因、原则、适用边界及反例，提供方法、实际结果和出处。
- 判断：检查通过只是供 Manager 核验的证据；证据不足或失败如实记录，不自行宣布 Active。
- 输出：Validation 材料及待验证项，交 Memory Manager。

### 6. Memory Manager

- 输入：候选、Reviewer 结果、Validation、相关 Index 与当前规则。
- 处理：Manager 核查合法转换，输出 PROMOTE、KEEP_CANDIDATE 或 DEPRECATE；主代理按预期 Revision 落实正文、历史和索引变更。
- 判断：完整验证后才 Candidate → Validated；激活条件满足再到 Active。验证失败按既有语义转 Discarded；失效的 Active 转 Deprecated。同步失败不加载。
- 输出：可追溯的状态变更和一致的 Index。独立 Agent 不可用时主代理分阶段执行相同契约，并明确降级限制。

## Future Retrieval：规划前使用经验

- 输入：当前任务、项目上下文和任务分类。
- 处理：User Task → Task Classification → Generate Retrieval Query → Search Memory Index → Load Relevant Memory → Task Planning。首先查询 [Index](memory/index.md)，只读取限额内的匹配条目并核验正文状态。
- 判断：只有 Active 且适用、证据和版本一致的经验可用。Candidate 不允许自动进入 Agent 上下文；Validated、Deprecated、Discarded 也不参与任务执行。禁止全量加载 lessons 或全部历史。
- 输出：匹配 Memory、逐条加载原因、不匹配原因和索引问题。无匹配时按当前规则继续；索引故障交治理修复，不回退为全文加载。
