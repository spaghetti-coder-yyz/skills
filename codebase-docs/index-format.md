# INDEX.md Format and Operations

This file defines the schema for `/docs/INDEX.md` and the procedures for reading, writing, and updating it. Read this file whenever you need to interact with the index.

---

## Schema

INDEX.md contains two markdown tables: **Documents** and **File Map**.

### Documents table

Tracks every doc file in `/docs` (excluding INDEX.md itself).

| Column | Type | Description |
|--------|------|-------------|
| Document | Link | Relative markdown link to the doc: `[name.md](name.md)` |
| Status | Enum | `Active`, `Draft`, `Deprecated`, or `Orphaned` |
| Covers | Code | Primary source path(s) this doc is about — directory or file |
| Last Updated | Date | `YYYY-MM-DD` of the most recent update |

### File Map table

Tracks every source file that has been documented, mapping it to its primary and secondary docs.

| Column | Type | Description |
|--------|------|-------------|
| Source File | Code | Path to the source file relative to project root |
| Primary Doc | Link | The doc that covers this file in depth |
| Also Referenced In | Links | Comma-separated links to docs that mention this file as a secondary reference. `-` if none |

---

## Skeleton

When creating INDEX.md for the first time:

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

---

## Operations

### Lookup

Given a source file path, find its entry in the File Map table.

- If found: return the Primary Doc and Also Referenced In values
- If not found: the file is undocumented

Use lookup before creating a new doc to avoid duplicating coverage.

### Add document

When a new doc is created:

1. Add a row to the **Documents** table with Status `Active` and today's date
2. For each file in the doc's Related Files table with role type `primary`:
   - If the file is not in the File Map: add a new row with this doc as Primary Doc
   - If the file is already in the File Map with a different Primary Doc: do not change the Primary Doc. Instead, add this doc to the Also Referenced In column
3. For each file with role type `reference`:
   - If the file is not in the File Map: add a new row with the file's own doc as Primary Doc (if known) and this doc in Also Referenced In
   - If the file is already in the File Map: add this doc to the Also Referenced In column

### Update document

When an existing doc is updated:

1. Update the **Last Updated** date in the Documents table
2. If the Related Files table changed (files added or removed):
   - Add new files to the File Map following the Add rules above
   - For removed files: remove this doc from their Also Referenced In column. If this doc was the Primary Doc, leave the Primary Doc column as-is and flag for review

### Remove document

When a doc is deleted (rare — usually manual):

1. Remove the row from the Documents table
2. For each file in the File Map that references this doc:
   - If this was the Primary Doc: set Primary Doc to the first entry in Also Referenced In (if any), or mark as `-`
   - Remove this doc from the Also Referenced In column

### Flag orphaned

When a source file is deleted from the codebase but still appears in the File Map:

1. Do not delete the File Map row immediately
2. Change the Primary Doc link text to include `(orphaned)`: `[auth.md](auth.md) (orphaned)`
3. In the Documents table, if all of a doc's covered files are orphaned, change Status to `Orphaned`
4. Report orphaned entries to the user for review — they may want to delete the doc or update it

### Merge check

Before creating a new doc, run this check:

1. Look up the target file in the File Map
2. If it already has a Primary Doc → do not create a new doc. Update the existing one instead.
3. If it appears only in Also Referenced In → it may warrant its own doc. Check whether it meets the shared utility threshold (imported by 3+ modules). If yes, create a new doc and reassign it as Primary. If no, add it to the existing module doc.
4. If the target file's directory already has a doc (check the Documents table Covers column) → add the file to that existing doc rather than creating a new one.

---

## Parsing notes

The agent parses INDEX.md as markdown tables. When reading:
- Split rows by `|`
- Trim whitespace from each cell
- Parse links with regex: `\[([^\]]+)\]\(([^)]+)\)`
- Parse comma-separated values in the Also Referenced In column

When writing:
- Align columns with consistent padding
- Sort the Documents table alphabetically by document name
- Sort the File Map table alphabetically by source file path
- Use `-` for empty Also Referenced In cells, not blank
