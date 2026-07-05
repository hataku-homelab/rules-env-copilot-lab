# MCP Tool Inventory (Safe Read/List/Status Operations)

This document inventories GitHub/MCP tools that can be used in this repository for **read-only**, **list**, and **status** checks.

## Scope and Safety

- Only use harmless operations: `list_*`, `get_*`, `search_*`, and other read/status APIs.
- Do **not** include or request secrets, tokens, credentials, or private keys.
- Prefer least-privilege reads and avoid write/update/delete operations for diagnostics and audits.

## Environments and Deployment Protection Settings

Use these tools to inspect environment protections without changing configuration:

- `github-mcp-server-actions_list` (`list_workflow_runs`)  
  Read workflow run context that may reference deployment/environment usage.
- `github-mcp-server-actions_get` (`get_workflow_run`, `get_workflow_job`)  
  Inspect run/job status related to protected deployments.
- `github-mcp-server-pull_request_read` (`get_check_runs`)  
  Review check run status for deployment-gating checks on PRs.
- `github-mcp-server-get_job_logs`  
  Read failed/specific job logs for environment/protection troubleshooting.

## Branch Protection, Rulesets, and Bypass Actors

Use safe listing/inspection APIs to audit policy coverage:

- `github-mcp-server-list_branches`  
  List branches to understand protection targets.
- `github-mcp-server-pull_request_read` (`get_status`, `get_check_runs`)  
  Inspect required status/check behavior on protected branches.
- `github-mcp-server-list_repository_collaborators`  
  Read collaborator access context relevant to bypass actor review.
- `github-mcp-server-search_issues` / `github-mcp-server-search_pull_requests`  
  Find policy-change discussions, bypass approvals, or governance evidence.

> Note: branch protection, ruleset, and bypass-actor configurations are often easiest to view in repository settings. In MCP workflows, use read/list/status APIs and run/check evidence to validate effective behavior and maintain traceability.

## Deployment APIs and Deployment Statuses

Use read-only status endpoints and related signals:

- `github-mcp-server-actions_list` (`list_workflow_runs`, `list_workflow_jobs`, `list_workflow_run_artifacts`)  
  Track deployment pipeline execution state.
- `github-mcp-server-actions_get` (`get_workflow_run`, `get_workflow_run_usage`, `get_workflow_run_logs_url`)  
  Gather run metadata and status details.
- `github-mcp-server-get_job_logs`  
  Inspect deployment failures and status transitions from logs.
- `github-mcp-server-pull_request_read` (`get_check_runs`, `get_status`)  
  Confirm deployment-related checks reported back to commits/PRs.

## Recommended Read-Only Audit Flow

1. List recent workflow runs for the repo (`list_workflow_runs`).
2. Open failed/in-progress runs (`get_workflow_run`).
3. Inspect failed jobs (`list_workflow_jobs`, `get_job_logs`).
4. Correlate with PR checks (`get_check_runs`, `get_status`).
5. Review branch/collaborator context (`list_branches`, `list_repository_collaborators`).
6. Capture findings without exposing secrets.

## Out-of-Scope for This Inventory

- Any operation that mutates repository settings, environments, rulesets, deployments, or branch protections.
- Any retrieval or publication of secrets/credentials.
