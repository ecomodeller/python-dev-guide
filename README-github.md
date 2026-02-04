# Social Coding (Git and GitHub)

## Repository creation

1. Create public repositories whenever possible. But make sure they are tidy.
2. Open Source is only open if you [add a license](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/licensing-a-repository). Default to MIT.
3. Naming: Prefer dashes `-` over underscores `_` in repo names (e.g. `my-python-package.git/my_python_package/__init__.py`).

## Branches and protection

1. Use `main` as the default branch name.
2. Enable branch protection on `main`:
   - Require pull request reviews before merging
   - Require status checks to pass (CI)
   - Do not allow bypassing the above settings
3. Delete branches after merging (enable "Automatically delete head branches" in repo settings).

## Git workflow

1. Great commit messages are short (`<=50` chars) and start with capitalized action words in imperative ("Add feature X", "Improve performance" rather than "Added ..." or "Fixes ...").
2. Write issues before you start to fix something and take the description seriously.
3. Always create feature branches and PRs, even for small fixes. Use `Closes #1` or `Fixes #1` to [link issues to the pull request](https://docs.github.com/en/issues/tracking-your-work-with-issues/using-issues/linking-a-pull-request-to-an-issue).
4. Write a clear PR title describing what you are fixing/adding, with a short description.
5. Open work-in-progress PRs as [Draft Pull Requests](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests#draft-pull-requests).
6. For large PRs, include a checklist (`- [x] task`) to track progress.
7. Only squash commits in exceptional circumstances (e.g. after many small changes that were reversed).
8. Only maintainers may assign issues/PRs, add labels, and merge pull requests.
9. Merge frequently, especially when multiple people work on the same repository.
10. Use git tags for [semantic](https://semver.org/) versioning: `v1.0.0`, `v2.0.0-beta1`, `v3.0.0-rc1`.

## GitHub CLI

Use the [GitHub CLI](https://cli.github.com/) (`gh`) for common operations:

```bash
gh repo clone owner/repo      # clone a repository
gh issue create               # create an issue
gh issue list                 # list issues
gh pr create                  # create a pull request
gh pr checkout 123            # check out a PR locally
gh pr merge                   # merge current PR
gh run list                   # list workflow runs
gh run watch                  # watch a running workflow
```

## GitHub Actions (CI/CD)

1. Place workflow files in `.github/workflows/`.
2. Run CI on both `push` and `pull_request` events.
3. Use specific action versions with SHA pins or version tags (e.g. `actions/checkout@v4`).
4. Cache dependencies to speed up builds.
5. Keep workflows focused—separate lint, test, and deploy jobs.

Example workflow structure:

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: astral-sh/setup-uv@v5
      - run: uv sync
      - run: uv run ruff check .
      - run: uv run pytest
```

## Raising issues

Well written issues are always welcome, even when you are not 100% sure about their relevance.

Write a descriptive title stating:
- the bug: "App breaks when hit with X"
- the question: "How to do Y?"
- the feature request: "Add function to perform task Z"

In the description, explain what you expect and why. For bug reports, describe when you encountered the issue. Provide a [Minimal Reproducible Example](https://stackoverflow.com/help/minimal-reproducible-example) so others can reproduce it—you'll often figure out the cause while creating one.

Note: Adding labels and assignment are for maintainers only.
