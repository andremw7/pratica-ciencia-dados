link google colab:
  Estratificação por região: https://colab.research.google.com/drive/1NinqFDrJEfxpv7NSHcP6VAH2_dYe9pGf

# 📊 Análise Exploratória do ENEM: Desempenho Regional e Impacto Socioeconômico (2020–2024)

## 📌 Apresentação e Objetivos do Projeto
Este projeto analisa as disparidades de desempenho no Exame Nacional do Ensino Médio (ENEM) entre 2020 e 2024. O foco técnico reside em investigar como fatores estruturais — especificamente o capital financeiro, o capital cultural e a classe ocupacional familiar — atuam sobre as notas dos candidatos, utilizando a mediana estatística como métrica central e visualizações interativas para apoio à tomada de decisão.

---

## 🗓️ Cronograma Metodológico e Histórico de Desenvolvimento

### 📅 Semana 1 — 13 de Agosto: Amostragem Estratificada e Preparação dos Dados
- **Objetivo Técnico:** Reduzir a carga computacional sem comprometer a representatividade estatística do território nacional.
- **Procedimento Executado:** Os microdados brutos abarcavam milhões de inscritos por edição. Foi desenvolvida uma rotina de amostragem aleatória estratificada proporcional às Unidades Federativas (UFs) e aos anos de aplicação (2020 a 2024). Essa técnica garantiu que estados com menor densidade demográfica mantivessem sua representatividade proporcional, permitindo cruzamentos sem viés de seleção regional.

### 📅 Semana 2 — 20 de Agosto: Análise Exploratória Geral com Boxplots Por Área
- **Objetivo Técnico:** Avaliar a dispersão, assimetria e presença de valores discrepantes (outliers) na distribuição das notas.
- **Procedimento Executado:** Criação de diagramas de caixa (Boxplots) para as cinco áreas do conhecimento:
  - Matemática e suas Tecnologias (`NU_NOTA_MT`)
  - Redação (`NU_NOTA_REDACAO`)
  - Linguagens, Códigos e suas Tecnologias (`NU_NOTA_LC`)
  - Ciências Humanas e suas Tecnologias (`NU_NOTA_CH`)
  - Ciências da Natureza e suas Tecnologias (`NU_NOTA_CN`)
- **Ajuste Estatístico:** A média aritmética mostrou-se sensível aos desempenhos atípicos nas extremidades. Estabeleceu-se formalmente a **mediana (Q2 / percentil 50)** como o indicador de centralidade oficial do projeto, isolando o efeito de *outliers*.

### 📅 Semana 3 — 27 de Agosto: Segregação Regional e Mapeamento dos Estados Extremos
- **Objetivo Técnico:** Mapear a assimetria educacional inter e intra-regional.
- **Procedimento Executado:** Os dados foram rotulados pelas cinco regiões geográficas do Brasil (Sudeste, Nordeste, Sul, Centro-Oeste e Norte). Para cada cruzamento entre **Ano × Área do Conhecimento × Região**, o algoritmo calcula as medianas dos 27 estados e seleciona automaticamente:
  - 🏆 **Estado Campeão:** A Unidade Federativa com a maior mediana regional.
  - 🔻 **Estado com Menor Desempenho:** A Unidade Federativa com a menor mediana regional.

### 📅 Semana 4 — 03 de Setembro: Modelagem Preditiva Por Área do Conhecimento e Matriz de Importância (Heatmap)

#### 🎯 Objetivo Técnico
Isolar o impacto das variáveis socioeconômicas em cada uma das 5 disciplinas do ENEM individualmente, respeitando as especificidades da Teoria de Resposta ao Item (TRI) e evitando a distorção provocada pela mistura de notas de provas distintas.

#### 🛠️ Procedimento Executado
1. **Decomposição do Target por Disciplina:**
   - Em vez de unificar as notas em uma métrica geral, foram treinados 5 modelos independentes de *Random Forest Regressor*, um para cada área do exame: Matemática (`NU_NOTA_MT`), Redação (`NU_NOTA_REDACAO`), Linguagens (`NU_NOTA_LC`), Ciências Humanas (`NU_NOTA_CH`) e Ciências da Natureza (`NU_NOTA_CN`).
