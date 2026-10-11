# Changelog

## [1.1.1] — 2026-10-11

### Added

- **Skill update self-check.** Once per session the skill compares its version with the repo's
  `VERSION` and says when a newer release is out. It defers to `fmp-dev-orchestrator`'s stack
  check when that's installed, which covers every CadenceUX skill plus Claris ADT, FileMaker Pro
  and the ADT project. New eval case: skill-update-check.

## [1.1.0] — 2026-10-11

First public release on GitHub, updated to Anthropic's skill authoring best practices using `cadenceux-skill-creator` 2.0.0.

### Checked (2026-10-11)

- Ottomatic's latest OttoFMS build is **4.18.3** (from `console.ottomatic.cloud/downloads/ottofms/latest/arm`). The runbook was verified on 4.18.2; the install script's interface (`install-linux.sh [-v <version>]`) is unchanged, so the steps apply, but 4.18.3 itself hasn't been run through them.

### Changed

- Frontmatter to the CadenceUX standard: third-person description with "Use when" and "Not for", `license: CC-BY-4.0`, `metadata.author`, three-part version.
- `VERSION`, `.claude-plugin/plugin.json` and a `.gitignore` added; published to the `cadenceux` plugin marketplace.

## v1.0 — 2026-08-11 (updated 2026-08-11, pre-release)

Initial release. Built directly from a real OttoFMS 4.18.2 install on top of the FileMaker
Server Docker container covered by `cadenceux-fms-docker-setup` (macOS/Apple Silicon, Docker
Desktop).

Covers: installing via Ottomatic's official script without a working `sudo`, the known
ARM64-specific systemd bug in versions 4.14.0/4.14.1, starting and verifying the service,
getting a free license and activating, and the outbound-vs-inbound connectivity distinction
between simple license activation and the Ottomatic Cloud Console's separate "Add Server"
remote-monitoring feature.

**Same-day correction (pre-release, amended in place, no version bump — unshipped):** added an
explicit check that the `cadenceux-fms-docker-setup` prerequisite is actually working (not just
assumed from conversational context) before starting, and a call-out to surface the local-only
vs. Cloud-Console-remote-monitoring choice with the developer before it becomes relevant, rather
than only explaining it after the fact in the "Add Server" section.

Local-only release — not yet published to a GitHub repo, no self-update-check.
