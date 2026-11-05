# Histórico do projeto ENEM 2020 a 2024

**Registro do desenvolvimento das Etapas 1 e 2**

Este documento reúne as decisões, revisões metodológicas, análises exploratórias e integrações de bases realizadas no projeto, cobrindo o ENEM de 2020 a 2024.

> **Estado atual:** Etapa 2 — análises exploratórias e integração de bases. O treinamento e a avaliação de modelos ainda não foram iniciados.

## Navegação

- [Registro do desenvolvimento das Etapas 1 e 2](#registro-do-desenvolvimento-das-etapas-1-e-2)
- [Desenvolvimento da Etapa 2](#desenvolvimento-da-etapa-2)
- [Integração das bases na Etapa 2](#integração-das-bases-na-etapa-2)
- [Onde estamos atualmente](#onde-estamos-atualmente)

---

## Registro do desenvolvimento das Etapas 1 e 2

### Como surgiu a ideia

Nosso trabalho começou com uma pergunta: quais características socioeconômicas e educacionais estão relacionadas ao desempenho dos participantes do ENEM?

Queríamos investigar por que existem diferenças nas notas entre estados e regiões do Brasil e se informações como renda, infraestrutura escolar, formação dos professores e localização das escolas poderiam ajudar a entender essas diferenças.

No início, tínhamos várias ideias, mas ainda não sabíamos quais seriam possíveis de investigar com os dados disponíveis. Por isso, decidimos começar pelos microdados do ENEM, conhecer melhor as notas e observar os padrões antes de procurar explicações.

O projeto foi dividido em etapas. Na primeira, trabalhamos com a preparação dos dados, amostragem e análises descritivas. Na segunda, ampliamos as análises, estudamos melhor quem estava representado nos dados e integramos informações de outras fontes.

Ao longo do trabalho, algumas escolhas foram modificadas. Também surgiram perguntas que não estavam previstas inicialmente, principalmente depois das discussões com o professor e da observação dos primeiros resultados.

## 6 de agosto. Primeiras discussões

Na primeira reunião, pensamos em utilizar os dados do ENEM para investigar diferenças de desempenho entre regiões brasileiras.

Levantamos algumas hipóteses sobre possíveis relações entre as notas e a renda dos participantes, as condições socioeconômicas, a infraestrutura das escolas, a formação dos professores, o porte das instituições e a localização urbana ou rural.

Uma questão que também nos interessava era identificar resultados que fugissem do esperado. Por exemplo, seria interessante encontrar escolas em regiões de menor nível socioeconômico que apresentassem notas elevadas e investigar o que poderia diferenciá-las das demais.

O professor sugeriu que começássemos pela estratificação e pela observação dos padrões presentes nos dados. Também levantou questões sobre quem participa do ENEM e quais são os objetivos dos inscritos, como o ingresso no ensino superior ou a busca por outras formas de formação.

Discutimos ainda a possibilidade de encontrar agrupamentos de participantes ou regiões com características semelhantes.

Como fontes de dados, pensamos inicialmente nos microdados do ENEM, Censo Escolar, Censo Demográfico, ENEM por Escola e indicadores de desenvolvimento humano.

Naquele momento, essas bases eram possibilidades. Ainda não tínhamos realizado nenhuma integração.

## 13 de agosto. Definição do período e das primeiras análises

Na semana seguinte, começamos a definir como investigar o problema.

O professor orientou que observássemos primeiro onde estavam as maiores e menores notas, quais regiões concentravam mais participantes e se essas diferenças se repetiam ao longo dos anos.

Também discutimos o caso de São Paulo. Queríamos verificar se o estado concentrava mais participantes e se isso estava relacionado apenas ao tamanho da população.

Percebemos que olhar somente para números absolutos não seria suficiente para responder a essa pergunta, pois seria necessário considerar também alguma população de referência.

Chegamos a discutir a possibilidade de trabalhar apenas com São Paulo, mas decidimos manter uma análise nacional para comparar as diferentes regiões.

Definimos o período de 2020 a 2024, totalizando cinco edições do ENEM, com as 27 unidades federativas e as cinco áreas de conhecimento: Linguagens, Ciências Humanas, Ciências da Natureza, Matemática e Redação.

Também começamos a pensar em gráficos com filtros por ano, área e estado.

Na divisão inicial das tarefas, André ficou responsável pela amostragem e pelos boxplots, Camila pelos quartis e Vinicius pelos agrupamentos das notas. A tarefa de Guilherme ainda não estava definida naquele registro.

Nossa primeira preocupação era entender como as notas estavam distribuídas antes de investigar possíveis relações com outras características.

## 20 de agosto. Preparação dos microdados e amostragem

Os microdados do ENEM possuem milhões de registros. Como trabalharíamos com cinco anos, precisávamos organizar os arquivos e reduzir o volume utilizado nas análises.

Selecionamos inicialmente as colunas necessárias para identificar as notas, a presença nas provas, a situação da Redação, o ano e a UF.

Também organizamos a leitura dos arquivos em blocos de 250 mil linhas para facilitar o processamento.

### Critérios de seleção

Decidimos trabalhar com notas válidas dos participantes que compareceram à respectiva prova.

Para a Redação, também consideramos a situação de correção.

Essa escolha permitiu aproveitar as informações disponíveis em cada área, mesmo quando o participante não tinha comparecido a todas as provas.

Por exemplo, uma pessoa poderia possuir uma nota válida em Linguagens e não possuir uma nota de Matemática.

Por isso, a seleção foi feita separadamente por área.

### Construção da amostra

Adotamos uma amostragem estratificada para reduzir o volume de dados.

Na versão consolidada, os grupos utilizados no sorteio foram definidos por ano, UF da escola e área.

Dentro de cada combinação, selecionamos aproximadamente 10% dos registros elegíveis, sem reposição e com uma semente fixa para permitir a reprodução do procedimento.

No total, obtivemos 675 estratos e 1.910.983 registros de notas amostradas.

É importante destacar que essa quantidade corresponde a registros de notas, não necessariamente a participantes diferentes, pois uma mesma pessoa pode contribuir com notas em várias áreas.

Também guardamos as quantidades completas de elegíveis antes do sorteio, permitindo comparar a amostra com o universo de referência.

A escolha dos 10% foi feita principalmente para facilitar o processamento. Posteriormente, passamos a verificar se o sorteio preservava as características observáveis dos grupos.

## 27 de agosto. Análise das distribuições

Depois de construir a amostra, começamos a analisar as notas.

Inicialmente, utilizávamos bastante a média para representar os resultados de cada estado, mas percebemos que um único valor poderia esconder características importantes da distribuição.

Dois estados podem ter medianas semelhantes e, mesmo assim, apresentar diferenças na concentração das notas, nos valores extremos e na dispersão.

Por isso, incluímos histogramas, boxplots, quartis, percentis, desvio-padrão e assimetria.

O professor também questionou o comportamento das distribuições, principalmente quando os histogramas apresentavam mais de um pico.

### Escolha entre média e mediana

Nas primeiras versões do trabalho, utilizamos o teste de Shapiro-Wilk como um critério automático para escolher entre média e mediana.

Depois, revisamos esse procedimento.

O teste ajuda a avaliar a normalidade dos dados, mas não determina sozinho qual medida deve ser utilizada.

Decidimos adotar a mediana como medida principal para comparar os grupos, mantendo a média e outras estatísticas para complementar a análise.

A mediana foi escolhida por ser menos sensível a valores extremos. Entretanto, ela não descreve sozinha toda a distribuição.

Por isso, mantivemos os gráficos de distribuição e os diagnósticos de normalidade como análises complementares.

## 3 de setembro. Revisão da UF utilizada

Outra mudança importante aconteceu quando revisamos a informação territorial utilizada no projeto.

Nas primeiras versões, agrupávamos os participantes pela UF de realização da prova, utilizando `SG_UF_PROVA`.

Como nosso objetivo era investigar características educacionais e escolares, percebemos que seria mais adequado utilizar a UF da escola, representada por `SG_UF_ESC`.

Uma pessoa pode estudar em determinado estado e fazer o exame em outro. Por isso, a UF de realização da prova não necessariamente representa seu contexto escolar.

Refizemos a seleção territorial utilizando a UF da escola.

### Participantes sem UF escolar

Essa decisão trouxe uma limitação importante.

Nem todos os participantes possuem a UF da escola informada nos microdados.

Os registros sem essa informação não poderiam ser distribuídos corretamente entre os estados em uma análise territorial escolar.

Por isso, passaram a ficar fora desse recorte, embora continuassem sendo considerados nas análises nacionais de cobertura.

Também começamos a contabilizar quantos registros possuíam UF conhecida e quantos não possuíam.

Isso foi importante para deixar claro que nossas comparações por estado não representavam necessariamente todos os participantes do ENEM.

### Consolidação da Etapa 1

Organizamos os gráficos de distribuição, mapas das medianas, evolução regional, rankings e comparações entre áreas.

Também calculamos resumos por UF, região e Brasil.

Para obter a mediana regional, utilizamos as notas dos participantes das UFs pertencentes àquela região, em vez de calcular simplesmente a mediana das medianas estaduais.

Ao final dessa etapa, já conseguíamos observar onde estavam as maiores e menores medianas, como as notas se distribuíam e como variavam entre regiões e edições.

Entretanto, ainda não havíamos integrado as bases externas previstas no trabalho.

---

# Desenvolvimento da Etapa 2

## Primeiras revisões. Quem realmente estamos analisando?

Na Etapa 2, nosso foco passou a ser a interpretação visual, a integração das informações e a representatividade dos dados.

Antes de acrescentar novas bases, voltamos a algumas decisões da Etapa 1.

Uma das principais preocupações era verificar se os participantes com UF escolar conhecida possuíam características diferentes daqueles sem essa informação.

Para isso, utilizamos resumos completos das notas elegíveis e comparamos a cobertura, as médias e a composição dos grupos.

Em Matemática, observamos que a cobertura da UF escolar passou de 20,8% em 2020 para 39,9% em 2024.

Também encontramos diferenças nas médias entre participantes com e sem UF escolar.

Em 2020, o grupo com UF conhecida apresentava média aproximadamente 25,1 pontos maior. Em 2024, a diferença passou a ser de aproximadamente 10,8 pontos a menos.

Esse resultado mostrou que os grupos não deveriam ser tratados como equivalentes.

Além disso, a proporção de participantes com UF escolar informada mudou bastante entre as edições.

Assim, percebemos que precisávamos considerar a cobertura e a composição dos participantes antes de interpretar diferenças nas notas como mudanças gerais no desempenho do ENEM.

## Situação escolar dos participantes

Outra análise acrescentada foi a situação escolar declarada no momento da inscrição.

Passamos a distinguir participantes que já haviam concluído o Ensino Médio, aqueles que declaravam concluir naquele ano, os que concluiriam posteriormente e aqueles que estavam fora do Ensino Médio sem tê-lo concluído.

Isso nos ajudou a compreender melhor quem participava do exame.

No conjunto nacional de inscritos, a proporção que declarou concluir o Ensino Médio no ano passou de 24,1% em 2020 para 37,3% em 2024.

Também analisamos a situação escolar conforme a presença ou ausência da UF da escola.

Em Matemática, no ano de 2023, todos os registros elegíveis com UF escolar conhecida pertenciam a participantes que declaravam concluir o Ensino Médio naquele ano. Entre os registros sem UF, essa proporção foi de 16,8%.

Esse resultado ajudou a explicar por que a população utilizada nas análises territoriais possuía um perfil específico.

Também encontramos uma limitação nos dados de 2024. Nessa edição, as informações de inscrição e de resultados foram disponibilizadas em arquivos separados, sem uma chave pública comum que permitisse relacionar diretamente situação escolar e nota.

Por isso, mantivemos as análises nacionais de inscritos em 2024, mas não realizamos o mesmo cruzamento individual entre situação escolar e desempenho para essa edição.

## Verificação da amostragem

Também voltamos à amostra da Etapa 1 para verificar se o sorteio preservava características dos participantes.

Reexecutamos a seleção a partir dos arquivos oficiais e conferimos as contagens, os estratos e os registros selecionados.

Além disso, comparamos a composição por rede de ensino e localização urbana ou rural entre a amostra e o conjunto completo de elegíveis.

Essas características não definiam os estratos do sorteio inicial, por isso era importante verificar como ficaram representadas.

A diferença absoluta mediana nas proporções de rede e localização ficou entre 0,14 e 0,19 ponto percentual, dependendo da edição.

Entretanto, encontramos diferenças maiores em alguns grupos pequenos, chegando a 6,40 pontos percentuais.

Isso mostrou que a amostra preservava relativamente bem a composição observável na maior parte dos recortes, mas que grupos menores exigiam cuidado.

Mantivemos as contagens e os denominadores nos gráficos para ajudar a interpretar os resultados.

## Aprofundamento das distribuições

Também ampliamos a análise dos histogramas.

Uma das perguntas era se determinadas distribuições apresentavam dois picos e se a mediana escondia grupos diferentes de participantes.

Passamos a comparar os histogramas com curvas suavizadas, alterando a largura das faixas e observando como os picos se comportavam.

Percebemos que a quantidade de picos identificados pode mudar conforme a suavização utilizada.

Por isso, não tratamos cada pico como um grupo real de estudantes.

Também acrescentamos gráficos de distribuição acumulada e comparações de percentis.

Essas visualizações permitiram comparar não apenas a posição central das notas, mas também as regiões inferiores e superiores das distribuições.

Um exemplo foi a comparação da Redação de 2023 entre Distrito Federal e Goiás.

Na amostra analisada, ambos apresentavam mediana de 660 pontos. Entretanto, o percentil 10 era de 480 no Distrito Federal e 400 em Goiás. Já o percentil 90 era de 900 no Distrito Federal e 920 em Goiás.

Isso mostrou que duas distribuições podem apresentar a mesma mediana e, mesmo assim, ter comportamentos diferentes nas extremidades.

Passamos também a utilizar os percentis 20 e 80 como referências para estudar a participação de notas nas partes inferior e superior das distribuições.

Esses limites foram utilizados para comparar os grupos, sem tratá-los como critérios oficiais de desempenho.

## Comparações entre estados, regiões e edições

Depois de aprofundar as distribuições, voltamos às diferenças territoriais e temporais.

Construímos visualizações para comparar medianas, quartis e percentis entre anos consecutivos.

Também utilizamos mapas de calor e gráficos com filtros por estado, região, área e período.

Uma das questões que mais chamou atenção foi a redução das notas em 2024.

Na comparação entre 2023 e 2024, encontramos queda na mediana em 19 das 25 combinações de região e área.

A maior redução observada nesse levantamento foi em Matemática no Sudeste, com aproximadamente 55,2 pontos.

Entretanto, a queda não teve a mesma intensidade em todas as regiões e áreas.

Também verificamos que ela não aparecia apenas na amostra de 10%, pois os resumos completos dos elegíveis apresentavam mudanças semelhantes.

A partir disso, começamos a investigar se parte dessas diferenças poderia estar associada à mudança na composição dos participantes.

## Comparações por rede e localização

Acrescentamos filtros para comparar resultados de diferentes redes de ensino e localizações urbanas ou rurais.

Queríamos verificar se as diferenças gerais entre estados permaneciam quando analisávamos grupos escolares mais próximos.

Uma das comparações envolveu o Ceará e estados do Sudeste, conforme uma questão levantada pelo professor.

Observamos que os resultados podiam mudar dependendo da área, da rede e da localização selecionadas.

Em Matemática de 2024, por exemplo, a amostra da rede estadual do Ceará apresentou mediana de 468,6 pontos entre participantes de localização urbana e 450,4 entre os de localização rural.

Entretanto, essas diferenças não demonstram que a localização seja a causa do desempenho, pois os grupos podem apresentar outras características diferentes.

Também discutimos a possibilidade de investigar a influência de cursinhos e escolas preparatórias.

Como os dados utilizados não identificavam essa informação de maneira comparável, não utilizamos a frequência em cursinhos como explicação para os resultados.

## Mudanças na composição dos participantes

Para investigar melhor as alterações nas notas entre edições, realizamos uma decomposição descritiva da média.

A ideia era verificar como a mudança na participação relativa dos grupos de UF, rede de ensino e localização se relacionava com a mudança observada nas notas.

Na comparação de Matemática entre 2023 e 2024, considerando os grupos comparáveis com UF escolar conhecida, encontramos uma redução média de aproximadamente 26,5 pontos.

Quando mantivemos a composição de 2023, a redução calculada foi de aproximadamente 12,6 pontos.

A diferença de 13,9 pontos correspondeu à parcela associada à alteração dos pesos desses grupos na decomposição.

Esse procedimento ajudou a perceber que uma parte da mudança observada estava relacionada à composição da população analisada.

Entretanto, a análise é descritiva. Não conseguimos afirmar o que provocou essas mudanças nem separar todos os fatores que poderiam estar envolvidos.

## Perfis de desempenho nas diferentes áreas

Outra investigação que desenvolvemos foi comparar o desempenho da mesma pessoa nas quatro provas objetivas.

Queríamos saber se participantes que apresentavam boas posições relativas em Matemática e Ciências da Natureza também apresentavam resultados semelhantes em Linguagens e Ciências Humanas.

Para isso, construímos uma amostra conjunta, diferente dos sorteios independentes realizados na Etapa 1.

Essa amostra reuniu 374.809 inscrições com quatro notas objetivas válidas e UF escolar conhecida, distribuídas entre as cinco edições.

Primeiro, calculamos a posição percentual dos participantes em cada prova, considerando a distribuição nacional de referência da respectiva edição.

Depois, comparamos a média das posições em Matemática e Ciências da Natureza com a média das posições em Linguagens e Ciências Humanas.

Inicialmente, a análise separava os participantes entre aqueles com maior posição relativa em Exatas, aqueles com maior posição em Linguagens e Humanas e aqueles com posições próximas nos dois conjuntos.

Entretanto, percebemos uma limitação nessa classificação.

Uma pessoa poderia ter posições próximas porque apresentava desempenho relativamente alto nos dois conjuntos ou porque apresentava desempenho baixo em ambos.

Por isso, acrescentamos uma análise complementar com quatro perfis:

1. Alto nos dois conjuntos de áreas.
2. Baixo nos dois conjuntos.
3. Alto em Matemática e Natureza e baixo em Linguagens e Humanas.
4. Alto em Linguagens e Humanas e baixo em Matemática e Natureza.

Também criamos uma classificação alternativa com uma faixa intermediária, utilizando limites de 40 e 60 pontos percentuais para verificar se as proporções mudavam conforme o critério adotado.

Na amostra conjunta da rede estadual de 2023, com 45.867 participantes e utilizando o corte 50, encontramos:

- 24,6% com posição relativamente alta nos dois conjuntos.
- 54,6% com posição relativamente baixa nos dois.
- 9,0% com posição alta em Matemática e Natureza.
- 11,8% com posição alta em Linguagens e Humanas.

Também investigamos como esses perfis estavam distribuídos entre UFs e regiões.

Entre as UFs com pelo menos 30 participantes na comparação, a proporção classificada como alta nos dois conjuntos variou de 12,4% no Piauí a 35,5% no Rio Grande do Sul.

Para 2023, também conseguimos comparar os perfis por faixas de renda familiar declarada.

Na amostra estadual analisada, a proporção de participantes classificados como altos nos dois conjuntos foi de 16,9% na faixa de até 1,5 salário mínimo, 33,8% na faixa acima de 1,5 até 4 salários mínimos e 45,6% na faixa superior a 4 salários mínimos.

Esses resultados mostram diferenças entre os grupos observados, mas não permitem concluir que a renda seja a causa dessas diferenças.

Também é importante lembrar que as classificações utilizam posições relativas dentro da distribuição de referência, e não limites oficiais de proficiência.

## Investigação da TRI

Outra pergunta levantada durante o desenvolvimento foi se mudanças nas notas poderiam estar relacionadas à Teoria de Resposta ao Item.

Para investigar essa questão, preparamos uma comparação entre a quantidade de acertos e as notas das provas objetivas de 2023 e 2024.

O objetivo era observar como as notas variavam para participantes com números semelhantes de acertos.

Essa análise foi realizada em nível nacional, incluindo registros com e sem UF escolar conhecida.

Observamos diferenças na relação entre acertos e notas, mas isso não foi suficiente para afirmar que houve mudança no método de correção ou que essa seria a causa da queda de 2024.

Mantivemos essa análise como um complemento para compreender melhor as notas do exame.

Também consideramos acontecimentos do período, como a pandemia e a ampliação das matrículas em tempo integral.

Entretanto, esses acontecimentos foram utilizados apenas como contexto. Não possuímos informações suficientes para medir seus efeitos individuais sobre os participantes.

---

# Integração das bases na Etapa 2

## Ampliação das fontes de informação

Depois das análises iniciais do ENEM, passamos a incorporar outras bases para investigar o contexto dos estados.

Na versão atual, trabalhamos com quatro fontes principais.

**Microdados do ENEM:** utilizados para calcular notas, medianas, médias, percentis, contagens e características dos participantes.

**Censo Escolar:** utilizado para obter informações sobre infraestrutura, redes de ensino, matrículas e oferta de tempo integral.

**PNAD Contínua:** utilizada para representar a renda domiciliar média por pessoa nos estados.

**Indicadores educacionais do Inep:** utilizados para acrescentar informações sobre formação docente, horas-aula e rendimento escolar, incluindo abandono.

Também utilizamos o IPCA para comparar valores de renda em preços de uma mesma referência.

A integração foi organizada para os anos de 2020 a 2024.

## Como fizemos a integração

Nossa unidade principal de análise territorial passou a ser a combinação de UF da escola, ano e área do conhecimento.

Assim, uma linha poderia representar, por exemplo, Matemática no Ceará em 2022.

As bases externas foram organizadas por UF e ano, permitindo acrescentar os indicadores territoriais aos resultados do ENEM.

Antes das junções, verificamos se existiam chaves duplicadas e se todas as combinações encontravam correspondência.

Também conferimos se as junções preservavam a quantidade de linhas e as contagens originais do ENEM.

Ao final, obtivemos uma base geral e uma base específica da rede estadual, cada uma com 675 linhas e 58 colunas.

As 675 linhas correspondem a 27 UFs, cinco anos e cinco áreas.

No recorte da rede estadual, utilizamos os resultados completos dos elegíveis dessa rede e os indicadores escolares correspondentes.

A renda permaneceu como uma característica da UF, e não especificamente dos estudantes da rede estadual.

Também fizemos verificações de valores ausentes e conferimos indicadores calculados a partir dos dados do Censo Escolar.

## Indicadores que não utilizamos

Durante a preparação, consideramos acrescentar informações do FUNDEB.

Entretanto, os dados que tínhamos disponíveis correspondiam apenas ao mês de dezembro.

Como queríamos investigar condições educacionais anuais, decidimos não utilizar esse valor como se representasse o investimento anual por estudante.

Mantivemos o registro da fonte e a justificativa para sua exclusão.

Essa decisão também mostrou que nem toda informação disponível necessariamente melhora a análise. Precisávamos considerar se o indicador era compatível com a pergunta, o período e a unidade territorial utilizada.

## Infraestrutura escolar

Com os dados integrados, passamos a explorar a disponibilidade de recursos nas escolas de Ensino Médio.

Construímos gráficos com filtros de ano, rede, localização e território, permitindo comparar a proporção de escolas com diferentes recursos.

Também analisamos a evolução desses indicadores ao longo do período.

Em 2024, por exemplo, no recorte estadual, a proporção de escolas com internet disponível aos alunos variou de 3,1% no Pará a 97,6% no Espírito Santo.

Para laboratórios de ciências, os valores variaram de 19,1% em Roraima a 88,2% no Ceará.

Já a proporção de matrículas em tempo integral variou de 4,0% em Roraima a 71,2% em Pernambuco.

Esses resultados mostram diferenças importantes entre os estados quanto às condições e à oferta escolar.

Entretanto, a existência de um recurso não significa necessariamente que ele seja utilizado ou que todos os estudantes tenham acesso a ele nas mesmas condições.

Também não temos, nessa integração territorial, uma ligação individual que identifique quais recursos estavam disponíveis na escola frequentada por cada participante.

Por isso, utilizamos esses indicadores para descrever os estados e investigar associações territoriais.

## Relação entre renda, indicadores escolares e notas

Depois de integrar as bases, começamos a comparar os indicadores de contexto com as medianas do ENEM.

Construímos gráficos de dispersão e correlação, permitindo selecionar ano, área e indicador.

Uma das relações observadas foi entre renda domiciliar por pessoa e mediana de Matemática na rede estadual.

Em 2024, encontramos uma correlação de Spearman de aproximadamente 0,75 entre as UFs.

Isso significa que, naquele recorte, estados com renda mais elevada tendiam a ocupar posições mais altas também nas medianas de Matemática.

Entretanto, esse resultado não demonstra que a renda tenha causado o desempenho observado.

Também investigamos se as associações permaneciam semelhantes em outras edições e áreas.

Além disso, passamos a comparar as mudanças anuais dentro das próprias UFs.

Nessa análise, a relação entre a variação da renda e a variação das notas foi bem menos consistente, com correlações entre aproximadamente -0,27 e +0,26 nas transições observadas.

Isso mostrou que comparar estados em um mesmo ano é diferente de comparar como esses estados mudaram ao longo do tempo.

Também analisamos outros indicadores, como tempo integral, formação docente, abandono e infraestrutura.

## Verificações das associações

Na parte final das análises, passamos a investigar melhor as relações encontradas.

Utilizamos diferentes medidas, incluindo as correlações de Pearson e Spearman e uma análise complementar baseada em distâncias.

Também verificamos se determinados resultados dependiam muito de uma única UF ou se mudavam ao considerar outras características dos estados.

Outro ponto foi a relação entre os próprios indicadores.

Por exemplo, tempo integral e horas-aula apresentaram correlação de Spearman de aproximadamente 0,82 no recorte examinado.

Isso mostrou que algumas variáveis podem carregar informações semelhantes.

Essas verificações foram importantes para compreender melhor as relações observadas antes de considerar qualquer utilização futura dos indicadores.

Ainda não avaliamos a importância dessas variáveis em um modelo preditivo.

---

# Onde estamos atualmente

Até o momento, concluímos uma série de análises descritivas e exploratórias e realizamos a integração de informações do ENEM com bases educacionais e socioeconômicas.

Também revisamos a amostragem e investigamos limitações relacionadas à cobertura da UF escolar.

Passamos a compreender melhor como a população observada muda entre as edições e por que essas mudanças devem ser consideradas nas comparações de notas.

Aprofundamos as distribuições, analisamos diferenças entre estados, regiões, redes e localização, investigamos a queda de 2024 e construímos perfis de participantes com base no desempenho relativo nas quatro provas objetivas.

A integração das bases permitiu acrescentar informações sobre renda, infraestrutura, formação docente, abandono e tempo integral.

Com esses indicadores, conseguimos estudar associações entre as características dos estados e os resultados do ENEM, verificando também se elas permaneciam semelhantes entre anos e áreas.

Uma das principais conclusões do nosso processo até aqui é que precisamos ter cuidado com aquilo que os dados realmente permitem afirmar.

Encontrar uma correlação não significa identificar uma causa. Da mesma forma, observar uma queda nas notas não permite atribuí-la automaticamente à pandemia, à TRI ou a alguma política educacional.

Também percebemos que existem diferenças entre investigar as características individuais dos participantes e investigar as condições médias dos estados.

Essas duas abordagens aparecem no nosso trabalho, mas não devem ser confundidas.

## Preparação para a próxima etapa

Na parte final do notebook, começamos apenas a organizar as informações que poderão ser utilizadas posteriormente.

A possibilidade considerada é investigar se indicadores socioeconômicos e escolares ajudam a prever a mediana do ENEM da rede estadual por UF e área.

Para isso, preparamos uma tabela preliminar com 540 combinações que relacionam anos consecutivos, incluindo a mediana e características observadas no ano anterior.

Também registramos quais informações ainda precisam ter suas datas de publicação verificadas antes de serem utilizadas em uma previsão.

Essa preparação faz parte do encerramento da Etapa 2.

**Ainda não iniciamos o treinamento ou a avaliação de modelos.** Portanto, não temos resultados de previsão, medidas de erro ou conclusões sobre quais atributos realmente melhoram o desempenho de um algoritmo.

Essas questões pertencem à próxima etapa do projeto.

Por enquanto, nosso trabalho está concentrado em compreender os dados, explicar as análises realizadas, verificar suas limitações e organizar as informações que poderão ser utilizadas posteriormente.

## Considerações sobre o desenvolvimento até aqui

Quando começamos o trabalho, nossa ideia era investigar como renda e condições escolares poderiam estar relacionadas às notas do ENEM.

A partir das orientações do professor, começamos pela preparação dos microdados e pelas distribuições das notas.

Conforme avançamos, percebemos que não bastava comparar médias ou medianas entre estados. Precisávamos entender melhor como as notas estavam distribuídas e quem estava sendo considerado em cada comparação.

A mudança para a UF da escola também trouxe uma questão importante sobre a cobertura dos dados. Ao investigar esse problema, encontramos diferenças entre os participantes com e sem essa informação, o que ajudou a compreender algumas limitações das análises territoriais.

Na Etapa 2, ampliamos as perguntas e incorporamos outras fontes de informação. Isso permitiu observar características socioeconômicas e educacionais que não estavam presentes nas primeiras análises.

Também conseguimos aprofundar algumas questões levantadas pelo professor, principalmente sobre a queda das notas em 2024, as diferenças entre redes de ensino, os formatos das distribuições e a relação entre desempenho nas diferentes áreas.

Nem todas as hipóteses puderam ser investigadas completamente. Algumas dependem de informações individuais que não estão disponíveis ou de dados mais específicos sobre escolas, políticas públicas e participação no exame.

Mesmo assim, conseguimos avançar na compreensão do problema e identificar quais resultados são sustentados pelos dados que temos.

**Nosso trabalho permanece na Etapa 2.** O objetivo agora é finalizar a interpretação das análises, organizar melhor as explicações e deixar claras as limitações e as informações que poderão ser aproveitadas na próxima etapa.

---

*Nota sobre as datas: os registros de 6 e 13 de agosto correspondem às primeiras anotações do grupo. Os marcos de 20 e 27 de agosto e 3 de setembro foram utilizados para organizar retrospectivamente a evolução da Etapa 1. Algumas decisões foram revisadas e consolidadas depois dessas datas.*
