# Agent Configuration Standard

为 Agent 提供可复用的项目初始化规范。Agent 读取规范后，分析目标项目的现有配置，生成适配方案并执行增量修改。

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
| [Memory 结构](template/memory/structure.md) | 事件、候选、经验及生命周期 |
| [集成示例](examples/integration-example.md) | 虚构项目的增量集成与验收场景 |

## 工作方式

用户纠正 → 记录事件 → 错误归因 → 判断复用价值 → 候选经验 → 验证 → 有效经验。

主代理负责任务实施和 Memory 管理；Reviewer 仅分析精简材料，返回 `DISCARD` 或 `CREATE_CANDIDATE`。验证通过后可自动激活，证据不足时保持候选。普通需求变化不会自动形成经验。

模板只携带通用能力。真实事件、经验及偏好保存在目标项目，能力发现和权限配置按照实际 Agent 宿主适配。
