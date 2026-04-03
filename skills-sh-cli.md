# skills.sh — CLI cheat sheet

Quick reference for sharing skills on [skills.sh](https://skills.sh) and keeping them current. The ecosystem is powered by the open-source [`skills` CLI](https://github.com/vercel-labs/skills) ([CLI docs](https://skills.sh/docs/cli)).

## How “uploading” works

There is **no separate upload command**. A skill appears in the directory when people install it from a **public Git repository** using `npx skills add …`. The [leaderboard](https://skills.sh/) ranks skills using **anonymous install telemetry** from that CLI (you can opt out with `DISABLE_TELEMETRY=1`; see [docs](https://skills.sh/docs)).

**Workflow:**

1. Put one or more skills in a GitHub (or GitLab / other git) repo. Each skill is a folder with a valid `SKILL.md` (YAML frontmatter with `name` and `description`). The CLI discovers skills under paths like `skills/`, repo root, `.cursor/skills/`, etc.—see [Creating Skills](https://github.com/vercel-labs/skills#creating-skills) in the upstream README.
2. **Push** changes to the default branch (or tag releases as you prefer).
3. **Share** the install line so others (and you on new machines) can add the skill:

   ```bash
   npx skills add YOUR_GITHUB_USER/YOUR_REPO
   ```

   Other supported sources include full GitHub URLs, paths to a subtree, GitLab, and local paths—see [Source Formats](https://github.com/vercel-labs/skills#source-formats).

4. **Updates:** After you push new commits, anyone who already installed from that repo can refresh with:

   ```bash
   npx skills check
   npx skills update
   ```

   Re-running `npx skills add YOUR_USER/YOUR_REPO` also pulls the latest when installing into a project.

## Commands worth memorizing

| Command | Purpose |
|--------|---------|
| `npx skills add <owner/repo>` | Install skill(s) from a repo (interactive: pick agents, symlink vs copy). |
| `npx skills add <owner/repo> --list` | List skills in the repo **without** installing. |
| `npx skills add <owner/repo> --skill <name> -y` | Install a specific skill non-interactively. |
| `npx skills add <url>` | Install from full GitHub/GitLab URL or path to a folder in the tree. |
| `npx skills list` (alias: `ls`) | List installed skills (`-g` for global only). |
| `npx skills find [query]` | Search skills (interactive or keyword). |
| `npx skills check` | See if installed skills have updates. |
| `npx skills update` | Update all installed skills to latest. |
| `npx skills init [name]` | Scaffold a new `SKILL.md` in the current dir or a subfolder. |
| `npx skills remove <name>` | Remove installed skills from agents (see `npx skills remove --help`). |

**Scope:** Default is **project** install; use `-g` / `--global` for user-wide installs.

**Telemetry:** `DISABLE_TELEMETRY=1` or `DO_NOT_TRACK=1` disables anonymous CLI telemetry ([docs](https://github.com/vercel-labs/skills#environment-variables)).

## Related links

- [skills.sh](https://skills.sh) — browse / leaderboard  
- [Documentation](https://skills.sh/docs)  
- [CLI reference](https://skills.sh/docs/cli)  
- [Agent Skills spec](https://agentskills.io)
