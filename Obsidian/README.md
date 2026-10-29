# Análise - Educação: Como as condições socioeconômicas e educacionais das regiões brasileiras estão relacionadas ao desempenho dos participantes do ENEM.

Este documento descreve o que foi feito no desenvolvimento do projeto e as justificativas para as decisões tomadas. O notebook Python utilizado (`ENEM_desigualdades_2020_2024.ipynb`) pode ser encontrado neste repositório na pasta `Obsidian`.

---

## 1. Visão geral

**Tema do projeto**: como as condições socioeconômicas e educacionais das regiões brasileiras se relacionam ao desempenho dos participantes do ENEM.

O projeto foi dividido em três etapas:

| Etapa | Seções | O que faz |
|---|---|---|
| **Etapa 1 - Análise Descritiva Inicial** | 1 a 23 | Constrói uma amostra padronizada dos microdados, valida a amostra e descreve a distribuição das notas por UF, região, área e ano |
| **Etapa 2 - Junção de Bases e Análises** | 24 a 46 | Integra o ENEM a Censo Escolar, PNAD/IBGE e indicadores do Inep; examina associações, composição dos participantes e perfis; define atributos candidatos e o problema de previsão |
| **Etapa 3 - Aprendizado de Máquina** | TBA | Construção do modelo preditivo |
<!-- Ajustar texto da etapa 3 -->

**Pergunta condutora:** onde estão as diferenças de desempenho, que características aparecem junto delas e quais informações já existentes antes do exame poderiam ajudar a antecipá-las?

> **Observação:** Originalmente, a proposta para o trabalho era encontrar todas as bases que seriam utilizadas e apenas depois fazer as visualizações. Entretanto, optamos por fazer, primeiro, algumas visualizações do dataset principal que queríamos como base, para, depois, procurarmos outras bases que complementassem os nossos dados com base no que precisássemos.

---

## 2. Etapa 1 — Definição do problema, construção da amostra e descrição das notas

### 2.0 Definição do problema

Tendo decidido a temática do trabalho, era necessário encontrar uma base de dados que serviria como pilar principal do projeto. Escolhemos utilizar os microdados do ENEM, obtidos no site do governo (`https://www.gov.br/inep/pt-br/acesso-a-informacao/dados-abertos/microdados/enem `).

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
| 15 | Shapiro–Wilk e gráfico Q–Q | Diagnóstico opcional, sem influenciar a medida adotada |
| 16 | Mapa coroplético da mediana por UF | Analisar distribuição geográfica das medidas visualmente |
| 17 | Evolução regional | Linhas por região, uma área ou todas, com eixos próprios |
| 18 | Ranking de UFs | Ordena pela mediana; empates recebem a mesma posição |
| 19 | Evolução dos estados dentro de uma região | Mostra também empates na maior mediana de cada ano |
| 20 | Mapa de calor região × área | A cor é a **posição** (ranking) dentro de cada área, para não misturar escalas |
| 21 | Percentual de notas acima de um limite | Limite escolhido pelo usuário; limite não é nota oficial de aprovação |
| 22 | Volume de participantes × mediana | Dispersão por UF (eixo logarítmico opcional); não demonstra causalidade |

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

### 2.6 Preparo para a próxima etapa (seção 23)

Prepara as bases estadual, regional e nacional, com uma linha por ano × território × área, fração amostral e medida principal. Exporta em CSV se `EXPORTAR_TABELAS = True`.

- O resultado do Shapiro **não entra** como atributo, pois é calculado a partir da própria distribuição que se quer prever.
- Alerta-se que UF × ano dá só **135 combinações**, e que as cinco áreas não são cinco observações territoriais independentes.
- A cobertura só existe em nível nacional por ano e área: como não se sabe a UF de quem não a informou, não é possível calcular cobertura por UF.

---

## 3. Etapa 2 — Desigualdades e condições dos territórios

### 3.1 Mudança de pergunta e de unidade (seções 24 e 31)

A etapa 2 passa a focar em **quais condições econômicas e escolares acompanham as diferenças** de desempenho. A finalidade declarada é apoiar o diagnóstico de necessidades educacionais, e **não** decidir sozinha a distribuição de recursos.

