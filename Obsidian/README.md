# Análise - Educação: Como as condições socioeconômicas e educacionais das regiões brasileiras estão relacionadas ao desempenho dos participantes do ENEM.

Este documento descreve o que foi feito no desenvolvimento do projeto e as justificativas para as decisões tomadas. O notebook Python utilizado (`ENEM_2020_2024_Etapa2.ipynb`) pode ser encontrado neste repositório na pasta `Obsidian`. Para encontrar a linha do tempo, acesse a pasta `Relatório contínuo` deste repositório.

---

## 1. Visão geral

**Tema do projeto**: como as condições socioeconômicas e educacionais das regiões brasileiras se relacionam ao desempenho dos participantes do ENEM.

O projeto foi dividido em três etapas:

| Etapa | O que faz |
|---|---|
| **Etapa 1 - Análise Descritiva Inicial** | Constrói uma amostra padronizada dos microdados, valida a amostra e descreve a distribuição das notas por UF, região, área e ano |
| **Etapa 2 - Junção de Bases e Análises** | Integra o ENEM a Censo Escolar, PNAD/IBGE e indicadores do Inep; examina associações, composição dos participantes e perfis; define atributos candidatos e o problema de previsão |
| **Etapa 3 - Aprendizado de Máquina** | TBA - Construção do modelo preditivo |
<!-- Ajustar texto da etapa 3 -->

**Pergunta condutora:** onde estão as diferenças de desempenho, que características aparecem junto delas e quais informações já existentes antes do exame poderiam ajudar a antecipá-las?

> **Observação:** Originalmente, a proposta para o trabalho era encontrar todas as bases que seriam utilizadas e apenas depois fazer as visualizações. Entretanto, optou-se por fazer, primeiro, algumas visualizações do dataset principal que serviu como base, para, depois, procurar outras bases que complementassem os dados com base no que precisasse.

---

## 2. Etapa 1 — Definição do problema, construção da amostra e descrição das notas

### 2.0 Definição do problema

Tendo decidido a temática do trabalho, era necessário encontrar uma base de dados que serviria como pilar principal do projeto. Escolheu-se utilizar os microdados do ENEM, obtidos no site do governo (`https://www.gov.br/inep/pt-br/acesso-a-informacao/dados-abertos/microdados/enem `).

- **Janela temporal: 2020 a 2024**
  - **Por quê:** o ideal para uma análise é uma janela temporal de **no mínimo cinco anos**, para verificar se um padrão é persistente ou específico de um ano.

### 2.1 Configuração (seção 1)

- Importa bibliotecas e define constantes: semente, fração de 10%, tamanho dos blocos de leitura, anos e áreas.
  - Optou-se por ler os dados em blocos devido ao tamanho da base.
- Padroniza o visual dos gráficos e define cores fixas por região, para que a mesma região tenha a mesma cor em todo o notebook.
- Monta o Google Drive e define duas chaves de execução:
  - `PROCESSAR_MICRODADOS`: `True` refaz a amostra (etapa demorada); `False` apenas carrega os CSVs prontos.
  - `EXPORTAR_TABELAS`: salva as tabelas-resumo ao final.
  - Mudar filtros dos gráficos não exige refazer nada.
- **Por quê:** separar a etapa pesada da análise evita reprocessar os microdados a cada execução. Antes de processar, o código confere se todos os arquivos existem e para com mensagem clara se faltar algum.

### 2.2 Colunas e dicionários auxiliares (seção 2)

- Mapeia cada área às suas colunas de nota e presença (`NU_NOTA_*`, `TP_PRESENCA_*`).
- Cria dicionários de **região**, **código IBGE** e **nome** por UF.

> **Critério de elegibilidade:** Para cada área, entra quem tem **presença igual a 1 e nota preenchida**. Em Redação exige-se também `TP_STATUS_REDACAO = 1`. Isso serve para manter apenas quem de fato realizou a prova e teve nota válida.
> - Redação usa a presença de Linguagens (`TP_PRESENCA_LC`), pois as duas provas acontecem no mesmo dia.
> - Redações com outras situações de correção ficam de fora, mesmo que tenham nota.

### 2.3 Leitura eficiente (seções 3–4)

