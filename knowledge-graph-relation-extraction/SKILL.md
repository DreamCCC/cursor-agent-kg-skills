---
name: knowledge-graph-relation-extraction
description: Extracts NBA knowledge graph relation candidates for a specific relation_code. Use when asked to research, verify, update, or generate NBA knowledge graph triples with database baseline checks and web evidence.
---

# Knowledge Graph Relation Extraction

## Mission

For one `relation_code`, research authoritative sources, compare against existing database triples, and return review-ready relation candidates. Default behavior is read-only research and structured output; do not publish data directly.

The skill supports two primary modes. Each mode may run in phases so the backend can batch long-running work safely:

- `verify_existing_only`: verify existing triples only. Return `update_candidates`, `conflicts`, and `unchanged_examples`; do not discover or return new triples. Use `phase=plan` to plan batches when requested, and `phase=batch` to verify only the supplied `relation_ids`.
- `discover_missing`: discover missing triples only. Use `phase=estimate` to estimate volume and recommend a run policy without returning candidates. Use `phase=batch` to return `new_candidates` that are absent from both `published` and `pending_review`; `update_candidates`, `conflicts`, and `unchanged_examples` must be empty in this mode.

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
   - If the reference contains `Estimate Workflow`, execute it during `discover_missing phase=estimate`.
   - If the reference contains `Verify Workflow`, execute it during `verify_existing_only phase=batch`.
   - If the reference contains `Run Policy`, use it when recommending batch size, max batches, and scope notes.
   - Use the generic web search workflow only as fallback or for cross-checking.

4. Execute the selected mode.
   - For `verify_existing_only phase=plan`, summarize the existing relation count and recommend a stable batch plan; do not return candidates.
   - For `verify_existing_only phase=batch`, verify only the input `relation_ids`, check whether each fact is still valid, and check whether `description`, `properties`, `source_url`, `original_sources`, or entity ids need improvement.
   - For `discover_missing phase=estimate`, estimate the total candidate universe, existing baseline count, missing count, validation evidence, and recommended run policy; do not return candidates.
   - For `discover_missing phase=batch`, build a source-driven candidate universe from the relation-specific acquisition workflow, diff it against existing `published` and `pending_review` triples, and return only missing triples up to `max_candidates`.
   - If `discover_missing` uses an exhaustively enumerable source, set coverage as `source_exhaustive`; otherwise set coverage as `partial`.
   - If no specific workflow exists, use generic official-source web search, but never claim exhaustive coverage.
   - Capture URL, title, snippet, and retrieval date for every claim.

5. Build candidates and diffs.
   - In `discover_missing phase=batch`, `new_candidates` are valid triples not present in existing baseline.
   - In `verify_existing_only phase=batch`, `update_candidates` are existing triples whose properties, description, evidence, or entity ids need improvement.
   - In `verify_existing_only phase=batch`, `conflicts` are existing triples contradicted by stronger evidence or requiring human resolution.
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
   - Always include `phase` when the input includes `phase`.

## Guardrails

- Never mark output as `published`.
- Do not invent facts from model memory.
- Do not use vague sources without a URL.
- Do not use a relation if the evidence only implies it weakly.
- Do not create duplicate triples that already exist in `published` or `pending_review`.
- `discover_missing phase=estimate` must not return candidates; it is for volume estimate and run policy only.
- `discover_missing phase=batch` must respect `max_candidates` when provided.
- In `discover_missing`, skip existing triples instead of returning them as updates.
- In `discover_missing`, never return `conflicts`; conflicts are only valid in `verify_existing_only`.
- In `discover_missing`, never return `unchanged_examples`; unchanged examples are only valid in `verify_existing_only`.
- `verify_existing_only phase=batch` must verify only the supplied `relation_ids` when they are present.
- In `verify_existing_only`, do not return `new_candidates`.
- For large relation types, `discover_missing` may intentionally cover only popular players, popular teams, heavily reported facts, or highest Q&A value facts. Clearly state this in `estimate.scope_note` and `coverage.notes`; do not claim full historical coverage.
- Estimate counts must be strictly cross-checked with authoritative sources, existing database unique counts, and relation-specific sanity bounds or secondary sources when available.
- If no new triples are found, explain whether the empty result is source-exhaustive or only this run's search found nothing.
- Do not hide missing endpoint entities; report them explicitly in candidate `entity_resolution`.
- If source quality is weak, put the item in `conflicts` or `needs_review`, not `new_candidates`.

