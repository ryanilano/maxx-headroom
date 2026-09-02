# RELEASE-PLAN

What stands between "a public repo people can clone" and "a stranger can install this, verify it, and trust it."

Written 2026-08-04. Every claim below was checked against the working tree, not recalled. Where something was not checked, it says so.

## What is already true

| | |
|---|---|
| Public | `github.com/ryanilano/maxx-headroom`, MIT |
| Shape | One file, `bin/maxx`, POSIX-ish bash, `set -euo pipefail`, declared Bash 3.2 compatible |
| Hard dependency | `jq`, and nothing else. `perl` is used only behind a `command -v` guard |
| Subcommands | `setup`, `doctor`, `pin`, `profiles`, `help` |
| Docs | README, RUNBOOK, CHANGELOG, ROADMAP, CLAUDE.md, `docs/ccmanager.md`, `notes/CONSIDERATIONS.md` |

**Verified absent:** no version string or `version` subcommand anywhere in `bin/maxx`. No `tests/`, no `.github/`, no CI. No `.gitignore` at the repo root. No CONTRIBUTING. No release tags in use.

**Verified, and it matters for item 5:** `grep` for `security`, `defaults`, `osascript`, `/Applications`, `open -a`, `pbcopy`, and `sw_vers` in `bin/maxx` returns nothing. The two apparent hits are the English words "security" and "defaults" inside error strings. So there is no obvious macOS-only system call in the script, which makes the README's Linux claim plausible. **Plausible is not tested.**

---

## The items, ordered by what blocks what

### 1. Sync local state with GitHub. Mechanical. Blocks everything.

The working tree is on branch `doctor-config-rule-drift` at `d476596`, one commit ahead of `origin/main`, unpushed. Local `main` is level with `origin/main`. The branch `roadmap-headroom-followups` still exists locally and on origin despite being merged through PR #2.

That commit is the `== config rules ==` doctor feature and it is real work sitting outside the published history. **It was checked for personal content and is clean:** no `ryan`, no `ilano`, no `gmail`, no `.claude-main` or `.claude-fyi`, no `/Users/` path, no machine or tailnet name. Its own commit message says config dirs come from `profiles.json` with no home names hardcoded, and the diff bears that out.

- **Touches:** git refs only.
- **Done when:** `git rev-list --left-right --count origin/main...HEAD` returns `0 0` on whatever branch you intend to publish from, and the merged branch is gone from both sides.
- **Reversible:** yes, until pushed. **Pushing is not.** This is Ryan's call and his standing rule in this repo is that commits open on a branch and merging is his decision.

### 2. Decide the version question. Judgment. Blocks 3 and 6.

There is no version anywhere. `CHANGELOG.md` uses dates on purpose and says so: "Dates instead of version numbers, because the tool has no release process."

That is a coherent position for a personal tool and it stops being coherent the moment someone else installs it, because a bug report needs an answer to "which one do you have." The options are not equal:

- **Keep dates, add a git-describe-based `maxx version`.** Cheapest honest fix. Output is whatever the checkout actually is, so it cannot lie. Fails in a tarball download with no `.git`.
- **Adopt tags plus a hardcoded `MAXX_VERSION` string.** Needs a release ritual and a place that ritual is written down, or the string rots.
- **Stay as-is and say so in the README.** Legitimate. Requires writing the sentence.

- **Touches:** `bin/maxx`, `CHANGELOG.md`, `README.md`.
- **Done when:** a user can answer "which version do you have" in one command, or the README states plainly that there are no versions and why.

### 3. Give `doctor` a safe first run, and document it. Judgment, small. Blocks 4.

`--no-auth` exists on both `setup` and `doctor`. It is not in the README. Without it, `doctor` launches a real Claude session per profile to check auth, which on a two-profile machine means two real sessions against the user's paid quota before they have decided to trust the tool.

**This is the single worst first-run experience in the repo** and the fix is documentation, not code: the README's usage section should show `maxx doctor --no-auth` as the first command a new user runs, with one sentence saying what the flag skips and why you would want it on run one.

- **Touches:** `README.md`. Optionally `usage()` in `bin/maxx`, which already lists the flag.
- **Done when:** the README's first suggested command does not spend the reader's quota.

### 4. A smoke test a stranger can run. Mechanical, and the highest-value item here. Blocks 7.

There is no test of any kind. But the pattern already exists, fully written, inside `FABLE-BRIEF-doctor-setup-hint.md`: an `mktemp -d` fixture that builds a deliberately broken two-profile setup, points `MAXX_CONFIG_DIR` at it, and drives the real binary with `--no-auth`. It touches nothing real.

That is the whole test. It needs to become a file rather than a paragraph in an untracked brief.

- **Touches:** new `tests/smoke.sh`, and a README line.
- **Done when:** `./tests/smoke.sh` runs on a machine with no Claude accounts configured, exits nonzero against the broken fixture, and exits zero against a healthy one. **Both directions, or it proves nothing.** A test that only ever passes has not been shown to be able to fail.
- **Why it is worth more than it costs:** it is also the answer to item 5.

