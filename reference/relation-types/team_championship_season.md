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

