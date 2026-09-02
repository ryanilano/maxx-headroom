# Fable brief — de-duplicate the `(run: maxx setup)` pointer in `maxx doctor`

**Local-only file.** Not for commit. It is a handoff brief; delete it once the
work lands. (If `.git/info/exclude` does not already list it, add it.)

**Repo:** `~/local-dev/maxx-headroom`, currently on `main` at a merged, clean
state. **Touch only `bin/maxx` and `CHANGELOG.md`.** Nothing else.

---

## The problem

`maxx doctor` attaches `(run: maxx setup)` (and a few `; run: maxx setup`
variants) to each individual failure it emits. When more than one setup-fixable
thing is broken, the same remedy prints on line after line. The worst case is a
fully-broken profile, where three consecutive lines each end the same way:

```
FAIL config dir missing: /Users/x/.claude-alt (run: maxx setup)
FAIL wrapper missing or not executable: /Users/x/bin/claude-alt (run: maxx setup)
FAIL ccmanager preset 'alt' missing (run: maxx setup)
```

On a fresh two-profile machine that is six identical strings. The remedy is
useful once; repeated per line it is noise, and it reads like a template. This
is the exact monotony the just-merged punctuation rule (see `CLAUDE.md`, tier 1)
set out to kill, left over because it is a *wording* problem, not a punctuation
one, so it was kept out of that PR.

## The decision (chosen — do this)

**Option A: say it once.** Stop attaching the pointer to each failure. Instead,
track whether any setup-fixable failure occurred, and print the remedy a single
time next to the final summary line.

Why A over the alternatives: it removes the repetition entirely rather than
thinning it, and it puts the remedy where a reader looks after scanning the
problems — at the bottom, with the count. B and C (below) are fallbacks only.

### Exactly what to change

1. **Add a tracker**, next to the existing `FAILS` / `WARNS` counters near the
   top of the file:

   ```sh
   SETUP_FIXABLE=0
   ```

2. **Strip the inline pointer from these 11 sites and set the flag instead.**
   These are every `bin/maxx` output string that currently names `maxx setup`
   *inside `cmd_doctor`* (line numbers are from the current `main`; confirm by
   grep, do not trust them blindly):

   | Line | Current tail to remove |
   |---|---|
   | 443 | `(run: maxx setup)` |
   | 458 | `(run: maxx setup)` (keep `; checking shipped defaults below`) |
   | 465 | `(run: maxx setup)` |
   | 494 | `(run: maxx setup)` |
   | 504 | `(run: maxx setup)` |
   | 513 | `(run: maxx setup)` |
   | 524 | `; run: maxx setup` |
   | 531 | `; run: maxx setup` |
   | 549 | `; run: maxx setup (pilotfish sync step)` — see note below |
   | 553 | `(run: maxx setup)` |

   At each of these, after the `err`/`warn` call, set `SETUP_FIXABLE=1`.

   Cleanest way to avoid 11 hand-edits that each have to remember the flag: add
   two tiny helpers next to `err`/`warn` that wrap them:

   ```sh
   errfix()  { err "$*";  SETUP_FIXABLE=1; }
   warnfix() { warn "$*"; SETUP_FIXABLE=1; }
   ```

   Then change those 11 sites from `err "...(run: maxx setup)"` to
   `errfix "..."` (and the two `warn` ones to `warnfix`), with the pointer text
   removed. This keeps the flag and the message in one place and is easy to
   review.

3. **Print the remedy once**, at the summary. Change line 601 from:

   ```sh
   printf '\n%d problem(s), %d warning(s).\n' "$FAILS" "$WARNS"
   ```

   to also emit, only when `SETUP_FIXABLE` is set, a single hint line. Keep it
   tier-1 clean: **no em-dash.** For example:

   ```sh
   printf '\n%d problem(s), %d warning(s).\n' "$FAILS" "$WARNS"
   if [ "$SETUP_FIXABLE" -gt 0 ]; then
     printf 'Some of these are fixed by: maxx setup\n'
   fi
   ```

