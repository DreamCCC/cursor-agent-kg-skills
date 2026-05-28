# family_legacy 父子 / 家庭传承

## Definition

表示 NBA 球员之间稳定、公开可核验的家庭关系。

## Entity Rule

```text
Player -> Player
```

## Discovery Coverage

- `discover_missing` should default to `partial` because family relations are open-ended and sources are not exhaustively enumerable.
- A run may be source-exhaustive only for a bounded input player list whose authoritative profile pages were all checked.
- Returning no candidates means this run found no more high-confidence relations, not that all NBA family relations are exhausted.

## Scope

- 一期优先父子、兄弟、亲属中与 NBA 问答价值高的关系。
- 关系方向建议从较新一代或当前提问高频球员指向亲属，并在属性里写 `relationship`。

## Preferred Sources

- Basketball Reference player pages `Relatives` field
- NBA.com player profile / news
- Basketball Reference
- Hall of Fame profile
- 球队官网人物介绍

## Acquisition Workflow

1. Start from a curated list of high-value NBA family relationships or current entity aliases.
2. For each player, check the Basketball Reference player page `Relatives` field first when available.
3. Cross-check high-profile relations with NBA.com, Hall of Fame profiles, or official team bios.
4. Generate `Player -> Player` only when the family relationship is directly stated.
5. Put the relationship type, such as `father`, `son`, `brother`, or `cousin`, in `properties.relationship`.
6. Prefer one direction per fact unless product requirements need bidirectional triples.
7. Do not treat generic search snippets as sufficient evidence; the source page must expose the relationship directly.

## Fallback Workflow

If official sources are unavailable:

1. Use Basketball Reference and Hall of Fame profiles as fallback.
2. Avoid gossip, social media claims, or unsourced biography snippets.

## Suggested Properties

- `relationship`
- `note`

## Exclusions

- 不抽取未经权威来源确认的私人关系。
- 不抽取泛泛的“同乡”“好友”，只处理家庭血缘/亲属关系。

