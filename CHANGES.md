# Frontend fixes (Nov 2026) — deploy right after the matching backend

- **Checkout uses server prices.** The page asks `/quote` for the breakdown and sends only
  `job_id`, `vehicle_id` and the insurance choice to `/paypal/create-order` and `/requests`.
- **No fake reservations.** If saving fails, the customer sees an honest error ("nothing was
  charged") and can retry (same PayPal order, no duplicate). Expired prices ask to search again.
- **Failed card hold** shows "Reservation saved — payment not completed", not "confirmed".
- **Ad conversions** fire only for a saved reservation (and, for paid checkouts, a successful
  hold), valued at the server-computed margin.
- **Escaping**: all API data and the `?city=` parameter are escaped; only https images load.
- Amounts show cents when needed, so the button matches the card hold exactly.
- Wording: insurance is presented as optional; no invented "cars scanned" counter, supplier
  ticks or "30+ suppliers / 400+ cars" claims. Popular-city prices come from cached real data
  (`/stats?cache_only=true`) or say "Live prices".
- SEO: canonical points to this site; sitemap uses clean URLs, no duplicate or sample page;
  robots.txt tidied; llms/ai/knowledge links use `/online`.
- Sample city page: placeholder GTM/Clarity IDs replaced (GA4 `G-HJ6XYK06QT`, Clarity
  `xt2ezrz5yd`), consent-mode default + cookie banner added.
- Removed the stray macOS `download` file; SETUP.md file names fixed.
