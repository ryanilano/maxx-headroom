# maxx-headroom

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

I had one $20 Claude account, and I was used to maxing it out. Then Fable 5 landed with an expiration date — included access was [supposed to end July 7, 2026](https://www.anthropic.com/news/redeploying-fable-5), got [pushed to July 12](https://x.com/claudeai/status/2074548242386178258), then [pushed again to July 19](https://cybersecuritynews.com/anthropic-extends-claude-fable-5-access/) before the hard cutoff softened into usage credits with no end date. My sister offered to cover the $100 Max upgrade for a month. I accepted — and then opened a second account and paid another $100 myself, to squeeze everything out of Fable while it's here: projects to finish, a first hackathon.

So now there are two tanks of quota, and the only unacceptable outcome is either one sitting full while the model is still around. maxx-headroom is the tool that makes sure that never happens: it sets up, verifies, and repo-pins multi-account Claude Code usage, so switching accounts is a solved problem instead of a nightly ritual.

Profiles are data, not structure. Today that means two Claude accounts — `main` and a secondary account we'll refer to as `alt`; tomorrow it might be one account again, or a GPT or Kimi subscription sitting alongside. Add or retire a profile by editing one JSON entry — the tool doesn't care.

## Why the name

Max Headroom was a fictional AI from 1985. Now it names a tool that keeps a real one fed. Headroom is literally the resource being managed here — which account has room left — and the double-x is the meme spelling (tokenmaxxing), which also keeps a polite distance from the trademark.

It has a companion: pilotfish decides which model does the work; maxx-headroom decides which account pays for it.

## Install

Works on macOS and Linux. The stock macOS bash (3.2) is enough, so there is nothing to install beyond the list below.

Required:

- `jq` (`brew install jq` or `apt install jq`). Every command except `help` stops without it.
- Claude Code, with `claude` on your PATH. The wrappers run it, and the auth checks start a trial session with it. `setup` and `pin` work without it, but `doctor` reports it as a failure.

Optional:

- ccmanager (`npm install -g ccmanager`, which needs node). `setup` writes its presets whether or not it is installed.
- VS Code, only for `maxx pin`, which writes the repo's `.vscode/settings.json`. A fork like VSCodium works the same, because forks read the same `.vscode` folder and terminal settings.

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

`pin` writes `CLAUDE_CONFIG_DIR` into a repo's `.vscode/settings.json` terminal environment, so every terminal you open in that repo bills the account you chose. Existing settings are preserved. You don't have to pin anything. Plain `claude` bills whatever account the default config dir is logged into, and typing a wrapper like `claude-main` picks an account for one session. Pin a repo when you keep catching yourself starting sessions there on the wrong account.

### ccmanager

ccmanager is the optional piece, and it is useful when you run several sessions at once. It is a terminal app that manages one Claude Code session per git worktree and shows which session is busy, which is waiting on you, and which is idle.

`maxx setup` writes one ccmanager preset per profile. Each preset launches that profile's wrapper, and ccmanager is set to ask which preset to use every time you start a session. So every session begins by answering which account pays for it.

Day to day, run `ccmanager` in a repo. Pick a worktree, pick a preset, and you are in a normal Claude Code session on that account. Press Ctrl+E to drop back to the list while the session keeps running. Start another session in a different worktree, on the other account if that one has more room, and come back to whichever session says it is waiting.

Because ccmanager asks per session, it is easy to end up working on one repo from both accounts, and you shouldn't. Session history and memory live inside the account's config dir, so a project touched from two accounts splits its record in two, and neither half can see the other. Keep each project on one account. The preset prompt is for picking that project's account, not a different one each time.

On day one, cd into any repo and run `ccmanager`. It will show the worktree list, and every session you start will ask main or alt. Remember the rule when it asks. Each project stays on one account. For repos you work in VS Code or VSCodium terminals, `maxx pin <repo> main` (or `alt`) makes the choice permanent there.

> [!WARNING]
> **The one thing that can break, and it's already covered.** Profiles share pilotfish through one symlink per profile that points at the canonical `agents/` directory. An installer or a Claude Code update that writes files into that directory is fine, and every profile sees the new files at once. A tool that instead deletes the directory and recreates it (`rm -rf agents/`, then a fresh directory) destroys the symlink, and the profiles silently disconnect. maxx is built for exactly that failure. `maxx doctor` detects it, and `maxx setup` or [RUNBOOK §3](RUNBOOK.md) repairs it. The only new habit maxx asks of you is to run `maxx doctor` after upgrading anything that touches `agents/`. It takes ten seconds and tells you whether a link got clobbered.

Everything else in a profile is independent by design. Each config dir is a complete, separate Claude Code installation — skills, plugins, slash commands, and hooks in one profile do not exist in the others, and maxx only bridges `agents/`. If you want profiles to share more than that, you can symlink the other directories by hand (`skills/`, `plugins/`, `commands/`) and copy the `enabledPlugins` key into the profile's `settings.json` — the plugins directory alone activates nothing. But be clear about what you are signing up for: **`maxx doctor` checks the `agents/` symlink and nothing else by default.** A hand-made link destroyed by the same failure mode described above goes undetected — unless you register it. Add an optional `watch_links` map to `~/.config/maxx/profiles.json` and doctor guards those links too, with the same real-directory-instead-of-symlink detection as the managed `agents/` check:

```json
"watch_links": {
  ".claude-main": ["skills", "plugins", "commands"]
}
```

Keys are config dirs ($HOME-relative or absolute, same as `config_dir`), values are the link names inside them. maxx never creates these links — registering only means doctor refuses to let them break silently. Links you don't register remain on you, especially after any Claude Code update and after the first login in a fresh profile, which scaffolds directory structure. Hooks are the exception that cannot be symlinked at all: they live inside each profile's `settings.json`, so they must be copied into every profile that wants them.

### Upgrading Claude Code

The ritual is doctor, upgrade, doctor, one grep:

1. `maxx doctor` — get a green baseline first, so anything the upgrade breaks is unambiguously the upgrade's doing.
2. Upgrade (brew or however you installed it).
3. `maxx doctor` — catches a clobbered `agents/` symlink. `maxx setup` auto-repairs it when the recreated directory is empty; [RUNBOOK §3](RUNBOOK.md) covers the merge when it has content.
4. If you made the optional hand-made links described above (`skills/`, `plugins/`, `commands/`), register them in `watch_links` (see above) and step 3's doctor run covers them too. Anything unregistered you check yourself: `ls -la <config dir> | grep '\->'` should still show every arrow. If an update replaced any link with a real directory, the same diff-then-relink procedure from [RUNBOOK §3](RUNBOOK.md) applies — compare against the link target, salvage anything real, remove, relink.

Upgrades don't rewrite `settings.json` or `CLAUDE.md`, so hooks and agent-pack config need no attention.

Profiles live in `~/.config/maxx/profiles.json`. The judgment-heavy stuff — installing pilotfish on a fresh machine, merging its CLAUDE.md block into a customized one, rescuing a symlink an update destroyed — is written up in [RUNBOOK.md](RUNBOOK.md) for a Claude Code session to execute.

Side note: maxx-headroom deliberately stays out of model routing. pilotfish, GSD, Superpowers — every agent pack manages its model choices through its own configuration, on its own turf. maxx-headroom just symlinks the canonical `agents/` directory across profiles, so whatever routing lives there travels to every account on its own.

## License

[MIT](LICENSE).
