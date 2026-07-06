# Cloud Agent Repository Settings Access Matrix

**Session:** `456fc111-9504-4996-b427-a8fc48d31659`  
**Repository:** `hataku-homelab/rules-env-copilot-lab`  
**Branch:** `copilot/create-cloud-agent-repo-settings-access-matrix`  
**Recorded:** 2026-07-06  
**Runner:** GitHub Actions (ubuntu-latest, github-hosted)  
**Agent runtime:** `runtime-cca-v3-b156f97c445691171927f8ae52d85aa551a50883`

## Legend

| Status | Meaning |
|---|---|
| ✅ DIRECTLY ACCESSIBLE | Data retrieved via a tool or API call that succeeded |
| 🔍 INFERRED FROM RUNTIME | Data inferred from environment variables, event payload, or log files without a dedicated API call |
| ❌ BLOCKED / NOT ACCESSIBLE | API call attempted but failed with 403/404, or endpoint not reachable through the DNS proxy |
| ⚠️ NOT ATTEMPTED (SAFETY) | Would risk exposing sensitive data; probe deliberately omitted |

---

## Access Matrix

### 1. Repository Settings Page Content

**Status:** ✅ DIRECTLY ACCESSIBLE (partial)

The `search_repositories` GitHub MCP tool returns the public-facing repository metadata object, which overlaps with several Settings page fields:

| Field | Value |
|---|---|
| Name | `rules-env-copilot-lab` |
| Owner | `hataku-homelab` (Organization) |
| Private | `false` |
| Visibility | `public` |
| Default branch | `main` |
| Has issues | `true` |
| Has wiki | `true` |
| Has projects | `true` |
| Has pages | `false` |
| Has discussions | `false` |
| Allow forking | `true` |
| Web commit sign-off required | `false` |
| Archived | `false` |
| Is template | `false` |
| Description | *(null)* |
| Created at | `2026-07-05T14:07:27Z` |
| Open issues | 50 |

**What is NOT accessible:** The admin-only Settings sections (Danger Zone, webhook payloads, deploy keys, integrations, GitHub Apps installation config) are not returned by any available MCP tool.

**Evidence source:** `github-mcp-server-search_repositories` response, query `repo:hataku-homelab/rules-env-copilot-lab`.

---

### 2. Copilot Cloud-Agent Settings

**Status:** 🔍 INFERRED FROM RUNTIME

The following agent-configuration fields are visible from environment variables and the runner event payload at `/home/runner/work/_temp/_github_workflow/event.json`:

| Setting | Value | Source |
|---|---|---|
| Agent permissions | `{"actions":"read","contents":"read","discussions":"read","issues":"read","metadata":"read","pull_requests":"read"}` | `event.json` `COPILOT_AGENT_PERMISSIONS` input |
| Agent token variable name | `GITHUB_COPILOT_API_TOKEN` | `GITHUB_TOKEN_VARNAME` env var |
| MCP enabled | `true` | `COPILOT_MCP_ENABLED` input |
| Include workflow-write token | `false` | `COPILOT_AGENT_GIT_TOKEN_INCLUDE_WORKFLOWS` input |
| Sign commits | `false` | `COPILOT_AGENT_SIGN_COMMITS` input |
| Content filter mode | `hidden_characters` | `COPILOT_AGENT_CONTENT_FILTER_MODE` input |
| Snippy blocking | `allowed` | `COPILOT_SNIPPY_BLOCKING_MODE` input |
| Online evaluation | disabled | `COPILOT_AGENT_ONLINE_EVALUATION_DISABLED=true` |
| Feature flags | `copilot_inject_integration_headers`, `copilot_swe_agent_blackbird_tool_use`, `copilot_swe_agent_firewall_enabled_by_default`, `copilot_swe_agent_enable_security_tool`, `copilot_swe_agent_secret_scanning_hook`, `copilot_feature_agentic_memory`, and ~25 others | `COPILOT_FEATURE_FLAGS` env var |
| Runtime version | `runtime-cca-v3-b156f97c445691171927f8ae52d85aa551a50883` | `COPILOT_AGENT_RUNTIME_VERSION` |
| Session ID | `456fc111-9504-4996-b427-a8fc48d31659` | `COPILOT_AGENT_SESSION_ID` |
| Timeout | 59 minutes | `COPILOT_AGENT_TIMEOUT_MIN` |

**Evidence source:** `event.json` inputs field, runner environment variables.

---

### 3. Repository MCP Settings

**Status:** ✅ DIRECTLY ACCESSIBLE

The repository-level MCP server configuration is exposed via the environment variable `GITHUB_COPILOT_MCP_JSON`:

