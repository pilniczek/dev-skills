# AGENTS.md - `work-report`

Contract pins and vocabulary unique to this skill. The four pins every skill carries (path, frontmatter `name`, frontmatter `description`, README to SKILL.md) are in the root [AGENTS.md](../../AGENTS.md#contract-pins-every-skill-carries) and apply here too. Everything pinned below is breaking for consumers in the same way: be deliberate, and mention it in the commit.

## Source of truth

This folder is the canonical home of the work-report spec. It supersedes the `work-report/` tool in `pilniczek/worktree-toolbox`, which shipped the same behaviour as a slash command installed by a `curl … | sh` installer. That copy is retired; edits land here.

Two consequences of the port from command to skill, so nobody re-adds them:

- There is no installer. Distribution is skills.sh and the plugin marketplace, both of which vendor `SKILL.md` directly. The `WORKTREE_TOOLBOX_*` env overrides and the `curl`/`tar` requirements died with it.
- The `.gitignore` entry moved into the skill. With no installer to append `/WORK-REPORT.md`, [SKILL.md](SKILL.md) Step 3 checks for the line on first run and offers to add it. A decline must never block the report.

## Skill-specific pins

- Frontmatter `allowed-tools`: a pre-approval, not a restriction. It removes the permission prompt for the tools listed during the invoking turn, and the grant clears on the next user message. It cannot stop the model reaching for a tool that isn't listed; the restricting counterpart is `disallowed-tools`, which this skill does not use. Two consequences: every command the body tells the model to run must appear here or the run stalls on a prompt (Step 4's title lookup needs `head`, `tail`, and `sed` alongside `ls` and `grep`); and widening the list widens auto-approval, a security-surface change that shows up in the pre-release Snyk scan. The skill's safety comes from its offer-first gates, not from this field.
- `/WORK-REPORT.md`, anchored: the gitignore line the skill offers. The leading slash is load-bearing. A bare `WORK-REPORT.md` would match by basename at any depth.
- The offer-first guarantee: stated in the frontmatter `description`, in the SKILL.md body gate, and in the README's Auto-activation section. The triplication is intended, since it must hold at activation time, not only once the body is loaded. All three must agree that proactive activation offers and waits, and that an explicit invocation is itself the go-ahead.

## Vocabulary is deliberate

The report is read by other agents, so its shape is an interface. Every term below appears verbatim in [SKILL.md](SKILL.md) and in every generated report. Renaming one is a multi-file change, and the six section headings ship in the listed order. [Step 4](SKILL.md#step-4--compose-the-report) defines what belongs in each; this table pins the naming and the *Avoid* list.

| Term | What it names | Avoid |
| ---- | ------------- | ----- |
| **Work report** (a.k.a. completion report) | `WORK-REPORT.md` at a working tree's root. One per tree, overwritten in place, gitignored by default. Works in any repo, worktree or not. | summary, handoff doc, session log, journal |
| **Self-report** | The report's stance: what the author *did and intended*, scoped to the current chat session, never reconstructed from git history. Git stays the factual record of the diff. | audit, proof, reconstruction |
| **The work** | The subject a report summarizes: everything done in the current chat session, not the branch's whole history. | the diff, the changes, the branch |
| **Goal** | The section stating what the work set out to do. | objective, purpose |
| **Problem → Solution** | The section logging each problem and the approach that worked. | challenges, issues, notes |
| **Verification** | The section's two sub-lists, exactly `Checked:` and `Not verified:`. | testing, QA |
| **Follow-ups** | The section listing what the next session picks up, each entry tagged `[question]` or `[action]`. | open questions, next actions, TODOs |
| **Suggested skills** | The section naming skills the next agent should invoke. | recommendations, tooling |
| **Abandoned approaches** | The section recording what was not kept, each entry tagged `[tried]` or `[considered]`. | discarded, dead ends, rejected |
| **Session title** | The mutable name the VS Code plugin shows and searches by (`custom-title` if `/rename`d, else the latest `aiTitle`). | session name, summary |
| **Session id** | The session's UUID, the transcript filename under `~/.claude/projects/`. Stable and unique, but not searchable in the plugin; it is the tie-breaker. | session, chat id |

Two more fixed strings: the title, exactly `# Work report`, and the three identifier lines `branch:` / `session title:` / `session id:`, one fact per line, never merged.
