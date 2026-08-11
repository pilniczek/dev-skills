# AGENTS.md — `work-report`

Maintainer-only contract pins and vocabulary for this skill. A change touching any pin is **breaking** for the skill's consumers — be deliberate and mention it in the commit. For repo-wide conventions, see the root [AGENTS.md](../../AGENTS.md).

---

## Source of truth

**This folder is the canonical home of the work-report spec.** It supersedes the `work-report/` tool in `pilniczek/worktree-toolbox`, which shipped the same behaviour as a slash command installed by a `curl … | sh` installer. That copy is retired; edits land here. Why the report was built at all is recorded in [docs/other-options.md](../../docs/other-options.md).

Two consequences of the command → skill port, so nobody re-adds them:

- **There is no installer.** Distribution is skills.sh (`npx skills add`) and the plugin marketplace, both of which vendor `SKILL.md` directly. The `WORKTREE_TOOLBOX_*` env overrides and the `curl`/`tar` requirements died with it.
- **The `.gitignore` entry moved into the skill.** With no installer to append `/WORK-REPORT.md`, [SKILL.md](SKILL.md) Step 3 checks for the line on first run and offers to add it. A decline must never block the report.

---

## Source-of-truth pins

These paths and strings are part of the skill's **public contract** — moving or renaming them breaks downstream consumers whose [skills-lock.json](../../skills-lock.json) files reference this skill by hash.

- **The skill path `skills/work-report/SKILL.md`** — once anyone runs `npx skills add https://github.com/pilniczek/dev-skills --skill work-report`, their lockfile pins this path. The root [marketplace.json](../../.claude-plugin/marketplace.json) also references this folder (`skills: ["./skills/work-report"]`). Don't move it without updating both.
- **Frontmatter `name: work-report`** — determines the install directory in consumer repos, the slash command users type, and the plugin `name` in [marketplace.json](../../.claude-plugin/marketplace.json). Renaming = breaking change for every consumer. (Verified: `npx skills add` vendors the folder to `.agents/skills/work-report/` and symlinks `.claude/skills/work-report` at it, so the name drives both paths.)
- **Frontmatter `description`** — drives auto-activation in every consumer's repo. **Wording changes are behaviour changes**, not docs tweaks. Treat as API. The offer-first guarantee is stated here as well as in the body on purpose: it must hold at activation time, not only once the body is loaded.
- **Frontmatter `allowed-tools`** — the list of tools the skill may use **without a permission prompt** during the turn that invokes it; the grant clears on the next user message. It is a pre-approval, **not** a restriction: it cannot stop the model from reaching for a tool that isn't listed, it only removes the prompt for the ones that are. The restricting counterpart is `disallowed-tools`, which this skill does not use. Two consequences: every command the body tells the model to run should appear here, or the run stalls on a prompt (Step 4's title lookup needs `head`, `tail`, and `sed` alongside `ls` and `grep`); and widening the list widens auto-approval, which is a real security-surface change and shows up in the pre-release Snyk scan ([CONTRIBUTING.md](../../CONTRIBUTING.md#pre-release-security-scan)). The skill's safety comes from its offer-first gates, not from this field.
- **`/WORK-REPORT.md`, anchored** — the gitignore line the skill offers. The leading slash is load-bearing: it scopes the rule to the working tree's root. Emitting a bare `WORK-REPORT.md` would match by basename at any depth.

### Output-format strings

The report is read by other agents, so its shape is an interface. These appear verbatim in [SKILL.md](SKILL.md) and in every generated report; each is also a glossary term below, so renaming one is a multi-file change:

- The title, exactly `# Work report`.
- The three identifier lines: `branch:`, `session title:`, `session id:` — one fact per line, never merged.
- The six H2 headings, in order: `Goal`, `Problem → Solution`, `Verification`, `Follow-ups`, `Suggested skills`, `Abandoned approaches`.
- The Verification sub-lists: `Checked:` and `Not verified:`.
- The entry tags: `[question]` / `[action]` under Follow-ups, `[tried]` / `[considered]` under Abandoned approaches.

---

## README ↔ SKILL.md relationship

[README.md](README.md) is the user-facing landing page; [SKILL.md](SKILL.md) is what the model loads and the canonical behaviour spec. The README names the six sections and links here for the contract; what belongs in each section, the altitude rules, the title-block lookup, and the gitignore offer live only in SKILL.md.

Three things must stay in sync between the files:

- **The offer-first behaviour** — described in the README's Auto-activation section, in SKILL.md's frontmatter `description`, and in the SKILL.md body gate. All three must agree that proactive activation offers and waits, and that an explicit invocation is itself the go-ahead.
- **The six section names** — the README lists them; SKILL.md defines them.
- **SKILL.md headings the README anchors into** — currently the offer-first gate and Step 4. Renaming either heading breaks the corresponding README link, so it is a two-file change.

---

## Vocabulary is deliberate

Use these terms precisely, and honor the *Avoid* notes — e.g. don't call the work report a "session", "journal", or "handoff doc".

**Work report** (a.k.a. completion report):
The standardized `WORK-REPORT.md` at a working tree's root, written via `/work-report` when work is checkpointed or handed off — **one per working tree**, overwritten in place on each run, so it always reflects the current state, never fragments. Works in any repo — a worktree, the main checkout, or a repo with no worktrees. Gitignored by default.
*Avoid*: summary, handoff doc, session log, journal.

**Self-report**:
The stance of the work report: it records what the author *did and intended*, scoped to the current chat session, and is **not** reconstructed from git history. Git remains the factual record of the diff.
*Avoid*: audit, proof, reconstruction.

**The work**:
The subject a work report summarizes — everything done in **the current chat session**, not the branch's whole history.
*Avoid*: the diff, the changes, the branch.

**Problem → Solution**:
The report section logging every problem hit and the approach that **worked** — triggered by a first attempt that failed and forced a change of course, or a real fork (≥2 viable options) that was chosen.
*Avoid*: challenges, issues, notes.

**Abandoned approaches**:
The report section recording approaches that were **not** kept, so the next agent does not re-walk them. Each entry is tagged `[tried]` (implemented or attempted, then reverted) or `[considered]` (evaluated and rejected without being built).
*Avoid*: discarded, dead ends, rejected.

**Follow-ups**:
The report section listing what the next session or reviewer picks up. Each entry is tagged `[question]` (an unresolved decision or unknown) or `[action]` (concrete work to be done).
*Avoid*: open questions, next actions, TODOs.

**Session title**:
The human-readable, **mutable** name the VS Code plugin shows for a session and searches its list by (`custom-title` if the session was `/rename`d, else the latest auto-generated `aiTitle`). Recorded in the work report's `session title:` line so a reader can find the session in the plugin.
*Avoid*: session name, summary.

**Session id**:
The session's UUID — the transcript filename under `~/.claude/projects/`. **Stable and unique**, but **not searchable** in the VS Code plugin; it serves as the tie-breaker that confirms an exact match.
*Avoid*: session, chat id.
