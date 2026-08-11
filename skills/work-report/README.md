# work-report

A Claude skill that writes a standardized **work report** — `WORK-REPORT.md` at your working tree's root — summarizing what an agent did. It is deterministic, read-prior-then-overwrite — which is the iterative-checkpointing case.

When an agent declares work "done," `/work-report` leaves a real file on disk that another agent (a reviewer, or a fresh session continuing the work) can read *instead of* re-deriving intent from the raw diff. The point is **efficiency**: a concise report is a faster substrate for the next step than studying the changes.

It works **anywhere** - a git worktree, the main checkout, or any git repo. It does **not** require a worktree, so it's useful on its own or alongside any worktree-based workflow.

The report references artifacts (PRs, diffs, ADRs) rather than duplicating them - it deliberately does **not** restate the git diff (commits, changed files); a reader gets those from git/the PR. Its value is the intent, reasoning, verification, and next steps the diff **cannot** show. Each run reads the existing report first and builds on it. Secrets are **redacted** - any API key, password, token, credential, or PII is noted by name and location, never reproduced by value.

---

## Install

As a Claude Code plugin (from the [dev-skills](https://github.com/pilniczek/dev-skills) marketplace):

```text
/plugin marketplace add pilniczek/dev-skills
/plugin install work-report@dev-skills
```

Or vendor it into your repo with skills.sh:

```bash
npx skills add https://github.com/pilniczek/dev-skills --skill work-report
```

[skills.sh/pilniczek/dev-skills](https://skills.sh/pilniczek/dev-skills/work-report)

---

## One living report per working tree

There is **one report per working tree** - `WORK-REPORT.md` at its root (`git rev-parse --show-toplevel`) - and each run **overwrites** it, so it always reflects the current state, never a pile of fragments:

- **Resumed over time** (day 1 unfinished → day 2 → day 3): each run replaces the last, so a reviewer at the end reads exactly **one** report. No cleanup step.
- **Resuming and reviewing are just *reading*** it - a session picking the work back up reads it to regain context; a reviewer reads it to assess. Neither needs a command; only `/work-report` writes.

---

## The six sections

The report has six sections — **Goal**, **Problem → Solution**, **Verification**, **Follow-ups**, **Suggested skills**, and **Abandoned approaches** (omitted if none). The exact contract for each — what belongs in it and how it's written — lives in [Step 4 of SKILL.md](SKILL.md#step-4--compose-the-report), which is the single source of truth; this README only names them.

---

## Auto-activation

Claude is instructed to raise this skill proactively when you signal work is finished — "done", "ready to commit", "wrapping up", or a handoff to another agent. It **never writes silently**: a proactive activation offers first and waits for your go-ahead. Typing `/work-report` yourself *is* the go-ahead, so an explicit invocation skips the offer. See [the gate in SKILL.md](SKILL.md#never-write-without-being-asked).

---

## Where it lives, and git

`WORK-REPORT.md` is meant to stay a **local working artifact** so it never pollutes the branch diff. On its first run in a repo the skill checks `.gitignore` for `/WORK-REPORT.md` and, if absent, offers to add it — declining doesn't block the report. To hand the report to a reviewer working from the PR (not the working-tree filesystem), commit it deliberately with `git add -f WORK-REPORT.md`.

---

## It is a self-report, not proof

The report is the author's account of intent and reasoning, built for **efficiency of the next step** - a fast, trustworthy-enough summary that points *at* the changes, not an audit and not a substitute for reviewing the diff when correctness matters.

---

## Contributing

See [CONTRIBUTING.md](../../CONTRIBUTING.md) for local setup and the pre-release security scan workflow.
