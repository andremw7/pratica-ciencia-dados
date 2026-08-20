
link google colab: https://colab.research.google.com/drive/1NinqFDrJEfxpv7NSHcP6VAH2_dYe9pGf

Relatório dia 13 de agosto

# Projeto Estratificação ENEM

![Python](https://img.shields.io/badge/Python-3.10%2B-blue?style=for-the-badge&logo=python)
![Pandas](https://img.shields.io/badge/Pandas-Data%20Analysis-150458?style=for-the-badge&logo=pandas)
![Google Colab](https://img.shields.io/badge/Google%20Colab-Cloud%20Execution-F9AB00?style=for-the-badge&logo=googlecolab)

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
