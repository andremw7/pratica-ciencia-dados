
link google colab:
  Estratificação por região: https://colab.research.google.com/drive/1NinqFDrJEfxpv7NSHcP6VAH2_dYe9pGf


# Relatório dia 13 de agosto

## 🎯 Amostragem Estratificada (10% Representativo)

Para garantir que os dados reflitam com precisão a diversidade do país, não fizemos uma escolha aleatória simples. Utilizou-se a **Amostragem Estratificada por Estado (UF)**:

* **Proporção Exata:** Retiramos exatamente **10% dos participantes de cada estado brasileiro**.
* **Representatividade Real:** Estados com maior número de inscritos mantêm seu peso proporcional exato na amostra final.
* **Agilidade na Análise:** A redução para 10% do volume total mantém total confiabilidade estatística e permite gerar gráficos e modelos com rapidez.

## 💡 Processamento em Blocos (Proteção de Memória RAM)

Como cada arquivo anual possui gigabytes de tamanho e mais de 100 colunas, carregar tudo de uma só vez estoura os limites de RAM do servidor.

A solução aplicada foi o **processamento em blocos (`chunksize`)**:
1. O código lê o arquivo em fatias de 100.000 linhas.
2. Extrai a amostra de 10% proporcional de cada estado para aquele bloco.
3. Descarte o excesso da memória temporária e passa para a próxima fatia.

## 📊 Análise Exploratória Inicial: Distribuição Regional por Área do Conhecimento

A partir da amostra estatística representativa de 10% previamente extraída dos dados brutos, iniciou-se a etapa de Análise Exploratória de Dados (EDA). Para evitar a métrica da média simples — que oculta a assimetria da distribuição e esconde tanto notas nulas quanto desempenhos de ponta —, adotou-se o **Boxplot (Gráfico de Caixa)** como ferramenta central de visualização.

### 🎯 Objetivos da Visualização
* **Mapeamento de Desempenho por Área:** Avaliação da distribuição de notas nas 5 frentes do exame: *Ciências da Natureza*, *Ciências Humanas*, *Linguagens e Códigos*, *Matemática* e *Redação*.
* **Identificação de Alta Performance:** Análise da concentração e alcance dos *outliers* (alunos no topo da cauda de distribuição com notas $\ge 700$) entre as 5 regiões do Brasil (Norte, Nordeste, Centro-Oeste, Sudeste e Sul).
* **Diagnóstico da Variabilidade Regional:** Comparação direta da mediana e do intervalo interquartil (IQR), revelando a dispersão central do desempenho educacional em cada território.

### 🛠️ Estrutura do Painel de Boxplots
* **Eixo Horizontal (X):** Agrupamento pelas 5 Regiões do Brasil, mapeadas a partir das siglas estaduais (`SG_UF_PROVA`) e diferenciadas por paleta de cores exclusiva.
* **Eixo Vertical (Y):** Pontuação no ENEM padronizada no intervalo de $0$ a $1.000$ pontos, com escala ajustada em intervalos de $100$ em $100$ pontos para garantir precisão gráfica.
* **Tratamento do Volume:** A amostragem otimizada permitiu a renderização completa dos microdados regionais de forma fluida, sem saturação visual ou perdas na amostragem de *outliers*.

# 20 de agosto

## 📈 Análise Temporal de Desempenho Regional (2019–2023)

Com a consolidação das amostras estratificadas de 10% para o quinquênio 2020–2024, foi construído um painel longitudinal de boxplots agrupados por ano e por região para investigar a evolução temporal das notas.

### 🔍 Principais Achados Científicos
* **Estabilidade Estrutural da Disparidade:** Ao longo dos 5 anos analisados, a hierarquia regional de desempenho manteve-se constante, com as regiões Sudeste e Sul apresentando medianas e limites superiores superiores às demais regiões em todas as edições.
* **Comportamento por Área:** 
  * **Matemática e Redação:** Apresentam a maior dispersão (caixas mais amplas) e os maiores tetos de nota, concentrando a maior parte dos *outliers* de alta performance.
  * **Linguagens e Códigos:** Apresenta a menor amplitude de notas, com forte concentração na faixa mediana e raros casos de notas extremas acima de 800 pontos.

 ## 🧬 Análise Temporal dos Impactos Socioeconômicos por Região (2019–2023)