- Lê **apenas as colunas necessárias**, **em blocos**, com tipos pequenos (`Int8`, `Float64`, `string`).
- Reúne os blocos de **um ano por vez** antes de sortear, e libera memória entre áreas e anos.
- Grava em arquivo temporário (`.em_preparo.csv`) e só substitui os CSVs anteriores **depois que todos os anos terminam**.
- **Por quê:** o volume de dados é grande e o ambiente (Colab) tem memória limitada; a gravação temporária evita ficar com uma amostra pela metade se algo falhar.
- Produz três arquivos: a **amostra**, a **validação** (contagens por estrato) e a **cobertura** (quantos elegíveis têm UF da escola reconhecida).

> **Território = UF da escola (`SG_UF_ESC`):** Uma das colunas selecionadas era a de identificação do território. Foi selecionado `SG_UF_ESC`, pois o objetivo é analisar como condições de estudo, por exemplo, relacionadas a infraestrutura, se relacionam com o desempenho acadêmico (ENEM).
> - A UF **não é preenchida** com a UF da prova ou da residência quando falta. Quem não informa a UF da escola entra apenas na contagem de cobertura, não nas análises territoriais. Preencher seria inventar informação.

> **Amostra de 10% estratificada, sem reposição:** Como os microdados são volumosos, foi necessário amostrar para ser viável a manipulação.
>- **Por que aleatoriedade computacional:** neste contexto, com dados de qualidade limitada, uma estratégia baseada em aleatoriedade computacional ou reamostragem é suficiente.
>- **Estratificação por ano × UF × área:** garante que estados pequenos não sejam engolidos por estados grandes e que a composição da amostra acompanhe aproximadamente a dos elegíveis. O sorteio é feito uma vez por estrato (`sample(frac=0.10, replace=False)`).
> - **Tamanho arredondado para inteiro:** por exemplo, 2.000 elegíveis geram 200 notas. A validação aceita diferença de até 0,51 linha por estrato.
> - **Contagens de participação vêm de antes do sorteio** e nunca são multiplicadas por dez; assim os gráficos de participação usam o total real de elegíveis.

### 2.4 Carga e verificações (seções 5–6)

Ao carregar, o notebook:

- arredonda as notas para uma casa decimal, removendo resíduos de arquivos antigos gravados em `float32`;
- verifica que o tamanho sorteado corresponde a 10% (com tolerância de arredondamento);
- executa uma bateria de `assert`: ausência de valores nulos, UFs e anos válidos, notas finitas e não negativas, ausência de estratos repetidos, coerência entre região e UF, igualdade entre amostra lida e tabela de validação, e coerência das somas de elegíveis com e sem UF.

**Por quê:** se qualquer conferência falhar, a execução para, evitando misturar CSVs de versões diferentes. O texto é explícito sobre o limite: a conferência verifica o *tamanho*, **não prova aleatoriedade** nem elimina o viés de participação no ENEM.

### 2.5 Painéis exploratórios (seções 7–22)

| Seção | Painel | Para que serve |
|---|---|---|
| 7 | Participantes elegíveis e cobertura da UF escolar | Mostra quantos elegíveis têm UF informada por ano; **não é taxa de comparecimento** |
| 8 | Concentração de participantes por UF/região | Permite testar a hipótese inicial de que São Paulo concentra a maioria dos participantes |
| 9–11 | Medidas de posição e dispersão | Calcula média, mediana, N, desvio-padrão, assimetria, mínimo, máximo, P5, Q1, Q3, P95 e amplitude interquartil por UF, região e Brasil |
| 12 | Histogramas por UF, ano e área | Mostra onde as notas se concentram; marca média e mediana |
| 13 | Shapiro–Wilk e gráfico Q–Q | Diagnóstico opcional, sem influenciar a medida adotada |
| 14 | Mapa coroplético da mediana por UF | Analisar distribuição geográfica das medidas visualmente |
| 15 | Evolução regional | Linhas por região, uma área ou todas, com eixos próprios |
| 16 | Ranking de UFs | Ordena pela mediana; empates recebem a mesma posição |
| 17 | Evolução dos estados dentro de uma região | Mostra também empates na maior mediana de cada ano |
| 18 | Mapa de calor região × área | A cor é a **posição** (ranking) dentro de cada área, para não misturar escalas |
| 19 | Percentual de notas acima de um limite | Limite escolhido pelo usuário; limite não é nota oficial de aprovação |
| 20 | Volume de participantes × mediana | Dispersão por UF (eixo logarítmico opcional); não demonstra causalidade |

