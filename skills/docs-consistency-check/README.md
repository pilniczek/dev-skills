# docs-consistency-check

A Claude skill that audits the **prose layer** of a project — READMEs, SKILL.md, CLAUDE.md, AGENTS.md, templates, plugin manifests, changelogs, installers — for drift: places where one file was updated and related files weren't, or where two files actively contradict each other.

It does **not** check code (no linting, AST, or imports), verify external links, enforce formatting/style, or run in CI — it's a Claude skill, not a shell command. For cross-file consistency in actual code (React, Java, etc.), use a different tool.

---

## Install

As a Claude Code plugin (from the [dev-skills](https://github.com/pilniczek/dev-skills) marketplace):

```text
/plugin marketplace add pilniczek/dev-skills
/plugin install docs-consistency-check@dev-skills
```

Or vendor it into your repo with skills.sh:

```bash
npx skills add https://github.com/pilniczek/dev-skills --skill docs-consistency-check
```

[skills.sh/pilniczek/dev-skills](https://skills.sh/pilniczek/dev-skills/docs-consistency-check)

---

## What it catches

Every finding is classified into one of four severity tiers:

| Icon | Tier |
| ---- | ---- |
| 🔴 | [Conflict](SKILL.md#-conflict) |
| ⚠️ | [Outdated](SKILL.md#️-outdated) |
| ↩️ | [Orphaned](SKILL.md#️-orphaned) |
| ❓ | [Unverifiable](SKILL.md#-unverifiable) |

---

## How it works

1. **[Load the ignore list and identify the file set](SKILL.md#step-1--apply-security-guards-then-identify-the-file-set)** — manage `.docs-consistency-check-ignore`; collect docs, templates, and manifests to audit
2. **[Load intentional variations](SKILL.md#step-2--load-intentional-variations)** — read `intentional-variations.md` for silent suppression of pre-marked findings
3. **[Count heuristic (fast first pass)](SKILL.md#step-3--count-heuristic-fast-first-pass)** — flag list-size mismatches across files before deep reading
4. **[Build the concept inventory](SKILL.md#step-4--build-the-concept-inventory)** — index shared vocabulary, prioritising terms named in `CLAUDE.md` / `AGENTS.md`
5. **[Cross-reference and classify](SKILL.md#step-5--cross-reference-and-classify)** — apply severity tiers to every concept comparison
6. **[Report findings](SKILL.md#step-6--report-findings)** — output the numbered list, ordered by severity, skipping intentional variations
7. **[Apply fixes and manage intentional variations](SKILL.md#step-7--apply-fixes-and-manage-intentional-variations)** — edit files; for ❓ findings, interactively decide fix / mark intentional / skip

---

## Ignoring files and directories

On first run, the skill offers to create `.docs-consistency-check-ignore` at your project root with `.agents/` and `.claude/` excluded by default (typical vendored skill directories) — it waits for your go-ahead before writing. After that you maintain it directly. See [Step 1](SKILL.md#step-1--apply-security-guards-then-identify-the-file-set) for the default template, gitignore syntax, and mid-run exclusions.

---

## Auto-activation

Claude is instructed to offer this skill proactively when you mention updating or creating a README, SKILL.md, CLAUDE.md, AGENTS.md, template, plugin.json, changelog, or installer — or use keywords like "docs", "sync", "feature added", or "I just updated". Offers always come first — the skill never runs silently — so you stay in control. Once invoked, the model stays alert for the rest of the conversation; see [Stay armed](SKILL.md#stay-armed-for-the-rest-of-the-session) for in-session re-trigger conditions.

---

## Suppressing a finding permanently

For ❓ findings that are genuinely intentional, you can mark them so the skill records them in `intentional-variations.md` and silently skips them on future runs. See [Step 7](SKILL.md#step-7--apply-fixes-and-manage-intentional-variations) for the marking flow and [Review intentional mode](SKILL.md#review-intentional-mode) for revisiting. Most projects won't need this.

---

## Contributing

See [CONTRIBUTING.md](../../CONTRIBUTING.md) for local setup and the pre-release security scan workflow.
