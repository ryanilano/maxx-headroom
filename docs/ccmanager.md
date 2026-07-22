# Using ccmanager with maxx

ccmanager is the optional piece, and it is useful when you run several sessions at once. It is a terminal app that manages one Claude Code session per git worktree and shows which session is busy, which is waiting on you, and which is idle. Install it with `npm install -g ccmanager` (needs node).

## What maxx sets up

`maxx setup` writes one ccmanager preset per profile into `~/.config/ccmanager/config.json`. Each preset launches that profile's wrapper, and ccmanager is set to ask which preset to use every time you start a session. So every session begins by answering which account pays for it. Presets merge into the existing config, and presets you made yourself are left alone. `maxx setup` writes the presets whether or not ccmanager is installed.

## Day to day

Run `ccmanager` in a repo. Pick a worktree, pick a preset, and you are in a normal Claude Code session on that account. Press Ctrl+E to drop back to the list while the session keeps running. Start another session in a different worktree, on the other account if that one has more room, and come back to whichever session says it is waiting.

## Keep each project on one account

Because ccmanager asks per session, it is easy to end up working on one repo from both accounts, and you shouldn't. Session history and memory live inside the account's config dir, so a project touched from two accounts splits its record in two, and neither half can see the other. The preset prompt is for picking that project's account, not a different one each time. For repos you work in VS Code terminals, `maxx pin <repo> <profile>` makes the choice permanent there.
