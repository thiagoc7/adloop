Gerar relatório mensal de performance do Google Ads + GA4.

Use o template em `reports/templates/mensal.md` como estrutura.

## Passos

1. `get_campaign_performance` — últimos 30 dias
2. `get_campaign_performance` — 30 dias anteriores (para comparação)
3. `analyze_campaign_conversions` — dados cruzados com GDPR gap e canais orgânicos
4. `landing_page_analysis` — performance das landing pages
5. `get_keyword_performance` — todas as keywords com quality scores
6. `get_ad_performance` — performance dos anúncios (headlines, CTR)
7. `get_search_terms` — termos completos do mês
8. `attribution_check` — verificar integridade do tracking

## Análises

- Tendência semanal dentro do mês (4 data points)
- Ranking de campanhas por eficiência (menor CPA)
- Top 10 keywords por conversão
- Keywords com gasto sem retorno
- Melhores e piores anúncios
- Landing pages: bounce rate, conversão
- Funil completo: impressões → clicks → sessões → WhatsApp clicks
- Paid vs Organic: qual canal converte melhor
- Impression share (via GAQL se disponível)

## Recomendações

Gerar lista priorizada de ações:
- **Alta**: problemas que estão custando dinheiro agora
- **Média**: otimizações que melhoram performance
- **Baixa**: melhorias para o próximo mês

## Output

Salvar em `reports/monthly/{ANO}-{MES}.md`.
Apresentar resumo executivo com top 5 insights e ações.
