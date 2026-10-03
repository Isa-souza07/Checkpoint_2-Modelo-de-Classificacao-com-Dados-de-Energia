# APIs, energias renováveis e aprendizado de máquina

Duas tarefas independentes de aprendizado de máquina com dados públicos de energia renovável, cada uma comparando **três algoritmos**:

| Tarefa | Problema | Fonte | Algoritmos |
|---|---|---|---|
| 1 | Classificar a fonte do empreendimento (Solar, Eólica, Hidráulica) | ANEEL — SIGA | Regressão Logística, kNN, Random Forest |
| 2 | Estimar a radiação solar horária em Petrolina (PE) | Open-Meteo (histórico) | Regressão Linear, Árvore de Decisão, Random Forest |

## Objetivo

1. Consultar duas APIs públicas e gerar os arquivos `aneel_classificacao_orange.csv` e `meteo_regressao_orange.csv`.
2. **Classificação:** verificar se potência outorgada e localização bastam para identificar a fonte de um empreendimento de geração.
3. **Regressão:** estimar a radiação solar global horizontal (W/m²) a partir de condições meteorológicas e da hora do dia.

## Origem e período dos dados

| Item | Tarefa 1 — ANEEL | Tarefa 2 — Open-Meteo |
|---|---|---|
| API | [SIGA — ANEEL](https://dadosabertos.aneel.gov.br/dataset/siga-sistema-de-informacoes-de-geracao-da-aneel) (CKAN/DataStore) | [API histórica Open-Meteo](https://open-meteo.com/en/docs/historical-weather-api) |
| Autenticação | Não exige token | Não exige token |
| Recorte | Até 1.200 registros por sigla: `UFV` (Solar), `EOL` (Eólica), `UHE`/`PCH`/`CGH` (Hidráulica) | Petrolina (PE), −9,39 / −40,50, **01/04/2025 a 30/06/2025**, fuso `America/Recife`, horas de 7h a 17h |
| Tamanho final | 3.876 empreendimentos (1.200 Solar, 1.200 Eólica, 1.476 Hidráulica) | 1.001 horas |
| Entradas (X) | `potencia_kw`, `latitude`, `longitude` | `temperatura_c`, `umidade_pct`, `nuvens_pct`, `vento_kmh`, `hora` |
| Alvo (y) | `fonte` | `radiacao_w_m2` |

**Cuidados com os dados**

- A potência da ANEEL é **outorgada** (capacidade autorizada), não energia gerada. O cadastro mistura empreendimentos em fases diferentes.
- Como a consulta limita os registros por sigla, as proporções entre classes **não representam a matriz elétrica brasileira**.
- Nome, código CEG, sigla e descrição **não** entram em X, pois revelariam a resposta.
- Os valores do Open-Meteo vêm de modelos/reanálise, não de um painel fotovoltaico. `data_hora` serve apenas para ordenar e dividir no tempo, e `radiacao_w_m2` não é usada para criar nenhuma entrada.

## Estrutura do repositório

```
├── README.md
├── Avaliacao_APIs_Energia_Renovavel_ML.ipynb   # notebook completo, executável na ordem
├── aneel_classificacao_orange.csv              # dados da Tarefa 1
└── meteo_regressao_orange.csv                  # dados da Tarefa 2
```

## Como executar

**No Google Colab (recomendado)**

1. Abra o notebook no Colab (*Arquivo → Abrir notebook → GitHub* e cole o link do repositório).
2. Use *Ambiente de execução → Executar tudo*.

**Localmente**

```bash
pip install pandas numpy matplotlib seaborn scikit-learn jupyter
jupyter notebook Avaliacao_APIs_Energia_Renovavel_ML.ipynb
```

O notebook consulta as APIs e regrava os CSVs. Se alguma consulta falhar, ele avisa e usa o CSV de contingência: o arquivo local, se existir, ou o do repositório do professor. Nenhuma credencial é necessária ou deve ser publicada.

## Metodologia

### Tarefa 1 — Classificação

- Divisão **estratificada 80% / 20%** (`random_state=42`), a mesma para os três modelos.
- Regressão Logística e kNN usam `StandardScaler` dentro de um `Pipeline`, ajustado **somente no treino**. Random Forest não exige padronização.
- Configurações: Regressão Logística (`max_iter=2000`), kNN (`k=5`), Random Forest (200 árvores).
- Métricas: Accuracy e Precision, Recall e F1 com média **macro** (todas as classes pesam igual). O F1 *weighted* é informado como complemento. Também há relatório por classe e matriz de confusão.

### Tarefa 2 — Regressão

- Divisão **temporal**, sem embaralhar: as primeiras 80% das horas para treino (até 12/06/2025 14h) e as 20% finais para teste (12/06 15h a 30/06/2025).
- Mesma divisão para os três modelos. A Regressão Linear usa `StandardScaler` no `Pipeline`.
- Configurações: Regressão Linear, Árvore de Decisão (`max_depth=6`), Random Forest (300 árvores). Semente 42.
- Métricas: MAE (W/m²), MSE ((W/m²)²), RMSE (W/m²) e R². Gráficos de real × previsto, série temporal do período de teste, importância das variáveis e erro por hora.

## Resultados

Valores da execução de referência (conjunto de teste, `random_state=42`). Podem variar levemente se a API devolver dados atualizados.

### Tarefa 1 — Classificação (776 exemplos de teste)

| Modelo | Accuracy | Precision (macro) | Recall (macro) | F1 (macro) | F1 (weighted) |
|---|---|---|---|---|---|
| Regressão Logística | 0,825 | 0,828 | 0,821 | 0,820 | 0,823 |
| kNN (k=5) | 0,965 | 0,966 | 0,964 | 0,965 | 0,965 |
| **Random Forest** | **0,976** | **0,977** | **0,974** | **0,975** | **0,975** |

### Tarefa 2 — Regressão (últimas 20% das horas)

| Modelo | MAE (W/m²) | MSE ((W/m²)²) | RMSE (W/m²) | R² |
|---|---|---|---|---|
| Regressão Linear | 145,2 | 30.034 | 173,3 | 0,360 |
| Árvore de Decisão (prof. 6) | 90,9 | 15.127 | 123,0 | 0,678 |
| **Random Forest** | **66,4** | **7.210** | **84,9** | **0,846** |

## Conclusões

### Tarefa 1 — Classificação

- **Random Forest** teve o melhor resultado em todas as métricas, seguido de perto pelo **kNN**. Como a diferença entre eles é pequena, outra semente ou divisão poderia inverter a ordem. Ambos ficaram bem acima da Regressão Logística.
- A localização separa as fontes por **regiões**, uma estrutura não linear que a fronteira linear da Regressão Logística não representa.
- **Classes mais confundidas (Random Forest):** Solar prevista como Hidráulica (7 casos), Solar como Eólica (5) e Eólica como Hidráulica (4). A Solar tem o menor recall: usinas fotovoltaicas, em geral de potência pequena e espalhadas por muitas regiões, se misturam às demais.
- **Limitações:**
  - Potência outorgada é capacidade cadastrada, não geração.
  - Só há três atributos, sem altitude, hidrografia, vento ou irradiação.
  - A amostra é limitada por sigla e não reflete a matriz brasileira.
  - Empreendimentos vizinhos, como complexos com várias unidades, podem cair no treino e no teste ao mesmo tempo. Isso favorece kNN e Random Forest, e a acurácia pode ser otimista para localizações novas.
  - Foi usada uma única divisão treino/teste, sem validação cruzada.

### Tarefa 2 — Regressão

- **Random Forest** foi o melhor (R² ≈ 0,85; MAE ≈ 66 W/m²). A **Regressão Linear** foi a pior (R² ≈ 0,36), porque a radiação não é linear na hora do dia (curva em sino) e depende de interações entre variáveis.
- **Papel da hora:** no Random Forest, `hora` é a variável mais importante (≈ 0,49), seguida de `temperatura_c` (≈ 0,30) e `umidade_pct` (≈ 0,18). `nuvens_pct` e `vento_kmh` pesam pouco (≈ 0,02 cada). A hora define o ângulo do Sol e, portanto, o limite físico de radiação. Como a relação é não linear, a hora quase não aparece na correlação de Pearson (≈ 0,12), mas é decisiva nos modelos de árvore.
- **Erros:** são maiores nas horas centrais do dia, quando os valores de radiação são altos e a nebulosidade pode alterá-los em centenas de W/m².
- **Divisão temporal:** treino e teste são períodos consecutivos, e a radiação cai com a chegada do inverno. Isso simula prever o futuro, mas cobre só três meses de uma única estação, então os resultados não valem para o ano todo.
- **Radiação não é geração elétrica:** a radiação (W/m²) é uma estimativa de reanálise para a superfície **horizontal** em um ponto. A geração fotovoltaica depende também de inclinação e orientação dos painéis, área e eficiência, temperatura da célula, sombreamento, sujeira, perdas no inversor e na rede, degradação e disponibilidade. Estimar a radiação é uma etapa para estimar energia (kWh), não a previsão da geração.

## Fontes

- [ANEEL — SIGA](https://dadosabertos.aneel.gov.br/dataset/siga-sistema-de-informacoes-de-geracao-da-aneel)
- [ANEEL — recurso e campos usados](https://dadosabertos.aneel.gov.br/dataset/siga-sistema-de-informacoes-de-geracao-da-aneel/resource/11ec447d-698d-4ab8-977f-b424d5deee6a)
- [Open-Meteo — API histórica](https://open-meteo.com/en/docs/historical-weather-api)
