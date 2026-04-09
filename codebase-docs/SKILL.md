---
name: codebase-docs
description: >-
  Creates and maintains structured technical documentation across an entire codebase.
  Supports single-file, codebase-wide, and refresh modes with an INDEX.md manifest
  that tracks what is documented where, prevents duplication, and resolves overlapping
  references. Use when the user asks to document a file, module, feature, the whole
  codebase, or refresh/update existing documentation.
---

# Codebase Docs

A documentation system that creates technical docs in `/docs`, maintains an `INDEX.md` manifest for cross-referencing, stamps source files with `@docs` back-references, and keeps everything in sync as the codebase evolves.

---

## Step 0 — Bootstrap the sync rule (once per project)

Check if `.cursor/rules/documentation-sync.mdc` exists in the project root.

If it does **not** exist, read [sync-rule-template.md](sync-rule-template.md) and write its contents to `.cursor/rules/documentation-sync.mdc`. Tell the user: "Installed documentation-sync rule for this project."

If it already exists, skip silently.

---

## Step 1 — Detect mode

Parse the user's intent to determine which mode to run:

| User says | Mode |
|-----------|------|
| "document this file" / "document X" / "create docs for component Y" | **Single-file** (Step 3) |
| "document the codebase" / "document everything" / "generate all docs" | **Codebase-wide** (Step 4) |
| "update docs" / "refresh documentation" / "sync docs" | **Refresh** (Step 5) |

If ambiguous, ask the user.

---

## Step 2 — Read or create INDEX.md

All modes begin here. Read `/docs/INDEX.md`.

If it does not exist, create a skeleton:

```md
# Documentation Index

> Auto-managed by codebase-docs. Manual edits will be overwritten.

## Documents

| Document | Status | Covers | Last Updated |
|----------|--------|--------|--------------|

## File Map

| Source File | Primary Doc | Also Referenced In |
|-------------|-------------|--------------------|
```

For the full INDEX.md schema and operations, see [index-format.md](index-format.md).

---

## Step 3 — Single-file workflow

This is the default mode and handles the most common case.

### 3a. Read the target file

Read the file the user wants to document. Understand its purpose, exports, key functions, and behavior.

### 3b. Trace imports and consumers (1 level deep)

Identify:
- **Dependencies**: files this file imports
- **Consumers**: files that import this file

Do not recurse beyond 1 level. These become the Related Files for the doc.

### 3c. Check INDEX.md for existing coverage

Look up the target file and all traced files in INDEX.md:

1. **Target file already has a primary doc** → go to Step 3d (update)
2. **Target file appears as secondary in another doc** → decide: is it significant enough for its own doc, or should it be folded into the existing module doc?
3. **Target file has no coverage** → check if it belongs to a directory that already has a module-level doc. If yes, add it to that doc. If no, create a new doc.

When overlap is detected between documentation units, read [overlap-resolution.md](overlap-resolution.md) for assignment rules.

### 3d. Create or update the document

**If creating:** use the document template (Step 6) to write `/docs/<topic-slug>.md`.

**If updating:** read the existing doc, then update only sections affected by changes:
- Refresh the Summary/Overview if the file's purpose changed
- Update the Mermaid diagram if dependencies or flow changed
- Update the API Reference if function signatures changed
- Update the Related Files table if imports/consumers changed
- Update the References section if new cross-doc links are needed

Do not rewrite sections that are unaffected.

### 3e. Stamp source files with @docs tags

For every file in the document's **Related Files** table, apply the JSDoc stamping procedure (Step 7).

### 3f. Update INDEX.md

Add or update entries in both the **Documents** table and the **File Map** table. See [index-format.md](index-format.md) for the exact operations.

---

## Step 4 — Codebase-wide mode

Read [planning.md](planning.md) for the full scan/plan/execute workflow. That file covers:

- Scanning the codebase file tree
- Building a dependency graph from imports
- Applying boundary heuristics to group files into documentation units
- Diffing the plan against INDEX.md to skip already-documented units
- Generating an ordered work plan
- Executing via subagents (if Task tool is available) or sequentially

Each documentation unit is processed using the single-file workflow (Step 3) with the unit's primary file as the target.

---

## Step 5 — Refresh mode

Use this mode when documentation exists but may be stale.

1. Read `/docs/INDEX.md`
2. For each entry in the Documents table, inspect the source files listed in the File Map:
   - Use `git diff` against the doc's Last Updated date if git is available
   - Otherwise, read each source file and compare its current state to what the doc describes
