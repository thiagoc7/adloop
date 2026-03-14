Analyze Google Ads + GA4 performance for $ARGUMENTS

Follow this workflow:

1. Run `get_campaign_performance` for the relevant date range (default: last 30 days)
2. Run `analyze_campaign_conversions` to get unified Ads + GA4 data with GDPR-aware metrics
3. If specific campaigns are mentioned, filter by campaign name
4. If keywords are relevant, also run `get_keyword_performance` and `get_search_terms`

Present a summary with:
- **Spend** (metrics.cost), **Clicks**, **Conversions**, **CPA**, **CTR**
- Comparison: paid vs organic conversion rates (from non_paid_channels)
- GDPR consent gap analysis (clicks vs sessions ratio)
- Flags: zero conversions, CPA > 3x target, QS < 5, wasteful search terms

If conversion issues are found, run `attribution_check` for deeper diagnosis.
If landing page issues suspected, run `landing_page_analysis`.

Always consider GDPR consent before diagnosing tracking as broken.
