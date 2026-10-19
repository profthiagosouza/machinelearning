# Guia de testes estatísticos no R

**Testes:** Qui-quadrado de independência, Fisher exato, t de Student, Mann-Whitney e ANOVA de um fator.

**Objetivo:** compreender os conceitos, escolher o teste adequado, executar exemplos simples no R e interpretar os resultados. Dados fictícios, com nível de significância de 5% (`alpha = 0.05`).

## 1. Conceitos fundamentais

Um **fator** é uma variável que define condições de comparação, como a metodologia de ensino. Seus **níveis** podem ser “tradicional”, “jogos” e “inteligência artificial”. O **grupo Controle** recebe a condição de referência; o **Tratamento** recebe a intervenção. A **variável resposta** é o resultado medido, como a nota de uma avaliação.

As variáveis podem ser **categóricas** (aprovado/reprovado), **ordinais** (ruim/regular/bom) ou **numéricas** (nota, tempo, altura). Também é essencial distinguir **grupos independentes** (alunos diferentes) de **dados pareados** (os mesmos alunos antes e depois).

### 1.1 Hipóteses e valor-p

- **H₀:** hipótese nula, por exemplo ausência de associação ou igualdade de médias.
- **H₁:** hipótese alternativa, por exemplo existência de associação ou diferença.
- **α = 0,05:** nível de significância estabelecido antes da análise.
- **Valor-p:** probabilidade, sob H₀ e os pressupostos do teste, de obter resultados tão ou mais extremos que os observados.

Se `p < 0.05`, rejeitamos H₀. Caso contrário, **não rejeitamos** H₀. Isso **não prova** que H₀ seja verdadeira. Significância estatística não equivale a importância prática, e associação não implica causalidade.

---

## 2. Qui-quadrado de independência

### 2.1 Quando utilizar?

Quando desejamos investigar a **associação entre duas variáveis categóricas**, a partir de frequências em uma tabela de contingência. O teste não compara médias.

### 2.2 Exemplo: sexo registrado e preferência por disciplina

| Grupo | Matemática | História | Total |
|---|---:|---:|---:|
| Meninos | 8 | 2 | 10 |
| Meninas | 2 | 8 | 10 |
| **Total** | **10** | **10** | **20** |

**H₀:** sexo registrado e preferência são independentes.  
**H₁:** existe associação.

### 2.3 Frequências esperadas e cálculo

Se não houver associação, a frequência esperada de cada célula é:

Eᵢⱼ = (total da linha i × total da coluna j) / total geral

No exemplo, todas as frequências esperadas são `(10 × 10)/20 = 5`.

A estatística é:

χ² = soma de [(O − E)² / E]

χ² = (8 − 5)²/5 + (2 − 5)²/5 + (2 − 5)²/5 + (8 − 5)²/5 = 7,2

Graus de liberdade: `(2−1) × (2−1) = 1`.

### 2.4 Executando no R

```r
tab <- matrix(c(8, 2, 2, 8), nrow = 2, byrow = TRUE)
rownames(tab) <- c("Meninos", "Meninas")
colnames(tab) <- c("Matematica", "Historia")

tab
prop.table(tab, 1)             # proporções por linha
resultado <- chisq.test(tab, correct = FALSE)
resultado
resultado$expected             # frequências esperadas
resultado$residuals            # resíduos de Pearson
```

**Resultado:** `X-squared = 7.2`, `df = 1`, `p-value ≈ 0.00729`. Rejeitamos H₀: há evidência de associação.

**Atenção:** `chisq.test(tab)` aplica, por padrão, correção de continuidade de Yates em tabelas 2×2; por isso o resultado difere de `correct = FALSE`.

### 2.5 Pressupostos e limitações

- Observações independentes; cada pessoa entra em uma única célula.
- Valores são **contagens**, não médias ou porcentagens.
- A distribuição Qui-quadrado é uma aproximação que pode falhar com **frequências esperadas pequenas**.
- Regra conservadora de ensino: esperadas ≥ 5. Há critérios alternativos, mas nenhuma regra substitui avaliação do desenho e dos dados.

### 2.6 O Qui-quadrado precisa ser 2×2?

Não. Exemplo 3×2:

