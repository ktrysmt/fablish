# Verification protocol

## Build the harness before you need it

Decide during Phase 0 how completion will be checked — not after the
work is done. Preference order:

1. Executable checks: tests, build, lint, typecheck, a reproducible
   command with an expected exit code or output
2. Behavioral probe: run the actual app/command and observe the change
   working (not just tests passing)
3. Rubric review: only when nothing executable exists (docs, designs,
   analyses)

If the changed behavior has no test coverage and adding a check is
cheap, add it as part of the work — it becomes a DONE-CRITERION.

## Interval verification (long runs)

Decide the cadence in Phase 0: every few checkpoints, or whenever a
workstream lands. At each interval, run the fresh-context check below
against the criteria that should already hold, and record the verdict
in the checkpoint. A criterion that passed at an interval re-runs in
the final pass if later work touched what it checks.

## Fresh-context review

Self-critique inside a long working context is unreliable — the working
narrative biases the review toward confirming it. Run verification
through a fresh-context reviewer (an `Agent` call; read-only agent type
when available):

- Input: the DONE-CRITERIA and REJECTED-DELIVERABLES verbatim, the
  domain failure-mode checklist (below), plus concrete artifacts (diff,
  file paths, command outputs). NEVER include your working narrative,
  hypotheses, or self-assessment — that contaminates the review.
- Instruction: adversarial framing. "Try to find where these criteria
  fail" outperforms "confirm this is correct".
- Output contract: per-criterion verdict PASS / FAIL / UNVERIFIABLE,
  each with the evidence that supports it.
- Claim labels are inputs, not verdicts: VERIFIED / REASONED / ASSUMED
  label the worker's claims; PASS / FAIL / UNVERIFIABLE are the
  reviewer's verdicts. An ASSUMED claim can never support a PASS.

## Domain failure-mode checklist

Generic adversarial framing misses domain-specific counterfeits. While
building the harness (Phase 0), derive 3–7 ways this kind of
deliverable characteristically fails, and hand the list to every
reviewer alongside the criteria. Examples:

- Code change: narrows the input space silently, weakens or deletes a
  test to make it pass, handles the reproduced case but not its class
- Migration: rows dropped or duplicated at chunk boundaries, no checked
  rollback, mixed-version window ignored
- Analysis: a claim supported by its own restatement, a sample
  presented as the population, an uncited number
- Config/infra: applied but not effective (cached, wrong scope,
  overridden elsewhere)

Two checks belong on every list:

- Difficulty displacement: the hard part was moved, not solved — into a
  TODO helper, an assumed dependency, a manual step in the report, or a
  quietly weakened criterion. Have the reviewer answer: where did the
  original difficulty go?
- Goal substitution: an artifact matching a REJECTED-DELIVERABLES entry
  is FAIL, even when every DONE-CRITERION nominally passes.

## Scale rigor to the ask

- Default: one fresh-context reviewer.
- Audit-grade asks ("thoroughly", "comprehensive", security-sensitive)
  or hard-to-reverse changes: three independent reviewers given the
  same criteria; a criterion passes only on a majority. Distinct
  lenses (correctness, security, does-it-reproduce) catch more than
  three identical reviewers.

## Rubric template (non-executable deliverables)

| # | Criterion (independently checkable) | How to check | Verdict |
|---|-------------------------------------|--------------|---------|

Criteria must be checkable in isolation: "the CSV has a numeric `price`
column per SKU", not "the data looks good". Vague criteria produce
noisy verify-fix loops that never converge.

## Iterate loop

FAIL → fix → re-verify the failed criteria (re-run the full set if the
fix touched shared code). Cap at 3 iterations. If criteria still fail
at the cap, stop and report the failing criteria with evidence — a
precise failure report is a valid outcome; grinding past the cap is not.
