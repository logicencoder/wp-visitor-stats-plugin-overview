# WP Visitor Stats — WordPress plugin

![WP Visitor Stats — Overview dashboard with KPI tiles and trend charts](assets/featured.png)

**WP Visitor Stats** is first-party analytics on [logicencoder.com](https://logicencoder.com): page views, geography, technology mix, UTM campaigns, custom events, live sessions, IP bans, and a built-in URL shortener in WordPress wp-admin.

## Tech stack

| Layer | Technologies |
|-------|--------------|
| WordPress plugin | PHP single-file (`wp-visitor-stats.php`, ~9.7k LOC) + `js/admin.js` (~3.8k LOC), inline admin CSS |
| Persistence | WordPress options + MySQL |
| Admin charts | Chart.js 3.9, DataTables 1.11 (paginated tables), Leaflet 1.9 on geo map |
| Site tracking | Page views, time on page, scroll depth, custom events, UTM capture |
| Geo / VPN | IP lookup with country, region, city, and VPN/proxy flags |
| Security | IP and CIDR bans at `init`, auto-ban on repeated 404 patterns, ban whitelist |
| Short links | Public `{yoursite}/go/{slug}` → 301 redirect with click attribution |
| Integration | Tracking snippet embeds on static HTML from [mexc-live-stats-plugin](https://github.com/logicencoder/mexc-live-stats-plugin-overview) snapshot pages |
| Hosting | WordPress on shared hosting; wp-admin UI only |

## Admin menu layout

Top-level wp-admin menu **Visitor Stats** (chart-bar icon). Thirteen submenu screens:

| Screen | Primary use |
|--------|-------------|
| **Overview** | KPI tiles, trend charts, traffic sources, heatmap by hour × weekday |
| **IP Addresses** | Searchable visit log, filters, CSV export, ban action |
| **Live Visitors** | Active sessions in the last five minutes |
| **Geo Reports** | World map and country/region/city tables |
| **Content Analysis** | Page performance, entry/exit pages, 404 report |
| **Technology** | Browser, OS, and device charts and tables |
| **Campaigns** | UTM attribution and in-admin tracking guide |
| **Custom Events** | Named event rollups and front-end API docs |
| **All Visitors** | Full visit log with source filter |
| **Ban List** | Whitelist, manual bans, auto-ban summary, unban |
| **URL Shortener** | Create `/go/` links, toggle active, per-link stats |
| **Diagnostics** | Environment info, self-tests, optional debug log tail |
| **Settings** | Tracking modes, retention, exclusions, database tools |

## Shared date range control

Analytic pages (Overview, Geo, Content, Technology, Campaigns, Custom Events) share the same **Date Range** bar at the top:

| Preset | Window |
|--------|--------|
| Today | Current calendar day |
| Yesterday | Previous calendar day |
| Last 24 Hours | Rolling twenty-four hours |
| Last 3 / 7 / 30 Days | Rolling windows (7 days is the default) |
| This Month / Last Month | Calendar month boundaries |
| Last 2 / 6 Months | Rolling multi-month windows |
| Custom Range | Start + end date pickers with **Apply** |

Each page has its own **Refresh Data** button (spinner while loading). Hidden fields carry the active start/end dates and page id so AJAX reloads stay scoped to the screen you are on. Admin timestamps can display in a **custom timezone** when enabled in Settings — storage remains in the WordPress site timezone.

## Overview dashboard

The **Overview** screen opens with ten **expandable metric tiles** in two rows — click any tile to flip open an inline breakdown table without leaving the page:

| Tile | Headline metric | Expand reveals |
|------|-----------------|----------------|
| **Total Visits** | All hits in range | Visit distribution detail |
| **Unique Visitors** | Distinct IPs/sessions | Unique breakdown |
| **Pages / Session** | Average depth | Session depth detail |
| **Avg. Session** | Mean session length | Duration breakdown |
| **Bounce Rate** | Single-page sessions | Bounce detail |
| **New Visitors** | First-time vs returning split | New/return detail |
| **Bot Visits** | Bot-classified hits (when tracking bots) | Bot volume detail |
| **VPN Visits** | VPN/proxy flagged hits | VPN detail |
| **Peak Hour** | Busiest hour of day | Hourly distribution |
| **Countries** | Country count | Top country list |

Four **Chart.js** panels sit below the tiles:

| Chart | What it shows |
|-------|----------------|
| **Visitor Trends** | Visit volume over the selected range |
| **New vs. Returning** | Doughnut split of first-time vs repeat sessions |
| **VPN & Bot Traffic Trend** | Line trend when bot stats feed alerts (settings-controlled) |
| **Traffic Sources** | Doughnut of referrer categories (direct, search, social, etc.) |

A **Traffic Sources & Alerts** panel can surface up to four security or anomaly notices when alert rules fire. Tables list **Top Referrers** and **Top Pages** with view counts and average time on page. The **Traffic Heatmap** is a time-of-week grid (hour × weekday) showing aggregate visit intensity.

![Overview — VPN and bot trend with traffic sources and alerts](assets/overview-alerts-sources.png)

![Overview — top referrers and top pages tables](assets/overview-referrers-pages.png)

![Overview — traffic heatmap by hour and day of week](assets/overview-heatmap.png)

## IP addresses

**IP Addresses** is a paginated visit log. A **DataTables** grid lists IP, visit time, country, page URL, referrer, browser, OS, device, bot flag, VPN flag, and actions. Column resize handles let you widen URL or referrer columns on large monitors.

**Filters** cover IP search, country dropdown, bot (All / Bots / Humans), VPN (All / VPN / Exclude VPN), and from/to date pickers. **Apply Filters**, **Reset**, and **Refresh** reload the grid without a full page reload.

**Export to CSV** downloads the current filter set as a spreadsheet (capped at a high export limit). Each row exposes:

| Action | Effect |
|--------|--------|
| **D** | Expand IP detail — geo context, ISP/org line when available |
| **B** | Open ban dialog — pre-fills IP for one-click block |

![IP Addresses — filtered visit log with export and ban actions](assets/ip-addresses.png)

## Live visitors

**Live Visitors** lists sessions active in the **last five minutes**.

The header shows a live **Active Visitors** count. The table lists IP, country flag (via flag CDN), time ago, page title and URL, browser, OS, and device.

| Control | Behaviour |
|---------|-----------|
| **Auto-refresh** toggle | Poll for new rows on an interval (default on) |
| Refresh rate | **5s**, **10s**, **30s**, or **60s** |
| **Refresh Now** | Immediate manual reload |

## Geo reports

**Geo Reports** opens with three tabs — **Countries**, **Regions**, and **Cities** — each honouring the shared date range and **Refresh** control.

The **Countries** tab combines a **Leaflet** world map (OpenStreetMap tiles) with choropleth-style country colouring and a **Top Countries** table: flag, visits, unique visitors, VPN/proxy percentage, and share of total traffic.

The **Regions** tab ranks states/provinces with the same visit and unique-visitor columns. The **Cities** tab ranks municipalities with the same columns.

![Geo Reports — world map and top countries table](assets/geo-reports.png)

## Content analysis

**Content Analysis** opens with four summary cards: total page views, unique pages, average time on page, and average bounce rate for the range.

Three doughnut charts break down **traffic sources**, **devices**, and **top browsers**. The **Page Performance** table lists each URL with views, unique views, average time, bounce rate, and exit rate — sortable by column.

Collapsible sections below expose:

| Section | Purpose |
|---------|---------|
| **Top 10 Entry Pages** | Where sessions start |
| **Top 10 Exit Pages** | Where sessions end |
| **Session Duration Distribution** | Histogram-style chart of session lengths |
| **Pages per Session** | Chart of depth distribution |
| **404 Error Pages** | URL, hit count, unique IPs — feeds auto-ban logic |

![Content Analysis — page performance table and traffic breakdown charts](assets/content-analysis.png)

## Technology

The **Technology** screen charts **browser distribution** and **device distribution** (desktop, mobile, tablet) for the selected period. Sortable tables list browsers, devices, and operating systems with visit counts and percentage share.

![Technology — browser and device distribution charts](assets/technology.png)

## Campaigns

The **Campaigns** screen rolls up **UTM parameters** captured on first touch and persisted in the browser for the session (via `localStorage`), so a visitor keeps campaign credit across multiple page views.

**Campaign Overview** summary cards load at the top for the selected date range. The **Top Campaigns** table lists campaign name, source, medium, visits, and unique visitors.

An in-page **How to Use UTM Tracking** guide (always visible below the data cards) documents each parameter with copy-paste examples:

| Parameter | Typical values |
|-----------|----------------|
| `utm_source` | google, facebook, newsletter |
| `utm_medium` | cpc, email, social, banner |
| `utm_campaign` | summer_sale, product_launch |
| `utm_term` | paid search keyword (optional) |
| `utm_content` | ad variation id (optional) |

Worked examples cover Facebook posts, email newsletters, and Google Ads query strings. The tip block explains that UTMs stick for the session — a landing-page tag attributes downstream pages until the session ends.

## Custom events

The **Custom Events** screen aggregates named events fired from your front-end JavaScript.

**Event Summary** cards load first for the date range. The **All Events** table lists event name, category, label, count, and total value (for numeric event payloads).

The **How to Track Custom Events** section documents the global helper:

```javascript
wpVisitorStatsTrackEvent(name, category, label, value)
```

## All visitors

**All Visitors** mirrors the IP Addresses grid — same columns, filters, **Apply** / **Reset** / **Refresh**, and row actions **D** and **B** — plus a **Source** dropdown to narrow by visit type. **CSV export** is on IP Addresses only.

## Ban list and edge blocking

**Ban List** opens with summary cards:

| Card | Meaning |
|------|---------|
| Total Bans | All active ban rows |
| Bans Today | Blocks added today |
| 404 Attempts (7d) | Not-found hits in the last week |
| Unique IPs on 404 (7d) | Distinct offenders |
| VPN Traffic (7d) | VPN-flagged volume |

A collapsible **Top IPs with 404 hits** table highlights repeat scanners before they trip auto-ban.

**Ban Whitelist** accepts single IPs, CIDR notation, or dash ranges with a description field. Whitelisted addresses **never** auto-ban. **+ Add to Whitelist**, **Reload**, edit in a modal, or remove rows.

**Manual ban** accepts a single IP or dash range plus an optional reason — **Ban** writes immediately to the ban table.

**All Banned IPs / Ranges** paginates with page-size selector (10 / 25 / 50 / 100) and **Unban** per row.

Enforcement runs on every front-end request before WordPress renders: banned clients receive **HTTP 403 Forbidden**. Logged-in administrators are never blocked. **Auto-ban** fires when an IP exceeds the **404 threshold** within twenty-four hours. **Stricter Rules for China** (Settings) applies a lower threshold for Chinese geo when enabled.

![Ban List — summary cards, whitelist, and 404 offender panel](assets/ban-list-whitelist.png)

![Ban List — manual ban form and banned IP table](assets/ban-list-table.png)

## URL shortener

Public short URLs live at **`{yoursite}/go/{slug}`** — a **301 redirect** to the target with click logging (geo, device, bot flag, referrer). Inactive links stay in the table but stop redirecting when toggled off.

**Summary cards:** total clicks, active links, clicks today, total links created.

**New Short Link** opens an inline form: title/note, slug, target URL → **Save** or cancel ✕.

The links table shows short URL, target, title, clicks, unique clicks, an **active/inactive** toggle, and row actions:

| Action | Effect |
|--------|--------|
| **Copy** | Clipboard the public `/go/` URL |
| **View Stats** | Inline expand with per-link analytics |
| **Edit** | Change slug, target, or title |
| **Delete** | Removes link and click history (confirm dialog) |

**Search links** filters the table client-side. **View Stats** expands a panel with day-range buttons (**7 / 30 / 90 days**), mini metrics, a bar chart of clicks over time, and top countries and referrers for that link.

![URL Shortener — link table with clicks and active toggles](assets/url-shortener.png)

## Settings and data hygiene

**Settings** groups everything that changes how data is collected and retained.

| Control | Behaviour |
|---------|-----------|
| **Track Admin Users** | Include logged-in administrators in analytics when enabled |
| **Track Bots** | Store and show bot traffic; when off, bots are never written |
| **Use bot stats in Alerts + VPN graph** | Feed bot signals into Overview security alerts and VPN/bot trend chart |
| **Tracking on/off toggles** | Control what gets recorded and which reports populate |
| **Display timezone override** | Pick a timezone for admin display only; storage stays in site TZ |
| **Auto-Ban: 404 Threshold** | Hits within 24h before auto-ban (default five) |
| **Stricter Rules for China** | Enable lower CN-specific threshold |
| **China 404 Threshold** | Separate numeric limit when strict China rules are on (default two) |
| **Data Retention** | 30 days, 90 days, 6 months, 1 year, 2 years, or never purge (0) |
| **Debug Mode** | File logging and Diagnostics log panel |

The **Database** card shows live row counts for visits, sessions, and page stats.

| Button | Effect |
|--------|--------|
| **Reset All Data** | Truncate core analytics tables after confirmation |
| **Remove Duplicate Visits** | Collapse same IP + URL within thirty seconds |
| **Backup DB (SQL)** | Browser download of dated SQL dump |

**Excluded IPs** textarea accepts one IP per line or comma-separated lists. **Save Excluded IPs** skips future tracking. **Remove Excluded IPs from Statistics** deletes historical rows for those addresses while keeping the exclusion list.

A daily scheduled job purges visits and sessions older than the retention setting. Page-level rollups older than one year are trimmed independently. Custom events, bans, short links, and ban whitelist rows are not removed by that cleanup job.

## Diagnostics

**Diagnostics** shows WordPress, PHP, MySQL, and plugin version in **System Information**, plus whether debug mode is on.

**Database Information** verifies expected tables exist and reports total visit and session counts.

**Run Tests** (or **Run Tests Again**) executes three grouped check suites:

| Suite | Checks |
|-------|--------|
| **Database** | Table presence, connectivity, row readability |
| **Tracking** | Tracking registration, endpoint reachability, sample write path |
| **Settings** | Toggle consistency, retention value, tracking mode flags |

Results render pass/fail per check with detail text. When **Debug Mode** is enabled, a **Debug Log** panel shows the tail of the plugin log file (~last two thousand lines) and a **Clear Log** button.

## Public site tracking

On public pages (including static HTML from [mexc-live-stats-plugin](https://github.com/logicencoder/mexc-live-stats-plugin-overview) snapshot pages when the plugin is active), the tracker records:

| Signal | When it fires |
|--------|----------------|
| **Page view** | Initial load with URL, title, screen size, language, referrer, UTM params |
| **Time on page** | On tab close or navigation away |
| **Scroll depth** | At **50%** and **90%** scroll |
| **Custom events** | When your code calls `wpVisitorStatsTrackEvent(name, category, label, value)` |

**Bot detection** classifies crawlers and automated clients. When **Track Bots** is off, bot hits are discarded. **VPN/proxy** flags come from geo lookup and appear in visitor tables, geo reports, and Overview alerts. **Excluded IPs** and tracking toggles in Settings skip recording before rows are stored.

Private code: [logicencoder/wp-visitor-stats-plugin](https://github.com/logicencoder/wp-visitor-stats-plugin) (v1.4.x runtime)

See [REPOS.md](REPOS.md).

---

**Made by [Logic Encoder](https://logicencoder.com)** · [GitHub](https://github.com/logicencoder) · [Contact](https://logicencoder.com/contact/)
