---
name: gh-workflow
description: Issue-first GitHub workflow using the gh CLI. Use when the user invokes /gh-workflow:init-repo, /gh-workflow:create-issue, /gh-workflow:workflow, or /gh-workflow:ship; asks to initialize a new GitHub repo or Rails project upstream, create GitHub issues, branch from issues, stage, commit, push, review, squash merge, delete branches, or follow a Conventional Commits GitHub flow.
---

# GH Workflow

Use this skill to run the user's preferred GitHub workflow with `gh`, Git, issues, branches, pull requests, and squash merges.

## Global Rules

- Treat `main` as the only head/base branch.
- Do not set up, use, or recommend GitHub wiki.
- Do not set up, use, or recommend Dependabot workflows.
- Always make local `main` current with `origin/main` before creating a work branch.
- Always squash merge pull requests.
- Always delete the branch after merge.
- Use Conventional Commit type prefixes for branches: `feat/<issue>`, `fix/<issue>`, `chore/<issue>`, `docs/<issue>`, `refactor/<issue>`, `test/<issue>`, or `ci/<issue>`.
- Use one-line Conventional Commit messages, such as `feat: add export button`.
- Prefer `gh` for GitHub state and actions.
- Read repo-local instructions first when present, but do not violate these global rules unless the user explicitly overrides them.
- Preserve user work. Inspect dirty worktrees before changing branches, staging, or committing.

## Common Checks

Run these before choosing workflow steps:

```bash
git status --short --branch
git remote -v
gh auth status
gh repo view --json nameWithOwner,defaultBranchRef
```

When issue assignment matters, get the authenticated username:

```bash
gh api user --jq .login
```

## `/gh-workflow:init-repo`

Use this when initializing or auditing a repo for the user's preferred GitHub project workflow.

1. Inspect current branch, remotes, default branch, and GitHub repo settings available through `gh`.
2. If the repo has no `origin` remote, treat it like a new Rails project or local-only Git repo:
   - Ask whether an upstream GitHub repo already exists.
   - If an upstream exists, add it as `origin`, fetch, and set local `main` to track `origin/main`.
   - If no upstream exists, create one with `gh repo create`. Use the current project folder name as the default repo name and create it as private unless the user says otherwise.
   - Ask only for missing repo creation details that `gh` needs, such as owner.
3. Ensure the repo uses `main` as the default branch or report the exact command/manual action needed to change it.
4. Verify local `main` tracks `origin/main` when a remote exists.
5. Disable wiki through `gh` if supported for the repo and auth scope; otherwise report the manual GitHub settings path.
6. Do not add Dependabot files or workflows. If existing Dependabot config is present, report it and ask before removing it.
7. Confirm PR merge settings where possible. Prefer squash merge only and branch deletion after merge; report UI-only settings when `gh` cannot change them.

Default no-upstream create flow:

```bash
repo_name=$(basename "$PWD")
git branch -M main
gh repo create "$repo_name" --private --source=. --remote=origin --push
```

If `gh repo create` prompts for owner and the user did not specify one, ask the user instead of guessing. Do not prompt for visibility unless the user asks; private is the default.

## `/gh-workflow:create-issue`

Use this when the user asks to create an issue or needs an issue before work begins.

1. Search open issues before creating a new one:

```bash
gh issue list --state open --search "<keywords>"
```

2. If the user supplied an issue number or says an issue already exists, use that issue. Do not create a duplicate.
3. Create a new issue only when no suitable issue exists.
4. Use the Three Cs format:

```markdown
## Card
<one or two sentences describing the work>

## Conversation
<context, constraints, decisions, and relevant links>

## Confirmation
- [ ] <observable acceptance criterion>
- [ ] <verification or review criterion>
```

5. Assign the issue to the authenticated `gh` user by default:

```bash
assignee=$(gh api user --jq .login)
gh issue create --title "<title>" --body-file <body-file> --assignee "$assignee"
```

Use `--body-file` for markdown bodies to avoid shell quoting bugs.

## `/gh-workflow:workflow`

Use this when the user wants the full issue-first work loop: issue, branch, implementation, verification, commit, push, and PR.

1. Inspect status and local instructions.
2. Update `main` before branching:

```bash
git fetch origin
git switch main
git pull --ff-only origin main
```

3. Use the supplied issue, or run `/gh-workflow:create-issue`.
4. Create the branch from current `main`:

```bash
git switch -c <type>/<issue-number>
```

5. Do the requested work.
6. Run the smallest verification that proves the change.
7. Stage intended files only after reviewing `git status --short`.
8. Commit with one-line Conventional Commit style:

```bash
git commit -m "<type>: <summary>"
```

9. Push and create a PR:

```bash
git push -u origin HEAD
gh pr create --base main --head "$(git branch --show-current)" --title "<title>" --body-file <body-file>
```

Include `Closes #<issue-number>` in the PR body when the PR should close the issue.

## `/gh-workflow:ship`

Use this when the user says `stage, commit, push, review, merge`, `stage, push, review, merge`, `ship this`, or asks to finish existing branch work.

1. Inspect branch and status:

```bash
git status --short --branch
git branch --show-current
```

2. Refuse to commit directly to `main` unless the user explicitly asks for a direct-main change.
3. Stage intended changes, including untracked files that belong to the task:

```bash
git add <paths>
git diff --cached --check
```

4. Commit with one-line Conventional Commit style if staged changes exist.
5. Push branch:

```bash
git push -u origin HEAD
```

6. Create or update the PR. Check for an existing PR first:

```bash
gh pr status
```

7. Review the actual diff before merging:

```bash
gh pr diff <pr-number>
```

8. Squash merge and delete the branch:

```bash
gh pr merge <pr-number> --squash --delete-branch
```

9. Return local repo to updated `main` when useful:

```bash
git switch main
git pull --ff-only origin main
```

## Failure Handling

- If `gh` lacks auth or scope, report the exact failing command and the minimal `gh auth refresh` or login command.
- If GitHub settings are UI-only for current auth, report the manual setting name and desired value.
- If a dirty worktree blocks branch switching, stop and explain which files are blocking.
- If remote/network calls fail, retry once before treating remote state as unavailable.
- If PR merge succeeds but metadata refresh fails, trust the merge command result first and retry metadata only if needed.
