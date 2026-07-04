# Anti-Patterns

Five ways an autonomous loop degrades. Each one is a single missing move, not a vague
"quality" problem — naming which pattern you're looking at tells you which one thing
broke, not the whole system. Run this checklist before deploying as a Cloud Routine,
and re-run it any time the loop's behavior feels off or its guards change.

## AP1 — Nodding Loop (verification skipped)

**Symptom:** proposals get approved with rubber-stamp merges; plausible-looking mistakes
slip through and surface days later.

**Status here: guarded.** `safeToPropose` (`loop-design.md`, §Verify) gates every item —
a change that fails `reviewing-code`'s adversarial pass, or whose `auditing-changes`
memo flags unsafe or unclear, is dropped, not proposed.

**Not guardable by architecture:** a human merging drafts without reading the risk memo
is still a Nodding Loop, just a human-side one. See `comprehension-rot.md`.

## AP2 — Amnesiac Loop (persistence skipped)

**Symptom:** no cumulative progress — the same issue gets "fixed" repeatedly because
nothing remembers it was already handled.

**Status here: guarded.** The draft PR *is* the persistent record (`loop-design.md`,
§Propose). Within a run, the `seen` set additionally blocks re-proposing the same item.

## AP3 — Manual Loop (scheduling skipped)

**Symptom:** it isn't really a loop — it's a script someone has to remember to run.

**Status here: guarded, but read the default correctly.** A manual `Workflow()` call in
`mode: "dry"` (`SKILL.md` §6) *is* a Manual Loop, on purpose — that's the safe on-ramp
before you trust it unattended. It only becomes a real loop once deployed as a Cloud
Routine or Action (`deployment.md`). The anti-pattern isn't the dry-mode default — it's
still running it by hand three months after "Unattended" was documented and never
graduating.

## AP4 — Blind Loop (discovery skipped)

**Symptom:** a human still decides every session what's worth working on.

**Status here: guarded.** `feedback-intake.md`'s four sources (issues, PR comments, CI,
idle-research) are pulled automatically; nobody hand-picks the day's work.

## AP5 — Tangled Loop (handoff skipped)

**Symptom:** parallel agents collide on the same files or working directory.

**Status here: guarded (worktree isolation on the Act stage).** Harness policy H1 is
explicit — under `pipeline()`, "item A may be in stage 3 while item B is still in stage
1." That's exactly how `improvement-loop.workflow.js` runs its Act→Verify stages:
multiple items' Act stages can be genuinely concurrent, and each checks out a different
`claude/` branch. Two concurrent checkouts in one working directory would corrupt each
other.

**Fix (applied):** in **live** mode the Act-stage `agent()` runs with
`isolation: 'worktree'` — the runtime's built-in per-agent git worktree (harness policy
H7), so each concurrent Act stage mutates files in its own isolated tree and the
collision cannot happen. Dry mode is read-only and needs no isolation. If you replace
worktree isolation with something cheaper, cap in-flight Act calls to 1 (a `parallel()`
barrier or a semaphore) instead — never run concurrent file-mutating Act stages in a
shared directory.

## Using this file

Don't treat it as a one-time read. A fix for one pattern can quietly reopen another —
e.g. raising `MAX_ROUNDS` to push past a false AP3 diagnosis increases concurrent items,
which is only safe because AP5 is now guarded by worktree isolation; if you ever remove
that isolation, AP5 reopens. Re-check all five rows whenever
`improvement-loop.workflow.js` or its guards change, not just the row you think you
touched.

| Code | Name | Move missing | Status |
|---|---|---|---|
| AP1 | Nodding Loop | Verification | ✅ guarded |
| AP2 | Amnesiac Loop | Persistence | ✅ guarded |
| AP3 | Manual Loop | Scheduling | ✅ guarded (mind the dry-mode default) |
| AP4 | Blind Loop | Discovery | ✅ guarded |
| AP5 | Tangled Loop | Handoff | ✅ guarded (worktree isolation, live mode) |
