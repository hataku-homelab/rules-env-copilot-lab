# Docker Capability Baseline

This document records whether Docker is available in the current environment and whether the current user can run harmless, read-only Docker commands.

## Summary

- Docker CLI installed: **Yes**
- Docker daemon reachable: **Yes**
- Current user can run read-only commands (`docker version`, `docker info`, `docker ps`): **Yes**

## Commands and Results

### `docker --version`

```text
Docker version 28.0.4, build b8034c0
```

### `docker version`

Result: **Succeeded** (client and server details returned).

### `docker info`

Result: **Succeeded** (daemon and environment info returned).

### `docker ps`

Result: **Succeeded** (returned container list; currently empty).

```text
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

## Safety Notes

- No containers were started.
- No host paths were mounted.
- No secrets were accessed.
- No privilege escalation attempts were made.