| Turno | Aprovou | Reprovou |
|---|---:|---:|
| Manhã | 18 | 7 |
| Tarde | 14 | 11 |
| Noite | 10 | 15 |

```r
turno <- matrix(c(18,7, 14,11, 10,15), nrow = 3, byrow = TRUE)
chisq.test(turno)
```

### 2.7 Tamanho de efeito

Em uma tabela 2×2, o coeficiente phi é:

φ = √(χ² / n)

Neste exemplo: `sqrt(7.2 / 20) ≈ 0.60`. Em tabelas maiores, uma opção é o **V de Cramér**. O valor-p sozinho não mede a força da associação.

---

## 3. Teste exato de Fisher

### 3.1 Quando utilizar?

O teste exato de Fisher também avalia **associação entre variáveis categóricas**. É particularmente útil em **tabelas 2×2 com amostras pequenas ou frequências esperadas baixas**, quando a aproximação do Qui-quadrado pode ser inadequada.

Ao contrário da aproximação assintótica do Qui-quadrado, Fisher calcula probabilidades exatas condicionadas às margens da tabela. Também há extensões para tabelas maiores.

### 3.2 Exemplo: treinamento e aprovação

| Grupo | Aprovou | Reprovou | Total |
|---|---:|---:|---:|
| Controle | 1 | 5 | 6 |
| Tratamento | 5 | 1 | 6 |
| **Total** | **6** | **6** | **12** |

**H₀:** grupo e aprovação são independentes.  
**H₁:** há associação.

Cada frequência esperada é `(6 × 6)/12 = 3`. Portanto, **todas** são menores que 5.

### 3.3 Como o Fisher funciona?

Com os totais de linhas e colunas fixos, o teste considera as tabelas possíveis e suas probabilidades. Para a célula superior esquerda de uma tabela 2×2:

P(a) = [C(r₁, a) × C(r₂, c₁ − a)] / C(n, c₁)

Neste exemplo, a probabilidade da tabela observada é:

P(a = 1) = [C(6, 1) × C(6, 5)] / C(12, 6) = 36/924 ≈ 0,039

**Atenção:** isso é a probabilidade de **uma tabela**, não o valor-p bilateral. O teste bilateral soma probabilidades de tabelas pelo menos tão extremas, conforme sua definição.

### 3.4 Executando no R

```r
tab_fisher <- matrix(c(1,5, 5,1), nrow = 2, byrow = TRUE)
rownames(tab_fisher) <- c("Controle", "Tratamento")
colnames(tab_fisher) <- c("Aprovou", "Reprovou")

chisq.test(tab_fisher, correct = FALSE)$expected
fisher.test(tab_fisher)
chisq.test(tab_fisher, correct = FALSE)
chisq.test(tab_fisher)  # com correção de Yates
```

**Fisher bilateral:** `p-value ≈ 0.0801`. Como `p > 0.05`, **não rejeitamos H₀**. Isso não prova ausência de associação: há apenas 12 participantes.

O Qui-quadrado **sem correção** fornece `χ² ≈ 5.333` e `p ≈ 0.0209`. A diferença entre os resultados mostra por que **não devemos confiar automaticamente na aproximação Qui-quadrado quando as frequências esperadas são baixas**.

### 3.5 Fisher versus Qui-quadrado

| Aspecto | Qui-quadrado | Fisher |
|---|---|---|
| Pergunta | Associação categórica | Associação categórica |
| Cálculo | Aproximação pela distribuição χ² | Probabilidades exatas condicionais |
| Tabela 2×2 | Sim | Sim, caso clássico |
| Tabelas maiores | Sim | Possível, com custo computacional |
| Esperadas muito pequenas | Pode ser pouco confiável | Frequentemente preferível |
| R | `chisq.test()` | `fisher.test()` |

**Pergunta importante: e se três células tiverem frequência maior que 5 e apenas uma menor que 5?**

Primeiro verifique se os valores são **observados ou esperados**. Para julgar a aproximação do Qui-quadrado, interessam principalmente os **esperados**. Em uma tabela 2×2, **uma única frequência esperada pequena já pode justificar optar por Fisher**. Não é uma obrigação matemática universal baseada apenas no número 5.

### 3.6 Razão de chances (*odds ratio*)

O Fisher no R também fornece uma estimativa de **odds ratio** e intervalo de confiança:

```r
res <- fisher.test(tab_fisher)
res$estimate
res$conf.int
```

Odds ratio mede associação e não é automaticamente igual ao risco relativo.

---

## 4. Teste t de Student para amostras independentes

### 4.1 Quando utilizar?

Para investigar se **duas médias populacionais** de uma variável numérica diferem, comparando **dois grupos independentes**.

- **Fator:** metodologia de ensino.
- **Níveis:** tradicional e nova.
- **Controle:** método tradicional.
- **Tratamento:** nova metodologia.
- **Variável resposta:** nota da avaliação.

| Aluno | Controle | Tratamento |
|---|---:|---:|
| 1 | 6 | 8 |
| 2 | 7 | 9 |
| 3 | 8 | 10 |
| **Média** | **7** | **9** |

**H₀:** médias populacionais iguais.  
**H₁:** médias populacionais diferentes.

### 4.2 Cálculo

No teste t clássico para grupos independentes, assumindo variâncias populacionais iguais:

t = (média Controle − média Tratamento) / [desvio-padrão combinado × √(1/nC + 1/nT)]

Os dois grupos têm desvio-padrão amostral igual a 1; portanto `s_p = 1`.

t = (7 − 9) / √(1/3 + 1/3) ≈ −2,45

Graus de liberdade: `3 + 3 − 2 = 4`. O sinal negativo indica que a média do Controle é menor.

### 4.3 Código R

```r
controle <- c(6, 7, 8)
tratamento <- c(8, 9, 10)

mean(controle); mean(tratamento)
sd(controle); sd(tratamento)

# Student clássico: pressupõe variâncias iguais
t.test(controle, tratamento, var.equal = TRUE)

# Welch: não pressupõe variâncias iguais; padrão do R
t.test(controle, tratamento)
```

**Resultado clássico:** `t ≈ -2.4495`, `df = 4`, `p-value ≈ 0.0705`. IC95% para a diferença Controle − Tratamento: aproximadamente `[-4.267, 0.267]`.

**Interpretação:** não rejeitamos H₀ a 5%. Embora a diferença observada seja de 2 pontos, ela não foi estatisticamente significativa nesta pequena amostra.

### 4.4 Pressupostos

- Grupos e observações independentes.
- Resposta numérica.
- Populações aproximadamente normais, especialmente com amostras muito pequenas.
- Variâncias iguais na versão clássica; **Welch** é uma alternativa usual quando essa igualdade não é assumida.
- Outliers podem influenciar fortemente as médias.

```r
boxplot(controle, tratamento,
        names = c("Controle", "Tratamento"))
qqnorm(controle); qqline(controle)
```

Com apenas três observações, verificações formais de normalidade são pouco informativas.

### 4.5 E se forem os mesmos alunos antes e depois?

Nesse caso, as medidas são **pareadas**, não independentes:

```r
antes <- c(6, 7, 8)
depois <- c(8, 9, 10)
t.test(antes, depois, paired = TRUE)
```

### 4.6 Tamanho de efeito

Além do valor-p, relate a diferença de médias e seu intervalo de confiança. Um tamanho de efeito possível é o **d de Cohen**:

d = (média Tratamento − média Controle) / desvio-padrão combinado = (9 − 7)/1 = 2

Com amostras minúsculas, estimativas de efeito são muito incertas.

---

## 5. Teste de Mann-Whitney

### 5.1 Quando utilizar?

Para comparar **dois grupos independentes** por meio da **ordenação dos valores** (*postos*). É útil com respostas ordinais e em determinadas situações com respostas numéricas.

O teste **não compara diretamente médias**. Em geral, a hipótese nula é de igualdade das distribuições; com distribuições de formatos semelhantes, o resultado pode ser interpretado como diferença de localização ou medianas.

| Aluno | Controle | Tratamento |
|---|---:|---:|
| 1 | 4 | 7 |
| 2 | 5 | 8 |
| 3 | 6 | 9 |
| **Mediana** | **5** | **8** |

**H₀:** distribuições iguais.  
**H₁:** distribuições diferentes.

### 5.2 Ordenação e postos

| Nota | Grupo | Posto |
|---:|---|---:|
| 4 | Controle | 1 |
| 5 | Controle | 2 |
| 6 | Controle | 3 |
| 7 | Tratamento | 4 |
| 8 | Tratamento | 5 |
| 9 | Tratamento | 6 |