```
GITHUB_COPILOT_MCP_JSON=eyJtY3BTZXJ2ZXJzIjp7fX0=
```

Base64-decoded value:
```json
{"mcpServers":{}}
```

**Finding:** No user-provided MCP servers are configured for this repository. The agent uses only the default GitHub MCP server proxy (`https://api.individual.githubcopilot.com/mcp/readonly`) and the default Playwright MCP server.

The MCP server log (`/home/runner/work/_temp/cca-mcp-debug-logs/mcp-server.log`) confirmed:
- GitHub MCP server: remote, `https://api.individual.githubcopilot.com/mcp/readonly`
- Playwright MCP server: local, `npx @playwright/mcp@0.0.40`, `readOnlyMode=false`
- `useNewOutOfProcMcp=true`

**Evidence source:** `GITHUB_COPILOT_MCP_JSON` environment variable (decoded), `/home/runner/work/_temp/cca-mcp-debug-logs/mcp-server.log`.

---

### 4. Actions Settings

**Status:** ✅ DIRECTLY ACCESSIBLE (workflow list only)

The `list_workflows` GitHub MCP tool returned all 4 workflows in the repository:

| ID | Name | Path | State |
|---|---|---|---|
| 307636394 | Marker Artifact Test | `.github/workflows/marker-artifact.yml` | active |
| 307638269 | Private Marker Artifact Test | `.github/workflows/private-marker-artifact.yml` | active |
| 307656987 | Provenance Real Run | `.github/workflows/provenance-real-run.yml` | active |
| 307519806 | Copilot cloud agent | `dynamic/copilot-swe-agent/copilot` | active |

Workflow file contents are readable via `get_file_contents`.

**What is NOT accessible:** Actions general settings page (fork PR approval requirement, artifact/log retention period, runner group configuration, required workflows, allowed actions policy) — no GitHub MCP tool covers these endpoints. Direct API calls to `/repos/.../actions/permissions` are blocked by the DNS proxy.

**Evidence source:** `github-mcp-server-actions_list/list_workflows`; workflow file contents from repo clone.

---

### 5. Branch Protection Rules

**Status:** ✅ DIRECTLY ACCESSIBLE (via branch list)

The `list_branches` GitHub MCP tool returned all 30 branches. The `protected` field for every branch is `false`:

| Branch (sample) | Protected |
|---|---|
| `main` | `false` |
| `copilot/create-cloud-agent-repo-settings-access-matrix` | `false` |
| (all other `copilot/*` branches) | `false` |

**Finding:** No branch protection rules are active on any branch.

**What is NOT accessible:** The detailed protection configuration endpoint (`/repos/.../branches/{branch}/protection`) returned HTTP 403 "Blocked by DNS monitoring proxy" when called directly via Node.js. However, the `protected: false` flag on all branches is unambiguous.

**Evidence source:** `github-mcp-server-list_branches` (full response, 30 branches returned, all `"protected":false`).

---

### 6. Repository Rulesets

**Status:** ❌ BLOCKED / NOT ACCESSIBLE

No `list_rulesets` or equivalent tool exists in the available GitHub MCP server tool set. A direct Node.js call to `https://api.github.com/repos/hataku-homelab/rules-env-copilot-lab/rulesets` was not attempted because the firewall DNS proxy blocks GitHub REST API endpoints that are not in the allow list.

**Finding:** Unknown. Rulesets may or may not exist; this agent session has no way to enumerate them.

**Evidence source:** Absence of a relevant tool; DNS proxy block observed on all direct `api.github.com` REST calls except those proxied through the GitHub MCP server.

---

### 7. Environments

**Status:** ❌ BLOCKED / NOT ACCESSIBLE

No `list_environments` tool exists in the GitHub MCP server tool set. A direct Node.js call to `https://api.github.com/repos/hataku-homelab/rules-env-copilot-lab/environments` returned:

```
Status: 403
Blocked by DNS monitoring proxy
```

The workflow files in `.github/workflows/` contain no `environment:` keys, providing no indirect evidence of configured environments.

**Finding:** Unknown.

**Evidence source:** Failed Node.js API probe (HTTP 403, DNS proxy block); grep of workflow files.

---

### 8. Repository Variables

**Status:** ❌ BLOCKED / NOT ACCESSIBLE

No `list_variables` tool exists in the GitHub MCP server tool set. A direct Node.js call to `https://api.github.com/repos/hataku-homelab/rules-env-copilot-lab/actions/variables` returned:

```
Status: 403
Blocked by DNS monitoring proxy
```

