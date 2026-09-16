# release-pr-demo

Demo of the `main -> production` release PR workflow proposed in
[useblacksmith/web#12282](https://github.com/useblacksmith/web/pull/12282).

- `main` is the integration branch. Feature PRs squash-merge here.
- `production` is what is deployed. It only moves via PRs merged with a merge commit.
- `.github/workflows/release-pr.yml` keeps one `main -> production` PR open, listing the unreleased commits.
- `.github/workflows/release-validate.yml` runs on any PR targeting `production`.

`app.txt` stands in for the application.
