Analisar e otimizar campanha do Google Ads: $ARGUMENTS

## Checklist de Otimização (seguir nesta ordem)

### 1. Diagnóstico
- `get_campaign_performance` — métricas atuais
- `attribution_check` — tracking funcionando?
- Se zero conversões + gasto alto → PARAR e resolver tracking primeiro

### 2. Limpeza de Search Terms
- `get_search_terms` — identificar irrelevantes
- `get_negative_keywords` — o que já está bloqueado
- Propor negativas com `add_negative_keywords` (mostrar preview)

### 3. Quality Score
- `get_keyword_performance` — verificar QS de cada keyword
- QS < 5 → problema de relevância (landing page + anúncio)
- Sugerir melhorias no anúncio ou landing page

### 4. Anúncios
- `get_ad_performance` — quais anúncios performam melhor
- Se CTR < 2% → reescrever headlines
- Se poucos anúncios → criar novos com `draft_responsive_search_ad`

### 5. Bidding Strategy
- Verificar se usa Smart Bidding ou Manual CPC
- Manual CPC + Broad Match → MIGRAR para Phrase/Exact OU mudar bidding
- Recomendar Maximize Conversions se tem dados suficientes

### 6. Budget
- Budget atual vs CPA → está cobrindo >= 5x CPA?
- Se não → ou aumentar budget ou reduzir CPA

## Regras
- Uma mudança por vez — mostrar preview, esperar aprovação
- Nunca fazer mudança durante Learning Phase
- Priorizar: tracking > negativas > QS > anúncios > bidding > budget
- Explicar cada recomendação em português simples
