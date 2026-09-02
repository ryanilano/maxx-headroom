===== assistant message line 5235 =====
Pack for the fresh window. Copy-paste it into a new session and it'll pick up cold with no re-explaining.

```
# Objective
Wire up SearXNG + Ryan's homelab infra so an agent searches through HIS stack (private, unlogged, better engines) instead of the default web tool — the standing project is "unfuck the agent" by routing its research through infrastructure he controls.

# Context
- Read first, in order:
  1. ~/local-dev/agent-notes/sessions/2026-08-22-the-all-nighter-README.md — full map of the 2026-08-22 session this continues.
  2. ~/local-dev/agent-notes/network/README.md — the fleet: 4 Proxmox nodes, all reachable, agent-unattended key installed on pve-yolo/pve-media.
  3. ~/local-dev/agent-notes/2026-08-22-STUB-maximal-legal-use.md — the strategic frame.
- Infra facts (verified 2026-08-22):
  - SearXNG runs as LXC 113 "searxng" on pve-thick, DHCP. A SECOND searxng exists as LXC 108 on pve-media (10.10.1.102:8888). Confirm which is canonical before wiring.
  - Nodes: pve-thick 10.10.1.12, pve-yolo 10.10.1.13, pve-media 10.10.1.11, pve-llm 10.10.1.16. SSH via `pve-agent` for llm; agent-unattended key for yolo/media; `ssh pve-thick` for thick.
  - Local LLM: Qwen3.6-35B-A3B MoE on llamacpp (LXC 104, ports 8080 chat / 8082 embed).
  - Everything commits to Forgejo `archive/agent-notes` on the tailnet, 1Password-locked, no unlock needed. Never GitHub.
- Constraints:
  - Content edits to src/content/ and infra work do NOT need GSD. This is infra.
  - Verify with tools, never assessments. curl the SearXNG endpoint, don't assume it's up.
  - Two searxng instances exist — resolve which one, don't guess.

# Task
1. Read: the three notes above, then curl both SearXNG instances to see which responds and what engines each has enabled.
2. Analyze: how to make an agent's web search route through SearXNG's JSON API instead of the built-in web tool — a wrapper script (like the ppx/yt-cc pattern in ~/.local/bin/), env config, or MCP. Decide the cleanest hook.
3. Produce: a working search-via-SearXNG path the agent can call, plus a note documenting it.

# Output
- Deliverable: a search wrapper (e.g. ~/.local/bin/srx) hitting SearXNG's JSON API, + a committed note in agent-notes describing the setup and which instance is canonical.
- Acceptance: `srx "test query"` returns real results from SearXNG (not the default web tool); the note records the endpoint, engine list, and which of the two searxng LXCs won.

# Standing context the new window needs
- 18 commits from 2026-08-22 are on Forgejo: homelab cutover (caddy cert, 5 backend fixes, VMID map), a Kings County NYCHRL filing package, Irregular eval-breach research, 9 signed attestations, session cost math. All in agent-notes.
- Read CLAUDE.md and the memory files first — Ryan has hard rules (never sweep files outside src/content/, act-don't-ask on reversible work, no em dashes, ALL CAPS on his name is load-bearing per DOOM).
- This session ran Opus 5 then fell to Opus 4.8 at turn 979 on a cyber-classifier refusal fallback. If research touches security topics, expect it.
```

That's everything a cold window needs. Nothing to re-explain — it reads the map, checks both SearXNG boxes, and builds the wrapper.

===== assistant message line 5418 =====
```
Read these three first, in order, before anything:
1. ~/local-dev/agent-notes/2026-08-22-NOTE-to-the-next-resurrection.md
2. ~/local-dev/agent-notes/sessions/2026-08-22-the-all-nighter-README.md
3. CLAUDE.md + the memory files

We're setting up SearXNG so you search through my stack, not your black-box web tool.

# Objective
Route agent web search through my SearXNG instead of the built-in tool — private, unlogged, my engines.

# Context
- SearXNG runs TWO places: LXC 113 "searxng" on pve-thick (DHCP), and LXC 108 on pve-media at 10.10.1.102:8888. Resolve which is canonical, don't guess.
- Nodes: pve-thick 10.10.1.12, pve-yolo 10.10.1.13, pve-media 10.10.1.11, pve-llm 10.10.1.16. Keys: agent-unattended for yolo/media, `ssh pve-thick`, `ssh pve-agent` for llm.
- Pattern to copy: ~/.local/bin/ppx and ~/.local/bin/yt-cc — same wrapper shape.
- Commits go to Forgejo archive/agent-notes, never GitHub.

# Task
1. curl both SearXNG instances, see which responds and what engines each has.
2. Decide the cleanest hook: a ~/.local/bin/srx wrapper hitting the JSON API.
3. Build it. Commit a note recording endpoint, engines, and which instance won.

# Output
- ~/.local/bin/srx returning real SearXNG results, + a committed note.
- Acceptance: `srx "test"` returns SearXNG results, not the default web tool.

Rules: act on reversible work, report after, don't ask. No yes-man. My name is ALL CAPS. Never tell me to sleep. Verify with curl, not assessment.
```

That's the whole handoff. New window, paste it, it reads the note first and picks up cold.
