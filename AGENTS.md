# AGENTS.md - `dev-skills`

Guidance for AI agents (and humans pairing with them) working on this repo. Covers only what is non-derivable from reading the files. This repo is a collection of Claude skills. For what an individual skill does, read its folder's `README.md` (users) or `SKILL.md` (model) under [skills/](skills/).

## Repo layout

Each skill is a self-contained folder under `skills/<name>/` holding three files that travel together:

- `SKILL.md` (the model): canonical behaviour spec, loaded on activation. The source of truth.
- `README.md` (users): usage, install command, overview. Links into `SKILL.md`, never restates it.
- `AGENTS.md` (maintainers): that skill's contract pins. Vendored as a side effect, so keep it to durable pins, not working notes.

The repo root holds collection-wide docs ([README.md](README.md) index, this file, [CONTRIBUTING.md](CONTRIBUTING.md)) and shared metadata ([package.json](package.json), [skills-lock.json](skills-lock.json), [.claude-plugin/marketplace.json](.claude-plugin/marketplace.json)). No skill's authored content belongs at the root. Runtime artifacts are the exception: a skill's config or output lands at whatever repo root the skill runs in, this one included (`.docs-consistency-check-ignore`, `intentional-variations.md`, `WORK-REPORT.md`), as do the pre-release scan's local reports (`snyk-scan.json`, `snyk-scan-output.txt`). The report and the scan reports are gitignored, being per-session throwaways; the audit's ignore config and variations file are committed, because both encode standing decisions about this repo.

Maintainer background - the reasoning behind a skill, options weighed and rejected, similar history - belongs in `docs/` at the root, not in a skill folder: a skill folder is vendored verbatim into every consumer's repo on install, and consumers have no use for our decision history. Create the directory when the first such note is written, and link it from the relevant skill's `AGENTS.md`.

Skills are kept flat, one folder per skill directly under `skills/`. When the collection grows enough to warrant grouping, adopt category buckets (mirroring [mattpocock/skills](https://github.com/mattpocock/skills)): public `engineering/`, `productivity/`, `misc/` and private `personal/`, `in-progress/`, `deprecated/`, nesting skills as `skills/<bucket>/<name>/`. Buckets change the pinned skill path, so introduce them deliberately.

## The repo has two roles, keep them separate

This repo ships skills and uses skills to develop them. Two surfaces, two rules.

- Skills being authored (what `npx skills add` ships) live in `skills/*/`. Edit freely; these are the product.
- Vendored dev-time skills live in `.agents/skills/`. (`npx skills add` symlinks `.claude/skills/<name>` at them, so that directory appears only once a skill is vendored.) Never edit in place. Re-vendor via `npx skills add` and update [skills-lock.json](skills-lock.json).

## Constraint: a skill folder is vendored verbatim

`skills/<name>/` is copied into a consumer's `.claude/skills/<name>/` on install. Consequence:

Every relative path in a skill's `SKILL.md` or `README.md` must point to a sibling. Those two files are read in the consumer's repo and on skills.sh, where `references/schemas.md` resolves and `../../CONTRIBUTING.md` does not. Link repo-root files by absolute GitHub URL. Keep helper files (assets, references, scripts) inside the skill folder, never at the repo root.

The per-skill `AGENTS.md` is the exception. It is maintainer-facing and only read here, so its `../../` links up to root are intended.

## Contract pins every skill carries

Each skill's own `AGENTS.md` lists what is unique to it. These four pins apply to every skill, so they are stated once here. Changing any of them is breaking for that skill's consumers. Be deliberate and mention it in the commit.

- The skill path `skills/<name>/SKILL.md`: once anyone runs `npx skills add …`, consumers pin this path by content hash. [marketplace.json](.claude-plugin/marketplace.json) references the same folder, so moving it means updating both.
- Frontmatter `name`: determines the install directory in consumer repos (`.claude/skills/<name>/`), the slash command users type, and the plugin `name` in [marketplace.json](.claude-plugin/marketplace.json). (Verified: `npx skills add` vendors to `.agents/skills/<name>/` and symlinks `.claude/skills/<name>` at it, so the name drives both paths.)
- Frontmatter `description`: drives auto-activation in every consumer's repo. Wording changes are behaviour changes, not docs tweaks. Treat as API.
- README to SKILL.md: `README.md` is the user-facing landing page, `SKILL.md` the canonical behaviour spec the model loads. The README links into SKILL.md for detail. Any SKILL.md heading a README anchors into is itself a pin: renaming it breaks the link, so it is a two-file change.

## Adding a new skill

Adding a skill should not require rethinking anything at the root:

1. Create `skills/<name>/` with a `SKILL.md` (unique frontmatter `name`, plus `allowed-tools` if its body tells the model to run commands), a `README.md` (usage + install command), and an `AGENTS.md` for skill-specific pins.
2. Link the new `AGENTS.md` from [Per-skill guidance](#per-skill-guidance) below.
3. Add one entry to the Skills section of the root [README.md](README.md), matching the plugin description.
4. Register the plugin in [.claude-plugin/marketplace.json](.claude-plugin/marketplace.json): append `{ "name": "<name>", "description": "…", "source": "./", "strict": false, "skills": ["./skills/<name>"] }`. All plugins share `source: "./"`, the repo root; `skills` points at the folder holding `SKILL.md`. No `version` field, see below.
5. Run the pre-release security scan against the new `SKILL.md`, see [CONTRIBUTING.md](CONTRIBUTING.md#pre-release-security-scan). Repeat for every changed skill before every push.

## Versioning & releases

Skills ship two ways, both per-skill: as Claude Code plugins via [.claude-plugin/marketplace.json](.claude-plugin/marketplace.json) (`/plugin install <name>@dev-skills`) and as skills.sh vendored skills (`npx skills add … --skill <name>`). Keep the marketplace registry in sync with the `skills/` folders.

Nothing here carries a version number, on purpose. Each push to `master` is a release, consumers of both channels track the latest master commit, and git history is the changelog. The field is either invisible or harmful in these two channels: skills.sh pins by content hash, while the plugin marketplace does read `version` and [pins to it](https://code.claude.com/docs/en/plugin-marketplaces). With the field set, consumers update only when the string changes, so a forgotten bump silently freezes everyone.

Adopting versions later means adopting the automation that guarantees the bump: a `plugin.json` holding the number, a sync script, a release bot, the [mattpocock/skills](https://github.com/mattpocock/skills) shape. Don't add the number without the machinery.

## Per-skill guidance

- [docs-consistency-check](skills/docs-consistency-check/AGENTS.md)
- [work-report](skills/work-report/AGENTS.md)
