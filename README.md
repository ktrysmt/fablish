# fablish

Contract-driven long-horizon execution protocol for Claude Code — a
procedural mimicry of Claude Fable 5's documented working style.

[日本語版 README](./README-ja.md)

## What it does

When a task is expected to span many tool calls (feature implementation,
refactors, migrations, investigations, multi-file changes), this skill
makes the agent:

1. Write a task contract (GOAL / CONSTRAINTS / DONE-CRITERIA / NON-GOALS /
   RISKIEST-ASSUMPTION) before any mutating tool call
2. Decompose work by independence, delegate parallel workstreams, and
   check the riskiest assumption first while invalidation is still cheap
3. Record evidence-grounded checkpoints — each claim labeled VERIFIED /
   REASONED / ASSUMED, never silently upgraded — so interrupted work can
   resume
4. Verify completion through a fresh-context reviewer that sees only the
   DONE-CRITERIA and artifacts, never the working narrative
5. Report by re-grounding: outcome first, evidence per criterion
6. Feed universal, protocol-level insights back upstream as GitHub
   issues on ktrysmt/fablish (one insight per issue, following the
   repo's issue template) instead of keeping a local memory pool

Trivial work (single-file edits, lookups) is explicitly exempted via an
escape hatch, so the protocol does not add ceremony where none is needed.

## Install

```console
claude plugin marketplace add ktrysmt/claude-plugins
claude plugin install fablish@ktrysmt
```

## Usage

Invoke explicitly:

```
/fablish <task description>
```

or let it trigger automatically on multi-step work. To make it the
default for all long-horizon tasks, add a rule like this to your global
`~/.claude/CLAUDE.md`:

```markdown
- Long-horizon tasks: For multi-step work expected to span many tool
  calls, invoke the `fablish` skill before starting
```

## Working files

The skill keeps its state in a `.task/` directory at the project root
(`state.md` for the current task contract and checkpoints) and registers
it in `.git/info/exclude` so it is never committed. State is scoped to a
single task on purpose: the skill maintains no cross-task memory pool,
because unreviewed persistent memory goes stale and biases future runs.
Durable insights leave through review instead — universal ones as GitHub
issues on this repo, project-specific ones into that project's CLAUDE.md
or knowledge base.

## Notes

- The skill references Tier-1 invariants (grounded claims, act on
  sufficiency, no promissory endings, minor-choice autonomy) and a
  TeamCreate-over-parallel-subagents rule assumed to live in your global
  CLAUDE.md. It degrades gracefully without them, but works best when
  those rules are present.
- On Fable-class models the phases act as reminders; on other models
  (Opus, Sonnet, Haiku) each phase gate is followed explicitly.

## License

MIT
