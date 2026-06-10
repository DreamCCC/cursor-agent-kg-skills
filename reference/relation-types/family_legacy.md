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

## Estimate Workflow

1. Treat this as an open-ended relation unless the input provides a bounded player list.
2. For broad runs, estimate a popular/high-value subset rather than all NBA family relations.
3. Build the subset from famous players, current high-interest players, Hall of Fame profiles, second-generation NBA stories, and heavily reported family relationships.
4. Query existing `published` and `pending_review` `family_legacy` rows and count unique `Player -> Player` triples.
5. Cross-check candidate counts with Basketball Reference `Relatives` fields and NBA.com or Hall of Fame profile evidence.
6. In `estimate.scope_note`, state that broad estimates cover high-value public family relationships, not every historical NBA relative.

## Verify Workflow

1. Verify only the input `relation_ids`.
2. Check Basketball Reference `Relatives` field or official NBA/Hall of Fame/team profile evidence.
3. If a valid row lacks `properties.relationship`, direction notes, or stronger sources, return `update_candidates`.
4. If the relationship is unsourced, only rumored, or not a family relationship, return `conflicts`.

## Run Policy

- `discover_missing.phase.batch.max_candidates`: 5 for broad popular-scope runs, 10 for a bounded player list.
- `discover_missing.phase.batch.max_batches`: 20 for popular/high-value scope.
- `verify_existing_only.phase.batch.batch_size`: 10 because evidence often requires profile-level checking.

