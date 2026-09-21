# Modelo Preditivo: Premier League com Poisson e EMA

Este projeto aplica conceitos de modelagem estatística para prever resultados de partidas da Premier League (futebol inglês), buscando ineficiências nas cotações (Odds) das casas de apostas.

## O Problema
Modelos estáticos que utilizam a média de gols da temporada inteira falham ao ignorar a forma recente das equipes. Uma equipe com média de 2.5 gols na temporada pode estar passando por uma seca de gols devido a lesões nas últimas rodadas.

## A Solução (Metodologia)
Desenvolvi um algoritmo em Python que:
1. Extrai dados históricos diretamente da base do Football-Data.
2. Calcula o "Poder de Ataque" e "Força de Defesa" de cada equipe.
3. Aplica **Média Móvel Exponencial (EMA)** para dar um peso maior (aprox. 86%) aos últimos 5 jogos, capturando a forma exata da equipe naquele momento.
4. Cruza esses dados usando a **Distribuição de Poisson** em uma matriz de probabilidades para simular o placar exato (xG) e calcular a Odd Justa (Fair Line).

## Tecnologias Utilizadas
* Python
* Pandas (Manipulação de dados e cálculo de EMA)
* SciPy (Cálculo de probabilidade com Poisson)

## Resultado Prático
No *backtest* de cenários como Manchester City x Arsenal, o modelo dinâmico ajustou a probabilidade de vitória do mandante, identificando um xG menor do que a média estática sugeria devido à forte defesa recente do visitante. No exemplo Manchester City × Arsenal, o modelo dinâmico indicou um xG do mandante menor do que a média estática sugeria. Vale registrar que esta primeira célula é uma demonstração do motor de cálculo, não uma previsão: ela usa a EMA sobre toda a temporada, incluindo jogos posteriores ao confronto. O backtest da Fase 2 é a versão temporalmente honesta.

## Fase 2: Backtesting e Fator de Segurança (Edge)

Para validar a utilidade prática do modelo no mundo real, desenvolvi um script de **Backtesting** iterativo simulando a temporada 2023/2024 inteira. O objetivo foi testar a rentabilidade do modelo contra as Odds reais de fechamento da casa de apostas Bet365.

### Engenharia do Backtest
Para garantir um teste cego e evitar o *Data Leakage* (vazamento de dados do futuro), o robô foi programado com regras rígidas:
1. **Período de Aquecimento (Burn-in):** O algoritmo ignora as primeiras 50 partidas da temporada, usando-as apenas para calibrar a Média Móvel Exponencial (EMA) das equipes.
2. **Cegueira do Futuro:** Para prever o jogo da rodada 15, o modelo recalcula todas as forças usando estritamente os dados das rodadas 1 a 14.
3. **Fator de Segurança (Edge):** Na Engenharia, não operamos no limite da tolerância. O robô foi programado para exigir um *Edge* mínimo de **10%** sobre a casa de apostas. Se a Odd Justa for 2.00, ele só aposta se o mercado pagar 2.20 ou mais.

### Resultados e Aprendizados
O teste validou a hipótese matemática, reduzindo drasticamente o prejuízo esperado pelas taxas das casas de apostas (o *Juice*).
* **O Problema:** Uma tentativa inicial de apostar em Empates e Visitantes resultou em perdas severas (ROI de -14%). O diagnóstico revelou a fraqueza da Distribuição de Poisson em modelar empates no futebol real (times recuam quando o jogo está empatado no fim).
* **O Rollback (Versão Otimizada):** O modelo foi restrito a buscar valor apenas na **Vitória do Mandante** com margem de 10%. 
* **Resultado Final:** O robô reduziu o número de entradas (apenas 103 apostas filtradas), entregando um ROI de **-2.91%**. Apesar do prejuízo nominal, o ROI de −2,91% ficou acima do retorno esperado de apostas aleatórias contra a margem da casa, que no mercado 1X2 costuma ficar entre −5% e −7%. Com 103 entradas, porém, o erro padrão do ROI é de cerca de 12 pontos percentuais, a diferença de 2 a 4 pontos não é estatisticamente significativa. Serve como indício de que o filtro de valor tem efeito, não como comprovação.

