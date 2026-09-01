
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

### 📅 Semana 4 — 03 de Setembro: Arquitetura de Interatividade e Tratamento do Pipeline
- **Objetivo Técnico:** Desenvolver uma interface gráfica reativa imune a travamentos do ambiente de execução.
- **Procedimento Executado:** Estruturação do controle de fluxo via `ipywidgets`. Foi resolvido o gargalo de renderização em nuvem substituindo a geração dinâmica de JavaScript instável pela renderização estática reativa via gerenciador de saída (`Output`), limpando a memória a cada clique e garantindo a exibição do painel sem telas brancas.

### 📅 Semana 5 — 10 de Setembro: Painel Expandido de Raio-X Socioeconômico
- **Objetivo Técnico:** Mapear o perfil socioeconômico comparativo entre os estados de maior e menor mediana.
- **Procedimento Executado:** Construção de um painel visual composto por gráficos de barras horizontais ordenadas (eliminando gráficos de pizza por questões de usabilidade perceptiva), agrupando os indicadores socioeconômicos essenciais.

---

### 📅 Semana 4 — 03 de Setembro: Modelagem Não-Linear e Ranking de Importância de Variáveis (Feature Importance)

#### 🎯 Objetivo Técnico
Quantificar a capacidade explicativa das variáveis socioeconômicas sobre a nota média geral dos candidatos, superando as limitações da correlação linear clássica (Pearson) diante de dados mistos (ordinais e nominais).

#### 🛠️ Procedimento Executado
1. **Engenharia de Recursos e Codificação (Feature Engineering):**
   - **Variáveis Ordinais (`Q001`, `Q002`, `Q006`, `Q022`, `Q024`):** Mapeamento hierárquico numérico para preservar a escala de intensidade (ex: faixas de renda de A a Q e graus de escolaridade).
   - **Variáveis Nominais (`TP_ESCOLA`, `Q003`, `Q004`, `Q025`):** Aplicação de codificação Dummy/One-Hot Encoding para permitir o processamento algébrico sem impor ordem hierárquica artificial.

2. **Treinamento do Modelo Preditivo (Random Forest Regressor):**
   - Treinamento do modelo na totalidade da amostra nacional (2020–2024) utilizando a nota média das 5 áreas como variável alvo (*target*).
   - Configuração de hiperparâmetros otimizada (`n_estimators=100`, `max_depth=10`, `n_jobs=-1`) para garantir convergência rápida e prevenir *overfitting*.

3. **Agrupamento e Normalização dos Impactos:**
   - As contribuições das variáveis nominais codificadas foram reagrupadas às suas colunas de origem.
   - Cálculo da diminuição média de impureza (Mean Decrease in Impurity - MDI) convertida em porcentagem relativa de explicação.

#### 📊 Resultados Obtidos
- **Classificação Visual:** Construção de gráfico de barras horizontais categorizado por cores para diferenciar a natureza das variáveis (Azul = Ordinais; Verde = Nominais).
- **Hierarquia de Impacto:** Identificação clara dos fatores com maior poder explicativo no desempenho final do candidato, estabelecendo o ranking definitivo para a elaboração das conclusões da pesquisa.
## 🔑 Mapeamento das Variáveis Socioeconômicas Selecionadas
Para evitar ruído visual com perguntas de pouca relevância para a nota (como bens domésticos supérfluos), o projeto concentrou a análise em 8 variáveis chaves divididas em três pilares:

### 1. Capital Cultural Familiar
- **Escolaridade da Mãe (`Q002`):** Nível de instrução formal da responsável feminina.
- **Escolaridade do Pai (`Q001`):** Nível de instrução formal do responsável masculino.

### 2. Status Sócio-Profissional e Econômico
- **Renda Familiar Mensal (`Q006`):** Faixas salariais divididas de A a Q.
- **Ocupação do Pai (`Q003`):** Grupo ocupacional do pai (trabalho braçal, técnico, superior, gestão).
- **Ocupação da Mãe (`Q004`):** Grupo ocupacional da mãe.
- **Tipo de Escola (`TP_ESCOLA`):** Percurso escolar no Ensino Médio (Pública vs. Privada).

### 3. Inclusão e Infraestrutura Digital
- **Acesso à Internet (`Q025`):** Presença de conexão banda larga/móvel na residência.
- **Computadores em Casa (`Q024`):** Quantidade de desktops/notebooks disponíveis para estudo.

---

