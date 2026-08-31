# AGENTS.md - `docs-consistency-check`

Contract pins unique to this skill. The four pins every skill carries (path, frontmatter `name`, frontmatter `description`, README to SKILL.md) are in the root [AGENTS.md](../../AGENTS.md#contract-pins-every-skill-carries) and apply here too. Everything pinned below is breaking for consumers in the same way: be deliberate, and mention it in the commit.

## Skill-specific pins

- Severity tier strings: `🔴 Conflict`, `⚠️ Outdated`, `↩️ Orphaned`, `❓ Unverifiable`. They appear verbatim in [SKILL.md](SKILL.md), [README.md](README.md), and every report the skill emits. Renaming a tier is a multi-file change.
- Auto-activation triggers: the file-types list and the keywords list appear verbatim in SKILL.md's frontmatter `description` and in the README's Auto-activation section. Change both together.
- Git is optional, not required: where a repo is present the audit may read git to sharpen a run, and where it is not every functionality falls back to the conversation and the filesystem. Both paths are load-bearing, so a step that works only with git is as broken as one that ignores git when it is there.
- Report fixed strings: the per-finding template, the `Found N issues:` summary line, the skipped-files line, and the clean verdict's evidence lines. Other agents read a report, so its shape is an interface; adding an evidence line means adding it here.
- Frontmatter `allowed-tools`: a pre-approval, not a restriction, so every command the body tells the model to run must appear there or the run stalls on a permission prompt. Widening it widens auto-approval and shows up in the pre-release Snyk scan; the git entries are read-only by design.
- SKILL.md headings the README anchors into: the 7 step headings, the 4 tier sub-headings under Step 5, "Security invariants", "Stay armed for the rest of the session", and "Review intentional mode".

## `.docs-consistency-check-ignore` conventions

This is the skill's runtime config. It stays at the consumer's repo root, not in the skill folder. `.agents/` and `.claude/` are seeded as default exclusions because those are vendored dev-tool skill directories; auditing them surfaces false positives about *other people's* docs drift.
