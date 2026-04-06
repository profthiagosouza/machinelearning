
# Algoritmos de Machine Learning Baseados em Árvore com R

---

# 1. Introdução aos Algoritmos Baseados em Árvore

Algoritmos baseados em árvore são modelos de Machine Learning supervisionado que tomam decisões por meio de uma sequência de regras lógicas. Essas regras dividem os dados em subconjuntos cada vez mais homogêneos até chegar a uma previsão final.

A ideia central é simples: o algoritmo procura responder, em sequência, perguntas do tipo:

- uma variável é menor ou maior que certo valor?
- uma observação pertence a um grupo específico?
- qual divisão produz subconjuntos mais “puros”?

Essa lógica gera uma estrutura semelhante a um fluxograma.

## 1.1 Estrutura básica de uma árvore

Uma árvore é composta por:

- **Nó raiz:** primeiro ponto de decisão, contendo todo o conjunto de dados
- **Nós internos:** pontos intermediários em que novas divisões são feitas
- **Ramos:** caminhos resultantes das decisões
- **Folhas:** nós terminais, onde está a classe prevista ou o valor estimado

Exemplo conceitual:

```text
Se Petal.Length <= 2.45 → classificar como setosa
Se Petal.Length > 2.45 → continuar dividindo
```

## 1.2 Como o algoritmo decide onde dividir?

Em classificação, o modelo procura divisões que tornem os grupos mais homogêneos em relação à variável alvo. Para isso, utiliza métricas como:

- **Índice de Gini**
- **Entropia**
- **Ganho de informação**

A lógica é: uma boa divisão é aquela que separa melhor as classes.

## 1.3 Vantagens gerais

Modelos baseados em árvore têm várias características úteis:

- são fáceis de interpretar;
- capturam relações não lineares;
- não exigem normalização das variáveis;
- funcionam bem com variáveis numéricas e categóricas;
- servem como base conceitual para métodos mais avançados.

## 1.4 Limitações gerais

Apesar de úteis, esses modelos também têm limitações:

- árvores simples podem sofrer overfitting;
- alguns métodos podem perder interpretabilidade;
- algoritmos ensemble podem exigir maior custo computacional.

## 1.5 Principais algoritmos estudados

Nesta apostila, serão abordados três algoritmos principais:

- **Árvore de Decisão**
- **Random Forest**
- **XGBoost**

Esses métodos formam uma progressão natural de complexidade:

- a **Árvore de Decisão** é o modelo mais interpretável;
- o **Random Forest** melhora robustez e precisão ao combinar muitas árvores;
- o **XGBoost** introduz boosting e refinamento iterativo para maximizar desempenho.

---

# 2. Separação de Dados

A separação entre treino e teste é uma etapa essencial em Machine Learning. Ela permite avaliar se o modelo aprendeu padrões gerais ou se apenas decorou os dados utilizados no treinamento.

## 2.1 Objetivo da separação

A ideia é dividir o dataset em duas partes:

- **conjunto de treino:** usado para ajustar o modelo;
- **conjunto de teste:** usado para avaliar o desempenho em dados não vistos.

Uma divisão bastante comum é:

- 70% para treino
- 30% para teste

## 2.2 Por que essa etapa é importante?

Sem essa separação, o modelo pode parecer excelente simplesmente porque está sendo avaliado nos mesmos dados que utilizou para aprender. Isso gera uma falsa sensação de desempenho.

## 2.3 Separação estratificada com `caret`

No dataset `iris`, temos três classes com a mesma quantidade de observações. Ainda assim, é boa prática usar amostragem estratificada para preservar essa proporção nos subconjuntos.

```r
library(caret)

data(iris)

set.seed(123)

indice_treino <- createDataPartition(
  y = iris$Species,
  p = 0.7,
  list = FALSE
)

treino <- iris[indice_treino, ]
teste  <- iris[-indice_treino, ]
```

## 2.4 Verificação da divisão

