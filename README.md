# AGM Real Estate Group — West Haven Homeowners' Association Proposal Micro-Site

**CONFIDENTIAL — proposal material. Private repository. Do not make public.**

A digital micro-site version of AGM's *Proposal for Management Services* for the **West Haven
Homeowners' Association** (Bellevue, WA). Built from AGM's shared HOA / community-association
proposal template — the design system, structure, and code are unchanged; the property name, the
Governance section, and the fee figures are tailored to West Haven. Each proposal slide is its own
page in a single-file static site (`index.html`, no build step, no dependencies). Fonts load from
Google Fonts; everything else is inline. This repo shares the design system used across AGM's
proposal micro-sites.

Because West Haven is administratively dissolved and has no active Board, the Governance section
leads with the path to reinstatement — restoring corporate standing, calling a special meeting to
elect a Board under RCW 64.34 / 64.90, and stabilizing operations (including the neglected common
area along W Lake Sammamish Road) — before flowing into ongoing board support.

## What this is
The deck's sections, rebuilt as an institutional, navigable micro-site (nine tabs — Fees is split
into A and B):

1. About AGM · 2. Management Team · 3. Governance & Board Support · 4. Financial Management &
Fiduciary Oversight · 5. Interim Compliance Review · 6. Facilities, Maintenance & Capital Projects ·
7. Tools & Technology · 8. Fees A (Fee Structure) · 9. Fees B (fee summary).

Slide copy is reproduced from the source HOA deck. The layout, palette (navy `#00202F`, brand
blue `#3A8DDE`, serif/sans pairing), and rail-and-content structure follow the AGM proposal design
system shared with AGM's other proposal micro-sites. Each page adds whitespace and interaction — a
per-page navy/blue summary rail, an interactive history timeline, hover-reactive cards and pills,
reveal-on-scroll, prev/next paging, a reading-progress bar, and a light/dark toggle.

## Local preview
Open `index.html` in a browser. That's it. Deep-link a section with the URL hash, e.g.
`index.html#compliance`.

## Deploy — Cloudflare Pages (AGM standard pattern)
1. Cloudflare dashboard → **Workers & Pages → Create → Pages → Connect to Git** → select this repo.
2. Settings: Framework preset **None** · Build command **(empty)** · Build output directory **/**
3. Every push to `main` auto-deploys production; every branch/PR gets its own preview URL.

## Access gate — custom password screen (Pages Functions)
The site is protected by a **custom-designed cover / login screen** (a minimal, institutional
white split layout — explanatory copy on the left, password panel on the right, AGM logo top-right),
served by a Cloudflare Pages Function (`functions/_middleware.js`). This runs **server-side**: until
the correct password is submitted, the visitor only ever receives the cover page — the actual
proposal (`index.html`) is never sent to the browser. The password lives only as an encrypted
Cloudflare secret, never in the code or the client.

This replaces the standard Zero Trust login screen with AGM's own branded page.

### One-time setup (required before the site will unlock)
Cloudflare dashboard → **Workers & Pages → this project → Settings → Variables and Secrets**. Add
both of these for **Production _and_ Preview**, then redeploy:

| Name | Value | Mark as |
|------|-------|---------|
| `SITE_PASSWORD` | the shared password you give recipients | **Secret** |
| `GATE_SECRET` | any long random string (40+ chars) — used to sign the session cookie | **Secret** |

- Until `SITE_PASSWORD` is set, the site fails closed (shows a "not configured yet" notice).
- **Change the password** anytime by editing `SITE_PASSWORD` (existing links keep working; open
  sessions stay valid because `GATE_SECRET` is unchanged).
- **Force everyone to re-enter** by rotating `GATE_SECRET` (or bumping `TOKEN_VERSION` in the
  middleware).
- Sessions last 7 days (`MAX_AGE`); `/__logout` clears the cookie.
- The cover screen's copy (title, description, "Inside this proposal" list, contact line) lives in the
  `coverHTML()` function at the bottom of `functions/_middleware.js`.

### Local preview
Copy `.dev.vars.example` → `.dev.vars` (git-ignored), fill in the two values, and run
`npx wrangler pages dev .`.

### Note on Zero Trust
This shared-password gate is intentionally simple and needs no per-user setup. If you ever need
**per-person access with an audit trail** (who opened it, when), use Cloudflare Zero Trust Access
instead — but that uses Cloudflare's own login flow, not this custom screen. Don't enable both at once.

## Analytics (PostHog)
The site is fully instrumented for PostHog across **both** the login/cover page and the proposal.
Turn it on by setting these as **environment variables** in the Cloudflare Pages project (Settings →
Variables and Secrets), for **Production _and_ Preview**, then redeploy:

| Name | Value |
|------|-------|
| `POSTHOG_KEY` | your **Project API Key** (PostHog → Settings → Project → *Project API Key*, starts with `phc_`) |
| `POSTHOG_HOST` | *(optional)* `https://us.i.posthog.com` (US, default) or `https://eu.i.posthog.com` (EU) |

Until `POSTHOG_KEY` is set, analytics stays off — no requests, no errors. The key lives only in
Cloudflare (nothing committed to the repo): the gate function injects it into the cover page and fills
in the proposal's inline placeholder as `index.html` is served. (You can still hard-code the key
directly in `index.html`'s marked `<head>` block instead, but that only covers the proposal, not the
gate page, and puts the key in the repo — the env var is preferred.)

What it tracks once the key is set:
- **Cover / login page** — a `$pageview` on load and a `gate_viewed` event (tagged `surface: gate`),
  plus autocapture of the Access click. Lets you see who reaches the gate and whether they bounce.
  The password field is masked in session replays.
- **Proposal — visits** — a virtual `$pageview` per section (URL carries the `#section` hash).
- **Proposal — tab navigation** — a `tab_click` event with `to`, `from`, and `method` (`nav_tab`,
  `pager`, `rail_ticker`, `keyboard`, `brand`).
- **Proposal — time on each tab** — a `section_time` event with `section`, `section_label`, and
  `seconds` when a section is left (open section flushed on tab-hide / exit via `capture_pageleave`).
- **Everything, both pages** — `autocapture` (every click/interaction), **session replays**, and
  click/scroll **heatmaps** are enabled.

All events are tagged with `proposal: hoa-microsite` (and `surface: gate` on the cover
page) so you can filter gate traffic from in-proposal activity.

## Operational notes
- `_headers` enforces `noindex` and security headers at the edge.
- Featured-asset tiles are placeholders; drop in property photography by swapping the `.asset-tile`
  elements for `<img>` tags when imagery is available.
- Two Fees pages per request: **Fees A** (Fee Structure — retainer band + "Included in the base fee"
  grid + onboarding) and **Fees B** (condensed fee summary). Fee figures on both are set for West
  Haven: **$800 / month** management retainer and a **$2,500–$3,000** one-time onboarding range
  (finalized once the scope of past-due work is confirmed).
- The `topbar-prop` label in `index.html` and the `prop-name` on the cover page are set to
  **West Haven Homeowners' Association**.
- Transition & Onboarding slides (8 and 9) from the master deck are intentionally excluded.
