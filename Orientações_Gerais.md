# Checkpoint 2 – Aplicações de Machine Learning para dados de energia <br>
Disciplina: Soluções em Energias Renováveis e Sustentáveis <br>
Turma: 1CCPQ <br>
Alunos: <br>
Artur Souza Pereira - RM 570880 <br>
Gustavo Gamba Zancopé - RM 569287 <br>
Isabela Camargo Souza - RM 569196 <br>
Miguel Silvério de Avila - RM 568873 <br>
Victor Vieira Galvão - RM 571483 <br>

Este repositório será utilizado para o desenvolvimento do Checkpoint 2, relacionado à aplicação de técnicas de Machine Learning em dados de estabilidade de redes elétricas.<br>
As atividades utilizarão como referência o conjunto de dados Electrical Grid Stability Simulated Data, disponibilizado pela UCI Machine Learning Repository.<br>
Fonte dos dados Aula 6 e 7: [Electrical Grid Stability Simulated Data – UCI](https://archive.ics.uci.edu/dataset/471/electrical+grid+stability+simulated+data) <br>
Fonte de dados Desafio Final: https://dadosabertos.aneel.gov.br/dataset/siga-sistema-de-informacoes-de-geracao-da-aneel
https://open-meteo.com/en/docs/historical-weather-api

### **Organização do Checkpoint**<br>
O Checkpoint será distribuído em quatro partes:

**Classificação (Aula 06)**
Desenvolvimento de um modelo de classificação utilizando Regressão Logística para prever a condição da rede elétrica.
variável target: stabf;
classes previstas: estável ou instável;
separação dos dados em treino e teste;
treinamento do modelo;
geração das previsões;
avaliação dos resultados por meio de métricas de classificação e matriz de confusão.

**Regressão (Aula 07)**
Desenvolvimento de modelos de Regressão Linear para prever o valor numérico da variável stab.<br>
Nesta etapa, deverão ser treinados e comparados dois modelos:
modelo utilizando as cinco variáveis com maior correlação absoluta com stab;
modelo utilizando todas as variáveis cujos nomes começam com tau ou g.
Os modelos deverão ser avaliados comparativamente por meio das métricas:
R²;
MAE;
MSE.
A análise deverá considerar os resultados dos dois modelos, identificando o efeito da seleção das variáveis sobre o desempenho das previsões.

**Desafio final**
Desenvolvimento de um desafio final que reunirá os conhecimentos trabalhados nas etapas anteriores.
O grupo deverá analisar os resultados obtidos, justificar as decisões tomadas durante o desenvolvimento.
