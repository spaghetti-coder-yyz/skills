---
alwaysApply: true
---

# Documentation Sync

> Installed by the auto-docs skill. Enforces that `/docs` files stay current when their linked source files are edited.

## Rule: Keep `/docs` in sync with source files (REQUIRED)

If a file you are editing contains a `@docs` tag in its JSDoc — for example:

```ts
/**
 * Does something useful.
 *
 * @docs /docs/some-feature.md
 */
```

You MUST:

1. Read the linked documentation file before finishing your task.
2. Update any sections affected by your changes — diagrams, usage examples, API reference, Related Files table.
3. Do this as part of the same task, not as a follow-up.
