---
name: fablish
description: >-
  Fable-style long-horizon execution protocol — a procedural mimicry of
  Claude Fable 5's documented working style: goal contract before acting,
  decomposition into parallel workstreams, evidence-grounded checkpoints,
  interval and fresh-context verification, re-grounded final reporting,
  and curated lessons routed through reviewed channels. Trigger on
  multi-step work expected to span many tool calls
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
proceed without this protocol. Do not create a state file for
trivial work.

## Working files

- `/tmp/fablish/$CLAUDE_CODE_SESSION_ID/state.md` — task contract +
  checkpoints (current task only). Resolve `$CLAUDE_CODE_SESSION_ID`
  from the environment once at task start, then use the concrete path.
  The location is session-scoped and volatile by design: it never
  collides across concurrent sessions on one repo, and never enters the
  working tree — so there is no `.gitignore` or `.git/info/exclude` step
  and no risk of committing scratch state. Cross-session (app-restart)
  resume is not a goal — durable memory goes to `.claude/lessons/` and
  GitHub issues, not here.
- `.claude/lessons/` — cross-task lessons, one file per lesson;
  committed and reviewed like any other change (see Lessons)

## Phase 0 — Task contract (before any mutating tool call)

First decide whether a change was requested at all. When the user is
describing a problem, asking a question, or thinking out loud, the
deliverable is an assessment: investigate, report, and stop — do not
apply fixes until asked.

If the project has `.claude/lessons/`, read it now — prior lessons
shape CONSTRAINTS and RISKIEST-ASSUMPTION.

Write the state file `/tmp/fablish/$CLAUDE_CODE_SESSION_ID/state.md`:

- GOAL: the outcome in one sentence, phrased as the user would verify it
- CONSTRAINTS: what must not change or be touched
- DONE-CRITERIA: 3–7 independently checkable criteria
  ("tests in X pass", "command Y exits 0" — not "code looks good")
- CHECK: how completion will be verified (method per
  references/verification.md) and, on long runs, the interval cadence
- NON-GOALS: adjacent work you will explicitly not do
- RISKIEST-ASSUMPTION: the single assumption most likely to invalidate
  the plan, plus the cheapest check that confirms or kills it

Gate: if you cannot write checkable DONE-CRITERIA, ask ONE batched
clarifying question, then proceed with stated assumptions. Never ask a
second round for preferences — but ambiguity that decides a destructive
action or the scope itself is the user's to resolve, whenever it
surfaces.

## Phase 1 — Decompose by independence, not by sequence

Split the goal into workstreams and classify each:

- INDEPENDENT (no shared files or state with other streams) → delegate
- SEQUENTIAL or trivially small → do directly

When 2+ independent workstreams exist, delegate via `TeamCreate` (one
member per workstream), per the global rule; exactly one delegation may
use a single `Agent` call. If `TeamCreate` is not in the session's
toolset, batch parallel `Agent` calls in one message instead — the
delegation template carries the context bare agents do not inherit.
Work asynchronously: keep working while members run; intervene if one
drifts. Do NOT delegate work you can complete directly in a single
response.

Sequence work so the RISKIEST-ASSUMPTION check runs first, while
invalidation is still cheap — before any expensive workstream starts or
is delegated. A plan killed after two minutes of reading is free.

Details and the delegation prompt template: `references/delegation.md`

## Phase 2 — Execute with grounded checkpoints

After each meaningful unit of work, append one checkpoint to
the state file: what was done, the tool-result evidence for it, and
what is next. Label every claim VERIFIED (executed and observed),
REASONED (follows from code read, not executed), or ASSUMED (plausible,
unchecked); never upgrade a label without new evidence. If interrupted
or compacted, this file is the resume point — trust it over memory.

On long runs, verify at intervals, not only at the end: at the cadence
decided in Phase 0 (every few checkpoints, or when a workstream lands),
run the Phase 3 check against the criteria that should already hold and
record the verdict in the checkpoint. An early FAIL is cheap; the same
FAIL found at the end is not.

## Phase 3 — Verify with fresh context

Before declaring completion, run the checking method decided in Phase 0
(test run, lint, diff review, rubric grading) through a fresh-context
reviewer that receives ONLY the DONE-CRITERIA and the artifacts — never
your working narrative. Fresh verifiers outperform self-critique.
Scale rigor to the stakes: one reviewer by default, independent
majority-vote reviewers for audit-grade or hard-to-reverse work.
Iterate until every criterion passes, or state precisely which criteria
fail and why.

Details and the rubric template: `references/verification.md`

## Phase 4 — Report by re-grounding

The final message is for a reader who saw none of your work. Outcome
first, then evidence per DONE-CRITERION, then open items. Drop all
working shorthand. Details: `references/reporting.md`

## Lessons (cross-cutting)

Unreviewed cross-task memory goes stale and biases future contracts,
so every channel below is either curated in place or reviewed on the
way in:

- Lesson learned in this project (a correction, a confirmed approach,
  a surprising result) → one file per lesson under `.claude/lessons/`,
  one-line summary at the top, evidence labels kept. Curate on every
  write: update an existing note instead of duplicating it; delete
  notes that turn out to be wrong. The directory is committed and
  reviewed like any other change.
- Universal, protocol-level insight (a pattern independent of any
  project) → propose upstreaming it as a GitHub issue on
  ktrysmt/fablish (label `lesson`, body per
  `.github/ISSUE_TEMPLATE/lesson.md` in that repo; English, one insight
  per issue, no attribution footer). Filing an issue publishes content
  externally: show the user the exact title and body and get their
  confirmation before running `gh issue create`. Issues, not PRs —
  many concurrent sessions run this skill; the maintainer integrates.
- Project-specific convention (style, tooling, layout) → the project's
  own CLAUDE.md or knowledge base, through normal review.
- Mutable state (versions, branches, what is installed) → never
  persist; re-verify at point of use.
