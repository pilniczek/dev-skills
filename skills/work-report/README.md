# work-report

A Claude skill that writes a standardized work report, `WORK-REPORT.md` at your working tree's root, summarizing what an agent did. A reviewer or a fresh session continuing the work reads it instead of re-deriving intent from the raw diff. That's the whole point: a concise report is a faster substrate for the next step than studying the changes.

There's one report per working tree. Each run reads the existing one and then overwrites it, so it always reflects the current state rather than a pile of fragments, and a job resumed across three days still leaves a reviewer exactly one report to read. It works in any git repo: a worktree, the main checkout, or a plain clone with no worktrees at all.

The report doesn't restate the git diff. Commits and changed files come from git or the PR; what the diff can't show is the intent, the reasoning, what was verified, and what's left over. It references artifacts (PRs, ADRs, file paths) instead of duplicating them, and it redacts secrets: any API key, password, token, credential, or PII is noted by name and location, never by value.

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

## The six sections

**Goal**, **Problem → Solution**, **Verification**, **Follow-ups**, **Suggested skills**, and **Abandoned approaches** (omitted if none). What belongs in each and how it's written lives in [Step 4 of SKILL.md](SKILL.md#step-4--compose-the-report), the single source of truth.

## Auto-activation

Claude raises this skill when you signal work is finished: "done", "ready to commit", "wrapping up", or a handoff to another agent. It never writes silently. A proactive activation offers first and waits for your go-ahead; typing `/work-report` yourself is the go-ahead, so an explicit invocation skips the offer. See [the gate in SKILL.md](SKILL.md#never-write-without-being-asked).

## Where it lives, and git

`WORK-REPORT.md` stays a local working artifact so it never pollutes the branch diff. On its first run in a repo the skill checks `.gitignore` for `/WORK-REPORT.md` and offers to add it if it's missing. Declining doesn't block the report. To hand the report to a reviewer working from the PR rather than the filesystem, commit it deliberately with `git add -f WORK-REPORT.md`.

## Self-report

The author's account of intent and reasoning, pointing at the changes. Not an audit, and not a substitute for reading the diff when correctness matters.

## Contributing

See [CONTRIBUTING.md](https://github.com/pilniczek/dev-skills/blob/master/CONTRIBUTING.md) for local setup and the pre-release security scan workflow.
