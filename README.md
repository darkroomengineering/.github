# .github

Organization-level profile and default community health files for
[Darkroom Engineering](https://github.com/darkroomengineering).

This is GitHub's special organization repository. It configures the org's public
presence, provides fallback community health files, and hosts explicitly adopted
CI components. Workflows and actions are not inherited by other repositories.

## Contents

| Path | Purpose |
| --- | --- |
| `profile/README.md` | Renders as the org landing page at [github.com/darkroomengineering](https://github.com/darkroomengineering). |
| `SECURITY.md` | Default vulnerability-disclosure policy. |
| `CONTRIBUTING.md` | Default contribution guidelines. |
| `PULL_REQUEST_TEMPLATE.md` | Default PR template. |
| `.github/ISSUE_TEMPLATE/` | Default bug and feature issue templates, plus chooser config. |
| `.github/FUNDING.yml` | Sponsor button configuration. |
| `actions/bun-web-ci/action.yml` | Shared Bun installation, production build and project checks. |
| `.github/workflows/dependabot-merge.yml` | Guarded merge workflow for explicitly selected Dependabot policies. |
| `.github/workflows/ci-gate.yml` | Org-required check on every default branch (see below). |

## Shared CI components

Callers must pin these components to a reviewed full commit SHA. Publishing a
component does not enable it anywhere. Verify a real pull request and its
observed check names before changing required checks or adopting merge automation.

The Bun web action expects a root `package.json` with an exact stable
`packageManager` pin such as `bun@1.3.5`, a text `bun.lock`, and working `build`
and `check` scripts. It installs with `--frozen-lockfile`, caches `.next/cache`,
builds, and runs `bun run check`. The project's check script must retain its
lint, typecheck and test coverage; absent or failing scripts fail the job.

| Input | Accepted values | Default |
| --- | --- | --- |
| `node-version-file` | Empty, or `.node-version` to set Node before installation. | Empty |
| `oxlint-annotations` | `true` to run the existing Oxlint annotation command, or `false`. | `false` |

Use `darkroomengineering/.github/actions/bun-web-ci@<reviewed-full-commit-sha>`
as a step after checkout. The caller owns its triggers, job names, read-only
permissions, runner, timeout and cancellation policy. Keep browser tests and
advisory jobs in the caller. For Satus, pass both inputs; for Lenis Showcase Admin,
use the defaults. This action does not skip tests or accept arbitrary commands.

## Required default-branch gate

The org ruleset **Default branch gate** requires `ci-gate.yml` from this repository's
`main` on the default branch of every repository except forks and repositories named
in the ruleset's exclusions. Changes reach a default branch only through a pull request.
Organization owners can bypass a failing gate on a pull request, never with a direct push.

For a root Bun project (`package.json` plus `bun.lock`), the gate installs with
`--frozen-lockfile`, generates Next.js route types when the project uses Next.js,
and runs the `check` script, or `typecheck` when `check` is absent. It warns when
`packageManager` lacks an exact `bun@x.y.z` pin and when neither script exists.
Other repositories pass with a notice. Production builds stay with Vercel previews,
which carry each project's environment variables.

A repository with its own CI makes that CI a required status check in its own
ruleset for the default branch. The gate then passes with a notice and installs
and runs nothing. A file in the repository does not count: only a required check
does, so a repository cannot leave the gate without enforced CI. The gate still
starts a job, and GitHub bills each job as at least one minute. To save that
minute too, an owner names the repository in the ruleset's exclusions.

Every change to `ci-gate.yml` changes the gate for the whole organization.

## Dependabot merge policy

Call `darkroomengineering/.github/.github/workflows/dependabot-merge.yml@<reviewed-full-commit-sha>`
as a job from a `workflow_run` caller listening for completion of its verified
CI workflow. Pass the required `update-policy` input, and optionally
`min-release-age-days`:

| Policy | Eligible updates |
| --- | --- |
| `stable-dependencies` | Individual stable patch/minor dependency or Actions bumps using the existing Dependabot titles. |
| `actions-only` | Individual stable patch/minor Actions bumps that change only top-level `.github/workflows/*.yml` or `*.yaml` files. Application updates remain manual. |

Major, 0.x, prerelease, grouped and unrecognized titles remain manual. The workflow
requires a unique open, nondraft Dependabot PR from the same repository to `main`,
the exact current head, and a successful latest run and attempt of the triggering
CI workflow. Actions-only changes require the complete paginated file list,
including previous paths for renames. Merge uses `--match-head-commit` without
an administrative bypass.

Two more guards apply to every caller:

- **Real CI only.** The workflow does not merge when the triggering run is
  `ci-gate`. That gate only type-checks, which does not prove that an update
  works. The caller's workflow must build and test the project.
- **Release age.** The updated version must be public for at least
  `min-release-age-days` (default 7). Malicious releases are usually found and
  removed within days. The workflow reads the publish date from the npm registry,
  or from the GitHub release for an Action. A release without a readable date
  stays manual.

Set the same delay as a Dependabot `cooldown` in the caller's `dependabot.yml`,
so that a PR only opens for a release that is old enough:

```yaml
cooldown:
  default-days: 7
```

Without the cooldown, a PR for a young release stays open and is not merged.
The workflow runs again only when its CI runs again.

The caller grants `actions: read`, `contents: write` and `pull-requests: write`.
The workflow uses the caller's automatic `GITHUB_TOKEN`; do not pass a token or
use `secrets: inherit`. Only `darkroomengineering` callers are eligible. The
privileged job performs API requests and does not check out or execute PR code.
Success proves the triggering workflow passed, not that every possible check is
required. Preserve repository branch rules and verify actual caller runs.

## Bun security updates

Native Bun Dependabot currently supports scheduled **version updates**, not
automated **security updates**. Keep an explicit manual security-update process;
setting `open-pull-requests-limit: 0` does not create a Bun security-only updater.
See GitHub's [supported ecosystems](https://docs.github.com/en/code-security/reference/supply-chain-security/supported-ecosystems-and-repositories).

For a security patch, verify current upstream advisories and releases, update the
manifest and Bun lockfile together using the pinned runtime, and prove frozen
installation, build and checks before merging. Refresh alerts after the dependency
graph processes the merged commit. A stale or empty alert list alone does not
establish that the installed dependencies are current or patched.

## Notes

- **Inheritance.** A repository that ships its own `SECURITY.md`,
  `CONTRIBUTING.md`, issue templates, etc. overrides the default here.
  Inheritance covers community health files only. It does not cover
  `FUNDING.yml`, which GitHub reads per repository (each repo needs its own
  `.github/FUNDING.yml` for the Sponsor button to appear on it).
- **Sponsors block.** The `<!-- sponsors -->` markers in `profile/README.md`
  fence an auto-generated region. Leave the markers in place and edit content
  outside them.