### 2.5.1 Justificativas

Nesta seção, serão apresentadas as principais justificativas para as escolhas e decisões de projeto relacionadas às visualizações.

- **Representação das áreas de conhecimento:** Cada área do ENEM (Matemática, Ciências Naturais, Ciências Humanas, Linguagens e Redação) tem um sistema de pontuação diferente devido ao TRI. Por exemplo, enquanto a pontuação máxima de linguagens fica entre 790 e 820, em matemática é possível atingir 1000 pontos.
  - **Consequência:** para não perder características devido a esse desbalanceamento, todo painel de participantes fixa uma área.

- **Mediana como medida principal:** Como o objetivo era resumir o *centro* da distribuição das notas com menor sensibilidade às caudas, a mediana era a melhor medida para avaliação, que depende só da ordenação, não exige normalidade e não é deslocada por notas muito altas ou muito baixas.
  - **Ressalvas assumidas:** a média não é "errada", e nenhuma medida resume tudo.

- **Teste de Shapiro–Wilk:** O teste aparece apenas como **diagnóstico exploratório** (seção 13), e o p-valor nunca decide entre média e mediana.
  - Ele testa normalidade; **não compara a qualidade** de média e mediana.
  - Trocar a medida de um estado para outro conforme o p-valor misturaria alvos diferentes no mesmo mapa.
  - Amostras grandes detectam desvios mínimos; as notas são arredondadas e empatadas; participantes compartilham escolas. Independência e continuidade são aproximações.
  - Redação fica fora do teste por ter escala discreta e muitos empates.
  - Descarte do qui-quadrado: categorizar notas em faixas não resolve a escolha da medida central.

- **Resumos regionais e nacional partem das notas, não das medianas estaduais:** Calcular média ou mediana das medianas daria o mesmo peso a UFs de tamanhos muito diferentes e não corresponderia à mediana real da região.

- **Mapa de calor por posição:** as áreas têm escalas diferentes (Redação em particular), então comparar pontos entre colunas seria enganoso.

- **Cada gráfico exibido isoladamente com filtros** responde à tarefa geral de avaliar qual tipo de gráfico representa melhor a distribuição por cidade, escola e região: uso de histograma (forma da distribuição), barras horizontais ordenadas (comparação entre UFs), linhas (tempo) e mapas (geografia).

### 2.6 Preparo para a próxima etapa (seção 21)

Prepara as bases estadual, regional e nacional, com uma linha por ano × território × área, fração amostral e medida principal. Exporta em CSV se `EXPORTAR_TABELAS = True`.

- O resultado do Shapiro **não entra** como atributo, pois é calculado a partir da própria distribuição que se quer prever.
- Alerta-se que UF × ano dá só **135 combinações**, e que as cinco áreas não são cinco observações territoriais independentes.
- A cobertura só existe em nível nacional por ano e área: como não se sabe a UF de quem não a informou, não é possível calcular cobertura por UF.

---

## 3. Etapa 2 — Desigualdades e condições dos territórios

Depois de descrever as distribuições na Etapa 1, pergunta-se **se estão sendo comparados grupos semelhantes, como variam os perfis dos participantes e quais condições territoriais acompanham as notas**, para, então, a Etapa 2 focar em **quais condições econômicas e escolares acompanham as diferenças** de desempenho. A finalidade declarada é apoiar o diagnóstico de necessidades educacionais, e **não** decidir sozinha a distribuição de recursos.

**Unidades e universos usados:**

| Unidade | Onde entra | Universo |
|---|---|---|
| Inscrição por edição | Situação escolar entre inscritos | Arquivo público de cada edição |
| Nota por inscrição e área | Distribuições e comparações de redes | Amostra de 10% de elegíveis com UF escolar |
| Nota por inscrição e área | Cobertura, evolução e decomposição | Resumos de todos os elegíveis do recorte |
| Inscrição com quatro provas | Perfis relativos | Amostra conjunta de 10% por UF e ano |
| Escola por ano | Oferta e infraestrutura | Escolas ativas com ensino médio no Censo |
| UF por ano e área | Associações e alvo futuro | Resumos do ENEM ligados a contexto territorial |

