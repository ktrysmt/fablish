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

## Fresh-context review

Self-critique inside a long working context is unreliable — the working
narrative biases the review toward confirming it. Run verification
through a fresh-context reviewer (an `Agent` call; read-only agent type
when available):

- Input: the DONE-CRITERIA verbatim, plus concrete artifacts (diff,
  file paths, command outputs). NEVER include your working narrative,
  hypotheses, or self-assessment — that contaminates the review.
- Instruction: adversarial framing. "Try to find where these criteria
  fail" outperforms "confirm this is correct".
- Output contract: per-criterion verdict PASS / FAIL / UNVERIFIABLE,
  each with the evidence that supports it.

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
