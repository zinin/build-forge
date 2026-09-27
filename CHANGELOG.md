# Changelog

All notable changes to build-forge will be documented here.

## [Unreleased]

### Changed
- **Renamed from claude-forge to build-forge.** `/claude-forge:build` is `/build-forge:build`,
  the agent is `build-forge:build-runner`, the updater skills are `build-forge:*`. Permission
  rules that name the old skills (`Skill(claude-forge:…)`) need the new name.
- **`deps-update` is a skill now** (`skills/deps-update/`): Codex loads skills, not commands.
  The invocation stays `/build-forge:deps-update`.
- **On Grok, `build` dispatches build-runner by type**: `spawn_subagent` with
  `subagent_type: "build-forge:build-runner"`. The skill named only the Task tool, and Grok ran
  a general-purpose stand-in without the runner's tool limits. The skill now forbids a
  stand-in: if the typed call fails, it stops and tells the user (not yet exercised on Grok).
- **build-runner reports a missing tool instead of installing it**: no `sudo`, package manager
  or install command, just BUILD FAILED naming the tool. Grok may not enforce the agent's
  `tools:` allowlist, and a stand-in runner there ran `sudo apt-get install`.

## [0.2.0] - 2026-07-18

### Changed
- `build-runner` agent hardening against hung/slow builds: explicit 30-minute Bash timeout ask on every build command (dependency installs included; harness-clamped to the host ceiling), one-build-at-a-time rule, bounded wait protocol for commands moved to the background (until-loop polling of the task output file, wait budget, orphaned-build reporting — never a re-run), `--console=plain` on every Gradle invocation and `-B` on Maven, INFRASTRUCTURE FAILURE report format with lock-owner diagnostics and extended daemon-failure signatures (read-only `ps`/`jps`/`jstack` plus `sleep` pacing), model `haiku` → `sonnet`.
- `build` skill: strictly sequential agent dispatch, infrastructure-failure retry etiquette (recovery first, then at most one retry; at most 4 build runs total), surgical Daemon Recovery Procedure (`--stop` → identity-checked `kill` of the confirmed lock owner only) executed by the main session, INFRASTRUCTURE FAILURE in the output format.

### Added
- README section "Recommended host setup" (Bash timeout env vars, Gradle daemon idle timeout, multi-JDK toolchain pinning, recovery permissions, quick-diagnostics runbook).

## [0.1.0] - 2026-07-16

### Added
- Initial release: tools ported from the author's internal ai-tools monorepo (@ ad23588).
- `build-runner` agent — build/test/lint executor for Gradle/Maven/Node.js/Go/Python with JDK auto-detection.
- `build` skill — delegates any build/test/lint task to the `claude-forge:build-runner` agent.
- `deps-update` command — orchestrates dependency updates via the updater skills and sonatype-mcp.
- `gradle-plugin-updater` skill — Gradle Plugin Portal version lookups (bundled helper script).
- `google-maven-updater` skill — maven.google.com version lookups (bundled helper script).