3. **Skip unchanged files** — do not re-read or rewrite docs for files that haven't changed
4. For changed files:
   a. Read the linked doc
   b. Update only affected sections (same logic as Step 3d)
   c. Update `@docs` JSDoc tags if dependencies changed
5. Update INDEX.md timestamps for all refreshed docs
6. Flag orphaned entries — if a source file in the File Map no longer exists, mark its row with status `Orphaned` for user review

---

## Step 6 — Document template

Use this structure for every doc in `/docs`:

```md
# <Title: clear, descriptive>

**Date:** YYYY-MM-DD
**Author:** Coding Assistant
**Status:** Draft | Active | Deprecated

---

## Summary

One or two sentences describing what this document covers and who it is for.

---

## Overview

Purpose, context, and scope of the feature, API, or system.

---

## Architecture / Flow

[Mermaid diagram — see diagram rules below]

---

## Key Concepts

Bullet-point list of important terms, types, or patterns.

---

## Usage Examples

\`\`\`typescript
// Concrete example
\`\`\`

---

## API Reference

Function signatures, parameters, return types, and side effects (omit if not applicable).

---

## Related Files

| File | Role | Role Type |
|------|------|-----------|
| `path/to/file.ts` | What this file does in this context | primary |
| `path/to/dep.ts` | Dependency used by the primary file | reference |

---

## References

Links to related `/docs/*.md` files or external documentation.
- [other-feature.md](other-feature.md) — shares `path/to/dep.ts`
```

**Role Type** values:
- `primary` — this doc covers the file in depth (architecture, API, examples)
- `reference` — the file is mentioned here but documented in depth elsewhere

### Mermaid diagram rules

Include a diagram whenever there is a flow, relationship, or state to describe.

| Diagram | Use when |
|---------|----------|
| `flowchart TD` | Data flow or processing steps |
| `sequenceDiagram` | Request/response or service-to-service calls |
| `erDiagram` | Database table relationships |
| `stateDiagram-v2` | Status transitions or state machines |

Syntax rules:
- Node labels containing `[]`, `()`, `{}`, `/`, or `"` MUST be wrapped in double-quotes
- Edge labels containing special characters MUST be quoted
- Diamond nodes `{label}` must NOT contain parentheses inside braces
- No backticks, colons, or angle brackets inside labels
- Prefer plain descriptive English over code expressions

---

## Step 7 — JSDoc stamping

For every file in the document's **Related Files** table:

1. **Read the full file** to understand its purpose, behavior, dependencies, and consumers.
2. **Write or update the file-level JSDoc block** at the top. The JSDoc must describe the file broadly — not just in the context of this document. Include:
   - `@description` — what the file does and its role in the system
   - `@behavior` — key steps, logic, or side effects (omit if trivial)
   - `@depends-on` — other files or services this file relies on
   - `@depended-by` — files that import or call into this file
   - `@example` — usage example (for hooks, actions, utilities, reusable modules)
   - `@docs` — comma-separated paths to linked docs, primary first

3. If the file already has a JSDoc:
   - **Preserve** existing tags that are still accurate
   - **Update** tags affected by changes
   - **Merge** `@docs` paths — do not replace. Add new paths, keep existing ones

**JSDoc template:**

```ts
/**
 * @description <What this file does and its role in the system.>
 *
 * @behavior
 * - <Key step or behaviour 1>
 * - <Key step or behaviour 2>
 *
 * @depends-on <path/to/dependency.ts>, <ExternalService>
 * @depended-by <path/to/consumer.ts>
 *
 * @example
 * import { something } from './this-file'
 * something()
 *
 * @docs /docs/primary-doc.md, /docs/secondary-doc.md
 */
```

Omit tags that are not applicable. Confirm to the user which files were stamped.

---

## Step 8 — Verification checklist

After completing any mode, verify:

- [ ] Doc saved to `/docs/<filename>.md`
- [ ] Doc has title, date, status, and summary
- [ ] Mermaid diagram included (if warranted)
- [ ] Mermaid syntax self-reviewed: no `()` inside `{}` nodes, no unquoted special chars
- [ ] Usage examples included
- [ ] Related Files table complete with Role Type column
- [ ] Every related source file has `@docs` tag(s) in its JSDoc
- [ ] `/docs/INDEX.md` updated — Documents table and File Map table
- [ ] No orphaned entries in INDEX.md (source files that no longer exist)
- [ ] Cross-references consistent: if doc A references doc B, doc B links back in its References section
- [ ] `documentation-sync.mdc` rule exists in `.cursor/rules/`
