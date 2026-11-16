# ICF Portugal — Wix to React migration

Design, build and launch of [www.icf.pt](https://www.icf.pt), the website of the
Portugal Chapter of the International Coaching Federation — replacing a Wix site
that the organisation could no longer maintain.

Shipped November 2026. 87 commits over four months.

**Live site: [www.icf.pt](https://www.icf.pt)**

<!-- Add 2–3 screenshots here: homepage, the event page, and the /admin editor.
     A screenshot of the CMS is worth more than one of the site — it shows the
     part a visitor never sees. -->

---

## The problem

The chapter had a Wix site, a `@icf.pt` email service running on Microsoft 365,
and nobody on the team who writes code. Three constraints followed from that:

1. **The email could not break.** The domain's DNS carried the MX, SPF, DKIM and
   DMARC records the organisation's email depends on. A careless cutover would
   have taken down the mail of a national professional body.
2. **Non-technical staff had to be able to edit the site** after I was gone.
   A site only I could update would have been a worse outcome than Wix.
3. **The existing search presence had to survive.** Years of indexed Wix URLs
   pointed at pages that would no longer exist.

## What I built

A static-rendered React site with a Git-backed CMS, deployed on Vercel.

- **15 pages**, 17 shared components, ~5,400 lines of TypeScript.
- **A content layer in JSON**, edited through [Decap CMS](https://decapcms.org/)
  at `/admin`. Editors log in with GitHub; every save is a commit. No database,
  no separate backup story, and the full history of every content change.
- **A dynamic event page type** — programme grouped by day, speakers, sponsor
  tiers, pricing — so a new conference is a content task, not a code task.
- **22 redirects** mapping the old Wix URLs onto the new structure, so existing
  search results and external links keep working.

## Technical decisions worth explaining

**Prerendering without a browser.** The site needed real HTML for search engines
and link previews, but pulling in Puppeteer to render 14 pages meant a ~200 MB
dependency and a slow, fragile build. I wrote a build step that parses the `<Seo>`
component's literal props straight out of the page source and injects the correct
title, description, canonical URL, Open Graph tags and JSON-LD into each generated
page. No browser, no headless Chrome in CI, and a build that finishes in seconds.
The sitemap falls out of the same pass.

**Git as the content database.** Decap CMS commits content changes to the
repository. That sounds like a constraint and is actually the feature: content
review is a diff, rollback is `git revert`, and the backup is the repository
itself. The CMS bundle is self-hosted rather than loaded from a CDN — a 4.8 MB
script from a third-party origin is both a performance problem and something ad
blockers were silently breaking.

**The DNS cutover.** The migration touched exactly two records — the A record of
the apex and the CNAME of `www` — and nothing else. Before changing anything I
lowered the TTL, then tested the new origin with `curl --resolve` to confirm it
answered correctly for the real hostname while DNS still pointed at Wix. Email
never went down.

**Accessibility as a build concern, not a checklist.** Reduced-motion variants on
every animation, focus-visible outlines on interactive elements, a real `<dialog>`
pattern with focus trapping and restoration for the announcement modal, and
minimum 24×24 px touch targets.

**A privacy policy that matches the code.** The site stores one flag in
`localStorage` to remember a dismissed announcement. That is declared in the
privacy policy, in plain language, because an undeclared storage write is still a
storage write.

## Stack

| | |
|---|---|
| Framework | React 19, TypeScript 7, Vite 8 |
| Styling | Tailwind CSS v4 |
| Motion | Framer Motion 12 |
| Routing | React Router 7 |
| CMS | Decap CMS 3.14.1, GitHub backend, self-hosted |
| Hosting | Vercel |

## Things I got wrong, and fixed

- **Soft 404s.** Every unknown URL returned HTTP 200 with the homepage's title,
  because the SPA fallback served `index.html` for everything. Humans saw the
  correct 404 page; Google saw a valid page. Found it by inspecting the response
  headers rather than the rendered page — the browser gives you no reason to look.
- **A logo with a baked-in black background** shipped to production because I
  checked that the file loaded, not what it contained. Caught by inspecting the
  actual pixels.
- **An unverified claim about a vendor's free tier**, presented confidently from
  documentation, that turned out to require a credit card. The lesson stuck: test
  the thing, don't quote the docs about the thing.

## A note on the source

The application code lives in the organisation's own repository and is not mine
to republish. This repository is a write-up of the work. I'm happy to walk
through the code in an interview.

---

**Francisco Figueiredo** · [www.icf.pt](https://www.icf.pt)
