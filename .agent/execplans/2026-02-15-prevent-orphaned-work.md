# ExecPlan: Prevent Orphaned Work on Feature Branches

**Date:** 2026-02-15
**Status:** In Progress

## Problem

Commits are silently lost when agents push to feature branches after PR merge, mix unrelated work onto feature branches, or leave stale branches around.

## Root Causes

1. **Late commits after squash-merge.** When a PR is squash-merged, the feature branch commits are replaced by a single merge commit on the target branch. Any commits pushed to the feature branch after merge are orphaned — they exist in the reflog but are not reachable from any branch head once the branch is deleted.

2. **Unrelated work on feature branches.** Agents sometimes add unrelated fixes or config changes to an existing feature branch. When the PR is squash-merged, the unrelated changes are bundled into the merge commit, but if the branch is reused or if work continues after merge, unrelated commits can be lost.

3. **No branch cleanup.** Stale feature branches accumulate on the remote. Without auto-deletion after merge, old branches linger and create confusion about what is active vs. merged.

4. **No guardrails.** No branch protection rules exist. Anyone can push directly to the default branch, force-push, or bypass review.

## Milestones

### Milestone 1: Enable Branch Protection on gh-pages
- Require status checks to pass before merge (CI job: `orphan-check`)
- Block direct push to gh-pages
- Block force push to gh-pages
- **Tool:** `gh api repos/clyons/resume/branches/gh-pages/protection`

### Milestone 2: Enable Auto-Delete Branches After Merge
- Set `delete_branch_on_merge=true` on the repository
- Ensures feature branches are removed immediately after PR merge
- **Tool:** `gh api repos/clyons/resume --method PATCH`

### Milestone 3: Post-Merge Orphan Detection CI Workflow
- Create `.github/workflows/orphan-check.yml`
- Trigger on `pull_request: [closed]` where `merged == true`
- Fetch full history, check for commits on the branch not reachable from gh-pages
- If orphaned commits found, open a GitHub issue with the commit list
- Handle already-deleted branches gracefully (exit 0)

### Milestone 4: Add Inviolate Branch Hygiene Rules
- Create `CLAUDE.md` with branch hygiene rules
- Rules cover: one branch per feature, never push after merge, verify branch before merge, follow-up work on new branch, orphan-check is P0

### Milestone 5: Clean Up Stale Remote Branches
- Delete all remote branches that are fully merged into gh-pages
- Verify with `git branch -r`

## Validation Criteria

- [ ] `gh api repos/clyons/resume/branches/gh-pages/protection` returns 200
- [ ] `gh api repos/clyons/resume --jq .delete_branch_on_merge` returns true
- [ ] `git branch -r` shows only active unmerged branches
- [ ] All changes committed on own branch with PR opened
