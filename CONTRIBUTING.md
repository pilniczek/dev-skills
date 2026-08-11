# Contributing to `dev-skills`

Thanks for considering a contribution. This file covers the local setup needed to develop and release safely. For repo conventions and per-skill contract pins, see [AGENTS.md](AGENTS.md).

---

## Vendored dev-time skills

After a fresh clone, run `npm run install:skills` to restore every vendored dev-time skill under `.agents/skills/` from [skills-lock.json](skills-lock.json). That command restores the whole set; to add or update a single skill instead, run `npx skills add` and commit the changed lockfile. Never edit a vendored copy in place, see [AGENTS.md](AGENTS.md).

## Pre-release security scan

Because every push to `master` is a release (see [AGENTS.md "Versioning & releases"](AGENTS.md#versioning--releases)), run Snyk's `agent-scan` against the `SKILL.md` of **each skill you changed** before pushing. It uses the same engine that produces the security badge on skills.sh, so the local result closely matches the post-release score.

### One-time setup

1. Install `uv` (Python tool runner) — see [astral.sh/uv](https://docs.astral.sh/uv/getting-started/installation/). Windows: `winget install --id=astral-sh.uv -e`.
2. Create a free [Snyk account](https://snyk.io/), grab the API token from [app.snyk.io/account](https://app.snyk.io/account), and save it to `.env` (gitignored) as a single line: `SNYK_TOKEN=<your-token>`.

### Each release

Load the token from `.env` — pick your shell:

```powershell
# PowerShell
$env:SNYK_TOKEN = (Get-Content .env | Where-Object { $_ -match '^SNYK_TOKEN=' }) -replace '^SNYK_TOKEN=',''
```

```cmd
:: cmd.exe
for /f "tokens=1,* delims==" %i in (.env) do @if /i "%i"=="SNYK_TOKEN" set SNYK_TOKEN=%j
```

```bash
# bash / zsh
export SNYK_TOKEN=$(grep ^SNYK_TOKEN= .env | cut -d= -f2-)
```

Then scan, once per changed skill (Windows shells take `.\skills\<skill-name>\SKILL.md`):

```bash
uvx snyk-agent-scan@latest --skills ./skills/<skill-name>/SKILL.md
```

Triage any High / Critical findings before pushing. The risk classes the scanner covers are documented at [snyk/agent-scan](https://github.com/snyk/agent-scan).