```r
dim(treino)
dim(teste)

prop.table(table(iris$Species))
prop.table(table(treino$Species))
prop.table(table(teste$Species))
```

Essas verificações ajudam a confirmar se a divisão foi realizada corretamente.

---

# 3. Overfitting

## 3.1 Definição

**Overfitting** ocorre quando o modelo aprende excessivamente os detalhes do conjunto de treino, incluindo ruídos e particularidades que não representam padrões gerais.

Em vez de aprender a regra geral do problema, o modelo “memoriza” o treino.

## 3.2 Intuição didática

Imagine um aluno que decorou exatamente as respostas da prova passada, mas não entendeu o conteúdo. Se a prova mudar, ele terá dificuldade. Em Machine Learning, isso significa:

- desempenho muito bom no treino;
- desempenho ruim no teste.

## 3.3 Exemplo típico

| Conjunto | Accuracy |
|----------|----------|
| Treino   | 99%      |
| Teste    | 65%      |

Esse comportamento é um forte indício de overfitting.

## 3.4 Overfitting em árvores

Em árvores de decisão, o overfitting costuma ocorrer quando:

- a árvore cresce demais;
- há muitas divisões;
- o modelo cria regras excessivamente específicas.

Exemplo conceitual:

```text
Se Petal.Length <= 2.45
Se Sepal.Width <= 3.15
Se Petal.Width <= 0.23
→ classe = setosa
```

Quanto mais específicas forem as regras, maior o risco de o modelo se ajustar demais ao treino.

## 3.5 Principais causas

- modelo muito complexo;
- poucos dados;
- presença de ruído;
- parâmetros excessivamente flexíveis;
- falta de validação adequada.

## 3.6 Como reduzir overfitting

Algumas estratégias importantes:

- limitar profundidade da árvore;
- podar a árvore;
- usar ensemble learning;
- realizar validação cruzada;
- ajustar hiperparâmetros.

---

# 4. Árvores de Decisão

## 4.1 Definição

A **Árvore de Decisão** é um modelo supervisionado que organiza regras de decisão em uma estrutura hierárquica. Em classificação, o objetivo é separar as observações em grupos cada vez mais homogêneos até chegar à classe final.

No contexto do dataset `iris`, a árvore procura regras que permitam distinguir as espécies a partir de medidas das sépalas e pétalas.

## 4.2 Intuição conceitual

A árvore funciona como uma sequência de perguntas. Por exemplo:

- o comprimento da pétala é menor ou igual a certo valor?
- se sim, vá para a esquerda;
- se não, vá para a direita.

Essa lógica é repetida até que o modelo chegue a uma folha terminal.

## 4.3 Quando usar

A Árvore de Decisão é especialmente útil quando se deseja:

- forte interpretabilidade;
- entendimento das regras de decisão;
- exploração inicial do problema;
- base conceitual para algoritmos mais avançados.

## 4.4 Vantagens

- fácil interpretação;
- visualização intuitiva;
- não exige normalização;
- captura relações não lineares.

## 4.5 Limitações

- pode sofrer overfitting;
- pode ser instável a pequenas variações nos dados;
- costuma ter menor desempenho que métodos ensemble.

## 4.6 Pacotes necessários

```r
library(rpart)
library(rpart.plot)
library(caret)
```

## 4.7 Preparação dos dados

Usaremos a separação treino/teste já apresentada na Seção 2.

## 4.8 Treinamento do modelo

```r
modelo_tree <- rpart(
  Species ~ .,
  data = treino,
  method = "class"
)
```

### Explicação

- `Species ~ .` indica que queremos prever `Species` com todas as outras variáveis;
- `data = treino` define o conjunto de treinamento;
- `method = "class"` informa que se trata de um problema de classificação.

## 4.9 Visualização do modelo

```r
rpart.plot(modelo_tree)
```

Ao observar a árvore, é possível identificar:

- qual variável aparece na primeira divisão;
- quais valores de corte foram escolhidos;
- quais classes predominam em cada folha.

## 4.10 Previsão

