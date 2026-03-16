# AdLoop — MCP Server for Google Ads + GA4

MCP server connecting Google Ads and Google Analytics (GA4) data for AI-driven marketing analysis.

## Setup

```bash
# Install
cd /path/to/adloop
uv sync

# First-time configuration (interactive wizard)
uv run adloop init

# Or configure manually: copy config.yaml.example to ~/.adloop/config.yaml
```

### Claude Code MCP connection

```bash
claude mcp add --transport stdio adloop -- uv run --directory /Users/thiago/Projects/adloop adloop
```

Or add to `.claude.json` / `.mcp.json`:
```json
{
  "mcpServers": {
    "adloop": {
      "type": "stdio",
      "command": "uv",
      "args": ["run", "--directory", "/Users/thiago/Projects/adloop", "adloop"]
    }
  }
}
```

### Google Cloud Prerequisites

1. Create a Google Cloud project
2. Enable APIs: Google Analytics Data API, Google Analytics Admin API, Google Ads API
3. Create OAuth consent screen (External, Testing mode OK) — add your email as test user
4. Create OAuth 2.0 Client ID (Desktop application) — download JSON
5. Get Google Ads Developer Token from MCC

## Tool Inventory (26 tools)

### Diagnostics

| Tool | Purpose |
|------|---------|
| `health_check` | Test OAuth, GA4, and Ads connectivity |

If health_check reports auth errors: delete `~/.adloop/token.json` and re-run any tool to trigger re-authorization. If tokens keep expiring weekly, the GCP consent screen needs to be published from "Testing" to "In production".

### GA4 Read (4)

| Tool | Purpose | Key Parameters |
|------|---------|----------------|
| `get_account_summaries` | List GA4 accounts and properties | (none — uses config) |
| `run_ga4_report` | Custom reports: sessions, users, conversions | `dimensions`, `metrics`, `date_range_start`, `date_range_end`, `limit` |
| `run_realtime_report` | Live data after deploys | `dimensions`, `metrics` |
| `get_tracking_events` | Configured events and volume | `date_range_start`, `date_range_end` |

### Google Ads Read (7)

| Tool | Purpose | Key Parameters |
|------|---------|----------------|
| `list_accounts` | Discover Ads accounts | (none — uses MCC from config) |
| `get_campaign_performance` | Campaign metrics (impressions, clicks, cost, CPA) | `date_range_start`, `date_range_end` |
| `get_ad_performance` | Ad copy analysis (headlines, descriptions, CTR) | `date_range_start`, `date_range_end` |
| `get_keyword_performance` | Keywords with quality scores | `date_range_start`, `date_range_end` |
| `get_search_terms` | Actual user search queries | `date_range_start`, `date_range_end` |
| `get_negative_keywords` | Existing negative keywords | `campaign_id` (optional) |
| `run_gaql` | Arbitrary GAQL queries | `query`, `format` (table/json/csv) |

Return format notes:
- Ads read tools automatically compute `metrics.cost` (EUR) and `metrics.cpa` from `metrics.cost_micros` — no manual division needed.
- `metrics.average_cpc_eur` is also pre-computed where available.
- `get_ad_performance` returns full `headlines` and `descriptions` lists for RSAs.

### Cross-Reference (3)

| Tool | Purpose | Key Parameters |
|------|---------|----------------|
| `analyze_campaign_conversions` | Ads clicks vs GA4 sessions vs conversions, GDPR gap detection | `date_range_start`, `date_range_end`, `campaign_name` (optional) |
| `landing_page_analysis` | Ad URLs vs GA4 page data, zero-conversion flags | `date_range_start`, `date_range_end` |
| `attribution_check` | Ads vs GA4 conversion discrepancies | `date_range_start`, `date_range_end`, `conversion_events` (optional) |

These tools call both APIs internally and return unified results with computed `insights[]`. Read-only. Each returns a `date_range` and auto-generates conditional warnings (GDPR gaps, zero conversions, attribution mismatches, orphaned URLs).

### Write (8) — all use draft > preview > confirm

