Gerar relatório de avaliação do trabalho da agência V4 e salvar no Diesel BI.

## 1. Puxar dados (adloop MCP)

- `get_campaign_performance` — últimos 30 dias
- `get_search_terms` — V4 está negativando termos irrelevantes?
- `get_keyword_performance` — quality scores melhorando ou piorando?
- `get_negative_keywords` — negativas adicionadas recentemente?
- `get_ad_performance` — anúncios novos? CTR melhorou?

## 2. Checklist da agência

Avaliar cada item com nota:
- [ ] Revisaram search terms este mês?
- [ ] Adicionaram negativas?
- [ ] CPA está dentro do target?
- [ ] Campanhas ruins foram pausadas ou otimizadas?
- [ ] Quality scores aceitáveis (>5)?
- [ ] Anúncios testados/renovados?
- [ ] Budget bem distribuído?

## 3. Comparar com check anterior

Se existir relatório v4_check anterior no Diesel BI (`listar_relatorios` com report_type "v4_check"), comparar evolução.

## 4. Salvar no Diesel BI (diesel-bi MCP, company_slug "disbra")

- `salvar_relatorio` — report_type: "v4_check", body com scorecard
- `salvar_insights` — problemas encontrados como insights
- `salvar_kpi_snapshot` — KPIs do período

## 5. Salvar .md local

`reports/v4-check-{DATA}.md`

**IMPORTANTE:** Não modificar campanhas. Apenas analisar.