A unidade de análise é **UF × ano × área**, abrangindo 27 UFs, cinco regiões, cinco áreas e as edições de 2020 a 2024.

**Fontes integradas:**

| Fonte | O que traz | Chave |
|---|---|---|
| ENEM (Inep) | Notas, presença, UF, rede | UF + ano + área |
| Censo Escolar (Inep) | Recursos de infraestrutura, matrículas, tempo integral | UF + ano (rede, no recorte estadual) |
| PNAD Contínua (IBGE, tabela 7531) | Renda domiciliar por pessoa | UF + ano |
| Indicadores educacionais (Inep) | Formação docente, horas-aula, rendimento | UF + ano (+ rede quando disponível) |

ENEM, Censo Escolar e PNAD formam o núcleo da integração; os indicadores educacionais entram como fontes complementares. O FUNDEB, que chegou a fazer parte da proposta, foi retirado (ver 3.7).

**Escolhas importantes:**

- **A junção é territorial (UF), não individual.** A expectativa inicial era ligar ENEM e Censo Escolar para avaliar a infraestrutura da escola de cada aluno. Entretanto, como não há chave de escola comparável em todos os outros arquivos usados, o Censo agora representa a **oferta de ensino médio da UF**, e não a escola frequentada por cada participante. Não se tenta localizar pessoas por nome, nota ou posição de linha.
- **Renda deflacionada pelo IPCA** para preços de 2024, mantendo também a versão original para conferência. A variável 10824 da PNAD informa o rendimento médio mensal real domiciliar por pessoa, a preços médios do próprio ano. Ressalva: o IPCA é um índice nacional, não o custo de vida de cada UF.
- **Resumir o Censo por UF e ano antes de juntar:** o Censo tem uma linha por escola; juntar direto multiplicaria linhas do ENEM.
- **Junção `many_to_one` com `validate` e `indicator`:** as cinco áreas de uma UF/ano recebem o mesmo contexto, sem criar participantes novos.

### 3.2 Diagnósticos sobre o próprio ENEM (seções 25–30)

A seção 25 prepara os filtros e as regras de agregação dos painéis seguintes: nas comparações nacionais e regionais agregam-se as notas ou os registros originais (**não se calcula a média das medianas dos estados**). "Públicas" reúne federal, estadual e municipal; rede não informada fica identificada e não é atribuída à rede pública.

| Seção | Pergunta | Cuidado metodológico |
|---|---|---|
| 26 | A parcela com UF conhecida mudou? E a média de quem tem UF difere da de quem não tem? | Comparação nacional (não se atribui UF a quem não informa); registra a mudança de documentação do Inep em 2024 sem atribuir a ela, sozinha, a queda |
| 27 | A mediana representa bem a distribuição? Há dois picos? | Curvas suaves com três larguras de suavização; se o pico some ao mudar a suavização, não é chamado de grupo. Notas zero ficam nas medidas, mas saem das curvas |
| 28 | Como as notas evoluíram? | Variação do ano anterior; variação de 2020 fica ausente (não há 2019), o que **não** equivale a zero |
| 29 | A queda se repete nas cinco áreas? | Mapa de calor de variação por área; a hipótese "tudo caiu em todo lugar" deve ser conferida célula a célula |
| 30 | A comparação muda ao separar rede e localização? | Curvas só para grupos com pelo menos 30 notas (precaução de leitura, não garantia de precisão) |

### 3.3 Oferta escolar (seções 32–34)

- **Seção 32 — carências:** percentual de escolas de ensino médio com cada recurso (água, energia, esgoto, internet, laboratório, biblioteca etc.). Valores ausentes **não viram "não"**: o denominador é o número de respostas válidas de cada item. A cor representa disponibilidade, não qualidade.
- **Seção 33 — evolução da oferta:** acompanha um recurso ou o tempo integral entre 2020 e 2024. O tempo integral usa **matrículas** como peso; os demais recursos usam escolas com resposta válida. O texto evita atribuir todo aumento ao programa federal criado em 2023.
- **Seção 34 — recorte coerente da rede estadual:** cria uma base em que notas, infraestrutura e indicadores se referem à **rede estadual**, aproximando o diagnóstico da atuação de uma secretaria. A renda segue sendo da UF inteira, com essa diferença explícita. Não se junta essa base à geral como se fossem observações independentes, pois compartilham participantes.

