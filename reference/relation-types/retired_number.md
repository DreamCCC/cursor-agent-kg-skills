# retired_number 退役号码

## Definition

表示某球员的球衣号码被某球队退役。

## Entity Rule

```text
Player -> Team
```

## Discovery Coverage

- `discover_missing` can use `source_exhaustive` when all official team retired-number pages or team history pages in the current bounded team set are checked.
- Exclude honored-only numbers unless the source explicitly states the number is officially retired.
- Do not assume a fixed team count; derive the team set from current official sources.

## Scope

- 只抽取球队正式退役号码。
- 号码、退役日期、展示名称放入 `properties`。
- 联盟级纪念号码可记录，但需在属性里标注适用范围。

## Preferred Sources

- NBA.com team-specific retired numbers articles, such as `https://www.nba.com/news/la-lakers-retired-numbers`
- NBA.com
- 球队官网 retired numbers / history 页面
- Basketball Reference 仅作辅助核验

## Acquisition Workflow

1. Start from current NBA team list.
2. Search NBA.com for team-specific retired numbers pages first.
3. If NBA.com lacks a team page, search the official team site for retired numbers, rafters, history, or jersey retirement pages.
4. Extract player name, team, retired number, and retirement or ceremony date when available.
5. Cross-check against Basketball Reference only when official pages are incomplete.
6. Put the jersey number in `properties.number`; do not encode it into the target team entity.

## Suggested Properties

- `number`
- `retired_date`
- `ceremony_date`
- `note`

## Exclusions

- 不抽取“球员穿过某号码”，那不是退役号码。
- 不抽取未正式退役、只是暂停使用或传闻中的号码。