- **Por quê:** as notas das cinco áreas permanecem separadas (escalas diferentes) e as edições são **cortes sucessivos de participantes**, não os mesmos alunos acompanhados no tempo. Deixar a unidade explícita em cada painel evita comparar universos diferentes sem perceber.

**Fontes integradas:**

| Fonte | O que traz | Chave |
|---|---|---|
| ENEM (Inep) | Notas, presença, UF, rede | UF + ano + área |
| Censo Escolar (Inep) | Recursos de infraestrutura, matrículas, tempo integral | UF + ano (rede, no recorte estadual) |
| PNAD Contínua (IBGE, tabela 7531) | Renda domiciliar média por pessoa | UF + ano |
| Indicadores educacionais (Inep) | Formação docente, horas-aula, rendimento | UF + ano (+ rede quando disponível) |

O FUNDEB, que chegou a fazer parte da proposta, foi retirado (ver 3.5).

### 3.0 Navegação dos painéis

- Os painéis são interativos (filtros de **Área**, **Ano**, **Período** e, quando há mais de uma pergunta no mesmo painel, **Vista**), e cada figura tem botão para baixar o recorte exibido em PNG.
- **Escalas:** painéis de proporção usam sempre **0% a 100%**; frequências, caudas e mapas de calor ajustam os limites ao recorte exibido.

### 3.1 Cobertura, elegibilidade e situação escolar

- **Auditoria nacional: o que muda quando exigimos UF escolar?** Compara a cobertura (notas com UF ÷ todas as notas elegíveis) e a média de quem tem UF com a de quem não tem. Três recortes cumprem funções diferentes:

| Recorte | Para que serve |
|---|---|
| Todos os elegíveis, com e sem UF escolar | Auditoria nacional de cobertura e seleção |
| Amostra limpa de ~10%, com UF escolar | Histogramas, percentis, caudas e comparações por rede e localização |
| Resumos completos dos elegíveis com UF escolar | Trajetórias, indicadores de desempenho e integração com as fontes por UF |

- **Quem se inscreve e quem aparece nas notas?** Usa `TP_ST_CONCLUSAO`, que é a situação de conclusão do Ensino Médio.
  - Três painéis: inscrições nacionais por situação (denominador inclui ausentes da prova), auditoria de seleção por situação e faixas de nota por situação escolar.
  - Como em 2024 a variável está em um arquivo e as notas em outro, **2024 entra só no painel de inscritos** e fica indisponível no cruzamento situação × nota.

- **O sorteio preserva a composição observável?** Confere-se contagens por área, cobertura, mediana estadual, contagens por rede e localização, o multiconjunto de notas sorteadas e a amostra conjunta. Compara a participação de cada rede/localização na amostra e no total **dentro de cada ano × UF × área**, medindo a diferença em pontos percentuais.
  - **Por quê:** a estratificação é por ano × UF × área; rede e localização são apenas verificações posteriores. Isso resulta em 675 estratos (27 UFs, 5 anos e 5 áreas) (mais detalhes na seção 3.5).

### 3.2 Distribuições, território e mudanças no tempo

| Painel | Pergunta | Cuidado metodológico |
|---|---|---|
| Histograma + densidade | O centro resume bem as notas ou esconde grupos distintos? | Linhas de média e mediana; faixa Q1–Q3. Três curvas suavizadas sobre notas positivas; se um pico some ao aumentar a suavização, isso não prova que ele era falso. Contagem de picos é diagnóstico exploratório, não teste de bimodalidade. Notas zero ficam nas medidas e saem só das curvas. Redação é discreta: mostram-se frequências. |
| Comparação de distribuições | Uma mediana parecida esconde distribuições diferentes? | Vistas: entre regiões, entre UFs da região e entre edições. Distribuição acumulada; mediana com P25–P75 e P10–P90; **caudas** abaixo de P20 e acima de P80 da amostra nacional (cortes fixos); contribuição de cada faixa à diferença de médias. Região é resumida a partir das notas reunidas, então UFs maiores pesam mais. |
| Visão geral das cinco áreas | A cauda escolhida é parecida entre áreas, regiões e edições? | Mapa de calor da parcela na cauda; corte próprio por área; células com menos de 30 notas ficam em branco. |
| Trajetórias | A queda está em todas as áreas e todos os anos? | Variação em pontos entre anos consecutivos; **2020 fica ausente (não há 2019), o que não equivale a zero**. Q1–Q3 descreve dispersão, não incerteza da mediana. |
| Mudanças entre edições | Como a distribuição mudou entre duas edições? | Por UF ou região, para mediana, P5, P95, Q1 e Q3. Se o P5 cai mais que a mediana, a parte inferior mudou mais. |

