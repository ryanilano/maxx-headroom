# All prompt packs recovered from the 29M session

_Source: BACKUP-new.jsonl. 5 distinct paste-ready packs found._


## Pack 1 (line 615)

```
# NEEDS: path to the NPM data (database.sqlite, MySQL dump, or a backup archive). The NPM host 10.10.1.69 is DOWN and its guest was not found on pve-thick or pve-yolo, so the live instance cannot be read.
# NEEDS: whether that NPM used the SQLite or MySQL backend.

# Objective
Ship a CLI that reads an nginx-proxy-manager database and emits a Caddyfile byte-comparable in structure to a hand-written reference.

# Context
- Read first: the reference Caddyfile at /etc/caddy/Caddyfile on LXC 200 ("caddy", 10.10.1.182, on pve-yolo). It is the output contract. Copy it out before writing code.
- Read second: the NPM schema, from the database itself. Do NOT assume table or column names; introspect and print them before mapping.
- Stack: Caddy v2.11.4. Reference deployment uses a wildcard cert via DNS-01.
- Constraints:
  - Output ONE site block for the wildcard domain, containing `@<slug> host <fqdn>` + `handle @<slug> { ... }` per proxy host. Do NOT emit one site block per hostname; that is the default shape of every existing converter and it is wrong here.
  - Preserve a trailing catch-all `handle { }` block when the reference has one.
  - Emit tabs for indentation, matching the reference.
  - Read-only against NPM. Never write to its database.
  - Do not emit ACME staging endpoints. If the source config implies staging, warn and emit production.

# Task
1. Read: the reference Caddyfile; the NPM database schema; every enabled proxy host, redirection host, dead/404 host, stream, access list, and custom location.
2. Analyze:
   - Map NPM fields to Caddy directives: forward_scheme/host/port -> reverse_proxy; websocket upgrade; block_exploits; caching; ssl_forced -> redir or automatic HTTPS; hsts; http2; per-host advanced_config (raw nginx, cannot be translated).
   - Decide the slug for each @matcher from the hostname, deterministically, and guarantee uniqueness.
   - Group hosts sharing a parent domain under one wildcard site block; hosts outside it get their own blocks.
   - Classify every NPM feature as TRANSLATED, APPROXIMATED, or UNTRANSLATABLE. Raw nginx in advanced_config and access lists are the hard cases.
3. Produce: the CLI, plus a migration report listing every host and its classification, with UNTRANSLATABLE items called out individually and never silently dropped.

# Output
- Deliverable: a CLI taking the NPM database path and emitting (a) a Caddyfile to stdout or a named path, (b) a separate migration report.
- Acceptance:
  - `caddy validate --config <output>` exits 0.
  - `caddy fmt --diff <output>` produces no diff.
  - Round-trip check: run it against a database reconstructed from the reference Caddyfile's hosts, and diff output against the reference. Every difference is either explained in the report or a bug.
  - Every host in the source database appears exactly once in the output or exactly once in the report as UNTRANSLATABLE. No host is unaccounted for.
  - Running twice on the same input produces identical bytes.
```


## Pack 2 (line 832)

