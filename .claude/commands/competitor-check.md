Analisar concorrência e posicionamento no Google Ads.

## Dados disponíveis hoje

Usar `run_gaql` para dados de impression share:

```sql
SELECT campaign.name, metrics.search_impression_share,
       metrics.search_rank_lost_impression_share_budget,
       metrics.search_rank_lost_impression_share_rank,
       metrics.search_top_impression_percentage,
       metrics.search_absolute_top_impression_percentage
FROM campaign
WHERE segments.date DURING LAST_30_DAYS AND campaign.status = 'ENABLED'
ORDER BY metrics.search_impression_share DESC
```

## Análise

- **Impression Share**: % de vezes que aparecemos vs oportunidades. <50% = estamos perdendo metade do tráfego.
- **Lost to Budget**: perdendo impressões por budget insuficiente → considerar aumentar.
- **Lost to Rank**: perdendo por Quality Score/bid baixo → melhorar QS ou bid.
- **Top Impression %**: % de vezes que aparecemos no topo. <30% = competidores passando na frente.

## Quando Auction Insights estiver disponível (Sprint 3)

Usar `get_auction_insights` para ver:
- Quem são os concorrentes
- Overlap rate (frequência que aparecemos juntos)
- Position above rate (quem fica na frente)
- Outranking share (quem ganha mais vezes)
- Tendência temporal

## Output

Relatório com:
1. Impression share por campanha
2. Onde estamos perdendo (budget vs rank)
3. Estimativa de tráfego perdido
4. Recomendações para ganhar share
