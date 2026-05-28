# 数据库查询接口

## SQL Proxy

云端 Cursor Agent 默认使用 SQL Proxy，不直接调用 NBA SQL 网关。

服务地址：

```text
https://8.152.222.232/api/db/query
```

使用 HTTP POST 调用：

```bash
curl -k -X POST "https://8.152.222.232/api/db/query" \
  -H "Authorization: Bearer ${DB_QUERY_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{"sql":"SELECT 1 AS ok","limit":1}'
```

鉴权：

```text
Authorization: Bearer <DB_QUERY_API_KEY>
```

Agent 必须从服务端环境变量或 Secret 读取 `DB_QUERY_API_KEY`，不要把 token 写入文档、日志或输出。

当前服务器使用 IP 自签 HTTPS 证书。调用方如果未导入证书，开发阶段可临时关闭证书校验；生产应使用正式 CA 证书。

## 查询限制

- 只允许单条 `SELECT` 或 `WITH` 查询。
- 禁止分号和多语句。
- 禁止 SQL 注释，例如 `--`、`/* */`、`#`。
- 禁止写操作和高危关键字，例如 `INSERT`、`UPDATE`、`DELETE`、`DROP`、`ALTER`、`TRUNCATE`、`CREATE`、`GRANT`、`CALL`、`EXEC`。
- 请求体 `limit` 默认 `200`，最大 `1000`。
- 代理层返回 `rows`、`row_count`、`truncated`、`query_hash`、`elapsed_ms`。

## ai_data 查询约束

查询知识运营库时，SQL 仍必须使用完整表名前缀：

```text
ai_data.<table_name>
```

例如：

```sql
SELECT status, COUNT(*) AS cnt
FROM ai_data.nba_ko_relations
WHERE is_deleted = 0
GROUP BY status
ORDER BY status
```

上游路由仍应走 `nba_data_biz`。SQL Proxy 服务端负责向上游网关携带对应数据库路由；agent 侧不再直接传 `db=nba_data_biz`。

SQL Proxy 服务端通过环境变量 `NBA_SQL_AI_DATA_ROUTE=nba_data_biz` 配置该路由。只有当 SQL 文本包含 `ai_data.` 时，Proxy 才会向上游 SQL 网关附加 `db=nba_data_biz`；普通 `nba_game_prod` 查询不应携带该路由参数。

## Legacy 直连网关

仅在本地调试或 SQL Proxy 不可用时，才使用旧网关：

```bash
curl --get "https://tech-api.nbadata.cn/dev/sql" \
  --data-urlencode "db=nba_data_biz" \
  --data-urlencode "sql=SELECT 1 AS ok"
```

旧网关同样只按只读查询使用。不要假设它支持 `UPDATE` / `INSERT` / `DELETE`。

## 必查 SQL

确认关系类型：

```sql
SELECT rt.id, rt.relation_code, rt.relation_name, rt.is_active, rt.is_blacklisted, rt.is_deleted
FROM ai_data.nba_ko_relation_types rt
WHERE rt.relation_code = '${relation_code}'
```

确认实体规则：

```sql
SELECT r.source_type, r.target_type
FROM ai_data.nba_ko_relation_types rt
JOIN ai_data.nba_ko_relation_type_rules r
  ON rt.id = r.relation_type_id
WHERE rt.relation_code = '${relation_code}'
ORDER BY r.source_type, r.target_type
```

读取现有关系实例：

```sql
SELECT id, source_entity_type, source_entity_name, source_entity_id,
       relation_type, target_entity_type, target_entity_name, target_entity_id,
       properties, description, source_url, status, asknba_status,
       ai_confidence, original_sources, created_at, updated_at
FROM ai_data.nba_ko_relations
WHERE is_deleted = 0
  AND relation_type = '${relation_code}'
ORDER BY status, source_entity_name, target_entity_name
```

读取实体标签用于规范化和实体存在性检查：

```sql
SELECT id, entity_type, entity_name, aliases, external_id
FROM ai_data.nba_ko_entity_tags
WHERE is_deleted = 0
  AND entity_type IN (${entity_types})
```

按候选实体名称精确查找实体：

```sql
SELECT id, entity_type, entity_name, aliases, external_id
FROM ai_data.nba_ko_entity_tags
WHERE is_deleted = 0
  AND entity_type = '${entity_type}'
  AND entity_name = '${entity_name}'
```

按别名查找实体（MySQL JSON 数组）：

```sql
SELECT id, entity_type, entity_name, aliases, external_id
FROM ai_data.nba_ko_entity_tags
WHERE is_deleted = 0
  AND entity_type = '${entity_type}'
  AND JSON_CONTAINS(aliases, JSON_QUOTE('${alias}'))
```

## 实体实例处理规则

- Agent 必须先检查候选三元组两端实体是否已存在于 `nba_ko_entity_tags`。
- 如实体已存在，输出对应 `entity_id`。
- 如实体不存在，候选仍可返回，但必须标记 `needs_create = true`，并说明新建原因和证据。
- 不允许静默创建实体，也不允许假设实体存在。
- 系统审核通过时可能会自动创建缺失实体，但 agent 输出必须提前提示审核人。

## 状态语义

- `draft`：草稿。
- `pending_review`：待审核，agent 产出的候选最终应进入此状态。
- `published`：已发布。
- `rejected`：已驳回。
- `superseded`：已被替换。

