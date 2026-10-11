---
name: cadenceux-ottofms-docker-setup
description: >-
  Installs and activates OttoFMS, the FileMaker Server deployment and automation companion by
  Proof+Geist (Ottomatic), inside a Docker container already running FileMaker Server: the
  install, starting the systemd service, reaching it through FileMaker Server's Nginx at
  /otto/, committing the container, and licence activation. Use when asked to install, set up,
  activate or troubleshoot OttoFMS on Docker or in a container ("install OttoFMS", "OttoFMS
  license key", "OttoFMS on Docker"), or on symptoms it covers: OttoFMS asking for a licence
  key, confusion between the Cloud Console's "Add Server" flow and local activation, or a
  service that installs but won't start on arm64. Requires a working base install from
  cadenceux-fms-docker-setup. Not for the base FileMaker Server install, or for FileMaker Pro
  schema, script or layout work.
license: CC-BY-4.0
compatibility: >-
  Claude Code on macOS (Apple Silicon) with Docker Desktop, on a container built with
  cadenceux-fms-docker-setup.
metadata:
  author: Darrin Southern, CadenceUX
  version: "1.1.1"
---

# OttoFMS on Docker — Setup

Covers installing OttoFMS on top of a working FileMaker Server Docker container, getting a free
license, and activating — including two things that aren't obvious from Ottomatic's own docs:
which network direction activation actually needs, and a known arm64-specific version to avoid.

**Prerequisite:** a working FileMaker Server container per **cadenceux-fms-docker-setup**.

**Verified environment:** same as the base skill — macOS/Apple Silicon, Docker Desktop, OttoFMS
4.18.2. Newer builds install with the same script; note the version you get in Step 4's commit.

## Verify the prerequisite is actually met — don't assume it from context

Even if the conversation implies FMS is already set up, check rather than trust the framing —
a stale assumption here wastes the whole OttoFMS install against a container that isn't ready:

```bash
docker exec <container> systemctl is-active fmshelper          # expect: active
curl -sk -o /dev/null -w "%{http_code}\n" https://localhost:<mapped-port>/admin-console/   # expect: 200
```

If either check fails, stop and resolve it via **cadenceux-fms-docker-setup** first rather than
proceeding — installing OttoFMS on top of a broken or half-configured FMS install just adds a
second layer of things to debug at once.

## A decision worth surfacing before Step 5

Before getting a license key, it's worth a one-line check-in with the developer: do they want
**just local activation** (the default this skill covers), or are they also interested in the
Cloud Console's remote-monitoring dashboard later? It doesn't change anything about Steps 1-4,
but it's worth knowing up front so the license-signup step and the "Add Server" section below
land with the right context rather than surprising them later.

## Step 1 — Install OttoFMS

```bash
docker exec <container> bash -c "apt-get install -y sudo"
docker exec <container> bash -c "curl -sSL 'https://downloads.ottomatic.cloud/ottofms/install-scripts/install-linux.sh' | bash"
```

Ottomatic's official one-liner is prefixed with `sudo` throughout its install script. This
minimal container has no `sudo` installed (you're already root via `docker exec`) — same trap
as the Nginx patch in the base skill. Installing the `sudo` package first lets the vendor
script run completely unmodified rather than needing every line hand-edited.

**Known version-specific bug worth checking for:** OttoFMS 4.14.0 and 4.14.1 had an ARM64-only
bug where the installer wrote a broken systemd service file — it started fine when launched
manually but failed via `systemctl`. Fixed in later releases. After install, confirm the
version isn't in that range:

```bash
docker exec <container> bash -c "dpkg -l | grep ottofms-app"
```

If it is (unlikely on a fresh install, but possible if a specific version was pinned), install a
newer version explicitly: the script supports `install-linux.sh -v <version>`.

## Step 2 — Start the service

The package installs a systemd service but — like most package installs inside a container —
doesn't auto-start it (`policy-rc.d returned 101` in the install output is expected, not an
error). Start it explicitly:

```bash
docker exec <container> bash -c "systemctl start ottofms-proofgeist-com.service"
docker exec <container> bash -c "systemctl status ottofms-proofgeist-com.service --no-pager"
```

Confirm `Active: active (running)`.

## Step 3 — Verify it's reachable

OttoFMS runs its own internal port (commonly 3061) but is meant to be accessed **through FMS's
own Nginx**, at a `/otto/` path — not by exposing its internal port separately:

```bash
curl -sk -o /dev/null -w "%{http_code}\n" https://localhost:<your-mapped-https-port>/otto/
```

Expect `200`. If you get a connection refused or a generic Nginx page instead, confirm the
service is actually running (Step 2) before assuming a networking problem — it's almost always
the service not being started yet.

## Step 4 — Lock in the install

Same trap as the base skill: OttoFMS is `apt`-installed into the container's writable layer,
not the image. Re-commit immediately:

```bash
docker commit --message "Added OttoFMS <version> - $(date '+%Y-%m-%d %H:%M')" <container> fmsdocker:installed
```

## Step 5 — Get a license key

OttoFMS is free, but does require a license key to activate:

1. Sign up at **console.ottomatic.cloud/auth/signup** (the developer does this themselves — it's
   their account, and signup may ask for a credit card even though the license itself is free,
   which is a hard line an agent shouldn't cross on someone's behalf regardless of the charge
   amount).
2. The license key appears in the Ottomatic Cloud Console dashboard (look for a **Licenses**
   section) and is also emailed at signup.
3. In the OttoFMS UI (`https://.../otto/`), paste the key into the **License** field and click
   **Activate**.

That's the entire activation flow — there is no separate multi-step "activation wizard" beyond
this single screen.

## A step you almost certainly don't need: "Add Server" in the Cloud Console

Ottomatic's Cloud Console has a separate **"Add Server"** flow (for its optional remote
monitoring/management dashboard) that explicitly asks for your server's URL so the Cloud
Console can reach *in* to verify and connect to it. **This is unrelated to licensing** — do not
route a developer through it just to get OttoFMS activated.

