# Deploying the Schools+ Ops WOW dashboard

Two steps: put this folder on GitHub, then point Netlify at it. Netlify Blobs
turns itself on, so there is no database to create and no environment variable
to set.

## 1. Push to GitHub

From inside this folder:

```bash
git init
git add .
git commit -m "Schools+ Ops WOW dashboard"
git branch -M main
git remote add origin https://github.com/<your-org>/sbs-schools-wow-dashboard.git
git push -u origin main
```

## 2. Connect Netlify

1. Netlify, **Add new site**, **Import an existing project**, pick the repo.
2. Build settings:
   - Build command: leave blank
   - Publish directory: `.`
   - Functions directory: `netlify/functions`

   `netlify.toml` already sets all three, so the values Netlify shows should
   already be correct.
3. Deploy. Netlify installs `@netlify/blobs` from `package.json` on the first
   build.

Every push to `main` redeploys automatically.

### Or deploy straight from your machine

```bash
npm install -g netlify-cli
netlify login
netlify deploy --build --prod
```

## 3. Smoke test

1. Open the site. The badge in the header should read **Shared**.
2. Drop a Service Line Detail Report CSV anywhere on the page. It uploads, and
   the badge shows the import count.
3. Open the same URL in a different browser or on your phone. The same data and
   the same targets should be there.

If the badge says **This browser** on the hosted site, the functions are not
answering. Check the Netlify deploy log for a functions bundling error, and
confirm the functions directory is `netlify/functions`.

## What is stored where

| Thing | Where it lives |
| --- | --- |
| Uploaded SLDR imports | Netlify Blobs, store `sbs-wow`, keys `import/<id>` plus an `imports-index` |
| Targets, forecast, Needs List figures | Netlify Blobs, key `targets` |
| Fallback snapshot | Inside `index.html`, used only when the functions are unreachable |
| Per viewer conveniences | `localStorage`, used only when there is no backend |

Uploads accumulate. When two imports both cover a week, the newer upload wins,
so re-uploading a corrected export supersedes the old figures without anyone
deleting anything. The 24 most recent imports are kept and older ones drop off
automatically.

## Endpoints

| Route | Methods | Purpose |
| --- | --- | --- |
| `/api/targets` | GET, PUT, DELETE | The shared targets document |
| `/api/imports` | GET, POST, DELETE | Import index, one import by `?id=`, upload, remove by `?id=` or `?all=1` |

## Access

The site and both endpoints are open to anyone with the URL, which is the usual
starting point for an internal tool. To restrict it, the two straightforward
options are a shared write key (reads stay open, writing needs a key entered
once per browser) or Netlify Identity logins. Either is a small change.

## Embedding in Smartsheet

Use a Web Content widget pointed at the Netlify URL. A self hosted page like
this one is embeddable, which is the main reason to host it here rather than
leaving it on claude.ai.
