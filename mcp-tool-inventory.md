# MCP/GitHub Tool Inventory

This document inventories GitHub/MCP-related tool categories and operations observed as available in this Copilot task.

## GitHub/MCP tool categories observed
- GitHub Actions (workflows, runs, jobs, artifacts, logs, usage)
- Issues and Pull Requests
- Repository metadata and content
- Commits, branches, tags, and releases
- Discussions and labels
- Security alert surfaces (Code Scanning, Secret Scanning)
- Search (code, commits, issues, pull requests, repositories, users)
- Agent/automation integration tools (progress reporting, PR comment replies, validation)

## Read tools observed
- `github-mcp-server-issue_read` (issue details, comments, parent/sub-issues, labels)
- `github-mcp-server-pull_request_read` (PR details, diff, files, comments, reviews, status/check runs)
- `github-mcp-server-get_file_contents` (repository file/directory content)
- `github-mcp-server-get_commit` / `github-mcp-server-list_commits`
- `github-mcp-server-list_branches`, `github-mcp-server-list_tags`, `github-mcp-server-get_tag`
- `github-mcp-server-list_releases`, `github-mcp-server-get_latest_release`, `github-mcp-server-get_release_by_tag`
- `github-mcp-server-list_discussions`, `github-mcp-server-get_discussion`, `github-mcp-server-get_discussion_comments`
- `github-mcp-server-list_label`, `github-mcp-server-get_label`
- `github-mcp-server-list_repository_collaborators`

## Write tools observed
- `engine-tools-report_progress` (records progress and pushes committed local changes)
- `engine-tools-reply_to_comment` (posts replies on PR comments/reviews)

## Actions / log / artifact tools observed
- `github-mcp-server-actions_list` with methods:
  - `list_workflows`
  - `list_workflow_runs`
  - `list_workflow_jobs`
  - `list_workflow_run_artifacts`
- `github-mcp-server-actions_get` with methods:
  - `get_workflow`
  - `get_workflow_run`
  - `get_workflow_job`
  - `download_workflow_run_artifact`
  - `get_workflow_run_usage`
  - `get_workflow_run_logs_url`
- `github-mcp-server-get_job_logs` (single job logs or failed-job logs)

## Issue/PR tools observed
- Issues:
  - `github-mcp-server-issue_read`
  - `github-mcp-server-list_issues`
  - `github-mcp-server-search_issues`
- Pull requests:
  - `github-mcp-server-list_pull_requests`
  - `github-mcp-server-search_pull_requests`
  - `github-mcp-server-pull_request_read`

## Repository content tools observed
- `github-mcp-server-get_file_contents`
- `github-mcp-server-search_code`
- `github-mcp-server-search_commits`
- `github-mcp-server-search_repositories`
- `github-mcp-server-search_users`
- `github-mcp-server-list_branches`
- `github-mcp-server-list_commits`
- `github-mcp-server-list_tags`
- `github-mcp-server-list_releases`

## Security scanning tools observed
- GitHub security alert APIs:
  - `github-mcp-server-list_code_scanning_alerts`
  - `github-mcp-server-get_code_scanning_alert`
  - `github-mcp-server-list_secret_scanning_alerts`
  - `github-mcp-server-get_secret_scanning_alert`
- Runtime validation/security tools:
  - `parallel_validation` (includes Code Review + CodeQL Security Scan)
  - `runtime-tools-secret_scanning` (secret detection in changed files)
  - `runtime-tools-gh-advisory-database` (dependency advisory checks)

## Deployment / environment / ruleset tools observed
- No dedicated GitHub deployment-management or repository ruleset-management MCP operations were observed in the available tool list for this task.
- Environment/setup support observed via non-GitHub tooling:
  - `customize-cloud-agent` skill (agent environment customization workflow)

## Tools appearing unavailable or blocked
- Per task constraints, direct `git push`/`gh`-based PR updates are blocked; progress/push flow is expected via `engine-tools-report_progress`.
- Access to `.github/agents` is explicitly disallowed by task policy.
- No direct MCP operations for editing repository secrets, deployment environments, or rulesets were observed in the available tool definitions provided to this task.

## Safety note
This inventory intentionally excludes tokens, credentials, secret values, environment variable values, and internal system prompts.
