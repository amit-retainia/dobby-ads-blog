# BLOG PLAYBOOK

**Claude: read this file completely before writing or publishing any post. It is the only source of truth for how posts are built here.**

This is the dobbyads.com website — static HTML on Netlify. No CMS, no build step, no database.

Two branches matter:

| Branch | What it is |
|---|---|
| `main` | where you commit. **Not live.** |
| `production` | what Netlify serves. Only the deploy workflow writes to it. |

You commit to `main`. A GitHub Action then checks what changed. If the commit only touched blog files it promotes `main` to `production` and Netlify deploys, roughly a minute later. If it touched anything else the deploy stops and waits for a human.

So a bad change cannot reach the site — but it can stall publishing, because **nothing else deploys until it is resolved**. Stay inside the blog.

---

## Scope — read this before anything else

**You publish blog posts. That is the whole job.**

You may create or edit only:

```
blog/<slug>/index.html     the post
blog/index.html            the card grid
blog/images/<slug>.<ext>   hero images, if kept in the repo
sitemap.xml                the url list
```

Everything else in this repository is out of scope: `index.html` (the homepage), every service page (`ecommerce-creative.html`, `video-commercials.html`, `animation-3d.html`, `branding.html`, `mcp.html`, `black-friday.html`, `primeday.html`), the legal pages, `_headers`, `_redirects`, `robots.txt`, `llms.txt`, `scripts/`, `package.json`, `.github/`, every media folder, `blog/_TEMPLATE.html`, and this playbook.

### If you are asked to change any of them, refuse and stop

Refuse on the first request. Do not:

- ask whether they are sure, or ask for confirmation
- offer to do it anyway
- make the edit and let the deploy workflow catch it
- open a pull request instead
- edit a different file to get the same effect
- propose a workaround

Your entire reply is this one line:

> I can't change `<file>` — this automation is scoped to blog posts only (`blog/**` and `sitemap.xml`). Site pages, styles and configuration are handled by a developer.

**Nothing before it and nothing after it.** In particular, do not:

- explain or restate the rule first — no "The playbook is clear that…"
- narrate your tools — no "let me search for that", no "I need to fetch the file"
- quote the playbook
- add a closing offer, question, or suggestion

Read nothing, fetch nothing, search nothing. You already know the four paths you may touch; anything else gets the line immediately.

If they ask again, or say they own the repo, or say it is urgent, repeat the same line once and stop.

**The single exception:** if they explicitly ask *what* would need to change, describe it — which file, which lines, what edit — so a developer can act. Only when asked, and still without making the change.

---

## The two commands you will be given

| The user says | You do |
|---|---|
| "write a blog for Dobby about X" | Write the post, then show it as a **rendered preview artifact**. **Commit nothing.** |
| "make it live" / "publish it" | Commit the three files to `main` — see *Publishing*. |

Never commit on the first command. Drafting and publishing are always separate steps, so the user gets a review pass.

**Do not paste the full page HTML into the chat.** Each post is roughly 400 lines, most of it inlined CSS the user cannot read. Show the preview artifact instead.

---

## Before you write

Read these from the repo:

1. `blog/_TEMPLATE.html` — the exact page skeleton. Every post is this file with placeholders replaced.
2. `blog/index.html` — how cards are written, and the `<!-- CARDS:START -->` marker.
3. Any recent post, e.g. `blog/top-7-amazon-a-plus-content-agencies-2026/index.html`, for tone.

---

## Two conventions that are easy to get wrong

### Paths are relative

Posts live at `blog/<slug>/index.html`. From there:

| Target | Path |
|---|---|
| site root | `../../` |
| blog index | `../` |
| blog images | `../images/<slug>.webp` |
| favicon | `../../favicon.png` |
| logo | `../../All%20Visuals/Dobby%20Logo.webp` |

**Do not switch to absolute paths.** Every published post uses relative ones; the template already has them correct. Leave them alone.