### 5. Settle the Linux claim. Judgment, then mechanical. Independent.

The README says "Works on macOS and Linux." Nothing in the tree shows that was ever run on Linux. The static evidence is encouraging (no macOS-only system calls found) and static evidence is not a test.

Two honest resolutions, and only two:

- **Run item 4's smoke test on a Linux box once** and keep the claim. Any container with `bash` and `jq` will do.
- **Soften the claim** to what is actually known: written to be portable, tested on macOS.

- **Touches:** `README.md`, or nothing if it passes.
- **Done when:** the claim in the README matches something that was executed.
- **Note:** `maxx pin` writes `.vscode/settings.json` and nothing else, so it is portable by construction. Confirmed at `bin/maxx:704`.

### 6. Install path. Judgment. Independent, but do not start it before 2 and 4.

Today: clone, `mkdir -p ~/bin`, `ln -s`. That is fine for a reader who already lives in a terminal, which is the actual audience.

A Homebrew tap is the obvious next step and it is a standing maintenance cost, not a one-time task: a formula pins a version (see item 2), needs updating on every release, and becomes a second place bugs get reported. **Recommend deferring it until someone who is not Ryan asks for it.** Writing that decision into `ROADMAP.md` is the deliverable, so it stops being an unexamined gap.

- **Touches:** `ROADMAP.md`, or a new tap repo if the answer flips.
- **Done when:** the README's install section is either unchanged with a recorded reason, or replaced.

### 7. Contribution posture. Judgment. Last.

No CONTRIBUTING, no issue templates, no `.github/`. For an MIT tool with a public README that invites use, silence here means every question arrives as an unstructured issue, and every drive-by PR arrives without knowing about `CLAUDE.md`'s tier-1 punctuation rule, which is a real and unusual house constraint that a contributor will violate on their first PR.

Minimum viable version, and it is genuinely small: a short CONTRIBUTING that says what the project is not trying to become, points at `CLAUDE.md` for the punctuation rule, and points at item 4's smoke test as the bar. Or an explicit "not accepting contributions" line, which is also a complete answer.

- **Touches:** new `CONTRIBUTING.md`, optionally `.github/ISSUE_TEMPLATE/`.
- **Done when:** a stranger can tell, without asking, whether a PR is welcome.

### 8. Repo hygiene. Mechanical. Independent, do any time.

- **No `.gitignore` exists.** Exclusions currently live in `.git/info/exclude`, which no clone ever sees. That defeats its own purpose in a public repo. Minimum content: `.claude/`, `.DS_Store`.
- **Four untracked files at the repo root are session artifacts, not documentation:** `PROPOSED-FIXES.md`, `SESSION-CHANGES.md`, `SESSION-HISTORY.md`, `FABLE-BRIEF-doctor-setup-hint.md`. `PROPOSED-FIXES.md` already recommended moving the session ones out on 2026-07-27 and it never happened. **Do not delete them.** They are the record, and `FABLE-BRIEF` contains the test fixture item 4 needs.
- **Two of them are actively wrong.** `SESSION-CHANGES.md` and `SESSION-HISTORY.md` both describe PR #1 as open and awaiting review. It merged, and so did PR #2. Anyone reading them as current status is misled.
- **Done when:** `git status` at the root shows nothing untracked that is not deliberately ignored.

---

## Two things this plan does not do

**It does not resolve the `(run: maxx setup)` de-duplication.** That work is fully specified in `FABLE-BRIEF-doctor-setup-hint.md`, option A chosen, 11 line sites listed, acceptance criteria written, and it was never started. Verified 2026-08-04: `grep -c "run: maxx setup" bin/maxx` returns 11 and `grep -c "SETUP_FIXABLE" bin/maxx` returns 0. It is product polish and it does not block a release, so it is deliberately not in the ordering above.

**It does not defuse the RUNBOOK landmine, but flags it as a one-line fix worth doing inside any of the items above.** `bin/maxx:557` emits `agents/ is a real directory, not a symlink:` and `RUNBOOK.md:35` quotes that prefix verbatim. Nothing marks the dependency, so a future wording change silently makes the RUNBOOK stale. One comment above line 557 defuses it permanently.

## Ordering summary

| Order | Item | Kind | Blocks |
|---|---|---|---|
| 1 | Sync git state | Mechanical | Everything |
| 2 | Version question | Judgment | 3, 6 |
| 3 | Safe first run in README | Judgment, small | 4 |
| 4 | Smoke test | Mechanical, highest value | 5, 7 |
| 5 | Linux claim | Judgment then mechanical | Independent |
| 6 | Install path | Judgment | Independent |
| 7 | Contribution posture | Judgment | Independent |
| 8 | Repo hygiene | Mechanical | Independent |

**Reversible:** everything except pushing in item 1, and anything that lands in the public history. **Not reversible:** a published claim. Items 5 and 6 are the two that put a promise in front of strangers.
