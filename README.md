# Elasticidade-Preço da Demanda de Abacate (Califórnia)

Estudo aplicado de econometria estimando a elasticidade-preço da demanda de abacate no mercado da Califórnia, comparando o comportamento entre abacates convencionais e orgânicos.

## Objetivo

Responder à pergunta: **quão sensível é a quantidade de abacate demandada a variações no seu preço?**, e verificar se essa sensibilidade difere entre o segmento convencional e o orgânico.

## Base de dados

- **Fonte:** [Avocado Prices](https://www.kaggle.com/datasets/neuromusic/avocado-prices) (Hass Avocado Board), disponibilizada publicamente no Kaggle.
- **Arquivo:** `avocado_prices.csv`
- **Granularidade original:** 18.249 observações semanais, 54 regiões dos EUA, 2015–2018, com preço médio (`AveragePrice`) e volume vendido (`Total Volume`), segmentados por tipo (`conventional` / `organic`).
- **Recorte usado neste estudo:** apenas a região `California`, resultando em 338 observações (169 semanas × 2 tipos).

## Metodologia

1. **Filtragem e limpeza:** seleção da região Califórnia, conversão da coluna `Date` para datetime, ordenação cronológica e separação do conjunto em dois subgrupos — `df_conv` (convencional) e `df_org` (orgânico) — já que se espera que os dois produtos tenham elasticidades distintas.

2. **Transformação log-log:** aplicação de logaritmo natural sobre preço (`AveragePrice`) e quantidade (`Total Volume`) para cada subgrupo. Em um modelo log-log, o coeficiente angular da regressão corresponde diretamente à elasticidade-preço da demanda, pois representa a variação percentual em Y para cada variação percentual em X.

3. **Regressão OLS:** estimação de `log(Quantidade) ~ log(Preço)` via `statsmodels.OLS`, separadamente para cada tipo.

4. **Correção de erro padrão (HAC / Newey-West):** o teste de Durbin-Watson nos dois modelos indicou autocorrelação positiva forte nos resíduos (esperado em dados semanais, já que choques de oferta/demanda persistem por várias semanas). Para evitar subestimar os erros padrão — e, com isso, inflar artificialmente a significância estatística —, os modelos foram reestimados com erros padrão robustos a heterocedasticidade e autocorrelação (HAC, 4 defasagens).

## Resultados

| | Convencional | Orgânico |
|---|---|---|
| Elasticidade-preço (coef. `log_price`) | **-0,68** | **-0,83** |
| Erro padrão (HAC) | 0,092 | 0,184 |
| p-valor (HAC) | < 0,001 | < 0,001 |
| R² | 0,504 | 0,228 |
| Durbin-Watson | 0,657 | 0,207 |
| N | 169 | 169 |

**Leitura:** um aumento de 1% no preço reduz a quantidade demandada em aproximadamente 0,68% no convencional e 0,83% no orgânico. Ambos os coeficientes são estatisticamente significativos mesmo após a correção HAC, e o sinal negativo é consistente com a teoria (lei da demanda).

**Comparação:** o abacate **orgânico se mostrou mais elástico** que o convencional — os consumidores desse segmento reagem proporcionalmente mais a mudanças de preço. Ainda assim, tecnicamente os dois produtos permanecem na faixa inelástica (|elasticidade| < 1), com o orgânico mais próximo da elasticidade unitária.

O R² bem mais baixo no orgânico (0,228 vs. 0,504) indica que o preço explica uma fração menor da variação da quantidade demandada nesse segmento — outros fatores (renda do consumidor, disponibilidade em loja, sazonalidade, marketing) provavelmente têm peso maior ali.

## Limitações

- Modelo simples de duas variáveis: não controla fatores como sazonalidade, renda, preço de substitutos ou promoções, que podem confundir a relação estimada.
- Amostra relativamente pequena (169 semanas por grupo) e restrita a uma única região.
- A correção HAC ajusta os erros padrão, mas não resolve endogeneidade (por exemplo, preço e quantidade sendo determinados simultaneamente pelo mercado).

## Como reproduzir

```bash
pip install pandas numpy matplotlib statsmodels
jupyter notebook elasticidade_abacates.ipynb
```

Os arquivos `avocado_prices.csv` e `elasticidade_abacates.ipynb` devem estar na mesma pasta.

## Ferramentas

Python, pandas, numpy, matplotlib, statsmodels.

---

*Autor: Luca Bempack