# dev-skills

A collection of [Claude](https://claude.com/claude-code) skills for developer documentation and workflow tasks. Each skill lives in its own folder under [`skills/`](skills/) and is installed individually — either as a Claude Code plugin or via [skills.sh](https://skills.sh).

---

## Skills

### [docs-consistency-check](skills/docs-consistency-check/README.md)

Audits the prose layer of a project (READMEs, SKILL.md, CLAUDE.md, AGENTS.md, manifests, changelogs) for drift between files.

### [work-report](skills/work-report/README.md)

Writes `WORK-REPORT.md` — a standardized self-report of a session's work that a reviewer or a later session reads instead of re-deriving intent from the diff.

---

## Install

This repo is a Claude Code **plugin marketplace** ([`.claude-plugin/marketplace.json`](.claude-plugin/marketplace.json)). Add it once, then install any skill by name:

```text
/plugin marketplace add pilniczek/dev-skills
/plugin install docs-consistency-check@dev-skills
```

Or vendor a single skill into your repo with [skills.sh](https://skills.sh):

```bash
npx skills add https://github.com/pilniczek/dev-skills --skill docs-consistency-check
```

---

Each skill's folder holds its own `README.md` (usage), `SKILL.md` (the canonical behaviour spec the model loads), and `AGENTS.md` (maintainer contract pins). The repo-root [`.claude-plugin/marketplace.json`](.claude-plugin/marketplace.json) is the plugin registry — one entry per skill. Skills are unversioned: every push to `master` is the release, so `npx skills add` and `/plugin install` always give you the current state.

---

## Adding a skill

New skills are added by dropping a folder under `skills/<name>/` and adding one entry to the Skills section above — nothing else at the repo root needs to change. See the [Adding a new skill](AGENTS.md#adding-a-new-skill) checklist in AGENTS.md, and [CONTRIBUTING.md](CONTRIBUTING.md) for local setup and the pre-release security scan.

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for local setup and the pre-release security scan workflow, and [AGENTS.md](AGENTS.md) for repo conventions and per-skill contracts.