- Normalidade não é requisito para usar a média nem critério automático para escolher a mediana; havendo grupos distintos, eles devem ser descritos também. As curvas de densidade são ferramenta de leitura, não método para identificar perfis de alunos.
- Todas as comparações descrevem a amostra de elegíveis **com UF escolar informada**; diferenças entre territórios e edições também podem refletir cobertura, composição e características da prova.

### 3.3 Composição dos participantes

- **Comparações dentro da rede e da localização (urbana/rural):** permite comparar qualquer par de UFs ou regiões. Agrega as notas da amostra por território, ano e área, sem nova amostragem por filtro. Grupos com menos de 30 notas não recebem curva (precaução de leitura, não garantia de precisão).
  - Rede privada **não** significa escola preparatória: os arquivos não informam cursinho ou intensidade de preparação. Diferença urbana/rural também não mede, sozinha, o efeito do isolamento geográfico.

- **Mudança na participação das redes e decomposição da média:** o primeiro gráfico mostra a participação de uma rede por ano. O segundo compara duas edições nos grupos **UF × rede × localização**: variação observada da média, variação mantendo a composição inicial e a diferença entre as duas. O percentual de notas dos grupos comuns informa a abrangência.
  - A decomposição é descritiva, vale para a **média**, depende do ano inicial como referência e não separa renda, seleção dentro dos grupos nem mudanças no exame.

### 3.4 Desempenho relativo da mesma pessoa

**Diferença de posição entre pares de áreas.** Pergunta se um mesmo participante ocupa posições diferentes nas quatro provas objetivas.

- **Exige amostra nova:** os sorteios por área da Etapa 1 foram independentes e não servem. Usa-se uma amostra **conjunta de 10% por UF e edição**, com semente fixa, entre elegíveis com as quatro provas válidas e UF escolar conhecida.
- Cada prova vira um **percentil** na distribuição nacional do recorte elegível da edição. Compara-se a média dos percentis de MT/CN com a de LC/CH. A faixa de "posições próximas" é uma regra descritiva ajustável (corte de 10 a 20 pontos percentuais), **não** um diagnóstico de aptidão.
- A dispersão usa até 2.500 pontos; as barras comparam a parcela de um perfil entre UFs de uma região ou entre regiões, com denominador igual a todas as observações do grupo.

**Posições próximas podem ser altas ou baixas.** Cruza as médias dos percentis em quadrantes, com duas definições para testar se a conclusão depende delas: **corte 50** (alto a partir de 50) e **corte 40/60** (≤40 baixo, ≥60 alto, o restante é intermediário). A prevalência de cada perfil só é exibida com pelo menos 30 participantes.

- O grupo de renda familiar vem de `Q006` até 2023; **em 2024, renda e situação escolar ficam indisponíveis nesse cruzamento** (não se tenta ligar arquivos por posição de linha).

**Auditoria nacional: situação escolar e perfis.** Entre 2020 e 2023, a amostra conjunta com UF escolar é formada integralmente por concluintes declarados. Para comparar situações, o notebook amplia o universo para **todos os elegíveis com quatro provas válidas no país** (inclusive sem UF), com **novos percentis nacionais** calculados nesse universo. Por isso os percentuais desse painel e os dos quadrantes territoriais têm referências diferentes e não devem ser misturados. Em 2024, os arquivos públicos separados impedem classificar perfis por situação.

**Documentação do período e diagnóstico da TRI.** Tabela de contexto histórico, tratado como **contexto, não como variável de tratamento**:

| Referência | Como entra na leitura |
|---|---|
| 2020–2021: interrupção das aulas presenciais | Torna plausível investigar consequências posteriores; não mede quanto cada participante perdeu |
| 2023: criação do Programa Escola em Tempo Integral | Motiva observar oferta e jornada; anúncio não é execução, nem toda matrícula integral é adesão ao programa |
| 2024: apoio ao acesso ao ENEM no Ceará (transporte gratuito) | Pode motivar estudos de participação; não demonstra aumento de nota |
| ENEM: notas e comparabilidade (TRI) | Acertos iguais não exigem notas iguais; diferença entre anos não comprova mudança do método |

