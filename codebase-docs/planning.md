# Codebase-Wide Documentation Planning

Read this file when the user requests documentation for the entire codebase. This workflow produces an ordered plan of documentation units, then executes each one using the single-file workflow from SKILL.md Step 3.

---

## Phase 1 — Scan the codebase

List all source files in the project. Exclude:
- `node_modules/`, `dist/`, `build/`, `.next/`, `.git/`, `coverage/`
- Generated files (lockfiles, `.d.ts` unless hand-written)
- Static assets (images, fonts, favicons)
- Test files (unless the user explicitly asks to document tests)

Use `Glob` or directory listing. Do not read file contents yet — only collect paths.

---

## Phase 2 — Build a dependency graph

For each source file, scan its import/require statements to build an adjacency list:

```
file → [imported files]
```

Keep it lightweight:
- Resolve relative imports to absolute paths
- Ignore external packages (npm modules)
- Track both directions: `imports` (what this file depends on) and `importedBy` (what depends on this file)

You do not need to parse ASTs. Scanning import lines with grep/regex is sufficient:
- `import ... from './...'` / `import ... from '../...'`
- `require('./...')` / `require('../...')`
- `export ... from './...'`

---

## Phase 3 — Apply boundary heuristics

Group files into **documentation units** using these rules (in priority order):

### Rule 1: Module-level docs

Any directory containing 3 or more related source files becomes a documentation unit.

- The doc is named after the directory: `src/lib/auth/` → `auth.md`
- All files in the directory are included in the unit
- Nested subdirectories are included in the parent module's doc unless they qualify as their own unit (3+ files with distinct purpose)

### Rule 2: Shared utility docs

Any single file imported by 3 or more modules (distinct top-level directories) gets its own documentation unit.

- Example: `src/utils/validation.ts` imported by `src/lib/auth/`, `src/lib/forms/`, and `src/lib/api/` → `validation-utils.md`
- The threshold of 3 prevents over-splitting. Files imported by only 1-2 modules are covered in those modules' docs instead.

### Rule 3: Leaf files

Files that:
- Exist within a module directory AND
- Are only imported within that same module

These are **not** standalone documentation units. They appear in their parent module's doc as Related Files with role type `primary`.

### Rule 4: Route/page docs

For frameworks with file-system routing (Next.js, SvelteKit, Nuxt, Remix):
- Each route group with server actions, loaders, or complex logic gets its own doc
- Simple pages that only render components are covered in their parent route group or module doc
- Name the doc after the route: `app/(dashboard)/` → `dashboard-routes.md`

### Rule 5: Configuration docs

Config files (`next.config.ts`, `tailwind.config.ts`, `tsconfig.json`, `.env.example`, etc.) are grouped into a single `configuration.md` unless one config file is complex enough to warrant its own doc (50+ lines of non-trivial logic).

### Rule 6: Entry points

Top-level entry files (`app/layout.tsx`, `src/index.ts`, `main.ts`) get their own doc if they contain significant setup logic (providers, middleware chains, initialization). Otherwise they are covered in the most relevant module doc.

### Handling ambiguity

If a file could belong to multiple units, assign it to the unit whose directory it lives in. It will appear as a secondary reference in the other unit's doc. See [overlap-resolution.md](overlap-resolution.md) for the full assignment rules.

---

## Phase 4 — Diff against INDEX.md

Compare the planned documentation units against the existing INDEX.md:

| Situation | Action |
|-----------|--------|
| Unit exists in plan and INDEX.md, sources unchanged | **Skip** — no work needed |
| Unit exists in plan and INDEX.md, sources changed | **Update** — refresh the existing doc |
| Unit exists in plan but not in INDEX.md | **Create** — new documentation unit |
| Unit exists in INDEX.md but not in plan | **Orphan** — flag for user review (source files may have been deleted or restructured) |

This diff is the key to incremental documentation. On a partially-documented codebase, only new or changed units produce work.

---

## Phase 5 — Generate the work plan

Order the units for execution:

1. **Shared utilities first** — they have no upstream dependencies and will be referenced by everything else
2. **Core modules next** — in dependency order (if module A imports from module B, document B first)
3. **Feature modules** — modules that depend on core modules
4. **Routes/pages last** — they import from everything and reference the most docs
5. **Configuration** — can be done at any point, typically last

Present the plan to the user as a numbered checklist:

```
Documentation plan (12 units):
1. [ ] validation-utils — src/utils/validation.ts (shared utility, 1 file)
2. [ ] database — src/lib/db/ (core module, 4 files)
3. [ ] auth — src/lib/auth/ (core module, 6 files)
4. [ ] billing — src/lib/billing/ (feature module, 3 files)
5. [ ] dashboard-routes — app/(dashboard)/ (routes, 5 files)
...
```

Cap each unit at **~5-8 source files**. If a module has more, split it into sub-units by subdirectory or functional area.

Ask the user to confirm before executing, or if they want to adjust boundaries.

---

## Phase 6 — Execute the plan

### Strategy A: Subagents (preferred when Task tool is available)

For each documentation unit, dispatch a subagent with:
- The single-file workflow instructions (SKILL.md Steps 3, 6, 7)
- The specific file list for this unit
- The current INDEX.md content (so it knows what exists)
- The document template
- Instructions to return: the doc path created/updated, the files stamped, and the INDEX.md entries to add

After each subagent completes, merge its INDEX.md entries into the master INDEX.md before dispatching the next one. This ensures later subagents see an up-to-date manifest.

Subagents can run in parallel **only** when their units have no dependency relationship (neither references the other's files). Otherwise, execute sequentially.

### Strategy B: Sequential (when Task tool is not available)

Execute units one at a time using the single-file workflow:

1. Process unit N using SKILL.md Step 3
2. Update INDEX.md
3. Re-read INDEX.md (to see the updated state)
4. Process unit N+1
5. Repeat

Use the TodoWrite tool to track progress across units. If the context window is getting full, tell the user which units remain and suggest continuing in a new conversation.

### Context efficiency during execution

- **Do not re-read INDEX.md** between units unless a unit created or modified it
- **Do not re-read source files** that were already read for dependency graphing — reuse the information gathered in Phase 2
- **Summarize, don't quote** — when passing context to subagents, summarize the dependency graph rather than including raw file contents
- If the codebase has 50+ files, present the plan and ask the user whether to proceed with all units or select a subset

---

## Phase 7 — Final verification

After all units are processed:

1. Read the final INDEX.md
2. Check for orphaned entries (source files that no longer exist)
3. Check for missing cross-references (doc A references file F, file F's primary doc is doc B, but doc A doesn't link to doc B in its References section)
4. Report to the user: "Documented X units covering Y files. INDEX.md updated."
