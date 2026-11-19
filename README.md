# LemonShot — Next.js edition (Vercel ready)

LemonShot inspects up to 30 URLs of a website, picks up to five representative page types (one blog post when found), takes full-page screenshots on Desktop (1440 px), Tablet (834 px) and Mobile (390 px), collects every image, and downloads everything as one ZIP.

## Deploy on Vercel

1. Push this folder to a GitHub repository and import it at vercel.com/new (framework: Next.js, no settings needed).
2. In the Vercel project open **Storage → Create → Blob** and connect it to the project. This adds `BLOB_READ_WRITE_TOKEN`; finished ZIPs are stored there.
3. Optional: add `CRON_SECRET` (protects the daily cleanup in `vercel.json` that deletes ZIPs older than `EXPORT_RETENTION_HOURS`, default 24).
4. Redeploy. See `.env.example` for all settings.

The capture route runs for up to 300 s (`maxDuration` in `app/api/capture/route.js`). The crawler stops starting new pages after `CAPTURE_TIME_BUDGET_SECONDS` (default 240) and packages what it has. On Vercel Pro you can raise both.

## Run locally

```bash
npm install
npx playwright install chromium
npm run dev        # or: npm run build && npm start
```

Open http://localhost:4173. On macOS you can also double-click `start.command`. Locally, ZIPs are served from your own machine — no Blob store needed.

## How it works

- `POST /api/capture` runs the whole capture inside one request and streams progress as newline-delimited JSON. Pressing **Stop** (or closing the tab) cancels it.
- On Vercel, Chromium comes from `@sparticuz/chromium` with `playwright-core`; locally, Playwright's own Chromium is used (or `CHROME_EXECUTABLE_PATH`).
- The ZIP is built in JavaScript (`archiver`), so no system `zip` command is needed.

## No lazy loading

Every captured page has lazy loading turned off before any of its scripts run: `loading="lazy"` on images and iframes is rewritten to `eager`, lazy `data-src` / `data-srcset` / `data-bg` sources are promoted, the page is scrolled to trigger viewport loaders, and all images are decoded before each screenshot. The LemonShot interface itself uses no lazy-loaded images or deferred code.

## Export contents

- `screenshots/desktop|tablet|mobile/` — full-page PNGs
- `image-assets/` — downloaded images (img, picture/source, srcset candidates, SVG images, posters, CSS backgrounds)
- `representatives.csv`, `scan-log.csv`, `image-inventory.csv`, `image-elements.json`, `report.json`, `README.txt`

## Notes

- Only same-site links are followed (subdomains optional); include/exclude path filters are in Advanced settings.
- On the hosted app, localhost, private and internal network addresses are blocked (`ALLOW_PRIVATE_NETWORKS=true` turns this off).
- Sign-in pages, bot protection, cookie prompts and infinite scroll may need custom handling.
- Only capture websites you own or have permission to archive.
# lemonshott