2. **Engenharia de Recursos e Codificação (Feature Engineering):**
   - **Variáveis Ordinais (`Q001`, `Q002`, `Q006`, `Q022`, `Q024`):** Mapeamento numérico hierárquico para preservar a escala de intensidade de Renda, Escolaridade e Equipamentos.
   - **Variáveis Nominais (`TP_ESCOLA`, `Q003`, `Q004`, `Q025`):** Aplicação de One-Hot Encoding para categorias sem ordem intrínseca (Ocupação dos pais, Tipo de escola e Internet).
3. **Mensuração do Poder Explicativo:**
   - Extração do MDI (*Mean Decrease in Impurity*) de cada modelo e reagrupamento das variáveis Dummy às suas colunas originais, convertendo as contribuições em porcentagens de explicação por disciplina.
4. **Matriz Comparativa de Impacto (Heatmap):**
   - Construção de um Mapa de Calor relacionando as variáveis socioeconômicas às 5 áreas do conhecimento, permitindo identificar visualmente quais fatores mais pesam em cada prova.

#### 📊 Principais Achados
- **Disparidade por Disciplina:** Fatores econômicos (Renda e Computadores) possuem peso desproporcionalmente maior em provas exatas (Matemática).
- **Peso do Capital Cultural:** A Escolaridade da Mãe e o Tipo de Escola exercem papel predominante no desempenho em Redação e Ciências Humanas, evidenciando como a bagagem cultural e a infraestrutura escolar afetam a capacidade interpretativa e de escrita.
- **Acesso à Internet (`Q025`):** Presença de conexão banda larga/móvel na residência.
- **Computadores em Casa (`Q024`):** Quantidade de desktops/notebooks disponíveis para estudo.

### 📅 Semana 5 — 10 de Setembro: Validação Preditiva e Métricas de Generalização (80/20 Train-Test)

#### 🎯 Objetivo Técnico
Validar a capacidade preditiva dos 5 modelos de *Random Forest* em dados inéditos, mensurando a margem de erro das previsões e avaliando a proporção de variância das notas explicada exclusivamente pelo contexto socioeconômico do estudante.

#### 🛠️ Procedimento Executado
1. **Divisão Treino/Teste (80/20):**
   - A amostra foi particionada de forma aleatória: 80% dos registros foram destinados ao treinamento do algoritmo e 20% foram isolados estritamente para simular a previsão em alunos inéditos.
2. **Avaliação por Métricas de Regressão:**
   - **$R^2$ (Coeficiente de Determinação):** Mede a porcentagem da variação da nota explicada pelo modelo.
   - **MAE (Erro Médio Absoluto):** A margem de erro média direta na escala de pontos do ENEM.
   - **RMSE (Erro Quadrático Médio):** Penaliza erros mais graves, identificando oscilações atípicas no resultado.

#### 📊 Desempenho Preditivo do Modelo por Disciplina

| Área do Conhecimento | $R^2$ (Poder Explicativo) | Erro Médio (MAE) | Erro Quadrático (RMSE) |
| :--- | :--- | :--- | :--- |
| **Matemática** | 21,24% | 84,88 pts | 104,48 pts |
| **Redação** | 12,77% | 146,03 pts | 194,54 pts |
| **Linguagens** | 18,37% | 52,78 pts | 66,52 pts |
| **Ciências Humanas** | 18,00% | 63,85 pts | 80,07 pts |
| **Ciências da Natureza** | 18,39% | 57,62 pts | 71,58 pts |

