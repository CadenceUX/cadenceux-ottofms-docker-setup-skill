# cadenceux-ottofms-docker-setup

A Claude Code skill for installing and activating OttoFMS on top of a FileMaker Server Docker
container — verified end-to-end on macOS (Apple Silicon) with Docker Desktop.

Built and maintained by [Darrin Southern](https://www.linkedin.com/in/darrin-southern/) from [CadenceUX](https://cadenceux.com.au).

## What it covers

- Installing OttoFMS via Ottomatic's own installer script, without a working `sudo` in a
  minimal container
- A known arm64-specific systemd bug in OttoFMS 4.14.0/4.14.1 worth checking for
- Starting the service and verifying it's reachable through FMS's own Nginx at `/otto/`
- Getting a free license key and activating — the actual two-field flow, not a multi-step wizard
- **The key clarification most people get wrong:** license activation only needs *outbound*
  connectivity from the server — a home/NAT'd Docker Desktop setup with no public reachability
  activates fine. The Ottomatic Cloud Console's separate "Add Server" feature (for its optional
  remote-monitoring dashboard) is a different thing entirely, and does need inbound access —
  don't confuse the two

## Prerequisite

Requires **cadenceux-fms-docker-setup**'s base FileMaker Server install to already be working.
This skill doesn't repeat those steps.

## Install

**Claude Code** (from the `cadenceux` plugin marketplace):

```bash
claude plugin marketplace add https://github.com/CadenceUX/cadenceux-skills.git
claude plugin install cadenceux-ottofms-docker-setup@cadenceux
```

**claude.ai and Cowork:** Customize → Plugins → Add marketplace from GitHub →
`CadenceUX/cadenceux-skills`, then install **cadenceux-ottofms-docker-setup**.

**Claude desktop app (macOS):** download `cadenceux-ottofms-docker-setup-v1.1.0.skill` from the
[latest release](https://github.com/CadenceUX/cadenceux-ottofms-docker-setup-skill/releases) and double-click it.

**Claude.ai in a browser, as a single skill:** Customize → Skills → upload the release `.zip`.

## Files

```
cadenceux-ottofms-docker-setup/
├── SKILL.md
├── README.md
└── CHANGELOG.md
```

## Licence

Released under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).

Built and maintained by [Darrin Southern](https://www.linkedin.com/in/darrin-southern/) from [CadenceUX](https://cadenceux.com.au).
