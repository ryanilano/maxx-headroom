# Session changes — 2026-07-23

Local-only notes. Excluded from git via `.git/info/exclude`.

## State at end of session

| | |
|---|---|
| Branch | `chore/cli-string-punctuation` (still checked out) |
| Commit | `4c99f97` (amended once from `69bc3f7`) |
| Remote branch | `4c99f97`, force-pushed with `--force-with-lease` |
| `origin/main` | `106218c` — **unchanged, never pushed to** |
| PR | https://github.com/ryanilano/maxx-headroom/pull/1 |
| PR state | `OPEN`, `mergedAt: null` — **not merged** |

## Files changed by the commit

```
M  CHANGELOG.md      +5
A  CLAUDE.md         +47
M  bin/maxx          46 lines changed (punctuation) + 2 (sibling fix)
```

`README.md`, `RUNBOOK.md`, `ROADMAP.md`, `docs/`, and `notes/` untouched by the
commit — verified with `git diff --name-only main..HEAD -- <paths>` (empty).

### 1. `CLAUDE.md` (new, repo root)

Two-tier punctuation rule.

- **Tier 1** — user-facing strings in `bin/maxx` (`warn`/`err`/`die`/`note`/`ok`/
  bare `printf`/the `usage` heredoc) and comments: no em-dashes. Three stated
  rationales: mojibake in non-UTF-8 terminals and CI logs; not greppable; the 46
  repetitions of one "problem — remedy" shape made `doctor` monotone. Prescribes
  varied substitutes and forbids rewording.
- **Tier 2** — prose docs: em-dashes stay, rare and varied. Explicitly warns
  against mechanically stripping them, because a doc with zero reads
  over-edited.

Note it sits at the repo **root**, separate from the untracked `.claude/`
directory that was already present.

### 2. `bin/maxx`

All 46 em-dashes removed, in strings and comments. Shape distribution as
measured by the variety verifier:

| Shape | Count | % |
|---|---|---|
| Sentence split | 17 | 37% |
| Semicolon | 13 | 28% |
| Colon | 10 | 22% |
| Parentheses | 4 | 9% |
| Other (comma; parenthetical recast in `usage`) | 2 | 4% |

Plus the consolidated sibling fix in the amend:

- `bin/maxx:427` — semicolon changed to colon
- `bin/maxx:433` — restored to semicolon so it matches `bin/maxx:184` byte for byte

### 3. `CHANGELOG.md`

New `## 2026-07-23` section at the top, correct for the file's newest-first
ordering:

```
- Added [CLAUDE.md](CLAUDE.md) with a two-tier punctuation rule: no em-dashes in
  `bin/maxx` output, rare and varied em-dashes in prose docs.
- Removed all 46 em-dashes from `bin/maxx`, in strings and comments. Replacements
  vary by message (colon, semicolon, parentheses, sentence split) so `maxx doctor`
  no longer reads as one repeated construction. No message changed meaning or wording.
```

## Verification results

Three concurrent fresh-context verifiers, all **CONFIRMED**.

**Mechanical DoD.** Ten checks. `grep -c '—'` → 0; byte-level scan found zero
U+2014 and zero lookalikes (en-dash, figure dash, horizontal bar, minus sign,
fullwidth hyphen). Only non-ASCII remaining is 3× `§`, confirmed pre-existing via
`git show main:bin/maxx`. `shellcheck` 0.11.0 clean, validated with a negative
control (bad path → exit 2) to prove the clean exit was not a silent no-op.
Valid UTF-8, no BOM, no CRLF, trailing newline present. Remote head matches
local.

**Wording.** Per-site read of all 46 with a whitespace-run-aware tokenizer. No
word added, dropped, substituted, or reordered. 16 capitalizations, every one
following a newly introduced period, none after `;`, `:`, or `(`. The deliberate
triple space in `run: $wrapper   then /login` survives at both sites (369, 564).
Only three `printf` lines changed, none containing `%` conversions; `\n` counts
identical.

**Variety.** Built a 7-profile fixture producing 26 failures, plus mutated
fixtures for every branch that cannot co-occur. Verdict: not monotonous. The
`== watched links ==` block was called the clearest evidence against a global
pattern — four sibling branches in one `if/elif` chain, four different marks.

### Post-amend re-checks

`grep -c '—'` → 0. `shellcheck` → clean, exit 0. `bash -n` → ok.
`grep -o '"node not on PATH[^"]*"' | sort -u` → one unique literal, confirming
184 and 433 match.

**Not re-run after the amend:** `maxx doctor`. See `SESSION-HISTORY.md`.

## Open items

1. **PR #1** — open, not merged, awaiting review.
2. **Follow-up for the `(run: maxx setup)` triple** — **not started, decision
   pending.** Three options were put to Ryan:
   - **A (recommended)** — drop the per-line pointer; emit the hint once beside
     the existing `N problem(s), M warning(s)` summary when any setup-fixable
     failure occurred. Collapses six identical strings to one on a fresh
     two-profile machine. Touches control flow, so higher risk than the
     punctuation pass.
   - **B** — keep the pointer on the first failure per profile block only. Less
     code, less benefit.
   - **C** — keep the pointers, make each line say what is specifically wrong.
     Most editorial work, most reader value, biggest diff.

   Confirmed safe to proceed on the docs front: no doc contains a fenced block
   of doctor output, so this would not invalidate README or RUNBOOK — subject to
   the `RUNBOOK.md:35` prefix caveat in `SESSION-HISTORY.md`.
3. **`README.md` has an uncommitted 1-line change** predating this session,
   deliberately left alone and still dirty in the working tree.
4. **`.claude/` is untracked** — was untracked on arrival, never addressed.
   Holds `settings.json` and `settings.local.json`.
5. **Still on `chore/cli-string-punctuation`**, not switched back to `main`.

## Standing instruction recorded

"All future commits opened on a branch, never pushed to `main`." Saved to
persistent memory as `branch-never-push-main`, so it applies in future sessions
and not just this one. Merging remains Ryan's call.
