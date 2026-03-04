# AdLoop

MCP server that connects Google Ads + Google Analytics (GA4) into one AI-driven feedback loop inside your IDE.

> "You built your product with AI. Now manage your ads the same way."

## Why

Solo founders and small teams ship products fast with AI-assisted coding — but managing Google Ads still means switching between the Ads UI, GA4 dashboards, and your code editor. Whether you're running a SaaS, an e-commerce store, a local service, or anything else you drive traffic to with Google Ads — the workflow is the same fragmented mess.

AdLoop brings the entire **build → ship → market → measure → iterate** cycle into your IDE.

One MCP server gives your AI assistant access to both Google Analytics and Google Ads (read + write), with a safety layer that prevents accidental spend. Combined with your codebase context, it can do things no dashboard can — like diagnosing why conversions dropped by cross-referencing ad traffic, analytics events, and your actual frontend code.

## What's Included

### GA4 Read Tools

| Tool | What It Does |
|------|-------------|
| `get_account_summaries` | List GA4 accounts and properties |
| `run_ga4_report` | Custom reports — sessions, users, conversions, page performance |
| `run_realtime_report` | Live data — verify tracking fires after deploys |
| `get_tracking_events` | All configured events and their volume |

### Google Ads Read Tools

| Tool | What It Does |
|------|-------------|
| `list_accounts` | Discover accessible Ads accounts |
| `get_campaign_performance` | Campaign metrics — impressions, clicks, cost, conversions, CPA |
| `get_ad_performance` | Ad copy analysis — headlines, descriptions, CTR |
| `get_keyword_performance` | Keywords — quality scores, competitive metrics |
| `get_search_terms` | What users actually searched before clicking |
| `run_gaql` | Arbitrary GAQL queries for anything else |

### Google Ads Write Tools

All write operations follow a **draft → preview → confirm** workflow. Nothing executes without explicit approval.

| Tool | What It Does |
|------|-------------|
| `draft_responsive_search_ad` | Create RSA preview (3-15 headlines, 2-4 descriptions) |
| `draft_keywords` | Propose keyword additions with match types |
| `add_negative_keywords` | Propose negative keywords to reduce wasted spend |
| `pause_entity` | Pause a campaign, ad group, ad, or keyword |
| `enable_entity` | Re-enable a paused entity |
| `remove_entity` | Permanently remove an entity (irreversible — prefers pause) |
| `confirm_and_apply` | Execute a previously previewed change |

### Cursor Rules

AdLoop ships with orchestration rules (`.cursor/rules/adloop.mdc`) that teach the AI *how* to combine these tools — marketing workflows, GAQL syntax, safety protocols, and best practices. Without rules, the AI has tools but doesn't know the playbook.

## Safety Model

AdLoop manages real ad spend, so safety is not optional.

- **Two-step writes.** Every mutation returns a preview first. A separate `confirm_and_apply` call is required to execute.
- **Dry-run by default.** Even `confirm_and_apply` defaults to `dry_run=true`. Real changes require explicit `dry_run=false`.
- **Budget caps.** Configurable maximum daily budget — the server rejects anything above the cap.
- **Audit log.** Every operation (including dry runs) is logged to `~/.adloop/audit.log`.
- **New ads are PAUSED.** Created RSAs start paused — you review before they go live.
- **Destructive ops require double confirmation.** Removing entities or large budget increases trigger extra warnings.

## Setup

### Requirements

