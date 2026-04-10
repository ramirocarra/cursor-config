---
name: session-change-review
description: >-
  Runs an end-of-session impact, bug, database-index, and quality review from the
  current git diff. Use before wrapping a coding session, before commit/PR, or
  when the user asks for a full change review, impact analysis, or staged+unstaged diff audit.
agent: build
---

# End-of-session change impact and bug review

## Interaction rule

- If clarification is required for correct conclusions, ask **targeted** questions **before** the final report.
- Resolve open questions first, then publish the final report.

## Primary input

Current diff scope:

```bash
git diff --staged && git diff
```

Start from this diff; prioritize changed hunks, then expand per step 2.

## Execution

### 1) Anchor on the diff

- Use staged and unstaged diffs as the primary scope.
- Note every touched file path for downstream steps.

### 2) Expand impact context (subagent)

Launch a subagent to widen context around the diff:

- Read changed files and **related** unchanged files (callers, callees, same module, configs, tests, migrations).
- **Return**: complete file list reviewed (changed + unchanged).
- **Return**: concise summary emphasizing:
  - coupled logic (other places that may need matching changes)
  - significant business logic changes
  - possible race conditions
  - error propagation risks

### 3) Three parallel analyses

Use the **same** file list and step-2 findings for all three. Run these **in parallel** (separate subagent invocations in one turn).

**A) Bug-hunt subagent**

- Verify suspicious behavior from step 2.
- Check code integrity, project rules, and local consistency.
- Check business logic correctness.
- If behavior or requirements are unclear, **ask for clarification**; do not assume unspecified paths or handling.
- If there are no issues, state **explicitly: no issues** (do not invent findings).

**B) Database index analysis**

- Read and follow the **`database-index`** skill (`~/.cursor/skills/database-index/SKILL.md` after install from this repo’s `skills/database-index/SKILL.md`, or project `.cursor/skills/database-index/SKILL.md` if present).
- Same file list and step-2 findings as input.
- Present candidate indexes with rationale and trade-offs.
- If a migration strategy exists: **ask user confirmation** before creating/editing migrations.
- Offer: **new migration** vs **add to a migration created in this session**.

**C) Performance and code quality audit**

Check for:

- Copies/clones used once after the copy site where a **move** would suffice.
- `Copy`/`Clone` derives or manual impls with **no** call-site need (unused capability).
- Default expressions that **eagerly** evaluate when the language offers lazy defaults (`unwrap_or_else`, etc.).
- Fallible ops that propagate with `?` **without** context (hard to locate at runtime).
- `print!` / `println!` / bare `print` for diagnostics instead of the project logger.
- Iterator adapter mismatches (e.g. flatten vs `filter_map` when the closure returns `Option`).

If nothing applies, state **explicitly: no issues**.

### 4) Comment consistency

In changed and related areas:

- Comments must match current behavior.
- Rules: do not over-comment; add comments only where reasoning needs extra context; comments reference **only** the codebase (no session or external context).

### 5) README and agent instruction accuracy

- If a **README** exists: check that it still matches the code; **do not** expand scope—only fix inaccuracies.
- If **AGENTS.md** or **CLAUDE.md** exists (repo root, subprojects, or relevant user config): verify structure, conventions, and tooling still match.
- If updates are needed: show **proposed edits or diff** clearly.
- **Ask user confirmation** before applying any documentation change.

### 6) Clarification gate (before final report)

- If open questions **materially** affect conclusions, ask them **first**.
- **One clarification question at a time**; wait for the answer before the next.
- Do **not** issue the final report until answered, unless there are **no** open questions.
- Each question: keep it concise; state **what could change** based on the answer.

## Output format

Deliver the final report including:

1. **Reviewed files list** (complete: changed + related unchanged from step 2).
2. **Risk / findings summary** (high level).
3. **Confirmed issues** with **severity** (bug-hunt).
4. **Candidate database indexes** (if any), with confidence and rationale; exclusions/conflicts per `database-index` output.
5. **Performance and code quality findings** (if any), with severity.
6. **Clarifying questions asked** (if any) and **user answers**.
7. **README update needed**: yes/no + confirmation request before edits.
8. **AGENTS.md / CLAUDE.md update needed**: yes/no + confirmation request before edits.

## Guardrails

- Do not fabricate findings when the diff and context show no problems.
- Do not skip the clarification gate when business logic is ambiguous.
- Do not edit migrations or docs without explicit user confirmation where this skill requires it.
