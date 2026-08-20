
link google colab: https://colab.research.google.com/drive/1KZAAyHxwOvHBa5UGyTq9yncKTCEctppS?usp=sharing

Relatório dia 13 de agosto

# Projeto Estratificação ENEM


---

## 📌 Sobre o Projeto

Os Microdados do ENEM contêm milhões de registros a cada edição, gerando arquivos extremamente pesados que dificultam a leitura direta e costumam travar a memória RAM de ambientes em nuvem, como o Google Colab.

Para resolver esse entrave e viabilizar análises ágeis e precisas, este projeto realizou a **amostragem estatística dos dados entre 2020 e 2024**. O objetivo principal foi reduzir o tamanho dos arquivos sem perder a qualidade da informação, mantendo um retrato fiel da realidade educacional do país.

---

## 🎯 Amostragem Estratificada (10% Representativo)

Para garantir que os dados reflitam com precisão a diversidade do país, não fizemos uma escolha aleatória simples. Utilizou-se a **Amostragem Estratificada por Estado (UF)**:

* **Proporção Exata:** Retiramos exatamente **10% dos participantes de cada estado brasileiro**.
* **Representatividade Real:** Estados com maior número de inscritos mantêm seu peso proporcional exato na amostra final.
* **Agilidade na Análise:** A redução para 10% do volume total mantém total confiabilidade estatística e permite gerar gráficos e modelos com rapidez.

---

## 💡 Processamento em Blocos (Proteção de Memória RAM)

Como cada arquivo anual possui gigabytes de tamanho e mais de 100 colunas, carregar tudo de uma só vez estoura os limites de RAM do servidor.

A solução aplicada foi o **processamento em blocos (`chunksize`)**:
1. O código lê o arquivo em fatias de 100.000 linhas.
2. Extrai a amostra de 10% proporcional de cada estado para aquele bloco.
3. Descarte o excesso da memória temporária e passa para a próxima fatia.

Markdown
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

---
