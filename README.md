# .github

Organization-level profile and default community health files for
[Darkroom Engineering](https://github.com/darkroomengineering).

This is GitHub's special organization repository. Nothing here is application
code. It configures the org's public presence and provides fallback defaults
that every repo in the org inherits unless it ships its own copy.

## Contents

| Path | Purpose |
| --- | --- |
| `profile/README.md` | Renders as the org landing page at [github.com/darkroomengineering](https://github.com/darkroomengineering). |
| `SECURITY.md` | Default vulnerability-disclosure policy. |
| `CONTRIBUTING.md` | Default contribution guidelines. |
| `PULL_REQUEST_TEMPLATE.md` | Default PR template. |
| `.github/ISSUE_TEMPLATE/` | Default bug and feature issue templates, plus chooser config. |
| `.github/FUNDING.yml` | Sponsor button configuration. |

## Notes

- **Inheritance.** A repository that ships its own `SECURITY.md`,
  `CONTRIBUTING.md`, issue templates, etc. overrides the default here.
  Inheritance covers community health files only. It does not cover
  `FUNDING.yml`, which GitHub reads per repository (each repo needs its own
  `.github/FUNDING.yml` for the Sponsor button to appear on it).
- **Sponsors block.** The `<!-- sponsors -->` markers in `profile/README.md`
  fence an auto-generated region. Leave the markers in place and edit content
  outside them.