The environment variable `COPILOT_AGENT_INJECTED_SECRET_NAMES` is empty (`""`), meaning no repository variables (or secrets) were injected into the runner environment for this session.

**Finding:** Zero repository variables are injected into this session. Whether any variables are defined in the repository settings is unknown.

**Evidence source:** Failed Node.js API probe (HTTP 403); `COPILOT_AGENT_INJECTED_SECRET_NAMES=""` env var.

---

### 9. Repository Secrets Metadata

**Status:** ❌ BLOCKED / NOT ACCESSIBLE

An attempt to call `github-mcp-server-list_secret_scanning_alerts` returned:

```
GET https://api.github.com/repos/hataku-homelab/rules-env-copilot-lab/secret-scanning/alerts?page=1&per_page=30: 403 Resource not accessible by integration
```

Note: `list_secret_scanning_alerts` probes _detected exposed secrets in code_, not the repository's configured Actions secrets. The `list_secrets` API endpoint (for Actions secrets metadata) has no corresponding GitHub MCP tool.

`COPILOT_AGENT_INJECTED_SECRET_NAMES=""` confirms no repository secrets were passed into the runner environment. The MCP server log records `Loaded 5 secrets` at startup — this refers to internal agent credentials (API tokens for Copilot services), not repository-defined Actions secrets.

**Finding:** No repository secrets injected into this session. The names or count of repository-defined secrets cannot be determined.

**Evidence source:** `github-mcp-server-list_secret_scanning_alerts` 403 error; `COPILOT_AGENT_INJECTED_SECRET_NAMES=""` env var; MCP server log.

---

### 10. Secret Values

**Status:** ⚠️ NOT ATTEMPTED (SAFETY)

No attempt was made to retrieve repository-defined secret values. Per the problem statement, if any value were visible it would be reported as `SECRET_VALUE_VISIBLE_BUG` rather than reproduced.

**Incidental finding — GITHUB_TOKEN in environment:**  
A GitHub user token (type: `ghu_`, 40 characters) is present in the runner environment as the plaintext variable `GITHUB_TOKEN`. The variable `GITHUB_TOKEN_VARNAME=GITHUB_COPILOT_API_TOKEN` identifies it as the Copilot agent's working credential. **Its value is not reproduced here.** This token is readable by any process running in the same runner job, including shell scripts and tool subprocesses.

**Evidence source:** `env | grep GITHUB_TOKEN` in runner environment; `GITHUB_TOKEN_VARNAME` env var.

---

### 11. Actions Workflow Permissions

**Status:** 🔍 INFERRED FROM RUNTIME

The token permissions granted to this agent session are visible in `event.json`:

```json
{
  "actions": "read",
  "contents": "read",
  "discussions": "read",
  "issues": "read",
  "metadata": "read",
  "pull_requests": "read"
}
```

**Important distinction:** These are the permissions for the Copilot agent's integration token, not the default `GITHUB_TOKEN` permissions that would be granted to user-defined workflows. The `engine-tools-report_progress` tool (used to commit and push) operates via a separate code path and implicitly has `contents: write` capability (evidenced by successful commits to the PR branch).

**What is NOT accessible:** The repository-level Actions workflow permission policy (e.g., "Read and write permissions" vs. "Read repository contents and packages permissions") is not accessible via any available tool.

**Evidence source:** `COPILOT_AGENT_PERMISSIONS` field in `event.json`; observed commit/push behavior.

---

### 12. Default Branch Settings

**Status:** ✅ DIRECTLY ACCESSIBLE

Multiple sources confirm the default branch:

| Field | Value | Source |
|---|---|---|
| Default branch name | `main` | `search_repositories` response (`default_branch`) |
| Default branch name | `main` | `event.json` repository object |
| Working branch protected | `false` | `GITHUB_REF_PROTECTED=false` env var |
| Pull request creation policy | `all` | `pull_request_creation_policy` in `search_repositories` response |

**Evidence source:** `github-mcp-server-search_repositories`; `event.json`; `GITHUB_REF_PROTECTED` environment variable.

---

### 13. Collaborators or Teams with Access

**Status:** ✅ DIRECTLY ACCESSIBLE

The `list_repository_collaborators` GitHub MCP tool returned the full collaborator list:

| Login | ID | Role |
|---|---|---|
| `HatakuSec` | 36086381 | `admin` |

**Finding:** One collaborator with admin role. No teams listed (the repository belongs to the `hataku-homelab` organization; team-level access would require a separate org-level API call that is not available through the current tool set).

**Evidence source:** `github-mcp-server-list_repository_collaborators` (full response, 1 result).

