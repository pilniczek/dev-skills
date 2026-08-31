---
name: work-report
description: >
  Writes or overwrites WORK-REPORT.md at the working tree root - session self-report (goal,
  problem/solution, follow-ups, suggested skills, abandoned) a reviewer or later session reads
  instead of re-deriving intent from the diff. Use for "work report", "checkpoint this",
  "hand off", "write up what we did". Offer proactively - asking first - when work is finished:
  "done", "ready to commit", "wrapping up", handoff to another agent.
allowed-tools: Bash(git rev-parse:*), Bash(git branch:*), Bash(ls:*), Bash(grep:*), Bash(head:*), Bash(tail:*), Bash(sed:*), Read, Write, Edit
---

Write the **work report**, `WORK-REPORT.md` at the working tree root: what a reviewer or fresh session reads *instead
of* re-deriving intent from the raw diff. One per tree, **overwritten** each run, so it is current state, not fragments.
Any repo, git or not; a worktree is not required. Run **every time** work is checkpointed or handed off, whatever its
shape (commits, loose edits, cross-repo changes, editor settings).

Steps in order. STOP only if the path cannot be **written**; missing `git`, a directory outside any repo, and a quiet
session are **not** failures - still write a valid report.

## Never write without being asked

Self-activated (the user signalled work was finished rather than naming the report): **offer in one sentence and stop**,
wait for a go-ahead before Step 1. An explicit invocation (`/work-report`, "write the work report") *is* the go-ahead.

## Scope of "the work" = this chat session

A *self-report* of what you did this session: do **not** infer the change set from `git diff`/`git log`. A free-text note
passed with the invocation **emphasizes** (that topic leads each section), never **restricts**.

## Altitude - keep every entry the same size

Goal and every nested bullet 1-2 sentences; every list item one sentence plus an optional parenthetical. **No
paragraphs anywhere.**

## Step 1 - Locate the report

`git rev-parse --show-toplevel` gives the root; the report is `<root>/WORK-REPORT.md`. Outside a git repo, use the
current directory.

## Step 2 - Read the existing report and build on it

Carry the Abandoned list forward, keep resolved decisions, never regenerate from scratch. Disposition **every carried
follow-up** - dropping what fails this rule is what bounds the report across a week:

| Carried follow-up | Where it goes |
| ----------------- | ------------- |
| Done, and a course changed after a failed attempt, a fork taken, or a check the diff cannot show (pipeline run inspected, email sent, console flag toggled) | *Problem → Solution* |
| Done, and what it took is recoverable from the diff | dropped, no trace |
| Deliberately not done, rejected | *Abandoned*, tagged `[considered]` |
| Still open | kept verbatim |
| Obsolete, the goal moved | dropped, no trace |

## Step 3 - Offer the gitignore entry (first run in a repo only)

The report is a **local working artifact**, kept out of the branch diff. Outside a git repo, skip to Step 4. Skip it too
when Step 2 found an existing report - that is the "first run in this repo" test, and it is what stops a declined offer
being re-asked. Otherwise check the root `.gitignore` (`grep -qxF '/WORK-REPORT.md' <root>/.gitignore`, the Step 1 root,
not the current directory). Present: say nothing. Absent: say why it is normally ignored and ask whether to add it.
**Continue to Step 4 either way** - a "no" never blocks the report. On a yes, append to `<root>/.gitignore` (preserving a
missing trailing newline):

```text
# Work report (work-report skill) - git add -f to commit it
/WORK-REPORT.md
```

The **leading slash is required**: it anchors to the tree root, so a same-named file deeper in the tree is not ignored;
per-tree resolution still covers every worktree.

## Step 4 - Compose the report

The value is what the diff **cannot** show - intent, reasoning, verification, next steps - so **never restate the diff**
(commits, changed files). **Reference** artifacts (PR/issue URLs, file paths, ADRs), never paste them. **Redact
secrets**: never reproduce the value of an API key, password, token, credential, or PII; note only that it exists and
where.

