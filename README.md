# maxx-headroom

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

[About](#origin-story) · [Install](#install) · [Usage](#usage) · [Further Reading](#further-reading)

Do you have more than one Claude account? Don't let your weekly or 5-hour usage limits go to waste. Switch Claude Code between multiple paid accounts: each account gets its own profile, and maxx-headroom makes the setup, verification, and repo pinning painless.

Switching accounts becomes a solved problem instead of a daily ritual.

Today it's two Claude Max accounts. Next week it might be one Max account and one Pro account; next month maybe you throw in Kimi or DeepSeek. Profiles live in config, so adding or removing one is just an edit.

## Origin Story

I had one $20 Claude account, and I was used to maxing it out. Then Fable 5 landed with an expiration date — included access was [supposed to end July 7, 2026](https://www.anthropic.com/news/redeploying-fable-5), got [pushed to July 12](https://x.com/claudeai/status/2074548242386178258), then [pushed again to July 19](https://cybersecuritynews.com/anthropic-extends-claude-fable-5-access/) before the hard cutoff softened into usage credits with no end date. My sister offered to cover the $100 Max upgrade for a month. I accepted — and then opened a second account and paid another $100 myself, to squeeze everything out of Fable while it's here: projects to finish, a first hackathon.

## Why the Name

Max Headroom was a fictional AI from 1985. Now it names a tool that keeps a real one fed. Headroom is literally the resource being managed here — which account has room left — and the double-x is the meme spelling (tokenmaxxing), which also keeps a polite distance from the trademark. I am aware that "tokenmaxxing" is a ridiculous phrase. Unfortunately, it is also accurate: I'm trying not to waste paid quota.

It has a companion: pilotfish decides which model does the work; maxx-headroom decides which account pays for it.

## Install

Works on macOS and Linux. The stock macOS bash (3.2) is enough, so there is nothing to install beyond the list below.

Required:

- `jq` (`brew install jq` or `apt install jq`). Every command except `help` stops without it.
- Claude Code, with `claude` on your PATH. The wrappers run it, and the auth checks start a trial session with it. `setup` and `pin` work without it, but `doctor` reports it as a failure.

Optional, for `maxx pin`: VS Code — or a fork like VSCodium, which reads the same `.vscode` settings.

```sh
git clone https://github.com/ryanilano/maxx-headroom.git
cd maxx-headroom
mkdir -p ~/bin
ln -s "$PWD/bin/maxx" ~/bin/maxx   # ~/bin should be on your PATH
```

## Usage

The two profiles throughout are `main` and a secondary account we'll refer to as `alt`. The daily surface is three commands:

```sh
maxx setup                # interactive machine setup — asks before every step
maxx doctor               # verify everything; exits nonzero if anything's broken
maxx pin <repo> <profile> # pin a repo's VS Code terminals to one account
```

`setup` creates the per-profile config dirs, writes a `claude-<name>` wrapper per profile into `~/bin`, symlinks pilotfish agents from the canonical profile into the others, merges presets into ccmanager's config, and lazily checks auth by launching a real trial session per profile — it never pokes at your Keychain. Everything merges into existing files; nothing gets clobbered, and running it twice changes nothing.

`doctor` re-checks all of it — auth per profile, symlink integrity (Claude Code updates like to quietly replace the agents symlink with a fresh directory), model allowlists, ccmanager presets, wrappers on PATH — and tells you exactly what's missing.

`pin` writes `CLAUDE_CONFIG_DIR` into a repo's `.vscode/settings.json` terminal environment, so every terminal you open in that repo bills the account you chose. Existing settings are preserved. You don't have to pin anything. Plain `claude` bills whatever account the default config dir is logged into, and typing a wrapper like `claude-main` picks an account for one session. Pin a repo when you keep catching yourself starting sessions there on the wrong account.

### ccmanager

Do you work with git worktrees? Consider running ccmanager (`npm install -g ccmanager`), a terminal app that manages one Claude Code session per worktree. `maxx setup` gives it one preset per profile, so every session starts by picking which account pays — just keep each project on one account. The full workflow, and why that rule exists, is in [docs/ccmanager.md](docs/ccmanager.md).

> [!WARNING]
> **The one thing that can break, and it's already covered.** Profiles share pilotfish through one symlink per profile that points at the canonical `agents/` directory, and a tool that deletes and recreates that directory — Claude Code updates like to — destroys the symlink and silently disconnects the profiles. `maxx doctor` detects it, and `maxx setup` or [RUNBOOK §3](RUNBOOK.md) repairs it. The only new habit maxx asks of you is to run `maxx doctor` after upgrading anything that touches `agents/`.

Everything else in a profile is independent by design: each config dir is a complete, separate Claude Code installation, and maxx only bridges `agents/`. Profiles live in `~/.config/maxx/profiles.json`. The judgment-heavy procedures — installing pilotfish on a fresh machine, merging its CLAUDE.md block, rescuing a destroyed symlink (§3), sharing more across profiles (§5), the upgrade ritual (§6) — are written up in [RUNBOOK.md](RUNBOOK.md) for a Claude Code session to execute.

Side note: maxx-headroom deliberately stays out of model routing. pilotfish, GSD, Superpowers — every agent pack manages its model choices through its own configuration, on its own turf. maxx-headroom just symlinks the canonical `agents/` directory across profiles, so whatever routing lives there travels to every account on its own.

## Further Reading

- [CHANGELOG.md](CHANGELOG.md) — what changed and when. Dates, not versions.
- [ROADMAP.md](ROADMAP.md) — upcoming features and where the tool might go next.
- [CONSIDERATIONS.md](notes/CONSIDERATIONS.md) — tools and ideas evaluated but passed on.

## License

[MIT](LICENSE).
