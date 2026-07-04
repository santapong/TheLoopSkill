# Contributing to TheLoopSkill

Thanks for helping improve TheLoopSkill. This repo is a Claude Code plugin of skills, so "contributing" usually means **adding or improving a skill**, **deepening a skill's reference standards**, or **adding a lifecycle framework**. This guide covers the conventions and how to validate a change.

## Repository shape

Every skill lives in `.claude/skills/<name>/` and follows one shape:

```
<name>/
├── SKILL.md              # thin router — what the skill does + when, in <~400 lines
├── references/           # deep knowledge, loaded on demand (progressive disclosure)
│   ├── <topic>.md
│   └── standards.md      # the authoritative standards this skill applies (see below)
└── templates/            # runnable Workflow scripts (*.workflow.js) and/or scaffolds
```

The plugin is wired via `.claude-plugin/plugin.json` (which points at `./.claude/skills`) and published through `.claude-plugin/marketplace.json`. New skills under `.claude/skills/` are auto-discovered — no manifest edit needed.

## Authoring a `SKILL.md`

- **Frontmatter is required**: a YAML block with `name` and `description`.
  - `name` — lowercase letters, numbers, hyphens; gerund style matching the directory (`loop-review`, `loop-test`).
  - `description` — third person, **≤1024 characters**, stating *what it does* AND *when to use it* (the trigger phrases Claude matches on). No first/second person.
- **Keep the body a thin router** (aim under ~400 lines). Push depth into `references/` files and load them on demand — this keeps token cost low, since only `SKILL.md` stays in context.
- **Be prescriptive**: sane defaults with named escape hatches, not a neutral glossary. No leftover TODO/placeholder text.
- End with a **Reference files** list that names every file under `references/` and `templates/`, so nothing is a dangling link.

## The `references/standards.md` convention

Each skill carries a `references/standards.md` that grounds it in authoritative standards. For every standard it lists:

1. **Name** the framework/standard and its issuing body.
2. **Pin** the current version/edition (with an "edition discipline" note — standards get revised).
3. **Map** it to *this skill's* workflow — how a practitioner applies it here, not a generic description.

`loop-review/references/owasp-cwe.md` is the exemplar (OWASP Top 10 2021, CWE Top 25, ASVS 5.0, CVSS v4). Match that rigor.

## Authoring a workflow template (`*.workflow.js`)

Workflow templates run under the Workflow tool's runtime, which has hard constraints (harness policy **H10**):

- `export const meta = {...}` first, a **pure literal** (no variables/calls/spreads/interpolation); required `name`, `description`; `phases` titles must match the `phase:` strings used.
- **Plain JavaScript**, not TypeScript.
- **No `Date.now()`, `Math.random()`, or argless `new Date()`** — they throw (they'd break resume). Pass timestamps in via `args`; vary prompts by index for diversity.
- Normalize input: `const input = typeof args === 'string' ? JSON.parse(args) : args`.
- `.filter(Boolean)` every `parallel()` result (dead agents resolve to `null`); pass a `schema` to every consumed `agent()`; `log()` progress.
- Prefer `pipeline()`; use a `parallel()` barrier only when a stage needs all prior results (dedup/merge, early-exit). See the harness & loop policies under `.claude/skills/loop-engine/references/`.

## Adding a lifecycle framework

Copy `.claude/skills/loop-engine/frameworks/_TEMPLATE.md` to `frameworks/<Name>.md`, fill in the phases (each with a human gate where appropriate), and it's usable via `/loop-engine <task> --framework <Name>`. No skill changes needed.

## Validating your change

Before opening a PR, run:

```bash
# Plugin + marketplace manifests are valid (fails on unknown fields in --strict)
claude plugin validate . --strict

# Every workflow template parses (wrap as the runtime does: async body + top-level return)
for f in $(find .claude/skills -name '*.workflow.js'); do
  { echo 'async function wf(agent,parallel,pipeline,phase,log,args,budget,workflow){';
    sed 's/^export const meta/const meta/' "$f"; echo '}'; } | node --check /dev/stdin && echo "OK $f"
done

# No banned clock/random calls in template code (should print nothing)
grep -rnE 'Date\.now\(|Math\.random\(|new Date\(\)' .claude/skills --include='*.workflow.js' | grep -vE '//'

# Hook scripts (if you touched them) parse and behave
bash -n .claude/skills/loop-harness/templates/hooks/*.sh
```

Also confirm each new `SKILL.md` has valid `name` + `description` frontmatter, and that any file you reference in a `SKILL.md` actually exists.

## Pull requests

- Branch from `main`; keep PRs focused (one skill or one coherent change).
- Open as a **draft** until you've run the checks above.
- Describe what changed and how you verified it. If you added a workflow template, note that it passes `node --check` and, ideally, a bounded live run.

By contributing, you agree your contributions are licensed under the [MIT License](LICENSE).
