# coach_team 执教

## Definition

表示教练执教 NBA 球队。

## Entity Rule

```text
Coach -> Team
```

## Scope

- 一期优先主教练或明确官方教练组成员。
- 对历史执教关系必须记录任期或赛季范围。
- 现任教练组更新频率较高，抽取时要优先核验当前官方页面。

## Preferred Sources

- NBA.com team roster / coaching staff pages
- 球队官网 coaching staff 页面
- Basketball Reference coach pages
- NBA 官方新闻稿

## Acquisition Workflow

1. Start from the current NBA teams directory or database active `Team` entities; do not assume the league always has exactly 30 teams.
2. For each team, visit its official roster, coaching staff, or basketball operations page.
3. Extract Head Coach first; extract assistant coaches only when clearly listed by official source.
4. For historical coach-team relations, use Basketball Reference coach pages or official team history pages.
5. Record `role`, `start_season`, `end_season`, and `is_current` when available.
6. Compare with existing database rows to identify new coaches, ended tenures, or source updates.
7. Treat current-season changes as sensitive and require official confirmation.

## Fallback Workflow

If team pages are inconsistent:

1. Use NBA official news announcements for hiring or departure events.
2. Use Basketball Reference for historical tenure ranges.
3. Do not rely on rumor pages, interviews, or “expected to hire” reports.

## Suggested Properties

- `role`
- `start_season`
- `end_season`
- `is_current`

## Exclusions

- 不抽取临时传闻、候选人、面试对象。
- 不把球员效力关系误建为执教关系。

