---
name: fablish
description: >-
  Fable-style long-horizon execution protocol — a procedural mimicry of
  Claude Fable 5's documented working style: goal contract before acting,
  decomposition into parallel workstreams, evidence-grounded checkpoints,
  fresh-context verification, re-grounded final reporting, and
  issue-based upstreaming of universal lessons. Trigger on multi-step
  work expected to span many tool calls
  (feature implementation, refactors, migrations, investigations,
  multi-file changes), or when the user invokes /fablish. Do NOT trigger
  for single-file edits, lookups, or conversational questions.
argument-hint: "[task description]"
---

# fablish — long-horizon execution protocol

Procedural mimicry of Claude Fable 5's working style. This skill encodes
the procedure (when to decide what), not the capability itself.

Input `$ARGUMENTS`: optional task description. If present, use it as the
task for Phase 0; otherwise use the task from the conversation.

Tier-1 invariants in the global CLAUDE.md (Grounded claims, Act on
sufficiency, No promissory endings, Minor-choice autonomy) are assumed
present and are NOT restated here — do not duplicate them into working
files or delegation prompts.

## Calibration

If you are running as a Fable-class model, treat the phases below as
reminders — do not mechanically narrate each phase. On other models
(Opus, Sonnet, Haiku), follow each phase's gate explicitly and in order.

## Escape hatch

If, on reading the task, it clearly fits in a handful of tool calls
(single-file edit, one lookup, a rename), say so in one sentence and
proceed without this protocol. Do not create `.task/` artifacts for
trivial work.

## Working files

- `.task/state.md` — task contract + checkpoints (current task only)
- Ensure `.task/` is ignored via `.git/info/exclude` (not the shared
  `.gitignore`); never commit it.

## Phase 0 — Task contract (before any mutating tool call)

Write `.task/state.md`:

- GOAL: the outcome in one sentence, phrased as the user would verify it
- CONSTRAINTS: what must not change or be touched
- DONE-CRITERIA: 3–7 independently checkable criteria
  ("tests in X pass", "command Y exits 0" — not "code looks good")
- NON-GOALS: adjacent work you will explicitly not do
- RISKIEST-ASSUMPTION: the single assumption most likely to invalidate
  the plan, plus the cheapest check that confirms or kills it

Gate: if you cannot write checkable DONE-CRITERIA, ask ONE batched
clarifying question, then proceed with stated assumptions. Never ask a
second round.

## Phase 1 — Decompose by independence, not by sequence

Split the goal into workstreams and classify each:

- INDEPENDENT (no shared files or state with other streams) → delegate
- SEQUENTIAL or trivially small → do directly

When 2+ independent workstreams exist, delegate via `TeamCreate` (one
member per workstream), per the global rule; exactly one delegation may
use a single `Agent` call. Work asynchronously: keep working while
members run; intervene if one drifts. Do NOT delegate work you can
complete directly in a single response.

Sequence work so the RISKIEST-ASSUMPTION check runs first, while
invalidation is still cheap — before any expensive workstream starts or
is delegated. A plan killed after two minutes of reading is free.

Details and the delegation prompt template: `references/delegation.md`

## Phase 2 — Execute with grounded checkpoints

After each meaningful unit of work, append one checkpoint to
`.task/state.md`: what was done, the tool-result evidence for it, and
what is next. Label every claim VERIFIED (executed and observed),
REASONED (follows from code read, not executed), or ASSUMED (plausible,
unchecked); never upgrade a label without new evidence. If interrupted
or compacted, this file is the resume point — trust it over memory.

## Phase 3 — Verify with fresh context

Before declaring completion, run the checking method decided in Phase 0
(test run, lint, diff review, rubric grading) through a fresh-context
reviewer that receives ONLY the DONE-CRITERIA and the artifacts — never
your working narrative. Fresh verifiers outperform self-critique.
Iterate until every criterion passes, or state precisely which criteria
fail and why.

Details and the rubric template: `references/verification.md`

## Phase 4 — Report by re-grounding

The final message is for a reader who saw none of your work. Outcome
first, then evidence per DONE-CRITERION, then open items. Drop all
working shorthand. Details: `references/reporting.md`

## Lessons (cross-cutting)

Do not keep a local lessons pool: unreviewed cross-task memory goes
stale and biases future contracts. Route durable insights instead:

- Universal, protocol-level insight (a correction, a surprising result,
  a pattern independent of any project) → file it upstream with
  `gh issue create --repo ktrysmt/fablish --label lesson`, body
  following `.github/ISSUE_TEMPLATE/lesson.md` in that repo. English,
  one insight per issue, no attribution footer. Issues, not PRs — many
  concurrent sessions run this skill; the maintainer integrates.
- Project-specific convention → the project's own CLAUDE.md or
  knowledge base, through normal review.
- Mutable state (versions, branches, what is installed) → never
  persist; re-verify at point of use.
