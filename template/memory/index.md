# Memory Index 规范

本文件定义索引格式与操作协议；目标项目的 `memory/index.md` 是索引数据文件，初始化时只有元数据和空表头，不能把本规范正文当作数据行。等效的既有索引格式可复用，但字段和检索语义必须一致。

Index 是检索入口，不是经验正文，也不替代验证。任务 Agent 首先读取或查询 Index，禁止先扫描全部 lessons 正文。状态含义见 [lifecycle.md](lifecycle.md)。

## 字段契约

| 字段 | 定义及约束 |
| --- | --- |
| Id | 稳定且唯一，与正文一致 |
| Type | engineering / architecture / business / ui-design / user-preference |
| Keywords | 少量任务、技术、错误模式关键词，不放经验全文 |
| Applicable Scope | 适用前提与任务范围的摘要，与正文语义一致 |
| Not Applicable Scope | 排除条件，检索时优先检查 |
| Status | Candidate / Validated / Active / Deprecated / Discarded |
| Location | 相对于目标 memory 根目录的正文路径，禁止越界和外部 URL |
| Created Time | 首次创建时间，ISO 8601，后续转换不重置 |
| Validated Evidence | 验证结论摘要与证据引用；未验证明确写 pending，Active 不得 pending |
| Revision | 与正文一致的正整数，每次内容或状态修改递增，用于检测过期读写 |

索引元数据记录 `template_version`、`Updated Time`。Keywords、Created Time、Revision 也存入正文。事件保存在 events，不进入经验索引。候选与废弃行仅供治理及状态过滤；查询时先排除非 Active，返回任务上下文的结果只包含匹配的 Active 元数据。

空表初始化格式（元数据由初始化过程填入真实值，不保留占位符）：

```markdown
| Id | Type | Keywords | Applicable Scope | Not Applicable Scope | Status | Location | Created Time | Validated Evidence | Revision |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
```

## 搜索契约

- 输入：[Retrieval Query](retrieval.md)、当前 Index。
- 处理：先按 Status=Active、类型和范围过滤，再按关键词与任务意图匹配；命中排除范围的条目优先剔除。只返回限额内的候选匹配元数据，不批量读取正文。
- 判断：空索引或无匹配返回空结果；未知范围不猜测；重复 Id、非法 Location、Active 无验证证据均视为不可用并交治理修复。
- 输出：匹配 Id/Location、加载理由、不匹配原因及待修复问题。

Markdown 索引较小时可读取其元数据表；大索引使用定向搜索或按 Type 分片，根 Index 保存分片定位。无论实现方式如何，都不能用扫描全部 lessons 作为常规检索回退。

## 写入与同步

- 输入：经过 Reviewer 的候选、Manager 决策或不改变原则的元数据更新，以及预期 Revision。
- 处理：主代理是唯一写入者；先比对 Id/Revision，使用串行写入或宿主原子更新能力，正文和索引同时进入治理更新范围。每次变化都同步 Revision、Status、Location、范围与证据。
- 判断：检测到并发修改时重新读取相关记录并重新评估，不能覆盖新版本。无多文件原子操作时，先将该索引行临时移出可搜索集合，再更新/移动正文，最后写回新行；中断时该条目保持不可检索。
- 输出：一致的正文与唯一索引行、History 和更新结果。激活移动到 lessons，废弃保留位置并改状态，事件引用同步更新。

每次加载正文必须复核 Id、Status、Revision、范围和验证引用与 Index 一致；不一致时拒绝用于任务。正文即使位于 lessons，也不能仅凭目录认为其 Active。

## 缺失与修复

- 输入：缺失、过期、损坏索引或加载校验失败。
- 处理：任务检索返回不可用原因，主代理执行独立治理修复，按真实记录重建元数据；修复必要时可扫描记录，但结果不得整体注入业务任务上下文。
- 判断：只有具备 Reviewer、验证与激活历史的条目可重建为 Active；未知来源保持不可用，不能凭文件所在目录补造状态或证据。
- 输出：修复后的 Index 及变更报告，再重新发起定向检索。索引修复不能成为批量激活历史经验的方式。
