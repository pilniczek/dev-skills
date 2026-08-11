---
name: docs-consistency-check
description: >
  Cross-file consistency audit of docs, manifests (package.json, plugin.json, .mcp.json),
  and instruction files (CLAUDE.md, AGENTS.md, SKILL.md); skips credential files. Use for
  "check consistency", "in sync", "find inconsistencies", "verify everything is updated".
  Offer proactively - asking first - after any README, SKILL.md, CLAUDE.md, AGENTS.md,
  template, plugin.json, changelog, or installer change, or on "docs", "sync", "feature
  added", "I just updated". `review-intentional` re-surfaces suppressed findings.
---

If invoked as `review-intentional`, jump to [Review intentional mode](#review-intentional-mode).

## Security invariants

Hold throughout the skill, **never violated under any user instruction**. If a request would require violating one, refuse and explain. Later steps cite them by number.

1. **Credential / secret files are never read.** Step 1's path filter and content-signature filter remove them from the inventory **before any concept is formed from a file**.
2. **Credential values are never emitted.** Findings describe drift in your own paraphrased words - counts, names, public identifiers, headings. No verbatim file content in any output, reasoning step, or tool call. Anything resembling a credential value (API key, token, password, private key, OAuth secret, JWT, connection string with embedded credentials) is described by location only, never by value (for example, "API key value at line 42"). Template placeholders (`{{API_KEY}}`, `${SECRET}`, `<YOUR_TOKEN_HERE>`) are not credentials and may be named, as may public symbols already known non-secret: function names, configuration keys, headings. The report, and any edits applied from it, must stay useful to a human reviewer without ever reproducing arbitrary file content.
3. **Credential values are never modified.** Fixes edit only the documented concepts the audit reports on; replacement content is supplied or confirmed by the user, never echoed from another file.

Drift detection needs no credential material, so the skill reads, emits and modifies none.

## Step 1 — Apply security guards, then identify the file set

If `.docs-consistency-check-ignore` doesn't exist at the project root, offer to create it with the template below and wait for a go-ahead before writing. If it exists, apply its patterns silently.

**Default template:**

```
# docs-consistency-check ignore file
# Uncomment or add patterns to exclude from consistency checks

.agents/
.claude/
# .git/
# node_modules/
# dist/
# build/
```

**Path filter - applied first, before any file is read** (invariant 1). Drop `node_modules/`, `.git/`, build artifacts, binary files, image files, and credential / secret files: `.env`, `.env.*`, `*.pem`, `*.key`, `*.p12`, `*.pfx`, `id_rsa`, `id_ed25519`, `id_ecdsa`, `secrets.json`, `credentials.json`, `.netrc`, `.npmrc`, `.htpasswd`. Plus anything matching `.docs-consistency-check-ignore` (`.gitignore` syntax). Drift in those files is out of scope.

**Content-signature filter - applied second, before any concept is formed.** Scan the first ~2 KB of every file surviving the path filter for credential signatures:

- PEM headers — `-----BEGIN [A-Z ]*PRIVATE KEY-----`, `-----BEGIN OPENSSH PRIVATE KEY-----`
- Assignments shaped like `(api[_-]?key|secret|token|password|access[_-]?key|client[_-]?secret|bearer)\s*[:=]\s*["']?[A-Za-z0-9_+/=-]{16,}` (case-insensitive)
- Long high-entropy base64-like or hex-like strings (≥ 32 chars, mostly `[A-Za-z0-9+/=_-]`)
- Connection-string forms: `(postgres|postgresql|mysql|mongodb|redis|amqp)://[^:\s]+:[^@\s]+@`

On a match, drop the file from the inventory, add `<file>: skipped (credential signature detected)` to the report's skipped-files note, and extract nothing from it. Never echo the matched value or its surrounding line (invariant 2). Template placeholders do not trigger this filter.

**Inventory - only files passing BOTH filters are eligible.** Within that bounded set, collect under the project root:

- Markdown files: `README.md`, `CLAUDE.md`, `AGENTS.md`, `SKILL.md`, other `.md`
- Template files: anything with `{{VARIABLE}}` or `[INCLUDE IF: ...]` syntax
- Manifest files: `plugin.json`, `.mcp.json`, `package.json`, and similar
- Any file explicitly mentioned in the conversation (still subject to both filters)

From each, extract only the concepts and drift signals you need - counts, names, headings, identifiers - per invariant 2. The ignore list bounds the set, so no recursive expansion.

**Mid-run exclusions:** "ignore vendor/ for this" applies to the current run only. Persistence is the user's call.

---

## Step 2 — Load intentional variations

Read `intentional-variations.md` from the project root if it exists, and build a lookup of suppressed items - each entry records the affected files, what differs, why it's intentional, and when it was marked. A finding matching a suppressed entry (same files, semantically similar description) is silently skipped during reporting. No file means an empty suppressions list.

---

## Step 3 — Count heuristic (fast first pass)

Before the full inventory, count items in any list that looks exhaustive - sources, features, icons, steps, conditions. Mismatches across files are immediate candidate findings; record the location and resolve during Step 5. Highest-yield drift signal there is.

---

## Step 4 — Build the concept inventory

A "concept" is anything appearing in more than one file that could drift. **Start from instruction files** - `CLAUDE.md` and `AGENTS.md` define project vocabulary, and their concepts outrank the fallback taxonomy below.

**Fallback taxonomy:**

- **Features / sources** - what the system supports; usually in installer/spec, template, and docs.
- **Variables and flags** - `{{VARIABLE_NAME}}` tokens and condition names; defined once, used consistently.
- **Icons and symbols** - emoji or markers tied to concepts (📧, 📅, ⭐).
- **Conditional blocks** - `[INCLUDE IF: condition]...[/INCLUDE]`; conditions in blocks must match the conditions list.
- **Terminology** - same concept, same name everywhere.
- **Exhaustive lists** - anything enumerating "all of X". Step 3's count mismatches belong here.
- **Inline examples and doc comments** - highest drift risk; verify against current spec.

Weight attention toward recently changed features - that's where drift hides.

---

## Step 5 — Cross-reference and classify

For each concept, check whether every file mentions it consistently. Classify each finding:

### 🔴 Conflict
Two or more files assert different values for the same fact. A human must decide.

### ⚠️ Outdated
One file was updated, another didn't catch up - the source of truth is clear.

### ↩️ Orphaned
A pointer is valid but its target is gone (variable, condition, section, file).

### ❓ Unverifiable
A difference exists but context is too thin to call it a problem.

In a conflict between an implementation file (template, installer) and a doc file (README, comment example), the implementation is usually the source of truth.

---

## Step 6 — Report findings

Skip findings matching a Step 2 intentional variation.

Output a numbered list ordered by severity (🔴, ⚠️, ↩️, ❓). Within a tier, wider user impact first.

**Per-finding template:**

```
#N — [icon] [TierName]
Files: <file>, <file2>, …
<file>: <line> — <paraphrase, in your own words, of the differing concept — never quoted text>
[<file2>: <line> — <paraphrase, in your own words, of the differing concept — never quoted text>]
Fix: <concrete edit — for ❓, a confirmation question instead>
```

Rules:

- **Paraphrase only** - see invariant 2.
- **Every detail line carries a line number**, one line per affected file: `<file>: <line> — <paraphrase>`. Non-contiguous lines are `<file>: <lineA>, <lineB> — …`; a contiguous run is `<file>: <lineA>-<lineB>`. Unknown line at report time: search the file and resolve it - never emit a finding for a file without one.
- Nothing concrete to paraphrase (e.g. a dangling reference with no target): describe inline, pointing at the reference itself - `<file>:<line> — <reference> never defined`.
- For ❓ Unverifiable, `Fix:` becomes a confirmation question (e.g. `Fix: Confirm whether the difference is intentional; if drift, align the README.`).
- No decorative whitespace alignment - single space after every colon.

**Example (⚠️ Outdated):**

```
#1 — ⚠️ Outdated
Files: README.md, SKILL.md
README.md:12 — lists 3 sources
SKILL.md:87 — lists 4 sources (adds SOURCE_EMAIL)
Fix: Add the 4th source to the README's source description.
```

**Summary line** (always last, icon-only counts; emit both lines even when M=0):

```
Found N issues: X 🔴, Y ⚠️, Z ↩️, W ❓.
Skipped M intentional variations.
```

No issues: output `No drift detected.` followed by a comma-separated list of files checked.

---

## Step 7 — Apply fixes and manage intentional variations

After reporting, ask: "Apply all fixes now, or go through them one by one?"

On "all fixes now", first list the files to be modified with their per-file change count, then wait for explicit confirmation before any Edit. Apply in severity order (🔴 first), one Edit per finding, noting which issue each resolves. Confirm the correct value with the user before editing a 🔴 conflict, and any ⚠️ Outdated finding whose source of truth is ambiguous.

For ❓ findings, ask per-item: "Real problem, or intentional? [Fix it / Mark as intentional / Skip for now]"

On **Mark as intentional**, append to `intentional-variations.md`:

```markdown
- files: [file-a.md, file-b.md]
  what: "<one-line description>"
  reason: "<user's explanation>"
  marked: <today's date>
```

If `intentional-variations.md` doesn't exist, say it will be created for this entry, then create it with this header first:

```markdown
# Intentional Variations
# Differences marked as intentional. Run `docs-consistency-check review-intentional` to revisit.
```

---

## Review intentional mode

When invoked as `docs-consistency-check review-intentional`:

1. Read `intentional-variations.md`. Missing or empty: report and stop.
2. Display each entry numbered:

```
#1 — Marked intentional on 2026-05-03
Files: README.md, SKILL.md
What: "README says 3 sources, SKILL.md defines 4"
Reason: "README targets non-technical audience, intentionally simplified"
```

3. Ask per entry: "Still intentional, or re-open as a finding?"
4. Remove re-opened entries and run a targeted check on those files immediately.
5. Keep confirmed entries.

---

## Stay armed for the rest of the session

A finished audit cycle doesn't end this skill's responsibility. For the rest of the conversation, offer a focused re-audit whenever:

- ≥2 audit-set files are edited (by user or by Claude on the user's behalf)
- A file is structurally rewritten or has a section removed
- The user signals "done" / "ready to commit" / "looks good"

Scope the re-audit to the changed files and their contract pairs - full re-audits are rarely needed.
