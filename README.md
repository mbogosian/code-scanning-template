# Scanning template

This repository is a GitHub template for security and basic code-quality checks. It does not build, test, or lint a particular language. Copy it, then add the tools that project needs.

Mark the GitHub repository as a template under Settings, General, Template repository. Use it from the **Use this template** button.

## Tools

| Tool | Where it runs | What it checks |
| --- | --- | --- |
| [pre-commit](https://pre-commit.com/) | Your machine and the `pre-commit` workflow | File hygiene and spelling |
| [gitleaks](https://github.com/gitleaks/gitleaks) | pre-commit hook and the `gitleaks` workflow | Secrets in git history. Results upload to code scanning |
| [OSV-Scanner](https://google.github.io/osv-scanner/) | pre-commit hook and the `OSV-Scanner` workflow | Known vulnerabilities in dependency manifests |
| [Semgrep](https://semgrep.dev/docs) | pre-commit hook and the `Semgrep` workflow | Code patterns. Results upload to code scanning |
| [Trivy](https://github.com/aquasecurity/trivy-action) | pre-commit hooks and the `trivy` workflow | Filesystem and configuration findings. The workflow uploads `CRITICAL` and `HIGH` results to code scanning |
| [Dependency Review](https://docs.github.com/en/code-security/supply-chain-security/understanding-your-software-supply-chain/about-dependency-review) | `Dependency review` workflow | Vulnerable dependencies introduced by a pull request |
| [OSSAR](https://github.com/github/ossar-action) | `OSSAR` workflow, Windows | Open-source static analysis. Results upload to code scanning |
| [Dependabot](https://docs.github.com/en/code-security/dependabot) | Weekly | Updates for GitHub Actions and pre-commit hook revisions |
| [mise](https://mise.jdx.dev/) | Your machine and the `pre-commit` workflow | Installs the `pre-commit` command declared in `mise.toml` |

Code scanning uploads need GitHub code scanning enabled for the repository. A private repository also needs GitHub Advanced Security for those uploads.

## Set up a checkout

Install mise, then install this repository's tools and git hooks:

```sh
mise install
mise run setup
```

`mise install` reads `mise.toml` and installs `pre-commit` 4.6.2. On Linux and macOS, mise uses the upstream zipapp, which runs with a Python interpreter already on the machine. GitHub-hosted Ubuntu runners have one. On Windows, mise installs pre-commit with pipx and can provision Python for that install. The template does not select a project Python version.

`mise run setup` runs `pre-commit install`. A normal commit then runs the hooks. A developer can skip them with `git commit --no-verify`. The hooks are a convenience. The GitHub workflows are the enforcement.

Run every hook against the whole tree:

```sh
mise run check
```

If your shell is not activated for mise, prefix those commands with `mise exec --`.

## What GitHub runs

The Dependency Review, OSV-Scanner, Semgrep, Trivy, and OSSAR workflows started from the dyce-zensical repository. The `gitleaks` and `pre-commit` workflows were added for this template. Python test, coverage, documentation, and publish jobs are not included.

* `pre-commit` runs on every push and every pull request. `jdx/mise-action` installs the tools in `mise.toml`, then runs `pre-commit run --all-files`. The job fails if a hook changes a tracked file.
* `Dependency review` comments on pull requests that target `main`.
* `OSV-Scanner` scans the tree on pull requests, merge groups, pushes to `main`, and a weekly schedule.
* `Semgrep` scans on pull requests, pushes to `main`, and a weekly schedule.
* `trivy` scans on pull requests, pushes to `main`, and a weekly schedule. It reports `CRITICAL` and `HIGH` findings. The local hooks run the `aquasec/trivy:0.74.0` image. The workflow uses `aquasecurity/trivy-action` v0.36.0.
* `gitleaks` scans git history on pull requests, pushes to `main`, and a weekly schedule. It uses gitleaks 8.30.0, the same version as the local hook.
* `OSSAR` scans on pull requests, pushes to `main`, and a weekly schedule. It runs on `windows-latest`.

The CI hook job skips `gitleaks`, `osv-scanner`, `semgrep-ci`, `trivyfs-docker`, and `trivyconfig-docker`. Their workflows already enforce those checks. The same hooks remain available locally. The CI hook job still runs codespell and the file checks, because no other workflow does. Dependency Review and OSSAR run only as workflows.

The Semgrep workflow and the `semgrep-ci` hook call Semgrep CI. Create a free Semgrep account and set these repository secrets before those runs can publish results:

* `SEMGREP_APP_TOKEN`
* `SEMGREP_DEPLOYMENT_ID`

Until those secrets exist, remove the `semgrep-ci` hook or expect that hook and the Semgrep workflow to fail.

## Add a language later

Add the compiler, package manager, linter, and test runner to `[tools]` in `mise.toml`. mise installs them in CI because the pre-commit workflow already runs `jdx/mise-action` with `install` left at its default, `true`.

Hooks in `.pre-commit-config.yaml` that use `language: system` must find their programs on `PATH`. mise puts the tools from `mise.toml` on `PATH` before `pre-commit` runs. Hooks with another `language`, such as `python` or `golang`, still install their own copies. pre-commit does that. You do not add those copies to `mise.toml`.

A Python project that uses uv can add `uv` to `mise.toml` and a workflow step that runs `uv sync`. This template does not do that. uv is a project toolchain. mise is only here to install the commands the template itself runs.
