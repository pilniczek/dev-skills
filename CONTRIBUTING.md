# Contributing to `dev-skills`

Thanks for considering a contribution. This file covers the local setup needed to develop and release safely. For repo conventions and per-skill contract pins, see [AGENTS.md](AGENTS.md); to add a new skill, follow the [Adding a new skill](AGENTS.md#adding-a-new-skill) checklist.

---

## Pre-release security scan

Because every push to `master` is a release (see [AGENTS.md "Versioning & releases"](AGENTS.md#versioning--releases)), run Snyk's `agent-scan` against the `SKILL.md` of **each skill you changed** before pushing. It uses the same engine that produces the security badge on skills.sh, so the local result closely matches the post-release score.

### One-time setup

1. Install `uv` (Python tool runner) — see [astral.sh/uv](https://docs.astral.sh/uv/getting-started/installation/). Windows: `winget install --id=astral-sh.uv -e`.
2. Create a free [Snyk account](https://snyk.io/), grab the API token from [app.snyk.io/account](https://app.snyk.io/account), and save it to `.env` (gitignored) as a single line: `SNYK_TOKEN=<your-token>`.

### Each release

Replace `<skill-name>` with the skill you changed (run once per changed skill).

PowerShell:

```powershell
$env:SNYK_TOKEN = (Get-Content .env | Where-Object { $_ -match '^SNYK_TOKEN=' }) -replace '^SNYK_TOKEN=',''
uvx snyk-agent-scan@latest --skills .\skills\<skill-name>\SKILL.md
```

cmd.exe:

```cmd
for /f "tokens=1,* delims==" %i in (.env) do @if /i "%i"=="SNYK_TOKEN" set SNYK_TOKEN=%j
uvx snyk-agent-scan@latest --skills .\skills\<skill-name>\SKILL.md
```

bash / zsh:

```bash
export SNYK_TOKEN=$(grep ^SNYK_TOKEN= .env | cut -d= -f2-)
uvx snyk-agent-scan@latest --skills ./skills/<skill-name>/SKILL.md
```

Triage any High / Critical findings before pushing. The risk classes the scanner covers are documented at [snyk/agent-scan](https://github.com/snyk/agent-scan).
