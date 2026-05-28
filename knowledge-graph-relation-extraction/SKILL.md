---
name: knowledge-graph-relation-extraction
description: Extracts NBA knowledge graph relation candidates for a specific relation_code. Use when asked to research, verify, update, or generate NBA knowledge graph triples with database baseline checks and web evidence.
---

# Knowledge Graph Relation Extraction

## Mission

For one `relation_code`, research authoritative sources, compare against existing database triples, and return review-ready relation candidates. Default behavior is read-only research and structured output; do not publish data directly.

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

4. Acquire evidence.
   - Follow relation-specific acquisition steps when present, such as reading official PDFs, team roster pages, fixed award history pages, or internal documents.
   - If no specific workflow exists, search the web using official NBA, team, arena, league, Hall of Fame, Basketball Reference, ESPN, or clearly cited encyclopedia-style sources.
   - Use secondary sources only when official sources do not cover the fact or to cross-check official data.
   - Capture URL, title, snippet, and retrieval date for every claim.

5. Build candidates and diffs.
   - `new_candidates`: valid triples not present in existing baseline.
   - `update_candidates`: existing triples whose properties or evidence need improvement.
   - `conflicts`: web evidence contradicts an existing triple.
   - `unchanged`: existing triples verified with no action needed.

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
- Do not hide missing endpoint entities; report them explicitly in candidate `entity_resolution`.
- If source quality is weak, put the item in `conflicts` or `needs_review`, not `new_candidates`.

