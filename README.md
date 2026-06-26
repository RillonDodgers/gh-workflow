# GH Workflow

`gh-workflow` is a Codex plugin that adds an issue-first GitHub workflow powered by `gh`.

It is meant for repos where work should follow this loop:

1. Create or reuse issue
2. Branch from `main`
3. Make change
4. Verify
5. Commit, push, open PR
6. Review diff
7. Squash merge and delete branch

## What This Repo Ships

- Codex plugin manifest at [`plugins/gh-workflow/.codex-plugin/plugin.json`](plugins/gh-workflow/.codex-plugin/plugin.json)
- Skill instructions at [`plugins/gh-workflow/skills/gh-workflow/SKILL.md`](plugins/gh-workflow/skills/gh-workflow/SKILL.md)
- Agent UI metadata at [`agents/openai.yaml`](agents/openai.yaml)
- Local marketplace entry at [`.agents/plugins/marketplace.json`](.agents/plugins/marketplace.json)

## Plugin Behavior

The skill is built around a few repo rules:

- `main` is only base branch
- protect `main` during repo init when GitHub admin access allows it
- no tracked-file edits on `main`; create or reuse issue, then branch first
- prefer `gh` for GitHub actions
- update local `main` from `origin/main` before branching
- create issue before work when needed
- use Conventional Commit style branch names like `docs/3` or `feat/12`
- squash merge pull requests
- delete merged branches

## Install In Codex

If you want this repo available as a local plugin, point Codex at the plugin directory:

- plugin root: `plugins/gh-workflow`
- plugin manifest: `plugins/gh-workflow/.codex-plugin/plugin.json`

This repo already includes local marketplace metadata for development and testing under `.agents/plugins/marketplace.json`.

## Use

Example prompts:

- `Use $gh-workflow to initialize this repo.`
- `Use $gh-workflow to initialize this repo and protect main.`
- `Use $gh-workflow to create an issue and branch.`
- `Use $gh-workflow to stage, push, review, and merge.`

Example request:

```text
$gh-workflow create an issue for adding release docs, implement it, and ship it
```

## Repo Layout

```text
.
├── SKILL.md
├── README.md
├── agents/
│   └── openai.yaml
└── plugins/
    └── gh-workflow/
        ├── .codex-plugin/
        │   └── plugin.json
        └── skills/
            └── gh-workflow/
                ├── agents/
                │   └── openai.yaml
                └── SKILL.md
```

## Development

When changing workflow behavior, update both:

- plugin skill at `plugins/gh-workflow/skills/gh-workflow/SKILL.md`
- top-level `README.md` if user-facing behavior changes

Repo init now includes branch protection on `main` when `gh` auth can administer repo. Fallback: report exact failing command plus manual GitHub path under branch settings.

When `gh-workflow` is active in repo with `origin`, tracked-file edits must wait until issue exists and work branch has been created. Read-only inspection on `main` is still allowed.

## License

MIT
