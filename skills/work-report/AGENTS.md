# AGENTS.md - `work-report`

Contract pins and vocabulary unique to this skill; the four every skill carries (path, frontmatter `name`, frontmatter `description`, README to SKILL.md) are in the root [AGENTS.md](../../AGENTS.md#contract-pins-every-skill-carries) and apply here too. Everything below breaks consumers the same way: be deliberate, mention it in the commit.

## Skill-specific pins

- Distribution vendors [SKILL.md](SKILL.md) directly - skills.sh and the plugin marketplace copy the folder as-is. Nothing outside it configures the skill: no install step, no env overrides, no files expected on disk. A pin that needs setup is not a pin.
- The `.gitignore` line lives in the skill: [SKILL.md](SKILL.md) Step 3 checks for it on first run in a repo and offers to add it. A decline must never block the report.
- Frontmatter `allowed-tools`: a pre-approval, not a restriction. It drops the permission prompt for listed tools during the invoking turn and clears on the next user message; it cannot stop the model reaching for an unlisted tool (that is `disallowed-tools`, unused here). So every command the body runs must be listed or the run stalls on a prompt (the title lookup needs `head`, `tail`, `sed` alongside `ls` and `grep`), and widening the list widens auto-approval, a security-surface change the pre-release Snyk scan flags. Safety comes from the offer-first gates, not this field.
- `/WORK-REPORT.md`, anchored: the gitignore line the skill offers. The leading slash is load-bearing - a bare `WORK-REPORT.md` would match by basename at any depth.
- Shipping steps (commit, push, PR, merge, tag, release) are never a *Follow-ups* item.
- Skill invocations live in *Suggested skills* alone, never in *Follow-ups*. The `only if it advances an [action] item` rule maps means to outcome: the follow-up states the outcome, the *Suggested skills* line names the tool reaching it.
- *Problem → Solution*'s entry rule governs what the report records anywhere: an entry is **a course changed after a first attempt failed**, **a real fork (≥2 viable options) taken**, or **a follow-up the report already listed, once finished and not recoverable from the diff** - whoever closed it, the writing session included. That same test dispositions every carried follow-up in [Step 2](SKILL.md#step-2---read-the-existing-report-and-build-on-it): a finished follow-up passing it becomes an entry, one failing it is already shown by the diff and leaves no trace - which bounds the report across a week of resumed sessions. Do not add a rule recording work for its own sake; that is the changelog git already is.
- Verification is content, not a section. A check already exercised is a *Problem → Solution* entry or a clause in the **Solution** it verifies; an unexercised one is an `[action]` follow-up naming it. Those two carry coverage in both directions.
- An `[action]` with nobody lined up to do it is valid: a follow-up earns its place by the fact it states, not by whether anyone owns it. Unclaimed is not rejected - a gap already decided against (accepted risk, platform out of scope) is *Abandoned* `[considered]`, which is also where [Step 2](SKILL.md#step-2---read-the-existing-report-and-build-on-it) sends a carried follow-up once it is rejected. Keeping a decided-against item in *Follow-ups* would re-offer work already refused.
- Why a purpose-built report rather than an existing tool: [docs/other-options.md](../../docs/other-options.md) records the options weighed and rejected.
- The offer-first guarantee is stated three times - frontmatter `description`, the SKILL.md body gate, the README's Auto-activation section - because it must hold at activation time, not only once the body is loaded. All three must agree: proactive activation offers and waits, an explicit invocation is itself the go-ahead.

## Vocabulary is deliberate

The report is read by other agents, so its shape is an interface. Every term below appears verbatim in [SKILL.md](SKILL.md); renaming one is a multi-file change. Five - **Goal**, **Problem → Solution**, **Follow-ups**, **Suggested skills**, **Abandoned** - are also emitted as the report's headings, in that order, *Abandoned* omitted when empty. The rest is vocabulary for talking about the report, not text it contains. [Step 4](SKILL.md#step-4---compose-the-report) defines what belongs in each; this table pins the naming and the *Avoid* list.

| Term | What it names | Avoid |
| ---- | ------------- | ----- |
| **Work report** (a.k.a. completion report) | `WORK-REPORT.md` at a working tree's root. One per tree, overwritten in place, gitignored by default. Any repo, worktree or not. | summary, handoff doc, session log, journal |
| **Self-report** | The report's stance: what the author *did and intended*, scoped to the current chat session, never reconstructed from git history. Git stays the factual record of the diff. | audit, proof, reconstruction |
| **The work** | Everything done in the current chat session, not the branch's whole history. | the diff, the changes, the branch |
| **Goal** | The section stating what the work set out to do. | objective, purpose |
| **Problem → Solution** | The section logging each problem and the approach that worked, plus any choice or check whose reasoning is not recoverable from the diff. | challenges, issues, notes, changelog, verification |
| **Follow-ups** | The section listing what the next session picks up, each entry tagged `[question]` or `[action]`. Excludes gaps already decided against, shipping steps and skill invocations. | open questions, next actions, TODOs |
| **Suggested skills** | The section naming skills the next agent should invoke. | recommendations, tooling |
| **Abandoned** | The section recording what the next agent should not re-walk - dropped approaches and rejected follow-ups - each entry tagged `[tried]` or `[considered]`. | abandoned approaches, discarded, dead ends, rejected |
| **Session title** | The mutable name the VS Code plugin shows and searches by (`customTitle` if `/rename`d, else the latest `aiTitle`). | session name, summary |
| **Session id** | The session's UUID, the transcript filename under `~/.claude/projects/`. Stable and unique but not searchable in the plugin; the tie-breaker. | session, chat id |

Two more fixed strings: the title, exactly `# Work report`, and the three identifier lines `branch:` / `session title:` / `session id:`, one fact per line, never merged. Readers grep for the fallback values, so those are fixed too: `(detached)`, `(no git)`, `(untitled)`, `unknown`.