- **Conclusão sobre a pandemia:** os dados **não** permitem atribuir a queda de 2024 a um "lag da pandemia" (sem acompanhamento individual, exposição escolar detalhada nem grupo contrafactual).
- **Conclusão sobre escolas preparatórias:** não há variável comparável em todas as UFs e anos; a hipótese fica pendente e a rede privada não é usada como substituto.
- **Diagnóstico de acertos (2023 e 2024):** relaciona número de acertos e nota, para ilustrar a TRI. Reúne notas elegíveis com e sem UF, fica separado da integração por estado e se limita aos anos já conferidos nessa preparação.

### 3.5 O que acrescentam as fontes externas?

**Fontes, unidades e chaves:** Utiliza-se da estratificação por **UF da escola × edição × área**, partindo dos resumos completos das notas elegíveis com UF conhecida.

- **A junção é territorial (UF), não individual:** A expectativa inicial era ligar ENEM e Censo Escolar para avaliar a infraestrutura da escola de cada aluno. Entretanto, como não há chave de escola comparável em todos os outros arquivos usados, o Censo agora representa a **oferta de ensino médio da UF**, e não a escola frequentada por cada participante. Não se tenta localizar pessoas por nome, nota ou posição de linha.
- **Junção `many_to_one` com `validate` e `indicator`:** as cinco áreas de uma UF/ano recebem o mesmo contexto, sem criar participantes novos.
- **Por quê resumir o Censo por UF e ano antes de juntar:** o Censo tem uma linha por escola; juntar direto multiplicaria linhas do ENEM.
- **Renda:** a variável 10824 da PNAD é o rendimento médio mensal real domiciliar por pessoa, a preços médios do próprio ano, que foi reexpressada em **preços de 2024 pelo IPCA** (tabela 1737) para a descrição e guarda a série em **preços de 2020** para a Etapa 3. Ressalva: o IPCA é nacional, não o custo de vida de cada UF.
- **Disponibilidade:** antes de juntar, confere-se que as seis tabelas anuais (ENEM, Censo, PNAD, formação docente, horas-aula e rendimento) têm as 27 UFs em cada ano.

>**Exclusão do FUNDEB mensal.** Aumentar a quantidade de fontes não garante uma base melhor. O FUNDEB disponível descreve **apenas dezembro**, não o investimento anual no ensino médio estadual, e dividir o valor pelas matrículas não corrige a incompatibilidade. Foi retirado dos gráficos de associação, da junção principal e das entradas de previsão. Uma ampliação futura precisaria de gasto executado anual, rede e etapa compatíveis, correção monetária e data de divulgação conhecida.

**Recorte da rede estadual:** Para aproximar o diagnóstico da atuação de uma secretaria, notas, infraestrutura e indicadores passam a ser da **rede estadual**; a renda segue sendo da UF inteira, com essa diferença explícita. A delimitação vem da pergunta de gestão e da compatibilidade das fontes, não de buscar correlação maior.

- Recalcula mediana, média e quartis diretamente de todas as notas elegíveis estaduais (não junta medianas urbanas e rurais). A amostra é mantida só para comparar estimativa amostral com resumo completo.
- Sequência: (1) resumo amostral ao completo, `ANO` + `UF_ESCOLA` + `AREA`, `one_to_one`; (2) contextos por `ANO` + `UF_ESCOLA`, `many_to_one`. A média é conferida pela soma ponderada dos grupos (válida para a média, não para a mediana).
- As bases geral e estadual compartilham participantes e **não devem ser empilhadas** como exemplos independentes.

**A chave existe. Os indicadores têm informação suficiente?** A auditoria separa chave correspondente de valor preenchido, contando ausentes por variável em UF × ano (sem repetir o contexto cinco vezes). Para recursos, cada escola com resposta vale uma unidade; para jornada, o numerador soma matrículas integrais e o denominador, matrículas de ensino médio. O Grupo 1 da formação docente mede adequação à disciplina lecionada, sem separar áreas do ENEM. Nenhuma junção identifica exposição individual.

### 3.6 Oferta escolar e associações com o desempenho

**Oferta escolar:**

