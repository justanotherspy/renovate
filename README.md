# renovate

The shared [Renovate](https://docs.renovatebot.com/) preset for every
`justanotherspy` repository. Each repository's `renovate.json` extends it:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["local>justanotherspy/renovate"]
}
```

`local>justanotherspy/renovate` reads [`default.json`](default.json) from this
repository's default branch, so a change merged here reaches every repository on
its next Renovate run. Keep repository-specific rules (a custom manager for a
hand-pinned version, a hold on a major that upstream tooling cannot take yet) in
that repository's own `renovate.json`, next to the file they describe.

## What the preset does

| Setting | Why |
| --- | --- |
| `config:best-practices` | Renovate's maintained baseline: dependency dashboard, digest pinning for Docker images and GitHub Actions, dev-dependency pinning, config migration PRs, abandoned-package reporting, weekly lock file maintenance, a 3-day minimum release age for npm. |
| `:semanticCommits` | Conventional Commit titles (`chore(deps): …`, `fix(deps): …`); release-drafter and git-cliff in these repos group changelogs by them. |
| `helpers:pinGitHubActionDigestsToSemver` | Action pins read `@<sha> # vX.Y.Z` (full version in the comment, never a floating `# v4`). |
| `security:minimumReleaseAgeCrate`, `security:minimumReleaseAgePypi` | The same 3-day release-age cooldown for crates and PyPI that `config:best-practices` applies to npm. |
| `security:gomodIndirectSecurityUpdates` | Indirect Go modules are updated only when they carry a vulnerability. |
| `customManagers:githubActionsVersions`, `customManagers:makefileVersions`, `customManagers:dockerfileVersions` | Any `*_VERSION` variable in a workflow, Makefile or Dockerfile is kept current once it carries a `# renovate: datasource=… depName=…` comment on the line above. |
| `labels: ["dependencies"]`, `major` label on major updates | Filtering. Vulnerability fixes also get `security`. |
| `osvVulnerabilityAlerts`, `dependencyDashboardOSVVulnerabilitySummary: "unresolved"` | Vulnerability PRs from osv.dev in addition to GitHub's advisories, and a list of unresolved CVEs on the dashboard. |
| `postUpdateOptions: gomodTidy, gomodUpdateImportPaths` | Go updates run `go mod tidy`, and a major bump (`/v89` → `/v92`) rewrites the import paths, so the PR builds and passes a tidy check. |
| `pip-compile` on `.github/requirements/*.txt` | The hash-pinned CI tool requirements (semgrep, zizmor) are recompiled from their `.in` files. See below. |
| Non-major updates grouped and automerged | One `all non-major dependencies` PR, merged once CI is green. |
| Digest and pin updates automerged | SHA-pinned actions and image digests. |
| Lock file maintenance automerged, before 5am Monday (UTC) | Transitive dependencies stay fresh without a PR to click. |
| Majors never automerged | Breaking changes wait for a human. |

## `.github/requirements` files

Renovate's `pip-compile` manager re-runs the command recorded in the header of
each `.txt` file, and it only understands `--flag=value` spelling. Generate the
files like this, or Renovate reports an error on the dashboard and skips them:

```sh
uv pip compile --universal --generate-hashes --python-version=3.13 \
  .github/requirements/zizmor.in --output-file=.github/requirements/zizmor.txt
```

## Validating a change

CI runs `renovate-config-validator --strict` on every pull request. Locally
(Renovate 44 needs Node 24):

```sh
npx --yes --package renovate -- renovate-config-validator --strict --no-global default.json renovate.json
```
