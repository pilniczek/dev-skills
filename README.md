# dev-skills

A collection of [Claude](https://claude.com/claude-code) skills for developer documentation and workflow tasks. Each skill lives in its own folder under [`skills/`](skills/) and installs on its own. Nothing here is versioned: every push to `master` is the release, so both install routes give you the current state.

## Skills

### [docs-consistency-check](skills/docs-consistency-check/README.md)

Audits the prose layer of a project (READMEs, SKILL.md, CLAUDE.md, AGENTS.md, templates, manifests, changelogs, installers) for cross-file drift.

### [work-report](skills/work-report/README.md)

Writes `WORK-REPORT.md`, a standardized self-report of a session's work that a reviewer or a later session reads instead of re-deriving intent from the diff.

## Install

This repo is a Claude Code plugin marketplace ([`.claude-plugin/marketplace.json`](.claude-plugin/marketplace.json)). Add it once, then install any skill by name:

```text
/plugin marketplace add pilniczek/dev-skills
/plugin install docs-consistency-check@dev-skills
```

Or vendor a single skill into your repo with [skills.sh](https://skills.sh):

```bash
npx skills add https://github.com/pilniczek/dev-skills --skill docs-consistency-check
```

## Contributing and conventions

[AGENTS.md](AGENTS.md) covers the repo layout, the [checklist for adding a skill](AGENTS.md#adding-a-new-skill), the contract pins, and why nothing is versioned. [CONTRIBUTING.md](CONTRIBUTING.md) covers local setup and the pre-release security scan.
