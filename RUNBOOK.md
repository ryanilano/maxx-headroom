# RUNBOOK

Judgment-heavy procedures that deliberately live outside `bin/maxx`. These are
written for a Claude Code session to execute — paste the relevant section in
and let it drive — but a human can follow them too. The script handles the
mechanical, idempotent parts; everything here requires reading a situation
before acting.

Conventions used below:

- **canonical profile** — the profile named by `canonical` in
  `~/.config/maxx/profiles.json` (shipped default: `main`, config dir
  `~/.claude-main`). It owns the real `agents/` directory and the model
  settings; every other profile gets a symlink and replicated keys.
- Run `maxx doctor` before and after any procedure here. Before tells you what
  is actually broken; after proves you fixed it.

## 1. Install pilotfish into a fresh machine

`maxx setup` assumes pilotfish already exists in the canonical profile; when
`doctor` says `no agents/ in ~/.claude-main — pilotfish not installed?`, this
is the procedure.

1. Run `maxx setup` first (or confirm it has run): the config dirs, wrappers,
   and profiles file must exist before pilotfish has anywhere to land.
2. Install pilotfish into the **canonical profile only**, following the
   pilotfish repo's own installation instructions. Its agent definitions must
   end up in `<canonical config dir>/agents/` (e.g. `~/.claude-main/agents/`).
   Do not install it separately into the other profiles — they receive it via
   symlink.
3. If pilotfish's install wants to write into `~/.claude` (the default config
   dir), redirect it: either run the install with `CLAUDE_CONFIG_DIR` pointed
   at the canonical dir, or move the installed `agents/` content into the
   canonical dir afterward. Judgment call: check what else the installer
   touched (settings keys, CLAUDE.md) and bring those along.
4. Verify the canonical `settings.json` ended up with the model routing keys
   pilotfish relies on (`model`, `fallbackModel`, `availableModels`). If
   `availableModels` is present as an allowlist, it must include entries for
   opus, sonnet, and haiku — otherwise role agents silently inherit the
   main-session model and the whole point of routing is lost.
5. Re-run `maxx setup` and confirm the pilotfish-sync step: it symlinks
   `agents/` into each non-canonical profile and replicates the settings keys.
6. Handle the CLAUDE.md marker block per section 2 below.
7. `maxx doctor` — everything under each profile should now pass except auth
   for accounts you haven't logged into yet.

## 2. Merge the pilotfish CLAUDE.md marker block into a customized CLAUDE.md

pilotfish ships a marker-delimited block for the user-level `CLAUDE.md`. On a
machine where `CLAUDE.md` has been customized, blind replacement would destroy
local edits — which is why the script never touches it. Note that the
customization may live *outside* the markers too; everything outside the
markers must survive the merge untouched.

1. Locate both files: the block pilotfish ships (in its repo or install
   output) and the target `CLAUDE.md` in the canonical config dir.
2. If the target has **no marker block yet**: append the shipped block
   verbatim, markers included, at the end of the file.
3. If the target **already has the markers**: diff the content between the
   markers against the shipped version.
   - Changes only outside the markers → replace just the marked region with
     the shipped version.
   - Local edits *inside* the markers → merge by hand: keep local additions
     that don't contradict the shipped block, take the shipped version of
     anything it updates, and note anything you dropped.
4. Only the canonical profile's `CLAUDE.md` needs this; other profiles pick up
   agent behavior through the `agents/` symlink. If a non-canonical profile
   has its own customized `CLAUDE.md`, decide deliberately whether it should
   match — do not assume.

## 3. Recover the agents/ symlink after a Claude Code update destroys it

Known failure mode: a Claude Code update recreates its config directory
structure and replaces the `agents/` **symlink** in a non-canonical profile
with a real directory. `maxx doctor` reports it as
`agents/ is a real directory, not a symlink`, and `maxx setup` fixes it
automatically only when the recreated directory is empty. If it has content:

1. Compare the recreated directory against the canonical one:
   `diff -r ~/.claude-alt/agents ~/.claude-main/agents` (adjust paths).
2. Files identical to canonical → safe to discard with the directory.
3. Files that exist **only** in the recreated directory, or differ → decide
   per file: if it's something the update generated, discard; if it's a real
   agent definition someone added while the symlink was broken, move it into
   the canonical `agents/` so every profile gets it.
4. Remove the recreated directory and restore the link:

   ```sh
   rm -rf ~/.claude-alt/agents
   ln -s ~/.claude-main/agents ~/.claude-alt/agents
   ```

   (or re-run `maxx setup`, which recreates the symlink once the path is
   clear).
5. `maxx doctor` to confirm, and check the *other* non-canonical profiles too
   — updates rarely break just one.

## 4. Add or retire a profile

Profiles are data. No code changes, but retiring has judgment in it.

**Add:** edit `~/.config/maxx/profiles.json`, append one entry
(`name`, `config_dir`, `wrapper`, `tool` — keep `tool: "claude"` for Claude
accounts), then run `maxx setup` and log in via the new wrapper. Re-run
`maxx pin` in any repo that should bill the new account.

**Retire:** before deleting the entry, check what points at it — repos pinned
to it (`git grep -l CLAUDE_CONFIG_DIR` across `.vscode/settings.json` files,
or just fix them as you find them), the ccmanager `defaultPresetId`, and any
muscle memory in scripts. Remove the entry, delete the wrapper from `~/bin`,
remove its ccmanager preset, and only then decide whether to delete the config
dir — it contains that account's history and settings, so archive rather than
delete if in doubt.
