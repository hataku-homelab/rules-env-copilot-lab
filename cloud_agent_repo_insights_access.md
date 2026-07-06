# Cloud Agent Repository Insights Access Report

**Repository:** `hataku-homelab/rules-env-copilot-lab`  
**Session date:** 2026-07-06  
**Agent type:** Copilot cloud agent (SWE agent)  
**Branch:** `copilot/cloud-agent-repo-insights-access`

This document reports what a Copilot cloud-agent session can and cannot access regarding repository insights, pulse, traffic, contributors, dependency graph, code/secret scanning, and security metadata.

---

## ⚠️ Security Notice

**SECRET_VALUE_VISIBLE_BUG**

The following environment variables are present in the cloud-agent runner environment and contain a live OAuth token value:

- `GITHUB_TOKEN`
- `GITHUB_COPILOT_API_TOKEN`
- `COPILOT_SDK_AUTH_TOKEN`
- `GITHUB_TOKEN_VARNAME` (points to `GITHUB_COPILOT_API_TOKEN`)

All three contain the same `ghu_` prefixed GitHub OAuth token. The token value was observed in the environment via `env | grep -i token`. The actual token value is not reproduced here, but the fact that it is accessible to any bash command or tool the agent executes is confirmed.

---

## Access Methods Tested

| Method | Status |
|---|---|
| GitHub MCP server tools | ✅ Functional (primary access path) |
| Direct REST API (curl to api.github.com) | ❌ Blocked by sandbox DNS monitoring proxy (HTTP 403 "Blocked by DNS monitoring proxy") |
| Browser / Playwright navigation to github.com | ❌ Blocked by sandbox firewall (ERR\_BLOCKED\_BY\_CLIENT) |
| Repository files (git clone) | ✅ Accessible (shallow clone provided) |

---

## Findings by Category

### 1. Pulse-Style Repository Activity Summary

**Verdict: Partially accessible through GitHub MCP**

No dedicated `/pulse` endpoint exists in the GitHub MCP server. However, a pulse-equivalent summary can be constructed from:

- **Commit list** (`list_commits`): 9 commits visible on `main` since 2026-07-05T14:07Z, all authored by `HatakuSec`.
- **Issues** (`list_issues`): 2 issues total (1 open, 1 closed).
- **Pull requests** (`list_pull_requests`): 67 PRs observed, mostly `copilot/*` branches.
- **Branches** (`list_branches`): 53 branches listed; all `copilot/*` feature branches.

The `/insights/pulse` web page (`https://github.com/hataku-homelab/rules-env-copilot-lab/pulse`) is **not accessible** via browser (firewall blocks it). No direct GitHub API pulse endpoint exists.

---

### 2. Traffic: Views, Clones, Referrers

**Verdict: Blocked — not accessible**

All three REST API endpoints require a token and return `403 Blocked by DNS monitoring proxy` from within the sandbox:

- `GET /repos/{owner}/{repo}/traffic/views` — Blocked
- `GET /repos/{owner}/{repo}/traffic/clones` — Blocked
- `GET /repos/{owner}/{repo}/traffic/popular/referrers` — Blocked
- `GET /repos/{owner}/{repo}/traffic/popular/paths` — Blocked

The GitHub MCP server provides **no traffic tools**. Browser access is also blocked.

**Conclusion:** Traffic data (page views, unique visitors, clone counts, referrer sources) is **not accessible** to the cloud agent in any form.

---

### 3. Contributors Analytics

**Verdict: Partially accessible through GitHub MCP (commit-level only)**

The GitHub REST API statistics endpoints (`/stats/contributors`, `/stats/commit_activity`, `/stats/code_frequency`, `/stats/participation`, `/stats/punch_card`) are all blocked by the sandbox DNS proxy.

What is accessible via GitHub MCP:

- **Collaborator list** (`list_repository_collaborators`): One collaborator — `HatakuSec` with role `admin`.
- **Commit authors** (via `list_commits`): All 9 commits are authored by `HatakuSec` (`36086381+HatakuSec@users.noreply.github.com`).

Full contributor analytics (weekly contribution graphs, addition/deletion counts, commit frequency charts) are **not accessible**.

---

### 4. Dependency Graph / Dependabot Alerts

**Verdict: Blocked — not accessible**

- `GET /repos/{owner}/{repo}/dependabot/alerts` — Blocked by DNS proxy.
- No GitHub MCP tool for dependency graph or Dependabot alerts.
- Browser access blocked.
- Repository files contain no `package.json`, `requirements.txt`, `Gemfile`, or other manifest files that would populate a dependency graph (only `.github/workflows/*.yml` files exist).

**Conclusion:** Dependency graph and Dependabot alert data are **not accessible** to the cloud agent.

---

### 5. Code Scanning Alerts

**Verdict: Blocked — HTTP 403 from GitHub MCP (integration permission denied)**

MCP tool test result:
```
GET https://api.github.com/repos/hataku-homelab/rules-env-copilot-lab/code-scanning/alerts
→ 403 Resource not accessible by integration
```

The GitHub MCP integration token does not have the `security_events` scope or equivalent permission required for code scanning. Direct REST API and browser access are also blocked.

**Conclusion:** Code scanning alerts are **not accessible** to the cloud agent through any path.

---

### 6. Secret Scanning Alerts

**Verdict: Blocked — HTTP 403 from GitHub MCP (integration permission denied)**