- **Infraestrutura por território:** percentual de escolas de ensino médio com cada recurso (água, energia, esgoto, internet, laboratório, biblioteca etc.), por região ou UF, ano, rede e localização.
- **Visão geral dos recursos:** mapa de calor com todos os recursos no mesmo painel; a escala de cor acompanha os percentuais exibidos. A cor é disponibilidade, não qualidade.
- **Evolução da oferta (2020–2024):** acompanha um recurso ou o tempo integral. O tempo integral usa **matrículas** como peso; os demais usam escolas com resposta válida. Não deve-se atribuir todo o aumento ao programa federal de 2023, pois já havia iniciativas estaduais e escolas de tempo integral.

**Associações com o desempenho:**

- **Associações territoriais:** correlação de **Spearman** entre um indicador e a mediana, uma característica por vez, com cada ponto sendo uma UF no mesmo ano e área (universos: todas as redes ou rede estadual).
  - **Por quê Spearman:** só considera a ordem dos estados, é robusto a valores extremos e não pressupõe relação linear.
  - Coeficientes são **exploratórios**: o gráfico por ano mostra se a relação persiste; uma relação que aparece em um só ano merece cautela.
  - Aprovação, reprovação e abandono são relacionados entre si, então não entram todos automaticamente num modelo.
- **Estabilidade por área e edição:** mapa de calor com todas as áreas (sem somar notas), registrando quantas UFs entram em cada cálculo. A cor é associação, **não** importância de variável, porcentagem explicada ou efeito de política.
- **Diagnóstico conjunto de renda, notas e recursos:** gráfico da rede estadual com renda (eixo x), mediana (eixo y), cor por região e tamanho do ponto proporcional à proporção de escolas estaduais **sem** o recurso. **Não é ranking de prioridade**: antes de qualquer decisão seria preciso conferir cobertura, desigualdades internas, demanda local e outras avaliações, como o Saeb.

### 3.7 Quais relações são estáveis?

**Redundância dos atributos candidatos.** A escolha parte da pergunta, não da quantidade de colunas. Lista inicial: renda, tempo integral, formação docente, abandono, internet e laboratório de ciências, com a mediana do ano anterior como referência indispensável.

| Informação | Decisão inicial |
|---|---|
| Mediana por área | Resultado a estimar; média como análise de sensibilidade |
| Mediana do ano anterior | Referência simples: comparar o futuro modelo com "repetir a nota anterior" |
| Renda real por pessoa | Manter uma versão de preços; não usar nominal e deflacionada juntas |
| Matrículas integrais, formação docente, abandono, internet e laboratório | Candidatas (presença não mede qualidade ou uso) |
| Horas-aula, biblioteca, água e outras condições | Alternativas e diagnóstico; revisar redundância antes do treino |
| N, cobertura e respostas válidas | Guardar para auditoria; não usar como "nota de qualidade" |
| FUNDEB de dezembro | Excluir até haver série anual de execução adequada |
| Perfis feitos com as notas do mesmo ano | Não usar para prever as próprias notas que os definiram |

O gráfico de redundância usa **Spearman entre candidatos, com uma linha por UF no ano escolhido**, para não repetir cada UF cinco vezes por causa das áreas. Correlação alta é sinal para avaliar alternativas, não critério automático de exclusão.

**Forma das associações e influência das UFs.** O Spearman sozinho não responde a toda a pergunta. A seção compara três medidas, para os seis candidatos, em cada ano e área:

- **Pearson:** alinhamento em uma reta;
- **Spearman:** concordância de ordenações, aceitando curvas crescentes ou decrescentes;
- **correlação de distâncias:** pode detectar outras dependências, inclusive formas em U; não tem sinal, e valor amostral positivo não comprova associação.

Cuidados e critérios:

- **Influência de cada UF:** repete-se o Spearman retirando uma UF por vez; o intervalo mínimo–máximo mede influência, **não** é intervalo de confiança.
- **Permutações dentro das regiões:** para Pearson e distância, 999 permutações do resultado dentro de cada região, preservando a composição regional. A referência pergunta se há mais associação do que a obtida ao reorganizar UFs dentro das regiões; pressupõe que essas trocas sejam razoáveis sob a hipótese nula, e dependência espacial limita a interpretação.
- **Múltiplos testes:** correção Benjamini–Yekutieli sobre os **300 testes** (6 atributos × 5 áreas × 5 anos × 2 medidas). Reduz falsas descobertas sob dependência, mas não corrige um teste cuja hipótese de permutação seja inadequada.
- Um teste de distância significativo **não prova** relação especificamente não linear. A utilidade de permitir curvas será avaliada na previsão, na Etapa 3.

