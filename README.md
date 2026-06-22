# Ads Command Center

### ▶️ [**Open the live demo →**](https://script.google.com/macros/s/AKfycbxLdJehYqxPBmTLfTtMUhkcnE7OGgTpFx1iCNVaQD4TBOuvsmEN_n9KvrywxJ1Aq4dW/exec)

*A fully clickable demo running on fictional data — explore every page, run the
AI analysis, spin the globe. No login required.*

---

A single-file **Google Apps Script web app** that turns a multi-account Google Ads
setup into a fast, executive-ready dashboard — built for a property management
company running paid campaigns across multiple apartment communities.

It pulls **live data from the Google Ads API**, blends it with a budget /
occupancy Google Sheet, visualizes everything with Chart.js, and ships an
**AI analyst** (powered by Claude) that reads the live numbers and returns a
graded, prioritized action plan.

> Built by **Debashish**. This repository is a sanitized showcase version — all
> credentials, account IDs, and client identifiers have been replaced with
> placeholders.

---

## Why it exists

Agencies and in-house marketers managing many Google Ads accounts usually live
inside spreadsheets and the native Ads UI, switching accounts one at a time.
This dashboard collapses the whole portfolio into one screen: every account,
every campaign, budget pacing, occupancy, creative performance, and search-term
waste — plus an AI assistant that tells you what to do about it.

It runs entirely on Google Apps Script. **No server, no hosting bill, no build
step.** Deploy it once and the team opens a single permanent URL.

---

## Features

**Overview** — Portfolio-wide hero trend (spend / clicks / conversions) with
period-over-period deltas, KPI cards with sparklines, spend-distribution donut,
efficiency ranking, top performers, and a "needs attention" list that surfaces
accounts spending with zero conversions.

**Accounts** — Searchable, expandable per-account cards. Toggle between
Campaigns and Ad Groups with lazy loading, caching, and a watchdog timer.

**Analytics** — Six views: Rankings, an efficiency Quadrant (CTR vs conversion
rate, bubble = spend), Compare (share-of-spend vs share-of-conversions), a 3D
**Geo Globe**, a heat-shaded performance Matrix, and **Timing** (a day×hour
heatmap of when leads actually come in, with hover tooltips and best-window
detection).

**Search Terms & Keywords** — Wasted-spend detection, top converters, negative
keyword guidance, match-type breakdown, sortable tables.

**Ads & Creative** — Ad performance, creative assets, an image gallery with a
modal viewer, and video assets — with budget-drain flags and performance labels.

**Budget** — Blends live ad spend with a budget Google Sheet. Pace/day,
recommended daily spend, projected month-end, over/under flags, an
occupancy-vs-spend scatter, and a budget-vs-vacancy allocation analysis that
flags where money should move.

**Reports** — One-click email report plus an automated weekly Monday-morning
send.

**AI Insights** — Claude reads the live data and returns a health score, grade,
prioritized actions (with expected impact), opportunities, and wins. Scoped per
property and per focus area (Overview, Images, Videos, Search Terms, Keywords,
Budget). A floating **chat assistant** answers ad-hoc questions in plain
language, with optional voice input and read-aloud.

**Settings** — In-app Anthropic API key management (stored in Script Properties,
never in source), connection info, and an access log with allowed/denied stats.

---

## Tech stack

| Layer        | Technology |
|--------------|------------|
| Backend      | Google Apps Script (single `Code.gs`) |
| Ads data     | Google Ads API v21 (REST via `UrlFetchApp`, GAQL queries) |
| Budget data  | Google Sheets (`SpreadsheetApp`) |
| AI           | Anthropic API — `claude-sonnet-4-6` |
| Charts       | Chart.js |
| Maps / globe | globe.gl + MapLibre (loaded on demand) |
| Auth         | OAuth 2.0 refresh-token exchange; manager ID passed as `login-customer-id` |
| Fonts        | Bricolage Grotesque, Hanken Grotesk, JetBrains Mono |

The entire UI — HTML, CSS, and JavaScript — is generated as a string array
inside `Code.gs` and served by `doGet()`, so the whole app is one file with zero
dependencies to install.

---

## Architecture

```
Team member opens the /exec URL
        │
        ▼
doGet()  ──►  checks viewer email against ACCESS_DOMAINS (or access code)
        │           │
        │           └─►  not allowed ► access-gate page
        ▼
serves the dashboard HTML
        │
        ▼
browser JS calls server functions via google.script.run
        │
        ├─►  getAccessToken()  ──►  exchanges refresh token at
        │                            oauth2.googleapis.com/token
        │
        ├─►  Google Ads API v21  (googleAds:search, GAQL)
        │       login-customer-id: MANAGER_ID
        │
        ├─►  Budget Sheet  (SpreadsheetApp.openById)  ► blended with live spend
        │
        └─►  callClaude()  ──►  api.anthropic.com/v1/messages
                                 (key read from Script Properties)
```