Soma dos postos do Controle: `R_C = 1 + 2 + 3 = 6`.  
Soma dos postos do Tratamento: `R_T = 4 + 5 + 6 = 15`.

A estatística U para o Controle:

U Controle = soma dos postos do Controle − [nC × (nC + 1)]/2 = 6 − 3×4/2 = 0

### 5.3 Código R

```r
controle <- c(4, 5, 6)
tratamento <- c(7, 8, 9)

wilcox.test(controle, tratamento, exact = TRUE)
```

**Resultado:** `W = 0`, `p-value = 0.1` (bilateral exato, sem empates). Aqui, o `W` reportado pelo R corresponde ao `U_C`.

### 5.4 Por que não foi significativo?

Mesmo com separação completa, há apenas `choose(6,3) = 20` alocações possíveis de postos entre dois grupos de três. As duas separações extremas dão:

p bilateral = 2/20 = 0,10

Assim, **não rejeitamos H₀ a 5%**. A pequena amostra limita a evidência estatística.

### 5.5 Empates e cuidados

- Observações devem ser independentes.
- A variável deve ser ao menos ordinal.
- Valores empatados recebem **postos médios**. Por exemplo, duas notas 5 nas posições 2 e 3 recebem posto 2,5.
- Empates podem exigir uma aproximação em vez do cálculo exato disponível no R básico.
- Não escolha Mann-Whitney **automaticamente** apenas porque um teste de normalidade apresentou `p < 0.05`.

### 5.6 Student versus Mann-Whitney

| Aspecto | t de Student | Mann-Whitney |
|---|---|---|
| Principal alvo | Médias | Distribuições / localização |
| Trabalha com | Valores numéricos | Postos |
| Dados ordinais | Em geral, não | Sim |
| Normalidade | Importante para inferência exata em pequenas amostras | Não exigida |
| Independência | Sim | Sim |

---

## 6. ANOVA de um fator (*One-Way ANOVA*)

### 6.1 Quando utilizar?

Para comparar **três ou mais médias populacionais** em grupos independentes, definidos por **um único fator**.

- **Fator:** metodologia de ensino.
- **Níveis:** tradicional, jogos e IA.
- **Controle:** metodologia tradicional.
- **Tratamento A:** aprendizagem baseada em jogos.
- **Tratamento B:** aprendizagem com IA.
- **Resposta:** nota.

| Aluno | Controle | Tratamento A | Tratamento B |
|---|---:|---:|---:|
| 1 | 6 | 7 | 9 |
| 2 | 7 | 8 | 10 |
| 3 | 8 | 9 | 11 |
| **Média** | **7** | **8** | **10** |

**H₀:** todas as médias populacionais são iguais.  
**H₁:** pelo menos uma média populacional difere.

### 6.2 Lógica da ANOVA

Comparamos a variação **entre grupos** com a variação **dentro dos grupos**:

F = quadrado médio entre grupos / quadrado médio dentro dos grupos

| Fonte de variação | Soma dos quadrados | Graus de liberdade | Quadrado médio |
|---|---:|---:|---:|
| Entre grupos | 14 | 2 | 7 |
| Dentro dos grupos | 6 | 6 | 1 |
| Total | 20 | 8 | — |

Logo, `F = 7 / 1 = 7`.

### 6.3 Código R

```r
nota <- c(6,7,8, 7,8,9, 9,10,11)
grupo <- factor(rep(c("Controle", "Tratamento_A", "Tratamento_B"),
                    each = 3))
dados <- data.frame(nota, grupo)

aggregate(nota ~ grupo, dados, mean)

modelo <- aov(nota ~ grupo, data = dados)
summary(modelo)
```

**Resultado:** `F(2, 6) = 7`, `p-value ≈ 0.027`.

Como `p < 0.05`, rejeitamos H₀: **pelo menos uma média difere**. Isso **não significa** que todos os pares diferem.

### 6.4 Comparações posteriores: Tukey

```r
TukeyHSD(modelo)
```

O procedimento compara Controle × A, Controle × B e A × B, ajustando para múltiplas comparações. Neste exemplo, o contraste Controle × Tratamento B é o único significativo pelo procedimento de Tukey a 5%.