### 3.4 Associações (seções 35–36)

- **Correlação de Spearman** entre indicadores e mediana, uma característica por vez, com cada ponto sendo uma UF no mesmo ano e área.
  - **Por quê Spearman:** só considera a ordem dos estados, é robusta a valores extremos e não pressupõe relação linear.
- Não se calculam dezenas de p-valores para "provar fatores"; os coeficientes são **exploratórios**. Grupos com menos de 4 UFs ou sem variação ficam sem coeficiente.
- O gráfico à direita mostra se a associação **persiste entre os anos**; uma relação que aparece em um só ano merece cautela.
- A seção 34 reúne tudo em mapa de calor (todas as áreas, todos os anos), registrando o número de UFs em cada cálculo. A cor é associação, **não** importância de variável nem efeito de política.
- A seção 35 compara **mudanças** (ano final menos inicial) em vez de níveis, para confrontar a associação de níveis, que pode refletir diferenças históricas entre estados.
- Lembrando que, com apenas cinco edições, não é possível isolar um efeito atrasado da pandemia por simples correlação.

### 3.5 Composição dos participantes (seção 38)

Testa uma explicação verificável antes de atribuir a queda a um evento: *parte da variação vem de mudanças na proporção de redes e localizações?*

- Recalcula a média do ano final **mantendo os pesos do ano inicial** entre grupos UF × rede × localização.
- É uma decomposição descritiva da **média**, não da mediana, e não separa renda, seleção, pandemia, currículo ou políticas.

### 3.6 Perfis de desempenho (seção 39)

Pergunta se a mesma pessoa ocupa posições diferentes entre as áreas.

- **Exige amostra nova:** a amostra por área da etapa 1 não serve, porque os sorteios por área foram independentes. Usa-se uma amostra **conjunta de 10%** por UF e ano, entre quem tem as quatro provas objetivas válidas.
- Transforma notas em **posição percentual** dentro de área e edição, e compara a média das posições em MT/CN contra LC/CH. A diferença de ≥ 15 p.p. é uma regra descritiva ajustável, **não** um diagnóstico de aptidão.
- A renda familiar individual só existe até 2023 no material processado; em 2024 não se tenta ligar arquivos por posição de linha.

### 3.7 Uso da base FUNDEB (seção 40)

- **Variável disponível pode não ser adequada à pergunta (seção 40):** aumentar a quantidade de fontes não garante uma base melhor. O **FUNDEB** disponível descreve apenas dezembro, não o investimento anual no ensino médio estadual, e dividir o valor pelas matrículas não corrige a incompatibilidade. Por isso foi retirado dos gráficos de associação, da junção principal e das entradas de previsão. Uma ampliação futura precisaria de gasto executado anual, rede e etapa compatíveis, correção monetária e data de divulgação conhecida.

### 3.7 Contexto histórico (seção 41)

- **Contexto histórico em tabela:** interrupção das aulas (2020–2021), Programa Escola em Tempo Integral (2023), apoio de transporte no Ceará (2024) e a TRI do ENEM. Esses eventos são **contexto**, não variáveis de tratamento.
- **Conclusões deliberadas:** os dados **não** permitem atribuir a queda de 2024 a um "lag da pandemia" (sem acompanhamento individual nem grupo contrafactual) e **não** há variável comparável para cursinhos preparatórios (rede privada não é usada como substituto).
- **Diagnóstico de acertos (2023 e 2024):** relaciona número de acertos e nota média, para ilustrar que a TRI não dá notas iguais a quantidades iguais de acertos. Está limitado aos anos já conferidos nessa preparação.

### 3.8 Uso para gestão (seção 40)

Gráfico da rede estadual com renda (eixo x), mediana (eixo y), cor por região e tamanho do ponto proporcional à carência do recurso escolhido, sendo que as linhas tracejadas são medianas entre UFs. O texto insiste: **não é um ranking de prioridade** e nenhuma situação isolada demonstra qual intervenção funcionará. Antes de qualquer decisão, seria preciso conferir cobertura, desigualdades internas, demanda local e outras avaliações, como o Saeb.