```r
pred_tree <- predict(
  modelo_tree,
  teste,
  type = "class"
)
```

## 4.11 Avaliação

```r
confusionMatrix(
  pred_tree,
  teste$Species
)
```

## 4.12 Interpretação dos resultados

Na árvore ajustada ao `iris`, geralmente as variáveis mais relevantes são:

- `Petal.Length`
- `Petal.Width`

Isso acontece porque essas medidas ajudam muito a separar as espécies.

## 4.13 Código completo — Árvore de Decisão

```r
# Pacotes
library(rpart)
library(rpart.plot)
library(caret)

# Dados
data(iris)

# Separação treino/teste
set.seed(123)
indice_treino <- createDataPartition(
  y = iris$Species,
  p = 0.7,
  list = FALSE
)

treino <- iris[indice_treino, ]
teste  <- iris[-indice_treino, ]

# Treinamento
modelo_tree <- rpart(
  Species ~ .,
  data = treino,
  method = "class"
)

# Visualização
rpart.plot(modelo_tree)

# Previsão
pred_tree <- predict(
  modelo_tree,
  teste,
  type = "class"
)

# Avaliação
confusionMatrix(
  pred_tree,
  teste$Species
)
```

---

# 5. Random Forest

## 5.1 Definição

O **Random Forest** é um método ensemble que combina várias árvores de decisão. Em vez de confiar em uma única árvore, o algoritmo constrói muitas árvores e combina suas previsões.

Em problemas de classificação, a decisão final normalmente é feita por **votação majoritária**.

## 5.2 Intuição conceitual

A lógica é a seguinte:

- criar várias árvores;
- treinar cada uma com subconjuntos aleatórios dos dados;
- fazer com que cada árvore “vote” na classe prevista;
- escolher a classe mais votada.

Esse processo reduz a instabilidade de uma árvore única e melhora a robustez do modelo.

## 5.3 Quando usar

O Random Forest é indicado quando se deseja:

- melhorar desempenho preditivo;
- reduzir overfitting;
- manter um modelo relativamente fácil de aplicar;
- obter medidas de importância das variáveis.

## 5.4 Vantagens

- alta robustez;
- menor risco de overfitting que árvore única;
- bom desempenho com poucos ajustes;
- mede importância das variáveis.

## 5.5 Limitações

- menor interpretabilidade que uma árvore única;
- maior custo computacional;
- o conjunto de árvores não é facilmente visualizado como uma única estrutura.

## 5.6 Pacotes necessários

```r
library(randomForest)
library(caret)
```

## 5.7 Preparação dos dados

Usaremos a mesma separação treino/teste definida anteriormente.

## 5.8 Treinamento do modelo

```r
modelo_rf <- randomForest(
  Species ~ .,
  data = treino,
  ntree = 500,
  mtry = 2,
  importance = TRUE
)
```

### Explicação

- `ntree = 500` define a quantidade de árvores;
- `mtry = 2` define quantas variáveis podem ser testadas em cada divisão;
- `importance = TRUE` solicita o cálculo da importância das variáveis.

## 5.9 Inspeção do modelo

```r
print(modelo_rf)
```

Essa saída normalmente mostra:

- tipo do modelo;
- número de árvores;
- erro OOB (_out-of-bag_);
- matriz de confusão interna.

## 5.10 Previsão

```r
previsoes_rf <- predict(
  modelo_rf,
  newdata = teste
)
```

## 5.11 Avaliação

```r
table(
  Real = teste$Species,
  Previsto = previsoes_rf
)

mean(previsoes_rf == teste$Species)
```

## 5.12 Importância das variáveis

```r
importance(modelo_rf)
varImpPlot(modelo_rf)
```

No `iris`, geralmente `Petal.Length` e `Petal.Width` aparecem como as variáveis mais importantes.

## 5.13 Interpretação dos resultados

O Random Forest tende a apresentar desempenho superior ao da árvore única porque reduz a variabilidade do modelo. Em vez de depender de uma única estrutura, ele combina o conhecimento de várias árvores.

