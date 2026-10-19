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

## 9. Exercícios de fixação

1. Um pesquisador registra **aprovado/reprovado** em dois métodos de ensino, com frequências esperadas menores que 5. Qual teste é particularmente indicado?
2. Uma tabela 2×2 tem totais de linha 10 e 10 e totais de coluna 9 e 11. Quais são as quatro frequências esperadas?
3. Um teste t resulta em `p = 0.0705`. Podemos afirmar que as médias populacionais são iguais?
4. Por que o Mann-Whitney com três notas por grupo pode ter `p = 0.10` mesmo sem sobreposição?
5. A ANOVA fornece `p = 0.027`. Isso demonstra que todos os pares de grupos são diferentes?
6. Os **mesmos alunos** são avaliados antes e depois. É correto utilizar t para grupos independentes?
7. Explique por que `p = 0.03` não significa “97% de probabilidade de H₁ ser verdadeira”.
8. Controle, Tratamento A e Tratamento B representam quantos fatores e níveis se apenas a metodologia varia?

### Gabarito comentado

1. **Fisher exato**, especialmente para tabela 2×2 com esperadas pequenas.
2. **4,5; 5,5; 4,5; 5,5**.
3. **Não.** Não rejeitar H₀ não prova igualdade.
4. Com três observações por grupo e sem empates, o menor valor-p bilateral exato é **0,10**.
5. **Não.** A ANOVA aponta pelo menos uma diferença; usar pós-hoc apropriado, como Tukey.
6. **Não.** São dados pareados; considerar t pareado.
7. O valor-p é calculado **supondo H₀**, não estima diretamente a probabilidade de H₁.
8. **Um fator, três níveis.**

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

