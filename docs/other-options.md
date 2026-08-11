# Build a purpose-built completion report instead of adopting existing solution

We wanted per-worktree work summaries that a reviewing agent or a fresh session can read
instead of re-deriving intent from the diff.

Instead we built a small `/work-report` that writes a single `WORK-REPORT.md` at the working
tree's root, reusing claude-sessions' retrospective _format shape_ (goals, completed tasks,
problem→solution) and `handoff`'s discipline (reference artifacts, don't duplicate; keep it lean).
One living file per working tree, overwritten in place, sidesteps the single-session collision
entirely. It is standalone (not coupled to worktrees) because a work summary is useful in any repo.

## Considered options

### claude-sessions / a fork

We evaluated [iannuttall/claude-sessions](https://github.com/iannuttall/claude-sessions) (1207★, 137
forks) and its forks, and chose **not** to adopt it: it is archived, and its core model —
a single global `.current-session` pointer, one active session per project — is the
opposite of the many-parallel-worktrees model the report was built for. No fork is a live
re-architecture (the most-starred has 3★ and only adds a `session-load` command). The need
is validated but every implementation is dead.

### handoff skill

We got inspiration from mattpocock's [handoff skill](https://github.com/mattpocock/skills/blob/main/skills/productivity/handoff/SKILL.md). Good fit for the lateral agent → agent hop, but it lands in OS temp dir, not worktree-scoped dir. It is a one-shot, so it does not give durable per-worktree
records.

We ported its two portable content features (a "suggested skills" cue
and secret-redaction discipline) into `/work-report` without adopting its storage or lifecycle model.
