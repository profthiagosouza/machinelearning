# Algoritmos Baseados em Árvore com R

## 1. Introdução aos Algoritmos Baseados em Árvore

Os modelos baseados em árvore utilizam uma estrutura de decisão lógica para segmentar os dados em grupos cada vez mais homogêneos. Imagine um fluxograma onde cada nó interno representa um "teste" em um atributo (ex: a largura da pétala), cada ramo representa o resultado do teste, e cada nó folha representa uma classe (espécie).

### Por que usar árvores?
* **Interpretabilidade:** É fácil entender o "porquê" de uma decisão.
* **Flexibilidade:** Não exigem que os dados sigam uma distribuição normal ou que estejam na mesma escala.
* **Não-linearidade:** Conseguem capturar relações complexas que modelos lineares simples ignoram.

Eles funcionam criando uma sequência de decisões baseadas em regras.

Exemplo conceitual:

Se Petal.Length <= 2.45 → Setosa  
Se Petal.Length > 2.45 → continuar divisão  

Cada divisão é chamada de:

* Nó
* Ramo
* Folha

### Características principais

Algoritmos baseados em árvore:

* Lidam bem com dados não lineares
* Não exigem normalização
* Funcionam com dados categóricos e numéricos
* São interpretáveis

### Principais algoritmos baseados em árvore

Nesta apostila estudaremos:

* Árvore de Decisão
* Random Forest
* XGBoost

---

## 2. Separação de Dados (Treino e Teste)

Para avaliar se o modelo realmente aprendeu ou se apenas decorou os dados, dividimos o dataset original. Usaremos o pacote `caret` para garantir que a proporção das espécies seja mantida em ambos os conjuntos (amostragem estratificada).

* Treino → usado para aprender
* Teste → usado para avaliar

Geralmente, separamos 70% dos dados para treino e 30% para teste, mas essa proporção depende do tamanho do _dataset_.

Essa etapa é essencial para evitar _Overfitting_. 

## 3. Overfitting em Machine Learning

### 🎯 Definição

**Overfitting** (ou **sobreajuste**) ocorre quando um modelo de Machine Learning aprende **demais os dados de treino**, incluindo **ruídos e detalhes específicos**, e por isso **performa mal em dados novos**.

---

### 🧠 Definição simples

**Overfitting ocorre quando o modelo memoriza os dados em vez de aprender padrões gerais.**

Ou seja:

- Vai **muito bem no treino** ✅  
- Vai **mal no teste** ❌  

---

### 🎓 Intuição didática

Imagine um aluno que:

- Memoriza todas as respostas da prova passada  
- Mas não entende o conteúdo  

Quando muda a prova:

➡ Ele erra tudo.

Isso é **overfitting**.

---

### 📊 Exemplo típico

Um modelo treinado apresenta:

| Conjunto | Accuracy |
|----------|----------|
| Treino   | **99%** |
| Teste    | **65%** |

Isso é um forte indício de:

👉 **Overfitting**

---

### 📈 Representação conceitual

Imagine um gráfico onde:

- Uma linha simples representa um **bom modelo**
- Uma linha muito sinuosa representa **overfitting**

Um modelo com overfitting:

- Se adapta demais aos dados  
- Captura ruído  
- Perde capacidade de generalização  

---

### 🌳 Overfitting em modelos de árvore

Em **árvores de decisão**, o overfitting acontece quando:

- A árvore fica **muito profunda**
- Cria **muitas divisões**
- Memoriza os dados

Exemplo conceitual:

Se Petal.Length <= 2.45

Se Sepal.Width <= 3.15

Se Petal.Width <= 0.23

Classe = Setosa


Se muitas regras como essa forem criadas, o modelo pode se tornar **excessivamente específico**.

---

### 📌 Principais causas do overfitting

1. Modelo muito complexo  
2. Poucos dados  
3. Muitos parâmetros  
4. Muito tempo de treinamento  
5. Presença de ruído nos dados  

---

### 🔍 Como identificar overfitting

Compare:

- Desempenho no treino  
- Desempenho no teste  

Se houver grande diferença:

👉 Provável **overfitting**

## 4. Árvore de Decisão

A **Árvore de Decisão** é um algoritmo de Machine Learning supervisionado muito utilizado em tarefas de **classificação** e **regressão**. Sua principal vantagem é a **interpretabilidade**, pois o modelo toma decisões por meio de regras simples, organizadas em uma estrutura semelhante a uma árvore.

Em problemas de classificação, a árvore divide os dados em subconjuntos cada vez mais homogêneos em relação à variável alvo. Em outras palavras, ela cria perguntas do tipo:

- A variável `Petal.Length` é menor ou igual a determinado valor?
- Se sim, segue para um ramo.
- Se não, segue para outro.

Ao final, cada folha da árvore representa uma decisão final, como a classe prevista para uma observação.

---

### Quando usar Árvore de Decisão

A Árvore de Decisão é indicada quando se deseja:

- construir um modelo fácil de interpretar;
- entender quais variáveis mais influenciam a decisão;
- trabalhar com relações não lineares entre variáveis;
- criar uma base conceitual antes de estudar algoritmos mais avançados, como Random Forest e XGBoost.

---

### Vantagens

- Fácil interpretação
- Boa visualização das regras de decisão
- Não exige normalização dos dados
- Pode lidar com relações não lineares
- Funciona bem como introdução aos modelos baseados em árvore

---

### Limitações

- Pode sofrer **overfitting**
- Pequenas mudanças nos dados podem alterar bastante a árvore
- Em geral, tende a ter desempenho inferior a métodos ensemble, como Random Forest

