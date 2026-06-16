# WP Visitor Stats — WordPress plugin

**WP Visitor Stats** is first-party analytics for [logicencoder.com](https://logicencoder.com): page views, geography, technology mix, UTM campaigns, custom events, live sessions, IP bans, and a built-in URL shortener — all inside WordPress wp-admin. Data stays in your database under operator control; there is no third-party analytics dashboard or off-site clickstream export.

The plugin pairs a **browser beacon** (Chart.js dashboards, geo maps, campaigns) with an optional **server-side fallback** (raw request log for traffic that never runs JavaScript). Operators get one menu for marketing insight and abuse response without opening hosting panels.

## Tech stack

| Layer | Technologies |
|-------|--------------|
| WordPress plugin | PHP single-file (`wp-visitor-stats.php`, ~9.7k LOC) + `js/admin.js` (~3.8k LOC), inline admin CSS |
| Persistence | WordPress options + MySQL |
| Admin charts | Chart.js 3.9, DataTables 1.11 (server-side pagination), Leaflet 1.9 on geo map |
| Front-end tracking | Inline JS beacon, `sendBeacon` for exits/scroll/events, nonce-protected POST |
| Geo / VPN | IP lookup with country, region, city, and VPN/proxy flags |
| Security | IP and CIDR bans at `init`, auto-ban on repeated 404 patterns, ban whitelist |
| Short links | Public `{yoursite}/go/{slug}` → 301 redirect with click attribution |
| Integration | Tracking snippet embeds on static HTML from [mexc-live-stats-plugin](https://github.com/logicencoder/mexc-live-stats-plugin-overview) snapshot pages |
| Hosting | WordPress on shared hosting; wp-admin operator UI only |

## Why operators use it

Logic Encoder runs many public surfaces — gas tracker, MEXC coin pages, DNX tools, shop landings, member flows. You need to know **which pages convert**, **where traffic originates**, and **which IPs probe 404s** — without shipping full sessions to external SaaS. WP Visitor Stats answers that in one sidebar:

- **Dashboard KPIs and trends** with shared date presets across reports.
- **Per-IP forensics** with filters, expandable detail, and CSV export on the JavaScript visit stream.
- **Live visitor strip** with configurable auto-refresh for the last few minutes of activity.
- **Geo, content, and technology breakdowns** for editorial and product decisions.
- **UTM campaign tables** and **custom event counters** for experiments.
- **Ban list and auto-ban rules** that return HTTP 403 before WordPress renders abusive clients.
- **URL shortener** with per-link click stats on the same domain.

Front-end tracking respects toggles for admins, bots, excluded IPs, and JavaScript on/off. Server-side mode logs separately and appears on the **All Visitors** screen for a complete request picture.

## Admin menu layout

Top-level wp-admin menu **Visitor Stats** (chart-bar icon). Thirteen screens share a **date range** control on analytic pages: presets from Today through Last 6 Months, plus Custom Range with Apply and **Refresh Data** per section.

| Screen | Primary use |
|--------|-------------|
| **Overview** | KPI tiles, trend charts, traffic sources, heatmap by hour × weekday |
| **IP Addresses** | Searchable visit log (JavaScript hits only), filters, CSV export, ban action |
| **Live Visitors** | Active sessions in the last five minutes |
| **Geo Reports** | World map and country/region/city tables |
| **Content Analysis** | Page performance, entry/exit pages, 404 report |
| **Technology** | Browser, OS, and device charts and tables |
| **Campaigns** | UTM attribution and in-admin tracking guide |
| **Custom Events** | Named event rollups and front-end API docs |
| **All Visitors** | Combined JavaScript + server-side log with source filter |
| **Ban List** | Whitelist, manual bans, auto-ban summary, unban |
| **URL Shortener** | Create `/go/` links, toggle active, per-link stats |
| **Diagnostics** | Environment info, self-tests, optional debug log tail |
| **Settings** | Tracking modes, retention, exclusions, database tools |

## Overview dashboard

The **Overview** screen is the daily health check. Ten **expandable metric tiles** cover total visits, unique visitors, pages per session, average session length, bounce rate, new visitors, bot visits, VPN visits, peak hour, and country count. Click a tile to open its breakdown without leaving the page.

Four **Chart.js** panels sit below the tiles:

| Chart | What it shows |
|-------|----------------|
| **Visitor Trends** | Visit volume over the selected range |
| **New vs. Returning** | Doughnut split of first-time vs repeat sessions |
| **VPN & Bot Traffic Trend** | Line trend when bot stats feed alerts (settings-controlled) |
| **Traffic Sources** | Doughnut of referrer categories |

A **Traffic Sources & Alerts** panel can surface up to four security or anomaly notices when alert rules fire. Tables list **Top Referrers** and **Top Pages** with view counts and average time on page. The **Traffic Heatmap** is a time-of-week grid (hour × weekday) — aggregate visit intensity, not click coordinates.

Overview totals and charts use **JavaScript-tracked visits only** so charts reflect real browser sessions, not server-side prefetch rows.

## IP addresses and live visitors

**IP Addresses** is the forensic workbench for the JavaScript visit stream. A server-side **DataTables** grid lists IP, visit time, country, page URL, referrer, browser, OS, device, bot flag, VPN flag, and actions. Filters cover IP search, country, bot vs human, VPN vs exclude, and from/to dates — **Apply Filters**, **Reset**, and **Refresh** reload the grid.

**Export to CSV** downloads the current filter set (JavaScript visits, capped at a large operator limit). Each row exposes **D** to expand IP detail with geo context and **B** to open the ban dialog for that address.

**Live Visitors** shows who is active in the **last five minutes**: IP, country flag, time ago, page title and URL, browser, OS, and device. Turn **auto-refresh** on or off and pick **5s / 10s / 30s / 60s** intervals, or hit **Refresh Now**. The header shows a live **Active Visitors** count.

## Geo reports

**Geo Reports** opens with three tabs — **Countries**, **Regions**, and **Cities**.

The **Countries** tab combines a **Leaflet** world map (OpenStreetMap tiles) with a **Top Countries** table: flag, visits, unique visitors, VPN/proxy percentage, and share of traffic. **Regions** and **Cities** tabs rank subdivisions and cities with the same visit metrics. All geo views honour the shared date range and **Refresh** control.

## Content analysis

**Content Analysis** answers “which URLs earn attention.” Summary cards show total page views, unique pages, average time on page, and average bounce rate for the range.

Three doughnut charts break down **traffic sources**, **devices**, and **top browsers**. The **Page Performance** table lists each URL with views, unique views, average time, bounce rate, and exit rate.

Collapsible sections expose **Top 10 Entry Pages**, **Top 10 Exit Pages**, **Session Duration Distribution**, and **Pages per Session** charts. A **404 Error Pages** block lists URLs that returned not-found, hit counts, and unique IPs — the same signal that feeds auto-ban thresholds on the Ban List.

## Technology

The **Technology** screen charts **browser distribution** and **device distribution** (desktop, mobile, tablet) for the selected period. Sortable tables list browsers, devices, and operating systems with visit counts and percentage share. Use it to prioritise QA browsers and spot mobile-heavy landing pages.

## Campaigns and custom events

**Campaigns** rolls up UTM parameters captured on first touch and persisted in the browser for the session. The **Top Campaigns** table shows campaign name, source, medium, visits, and unique visitors. An in-page **How to Use UTM Tracking** guide documents `utm_source`, `utm_medium`, `utm_campaign`, `utm_term`, and `utm_content` with examples operators can copy into ad links.

**Custom Events** aggregates named events sent from the front end. The **All Events** table lists event name, category, label, count, and total value. The **How to Track Custom Events** section documents the global `wpVisitorStatsTrackEvent(name, category, label, value)` helper for product experiments (button clicks, funnel steps, calculator submits).

## All visitors — dual tracking view

**All Visitors** mirrors the IP Addresses grid but includes **both** JavaScript and server-side rows. A **Source** filter switches All, JavaScript only, or Server-side only. The badge reads **JS + Server** to remind you this is the raw combined log.

Use this screen when you need PHP-logged hits — crawlers, prefetch, or clients without JavaScript — that never appear on Overview charts. Row actions match IP Addresses: **D** for detail, **B** for ban. CSV export is available on IP Addresses; All Visitors is browse-and-ban focused.

## Ban list and edge blocking

**Ban List** is the security operations desk. Summary cards show total bans, bans today, 404 attempts in the last seven days, unique IPs on 404s, and VPN traffic volume.

A collapsible **Top IPs with 404 hits** table highlights repeat offenders. **Ban Whitelist** accepts single IPs or CIDR/range notation with descriptions — whitelisted addresses never auto-ban. Add entries, **Reload** the table, edit in a modal, or remove.

**Manual ban** accepts a single IP or dash range plus an optional reason; **Ban** writes immediately. **All Banned IPs / Ranges** paginates with page-size control and **Unban** per row.

Enforcement runs on every front-end request before WordPress renders: banned clients receive **HTTP 403 Forbidden**. Administrators are never blocked. **Auto-ban** increments when an IP exceeds the **404 threshold** within twenty-four hours; **Stricter Rules for China** lowers that threshold for Chinese geo when enabled in Settings.

## URL shortener

Public short URLs live at **`{yoursite}/go/{slug}`** — a **301 redirect** to the target with click logging (geo, device, bot flag, referrer).

Summary cards show total clicks, active links, clicks today, and total links created. **New Short Link** opens an inline form: title/note, slug, target URL, then **Save** or cancel.

The links table shows short URL, target, title, clicks, unique clicks, an **active/inactive** toggle, and actions: **Copy**, **View Stats**, **Edit**, **Delete** (confirms loss of click history). **Search links** filters the list.

**View Stats** expands an inline panel with day-range buttons (7 / 30 / 90 days), mini metrics, a bar chart of clicks over time, and top countries and referrers for that link.

## Settings and data hygiene

**Settings** groups everything that changes how data is collected and retained.

| Control | Behaviour |
|---------|-----------|
| **Track Admin Users** | Include logged-in administrators in analytics when enabled |
| **Track Bots** | Store and show bot traffic; when off, bots are never written |
| **Use bot stats in Alerts + VPN graph** | Feed bot signals into Overview security alerts and VPN/bot trend chart |
| **JavaScript tracking** | Master switch for the browser beacon and JS-only reports |
| **Server-side tracking** | PHP logs eligible requests separately for All Visitors |
| **Display timezone override** | Convert timestamps in admin only; storage stays in site timezone |
| **Auto-Ban: 404 Threshold** | Hits within 24h before auto-ban (default five) |
| **Stricter Rules for China** | Lower 404 threshold for Chinese IPs when enabled |
| **Data Retention** | 30 days through two years, or never purge |
| **Debug Mode** | File logging and Diagnostics log panel |

The **Database** card shows row counts for visits, sessions, and page stats. **Reset All Data** truncates core analytics tables after confirmation. **Remove Duplicate Visits** collapses same IP + URL within thirty seconds. **Backup DB (SQL)** downloads a dated SQL dump of plugin tables for offline archive.

**Excluded IPs** accepts one IP per line or comma-separated lists. **Save Excluded IPs** skips future tracking; **Remove Excluded IPs from Statistics** deletes historical rows for those addresses while keeping the exclusion list.

A daily scheduled job purges visits and sessions older than the retention setting. Page-level rollups older than one year are trimmed independently. Custom events, bans, and short links are not removed by that job.

## Diagnostics

**Diagnostics** confirms the stack before you trust charts. **System Information** lists WordPress version, PHP version, database server, plugin version, and whether debug mode is on. **Database Information** checks that expected tables exist and reports total visit and session counts.

**Run Tests** (or **Run Tests Again**) executes grouped checks: database connectivity, tracking pipeline health, and settings consistency. Results appear in three sections with pass/fail detail.

When **Debug Mode** is enabled, a **Debug Log** panel shows the tail of the plugin log file and a **Clear Log** button.

## Front-end tracking behaviour

When JavaScript tracking is on, an inline script on public pages sends an initial hit with URL, title, screen size, language, referrer, and UTM parameters, with automatic retry on transient failure. **Page exit** events use `sendBeacon` (with a synchronous fallback) to record time on page. **Scroll depth** fires at fifty and ninety percent milestones. **Custom events** use the same beacon path.

Rate limiting on the public endpoint reduces abuse. Excluded IPs, disabled JavaScript mode, and bot-tracking-off settings short-circuit before rows are stored. Server-side mode deduplicates the same IP and URL within thirty seconds and tags 404 titles for ban logic.

Private code: [logicencoder/wp-visitor-stats-plugin](https://github.com/logicencoder/wp-visitor-stats-plugin) (v1.4.x runtime)

See [REPOS.md](REPOS.md).

---

**Made by [Logic Encoder](https://logicencoder.com)** · [GitHub](https://github.com/logicencoder) · [Contact](https://logicencoder.com/contact/)
