# team_predecessor 球队前身

## Definition

表示 NBA 球队历史沿革中的前身球队或历史名称主体。

## Entity Rule

```text
Team -> Team
```

## Discovery Coverage

- `discover_missing` can use `source_exhaustive` for teams whose complete franchise history pages or Basketball Reference franchise index entries are fully checked.
- If only search snippets or partial history pages are available, coverage must be `partial`.
- Do not infer predecessor relations from nickname similarity without explicit historical evidence.

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

## Estimate Workflow

1. Build the current team set from official NBA teams or database active `Team` entities.
2. For each team, inspect NBA.com or official team history pages and Basketball Reference Franchise Index `Team Names`.
3. Count explicit predecessor or historical franchise-name identities, excluding simple aliases and nicknames.
4. Query existing `published` and `pending_review` `team_predecessor` rows and count unique `Team -> Team` triples.
5. Cross-check each counted predecessor with at least one official or structured history source.

## Verify Workflow

1. Verify only the input `relation_ids`.
2. Confirm that the target is a true predecessor or historical franchise identity, not just a nickname or city.
3. Use NBA.com team history and Basketball Reference franchise history for cross-checking.
4. If a valid row lacks season/year range or reason, return `update_candidates`.
5. If the row should be an alias or `team_historical_location` fact instead, return `conflicts`.

## Run Policy

- `discover_missing.phase.batch.max_candidates`: 10.
- `discover_missing.phase.batch.max_batches`: 10 for current NBA franchise histories.
- `verify_existing_only.phase.batch.batch_size`: 20.

