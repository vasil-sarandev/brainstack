# GitHub Actions

#technology

GitHub Actions is GitHub’s built-in CI/CD platform. You define workflows in YAML under `.github/workflows/`; events (push, PR, schedule, manual) trigger jobs that run steps on hosted runners or your own machines—build, test, deploy, and automate repo tasks without a separate pipeline service.

---

## Resources

- **Deep Dives**
	- [GitHub Actions: Understanding GitHub Actions](https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions)
	- [GitHub Actions: Workflow syntax](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)
	- [GitHub Actions: Expressions](https://docs.github.com/en/actions/learn-github-actions/expressions)
	- [GitHub Actions: Encrypted secrets](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions)

- **Docs & References**
	- [GitHub Actions Documentation](https://docs.github.com/en/actions)
	- [GitHub Marketplace: Actions](https://github.com/marketplace?type=actions)
	- [actions/checkout](https://github.com/actions/checkout) – standard repo checkout action
	- [actions/cache](https://github.com/actions/cache) – dependency and build cache
	- [actions/upload-artifact](https://github.com/actions/upload-artifact) – pass files between jobs or retain build outputs

---

## Core Concepts

- **Workflow**: A single automated process defined in a YAML file (e.g. `ci.yml`), versioned with the repo.
- **Event**: What starts a workflow—`push`, `pull_request`, `workflow_dispatch`, `schedule` (cron), releases, etc.
- **Job**: A set of steps that run on the same runner; jobs in one workflow can run in parallel or depend on each other via `needs`.
- **Step**: An individual task—run a shell command or use a published **Action** (`uses:`).
- **Runner**: The machine that executes jobs; GitHub-hosted (`ubuntu-latest`, `windows-latest`, `macos-latest`) or **self-hosted** on your infra.
- **Action**: Reusable unit (composite, JavaScript, or Docker) from the Marketplace or your org; keeps workflows DRY.
- **Matrix builds**: Run the same job across multiple OS, Node versions, or other axes in one workflow definition.
- **Secrets & variables**: `secrets` for sensitive values; `vars` for non-secret config at repo, environment, or org level.
- **Environments**: Named targets (e.g. `production`) with protection rules, required reviewers, and environment-specific secrets.
- **Artifacts & caching**: Store build outputs between jobs or runs; cache dependencies to shorten CI time.
- **Permissions (`GITHUB_TOKEN`)**: Scoped token per job; follow least privilege via `permissions:` in workflow or org settings.

---
