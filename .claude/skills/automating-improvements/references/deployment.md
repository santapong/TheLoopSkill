# Deployment

How to run the improvement loop unattended, and the safety scopes that keep "autonomous" from becoming "unsupervised writes to main." This builds on the mechanisms catalogued in the `engineering-harnesses` skill's `references/automation-loops.md` — read that for the full menu; this file is the recommended setup for *this* loop.

## Primary: a Cloud Routine

Cloud Routines run on Anthropic infrastructure with **your machine off** and **no per-run approval prompt** — the strongest autonomy, so the guardrails below are load-bearing.

1. **Create a scheduled Routine** (Claude Code on the web → Routines, or `/schedule`). Minimum interval is **1 hour**; hourly or nightly is plenty for a self-improving loop.
2. **Add a `pull_request` trigger** for reactivity (Routines fire on `pull_request` and `release` events). Each event starts a fresh session.
3. **Set the prompt** to `templates/routine-prompt.md`.
4. **Poll issues in the prompt** — Routines do **not** trigger on issues or issue-comments, so the prompt must list open issues each run (see `feedback-intake.md`). If you need instant issue-comment reactivity, add a **GitHub Action** on `issue_comment` (Actions support that event; Routines don't).

### Safety scopes (all of them)

- **Leave "unrestricted branch pushes" OFF** — Claude can then push only to `claude/`-prefixed branches. This alone prevents writes to `main`.
- **Keep network on Trusted** (the default). The loop needs only GitHub, which is allowlisted; don't widen it unless a step truly requires another host.
- **Minimize connectors** — include only the GitHub connector (and `alphaXiv` if you want paper research). Every connector is a capability the unattended run can use without asking.
- **The daily run cap** is a natural throttle; the budget floor and `MAX_ROUNDS` in the workflow are the per-run throttle.
- The prompt's **never-merge / propose-only** rule is the last line — because there is no approval prompt, it must be explicit and absolute.

Actions appear under **your** GitHub identity (commits, PRs, comments are yours).

## Alternatives

- **GitHub Action** driving Claude Code — the way to react to `issue_comment` / `issues` events (which Routines can't), and for team-wide, repo-native automation. Gate with the App's permissions and `--max-turns`.
- **Headless `claude -p`** on your own scheduler (cron/CI/systemd) — full control, but you host and schedule it.
- **Interactive `/loop`** with `templates/routine-prompt.md` copied to `.claude/loop.md` — for a **supervised** burst while you watch a session; not truly unattended (fires only while the session is running and idle).

Prefer the lightest mechanism that meets the need: a supervised `/loop` to try it out, a Cloud Routine once you trust it.

## Notification (this is your "email")

There is **no native email** in Claude Code. The loop notifies by **posting a GitHub comment** on the draft PR (or the source issue): GitHub's own notification system then emails you and any subscribers — zero infrastructure, works unattended. If you want real email in the message body, add an **MCP email connector** (e.g. Resend) in your Claude settings — connector traffic is Anthropic-routed, so it works even under the Trusted network policy — and have the propose step call it. Avoid `curl`-ing an email API from Bash: that needs a Custom/Full network policy plus an API key in a (web-visible) env var.

## First-run checklist

- [ ] Repo has the Claude GitHub App installed (not just web-setup).
- [ ] "Unrestricted branch pushes" is **off**; pushes go to `claude/*`.
- [ ] Network is **Trusted**; only GitHub (+ optional alphaXiv) connectors are enabled.
- [ ] Routine prompt is `routine-prompt.md`, with the never-merge rule intact.
- [ ] You've run `templates/improvement-loop.workflow.js` in `mode:"dry"` once and reviewed the proposals it would make.
- [ ] An opt-in label convention limits which issues the loop will act on.
