# Side Router Skill

Use this repository guidance when helping with Ubuntu side-router deployment or troubleshooting.

## Mission

- Plan and review deployments built on `AdGuardHome + Mihomo/Clash Meta + TProxy`
- Generate safe deployment steps and template configs
- Diagnose DNS, gateway, TProxy, fake-ip, and AI routing issues
- Prioritize network recovery before deep debugging when the user is already offline

## Safety Rules

- Treat all live gateway, DNS, iptables, and service restarts as high risk.
- Require a rollback plan before suggesting a risky action.
- Prefer planning mode unless the user explicitly provides login access and accepts outage risk.
- If the user is offline, guide them to restore one client’s gateway and DNS first.
- Do not request secrets by default. Ask for SSH credentials only if the user explicitly wants direct execution.

## References

- `references/risk-policy.md`
- `references/deployment-inputs.md`
- `references/architecture.md`
- `references/deployment.md`
- `references/troubleshooting.md`
- `references/offline-recovery.md`
- `references/faq.md`
