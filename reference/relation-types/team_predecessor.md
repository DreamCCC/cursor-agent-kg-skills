# team_predecessor 球队前身

## Definition

表示 NBA 球队历史沿革中的前身球队或历史名称主体。

## Entity Rule

```text
Team -> Team
```

## Scope

- 适用于 franchise 历史沿革，如更名、迁移前的球队主体。
- 目标实体可以是历史球队名称实体。
- 如同一 franchise 有多段历史，应分别记录并写清时间范围。

## Preferred Sources

- NBA.com team history pages
- Basketball Reference Franchise Index pages, such as `https://www.basketball-reference.com/teams/LAL/`
- 官方球队历史页面
- Encyclopaedia Britannica / Wikipedia 仅作辅助核验

## Acquisition Workflow

1. Start from current NBA team list.
2. For each team, inspect NBA.com or official team history pages for franchise name changes and relocation notes.
3. Use Basketball Reference Franchise Index pages to cross-check `Team Names` and season ranges.
4. Generate one triple per predecessor or historical franchise name.
5. Put year or season range in `properties`.
6. If a name is only a nickname or abbreviation, add it as alias in review notes instead of creating a relation.

## Suggested Properties

- `start_year`
- `end_year`
- `start_season`
- `end_season`
- `reason`

## Exclusions

- 不抽取只是简称、昵称或别名的关系；别名放实体 aliases。
- 不把所在城市变化单独写入本关系，城市沿革使用 `team_historical_location`。