## 5.14 Código completo — Random Forest

```r
# Pacotes
library(randomForest)
library(caret)

# Dados
data(iris)

# Separação treino/teste
set.seed(123)
indice_treino <- createDataPartition(
  y = iris$Species,
  p = 0.7,
  list = FALSE
)

treino <- iris[indice_treino, ]
teste  <- iris[-indice_treino, ]

# Treinamento
modelo_rf <- randomForest(
  Species ~ .,
  data = treino,
  ntree = 500,
  mtry = 2,
  importance = TRUE
)

# Inspeção do modelo
print(modelo_rf)

# Previsão
previsoes_rf <- predict(
  modelo_rf,
  newdata = teste
)

# Avaliação
table(
  Real = teste$Species,
  Previsto = previsoes_rf
)

mean(previsoes_rf == teste$Species)

# Importância das variáveis
importance(modelo_rf)
varImpPlot(modelo_rf)
```

---

# 6. XGBoost

## 6.1 Definição

O **XGBoost** é um algoritmo de boosting baseado em árvores de decisão. Diferentemente do Random Forest, em que as árvores são construídas de maneira mais independente, no boosting as árvores são construídas em sequência.

Cada nova árvore tenta corrigir os erros cometidos pelas árvores anteriores.

## 6.2 Intuição conceitual

A lógica do XGBoost é:

- construir uma primeira árvore;
- analisar os erros;
- construir outra árvore para corrigir esses erros;
- repetir esse processo várias vezes.

Assim, o modelo melhora iterativamente.

## 6.3 Quando usar

O XGBoost é indicado quando se deseja:

- alto desempenho preditivo;
- maior controle sobre hiperparâmetros;
- modelar relações complexas;
- trabalhar com problemas mais desafiadores.

## 6.4 Vantagens

- alta performance;
- grande flexibilidade;
- controle refinado dos parâmetros;
- muito utilizado em competições e aplicações reais.

## 6.5 Limitações

- maior complexidade de configuração;
- menor interpretabilidade que árvore única;
- pode exigir mais cuidado no ajuste de hiperparâmetros.

## 6.6 Pacotes necessários

```r
library(xgboost)
library(caret)
```

## 6.7 Preparação dos dados

No XGBoost, a variável alvo deve ser convertida em valores numéricos.

```r
treino$Species <- as.numeric(treino$Species) - 1
teste$Species  <- as.numeric(teste$Species) - 1
```

Separando atributos e rótulos:

```r
x_treino <- as.matrix(treino[, 1:4])
y_treino <- treino$Species

x_teste <- as.matrix(teste[, 1:4])
y_teste <- teste$Species
```

Criando matrizes próprias do XGBoost:

```r
dtreino <- xgb.DMatrix(
  data = x_treino,
  label = y_treino
)

dteste <- xgb.DMatrix(
  data = x_teste,
  label = y_teste
)
```

## 6.8 Treinamento do modelo

```r
parametros <- list(
  objective = "multi:softmax",
  num_class = 3,
  eval_metric = "merror",
  max_depth = 3,
  eta = 0.1
)

modelo_xgb <- xgb.train(
  params = parametros,
  data = dtreino,
  nrounds = 50,
  verbose = 0
)
```

### Explicação

- `objective = "multi:softmax"` define classificação multiclasse;
- `num_class = 3` informa que há três espécies;
- `eval_metric = "merror"` define a métrica de erro;
- `max_depth = 3` controla profundidade;
- `eta = 0.1` define a taxa de aprendizado;
- `nrounds = 50` define o número de iterações.

## 6.9 Previsão

```r
previsoes_xgb <- predict(
  modelo_xgb,
  dteste
)
```

## 6.10 Avaliação

```r
table(
  Real = y_teste,
  Previsto = previsoes_xgb
)

mean(previsoes_xgb == y_teste)
```

## 6.11 Conversão das classes para nomes

