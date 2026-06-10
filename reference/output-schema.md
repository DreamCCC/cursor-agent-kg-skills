# 输出 JSON Schema

Agent 最终只返回一个 JSON 对象，不要混入解释性正文。

```json
{
  "relation_code": "team_home_arena",
  "run_mode": "discover_missing",
  "phase": "batch",
  "summary": {
    "existing_count": 0,
    "new_candidates": 0,
    "update_candidates": 0,
    "conflicts": 0,
    "unchanged": 0
  },
  "coverage": {
    "status": "source_exhaustive",
    "empty_reason": null,
    "sources_checked": [
      {
        "url": "https://www.nba.com/...",
        "title": "Example source title",
        "coverage_scope": "Current NBA team arena list"
      }
    ],
    "source_items_checked": 0,
    "notes": []
  },
  "estimate": {
    "scope": "exhaustive",
    "scope_note": "For large relation types this may describe a hot/high-value subset rather than every historical relation.",
    "estimated_total": 0,
    "existing_count": 0,
    "estimated_missing": 0,
    "confidence": 0.95,
    "validation": {
      "cross_checked": true,
      "sources_checked": [
        {
          "url": "https://www.nba.com/...",
          "title": "Example authoritative count source",
          "coverage_scope": "Count source used for estimate"
        }
      ],
      "count_consistency_notes": []
    },
    "recommended_run_policy": {
      "max_candidates": 5,
      "max_batches": 20,
      "continuous": true,
      "safety_batches": 3
    },
    "risk_notes": []
  },
  "verify_plan": {
    "total_existing": 0,
    "batch_size": 20,
    "max_batches": 0,
    "batch_key": "relation_ids",
    "relation_ids": [],
    "notes": []
  },
  "new_candidates": [
    {
      "source_entity_type": "Team",
      "source_entity_name": "Los Angeles Lakers",
      "source_entity_id": "existing-source-entity-id",
      "relation_type": "team_home_arena",
      "target_entity_type": "Arena",
      "target_entity_name": "Crypto.com Arena",
      "target_entity_id": "existing-target-entity-id",
      "entity_resolution": {
        "source": {
          "status": "existing",
          "matched_by": "entity_name",
          "entity_id": "existing-source-entity-id",
          "canonical_name": "Los Angeles Lakers",
          "needs_create": false
        },
        "target": {
          "status": "missing",
          "matched_by": null,
          "entity_id": null,
          "canonical_name": "Crypto.com Arena",
          "needs_create": true,
          "create_suggestion": {
            "entity_type": "Arena",
            "entity_name": "Crypto.com Arena",
            "aliases": ["Staples Center"],
            "reason": "Official NBA source identifies this as the Lakers home arena, but no matching Arena entity exists."
          }
        }
      },
      "properties": {
        "start_season": "1999-00"
      },
      "description": "The Los Angeles Lakers play home games at Crypto.com Arena.",
      "source_url": "https://www.nba.com/lakers/...",
      "original_sources": [
        {
          "url": "https://www.nba.com/lakers/...",
          "title": "Example source title",
          "snippet": "Evidence snippet supporting the triple.",
          "retrieved_at": "2026-05-28"
        }
      ],
      "confidence": 0.95,
      "why_valid": "Official team source states this arena is the team's home venue.",
      "dedupe_key": "Team|Los Angeles Lakers|team_home_arena|Arena|Crypto.com Arena"
    }
  ],
  "update_candidates": [
    {
      "existing_relation_id": "relation-id",
      "change_type": "properties_or_source_update",
      "reason": "Existing triple is valid but lacks start_season and stronger source evidence.",
      "proposed_patch": {
        "properties": {
          "start_season": "1999-00"
        },
        "source_url": "https://..."
      },
      "evidence": []
    }
  ],
  "conflicts": [
    {
      "existing_relation_id": "relation-id",
      "issue": "Source suggests a different target entity.",
      "existing_value": "Old value",
      "observed_value": "New value",
      "evidence": []
    }
  ],
  "unchanged_examples": [],
  "self_check": {
    "relation_type_exists": true,
    "entity_rules_match": true,
    "existing_instances_checked": true,
    "entity_instances_checked": true,
    "missing_entities_reported": true,
    "duplicates_checked": true,
    "sources_verified": true,
    "out_of_scope_removed": true,
    "notes": []
  }
}
```

## Candidate Rules

- `relation_type` must equal input `relation_code`.
- `run_mode` must equal the input mode.
- `phase` must equal the input phase when a phase is provided.
- In `verify_existing_only`, `new_candidates` must be empty.
- In `discover_missing`, `update_candidates`, `conflicts`, and `unchanged_examples` must be empty arrays.
- In `discover_missing`, existing triples with better evidence or contradictory evidence must be skipped and left for `verify_existing_only`; mention them only in `self_check.notes` or `coverage.notes`.
- In `discover_missing phase=estimate`, `new_candidates`, `update_candidates`, `conflicts`, and `unchanged_examples` must all be empty arrays.
- In `discover_missing phase=estimate`, `estimate` must include `estimated_total`, `existing_count`, `estimated_missing`, `validation`, and `recommended_run_policy`.
- In `discover_missing phase=batch`, `new_candidates` must respect `max_candidates` if that parameter is provided.
- In `verify_existing_only phase=plan`, `new_candidates`, `update_candidates`, `conflicts`, and `unchanged_examples` must all be empty arrays.
- In `verify_existing_only phase=plan`, `verify_plan` must include `total_existing`, `batch_size`, `max_batches`, and `batch_key`.
- In `verify_existing_only phase=batch`, only verify the supplied `relation_ids` when present.
- `new_candidates`, `update_candidates`, and `conflicts` together should respect `max_candidates` if that parameter is provided, except `verify_existing_only` may return one result per supplied relation id.
- Entity types must match configured rules.
- Both endpoint entities must include `entity_resolution`.
- If an entity exists in `nba_ko_entity_tags`, include its `entity_id`.
- If an entity is missing, set `needs_create = true` and include `create_suggestion`.
- `source_url` must be the strongest source URL.
- `original_sources` should include every supporting source used.
- `confidence` range is `0.0` to `1.0`.
- If confidence is below `0.75`, omit the candidate in `discover_missing`; in `verify_existing_only`, prefer `conflicts` or omit the item.
- If no new candidates are returned in `discover_missing`, set `coverage.empty_reason` to `source_exhaustive_empty` or `search_exhausted_empty`.

## Phase Rules

### `discover_missing phase=estimate`

Use this phase to plan the batch run. Do not return candidate triples.

Required estimate behavior:

- Estimate the relation universe for the requested scope.
- Count existing `published` and `pending_review` unique triples.
- Compute `estimated_missing`.
- Strictly cross-check the estimate with authoritative sources, database counts, and relation-specific sanity bounds or secondary sources when available.
- For large relation types, scope may be a popular/high-value subset rather than full historical coverage. Clearly state this in `estimate.scope_note` and `coverage.notes`.
- Recommend `max_candidates`, `max_batches`, and `safety_batches`.

### `discover_missing phase=batch`

Use this phase to return only missing triples. Each run is stateless, so query the database again and treat `published` and `pending_review` as existing.

### `verify_existing_only phase=plan`

Use this phase only to summarize or validate a batch plan. The backend may also build the batch plan directly from database relation ids.

### `verify_existing_only phase=batch`

Use this phase to verify only the existing relations provided by `relation_ids`. Do not search for unrelated existing rows and do not discover new triples.

