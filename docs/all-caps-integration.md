# ALL CAPS and maxx-headroom: one seam, two repos

Written 2026-08-04. Every structural claim here was read out of both working trees, not recalled. Where nothing was checked, it says so.

## The finding that decides this

**Both tools already model an account the same way, and neither knows the other exists.**

`~/.config/maxx/profiles.json`:

```json
{ "canonical": "main",
  "profiles": [
    { "name": "main", "config_dir": ".claude-main", "wrapper": "claude-main", "tool": "claude" },
    { "name": "fyi",  "config_dir": ".claude-fyi",  "wrapper": "claude-fyi",  "tool": "claude" }
  ] }
```

`all-caps/claude-usage-check.py:52`:

```python
ACCOUNTS = [
    {"name": "ryanilano@gmail.com", "config_dir": "~/.claude"},
    {"name": "ryan@ilano.fyi",      "config_dir": "~/.claude-fyi"},
]
```

The joining key is `config_dir`, and it is not a coincidence. ALL CAPS' own comment says it: "Each account is a CLAUDE_CONFIG_DIR." maxx's profile is a config dir with a wrapper attached. **An account is the same object in both tools.**

Two things make this worth acting on rather than admiring:

1. **`grep -rn "profiles.json"` across ALL CAPS returns nothing.** There is no link today, in either direction.
2. **ALL CAPS hardcodes that roster twice**, once in `claude-usage-check.py:52` and again in `claude-usage.1m.py:47`. Adding a third account today means editing two files and remembering both. That is a duplication defect independent of any integration, and the integration removes it as a side effect.

## Recommendation: two repos, one seam, and profiles.json is the roster of record

`~/.config/maxx/profiles.json` becomes the single place an account roster is declared. ALL CAPS reads it when present and falls back to its hardcoded list when it is not. maxx gains the `maxx headroom` subcommand its own `ROADMAP.md` already names as the missing piece, which shells out to the ALL CAPS checker when that is installed and says so plainly when it is not.

Neither tool acquires a hard dependency on the other. Both keep working alone.

### Why the alternatives lose

**Merge into one repo.** Forces one publishing decision onto two bodies of work with opposite exposure. maxx is public and MIT today. ALL CAPS' own README says "Keep this repo private," and it is right to (see the exposure section). Merging means either publishing material that should not be public or un-publishing a tool that already is. It is also bash plus Python in one tree with one README trying to explain both.

**Make one a dependency of the other.** maxx's install story is "clone, symlink, and you need `jq`." That single hard dependency is a feature and the README leads with it. Requiring Python and a second repo to run `maxx doctor` gives that away for a subcommand most users will never call. In the other direction, requiring maxx to run ALL CAPS breaks its standalone value for anyone with one account.

**Do nothing.** Defensible, and it has a cost that grows: the rosters drift, and the roster is now declared in three places rather than two, since the `headroom` skill also reads `profiles.json`.

## The data contract, concretely

**Direction 1: ALL CAPS reads maxx's roster.**

- **Source of truth:** `$MAXX_CONFIG_DIR/profiles.json`, defaulting to `~/.config/maxx/profiles.json`. Respect `MAXX_CONFIG_DIR` because `bin/maxx` does; that is what makes maxx's own test fixture possible.
- **Read exactly two keys per profile:** `name` and `config_dir`. Ignore `wrapper`, `tool`, `canonical`, and `watch_links`. Those are maxx's business.
- **`config_dir` may be `$HOME`-relative or absolute.** maxx's shipped defaults are relative (`.claude-main`), while ALL CAPS' are absolute-ish (`~/.claude`). Normalize with expanduser plus an isabs check, or accounts silently resolve to the wrong path.
- **Filter to `tool == "claude"` or a missing `tool`.** maxx's roadmap explicitly anticipates non-Claude profiles, and the Anthropic usage endpoint cannot speak for a Kimi subscription. Reading one would produce a confident wrong number.
- **Fall back, do not fail.** No file, unreadable file, malformed JSON, or zero usable profiles all mean "use the built-in `ACCOUNTS` list." A usage dashboard that dies because a config manager is not installed has the dependency arrow backwards.
- **Where it goes:** one shared resolver, imported by both Python entry points. That is what kills the double-hardcoding.

**The one real mismatch, and it needs a deliberate answer.** maxx's `name` is an identifier: short, stable, used to select a profile on the command line (`maxx pin <repo> fyi`). ALL CAPS' `name` is a display label and today it holds an email address. They are not the same field.

Recommended: **display maxx's `name` as-is and do not try to recover the email.** `MAIN 22% 3d23h` reads at least as well as an email in a menu bar chip, it is shorter, and it matches what the user types elsewhere. Deriving an email would mean reading `oauth-account.json` per account, which is more code and more surface for a cosmetic gain.

**Direction 2: `maxx headroom`.**

- A subcommand that locates the ALL CAPS checker and executes it, passing through its exit code and output.
- **Locate it, do not assume it.** In order: `$MAXX_HEADROOM_CMD` if set, then a `headroom_cmd` key in `profiles.json`, then `claude-usage-check.py` on `PATH`.
- **When it is absent, say so and exit nonzero**, in the same register as the existing `jq is required` message. Do not install anything, do not offer to.
- **It must not run inside `maxx doctor`.** doctor is a correctness check and this is a network call against a rate-limited endpoint that backs off for hours on a 429, per ALL CAPS' own notes.

## What would have to become public, and what must not

This is the part to get right before any of it ships, because it is the only irreversible piece.

**Safe to publish.** `claude-usage-check.py`, `claude-usage.1m.py`, `claude-reset-dashboard.html`, `INSTALL.md`. They contain no credential. The dashboard was verified on 2026-08-04 to make zero external requests. The credential model reads what the Claude CLI already stores; it introduces no new secret.

**Must not be published as written.**

- **`docs/CONTEXT.md` names both real account emails** at line 27, alongside each one's observed weekly reset anchor, and states the billing renewal date at line 31. That is an account inventory with a schedule attached.
- **`.planning/01-WHY.md`, `02-NAMING.md`, `03-PRODUCT.md`** are hand-written personal framing. `01-WHY.md` marks itself "strip before shipping." Treat that as binding, because its author wrote it.
- **The hardcoded `ACCOUNTS` lists** in both Python files carry the same two emails. They have to become an example roster before the code goes anywhere.

**ALL CAPS has no LICENSE file at all.** Verified 2026-08-04. That is all rights reserved by default, which is the most restrictive state, arrived at by omission rather than decision. Nothing can be contributed, forked, or packaged until that is chosen deliberately. It is free to decide now and expensive to change once anyone depends on it.

**Recommended split if publishing happens:** the three tools plus `INSTALL.md` in a small public repo with an example roster, and `docs/CONTEXT.md` plus `.planning/` staying private. ALL CAPS' own README already reaches the same conclusion for a different reason, noting that GitHub Pages on a private repo needs Pro and publishes publicly anyway.

## What this document did not check

- **Neither tool was run.** This is a read of both source trees and their own documentation. No integration was implemented and no behavior was observed.
- **Nothing was checked against the live usage API.** All API claims here are quoted from `docs/CONTEXT.md`, which dates them July 2026 and says to verify.
- **The `headroom` skill in `ilano-skills` was not read.** It is described in maxx's `ROADMAP.md` as already reading `profiles.json` for its roster, which would make it a third consumer of this contract and possibly a better home for the resolver than either repo. **[NEEDS SOURCE]** on anything about how it actually works.
