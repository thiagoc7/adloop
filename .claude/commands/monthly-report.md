Gerar relatório mensal de performance do Google Ads + GA4 e salvar no Diesel BI.

## 1. Puxar dados (adloop MCP)

- `get_campaign_performance` — últimos 30 dias
- `get_campaign_performance` — 30 dias anteriores (para comparação)
- `analyze_campaign_conversions` — dados cruzados com GDPR gap e canais orgânicos
- `landing_page_analysis` — performance das landing pages
- `get_keyword_performance` — todas as keywords com quality scores
- `get_ad_performance` — performance dos anúncios (headlines, CTR)
- `get_search_terms` — termos completos do mês
- `attribution_check` — verificar integridade do tracking

## 2. Analisar

- Tendência semanal dentro do mês (4 data points)
- Ranking de campanhas por eficiência (menor CPA)
- Top 10 keywords por conversão + piores por gasto sem retorno
- Melhores e piores anúncios
- Landing pages: bounce rate, conversão
- Funil: impressões → clicks → sessões → WhatsApp clicks
- Paid vs Organic
- Recomendações priorizadas (alta/média/baixa)

## 3. Salvar no Diesel BI (diesel-bi MCP, company_slug "disbra")

- `salvar_relatorio` — report_type: "monthly", body completo, summary, kpis
- `salvar_insights` — cada recomendação com category, priority, impact
- `salvar_kpi_snapshot` — period_type: "monthly", todos os KPIs

## 4. Salvar .md local

`reports/monthly/{ANO}-{MES}.md`

## 5. Apresentar

Top 5 insights e ações. Se Diesel BI MCP indisponível, gerar só .md e avisar.
