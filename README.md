# Koelnmesse Inc. — Chicago Office TV Dashboard

**Version 2.0** · Maintained by the Chicago Office · Owner: Darrin Stern (Managing Director)

A single-page, live office dashboard designed for continuous display on the 75" TV in the Chicago office (Amazon Fire TV Stick 4K Max). Combines team calendar, upcoming Koelnmesse shows, live weather + FX, industry + press headlines, top news across four regions, quote/word of the day, a live Sky News video feed, and a rolling news ticker — all on one 1920×1080 screen designed to be read from across the room.

**Live URL:** [`https://darrinstern-km.github.io/koelnmesse-dashboard/dashboard.html`](https://darrinstern-km.github.io/koelnmesse-dashboard/dashboard.html)

---

## What it shows

The dashboard is designed to answer four questions instantly for anyone walking past the TV:

1. **Who's around today?** → Team Calendar block (birthdays, work anniversaries, office closed, out of office)
2. **What Koelnmesse shows are coming up?** → Upcoming Shows column + rotating Featured Show spotlight
3. **What's happening in the world / Germany / Chicago / sports?** → 4 top-news cards + Sky News PIP
4. **What's happening in our industry and at Koelnmesse HQ?** → Trade Show News + Koelnmesse News + live ticker

Plus ambient context: local time in Chicago and Köln, current weather in both cities, live USD/EUR ↔ EUR/USD FX, Quote of the Day, Wort des Tages (German Word of the Day).

### Panel map (1920×1080)

```
┌────────────────────────────────────────────────────────────────────────┐
│  [Koelnmesse logo] Chicago Office · Live Dashboard    Chicago  Köln    │  header
├────────────┬──────────────────┬────────────────────┬───────────────────┤
│ Weather    │ Trade Show News  │ Upcoming Shows     │ Team Calendar     │
│ (Chi, Köln)│ (from JSON)      │ (5 shows w/ logos) │ • Birthdays       │
│            ├──────────────────┤                    │ • Anniversaries   │
│ Markets    │ Koelnmesse News  │                    │ • Office Closed   │
│ (USD/EUR)  │ (from JSON)      │                    │ • Out of Office   │
│ ┌────────┐ │                  │                    │                   │
│ │Feature ├─┘                  │                    │                   │
│ │Show    │  (spotlight card rotates every 45s)     │                   │
│ │Spotlight│                                        │                   │
│ └────────┘                                         │                   │
├────────────┬──────────────────┬────────────────────┼───────────────────┤
│ Quote      │ Word of the Day  │ [World][Germany]   │  ┌──────────────┐ │
│ of the Day │ (Wort des Tages) │ [Chicago][Sports]  │  │ Sky News PIP │ │
│            │                  │ (4 top-news cards) │  │ + CC         │ │
├────────────┴──────────────────┴────────────────────┴──┴──────────────┴─┤
│ ● LIVE  [ticker: industry + Koelnmesse press headlines, dated]        │  ticker
└────────────────────────────────────────────────────────────────────────┘
                                              v2.0 · synced [timestamp]
```

---

## Architecture

**Static HTML on GitHub Pages, dynamic data fetched from a small JSON file, updated daily by a scheduled task.**

```
   ┌─────────────────────┐         ┌──────────────────────┐
   │  Outlook shared cal │         │  Web sources         │
   │  (OOO / travel)     │         │  (TSNN, Koelnmesse   │
   │                     │         │   Press, DW, etc.)   │
   └──────────┬──────────┘         └──────────┬───────────┘
              │                               │
              └──────────────┬────────────────┘
                             ▼
              ┌─────────────────────────────────┐
              │  Daily 6am Chicago sync task    │  Claude-run,
              │  (Zapier ↔ GitHub write action) │  6:07 AM CT
              └──────────────┬──────────────────┘
                             │  commits dashboard-data.json
                             ▼
              ┌─────────────────────────────────┐
              │  darrinstern-KM/                │
              │  koelnmesse-dashboard (main)    │
              └──────────────┬──────────────────┘
                             │  ~30s auto-publish
                             ▼
              ┌─────────────────────────────────┐
              │  GitHub Pages (public)          │
              │  darrinstern-km.github.io/…     │
              └──────────────┬──────────────────┘
                             │  fetch on load + every 30 min
                             ▼
              ┌─────────────────────────────────┐
              │  Fire Stick 4K Max              │
              │  75" TV, Chicago office         │
              └─────────────────────────────────┘
```

**Reload cadence:**
- Dynamic data (OOO + news) refetches every 30 minutes without a page reload
- Full page reload at 24 hours from last load
- Additional page reload at 3am Chicago (defense-in-depth against timer drift)
- 6am Chicago daily sync pushes fresh JSON

---

## File map

Only two files matter. Everything else is documentation.

| File | Purpose | Who updates it |
|---|---|---|
| `dashboard.html` | The entire app — HTML, CSS, JS in one file. ~92KB, ~2,300 lines. | Manually, when features change (versioned) |
| `dashboard-data.json` | ~3KB — OOO + top news + industry news + Koelnmesse press | Auto-updated daily at 6am CT by the scheduled sync |
| `README.md` | This file | Manually |

**Local working copies** live on the Managing Director's Mac at `/Users/darrinstern/Documents/Claude/Projects/Managing Director/`. The scheduled task writes local copies as backups after each successful GitHub push.

---

## Data sources

| Panel | Source | Refresh cadence | Auth |
|---|---|---|---|
| Weather (Chicago + Köln) | [Open-Meteo](https://open-meteo.com/) | Every 15 min in-browser | None (free) |
| USD ↔ EUR | Frankfurter (ECB) + open.er-api.com fallback | Hourly + 8am CT anchor | None (free) |
| Out of Office | Outlook shared calendar `SharedCal@koelnmesse.us`, category `Out-Vac-Travel-Other` | Daily 6am CT sync | Microsoft 365 (via scheduled task) |
| Top News (World / Germany / Chicago / Sports) | Web search — Germany constrained to German-domiciled outlets (DW, The Local, Tagesschau, Der Spiegel, ZDF, Süddeutsche Zeitung, FAZ, Kölner Stadt-Anzeiger) | Daily 6am CT sync | None |
| Trade Show industry news | TSNN, Trade Show Executive, BizBash, Exhibit City News | Daily 6am CT sync | None |
| Koelnmesse press | koelnmesse.com press office + Google News for "Koelnmesse" + partner trade sites | Daily 6am CT sync | None |
| Upcoming Koelnmesse Shows | Hardcoded in `dashboard.html` `CONFIG.upcomingShows[]`; official show logos pulled from `media.koelnmesse.io` CDN or favicons | Static (updated when show calendar changes) | None |
| Team Calendar (birthdays, anniversaries, office closed) | Hardcoded in `dashboard.html` `CONFIG.employees[]` and `CONFIG.holidays[]` | Static (edit HTML annually) | None |
| Live TV feed | Sky News 24/7 YouTube live stream (`UCoMdktPbSTixAyNGwb-UYkQ`) with CC forced on | Continuous | None |

---

## How to update

**Content changes only (fastest — no rebuild):**

Most day-to-day content (OOO, news, KM press) updates automatically at 6am. Nothing to do.

**Ad-hoc edit of `dashboard-data.json`** (e.g., manual override of a news headline):
1. Edit the file in the GitHub UI at `github.com/darrinstern-KM/koelnmesse-dashboard/blob/main/dashboard-data.json`
2. Commit to `main`
3. Fire Stick picks it up within 30 minutes (or force reload)

**Layout / feature changes (`dashboard.html`):**
1. Edit `dashboard.html` locally on the Managing Director's Mac
2. Verify JS syntax passes (`node --check` on the extracted `<script>` block)
3. Bump the version stamp near the top of the file
4. Upload the new file to GitHub via web UI or drag-drop
5. Commit to `main` — GitHub Pages republishes within 30 seconds

**Adding a new upcoming show:**
Edit `CONFIG.upcomingShows[]` in `dashboard.html`. Each entry needs: `name`, `sub`, `start`, `end`, `loc`, `color` (brand hex), `domain` (for favicon fallback), and optionally `logoUrl` (real brand logo from `media.koelnmesse.io`).

**Adding/removing employees:**
Edit `CONFIG.employees[]` in `dashboard.html`. Each entry: `name`, `birthday` (MM-DD), `startDate` (YYYY-MM-DD).

**Adding company holidays:**
Edit `CONFIG.holidays[]`. Review each January.

---

## Deployment

- **Hosting:** GitHub Pages, served from the `main` branch, root folder
- **Repo:** `darrinstern-KM/koelnmesse-dashboard`
- **Domain:** `darrinstern-km.github.io` (default GitHub Pages URL)
- **Custom domain option:** none currently; could point `dashboard.koelnmesse.us` at Pages if IT sets up DNS
- **TLS:** GitHub-provided, auto-renewed
- **Fire Stick browser:** Amazon Silk (identified as `AFTCA002`)
- **Resolution:** 1920×1080 @ 1× DPR (confirmed via earlier diagnostic)

---

## Version history

| Version | Date | Notes |
|---|---|---|
| **v2.0** | 2026-09-03 | JSON schema mismatch fix (accepts old + new shapes); ticker de-duped vs news cards (now shows industry + KM press with date labels); Sky News replaces NBC News NOW; version badge added; 3am Chicago reload trigger added; repo migrated from personal `darrinstern` to `darrinstern-KM`; wired via new Zapier connection |
| v1.x | May–Aug 2026 | Iterative build: TV channel iterations (Bloomberg → Yahoo Finance → NBC → Sky), social wall added then replaced by Featured Show Spotlight, viewport diagnostic added and removed, Wort des Tages truncation fixes, layout restructure for PIP + news row, JSON-driven data pipeline (moved OOO + news off client-side RSS), Zapier SharePoint pipeline (later replaced by GitHub Pages), favicon added, anti-sleep Wake Lock + heartbeat added, Bloomberg captions forcer, 3-tier logo strategy (real logos → favicon@256 → wordmark fallback) |
| v1.0 | May 2026 | Initial one-page dashboard: weather, markets, industry news, KM news, upcoming shows, team calendar |

---

## Roadmap (post-v2.0)

Not committed — ideas parked for future versions:

- **v2.1 — ISM Middle East + full show list audit** (blocked on brand-provided source data)
- **v2.2 — Private hosting migration** (Azure Static Web Apps + Entra ID SSO) so employee birthdays / hire dates / OOO aren't world-readable
- **v2.3 — HR system integration** (pull employee list + PTO from `koelnmesse-hr-production.up.railway.app` — requires HR-side API endpoints per audit; only ships after private hosting is in place)
- **v3.0 — Rotating globe of Koelnmesse's global show footprint** (globe.gl, ~200KB add), replacing or augmenting the Featured Show Spotlight
- **v3.1 — AI-synthesized "Today's Executive Brief"** strip below the header (single line, decision-tool feel)
- **v3.2 — Freshness stamps** on every panel

---

## Known limitations

- **Public repo = public employee data.** Birthdays and hire dates (via anniversary math) are visible in the source. Migration to private hosting is the v2.2 priority.
- **Live TV PIP autoplay** may require a one-time click on the Fire Stick to grant autoplay permission; captions availability depends on Sky News's live captioners.
- **Fire Stick browser (Silk) does not support some modern web APIs** as reliably as Chrome. Wake Lock works but sometimes silently drops.
- **GitHub Pages has no auth.** Anyone with the URL can view the dashboard.
- **News search is best-effort.** The daily sync uses web search to find headlines; occasional stale or misaligned stories are possible until the next sync.

---

## Support / ownership

- **Product owner:** Darrin Stern (Managing Director, Koelnmesse Inc. Chicago)
- **Content maintenance:** happens automatically via the 6am scheduled sync task (Claude Code + Zapier)
- **Layout/feature changes:** manual edits to `dashboard.html`, versioned above
- **Repo access:** `darrinstern-KM` GitHub account owns the repo
- **If something breaks:** most issues resolve by force-reloading the Fire Stick browser (long-press remote → refresh). Persistent issues → edit locally, re-upload `dashboard.html`.

---

## Credits

Built collaboratively with Claude (Anthropic). Data feeds attributed on the dashboard itself (weather via Open-Meteo, FX via ECB/Frankfurter, live news via Sky News, headline curation via industry + press sources listed above).

Koelnmesse brand identity: Koelnmesse GmbH.
