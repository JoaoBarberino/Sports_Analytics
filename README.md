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