# Changelog

Notable changes, newest first. Dates instead of version numbers, because the tool has no release process.

## 2026-07-24

- Added a banner image at [assets/tokenmaxxing.png](assets/tokenmaxxing.png) and placed it at the top of the README.
- Rewrote the README: a new "What It Does" section up front, an expanded origin story, and a nav row linking the main sections.
- Set the GitHub repo description and topics so the project is discoverable from search and the repo sidebar.

## 2026-07-23

- Added [CLAUDE.md](CLAUDE.md) with a two-tier punctuation rule: no em-dashes in `bin/maxx` output, rare and varied em-dashes in prose docs.
- Removed all 46 em-dashes from `bin/maxx`, in strings and comments. Replacements vary by message (colon, semicolon, parentheses, sentence split) so `maxx doctor` no longer reads as one repeated construction. No message changed meaning or wording.

## 2026-07-22

- Evaluated [cc-switch](https://github.com/farion1231/cc-switch) and decided not to adopt it. Reasoning in [CONSIDERATIONS.md](notes/CONSIDERATIONS.md).
- Added [ROADMAP.md](ROADMAP.md) and this changelog, and linked both from the README.
- Slimmed the README. Profile sharing moved to [RUNBOOK §5](RUNBOOK.md), the upgrade ritual to [RUNBOOK §6](RUNBOOK.md), and the ccmanager workflow to [docs/ccmanager.md](docs/ccmanager.md), each replaced with a short pointer.
- Rewrote the RUNBOOK in plain language. Same procedures, same section numbers.

## 2026-07-20

- Initial release: the `maxx` CLI with `setup`, `doctor`, and `pin`, plus README, RUNBOOK, and MIT license.
- Defined scope: model routing stays with the agent packs (pilotfish, GSD, Superpowers). maxx only symlinks the canonical `agents/` directory across profiles.
- Acceptance checks pass on the initial CLI.
- README filled out: prerequisites, daily usage, the ccmanager workflow, profile sharing with `watch_links`, and the symlink warning.
- Documented the Claude Code upgrade ritual: doctor, upgrade, doctor, one grep.
