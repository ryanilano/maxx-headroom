# maxx-headroom — proposed fixes

Written 2026-07-27. Ryan hasn't touched this repo since 24 July. Verified
against actual git state (fetched from `origin`), not assumptions from the
task brief — one finding below corrects the brief's framing.

## 1. What it is, and what state it's in

maxx-headroom is a working, shipped CLI tool (`bin/maxx`, ~690 lines of
bash) that lets Ryan run multiple Claude Code accounts side by side without
manually juggling config directories — `maxx setup` provisions a profile,
`maxx doctor` verifies everything's still wired correctly, `maxx pin` locks
a repo's terminal environment to one account. It has a companion tool
(`pilotfish`, for model routing) and a real README, RUNBOOK, CHANGELOG, and
ROADMAP. This is not a prototype — it's a finished v1 that Ryan uses daily,
with real install instructions and a public repo.

**In plain language: it's done and working, and the thing that looked
"stalled" is actually just two small, already-completed pieces of follow-up
work that never got pulled down locally.** There's nothing broken or
half-built at the core.

## 2. The branch question — corrected finding

The brief describes `roadmap-headroom-followups` as "13 commits, 0 ahead of
upstream," posing "merge, continue, or abandon" as the live question. After
fetching from `origin`, that framing is out of date:

**Both feature branches in this repo have already been merged into `main`
on GitHub.** `git log` on `origin/main` shows:

```
4f0266d Merge pull request #2 from ryanilano/roadmap-headroom-followups
b1d597f roadmap: note headroom skill ships the reporting; add multi-provider entry
4dc86fb docs: add banner, rewrite README, set repo metadata
1d41cd4 Merge pull request #1 from ryanilano/chore/cli-string-punctuation
4c99f97 chore: drop em-dashes from maxx CLI output; add punctuation rule
```

So: PR #1 (the punctuation work `SESSION-CHANGES.md`/`SESSION-HISTORY.md`
describe as "open, not merged, awaiting review") and PR #2
(`roadmap-headroom-followups`, the branch currently checked out) are **both
merged**. There is no merge decision left to make — GitHub already made it.

**What's actually true locally right now:**
- The checked-out branch (`roadmap-headroom-followups`) matches its remote
  tracking branch exactly — no local work is unpushed or at risk.
- Local `main` is two commits **behind** `origin/main` (it never pulled the
  two merges).
- The working tree is otherwise clean except the three untracked items
  (§3).

**Recommendation: fast-forward local `main`, switch to it, and delete the
now-merged local feature branch.** This isn't a judgment call, it's
bookkeeping:

```sh
git checkout main
git pull origin main          # fast-forward, no conflicts possible (already an ancestor)
git branch -d roadmap-headroom-followups   # safe delete: fully merged
git push origin --delete roadmap-headroom-followups   # optional, tidy up the remote too
```

## 3. The untracked files

**`.claude/` — already effectively handled, no action needed.** It holds
one file, `settings.json`, containing only a permissions allowlist (three
`Bash(...)` entries — `npm root -g`, `claude plugin validate *`,
`docker info`). No secrets, no paths with usernames, nothing sensitive.
It's untracked (not gitignored — there is no `.gitignore` in this repo at
all), which matches the session notes' observation ("`.claude/` is
untracked — was untracked on arrival, never addressed"). Recommend adding a
real `.gitignore` with `.claude/` in it (see below) rather than leaving it
merely untracked, since "untracked" only protects against `git add -A`
mistakes, not against someone doing `git add .claude/settings.json`
directly on a bad day.

**`SESSION-CHANGES.md` and `SESSION-HISTORY.md` — move to `agent-notes`, do
not commit here, do not delete.** Checked both files line by line for
exposure: no keys, tokens, secrets, or internal hostnames. They do contain
Ryan's name and a public GitHub PR URL, which is fine content for a private
notes repo but adds no value inside `maxx-headroom` itself — they're a
blow-by-blow account of *how* one PR's punctuation pass was done and
verified, not user-facing documentation. That's exactly the kind of
artifact `~/local-dev/agent-notes` exists for. Concretely:

```sh
mkdir -p ~/local-dev/agent-notes/maxx-headroom
mv ~/local-dev/maxx-headroom/SESSION-CHANGES.md ~/local-dev/agent-notes/maxx-headroom/2026-07-23-cli-punctuation-CHANGES.md
mv ~/local-dev/maxx-headroom/SESSION-HISTORY.md ~/local-dev/agent-notes/maxx-headroom/2026-07-23-cli-punctuation-HISTORY.md
```

(Rename with a date prefix since `agent-notes` likely accumulates many of
these across projects — worth confirming Ryan's existing naming convention
there before running this, since I didn't inspect that repo.)

**Add a `.gitignore` while touching this,** since one doesn't exist and
these two files aren't the only thing that will land untracked in this
repo's lifetime:

```
.claude/
*.local.md
```

This also matches the pattern already established in `.git/info/exclude`
(`notes/LOCAL-*.md` is excluded there) — worth eventually consolidating
that into a real `.gitignore` too, since `.git/info/exclude` isn't shared
with anyone who clones the repo fresh, which defeats its own purpose for a
public repo other people might contribute to.

## 4. Next three actions, in order

1. **Sync `main` and clean up the merged branch** (§2 commands). Reason:
   this is the only actual inconsistency in the repo right now — local
   state disagreeing with GitHub — and every other recommendation here
   assumes you're working from current `main`. Five minutes, zero risk
   (fast-forward only, branch deletion gated on "fully merged").

2. **Move the session files, add `.gitignore`, commit that small hygiene
   change on `main`.** Reason: this is the only other loose end sitting in
   the working tree, and it's small enough to do in the same sitting as
   step 1. Suggested commit message (no AI attribution, per standing
   rule): `chore: gitignore .claude/, move session notes to agent-notes`.

3. **Decide the deferred `(run: maxx setup)` follow-up (§5) — or explicitly
   defer it again with a note in ROADMAP.md.** This is the one real open
   decision left from the last session (three options, laid out in
   `SESSION-CHANGES.md` before it moves to `agent-notes` — option A is
   recommended there and still looks right: collapse the repeated pointer
   to one line beside the summary count). Reason: it's the only piece of
   actual unfinished product work left, everything else is bookkeeping;
   worth either doing it in one sitting (it's scoped as "higher risk than
   punctuation" but still a single-function change) or writing the decision
   down in ROADMAP.md so it stops silently blocking on nothing.

## 5. Anything broken, with file:line and a fix

**Nothing found broken.** `shellcheck bin/maxx` is clean (matches the
prior session's own finding). I did not re-run `maxx doctor` end to end
against a live multi-profile setup — that would need real Claude account
state on this machine, which is outside what I should be poking at for a
proposal document — so treat "no bugs found" as bounded by static review
plus the prior session's own verification, not a fresh dynamic test pass.

The one real gap, already known and explicitly logged: `RUNBOOK.md:35`
quotes a doctor message verbatim (`agents/ is a real directory, not a
symlink`) and the punctuation pass appended a colon *after* that fragment
by luck, not by checking. **Recommended fix, cheap and worth doing now
rather than waiting for it to break:** add a one-line comment directly
above the matching message at `bin/maxx:529`
(`err "agents/ is a real directory, not a symlink: ..."`) — something like
`# RUNBOOK.md:35 quotes the "agents/ is a real directory, not a symlink"
prefix verbatim; keep it intact` — so the next edit to that message doesn't
require re-discovering the dependency. This is the only thing in the whole review that's a real
landmine (a future wording change could silently make the RUNBOOK stale),
and it costs one comment line to defuse permanently.
