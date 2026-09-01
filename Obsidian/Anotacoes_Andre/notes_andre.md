
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

