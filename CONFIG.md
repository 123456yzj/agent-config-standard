# Agent 配置入口规范

## 执行契约

输入是本规范仓库、目标项目位置、用户授权范围和目标 Agent 宿主。输出为下述四项报告及经授权的增量修改。

本流程不要求固定模型或宿主。所有目标路径以目标项目根目录为基准；规范仓库路径仅用于读取模板。模板中的相对链接描述源文件关系，集成时必须改写为目标项目的实际路径。

只读分析可立即执行。用户已授权实施时，完成适配方案后直接推进；用户仅要求规划时，输出方案后等待实施授权。涉及关键业务冲突或目标不明时，只暂停依赖该答案的部分。

## 配置版本协议

发布版本由根目录 [VERSION](VERSION) 给出，当前为 `1.1.0`。目标项目使用独立的 `.agent-config.json` 记录下列字段；已有等效清单时复用并记录位置。不要把这些字段加入未经确认支持它们的宿主配置。

| 字段 | 含义 | 当前要求 |
| --- | --- | --- |
| config_version | 目标项目完成安装并通过静态验收的配置协议版本 | 完整升级完成后为 1.1.0；部分失败不得提前写新版本 |
| template_version | 本次集成所使用模板的发布版本 | 读取源仓库 VERSION；本发布为 1.1.0 |
| compatible_agents | 目标项目实际评估的 Agent 宿主兼容记录数组 | 每项有 name、version、mode、verification、limitations；不预设所有宿主兼容 |

name 是宿主名，version 是实际检测版本，无法检测则为 `unknown`；mode 为 `subagents` 或 `main-agent-fallback`；verification 为 `runtime-tested` 或 `static-only`；limitations 为限制字符串数组。只有发现、Reviewer/Manager 调用和权限检查均实际验证后才可标 runtime-tested。尚未评估时数组为空，不代表兼容所有宿主。

配置清单形状（版本字段仅在完成配置后落盘）：

```json
{
  "config_version": "1.1.0",
  "template_version": "1.1.0",
  "compatible_agents": []
}
```

- 输入：源 VERSION、目标版本清单、当前配置及宿主能力。
- 处理：无清单按 legacy/unversioned 调查，不假定旧版本；旧版本按语义差异增量升级；相同版本仍检查实际配置是否漂移。记录旧值、目标值及验证结果。
- 判断：源版本与协议不一致时核对来源后再写入；遇到目标较新版本或跨主版本变更，不自动降级/迁移，先输出差异方案。只有配置及验收完成后更新清单，失败保留旧版本并报告部分变更。
- 输出：四项报告内的版本差异、迁移结果及兼容限制。升级不赋予历史经验 Active 资格；重复执行不得生成副本或无意义地改写版本记录。

## Step 1：读取规范

**输入**：本文件及以下模板：

- [行为规范](template/AGENTS.md)
- [Experience Review Skill](template/skills/experience-review/SKILL.md)
- [Reviewer 定义](template/agents/experience-reviewer.md)
- [Memory 结构](template/memory/structure.md)
- [Memory Lifecycle](template/memory/lifecycle.md)
- [Memory Retrieval](template/memory/retrieval.md)
- [Memory Index](template/memory/index.md)
- [Memory Manager](template/agents/memory-manager.md)
- [VERSION](VERSION)

**处理**：读取完整契约，提取纠正事件、错误归因、复用筛选、候选验证、Memory 生命周期及角色边界。

**判断条件**：必需文件缺失或契约互相矛盾时，报告具体位置，暂停受影响的配置，不能根据文件名猜测内容。

**输出：Template Capability Summary**。列出能力、触发条件、输入输出、依赖、主代理/Reviewer/Manager 的分工，以及 config_version、template_version 和兼容要求。

## Step 2：分析目标项目

**输入**：能力摘要、目标目录及该目录适用的现有规则。

**处理**：先确认目标根目录、Git 状态和用户未提交修改，再检查以下位置是否存在及其生效方式：

- 根目录及适用父级、子目录的 `AGENTS.md`、`CLAUDE.md`。
- `.cursor/rules`、`.agents`、`skills` 和现有 Skill/子 Agent 定义。
- 宿主相关目录与配置，例如 `.opencode`、`.claude`、配置中的 Skill 搜索路径和工具权限。
- 现有自我改进流程、纠正记录、Memory 或 lessons 的目录与引用。
- 现有 Index、生命周期治理与任务检索入口、配置版本清单、宿主兼容记录。

只读取与配置有关的必要内容；不把完整历史、真实 lessons 或凭据带入规范仓库。检索定义和引用后判断能力是否等效，不以名称相同代替语义检查。

**判断条件**：

- 已有等效能力：标记复用。
- 部分存在：标记补充，指出缺失契约。
- 完全缺失：标记新增。
- 与现有规则冲突：记录两方位置、影响及待解决问题。
- 宿主或加载机制未知：查阅其实际配置或权威文档；无法确认时标记未验证，不臆造配置字段。

**输出：Project Configuration Analysis**。列出路径、现状、适用范围、宿主支持程度、用户修改和冲突。

## Step 3：生成适配方案

**输入**：能力摘要和项目分析。

**处理**：逐项确定复用、新增、补充或暂缓，明确最终文件路径、链接、权限、Memory 位置和验证方式。

**判断条件**：

