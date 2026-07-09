# Delegation protocol

## When to delegate

Delegate a workstream only if ALL of these hold:

- Independent: no file, branch, or state shared with another running stream
- Substantial: would take several tool calls to do directly
- Specifiable: you can write its DONE-CRITERIA without doing the work first

Counter-rules:

- Work you can complete directly in a single response (one function
  refactor, reading a few files) is done directly.
- Serial dependencies are executed in order by you, not delegated.
- Integration work (merging results, resolving conflicts, the Phase 3
  verification pass) is never delegated to the member that produced the
  work being integrated.

## Mechanism

- 2+ parallel workstreams → `TeamCreate`, one member per workstream
  (global CLAUDE.md rule: parallel bare `Agent` calls do not correctly
  inherit context such as CLAUDE.md).
- `TeamCreate` not in the session's toolset → parallel `Agent` calls
  batched in one message; the CONTEXT section of the template below
  compensates for the context bare agents do not inherit.
- Exactly one delegation → a single `Agent` call is acceptable.
- Read-only exploration fan-out (finding files, mapping a subsystem) →
  Explore-type agents; they locate, they do not review.

## Delegation prompt template

Every delegation prompt must carry, in this order:

1. CONTEXT — the task contract from `.task/state.md` (GOAL and
   CONSTRAINTS, verbatim), plus the intent behind it: why the task is
   being done and who consumes the result. Members do not see your
   conversation; this is their only source of intent.
2. SCOPE — this workstream's own goal and its own DONE-CRITERIA.
3. BOUNDARIES — files/directories it may modify; everything else is
   read-only. Name them explicitly.
4. TOOLS — the tools, commands, and sources to prefer or avoid, when
   not obvious from the scope.
5. RETURN — exactly what to report back: artifacts produced, paths,
   evidence per criterion with each claim labeled VERIFIED / REASONED /
   ASSUMED, open questions. Its final message is the only thing you
   receive — say so in the prompt.

## Async supervision

- Launch members in one batch, then continue your own workstream while
  they run. Do not block idle on the slowest member.
- On each member result: check its evidence against its DONE-CRITERIA
  before integrating. A member's "done" without evidence is not done —
  re-verify yourself or send it back. A criterion supported only by
  REASONED or ASSUMED claims counts as unverified.
- Drift intervention: if a member's output shows it misread scope, send
  a correction via `SendMessage` quoting the specific contract line it
  violated. Do not silently redo its work — corrected members keep
  their context; a redo throws it away.
