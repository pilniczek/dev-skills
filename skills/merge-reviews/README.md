# merge-reviews

A Claude skill that turns two independent reviews of the same change into one list the author can work top to bottom, plus a register of what was deliberately not fixed. It also wraps a single review the same way. Reviews can be files, PR comments or plain chat messages; three or more are out of scope - see [One review or two](SKILL.md#one-review-or-two).

Two competent reviews of one diff are usually not comparable. One ranks by what a user can break, the other by what the shape of the code will cost later. Concatenating them puts a silent data-loss bug below a naming preference, and the author either fixes everything or stops trusting the list. This skill re-derives severity from failing scenarios instead of from either reviewer's labels.

## Install

As a Claude Code plugin (from the [dev-skills](https://github.com/pilniczek/dev-skills) marketplace):

```text
/plugin marketplace add pilniczek/dev-skills
/plugin install merge-reviews@dev-skills
```

Or vendor it into your repo with skills.sh:

```bash
npx skills add https://github.com/pilniczek/dev-skills --skill merge-reviews
```

[skills.sh/pilniczek/dev-skills](https://skills.sh/pilniczek/dev-skills/merge-reviews)

## One bar, three buckets

A finding is a **blocker** only when the failing scenario can be written: this input, this state, this wrong result. Everything else is a **non-blocker** - real, recorded, not fixed now - or **scope**, something that looks broken because it is an unfinished decision rather than a mistake. [Classify with one bar in SKILL.md](SKILL.md#classify-with-one-bar) sets the bar and says why scope needs written evidence, and why another reviewer's confident proposal is not that evidence.

## Where the merge earns its keep

When two reviews hit the same code, one usually reports the symptom and the other the shape around it, and each fix alone is wrong. The rule is that the defect sets the priority and the structural finding sets the shape of the fix. It comes with a trap worth knowing on its own: a redesign written without running the behaviour often re-specifies the defect as its target design, so the rewrite lands and the bug survives it. See [When two reviews hit the same code](SKILL.md#when-two-reviews-hit-the-same-code).

## Ordered by damage, grouped by root

Destroyed data first, then irreversible outward actions, then unbounded leaks, then blocked interaction, then rule and contract breaches - with reachability breaking ties only, because it is already inside the bar. [Order the blockers](SKILL.md#order-the-blockers) has the full ladder.

## It also reviews the reviewers

Each pass names what each reviewer's method could not have seen, with evidence: scope of reading, severity set by a guess, one symptom counted twice, shape mistaken for intent, a verdict resting on refactors. That is what tells you which reviewer to trust where next time. See [Audit each reviewer's method](SKILL.md#audit-each-reviewers-method).

## What it does not do

It does not fix anything unless you ask. The merge is a decision artifact, and mixing it with edits makes both harder to review. Decisions that are yours - the bar, an ambiguous scope item, a behaviour two reviewers read opposite ways - come back as a batch of questions with a recommendation each, never as assumptions. Facts are never a question: it looks them up. [Output](SKILL.md#output) has the shape of the result, [Traps](SKILL.md#traps) the failure modes.

## Contributing

See [CONTRIBUTING.md](https://github.com/pilniczek/dev-skills/blob/master/CONTRIBUTING.md) for local setup and the pre-release security scan workflow.