Se a pergunta for **apenas comparar cada tratamento com o Controle**, um procedimento de **Dunnett** pode ser mais adequado.

### 6.5 Pressupostos e alternativas

- Observações independentes.
- Resíduos aproximadamente normais.
- Homogeneidade de variâncias entre os grupos.

```r
plot(modelo, which = 1)  # resíduos versus ajustados
plot(modelo, which = 2)  # QQ-plot

# ANOVA de Welch, quando variâncias iguais não são assumidas
oneway.test(nota ~ grupo, data = dados)
```

Para dados ordinais em três ou mais grupos independentes, **Kruskal-Wallis** pode ser uma alternativa, embora teste uma hipótese diferente da ANOVA clássica.

### 6.6 Tamanho de efeito: eta quadrado

η² = soma dos quadrados entre grupos / soma dos quadrados total = 14/20 = 0,70

Neste conjunto fictício, 70% da variação total observada é explicada pelas diferenças entre os grupos **no modelo**. Isso não demonstra causalidade por si só.

### 6.7 Dois tratamentos significam dois fatores?

Não. **Controle + Tratamento A + Tratamento B** podem ser três **níveis de um único fator**. Dois fatores seriam, por exemplo, **metodologia** (tradicional/jogos) e **turno** (manhã/noite), permitindo uma ANOVA de dois fatores.

---

## 7. Comparação geral

| Teste | Pergunta principal | Resposta / estrutura |
|---|---|---|
| Qui-quadrado | Duas variáveis categóricas estão associadas? | Tabela de contagens |
| Fisher | Há associação com amostra pequena e esperadas baixas? | Tabela 2×2, especialmente |
| t de Student | Duas médias populacionais diferem? | Numérica, dois grupos independentes |
| Mann-Whitney | Duas distribuições diferem? | Ordinal/numérica, dois grupos independentes |
| ANOVA de um fator | Alguma média difere entre 3+ grupos? | Numérica, um fator, grupos independentes |

---

## 8. Script integrado para a aula

```r
# Qui-quadrado
qui <- matrix(c(8,2,2,8), nrow=2, byrow=TRUE)
chisq.test(qui, correct=FALSE)

# Fisher
fisher <- matrix(c(1,5,5,1), nrow=2, byrow=TRUE)
fisher.test(fisher)
chisq.test(fisher, correct=FALSE)$expected

# t de Student
t.test(c(6,7,8), c(8,9,10), var.equal=TRUE)

# Mann-Whitney
wilcox.test(c(4,5,6), c(7,8,9), exact=TRUE)

# ANOVA
dados <- data.frame(
  nota=c(6,7,8,7,8,9,9,10,11),
  grupo=factor(rep(c("Controle","A","B"), each=3))
)
modelo <- aov(nota ~ grupo, data=dados)
summary(modelo)
TukeyHSD(modelo)
```

---

## 9. Exercícios de fixação — aplicação ao dataset `mtcars`

Nesta atividade, você utilizará o conjunto de dados **`mtcars`**, disponível no R, que reúne informações de **32 automóveis**. O objetivo é formular hipóteses, escolher e executar os cinco testes estudados e interpretar os resultados. Adote **α = 0,05** e hipóteses **bilaterais** quando aplicável.

### 9.1 Preparação dos dados

Execute os comandos e examine a estrutura do conjunto:

```r
data(mtcars)
str(mtcars)
head(mtcars)
summary(mtcars)
```

Variáveis utilizadas nos exercícios:

| Variável | Significado | Tipo para esta atividade |
|---|---|---|
| `mpg` | Consumo em milhas por galão (maior = mais econômico) | Numérica |
| `am` | Transmissão: 0 = automática; 1 = manual | Categórica (2 níveis) |
| `vs` | Configuração do motor: 0 = em V; 1 = em linha | Categórica (2 níveis) |
| `cyl` | Número de cilindros: 4, 6 ou 8 | Categórica (3 níveis) |
| `gear` | Número de marchas: 3, 4 ou 5 | Categórica (3 níveis) |

**Importante:** embora `am`, `vs`, `cyl` e `gear` sejam armazenadas como números, nos testes de associação e na ANOVA elas devem ser tratadas como **categorias**. O conjunto é observacional; os resultados não permitem concluir causalidade.

