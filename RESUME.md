---
domd-id: 4a1ca9f6-0efd-43ea-bc97-999b4716c9c1
---

# Resume prompt

Paste everything below into a fresh Claude Code session started in `~/local-dev/ilano-fyi`.

---

You are picking up a session that ran out of context. Two work threads were in
flight. Read the notes named below before acting; do not re-derive what is
already written down.

## Thread 1: home network cutover (the main event)

Goal: re-scheme VLANs and cut boxes over to the new layout, one device at a
time, with minimal downtime. Approach is device-by-device because nothing is in
"prod" yet, so a mistake on one box is cheap.

Hardware in play:

- MikroTik switches: CRS309 and CRS326. The CRS309 has been idle ("doing fuck
  all for 40 days") and is the current focus. A CRS326 has a stuck BIDI SFP
  adapter noted.
- Hasivo whitebox switch (model F1100WP-4SX-4XGT, firmware 7.1.5). A postmortem
  on its firmware exists.
- Omada / TP-Link WAP (a 250-300 unit bought at ~150 off). PoE-related reboot
  troubleshooting happened; WiFi was flaky on both bands during the session.
- Proxmox cluster: several nodes including pve-media, pve-llm, pve-yolo,
  pve-agent. Plex is being isolated into its own VLAN so its one open port can't
  reach anything else (the "dev laptop on the same network as Plex got pwned"
  scenario is the thing being designed out).

Serial console work was underway (USB-serial + DB9/RJ45 into the switches from a
MacBook Air).

Read these first, in this order:

- `~/local-dev/agent-notes/network/2026-08-22-RUNBOOK-saturday-cutover.md`  (the runbook, most-touched file)
- `~/local-dev/agent-notes/network/2026-08-21-CONFIG-v2-final-scheme.md`
- `~/local-dev/agent-notes/network/2026-08-21-SCHEME-final-subnets.md`
- `~/local-dev/agent-notes/network/VLAN_Mapping.md`
- `~/local-dev/agent-notes/network/2026-08-21-BLOCKER-vmid-collision.md`  (a VM-ID collision blocker to resolve)
- `~/local-dev/agent-notes/network/2026-08-22-POSTMORTEM-whitebox-switch-firmware.md`

The single source of truth is these markdown notes. Keep writing state into them
as you go; the owner will forget anything not on disk.

## Thread 2: YouTube ingestion pipeline

Goal: slurp favorite YouTube channels into a searchable knowledge base, then
generate deep analysis across many videos at once (e.g. "ask it about 50
videos").

Pieces:

- `yt-cc` / `yt-transcript` skills pull captions. Always pass `-d` so a file is
  written; the bare form only copies to clipboard. Output goes to
  `~/local-dev/_yt-transcripts`, filename = video title.
- Open Notebook runs in Docker (compose project `forge`, containers
  `forge-open_notebook-1` and `forge-surrealdb-1`) on pve-llm, a Debian LXC. It
  is NOT set to autoload; a model script (something like `qwen35b.sh`) is kicked
  off by hand.
- KNOWN BLOCKER: `docker compose up` on the Open Notebook stack failed with
  `failed to create shim ... unsupported protocol: unix`. That container would
  not start. Fixing this is the first YT-thread task.
- Requested but not confirmed done: a script that writes each video's insights
  out to a SEPARATE PDF, saved into Dropbox `_inbox` (the owner's personal
  inbox).

## Immediate next actions

1. Open the cutover runbook and report the current step and what's left.
2. Diagnose the Open Notebook Docker shim error (`unsupported protocol: unix`),
   which is blocking the whole YT pipeline.
3. Confirm whether the insights-to-separate-PDFs script exists and works.

Ask the owner which thread to drive first. Keep responses short and lead with
the next concrete action; the owner has ADHD and works best from an obvious
small first step.
