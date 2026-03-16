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

### 3a. salvar_relatorio
- report_type: "custom", body completo, summary, kpis

### 3b. salvar_insights
- Problemas e recomendações com suggested_command

### 3c. salvar_kpi_snapshot
- Se for período standard (semana/mês)

### 3d. salvar_campaign_snapshots
- Para cada campanha analisada, gerar snapshot com health_score, health_label, summary, strengths, problems, recommendations

### 3e. salvar_action_proposals
- Cada ação recomendada que precisa aprovação vira proposal com action_type, title, priority, risk_level, suggested_command

### 3f. Check de duplicatas
- Chamar `listar_insights(status: "pending")` antes de salvar, evitar duplicatas

## 4. Salvar .md local

`reports/{DATA}_{DESCRICAO}.md`

Se Diesel BI MCP indisponível, gerar só .md e avisar.
