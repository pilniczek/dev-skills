# docs-consistency-check

A Claude skill that audits the prose layer of a project (READMEs, SKILL.md, CLAUDE.md, AGENTS.md, templates, manifests, changelogs, installers) for drift: places where one file was updated and the related ones weren't, or where two files contradict each other.

It does not check code logic, verify external links, enforce formatting, or run in CI. Installers and manifests are read for the concepts they document, never analysed as programs - there's no linting or AST work here; it's a Claude skill, not a shell command. For consistency inside actual code, use a different tool.

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

## What it catches

Every finding is classified into one of four severity tiers:

| Icon | Tier |
| ---- | ---- |
| 🔴 | [Conflict](SKILL.md#-conflict) |
| ⚠️ | [Outdated](SKILL.md#️-outdated) |
| ↩️ | [Orphaned](SKILL.md#️-orphaned) |
| ❓ | [Unverifiable](SKILL.md#-unverifiable) |

## How it works

1. [Apply security guards, identify the file set](SKILL.md#step-1--apply-security-guards-then-identify-the-file-set): filter, then inventory
2. [Load intentional variations](SKILL.md#step-2--load-intentional-variations): pre-marked findings, silently suppressed
3. [Count heuristic](SKILL.md#step-3--count-heuristic-fast-first-pass): declared invariants first, the highest-yield signal, then list sizes
4. [Build the concept inventory](SKILL.md#step-4--build-the-concept-inventory): shared vocabulary across files
5. [Cross-reference and classify](SKILL.md#step-5--cross-reference-and-classify): apply the severity tiers
6. [Report findings](SKILL.md#step-6--report-findings): numbered, ordered by severity
7. [Apply fixes](SKILL.md#step-7--apply-fixes-and-manage-intentional-variations): edit, or mark a ❓ finding intentional

Credential and secret files are excluded before any read, and findings paraphrase rather than quote. See [Security invariants](SKILL.md#security-invariants).

## Ignoring files

On first run the skill offers to create `.docs-consistency-check-ignore` at your project root, seeded with `.agents/` and `.claude/` (the usual vendored skill directories). It waits for your go-ahead. Gitignore syntax; see [Step 1](SKILL.md#step-1--apply-security-guards-then-identify-the-file-set).

## Auto-activation

Claude offers this skill when you mention updating or creating a README, SKILL.md, CLAUDE.md, AGENTS.md, template, plugin.json, changelog, or installer, or use keywords like "docs", "sync", "feature added", or "I just updated". Offers always come first; the skill never runs silently. Once invoked it stays alert for the rest of the conversation, which [Stay armed](SKILL.md#stay-armed-for-the-rest-of-the-session) describes.

## Suppressing a finding permanently

A ❓ finding that's genuinely intentional can be recorded in `intentional-variations.md` and skipped on later runs. [Step 7](SKILL.md#step-7--apply-fixes-and-manage-intentional-variations) covers marking one, and [Review intentional mode](SKILL.md#review-intentional-mode) covers revisiting it. Most projects won't need this.

## Contributing

See [CONTRIBUTING.md](https://github.com/pilniczek/dev-skills/blob/master/CONTRIBUTING.md) for local setup and the pre-release security scan workflow.
