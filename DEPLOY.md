# Deploy — Cloudflare Pages

We host on Cloudflare Pages (not GitHub Pages) because Apple requires the AASA
to be served with `Content-Type: application/json`. GitHub Pages serves it as
`application/octet-stream` and offers no way to override headers. Cloudflare
Pages honors a `_headers` file (already in this repo) and is free.

## 1. Push this folder to a GitHub repo

The folder `pushboss-links/` is intentionally **not gitignored** in the parent
`factorz` repo, but Cloudflare Pages needs its own repo to deploy from.

Easiest path: create a separate GitHub repo just for this folder.

```bash
cd C:/Users/viole/factorz/pushboss-links
git init
git add -A
git commit -m "init: PushBoss universal links"
git branch -M main
# create empty repo on github.com named `pushboss-links` (any name works)
git remote add origin https://github.com/<your-user>/pushboss-links.git
git push -u origin main
```

## 2. Connect to Cloudflare Pages

1. Go to https://dash.cloudflare.com/ → **Workers & Pages** → **Create** →
   **Pages** → **Connect to Git**.
2. Pick the `pushboss-links` repo.
3. Build settings:
   - **Framework preset**: None
   - **Build command**: *(leave empty)*
   - **Build output directory**: `/`
4. Deploy. First build takes ~30s. You get a URL like
   `pushboss-links.pages.dev`.

> **Important — `.well-known` directory.** Cloudflare Pages can be flaky with
> dotfile directories. We have **two safety nets**:
> 1. The AASA exists at both `/apple-app-site-association` (root) AND
>    `/.well-known/apple-app-site-association`. Apple tries `.well-known`
>    first, falls back to root.
> 2. `_redirects` rewrites `/.well-known/apple-app-site-association` → root
>    with status 200 in case the dotfile dir is dropped on deploy.

## 3. Verify AASA is served correctly

```bash
# Primary location (Apple checks this first)
curl -i https://pushboss-links.pages.dev/.well-known/apple-app-site-association

# Fallback location (Apple checks if .well-known 404s)
curl -i https://pushboss-links.pages.dev/apple-app-site-association
```

Both must show:
- `HTTP/2 200`
- `content-type: application/json` ← critical, Apple silently rejects others
- JSON body matching the AASA file

If `content-type` is wrong, the `_headers` file isn't being applied. Check the
deploy log in Cloudflare for warnings.

You can also validate via Branch.io's tool:
https://branch.io/resources/aasa-validator/

## 4. Verify Apple's CDN can fetch it

```bash
curl -i "https://app-site-association.cdn-apple.com/a/v1/pushboss-links.pages.dev"
```

May 404 for up to 24h after first publish. Apple refetches on its own schedule.

## 5. Update app.json + referral.ts to the live domain

Once the Cloudflare URL is final, update:

- `factorz/app.json` → `ios.associatedDomains` →
  `["applinks:pushboss-links.pages.dev"]` (or your custom domain)
- `factorz/src/services/referral.ts` → `CHALLENGE_WEB_BASE` and the invite
  URL builder → same domain.

## 6. Rebuild the iOS app

`associatedDomains` is an entitlement — iOS only honors it when baked into
the provisioning profile.

```bash
cd C:/Users/viole/factorz
eas build --platform ios --profile production
```

EAS detects the new entitlement, prompts to update the provisioning profile,
and produces a build with the `applinks:` entitlement set.

## 7. Test

Install the new build on a device. Send yourself a challenge link (e.g.
`https://pushboss-links.pages.dev/pushboss/challenge/4/50/150000`) via
Messages or Mail and tap it. iOS should open the app directly and show the
challenge banner on Home.

If it opens Safari instead:
- Long-press the link → "Open in PushBoss" should appear. If yes, Universal
  Links work but iOS opted out for that link (one-time user override). Tap
  "Open in PushBoss".
- If "Open in PushBoss" doesn't appear, AASA isn't reachable from Apple's CDN
  yet (wait 24h) or the Content-Type isn't `application/json` (recheck
  step 3).

## 8. Debug — Universal Links not opening the app

```bash
# Signed AASA from Apple's CDN
curl -i "https://app-site-association.cdn-apple.com/a/v1/pushboss-links.pages.dev"

# Direct AASA from your origin
curl -i https://pushboss-links.pages.dev/.well-known/apple-app-site-association
```

Common causes:
- AASA cached old. Reinstall the app to force iOS to re-check on first launch.
- Path mismatch. `components` in AASA must include the exact path. We've set
  `/pushboss/challenge/*` and `/pushboss/invite/*` (and `/pushboss/invite`).
- Wrong Team ID in `appIDs`. Must be `LL3A7K88VF.com.pushboss.app`.
- Wrong Content-Type. Must be `application/json` exactly.