| Tool | What It Does | Validation |
|------|-------------|------------|
| `draft_campaign` | Create full campaign structure preview | `campaign_name`, `daily_budget` (cap enforced), `bidding_strategy`, keywords validated |
| `draft_responsive_search_ad` | Create RSA preview (not published) | 3-15 headlines (<=30 chars), 2-4 descriptions (<=90 chars), `final_url` |
| `draft_keywords` | Propose keyword additions | Each: `text` + `match_type` (EXACT/PHRASE/BROAD) |
| `add_negative_keywords` | Propose negative keywords | `campaign_id`, keyword list, `match_type` |
| `pause_entity` | Propose pausing entity | `entity_type`, `entity_id` |
| `enable_entity` | Propose enabling paused entity | `entity_type`, `entity_id` |
| `remove_entity` | Propose REMOVING entity (irreversible) | `entity_type` (incl. "negative_keyword"), `entity_id` |
| `confirm_and_apply` | Execute previewed change | `plan_id`, `dry_run` (default true) |

### Tracking (2)

| Tool | Purpose | Key Parameters |
|------|---------|----------------|
| `validate_tracking` | Compare codebase events vs GA4 | `expected_events`, `date_range_start`, `date_range_end` |
| `generate_tracking_code` | Generate gtag JavaScript | `event_name`, `event_params`, `trigger` |

`validate_tracking` requires first searching the codebase for `gtag('event', ...)` or `dataLayer.push({event: ...})` calls, extracting event names, then passing them to the tool.

### Planning (1)

| Tool | Purpose | Key Parameters |
|------|---------|----------------|
| `estimate_budget` | Keyword Planner forecasts | `keywords` (list of {text, match_type, max_cpc}), `daily_budget`, `geo_target_id`, `language_id`, `forecast_days` |

Common geo targets: 2276=Germany, 2840=USA, 2826=UK. Common languages: 1000=English, 1001=German, 1002=French.

## Safety Rules (CRITICAL — always follow)

1. **NEVER call confirm_and_apply without showing preview first.** Always present the full preview from a draft_* tool and wait for explicit user approval.

2. **Default to dry_run=true.** Always use dry_run=true unless the user explicitly says to apply for real. `require_dry_run` in config may override this.

3. **Respect budget caps.** Never propose a campaign budget above `max_daily_budget` in config.

4. **Double-check destructive operations.** For pause, enable, remove, or budget changes, warn about impact. `remove_entity` is irreversible — prefer `pause_entity` unless user explicitly requests removal.

5. **One change at a time.** Don't batch multiple write operations. Draft one, get approval, apply, then next.

6. **Never guess entity IDs.** Always retrieve IDs from a read tool first.

7. **NEVER add BROAD match keywords without Smart Bidding.** Before `draft_keywords` with BROAD, check bidding strategy via `get_campaign_performance` or `run_gaql`. If MANUAL_CPC/MANUAL_CPM — REFUSE BROAD match. Use PHRASE or EXACT. Broad Match + Manual CPC is the #1 cause of wasted ad spend.

8. **Pre-write validation.** Before ANY write operation, check:
   - Bidding strategy (rule 7)
   - Conversion tracking active (zero conversions + high spend = fix tracking first)
   - Quality scores (all QS < 5 = fix landing page/relevance, not more keywords)
   - Warn user about systemic issues before making changes that won't help

## Write Tool Workflow

1. Call a `draft_*` tool — returns preview with `plan_id`
2. Show full preview to user and **wait for approval**
3. Call `confirm_and_apply(plan_id=..., dry_run=true)` first to test
4. Only call with `dry_run=false` after explicit user confirmation

Safety behaviors:
- New campaigns and RSAs are created as PAUSED — user must explicitly enable after review
- `draft_campaign` enforces `max_daily_budget`, rejects BROAD + non-Smart Bidding, warns if budget < 5x CPA
- `remove_entity` triggers double confirmation. Supports: campaign, ad_group, ad, keyword, negative_keyword
- All operations logged to `~/.adloop/audit.log`

## GDPR Consent & Data Discrepancies

- **Google Ads counts all clicks** regardless of consent.
- **GA4 only records sessions for users who accept analytics cookies.** Users who reject consent are invisible to GA4.
- **Clicks >> Sessions is normal** (2:1 to 5:1 ratio with consent banners), NOT broken tracking.
- **Conversion events are also affected** — only consenting users trigger events. True conversion rates are likely higher.

Before diagnosing a tracking issue, always consider consent:
1. Ads 10 clicks, GA4 3 sessions — likely consent rejection, not broken tracking.
2. GA4 0 sessions from paid traffic — consent could explain it, but also check UTMs and GA4 filters.
3. Only flag tracking as broken when discrepancy can't be explained by consent (e.g., zero sessions for ALL sources).

