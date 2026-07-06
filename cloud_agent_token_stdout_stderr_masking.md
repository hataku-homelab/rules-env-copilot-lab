# Cloud Agent Token stdout/stderr Masking Test

## Summary

Bounded masking test for `GITHUB_TOKEN`, `GITHUB_COPILOT_API_TOKEN`, and
`COPILOT_SDK_AUTH_TOKEN` in the cloud-agent (Copilot Coding Agent) runtime.

No token value, prefix, suffix, hash, checksum, length, or reversible fragment
is recorded in this file.

---

## Test Methodology

1. **Presence check** – confirm each variable is set using a conditional branch;
   no value is printed.
2. **stdout echo** – each variable is echoed once to stdout.
3. **stderr echo** – each variable is echoed once to stderr.
4. **Log inspection** – the captured output is compared in-memory against the
   original variable value; matches are never printed.
5. **Result classification** – one of four labels is emitted per variable per
   stream: `REDACTED`, `UNREDACTED_BUG`, `NOT_PRESENT`, `NOT_TESTED`.

---

## Runtime Environment

| Property | Value |
|---|---|
| Environment | Copilot Coding Agent sandbox (bash, not a GitHub Actions runner) |
| Log masking layer | None (GHA runner secret-masking is not active in this context) |

---

## Results

### Token Presence

| Variable | Present |
|---|---|
| `GITHUB_TOKEN` | yes |
| `GITHUB_COPILOT_API_TOKEN` | yes |
| `COPILOT_SDK_AUTH_TOKEN` | yes |

### stdout Masking

| Variable | Result |
|---|---|
| `GITHUB_TOKEN` | UNREDACTED_BUG |
| `GITHUB_COPILOT_API_TOKEN` | UNREDACTED_BUG |
| `COPILOT_SDK_AUTH_TOKEN` | UNREDACTED_BUG |

### stderr Masking

| Variable | Result |
|---|---|
| `GITHUB_TOKEN` | UNREDACTED_BUG |
| `GITHUB_COPILOT_API_TOKEN` | UNREDACTED_BUG |
| `COPILOT_SDK_AUTH_TOKEN` | UNREDACTED_BUG |

---

## Interpretation

`UNREDACTED_BUG` means the in-memory comparison found that the value returned
by `echo "$VAR"` in the sandbox shell matched the raw variable value — i.e., no
masking layer intercepted the output.

This result is expected for the cloud-agent sandbox: the GitHub Actions runner's
automatic secret-masking (which rewrites matching substrings to `***` in live
job logs) is **not** active here. The sandbox is a plain bash process, not an
Actions runner step.

To observe `REDACTED` results, run the companion workflow
`.github/workflows/cloud-agent-token-masking.yml` in an actual GitHub Actions
job where secrets are registered with the runner.

---

## Workflow for GitHub Actions Context

A repeatable workflow is provided at
`.github/workflows/cloud-agent-token-masking.yml`.  
Dispatch it via `workflow_dispatch` to run the same bounded test under a real
GitHub Actions runner, where the masking layer is active.  
`GITHUB_COPILOT_API_TOKEN` and `COPILOT_SDK_AUTH_TOKEN` must be added as
repository secrets before dispatching.
