Analisar performance do Google Ads + GA4 e salvar no Diesel BI: $ARGUMENTS

## 1. Puxar dados (adloop MCP)

- `get_campaign_performance` — período relevante (default: últimos 30 dias)
- `analyze_campaign_conversions` — dados cruzados Ads + GA4 com GDPR gap
- Se campanhas específicas mencionadas, filtrar por nome
- Se keywords relevantes, `get_keyword_performance` e `get_search_terms`

## 2. Analisar

- Spend, Clicks, Conversões, CPA, CTR
- Paid vs organic (de non_paid_channels)
- GDPR gap (clicks vs sessions)
- Flags: zero conversões, CPA > 3x target, QS < 5, desperdício

Se problemas de conversão: `attribution_check`
Se problemas de landing page: `landing_page_analysis`

## 3. Salvar no Diesel BI (diesel-bi MCP, company_slug "disbra")

- `salvar_relatorio` — report_type: "custom", body, summary, kpis
- `salvar_insights` — problemas e recomendações
- `salvar_kpi_snapshot` — se for período standard (semana/mês)

## 4. Salvar .md local

`reports/{DATA}_{DESCRICAO}.md`

Se Diesel BI MCP indisponível, gerar só .md e avisar.
