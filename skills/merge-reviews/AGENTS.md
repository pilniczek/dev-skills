# AGENTS.md - `merge-reviews`

Contract pins and vocabulary unique to this skill; the four every skill carries (path, frontmatter `name`, frontmatter `description`, README to SKILL.md) are in the root [AGENTS.md](../../AGENTS.md#contract-pins-every-skill-carries) and apply here too. Everything below breaks consumers the same way: be deliberate, mention it in the commit.

## Skill-specific pins

- **One bar, stated as an input producing a wrong output.** Every other severity axis a reviewer might use - risk, smell, criticality, "blocker" as a label - is re-derived through it. Softening the bar to "anything that could cause a bug later" re-admits every structural finding and the skill stops sorting anything.
- **Three buckets, not two: blocker, non-blocker, scope.** Dropping scope is the failure mode that made this skill necessary. An unaided pass produces a fix list and a dropped list, files deliberately unfinished work as defects, and the author re-argues the same MVP boundary at every review.
- **Scope needs written evidence - and another review is never that evidence.** A structural reviewer proposing a design that contains the behaviour another reviewer filed as a defect is two reviewers disagreeing, which is the strongest signal the owner must decide. An earlier draft lacked this line and a test run demoted a real question on the strength of the confident proposal.
- **The collision rule: the defect sets the priority, the structural finding sets the shape of the fix.** Both halves are load-bearing. Symptom-only repairs one case; structure-only refactors around a live bug and ships it in nicer code.
- **The re-specification trap stays in the collision section.** A redesign written without running the behaviour tends to write the defect into its target design. It is the least intuitive rule here and the one an unaided pass gets backwards.
- **One review or two, never three.** Every rule here is written and tested for a pair; a third review turns the merge into a vote, where agreement starts passing for severity. A single review is wrapped: same bar, buckets, ordering and method audit, no comparison.
- **Damage-first ordering, reachability as tie-break only.** Reachability is already inside the bar; using it as the primary sort counts it twice and buries the findings that destroy user input.
- **The method audit is part of the output, not a courtesy.** Naming what a reviewer's method could not see is what makes the next round of reviews better, and it must carry evidence rather than adjectives.
- **Owner questions are batched, with a recommendation each; facts are never asked.** A merge that stops on the first ambiguity is worth less than one that delivers the list and names the two or three real decisions.
- **Verification is compressed on purpose.** An earlier draft argued at length for checking cited lines, facts and consumers; measured against an unaided baseline, that behaviour happens anyway, and the section cost ~20 lines for nothing. What survived is the part that does not happen unprompted: saying what could not be verified, and why.
- **Frontmatter `allowed-tools`: `Grep, Glob, Read`.** A pre-approval, not a restriction, and read-only by design: the skill decides, it does not edit. Reviews that need a shell to verify (inspecting a tarball, running a test) should surface the claim as unverified rather than widen this. Adding `Bash` or a write tool changes the security surface and is flagged by the pre-release Snyk scan.
- **[README.md](README.md) anchors into seven SKILL.md headings:** `#classify-with-one-bar`, `#when-two-reviews-hit-the-same-code`, `#one-review-or-two`, `#order-the-blockers`, `#audit-each-reviewers-method`, `#output`, `#traps`. Renaming any is a two-file change.

## Known open risk

Two independent test runs both dissolved a boundary-validation finding on the same reasoning: the only sender is the host's own code, so no scenario exists. That is correct when both sides of the boundary ship from one build, and wrong when they are released independently. The skill has no line drawing that distinction, so it will keep resolving it the same way. If a real merge gets it wrong, the edit belongs in the bar: a trusted boundary whose two sides version apart produces a scenario even when today's only sender is your own code.

## Vocabulary is deliberate

Every term below appears verbatim in [SKILL.md](SKILL.md). The bucket names are the output contract.

| Term | What it names | Avoid |
| ---- | ------------- | ----- |
| **Blocker** | A finding whose failing scenario can be written: this input, this state, this wrong result. | critical, P1, must-fix |
| **Non-blocker** | A real observation with no scenario attached - duplication, layering, naming, measured cost. | nit, minor, low priority |
| **Scope** | Looks broken, is an unfinished decision, evidenced by something written. | wontfix, by design, expected |
| **Failing scenario** | The concrete inputs, state and wrong result that make a finding a blocker. | repro, impact, severity |
| **Fix shape** | What the repair is, including the structural half when two reviews meet on one defect. | solution, recommendation |
| **Reachability** | Whether a defect is live on a consumer's path today or latent on a published surface. | severity, priority |
| **Re-specification** | A proposed redesign that writes the disputed behaviour into its target design. | regression, oversight |
| **Method audit** | The per-reviewer account of what that reviewer's approach could not have seen. | critique, scoring |