This distinction matters a lot for any server without inbound/public internet access (a home
Docker Desktop setup, anything behind NAT with no port-forwarding): local license activation
only needs **outbound** connectivity from the server (to `edd-sl-proxy.geistgate.com` for
license verification and `downloads.ottomatic.cloud` for the installer/updates) — confirm with:

```bash
docker exec <container> bash -c "curl -sk -o /dev/null -w '%{http_code}\n' --connect-timeout 5 https://edd-sl-proxy.geistgate.com"
docker exec <container> bash -c "curl -sk -o /dev/null -w '%{http_code}\n' --connect-timeout 5 https://downloads.ottomatic.cloud"
```

Both `200` means activation will work regardless of whether the server is reachable from
outside. Only pursue "Add Server" if the developer specifically wants the Cloud Console's
optional remote dashboard — and that needs actual inbound reachability (port forwarding, a
tunnel like Cloudflare Tunnel or Tailscale) which is a separate, bigger conversation.

## Next: automating deployments (brief pointer, not covered in depth here)

OttoFMS's main purpose is running FileMaker's Data Migration Tool as part of automated
deployments, configured via a separate file called **OttoDeploy** (a `.fmp12` file — install
docs at Ottomatic's own **Installing OttoDeploy** page). One detail worth knowing before
planning a deployment: **the "Dev to Prod" pattern works on a single server**, but only cleanly
if the FileMaker solution is a single `.fmp12` file — differently-named dev/prod copies on the
same server (e.g. `MyApp_DEV.fmp12` / `MyApp.fmp12`). A solution split across multiple files is
easier to migrate with two separate servers instead. This is a deeper topic than this skill's
scope — treat it as a pointer, not a full guide, and check Ottomatic's current docs for the
exact deployment-pattern steps since that's product UI, not something this skill's testing
covered start-to-finish.

## Skill update self-check

Once per session, on the first request this skill handles: fetch
`https://github.com/CadenceUX/cadenceux-ottofms-docker-setup-skill/raw/main/VERSION` and compare it with this skill's version
(1.1.1). If newer, start the answer with:

> ⚠️ **Skill update available:** this skill is v[installed]; v[latest] is at
> https://github.com/CadenceUX/cadenceux-ottofms-docker-setup-skill/releases

Skip silently if the fetch fails. If the skill was installed as a plugin, suggest
`claude plugin update` (or the Plugins page on claude.ai) instead. If `fmp-dev-orchestrator` is
installed, skip this check: its stack check covers every CadenceUX skill, plus Claris ADT,
FileMaker Pro and the ADT project.

## Relationship to other skills

- **cadenceux-fms-docker-setup** — the required prerequisite. This skill assumes its steps are
  already done (working FMS container, committed image, ports known) and doesn't repeat them.
  Shares two of the same underlying traps (`sudo` missing in the container, `apt install`
  landing in the container's writable layer) — restated briefly here because they recur, not
  because this skill owns them independently.
- Not related to the `fmp-dev-*` skills or `claris-filemaker-pro` — those cover FileMaker Pro
  authoring, not server DevOps.

## Licence

This skill is released under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).

Built and maintained by [Darrin Southern](https://www.linkedin.com/in/darrin-southern/) from [CadenceUX](https://cadenceux.com.au).
