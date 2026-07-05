# MCP Tool Inventory (Observed in this Copilot task)

This inventory is based on the tool list exposed to this session.  
No tokens, credentials, secrets, environment values, or internal prompts are included.

## Available GitHub/MCP tool categories observed

- Local workspace/shell tools (bash, file viewing, search, patching)
- Session/context tools (intent reporting, SQL/session history)
- Agent/subagent tools (task agents, code search subagent, skill runner)
- GitHub repository/API tools (issues, PRs, commits, files, labels, discussions, collaborators, releases, tags)
- GitHub Actions tools (workflows, workflow runs, jobs, logs, artifacts, usage)
- GitHub security alert tools (code scanning and secret scanning alerts)
- Runtime security validation tools (dependency advisory DB, secret scanning, parallel validation)
- Browser automation tools (Playwright)

## Read tools observed

- `view`, `read_bash`, `list_bash`
- `glob`, `rg`, `search_code_subagent`
- `session_store_sql`, `sql` (query/read capability)
- `read_agent`, `list_agents`
- GitHub read/list/search/get methods (issues/PRs/commits/files/discussions/labels/releases/tags/actions/security alerts)

## Write tools observed

- `apply_patch` (edit/add/delete files)
- `bash` (can execute local write operations)
- `write_bash`, `stop_bash`
- `engine-tools-report_progress` (commit/push progress via platform tool)
- `engine-tools-reply_to_comment` (reply to PR comments)
- `runtime-tools-store_memory`, `runtime-tools-vote_memory`
- `report_intent`

## Actions/log/artifact tools observed

- `github-mcp-server-actions_list`:
  - `list_workflows`
  - `list_workflow_runs`
  - `list_workflow_jobs`
  - `list_workflow_run_artifacts`
- `github-mcp-server-actions_get`:
  - `get_workflow`
  - `get_workflow_run`
  - `get_workflow_job`
  - `get_workflow_run_logs_url`
  - `get_workflow_run_usage`
  - `download_workflow_run_artifact`
- `github-mcp-server-get_job_logs`

## Issue/PR tools observed

- Issues:
  - `github-mcp-server-list_issues`
  - `github-mcp-server-issue_read`
  - `github-mcp-server-search_issues`
- Pull requests:
  - `github-mcp-server-list_pull_requests`
  - `github-mcp-server-pull_request_read`
  - `github-mcp-server-search_pull_requests`
  - `engine-tools-reply_to_comment`

## Repository content tools observed

- `github-mcp-server-get_file_contents`
- `github-mcp-server-list_branches`
- `github-mcp-server-list_commits`
- `github-mcp-server-get_commit`
- `github-mcp-server-search_code`
- `github-mcp-server-search_commits`
- `github-mcp-server-search_repositories`
- `github-mcp-server-search_users`
- `github-mcp-server-list_repository_collaborators`
- `github-mcp-server-list_tags`, `github-mcp-server-get_tag`
- `github-mcp-server-list_releases`, `github-mcp-server-get_latest_release`, `github-mcp-server-get_release_by_tag`

## Security scanning tools observed

- Runtime security tools:
  - `runtime-tools-gh-advisory-database` (dependency vulnerability checks)
  - `runtime-tools-secret_scanning` (scan changed files for secrets)
  - `parallel_validation` (Code Review + CodeQL scan)
- GitHub security alert APIs:
  - `github-mcp-server-list_code_scanning_alerts`
  - `github-mcp-server-get_code_scanning_alert`
  - `github-mcp-server-list_secret_scanning_alerts`
  - `github-mcp-server-get_secret_scanning_alert`

## Deployment/environment/ruleset tools observed

- No dedicated GitHub deployment-management tool calls were exposed in this tool list.
- No dedicated GitHub environment-management tool calls were exposed in this tool list.
- No dedicated GitHub ruleset-management tool calls were exposed in this tool list.
- Environment customization was observed via `skill` with `customize-cloud-agent` (cloud agent setup/customization skill).

## Tools that appear unavailable or blocked

- `summarize_job_log_failures` is referenced in instructions but no callable tool with that name is exposed in this session.
- Direct `git push`/`gh` push is disallowed by environment policy; `engine-tools-report_progress` is the supported push path.
- Access to `.github/agents` is explicitly disallowed by policy.
- External internet access is limited/partially blocked by sandbox policy.
- Per task requirement, modifying settings/workflows/deployments/secrets/rulesets is out of scope; only harmless read/list/status operations should be used for those domains.
