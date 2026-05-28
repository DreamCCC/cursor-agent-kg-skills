# Agent 运行约定

## 角色定位

Agent 是“自动研究员 + 自动质检员”，不是最终发布者。它负责发现、核验、整理候选三元组，并把结果交给运营审核。

## 输入参数

每次运行至少传入：

```json
{
  "relation_code": "team_home_arena",
  "max_candidates": 50,
  "mode": "discover_and_verify"
}
```

可选模式：

- `discover_and_verify`：发现新增并检查现有数据。
- `verify_existing_only`：只核验现有三元组。
- `discover_only`：只找新增，仍需去重。

## 默认行为

- 通过 SQL Proxy 只读查询数据库。
- 优先执行关系类型文档中的 `Acquisition Workflow`。
- 没有专属获取流程时，使用 Web Search / 浏览器搜索公开信息源。
- 对候选三元组两端实体做存在性检查。
- 输出结构化 JSON。
- 不直接发布数据。

## 数据库访问方式

云端 agent 必须调用 `reference/database-api.md` 中的 SQL Proxy：

```text
POST https://8.152.222.232/api/db/query
```

不要在云端 agent 内直接调用 NBA SQL 网关。SQL Proxy 负责鉴权、安全校验和上游数据库路由。

## 获取工作流优先级

每个关系类型可在 `reference/relation-types/{relation_code}.md` 中定义：

- `Acquisition Workflow`：专属获取流程，优先执行。
- `Fallback Workflow`：专属流程失败后的备用方案。

执行顺序：

1. 读取关系类型文档。
2. 如果存在 `Acquisition Workflow`，先按该流程获取证据。
3. 如专属流程失败、证据不足或需要交叉验证，再执行 `Fallback Workflow`。
4. 如果两者都不存在，使用通用官方源 web search。
5. 无论使用哪种获取方式，后续都必须统一进入去重、diff、自检和 JSON 输出。

## 数据写入边界

如后续接入写入 API，默认只能写入 `pending_review` 候选，不允许直接写 `published` 或 AskNBA `live` 数据。

## 实体实例边界

现有审核流程可能在审核通过时自动创建缺失实体，但 agent 不应把这一步隐藏起来。每条候选都必须说明：

- source entity 是否已存在。
- target entity 是否已存在。
- 已存在时匹配到的 `entity_id`。
- 不存在时需要新建的实体类型、实体名、别名建议和证据来源。


