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

1. Create a Google Cloud project → https://console.cloud.google.com/projectcreate
2. Enable APIs: Google Analytics Data API, Google Analytics Admin API, Google Ads API
3. Create OAuth consent screen (External, Testing mode OK) — add your email as test user
4. Create OAuth 2.0 Client ID (Desktop application) — download JSON
5. Get Google Ads Developer Token from MCC → https://ads.google.com/aw/apicenter

## Tool Inventory (26 tools)

### Diagnostics
| Tool | Purpose |
|------|---------|
| `health_check` | Test OAuth, GA4, and Ads connectivity |

### GA4 Read (4)
| Tool | Purpose |
|------|---------|
| `get_account_summaries` | List GA4 accounts and properties |
| `run_ga4_report` | Custom reports: sessions, users, conversions |
| `run_realtime_report` | Live data after deploys |
| `get_tracking_events` | Configured events and volume |

### Google Ads Read (6)
| Tool | Purpose |
|------|---------|
| `list_accounts` | Discover Ads accounts |
| `get_campaign_performance` | Campaign metrics (impressions, clicks, cost, CPA) |
| `get_ad_performance` | Ad copy analysis (headlines, descriptions, CTR) |
| `get_keyword_performance` | Keywords with quality scores |
| `get_search_terms` | Actual user search queries |
| `get_negative_keywords` | Existing negative keywords |
| `run_gaql` | Arbitrary GAQL queries |

### Cross-Reference (3)
| Tool | Purpose |
|------|---------|
| `analyze_campaign_conversions` | Ads clicks → GA4 sessions → conversions, GDPR gap detection |
| `landing_page_analysis` | Ad URLs → GA4 page data, zero-conversion flags |
| `attribution_check` | Ads vs GA4 conversion discrepancies |

### Write (8) — all use draft → preview → confirm
| Tool | Purpose |
|------|---------|
| `draft_campaign` | Create campaign structure (PAUSED) |
| `draft_responsive_search_ad` | Create RSA preview |
| `draft_keywords` | Propose keyword additions |
| `add_negative_keywords` | Propose negative keywords |
| `pause_entity` | Pause campaign/ad group/ad/keyword |
| `enable_entity` | Re-enable paused entity |
| `remove_entity` | Remove entity (irreversible) |
| `confirm_and_apply` | Execute previewed change (dry_run=true default) |

### Other
| Tool | Purpose |
|------|---------|
| `validate_tracking` | Compare codebase events vs GA4 |
| `generate_tracking_code` | Generate gtag JavaScript |
| `estimate_budget` | Keyword Planner forecasts |

## Safety Rules

1. NEVER call `confirm_and_apply` without showing preview first
2. Default to `dry_run=true` always
3. Respect budget caps from config
4. One change at a time — never batch writes
5. Never guess entity IDs — always read first
6. NEVER add BROAD match keywords without Smart Bidding
7. Pre-write validation: check bidding strategy, conversion tracking, quality scores before any write

## GDPR Consent & Data Discrepancies

- Google Ads counts all clicks. GA4 only records sessions from users who accept cookies.
- Clicks >> Sessions is normal (2:1 to 5:1 ratio), NOT broken tracking.
- Check for Google Consent Mode v2 (`gtag('consent', ...)`) before diagnosing tracking issues.

## Key Marketing Rules

- Broad Match + Manual CPC = #1 cause of wasted budget. Never combine.
- Budget should be >= 5x target CPA.
- Don't edit campaigns in learning phase.
- Zero conversions + high spend = fix tracking first, don't add more ads.
- Quality Score < 5 = fix landing page/relevance, not keyword count.