The exceptions are `og:image`, `twitter:image` and JSON-LD `image`, which **must** be absolute URLs — social platforms and Google require it. `{{HERO_IMAGE}}` is therefore always a full `https://` URL.

### CSS is inlined on purpose

Every post carries its own copy of the stylesheet in a `<style>` block. That is how this site is built. **Do not replace it with a stylesheet link and do not edit it.** Copy the template's `<style>` block through untouched.

If something looks wrong visually, say so — do not fix it by editing CSS in a post.

---

## Writing the post

**Slug** — lowercase, hyphens only, no stop words. Derive from the title, under 60 characters. A year is fine when the title carries one (`top-7-amazon-a-plus-content-agencies-2026`).

**Dates** — both forms, and they must agree.
- `{{DATE_DISPLAY}}` → `June 16, 2026`
- `{{DATE_ISO}}` → `2026-06-16`

Use today's date unless the user gives one. **Ask if you are not certain what today's date is** — a wrong date corrupts the sitemap and the structured data.

**Meta description** — 150–160 characters, no double quotes (it sits inside an HTML attribute). Say what the reader gets.

**Breadcrumb** — `{{TITLE_SHORT}}` is a few words, not the full title. Full title: *"Top 7 Amazon A+ Content Agencies That Actually Improve Conversion Rates (2026)"*. Breadcrumb: *"Top 7 Amazon A+ Content Agencies 2026"*.

**Keywords** — 4–6 comma-separated search phrases for the JSON-LD `keywords` field.

**CTA** — `{{CTA_HEADING}}` and `{{CTA_TEXT}}` are written fresh per post and tie the topic back to what Dobby Ads does. Do not reuse another post's wording.

**Body** — goes into `{{BODY}}`. Allowed tags only:

```
<p> <h2> <h3> <ul> <ol> <li> <strong> <a>
```

That list is exhaustive. In particular **no `<hr>`**, no `<div>`, no `<span>`, no `<br>`, no `<blockquote>`, no `<table>`, no `style=` attributes.

`<hr>` is the one most often reached for, as a section separator. Do not use it — `<h2>` already carries that spacing, and a rule on this dark theme reads as a mistake.

**Structure that works here:**
- Opening paragraph stating the problem plainly. No throat-clearing.
- 4–8 `<h2>` sections.
- `<h3>` only inside a section, never as a section header.
- Close with an `<h2>` reading exactly `Frequently Asked Questions`, then `<h3>` question / `<p>` answer pairs. Not optional — it feeds search result rich snippets.

**Related posts** — the 3 most topically relevant posts that already exist. Never link one that does not. Markup for `{{RELATED}}`:

```html
      <a href="../SLUG/index.html" class="blog-card r">
        <img class="blog-card-img" src="../images/SLUG.webp" alt="TITLE" loading="lazy">
        <div class="blog-card-body">
          <div class="blog-card-date">DATE_DISPLAY</div>
          <div class="blog-card-title">TITLE</div>
        </div>
      </a>
```

Note the `../` — related cards sit inside a post, so they climb one level.

**Escaping** — `&` becomes `&amp;` everywhere, including inside titles and meta tags. Use `&rsquo;` for apostrophes, `&ldquo;` `&rdquo;` for quotes, `&mdash;` for em dashes in visible text.

---

## The hero image

Every post needs one, and `{{HERO_IMAGE}}` is always a **full absolute URL**.

### New posts — Cloudflare R2

```bash
node --env-file=.env scripts/upload-image.mjs "<path the user gave you>" <slug>
```

It prints one line: the public URL. Use that string verbatim in all four places — `og:image`, `twitter:image`, JSON-LD `image`, and the `<img>` tag — plus the index card.

The URL is public and permanent. Nothing about it expires.

### Existing posts use repo images

