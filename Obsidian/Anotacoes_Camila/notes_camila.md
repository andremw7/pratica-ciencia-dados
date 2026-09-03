# Anotações Camila

Neste arquivo, encontram-se todas as atividades realizadas pela Camila durante o período de desenvolvimento do projeto, assim como todas as escolhes e decisões tomadas por ela.

# 13/08

Foi feito um gráfico de barras empilhadas para visualizar onde no Brasil estão as notas altas, médias e baixas. Esse foi o gráfico escolhido, pois ele permite visualizar as proporções das notas entre os estados e dentro deles. 

A análise foi feita utilziando os dados de 2024, que são os mais recentes mais confiáveis.

# 20/08

### Criação de arquivos com as amostras

Temos uma quantidade muito grande de dados, e por isso, foi decidido em grupo que extrairíamos amostras de 10% de cada ano, estratificadas pelos estados (seguindo a proporção de dados dentre os estados). Foram feitas outras amostragens, mas essa servirá para a **padronização** no futuro.

- **Seleção de linhas**: Na nossa análise, focaremos apenas nos alunos que tenham registro de pelo menos uma das notas e tenham o UF registrado. Portanto, descartamos as linhas que tinham dados nulos em pelo menos uma das colunas `'NU_NOTA_CN'`, `'NU_NOTA_CH'`, `'NU_NOTA_LC'`, `'NU_NOTA_MT'` ou `'NU_NOTA_REDACAO'`; e todas as linhas em que `'SG_UF_ESC'` é nulo. (Obs.: não foi selecionado `'SG_UF_PROVA'`, pois é mais provável que o estudante more ou esteja mais próximo da escola onde concluiu o ensino médio, ao invés da escola  em que realizou a prova - que são menos selecioandas)

### Ajustes no gráfico de barras

O gráfico de barras empilhadas foi ajustado para que fosse possível selecionar o ano da visualização. 

### Análise de dados nulos

Foi feita uma análise de dados nulos, com possíveis correlações entre eles e onde eles estão localizados. Para essa análise, foi utilizado o arquivo com todos os dados filtrados.

**Conclusões**
- Há apenas 3 valores de correlação: 1, 0 e -1.
- As correlações 1 possivelmente indicam as variáveis em que, se uma é nula, as outras também serão.
- As correlações -1 seriam das variáveis em que se um é nulo, o outro também deve ser. Esse caso entra para as variáveis que existem nos datasets de 2020, 2021, 2022 e 2023 e não existem no de 2024 e vice-versa.

# 27/08

### Análise dos anos de conclusão

A ideia era fazer um gráfico de barras com os anos de conclusão do ensino médio para cada ano de aplicação do ENEM analisado. Entretanto, como extraímos a amostra com foco em quem iria concluir o ensino médio naquele ano e nas suas escolas, não há informações de alunos que concluíram o ensino médio em outros anos.

### Dados do Censo Escolar

Temos um código da escola que cada aluno do terceiro ano do Ensino Médio está no ano analisado. Portanto, buscamos dados do Censo Escolar para ligar aos nossos dados e analisar as condições de estudo dos estudantes, na tentativa de ligá-las ao desempenho no ENEM.