Para investigar as causas das disparidades regionais de desempenho, analisou-se a evolução do impacto das variáveis socioeconômicas sobre a nota geral dos estudantes no ENEM no período de 2019 a 2023. 

A análise utilizou $100\%$ da amostra estatística de $10\%$ dos microdados anuais do INEP. A medição foi realizada calculando o Coeficiente de Correlação de Pearson ($r$) isoladamente para cada combinação de **[Ano + Região]**, permitindo acompanhar se a influência de cada fator aumentou, diminuiu ou se manteve estável ao longo do tempo.

### 📖 Guia de Leitura dos Gráficos de Correlação

O coeficiente de correlação ($r$) mede a força da relação entre duas variáveis em uma escala que vai de **$-1{,}00$ a $+1{,}00$**:

* **Valores Positivos ($> 0$):** Indicam relação direta. Quanto maior o nível da variável (ex: maior renda), **maior** tende a ser a nota média do candidato.
* **Valores Próximos de Zero ($\approx 0$):** Indicam pouca ou nenhuma interferência daquela variável na pontuação do estudante.
* **Valores Negativos ($< 0$):** Indicam relação de inversão ou desvantagem estatística em relação ao grupo de referência.

### 📊 Detalhamento Didático das Variáveis Analisadas

| Variável Analisada | Codificação no Modelo | Sinal no Gráfico | O que o resultado indica na prática? |
| :--- | :--- | :---: | :--- |
| **Escola Pública (vs. Privada)** | $1 = \text{Pública}$, $0 = \text{Privada}$ | **Negativo** ($-0{,}40$ a $-0{,}50$) | **Hiato Institucional:** Como a rede privada obtém notas superiores, a pública fica no lado negativo. Quanto mais distante do zero (mais para baixo), maior é a desvantagem do aluno da escola pública. |
| **Renda Familiar** | $0$ (Classe A/Baixa) a $16$ (Classe Q/Alta) | **Positivo** ($+0{,}40$ a $+0{,}55$) | **Alavanca Financeira:** É o fator individual de maior impacto positivo. Famílias com maior renda garantem acesso a melhores materiais, cursos e infraestrutura de estudos. |
| **Escolaridade da Mãe e do Pai** | $0$ (Sem estudo) a $7$ (Pós-Graduação) | **Positivo** ($+0{,}25$ a $+0{,}40$) | **Capital Cultural:** O nível de instrução dos pais impulsiona as notas dos filhos, sendo o impacto da escolaridade materna ligeiramente superior ao paterno em quase todas as regiões. |
| **Acesso à Internet** | $0 = \text{Não}$, $1 = \text{Sim}$ | **Positivo** ($+0{,}15$ a $+0{,}25$) | **Inclusão Digital:** A presença de internet em casa atua como um facilitador pedagógico constante, mantendo correlação positiva com a nota em todo o país. |
| **Idade / Faixa Etária** | Idade contínua ou Faixa Etária | **Negativo** ($-0{,}15$ a $-0{,}25$) | **Defasagem Escolar:** Candidatos mais velhos pontuam menos devido ao tempo de afastamento dos estudos, reprovações anteriores ou necessidade de conciliar trabalho e escola. |

### 💡 Conclusões (2019–2023)

1. **Ridez do Hiato Público vs. Privado:** A correlação da rede pública manteve-se presa no patamar negativo de $-0{,}45$ durante todo o período. Isso demonstra que as perdas educacionais acumuladas durante o período pandêmico atingiram os estudantes da rede pública de forma severa e uniforme em todas as regiões brasileiras.
2. **Prevalência do Factor Econômico:** Renda familiar e o tipo de gestão escolar (pública ou privada) continuam sendo os dois maiores preditores do desempenho no ENEM. O gráfico evidencia que o exame não avalia apenas o conhecimento individual, mas reflete diretamente as desigualdades socioeconômicas estruturais do Brasil.
