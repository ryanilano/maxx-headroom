# Prompt packs, deduped — recovered from the all-nighter session

The 5 raw packs were 3 real projects (the NPM→Caddy and SearXNG packs each had
two revisions). Kept the latest/best of each. Paste any one into a fresh window.

---

## 1. NPM → Caddy converter CLI

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

---

## 2. Defensive threat model

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

---

## 3. SearXNG search routing

```
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
```
