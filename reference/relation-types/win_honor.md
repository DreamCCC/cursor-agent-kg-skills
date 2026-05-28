# win_honor 获得荣誉

## Definition

表示球员或球队获得稳定、权威、低频的 NBA 相关荣誉。

## Entity Rules

```text
Player -> Award
Team -> Award
```

## Discovery Coverage

- `discover_missing` can use `source_exhaustive` only for explicitly bounded award pages, such as NBA official MVP, Finals MVP, champions, or Hall of Fame lists.
- If the task asks for broad player honors without a fixed source list, coverage must be `partial`.
- Do not claim exhaustive coverage for every possible honor across the open web.

## Scope

一期只抽取重大荣誉：

- NBA MVP
- Finals MVP
- Defensive Player of the Year
- Rookie of the Year
- All-NBA Team
- NBA Champion / 总冠军
- Hall of Fame / 名人堂入选

## Preferred Sources

- NBA.com awards / history pages
- Naismith Memorial Basketball Hall of Fame
- Basketball Reference awards pages
- ESPN history pages

## Acquisition Workflow

1. Choose one award family at a time, such as MVP, Finals MVP, DPOY, Rookie of the Year, All-NBA, NBA Champion, or Hall of Fame.
2. Use NBA.com history or awards pages for NBA awards and champions.
3. Use the Naismith Memorial Basketball Hall of Fame official site for Hall of Fame inductees.
4. Use Basketball Reference award pages to cross-check seasons and names.
5. Normalize award target entities consistently, for example `NBA MVP`, `Finals MVP`, `Naismith Memorial Basketball Hall of Fame`.
6. Put `season` or `year` in `properties`; do not encode year into the award entity name unless the system explicitly requires it.
7. Compare against existing triples to avoid duplicating the same player/team honor.

## Fallback Workflow

If NBA.com pages are incomplete:

1. Use Basketball Reference as the primary fallback for historical awards.
2. Use ESPN history pages as a second cross-check.
3. For Hall of Fame, do not use non-official pages unless the official Hall of Fame site is unavailable.

## Suggested Properties

- `season`
- `year`
- `rank_or_team`
- `award_level`

## Exclusions

- 不抽取周最佳、月最佳等高频荣誉。
- 不抽取媒体自行评选或非官方榜单。
- 名人堂入选放在本关系内，不单独建关系类型。

