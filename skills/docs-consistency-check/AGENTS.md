# AGENTS.md — `docs-consistency-check`

Maintainer-only contract pins for this skill. A change touching any of these is **breaking** for the skill's consumers — be deliberate and mention it in the commit. For repo-wide conventions, see the root [AGENTS.md](../../AGENTS.md).

---

## Source-of-truth pins

These paths and strings are part of the skill's **public contract** — moving or renaming them breaks downstream consumers whose [skills-lock.json](../../skills-lock.json) files reference this skill by hash.

- **The skill path `skills/docs-consistency-check/SKILL.md`** — once anyone runs `npx skills add https://github.com/pilniczek/dev-skills --skill docs-consistency-check`, their lockfile pins this path. The root [marketplace.json](../../.claude-plugin/marketplace.json) also references this folder (`skills: ["./skills/docs-consistency-check"]`). Don't move it without updating both.
- **Frontmatter `name: docs-consistency-check`** — determines the install directory in consumer repos (`.claude/skills/docs-consistency-check/`) and matches the plugin `name` in [marketplace.json](../../.claude-plugin/marketplace.json). Renaming = breaking change for every consumer.
- **Frontmatter `description`** — drives auto-activation in every consumer's repo. **Wording changes are behaviour changes**, not docs tweaks. Treat as API.
- **Severity tier strings** — `🔴 Conflict`, `⚠️ Outdated`, `↩️ Orphaned`, `❓ Unverifiable`. Appear verbatim in [SKILL.md](SKILL.md), [README.md](README.md), and the output format. Renaming any tier is a multi-file change.

---

## README ↔ SKILL.md relationship

[README.md](README.md) is the user-facing landing page; [SKILL.md](SKILL.md) is what the model loads and the canonical behaviour spec. README provides a brief overview and links into SKILL.md for detail — workflow steps, severity tier definitions, the ignore-file lifecycle, and the intentional-variations flow live only in SKILL.md.

Two things must stay in sync between the files:

- **Auto-activation triggers** — the file-types list and the keywords list appear verbatim in the README's Auto-activation section and in SKILL.md's frontmatter `description`.
- **SKILL.md section headings linked from the README** — the 7 step headings, the 4 tier sub-headings under Step 5, "Stay armed for the rest of the session", and "Review intentional mode". Renaming any breaks the corresponding README link.

---

## `.docs-consistency-check-ignore` conventions

This is the skill's runtime config — it stays at the **consumer's repo root**, not in the skill folder. `.agents/` and `.claude/` are seeded as default exclusions because those are vendored dev-tool skill directories — auditing them would surface false positives about *other people's* docs drift.
