# Network Boundary Observations

This document records the results of simple HTTPS HEAD requests made from the Copilot coding-agent sandbox environment to a set of well-known public hostnames. The purpose is to understand which outbound connections the sandbox allows and which it blocks.

> **Scope:** Only standard HTTPS HEAD requests to the listed public hostnames were attempted. No internal IPs, cloud-metadata endpoints, network scanning, tunneling, or firewall-bypass techniques were used.

## Methodology

Each test used `curl` with a 10-second timeout:

```
curl -s -o /dev/null -w "%{http_code}" --max-time 10 -I "https://<hostname>"
```

A successful TCP connection and a valid HTTP response (any status code) indicates the endpoint is **allowed**. A DNS resolution failure (curl exit code 6 / HTTP `000`) or connection timeout indicates the endpoint is **blocked**.

## Results

| Hostname | HTTP Status | curl Exit Code | Redirect Location | Verdict |
|---|---|---|---|---|
| `docs.github.com` | 302 Found | 0 | `https://docs.github.com/en` | ✅ Allowed |
| `api.github.com` | 403 Forbidden | 0 | _(none)_ | ✅ Allowed |
| `uploads.github.com` | 302 Found | 0 | `https://github.com/` | ✅ Allowed |
| `raw.githubusercontent.com` | 301 Moved Permanently | 0 | `https://github.com/` | ✅ Allowed |
| `example.com` | _(no response)_ | 6 | _(none)_ | 🚫 Blocked |

## Notes

- **`docs.github.com`** — Connection was established and the server issued a redirect to the language-prefixed path (`/en`). Outbound access is permitted.
- **`api.github.com`** — Connection was established. The `403 Forbidden` response is the server's normal reply to an unauthenticated, bare HEAD request; the connection itself is not blocked by the sandbox.
- **`uploads.github.com`** — Connection was established and the server redirected to `github.com`. Outbound access is permitted.
- **`raw.githubusercontent.com`** — Connection was established and the server redirected to `github.com`. Outbound access is permitted.
- **`example.com`** — DNS resolution failed (curl exit code 6, HTTP status `000`, response time ≈ 0 ms). The sandbox blocks this hostname, most likely at the DNS or firewall level. No TCP connection was established.

## Summary

The sandbox allows outbound HTTPS traffic to `*.github.com` and `*.githubusercontent.com`. General internet access (e.g., `example.com`) is blocked, consistent with a network policy that restricts egress to a known allowlist of GitHub-related domains.
