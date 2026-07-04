# Changelog

All notable changes to TheLoopSkill are documented here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

_Nothing yet._

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