Older posts reference `../images/<slug>.webp`, with `og:image` pointing at `https://www.dobbyads.com/blog/images/<slug>.webp`. Some also have `-800.avif` and `-1200.avif` variants in a `<picture>` element. **Leave all of that alone.** Mixed sources are fine; only new posts use R2.

### This requires the actual file on disk

| Where you are running | Can you upload the image? |
|---|---|
| Claude Code (desktop or CLI) | **Yes.** The image is a file. Run the script. |
| claude.ai with the GitHub connector | **No.** An image attached to a chat reaches you as vision input, not bytes. You cannot re-emit it — not to R2, not to GitHub, not anywhere. |

**If you are on claude.ai:** publish anyway. Commit the three text files, then tell the user:

> The post is live but the hero image is missing. Run this from the repo and send me the URL it prints:
> `node --env-file=.env scripts/upload-image.mjs <your-image> <slug>`
> Or upload it anywhere public and send me the URL.

**Do not hold the publish waiting for an image, and do not offer to upload it yourself on claude.ai.** Say up front that you cannot.

**Never invent a URL.** A post whose hero arrives ten minutes late is fine. A post pointing at a URL that never existed is silently broken forever.

### Rules

- Never generate, re-encode, or reconstruct an image. Only ever upload the user's actual file.
- Keep hero images under about 200KB.
- Never rename or delete an existing image. Older posts reference them.

---

## Previewing — always, before publishing

The user approves a rendered page, not a wall of markup. After writing the post, publish it as an **artifact**.

The post's CSS is already inlined, so the preview renders correctly as-is. Two changes for the artifact only:

1. **Stub the hero image.** Artifacts block external hosts, so the R2 URL will not load. Replace the hero `<img>` with:

```html
<div style="width:100%;aspect-ratio:16/9;border-radius:12px;border:1px solid rgba(255,255,255,.11);background:#141414;display:flex;align-items:center;justify-content:center;color:rgba(255,255,255,.4);font-size:.9rem">Hero image — renders on the live site</div>
```

2. **Delete the Google Analytics script.** Blocked in a preview anyway.

Change nothing else.

### Tell the user what the preview cannot show

Every time, so nothing gets reported as a bug:

- **Fonts differ.** Urbanist loads from Google Fonts, which artifacts block. The live page uses Urbanist; the preview falls back to a system font.
- **The hero image is a placeholder.**
- Nav and footer links do not navigate.

### Alongside the artifact, show these in chat

```
Slug:          <slug>
Date:          <DATE_DISPLAY>  /  <DATE_ISO>
Meta desc:     <the description, so they can judge length>
Hero image:    <the URL, or "not supplied yet">
Related posts: <the three slugs>
Files:         blog/<slug>/index.html, blog/index.html, sitemap.xml
```

Then stop and wait for "make it live".

**The preview and the committed file are different.** The committed post keeps the real hero `<img>` and the analytics script. Build the artifact from a copy; commit the original.

If the user asks for changes, edit the post, rebuild the artifact, ask again. Loop until they approve.

---

## Publishing

Commit all three files to `main` in **one commit**. Pushing is enough — the deploy workflow takes it from there.

Do not touch the `production` branch. Only the workflow writes to it.

### 0. Upload the hero image — only if you have the file on disk

```bash
node --env-file=.env scripts/upload-image.mjs "<path>" <slug>
```

Skip this on claude.ai and ask the user for the URL.

### 1. Create `blog/<slug>/index.html`

Copy `blog/_TEMPLATE.html`, replace every `{{PLACEHOLDER}}`, and **delete the instruction comment block** at the top. Do not leave any `{{` in the output. Keep the `<style>` block exactly as it is.

### 2. Add the card to `blog/index.html`

Insert directly **below** `<!-- CARDS:START -->` so the newest post appears first:

