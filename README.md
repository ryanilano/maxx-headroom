# maxx-headroom

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

I had one $20 Claude account, and I was used to maxing it out. Then Fable 5 landed with an expiration date — included access was [supposed to end July 7, 2026](https://www.anthropic.com/news/redeploying-fable-5), got [pushed to July 12](https://x.com/claudeai/status/2074548242386178258), then [pushed again to July 19](https://cybersecuritynews.com/anthropic-extends-claude-fable-5-access/) before the hard cutoff softened into usage credits with no end date. My sister offered to cover the $100 Max upgrade for a month. I accepted — and then opened a second account and paid another $100 myself, to squeeze everything out of Fable while it's here: projects to finish, a first hackathon.

So now there are two tanks of quota, and the only unacceptable outcome is either one sitting full while the model is still around. maxx-headroom is the tool that makes sure that never happens: it sets up, verifies, and repo-pins multi-account Claude Code usage, so switching accounts is a solved problem instead of a nightly ritual.

Profiles are data, not structure. Today that means two Claude accounts (`main` and `fyi`); tomorrow it might be one account again, or a GPT or Kimi subscription sitting alongside. Add or retire a profile by editing one JSON entry — the tool doesn't care.

## Why the name

Max Headroom was a fictional AI from 1985. Now it names a tool that keeps a real one fed. Headroom is literally the resource being managed here — which account has room left — and the double-x is the meme spelling (tokenmaxxing), which also keeps a polite distance from the trademark.

It has a companion: pilotfish decides which model does the work; maxx-headroom decides which account pays for it.

## Install

Requires `jq`, and Claude Code itself. macOS and Linux.

```sh
git clone https://github.com/ryanilano/maxx-headroom.git
cd maxx-headroom
mkdir -p ~/bin
ln -s "$PWD/bin/maxx" ~/bin/maxx   # ~/bin should be on your PATH
```

## Use

The daily surface is three commands:

```sh
maxx setup                # interactive machine setup — asks before every step
maxx doctor               # verify everything; exits nonzero if anything's broken
maxx pin <repo> <profile> # pin a repo's VS Code terminals to one account
```

`setup` creates the per-profile config dirs, writes a `claude-<name>` wrapper per profile into `~/bin`, symlinks pilotfish agents from the canonical profile into the others, merges presets into ccmanager's config, and lazily checks auth by launching a real trial session per profile — it never pokes at your Keychain. Everything merges into existing files; nothing gets clobbered, and running it twice changes nothing.

`doctor` re-checks all of it — auth per profile, symlink integrity (Claude Code updates like to quietly replace the agents symlink with a fresh directory), model allowlists, ccmanager presets, wrappers on PATH — and tells you exactly what's missing.

`pin` writes `CLAUDE_CONFIG_DIR` into a repo's `.vscode/settings.json` terminal environment, so every terminal you open in that repo bills the account you chose. Existing settings are preserved.

Profiles live in `~/.config/maxx/profiles.json`. The judgment-heavy stuff — installing pilotfish on a fresh machine, merging its CLAUDE.md block into a customized one, rescuing a symlink an update destroyed — is written up in [RUNBOOK.md](RUNBOOK.md) for a Claude Code session to execute.

## License

[MIT](LICENSE).
