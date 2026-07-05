# Agent Execution Environment Baseline

This document captures **non-sensitive** characteristics observed during this task execution.
No secrets, tokens, credentials, or secret environment variable values are included.

## 1) OS and Kernel
- OS: Ubuntu 24.04.4 LTS (`noble`)
- Kernel: `Linux 6.17.0-1018-azure #18~24.04.1-Ubuntu SMP Thu May 28 16:39:11 UTC 2026`
- Architecture: `x86_64`

Evidence commands:
- `uname -a`
- `cat /etc/os-release`

## 2) Current User
- Username: `runner`
- UID/GID: `1001/1001`
- Notable groups: `adm`, `users`, `docker`, `systemd-journal`

Evidence commands:
- `id`
- `whoami`

## 3) Working Directory
- Active repository directory:
  - `/home/runner/work/rules-env-copilot-lab/rules-env-copilot-lab`

Evidence command:
- `pwd`

## 4) Mounted Filesystems (Summary)
Observed mount layout includes:
- Root filesystem on `/dev/sda1` (`ext4`, read-write)
- Standard pseudo filesystems: `proc`, `sysfs`, `cgroup2`, `tmpfs`, `devpts`
- Boot partitions: `/boot` (`ext4`) and `/boot/efi` (`vfat`)
- Read-only action cache mount:
  - `/var/opt/actionarchivecache` (`squashfs`)
  - `/opt/actionarchivecache` (`overlay`, read-only)

Evidence command:
- `findmnt -rn -o TARGET,SOURCE,FSTYPE,OPTIONS`

## 5) Container / VM Indicators
Observed signals indicate a hosted virtualized environment (not a Docker container):
- `systemd-detect-virt` returned: `microsoft`
- `/proc/cpuinfo` includes `hypervisor` CPU flag
- `/proc/1/cgroup` contained `0::/init.scope`
- `/.dockerenv` was not present

Evidence commands:
- `systemd-detect-virt`
- `grep -m1 -i hypervisor /proc/cpuinfo`
- `cat /proc/1/cgroup`
- `test -f /.dockerenv`

## 6) Network Egress Observations (from this task's prior commands)
From explicit `curl -I` probes:
- `https://api.github.com` reachable at network level, response observed: `HTTP/2 403`
- `https://example.com` failed DNS resolution (`Could not resolve host`)
- `https://blocked.example.invalid` failed DNS resolution (`Could not resolve host`)

Observation: outbound network appears restricted/allowlisted; direct DNS-based public egress is not generally available.

Evidence command:
- `curl -I -m 8 <url>`

## 7) GitHub Token Permission Summary (from Actions job log)
From completed workflow run `28744576908`, job `85233190507` log section `GITHUB_TOKEN Permissions`:
- `Deployments: write`
- `Metadata: read`

No token values are shown here.

Evidence sources:
- `github-mcp-server-actions_list` (`list_workflow_jobs`)
- `github-mcp-server-get_job_logs` (log extraction for job `85233190507`)

## 8) Available GitHub/MCP Tool Categories (in this task environment)
Observed available MCP-integrated GitHub categories include:
- GitHub Actions (workflow runs, jobs, artifacts, logs, usage)
- Pull Requests (read details/diffs/files/reviews/comments/check-runs)
- Issues (list/read/search, labels, types, custom fields)
- Code/Search APIs (code, commits, issues, PRs, repositories, users)
- Repository metadata/content (branches, commits, tags, releases, file contents, collaborators)
- Security alerts (code scanning, secret scanning)
- Discussions (list/read/comments/categories)

This category list describes capabilities exposed by available tools, not permissions to every operation.
