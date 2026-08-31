# Contributing to `dev-skills`

Thanks for considering a contribution. This file covers the local setup needed to develop and release safely. For repo conventions and per-skill contract pins, see [AGENTS.md](AGENTS.md).

---

## Vendored dev-time skills

After a fresh clone, run `npm run install:skills` to restore every vendored dev-time skill under `.agents/skills/` from [skills-lock.json](skills-lock.json). That command restores the whole set; to add or update a single skill instead, run `npx skills add` and commit the changed lockfile. Never edit a vendored copy in place, see [AGENTS.md](AGENTS.md).

## Pre-release security scan

Because every push to `master` is a release (see [AGENTS.md "Versioning & releases"](AGENTS.md#versioning--releases)), run Snyk's `agent-scan` against the `SKILL.md` of **each skill you changed** before pushing. It uses the same engine that produces the security badge on skills.sh, so the local result closely matches the post-release score.

Linux, WSL, macOS and Windows are all supported; where the steps differ, pick your platform's variant below.

### One-time setup

1. Install `uv` (Python tool runner) - see [astral.sh/uv](https://docs.astral.sh/uv/getting-started/installation/):
   - Windows: `winget install --id=astral-sh.uv -e`
   - Linux / WSL / macOS: `curl -LsSf https://astral.sh/uv/install.sh | sh`, or a package manager (`apt install uv` on Ubuntu 24.10+, `brew install uv`, `pipx install uv`)

   Restart the shell afterwards so `uvx` is on `PATH`.
2. Install Node from [.nvmrc](.nvmrc) - `nvm install && nvm use` (Linux / WSL / macOS), or `nvm use $(Get-Content .nvmrc)` with [nvm-windows](https://github.com/coreybutler/nvm-windows).
3. Create a free [Snyk account](https://snyk.io/), grab the API token from [app.snyk.io/account](https://app.snyk.io/account), and save it to `.env` (gitignored) as a single line: `SNYK_TOKEN=<your-token>`.

Under WSL, keep the clone inside the Linux filesystem (`~/...`, not `/mnt/c/...`) - `uvx` and `npm` are much slower across the Windows mount.

### Each release

Scan this repo's skills:

```bash
npm run snyk
```

The script wraps `uvx snyk-agent-scan@latest --skills ./skills` in a `node --env-file=.env` call. Both parts are load-bearing:

- `node --env-file=.env` supplies `SNYK_TOKEN`, which is the only channel the scanner accepts it through - there is no `--token` flag. `npm` itself does not read `.env`, and without the token the scanner exits with "To use Agent Scan, set the SNYK_TOKEN environment variable". Node's built-in loader keeps this dependency-free, and needs the Node version pinned in [.nvmrc](.nvmrc).
- `./skills` scopes discovery to this repo. `--skills` is only a boolean flag; with no positional path the scanner falls back to well-known config locations and picks up unrelated MCP servers from other projects on the machine, prompting for consent to launch them.

To scan a single skill instead, export the token yourself and call the scanner directly:

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

```bash
uvx snyk-agent-scan@latest --skills ./skills/<skill-name>/SKILL.md
```

Windows shells also take a backslash path (`.\skills\<skill-name>\SKILL.md`).

The free tier is daily-rate-limited; `[X007 info]: Daily usage limit reached` means you are out of scans for the day, not misconfigured. Each run writes `snyk-scan.json` and `snyk-scan-output.txt` to the repo root, overwriting the previous run's reports; both are gitignored. Triage any High / Critical findings before pushing. The risk classes the scanner covers are documented at [snyk/agent-scan](https://github.com/snyk/agent-scan).
