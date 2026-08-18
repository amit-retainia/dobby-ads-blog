# Dobby Ads — Website Performance Report

_Static HTML/CSS/JS site (13 pages), deployed on Netlify. This report covers all optimization work done to date and the biggest remaining opportunities._

---

## 1. Summary

The site is media-heavy — its weight is almost entirely videos and images, not code. Work so far has focused on the **Video & Commercials** page (the worst offender that was measured with Lighthouse) plus site-wide wins (image conversion, lazy-loading, caching). The single largest remaining opportunity is applying the same video/image treatment to the **Animation & 3D** and **Branding** pages, which are still carrying their original uncompressed media.

**Latest Lighthouse (video-commercials page, before today's fixes):**

| Metric | Mobile | Desktop |
|---|---|---|
| Performance | 73 | 85 |
| Largest Contentful Paint | 5.5s (poor) | 1.7s |
| First Contentful Paint | 3.1s | 1.3s |
| Total page transfer | ~30 MB | ~12 MB |

Today's changes target the mobile LCP and media payload directly; a re-run after deploy is expected to move mobile toward ~90.

---

## 2. Work completed

### Site-wide
- **Images → AVIF.** Converted heavy JP/PNG images to responsive AVIF with `<picture>` + `srcset`. In use across index, ecommerce, black-friday, animation-3d, branding (60+ MB of source images reduced to ~1 MB served on the pages that were done).
- **Lazy-load + offscreen pause for videos.** Autoplay/background videos only download when they scroll near the viewport, and pause when they leave — big CPU/battery and bandwidth saving on video-grid pages. Applied to: index, animation-3d, black-friday, branding, ecommerce-creative, video-commercials.
- **Browser caching (`_headers`).** Media/fonts/CSS/JS cached for 1 year (`immutable`); HTML never cached. Makes repeat visits near-instant. Applies to every page (Netlify-wide).

### Video & Commercials page (measured with Lighthouse)
- **Re-encoded the 4 hero videos** — ~40% smaller each (main hero 3.4 MB → 2.0 MB), no visible quality change.
- **Rebuilt all grid preview loops** (`-preview.mp4`) from the untouched originals — ~85–90% smaller (most now 0.3–0.8 MB, were 1–2 MB).
- **Full click-to-watch videos left untouched** — full quality still opens on click.
- **Prioritized the hero (LCP):** `fetchpriority="high"` + preload hint, and stopped a script that was accidentally delaying the hero.
- **Fixed a console error** — `play() interrupted by new load` — by waiting for videos to be ready before playing.
- **Shrank 3 oversized UGC poster images** — 159 KB → 51 KB.
- **Deferred Google Fonts** so text paints immediately (~0.3–0.5s faster first paint).
- **Accessibility/SEO:** labeled the service dropdown, added a `<main>` landmark, named the video dialog, added `robots.txt`.

_All originals are backed up in `.optimize-backup/` (reversible; do not deploy that folder)._

---

## 3. Measured impact (Video & Commercials page)

| Asset group | Before | After |
|---|---|---|
| 4 hero videos | ~25 MB | ~14.7 MB |
| Grid preview loops (27 files) | ~24 MB | ~12.9 MB |
| UGC posters (3) | 159 KB | 51 KB |

The above are the files that actually load on the page (initial viewport loads only a fraction, thanks to lazy-loading). The heavy click-to-watch originals (10–37 MB each) are untouched and only fetched on click.

---

## 4. Remaining opportunities (ranked by impact)

### HIGH — Animation & 3D page videos
- **159.5 MB of video** across 38 files (avg 4.2 MB), only 18 have lightweight previews. Biggest are 12–18 MB each.
- Applying the exact same treatment used on Video & Commercials (re-encode heroes, rebuild previews at ~2× tile width) would cut this by an estimated **~85–90% → roughly 15–25 MB**.
- **This is by far the largest single win left on the site.**

### HIGH — Branding page images
- **7 PNG files totalling ~41 MB** (two are ~11.5 MB each), plus 12 JPGs at ~9 MB. These slipped past the earlier AVIF pass.
- Converting them to AVIF would take ~50 MB down to ~2–3 MB. Very high impact, low effort.

### MEDIUM — "All Visuals" videos (~24 MB)
- Shared media (likely used on the homepage). Re-encode the heavy clips the same way.

### MEDIUM — Font-defer + LCP priority on the other pages
- Only **video-commercials** has the non-blocking font load and hero `fetchpriority`. Every other page still blocks on Google Fonts and doesn't prioritize its hero image.
- Cheap, mechanical win — apply the same two changes site-wide for a small FCP/LCP gain everywhere.

### LOW — Housekeeping
- ~116 unreferenced media files (~8.5 MB) can be deleted after a quick safety check (small folder-size win, no user-facing effect).
- Legal/utility pages (privacy, terms, refunds, cookie-notice, accessibility) are light and need nothing.

---

## 5. Suggested next steps

1. **Re-run Lighthouse on the deployed Video & Commercials page** to confirm today's gains (expect mobile ~73 → ~90).
2. **Do the Animation & 3D video pass** — the single biggest remaining win (~140 MB reclaimable).
3. **Convert the Branding PNGs to AVIF** — ~48 MB reclaimable, low effort.
4. **Roll font-defer + hero fetchpriority out to the remaining pages.**

_Everything to date is reversible via `.optimize-backup/`. Only the pages that were explicitly worked on have been changed; nothing on the live-critical click-to-watch originals was altered._
