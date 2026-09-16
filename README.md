# release-pr-demo

Demo of the `main -> production` release PR workflow proposed in
[useblacksmith/web#12282](https://github.com/useblacksmith/web/pull/12282).

- `main` is the integration branch. Feature PRs squash-merge here.
- `production` is what is deployed. It only moves via PRs merged with a merge commit.
- `.github/workflows/release-pr.yml` keeps one `main -> production` PR open, listing the unreleased commits.
- `.github/workflows/release-validate.yml` runs on any PR targeting `production`.

`app.txt` stands in for the application.

## Repo configuration

- Merge methods: merge commits and squash enabled, rebase disabled. Merge commit message = PR title only.
- Ruleset `production`: PRs only, merge-commit method only. Blocks force-push and deletion.
- Ruleset `main`: PRs only, squash **or** merge commit. Squash is the convention for feature PRs; merge commits are needed for back-merges from `production` (see #12 vs #13 below).
- Actions: "Allow GitHub Actions to create and approve pull requests" on.

## Normal flow

| PR | What |
| --- | --- |
| [#1](../../pull/1), [#2](../../pull/2) | Feature PRs squash-merged into `main`. |
| [#3](../../pull/3) | `Release 2026.9.0`, opened by the workflow after #2. Updated in place when [#4](../../pull/4) landed. Merged with a merge commit; `production..main` was empty afterwards. |
| [#5](../../pull/5) | Next feature on `main` opens [#6](../../pull/6) `Release 2026.9.1` (counter incremented). |

## Hotfix A: fix lands on `main` first, cherry-picked to `production`

| PR | What |
| --- | --- |
| [#7](../../pull/7) | Fix squash-merged into `main` like any change. |
| [#8](../../pull/8) | Branch off `production`, `git cherry-pick -x <#7 squash sha>`, PR into `production`, merge commit. Ships without #5. |
| [#9](../../pull/9) | Workflow change: read the `(cherry picked from commit …)` trailer and list such commits under *Already on production* instead of pending. Also run on pushes to `production`. |

The next release merge is clean: both sides carry the same edit.

## Hotfix B: fix written directly on `production`, then reconciled into `main`

| PR | What |
| --- | --- |
| [#10](../../pull/10) | Hotfix branch off `production` with a new commit, merge commit into `production`. `main` now lacks the fix. |
| [#11](../../pull/11) | `production -> main` back-merge: **CONFLICTING**, and so is release PR #6, because #5 edited a line adjacent to #10's. Neither branch accepts direct pushes, so the conflict can't be resolved in place. Closed. |
| [#12](../../pull/12) | Reconcile branch off `main`, `git merge origin/production`, resolve, **squash** into `main`. Content matches, but the squash drops the merge parentage, so the merge base doesn't move and #6 stays CONFLICTING. |
| [#13](../../pull/13) | Same reconcile, merged with a **merge commit** (required loosening `main`'s ruleset). `production` becomes an ancestor of `main`; #6 flips to MERGEABLE. |
| [#14](../../pull/14) | Workflow change: `--first-parent` so a back-merge is one line, and scan all of `production` for cherry-pick trailers. |
| [#6](../../pull/6) | `Release 2026.9.1` merged. `production..main` empty. |

Open release PRs: [`base:production is:open`](../../pulls?q=is%3Apr+is%3Aopen+base%3Aproduction).
