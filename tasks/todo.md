# Agent Configuration Standard 实施计划

- [x] 读取创建要求，确定七个必需文件及通用模板边界。
- [x] 编写根入口、配置流程与 README。
- [x] 编写行为、Skill、Reviewer 和 Memory 结构模板。
- [x] 编写虚构集成示例，检查重复执行与能力降级。
- [x] 验证文件、引用、字段、决策分支及模板数据边界。
- [x] 初始化独立 Git 仓库并检查本地状态。
- [x] 确认发布范围：用户选择先完成本地仓库，GitHub 发布留待后续。

## Review

已创建七个必需文件，另附 README 和本实施记录，共九个 Markdown 文件。

- 自动静态检查通过：必需文件、17 个本地链接、代码围栏、Skill 元数据、输入及候选字段、六类纠正、五种状态和流程标记。
- 模板数据边界检查通过；人工核对未包含真实业务经验、历史 lessons、个人偏好或本机路径。虚构项目仅位于 examples。
- 文档走查覆盖 14 个验收场景，包括正常需求变化、证据不足、验证失败、去重、冲突、废弃及重复集成；这属于契约审阅，未运行真实 Reviewer。
- Git 空白检查通过，仓库已初始化在 main 分支；尚无提交或远端。
- 当前交付为通用规范，未安装到实际 Agent 宿主，因此 Skill 发现、独立上下文、Reviewer 调用和权限隔离均未做运行验证。
- 用户选择先完成本地仓库；GitHub 创建与推送留待后续。

# 1.1.0 Memory Governance 升级

- [x] 检索现有配置、审查、Memory 契约与调用关系，确定增量方案。
- [x] 新增 Lifecycle、Retrieval、Index 和 Memory Manager 规范，明确状态与职责。
- [x] 更新 CONFIG、行为入口、存储结构和审查引用；添加 VERSION 与配置版本协议。
- [x] 更新 README 和虚构示例，覆盖索引检索、失效、激活、版本升级与降级流程。
- [x] 验证链接、必需字段、状态转换、角色一致性及差异，记录实际验收结果。

实施决策：Reviewer 只产生候选判断；主代理执行验证与持久化；Memory Manager 审核验证状态并输出治理决策；任务检索只通过 Index 加载匹配的 Active 条目。保留 Discarded 失败分支，与既有规范兼容。目标配置记录与宿主配置分离，避免向宿主写入未知字段。

## 1.1.0 Review

- 新增 Lifecycle、Retrieval、Index、Memory Manager 及 VERSION；同步修改九个已有文档，保留入口、配置协议及 Experience Review 的原有职责。
- 状态规范统一到 lifecycle.md，structure.md 只保留存储职责；Manager 只返回治理决策，主代理唯一写入并按 Revision 同步 Index。
- 静态校验通过：12 个必需文件、13 个 Markdown 文件的 48 个本地链接、VERSION 与配置示例均为 1.1.0、五步 Governance 的输入/处理/判断/输出、六个状态契约、十个索引字段及审查/治理输出契约。
- 人工契约走查覆盖示例中的 26 个场景，重点检查非 Active 排除、正文/索引不一致、同步中断、缺少 Reviewer 来源、验证失败、版本升级和重复执行。
- Git 差异与空白检查通过；模板数据边界检查通过，示例未混入真实 Memory。
- 本次升级的是能力规范，未在目标宿主安装或实际运行 Reviewer、Manager、索引检索及权限隔离；示例结果仅为预期行为，不能当作运行验证。
