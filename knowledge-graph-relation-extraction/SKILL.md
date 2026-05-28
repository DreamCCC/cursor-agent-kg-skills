---
name: knowledge-graph-relation-extraction
description: Extracts NBA knowledge graph relation candidates for a specific relation_code. Use when asked to research, verify, update, or generate NBA knowledge graph triples with database baseline checks and web evidence.
---

# Knowledge Graph Relation Extraction

## Mission

For one `relation_code`, research authoritative sources, compare against existing database triples, and return review-ready relation candidates. Default behavior is read-only research and structured output; do not publish data directly.

The skill supports two primary modes:

- `verify_existing_only`: verify existing triples only. Return `update_candidates`, `conflicts`, and `unchanged`; do not discover or return new triples.
- `discover_missing`: discover missing triples only. Return `new_candidates` that are absent from both `published` and `pending_review`; `update_candidates`, `conflicts`, and `unchanged_examples` must be empty in this mode.

## Required References

Read these before running:

- `../reference/database-api.md`
- `../reference/output-schema.md`
- `../reference/quality-gate.md`
- `../reference/workflow-validation.md`
- `../reference/relation-types/{relation_code}.md`

## Workflow

1. Confirm relation type exists and is active.
   - Query `ai_data.nba_ko_relation_types`.
   - Query entity rules from `ai_data.nba_ko_relation_type_rules`.
   - Stop if the relation is missing, inactive, blacklisted, or entity rules differ from the relation reference.

2. Load existing instances.
   - Query `ai_data.nba_ko_relations` where `is_deleted = 0` and `relation_type = {relation_code}`.
   - Treat `published` and `pending_review` rows as existing baseline.
   - Use source name, relation type, target name, and entity ids when available for duplicate checks.
   - Query existing `ai_data.nba_ko_entity_tags` for all source and target entity types allowed by the relation rules.
   - Before returning any candidate, match both endpoint entities against existing entity tags by exact name, alias, and external id when available.
   - If an endpoint entity is missing, keep the candidate but mark that endpoint as `needs_create`; do not silently assume it already exists.

3. Read relation-specific reference.
   - Follow the scope, exclusions, target entities, preferred sources, and evidence rules in `reference/relation-types/{relation_code}.md`.
   - If the reference contains `Acquisition Workflow`, execute that workflow first.
   - Use the generic web search workflow only as fallback or for cross-checking.

4. Execute the selected mode.
   - For `verify_existing_only`, iterate over existing baseline triples, verify whether each fact is still valid, and check whether `description`, `properties`, `source_url`, `original_sources`, or entity ids need improvement.
   - For `discover_missing`, build a source-driven candidate universe from the relation-specific acquisition workflow, diff it against existing `published` and `pending_review` triples, and return only missing triples.
   - If `discover_missing` uses an exhaustively enumerable source, set coverage as `source_exhaustive`; otherwise set coverage as `partial`.
   - If no specific workflow exists, use generic official-source web search, but never claim exhaustive coverage.
   - Capture URL, title, snippet, and retrieval date for every claim.

5. Build candidates and diffs.
   - In `discover_missing`, `new_candidates` are valid triples not present in existing baseline.
   - In `verify_existing_only`, `update_candidates` are existing triples whose properties, description, evidence, or entity ids need improvement.
   - In `verify_existing_only`, `conflicts` are existing triples contradicted by stronger evidence or requiring human resolution.
   - `unchanged` contains existing triples verified with no action needed.
   - In `discover_missing`, do not return `update_candidates`, `conflicts`, or `unchanged_examples`; existing-row observations may only be summarized in `self_check.notes` or `coverage.notes`.
   - Do not put the same triple in both `new_candidates` and `update_candidates`.

6. Self-check.
   - Validate entity types.
   - Validate whether each endpoint entity already exists.
   - Check duplicate triples.
   - Verify source credibility.
   - Confirm the evidence actually supports the relation.
   - Ensure no high-frequency or out-of-scope relation slipped in.

7. Return only the structured output.
   - Use the JSON format in `output-schema.md`.
   - If nothing changes, return an empty candidate list with a clear summary.

## Guardrails

- Never mark output as `published`.
- Do not invent facts from model memory.
- Do not use vague sources without a URL.
- Do not use a relation if the evidence only implies it weakly.
- Do not create duplicate triples that already exist in `published` or `pending_review`.
- In `discover_missing`, skip existing triples instead of returning them as updates.
- In `discover_missing`, never return `conflicts`; conflicts are only valid in `verify_existing_only`.
- In `discover_missing`, never return `unchanged_examples`; unchanged examples are only valid in `verify_existing_only`.
- In `verify_existing_only`, do not return `new_candidates`.
- If no new triples are found, explain whether the empty result is source-exhaustive or only this run's search found nothing.
- Do not hide missing endpoint entities; report them explicitly in candidate `entity_resolution`.
- If source quality is weak, put the item in `conflicts` or `needs_review`, not `new_candidates`.

