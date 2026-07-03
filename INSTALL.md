# Installing TheLoopSkill

TheLoopSkill ships nine Claude Code skills:

| Skill | What it does |
|---|---|
| `workflow` | Author & run multi-agent Workflow scripts (pipeline/parallel/loop) governed by engineering policies and a lifecycle framework |
| `reviewing-code` | Security + code-quality review (OWASP Top 10, CWE Top 25, ASVS; finder→verify orchestration) |
| `designing-systems` | System/architecture design: patterns, API, backend, frontend perf, deployment, NFRs, ADRs + C4 |
| `orchestrating-projects` | Project-manager planning layer: decompose into a task DAG and assign the right model+effort per task |
| `researching-topics` | Multi-source research with adversarial fact-checking: search fan-out → deep-read → refute-first verify → cited synthesis |
| `auditing-changes` | Change/impact audit → report: classify changes, trace blast radius, rate risk, check coverage (delegates security to reviewing-code) |
| `writing-tests` | Design + write tests matching the repo's stack; verify each runs and fails for the right reason |
| `diagnosing-bugs` | Hypothesis-driven debugging: reproduce → localize → root-cause → fix → regression test |
| `writing-docs` | Write + maintain docs (README, API, docstrings, ADRs) via the Diátaxis model, verified against code |

The **canonical location** is `.claude/skills/<name>/` — a single source of truth that works for all three install paths below. The plugin references these same files via the `skills` field in `.claude-plugin/plugin.json`, so nothing is duplicated.

---

## 1. Local (Claude Code CLI)

**Option A — use this repo directly.** Open a Claude Code session anywhere inside the repo. Project skills under `.claude/skills/` are auto-discovered; type `/workflow`, `/reviewing-code`, etc. No enable step.

**Option B — copy into another project.** Copy the skill folders you want into that project's `.claude/skills/`:

```bash
cp -r .claude/skills/workflow /path/to/your-project/.claude/skills/
```

Commit them so your team gets them too.

**Option C — make them personal (all your projects).** Copy into your user skills dir:

```bash
cp -r .claude/skills/workflow ~/.claude/skills/
```

---

## 2. Claude Code on the web (remote)

Web sessions start from a **fresh clone and see only committed project files** — so anything you want available must be committed to the repo.

- The skills under `.claude/skills/` are picked up automatically once committed. Nothing else is required to use them by name in a web session on this repo.
- To make the whole set available as an installable **plugin** in web sessions, this repo commits `.claude/settings.json` declaring the same-repo marketplace and enabling the plugin:

  ```json
  {
    "extraKnownMarketplaces": { "theloopskill": { "source": "./" } },
    "enabledPlugins": { "theloopskill@theloopskill": true }
  }
  ```

  > **Caveat:** the docs don't explicitly confirm that a *same-repo* marketplace auto-installs in a web session. If the plugin isn't active automatically, run the manual steps in section 3 once inside the session. Using the skills directly from `.claude/skills/` always works and needs none of this.

---

## 3. As a plugin (marketplace)

Install the bundle into any project or user scope via the plugin system.

```
# add this repo as a marketplace
/plugin marketplace add santapong/TheLoopSkill

# install the bundled plugin (all four skills)
/plugin install theloopskill@theloopskill
```

To test the marketplace from a local checkout instead of GitHub:

```
/plugin marketplace add ./
/plugin install theloopskill@theloopskill
```

Marketplace manifest lives at `.claude-plugin/marketplace.json`; the plugin manifest at `.claude-plugin/plugin.json` (its `skills` field points at `./.claude/skills`, so the plugin exposes the same files as the project skills — no duplication).

---

## Layout

```
TheLoopSkill/
├── .claude-plugin/
│   ├── plugin.json          # plugin manifest (skills → ./.claude/skills)
│   └── marketplace.json     # marketplace manifest (plugin source → ./)
├── .claude/
│   ├── settings.json        # extraKnownMarketplaces + enabledPlugins (web)
│   └── skills/
│       ├── workflow/
│       ├── reviewing-code/
│       ├── designing-systems/
│       ├── orchestrating-projects/
│       ├── researching-topics/
│       ├── auditing-changes/
│       ├── writing-tests/
│       ├── diagnosing-bugs/
│       └── writing-docs/
├── INSTALL.md
└── README.md
```