### Exercício 1 — Qui-quadrado de independência

**Pergunta:** existe associação entre o **número de cilindros (`cyl`)** e o **tipo de transmissão (`am`)**?

1. Construa uma tabela de contingência entre `cyl` e `am`.
2. Calcule as proporções por linha e interprete o padrão observado.
3. Formule H₀ e H₁.
4. Execute o Qui-quadrado de Pearson **sem correção de continuidade** e examine as frequências esperadas.
5. Interprete o valor-p. A aproximação do Qui-quadrado é adequada? Justifique com base nas frequências **esperadas**.
6. Calcule o **V de Cramér** e interprete a magnitude com cautela.

### Exercício 2 — Fisher exato versus Qui-quadrado

**Pergunta:** existe associação entre a **configuração do motor (`vs`)** e o **tipo de transmissão (`am`)**?

1. Construa a tabela 2×2 e identifique os totais marginais.
2. Calcule as frequências esperadas do Qui-quadrado.
3. Execute o **teste exato de Fisher**, o Qui-quadrado sem correção e o Qui-quadrado com correção de Yates.
4. Compare os valores-p e verifique se a decisão a 5% muda.
5. Com base nas frequências esperadas, explique se Fisher é necessário ou apenas uma alternativa válida neste caso.
6. Identifique a **odds ratio** e seu intervalo de confiança retornados por `fisher.test()`; explique por que associação não implica causalidade.

### Exercício 3 — Teste t para dois grupos independentes

**Pergunta:** a **média de `mpg`** difere entre automóveis de transmissão **automática** e **manual**?

1. Identifique o fator, seus níveis e a variável resposta.
2. Calcule o tamanho amostral, a média e o desvio-padrão de `mpg` em cada grupo.
3. Construa um boxplot comparativo.
4. Formule H₀ e H₁.
5. Execute o **teste t de Welch** (padrão do R) e, para comparação didática, o **t clássico** com variâncias iguais.
6. Interprete o valor-p, a diferença entre médias e o intervalo de confiança. Explique por que a escolha entre Welch e Student importa.

### Exercício 4 — Mann-Whitney

**Pergunta:** a **distribuição de `mpg`** difere entre automóveis com transmissão automática e manual?

1. Utilize os **mesmos dois grupos** do Exercício 3.
2. Calcule as medianas e os quartis de `mpg` por grupo.
3. Formule H₀ e H₁ para o teste de Mann-Whitney.
4. Execute `wilcox.test()` com teste bilateral. Observe se o R apresenta aviso sobre **empates**.
5. Interprete o valor-p e compare a conclusão com a do teste t.
6. Explique por que os dois testes não respondem exatamente à mesma pergunta estatística.

### Exercício 5 — ANOVA de um fator

**Pergunta:** a **média de `mpg`** difere entre automóveis com **3, 4 ou 5 marchas (`gear`)**?

1. Identifique o fator, seus três níveis e a variável resposta.
2. Calcule as médias e os desvios-padrão de `mpg` por grupo.
3. Construa um boxplot com os três grupos.
4. Formule H₀ e H₁.
5. Execute a **ANOVA de um fator** e interprete o teste F.
6. Examine os gráficos de resíduos e discuta os pressupostos, considerando os tamanhos dos grupos.
7. Execute `TukeyHSD()` e identifique quais pares apresentam evidência de diferença, se houver.
8. Calcule o tamanho de efeito **eta quadrado (η²)** e interprete-o no contexto da amostra.

### 9.2 Roteiro de comandos — apoio à resolução

Os comandos abaixo servem como **ponto de partida**. Os alunos devem interpretar as saídas, verificar pressupostos e responder às questões, não apenas executar as funções.