- Python 3.11+
- [uv](https://docs.astral.sh/uv/) for package management
- A Google Cloud project (free tier works)
- A Google Ads account with an MCC (Manager Account)

### Step 1 — Google Cloud Project

If you don't have a Google Cloud project yet:

1. Go to [console.cloud.google.com](https://console.cloud.google.com/) and create a new project
2. Enable these three APIs (search for each in the API Library):
   - **Google Analytics Data API** — for GA4 reports and events
   - **Google Analytics Admin API** — for listing GA4 properties
   - **Google Ads API** — for all ads operations

### Step 2 — OAuth Credentials

AdLoop authenticates as you (not as a service). You need OAuth Desktop credentials:

1. In your Google Cloud project, go to **APIs & Services → Credentials**
2. Click **Create Credentials → OAuth client ID**
3. Select **Desktop app** as the application type, give it any name
4. Download the JSON file and save it as `~/.adloop/credentials.json`

On first run, AdLoop opens a browser window where you sign in with your Google account and grant access. The resulting token is saved to `~/.adloop/token.json` and refreshed automatically.

> Service accounts are also supported — just place the service account key JSON at the same `credentials_path`. AdLoop detects the file type automatically.

### Step 3 — Google Ads Developer Token

The developer token lets AdLoop talk to the Google Ads API. You get one through a Manager Account (MCC):

1. **Create an MCC** (free) at [ads.google.com/home/tools/manager-accounts](https://ads.google.com/home/tools/manager-accounts/) if you don't have one. Link your regular Google Ads account to it.
2. In the MCC, go to **Tools & Settings → API Center**
3. Your **developer token** is shown there. Copy it.

Access levels:
- **Explorer** (automatic) — 2,880 operations/day on production accounts. Enough to get started.
- **Basic** (requires application) — 15,000 operations/day. Apply through the same API Center page if you need more.

### Step 4 — Find Your IDs

You need three IDs for the config:

| ID | Where to Find It |
|----|-------------------|
| **GA4 Property ID** | GA4 → Admin → Property Settings (numeric, e.g. `123456789`) |
| **Google Ads Customer ID** | Google Ads UI → top bar (e.g. `123-456-7890`) |
| **MCC Account ID** | MCC UI → top bar (e.g. `123-456-7890`) |

### Step 5 — Install and Configure

```bash
git clone https://github.com/kLOsk/adloop.git
cd adloop
uv sync

mkdir -p ~/.adloop
cp config.yaml.example ~/.adloop/config.yaml
```

Edit `~/.adloop/config.yaml` and fill in the values from the previous steps. See [`config.yaml.example`](config.yaml.example) for a fully documented template.

### Step 6 — Connect to Cursor

Add to your project's `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "adloop": {
      "command": "/absolute/path/to/adloop/.venv/bin/python",
      "args": ["-m", "adloop"],
      "env": {
        "ADLOOP_CONFIG": "~/.adloop/config.yaml"
      }
    }
  }
}
```

Then copy `.cursor/rules/adloop.mdc` from this repo into your project's `.cursor/rules/` directory. This teaches the AI how to orchestrate the tools — marketing workflows, GAQL syntax, safety protocols.

### Step 7 — Use It

Ask your AI assistant things like:

- *"How are my Google Ads campaigns performing this month?"*
- *"Which search terms are wasting budget? Add them as negative keywords."*
- *"My sign-up conversions dropped — check GA4 and Ads to find out why."*
- *"Draft a new responsive search ad for my main campaign."*

## Configuration Reference

All configuration lives in `~/.adloop/config.yaml`. See [`config.yaml.example`](config.yaml.example) for a documented template.

| Section | Key | Default | Description |
|---------|-----|---------|-------------|
| `google` | `project_id` | — | Your Google Cloud project ID |
| `google` | `credentials_path` | `~/.adloop/credentials.json` | Path to OAuth client JSON or service account key |
| `google` | `token_path` | `~/.adloop/token.json` | Where to store the OAuth token (auto-created) |
| `ga4` | `property_id` | — | Your GA4 property ID (found in GA4 Admin → Property Settings) |
| `ads` | `developer_token` | — | Your Google Ads API developer token |
| `ads` | `customer_id` | — | Default Google Ads customer ID |
| `ads` | `login_customer_id` | — | Your MCC account ID |
| `safety` | `max_daily_budget` | `50.00` | Maximum allowed daily budget per campaign (EUR) |
| `safety` | `require_dry_run` | `true` | Force all writes to dry-run mode |
| `safety` | `blocked_operations` | `[]` | Operations to block entirely |

## Project Structure

```
src/adloop/
├── server.py          # FastMCP server + tool registrations
├── config.py          # Config loader (~/.adloop/config.yaml)
├── auth.py            # OAuth 2.0 Desktop flow + service account support
├── ga4/
│   ├── client.py      # GA4 Data + Admin API clients
│   ├── reports.py     # Account summaries, reports, realtime
│   └── tracking.py    # Event discovery
├── ads/
│   ├── client.py      # Google Ads API client
│   ├── gaql.py        # GAQL query execution
│   ├── read.py        # Campaign, ad, keyword, search term reads
│   └── write.py       # Draft, pause, enable, remove, confirm
└── safety/
    ├── guards.py      # Budget caps, bid limits, blocked operations
    ├── preview.py     # Change plans and previews
    └── audit.py       # Mutation audit logging
```

## Built From Real Usage

AdLoop isn't a theoretical tool — it's built from running real Google Ads campaigns and hitting real problems. Every tool exists because of an actual situation: needing to diagnose a conversion drop without leaving the IDE, wanting to bulk-add negative keywords after seeing wasted spend in the search terms report, drafting ad variants that match a landing page the AI just helped rewrite.

The best features come from real workflows. If you're using AdLoop and find yourself wishing it could do something it can't, **that's exactly the kind of feedback that shapes what gets built next.** Open an issue describing your situation — not just "add feature X" but "I was trying to do Y and couldn't because Z." The context matters more than the request.

## Roadmap

Planned based on what's been most needed in practice:

- **Cross-reference intelligence** — tools that combine GA4 + Ads data (campaign-to-conversion mapping, landing page analysis, attribution comparison)
- **Tracking utilities** — generate GA4 event code snippets, validate tracking implementation against GA4 config
- **Setup wizard** — `adloop init` to guide credential setup
- **PyPI package** — `pip install adloop`

## License

MIT — see [LICENSE](LICENSE).
