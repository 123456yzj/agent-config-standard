# Agent Configuration Standard

为 Agent 提供可复用的项目初始化规范。Agent 读取规范后，分析目标项目的现有配置，生成适配方案并执行增量修改。

当前版本：[1.1.0](VERSION)。配置版本、模板版本与宿主兼容记录按 CONFIG 管理，支持既有配置增量升级。

## 使用

向 Agent 提供本仓库地址或本地路径，以及目标项目位置：

```text
请读取 <本仓库地址或本地路径> 的 AGENTS.md 和 CONFIG.md，
按照规范分析 <目标项目路径> 的现有 Agent 配置，生成适配方案并实施。
保留已有规则，复用等效能力，禁止直接复制模板覆盖文件。
完成后报告变更、验证结果、宿主限制和使用方法。
```

仅需方案时，将“生成适配方案并实施”替换为“只输出适配方案，不修改文件”。远程入口不能访问时，先取得仓库内容，再读取完整文件；不要只根据 README 推测配置步骤。

## 文件导航

| 文件 | 职责 |
| --- | --- |
| [AGENTS.md](AGENTS.md) | Agent 入口与规范维护约束 |
| [CONFIG.md](CONFIG.md) | 分析、适配、增量实施与验收 |
| [template/AGENTS.md](template/AGENTS.md) | 目标项目的行为规范模板 |
| [Experience Review Skill](template/skills/experience-review/SKILL.md) | 纠正事件审查及输入输出契约 |
| [Reviewer](template/agents/experience-reviewer.md) | 独立审查角色与上下文边界 |
| [Memory 结构](template/memory/structure.md) | 事件、候选、经验的存储及索引关系 |
| [Memory Lifecycle](template/memory/lifecycle.md) | 状态、转换条件、允许与禁止操作 |
| [Memory Retrieval](template/memory/retrieval.md) | 按任务分类和 Index 加载相关 Active Memory |
| [Memory Index](template/memory/index.md) | 索引字段、同步、修复与版本校验 |
| [Memory Manager](template/agents/memory-manager.md) | 候选管理、验证核验、激活与废弃决策 |
| [VERSION](VERSION) | 规范与模板发布版本 |
| [集成示例](examples/integration-example.md) | 虚构项目的增量集成与验收场景 |

## 工作方式

Correction Event → Experience Review → Candidate Memory → Validation → Memory Manager → Active Memory → Future Retrieval。

主代理实施任务、验证并持久化；Reviewer 返回 `DISCARD` 或 `CREATE_CANDIDATE`；Memory Manager 返回 `PROMOTE`、`KEEP_CANDIDATE` 或 `DEPRECATE`，决定生命周期。检索先查 Index，再核验并加载匹配的 Active 正文。候选不会自动进入任务上下文，Deprecated 停止检索；普通需求变化不会自动形成经验。

模板只携带通用能力。真实事件、经验及偏好保存在目标项目，能力发现和权限配置按照实际 Agent 宿主适配。