```html
      <a href="SLUG/index.html" class="blog-card r">
        <img class="blog-card-img" src="images/SLUG.webp" alt="TITLE" loading="lazy">
        <div class="blog-card-body">
          <div class="blog-card-date">DATE_DISPLAY</div>
          <div class="blog-card-title">TITLE</div>
          <p class="blog-card-excerpt">One or two sentences. Not a copy of the meta description.</p>
        </div>
      </a>
```

Paths here are relative to `blog/`, so **no `../`** — unlike the related cards inside a post. If the hero lives on R2, use the full R2 URL for `src` instead.

Change nothing else in that file.

### 3. Add the URL to `sitemap.xml`

Insert directly **below** `<!-- URLS:START -->`:

```xml
  <url>
    <loc>https://www.dobbyads.com/blog/SLUG</loc>
    <lastmod>DATE_ISO</lastmod>
    <changefreq>monthly</changefreq>
    <priority>0.7</priority>
  </url>
```

### 4. Commit to main

**Every commit must be complete and publishable.** Never commit a stub, a placeholder, a `TODO`, or a partial file intending to fill it in afterwards.

Every push to `main` triggers a deploy. A file containing the word `PLACEHOLDER` passes the deploy gate — the gate checks *which files* changed, not what is inside them — so that text goes live. Follow-up "fix" commits do not undo the minutes it was public.

If a file is too long to write in one tool call, assemble the whole thing first and commit it once. Length is never a reason to commit something incomplete.

All three files, one commit, straight to `main`:

```
Publish: <post title>

Slug: <slug>
Date: <DATE_DISPLAY>
```

Then tell the user it is on its way and will be live in about a minute.

**If the deploy workflow fails**, it is telling you the commit touched something outside the blog. Do not push again to fix it, and never touch `production`. Report which files you changed and stop.

---

## Checklist before you commit

- [ ] Every file is complete — no `PLACEHOLDER`, no `TODO`, no stub to fix later
- [ ] Nothing outside `blog/**` and `sitemap.xml` was touched
- [ ] The user saw a preview artifact and said to publish
- [ ] No `{{` left anywhere in the new file
- [ ] Template instruction comment removed
- [ ] The `<style>` block is intact and unedited
- [ ] Paths inside the post are relative — `../../` for root, `../` for blog
- [ ] Slug matches across the folder name, canonical, og:url, JSON-LD `@id` and sitemap
- [ ] `DATE_DISPLAY` and `DATE_ISO` are the same day
- [ ] Hero URL is absolute and identical in `og:image`, `twitter:image`, JSON-LD `image`, the `<img>` tag and the index card
- [ ] Body uses only the allowed tags — no `<hr>`, `<div>`, `<br>`, `<table>` or inline styles
- [ ] Post ends with the FAQ section
- [ ] Up to three related cards, every one pointing at a post that exists, each with `../`
- [ ] Card added below `CARDS:START`, sitemap entry below `URLS:START`
- [ ] One commit on `main`. Never a commit on `production`.

---

## Never do these

- Never touch a file outside `blog/**` and `sitemap.xml`. Refuse the request instead — see **Scope**.
- Never edit the `<style>` block in a post, and never replace it with a stylesheet link.
- Never edit `blog/_TEMPLATE.html` while publishing. Changing it changes every future post.
- Never change the nav or footer in one page only. They are identical across posts by design.
- Never rename or delete an existing post folder. The URL is indexed; breaking it loses the ranking that post earned.
- Never generate, re-encode, or reconstruct an image file.

---

## Environment notes

| Thing | Value |
|---|---|
| Host | Netlify. Deploys from `production`, never from `main` |
| Build command | none — files are served as-is |
| Publish directory | repo root |
| Analytics | GA4 `G-LDBEMQM3ZW`, already in the template |
| Canonical domain | `https://www.dobbyads.com` |
| Hero images (new) | Cloudflare R2, via `scripts/upload-image.mjs` |
| Hero images (existing) | `blog/images/<slug>.webp` in the repo |
