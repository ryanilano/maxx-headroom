<p align="center">
  <img src="./assets/tokenmaxxing.png" alt="maxx-headroom — tokenmaxxing: coordinate multiple Claude Code accounts across rolling usage limits" width="100%">
</p>

# maxx-headroom

> Coordinate multiple Claude Code accounts across rolling usage limits.

Switch Claude Code between multiple accounts without wasting weekly or 5-hour usage limits. Each account gets its own profile; maxx-headroom makes the setup, verification, and repo pinning painless. Switching accounts becomes a solved problem instead of a daily ritual.

[What It Does](#what-it-does) · [Origin Story](#origin-story) · [Install](#install) · [Usage](#usage) · [Further Reading](#further-reading)

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

## What It Does

If you run more than one Claude account, maxx-headroom helps you use them deliberately instead of juggling them by hand. It gives each account its own profile, makes machine setup repeatable and verifiable, and lets you pin a repo's terminal environment to the account you want paying for the session.

Today it's two Claude Max accounts. Next week it might be one Max and one Pro; next month maybe you throw in Kimi or DeepSeek. Profiles live in config, so changing the roster is just an edit.

## Origin Story

I had one $20 Claude account, and I was used to maxing it out using Opus. Then Fable 5 landed with an expiration date. Included access was [supposed to end July 7, 2026](https://www.anthropic.com/news/redeploying-fable-5), got [pushed to July 12](https://x.com/claudeai/status/2074548242386178258), then [pushed to July 19](https://cybersecuritynews.com/anthropic-extends-claude-fable-5-access/) before the hard cutoff softened into usage credits with no end date. My sister offered to cover the $100 Max upgrade for a month, the night before Fable came back from its export-control suspension. I accepted, and after a few days of seeing what Fable was capable of I opened a second account and paid another $100 myself. I need to squeeze everything out of Fable while it's here: projects to finish, a first hackathon.

## Why the Name

Max Headroom is a fictional AI from 1985. Now it names a tool that keeps a real one fed. Headroom is literally the resource being managed here: which account has Fable, 5-hour, or weekly usage left to burn? This is all about tokenmaxxing (burning paid tokens with intent) after ~~three~~ [four](https://x.com/claudeai/status/2078302417100394737?s=61) unexpected policy and billing changes. I am aware that "tokenmaxxing" is a ridiculous phrase. Unfortunately, it is also accurate: I'm trying not to waste paid quota.

maxx-headroom has a companion: [pilotfish](https://github.com/Nanako0129/pilotfish) decides which model does the work; maxx-headroom helps you decide which account pays for it.

## Install

Works on macOS and Linux. The stock macOS bash (3.2) is enough, so there is nothing to install beyond the list below.

Required:

- `jq` (`brew install jq` or `apt install jq`). Every command except `help` stops without it.
- Claude Code, with `claude` on your PATH. The wrappers run it, and the auth checks start a trial session with it. `setup` and `pin` work without it, but `doctor` reports it as a failure.

Optional, for `maxx pin`: VS Code, or a fork like VSCodium (both read the same `.vscode` settings).

```sh
git clone https://github.com/ryanilano/maxx-headroom.git
cd maxx-headroom
mkdir -p ~/bin
ln -s "$PWD/bin/maxx" ~/bin/maxx   # ~/bin should be on your PATH
```

## Usage

The two profiles throughout are `main` and a secondary account we'll refer to as `alt`. The daily surface is three commands:

```sh
maxx setup                # interactive machine setup. Asks before every step
maxx doctor               # verify everything; exits nonzero if anything's broken
maxx pin <repo> <profile> # pin a repo's VS Code terminals to one account
```

`setup` creates the per-profile config dirs, writes a `claude-<name>` wrapper per profile into `~/bin`, symlinks pilotfish agents from the canonical profile into the others, merges presets into ccmanager's config, and lazily checks auth by launching a real trial session per profile. It never pokes at your Keychain. Everything merges into existing files; nothing gets clobbered, and running it twice changes nothing.

`doctor` re-checks all of it: auth per profile, symlink integrity (Claude Code updates like to quietly replace the agents symlink with a fresh directory), model allowlists, ccmanager presets, wrappers on PATH. It tells you exactly what's missing.

`pin` writes `CLAUDE_CONFIG_DIR` into a repo's `.vscode/settings.json` terminal environment, so every terminal you open in that repo bills the account you chose. Existing settings are preserved. You don't have to pin anything. Plain `claude` bills whatever account the default config dir is logged into, and typing a wrapper like `claude-main` picks an account for one session. Pin a repo when you keep catching yourself starting sessions there on the wrong account.

### Managing Git Worktrees with ccmanager

Do you work with git worktrees? Consider running [ccmanager](https://github.com/kbwo/ccmanager) (`npm install -g ccmanager`), a terminal app that manages one Claude Code session per worktree. `maxx setup` gives it one preset per profile, so every session starts by picking which account pays. Just keep each project on one account. The full workflow, and why that rule exists, is in [docs/ccmanager.md](docs/ccmanager.md).

> [!WARNING]
> **The one thing that can break, and it's already covered.** Profiles share pilotfish through one symlink per profile that points at the canonical `agents/` directory, and a tool that deletes and recreates that directory (Claude Code updates like to) destroys the symlink and silently disconnects the profiles. `maxx doctor` detects it, and `maxx setup` or [RUNBOOK §3](RUNBOOK.md) repairs it. The only new habit maxx asks of you is to run `maxx doctor` after upgrading anything that touches `agents/`.

Everything else in a profile is independent by design: each config dir is a complete, separate Claude Code installation, and maxx only bridges `agents/`. Profiles live in `~/.config/maxx/profiles.json`. The judgment-heavy procedures — installing pilotfish on a fresh machine, merging its CLAUDE.md block, rescuing a destroyed symlink (§3), sharing more across profiles (§5), the upgrade ritual (§6) — are written up in [RUNBOOK.md](RUNBOOK.md) for a Claude Code session to execute.

Side note: maxx-headroom deliberately stays out of model routing. pilotfish, GSD, Superpowers all manage their model choices through their own configuration, on their own turf. maxx-headroom just symlinks the canonical `agents/` directory across profiles, so whatever routing lives there travels to every account on its own.

## Further Reading

- [CHANGELOG.md](CHANGELOG.md): what changed and when. Dates, not versions.
- [ROADMAP.md](ROADMAP.md): upcoming features and where the tool might go next.
- [CONSIDERATIONS.md](notes/CONSIDERATIONS.md): tools and ideas evaluated but passed on.

## License

[MIT](LICENSE).
