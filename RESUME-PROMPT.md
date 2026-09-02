# Resume prompt — ilano-fyi network session (2026-08-22)

Paste everything below the line into a fresh Claude Code session started in
`~/local-dev/ilano-fyi`.

---

## Context

You are picking up a session that ran 2026-08-22 02:35 to 18:45 UTC and died on
"Prompt is too long." 309 user turns, 468 tool calls. This is a homelab network
cutover plus three side projects. I am Ryan. Some turns from the original
session are deliberately withheld and are not available to you; do not ask about
gaps in the numbering.

Read these first, in this order:

- `~/local-dev/agent-notes/network/2026-08-22-RUNBOOK-saturday-cutover.md` (most edited, 9 touches)
- `~/local-dev/agent-notes/network/2026-08-21-CONFIG-v2-final-scheme.md` (7 touches)
- `~/local-dev/agent-notes/network/2026-08-21-SCHEME-final-subnets.md`
- `~/local-dev/agent-notes/network/VLAN_Mapping.md`
- `~/local-dev/agent-notes/network/2026-08-21-HASIVO-cli-corrections-from-owner.md`
- `~/local-dev/agent-notes/network/2026-08-21-BLOCKER-vmid-collision.md`
- `~/local-dev/agent-notes/network/2026-08-22-POSTMORTEM-whitebox-switch-firmware.md`

Those notes are the single source of truth. The session convention was: write
findings to `agent-notes/` as they happen, treat it as an internal blog, do not
rely on session memory.

## Hardware inventory established during the session

**Switches**

- MikroTik CRS309 — has a stuck BIDI SFP adapter. Idle and doing nothing for ~40 days.
  Suspected misconfiguration on the passthrough port. Serial console access is the plan.
- MikroTik CRS326 — also in play.
- Hasivo F1100WP-4SX-4XGT — firmware 7.1.5, build Oct 14 2025; uboot
  `uboot_9300_240711.bin`. CLI syntax differs from docs; corrections captured in
  the HASIVO-cli-corrections note. Read that before issuing any command to it.

**Serial access** — Ryan has both DB9 and RJ45