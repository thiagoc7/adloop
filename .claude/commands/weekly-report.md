Gerar relatório semanal COMPLETO de performance do Google Ads + GA4 e salvar no Diesel BI.

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

## 3. Gerar relatório (OBRIGATÓRIO: seguir esta estrutura)

O body do relatório DEVE conter TODAS estas seções em markdown. Não resumir, não pular seções.

```markdown
# Disbra — Relatório Semanal {ANO}-W{SEMANA}

> {data_inicio} a {data_fim} | Gerado via Claude + AdLoop

## Resumo

- **Gasto:** R$ X | **Conversões:** X | **CPA:** R$ X
- **Clicks:** X | **CTR:** X% | **CPC médio:** R$ X
- vs semana anterior: gasto {+/-X%}, conversões {+/-X%}, CPA {+/-X%}

## Campanhas

| Campanha | Status | Gasto | Conv | CPA | CTR | vs Anterior |
|----------|--------|-------|------|-----|-----|-------------|
(todas as campanhas com dados)

**Observações:**
- (destaques, problemas, oportunidades por campanha)

## Top Keywords

| Keyword | Match | Gasto | Conv | CPA | QS | Tendência |
|---------|-------|-------|------|-----|----|-----------|
(top 10 por gasto)

**Problemas:**
- (keywords com QS baixo, CPA alto, etc)

## Search Terms — Desperdício

**R$ X em Y termos sem conversão**

| Termo | Clicks | Gasto | Problema |
|-------|--------|-------|----------|
(todos os termos com gasto > R$5 e 0 conversões)

**Termos promissores:**

| Termo | Clicks | Conv | Taxa |
|-------|--------|------|------|
(termos com boa taxa de conversão)

## GDPR Gap

| Ads Clicks | GA4 Sessions | Ratio | Status |
|-----------|-------------|-------|--------|
(dados do analyze_campaign_conversions)

## Diagnóstico

### O que está bom
- (bullets)

### O que precisa de atenção
- (bullets com números)

## Ações Recomendadas

| # | Ação | Prioridade | Impacto Estimado |
|---|------|-----------|-----------------|
(todas as ações, numeradas, priorizadas)
```

## 4. Salvar no Diesel BI (diesel-bi MCP, company_slug "disbra")

### 4a. salvar_relatorio
- report_type: "weekly"
- title: "Relatório Semanal — {ANO}-W{SEMANA}"
- period_start / period_end: datas da semana
- body: **O MARKDOWN COMPLETO das seções acima. NÃO resumir.**
- summary: 2-3 frases do resumo executivo
- kpis: { "spend": X, "conversions": X, "cpa": X, "clicks": X, "ctr": X, "cpc": X, "impressions": X }

### 4b. salvar_kpi_snapshot
- period_type: "weekly", spend, clicks, impressions, conversions, cpa, ctr, cpc
- top_campaign, waste_amount, insights_count

### 4c. salvar_campaign_snapshots
Para cada campanha, gerar snapshot com health_score, health_label, summary, strengths, problems, recommendations.
- health_score 8-10 → "saudavel" | 4-7 → "atencao" | 1-3 → "problematica"

### 4d. salvar_action_proposals (PRINCIPAL — todo insight acionável vira proposal)
Cada recomendação acionável vira uma ActionProposal vinculada à campanha E ao relatório:
- action_type: pause|enable|add_negatives|create_ad|change_budget|add_keywords|create_campaign
- title, description, impact, priority, risk_level
- suggested_command: comando exato que o Claude executaria
- campaign_snapshot_id: ID do snapshot (do passo 4c)
- report_id: ID do relatório (do passo 4a)

O usuário vê essas ações no Diesel BI (na página da campanha e no relatório) e pode aprovar/rejeitar/comentar.

### 4e. salvar_insights (secundário, para relatório)
Mesmas recomendações salvas como insights para exibição no sidebar do relatório.
- category, priority, title, description, impact, suggested_command

### 4f. Verificar ações executadas
Chamar `listar_action_proposals(status: "executed")` — ações que foram executadas mas ainda não verificadas.
Para cada uma, comparar métricas antes vs depois e atualizar:
- `atualizar_action_proposal(status: "verified", verification_notes: "Desperdício caiu X%")`
- Mencionar no relatório: "Semana passada negativamos X termos. Resultado: desperdício caiu Y%"

### 4g. Processar ações aprovadas
Chamar `listar_action_proposals(status: "approved")` — ações aprovadas pelo usuário.
Para cada uma:
- Ler `user_notes` (comentários do usuário)
- Executar via AdLoop (draft → preview → confirm_and_apply)
- Atualizar: `atualizar_action_proposal(status: "executed", execution_notes: "...")`

### 4h. Check de duplicatas
Chamar `listar_action_proposals(status: "pending")` antes de salvar, evitar duplicatas.

## 5. Também salvar .md local

Salvar cópia em `reports/weekly/{ANO}-W{SEMANA}.md` para histórico git.

## 6. Apresentar ao usuário

Resumo executivo com as 3 ações mais importantes.
Se o Diesel BI MCP não estiver disponível, gerar só o .md e avisar.
