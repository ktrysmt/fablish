# fablish

Contract-driven long-horizon execution protocol for Claude Code — a
procedural mimicry of Claude Fable 5's documented working style.

[日本語版 README](./README-ja.md)

## What it does

When a task is expected to span many tool calls (feature implementation,
refactors, migrations, investigations, multi-file changes), this skill
makes the agent:

1. Decide first whether a change was requested at all — a described
   problem gets an assessment, not a fix
2. Write a task contract (GOAL / CONSTRAINTS / DONE-CRITERIA / CHECK /
   NON-GOALS / RISKIEST-ASSUMPTION) before any mutating tool call
3. Decompose work by independence, delegate parallel workstreams
   (falling back to batched parallel `Agent` calls when `TeamCreate` is
   absent), and check the riskiest assumption first while invalidation
   is still cheap
4. Record evidence-grounded checkpoints — each claim labeled VERIFIED /
   REASONED / ASSUMED, never silently upgraded — so interrupted work can
   resume, and verify at intervals on long runs so failures surface
   while they are cheap
5. Verify completion through a fresh-context reviewer that sees only the
   DONE-CRITERIA and artifacts, never the working narrative — scaling to
   majority-vote independent reviewers for audit-grade work
6. Report by re-grounding: outcome first, evidence per criterion
7. Keep lessons in a curated, committed `.claude/lessons/` store (update
   over duplicate, delete what proves wrong) and, with explicit user
   confirmation, upstream universal protocol insights as GitHub issues
   on ktrysmt/fablish

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
it in `.git/info/exclude` so it is never committed. Cross-task lessons
live separately in `.claude/lessons/` — committed and reviewed like any
other change, and curated on every write (update over duplicate, delete
what proves wrong) — because unreviewed persistent memory goes stale and
biases future runs. Universal protocol insights leave, with explicit
user confirmation, as GitHub issues on this repo; project-specific
conventions go to that project's CLAUDE.md or knowledge base.

## Notes

- The skill references Tier-1 invariants (grounded claims, act on
  sufficiency, no promissory endings, minor-choice autonomy) and a
  TeamCreate-over-parallel-subagents rule assumed to live in your global
  CLAUDE.md. It degrades gracefully without them: when `TeamCreate` is
  not in the session's toolset, delegation falls back to parallel
  `Agent` calls batched in one message.
- On Fable-class models the phases act as reminders; on other models
  (Opus, Sonnet, Haiku) each phase gate is followed explicitly.

## References

fablish is a procedural transcription of publicly documented behavior.
These are the sources the design was built and revised against.

Anthropic, on Fable 5's working style:

- [Introducing Claude Fable 5 and Claude Mythos 5](https://www.anthropic.com/news/claude-fable-5-mythos-5)
  — long-horizon autonomy; the model "improves its outputs using its
  own notes"; in the Slay the Spire evaluation, persistent file-based
  memory improved performance three times more than for Opus 4.8.
- [Claude Fable 5 & Claude Mythos 5 System Card](https://anthropic.com/claude-fable-5-mythos-5-system-card)
  — primary documentation of the model's behavior and evaluations.
- [Prompting Claude Fable 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5)
  — the most load-bearing source. Grounds: auditing progress claims
  against tool results (Phase 2 checkpoints), "separate, fresh-context
  verifier subagents tend to outperform self-critique" (Phase 3),
  parallel subagents with asynchronous supervision (Phase 1), acting
  on sufficiency and never ending a turn on a promise, re-grounded
  final communication for a reader who saw none of the work (Phase 4),
  and constructing a curated memory of lessons from runs (the
  `.claude/lessons/` store). Issue-based upstreaming is this skill's
  own mechanism, not a documented Fable behavior.

Anthropic, on long-horizon agent engineering:

- [Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
  — compaction, structured note-taking, and sub-agent architectures as
  the techniques for working beyond one context window; grounds the
  `.task/state.md` checkpoint design.
- [Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)
  — agents resume by reading a progress-notes file and re-verifying
  state before new work; grounds the resume-from-state.md rule.
- [Building effective agents](https://www.anthropic.com/engineering/building-effective-agents)
  — the orchestrator-workers and evaluator-optimizer patterns (Phase 1
  delegation; the Phase 3 verify-fix loop), and "finding the simplest
  solution possible, and only increasing complexity when needed" (the
  escape hatch).
- [How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system)
  — "each subagent needs an objective, an output format, guidance on
  the tools and sources to use, and clear task boundaries": grounds the
  delegation prompt template (CONTEXT / SCOPE / BOUNDARIES / TOOLS /
  RETURN),
  and why parallel subagents win on breadth-first decomposable work.
- [Claude Code best practices](https://code.claude.com/docs/en/best-practices)
  — give the agent a check it can run, and have it show evidence rather
  than assert success (DONE-CRITERIA, checkpoints); an adversarial
  reviewer in a fresh subagent context "sees only the diff and the
  criteria you give it, not the reasoning that produced the change"
  (Phase 3); "if you could describe the diff in one sentence, skip the
  plan" (the escape hatch).

Supporting research:

- Huang et al., [Large Language Models Cannot Self-Correct Reasoning Yet](https://arxiv.org/abs/2310.01798)
  (ICLR 2024) — LLMs do not reliably correct their own reasoning
  without external feedback; the premise for routing verification
  through an external fresh-context reviewer instead of self-critique.

Comparative sources behind later revisions:

- [toffyui/ccteams](https://github.com/toffyui/ccteams)
  — origin of the three-tier evidence labels (VERIFIED / REASONED /
  ASSUMED) and riskiest-assumption-first sequencing, adopted in v0.2.0.
- [Claude Code memory](https://code.claude.com/docs/en/memory.md) and
  the [memory tool](https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool.md)
  — surveyed for v0.3.0's decision to keep no local cross-task memory
  pool, revisited in v0.4.0: the local store returned as a curated,
  committed `.claude/lessons/` directory (per the prompting guide's
  memory-system guidance), kept reviewed so it cannot silently go
  stale.

## License

MIT
