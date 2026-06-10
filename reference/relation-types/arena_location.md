# arena_location 场馆所在地

## Definition

表示 NBA 场馆所在城市或地区，用于串联球队、场馆和地理位置。

## Entity Rule

```text
Arena -> City
```

## Discovery Coverage

- `discover_missing` can use `source_exhaustive` only for a bounded arena set, such as arenas already linked from `team_home_arena` or a fully checked official arena list.
- Do not claim exhaustive coverage for all possible historical or non-current arenas unless the input source set is explicitly bounded.

## Scope

- 抽取 NBA 球队主场馆所在城市。
- 城市实体优先使用官方英文名，如 `Los Angeles`、`Boston`。
- 如场馆位于都会区或临近城市，以官方场馆地址城市为准。

## Preferred Sources

- 场馆官网
- NBA.com 球队官网
- 球队官网
- 官方城市/场馆地址页面

## Acquisition Workflow

1. Use existing `team_home_arena` or a known arena list as the arena baseline.
2. For each arena, visit the arena official site or official contact/address page.
3. Extract the city from the official address.
4. Cross-check with NBA.com or team pages only if the arena official site is unavailable.
5. Put state, country, or full address in `properties`, not in the target city name.

## Suggested Properties

- `state`
- `country`
- `address`

## Exclusions

- 不抽取球队所在地，球队历史城市使用 `team_historical_location`。
- 不把州、省、国家作为目标，除非系统后续新增对应实体规则。

## Estimate Workflow

1. Build the bounded arena set from existing `team_home_arena` rows or a fully checked current NBA arena list.
2. Count unique arenas in that bounded set.
3. Query existing `published` and `pending_review` `arena_location` rows and count unique `Arena -> City` triples.
4. Cross-check the arena set with NBA.com arena/team pages or team official pages.
5. If historical or non-current arenas are included, state the bounded source set in `estimate.scope_note`.

## Verify Workflow

1. Verify only the input `relation_ids`.
2. Check each arena city against the arena official address page first.
3. If the row is valid but lacks `state`, `country`, `address`, or a stronger source URL, return `update_candidates`.
4. If the target is a metro area, state, country, or team city rather than official address city, return `conflicts`.

## Run Policy

- `discover_missing.phase.batch.max_candidates`: 10.
- `discover_missing.phase.batch.max_batches`: 6 for current NBA arenas.
- `verify_existing_only.phase.batch.batch_size`: 20.

