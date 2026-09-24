# AuraTide — Phase 1 MVP Build Notes

**Built:** 2026-09-23 · Single-file vanilla HTML/CSS/JS PWA · zero dependencies, works offline.

## Files
- `index.html` — the entire app (markup + CSS + JS)
- `manifest.json` — PWA manifest (name AuraTide, theme near-black, SVG icon)
- `sw.js` — cache-first service worker (`auratide-v1`), caches the app shell
- `icon.svg` — crescent-moon + tide-waves mark, electric-blue→violet on near-black

## What works (all client-side, localStorage-persisted)
1. **Onboarding ritual** — 3-step flow (name → birth date/time → birthplace), validates
   dates (1900 → today, not future), "don't know" birth time defaults to noon.
2. **Daily reading** — seeded RNG from `birthDate|YYYY-MM-DD`: stable all day, unique
   per user, fresh daily. Opener + Love/Focus/Luck paragraphs + one "harder truth"
   + 4 energy meters + Co–Star-style "day at a glance" one-liner. Weaves in the real
   sun sign, moon sign, and today's actual moon phase. Tomorrow preview is a
   locked freemium tease.
3. **Horoscope dice oracle** — animated Planet × Sign × House roll, gods-address-you
   voice, one free roll/day (extras → Premium upsell), last 7 rolls kept in history.
4. **Ritual loop** — daily check-in, consecutive-day streak counter, share via
   Web Share API with clipboard fallback (text card of the day's one-liner).
5. **Freemium gates** — unlimited rolls, tomorrow preview, compatibility, and the
   Premium screen all open the upsell sheet. **Placeholder only — no payment code.**
6. **PWA** — installable, offline-capable via service worker, apple-touch meta.

## Real math vs. template engine
| Piece | Status |
|---|---|
| Sun sign | **Real** — true solar ecliptic longitude → tropical sign (handles cusps astronomically) |
| Moon sign | **Real** — lunar longitude via truncated Meeus low-precision series (6 terms, ≈±0.3°; wrong only within ~0.3° of a sign cusp, <1% of births) |
| Moon phase (any date) | **Real** — synodic cycle from the 2000-01-06 new-moon epoch; phase name + illumination % |
| Reading/oracle prose | **Template engine** — curated Barnum banks, seeded per user/day. Feels personal; is not AI-generated. |
| Ascendant | **Omitted honestly** — needs lat/lon + sidereal time; birthplace is free text and the app is offline. Code comment marks the Phase-2 hook. Never faked. |

## Placeholders (Phase 2)
- **Payments:** Premium sheet is UI only. Phase 2 wires it to PayPal Subscriptions
  (mirror the PulseMatrix/BiteFact pattern: PayPal REST app webhook → server
  entitlement → client unlock).
- **OpenAI proxy:** Exact integration points are marked `PHASE-2 HOOK` in
  `index.html` §4 (copy engine) and §10 (premium). Architecture per the research
  page: client sends chart context (never an API key) → your server holds the key
  → model returns paragraphs/energies/one-liner → client renders. The local
  engine stays as the offline fallback.
- **Ascendant:** needs birthplace geocoding (server-side or bundled gazetteer).
- **No API keys anywhere in the client.** Verified by construction.

## Phase-2 hook list
1. `POST /api/reading` — OpenAI proxy; request = chart context, response = reading JSON.
2. `POST /api/oracle` — OpenAI proxy for dice-roll messages (or reuse /api/reading).
3. PayPal webhook → entitlement flag → unlock unlimited rolls / tomorrow / compatibility.
4. Geocode birthplace → ascendant computation → third chart pillar on the You tab.
5. Optional: push-notification habit loop (Nebula-style daily nudge).

## Verification
- JS extracted from `index.html` and syntax-checked with `node --check` — clean.
- `manifest.json` parsed as valid JSON.
- All localStorage / Share / Clipboard calls guarded with try/catch + fallbacks.
- No console errors by construction (no undefined refs; all `$()` ids exist in markup).
- Not deployed, no repo created, no DNS touched — prototype only.

## 2026-09-24 — Sean's UI feedback round
- Compacted UI: cards (18px→14px padding), dice (96×112→84×98), headings
  (34px→29px), buttons, chart cells, section spacing.
- Added footer: "POWERED by Toastid Tech, LLC" (bottom of main view).
- Added promo-code section to the Premium sheet. Boss code TST2026 unlocks
  premium client-side (unlimited oracle rolls + a genuinely computed
  tomorrow's preview). Compatibility stays a Phase-2 placeholder.
  NOTE: the code is visible in page source — fine for a boss code; move
  entitlement server-side with the PayPal webhook in Phase 2.
- Profile persistence verified working (localStorage auratide_profile;
  onboarding skipped on launch when present; 15/15 smoke-test assertions
  pass). If Sean is asked to re-enter, likely cause is opening a different
  URL/origin than the installed PWA (reinstalls wipe storage).
- Service worker cache bumped to auratide-v3 so installed copies update.
