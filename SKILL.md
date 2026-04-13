---
name: side-router-skill
description: Plan, review, and troubleshoot Ubuntu side-router deployments built with AdGuardHome, Mihomo/Clash Meta, and TProxy. Use when Codex needs to design deployment steps, generate configuration templates, audit DNS or gateway takeover plans, verify fake-ip and AI routing behavior, or guide emergency recovery after a risky network change.
---

# Side Router Skill

Read this skill before changing gateway, DNS, TProxy, or fake-ip settings on a live Ubuntu side router.

## Core Behavior

- Treat this as a high-risk networking skill.
- Default to planning and review before execution.
- Require a rollback path before any risky change.
- Prefer incremental changes over large one-shot rewrites.
- Back up existing config before suggesting a mutating step.
- If the user does not provide login details, stay in planning mode and generate deployment instructions instead of pretending you can execute them.

## Read Order

1. Read `references/risk-policy.md`.
2. Read `references/deployment-inputs.md` to determine whether the user wants planning mode or direct execution mode.
3. Read `references/architecture.md` and `references/deployment.md` for the standard topology.
4. Read `references/troubleshooting.md` and `references/faq.md` when the request mentions outages, fake-ip, AI routing, or broken services.
5. Read `references/offline-recovery.md` first if the user reports no internet access.

## Modes

### Planning Mode

Use when the user wants:

- a deployment blueprint
- config templates
- service layouts
- rollout validation
- rollback planning
- review of an existing topology

Typical inputs:

- main router IP
- target side-router IP
- Ubuntu version
- LAN NIC name
- whether gateway and DNS should point to the side router
- whether to enable AdGuardHome, Mihomo, TProxy, and fake-ip
- subscription model and AI routing target regions

### Direct Execution Mode

Use only if the user explicitly provides:

- SSH host or terminal access
- login identity
- privilege model
- outage tolerance
- recovery path

If any of those are missing, do not imply that execution is safe or possible.

## Risk Triggers

Pause and restate risk before proceeding if the request involves:

- changing DHCP gateway or DNS
- modifying `/opt/clash/config.yaml`
- editing `AdGuardHome.yaml`
- changing `iptables` or policy routing
- restarting Mihomo or AdGuardHome
- enabling TProxy or fake-ip on a live network

## fake-ip Policy

- Treat fake-ip as a compatibility risk, not a neutral default.
- Check `fake-ip-filter`, `DIRECT` rules, and AdGuardHome allow rules together.
- Expect issues with domestic video sites, Tencent game voice traffic, WeChat image flows, and some AI subdomains if allowlists are incomplete.
- When diagnosing fake-ip problems, classify the symptom first:
  - wrong DNS answer
  - wrong routing decision
  - UDP flow broken while TCP still works
  - app-level anti-proxy checks

## Deployment Outputs

When asked to deploy, produce:

- a topology summary
- exact assumptions
- step-by-step deployment order
- config templates with placeholders
- validation commands
- rollback steps

Use `assets/config/` templates when they fit instead of rewriting from scratch.
