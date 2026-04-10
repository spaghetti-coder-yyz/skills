# Skills

This repository is a collection of **agent skills**—markdown instructions and templates I use with Cursor and other agentic development tools. I maintain them here because they have been genuinely helpful as I’ve shifted toward more AI-assisted, agent-powered workflows: clearer prompts, repeatable patterns, and less context-switching between “how I work” and what the model sees.

Each skill typically lives in its own folder with a `SKILL.md` (and sometimes supporting files). You can copy a folder into your own tooling setup, adapt the wording, or use this repo as a reference when authoring new skills.

## Skills

### codebase-docs (recommended)

A full documentation system that creates, maintains, and keeps docs in sync across your entire codebase. It tracks what's documented where via an `INDEX.md` manifest, resolves overlapping references between components, and only touches files that need updating.

**Install:**

```bash
npx skills add https://github.com/spaghetti-coder-yyz/skills/tree/main/codebase-docs
```

**Usage — three modes:**

| What you say | What happens |
|---|---|
| "Document `src/lib/auth/session.ts`" | **Single-file mode** — documents the file and its immediate imports/consumers, stamps `@docs` tags, updates `INDEX.md` |
| "Document the codebase" | **Codebase-wide mode** — scans the project, groups files into documentation units, presents a plan, then executes unit by unit |
| "Refresh documentation" | **Refresh mode** — checks which source files changed since the last update and only refreshes stale docs |

After the first run, a sync rule is installed so that whenever the agent edits a source file with `@docs` tags, the linked docs update automatically — no explicit request needed.

### auto-docs

A simpler, single-topic documentation generator. Creates one doc at a time in `/docs`, stamps source files with `@docs` back-references, and installs a sync rule. Use this if you only need to document an individual feature or component without codebase-wide orchestration.

**Install:**

```bash
npx skills add https://github.com/spaghetti-coder-yyz/skills/tree/main/auto-docs
```

**Usage:** Ask the agent to document a specific topic — "Document the webhook system", "Create docs for the auth flow". It will create `/docs/<topic>.md`, add Mermaid diagrams and usage examples, and stamp related source files with `@docs` tags.

> **Note:** `codebase-docs` is a superset of `auto-docs`. If you install `codebase-docs`, you get everything `auto-docs` does plus codebase-wide orchestration, the `INDEX.md` manifest, overlap resolution, and incremental refresh. The two are compatible — docs created by either skill use the same format.

---

*Personal collection; use and remix as you like.*

Enjoy! Reference my X account: [https://x.com/codewrangler_](https://x.com/codewrangler_)
