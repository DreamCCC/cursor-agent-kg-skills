# arena_location 场馆所在地

## Definition

表示 NBA 场馆所在城市或地区，用于串联球队、场馆和地理位置。

## Entity Rule

```text
Arena -> City
```

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

