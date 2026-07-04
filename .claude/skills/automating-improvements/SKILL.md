---
name: automating-improvements
description: Run an autonomous engineering loop that improves this project — read feedback (GitHub issues, PR comments, CI failures), act on it as draft pull requests with tests, and when there is no feedback, research improvements (market trends, papers, tech-debt scan) and propose them. Use when the user wants to automate project maintenance, set up a self-improving or autonomous engineering loop, continuously triage and act on issues/PRs, or have Claude propose improvements on a schedule. Composes the plugin's other skills; proposes via draft PR + comment and never merges.
---

# Automating Improvements

You are about to run a project's **autonomous engineering loop**: read what needs doing, do it on a branch, and propose it — on repeat, unattended if configured. The loop is **propose-only**. It opens draft pull requests with a test and a risk memo and comments the result; **it never merges, and never pushes to `main`.** A human always approves.

This is a **composition layer over the `workflow` skill, not a new engine** (same pattern as `orchestrating-projects`). Every stage below is an existing skill invoked inside a budget-guarded loop.

**Because an unattended runner (a Cloud Routine) has no per-run approval prompt, the guardrails are structural, not interactive: draft-PR-on-a-`claude/`-branch, an explicit never-merge rule, budget/round caps, and dedup against what already exists.** Those are non-negotiable — see §6.

## 1. The loop

Each round: **intake → triage → act → verify → propose**; when intake is empty, **research/scan** for improvements instead. Full design in **`references/loop-design.md`**.

- Guarded like every loop in this plugin: a **budget floor** (loop-policy L2), a **dry counter** (L1: stop after K idle rounds), and a **hard round cap** (L4) — `while (budget.total && budget.remaining() > FLOOR && round < MAX_ROUNDS && dry < K)`.
- **Converges** by deduping every candidate against everything already *seen* **and** against already-open issues/PRs (L3), so it never re-proposes the same thing.

## 2. Intake — read the feedback

Gather actionable items from the four sources, using the GitHub tools listed in **`references/feedback-intake.md`**:

- **Issues** (polled): `search_issues` / `list_issues` for new or updated open issues.
- **PR comments + CI**: `pull_request_read` with `get_comments` / `get_review_comments`, and `subscribe_pr_activity` for CI/comment events.
- **Idle → research**: when nothing is pending, scan market trends and research papers with `researching-topics`.
- **Tech-debt scan**: proactively surface candidates with `reviewing-code`, `auditing-changes`, and `finding-frameworks`.

Dedup intake against open issues/PRs before acting — an item already tracked is not fresh work.

## 3. Act — do the work (compose skills)

Route each item to the skill that owns that job, then run it as an AIDLC Construction pass (design → implement → test):

- A bug / failing test / red CI → **`diagnosing-bugs`** (reproduce → root-cause → fix + regression test).
- A feature or change → **`designing-systems`** (minimal design + ADR) → implement.
- Before building anything non-trivial → **`finding-frameworks`** (don't reinvent a solved problem).
- Every change ships a test via **`writing-tests`** (fails before, passes after) and doc updates via **`writing-docs`**.

Work on a `claude/`-prefixed branch only.

## 4. Verify — prove it before proposing

- Adversarially self-review the diff with **`reviewing-code`** (fix confirmed findings; feed anything larger back as a new item).
- Produce the impact/risk memo with **`auditing-changes`** — this becomes the PR body.
- If verification says the change isn't safe or clear, **don't propose it** — drop it or open an issue describing the blocker instead.

## 5. Propose — the human gate

- Open a **draft** PR from the `claude/` branch with the `auditing-changes` risk memo as the body, then post a **comment** summarizing what changed and why (this is also how the human is notified — GitHub emails subscribers automatically; see `references/deployment.md`).
- **Label the PR** `automated` and `kind:<kind>` (the item's intake kind), or embed a `<!-- credit-kind: <kind> -->` marker in the body. The credit ledger keys off these to learn which kinds get merged (`references/credit-horizon.md`); the comprehension digest filters on `automated`.
- Stop there. **Never** call merge, never push to `main`, never mark ready-for-review without a human. This is harness policy H11 (one workflow per human gate).

## 6. Run it — supervised or unattended

- **A single supervised pass**: run **`templates/improvement-loop.workflow.js`** (defaults to `mode: "dry"` — produces proposal objects, opens nothing). Flip to live only when you want it to actually open draft PRs.
- **Before deploying unattended, run every row of `references/anti-patterns.md` against the current design** — it's the pre-flight checklist for the five ways an autonomous loop degrades.
- **Unattended**: deploy as a **Cloud Routine** using **`templates/routine-prompt.md`**, on a schedule plus a `pull_request` trigger. Full setup, safety scopes, and the issue-polling caveat (Routines don't trigger on issues) are in **`references/deployment.md`**, which builds on the `engineering-harnesses` skill's `automation-loops.md`.
- **Two companion Routines** run separately (both in `references/deployment.md`): the **credit-ledger reconcile** (`templates/credit-ledger.workflow.js`, daily) so the loop learns which proposal kinds get merged, and the **comprehension digest** (`templates/comprehension-digest.routine.md`, weekly) so a human actually reads what shipped.

### Non-negotiable safety rules

1. **Never merge and never push to `main`.** Propose via draft PR only.
2. **Push only to `claude/`-prefixed branches** (leave "unrestricted branch pushes" off in Routines).
3. **Cap the loop**: budget floor + dry counter + `MAX_ROUNDS`. No unbounded runs.
4. **Dedup** against open issues/PRs so it never spams duplicate proposals.
5. **Keep the network Trusted** and connectors minimal — the runner has no approval prompt.

## Reference files

- `references/loop-design.md` — the intake→act→verify→propose loop, guards, and convergence
- `references/feedback-intake.md` — the four sources and the exact GitHub tools; dedup
- `references/deployment.md` — running it unattended (Cloud Routine / Action), safety scopes, notification, and the two companion Routines
- `references/anti-patterns.md` — the five ways an autonomous loop degrades (AP1–AP5), mapped to this loop's guards; the pre-deploy checklist
- `references/comprehension-rot.md` — the one cost with no structural guard, and the forced random-sample digest that makes it visible
- `references/credit-horizon.md` — closing the outcome-feedback gap: the trust ledger design and the three-knob mapping
- `templates/improvement-loop.workflow.js` — the budget-guarded loop skeleton (dry by default)
- `templates/routine-prompt.md` — copy-paste prompt for a Cloud Routine / `.claude/loop.md`
- `templates/credit-ledger.workflow.js` — the ledger reconcile pass (deploy as its own daily Routine)
- `templates/comprehension-digest.routine.md` — the weekly comprehension-check digest prompt