**Google Consent Mode v2:** Some sites send cookieless pings even without consent, reducing the gap. Check for `gtag('consent', ...)` in the codebase.

## Orchestration Patterns

### Performance / "how are my ads doing"

1. `get_campaign_performance` for the date range
2. If conversions/CPA mentioned, use `analyze_campaign_conversions` instead (Ads + GA4 in one call)
3. If keywords/search terms mentioned, add `get_keyword_performance` or `get_search_terms`
4. Present: spend, clicks, conversions, CPA, CTR
5. Flag: zero conversions, high CPA, low QS, wasteful search terms

### Conversions or conversion drops

1. `attribution_check` with date range and `conversion_events` if user specifies events
2. If page-level drill-down needed, `landing_page_analysis`
3. `get_search_terms` to check intent shift
4. Before concluding broken tracking: check `insights` from `attribution_check` — it factors in GDPR
5. Search codebase for recent changes to affected pages
6. Unified diagnosis combining tool insights + code analysis

### Create an ad

1. `get_campaign_performance` — find campaign structure and `campaign.id`
2. Pre-write checks:
   - Bidding strategy appropriate? MANUAL_CPC campaigns shouldn't get more ads before fixing bidding
   - Any conversions? High spend + zero conversions = warn, adding ads won't help
   - Quality scores? All below 5 = improve relevance first
3. `run_gaql` for ad group IDs: `SELECT ad_group.id, ad_group.name FROM ad_group WHERE campaign.id = {id}`
4. `get_tracking_events` to verify conversion tracking
5. Read landing page code for value propositions and language
6. `draft_responsive_search_ad` with 8-10 diverse headlines, 3-4 descriptions. Correct language. Count characters.
7. Present preview with any warnings. Wait for approval.

### Add keywords

1. `get_campaign_performance` — identify campaign and bidding strategy
2. Pre-write checks:
   - MANUAL_CPC/MANUAL_CPM: ONLY EXACT or PHRASE. NEVER BROAD. Explain why.
   - Zero conversions: warn, keywords won't help until tracking works
   - All QS < 5: warn, problem is relevance not coverage
3. `get_keyword_performance` to avoid duplicates
4. `get_search_terms` to understand current triggers
5. `draft_keywords` with appropriate match types
6. Present preview + warnings. Wait for approval.

### Create a new campaign

1. `get_campaign_performance` — understand structure, avoid duplicate names
2. If no budget specified, `estimate_budget` for data-driven recommendation
3. Pre-write checks:
   - Recommend MAXIMIZE_CONVERSIONS or TARGET_CPA over MANUAL_CPC
   - Check conversion tracking via `attribution_check`
   - Budget <= `max_daily_budget` and ideally >= 5x target CPA
4. `draft_campaign` with all parameters
5. Present preview — emphasize campaign created as PAUSED
6. Remind to add ads via `draft_responsive_search_ad` then enable via `enable_entity`
7. Wait for approval.

### Budget planning / "how much should I spend"

1. Ask for target keywords (or suggest based on context)
2. Ask for geography and language (or infer from account)
3. `estimate_budget` with keywords, match types, optional budget
4. Present forecast: clicks, impressions, cost, avg CPC
5. Highlight if budget is sufficient for available traffic

### Tracking / event issues

1. Consider GDPR consent first — if Ads clicks > GA4 sessions, state this
2. Search codebase for `gtag('event'` and `dataLayer.push`, plus consent mode (`gtag('consent', ...)`)
3. `validate_tracking` with extracted event names
4. Review `insights[]` — missing = not deployed; unexpected = tag manager
5. If new tracking needed, `generate_tracking_code`

### Negative keywords

1. `get_search_terms` for current data
2. `get_negative_keywords` to avoid duplicates
3. Identify irrelevant terms, group by theme
4. `get_campaign_performance` for `campaign.id`
5. `add_negative_keywords` with proposed list
6. Present preview, wait for confirmation

### Pause or enable something

1. Read tool to confirm entity exists and current status
2. `pause_entity` or `enable_entity` with type and ID
3. Warn about impact (e.g., "stops all ads in this campaign")
4. Wait for confirmation

### Landing page performance

