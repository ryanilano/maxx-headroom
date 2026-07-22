# RUNBOOK

These are the procedures that need judgment, so they live outside `bin/maxx`. The script handles the mechanical, idempotent parts. Every procedure here requires reading the situation before acting. Each section is written so you can paste it into a Claude Code session and let it drive, but a human can follow it too.

Conventions:

- The canonical profile is the profile named by `canonical` in `~/.config/maxx/profiles.json`. The shipped default is `main`, with config dir `~/.claude-main`. It owns the real `agents/` directory and the model settings. Every other profile gets a symlink and replicated keys.
- Run `maxx doctor` before and after any procedure here. The run before tells you what is broken. The run after proves you fixed it.

## 1. Install pilotfish on a fresh machine

`maxx setup` assumes pilotfish already exists in the canonical profile. When `doctor` says `no agents/ in ~/.claude-main — pilotfish not installed?`, follow this procedure.

1. Run `maxx setup` first, or confirm it has run. The config dirs, wrappers, and profiles file must exist before pilotfish has anywhere to land.
2. Install pilotfish into the canonical profile only, following the install instructions in the pilotfish repo. Its agent definitions must end up in `<canonical config dir>/agents/`, e.g. `~/.claude-main/agents/`. Do not install it into the other profiles. They receive it through the symlink.
3. If the pilotfish install wants to write into `~/.claude` (the default config dir), redirect it. Either run the install with `CLAUDE_CONFIG_DIR` pointed at the canonical dir, or move the installed `agents/` content into the canonical dir afterward. Then check what else the installer touched, e.g. settings keys or CLAUDE.md, and bring those along.
4. Verify the canonical `settings.json` has the model routing keys pilotfish relies on: `model`, `fallbackModel`, and `availableModels`. If `availableModels` is present as an allowlist, it must include entries for opus, sonnet, and haiku. Otherwise the role agents silently inherit the main-session model and routing does nothing.
5. Re-run `maxx setup` and confirm the pilotfish-sync step. It symlinks `agents/` into each non-canonical profile and replicates the settings keys.
6. Handle the CLAUDE.md marker block per section 2.
7. Run `maxx doctor`. Everything under each profile should now pass, except auth for accounts you haven't logged into yet.

## 2. Merge the pilotfish CLAUDE.md block into a customized CLAUDE.md

pilotfish ships a block for the user-level `CLAUDE.md`, delimited by marker comments. On a machine where `CLAUDE.md` has been customized, blind replacement would destroy local edits, which is why the script never touches the file. The customization may also live outside the markers, and everything outside the markers must survive the merge untouched.

1. Locate both files: the block pilotfish ships (in its repo or install output) and the target `CLAUDE.md` in the canonical config dir.
2. If the target has no marker block yet, append the shipped block at the end of the file, markers included.
3. If the target already has the markers, diff the content between the markers against the shipped version.
   - If the local changes are only outside the markers, replace just the marked region with the shipped version.
   - If there are local edits inside the markers, merge by hand. Keep local additions that don't contradict the shipped block, take the shipped version of anything it updates, and note anything you dropped.
4. Only the canonical profile's `CLAUDE.md` needs this. Other profiles pick up agent behavior through the `agents/` symlink. If a non-canonical profile has its own customized `CLAUDE.md`, decide deliberately whether it should match. Do not assume.

## 3. Recover the agents/ symlink after a Claude Code update destroys it

A Claude Code update can recreate its config directory structure and replace the `agents/` symlink in a non-canonical profile with a real directory. `maxx doctor` reports it as `agents/ is a real directory, not a symlink`. `maxx setup` fixes it automatically only when the recreated directory is empty. If it has content:

1. Compare the recreated directory against the canonical one: `diff -r ~/.claude-alt/agents ~/.claude-main/agents` (adjust paths).
2. Files identical to canonical are safe to discard with the directory.
3. For files that exist only in the recreated directory, or that differ, decide per file. If the update generated it, discard it. If it is a real agent definition someone added while the symlink was broken, move it into the canonical `agents/` so every profile gets it.
4. Remove the recreated directory and restore the link:

   ```sh
   rm -rf ~/.claude-alt/agents
   ln -s ~/.claude-main/agents ~/.claude-alt/agents
   ```

   Or re-run `maxx setup`, which recreates the symlink once the path is clear.
5. Run `maxx doctor` to confirm, and check the other non-canonical profiles too. Updates rarely break just one.

## 4. Add or retire a profile

Profiles are data, so neither operation changes code. Retiring takes judgment.

To add: edit `~/.config/maxx/profiles.json` and append one entry with `name`, `config_dir`, `wrapper`, and `tool` (keep `tool: "claude"` for Claude accounts). Run `maxx setup`, then log in through the new wrapper. Re-run `maxx pin` in any repo that should bill the new account.

To retire: before deleting the entry, check what points at it. Look for repos pinned to it (`git grep -l CLAUDE_CONFIG_DIR` across `.vscode/settings.json` files, or fix them as you find them), the ccmanager `defaultPresetId`, and any scripts that call the wrapper. Remove the entry, delete the wrapper from `~/bin`, and remove its ccmanager preset. Only then decide whether to delete the config dir. It contains that account's history and settings, so archive it rather than delete it if in doubt.

## 5. Share more than agents/ across profiles

Each config dir is a complete, separate Claude Code installation. Skills, plugins, slash commands, and hooks in one profile do not exist in the others, and maxx only bridges `agents/`. You can share more, but know what you are signing up for: by default `maxx doctor` checks the `agents/` symlink and nothing else. A hand-made link destroyed by the failure mode in section 3 goes undetected unless you register it.

1. Symlink the directories by hand from the canonical profile into the others: `skills/`, `plugins/`, `commands/`.
2. Copy the `enabledPlugins` key into each profile's `settings.json`. The plugins directory alone activates nothing.
3. Hooks cannot be symlinked at all. They live inside each profile's `settings.json`, so copy them into every profile that wants them.
4. Register the links by adding a `watch_links` map to `~/.config/maxx/profiles.json`. doctor then guards them with the same check it runs on the managed `agents/` symlink:

   ```json
   "watch_links": {
     ".claude-main": ["skills", "plugins", "commands"]
   }
   ```

   Keys are config dirs ($HOME-relative or absolute, same as `config_dir`). Values are the link names inside them.

maxx never creates these links. Registering only means doctor refuses to let them break silently. Links you don't register remain on you, especially after a Claude Code update, and after the first login in a fresh profile, which scaffolds directory structure.

## 6. Upgrade Claude Code

The ritual is doctor, upgrade, doctor, one grep.

1. Run `maxx doctor` to get a green baseline first, so anything the upgrade breaks is clearly the upgrade's doing.
2. Upgrade Claude Code (brew or however you installed it).
3. Run `maxx doctor` again. It catches a clobbered `agents/` symlink. `maxx setup` repairs it automatically when the recreated directory is empty. Section 3 covers the merge when the directory has content.
4. If you made the hand-made links from section 5, register them in `watch_links`, and the doctor run in step 3 covers them too. Check anything unregistered yourself: `ls -la <config dir> | grep '\->'` should still show every arrow. If an update replaced a link with a real directory, follow section 3: compare against the link target, salvage anything real, remove the directory, and relink.

Upgrades don't rewrite `settings.json` or `CLAUDE.md`, so hooks and agent-pack config need no attention.
