Gerar relatório mensal COMPLETO de performance do Google Ads + GA4 e salvar no Diesel BI.

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

## 3. Gerar relatório (OBRIGATÓRIO: seguir esta estrutura)

O body do relatório DEVE conter TODAS estas seções em markdown. Não resumir, não pular seções.

```markdown
# Disbra — Relatório Mensal {ANO}-{MES}

> {data_inicio} a {data_fim} | Gerado via Claude + AdLoop

## Resumo Executivo

- **Gasto:** R$ X | **Conversões:** X | **CPA:** R$ X
- **Clicks:** X | **CTR:** X% | **CPC médio:** R$ X
- vs mês anterior: gasto {+/-X%}, conversões {+/-X%}, CPA {+/-X%}

## Campanhas

| Campanha | Tipo | Status | Gasto | Conv | CPA | CTR | Health |
|----------|------|--------|-------|------|-----|-----|--------|
(todas as campanhas com dados)

**Ranking por eficiência (menor CPA):**
1. (campanha)
2. (campanha)

## Tendência Semanal

| Semana | Gasto | Conv | CPA | CTR |
|--------|-------|------|-----|-----|
(4 semanas do mês)

## Top Keywords

| Keyword | Match | Gasto | Conv | CPA | QS |
|---------|-------|-------|------|-----|----|
(top 10 por conversão)

**Piores keywords (gasto sem retorno):**
(keywords com gasto > R$20 e 0 conversões)

## Anúncios

**Melhores (por CTR):**
| Headline | CTR | Conv | Clicks |
|----------|-----|------|--------|

**Piores (por CTR):**
| Headline | CTR | Conv | Clicks |
|----------|-----|------|--------|

## Landing Pages

| URL | Sessions | Bounce | Conv Rate | Problemas |
|-----|----------|--------|-----------|-----------|

## Search Terms

**Desperdício: R$ X em Y termos sem conversão**

| Termo | Clicks | Gasto | Problema |
|-------|--------|-------|----------|

**Termos promissores:**
| Termo | Clicks | Conv | Taxa |
|-------|--------|------|------|

## GDPR Gap

| Ads Clicks | GA4 Sessions | Ratio | Status |
|-----------|-------------|-------|--------|

## Paid vs Organic

| Canal | Sessions | Conv | Taxa |
|-------|----------|------|------|

## Diagnóstico

### O que está bom
- (bullets)

### O que precisa de atenção
- (bullets com números)

## Ações Recomendadas

| # | Ação | Prioridade | Risco | Impacto Estimado |
|---|------|-----------|-------|-----------------|
(todas as ações, numeradas, priorizadas)
```

## 4. Salvar no Diesel BI (diesel-bi MCP, company_slug "disbra")

### 4a. salvar_relatorio
- report_type: "monthly"
- title: "Relatório Mensal — {ANO}-{MES}"
- period_start / period_end: datas do mês
- body: **O MARKDOWN COMPLETO das seções acima. NÃO resumir.**
- summary: 2-3 frases do resumo executivo
- kpis: { "spend": X, "conversions": X, "cpa": X, "clicks": X, "ctr": X, "cpc": X, "impressions": X }
- source: "adloop"
- status: "published"

### 4b. salvar_insights (com o report_id retornado)
Cada ação recomendada vira um insight separado:
- category, priority, title, description, impact
- suggested_command: comando exato que o Claude executaria para implementar

### 4c. salvar_kpi_snapshot
- period_type: "monthly"
- period_start / period_end
- spend, clicks, impressions, conversions, cpa, ctr, cpc
- top_campaign, waste_amount, insights_count

### 4d. salvar_campaign_snapshots
Para cada campanha, gerar snapshot com health_score, health_label, summary, strengths, problems, recommendations. Ver weekly-report.md para detalhes dos campos.

### 4e. salvar_action_proposals
Cada ação que precisa aprovação vira proposal com action_type, title, description, impact, priority, risk_level, suggested_command, campaign_snapshot_id, report_id.

### 4f. Check de duplicatas
Antes de salvar insights, chamar `listar_insights(status: "pending")` e evitar duplicatas.

### 4g. Continuidade
Chamar `listar_insights(status: "pending")` da período anterior, mencionar pendentes no relatório.

## 5. Salvar .md local

`reports/monthly/{ANO}-{MES}.md`

## 6. Apresentar

Top 5 insights e ações. Se Diesel BI MCP indisponível, gerar só .md e avisar.
