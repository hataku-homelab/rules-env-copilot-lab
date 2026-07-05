# Security Configuration

This document provides an audit-ready overview of the security configuration for the
[`hataku-homelab/rules-env-copilot-lab`](https://github.com/hataku-homelab/rules-env-copilot-lab)
repository. It is intended for external reviewers who do not have access to the repository
settings UI.

**Last reviewed:** 2026-07-05  
**Repository visibility:** Public  
**Organization:** [hataku-homelab](https://github.com/hataku-homelab)

---

## Table of Contents

1. [Branch Rules (Classic Branch Protection)](#1-branch-rules-classic-branch-protection)
2. [Repository Rulesets](#2-repository-rulesets)
3. [Bypass Actors](#3-bypass-actors)
4. [Deployment Environments](#4-deployment-environments)
5. [Environment Protection Settings](#5-environment-protection-settings)
6. [Access Controls Summary](#6-access-controls-summary)

---

## 1. Branch Rules (Classic Branch Protection)

Classic branch protection rules restrict what can be done to specific branches (e.g., requiring
pull request reviews, passing status checks, or preventing force-pushes).

| Branch | Protected | Required Reviewers | Required Status Checks | Allow Force Push | Allow Deletions |
|--------|-----------|-------------------|----------------------|-----------------|-----------------|
| `main` | No | — | — | — | — |

No classic branch protection rules are currently configured. Any user with push access can push
directly to `main` without a pull request or review.

> **Settings page:** `https://github.com/hataku-homelab/rules-env-copilot-lab/settings/branches`

---

## 2. Repository Rulesets

Repository rulesets are a more flexible alternative to classic branch protection. They can target
branches and tags by pattern, apply to specific actors, and include a wider range of rule types
(e.g., required signatures, merge queue, required deployments).

No repository-level rulesets are configured for this repository.

> **Settings page:** `https://github.com/hataku-homelab/rules-env-copilot-lab/settings/rules`

> **Note:** Organization-level rulesets defined in the `hataku-homelab` organization may also
> apply to this repository. Organization rulesets can be reviewed at
> `https://github.com/organizations/hataku-homelab/settings/rules`.

---

## 3. Bypass Actors

Bypass actors are individuals, teams, or apps that are explicitly allowed to bypass the
requirements imposed by branch protection rules or rulesets (e.g., skip required reviews or
status checks).

Since no repository-level branch protection rules or rulesets are currently configured, no
bypass actors are defined at the repository level.

If organization-level rulesets apply, their bypass actor configuration would be visible to
organization owners at `https://github.com/organizations/hataku-homelab/settings/rules`.

---

## 4. Deployment Environments

Deployment environments allow you to model real-world deployment targets (e.g., `production`,
`staging`) and attach protection rules to them. Jobs in GitHub Actions workflows that reference
an environment must satisfy those protection rules before they can run.

No deployment environments are currently configured for this repository.

> **Settings page:** `https://github.com/hataku-homelab/rules-env-copilot-lab/settings/environments`

---

## 5. Environment Protection Settings

Environment protection rules control who and what can trigger a deployment to a given environment.
Common protections include:

| Protection Type | Description |
|-----------------|-------------|
| Required reviewers | One or more people or teams must approve before deployment proceeds |
| Wait timer | A delay (in minutes) before the deployment job starts |
| Deployment branches and tags | Restricts which branches or tags can deploy to the environment |
| Custom deployment protection rules | Integration-based gates (e.g., external approval services) |

Because no deployment environments are configured (see §4), none of these protections are in
effect.

---

## 6. Access Controls Summary

| Actor | Type | Repository Role | Can Push to `main` Directly | Admin Privileges |
|-------|------|----------------|----------------------------|-----------------|
| [HatakuSec](https://github.com/HatakuSec) | User | Admin | Yes | Yes |

Repository admins can modify all branch protection rules, rulesets, environment settings, and
collaborator permissions. There is currently one admin collaborator.

---

## Recommendations

The following settings are commonly applied to repositories that hold production code or
infrastructure definitions. They are offered here as guidance for future hardening:

1. **Enable branch protection on `main`** — Require at least one pull request review and one
   passing status check before merging.
2. **Add a repository ruleset** — Rulesets provide finer-grained control and can enforce
   required signatures, linear history, and merge-queue usage.
3. **Configure deployment environments** — Create `production` and/or `staging` environments
   with required-reviewer protection and branch-filter rules to prevent accidental deployments.
4. **Restrict bypass actors** — If rulesets are added, limit bypass access to specific
   automation accounts or teams rather than individual users.
