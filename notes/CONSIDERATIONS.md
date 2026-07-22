# Considered alternatives

Tools and ideas evaluated but passed on, with the reasoning written down so the question does not get reopened from scratch.

## cc-switch

Evaluated 2026-07-22.

[cc-switch](https://github.com/farion1231/cc-switch) is a desktop app (Tauri, Rust, MIT) that manages provider configuration for eight AI coding tools, Claude Code among them. It stores providers, MCP servers, prompts, and skills in a SQLite database under `~/.cc-switch/`, and it switches by rewriting each tool's live config file. At evaluation time it had about 120k stars and daily commit activity.

### Why it does not replace maxx-headroom

The two tools switch different things. maxx-headroom switches which account pays, per terminal and concurrently, through `CLAUDE_CONFIG_DIR`. cc-switch switches which provider endpoint one config directory points at, globally and one provider at a time.

In cc-switch's source, `get_claude_config_dir()` in [`src-tauri/src/config.rs`](https://github.com/farion1231/cc-switch/blob/main/src-tauri/src/config.rs) reads an app-internal setting and falls back to `~/.claude`. It never reads the `CLAUDE_CONFIG_DIR` environment variable, so it manages exactly one Claude config directory and cannot see `main` and `alt` at the same time. Its own [FAQ](https://github.com/farion1231/cc-switch#faq) says that switching back to an official login requires the log-out and log-in flow, and that only Codex supports switching between official accounts cleanly. So the core maxx job, two paid Claude accounts with no stranded quota and per-repo billing, stays unsolved.

### Why not run it alongside

cc-switch is a second writer on the same files maxx watches. It rewrites `settings.json`, syncs CLAUDE.md across tools, and symlinks skills from `~/.cc-switch/skills/` into app directories. `maxx doctor` exists because Claude Code upgrades already clobber the `agents/` symlink, and an uncoordinated second writer multiplies the failure modes doctor has to catch. maxx also replicates the model keys (`model`, `fallbackModel`, `availableModels`) from the canonical profile to the others, and provider keys that cc-switch injects into the canonical `settings.json` would sit outside that replication and quietly diverge.

Its ecosystem also leans a different way. The project's sponsor list is mostly API relay resellers, and its center of gravity is switching between third-party endpoints, not managing official subscription accounts.

### What it does well

One-click provider switching with 50+ presets, a usage and cost dashboard, and unified MCP and skills management across tools. The ideas worth borrowing are on the [roadmap](../ROADMAP.md).

### When to look again

If a profile ever points at an API relay or a non-Claude provider, cc-switch is the mature tool for that axis. Scope it to one dedicated config directory through its override setting, keep it away from the canonical profile, and add a doctor check for interference.