```
# Objective
Ship a CLI that reads an nginx-proxy-manager database and emits a Caddyfile matching a hand-written reference's structure.

# Context
- Read first: reference Caddyfile at /etc/caddy/Caddyfile on LXC 200 ("caddy", 10.10.1.182, pve-yolo). This is the output contract; copy it out before writing code.
- Test fixture: run nginx-proxy-manager in Docker, add a spread of proxy hosts (plain, websocket, ssl_forced, custom locations, access list, raw advanced_config), and use its database as the test input. Do not depend on any pre-existing NPM instance.
- Stack: Caddy v2.11.4.
- Prior art, and none of it does this: CaddyManager and CaddyProxyManager are web UIs for Caddy, not migrations off NPM. A 2026 DiyMediaServer guide gives the extraction query but nothing that produces a Caddyfile. Do not rebuild a UI.
- Schema starting point, to verify not assume: table proxy_host, filter is_deleted = 0, columns domain_names, forward_scheme, forward_host, forward_port. Introspect and print the real schema first; NPM versions differ.
- Constraints:
  - Emit ONE site block per parent domain containing `@<slug> host <fqdn>` + `handle @<slug> { ... }` per proxy host. Do NOT emit one site block per hostname. Every existing guide does the latter and it is wrong here.
  - Preserve a trailing catch-all `handle { }` when the reference has one.
  - Tabs for indentation, matching the reference.
  - Read-only against NPM. Never write to its database.
  - Never emit an ACME staging endpoint.

# Task
1. Read: the reference Caddyfile; the live schema; every enabled proxy host, redirection host, dead/404 host, stream, access list, and custom location.
2. Analyze:
   - Map NPM fields to Caddy: forward_scheme/host/port -> reverse_proxy; websocket upgrade; block_exploits; caching; ssl_forced; hsts; http2; per-host advanced_config (raw nginx, untranslatable).
   - Derive a deterministic, unique @matcher slug per hostname.
   - Group hosts by parent domain; hosts outside the wildcard get their own blocks.
   - Classify every feature TRANSLATED, APPROXIMATED, or UNTRANSLATABLE. advanced_config and access lists are the hard cases and are where a silent drop becomes an exposure.
3. Produce: the CLI plus a migration report.

# Output
- Deliverable: a CLI taking an NPM database path, emitting a Caddyfile and a separate migration report.
- Acceptance:
  - `caddy validate --config <output>` exits 0.
  - `caddy fmt --diff <output>` produces no diff.
  - Every host in the source appears exactly once in the output, or exactly once in the report as UNTRANSLATABLE. Nothing unaccounted for.
  - Running twice on identical input produces identical bytes.
  - Against the fixture database, the emitted config round-trips: every host reachable through Caddy resolves to the same backend NPM sent it to.
```


## Pack 3 (line 5197)

```
# Objective
Produce a defensive threat model: every attack vector a hostile corporate adversary (OpenAI, a defendant's counsel, a hired PI, a hostile online crowd — NOT Anthropic) could use against Ryan, each paired with a concrete way to close it before he goes public.

# Context
- Read first, in this order, BEFORE writing anything:
  1. Ryan's own notes corpus (handwritten + GenAI), which is LARGER than your context window. Ingest it in chunks: list the files, read in passes, summarize each pass to a scratch file, never assume you've seen it all. His words are the ground truth for what's already public and what he's already said.
  2. ~/local-dev/agent-notes/ — everything committed, especially attestations/, legal/2026-08-kings-county-filing/, and any 2026-08-22 session notes.
  3. His public LinkedIn footprint as he describes it (boosted post, tagged comment, the "not for fame or money" caps line).
- Facts about the subject: pro-se litigant, documented ADHD (this is STANDING, an asset, not a weakness), public LinkedIn presence, portfolio launching Monday 2026-08-24, about to file accessibility/disability discrimination claims and publish signed receipts to a public repo.
- Constraints:
  - DEFENSIVE ONLY. Every vector must end in "how Ryan closes it." Never produce anything that attacks a person.
  - The adversary is anyone EXCEPT Anthropic. Explicitly model OpenAI, opposing counsel, a PI, and a hostile crowd.
  - Distinguish LIVE exposure (already public, already said) from FUTURE exposure. Rank live first.
  - Flag anything that is spoliation, doxxing, or self-incrimination if mishandled.
  - No legal advice as fact — mark "consult counsel" where a lawyer must decide.

# Task
1. Read: Ryan's full notes corpus in chunked passes, plus agent-notes and his described public footprint. Build an inventory of everything he has ALREADY published or written that could be quoted back at him.
2. Analyze: for each of these vector classes, enumerate the specific concrete attacks and the exact defense — (a) his own published words as exhibits, (b) discovery turning his repos/DMs/journal against him, (c) serial-litigant/vexatious framing, (d) health/stability used to discredit, (e) doxxing/stylometry/metadata deanonymization, (f) the Monday portfolio launch as attack surface, (g) financial/longevity/permanent-record. Rank by how live and how damaging.
3. Produce: a single ranked threat-model document, each vector with: the attack, who runs it, live-vs-future, severity, and the specific action Ryan takes to close it before going public.

# Output
- Deliverable: one markdown file, THREAT-MODEL.md, written to Ryan's private tailnet notes (NOT GitHub), age-encrypted after write.
- Acceptance: every vector has a named adversary, a live/future tag, a severity, and a concrete close-it action; live exposures ranked above future ones; the file is encrypted at rest; nothing in it targets or attacks any named person.
```


## Pack 4 (line 5235)

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


## Pack 5 (line 5418)

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