**Title block** - title exactly `# Work report`, then three lines naming the session that **last wrote** the report, one
fact per line, never merged:

```text
# Work report

- **branch:** <git branch --show-current, `(detached)` on a detached head, `(no git)` outside a repo or with git unavailable>
- **session title:** <session title, `(untitled)` if the transcript carries none, `unknown` if the session id is unset>
- **session id:** <$CLAUDE_CODE_SESSION_ID, or `unknown` if unset>
```

*Session title* is the mutable name the VS Code plugin shows and searches by; *session id* is the stable UUID
tie-breaker. Read the title - last title-bearing line wins, so a `/rename`d `customTitle` beats the auto-`aiTitle`:

```bash
TRANSCRIPT=$(ls -t "$HOME"/.claude/projects/*/"$CLAUDE_CODE_SESSION_ID".jsonl 2>/dev/null | head -1)
grep -hoE '"(customTitle|aiTitle)":"[^"]*"' "$TRANSCRIPT" | tail -1 | sed -E 's/^"[^"]*":"(.*)"$/\1/'
```

It is a snapshot and may drift from what the plugin later shows; `/rename` locks a stable one. An embedded `"` is a tolerated edge case.

Then these five `##` headings, in order:

- **Goal** - what this work set out to do.
- **Problem → Solution** - every problem you hit and how it was ultimately resolved, as a numbered list. Log an entry
  whenever **your first attempt failed and you changed course**, **a real fork (≥2 viable options) was chosen**, or **a
  follow-up the report already listed was finished** - whoever closed it, this session included - and what closing it
  took is **not recoverable from the diff**, or it was itself a fork taken. Record the problem and the approach that
  **worked** (or the fork taken and why); a finished follow-up failing that test just disappears from *Follow-ups*,
  leaving no entry. **Verification belongs here**: anything you exercised that the diff, the PR and CI **cannot** show
  (retry path clicked through with the proxy down, a pipeline run inspected); fold it into the **Solution** it verifies.
  A green test run is **not** an entry. Attempts abandoned along the way go under *Abandoned*, not here. Each entry is a
  number with two nested bullets:

  ```text
  1.
      - **Problem:** description
      - **Solution:** description
  ```

- **Follow-ups** - one flat list of what the next session or reviewer picks up, each **tagged**:
  - `[question]` - an unresolved decision or unknown, phrased as a question.
  - `[action]` - concrete work, phrased as an imperative. **Verification counts as work**: something believed to work
    but never exercised is an `[action]` naming the check that would settle it. An entry with **nobody lined up to do
    it** is still valid - a follow-up states what remains, not who owns it.

  Three kinds are **never** follow-ups:

  - **A gap already decided against** (accepted risk, platform out of scope) - the decision is made, so it goes to
    *Abandoned* as `[considered]`.
  - **Shipping** (commit, push, PR, merge, tag, release) - the user's own move, even when the session ends uncommitted.
  - **Invoking a skill or slash-command** - a *means*, so it belongs to *Suggested skills*: state the outcome
    (`[action] Add regression tests for the retry path`), never the tool (`[action] Run /test-writer`).
- **Suggested skills** - the *means* for a follow-up that states only its outcome. List a skill **only if it advances an
  `[action]` item**, formatted `/skill - which follow-up it advances`. Mandatory; write "none" if nothing maps.
- **Abandoned** - what the next agent should **not** re-walk: dropped approaches, and follow-ups the report already
  listed, then rejected. Flat numbered list, one sentence plus the reason. Omit the section entirely if empty.

  ```text
  1. [tried] implemented or attempted, then reverted - why it was backed out
  2. [considered] evaluated and rejected without building it - why it was rejected
  ```

## Step 5 - Write the file and report back

Write to `<root>/WORK-REPORT.md`, then report the path plus a one-line summary. If it is gitignored (Step 3), note that
showing it to a reviewer on the PR takes a deliberate `git add -f WORK-REPORT.md`.
