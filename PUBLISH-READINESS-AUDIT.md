# Darkmoon integrations — publish-readiness audit

_As of 2026-09-24. Nothing was pushed or published — this drives Wave-4._

## Authentication state (this environment)

| Tool | State | Detail |
|---|---|---|
| `gh` | **authenticated** | account `MBK-fr`, **admin** of the `ASCIT31` GitHub org; scopes include `repo`, `workflow`, `admin:org`, `write:packages`, `delete_repo`. (The `Dark-Moon-X` gh account token is invalid — ignore.) |
| `npm` | **NOT authenticated** | `npm whoami` fails; no `~/.npmrc`; no `NPM_TOKEN`. `@darkmoon_ai/client` is unregistered (name free), but the `@darkmoon` **scope/org must be created on npmjs by a human**. |
| GitLab (`glab`/token) | **none** | `glab` not installed; no `GITLAB_TOKEN`/`CI_JOB_TOKEN`. |
| VS Code (`vsce`/`ovsx`) | **no token** | no `VSCE_PAT`/`OVSX_PAT` (vsce runnable via `npx`). |
| JetBrains | **no token** | no `PUBLISH_TOKEN`/signing material. |

All six repo names are **free** under `ASCIT31` (none exist yet).

**Bottom line:** every **GitHub repo** can be created and pushed now with the
`MBK-fr` token. Every **registry / marketplace** publication is **human-only** —
the platform accounts and tokens are not present here.

## Per-target table

| Target | Auto-publishable now? | Exact next step |
|---|---|---|
| **client → npm** (`@darkmoon_ai/client`) | **No — human-only** | Create the `@darkmoon` org/scope on npmjs, `npm login` (or set `NPM_TOKEN`), then `cd darkmoon-client && npm publish --access public` (scoped package → `--access public` required). |
| **client → GitHub** (`ASCIT31/darkmoon-client`) | **Yes (gh)** | `gh repo create ASCIT31/darkmoon-client --private --source=/home/mehdi/darkmoon-client --remote=origin --push` (choose `--public` per policy — see note ①). |
| **action → GitHub** (`ASCIT31/darkmoon-action`) | **Yes (gh)** for the repo | `gh repo create ASCIT31/darkmoon-action --public --source=/home/mehdi/darkmoon-action --remote=origin --push`. Marketplace **listing is human-only**: tag `v0.1.0` + a moving `v0`/`v1`, then "Publish to Marketplace" in the GitHub UI (accept the Marketplace agreement once). |
| **gitlab → CI/CD Catalog** | **No — human-only (different platform)** | Create a GitLab project (pick the Darkmoon namespace), push, tag semver; the `release` job publishes to the Catalog via `CI_JOB_TOKEN`. Needs a GitLab account/namespace + a runner — none configured here. |
| **jenkins → Update Center** | **No — human-only** | Requires jenkinsci onboarding (hosting request; repo lands at `github.com/jenkinsci/darkmoon-scan-plugin`), a jenkins.io account, and a Maven release of a **non-SNAPSHOT** (`pom.xml` is `0.1.0-SNAPSHOT`) with Artifactory creds. A source mirror can still be pushed to `ASCIT31/darkmoon-jenkins` via `gh` now. |
| **vscode → VS Code Marketplace** | **No — human-only** (repo: yes via gh) | Create the `darkmoon` publisher on the Marketplace, get an Azure DevOps PAT, then `npx @vscode/vsce publish` (or upload `darkmoon-vscode-0.1.0.vsix`). Optionally `ovsx publish` for Open VSX. |
| **jetbrains → JetBrains Marketplace** | **No — human-only** (repo: yes via gh) | JetBrains account + ASC-IT vendor profile; **first upload manual** via the Marketplace UI; then `PUBLISH_TOKEN` (+ signing material) drives `publishPlugin`. Plugin id `fr.ascit.darkmoon` is permanent after first upload. |

### Note ① — repo visibility & dev fixtures

The real OWASP Juice Shop lab fixtures were **removed from every shipped artifact**
(npm tarball, VSIX, HPI, plugin zip, GitLab templates, Action bundle+vendor).
They remain only in **dev-time test trees** (`darkmoon-client/fixtures`,
`*/test/**`, `*/e2e/**`, `*/src/test/resources/**`). Juice Shop is standard,
non-sensitive security-training data, so these are acceptable to keep, but if the
GitHub repos are made **public** they will be visible there. Decide public/private
per repo before pushing (the commands above default `darkmoon-client` to private
and the marketplace-facing repos to public — adjust to policy).

## Readiness checklist (done in this pass)

- All six repos: **MIT LICENSE + CHANGELOG**, `version 0.1.0`
  (Jenkins pom is `0.1.0-SNAPSHOT` by design — bump at release).
- `repository` / `homepage` / `bugs` metadata set on client + action `package.json`;
  vscode/jetbrains already had `ASCIT31/...` repository URLs; jenkins pom `scm` →
  jenkinsci (correct for its channel).
- `@darkmoon_ai/client` `files` allowlist ships **`dist` + docs only** (fixtures no
  longer shipped); tarball verified 12 files, clean; `npm run build` + `npm test`
  (60/60) green.
- Action vendors the **fixture-free** client tarball; `npm ci` reproduces clean.
- VSIX repackaged from **synthetic** demo fixtures; typechecks + compiles.
- `.vscodeignore` excludes `test/**`, `src/**`, `**/*.map`; HPI/plugin-zip exclude
  test resources.
