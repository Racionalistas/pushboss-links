# pushboss-links

Cloudflare Pages site for PushBoss app deep links + Universal Links.

## What this serves

- `/.well-known/apple-app-site-association` — iOS Universal Links manifest
  (Team ID `LL3A7K88VF`, bundle `com.pushboss.app`). Served as
  `application/json` via the `_headers` file (Apple requirement).
- `/pushboss/challenge/{level}/{reps}/{ms}` — challenge landing. Tries to
  open the app via Universal Link → `pushboss://challenge/...` deep link →
  falls back to the App Store. The actual content lives in
  `pushboss/challenge.html`; `404.html` rewrites unknown challenge paths to
  that file via JS.
- `/pushboss/invite/{CODE?}` — invite landing, same pattern.

## Why Cloudflare Pages and not GitHub Pages

Apple requires AASA to be served with `Content-Type: application/json`.
GitHub Pages serves it as `application/octet-stream` and offers no header
override. Cloudflare Pages reads `_headers` and lets us pin the Content-Type.

## Adding a new path

1. Add the path glob to `.well-known/apple-app-site-association` under
   `applinks.details[0].components`.
2. Add a `pushboss/<thing>.html` page.
3. Add a routing rule to `404.html`.
4. Push to the connected GitHub repo. Cloudflare Pages auto-deploys.
5. Wait up to 24h for Apple's CDN to refetch the AASA file.

See `DEPLOY.md` for first-time setup.
