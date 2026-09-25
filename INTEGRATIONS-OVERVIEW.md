# Darkmoon Integrations — Overview

The five official Darkmoon integrations share a single frozen foundation:
[`@darkmoon_ai/client`](../darkmoon-client) (`CONTRACT_VERSION = 1.0.0`). Each
integration links that package (or its `darkmoon-ci` CLI) and speaks to **Darkmoon
OSS** (local CLI + on-disk JSON) or **Darkmoon Pro** (`/api/v1` REST + JWT + SSE)
behind the same normalized contract.

> ⭐ Darkmoon is open-source — **a star really helps us grow.** [![Star the Darkmoon core](https://img.shields.io/github/stars/ASCIT31/Dark-Moon?style=social&label=Star%20Darkmoon)](https://github.com/ASCIT31/Dark-Moon) · 🌐 [dark-moon.org](https://dark-moon.org) · 📚 [docs.dark-moon.org](https://docs.dark-moon.org)

| Repo | Artifact | Distribution channel |
|---|---|---|
| [`darkmoon-client`](../darkmoon-client) | `@darkmoon_ai/client` (lib) + `darkmoon-ci` (CLI) | npm |
| [`darkmoon-action`](../darkmoon-action) | GitHub Action (node20, bundled `dist/`) | GitHub Marketplace (git tag) |
| [`darkmoon-gitlab`](../darkmoon-gitlab) | GitLab CI/CD Component (`scan`) | GitLab CI/CD Catalog (git tag) |
| [`darkmoon-jenkins`](../darkmoon-jenkins) | Jenkins plugin (`darkmoon-scan.hpi`) | Jenkins Update Center |
| [`darkmoon-vscode`](../darkmoon-vscode) | VS Code extension (`.vsix`) | VS Code Marketplace |
| [`darkmoon-jetbrains`](../darkmoon-jetbrains) | JetBrains plugin (`.zip`, + Kotlin client) | JetBrains Marketplace |

All six repos are **MIT** licensed and carry a `CHANGELOG.md`, matching the
`n8n-nodes-darkmoon` precedent.

---

## Compatibility matrix (integration × capability)

`✓` = supported · `Pro` = Pro-only (capability-gated, hidden/disabled on OSS) ·
`–` = intentionally not exposed by this integration.

| Capability | client / `darkmoon-ci` | GitHub Action | GitLab | Jenkins | VS Code | JetBrains |
|---|:--:|:--:|:--:|:--:|:--:|:--:|
| **launch** campaign | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| **campaigns** (browse/list) | ✓ | run-scoped | run-scoped | run-scoped | ✓ | ✓ |
| **findings** (redaction-safe) | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| **report** (redacted default) | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| report **full** (two-key opt-in) | ✓ | ✓ (private file) | opt-in artifact | ✓ (`includeFullReport`) | ✓ (local only) | ✓ (confirm dialog) |
| **live progress** | ✓ (SSE/poll) | Pro (SSE) / poll | – (waits) | – (waits) | Pro | Pro¹ |
| **fail-policy** (findings-based) | ✓ | ✓ | ✓ | ✓ | – (browse-only) | – (browse-only) |
| **scheduler** | Pro² | – | – | – | – | – |
| **remediation → PR** | Pro² | – | – | – | – (hidden) | – (hidden) |

¹ JetBrains gates streaming on `features.streaming` but ships no streaming UI —
clean degradation, not a live view.
² `scheduler` and `remediation` are Pro backend capabilities surfaced through
`detect().features`. They are **deliberately not driven** by the published
integrations: the web dashboard, scheduled campaigns and the remediation→PR flow
are **Pro** and are never presented as open source (see the OSS/Pro boundary).

### Edition boundary (OSS vs Pro)

| | OSS (`OssLocalBackend`) | Pro (`ProHttpBackend`) |
|---|---|---|
| Transport | on-disk JSON tree + `darkmoon.sh` | `/api/v1` REST + JWT + SSE |
| Detection | CLI/root probe | `GET /system/info` (fallback root probe) |
| Launch | `opencode run` w/ PROGRAM nonce | `POST /run/campaign` |
| Correlation | snapshot-diff + session-id + mtime (warns on collision) | `run_id` → campaign |
| Live progress | synthetic poll | live SSE |
| Dashboard / scheduler / remediation | no | yes (never presented as OSS) |

Every backend value is normalized into the same canonical enums:

- **Severity** `critical | high | medium | low | info` (unknown → `info`, never escalated)
- **FindingStatus** `exploited | confirmed | unconfirmed | remediated`
- **CampaignStatus** `queued | running | completed | stopped | failed | unknown`

---

## Cross-cutting safety contract (all integrations honor)

- **Redaction-safe by default.** `getReport()` is redacted (evidence blanked,
  secrets scrubbed); `listFindings()`/`getFinding()` return `evidence: null`
  unless `includeEvidence`. Un-redacted values require the deliberate two-key
  opt-in `{ full: true, private: true }` (passing `full` alone throws). No
  integration writes un-redacted evidence to CI logs, SARIF, store listings, or
  editor panes by default.
- **`raw` is never emitted.** `Campaign.raw` / `Finding.raw` are never logged or
  serialized to CI output by any integration.
- **Findings-based pass/fail.** `computeFailPolicy(findings, "critical,high")` is
  the only source of CI verdicts; the pentest process exit code is ignored
  (`darkmoon-ci` exit: `0` pass, `2` policy, `1` error).
- **OSS concurrency.** OSS runs share one data dir; run **one container /
  compose-project per CI job**. Documented in the Action, GitLab and Jenkins READMEs.
- **Version negotiation is uniform** because it is centralized in the client's
  `detect()` (`negotiateProVersion`, `SUPPORTED_API_MAJORS = [1]`): every
  integration that goes through `DarkmoonClient` / `darkmoon-ci` inherits the same
  major-version guard and the same `/system/info` capability discovery.
- **Secrets at rest.** VS Code uses `SecretStorage`; JetBrains uses `PasswordSafe`;
  Jenkins/GitLab/Action pass tokens via env/credentials, never on argv.

## Documents

- [`CLIENT-API.md`](./CLIENT-API.md) — developer/API reference for
  `@darkmoon_ai/client` + `darkmoon-ci` (the contract surface).
- [`PUBLISH-READINESS-AUDIT.md`](./PUBLISH-READINESS-AUDIT.md) — per-target
  publishability, tokens, and the `gh` / `npm` authentication state.
- [`../darkmoon-client/CONTRACT.md`](../darkmoon-client/CONTRACT.md) — the frozen
  contract itself.
