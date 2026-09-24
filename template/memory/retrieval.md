# Memory Retrieval

本规范负责未来任务的按需加载；候选审查与状态维护分别由 Experience Review 和 Memory Manager 负责。

```text
User Task → Task Classification → Generate Retrieval Query
          → Search Memory Index → Load Relevant Memory → Task Planning
```

## 输入与输出

输入：当前任务、项目上下文、任务分类。调用方已有分类时先校验；尚无分类时由第一步生成。项目上下文限于相关模块、技术约束、业务范围及当前明确规则，不包含完整历史会话。

输出固定包含：

```text
Task Classification: <主要类型、次要类型及理由>
Retrieval Query: <types、keywords、scope、exclusions、status=Active、limit>
Matched Memory: <加载成功的 Id、Revision、Location，或空列表>
Load Reasons: <每条匹配为何影响当前决策>
Not Matched Reasons: <评估后排除的 Id/原因，或无匹配的原因>
Issues: <索引缺失、证据/范围不一致等，或无>
```

不匹配原因只涵盖本次查询实际检查到的元数据，不为解释排除而遍历全部历史。不能从已排除的 Candidate 正文摘取“可能有用的建议”。

## 1. Task Classification

- 输入：当前任务、项目上下文及可选已有分类。
- 处理：识别 engineering、architecture、business、ui-design、user-preference 中与任务相关的主要/次要类型，提取模块、约束和决策点。
- 判断：分类可多选，但需明确理由；无法确定的业务前提标为未知，不能默认为适用。
- 输出：任务分类与范围。

## 2. Generate Retrieval Query

- 输入：分类、当前任务中的关键术语、项目约束。
- 处理：生成 types、keywords、scope、exclusions、status=Active、limit。默认最多加载 5 条；项目可按上下文预算设更小上限，扩大必须说明具体需要。
- 判断：关键词用于召回，范围用于判断适用；不允许以关键词相同代替范围匹配，不使用“所有经验”查询。
- 输出：可解释的 Retrieval Query，不要求特定搜索引擎。

## 3. Search Memory Index

- 输入：Query 与项目 [Index](index.md)。
- 处理：首先读取或定向搜索 Index 元数据，过滤非 Active、排除范围及与任务无关的类型；按适用前提吻合程度、决策相关性和关键词关联排序，Id 用于同等相关项的稳定排序。
- 判断：Candidate、Validated、Deprecated、Discarded 不参与召回；无结果则返回空集合。Index 缺失或损坏时报告治理修复需求，不降级为加载全部 lessons。
- 输出：待加载的限额内 Id、Location 和逐条理由，以及元数据排除原因。

## 4. Load Relevant Memory

- 输入：Index 匹配结果及对应正文位置。
- 处理：仅加载选中的记录，核对 Id、Status=Active、Revision、适用/排除范围、验证证据及当前规则；提取对决策必要的原则和边界。
- 判断：正文与索引不一致、证据缺失、当前约束冲突或前提未知时停止使用该条并记录原因。必要时从本次 Index 查询的剩余匹配项补足限额，不能直接扫描正文寻找替代。
- 输出：Matched Memory、Load Reasons、Not Matched Reasons、Issues。空结果也是正常结果。

## 5. Task Planning

- 输入：任务、项目规则及已核验的 Active Memory。
- 处理：引用相关原则辅助规划，保留来源 Id；范围不适用或任务改变时重新检索。执行关键决策前若 Memory 已发生变更，重新核对状态与 Revision。
- 判断：当前明确要求及更高优先级规则优先；Memory 不能覆盖它们。不得每次加载全部 lessons 或把全部历史经验加入上下文；曾加载但已失效的内容不得继续指导任务。
- 输出：任务计划及必要的 Memory 使用依据。无匹配时按项目规则和实际代码继续工作，不编造经验。
