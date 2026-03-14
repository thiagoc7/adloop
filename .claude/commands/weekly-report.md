Gerar relatório semanal de performance do Google Ads + GA4.

Use o template em `reports/templates/semanal.md` como estrutura.

## Passos

1. `get_campaign_performance` — últimos 7 dias
2. `get_campaign_performance` — 7 dias anteriores (para comparação)
3. `analyze_campaign_conversions` — dados cruzados Ads + GA4 com GDPR gap
4. `get_keyword_performance` — top keywords por gasto
5. `get_search_terms` — identificar termos irrelevantes e promissores
6. `get_negative_keywords` — verificar o que já está bloqueado (evitar duplicatas)

## Regras

- Calcular variação % semana vs semana para cada KPI
- Destacar com ⚠️ qualquer campanha com gasto > R$200 e 0 conversões
- Destacar com ⚠️ qualquer keyword com QS < 5
- Listar search terms irrelevantes como candidatos a negativa
- Listar search terms com boa conversão como candidatos a keyword
- Calcular gasto desperdiçado em termos irrelevantes
- Considerar GDPR antes de diagnosticar tracking como quebrado

## Output

Salvar o relatório em `reports/weekly/{ANO}-W{SEMANA}.md` usando o formato do template.
Apresentar um resumo executivo ao usuário com as 3 ações mais importantes.