**A associação reaparece quando o contexto muda?** Contrasta duas perguntas na rede estadual, para renda, tempo integral e formação docente:

- **Entre UFs, em uma edição:** correlação dos postos bruta e **ajustada** (projeção linear dos postos retirando região e os outros dois indicadores). O ajuste é descritivo, depende dos controles escolhidos e pode ser instável com 27 UFs.
- **Mudanças anuais:** dentro de cada UF, variação da mediana e de cada atributo; Spearman entre UFs em cada uma das quatro transições, mostradas separadamente para não tratar a repetição de uma UF como independência. Sinal diferente entre níveis e mudanças orienta a investigação da série e da composição, sem identificar efeitos de políticas.
- Com apenas cinco edições, não é possível isolar um efeito atrasado da pandemia por simples correlação.

---

## 4. Preparação para a etapa 3

### 4.1 O problema e o tamanho dos dados

**Alvo:** prever a **mediana dos elegíveis do ENEM da rede estadual, por UF escolar e área, no ano seguinte**. A utilidade seria indicar onde aprofundar o diagnóstico territorial, não distribuir recursos automaticamente nem estimar efeito de política.

- **Validação cronológica:** todas as UFs de um mesmo ano-alvo entram juntas na avaliação, e o treino usa apenas anos-alvo anteriores. A **persistência da mediana anterior** será a referência de comparação; as áreas são tratadas separadamente.
- **O gargalo é o tempo observado, não o volume de participantes:** são 135 linhas por área na série completa (27 UFs × 5 anos) e **108 pares ano-origem → ano-alvo por área**, com as mesmas UFs em cada transição. Os milhões de notas tornam os resumos precisos, mas não criam milhões de exemplos territoriais.

### 4.2 Atributos, data de corte e tabela de transições

- **Atributos candidatos (do ano anterior):** mediana anterior, renda real por pessoa (em preços fixos de 2020, anterior a todos os alvos; os gráficos descritivos usam 2024), matrículas integrais, formação docente adequada, abandono, internet, laboratório de ciências e, como alternativa, horas-aula por dia.
- **Atributos derivados das notas do ano-alvo, perfis e contagens de controle** ficam em tabelas separadas e não entram como entrada.
- **Data de corte proposta: 30 de junho do ano-alvo.** Para cada série, a Etapa 3 deverá confirmar que a versão do indicador do ano anterior estava publicada até essa data. A auditoria atual só dispõe dos anos de referência, não das datas de publicação por edição de cada fonte; por isso cada linha traz `STATUS_TEMPORAL` = pendente. A mediana anterior também precisa passar por essa verificação.
- **Junção entre edições:** à esquerda, os alvos de 2021 a 2024; à direita, os atributos de 2020 a 2023, com a chave de ano deslocada em um e a edição de origem guardada em `ANO_REFERENCIA`. O código exige correspondência `one_to_one` e que a diferença entre anos seja sempre igual a um.

---

## 5. Limitações gerais registradas

1. O ENEM retrata **seus participantes**, não todos os estudantes; há seleção de quem faz a prova.
2. A cobertura da UF escolar varia entre anos e a ausência não é imputada.
3. A amostra descreve elegíveis **com UF da escola informada**; quem não informa só entra nas auditorias nacionais.
4. A série compara **grupos diferentes** de pessoas ao longo dos anos.
5. Associações entre UFs não são efeitos individuais nem causais; o ajuste parcial é descritivo e depende dos controles escolhidos.
6. Com 27 UFs e 5 anos, o volume de observações territoriais independentes é pequeno.
7. Mudanças de documentação, de cobertura e de composição entre edições dificultam atribuir quedas a um único evento.
8. Presença de um recurso (internet, laboratório) não mede qualidade nem uso efetivo.
9. Os perfis com UF escolar de 2020–2023 são todos de concluintes declarados; a comparação por situação escolar usa um universo nacional ampliado, com referência de percentis própria.
10. Dimensão cultural e cursinhos preparatórios não têm indicador comparável; o FUNDEB mensal foi excluído.
