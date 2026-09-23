---
name: merge-reviews
description: >
  Merges two independent reviews of the same change into one urgency-ordered list of blockers,
  separating real defects from shape opinions and from deliberate scope - or wraps a single review
  the same way. Use whenever two reviews, reviewers, agents or tools have commented on the same
  diff, branch or PR and someone has to decide what to actually fix - "compare these reviews",
  "reconcile the feedback", "two reviewers disagree", "which of these findings matter", "I only
  want to fix blockers", "dedupe this feedback" - or when a review lands on a change another review
  already covered. Also for a single review whose findings need sorting into fix-now and
  record-only. Not for three or more reviews.
allowed-tools: Grep, Glob, Read
---

# Merging reviews

Two competent reviews of the same diff are usually not comparable. One ranks by what a user can
break, the other by what the code's shape will cost later. Concatenating them produces a list where
a silent data-loss bug sits below a naming preference, and the author either fixes everything or
loses faith in the list and fixes nothing.

Your job is to produce one list the author can work top to bottom, plus a register of what was
deliberately not fixed, so the next review does not re-file it.

## Gather the reviews

Reviews arrive as files, PR comments, pasted text or messages earlier in this session. Treat them
alike: label each reviewer (A, B) and restate every finding in one line before merging, so the
comparison survives the chat being compacted. When a review exists only in chat, that restatement
is its only durable copy. A finding with no cited location gets located before it is classified,
or listed as unverified. If one review is your own from this session, say so - you share its blind
spots, so audit it hardest.

## Verify first

Findings are claims. Open the cited lines, check any asserted fact about a library, browser or API,
and read the consumers - reachability changes a finding's rank and only the consuming code shows it.
Collapse findings that share one root; reviewers double-count a single cause described as two
symptoms, or one defect present at two call sites.

Say what you could not verify and why. "Needs a shadow-DOM composer to reproduce" is information;
silently dropping it, or silently promoting it, is not.

## Classify with one bar

The bar is the owner's call and it changes the whole list, so it goes first in the question batch
below; sort on your recommended bar meanwhile. The useful axis is whether a finding can be named as **an input producing a wrong output**:

- **Blocker** - you can write the failing scenario: this input, this state, this wrong result. That
  includes latent defects reachable only through a consumer that does not exist yet, if the code is
  a published surface. It also includes a hard project rule the owner states as non-negotiable, and
  a public contract that promises more than it delivers - both produce a wrong result for someone,
  just not through a keystroke.
- **Non-blocker** - a real observation with no scenario attached: duplication, state modelling,
  naming, layering, missing comments, measured-and-acceptable cost. Valuable, recorded, not fixed
  now.
- **Scope** - looks broken, is an unfinished decision. A disabled control with no handler, an
  English string in a product that will be localized later, a feature that stops at the MVP line.

The scope bucket is the one reviewers get wrong most often, because code cannot tell "unfinished"
from "broken". Call something scope only when something written says so - the ticket, the README, a
work report, a design doc, or a test that asserts the behaviour. With no written evidence, it is a
flaw, or it is a question for the owner. Guessing intent from shape is how deliberate behaviour gets
filed as a blocker.

Another review is not that evidence. When one reviewer proposes a design that contains the exact
behaviour another reviewer filed as a defect, that is two reviewers disagreeing, not proof the
behaviour is intended - and it is the strongest signal in the whole merge that the owner has to
decide. Raise it as a question with both readings, rather than letting the confident proposal settle
it.

## When two reviews hit the same code

This is where the merge earns its keep, and where a naive merge fails: one reviewer reports the
symptom, the other reports the shape around it, and each fix alone is wrong.

**The defect sets the priority. The structural finding sets the shape of the fix.**

A symptom fix that leaves the structure repairs one case and leaves the next one to be found again.
A structural fix that ignores the symptom refactors around a live bug and ships it in nicer code.
Merge them into one entry with one fix shape.

