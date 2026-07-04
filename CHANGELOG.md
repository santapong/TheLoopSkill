# Changelog

All notable changes to TheLoopSkill are documented here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

_Nothing yet._

## [0.4.0] — 2026-07-04

Renamed every skill into a collision-free **`loop-*`** namespace. The previous names were identical to Claude Code's built-in skills (`reviewing-code`, `diagnosing-bugs`, …), and the orchestration skill (`workflow`) additionally shadowed the built-in `/workflows` command. This is a **breaking change**: skill invocations change (e.g. `/workflow` → `/loop-engine`, `/reviewing-code` → `/loop-review`).

### Changed
- **All 12 skills renamed:**

  | Old | New | | Old | New |
  |---|---|---|---|---|
  | `workflow` | `loop-engine` | | `writing-docs` | `loop-docs` |
  | `orchestrating-projects` | `loop-orchestrate` | | `researching-topics` | `loop-research` |
  | `reviewing-code` | `loop-review` | | `finding-frameworks` | `loop-scout` |
  | `auditing-changes` | `loop-audit` | | `engineering-harnesses` | `loop-harness` |
  | `diagnosing-bugs` | `loop-debug` | | `automating-improvements` | `loop-autopilot` |
  | `designing-systems` | `loop-design` | | `writing-tests` | `loop-test` |

- Every `SKILL.md` `name:` field, all cross-skill references and relative reference paths, the README skill map + dependency/autonomy diagrams, and `INSTALL.md` updated to the new names. Generic uses of "workflow" (the Workflow tool, `*.workflow.js` templates) are unchanged. `plugin.json` keywords are unchanged (they remain search terms, not skill names).
- Bumped plugin + marketplace version to `0.4.0`.

### Fixed
- Corrected a pre-existing broken reference link in `loop-orchestrate/references/standards.md` (`../` → `../../` for the OWASP/CWE cross-reference).

## [0.3.0] — 2026-07-04

The autonomy ladder's two open rungs — **SUSTAIN** and **SCALE** — land in `automating-improvements`. Default behavior is unchanged: the loop is still propose-only; SCALE ships off by default.

### Added
- **SUSTAIN — AP6 "Gamed Loop"** (verification runs but is fooled): `references/verifier-integrity.md` — three structural guards (impossible-test canary, protected-path diff-integrity, sampled cross-judge) — and `references/held-out-eval.md` — the out-of-band detector: a frozen task suite with hidden deterministic oracles whose rising false-accept rate is the meta-overfit alarm. Runnable gates: `templates/verifier-canary.workflow.js` (in-band, pre-Propose, hard stop) and `templates/held-out-eval.workflow.js` (deploys as a third companion Routine).
- **SCALE — autonomous delivery, off by default**: `references/deployment.md` §"Advanced: autonomous delivery" — preconditions on every SUSTAIN signal, a hard NEVER-list, merge-behind-canary, agent-driven rollback, a rollback-rate + held-out tripwire that self-revokes autonomy back to propose-only, and a pinned autonomy-state audit issue — plus the control-flow skeleton `templates/canary-merge.workflow.js`.
- **README "The autonomy ladder"** — OBSERVE → VERIFY → SUSTAIN → SCALE as a named progression, with the degradation guarantee: any alarm drops the loop one rung; the floor is always propose-only.

### Changed
- `automating-improvements/SKILL.md` gains §7 (keeping the loop honest over time) and non-negotiable safety rule 6 (the held-out suite is never visible to the Act stage); `references/anti-patterns.md` gains the AP6 row.
- Bumped plugin + marketplace version to `0.3.0`.

## [0.2.0] — 2026-07-04

Professionalization and standards-depth pass. No behavior change to any workflow template.

### Added
- **`references/standards.md` for every skill** — each names, version-pins, and maps the authoritative standards it applies. Highlights: NIST SSDF / SLSA / SBOM / MITRE ATT&CK and compliance cross-maps (`reviewing-code`); arc42 / ISO-25010 / Google SRE / CAP-PACELC (`designing-systems`); CRAAP / SIFT / PRISMA / GRADE (`researching-topics`); ISO 31000 / DORA change-failure-rate (`auditing-changes`); 5 Whys / ODC / OpenTelemetry (`diagnosing-bugs`); WSJF / RICE / critical-path (`orchestrating-projects`); POLP / CIS / OWASP CI-CD Top 10 (`engineering-harnesses`); mutation testing / Pact / FIRST (`writing-tests`); Google style guide / CommonMark / Conventional Commits / MADR (`writing-docs`); SPDX / OpenSSF Scorecard (`finding-frameworks`); DORA-SPACE / trunk-based (`automating-improvements`).
- **Professional `README.md`** — tagline, badges, table of contents, "Why", quickstart, a "how the skills compose" Mermaid diagram, and an architecture & philosophy section.
- **`CONTRIBUTING.md`** — SKILL.md conventions, the workflow-template runtime rules (H10), the `standards.md` convention, and the validation checklist.
- **`CHANGELOG.md`** — this file.

### Fixed
- Corrected a stale "all four skills" reference in `INSTALL.md` (the plugin ships twelve).

### Changed
- Bumped plugin + marketplace version to `0.2.0`.

## [0.1.0] — 2026-07-02

Initial release: the 12-skill TheLoopSkill plugin, built and merged across PRs #1–#4.

### Added
- **`workflow`** — the multi-agent orchestration engine: pipeline / parallel / loop-until-dry / loop-until-budget templates, the Harness (H1–H12) and Loop (L1–L8) engineering policies, and the pluggable **AIDLC** lifecycle framework.
- **Domain skills** — `reviewing-code`, `designing-systems`, `orchestrating-projects`, `researching-topics`, `auditing-changes`, `writing-tests`, `diagnosing-bugs`, `writing-docs`, `finding-frameworks`, and `engineering-harnesses` (Claude Code harness scaffolds: permissions, hooks, MCP, automation loops).
- **`automating-improvements`** — a propose-only autonomous engineering loop that composes the other skills, plus the **credit-horizon** self-learning extension (per-kind trust ledger), an anti-patterns checklist, and a comprehension-rot digest.
- **Plugin packaging** — `.claude-plugin/plugin.json` + `marketplace.json`, web enablement via `.claude/settings.json`, the MIT `LICENSE`, and `INSTALL.md` covering local, web, and marketplace install paths.

[Unreleased]: https://github.com/santapong/TheLoopSkill/compare/v0.2.0...HEAD
[0.2.0]: https://github.com/santapong/TheLoopSkill/compare/v0.1.0...v0.2.0
[0.1.0]: https://github.com/santapong/TheLoopSkill/releases/tag/v0.1.0
