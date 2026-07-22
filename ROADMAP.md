# Roadmap

Ideas under consideration, not commitments. Items land here when they survive a discussion, and they leave by shipping or by being written off in [CONSIDERATIONS.md](notes/CONSIDERATIONS.md).

- **Non-Claude subscription profiles.** Profiles are data, so a GPT or Kimi coding-agent subscription should slot in as one more entry in `profiles.json` with its own `tool` and wrapper. The plumbing exists, but nothing has exercised it against a real second tool yet.
- **Per-profile provider data.** Borrowed from cc-switch. If a profile ever needs a non-default endpoint or provider settings, that belongs in its `profiles.json` entry rather than in hand-edited config files.
- **Headroom visibility.** Borrowed from cc-switch's usage dashboard. The resource being managed is which account has room left, and today nothing shows it. A `maxx` subcommand that reports recent usage per profile would make the pick-an-account decision informed instead of guessed.
- **Re-evaluate cc-switch** if API relays or non-Claude providers enter the picture. The conditions are written down in [CONSIDERATIONS.md](notes/CONSIDERATIONS.md).
