# Overlap Resolution

Read this file when a source file appears in multiple documentation units and you need to decide which doc is the primary and how to cross-reference.

---

## Core rule

Every source file has exactly **one** primary doc and zero or more secondary references.

- **Primary doc**: covers the file in depth — architecture, API reference, usage examples, behavior
- **Secondary reference**: mentions the file in a Related Files table with role type `reference`, may include it in a Mermaid diagram, but does not duplicate deep coverage

---

## Assigning primary

Use these rules in order. The first matching rule wins.

### 1. Directory ownership

If the file lives inside a directory that has its own documentation unit, that unit's doc is the primary.

Example: `src/lib/auth/session.ts` → primary doc is `auth.md` (the doc for `src/lib/auth/`).

### 2. Dedicated utility doc

If the file is a shared utility with its own standalone doc (because it's imported by 3+ modules), that dedicated doc is the primary.

Example: `src/utils/validation.ts` has its own `validation-utils.md` → that is the primary, even though `auth.md` and `forms.md` also reference it.

### 3. Fewer files wins

If two docs could equally claim a file as primary, prefer the doc that covers fewer files. A more focused doc provides better depth.

### 4. Ask the user

If the above rules don't resolve the ambiguity, ask the user which doc should be the primary for this file.

---

## What goes in a primary doc vs a secondary reference

| Content | Primary doc | Secondary reference |
|---------|-------------|---------------------|
| File appears in Related Files table | Yes, with role type `primary` | Yes, with role type `reference` |
| File's API surface documented | Yes — signatures, params, return types | No — link to the primary doc instead |
| File's behavior explained | Yes — step-by-step logic | Brief mention only if relevant to this doc's flow |
| File appears in Mermaid diagram | Yes — as a detailed node | Yes — as a simplified node linking to primary doc |
| File's usage examples shown | Yes | No — reference the primary doc for examples |

---

## Cross-reference procedure

When doc A includes file F as a secondary reference, and file F's primary doc is doc B:

1. In doc A's **Related Files** table, include file F with role type `reference`
2. In doc A's **References** section, add a link: `- [doc-b.md](doc-b.md) — primary documentation for \`path/to/F\``
3. In doc B's **References** section, add a back-link: `- [doc-a.md](doc-a.md) — references \`path/to/F\``
4. In INDEX.md's File Map, add doc A to file F's Also Referenced In column

### When creating a new doc

After assigning all files to primary/secondary roles:

1. Check INDEX.md for any existing docs that already reference files in this unit
2. For each such doc, add a link in the new doc's References section
3. Update those existing docs' References sections to link back to the new doc

### When updating an existing doc

If a file is added to or removed from the Related Files table:

- **Added file**: look up its primary doc in INDEX.md, add cross-references as above
- **Removed file**: remove this doc from the file's Also Referenced In in INDEX.md, remove the back-link from the other doc's References section

---

## Avoiding duplication

The overlap resolution system prevents duplicated documentation:

- Before writing API reference for a file, check: is this file's role type `primary` in this doc? If `reference`, do not write API reference — link to the primary doc instead.
- Before writing usage examples for a file, same check. Only the primary doc includes examples.
- The Related Files table explicitly marks role types so the agent (and future agents) can tell at a glance whether deep coverage belongs here or elsewhere.

---

## Example scenario

Codebase structure:
```
src/lib/auth/login.ts      → imports validation.ts, db-client.ts
src/lib/auth/session.ts     → imports db-client.ts
src/lib/db/client.ts        → standalone
src/utils/validation.ts     → imported by auth, forms, api modules
```

Documentation units and assignments:

| File | Primary Doc | Secondary In | Reason |
|------|-------------|--------------|--------|
| `src/lib/auth/login.ts` | auth.md | - | Directory ownership (Rule 1) |
| `src/lib/auth/session.ts` | auth.md | - | Directory ownership (Rule 1) |
| `src/lib/db/client.ts` | database.md | auth.md | Directory ownership (Rule 1) |
| `src/utils/validation.ts` | validation-utils.md | auth.md, forms.md, api.md | Dedicated utility doc (Rule 2) |

In `auth.md`:
- `login.ts` and `session.ts` appear with role type `primary` (full API docs, examples)
- `client.ts` appears with role type `reference` (mentioned in diagram, no API docs)
- `validation.ts` appears with role type `reference`
- References section links to `database.md` and `validation-utils.md`