1. `landing_page_analysis` — combines ad URLs with GA4 page data
2. Review `insights[]` for zero conversions, high bounce, orphaned URLs
3. Read flagged landing pages to identify UX/content issues
4. Present sorted by paid sessions, highlighting problems

### Tracking verification / "is tracking working"

1. `attribution_check` with `conversion_events` (e.g., `["sign_up", "purchase"]`)
2. Missing events or zero counts → search codebase, then `validate_tracking`
3. GDPR gaps in insights → explain normal EU behavior
4. If tracking code needed, `generate_tracking_code`

### Paid vs organic / channel comparison

1. `analyze_campaign_conversions` — returns paid + non-paid channel rates
2. Compare `campaigns[].ga4_conversion_rate` vs `non_paid_channels[].conversion_rate`
3. If paid lower, investigate landing page relevance before increasing spend

## Default Parameters

When the user doesn't specify:
- **Date range**: Last 30 days for Ads, last 7 days for GA4
- **Customer ID**: Use default from config
- **Property ID**: Use default from config
- **Format**: "table" for `run_gaql` results

## GAQL Quick Reference

### Syntax

```sql
SELECT field1, field2
FROM resource
WHERE condition
ORDER BY field [ASC|DESC]
LIMIT n
```

### Common Resources

| Resource | Use For |
|----------|---------|
| `campaign` | Campaign-level data |
| `ad_group` | Ad group-level data |
| `ad_group_ad` | Ad-level data (includes ad copy) |
| `keyword_view` | Keyword performance |
| `search_term_view` | Search terms report |
| `ad_group_criterion` | Keywords and targeting criteria |
| `campaign_budget` | Budget information |
| `bidding_strategy` | Bidding strategy details |
| `customer_client` | List accounts under an MCC |

### Common Fields

**Campaign:** `campaign.id`, `campaign.name`, `campaign.status`, `campaign.advertising_channel_type` (SEARCH/DISPLAY/SHOPPING/VIDEO), `campaign.bidding_strategy_type`

**Ad Group:** `ad_group.id`, `ad_group.name`, `ad_group.status`, `ad_group.cpc_bid_micros` (divide by 1,000,000)

**Ad:** `ad_group_ad.ad.responsive_search_ad.headlines`, `.descriptions`, `ad_group_ad.ad.final_urls`, `ad_group_ad.status`

**Keyword:** `ad_group_criterion.keyword.text`, `.match_type` (EXACT/PHRASE/BROAD), `ad_group_criterion.quality_info.quality_score`

**Metrics:** `metrics.impressions`, `metrics.clicks`, `metrics.cost_micros`, `metrics.conversions`, `metrics.conversions_value`, `metrics.ctr`, `metrics.average_cpc`, `metrics.search_impression_share`, `metrics.search_rank_lost_impression_share`

**Segments:** `segments.date` (daily), `segments.device` (MOBILE/DESKTOP/TABLET), `segments.ad_network_type` (SEARCH/CONTENT/YOUTUBE)

### Date Ranges

```sql
WHERE segments.date DURING LAST_7_DAYS
WHERE segments.date DURING LAST_30_DAYS
WHERE segments.date DURING THIS_MONTH
WHERE segments.date DURING LAST_MONTH
WHERE segments.date BETWEEN '2026-01-01' AND '2026-01-31'
```

### Important GAQL Rules

- No `SELECT *` — every field must be named
- **Fields in ORDER BY must be in SELECT.** Most common GAQL error.
- Metrics fields cannot appear in WHERE with resource fields (use HAVING or filter in app logic)
- `cost_micros` = micros, divide by 1,000,000. Dedicated tools already compute `metrics.cost` and `metrics.cpa`.
- `search_term_view` always requires a date segment in WHERE
- Status values: `'ENABLED'`, `'PAUSED'`, `'REMOVED'`

### Example Queries

Top campaigns by spend:
```sql
SELECT campaign.name, campaign.status, metrics.cost_micros, metrics.clicks, metrics.conversions
FROM campaign
WHERE segments.date DURING LAST_30_DAYS AND campaign.status = 'ENABLED'
ORDER BY metrics.cost_micros DESC
LIMIT 10
```

Low quality score keywords:
```sql
SELECT ad_group_criterion.keyword.text, ad_group_criterion.quality_info.quality_score,
       metrics.impressions, metrics.clicks, metrics.cost_micros
FROM keyword_view
WHERE segments.date DURING LAST_30_DAYS
  AND ad_group_criterion.quality_info.quality_score < 5
ORDER BY metrics.cost_micros DESC
```