## Fase 3: Machine Learning (Random Forest) e a Lição do GIGO

Para superar as limitações estatísticas do modelo de Poisson (especialmente a dificuldade em prever empates), evoluímos o projeto para o campo da Inteligência Artificial, implementando um modelo de **Random Forest Classifier**.

### Engenharia de Variáveis (Feature Engineering) e Treinamento
* **As Pistas (Features):** Construímos um script iterativo para calcular a média móvel dos últimos 5 jogos de cada equipe (Gols Feitos e Sofridos), garantindo que a IA olhasse apenas para a "forma recente" e sem vazar dados do futuro.
* **O Alvo (Target):** Convertemos os resultados do mercado 1x2 (H, D, A) em classes numéricas (2, 1, 0).
* **Treino e Teste:** Dividimos a base de dados temporalmente (`shuffle=False`). A IA estudou os primeiros 80% do campeonato e foi testada nos 20% finais.
* **Odds Justas:** Utilizamos o método `.predict_proba()` para extrair a probabilidade exata calculada pela árvore de decisão para cada cenário, convertendo-as em "Odds da IA".

### Resultados e a Lição de Engenharia de Dados
O Backtest do modelo de Machine Learning (buscando um Edge de 10% em todos os mercados) resultou em um **ROI de -16.89%** na reta final da temporada.

**Por que a IA falhou em bater o mercado?**
A resposta está no princípio fundamental da Ciência de Dados: **GIGO (Garbage In, Garbage Out)**. 
Embora o Random Forest seja um algoritmo de ponta, nós o alimentamos apenas com "Gols". No futebol, o gol é um evento de alta variância. Um time pode dominar uma partida (20 chutes a gol) e empatar em 0x0. Para a IA atual, o desempenho foi ruim (0 gols). 
Isso prova metodologicamente que **nenhum modelo matemático avançado sobrevive à falta de profundidade de dados**. 

## Limitações Metodológicas Conhecidas Revisitando este projeto, identifiquei os seguintes problemas. Estão documentados aqui porque afetam diretamente a interpretação dos resultados acima. 

**1. Tamanho de amostra insuficiente para qualquer conclusão.** O backtest de Poisson gerou 103 entradas com ROI de −2,91%; o de Machine Learning, 61 entradas com −16,89%. Calculando o erro padrão do ROI a partir da taxa de acerto e da odd média dos acertos, os intervalos de confiança de 95% ficam entre −27% e +22% no primeiro caso, e entre −53% e +20% no segundo. Ambos contêm o zero com folga: nenhum dos dois resultados é distinguível de acaso. 

**2. A Distribuição de Poisson assume independência entre os gols das equipes.** A matriz multiplica `P(i gols do mandante)` por `P(j gols do visitante)` como eventos independentes. No futebol real existe correlação negativa em placares baixos, times recuam quando o jogo está empatado, e é exatamente por isso que o modelo falhou em prever empates. O tratamento padrão para isso é a correção de Dixon-Coles (1997), que ajusta as probabilidades dos placares 0-0, 1-0, 0-1 e 1-1 sobre o modelo de Poisson independente de Maher (1982). 

**3. A matriz de placares é truncada em 5 gols e as probabilidades não são renormalizadas.** Cerca de 1% da massa de probabilidade fica de fora, o que enviesa levemente todas as odds justas calculadas. A correção é dividir cada probabilidade pela soma total da matriz. 

**4. `predict_proba()` do Random Forest não produz probabilidade calibrada.** Converter essa saída diretamente em odd justa é incorreto: Random Forest é conhecidamente mal calibrado, tipicamente superconfiante. O tratamento seria Platt scaling ou regressão isotônica, implementado no projeto seguinte via `CalibratedClassifierCV`. 

**5. Nenhuma métrica de qualidade do modelo foi calculada.** `accuracy_score` é importado e nunca utilizado. Além disso, acurácia é a métrica errada para este problema: o que importa em aposta é ter probabilidade melhor calibrada que o mercado, não acertar mais jogos. As métricas corretas seriam Brier score ou log-loss.
