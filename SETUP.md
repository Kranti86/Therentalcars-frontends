# Therentalcars — Online Front-End: Setup & Roadmap

This is the **online self-serve funnel** (separate from your phone funnel). It's a
single static file, `online.html`, that talks to your FastAPI backend. Plus a sample
SEO landing page (`sample_miami_page.html`) — your real city pages are produced by the
backend's `seo/generate.py`.

---

## 1. What YOU need to populate

### A) In `online.html` — the `CONFIG` block near the top of the `<script>`

| Field | What to put | Where to get it |
|---|---|---|
| `API_BASE` | Your backend URL, e.g. `https://carfinder-xyz.herokuapp.com` | Your Heroku app address. **Setting this switches the page from demo to live.** |
| `GA4_ID` | A **separate** GA4 measurement id (`G-XXXXXXX`) for the ONLINE funnel | analytics.google.com → create a NEW property (do NOT reuse the phone funnel's) |
| `ADS_CONVERSION` | A **separate** Google Ads conversion `send_to` (`AW-XXXX/xxxxx`) | Google Ads → Conversions → new "Reserve" action → get the send_to |
| `REQUEST_ENDPOINT` | Leave as `/requests` | Wired in Phase 2 (reservation backend) |
| `INSURANCE` | Demo-mode display only | Live insurance price comes from the backend (`INSURANCE_*` config vars) |

> Note: **every amount is decided by the BACKEND.** Markup tiers live in `app/pricing.py`
> and insurance in the `INSURANCE_*` config vars. At checkout the page only sends the
> search `job_id`, the `vehicle_id` and whether insurance was chosen; `/quote`,
> `/paypal/create-order` and `/requests` work out the amounts. `feePct` in the frontend
> is only used by demo mode (`?demo=1`).

### B) In the BACKEND `.env` / Heroku Config Vars

`MAPBOX_TOKEN`, `RAPIDAPI_KEY`, `RAPIDAPI_HOST`, `RAPIDAPI_BASE_URL`, `REDIS_URL`
(from a Redis add-on), `DEFAULT_COUNTRY_OF_RESIDENCE=us`, `CANARY_TOKEN`,
and **`CORS_ALLOW_ORIGINS`** = your front-end domain (e.g.
`https://book.thecarrentals.globetradee.pro`). ← important, or the browser blocks it.

### C) SEO generator (`seo/` in the backend zip)

Edit `cities.json` (your target cities), then:
```
python generate.py --api-base https://YOUR-BACKEND --site https://YOUR-FRONTEND
```
Push `seo/out/` to Fastly. Schedule it nightly.

### ⚠️ Security
Regenerate your **RapidAPI key** — it was exposed in shared screenshots. Never put
keys in `online.html`; only `API_BASE` (a public URL) goes there.

---

## 2. Where to host

You have three separate things; they host differently:

| Piece | Best host | Why |
|---|---|---|
| **Online app** (`online.html`) | **Vercel / Cloudflare Pages / Netlify** (or your Fastly) | It's a static file. These give free global CDN, HTTPS, and easy custom domains. |
| **SEO city pages** (generated) | **Fastly** (you already have it) | Edge-served, fast, where your programmatic pages already live. |
| **Backend** (FastAPI) | **Heroku** (keep it) — or Render/Railway | It's a Python server; needs a real runtime, not static hosting. |

**Do NOT** host `online.html` on Heroku (Heroku is for the Python backend).
**Namecheap shared hosting** works for the static files too, but it's slower and has
no CDN — fine to start, but Vercel/Cloudflare Pages (both free) are better and just as
easy.

**Recommended layout:**
- `book.thecarrentals.globetradee.pro` → the online app (Vercel/Cloudflare Pages)
- `thecarrentals.globetradee.pro/car-rental/...` → SEO pages (Fastly)
- backend on Heroku, with `CORS_ALLOW_ORIGINS` set to the `book.` subdomain

Keeping the online app on its **own subdomain** is deliberate — it keeps the online
audience/analytics separate from the phone funnel.

---

## 3. Roadmap — what's done and what's left

### ✅ Done
- **Backend**: live search (Booking/RapidAPI + Mapbox), ranking (cheapest→closest,
  best-in-type), server-side price markup, full-address hidden until reserve, insurance
  math (placeholder), `search_key` expiry recovery, metrics, canary, `/stats` for SEO.
- **Online app** (Aurora Glass): AI search → staged loader → best-match results →
  **refine filters** (type / passengers / budget / distance / supplier) → insurance
  step → checkout with price breakdown → lead capture. Tracking events throughout.
- **SEO engine**: page generator + template + `cities.json` + sitemap; `/stats`
  Redis-cached and refreshed free by real searches (doesn't burn booking credits);
  clickable pickup/supplier chips deep-link into the app pre-filtered.

### 🔜 Phase 2 — Reservation backend (needs your inputs)
- Postgres storage + `POST /requests` (persist-first).
- **Payment**: authorize (don't collect) → **auto-capture after 3 days** → manual
  refund/void → **insurance clawback** (you're merchant of record). Needs your
  **payment processor** (auth/capture + split payouts).
- **Insurance**: wire **rentalcover.com** API for the real CDW net rate + split payout.
- **Agent dashboard**: deal-funnel portal (name, email, phone, travel dates, policy
  number, amount collected, editable supplier reservation number, capture/refund/cancel).
- **Slack** notification + **email** confirmation. Needs: Slack webhook URL, email
  method (SendGrid/Postmark/SMTP or your server).

### 🏁 Final / streamline
- Post-reservation full-detail email (car, dates, exact address, certificate, breakdown).
- **Reviews**: BCC review invites to **Trustpilot + TripAdvisor**.
- **Legal pages**: Privacy, About Us, Terms & Conditions, **Cancellation** T&C.
- **Tracking upgrades**: Enhanced Conversions (hash email/phone) + Offline Conversion
  Import (send confirmed commission + gclid back to Google) → lowers CPC.

### 📣 Your workstream (not code)
- **Off-page SEO / backlinks** (high-DA authority) — the marketing side of ranking.

---

## 4. Files in this zip
- `online.html` — the online funnel app (set `CONFIG.API_BASE` and host it)
- `sample_miami_page.html` — a sample SEO landing page (reference; real pages come from `seo/generate.py`)
- `SETUP.md` — this file
