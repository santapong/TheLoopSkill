# TheLoopSkill

A Claude Code **plugin** — a set of skills for running real engineering work as multi-agent workflows. At its core is the `workflow` skill (pipeline/parallel/loop orchestration governed by explicit engineering policies and a pluggable lifecycle framework); domain skills build on top of it to cover the build lifecycle — design, project orchestration, code review, change audit, testing, debugging, documentation, and research.

## Skills

| Skill | Invoke | What it does |
|---|---|---|
| **workflow** | `/workflow <task> [--framework <name>] [--dry-run]` | Authors and executes a multi-agent Workflow script for a task — pipeline by default, earned parallel barriers, loops for unknown-size discovery — governed by the harness & loop engineering policies and a lifecycle framework (default AIDLC). |
| **reviewing-code** | `/reviewing-code <target>` | Security + code-quality review using OWASP Top 10, CWE Top 25, ASVS, and CVSS v4. LLM-native (zero required deps), finder-per-category → dedup → adversarial verify, reports HIGH/MEDIUM findings at confidence ≥0.8. |
| **designing-systems** | `/designing-systems <problem>` | Architecture & system design: pattern selection, API design, backend/data modeling, frontend performance, deployment strategy, and NFRs. Emits ADRs and C4 diagrams. |
| **orchestrating-projects** | `/orchestrating-projects <project>` | A planning layer on top of `workflow`: decomposes a project into a typed task DAG, chooses pipeline/parallel shapes, and routes the right Claude model + effort tier to each task ("right model for the right job"). |
| **researching-topics** | `/researching-topics <question>` | Multi-source research with adversarial fact-checking: multi-modal search fan-out → deep-read → refute-first verification → cited synthesis. Every reported claim carries a source and has survived a refutation attempt. |
| **auditing-changes** | `/auditing-changes <diff\|PR\|range>` | Change/impact audit that produces a *report*: classifies changes, traces blast radius, rates risk, checks test coverage, and delegates the security dimension to `reviewing-code`. Not a defect-list — that's `reviewing-code`. |
| **writing-tests** | `/writing-tests <target>` | Designs and writes tests (happy/edge/error/property), matches the repo's existing test stack, and verifies each test runs and fails for the right reason. |
| **diagnosing-bugs** | `/diagnosing-bugs <symptom>` | Hypothesis-driven debugging: reproduce → localize → root-cause → minimal fix → regression test, with a workflow template for parallel hypothesis elimination. |
| **writing-docs** | `/writing-docs <target>` | Writes and maintains docs (README, API reference, docstrings, guides, ADRs) using the Diátaxis doc-type model, verifying claims against the actual code. |

## The `workflow` skill

```
/workflow <task> [--framework <name>] [--dry-run]
```

Examples:

```
/workflow audit this codebase for security issues
/workflow build the CSV export feature --framework AIDLC
/workflow find all flaky tests --dry-run
```

- `--framework <name>` — which lifecycle framework governs the phases (default: `AIDLC`; resolves to `.claude/skills/workflow/frameworks/<name>.md`)
- `--dry-run` — author and show the workflow script without executing it

When invoked, the skill reads the two engineering policies and the chosen framework → maps the task onto the framework's phases → picks the orchestration shape (pipeline by default; barriers and loops only where the policies allow) → authors a script from the JS templates → runs it via the Workflow tool → reports results, pausing at the framework's human gates.

### What a run looks like

For `/workflow audit the docs for quality issues`, the skill authors a script from `templates/pipeline.workflow.js` — an Analyze stage fanning out one agent per file, feeding a Verify stage that adversarially checks each finding — and executes it. A real run of that shape produced:

```
Analyze  ✔ analyze:README.md            ✔ analyze:frameworks/AIDLC.md
Verify   ✔ verify:… ×4  (3 findings refuted, 1 confirmed)
→ { confirmed: [{ title: "No example of output…", location: "README.md:24", ... }] }
```

Confirmed findings come back as structured data, with everything the verifiers refuted filtered out. With `--dry-run` you get the authored script itself; when a framework phase ends at a human gate, the run stops and presents that phase's deliverable for approval.

## Installation

Three ways to use these skills — see **[INSTALL.md](INSTALL.md)** for full detail.

- **Local (project skills)** — the skills live in `.claude/skills/` and are auto-discovered in any Claude Code session opened in this repo. Copy an individual skill directory into another project's `.claude/skills/` to reuse it there.
- **Remote (Claude Code on the web)** — because web sessions only see committed project files, everything here is committed to the repo; open the repo on [code.claude.com](https://code.claude.com) and the skills are available. `.claude/settings.json` enables the plugin for web sessions.
- **Plugin (marketplace)** — `/plugin marketplace add santapong/TheLoopSkill` then `/plugin install theloopskill@theloopskill`.

## Repository layout

| Path | What it is |
|---|---|
| `.claude/skills/workflow/` | The workflow skill: `SKILL.md`, `references/` (harness & loop policies), `templates/` (pipeline, parallel, loop-until-dry, loop-until-budget), `frameworks/` (AIDLC + scaffold) |
| `.claude/skills/reviewing-code/` | Security + quality review skill: `SKILL.md`, `references/` (methodology, OWASP/CWE, playbooks, severity model, quality checks), `templates/security-review.workflow.js` |
| `.claude/skills/designing-systems/` | Architecture skill: `SKILL.md`, `references/` (patterns, API, backend, frontend, deployment, NFR), `templates/` (ADR + C4 diagrams) |
| `.claude/skills/orchestrating-projects/` | Project orchestration skill: `SKILL.md`, `references/` (model routing, task decomposition), `templates/project-plan.workflow.js` |
| `.claude/skills/researching-topics/` | Research skill: `SKILL.md`, `references/` (methodology, source evaluation), `templates/research.workflow.js` |
| `.claude/skills/auditing-changes/` | Change-audit skill: `SKILL.md`, `references/` (methodology, report template), `templates/change-audit.workflow.js` |
| `.claude/skills/writing-tests/` | Testing skill: `SKILL.md`, `references/` (test design, framework conventions), `templates/test-generation.workflow.js` |
| `.claude/skills/diagnosing-bugs/` | Debugging skill: `SKILL.md`, `references/` (methodology, hypothesis testing), `templates/bug-diagnosis.workflow.js` |
| `.claude/skills/writing-docs/` | Documentation skill: `SKILL.md`, `references/` (doc types, style), `templates/doc-generation.workflow.js` |
| `.claude-plugin/plugin.json` | Plugin manifest (references skills via `./.claude/skills`) |
| `.claude-plugin/marketplace.json` | Marketplace manifest listing the `theloopskill` plugin |
| `.claude/settings.json` | Enables the plugin/marketplace for Claude Code on the web |
| `INSTALL.md` | The three installation paths in detail |

## Engineering policies

The `workflow` skill's behavior is governed by two policy documents that every authored workflow must obey:

- **[Harness Engineering Policy](.claude/skills/workflow/references/harness-policy.md)** — orchestration design: pipeline vs. earned parallel barriers, adversarial/diverse-lens verification, budget & concurrency, isolation, phase discipline.
- **[Loop Engineering Policy](.claude/skills/workflow/references/loop-policy.md)** — iteration: loop-until-dry, budget-guarded loops, seen-set convergence, runaway prevention.

## Adding your own framework

Frameworks are pluggable markdown files. Copy `.claude/skills/workflow/frameworks/_TEMPLATE.md` to `frameworks/<Name>.md`, fill in the phases, and invoke with `--framework <Name>`. See [frameworks/README.md](.claude/skills/workflow/frameworks/README.md).
