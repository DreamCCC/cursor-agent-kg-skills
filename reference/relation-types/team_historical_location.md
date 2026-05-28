# team_historical_location 球队历史所在地

## Definition

表示 NBA 球队历史上曾经所在的城市或地区。

## Entity Rule

```text
Team -> City
```

## Discovery Coverage

- `discover_missing` can use `source_exhaustive` for teams whose complete franchise history pages or Basketball Reference franchise index entries are fully checked.
- If only current team pages or partial history sources are available, coverage must be `partial`.
- Do not duplicate `team_predecessor`; this relation captures location, not historical team-name identity.

## Scope

- 用于表达 franchise 搬迁和所在地沿革。
- 可记录当前所在地和历史所在地，但一期更关注历史所在地。
- 与 `team_predecessor` 配合使用：前者表达城市，后者表达球队主体/名称沿革。

## Preferred Sources

- NBA.com team history pages
- Basketball Reference Franchise Index pages, such as `https://www.basketball-reference.com/teams/LAL/`
- 官方球队历史页面
- Wikipedia 仅作辅助核验

## Acquisition Workflow

1. Start from current NBA team list.
2. Inspect official team history pages for relocation and city history.
3. Use Basketball Reference Franchise Index pages to infer city/name periods from historical team names and season rows.
4. Extract cities or regions associated with each franchise period.
5. Generate `Team -> City` triples for historical locations; include current location only if needed for completeness.
6. Put season or year range in `properties`.
7. Cross-check relocations against at least two sources when official pages are vague.

## Suggested Properties

- `start_year`
- `end_year`
- `start_season`
- `end_season`
- `is_current`

## Exclusions

- 不把主场馆所在地写到本关系，场馆城市使用 `arena_location`。
- 不抽取短期临时比赛地或中立场城市。