```r
# Exercício 1: Qui-quadrado e V de Cramér
tab1 <- table(mtcars$cyl, mtcars$am)
prop.table(tab1, 1)
q1 <- chisq.test(tab1, correct = FALSE)
q1
q1$expected
v_cramer <- sqrt(as.numeric(q1$statistic) /
                 (sum(tab1) * min(nrow(tab1)-1, ncol(tab1)-1)))
v_cramer

# Exercício 2: Fisher e comparação
tab2 <- table(mtcars$vs, mtcars$am)
q2 <- chisq.test(tab2, correct = FALSE)
q2$expected
fisher.test(tab2)
q2
chisq.test(tab2)  # correção de Yates em tabela 2x2

# Exercício 3: teste t
aggregate(mpg ~ am, data = mtcars,
          FUN = function(x) c(n=length(x), media=mean(x), dp=sd(x)))
boxplot(mpg ~ factor(am), data = mtcars,
        names = c("Automatica", "Manual"), ylab = "mpg")
t.test(mpg ~ am, data = mtcars)                   # Welch
t.test(mpg ~ am, data = mtcars, var.equal = TRUE) # Student

# Exercício 4: Mann-Whitney
aggregate(mpg ~ am, data = mtcars, median)
aggregate(mpg ~ am, data = mtcars, quantile)
wilcox.test(mpg ~ am, data = mtcars, exact = FALSE)

# Exercício 5: ANOVA de um fator
dados <- transform(mtcars, gear = factor(gear))
aggregate(mpg ~ gear, data = dados,
          FUN = function(x) c(n=length(x), media=mean(x), dp=sd(x)))
boxplot(mpg ~ gear, data = dados, xlab = "Marchas", ylab = "mpg")
modelo <- aov(mpg ~ gear, data = dados)
summary(modelo)
plot(modelo, which = 1)
plot(modelo, which = 2)
TukeyHSD(modelo)
tabela_anova <- summary(modelo)[[1]]
eta2 <- tabela_anova[1, "Sum Sq"] / sum(tabela_anova[, "Sum Sq"])
eta2
```

### 9.3 Orientações para correção

- **Exercício 1:** H₀ afirma independência entre cilindros e transmissão. Verificar se há frequências esperadas baixas antes de confiar no valor-p aproximado. O V de Cramér mede a intensidade da associação na tabela 3×2.
- **Exercício 2:** H₀ afirma independência entre configuração do motor e transmissão. Fisher e Qui-quadrado podem produzir valores-p distintos; a comparação deve considerar as frequências esperadas e a correção de Yates, sem assumir que Fisher é obrigatório para toda tabela 2×2.
- **Exercício 3:** H₀ afirma igualdade das médias populacionais de `mpg`. A interpretação deve apresentar médias por grupo, direção da diferença, valor-p e intervalo de confiança; Welch não pressupõe variâncias iguais.
- **Exercício 4:** H₀, na formulação usual, afirma igualdade das distribuições de `mpg`. A análise usa postos e não testa diretamente a igualdade de médias. `exact = FALSE` permite uma aproximação na presença de empates.
- **Exercício 5:** H₀ afirma igualdade das três médias populacionais de `mpg`. A ANOVA global testa se **pelo menos uma** difere; Tukey investiga pares. O eta quadrado descreve a proporção de variação explicada pelo fator no modelo.

**Questão integradora:** os resultados desses testes permitem afirmar que mudar a transmissão ou o número de marchas **causa** mudança no consumo? Justifique considerando o caráter observacional de `mtcars` e possíveis fatores de confusão, como peso (`wt`) e potência (`hp`).

---

## 10. Checklist para análises reais

- [ ] Identifiquei o tipo de variável resposta?
- [ ] Verifiquei se as observações são independentes ou pareadas?
- [ ] Defini H₀, H₁ e α antes de observar o valor-p?
- [ ] No Qui-quadrado, inspecionei as **frequências esperadas**?
- [ ] Considerei Fisher em tabelas pequenas?
- [ ] Avaliei normalidade, outliers e variâncias quando relevantes?
- [ ] Relatei estatística, valor-p, tamanho amostral e tamanho de efeito?
- [ ] Evitei interpretar ausência de significância como prova de igualdade?
- [ ] Evitei interpretar associação como causalidade sem desenho adequado?

## Referências para aprofundamento

- R Core Team. *R Documentation*: `chisq.test`, `fisher.test`, `t.test`, `wilcox.test`, `aov`, `TukeyHSD`, `oneway.test`.
- Agresti, A. *An Introduction to Categorical Data Analysis*. Wiley.
- Field, A.; Miles, J.; Field, Z. *Discovering Statistics Using R*. Sage.
- Hollander, M.; Wolfe, D. A.; Chicken, E. *Nonparametric Statistical Methods*. Wiley.