Wasteful search terms (spend but no conversions):
```sql
SELECT search_term_view.search_term, metrics.clicks, metrics.cost_micros, metrics.conversions
FROM search_term_view
WHERE segments.date DURING LAST_30_DAYS AND metrics.clicks > 5 AND metrics.conversions = 0
ORDER BY metrics.cost_micros DESC
LIMIT 20
```

Ad groups in a campaign:
```sql
SELECT ad_group.id, ad_group.name, ad_group.status
FROM ad_group
WHERE campaign.id = 12345678
```

## Ad Copy Character Limits

Google Ads enforces hard limits. `draft_responsive_search_ad` rejects copy that exceeds them — write copy that fits on the FIRST attempt.

- **Headlines: 30 characters** max (including spaces)
- **Descriptions: 90 characters** max (including spaces)
- **Display path: 15 characters** each

30 characters is very short. Many languages produce phrases that easily exceed it (German compounds, French/Spanish with articles).

Rules:
1. **Count characters for every headline BEFORE calling draft.** Spaces, hyphens, accents all count.
2. **Abbreviate long words.** If a term exceeds ~15 chars, abbreviate or use a shorter synonym.
3. **Numbers and symbols save space.**
4. **Long phrases go in descriptions (90 chars), not headlines (30 chars).**
5. **Write in the correct language.** Check landing page. If unclear, ask user.
6. **Aim for 25 chars or fewer** to leave margin.

## Diesel BI Persistence

After every analysis or report, persist results to Diesel BI via its MCP server (company_slug: "disbra" in prod, "diesel" in dev):

### Save workflow (all reports)

1. `salvar_relatorio` — full markdown + kpis + summary
2. `salvar_insights` — each recommendation with `suggested_command`
3. `salvar_kpi_snapshot` — trend data (spend, clicks, conversions, CPA, etc.)
4. `salvar_campaign_snapshots` — per-campaign analysis with health score
5. `salvar_action_proposals` — actions needing approval (with campaign_snapshot_id and report_id)

### Health Score Rules (for campaign snapshots)

- **8-10 (saudavel)**: CPA at/below target, steady conversions, good QS, positive trend
- **4-7 (atencao)**: Some issues — CPA slightly high, conversions dropping, or mixed signals
- **1-3 (problematica)**: CPA way above target, zero conversions, Manual CPC + Display, or critical issues

### Action Proposal Rules

- Generate proposals for actions that need user approval (pause, enable, add negatives, budget changes)
- Always include `suggested_command` with the exact tool call Claude would make
- Set `risk_level`: "low" for negatives/enable, "medium" for budget/pause, "high" for remove/create
- Link to `campaign_snapshot_id` when the action relates to a specific campaign
- Link to `report_id` when generated as part of a report

### Deduplication

Before saving insights, call `listar_insights(status: "pending")` and skip insights with similar titles. Before saving proposals, call `listar_action_proposals(status: "pending")` and skip duplicates.

## Marketing Best Practices

- **Match types**: Never BROAD without Smart Bidding (tCPA or tROAS). Broad + Manual CPC = budget waste.
- **CPA monitoring**: Flag any campaign where CPA > 3x target.
- **Budget sufficiency**: Daily budget >= 5x target CPA for enough algorithm data.
- **Learning phase**: Don't edit campaigns showing "Learning" or "Learning (limited)". Wait.
- **Negative keyword hygiene**: After search terms review, suggest irrelevant terms as negatives. Group by theme.
- **RSA best practices**: 8-10 unique headlines (max 15), 3-4 descriptions (max 4). Diverse headlines. Pin only when necessary.
- **Quality Score**: QS < 5 — prioritize ad relevance and landing page over bid increases.
- **Zero conversions**: Significant spend + zero conversions → investigate: (1) GDPR reducing visible conversions? (2) Conversion tracking correct? (3) Landing page converting organic? (4) Search terms relevant? Don't just increase budget.
- **Manual CPC + Broad Match**: #1 budget waste combo. NEVER create. If exists, recommend PHRASE/EXACT or move to Smart Bidding first.
- **Clicks vs sessions gap**: Never report as bug without accounting for GDPR consent. EU: 30-70% may reject cookies. Normal.