Check the structural proposal for the trap: a redesign written without running the behaviour often
**re-specifies the defect as the target design**. If a proposed state machine says "on blur, fall
back to the other source" and the bug *is* that it falls back to a stale source, the rewrite lands
and the bug survives it. Say so in the entry.

## One review or two

The skill takes one review or two.

- **One review:** wrap that reviewer. Skip the comparison and the collision rule; classify, order,
  and audit its method - with no second review, the audit is the only thing naming what nobody
  looked at. Do not invent a second review; recommend one when the audit finds a class it could not
  see.
- **Two reviews:** everything below applies.
- **Three or more:** out of scope. Refuse and merge nothing.

## Order the blockers

Order by damage, group by root:

1. **Damage first** - destroyed or corrupted user data, then irreversible outward actions (something
   sent, submitted, published), then unbounded resource leaks, then blocked interaction the user can
   recover from, then rule and contract breaches.
2. **Reachability only breaks ties.** It is already inside the bar, so using it as the primary sort
   counts it twice and buries the findings that destroy input.
3. **Group by root**, so one commit closes several entries and the author does not fix one call site
   of a defect that lives at three.

## Audit each reviewer's method

The owner learns more from why a reviewer missed a class of finding than from the findings
themselves, and it tells them which reviewer to trust where next time. Say it with evidence, not
adjectives.

Patterns worth naming:

- **Scope of reading.** A review that never opened the consumer, the test suite or the vendored
  dependency will miss the contract defects that live there. Name what it could not have seen.
- **Wrong stop condition.** A finding can be right and its consequence wrong: "it leaks until the
  panel unmounts" when the panel never unmounts. The severity was set by a guess.
- **Symptom counted twice** - the inflation above.
- **Shape mistaken for intent.** Reporting deliberate, documented behaviour as a defect because it
  looks unfinished.
- **A verdict resting on refactors.** A review that blocks a change on four reshapes while four
  reproducible defects pass unremarked will get the code rewritten and every bug re-shipped.
- **Unverified facts asserted flatly** - a claim about a library, a browser or an API stated as
  given, which a two-minute check confirms or kills.

Be even-handed: each reviewer usually caught something the other structurally could not, and that is
the argument for running both.

## Ask the owner, in batches

The bar, an ambiguous scope item, a two-repo finding, a behaviour two reviewers read opposite ways -
these are the owner's calls. Gather them, ask them together with a recommendation each, and wait.
The answers are reusable: they define the bar for the next review of this codebase. Facts are never
a question for the owner - look them up.

## Output

Lead with the merge rules you applied, so the ranking can be argued with rather than trusted. Then a
finding-by-finding comparison marking which reviewer raised what: the shared findings are usually
few, and what only one reviewer could see is the interesting part.

Then the blockers, ordered, one entry each:

```
### N. <one-line defect, not a file name>

`file.ts:12-38` - reviewer A finding 1 + reviewer B finding 5

**Failure**: the concrete scenario. Inputs, state, wrong result.
**Fix shape**: the repair, including the structural half when there is one.
**Reachability**: today on which path, or latent and why.
```

Close with the non-blocker register - one line and one reason each, a record rather than a second
argument - the scope items with the evidence that made them scope, what you could not verify, and
the method audit.

Ask where the result should live. Chat alone loses it; a fresh file that is neither committed nor
ignored gets read next session as if it were current. A file the project already uses for handoff
notes, a tracker, or the PR itself are the durable options.

## Traps

- **Do not merge by severity labels.** Different reviewers' "critical" and "blocker" mean different
  things. Re-derive severity yourself from the failure scenario.
- **Do not let volume imply quality.** Thirteen structural findings and six defects is not a
  reviewer being twice as useful; it is two different axes.
- **Do not fix while merging** unless asked. The merge is a decision artifact; mixing it with edits
  makes both harder to review.
- **Do not re-argue a non-blocker.** Once it is off the fix list, one line is the whole entry.
