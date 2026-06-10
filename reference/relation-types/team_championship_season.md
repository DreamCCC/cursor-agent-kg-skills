# team_championship_season 球队夺冠赛季

## Definition

表示 NBA 球队获得总冠军的赛季。

## Entity Rule

```text
Team -> Season
```

## Discovery Coverage

- `discover_missing` can use `source_exhaustive` when the official NBA champions/history page is fully checked for the requested league scope.
- Only NBA championship seasons are in scope for phase one; do not include conference or division titles.

## Scope

- 一期只抽取 NBA 总冠军赛季。
- 目标实体命名建议使用 `2023-24 NBA Season` 或项目统一的赛季命名。
- 总冠军本身也可通过 `win_honor` 表达；本关系用于更直接回答“哪几年夺冠”。

## Preferred Sources

- NBA.com history / finals pages
- Basketball Reference champions page
- ESPN NBA history

## Acquisition Workflow

1. Use NBA.com history or finals pages as the first source for championship seasons.
2. Cross-check champion list with Basketball Reference champions page.
3. Generate one `Team -> Season` triple per championship season.
4. Put finals opponent, series result, and Finals MVP in `properties` when available.
5. Use consistent season entity names across all teams.

## Suggested Properties

- `season`
- `finals_opponent`
- `series_result`
- `finals_mvp`

## Exclusions

- 不抽取分区冠军、赛区冠军，除非后续扩展关系口径。
- 不抽取季中锦标赛冠军到本关系。

## Estimate Workflow

1. Use NBA.com history/finals champion pages as the primary count source for championship seasons.
2. Count one `Team -> Season` fact per NBA championship season in the requested league scope.
3. Query existing `published` and `pending_review` `team_championship_season` rows and count unique `Team -> Season` triples.
4. Cross-check champion seasons with Basketball Reference champions page or ESPN NBA history.
5. Ensure the estimate excludes conference titles, division titles, and in-season tournament titles.

## Verify Workflow

1. Verify only the input `relation_ids`.
2. Confirm the champion team and season from NBA.com history/finals pages.
3. Cross-check series result or finals opponent with Basketball Reference when properties need improvement.
4. If a valid row lacks `season`, `finals_opponent`, `series_result`, `finals_mvp`, or stronger source evidence, return `update_candidates`.
5. If the row describes a non-NBA championship or non-championship honor, return `conflicts`.

## Run Policy

- `discover_missing.phase.batch.max_candidates`: 10.
- `discover_missing.phase.batch.max_batches`: 10 for NBA championship seasons.
- `verify_existing_only.phase.batch.batch_size`: 20.

