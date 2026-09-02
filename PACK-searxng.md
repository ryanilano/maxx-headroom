Read these three first, in order, before anything:
1. ~/local-dev/agent-notes/2026-08-22-NOTE-to-the-next-resurrection.md
2. ~/local-dev/agent-notes/sessions/2026-08-22-the-all-nighter-README.md
3. CLAUDE.md + the memory files

We're setting up SearXNG so you search through my stack, not your black-box web tool.

# Objective
Route agent web search through my SearXNG instead of the built-in tool — private, unlogged, my engines. The standing project is "unfuck the agent" by routing its research through infrastructure I control.

# Context
- SearXNG runs TWO places: LXC 113 "searxng" on pve-thick (DHCP), and LXC 108 on pve-media at 10.10.1.102:8888. Resolve which is canonical, don't guess.
- Nodes: pve-thick 10.10.1.12, pve-yolo 10.10.1.13, pve-media 10.10.1.11, pve-llm 10.10.1.16. Keys: agent-unattended for yolo/media, `ssh pve-thick`, `ssh pve-agent` for llm.
- Local LLM: Qwen3.6-35B-A3B MoE on llamacpp (LXC 104, ports 8080 chat / 8082 embed).
- Pattern to copy: ~/.local/bin/ppx and ~/.local/bin/yt-cc — same wrapper shape.
- Commits go to Forgejo archive/agent-notes, never GitHub.

# Task
1. curl both SearXNG instances, see which responds and what engines each has.
2. Decide the cleanest hook: a ~/.local/bin/srx wrapper hitting the JSON API.
3. Build it. Commit a note recording endpoint, engines, and which instance won.

# Output
- ~/.local/bin/srx returning real SearXNG results, + a committed note recording endpoint, engine list, and which of the two searxng LXCs is canonical.
- Acceptance: `srx "test"` returns SearXNG results, not the default web tool.

Rules: act on reversible work, report after, don't ask. No yes-man. My name is ALL CAPS. Never tell me to sleep. Verify with curl, not assessment.