```r
rotulos <- c("setosa", "versicolor", "virginica")

real_nomes <- rotulos[y_teste + 1]
previsto_nomes <- rotulos[previsoes_xgb + 1]

table(
  Real = real_nomes,
  Previsto = previsto_nomes
)
```

## 6.12 Interpretação dos resultados

O XGBoost costuma alcançar excelente desempenho no `iris`, especialmente porque consegue refinar sucessivamente as divisões das árvores. Ainda assim, a interpretação do processo é menos direta do que em uma árvore de decisão.

## 6.13 Código completo — XGBoost

```r
# Pacotes
library(xgboost)
library(caret)

# Dados
data(iris)

# Separação treino/teste
set.seed(123)
indice_treino <- createDataPartition(
  y = iris$Species,
  p = 0.7,
  list = FALSE
)

treino <- iris[indice_treino, ]
teste  <- iris[-indice_treino, ]

# Conversão da variável alvo
treino$Species <- as.numeric(treino$Species) - 1
teste$Species  <- as.numeric(teste$Species) - 1

# Separação entre atributos e alvo
x_treino <- as.matrix(treino[, 1:4])
y_treino <- treino$Species

x_teste <- as.matrix(teste[, 1:4])
y_teste <- teste$Species

# Criação das DMatrix
dtreino <- xgb.DMatrix(
  data = x_treino,
  label = y_treino
)

dteste <- xgb.DMatrix(
  data = x_teste,
  label = y_teste
)

# Parâmetros
parametros <- list(
  objective = "multi:softmax",
  num_class = 3,
  eval_metric = "merror",
  max_depth = 3,
  eta = 0.1
)

# Treinamento
modelo_xgb <- xgb.train(
  params = parametros,
  data = dtreino,
  nrounds = 50,
  verbose = 0
)

# Previsão
previsoes_xgb <- predict(
  modelo_xgb,
  dteste
)

# Avaliação
table(
  Real = y_teste,
  Previsto = previsoes_xgb
)

mean(previsoes_xgb == y_teste)

# Conversão para nomes
rotulos <- c("setosa", "versicolor", "virginica")

real_nomes <- rotulos[y_teste + 1]
previsto_nomes <- rotulos[previsoes_xgb + 1]

table(
  Real = real_nomes,
  Previsto = previsto_nomes
)
```

---

# 7. Comparação Geral dos Modelos

## 7.1 Comparação conceitual

| Modelo | Ideia central |
|--------|---------------|
| Árvore de Decisão | Uma única árvore cria regras de decisão |
| Random Forest | Muitas árvores votam pela classe final |
| XGBoost | Árvores sequenciais corrigem erros anteriores |

## 7.2 Comparação prática

| Modelo | Interpretabilidade | Precisão | Risco de Overfitting | Complexidade |
|--------|--------------------|----------|----------------------|--------------|
| Árvore de Decisão | Alta | Média | Alto | Baixa |
| Random Forest | Média | Alta | Menor | Média |
| XGBoost | Média | Muito alta | Controlável | Alta |

## 7.3 Progressão didática

Uma forma natural de estudar esses modelos é:

1. começar com **Árvore de Decisão**;
2. evoluir para **Random Forest**;
3. avançar para **XGBoost**.

Essa sequência ajuda a compreender a evolução dos métodos baseados em árvore.

---

# 8. Resumo Geral

Os algoritmos baseados em árvore são extremamente importantes em Machine Learning porque unem boa capacidade preditiva com uma lógica de decisão intuitiva.

Neste material, vimos que:

- a **Árvore de Decisão** é excelente para interpretação;
- o **Random Forest** melhora robustez e precisão;
- o **XGBoost** oferece alto desempenho por meio de boosting.

Fluxo geral de trabalho:

1. carregar os dados;
2. separar treino e teste;
3. treinar o modelo;
4. fazer previsões;
5. avaliar o desempenho;
6. comparar resultados.

Esses três algoritmos são fundamentais tanto para fins didáticos quanto para aplicações reais em ciência de dados.

