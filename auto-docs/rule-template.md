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

You MUST do all of the following as part of the same task — not as a follow-up:

1. **Update the JSDoc in the file you just edited.** Refresh any tags affected by your changes — `@description`, `@behavior`, `@depends-on`, `@depended-by`, `@example`. Keep the `@docs` tag intact.
2. **Read the linked `/docs` file.**
3. **Update any sections of the doc affected by your changes** — diagrams, usage examples, API reference, Related Files table, Overview.
