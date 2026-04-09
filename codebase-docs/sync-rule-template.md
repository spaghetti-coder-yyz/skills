---
alwaysApply: true
---

# Documentation Sync

> Installed by the codebase-docs skill. Keeps `/docs` files and `/docs/INDEX.md` current when source files change.

## Rule: Keep `/docs` in sync with source files (REQUIRED)

If a file you are editing contains one or more `@docs` tags in its JSDoc — for example:

```ts
/**
 * @docs /docs/auth.md, /docs/middleware.md
 */
```

You MUST do **all** of the following as part of the same task — not as a follow-up:

1. **Update the JSDoc** in the file you just edited. Refresh any tags affected by your changes — `@description`, `@behavior`, `@depends-on`, `@depended-by`, `@example`. Keep `@docs` tags intact.

2. **For EACH doc path** listed in the `@docs` tag (they are comma-separated):
   a. Read that `/docs` file.
   b. Update any sections affected by your changes — diagrams, usage examples, API reference, Related Files table, Overview.
   c. Do not rewrite sections that are unaffected.

3. **Update `/docs/INDEX.md`**: set the Last Updated date to today for each doc you modified.
