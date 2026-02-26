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
No *backtest* de cenários como Manchester City x Arsenal, o modelo dinâmico ajustou a probabilidade de vitória do mandante, identificando um xG menor do que a média estática sugeria devido à forte defesa recente do visitante. Isso provou matematicamente o "Valor Negativo" de apostar no favorito naquela rodada específica.

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
* **Resultado Final:** O robô reduziu o número de entradas (apenas 103 apostas filtradas), entregando um ROI de **-2.91%**. Apesar do leve prejuízo nominal, o modelo provou capacidade de cortar grande parte da vantagem matemática da casa de apostas, servindo como uma base sólida para futuras adições de variáveis (como desfalques ou *Expected Goals* baseados em finalizações).