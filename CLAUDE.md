# CLAUDE.md

## Branch Hygiene (INVIOLATE)

These rules are mandatory. Violating them causes orphaned work that is silently lost.

1. **One branch = one feature.** Each feature branch must contain ONLY commits related to its named purpose. Unrelated fixes, config changes, etc. go on their own branches.
2. **Never push commits after PR merge.** Once a PR is squash-merged, the branch is dead. Any commits pushed afterward will be orphaned and lost. The branch is auto-deleted after merge.
3. **Verify branch is clean before merge.** Before requesting merge, confirm all branch commits are included: `git log origin/gh-pages..<branch> --oneline` should show only the commits you intend to merge.
4. **Follow-up work = new branch.** After a PR merges, pull latest gh-pages and create a fresh branch. Never reuse a merged branch.
5. **A post-merge CI check detects orphaned commits.** If you violate these rules, the `orphan-check` workflow will open an issue. Treat orphan issues as P0.