---

### Exemplo prático com o dataset `iris`

Nesta seção, utilizaremos o dataset `iris`, que já vem disponível no R.

Esse dataset possui:

- 150 observações;
- 4 variáveis numéricas:
  - `Sepal.Length`
  - `Sepal.Width`
  - `Petal.Length`
  - `Petal.Width`
- 1 variável categórica alvo:
  - `Species`

---

#### 1. Carregando os pacotes necessários

Para trabalhar com Árvore de Decisão em R, utilizaremos os pacotes:

- `rpart` para treinar o modelo;
- `rpart.plot` para visualizar a árvore;
- `caret` para separar treino e teste e avaliar o desempenho.

```r
install.packages("rpart")
install.packages("rpart.plot")
install.packages("caret")
library(rpart)
library(rpart.plot)
library(caret)
```


#### 2. Carregando e explorando o dataset

```r
data(iris)
head(iris)
summary(iris)
str(iris)
```

Comentário
* head(iris) mostra as primeiras linhas;
* summary(iris) apresenta um resumo estatístico;
* str(iris) mostra a estrutura do dataset.

#### 3. Separando treino e teste

Uma etapa importante em Machine Learning é separar os dados em dois subconjuntos:

* treino: usado para treinar o modelo;
* teste: usado para avaliar o desempenho em dados não vistos.

```r
# 1. Definindo a "Semente" (Seed)
# O computador gera números aleatórios de forma pseudo-aleatória.
# Definir uma semente garante que, toda vez que você rodar o código, 
# a separação entre treino e teste seja exatamente a mesma.
set.seed(123)

# 2. Criando o Índice de Partição
# A função createDataPartition do pacote 'caret' realiza uma amostragem estratificada.
# Isso significa que ela mantém a proporção das espécies (50 de cada) nos novos conjuntos.
trainIndex <- createDataPartition(
  y = iris$Species,   # Variável alvo que queremos balancear
  p = 0.7,            # Porcentagem dos dados para o treino (70%)
  list = FALSE        # Retorna o resultado como uma matriz/vetor, não como lista
)

# 3. Criando o Conjunto de Treino
# Selecionamos todas as linhas cujos números estão no 'trainIndex'
train <- iris[trainIndex, ]

# 4. Criando o Conjunto de Teste
# O sinal de menos (-) indica "exclusão". 
# Selecionamos todas as linhas que NÃO estão no 'trainIndex'
test  <- iris[-trainIndex, ]

# Dica de Professor: 
# Verifique sempre se a separação deu certo conferindo o número de linhas:
# nrow(train) deve ser 105 e nrow(test) deve ser 45.
dim(train)
dim(test)
```
#### 4. Treinando uma Árvore de Decisão

Agora vamos ajustar o modelo.

```r
modelo_tree <- rpart(
  Species ~ .,
  data = train,
  method = "class"
)

```
Explicação do código
* Species ~ . indica que queremos prever Species usando todas as demais variáveis;
* data = train define o conjunto de treino;
* method = "class" informa que o problema é de classificação.

#### 5. Visualizando a árvore

```r
rpart.plot(modelo_tree)

```

A árvore deve ser lida de cima para baixo:

1. O Nó Raiz (O Início de Tudo)

O primeiro quadrado no topo representa todos os dados de treino antes de qualquer divisão.

* setosa: É a classe majoritária (ou a "aposta" inicial).

* .33 .33 .33: Mostra que os dados estão perfeitamente divididos (33% para cada uma das 3 espécies).

* 100%: Indica que 100% das amostras estão neste nó.

2. A Primeira Pergunta (Regra de Ouro)

A árvore pergunta: "O comprimento da pétala (Petal.Length) é menor que 2.6?"

* Se SIM (yes): Você vai para a esquerda. Caímos em um nó vermelho onde 100% das flores são Setosa. A árvore "resolveu" esse grupo com apenas uma pergunta.

* Se NÃO (no): Você vai para a direita. Aqui estão as outras 67% das flores, que são uma mistura de Versicolor e Virginica.

3. O Nó de Decisão Intermediário

No lado direito (cinza), a árvore ainda está em dúvida (.00 .50 .50). Ela precisa de mais uma pergunta para separar Versicolor de Virginica: "A largura da pétala (Petal.Width) é menor que 1.7?"

4. As Folhas Finais (Os Vereditos)

As "folhas" são os quadrados na base da árvore. É onde a decisão final é tomada:

* Folha Esquerda (Vermelha): Se Petal.Length < 2.6, a planta é Setosa. (Confiança de 100%).

* Folha Central (Cinza): Se Petal.Length >= 2.6 E Petal.Width < 1.7, a planta é Versicolor.

Note que a confiança aqui é de 94% (há 6% de chance de ser Virginica, indicado pelo .06).

* Folha Direita (Verde): Se Petal.Length >= 2.6 E Petal.Width >= 1.7, a planta é Virginica.

Confiança de 97%.

#### 6. Realizando previsões

Depois de treinar o modelo, podemos usá-lo para prever as espécies no conjunto de teste.

```r
pred_tree <- predict(
  modelo_tree,
  test,
  type = "class"
)

```

Explicação
* predict() aplica o modelo aos dados de teste;
* type = "class" faz com que a saída seja a classe prevista, e não probabilidades.

#### 7. Avaliando o modelo com matriz de confusão

```r
confusionMatrix(
  pred_tree,
  test$Species
)

```

O que a matriz de confusão mostra? Ela permite observar:

* quantas observações foram classificadas corretamente;
* em quais classes ocorreram erros;
* a acurácia geral do modelo.
