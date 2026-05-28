# 专属获取流程验证记录

验证时间：2026-05-28

本文件记录一期 10 个关系类型的获取流程可行性。结论用于指导 Cursor Agent 优先选择稳定来源。

## 验证结论

| relation_code | 结论 | 已验证来源 | 注意事项 |
|---|---|---|---|
| `team_home_arena` | 可稳定获取 | NBA.com teams directory、NBA.com arenas by team、NBA.com team info 页面 | 先取当前官方球队列表，不把 30 作为固定数量；场馆列表和队伍页面用于交叉验证。 |
| `arena_location` | 可稳定获取 | 场馆官网地址页，如 Crypto.com Arena A-Z Guide | 以官方地址城市为准，不用都会区泛称。 |
| `retired_number` | 可稳定获取 | NBA.com team-specific retired numbers pages | 部分球队页面可能混有 honored numbers，需要排除非正式退役号码。 |
| `win_honor` | 可稳定获取 | NBA.com All-Time Awards、NBA Finals MVP、Hall of Fame official site | 只抽重大荣誉；周/月最佳不进入一期。 |
| `referee_officiate` | 可稳定获取 | Official NBA Officials Guide PDF | 使用官方 PDF 作为主来源；referee assignments 不用于创建 staff-list 关系。 |
| `family_legacy` | 可获取，但需谨慎 | Basketball Reference player page `Relatives` 字段、NBA.com profiles | 官方来源不总是完整；低置信度亲属关系不要产出候选。 |
| `coach_team` | 可获取，但页面不统一 | NBA.com team info、球队 coaching staff/staff directory 页面 | Head Coach 最稳定；助教列表页面结构差异较大。 |
| `team_predecessor` | 可获取 | NBA.com team history、Basketball Reference Franchise Index | Wikipedia 只作辅助，不作为唯一来源。 |
| `team_championship_season` | 可稳定获取 | NBA.com history champions page、season recaps | 只抽 NBA 总冠军赛季，不抽分区/赛区冠军。 |
| `team_historical_location` | 可获取 | NBA.com team history、Basketball Reference Franchise Index | 城市沿革需和 `team_predecessor` 区分，避免重复表达。 |

## 样例级 Smoke Test

以下样例用于验证专属流程能否真的产出目标三元组。Agent 后续可按同样方式扩展更多样例。

| relation_code | 样例三元组 | 验证来源 | 验证结论 |
|---|---|---|---|
| `team_home_arena` | `Los Angeles Lakers -> Crypto.com Arena` | NBA.com `A Complete List of all 30 NBA Arenas By Team` | 页面直接列出 `Los Angeles Lakers: Crypto.com Arena (Los Angeles, CA)`，并给出首个 NBA 赛季。 |
| `arena_location` | `Crypto.com Arena -> Los Angeles` | Crypto.com Arena A-Z Guide | 场馆官网直接给出 `1111 South Figueroa Street, Los Angeles, CA 90015`。 |
| `retired_number` | `Kobe Bryant -> Los Angeles Lakers` | NBA.com `LA Lakers retired numbers` | 页面直接列出 Kobe Bryant 的 No. 8 和 No. 24 被 Lakers 退役。 |
| `win_honor` | `Jaylen Brown -> NBA Finals MVP` | NBA.com `NBA Finals MVP Award Winners` | 页面直接列出 `2023-24: Jaylen Brown, Boston Celtics`。 |
| `referee_officiate` | `Ray Acosta -> NBA` | Official NBA Officials Guide PDF | 官方 PDF 的 `2025-26 NBA Officiating Staff` 列出 `54 Ray Acosta`。 |
| `family_legacy` | `Joe Bryant -> Kobe Bryant` | Basketball Reference Joe Bryant page | `Relatives` 字段直接写明 `Son Kobe Bryant`。 |
| `coach_team` | `JJ Redick -> Los Angeles Lakers` | NBA.com Lakers Coaching Staff | 页面直接写明 JJ Redick 是 Los Angeles Lakers Head Coach，并给出任职背景。 |
| `team_predecessor` | `Los Angeles Lakers -> Minneapolis Lakers` | NBA.com Lakers Season Capsule、Basketball Reference Franchise Index | 官方历史页说明 Minneapolis Lakers 在 1960-61 前迁至 Los Angeles；BR `Team Names` 包含 Los Angeles Lakers / Minneapolis Lakers。 |
| `team_championship_season` | `Boston Celtics -> 2023-24 NBA Season` | NBA.com `List of NBA champions` | 页面直接写明 `2023-24 — Boston Celtics def. Dallas Mavericks, 4-1`。 |
| `team_historical_location` | `Los Angeles Lakers -> Minneapolis` | NBA.com Lakers Season Capsule、Basketball Reference Franchise Index | 官方历史页说明球队从 Minneapolis 迁至 Los Angeles；BR 历史赛季行显示 Minneapolis Lakers。 |

## 总体判断

- 10 个专属流程都可执行。
- `team_home_arena`、`arena_location`、`retired_number`、`win_honor`、`referee_officiate`、`team_championship_season` 来源最稳定。
- `coach_team`、`family_legacy`、`team_predecessor`、`team_historical_location` 需要更多交叉验证，但仍适合一期低维护关系。

## Agent 执行建议

1. 优先使用本文件标记的已验证来源。
2. 若专属来源不可访问，再执行关系文档中的 `Fallback Workflow`。
3. 任何只由 Wikipedia 或非官方二手来源支持的候选，应降低置信度并进入人工审核提示。
4. 对页面结构差异大的关系类型，优先输出少量高置信候选，不要强行覆盖全量。
5. 如果官方网页整页抓取超时，可以使用搜索结果片段或站内页面摘要定位事实，但最终证据仍需记录到具体 URL。