#### 💡 Análise e Justificativa Científica dos Resultados
- **Sensibilidade Econômica em Exatas ($R^2 = 21,24\%$):** Matemática é a área onde o perfil socioeconômico mais abre vantagem, demonstrando que acesso a boas escolas e infraestrutura tecnológica afeta diretamente o desempenho lógico.
- **Autonomia Individual na Redação ($R^2 = 12,77\%$):** Registrou o menor poder explicativo e o maior erro médio (146 pts). Isso evidencia que o aprendizado de escrita, treino prático e o risco de desvios ao tema dependem de fatores individuais do candidato, e não da sua renda.
- **Validação Acadêmica:** Explicar entre 12% e 21% da variação com dados exclusivamente socioeconômicos é um resultado consistente na literatura educacional. Os aproximadamente 80% restantes da pontuação pertencem a fatores não computados no questionário (horas de estudo individual, suporte pedagógico e estabilidade emocional no dia da prova).

---

### 📅 09 de Outubro (Quinta-feira Retrasada): Análise Visual de Densidade e Bimodalidade

#### 🎯 Objetivo Técnico
Investigar a forma real da distribuição das notas na rede pública para justificar estatisticamente a escolha da Mediana como métrica oficial.

#### 🛠️ Procedimento Executado
- Construção de gráficos de densidade probabilística (KDE - *Kernel Density Estimation*) utilizando as notas da amostra validada para as cinco regiões do país, segmentando as curvas por matéria e ano.
- **Principais Achados:** A visualização gráfica comprovou que a distribuição das notas não é perfeitamente simétrica (normal). Em diversas áreas, a curva apresenta forte assimetria ou comportamento bimodal (múltiplos picos). Isso justificou e validou matematicamente ao professor que a média seria puxada para "vales" distorcidos por notas muito extremas, tornando imperativo o uso focado na **Mediana** e no formato da curva para análises de políticas públicas mais realistas.

---

### 📅 16 de Outubro (Quinta-feira Passada): Embate Regional (Ceará vs. Sudeste na Rede Pública)

#### 🎯 Objetivo Técnico
Realizar um cruzamento e comparação direta e isolada entre a rede pública estadual do Ceará (famosa por suas políticas de educação) contra os estados mais ricos do Sudeste (SP, RJ, MG e ES).

#### 🛠️ Procedimento Executado
- Criação de um painel interativo dinâmico (com `ipywidgets`, `seaborn` e `matplotlib` no Google Colab) para renderizar curvas sobrepostas apenas de estudantes da rede pública, testando o desempenho entre 2020 e 2024.
- **Principais Achados:** O cruzamento visual permitiu enxergar onde se posiciona o "bolo" principal de alunos. Ficou evidente como a curva do Ceará busca reduzir drasticamente a chamada "cauda inferior" (garantindo um piso de nota para alunos de menor renda), evidenciando os impactos do modelo cearense de tempo integral em manter os estudantes competitivos em relação aos gigantes econômicos do Sudeste.

---

### 📅 23 de Outubro (Hoje): Revisão Metodológica e Tratamento do Viés de Seleção (Orientação do Professor)

#### 🎯 Objetivo Técnico
Reavaliar a interpretação da análise de densidade após as observações do professor, buscando entender as anomalias onde o Sudeste sobressai em matérias de capital cultural.

#### 🛠️ Procedimento Executado e Próximos Passos
- Conduzimos uma análise crítica das curvas geradas (como a de Ciências Humanas em 2024), onde os picos do Sudeste se mostraram mais deslocados para a direita (notas maiores) quando comparados ao Ceará.
- **Conclusão Metodológica Central:** Entendemos, a partir da orientação recebida, que a nota bruta nas curvas ainda carrega fortemente o capital cultural e econômico familiar (PIB regional), além de sofrer de um **Viés de Seleção** fortíssimo. Como o estado do Ceará adota a política de impulsionar quase $100\%$ dos alunos da rede pública a prestarem o ENEM, a sua base representa a população real por completo. Em contrapartida, no Sudeste, a ausência dessa política faz com que apenas os alunos já mais bem preparados prestem a prova, inflacionando a curva e distorcendo a avaliação pura da política pública escolar.
- **Próximas Etapas no Projeto:** Os dados evidenciam que precisaremos cruzar as distribuições obtidas não apenas com as notas em si, mas controlando os resultados através da **Taxa de Cobertura do ENEM** de cada estado e dos dados estruturais do Censo Escolar (como o percentual de escolas estaduais em Regime de Tempo Integral).