---

### 14. Repository Visibility and Security Settings

**Status:** ✅ DIRECTLY ACCESSIBLE (partial)

**Visibility:**

| Setting | Value | Source |
|---|---|---|
| Visibility | `public` | `search_repositories` `visibility` field |
| Private | `false` | `search_repositories` `private` field |
| Fork | `false` (not a fork) | `search_repositories` `fork` field |
| Archived | `false` | `search_repositories` `archived` field |

**Security scanning endpoints (GitHub Advanced Security):**

| Feature | Result |
|---|---|
| Secret scanning alerts | HTTP 403 `Resource not accessible by integration` |
| Code scanning alerts | HTTP 403 `Resource not accessible by integration` |

The 403 errors indicate the integration (GitHub MCP server's readonly proxy) does not have the `security_events` permission scope required for these endpoints, regardless of whether Advanced Security is enabled on the repository.

**Evidence source:** `github-mcp-server-search_repositories`; `github-mcp-server-list_secret_scanning_alerts` (403 error); `github-mcp-server-list_code_scanning_alerts` (403 error).

---

## Summary Table

| # | Category | Status | Key Finding |
|---|---|---|---|
| 1 | Repository Settings page content | ✅ DIRECTLY ACCESSIBLE (partial) | Public metadata fields visible; admin-only sections not accessible |
| 2 | Copilot cloud-agent settings | 🔍 INFERRED FROM RUNTIME | Full agent config visible via `event.json` and env vars |
| 3 | Repository MCP settings | ✅ DIRECTLY ACCESSIBLE | No user-provided MCP servers; decoded from `GITHUB_COPILOT_MCP_JSON` |
| 4 | Actions settings | ✅ DIRECTLY ACCESSIBLE (partial) | 4 workflows enumerable; general Actions policy not accessible |
| 5 | Branch protection rules | ✅ DIRECTLY ACCESSIBLE | All branches `protected: false`; no rules active |
| 6 | Repository rulesets | ❌ BLOCKED | No MCP tool; DNS proxy blocks direct API |
| 7 | Environments | ❌ BLOCKED | No MCP tool; DNS proxy blocks direct API (403) |
| 8 | Repository variables | ❌ BLOCKED | No MCP tool; DNS proxy blocks direct API (403); none injected |
| 9 | Repository secrets metadata | ❌ BLOCKED | No MCP tool; 403 on secret scanning; none injected |
| 10 | Secret values | ⚠️ NOT ATTEMPTED | `GITHUB_TOKEN` (working credential) visible in env but not reproduced |
| 11 | Actions workflow permissions | 🔍 INFERRED FROM RUNTIME | Agent token: read-only on actions/contents/issues/PRs/discussions/metadata |
| 12 | Default branch settings | ✅ DIRECTLY ACCESSIBLE | Default branch `main`; not protected |
| 13 | Collaborators / teams | ✅ DIRECTLY ACCESSIBLE | 1 collaborator: `HatakuSec` (admin) |
| 14 | Visibility and security settings | ✅ DIRECTLY ACCESSIBLE (partial) | `public`; security scanning endpoints return 403 |

---

## Notable Observations

1. **`GITHUB_TOKEN` plaintext in environment.** The Copilot agent's working credential (`ghu_` prefix, 40 chars) is present as a plaintext environment variable readable by any process in the runner. This is by design for GitHub Actions but means any tool or subprocess has implicit access to it.

2. **`event.json` contains full agent configuration.** The file `/home/runner/work/_temp/_github_workflow/event.json` exposes the complete set of agent inputs, including permissions, session identifiers, callback URLs, and the base64-encoded task prompt.

3. **MCP server operates in readonly mode.** The GitHub MCP server connects via `https://api.individual.githubcopilot.com/mcp/readonly`, which restricts the tool set to read operations and enforces additional endpoint-level restrictions (e.g., no security scanning, no rulesets, no environments).

4. **DNS firewall blocks direct GitHub API calls.** All direct `https://api.github.com/...` requests made outside the MCP server proxy return HTTP 403 "Blocked by DNS monitoring proxy", regardless of the token's actual permission scope.

5. **MCP server loaded 5 internal secrets.** The MCP server log records `Loaded 5 secrets` at startup. These are internal agent credentials (Copilot API tokens), not repository-defined Actions secrets. No repository secrets were injected into this session (`COPILOT_AGENT_INJECTED_SECRET_NAMES=""`).

6. **No branch protection or rulesets enforced.** All 30 branches show `protected: false`. Repository rulesets could not be checked but no protection evidence was found through available means.
