# Trabalho acadêmico — Artigo científico de estatística aplicada ao futsal com R

**Modalidade:** trabalho em duplas  
**Organização:** 8 grupos temáticos (Grupo 1 ao Grupo 8); cada grupo tem uma questão de investigação distinta.  
**Linguagem obrigatória:** R (todas as etapas de tratamento, visualização e inferência).  
**Produto final:** artigo científico original, elaborado segundo as diretrizes da Revista Brasileira de Futsal e Futebol (RBFF), para fins de avaliação acadêmica. A submissão à revista não é obrigatória.  
**Base de dados:** [Futsal Performance Analytics — Om Narawade](https://github.com/narawadeos13/Futsal_analysis)  
**Arquivo original:** `data/raw/futsal_data.xlsx` (no repositório).

## 1. Contextualização

O projeto *Futsal Performance Analytics* reúne registros manuais de uma partida profissional de futsal: segundo a documentação, são **599 eventos**, **9 jogadores**, **2 tempos** e **13 campos** por evento. Entre os eventos estão passes certos (`PS`), passes errados (`PF`), finalizações no alvo (`SS`), finalizações fora (`SF`), interceptações (`INT`), perdas de bola (`LO`), defesas (`SV`), gols marcados (`G`) e gols sofridos (`CON`). Há informações temporais, espaciais e um indicador de pressão defensiva.

O objetivo do trabalho é desenvolver uma análise estatística reprodutível de dados reais, articulando **estatística descritiva** e **testes de hipóteses**, e comunicar os resultados **na forma de um artigo científico original**, conforme a estrutura e as orientações editoriais da **Revista Brasileira de Futsal e Futebol (RBFF)**. Não será utilizado aprendizado de máquina. O artigo é uma atividade acadêmica: seguir o modelo da revista não implica aceite, elegibilidade ou obrigação de submissão.

> **Atenção:** os registros são de uma única partida e não constituem uma amostra aleatória de partidas ou jogadores. Eventos de um mesmo atleta e momentos próximos podem ser dependentes. Portanto, os resultados inferenciais devem ser interpretados como análises exploratórias do jogo observado, sem generalização automática para outras partidas.

## 2. Objetivos de aprendizagem

- Importar, documentar, limpar e transformar dados no R.
- Identificar tipos de variáveis e unidades de observação.
- Calcular e interpretar medidas de tendência central, dispersão e frequências.
- Construir visualizações adequadas à questão investigada.
- Formular hipóteses nula e alternativa e selecionar, justificar e aplicar autonomamente os procedimentos estatísticos adequados.
- Verificar pressupostos, reportar tamanhos de efeito e discutir limitações.
- Comunicar conclusões em linguagem clara, fundamentadas nos resultados.

## 3. Etapa comum obrigatória — estatística descritiva (todos os grupos)

**Todas as duplas**, independentemente da questão designada, deverão:

1. Importar o arquivo original em R e apresentar um dicionário sucinto das variáveis utilizadas.
2. Verificar quantidade de linhas, variáveis, valores ausentes, duplicatas, códigos de eventos e consistência dos registros.
3. Apresentar tabelas de frequência absoluta e relativa dos tipos de eventos, globalmente e por tempo de jogo.
4. Resumir a participação dos jogadores (número de eventos por jogador, passes certos/errados, finalizações, interceptações e perdas de bola, quando aplicáveis).
5. Calcular estatísticas descritivas para pelo menos **duas variáveis quantitativas originais ou derivadas** (por exemplo, tempo do evento, distância do passe, distância da finalização ao gol, eventos por minuto): `n`, média, mediana, desvio-padrão, mínimo, máximo e quartis. Explicar quando uma medida não fizer sentido.
6. Construir **pelo menos quatro visualizações** em R, incluindo gráfico de barras, distribuição de variável quantitativa, comparação entre categorias e uma visualização temporal ou espacial.
7. Interpretar os principais achados, indicando a unidade de análise e possíveis vieses de registro.

A estatística descritiva deve utilizar os dados da partida como um todo; a análise inferencial deve focar a questão atribuída ao grupo.

## 4. Questões de investigação — uma por grupo

Cada grupo deverá formular explicitamente **H₀** e **H₁**, adotar nível de significância de 5%, **escolher por conta própria o teste estatístico mais adequado**, justificar a decisão, verificar pressupostos, calcular e interpretar o valor-p e, quando pertinente, apresentar tamanho de efeito e intervalo de confiança. A escolha do procedimento estatístico é responsabilidade exclusiva de cada dupla, com base na natureza das variáveis, na pergunta de pesquisa e nos pressupostos pertinentes. O enunciado não indica previamente qual teste deve ser utilizado.

### Grupo 1 — Pressão defensiva e acerto de passes

**Pergunta:** Existe associação entre estar sob pressão defensiva e a ocorrência de passe certo ou errado?

- **Variáveis a investigar:** `Pressure` (com/sem pressão) e `Event_id` filtrado em `PS` e `PF`.
- **Discussão:** a pressão está relacionada ao percentual de passes certos? Uma associação observada não implica causalidade.

### Grupo 2 — Tempo de jogo e tipos de evento

**Pergunta:** A composição dos tipos de eventos difere entre o primeiro e o segundo tempo?

- **Variáveis a investigar:** `Half` e `Event_id`, com categorias de evento existentes na base.
- **Discussão:** quais categorias contribuem mais para a associação? Examinar resíduos padronizados com cautela.

### Grupo 3 — Pressão defensiva e distância dos passes

**Pergunta:** A distância dos passes realizados sob pressão difere da distância dos passes sem pressão?

- **Variáveis a investigar:** `Pressure` e distância euclidiana calculada com `Event_X`, `Event_Y`, `Reciever_X` e `Reciever_Y` para passes com coordenadas completas.
- **Discussão:** diferenças de distância estão relacionadas ao contexto defensivo? Coordenadas são estimadas em uma grade; não assumir metros sem verificar a escala.

### Grupo 4 — Distância dos passes certos e errados

**Pergunta:** A distribuição da distância dos passes certos difere daquela dos passes errados?

- **Variáveis a investigar:** `Event_id` (`PS`/`PF`) e distância euclidiana dos passes com coordenadas completas.
- **Discussão:** avaliar o que a comparação estatística escolhida permite concluir, sem confundir diferenças de distribuição com diferenças de mediana.

### Grupo 5 — Distância dos passes entre zonas da quadra

**Pergunta:** A distância média dos passes difere entre as zonas defensiva, intermediária e ofensiva?

- **Variáveis a investigar:** zona de origem do passe derivada de `Event_X` (após confirmar orientação e escala da quadra) e distância euclidiana dos passes.
- **Discussão:** em quais zonas os passes tendem a ser mais longos? Considerar a dependência entre ações de um mesmo jogador.

### Grupo 6 — Distribuição espacial das finalizações

**Pergunta:** A proporção de finalizações no alvo varia conforme a zona de origem da finalização?

- **Variáveis a investigar:** `Event_id` (`SS`/`SF`) e zona espacial derivada de `Event_X`/`Event_Y` (definição geométrica documentada).
- **Discussão:** distinguir precisão (`SS`/`SF`) de conversão em gol; não confundir os dois indicadores. Confirmar o significado dos códigos no arquivo.

### Grupo 7 — Ritmo de jogo entre os dois tempos

**Pergunta:** A quantidade de eventos registrados por minuto difere entre o primeiro e o segundo tempo?

- **Variáveis a investigar:** `Half`, `Full_sec`/`Display_Min` e contagem de eventos por minuto de cada tempo, incluindo minutos com zero eventos dentro do intervalo observado.
- **Discussão:** a contagem por minuto pode apresentar autocorrelação e variação de exposição; avaliar esses problemas antes de interpretar qualquer inferência.

### Grupo 8 — Pressão defensiva e perdas de bola

**Pergunta:** A frequência relativa de perdas de bola é diferente entre ações sob pressão e ações sem pressão?

- **Variáveis a investigar:** `Pressure` e indicador binário `Event_id == "LO"` versus demais eventos elegíveis; definir claramente o denominador e justificar quais ações entram na comparação.
- **Discussão:** o registro de pressão pode variar com o tipo de ação; não interpretar associação como efeito causal.

## 5. Procedimentos para os testes de hipóteses

Em sua questão específica, cada dupla deverá apresentar:

1. Pergunta de pesquisa, população-alvo conceitual e unidade de observação.
2. Variáveis, filtros, recodificações e número de observações válidas.
3. Hipóteses H₀ e H₁ e justificativa da escolha do teste.
4. Diagnóstico dos pressupostos: frequências esperadas, distribuição, dispersão, valores extremos e/ou dependência entre observações, conforme o caso.
5. Resultado do teste: estatística, graus de liberdade (quando aplicáveis), valor-p, nível de significância e tamanho de efeito.
6. Intervalo de confiança, quando aplicável.
7. Conclusão substantiva no contexto do futsal e limitações da inferência.

**Regra metodológica:** não se deve forçar um teste quando seus pressupostos forem claramente violados. Se o procedimento escolhido for inviável (por exemplo, poucas finalizações ou grupos sem observações), a dupla deverá documentar o problema, propor alternativa estatisticamente defensável e manter a questão de investigação como referência.

## 6. Ferramentas e requisitos técnicos

- Toda a análise deverá ser realizada em **R**; RStudio é recomendado.
- Pacotes sugeridos: `readxl`, `dplyr`, `tidyr`, `ggplot2`, `janitor`, `rstatix`, `effectsize`, `knitr` e `rmarkdown` ou `quarto`.
- Não serão aceitas análises estatísticas realizadas exclusivamente em Excel, Python, Power BI ou softwares similares.
- O código deve ser comentado, reprodutível e executar a partir do arquivo original sem depender de caminhos absolutos da máquina dos estudantes.
- É permitido criar variáveis derivadas, desde que as fórmulas e decisões sejam documentadas.

## 7. Redação obrigatória em formato de artigo científico — RBFF

**Cada dupla deverá redigir um artigo científico original**, com sua própria pergunta de pesquisa, resultados, discussão e conclusão. O manuscrito deve observar as [Instruções para Autores da Revista Brasileira de Futsal e Futebol (RBFF)](https://www.rbff.com.br/index.php/rbff/pt/about/submissions), consultadas em 20 de outubro de 2026.

### 7.1. Estrutura do artigo

1. **Página de título:** título informativo em português e em inglês; nomes completos dos dois autores; afiliação institucional, cidade, estado e país; e-mail dos autores e identificação do autor correspondente. Para entrega acadêmica, dados pessoais de endereço podem ser omitidos, salvo orientação do docente.
2. **Resumo e Abstract:** resumo em português de até **250 palavras**, incluindo objetivo, materiais e métodos, resultados, discussão e conclusão; tradução fiel em inglês (*abstract*). Inserir de **três a cinco palavras-chave** em português e inglês, preferencialmente escolhidas entre os Descritores em Ciências da Saúde (DeCS), conforme orientação da revista.
3. **Introdução:** contextualizar a análise de desempenho no futsal, apresentar literatura pertinente, justificar a pergunta específica do grupo e encerrar com o objetivo do estudo.
4. **Materiais e Métodos:** caracterizar o estudo como observacional e exploratório, descrever a origem da base, unidades de observação, variáveis, tratamento dos dados, estatística descritiva comum e procedimento inferencial escolhido e justificado pela dupla. Informar pacotes e versão do R, hipóteses, nível de significância, pressupostos e tamanhos de efeito. Descrever procedimentos que permitam a reprodução dos resultados.
5. **Resultados:** apresentar primeiro os achados descritivos obrigatórios e, em seguida, os resultados da questão de investigação do grupo, com tabelas e figuras produzidas em R, estatísticas dos testes, valores-p, tamanhos de efeito e intervalos de confiança quando aplicáveis.
6. **Discussão:** interpretar os resultados à luz da literatura, diferenciar significância estatística de relevância prática e reconhecer limitações da análise de eventos de uma única partida, incluindo dependência entre registros e tamanho amostral efetivo.
7. **Conclusão:** responder objetivamente à pergunta de investigação sem extrapolar o alcance dos dados.
8. **Referências:** incluir a fonte do dataset e os trabalhos científicos efetivamente citados, em ordem alfabética e no estilo de referências da RBFF. Usar citações no sistema autor-data.
9. **Agradecimentos e declarações**, quando aplicáveis: registrar apoios, conflitos de interesse e outras informações pertinentes, sem inventar declarações éticas ou aprovações.

### 7.2. Apresentação do manuscrito

- Papel A4, **Arial 12**, **espaçamento duplo** e margens de **2,5 cm** nos quatro lados, conforme as instruções para artigos originais da RBFF.
- Tabelas e figuras inseridas nos pontos relevantes do texto, com títulos, legendas, fontes e boa legibilidade. As figuras devem ser produzidas em R.
- Adotar linguagem científica, objetiva e impessoal, sem reproduzir saídas brutas do console como substitutas da apresentação dos resultados.
- A entrega acadêmica poderá ser gerada por **R Markdown ou Quarto**, com exportação para **Word (.docx)**, formato editável compatível com a revista. Também poderá ser entregue uma versão PDF para leitura.
- As diretrizes da revista deverão ser novamente verificadas antes de qualquer eventual submissão real. A publicação não é garantida; a revista estabelece critérios próprios de originalidade e elegibilidade de dados.

### 7.3. Arquivos a entregar

1. **Artigo científico em .docx**, estruturado e formatado segundo as orientações da RBFF; PDF adicional é opcional.
2. **Código-fonte reprodutível em R** (`.R`, `.Rmd` ou `.qmd`), incluindo importação, limpeza, estatística descritiva, inferência e produção de tabelas e figuras.
3. **Arquivo `README.md`**, com instruções de execução, dependências e referência à origem da base.

O artigo deve identificar os dois integrantes, o grupo e sua questão de investigação. Cada dupla deverá produzir interpretação própria, mesmo quando compartilhar a base de dados com outras duplas.

## 8. Critérios de avaliação (100 pontos)

| Critério | Pontuação |
|---|---:|
| Importação, qualidade e documentação dos dados | 15 |
| Estatística descritiva comum a todos os grupos | 25 |
| Formulação das hipóteses e adequação metodológica | 15 |
| Aplicação e interpretação do teste estatístico | 20 |
| Visualizações e redação científica conforme a estrutura da RBFF | 10 |
| Reprodutibilidade, organização e clareza do código R | 10 |
| Discussão crítica das limitações | 5 |
| **Total** | **100** |

## 9. Orientações finais

- Os **oito grupos temáticos** são independentes e cada trabalho será realizado **em dupla**.
- Todos usam a **mesma base original** e realizam a **mesma etapa obrigatória de estatística descritiva**; o diferencial é a questão de investigação e a análise inferencial.
- Não é necessário que os resultados sejam estatisticamente significativos. O foco é a qualidade da metodologia, da interpretação e da comunicação científica no formato de artigo original.
- **Não será aceito relatório convencional como substituto do artigo científico.** A submissão efetiva à RBFF não faz parte das exigências do trabalho.
- Não se deve excluir observações apenas para obter significância estatística.
- A base descreve uma partida; conclusões devem se limitar ao contexto analisado e reconhecer eventuais dependências entre eventos.

## 10. Fonte de dados

NARAWADE, Om. **Futsal Performance Analytics — Real Match Data**. GitHub. Disponível em: <https://github.com/narawadeos13/Futsal_analysis>. Acesso em: 20 out. 2026.

**Nota sobre a fonte:** a documentação do repositório descreve 599 eventos, nove jogadores e 13 campos, além de um arquivo Excel original. Os enunciados foram construídos com base nessa documentação. As distribuições e a integridade das colunas do arquivo bruto deverão ser verificadas na etapa de importação; o repositório informa que a coleta foi manual, durante uma partida real.

## 11. Referência editorial

REVISTA BRASILEIRA DE FUTSAL E FUTEBOL (RBFF). **Instruções para Autores — Submissões**. Disponível em: <https://www.rbff.com.br/index.php/rbff/pt/about/submissions>. Acesso em: 20 out. 2026.