---

## 4. Preparação para a etapa 3 (seções 43–46)

### 4.1 Escolha de atributos (seção 43)

Parte da pergunta, não da quantidade de colunas. **Candidatos:** renda real por pessoa, matrículas em tempo integral, formação docente adequada, abandono, internet e laboratório de ciências, além da mediana do ano anterior como referência indispensável.

- Gráfico de **redundância** (Spearman entre candidatos), usando uma linha por UF e ano, para não repetir cada UF cinco vezes por causa das áreas. Correlação alta é sinal para avaliar alternativas, não critério automático de exclusão.
- Aprovação, reprovação e abandono são relacionados entre si, então não entram todos automaticamente.
- Renda nominal e deflacionada não entram juntas; FUNDEB de dezembro fica fora; perfis derivados das notas do ano-alvo **não** podem prever essas mesmas notas.

### 4.2 O que prever e o tamanho do problema (seção 44)

**Pergunta de continuidade:** com informações anteriores, é possível estimar a mediana do ENEM dos participantes da rede estadual em cada UF e área no ano seguinte? A utilidade seria sinalizar territórios para investigação de necessidades, não distribuir recursos automaticamente nem estimar o efeito de uma política.

- **Por que manter as 27 UFs:** são comparáveis nas três fontes centrais durante 2020–2024. Selecionar só São Paulo deixaria cinco registros anuais por área. Passar para municípios ou escolas exigiria outra população-alvo, outras chaves e auditoria de cobertura. O código de escola voltou ao ENEM público em 2024, mas a série escolar anterior não é reconstruída por notas ou posição de linha.
- **O gargalo é o tempo observado, não o volume de participantes:** são 135 registros por área (27 UFs × 5 anos), ou seja, cinco problemas separados. Os milhões de notas tornam os resumos mais precisos, mas não criam milhões de exemplos territoriais para previsão.
- **Ajustes simples de diagnóstico:** o notebook anuncia ajustes simples para testar a viabilidade da base, que não constituem o modelo final. Até a seção 46 ainda não há treino de modelo (apenas a importação das ferramentas, como regressão Ridge e padronização). Toda comparação de previsão usará anos posteriores ao treino, e, como 2020–2024 já foram explorados, **nenhum desses anos será chamado de teste cego**.
- **Edição de 2025:** a página oficial do ENEM já a listava na consulta de 27/10/2026. Ela não foi incorporada nem avaliada; poderá servir de validação adicional após congelar o protocolo, se o grupo não a tiver explorado, e ainda será preciso verificar sua comparabilidade.
- A construção de uma base com **atributos de anos anteriores** ao resultado (para evitar usar informação que não estaria disponível antes do exame) é um princípio declarado, mas essa tabela ainda não é gerada no notebook.

### 4.3 Formas de associação (seção 46)

O Spearman sozinho não responde a toda a pergunta. A seção compara três medidas, para os seis atributos candidatos, em cada ano e área da rede estadual:

- **Pearson:** alinhamento em uma reta;
- **Spearman:** concordância de ordenações, aceitando relações crescentes ou decrescentes curvas;
- **correlação de distâncias:** pode detectar outras dependências, inclusive formas em U. Não tem sinal, e seu valor amostral positivo não comprova associação.

Cuidados e critérios:

- **Influência de cada UF:** repete-se o Spearman retirando uma UF por vez; o intervalo mínimo–máximo mede influência, **não** é intervalo de confiança. Nenhuma UF é excluída por alterar o resultado.
- **Permutações dentro das regiões:** para Pearson e distância usam-se 999 permutações do resultado **dentro de cada região**, preservando a composição regional. A referência pergunta se há mais associação do que a obtida ao reorganizar as UFs dentro das regiões; ela pressupõe que essas trocas sejam razoáveis sob a hipótese nula, e dependência espacial e diferenças internas limitam a interpretação.
- **Múltiplos testes:** aplica-se a correção Benjamini–Yekutieli sobre os **300 testes** (6 atributos × 5 áreas × 5 anos × 2 medidas). Ela reduz falsas descobertas sob dependência entre testes, mas não corrige um teste cuja hipótese de permutação seja inadequada.
- Um teste de distância significativo **não prova** que a relação seja especificamente não linear, pois relações lineares também são dependentes. A utilidade de permitir curvas será avaliada adiante, na previsão de anos posteriores.
- O painel mostra os pontos com uma reta e uma curva descritivas (que descrevem os pontos atuais, não demonstram capacidade de previsão), as três medidas lado a lado, o intervalo de influência e os p-valores ajustados. Os resultados são salvos em `associacoes_lineares_nao_lineares.csv`.

