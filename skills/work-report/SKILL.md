---
name: work-report
description: >
  Writes or overwrites WORK-REPORT.md at the working tree root - session self-report (goal,
  problem/solution, verification, follow-ups, suggested skills, abandoned approaches) a
  reviewer or later session reads instead of re-deriving intent from the diff. Use for
  "work report", "checkpoint this", "hand off", "write up what we did". Offer proactively -
  asking first - when work is finished: "done", "ready to commit", "wrapping up", handoff to
  another agent.
allowed-tools: Bash(git rev-parse:*), Bash(git branch:*), Bash(ls:*), Bash(grep:*), Bash(head:*), Bash(tail:*), Bash(sed:*), Read, Write, Edit
---

Write (or update) the **work report**: a standardized summary another agent (a reviewer, or a fresh session continuing
the work) reads *instead of* re-deriving intent from the raw diff. **One report per working tree** -
`WORK-REPORT.md` at its root - and each run **overwrites** it, so it is always current state, never a pile of
fragments. Works in any git repo; a worktree is not required. Run it **every time** work is checkpointed or handed
off, whatever its shape (commits, loose edits, cross-repo changes, editor settings).

## Never write without being asked

If you activated this skill yourself - the user signalled work was finished rather than naming the report -
**offer and stop**: one sentence saying a work report would capture this checkpoint, then wait for a go-ahead before
Step 1. An explicit invocation (`/work-report`, "write the work report") *is* the go-ahead - skip the offer, start at
Step 1.

## Scope of "the work" = this chat session

The report is a *self-report* of what you did this session, not a reconstruction from git history: do **not** infer
the change set from `git diff`/`git log`, report from what you know. An optional free-text note passed with the
invocation **emphasizes**, never **restricts** - the report always covers the session's work in full, and a note only
makes the named topic lead each section.

## Altitude — keep every entry the same size

Goal is 1–2 sentences. Every nested bullet (Problem, Solution, Checked, etc.) is 1–2 sentences. Every list item is one
sentence plus an optional parenthetical. **No paragraphs anywhere.**

Do the steps below in order; STOP and report only if a **required command errors** (working-tree root unresolvable, or
`git` unavailable). A quiet session with little to report is **not** a failure - still write a valid report.

## Step 1 — Locate the report

Get the working-tree root with `git rev-parse --show-toplevel`; the report is `<root>/WORK-REPORT.md`. Outside a git
repo, use the current directory.

## Step 2 — Read the existing report and build on it

Carry forward the Abandoned approaches list, close or keep Follow-ups, keep resolved decisions - never regenerate from
scratch and lose prior context. (Resuming and reviewing are the same read-only act: a session picking the work back up
reads this file to regain context; a reviewer reads it to assess. Only this skill writes.)

## Step 3 — Offer the gitignore entry (first run in a repo only)

The report is a **local working artifact**, meant to stay out of the branch diff. Check `.gitignore` at the
working-tree root for the exact line `/WORK-REPORT.md` (`grep -qxF '/WORK-REPORT.md' .gitignore`). Present: say nothing
and continue. Absent: say why the report is normally ignored and ask whether to add it. **Continue to Step 4 either
way** - a "no" never blocks the report. On a yes, append (preserving a missing trailing newline on the existing file):

```text
# Work report (work-report skill) — git add -f to commit it
/WORK-REPORT.md
```

The **leading slash is required**: it anchors the rule to the working tree's root, so a similarly-named file deeper in
the tree is not ignored. Ignores resolve per working tree, so the anchored form still covers every worktree. Ask once
per repo - never re-ask once the line is present or the user declined this session.

## Step 4 — Compose the report

Start with the title block, then the six sections below, in order. The report's value is what the diff **cannot** show
- intent, reasoning, verification, next steps - so **never restate the git diff** (commits, changed files); the reader
gets that from git/the PR. **Reference** artifacts (PR/issue URLs, file paths, ADRs) rather than pasting their
contents. **Redact secrets** - never reproduce the value of any API key, password, token, credential, or PII; note
only that it exists and where.

**Title block** - the title is exactly `# Work report`, followed by three identifier lines:

```text
# Work report

- **branch:** <git branch --show-current, or `(detached)` if none>
- **session title:** <session title, or `(untitled)` if none found>
- **session id:** <$CLAUDE_CODE_SESSION_ID, or `unknown` if unset>
```

These name the session that **last wrote** this report (the file may be resumed across days), on **separate lines** so
each label carries exactly one fact. *Session title* is the mutable name the VS Code plugin shows and searches its list
by; *session id* is the stable UUID confirming the exact match. Read the title as follows (glob locates the
transcript; the last title-bearing line wins, so a `/rename`d `custom-title` beats the auto-`aiTitle`):

```bash
TRANSCRIPT=$(ls -t "$HOME"/.claude/projects/*/"$CLAUDE_CODE_SESSION_ID".jsonl 2>/dev/null | head -1)
grep -hoE '"(customTitle|aiTitle)":"[^"]*"' "$TRANSCRIPT" | tail -1 | sed -E 's/^"[^"]*":"(.*)"$/\1/'
```

The title is a **point-in-time snapshot** and an auto-generated one may drift from what the plugin later shows - run
`/rename` to lock a stable one. Fallbacks: no title found → **session title:** `(untitled)`;
`$CLAUDE_CODE_SESSION_ID` unset → both lines `unknown`. (A title with an embedded `"` is a tolerated edge case - don't
over-engineer it.)

Then, each as an `##` (H2) heading:

- **Goal** - what this work set out to do.
- **Problem → Solution** - every problem you hit and how it was ultimately resolved, as a numbered list. Log an entry
  whenever **your first attempt failed and you changed course**, OR **a real fork (≥2 viable options) was chosen** -
  record the problem and the approach that **worked** (or the fork taken and why). Attempts abandoned along the way go
  under *Abandoned approaches*, not here. Each entry is a number with two nested bullets:

  ```text
  1.
      - **Problem:** description
      - **Solution:** description
  2.
      - **Problem:** description
      - **Solution:** description
  ```

- **Verification** - two bulleted sub-lists: **Checked:** what you exercised to believe it works, and **Not verified:**
  the gaps you are leaving open. Be honest about the second.
- **Follow-ups** - one flat list of what the next session or reviewer should pick up, each item **tagged** by kind:
  - `[question]` - an unresolved decision or unknown needing an answer (phrase as a question).
  - `[action]` - concrete work to be done (phrase as an imperative).
- **Suggested skills** - skills or slash-commands the next agent should invoke. List a skill **only if it advances an
  `[action]` item** in Follow-ups, formatted `/skill — which action it advances`. Mandatory; write "none" if nothing
  maps.
- **Abandoned approaches** - what you did **not** keep, so the next agent does not re-walk it. Flat numbered
  list, each entry **tagged** by kind, one sentence plus the reason:
  - `[tried]` - implemented or attempted, then reverted.
  - `[considered]` - evaluated and rejected without building it.

  ```text
  1. [tried] description — why it was backed out
  2. [considered] description — why it was rejected
  ```

  Omit this section entirely if there are none.

## Step 5 — Write the file

Write to `<root>/WORK-REPORT.md`.

## Step 6 — Report back

Report the path written plus a one-line summary. If `WORK-REPORT.md` is gitignored (Step 3), note that showing it to a
reviewer on the PR takes a deliberate `git add -f WORK-REPORT.md`.
