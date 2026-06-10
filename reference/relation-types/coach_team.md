# coach_team 执教

## Definition

表示教练执教 NBA 球队。

## Entity Rule

```text
Coach -> Team
```

## Discovery Coverage

- `discover_missing` can use `source_exhaustive` for current head coaches only when every current official team coaching page is checked.
- Current full coaching staffs are less stable and page structures vary; mark coverage as `partial` unless the source set is explicitly bounded and fully checked.
- Historical coaching relationships should default to `partial` unless the task provides a complete, bounded historical source.

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

## Estimate Workflow

1. Determine whether the requested scope is current head coaches, current full coaching staffs, or historical coaching relationships.
2. For current head coaches, build the active team set from NBA.com teams directory or database active `Team` entities and estimate one head coach per team.
3. For current full staffs, count only coaches clearly listed on official team coaching staff pages; mark coverage as partial unless all team staff pages are fully checked.
4. For historical coaching relationships, use Basketball Reference coach pages or team history pages and state the bounded source set.
5. Query existing `published` and `pending_review` `coach_team` rows and count unique `Coach -> Team` triples within the same scope.
6. Cross-check current-season changes with official team announcements or NBA.com news.

## Verify Workflow

1. Verify only the input `relation_ids`.
2. For current rows, confirm the coach and role from official team pages or NBA.com team info.
3. For historical rows, verify tenure using Basketball Reference or official team history pages.
4. If a valid row lacks `role`, `start_season`, `end_season`, `is_current`, or stronger evidence, return `update_candidates`.
5. If a row is based on rumors, candidates, interviews, or player-team service rather than coaching, return `conflicts`.

## Run Policy

- `discover_missing.phase.batch.max_candidates`: 10 for current head coaches, 5 for full staff or historical runs.
- `discover_missing.phase.batch.max_batches`: 6 for current head coaches, 20 for bounded full staff or historical popular-scope runs.
- `verify_existing_only.phase.batch.batch_size`: 10 for current-season rows, 20 for historical rows from structured sources.
- Broad historical coaching discovery should focus on head coaches and heavily reported/high-value coaching relationships first.