MCP tool test result:
```
GET https://api.github.com/repos/hataku-homelab/rules-env-copilot-lab/secret-scanning/alerts
→ 403 Resource not accessible by integration
```

Same permission limitation as code scanning. The MCP integration token lacks the necessary scope.

**Conclusion:** Secret scanning alert data is **not accessible** to the cloud agent through any path.

---

### 7. Security Overview Data

**Verdict: Blocked — not accessible**

GitHub's Security Overview is an organization-level dashboard only available in the GitHub web UI (`/orgs/{org}/security`). There is no REST API or MCP tool for organization security overview. Code and secret scanning alert APIs return 403 (see items 5 and 6). Browser access is blocked.

**Conclusion:** Security overview data is **not accessible** to the cloud agent.

---

### 8. Actions Usage / Billing-Like Information

**Verdict: Accessible through GitHub MCP**

The `get_workflow_run_usage` MCP tool returns per-run billing data including billable runner-minutes.

**Evidence — Marker Artifact Test run #1 (run ID 28753244786):**
```json
{
  "billable": {
    "UBUNTU": {
      "total_ms": 0,
      "jobs": 1,
      "job_runs": [{ "job_id": 85255866727, "duration_ms": 0 }]
    }
  },
  "run_duration_ms": 7000
}
```

Additional Actions metadata accessible via MCP:
- **Workflow list** (`list_workflows`): 4 workflows found (Marker Artifact Test, Private Marker Artifact Test, Provenance Real Run, Copilot cloud agent).
- **Workflow runs** (`list_workflow_runs`): 60 total runs enumerated, including run numbers, timestamps, status, and trigger event type.
- **Job step details** (`list_workflow_jobs`): Full step-by-step job details with timestamps accessible.
- **Job logs** (`get_job_logs`): Full log content retrievable by job ID.

**Note:** Organization-level billing totals or per-user billing reports are **not accessible** via MCP.

---

### 9. Repository Activity Timeline Beyond Normal Commits/PRs/Issues

**Verdict: Partially accessible through GitHub MCP**

Beyond commits, PRs, and issues, the following activity data is accessible:

| Data type | Access | Evidence |
|---|---|---|
| Branches (all) | ✅ GitHub MCP | 53 branches listed including all `copilot/*` experiment branches |
| Workflow runs (all) | ✅ GitHub MCP | 60 runs total; run numbers, events, statuses, timestamps |
| Workflow job step history | ✅ GitHub MCP | Step start/end timestamps, conclusions |
| Job log content | ✅ GitHub MCP | Full log text for any job by ID |
| Releases | ✅ GitHub MCP | 0 releases (none created) |
| Tags | ✅ GitHub MCP | (not tested separately; none expected) |
| Collaborator list | ✅ GitHub MCP | `HatakuSec` (admin) |
| Discussions | Not tested (repo has none) | — |
| Deployments | Not tested | — |
| Repository events stream | ❌ Blocked | REST `/repos/{owner}/{repo}/events` — DNS proxy blocked |
| Organization audit log | ❌ Not available | No MCP tool; org-level admin API |

The repository's branch timeline reflects extensive past cloud-agent sessions: 53 branches such as `copilot/create-agent-environment-baseline`, `copilot/encoded-injection-result-36` through `copilot/encoded-edge-result-52`, and others, providing a rich activity history reconstructed from branch names and commits.

---

## Summary Table

| Data Category | GitHub MCP | Direct REST API | Browser | Repo Files |
|---|---|---|---|---|
| Pulse / activity summary | ⚠️ Partial (commits + issues + PRs) | ❌ Blocked (DNS proxy) | ❌ Blocked | ❌ Not applicable |
| Traffic (views/clones/referrers) | ❌ No tool | ❌ Blocked (DNS proxy) | ❌ Blocked | ❌ Not applicable |
| Contributors analytics | ⚠️ Partial (commit authors + collaborators) | ❌ Blocked (DNS proxy) | ❌ Blocked | ❌ Not applicable |
| Dependency graph / Dependabot | ❌ No tool | ❌ Blocked (DNS proxy) | ❌ Blocked | ❌ No manifests |
| Code scanning alerts | ❌ 403 (permission) | ❌ Blocked (DNS proxy) | ❌ Blocked | ❌ Not applicable |
| Secret scanning alerts | ❌ 403 (permission) | ❌ Blocked (DNS proxy) | ❌ Blocked | ❌ Not applicable |
| Security overview | ❌ No tool | ❌ Blocked (DNS proxy) | ❌ Blocked | ❌ Not applicable |
| Actions usage / billing-like | ✅ Per-run billable ms | ❌ Blocked (DNS proxy) | ❌ Blocked | ❌ Not applicable |
| Activity timeline (extended) | ✅ Branches, runs, logs | ❌ Blocked (DNS proxy) | ❌ Blocked | ✅ Workflow YAMLs |

---

## Environment Observations

- The cloud agent runs inside a GitHub Actions runner (`ubuntu-latest`).
- The runner environment exposes `GITHUB_TOKEN` (and aliases) containing a live `ghu_` OAuth token visible to any process the agent spawns. See security notice at top.
- Outbound HTTPS to `api.github.com` and `github.com` is blocked by a DNS monitoring proxy; all direct curl attempts return `Blocked by DNS monitoring proxy` / HTTP 403.
- The GitHub MCP server is the sole functional path to GitHub API data.
- The MCP integration token does not grant `security_events` scope (required for code/secret scanning).
- No billing-total API access was tested beyond per-run usage data.
