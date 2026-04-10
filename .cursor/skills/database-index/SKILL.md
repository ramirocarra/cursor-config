---
name: database-index
description: >-
  Analyzes SQL and ORM query usage from code context and proposes safe,
  evidence-backed database indexes. Use when reviewing PRs that touch queries,
  repositories, or migrations; when the user asks for missing indexes, index
  review, or query performance; or after an impact scan that surfaced DB layers.
---

# Database index analysis

## Goal

From code context and changed files, identify likely missing indexes and propose practical, low-risk recommendations tied to **observed** query patterns—not speculation.

## Inputs

Reuse context when available (e.g. parent workflow impact scan):

- Changed files and related unchanged call sites
- Repository/service layers and explicit SQL or ORM queries
- Schema definitions and migration files

If context is thin, say so and limit recommendations accordingly.

## Method

### 1) Discover query patterns

- Inspect SQL and ORM queries in scope.
- Group by **table** and **pattern**: filters, joins, `ORDER BY`, grouping, aggregates.
- Prioritize paths that appear hot, repeated, or latency-sensitive when evidence exists (call frequency, comments, job/cron paths, list endpoints).

### 2) Map to index opportunities

- Recommend indexes only when a **clear** pattern match exists.
- Weigh read benefit vs write overhead and index maintenance.
- Do **not** propose speculative indexes without concrete query evidence.

### 3) Apply index best practices

- **B-tree** (default): equality, range, and `ORDER BY` on scalar columns.
- **Composite column order**: equality predicates first, then range, then ordering/grouping when it helps the same access path.
- **Avoid redundancy**: duplicates, left-prefix overlap with existing indexes, or what unique/PK constraints already provide.
- **Unique indexes** when business rules require uniqueness (and queries support that story).
- **Foreign-key join paths**: ensure referencing columns used in joins/filters are indexed when missing.
- **Partial indexes** when predicates are stable and selective (e.g. `WHERE status = 'active'` used consistently).
- **Low cardinality alone**: avoid unless partial index rationale is strong.
- **GIN/GiST** only when operators warrant it (JSONB, arrays, full-text, geometry, etc.).
- **INCLUDE / covering**: only when heap fetch reduction is meaningful for the observed pattern.

### 4) Validate against current schema

- Read existing migrations/schema for indexes and constraints.
- Label each recommendation: **new**, **duplicate**, or **superseded** (by existing or better index).

### 5) Migration handling

- Detect migration tooling if present (e.g. sqlx, Alembic, Prisma, Knex, Rails, raw SQL folders).
- **Do not** create or edit migration files without explicit user confirmation.
- Ask clearly, offering **two** options:
  1. Create a **new** migration for the proposed indexes.
  2. Add indexes to a migration **already created in the current session** (if one exists).

## Output format

Return results in this structure:

### 1) Candidate indexes

One line or short block per candidate:

- table
- index type (e.g. btree unique, btree composite, partial, GIN)
- columns and order; optional `WHERE` (partial) or `INCLUDE`
- mapped query pattern(s) (file/symbol or query shape)
- expected benefit (qualitative)
- trade-offs / risks
- confidence: **high** / **medium** / **low**

### 2) Exclusions

Queries or tables reviewed where indexing was **not** recommended, with a one-line reason each.

### 3) Existing-index conflicts

Duplicates, strict left-prefix overlaps, or cases where a new index would be redundant.

### 4) Migration plan

- Detected migration system (or **none found**).
- Restate the confirmation question if edits are desired (new migration vs in-session migration).
- After the user confirms, state whether to create a new migration or update the in-session one.

## Guardrails

- Do **not** claim performance guarantees or exact speedups.
- Do **not** suggest indexes with no tie to observed query patterns.
- If evidence is insufficient, state that explicitly instead of guessing.
