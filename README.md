# TheLoopSkill

A Claude Code **workflow skill**: invoke it with a task, and Claude authors and executes a multi-agent Workflow script for it — pipeline/parallel/loop orchestration governed by explicit engineering policies and a pluggable lifecycle framework.

## Usage

In a Claude Code session in a repo containing this skill:

```
/workflow <task> [--framework <name>] [--dry-run]
```

Examples:

```
/workflow audit this codebase for security issues
/workflow build the CSV export feature --framework AIDLC
/workflow find all flaky tests --dry-run
```

- `--framework <name>` — which lifecycle framework governs the phases (default: `AIDLC`; resolves to `frameworks/<name>.md`)
- `--dry-run` — author and show the workflow script without executing it

When invoked, the skill: reads the two engineering policies and the chosen framework → maps the task onto the framework's phases → picks the orchestration shape (pipeline by default, barriers and loops only where the policies allow) → authors a script from the JS templates → runs it via the Workflow tool → reports results, pausing at the framework's human gates.

## What a run looks like

For `/workflow audit the docs for quality issues`, the skill authors a script from `templates/pipeline.workflow.js` — an Analyze stage fanning out one agent per file, feeding a Verify stage that adversarially checks each finding — and executes it. A real run of that shape produced:

```
Analyze  ✔ analyze:README.md            ✔ analyze:frameworks/AIDLC.md
Verify   ✔ verify:… ×4  (3 findings refuted, 1 confirmed)
→ { confirmed: [{ title: "No example of output…", location: "README.md:24", ... }] }
```

Confirmed findings come back as structured data (per the schemas in the script), with everything the verifiers refuted filtered out. With `--dry-run` you get the authored script itself, printed for review instead of executed. When a framework phase ends at a human gate, the run stops there and presents the phase's deliverable (e.g. AIDLC Inception returns the unit-of-work breakdown for approval before any code changes).

## Repository layout

| Path | What it is |
|---|---|
| `.claude/skills/workflow/SKILL.md` | Skill entry point — invocation contract and execution flow |
| `.claude/skills/workflow/references/harness-policy.md` | **Harness Engineering Policy** — orchestration design rules: pipeline vs parallel barriers, verification patterns, budget/concurrency, isolation |
| `.claude/skills/workflow/references/loop-policy.md` | **Loop Engineering Policy** — iteration rules: loop-until-dry, loop-until-budget, convergence/dedup, runaway prevention |
| `.claude/skills/workflow/templates/pipeline.workflow.js` | JS template: known items through independent stages (default shape) |
| `.claude/skills/workflow/templates/parallel.workflow.js` | JS template: fan-out finders + dedup barrier + verify |
| `.claude/skills/workflow/templates/loop-until-dry.workflow.js` | JS template: unknown-size discovery loop |
| `.claude/skills/workflow/templates/loop-until-budget.workflow.js` | JS template: budget-scaled loop |
| `.claude/skills/workflow/frameworks/AIDLC.md` | AI-Driven Development Life Cycle: Inception → Construction → Operation, human-gated |
| `.claude/skills/workflow/frameworks/_TEMPLATE.md` | Scaffold for authoring new frameworks |
| `.claude/skills/workflow/frameworks/README.md` | How framework resolution works and how to add one |

## Adding your own framework

Frameworks are pluggable markdown files. Copy `frameworks/_TEMPLATE.md` to `frameworks/<Name>.md`, fill in the phases (purpose, entry criteria, agent activities, orchestration hint, exit gate), and invoke with `--framework <Name>`. See [frameworks/README.md](.claude/skills/workflow/frameworks/README.md).

## Using the skill in other projects

Copy (or symlink) `.claude/skills/workflow/` into the target project's `.claude/skills/` directory. The skill is self-contained — policies, templates, and frameworks all live under its own folder.
