# WP Visitor Statistics — public overview

First-party **WordPress visitor analytics** for [logicencoder.com](https://logicencoder.com): browser tracking, optional server-side logging, operator dashboard, IP bans, UTM campaigns, custom events, and a built-in URL shortener (`/go/{slug}`).

**Implementation (private):** [wp-visitor-stats-plugin](https://github.com/logicencoder/wp-visitor-stats-plugin)

---

## The problem

LogicEncoder runs many public surfaces — gas tracker pages, MEXC live snapshots, DNX tools, shop catalog, marketing landings. Third-party analytics send full clickstreams off-site, complicate GDPR-style questions, and mix bot traffic with real users. Operators need:

- **On-server storage** of visits with geo, referrer, and UTM fields they control.  
- **Split views**: dashboard charts that reflect JavaScript-tracked browsers vs a raw log that can include PHP-logged hits (crawlers, no-JS clients).  
- **Security adjacent tooling**: ban abusive IPs at WordPress `init`, auto-ban repeat 404 scanners, optional VPN/bot flags.  
- **Campaign proof** without exporting spreadsheets from an external SaaS.

This plugin is the shared analytics layer; it is **not** the MEXC trading bot, **not** `le-settings-plugin` (SEO/security), and **not** shop checkout logic.

---

## Who benefits (overall)

| Audience | Benefit |
|----------|---------|
| **Site owner / operator** | One wp-admin menu for traffic, abuse response, and short links |
| **Marketing** | UTM and referrer reporting on owned domains |
| **Product / engineering** | Custom events from front-end snippets; verify tracking in Diagnostics |
| **Visitors** | No extra third-party tracker script required when first-party mode is enough |

---

## How this fits the stack

| Piece | Role |
|-------|------|
| **Hostinger WordPress** | Plugin in `wp-content/plugins/wp-visitor-stats/` |
| **wp-visitor-stats-plugin** (private) | PHP + admin JS source |
| **mexc-live-stats-plugin** | Optionally embeds the same tracking snippet on MEXC snapshot HTML when the plugin is active — analytics on static tool pages without a separate product |
| **le-shop-plugin**, **le-settings-plugin** | Siblings; shop sells products, settings handles site config — analytics stays here |

---

## Tracking model (site-wide)

### JavaScript tracker (default on)

**What:** An inline script on public pages posts the first page view to WordPress `admin-ajax.php`, then sends time-on-page on exit and optional scroll-depth milestones. UTM query parameters are read from the URL and persisted in `localStorage` for first-touch attribution across internal navigation.

**Why:** Real browsers are the signal operators want on Overview charts; AJAX allows geo enrichment and session cookies without full page reloads.

**Who benefits:** Operators measuring human traffic; marketing attributing campaigns that land on tool pages then navigate internally.

### Server-side tracking (optional, default off)

**What:** When enabled in Settings, PHP logs a minimal visit per eligible request with `visit_source = server`. These rows appear under **All Visitors** but are **excluded** from Overview/Geo/Content totals so dashboards stay “browser-realistic.”

**Why:** Captures crawlers, prefetch, or clients that never run JS — useful for abuse forensics without polluting marketing KPIs.

**Who benefits:** Security-minded operators reviewing raw traffic; still optional so normal sites avoid double-counting confusion.

### Bots and rate limits

**What:** User-agent heuristics mark bots; by default bot hits are skipped unless “track bots” is enabled. Public `log_visitor_data` requires a valid nonce and enforces per-IP rate limits (~120/minute) to reduce spam and geo-API abuse.

**Why:** Public AJAX endpoints are attractive targets; nonce + rate limit protect the database without exposing wp-admin.

**Who benefits:** Everyone hosting high-traffic crypto tools that attract scanners.

---

## Admin areas (What / Why / Who)

Capability required: **`manage_options`** (site administrators).

### Overview (`wp-visitor-stats`)

**What:** KPI cards (visits, uniques, bots, VPNs, referrers), date-range picker with timezone-aware labels, trend charts, top pages/referrers, and an hour×day-of-week **traffic heatmap** (aggregated from visit timestamps — not click coordinates).

**Why:** Daily operational picture: is traffic up, which pages spike, are bots skewing numbers, when do peaks occur.

**Who benefits:** Operator on call; marketing reviewing weekly performance without SQL.

### IP Addresses (`wp-visitor-stats-ips`)

**What:** Searchable history per IP: pages seen, geo, bot/VPN flags, ban button opening a dialog (single IP or dash range). IP detail modal with network reputation hints and purge action.

**Why:** Abuse investigations start at “what did this IP do?” — not only aggregate charts.

**Who benefits:** Operator responding to scan bursts or support tickets tied to an address.

### Live Visitors (`wp-visitor-stats-live`)

**What:** Recent sessions still active within a short window — who is on the site right now.

**Why:** Validates launches, ads, or social spikes in real time.

**Who benefits:** Operator during announcements; marketing watching campaign go-live.

### Geo Reports (`wp-visitor-stats-geo`)

**What:** Country/region/city tables and map visualization from stored visit geo fields.

**Why:** Audience geography drives language, compliance awareness, and CDN expectations.

**Who benefits:** Marketing planning; operator spotting unexpected country clusters (often abuse).

### Content Analysis (`wp-visitor-stats-content`)

**What:** Top URLs and titles, engagement/time-on-page style metrics from logged visits.

**Why:** Shows which tools and articles actually get use — not just homepage hits.

**Who benefits:** Product owner prioritizing docs; SEO/editor tuning high-traffic pages.

### Technology (`wp-visitor-stats-tech`)

**What:** Breakdown of browser, OS, and device classes parsed from user agents.

**Why:** Detect broken layouts on specific browsers; see mobile vs desktop mix for tool UIs.

**Who benefits:** Front-end maintainer; operator verifying bot UA patterns.

### Campaigns (`wp-visitor-stats-campaigns`)

**What:** Aggregates `utm_source`, `utm_medium`, `utm_campaign` (and related fields) stored on each visit.

**Why:** Measures owned campaigns (newsletter, Twitter, partner links) without exporting to external analytics.

**Who benefits:** Marketing; growth experiments on logicencoder.com landings.

### Custom Events (`wp-visitor-stats-events`)

**What:** Lists named events submitted via `wpVisitorStatsTrackEvent(...)` from front-end code (category, label, optional value).

**Why:** Product-specific funnels — button clicks, feature toggles, errors — beyond page views.

**Who benefits:** Engineering measuring feature adoption; operator validating instrumentation after deploy.

### Diagnostics (`wp-visitor-stats-diagnostics`)

**What:** Plugin version, table checks, tracking verification AJAX, debug log viewer/clear, visit dedupe tool, SQL backup download of plugin tables.

**Why:** When tracking “suddenly zero,” operators need proof whether JS, nonce, Cloudflare, or DB schema failed — without SSH.

**Who benefits:** Operator and implementer during incidents; safe maintenance before migrations.

### Settings (`wp-visitor-stats-settings`)

**What:** Toggles for admin tracking, bot tracking, JS vs server tracking, retention days, excluded IPs, display timezone, auto-ban thresholds (global and China-specific), VPN/bot chart mode, debug logging.

**Why:** Central policy: how long to keep data, whether to count bots in charts, whether server rows are collected.

**Who benefits:** Site owner defining privacy/retention posture; operator tuning abuse rules.

### All Visitors (`wp-visitor-stats-all`)

**What:** DataTables log of **all** visit rows including `visit_source = server`, with filters for bot/VPN/source.

**Why:** Raw forensic log when dashboards intentionally hide server/bot noise.

**Who benefits:** Operator debugging scanners; contrast with Overview’s JS-only totals.

### Ban List (`wp-visitor-stats-bans`)

**What:** Active bans with reason and timestamp; whitelist editor; stats (bans today, total). Manual ban/unban; auto-bans labeled `auto-rule` after 404 thresholds.

**Why:** Stops repeat offenders at WordPress edge (HTTP 403) before they stress tools or comment systems.

**Who benefits:** Operator under attack; reduces need for manual server firewall edits for common cases.

### URL Shortener (`wp-visitor-stats-links`)

**What:** Create short paths `https://yoursite/go/{slug}` → 301 to target URL; per-link click counts with geo/device/referrer on each hit.

**Why:** Share tracked links on social/chat without a separate shortener SaaS; clicks stay in the same database as visits.

**Who benefits:** Marketing sharing campaign links; operator auditing click fraud on promos.

---

## Security and abuse features

### IP ban at request edge

**What:** On `init` (priority 1), matching banned IPs receive HTTP 403 before WordPress renders the page. Logged-in administrators are never blocked.

**Why:** Fast cut-off for scanners without waiting for page render or external WAF rules.

**Who benefits:** Operator; legitimate users unaffected when bans are accurate.

### Auto-ban on 404 storms

**What:** Configurable count of 404-titled visits per IP in 24 hours triggers automatic ban entry; stricter default threshold for Chinese country code when enabled.

**Why:** Common vulnerability-scan pattern is rapid random URL probes — auto-ban reduces manual work.

**Who benefits:** Operator hosting many plugin endpoints and tool URLs.

### Manual ban dialog (admin JS)

**What:** From IP tables, choose single IP or suggest dash range (e.g. `1.2.3.1-1.2.3.50`) with country metadata preserved.

**Why:** One-click response from the same UI where abuse was spotted.

**Who benefits:** Operator during incident response.

---

## Data retention

**What:** Daily cron deletes visit and session rows older than configured retention (default **180 days**; set **0** to keep forever). Page-stat rollups older than one year are removed separately.

**Why:** Limits MySQL growth on high-traffic crypto sites without manual phpMyAdmin jobs.

**Who benefits:** Site owner controlling storage; visitors’ historical rows age out per policy.

---

## Disabled / legacy features (honest scope)

**What:** User **journey** step UI and front-end **click heatmap capture** were removed from the public tracker (comment in source ~line 1802). Database tables `visitor_journeys` and `visitor_heatmap` remain; admin still registers a `save_heatmap_data` AJAX hook without an active handler. Overview “heatmap” is **time-of-week traffic density**, not mouse coordinates.

**Why:** Operator chose simpler scope — reliable visit counts over heavy behavioral recording.

**Who benefits:** Visitors (less JS work); operators (clearer metrics). Teams needing click heatmaps should use a dedicated product.

---

## MEXC snapshot integration (mention only)

**What:** The MEXC live-stats WordPress plugin can inject the same visitor-tracking bootstrap into generated snapshot HTML when WP Visitor Stats is installed — same nonce and `log_visitor_data` endpoint as normal pages.

**Why:** Static MEXC HTML views still need first-party analytics without duplicating a second tracker implementation.

**Who benefits:** Operator measuring traffic on MEXC tool pages; **not** a dependency of the trading backend on SOL.

---

## Sessions and engagement metrics

### Session cookie

**What:** Each browser receives a `wp_visitor_session` cookie; visits with the same session ID roll into session aggregates (`visitor_sessions` table) with entry/exit pages and duration.

**Why:** Distinguish bounce vs multi-page exploration without treating every hit as a new “visitor” in narrative reports.

**Who benefits:** Marketing evaluating landing-page quality; operator spotting stuck sessions during incidents.

### Time on page and scroll depth

**What:** After the initial `log_visitor_data` response returns `visit_id`, exit handlers send `log_page_exit` with seconds on page; scroll milestones optionally call `log_scroll_depth`.

**Why:** Raw hit counts miss engagement — a popular page with 3-second median time tells a different story than a guide with minutes.

**Who benefits:** Content team prioritizing rewrites; product validating whether users reach below-the-fold CTAs.

### Page stats rollups

**What:** Daily aggregates per URL (`visitor_page_stats`) update on each logged visit — views, uniques, average time, bounce/exit rates for Content Analysis charts.

**Why:** Speeds admin queries on busy sites instead of scanning millions of raw rows for every chart load.

**Who benefits:** Operator opening Content Analysis during traffic spikes.

---

## Custom events (developer contract)

**What:** Front-end code calls `wpVisitorStatsTrackEvent(name, category, label, value)` which POSTs to `log_custom_event` with the same nonce as page views. Rows land in `visitor_custom_events` linked by session and optional `visit_id`.

**Why:** Page views alone cannot measure “clicked Buy”, “opened advanced chart”, or “saw error banner” — events carry product semantics.

**Who benefits:** Engineers instrumenting tools; operator confirming event volume in Custom Events after releases.

**Contract:** Event names should be stable snake_case or dotted identifiers; avoid PII in labels. Details in private `ARCHITECTURE.md`.

---

## URL shortener operations

### Creating links

**What:** Admin enters slug (alphanumeric + hyphen/underscore), target URL, optional title. Public URL is `{site}/go/{slug}`.

**Why:** Branded short links on the same domain inherit TLS and trust; click stream stays beside visit analytics.

**Who benefits:** Marketing sharing Telegram/Twitter links; operator auditing which promo drove clicks.

### Click logging

**What:** Each redirect writes one row to `visitor_link_clicks` with IP, geo, referrer, parsed browser/OS/device, bot flag — then 301 to destination.

**Why:** Detect bot inflation on campaigns; compare referrers per link in link stats modal.

**Who benefits:** Marketing post-mortems; operator disabling `is_active` on abused slugs without deleting history.

---

## Privacy and secrets

This overview intentionally omits API keys, internal hostnames, and excluded-IP lists. Geo lookups use server-side services configured in the private repo; operators manage exclusions and retention in wp-admin Settings.

---

## Related repositories

| Repo | Visibility |
|------|------------|
| [wp-visitor-stats-plugin](https://github.com/logicencoder/wp-visitor-stats-plugin) | private — source + `ARCHITECTURE.md` |
| [wp-visitor-stats-plugin-overview](https://github.com/logicencoder/wp-visitor-stats-plugin-overview) | public — this document |

**Made by [logicencoder](https://github.com/logicencoder)**
