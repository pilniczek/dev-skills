# AGENTS.md — `dev-skills`

Guidance for AI agents (and humans pairing with them) working on this repo. Covers only what is **non-derivable** from reading the files. This repo is a **collection** of Claude skills; for what any individual skill does, read its folder's `README.md` (user-facing) or `SKILL.md` (model-facing) under [skills/](skills/).

---

## Repo layout

Each skill is a self-contained folder under `skills/<name>/`:

- **`SKILL.md`** (the model) — canonical behaviour spec, what Claude loads on activation. The source of truth.
- **`README.md`** (users) — usage, install command, overview. Links into `SKILL.md` for detail.
- **`AGENTS.md`** (maintainers) — that skill's contract pins and sync rules.

The repo root holds only **collection-wide** docs ([README.md](README.md) index, this file, [CONTRIBUTING.md](CONTRIBUTING.md)) and shared metadata ([package.json](package.json), [skills-lock.json](skills-lock.json), [.claude-plugin/marketplace.json](.claude-plugin/marketplace.json)). Nothing skill-specific belongs at the root.

**`docs/` holds maintainer background** — the reasoning behind a skill, options that were weighed and rejected, and similar history. It lives at the root rather than inside a skill folder because a skill folder is vendored verbatim into every consumer's repo on install, and consumers have no use for our decision history. Link to these notes from the relevant skill's `AGENTS.md`.

Skills are currently kept **flat** — one folder per skill directly under `skills/`. When the collection grows enough to warrant grouping, adopt category buckets (mirroring [mattpocock/skills](https://github.com/mattpocock/skills)): public `engineering/`, `productivity/`, `misc/` and private `personal/`, `in-progress/`, `deprecated/`, nesting skills as `skills/<bucket>/<name>/`. Buckets change the pinned skill path, so introduce them deliberately (see the per-skill `AGENTS.md` for which paths are contracts).

---

## The repo has two roles — keep them separate

This repo simultaneously **ships skills** and **uses skills to develop them**. Two different surfaces, two different rules.

- **Skills being authored** (what `npx skills add` ships) — live in `skills/*/`. Edit freely; these are the product.
- **Vendored dev-time skills** — live in `.agents/skills/` (`npx skills add` symlinks `.claude/skills/<name>` at them, so that directory appears only once a skill is vendored). **Never edit in place.** Re-vendor via `npx skills add` and update [skills-lock.json](skills-lock.json).

---

## Adding a new skill

Adding a skill should not require rethinking anything at the root:

1. Create `skills/<name>/`.
2. Write `SKILL.md` with a **unique** frontmatter `name` (it determines the install directory in consumer repos — see the skill's own `AGENTS.md`).
3. Add a `README.md` (usage + install command).
4. Add an `AGENTS.md` in the skill folder for any maintainer-only contract pins, and link it from [Per-skill guidance](#per-skill-guidance) below.
5. Add one entry to the **Skills** section of the root [README.md](README.md).
6. Register the skill as a plugin in [.claude-plugin/marketplace.json](.claude-plugin/marketplace.json) — append a `plugins` entry: `{ "name": "<name>", "description": "…", "source": "./", "strict": false, "skills": ["./skills/<name>"] }`. (Plugins all share `source: "./"`, the repo root; the `skills` path points to the skill folder that holds `SKILL.md`. Deliberately **no `version`** — see [Versioning & releases](#versioning--releases).)
7. Run the pre-release security scan against the new `SKILL.md` — see [CONTRIBUTING.md](CONTRIBUTING.md#pre-release-security-scan).

Keep helper files (assets, references, scripts) **inside** the skill folder, never at the repo root — see the constraint below.

---

## Constraints inside a skill directory

The contents of `skills/<name>/` get vendored verbatim into a consumer's `.claude/skills/<name>/` on install. Consequence:

**All relative paths in a `SKILL.md` must point to siblings.** References like `references/schemas.md` work after vendoring; references like `../../README.md` or repo-root paths don't. If a skill needs helper files, put them inside its own folder.

A skill's `README.md` and `AGENTS.md` live alongside its `SKILL.md` in the skill folder. The README is user-facing and meant to travel on install; the per-skill `AGENTS.md` is maintainer-facing contract documentation that may also be vendored as a side effect — keep it to durable contract pins, not transient working notes.

---

## Versioning & releases

**Nothing here carries a version number, on purpose.** Each push to `master` is a release, and consumers of both channels track the latest master commit. Git history is the changelog.

Why no versions — the field is either invisible or actively harmful in the two channels this repo ships through:

- **skills.sh** pins by **content hash**, not version. [skills-lock.json](skills-lock.json) records a `computedHash` per skill and has no version field at all, so a version number would change nothing for anyone.
- **The plugin marketplace** does read `version`, and [pins to it](https://code.claude.com/docs/en/plugin-marketplaces): with the field set, consumers receive an update **only when the string changes**. With no version anywhere, updates flow from the latest commit automatically. Setting one by hand would mean a forgotten bump silently freezes every consumer — a failure mode worth more than the field is.

Adopting versions later means adopting the automation that guarantees the bump (a `plugin.json` holding the number, a sync script, and a release bot — the [mattpocock/skills](https://github.com/mattpocock/skills) shape). Don't add the number without the machinery; see [docs/other-options.md](docs/other-options.md) for the comparison.

- Skills ship two ways, both per-skill: as Claude Code plugins via [.claude-plugin/marketplace.json](.claude-plugin/marketplace.json) (`/plugin install <name>@dev-skills`) and as skills.sh vendored skills (`npx skills add … --skill <name>`). Keep the marketplace registry in sync with the `skills/` folders.
- Run the pre-release security scan for each skill you changed before each push — see [CONTRIBUTING.md](CONTRIBUTING.md#pre-release-security-scan).

---

## Per-skill guidance

Skill-specific contract pins and sync rules live in each skill's own `AGENTS.md`:

- [docs-consistency-check](skills/docs-consistency-check/AGENTS.md)
- [work-report](skills/work-report/AGENTS.md)
