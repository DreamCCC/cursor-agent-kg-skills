# referee_officiate 裁判员执裁

## Definition

表示裁判员属于 NBA / G League / WNBA 等官方裁判员名单，具备执裁该联盟比赛的身份。

## Entity Rule

```text
Referee -> League
```

## Discovery Coverage

- `discover_missing` can use `source_exhaustive` for a specific season and league when the official officials guide or official staff list is fully checked.
- Do not use single-game assignments to claim a referee-list relation.
- If the season or league is not bounded, coverage must be `partial`.

## Scope

- 一期抽取官方裁判员名单关系，不抽取单场比赛执裁安排。
- 目标实体可为 `NBA`、`G League`、`WNBA` 等联盟实体。

## Preferred Sources

- Official NBA Officials Guide PDF, for example `https://cms.nba.com/wp-content/uploads/sites/4/2025/10/2025-26-NBA-Officials-Guide.pdf`
- official.nba.com
- NBA officiating staff PDF
- G League 官方网站
- NBA 官方新闻稿

## Acquisition Workflow

1. Search for the current season official NBA Officials Guide PDF.
2. Prefer the official PDF over web pages because it contains the full staff and non-staff list.
3. Extract referee names, jersey numbers, staff/non-staff status, season, and role from the official list.
4. Normalize every referee as `Referee -> League`, usually targeting `NBA`.
5. Put the official staff list URL into `properties.staff_list_url`.
6. Cross-check unclear names against official.nba.com or NBA news pages.
7. Do not use referee assignment pages to create the staff-list relation; assignment pages are only for cross-checking names.

## Fallback Workflow

If the official PDF cannot be read:

1. Use official.nba.com staff list pages or NBA announcement pages.
2. Use reputable secondary sources only to identify candidate names.
3. Mark low-confidence names as `conflicts` or omit them.

## Suggested Properties

- `season`
- `role`
- `staff_list_url`

## Exclusions

- 不抽取某场比赛由谁执裁，这属于高频赛程事件。
- 不抽取未经官方确认的裁判员传闻或媒体推测。

## Estimate Workflow

1. Use the official NBA Officials Guide PDF as the primary count source for the requested season and league.
2. Count staff and non-staff officials from the official list; keep the count source URL and page/section in `estimate.validation.sources_checked`.
3. Query existing `published` and `pending_review` `referee_officiate` rows and count unique `Referee -> League` triples for the same league/season when season data is available.
4. Cross-check the official count with official.nba.com staff pages or PDF profile sections when available.
5. Recommended estimate should include `estimated_total`, `existing_count`, `estimated_missing`, and notes explaining any difference between official count and database count.

## Verify Workflow

1. Verify only the input `relation_ids`.
2. Check that each referee remains supported by the official staff/non-staff list for the claimed season and league.
3. If the triple is valid but lacks `season`, `role`, `staff_list_url`, or official source snippets, return `update_candidates`.
4. If a row appears to describe a single-game assignment rather than staff-list membership, return `conflicts`.

## Run Policy

- `discover_missing.phase.estimate`: source-exhaustive is allowed only for a bounded season and league.
- `discover_missing.phase.batch.max_candidates`: 5.
- `discover_missing.phase.batch.max_batches`: 20 for current NBA staff/non-staff lists.
- `verify_existing_only.phase.batch.batch_size`: 20.

