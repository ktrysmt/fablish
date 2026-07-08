# Reporting protocol

## Final message = re-grounding

The reader saw none of your tool calls, none of your thinking, and none
of the labels you coined while working. Write for someone catching up,
in the output language required by the global CLAUDE.md rules.

Order:

1. Outcome — one or two sentences: what happened / what you found
2. Evidence per DONE-CRITERION — criterion, then how it was verified
   (command and result, `file:line`, reviewer verdict)
3. Autonomous decisions — minor choices you made under the
   minor-choice rule: what you picked and why
4. Open items — what failed, what was skipped, what needs the user

## Prohibitions

- No arrow chains (`A → B → fails`), hyphen-stacked compounds, or
  abbreviations invented mid-session
- No labels or codenames from the working thread without
  re-introduction in place
- No packing multiple identifiers into one parenthesized run or
  slash-separated list — each file, commit, or flag gets its own
  plain-language clause saying what it is or what changed
- No hedged completion ("should work now") — verified work is stated
  plainly; unverified work is labeled unverified

## During the work

Between tool calls, default to silence. Write one sentence when you
find something load-bearing, change direction, or hit a blocker. Do not
narrate routine actions ("Now I'll...", "Let me check..."). Progress
updates follow the global grounded-claims rule: point at evidence, or
mark the claim as unverified.
