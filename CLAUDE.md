# CLAUDE.md

Repo conventions for agents working in maxx-headroom.

## Punctuation: two tiers

Em-dashes are treated differently in code and in prose. The rule is not "remove
every em-dash"; it is "none in CLI output, and few and varied in docs."

### Tier 1: user-facing strings in `bin/maxx` — no em-dashes

Anything printed by `warn`, `err`, `die`, `note`, `ok`, a bare `printf`, or the
`usage` heredoc must not contain an em-dash. Comments in `bin/maxx` follow the
same rule, so a later copy-paste into a string cannot reintroduce one.

Three reasons:

1. They render badly in non-UTF-8 terminals and in CI logs, where a multibyte
   character can come out as mojibake in the middle of an error message.
2. They are awkward to grep. Nobody types an em-dash at a shell prompt, so
   searching for a message means searching around the dash instead of for it.
3. Overuse flattens the output. Before this rule, `bin/maxx` carried 46
   em-dashes, nearly all of them the same "problem — remedy" construction, which
   made `maxx doctor` read as one long monotone list.

Replace with a colon, a semicolon, parentheses, or a sentence split, choosing per
message so that adjacent lines of output do not land on the same shape. Vary the
choice; do not apply one substitution pattern across the file. Reach for:

- a **colon** when the second half explains the first
- a **semicolon** when the two halves are equal and closely linked
- **parentheses** for an aside, especially a `(run: maxx setup)` style pointer
- a **sentence split** when the second half is an imperative

Punctuation is the only thing that changes. Never reword a message, alter its
meaning, or drop detail in order to fit a shorter form.

### Tier 2: prose docs — rare and varied em-dashes

In `README.md`, `RUNBOOK.md`, `ROADMAP.md`, `CHANGELOG.md`, `docs/`, and
`notes/`, em-dashes are allowed. Do not mechanically strip them: prose with zero
em-dashes reads over-edited, and a global find-and-replace produces exactly that.

Keep them rare and keep them varied. If a paragraph already has one, the next
break should be a colon, parentheses, a semicolon, or a new sentence. The test is
whether the punctuation is doing work the alternatives could not do as well; if
another mark reads at least as cleanly, use the other mark.

## Working with Ryan: operating rules

Deliver, do not discuss. Do the thing first; explain only when asked. No
hedging, no unsolicited caveats, no "here is why," no offers of extra help, and
never tell Ryan to rest or sleep.

Every command goes in a fenced code block with literal paths. No `<placeholders>`
and no bare `$VARS` in prose; his zsh mangles both. Claude Code project dirs
start with `-`, so glob as `./*/*.jsonl`, never `*/*.jsonl`.

One decision at a time. When a choice is needed, present A/B/C and take a single
letter back. When Ryan repeats a request, the previous delivery failed: change
the format and re-ship, do not argue it was already done.

Act on reversible work and report after; do not ask permission for safe steps.
No yes-man behavior: state what went wrong in one line, fix it, and skip the
self-abasement. Ryan has ADHD, so front-load the answer and cut the preamble.
His name in ALL CAPS is load-bearing. Verify with real commands, never with
assessments.