---

<!-- ## 6. Relação com o relatório contínuo

| Data | Orientação ou tarefa do relatório | Onde aparece no notebook |
|---|---|---|
| 06/08 | Tema e bases candidatas (ENEM, Censo Demográfico, ENEM por escola, Censo Escolar) | Etapas 1 e 2 usam microdados do ENEM e Censo Escolar, mais PNAD/IBGE e indicadores do Inep |
| 13/08 | Verificar se SP concentra participantes | Seção 10 (concentração por UF e região) |
| 13/08 | Mapear notas nos estados, sem taxas por falta de população de referência | Seções 16, 18 e 22; o texto reforça que mais participantes ≠ maior taxa |
| 13/08 | Análise temporal de ao menos cinco anos | Período fixo 2020–2024 em todo o notebook |
| 13/08 | Aleatoriedade computacional suficiente para a amostra | Amostra de 10% sem reposição com semente fixa |
| 13/08 | Avaliar o tipo de gráfico mais adequado e lidar com grande volume | Histogramas, barras, linhas, mapas e mapas de calor; leitura em blocos e amostragem |
| 20/08 | Mapa coroplético sem média; examinar a distribuição | Mediana como medida única (seção 11), histogramas e diagnóstico de normalidade (seções 14–15 e 27) |
| 20/08 | Segregação regional e estados extremos | Região por UF; rankings de maiores e menores (seção 18) |
| 20/08 | Análise de valores nulos | Cobertura da UF escolar (seções 9 e 26), auditoria de ausentes (seção 45) |
| 27/08 | Padronizar uma amostra para todos os gráficos | Amostra única estratificada, com validação |
| 03/09 | Ligar microdados do ENEM ao Censo Escolar para avaliar infraestrutura | Seções 24–35, com junção **territorial** por UF e ano (limitação explicada na seção 4.1 deste documento) |

**Pontos do relatório que não aparecem como análises separadas no notebook:** os boxplots por área e as barras empilhadas (tarefas individuais de 13/08 e 20/08) e a modelagem preditiva com matriz de importância (27/08). O notebook mantém como equivalentes os histogramas, os quartis/percentis e a preparação da base para a etapa de modelagem; a modelagem em si foi deliberadamente deixada para a etapa 3.

--- -->

## 5. Limitações gerais registradas

1. O ENEM retrata **seus participantes**, não todos os estudantes; há seleção de quem faz a prova.
2. A cobertura da UF escolar varia entre anos e a ausência não é imputada.
3. A amostra descreve elegíveis **com UF da escola informada**.
4. A série compara **grupos diferentes** de pessoas ao longo dos anos.
5. Associações entre UFs não são efeitos individuais nem causais, e não há ajuste por outras variáveis.
6. Com 27 UFs e 5 anos, o volume de observações territoriais independentes é pequeno.
7. Mudanças de documentação, de cobertura e de composição entre edições dificultam atribuir quedas a um único evento.
8. Presença de um recurso (internet, laboratório) não mede qualidade nem uso efetivo.

## 6. Observações técnicas sobre o notebook

- A **Etapa 2 depende de uma pasta pré-processada** (`estudo_enem/dados/`, com arquivos `.csv` e `.parquet`) que não é gerada pelas células do notebook; ela precisa estar no Drive ou na pasta de trabalho. Caso contrário, a célula de carga levanta `FileNotFoundError`.
- O mapa depende de `malha_ufs_plotly.geojson`; sem ele, só o painel do mapa fica indisponível.
