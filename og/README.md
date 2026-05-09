# OG preview images

Drop two PNG files here:

- **`challenge.png`** — preview shown when a Challenge link is shared in
  WhatsApp / Telegram / iMessage / Messenger.
- **`invite.png`** — preview shown for Invite links.

## Specs

- **Size**: 1200×630 px (Open Graph standard, used by all major messengers)
- **Format**: PNG (or JPG works too — keep filename matching the meta tag)
- **Weight**: ≤ 300 KB ideally so the preview loads instantly. Compress with
  TinyPNG / Squoosh if larger.
- **Safe area**: keep important text within the centered 900×500 area —
  some clients crop edges.

## What to include (suggestion)

- Big PushBoss branding / monster emoji
- "Challenge" / "Invite" tagline so the receiver instantly understands what
  the link is
- Avoid putting reps/time/level numbers — the preview is static, JS only
  populates the on-page card after click