---

## Setup

### Prerequisites
- A Google Ads **Manager (MCC)** account with API access
- A Google Ads **API developer token**
- A Google Cloud project with an **OAuth 2.0 client** (consent screen in
  *Production*, not *Testing* — testing tokens expire after 7 days)
- A Google Sheet for budget / occupancy data (optional but recommended)
- An **Anthropic API key** (for the AI features)

### Steps

1. **Create the Apps Script project**
   Go to [script.google.com](https://script.google.com) → New project.

2. **Add the code**
   Paste `Code.gs` over the default file. Replace the manifest by enabling
   *Project Settings → Show "appsscript.json"* and pasting `appsscript.json`.

3. **Fill in your credentials**
   Edit the config block at the top of `Code.gs` — replace every `YOUR_*`
   placeholder:

   ```js
   const DEV_TOKEN       = 'YOUR_DEVELOPER_TOKEN';
   const MANAGER_ID      = 'YOUR_MANAGER_ID';      // digits only, no dashes
   const CLIENT_ID       = 'YOUR_OAUTH_CLIENT_ID';
   const CLIENT_SECRET   = 'YOUR_OAUTH_CLIENT_SECRET';
   const REFRESH_TOKEN   = 'YOUR_REFRESH_TOKEN';
   const BUDGET_SHEET_ID = 'YOUR_BUDGET_SHEET_ID';
   const REPORT_EMAIL    = 'reports@example.com';
   const ACCESS_DOMAINS  = ['example.com'];
   var   ACCESS_CODE     = 'YOUR_ACCESS_CODE';
   ```

4. **Generate a refresh token**
   Use the [OAuth 2.0 Playground](https://developers.google.com/oauthplayground)
   with your client ID/secret and the `https://www.googleapis.com/auth/adwords`
   scope, then exchange the authorization code for a refresh token.

5. **Deploy**
   *Deploy → New deployment → Web app*. Set **Execute as: Me** and
   **Who has access: Anyone**. Authorize the scopes when prompted.

6. **Add your Anthropic key**
   Open the deployed app, go to **Settings**, and paste your `sk-ant-...` key.
   It is saved to Script Properties — never written to source.

### Notes & gotchas
- **API version matters.** Google Ads API v15–v17 were sunset in June 2025 and
  return a misleading HTML 404. Keep `API_VERSION = 'v21'` (or newer).
- **Publish the consent screen to Production.** Tokens minted under a *Testing*
  consent screen expire after 7 days.
- **The `/exec` URL is permanent** across redeploys — only a brand-new
  deployment changes it.
- When `appsscript.json` declares explicit `oauthScopes`, automatic scope
  detection is off, so the listed scopes must be complete (they are, in this
  repo).

---

## Screenshots

> 🔗 **Best experienced live:** [open the interactive demo](https://script.google.com/macros/s/AKfycbxLdJehYqxPBmTLfTtMUhkcnE7OGgTpFx1iCNVaQD4TBOuvsmEN_n9KvrywxJ1Aq4dW/exec)

**AI Insights** — Claude reads the live data and returns a graded, prioritized action plan.
![AI Insights](ai-insights.png)

**Overview** — portfolio-wide trend, KPI cards, and accounts that need attention.
![Overview](overview.png)

**Analytics** — geo globe, efficiency quadrant, and a day-by-hour timing heatmap.
![Analytics](analytics.png)

**Budget** — live spend blended with budget and occupancy, with reallocation analysis.
![Budget](budget.png)

---

## Security

This is a **sanitized** version. Before this code was published, every secret
was replaced with a placeholder:

- Developer token, OAuth client ID/secret, and refresh token → `YOUR_*`
- Manager account ID and budget sheet ID → `YOUR_*`
- Real email addresses and access domains → `example.com`
- The access code → `YOUR_ACCESS_CODE`
- Client/company name and account names → generic copy

The Anthropic API key was never in source to begin with — it lives in Script
Properties and is managed from the in-app Settings page.

If you fork this, **keep your real credentials out of the committed file.** For a
production setup, prefer storing all secrets in Script Properties and reading
them at runtime rather than hard-coding them in `Code.gs`.

---

## License

MIT — see [LICENSE](LICENSE).

---

*Built by Debashish — a marketing technologist who turns Google Ads data into
decisions, with a soft spot for analytics and AI.*
