# ConvRadar MCP Server — Complete Functionality Reference

> **Purpose of this document.** Exhaustive English-language reference for every tool, prompt, resource, diagnostic algorithm, and operational behaviour exposed by the ConvRadar MCP server. Designed to be shared with LLMs as context so they can reason about the server's capabilities without access to the source code.
>
> **Endpoint:** `https://mcp.convradar.com/mcp`
> **Transport:** Streamable HTTP (MCP 2025-06-18)
> **Auth:** OAuth 2.1 with PKCE (RFC 9728 discovery at `/.well-known/oauth-protected-resource`)

---

## Table of Contents

1. [Architecture Overview](#1-architecture-overview)
2. [Authentication & Authorization](#2-authentication--authorization)
3. [Tools — Read-Only Analytics (Tier 1)](#3-tools--read-only-analytics-tier-1)
4. [Tools — Diagnostic Engine (Tier 2)](#4-tools--diagnostic-engine-tier-2)
5. [Tools — Flexible Querying](#5-tools--flexible-querying)
6. [Tools — Traffic Quality Assessment](#6-tools--traffic-quality-assessment)
7. [Tools — Hypothesis Library (Tier 3)](#7-tools--hypothesis-library-tier-3)
8. [Tools — Verification, Web Fetch & Visual Proof](#8-tools--verification-web-fetch--visual-proof)
9. [Tools — Page Speed & UX Checks](#9-tools--page-speed--ux-checks)
10. [Tools — User-State Writes (Gated)](#10-tools--user-state-writes-gated)
11. [Tools — Cross-Session Continuity](#11-tools--cross-session-continuity)
12. [Tools — Full Audit (Golden First-Run)](#12-tools--full-audit-golden-first-run)
13. [Prompts](#13-prompts)
14. [Resources](#14-resources)
15. [Diagnostic Engine Internals](#15-diagnostic-engine-internals)
16. [Benchmark Data](#16-benchmark-data)
17. [Hypothesis Matching Engine](#17-hypothesis-matching-engine)
18. [Rate Limiting](#18-rate-limiting)
19. [Audit Logging](#19-audit-logging)
20. [Response Envelope](#20-response-envelope)
21. [Operational Notes](#21-operational-notes)

---

## 1. Architecture Overview

ConvRadar is a hosted MCP server that connects to a user's Google Analytics 4 property (read-only) and exposes 32 conversion-diagnostic tools to any MCP-compatible client (Claude, ChatGPT, Cursor, Cline, MCP Inspector).

**Stack:**
- **Runtime:** Python (Starlette + FastMCP), deployed on Render
- **Data store:** Supabase (PostgreSQL) — GA4 fact tables, hypothesis library, user state
- **Data source:** GA4 Data API (read-only, synced nightly by a background worker)
- **Page speed:** Google PageSpeed Insights (Lighthouse lab data; CrUX field data when available)
- **Visual proof:** anti-bot screenshot capture via a remote scraping browser, plus a vision-model page review
- **Billing:** Free during open beta (subscription gate bypassed, no card). Post-beta: Stripe, $9.99/month flat with a 7-day trial.
- **Auth:** OAuth 2.1 with PKCE (RS256 JWTs), legacy Supabase HS256 fallback, path-embedded connector tokens

**Middleware chain (order matters):**
1. CORS (browser preflight for claude.ai/claude.com)
2. AuthMiddleware — resolves tenant from Bearer JWT or path token (`/mcp/u_<token>`)
3. RateLimitMiddleware — per-tenant minute/day counters

**Tenant isolation:** Every query is scoped to `tenant_id` + `property_id` at the data-access layer. There is no cross-tenant data leakage path.

---

## 2. Authentication & Authorization

### OAuth 2.1 Flow (Primary)
1. Client calls `GET /oauth/authorize` with PKCE (`code_challenge`, `code_challenge_method=S256`), `client_id`, `redirect_uri`, `scope`, `state`, `resource`.
2. Server redirects to consent page on convradar.com.
3. User approves; web app calls `POST /internal/oauth/issue-code` (server-to-server, protected by `X-Internal-Secret`).
4. Client exchanges code at `POST /oauth/token` with `code_verifier` → receives `access_token` (RS256 JWT, 1h TTL) + `refresh_token` (30d, rotate-on-use).
5. Subsequent MCP calls include `Authorization: Bearer <access_token>`.

### Path-Token Auth (Legacy)
URL shape: `https://mcp.convradar.com/mcp/u_<token_id>` — the token is an auth artefact, not part of MCP routing. The middleware rewrites the path to `/mcp` and resolves the tenant from `connector_tokens`.

### Scopes
- `read:metrics` — all read-only tools
- `write:hypotheses` — user-state write tools (when enabled)

### Subscription Gate
The subscription check runs inside `resolve_context()` (not at the transport level) so the MCP connection succeeds and individual tool calls return a friendly text message telling the user how to fix their billing. `FREE_BETA=true` bypasses the status check (tenant must still exist). During the current open beta, `FREE_BETA=true` is set in production, so the gate is bypassed and every tool is free.

### Discovery
- `GET /.well-known/oauth-protected-resource` — RFC 9728 Protected Resource Metadata
- `GET /.well-known/oauth-authorization-server` — OAuth Authorization Server Metadata
- `GET /.well-known/jwks.json` — Public key set for RS256 token verification
- `GET /.well-known/mcp/server-card.json` — SEP-1649 static server descriptor (tool catalog for registries)

---

## 3. Tools — Read-Only Analytics (Tier 1)

All Tier 1 tools are always enabled, read-only, and return structured JSON inside the standard response envelope (see [Response Envelope](#20-response-envelope)).

### `cr_get_account_info`

**Purpose:** Return metadata for the connected GA4 property.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| *(none)* | — | — | No parameters |

**Returns:**
- `property_id` — GA4 property identifier
- `display_name` — human-readable property name
- `website_url` — the property's primary URL (used for constructing page URLs for verification)
- `timezone` — reporting timezone
- `currency_code` — reporting currency
- `industry_category` — GA4 industry
- `vertical` — ConvRadar vertical classification (e.g. `dtc_apparel`, `saas`, `ecommerce_general`)

---

### `cr_get_overview_metrics`

**Purpose:** Headline KPIs for a date window with automatic prior-period comparison.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `date_from` | string (ISO) | No | 30 days ago | Start date |
| `date_to` | string (ISO) | No | yesterday | End date |

**Returns:** `sessions`, `total_users`, `new_users`, `engaged_sessions`, `engagement_rate`, `bounce_rate`, `avg_session_duration`, `screen_page_views_per_session`, `purchases`, `purchase_revenue`, `conversion_rate` — each with `current`, `prior`, `delta`, `delta_pct` for automatic period comparison. The prior window is the same duration immediately preceding `date_from`.

**Date window:** Max 90 days. Default 30 days.

---

### `cr_get_traffic_breakdown`

**Purpose:** Top traffic sources by sessions, conversions, or revenue.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `date_from` | string (ISO) | No | 30 days ago | Start date |
| `date_to` | string (ISO) | No | yesterday | End date |
| `top_n` | int | No | 10 | Max sources to return (1–50) |
| `sort_by` | string | No | `"sessions"` | Sort column: sessions, purchases, purchase_revenue, conversion_rate |

**Returns:** List of `{session_source, session_medium, sessions, engaged_sessions, purchases, purchase_revenue, conversion_rate, bounce_rate, ...}` with prior-period deltas.

---

### `cr_get_device_breakdown`

**Purpose:** Sessions and conversions by device category.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `date_from` | string (ISO) | No | 30 days ago | Start date |
| `date_to` | string (ISO) | No | yesterday | End date |

**Returns:** Rows for `mobile`, `desktop`, `tablet` (and `smart tv` if present) with sessions, purchases, conversion_rate, bounce_rate, engagement_rate, avg_session_duration, share of total. Prior-period deltas included.

---

### `cr_get_landing_pages`

**Purpose:** Top landing pages ranked by traffic volume or conversion.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `date_from` | string (ISO) | No | 30 days ago | Start date |
| `date_to` | string (ISO) | No | yesterday | End date |
| `top_n` | int | No | 10 | Max pages to return (1–50) |
| `sort_by` | string | No | `"sessions"` | Sort column |

**Returns:** List of `{landing_page, sessions, purchases, purchase_revenue, conversion_rate, bounce_rate, engagement_rate, avg_session_duration}` with prior-period deltas.

---

### `cr_get_funnel`

**Purpose:** Step-by-step conversion funnel with ranked leak points.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `date_from` | string (ISO) | No | 30 days ago | Start date |
| `date_to` | string (ISO) | No | yesterday | End date |
| `steps` | list[string] | No | auto-detect | Funnel step names (e.g. `["session_start", "view_item", "add_to_cart", "begin_checkout", "purchase"]`) |
| `top_leaks` | int | No | 3 | How many leak points to highlight |

**Returns:** Ordered funnel steps with `{step_name, count, drop_rate, drop_count}` and a `leaks` array ranking the largest drop-off points by absolute volume lost. Prior-period comparison included.

---

### `cr_get_geo_breakdown`

**Purpose:** Top countries by sessions, conversions, or revenue.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `date_from` | string (ISO) | No | 30 days ago | Start date |
| `date_to` | string (ISO) | No | yesterday | End date |
| `top_n` | int | No | 10 | Max countries to return (1–50) |
| `sort_by` | string | No | `"sessions"` | Sort column |

**Returns:** List of `{country, sessions, purchases, purchase_revenue, conversion_rate, bounce_rate}` with prior-period deltas.

---

### `cr_get_product_performance`

**Purpose:** Per-product views, add-to-cart, purchases, and revenue.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `date_from` | string (ISO) | No | 30 days ago | Start date |
| `date_to` | string (ISO) | No | yesterday | End date |
| `top_n` | int | No | 10 | Max products to return (1–50) |
| `sort_by` | string | No | `"item_views"` | Sort column |
| `category` | string | No | all | Filter by product category |

**Returns:** List of `{item_name, item_id, item_category, item_views, add_to_carts, purchases, item_revenue, view_to_cart_rate, cart_to_purchase_rate}` with prior-period deltas.

---

### `cr_compare_segments`

**Purpose:** Side-by-side comparison of two segment values across all metrics.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `dimension` | string | **Yes** | — | Dimension to split on: `device_category`, `country`, `session_source_medium`, `landing_page` |
| `value_a` | string | **Yes** | — | First segment value (e.g. `"mobile"`) |
| `value_b` | string | **Yes** | — | Second segment value (e.g. `"desktop"`) |
| `date_from` | string (ISO) | No | 30 days ago | Start date |
| `date_to` | string (ISO) | No | yesterday | End date |

**Returns:** Two rows (one per segment) with all available metrics. Difference and percentage-difference computed for each metric.

---

### `cr_get_product_analysis`

**Purpose:** Pre-computed product diagnosis: which products are underperforming vs category peers.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `top_n` | int | No | 5 | Number of products to analyze |
| `status` | string | No | `"all"` | Filter: `"underperforming"`, `"overperforming"`, `"all"` |

**Returns:** List of products with `{item_name, diagnosis, view_to_cart_rate, cart_to_purchase_rate, revenue_share, category_avg_view_to_cart, gap_vs_category}`.

---

## 4. Tools — Diagnostic Engine (Tier 2)

### `cr_find_conversion_anomalies`

**Purpose:** Statistically significant anomalies in time-series metrics using rolling z-score detection.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `date_from` | string (ISO) | No | 30 days ago | Start date |
| `date_to` | string (ISO) | No | yesterday | End date |
| `metrics` | list[string] | No | `["sessions", "purchases", "purchase_revenue"]` | Metrics to scan |

**Algorithm:** 28-day rolling baseline with weekly seasonality adjustment. Z-score thresholds: `|z| ≥ 2.5` = notable, `|z| ≥ 3.5` = severe. Both spikes and drops are flagged. Minimum 7 baseline points required. See [Diagnostic Engine Internals](#15-diagnostic-engine-internals).

**Returns:** List of `Anomaly` objects with `{date, metric, value, baseline_mean, baseline_std, z_score, severity, direction}`. Anomalies are grouped by metric.

**Caching:** Results can be pre-computed by a nightly cron job (`precompute_anomalies.py`); the tool falls back to live computation on cache miss.

**Hypothesis matching:** Each anomaly is converted to a `Finding` and run through the hypothesis matching engine. Matched hypotheses are included in the response.

---

### `cr_diagnose_funnel_drop`

**Purpose:** When a metric drops, identify which segments (device, source, country, landing page) contributed most to the loss.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `date_from` | string (ISO) | No | 30 days ago | Start date (current window) |
| `date_to` | string (ISO) | No | yesterday | End date (current window) |
| `metric` | string | No | `"conversion_rate"` | Metric to diagnose: `conversion_rate`, `revenue`, `purchases` |
| `top_n_per_dimension` | int | No | 5 | Top contributors per dimension |

**Algorithm:** For each of 4 dimensions (device_category, session_source_medium, country, landing_page):
1. Aggregate prior and current window per segment value
2. Compute each segment's contribution to the overall metric delta, weighted by traffic share
3. Rank by absolute contribution (favors "where most loss came from" over "biggest relative drop")

**Returns:** `{findings: [{dimension, segment, prior_metric, current_metric, delta, contribution, weight}], per_dimension_summary, matched_hypotheses}`.

**Hypothesis matching:** Each `Finding` (kind=`segment_drop`) is run through the matching engine.

---

### `cr_compare_to_benchmark`

**Purpose:** Compare the property's metrics against industry vertical benchmarks.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `date_from` | string (ISO) | No | 30 days ago | Start date |
| `date_to` | string (ISO) | No | yesterday | End date |
| `vertical` | string | No | auto-detect | Industry vertical (see [Benchmark Data](#16-benchmark-data)) |
| `metrics` | list[string] | No | all available | Which metrics to compare |

**Returns:** For each metric: `{metric, value, p25, p50, p75, percentile_band, gap_to_median, relative_to_median, source}`.

**Percentile bands:** `below_p25`, `p25_to_p50`, `p50_to_p75`, `above_p75`.

---

### `cr_detect_traffic_quality_change`

**Purpose:** Detect step-changes in traffic composition or quality over a window.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `date_from` | string (ISO) | No | 30 days ago | Start date |
| `date_to` | string (ISO) | No | yesterday | End date |

**Returns:** Two types of findings:
- `traffic_anomaly` — a specific source/medium had a statistically significant spike or drop in sessions
- `mix_shift` — the share of traffic from a source/medium changed significantly (e.g., paid traffic went from 20% to 40% of total)

Each finding includes hypothesis matches.

---

## 5. Tools — Flexible Querying

### `cr_describe_data`

**Purpose:** Return a plain-English schema of available fact tables, dimensions, and metrics for the connected property.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| *(none)* | — | — | No parameters |

**Returns:** List of fact tables with their grain columns (dimensions) and metric columns. Each entry includes:
- `fact_table` — table name (e.g. `ga4_fact_traffic_daily`, `ga4_fact_geo_daily`)
- `grain_columns` — dimensions available for grouping
- `metric_columns` — metrics available for aggregation
- `dimension_filter` — if the table is restricted to specific event names or values

**Use case:** Call this before `cr_query_metrics` to discover what's queryable.

---

### `cr_query_metrics`

**Purpose:** Ad-hoc aggregation over any GA4 fact table — pick metrics + optional dimensions and the tool finds the right fact table, runs the query, and aggregates.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `metrics` | list[string] | **Yes** | — | Metric column names (e.g. `["sessions", "purchase_revenue"]`) |
| `dimensions` | list[string] | No | none (grand total) | Dimension columns to group by (e.g. `["date"]`, `["device_category", "country"]`) |
| `date_from` | string (ISO) | No | 30 days ago | Start date |
| `date_to` | string (ISO) | No | yesterday | End date |
| `filters` | dict | No | none | `{dimension: value}` pairs to filter before aggregation |
| `top_n` | int | No | 25 | Max rows to return after grouping (1–100) |
| `sort_by` | string | No | first metric | Metric to sort by (descending) |

**Fact table selection:** The tool automatically picks the smallest-grain fact table that covers all requested dimensions AND metrics. Prefers unfiltered tables over filtered ones.

**Aggregation:** Additive metrics are summed; ratio metrics (conversion_rate, bounce_rate, engagement_rate, etc.) are weighted-averaged by their natural denominator (e.g., conversion_rate weighted by sessions).

**Returns:** `{items: [{dimension_values..., metric_values...}], fact_table, dimensions, metrics, filters, sort_by, top_n}`.

---

## 6. Tools — Traffic Quality Assessment

### `cr_assess_traffic_quality`

**Purpose:** Score how much of the property's traffic looks like analytics noise (referral spam, internal/test traffic, datacenter sources, low-quality paid) and estimate how that noise distorts headline conversion rate and revenue.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `date_from` | string (ISO) | No | 28 days ago | Start date |
| `date_to` | string (ISO) | No | yesterday | End date |
| `comparison_days` | int | No | 28 | Baseline window in days |
| `sensitivity` | string | No | `"conservative"` | `"conservative"` (threshold 75), `"balanced"` (60), `"aggressive"` (45) |
| `top_n` | int | No | 10 | Max suspicious segments to return |
| `min_sessions` | int | No | 50 | Min sessions for a source to be scored |

**Algorithm:** For each source/medium/campaign combination:
1. Build a `TrafficSegment` with engagement metrics, purchases, revenue, and comparison-window data
2. Score across multiple signals: volume spike, engagement abnormality, zero-value pattern, source/referrer reputation, user-session pattern, geo-language mismatch, landing page concentration
3. Classify as: `referral_spam`, `internal_test`, `datacenter_bot`, `low_quality_paid`, `suspicious_organic`, `clean`
4. Compute cleaned metrics (CR and revenue) by removing suspicious sessions

**Returns:**
```json
{
  "summary": {
    "verdict": "noise_detected" | "clean" | "insufficient_data",
    "estimated_noise_sessions": 1234,
    "estimated_noise_share": 0.15,
    "raw_conversion_rate": 0.021,
    "cleaned_estimated_conversion_rate": 0.025,
    "cr_distortion_pct": -16,
    "confidence": "high" | "medium" | "low"
  },
  "top_suspicious_segments": [
    {
      "segment_label": "bot.example / referral",
      "noise_score": 87,
      "classification": "referral_spam",
      "evidence": ["source on known referrer-spam list", ...],
      "recommendation": "Add regex filter in GA4..."
    }
  ]
}
```

**Minimum data:** Requires ≥1,000 sessions in the window for a reliable assessment.

**Persistence:** Results are written to `traffic_noise_runs` table for historical tracking.

---

## 7. Tools — Hypothesis Library (Tier 3)

### `cr_list_hypotheses`

**Purpose:** Browse the verified hypothesis catalog. Filter by CRO category or industry vertical.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `category` | string | No | all | CRO category (e.g. `"pdp_information"`, `"checkout_friction"`, `"trust_signals"`, `"mobile_checkout"`) |
| `vertical` | string | No | auto-detect | Industry vertical filter |
| `top_n` | int | No | 10 | Max hypotheses to return |

**Returns:** List of `{id, title, category, description, applicable_verticals, capture_set_id, expected_impact}`.

If no results, returns `available_categories` so Claude can retry with a valid category name.

---

### `cr_get_hypothesis`

**Purpose:** Full detail for one hypothesis by ID.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `hypothesis_id` | string | **Yes** | Hypothesis ID (e.g. `"H-PDP-001"`) |

**Returns:**
- `id`, `title`, `category`, `description`
- `applicable_verticals` — which industry verticals this hypothesis applies to
- `capture_set` — which pages need to be inspected to verify this hypothesis
- `inspection_targets` — specific elements to look for on the page
- `conclusion_rules` — rule definitions that convert observables into verdicts
- `expected_impact` — `{metric, min_pct, max_pct, confidence, evidence_basis}`
- `ab_test_design` — `{primary_metric, min_sample_per_arm, ...}`
- `remediation_prototype_hint` — description of the fix for prototyping
- `source_kb_rows` — provenance (which knowledge-base entries backed this hypothesis)
- `attached_to_triggers` — which diagnostic triggers map to this hypothesis

---

## 8. Tools — Verification, Web Fetch & Visual Proof

### `cr_capture_via_web_fetch`

**Purpose:** Fetch a URL server-side and return its raw HTML for observable extraction. Always enabled (not gated by `MCP_ENABLE_WRITE_TOOLS`).

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `capture_set_id` | string | **Yes** | ID grouping captures into a verification batch (e.g. `"CS-pdp-desktop"`) |
| `url` | string | **Yes** | Absolute URL to fetch |

**Behaviour:**
1. UTM parameters are stripped before caching
2. Check cache: if a fresh (unexpired, 7-day TTL) verification exists for `(capture_set_id, url)`, return cached observables
3. Otherwise: `httpx.get(url)` with 8s timeout, `User-Agent: ConvRadar/1.0 (+verification)`
4. HTML truncated to 200KB if larger
5. Returns raw HTML + the capture set's observable definitions (keys to extract)

**Limitations (critical for accuracy):**
- Cannot see JS-rendered DOM (SPA content injected after hydration)
- Cannot see mobile-specific layout (responsive sites return identical markup; only media queries differ)
- Cannot see cross-step flow (modals, walls, gates between funnel steps)
- **Rule:** Absence of an element from raw HTML is inconclusive, not proof of absence

---

### `cr_capture_screenshots`

**Purpose:** Capture real **desktop + mobile screenshots** of a page through an anti-bot remote browser so the model can *see* the page (above-the-fold content, mobile layout) before drawing conclusions — the visual companion to `cr_capture_via_web_fetch`, which only sees raw HTML.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `url` | string | **Yes** | — | Absolute URL of the problematic page (e.g. the PDP where add-to-cart drops, or the checkout step flagged by `cr_diagnose_funnel_drop`) |
| `page_type` | string | No | — | Targets the element crop + observables: `homepage`, `pdp`, `cart`, `checkout_step`, `landing_page`, `category_page`, `off_site` |
| `capture_set_id` | string | No | auto-resolved from `page_type` | Capture set (`CS-*`) whose observables define what to look for |
| `force_refresh` | bool | No | `false` | Re-capture even if a fresh (<7 day) screenshot exists |

**Behaviour:**
1. **Cache-first:** a fresh (7-day TTL) screenshot for the URL is served immediately.
2. **Inline path** (remote scraping browser): returns legible scroll-section images in the same call, plus a **signed URL of the real "before" screenshot** for use in before/after mockups.
3. **Async path:** returns `{status: "capturing", request_id, retry_after_s: 30}` — poll `cr_get_screenshots(request_id)`.
4. Every payload carries a **verdict contract**: for each observable the model must state observed / not-observed / unclear, say which section shows it, and quote the visual evidence — no visual claim without something to point to.
5. Closing the loop: `cr_record_verification(method='screenshot', ...)` converts extracted observables into hypothesis verdicts; the user-facing deliverable is a before/after HTML artifact with numbered, tooltipped annotations.
6. If capture is unconfigured or fails, the tool instructs the model to ask the user for a screenshot — and to never claim it has seen the page.

---

### `cr_get_screenshots`

**Purpose:** Fetch the captured screenshots for a `request_id` returned by `cr_capture_screenshots`. Read-only.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `request_id` | string | **Yes** | ID from `cr_capture_screenshots` |

**Statuses:** `capturing` (retry in ~15s) → `ready` (desktop + mobile + element-crop images, plus the observable checklist and verdict contract) → or `failed` (site is bot-protected; fall back to asking the user for a screenshot).

---

### `cr_list_capture_sets`

**Purpose:** List the capture sets a page can be verified against — each with its `CS-*` id, target page type, and the exact observable keys to extract from a screenshot. Pick the matching set before `cr_record_verification`: `CS-landing-page` (lead-gen/landing), `CS-pdp-desktop` / `CS-pdp-mobile` (product page), `CS-cart`, `CS-checkout-flow`, `CS-homepage`. No parameters. Read-only.

---

## 9. Tools — Page Speed & UX Checks

Both tools measure the **live public site**, so they are exempt from the data-import gate — they work for brand-new tenants whose GA4 backfill is still running, and for app-only properties (pass the marketing-site URL).

### `cr_check_page_speed`

**Purpose:** Measure a page's Core Web Vitals via Google PageSpeed Insights and join the result to the property's own GA4 conversion data.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `url` | string | No | property site root (derived from GA4 `page_location` data) | Absolute URL to measure |
| `page_type` | string | No | — | Page role for context: `home`, `landing`, `category`, `pdp`, `cart`, `checkout`, `other` |

**Behaviour:** cache-first (7-day freshness); on a miss, measures **mobile live** (~10–30s) and persists the snapshot; desktop is back-filled by a weekly job. Lab (Lighthouse) numbers by default; real-user (CrUX field) data is used and labeled when available — small sites usually have none, so a "lab" label is normal.

**Returns:**
- `mobile` — `perf_score` (0–100) + grade (`good` / `needs-improvement` / `poor`), `lcp_ms`, `cls`, `tbt_ms`, `inp_ms`, page weight (`total_bytes_kb`), `dom_elements`, `resource_summary`, `third_party_count`, and the `lcp_element` itself
- `desktop` — score + LCP when already measured
- `top_opportunities` — top-3 Lighthouse fixes with estimated savings
- `conversion_cost` — when GA4 has enough data: the mobile-vs-desktop conversion gap and the monthly revenue (or conversions) slow mobile load likely contributes to
- `observables_for_verification` — pre-shaped for `cr_record_verification`, so the page-speed hypothesis (`H-NAV-001`) can be recorded in one hop

Requires `PAGESPEED_API_KEY` on the instance; otherwise returns `status: not_configured`.

---

### `cr_heuristic_check`

**Purpose:** One-call **page speed + UX review with no GA4 required**: measures the page via PageSpeed Insights AND reads a mobile screenshot with a vision model against the conversion hypothesis library. Returns the speed grade, specific *hedged* "likely issues" (each with a grounded fix), and a free-form AI page review — all from the live page.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `url` | string | No | the property's main page | Absolute URL to check; an explicit URL always checks exactly one page. For app-only properties, pass the marketing/landing site URL |
| `max_pages` | int | No | `1` | 1–3. Raise to also check the next most-visited / leakiest pages (each adds ~30–60s); ignored when `url` is given |

**Page selection (when GA4 data exists):** pages are **traffic-ranked by lost sessions** (sessions × bounce), surfacing pages that are both popular *and* leaky; high-intent ecommerce pages the funnel resolves (PDP / cart / checkout) are kept. The response then also flags where the page actually leaks.

**Honest-by-design contract:** measured speed is a fact; visual issues are hedged hypotheses behind a confidence gate; no revenue figure is invented without GA4 funnel data.

---

## 10. Tools — User-State Writes (Gated)

These tools are gated behind `MCP_ENABLE_WRITE_TOOLS=true` (default: off). They modify user state (hypotheses, change journal, verifications) — never the GA4 property itself. **Currently enabled in production** during the open beta, so connected clients see the full 32-tool surface.

### `cr_record_verification`

**Purpose:** Submit observables Claude extracted from a page (HTML, screenshot, or browser tool). Evaluates conclusion rules against observables and upserts user_hypotheses with verdicts.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `capture_set_id` | string | **Yes** | — | Capture batch ID |
| `url` | string | **Yes** | — | URL that was verified |
| `observables_extracted` | dict | **Yes** | — | `{observable_key: value}` pairs |
| `method` | string | No | `"user_screenshot"` | One of: `web_fetch`, `user_screenshot`, `claude_browser`, `cached`, `web_search`, `ga4_facts` |
| `quality` | string | No | `"complete"` | One of: `complete`, `partial`, `failed`, `insufficient` |
| `notes` | string | No | — | Free-form notes |

**Behaviour:** For each hypothesis using this capture_set, applies conclusion_rules to observables → produces a verdict (`confirmed`, `rejected`, `rejected_with_note`, `inconclusive`) → upserts `user_hypotheses` row.

**Returns:** `{verification_id, verdicts: {hypothesis_id: {title, verdict, severity, message, missing_observables}}, confirmed_count, rejected_count}`.

---

### `cr_mark_hypothesis_status`

**Purpose:** Change the status of a hypothesis for this tenant.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `hypothesis_id` | string | No* | — | Required when `source='library'` |
| `status` | string | No | `"surfaced"` | One of: `surfaced`, `testing`, `confirmed`, `rejected`, `dismissed`, `archived` |
| `note` | string | No | — | One-line rationale |
| `source` | string | No | `"library"` | `"library"` or `"ai_suggested"` |
| `ai_suggested_payload` | dict | No | — | Required when `source='ai_suggested'` |

---

### `cr_save_ai_suggested`

**Purpose:** Persist an AI-generated hypothesis when no library match exists for a finding.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `finding_id` | string | No | ID of the diagnostic finding this hypothesis is tied to |
| `payload` | dict | **Yes** | Must include: `title`, `description`, `expected_impact_metric`, `expected_impact_min_pct`, `expected_impact_max_pct`, `suggested_test` |

**Rule:** AI-suggested hypotheses must be surfaced in chat with the marker: "Not in our verified library — based on general CRO principles."

---

### `cr_log_change`

**Purpose:** Log a change the user shipped (copy edit, design change, pricing tweak, A/B test). Captures `deployed_at` so `cr_verify_change_impact` can compute pre/post metrics later.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `description` | string | **Yes** | — | What was changed |
| `change_type` | string | **Yes** | — | One of: `design`, `copy`, `pricing`, `flow`, `infrastructure`, `experiment`, `other` |
| `affected_segment` | string | No | sitewide | e.g. `"mobile-checkout"` |
| `hypothesis_id` | string | No | — | Link to the hypothesis being tested |
| `deployed_at` | string (ISO) | No | now | When the change went live |

---

### `cr_verify_change_impact`

**Purpose:** Compute pre/post metrics for a logged change and produce a statistical verdict.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `change_id` | string | **Yes** | — | ID from `cr_log_change` |
| `days_after` | int | No | 14 | Days post-deploy to measure (1–60) |

**Algorithm:**
- Pre window: 7 days before deployment
- Post window: `days_after` days starting 1 day after deployment
- Two-proportion z-test on conversion_rate
- Verdict: `improved` (p < 0.05, z > 0), `regressed` (p < 0.05, z < 0), `no_change`, `insufficient_data`

**Returns:** `{pre: {sessions, purchases, revenue, conversion_rate}, post: {...}, deltas: {...}, z_score, p_value, verdict}`.

---

## 11. Tools — Cross-Session Continuity

### `cr_get_current_state`

**Purpose:** Per-tenant snapshot of what's already in flight — enables cross-session continuity so Claude doesn't need the user to re-explain context.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| *(none)* | — | — | No parameters |

**Returns:**
- `overview` — last 30-day metrics (sessions, purchases, revenue, conversion_rate)
- `active_user_hypotheses` — up to 10 hypotheses with status `surfaced`, `testing`, or `confirmed`, sorted by `marked_at` DESC
- `recent_changes` — last 5 change_journal entries
- `recent_findings` — up to 3 most recent findings (severity DESC)
- `markdown` — pre-rendered Markdown summary (kept under ~3K tokens)

**When to call:** At the start of any session, especially when the user asks "what's in progress", "what was I doing", "what's pending".

---

## 12. Tools — Full Audit (Golden First-Run)

### `cr_full_audit`

**Purpose:** Complete conversion audit in one call — composes 5–8 internal queries into 2–5 ranked actionable findings with next-step actions and matched hypotheses.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `date_from` | string (ISO) | No | 30 days ago | Start date |
| `date_to` | string (ISO) | No | yesterday | End date |

**What it runs internally:**
1. Overview metrics (current vs prior 30 days)
2. Funnel analysis (biggest leak)
3. Top 3 traffic sources
4. Top 3 landing pages
5. Anomaly detection
6. Segment drop diagnosis
7. Benchmark comparison

**Returns:**
- `headline_kpis` — sessions, purchases, revenue, conversion_rate with deltas
- `context` — biggest funnel leak, top traffic sources, top landing pages
- `findings` — 2–5 ranked findings, each with `title`, `evidence`, `confidence`, `next_action`, `matched_hypotheses`

**Trigger phrases:** "run a full audit", "audit my analytics", "where do I start", "what's wrong with my conversion", "first-time analysis", "complete check".

---

## 13. Prompts

MCP prompts are structured instruction templates that Claude reads to execute multi-step workflows. They reference tools but don't call them — Claude calls the tools based on the prompt's instructions.

### `/full_audit`
**Title:** ConvRadar — Full Audit
**Trigger:** "run an audit", "complete review", "what should I fix"
**Workflow:**
1. Call `cr_full_audit()` — gets headline KPIs, context, and 2–5 findings
2. Present the report: one paragraph with headline KPIs, then each finding with evidence and suggested next action
3. For any finding the user wants to act on: verify via `cr_capture_via_web_fetch` before recommending fixes
4. If mobile segment is dominant gap: request a mobile screenshot (HTML fetch can't see responsive layout differences)
5. If leak is between funnel steps: ask user about between-step blockers (sign-up walls, modals, age gates)
6. Produce a "fixed-version" mockup (required for UI fixes — textual recommendation alone fails the bar)
7. Save commitment via `cr_mark_hypothesis_status` for cross-session continuity

### `/weekly_checkup`
**Title:** ConvRadar — Weekly Checkup
**Trigger:** "checkup", "status", "what's changed this week"
**Workflow:**
1. `cr_get_overview_metrics` for last 7 days
2. Read `convradar://current-state` for active hypotheses and recent changes
3. `cr_find_conversion_anomalies` (most weeks returns nothing — that's worth reporting)
4. Prompt user about in-flight hypotheses
5. Verify recently shipped changes (≥14 days old) via `cr_verify_change_impact`
**Output:** Under 200 words: headline CR direction, anomalies, active hypotheses, change verdicts, one recommendation.

### `/diagnose_drop`
**Title:** ConvRadar — Diagnose Drop
**Trigger:** "my conversion rate dropped", "what's going on with mobile traffic", "where's the leak"
**Workflow:**
1. Confirm the metric and date window
2. `cr_diagnose_funnel_drop` — splits across device/source/country/landing_page
3. `cr_find_conversion_anomalies` — check if drop coincided with specific date(s)
4. Present top 1–2 matched hypotheses (or generate AI-suggested with marker text)
5. Offer to verify, tailored to the gap type (mobile → screenshot request, between-step → ask user)
**Output:** Under 250 words.

### `/suggest_next_test`
**Title:** ConvRadar — Suggest Next Test
**Trigger:** "what should I test next", "next experiment"
**Workflow:**
1. Read `convradar://current-state` for confirmed-but-unshipped hypotheses
2. If one exists, pull full detail via `cr_get_hypothesis`
3. Frame the test using `convradar://playbook/ab-test-design`: primary metric, MDE, sample-size guidance
4. If nothing confirmed, suggest running `/full_audit` first

### `/verification-policy`
**Title:** ConvRadar — Verification Policy
**Purpose:** Guardrails for turning tool output into recommendations. Referenced by every analytics tool description. Consolidates hard rules:
- **Required workflow:** Get URL → fetch with `cr_capture_via_web_fetch` → mobile gap = request screenshot → between-step = ask user → recommend citing page state
- **Cannot see from HTML:** JS-rendered content, mobile-specific layout, cross-step flow
- **Forbidden conclusions from HTML alone:** Cannot claim elements are missing; cannot make layout/position/size/color claims
- **After verification:** Must produce a mockup (artifact, SVG, HTML prototype)
- **Anomaly guidance:** Anomalies tell date and severity, not cause — pair with `cr_diagnose_funnel_drop` and ask user about recent changes
- **Exception:** If user says "give me unverified ideas", list candidates with "Unverified — pending page inspection" marker

---

## 14. Resources

MCP resources are passive data sources that Claude can read. They use custom URI schemes.

### `convradar://current-state`
**Type:** Dynamic per-tenant snapshot (Markdown)
**Content:** Last 30 days overview + active hypotheses + recent changes + recent findings. Same data as `cr_get_current_state` tool but as a resource. Budget-capped at ~3K tokens.

### `convradar://hypothesis-library`
**Type:** Paginated index (JSON)
**Content:** All hypotheses grouped by category — counts + titles only. Use per-category URI for full detail.

### `convradar://hypothesis-library/{category}`
**Type:** Full category detail (JSON)
**Content:** 5–15 hypotheses in one category with description, expected impact, applicable verticals, remediation hints.

### `convradar://benchmarks/{vertical}`
**Type:** Static benchmark data (JSON)
**Content:** p25/p50/p75 percentiles for headline metrics in one vertical. See [Benchmark Data](#16-benchmark-data).

### `convradar://playbook/funnel-diagnosis`
**Type:** Static methodology (Markdown)
**Content:** End-to-end methodology for diagnosing a conversion-funnel drop.

### `convradar://playbook/ab-test-design`
**Type:** Static methodology (Markdown)
**Content:** Methodology for designing a CRO A/B test: hypothesis statement, primary metric, MDE, sample-size formula, ramp + analysis.

---

## 15. Diagnostic Engine Internals

### Anomaly Detection (`diagnostic/anomaly.py`)

**Algorithm:** Rolling z-score on time-series data.

- **Baseline window:** 28 days (configurable)
- **Seasonality adjustment:** Weekly — subtracts the day-of-week mean from each point so a Saturday is compared against a Saturday-only baseline
- **Minimum baseline:** 7 points required; fewer → point is skipped
- **Thresholds:**
  - `|z| ≥ 2.5` → `notable`
  - `|z| ≥ 3.5` → `severe`
- **Direction:** Both `spike` and `drop` are flagged
- **Flat baseline:** If baseline standard deviation is 0 (flat line), point is skipped (can't compute meaningful z)

### Segment Analysis (`diagnostic/segments.py`)

**Algorithm:** "Find the leak" — given a measured drop, rank dimension values by contribution.

- **Contribution formula:** `contribution = metric_delta × max(prior_weight, current_weight)`
- **Design choice:** Favors "where most of the loss came from" over "which segment had the biggest relative drop". A 90% drop in a 1%-traffic segment scores lower than a 20% drop in a 60%-traffic segment.
- **Aggregation:** When numerator/denominator are provided (e.g., CR = purchases/sessions), metric is computed as ratio per segment. Otherwise reads directly.

### Finding Data Structure (`diagnostic/findings.py`)

```
Finding:
  kind: "anomaly" | "segment_drop" | "traffic_anomaly" | "mix_shift"
  metric: str
  severity: "notable" | "severe"
  confidence: "low" | "medium" | "high"
  direction: "spike" | "drop" | null
  dimension: str | null
  segment: str | null
  baseline_value, observed_value, magnitude_pct: float | null
  period: [ISO date strings]
  sample_size: int | null
  z_score: float | null
```

### Confidence Scoring

| Condition | Confidence |
|-----------|-----------|
| sample ≥ 1,000 AND (|z| ≥ 3.5 OR no z) AND has_baseline | `high` |
| sample ≥ 100 AND has_baseline | `medium` |
| Everything else | `low` |

---

## 16. Benchmark Data

Hardcoded vertical benchmarks from public industry reports. Metrics: `conversion_rate`, `aov`, `bounce_rate`, `engagement_rate`.

| Vertical | CR p25 | CR p50 | CR p75 | AOV p50 | Source |
|----------|--------|--------|--------|---------|--------|
| `ecommerce_general` | 1.2% | 2.2% | 3.4% | $68 | Littledata + IRP Commerce 2024 |
| `dtc_apparel` | 1.4% | 2.6% | 4.1% | $92 | Klaviyo DTC 2024 |
| `beauty_ecom` | 1.5% | 2.9% | 4.6% | $62 | Klaviyo Beauty 2024 |
| `saas` | 1.8% | 3.8% | 7.5% | n/a | Capterra B2B SaaS 2024 |
| `marketplace` | 2.2% | 3.5% | 5.2% | $42 | Marketplace composite 2024 |
| `subscription_ecom` | 2.0% | 3.8% | 5.8% | $58 | Recharge subscription 2024 |
| `ecom_electronics` | 0.8% | 1.5% | 2.5% | $210 | Statista + Adobe DI 2024 |

Default vertical: `ecommerce_general`.

---

## 17. Hypothesis Matching Engine

### How It Works

The matching engine is pure, deterministic, and rule-based (no ML). It maps a `Finding` to candidate hypotheses through a trigger catalog:

1. **Trigger catalog** (~35 rows in `triggers` table): Each trigger defines conditions — metric, direction, severity_min, dimension, segment, applicable_verticals.
2. **Trigger matching:** A trigger fires for a finding when ALL conditions match:
   - `metric`: exact match OR alias-equivalent (see below)
   - `direction`: `drop`, `spike`, or `any` (wildcard)
   - `severity_min`: `low` matches any; `medium` matches `notable`+`severe`; `high` matches `severe` only
   - `dimension`/`segment`: if pinned on trigger, must match finding
   - `applicable_verticals`: empty = all; otherwise must contain tenant's vertical (with alias bridging)
3. **Hypothesis lookup:** Matched triggers are joined via `trigger_hypotheses` to get candidate hypotheses.
4. **Hypothesis filtering:** Each hypothesis's own `applicable_verticals` is checked against the tenant's vertical.
5. **Ranking:** By trigger priority ASC, then expected_impact_max_pct DESC.

### Metric Aliases

The engine bridges different naming conventions between diagnostic tools and the hypothesis library:

- `conversion_rate` → also matches: `mobile_conversion_rate`, `desktop_conversion_rate`, `purchase_rate`, `pdp_to_cart_rate`, `add_to_cart_rate`, `cart_to_checkout_rate`, `checkout_to_purchase_rate`, `checkout_completion_rate`, `form_completion_rate`, `trial_to_paid_conversion_rate`, and 15+ more
- `revenue` → also matches: `purchase_revenue`, `total_revenue`, `revenue_per_session`, `average_order_value`, `aov`
- `purchases` → also matches: `purchase_count`, `completed_orders`, `conversion_count`
- `sessions` → also matches: `session_count`, `traffic_volume`, `bounce_rate`, `page_load_time_ms`, `traffic_quality_score`

### Vertical Aliases

The engine bridges naming differences between tenant settings and the hypothesis library:

- `dtc_apparel` ↔ `ecom_apparel` ↔ `ecom_general`
- `beauty_ecom` ↔ `ecom_beauty` ↔ `ecom_general`
- `saas` ↔ `saas_b2c` ↔ `saas_b2b`
- etc.

### HypothesisMatch Output

```
HypothesisMatch:
  id: str
  title: str
  category: str
  rationale: str (why this trigger fired)
  expected_impact: "X–Y% lift in metric (confidence)"
  suggested_test: str (A/B test design or remediation hint)
  capture_set_id: str (which pages to inspect)
  priority: int
  description: str
  inspection_targets: list
  confidence: str
```

---

## 18. Rate Limiting

Per-tenant, fixed-window rate limiting using Supabase counter rows.

| Limit | Default | Configurable via |
|-------|---------|-----------------|
| Per minute | 60 | `MCP_RATE_LIMIT_PER_MINUTE` |
| Per day | 1,000 | `MCP_RATE_LIMIT_PER_DAY` |

Set either to `0` to disable. Counter rows are cleaned up nightly (rows older than 2 days).

When a limit is hit, the middleware returns HTTP 429 with a `Retry-After` header.

---

## 19. Audit Logging

Every tool call is logged to `mcp_audit_log` via a SECURITY DEFINER RPC. Each row captures:
- `tenant_id`, `property_id` — who
- `tool_name`, `arguments_hash` — what
- `response_summary` — abbreviated result (not full payload)
- `duration_ms` — how long
- `created_at` — when

Toggle: `MCP_AUDIT_LOG_ENABLED=true` (default). 90-day retention enforced by nightly cleanup cron.

---

## 20. Response Envelope

All tools return a standardized JSON envelope:

```json
{
  "summary": "One-line plain-English summary of the result.",
  "data": { ... },
  "meta": {
    "property_id": "...",
    "currency": "USD",
    "timezone": "America/New_York",
    "property_display_name": "My Store",
    "date_from": "2026-04-21",
    "date_to": "2026-05-20"
  },
  "notes": ["Optional caveats or context."]
}
```

Empty results use `empty_response()` which returns `{"summary": "...", "data": {"items": []}, "meta": {...}, "notes": [...]}` with a human-readable reason.

---

## 21. Operational Notes

### Date Window Defaults
- Default: last 30 days (yesterday inclusive)
- Maximum: 90 days
- Prior period: same duration immediately before `date_from`

### Data Source
GA4 data is synced nightly by a background worker into `ga4_fact_*` tables. The MCP server reads these pre-aggregated fact tables, not the GA4 API directly. This means:
- Data is available with ~24h latency (yesterday's data is available today)
- No GA4 API quota concerns during tool calls
- Queries are fast (Supabase PostgreSQL, not API round-trips)

### Environment Variables (Key)
| Variable | Purpose |
|----------|---------|
| `SUPABASE_URL` | Supabase project URL |
| `SUPABASE_SERVICE_ROLE_KEY` | Bypasses RLS; tenant scoping enforced in application layer |
| `SUPABASE_JWT_SECRET` | Verifies legacy HS256 JWTs |
| `MCP_OAUTH_ISSUER` | OAuth 2.1 issuer URL (default: `https://mcp.convradar.com`) |
| `MCP_INTERNAL_SECRET` | Protects server-to-server endpoints |
| `MCP_ENABLE_WRITE_TOOLS` | Gate for user-state write tools (default: `false`) |
| `MCP_RATE_LIMIT_PER_MINUTE` | Per-tenant rate limit (default: 60) |
| `MCP_RATE_LIMIT_PER_DAY` | Per-tenant rate limit (default: 1000) |
| `MCP_AUDIT_LOG_ENABLED` | Toggle audit logging (default: `true`) |
| `PAGESPEED_API_KEY` | Google PageSpeed Insights key — enables `cr_check_page_speed` and the speed half of `cr_heuristic_check` |
| `PAGESPEED_DAILY_BUDGET` | Daily cap on live PSI measurements |
| `ANTHROPIC_API_KEY` / `HEURISTIC_VISION_MODEL` | Vision model behind the `cr_heuristic_check` AI page review |
| `BRIGHTDATA_*` | Anti-bot screenshot capture (scraping browser / web unlocker) for `cr_capture_screenshots` |
| `VISUAL_PROOF_FRESHNESS_DAYS` | Screenshot cache TTL in days (default: 7) |
| `FREE_BETA` | Bypass subscription gate (default: unset = paid mode) |

### Cron Jobs
| Cron | Schedule | Purpose |
|------|----------|---------|
| `precompute_anomalies` | Daily 04:00 UTC | Pre-compute anomaly cache for fast `cr_find_conversion_anomalies` |
| `cleanup_rate_limit` | Daily 04:30 UTC | Delete counter rows older than 2 days |
| `cleanup_audit_log` | Daily 05:30 UTC | 90-day retention for `mcp_audit_log` |
| `e2e_smoke` | Daily 06:00 UTC | End-to-end smoke test: magic link → OAuth → MCP tool call |

### DNS Rebinding Protection
The streamable HTTP transport rejects requests whose `Host` header isn't in the allowed list: `mcp.convradar.com`, `convradar-mcp.onrender.com`, `127.0.0.1:*`, `localhost:*`, `[::1]:*`.

### Public Paths (No Auth)
`/healthz`, `/health`, `/.well-known/*`, `/oauth/*`, `/internal/oauth/*`, `/internal/e2e-smoke`.

---

*Last updated: 2026-07-16. Source: ConvRadar MCP server codebase analysis.*