1. 已有同义规则保持单一来源，使用引用或局部补充，不叠加第二套循环。
2. 同名但职责不同的 Skill/Agent 不覆盖；复用、改名或合并需说明语义依据。
3. Skill 的名称、元数据和目录需符合目标宿主；Reviewer 的通用角色定义需转换为该宿主支持的配置，不能把通用 Markdown 当成已注册子 Agent。
4. Reviewer 使用新的独立任务上下文，仅接收 Skill 契约及五项事件材料。宿主支持时禁用文件读取、写入、shell、网络、会话检索及再次委派能力；明确规则通过主代理传入，无需 Reviewer 读取 Skill 文件。
5. 无子 Agent 或无法隔离上下文时，由主代理按同一契约审查，并在报告中明确“主代理降级执行，未实现独立上下文隔离”。不得伪称已运行独立 Reviewer。
6. Memory 目录复用现有等效结构。历史经验不自动迁移为 Active，也不因本次集成而全量审查；迁移需有相应任务授权和证据。
7. 同一目标重复执行时，只补齐语义差异，不重复新增规则、Agent、Skill 或经验副本。
8. Memory Manager 转换为宿主支持的角色配置，只接收定向治理包，禁用业务编辑与执行工具。主代理负责验证和唯一写入，Manager 返回决策。无子 Agent 时分阶段执行 Reviewer 与 Manager 契约，明确降级限制。
9. 将下述 Memory Governance 纳入方案，指定 Index、版本清单、治理规范位置及规划前检索入口。旧规则若允许全量 lessons 加载，局部替换并检查引用，不能保留冲突入口。

**输出：Configuration Plan**。用“能力 / 操作 / 目标路径 / 理由 / 验证方式”描述变更，并列出未解决问题及降级项。

## Step 4：实施修改与验收

**输入**：适配方案和实施授权。

**处理**：

1. 修改前复核目标文件没有新的冲突性变更，按方案进行最小增量编辑。
2. 保留原有规则及用户修改，不用整个 template 文件覆盖目标文件。
3. 建立可发现的 Skill、可调用的 Reviewer/Memory Manager 或明确的降级流程；修正全部目标引用。
4. 执行下述 Memory Governance，只创建必要结构和索引，不填入演示经验。模板示例不算真实事件。
5. 检查配置语法、加载路径、权限、输出字段和生命周期；宿主可用时实际验证发现与调用。
6. 使用 [集成示例](examples/integration-example.md) 中的验收场景检查分支，再做一次只读配置分析，确认没有重复项。
7. 验收完成后保存 config_version、template_version、compatible_agents；运行检查不可用时按 static-only 记录限制，不能宣称运行兼容。

**判断条件**：静态检查失败先修复；缺宿主、权限或运行环境时记录未验证项，不把静态通过当作运行通过。若宿主需重启，说明重启后验证步骤。

**输出：Change Report**。包括新增/修改/复用文件及职责、实际生效方式、验证证据、未验证项、冲突或降级、再次执行结果和使用提示词。未改动时也说明复用依据。

## Memory Governance：Step 4 内的配置流程

```text
读取 Memory 生命周期规范 → 读取 Memory Retrieval 规范
→ 初始化 Memory 目录 → 初始化 Memory Index → 配置 Memory 管理流程
```

本流程复用前述分析、授权和四项报告。规范文件与索引数据必须区分：如治理规范安装到目标 `memory/rules/`，索引数据仍在 `memory/index.md`；也可复用既有规则目录。模板链接必须按实际落点重写。

### G1. 读取 Memory 生命周期规范

- 输入：Lifecycle、Memory 结构、现有状态规则及配置版本。
- 处理：核对 Correction Event → Candidate → Validated → Active → Deprecated，保留 Discarded 失败分支和目标更严格的激活要求。
- 判断：没有 Reviewer 或验证证据的历史记录不能认定 Active；不同状态含义需明确映射和冲突，不能静默重命名。
- 输出：状态映射、激活门槛和增量规则变更。

### G2. 读取 Memory Retrieval 规范

- 输入：Retrieval、Index 规范、任务分类及现有上下文加载方式。
- 处理：指定任务分类、查询生成、Index 搜索、定向加载和 Task Planning 的接入点及上限。
- 判断：加载仅限 Active；宿主自动注入全部 lessons 或候选时，调整入口后再宣称满足隔离要求。无法调整则报告不兼容，不标记治理验收通过。
- 输出：检索方案、匹配/不匹配原因契约及禁止全量加载的执行位置。

### G3. 初始化 Memory 目录

- 输入：目标路径、现有目录、存储规范及授权方案。
- 处理：复用或补齐 events、candidates、lessons 的五种类型目录；保留原数据，建立规范引用。
- 判断：空项目只建结构；历史经验按授权范围核验来源，来源不明不激活。目录存在不等于能力已接入。
- 输出：目录映射与变更清单。

### G4. 初始化 Memory Index

- 输入：Index 字段契约、目录映射、已有索引及经核验的记录元数据。
- 处理：无数据时创建带元数据的空表；有数据时按 Id 增量建立/修复索引，校验 Location、Revision、状态和证据，保留 Created Time。
- 判断：不得覆盖索引、插入演示经验或把全部 lessons 自动列为 Active。未知来源保持不可检索并报告；重复 Id 或损坏位置先修复。
- 输出：有效 Index、排除/待修复清单及同步方式。

### G5. 配置 Memory 管理流程

- 输入：Manager 定义、Reviewer 入口、Validation 来源、Index 及生命周期方案。
- 处理：配置候选管理、验证核验、激活和废弃的调用时机；主代理执行验证并应用 PROMOTE/KEEP_CANDIDATE/DEPRECATE，按 Revision 同步正文和索引。Future Retrieval 接到任务规划前。
- 判断：不能绕过 Reviewer 或 Validation；无子 Agent 可明确降级，但语义须一致。验证失败、待验证、激活、废弃和重复请求必须得到合法处理。
- 输出：治理流程、权限限制、状态/索引验收结果和 compatible_agents 实际记录。
