# Scanning template

This repository is a GitHub template. It contains workflows and hooks for security scans and file checks. It does not build, test, or lint a particular language. Copy the repository. Then add the tools that your project needs.

Open Settings, then General. Select Template repository. Create a new repository with the **Use this template** button.

## Tools

| Tool | Where it runs | What it checks |
| --- | --- | --- |
| [pre-commit](https://pre-commit.com/) | Your machine and the `pre-commit` workflow | Hooks from `pre-commit/pre-commit-hooks`, plus spelling |
| [gitleaks](https://github.com/gitleaks/gitleaks) | pre-commit hook and the `gitleaks` workflow | Secrets in git history. The workflow uploads results to GitHub code scanning |
| [OSV-Scanner](https://google.github.io/osv-scanner/) | pre-commit hook and the `OSV-Scanner` workflow | Known vulnerabilities in dependency manifests |
| [Semgrep](https://semgrep.dev/docs) | pre-commit hook and the `Semgrep` workflow | Code patterns. The workflow uploads results to GitHub code scanning |
| [Trivy](https://github.com/aquasecurity/trivy-action) | pre-commit hooks and the `trivy` workflow | Filesystem and configuration findings. The workflow uploads `CRITICAL` and `HIGH` results to GitHub code scanning |
| [Dependency Review](https://docs.github.com/en/code-security/supply-chain-security/understanding-your-software-supply-chain/about-dependency-review) | `Dependency review` workflow | Vulnerable dependencies introduced by a pull request |
| [OSSAR](https://github.com/github/ossar-action) | `OSSAR` workflow on Windows | Open Source Static Analysis Runner. It runs Bandit, ESLint, and BinSkim. The workflow uploads results to GitHub code scanning |
| [Dependabot](https://docs.github.com/en/code-security/dependabot) | Each week | Updates for GitHub Actions and pre-commit hook revisions |
| [mise](https://mise.jdx.dev/) | Your machine and the `pre-commit` workflow | Installs the `pre-commit` command from `mise.toml` |

GitHub code scanning must be enabled for the repository before an upload can succeed. A private repository also needs GitHub Advanced Security.

## Set up a checkout

Install mise. Then install the tools and the git hooks:

```sh
mise install
mise run setup
```

`mise install` reads `mise.toml`. It installs pre-commit 4.6.2. On Linux and macOS, mise uses the upstream zipapp. The zipapp needs a Python interpreter that is already on the machine. GitHub-hosted Ubuntu runners have that interpreter. On Windows, mise installs pre-commit with pipx. mise can install Python for that pipx install. This template does not select a Python version for your project.

`mise run setup` runs `pre-commit install`. A commit then runs the hooks. Skip the hooks with `git commit --no-verify`. The GitHub workflows run even when you skip the hooks.

Run every hook on all files:

```sh
mise run check
```

Activation puts `pre-commit` on `PATH`. If you did not activate mise, put `mise exec --` before each command.

## What GitHub runs

This template copies the Dependency Review, OSV-Scanner, Semgrep, Trivy, and OSSAR workflows from the dyce-zensical repository. It adds the `gitleaks` workflow and the `pre-commit` workflow. It has no workflow for Python tests, coverage, documentation, or publishing.

* The `pre-commit` workflow runs on every push and every pull request. `jdx/mise-action` installs the tools in `mise.toml`. The workflow then runs `pre-commit run --all-files`. The workflow fails if a hook changes a tracked file.
* The `Dependency review` workflow comments on pull requests that target `main`.
* The `OSV-Scanner` workflow scans on pull requests, merge groups, and pushes to `main`. It also runs once each week.
* The `Semgrep` workflow scans on pull requests and on pushes to `main`. It also runs once each week.
* The `trivy` workflow scans on pull requests and on pushes to `main`. It also runs once each week. It reports `CRITICAL` and `HIGH` findings. The local hooks use the image `aquasec/trivy:0.74.0`. The workflow uses `aquasecurity/trivy-action` v0.36.0.
* The `gitleaks` workflow scans git history on pull requests and on pushes to `main`. It also runs once each week. It uses gitleaks 8.30.0. The local hook uses the same version.
* The `OSSAR` workflow scans on pull requests and on pushes to `main`. It also runs once each week. It runs on `windows-latest`. Expect this workflow to fail until the repository contains Python, JavaScript, or a binary. OSSAR runs Bandit, ESLint, and BinSkim. OSSAR exits with an error when none of those inputs are present.

The `pre-commit` workflow sets `SKIP` to `gitleaks,osv-scanner,semgrep,trivyfs-docker,trivyconfig-docker`. Each of those hooks has its own workflow. Those hooks still run on your machine. With that `SKIP` list, the workflow runs `codespell`, `check-useless-excludes`, and the hooks from `pre-commit/pre-commit-hooks`. Only that workflow runs those hooks. Dependency Review and OSSAR have no pre-commit hook.

The Semgrep workflow and the local `semgrep` hook run `semgrep scan` with the public ruleset `p/ci`. You do not need a Semgrep account. The hook and the workflow install Semgrep 1.178.0.

## Add a language later

Add the compiler, package manager, linter, and test runner under `[tools]` in `mise.toml`. The `pre-commit` workflow runs `jdx/mise-action`. The `install` input stays at its default, `true`. mise installs those tools in GitHub Actions.

A hook with `language: system` in `.pre-commit-config.yaml` must find its program on `PATH`. mise puts the tools from `mise.toml` on `PATH` before `pre-commit` runs. A hook with another `language`, such as `python` or `golang`, installs its own copy. pre-commit does that install. Do not add that copy to `mise.toml`.

A Python project that uses uv can add `uv` to `mise.toml`. Add a workflow step that runs `uv sync`. This template does not do that. uv installs the toolchain for a project. mise installs the commands that this template runs.