### Important nuances — do not get these wrong

- **Line 549 is more specific** (`pilotfish sync step`). If collapsing it into a
  generic bottom-line hint loses information a reader needs, keep 549's inline
  pointer as-is and simply do not route it through `warnfix`. Use judgment; the
  goal is less repetition, not zero pointers at any cost.
- **Do not touch line 650.** It is in `cmd_profiles` (the `maxx profiles`
  command), not `doctor`. Its `run: maxx setup` is a one-off and correct.
- **Do not add the pointer to failures that are not setup-fixable.** Auth
  failures (`run: $wrapper then /login`), `jq not found`, a tool not on PATH,
  and `agents/ is a real directory` (which points at RUNBOOK, not setup) must
  keep their own remedies and must **not** set `SETUP_FIXABLE`.
- **Honor `CLAUDE.md` tier 1 throughout:** no em-dash in any string or comment
  you add or edit. The new summary line included.
- **Wording rule still applies:** you are removing a repeated pointer and adding
  one new line, which is a deliberate output redesign. Do not reword any message
  beyond removing the trailing pointer.

## Fallbacks (only if A proves bad in practice)

- **Option B:** keep the pointer on the *first* setup-fixable failure in each
  profile block, drop it from the rest. Less code, weaker fix.
- **Option C:** keep every pointer, but make each line say what is specifically
  wrong so the repetition carries information. Most work, biggest diff.

If you switch to B or C, say so in the PR body and why A did not work.

## How to test (you must actually run this and read it)

Build a deliberately broken fixture and drive the real binary at it. Always pass
`--no-auth` or doctor launches a real 90-second Claude session per profile.

```sh
S=$(mktemp -d)
mkdir -p "$S/canon/agents" "$S/alt"
printf '{ "model":"opus","availableModels":["opus","sonnet","haiku"] }\n' > "$S/canon/settings.json"
cat > "$S/profiles.json" <<JSON
{ "canonical":"canon",
  "profiles":[
    { "name":"canon","config_dir":"$S/canon","wrapper":"claude-canon","tool":"claude" },
    { "name":"alt","config_dir":"$S/nope","wrapper":"claude-alt","tool":"claude" }
  ] }
JSON
MAXX_CONFIG_DIR="$S" ./bin/maxx doctor --no-auth
```

The `alt` profile block should now show its failures with **no** per-line
`(run: maxx setup)`, and the remedy should appear **once** at the bottom. Read
the whole output and confirm it no longer repeats one shape down the list.

## Acceptance criteria (all must hold)

- [ ] `grep -c '—' bin/maxx` returns `0` (no em-dash reintroduced).
- [ ] `shellcheck bin/maxx` is clean; `bash -n bin/maxx` passes.
- [ ] In a fully-broken profile block, the `(run: maxx setup)` pointer no longer
      repeats; the remedy prints once at/after the summary.
- [ ] Non-setup-fixable failures (auth, jq, PATH, agents-is-a-real-dir) still
      carry their own correct remedies and did not gain the setup hint.
- [ ] `maxx doctor` on a fully-healthy setup prints **no** stray setup hint
      (flag stays 0 when nothing setup-fixable failed).
- [ ] `maxx profiles` output unchanged (line 650 untouched).
- [ ] Diff is `bin/maxx` + `CHANGELOG.md` only.
- [ ] `CHANGELOG.md` gets a dated entry describing the doctor-output change (a
      behavior change to output, not a punctuation pass — say so honestly).

## Ship it

- New branch off `main`, e.g. `chore/doctor-setup-hint-dedup`. **Never commit to
  `main`.**
- Open a **draft** PR. **Do not merge** — that is Ryan's call.
- PR body: what changed, why (the monotony problem above), the fixture output
  before/after, and confirmation of the acceptance list.
