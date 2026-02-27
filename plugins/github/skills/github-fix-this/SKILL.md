---
name: github-fix-this
description: 'End-to-end GitHub fix workflow: given an issue description or issue number, create a fix branch from the current branch, implement the fix, push the branch, open a pull request, and report the issue and PR URLs. Use when users say "fix this", "fix issue #N", "fix this bug", or provide an issue description to resolve.'
---

# GitHub Fix This

Automate the full workflow from issue to pull request: create (or use) a GitHub issue, branch, implement the fix, push, and open a PR.

## Prerequisites

- `gh` CLI authenticated (`gh auth status`)
- Git repository with a remote configured

## Workflow

### Step 1 — Establish the base branch

```bash
BASE_BRANCH=$(git branch --show-current)
```

Use `$BASE_BRANCH` as the PR target throughout.

### Step 2 — Obtain or create the GitHub issue

**If the user supplies an issue number** (e.g. `#42` or `42`):

```bash
gh issue view <NUMBER> --json number,title,body,url
```

Store `number` and `title` for the next steps.

**If the user supplies an issue description** (no number yet):

```bash
gh issue create \
  --title "<concise title>" \
  --body "<detailed description>" \
  --label "bug"          # or "enhancement" / "task" as appropriate
```

Capture the printed issue URL, derive `number` from it (`${url##*/}`).

### Step 3 — Create a fix branch

Branch naming: `fix/issue-<NUMBER>-<slug>` where `<slug>` is a short kebab-case summary of the title.

```bash
git checkout -b fix/issue-<NUMBER>-<slug>
```

### Step 4 — Implement the fix

Analyze the issue description and repository code to determine the required changes, then edit the relevant files. Apply only changes necessary to resolve the issue.

### Step 5 — Stage changed files

Stage only the files you modified:

```bash
git add <file1> <file2> ...
```

Do **not** stage unrelated files. Use `git diff --name-only` to verify which files were changed.

### Step 6 — Commit

```bash
git commit -m "fix: <concise description of fix> (closes #<NUMBER>)"
```

### Step 7 — Push the branch

```bash
git push -u origin fix/issue-<NUMBER>-<slug>
```

### Step 8 — Open a pull request

```bash
gh pr create \
  --base "$BASE_BRANCH" \
  --title "fix: <title> (closes #<NUMBER>)" \
  --body "## Summary

Closes #<NUMBER>

## Changes

<bullet list of changes made>

## Testing

<brief description of how to verify the fix>"
```

Capture the PR URL from the output.

### Step 9 — Report URLs

Present both URLs clearly:

```
Issue : <issue_url>
Pull Request: <pr_url>
```

## Notes

- If `gh issue create` or `gh pr create` fails due to missing remote or auth, surface the error and guide the user to run `gh auth login`.
- If the fix spans multiple logical changes, use a single commit per logical change (repeat Steps 4–6) before pushing.
- Never push directly to the base branch.
