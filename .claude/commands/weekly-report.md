Gerar relatório semanal de performance do Google Ads + GA4 e salvar no Diesel BI.

## 1. Puxar dados (adloop MCP)

- `get_campaign_performance` — últimos 7 dias
- `get_campaign_performance` — 7 dias anteriores (para comparação semana vs semana)
- `analyze_campaign_conversions` — dados cruzados Ads + GA4 com GDPR gap
- `get_keyword_performance` — top keywords por gasto
- `get_search_terms` — identificar termos irrelevantes e promissores
- `get_negative_keywords` — verificar o que já está bloqueado (evitar duplicatas)

## 2. Analisar

- Calcular variação % semana vs semana para cada KPI
- Destacar campanhas com gasto > R$200 e 0 conversões
- Destacar keywords com QS < 5
- Listar search terms irrelevantes como candidatos a negativa
- Listar search terms com boa conversão como candidatos a keyword
- Calcular gasto desperdiçado em termos irrelevantes
- Considerar GDPR antes de diagnosticar tracking como quebrado

## 3. Gerar relatório

Usar o padrão conciso: bullets, tabelas pequenas, ações claras.
Estrutura: Resumo → Campanhas → Keywords → Desperdício → Ações

## 4. Salvar no Diesel BI (diesel-bi MCP)

Chamar as 3 tools do Diesel BI MCP com company_slug "disbra":

### 4a. salvar_relatorio
```
report_type: "weekly"
title: "Relatório Semanal — {ANO}-W{SEMANA}"
period_start / period_end: datas da semana
body: markdown completo do relatório
summary: 2-3 frases do resumo executivo
kpis: { spend, conversions, cpa, clicks, ctr, cpc }
```

### 4b. salvar_insights (com o report_id retornado)
Cada ação recomendada vira um insight:
```
category: "negative_keywords" | "budget" | "bidding" | "ad_copy" | "campaign_structure" | "tracking"
priority: "high" | "medium" | "low"
title: descrição curta
description: detalhe
impact: estimativa de economia/melhoria
```

### 4c. salvar_kpi_snapshot
```
period_type: "weekly"
spend, clicks, impressions, conversions, cpa, ctr, cpc
waste_amount: total gasto em termos sem conversão
insights_count: quantos insights gerados
```

## 5. Também salvar .md local

Salvar cópia em `reports/weekly/{ANO}-W{SEMANA}.md` para histórico git.

## 6. Apresentar ao usuário

Resumo executivo com as 3 ações mais importantes.
Se o Diesel BI MCP não estiver disponível, gerar só o .md e avisar.
