# team_home_arena 球队主场馆

## Definition

表示 NBA 球队当前或明确历史阶段的主场馆。

## Entity Rule

```text
Team -> Arena
```

## Discovery Coverage

- `discover_missing` can use `source_exhaustive` for current active teams when the current NBA official team list and official arena/team pages are fully checked.
- Do not assume a fixed team count; derive the source team set from current official sources.
- Historical home arenas are not source-exhaustive in phase one unless the task provides a bounded franchise history source set.

## Scope

- 优先抽取当前 NBA 官方球队列表中的 active teams 主场馆。
- 如抽取历史主场，必须在 `properties` 写清 `start_season` / `end_season` 或说明。
- 一期优先当前主场，不主动扩展完整历史主场链。

## Preferred Sources

- NBA.com `A Complete List of all 30 NBA Arenas By Team`
- NBA.com team info pages, such as `https://www.nba.com/team/{team_id}`
- NBA.com 球队官网
- 球队官方官网
- 场馆官网
- Basketball Reference / Wikipedia 仅作辅助核验

## Acquisition Workflow

1. Build the team baseline from the current NBA.com teams directory or database active `Team` entities; do not assume the league always has exactly 30 teams.
2. Use NBA.com `A Complete List of all 30 NBA Arenas By Team` only as a currently verified source, not as a permanent count invariant.
3. Cross-check each team with its NBA.com team info page, which usually exposes `City`, `Arena`, and `Head Coach`.
4. If a team page is incomplete, use the team official website or arena official site.
5. Cross-check the arena name with the arena official site when possible.
6. Mark `is_current = true` for current home arenas.
7. If a team has temporary or alternate venues, ignore them unless the official source states they are home arenas.

## Suggested Properties

- `start_season`
- `end_season`
- `is_current`

## Exclusions

- 训练馆、临时比赛地、中立场不算主场馆。
- 不要把城市建成目标实体，城市使用 `arena_location`。

## Estimate Workflow

1. Build the current team universe from the current NBA.com teams directory or database active `Team` entities.
2. Count the current active teams from that source; do not hardcode 30 as a permanent invariant.
3. Query existing `published` and `pending_review` `team_home_arena` rows and count unique `Team -> Arena` triples.
4. Cross-check arena counts with NBA.com arena-by-team pages, team info pages, or team official pages.
5. If historical home arenas are requested, estimate only for the bounded franchise/source set and mark coverage as partial unless all franchise histories are fully checked.

## Verify Workflow

1. Verify only the input `relation_ids`.
2. Cross-check each team-arena fact with NBA.com team info pages and a team or arena official source when possible.
3. If a current home arena fact is valid but lacks `is_current`, `start_season`, or stronger official source evidence, return `update_candidates`.
4. If a row points to a training facility, neutral site, temporary venue, or city entity, return `conflicts`.

## Run Policy

- `discover_missing.phase.batch.max_candidates`: 10 for current active teams.
- `discover_missing.phase.batch.max_batches`: 6 for current active teams.
- `verify_existing_only.phase.batch.batch_size`: 20.
- Historical home arenas should be run only with a bounded source set and explicit scope notes.

