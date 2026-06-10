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

## Estimate Workflow

1. First determine the requested honor scope. If the request is broad, scope the estimate to high-value honors only: MVP, Finals MVP, DPOY, Rookie of the Year, All-NBA Team, NBA Champion, and Hall of Fame.
2. Use NBA.com award/history pages and the Hall of Fame official site as primary count sources for bounded award families.
3. Cross-check award winners and seasons with Basketball Reference award pages or ESPN history pages.
4. Query existing `published` and `pending_review` `win_honor` rows and count unique `Player/Team -> Award` triples within the same bounded honor scope when possible.
5. If the estimated universe is large, do not attempt full historical coverage. Set `estimate.scope_note` to explain the popular/high-value subset, for example famous players, championship teams, and heavily reported awards.
6. Estimate must clearly state which award families are included and which are excluded.

## Verify Workflow

1. Verify only the input `relation_ids`.
2. Check whether each award fact is supported by NBA.com, the Hall of Fame official site, Basketball Reference, or another authoritative history page.
3. If the fact is valid but lacks `season`, `year`, `award_level`, stronger source URL, or original sources, return `update_candidates`.
4. If the row uses an overly broad or non-official award entity, return `conflicts` with a normalization recommendation.

## Run Policy

- `discover_missing.phase.estimate`: strict cross-checking is required because this relation can become very large.
- `discover_missing.phase.batch.max_candidates`: 5 for broad or popular-scope runs; 10 only for a single bounded award family.
- `discover_missing.phase.batch.max_batches`: 20 for popular/high-value scope; use repeated manual runs if more coverage is desired.
- `verify_existing_only.phase.batch.batch_size`: 10 for rows needing web verification, 20 for rows from structured award pages.
- For broad runs, prioritize famous players, championship teams, MVP/FMVP winners, Hall of Fame inductees, and heavily reported honors rather than full historical coverage.

