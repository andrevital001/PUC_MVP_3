# MVP - Engenharia de Dados

**Nome:** André Vital de Medeiros

**Matrícula:** 4052025000626

## 1. Objetivo

## Descrição do Problema

  Os dados de produção energética brasileira são disponibilizados no **Anuário Estatístico da ANP**. Estes encontram-se distribuídos em diferentes arquivos e estruturados originalmente para consulta individual. 
  Para realização de análises integradas sobre a diversidade e a evolução territorial da produção energética brasileira torna-se necessário consolidar e estruturar estes arquivos.

  A fim de responder algumas questões estratégicas sobre a produção energética brasileira, foi proposta a criação de um pipeline de Engenharia de Dados baseado na arquitetura Medalhão (Bronze, Silver e Gold), utilizando os dados públicos da ANP (Agência Nacional do Petróleo, Gás Natural e Biocombustíveis) de 2014 a 2023. Deste modo será obtido ao final do processo um conjunto de dados analíticos capaz de responder as questões abaixo.


## Perguntas
> 1. Quais Unidades da Federação apresentam maior diversidade de produtos energéticos produzidos ao longo da série histórica?

> 2. Existe tendência de diversificação da matriz produtiva estadual entre 2014 e 2023?

> 3. Quais produtos apresentaram maior expansão territorial ao longo dos anos?


## 2. Preparação

  Está documentado no Notebook **01MVP_preparacao** o processo de criação da estrutura do catálogo de modo que seja aderente ao conceito de **Arquitetura Medalhão**.

### 2.1 Criação de Catálogo **_mvp_** e Schemas a serem utilizados

### 2.2. Criação dos Schemas:
  Foram criados no catálogo **_mvp_** os seguintes Schemas:
* _staging_ (onde será criado o volume _dados_anp_ para carregamento dos arquivos utilizados como base do projeto);
* _bronze_ (onde será criada a tabela da camada bronze, transformando os arquivos carregados numa tabela no Databricks);
* _silver_ (onde será criada a tabela limpa da camada silver, limpa, padronizada e estruturada para consumo analítico, com base na tabela gerada na camada bronze); 
* _gold_ (onde serão criadas as tabelas, modeladas segundo a análise proposta, com base no conteúdo criado na camada silver);
* _analise_ (onde serão realizadas as análises a fim de responder às perguntas presentes no início deste trabalho).

> Segue imagem dos catálogos e schemas criados:
<img width="625" height="847" alt="image" src="https://github.com/user-attachments/assets/41a49ef6-11f0-4e53-b3af-69e29fbfe5d9" />

  
## 3. Carga de Dados

  O processo de carregamento dos arquivos extraídos do repositório do GitHub está documentado no Notebook **02MVP_download**.

  Os arquivos carregados fazem parte do anuário estatístico da ANP, no link abaixo: [Dados de produção do anuário estatístico da ANP (Agência Nacional do Petróleo, Gás Natural e Biocombustíveis)](https://www.gov.br/anp/pt-br/centrais-de-conteudo/publicacoes/anuario-estatistico/anuario-estatistico-2023)

  Como há o objetivo de construir um MVP que dê continuidade aos anteriores, acessaremos os arquivos por meio do diretório público github já criado para as primeiras sprints. [Diretório no Github](https://github.com/andrevital001/PUC)

Segue abaixo os arquivos utilizados:

* Tabela 2.9 - Produção de petróleo, por localização (terra e mar, pré-sal e pós-sal), segundo unidades da Federação - 2014-2023

* Tabela 2.10 - Produçao de LGN, segundo unidades da Federaçao - 2014-2023

* Tabela 2.13 - Produção de gás natural, por localização (terra e mar, pré-sal e pós-sal), segundo unidades da Federação - 2014-2023

* Tabela 4.1 - Producao de etanol anidro e hidratado, segundo grandes regioes e unidades da Federaçao - 2014-2023

* Tabela 4.10 - Produção de biodiesel (B100), segundo grandes regiões e unidades da Federação - 2014-2023

* Tabela 4.17 - Producao de biometano, segundo grandes regiões e unidades da Federacao - 2020-2023

> Estes arquivos serão carregados no volume **dados_anp**, schema **staging**, no catálogo **mvp**, conforme a imagem abaixo:
  <img width="1880" height="855" alt="image" src="https://github.com/user-attachments/assets/50bc67c9-8f53-4c83-a3d9-cb5f4f6d8e0d" />


## 4. Bronze

  A criação da Tabela _producao_energetica_ no schema **bronze** está documentada no Notebook **03MVP_bronze**.

  O objetivo desta etapa é carregar no Databricks uma única tabela que contenha os de todos os arquivos carregados.
  
  **Este Notebook é composto pelas seguintes ações:**
* Definição do uso do catálogo **mvp** e schema **bronze**;
* Consolidação dos arquivos num único dataframe via python;
* Renomear os cabeçalhos de modo a remover os caracteres especiais e simplificar os títulos.

> Segue abaixo a consulta da tabela **producao_energetica** criada no schema **bronze**:

<img width="1907" height="898" alt="image" src="https://github.com/user-attachments/assets/b5901cb4-8785-439e-ab96-1ce3d79e3bc7" />


## 4.1. Qualidade dos Dados

**_4.1.1. Estrutura_**

  Todos os arquivos carregados apresentaram a mesma estrutura, composta por quatro colunas descritivas (_Regiao, Unidades da Federacao, Unidade e Produto_) e dez colunas referentes aos anos de 2014 a 2023.
  Essa padronização estrutural possibilitou a consolidação dos arquivos em uma única tabela na camada Bronze, sem necessidade de adaptações específicas para cada conjunto de dados.

**_4.1.2. Valores Ausentes_**

* Foram localizados valores nulos, principalmente na coluna _Regiao_.
* Foi encontrado o caractere "-" nas colunas onde estão os valores de produção, representando a inexistência de produção para determinado produto, Unidade da Federação e ano.
  
**_4.1.3. Inconsistência de tipos_**

  Foi observado que as colunas anuais apresentavam diferentes tipos de dados entre os arquivos.

  Enquanto alguns arquivos foram interpretados automaticamente como numéricos (float), outros foram carregados como texto (string) em razão da presença simultânea de:
* separadores de milhares (.);
* separadores decimais (,);
* valores representados pelo caractere "-".

**_4.1.4. Padronização textual_**

  Os atributos descritivos apresentavam pequenas inconsistências de formatação, como espaços em branco excedentes.

**_4.1.5. Estrutura Wide_**

  Os dados disponibilizados pela ANP encontram-se originalmente no formato Wide, no qual cada ano é representado por uma coluna distinta.

**_4.1.6. Avaliação da Qualidade dos Dados_**
  
  A análise de qualidade demonstrou que os dados apresentam boa consistência estrutural, sendo as principais necessidades de tratamento relacionadas à padronização dos tipos de dados, no tratamento dos valores ausentes e na reorganização do formato dos registros.

  Os tratamentos adequados serão realizados na camada Silver, onde espera-se obter um conjunto de dados padronizado, consistente e adequado para a construção das tabelas analíticas da camada Gold e para responder às perguntas de negócio definidas neste MVP.


## 5. Silver

  A criação da Tabela _producao_energetica_ no schema **silver** está documentada no Notebook **04MVP_silver**.

  O objetivo desta etapa é transformar os dados consolidados da camada Bronze em um conjunto de dados limpo, padronizado e estruturado para consumo analítico.


### 5.1. Etapas do tratamentos dos dados:
* Padronização e simplificação dos cabeçalhos, removendo os caracteres especiais e maiúsculas;
  
    _(Adoção do snake_case a fim de padronizar e melhorar a legibilidade do código)_
  
* Remoção do campo 'região';
  
   _(A informação não será necessária por se tratar apenas de um agrupamento do campo 'uf')_
  
* Padronização de valores vazios, identificando-os como _null_;

  _(Algumas células continham '-' no lugar de null, o que prejudicava a interpretação dos dados)_
  
* Adequação do tipo dos dados de cada coluna (inteiro, texto ou decimal);
  
  _(Todos os dados estavam com o tipo string)_

* Transposição das colunas relativas aos anos para a coluna 'ano'.
  
  _(Transformação do formato Wide para Long)_

> Segue abaixo a consulta da tabela **producao_energetica** criada no schema **silver**:
<img width="1906" height="892" alt="image" src="https://github.com/user-attachments/assets/5d79bcc1-1ece-4a41-b3eb-c67625708617" />


### 5.2. Catálogo de Dados

| Coluna             | Tipo   | Descrição                                                                                                                                                                                                                        |
| ------------------ | ------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **uf**             | string | Unidade da Federação responsável pela produção energética. Corresponde ao estado brasileiro onde a produção foi registrada.                                                                                                      |
| **unidade_medida** | string | Unidade de medida utilizada para representar a produção do produto energético (ex.: m³, mil m³, mil barris).                                                                                                                     |
| **produto**        | string | Produto energético produzido, conforme classificação da ANP (ex.: Petróleo, Gás Natural, Biodiesel, Biometano, Etanol Anidro e Hidratado, LGN).                                                                                  |
| **ano**            | int    | Ano de referência da produção energética. Valores compreendidos entre 2014 e 2023.                                                                                                                                               |
| **producao**       | double | Quantidade produzida do respectivo produto energético na Unidade da Federação, expressa na unidade de medida correspondente. Valores ausentes foram convertidos para **NULL** durante o processo de tratamento da camada Silver. |


## 6. Gold
   
  O objetivo desta camada é realizar a transformação dos da tabela **mvp.silver.producao_energetica** em um nova camada **gold** modelada para a análise proposta.


### 6.1. Data Marts criados

  Nesta camada serão criados 3 Data Marts à partir da tabela **mvp.silver.producao_energetica**. **São eles:**
* **diversidade_ano**
* **diversidade_uf**
* **expansao_produtos**


### 6.2. Criação do Data Mart _gold.diversidade_uf_

  Neste Data Mart são agrupados os produtos por UF e são consideradas apenas as produções maiores do que 0.

  O objetivo é fornecer uma base que responda a primeira pergunta deste trabalho:
> *"Como seria o ranking das Unidades da Federação considerando a diversidade de produtos?"*

**Segue abaixo imagem do Data Mart criado e executado:**
<img width="1205" height="587" alt="image" src="https://github.com/user-attachments/assets/a3127a69-3720-4027-ba9b-774836fec677" />


 **_6.2.1. Catálogo de dados - gold.diversidade_uf_**

 
| Coluna           | Tipo   | Descrição                                                                                                                                                                             |
| ---------------- | ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **uf**           | string | Unidade da Federação considerada na análise da diversidade da produção energética.                                                                                                    |
| **qtd_produtos** | int    | Quantidade de produtos energéticos distintos produzidos pela Unidade da Federação durante o período analisado (2014–2023), considerando apenas registros com produção maior que zero. |


### 6.3. Criação do Data Mart _diversidade_ano_

  Neste Data Mart serão consideradas as colunas 'ano' e 'uf', e contabilizados aqueles onde a quantidade de produção for maior que 0.

  Este Data Mart tem por objetivo auxiliar na resposta à segunda pergunta:
> *"Há uma tendência de diversificação da produção energética ao longo do período analisado?"*

**Segue abaixo imagem do Data Mart criado e executado:**
<img width="1192" height="592" alt="image" src="https://github.com/user-attachments/assets/9357798c-1828-44e9-b998-ef71d1b97009" />


 **_6.3.1. Catálogo de dados - gold.diversidade_ano_**

 | Coluna           | Tipo   | Descrição                                                                                                                                                       |
| ---------------- | ------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **ano**          | int    | Ano de referência da produção energética, compreendido entre 2014 e 2023.                                                                                       |
| **uf**           | string | Unidade da Federação considerada na análise da diversificação anual da produção energética.                                                                     |
| **qtd_produtos** | int    | Quantidade de produtos energéticos distintos produzidos pela Unidade da Federação no respectivo ano, considerando apenas registros com produção maior que zero. |


### 6.4. Criação do Data Mart _expansao_produtos_

  Este Data Mart contém as colunas 'ano' e 'produto', contabilizando a quantidade de estados onde a produção foi maior que 0. 

  O objetivo é auxiliar na resposta à terceira pergunta:
> _"Quais produtos apresentaram maior expansão territorial entre 2014 e 2023?"_

**Segue abaixo imagem do Data Mart criado e executado:**

<img width="1196" height="633" alt="image" src="https://github.com/user-attachments/assets/08f94722-577a-43b3-afbd-f74aa2991381" />


 **_6.4.1. Catálogo de dados - gold.expansao_produtos_**


| Coluna      | Tipo   | Descrição                                                                                                                                                       |
| ----------- | ------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **ano**     | int    | Ano de referência da produção energética, compreendido entre 2014 e 2023.                                                                                       |
| **produto** | string | Produto energético analisado conforme classificação da ANP.                                                                                                     |
| **qtd_ufs** | int    | Quantidade de Unidades da Federação que registraram produção do respectivo produto no ano analisado, considerando apenas registros com produção maior que zero. |


## 7. Análise


### 7.1. Introdução da Análise

  Nesta etapa são utilizadas as tabelas da camada Gold para responder às perguntas de negócio definidas no início do projeto.

  As consultas foram realizadas utilizando SQL, tendo como base os Data Marts construídos durante a etapa de modelagem.


### 7.2. Pergunta 1

> Quais Unidades da Federação apresentam maior diversidade de produtos energéticos produzidos ao longo da série histórica?

  A resposta a esta pergunta será baseada no Data Mart **gold.diversidade_uf**

**Segue abaixo a imagem da consulta SQL utilizada, ordenando os UFs de acordo com a quantidade de produtos produzidos na série:**

<img width="1335" height="202" alt="image" src="https://github.com/user-attachments/assets/320c5e8e-4b1b-43ff-88bd-175121a68ff4" />


**Segue abaixo a imagem da tabela criada a partir da consulta SQL:**

_Tabela - Ranking da Diversidade de Produtos por Unidade da Federação_

<img width="507" height="456" alt="image" src="https://github.com/user-attachments/assets/dab18591-f2eb-43e6-ba6b-318b506b8126" />


**Segue abaixo a imagem do gráfico de barras gerado a parte da tabela gerada:**

_Gráfico - Ranking da Diversidade de Produtos por Unidade da Federação_

<img width="1337" height="382" alt="image" src="https://github.com/user-attachments/assets/734de824-eaf9-428c-84c7-f221048819f9" />


  Conforme apresentado na tabela e no gráfico acima, vemos que Ceará, Rio de Janeiro e São Paulo possuem 6 produtos diferentes cada, seguidos por Bahia e Rio Grande do Norte, com 5 produtos cada. Neste resultado observamos a proeminência das Regiões Sudeste e Nordeste na diversidade da produção energética brasileira.


### 7.3. Pergunta 2

> Existe tendência de diversificação da matriz produtiva estadual entre 2014 e 2023?

  A resposta a esta pergnta será baseada no Data Mart **gold.diversidade_ano**

**Segue abaixo a imagem da consulta SQL utilizada, calculando a média da variedade de produtos produzidos por Unidade da Federação no período de 2014 a 2023:**

<img width="1342" height="186" alt="image" src="https://github.com/user-attachments/assets/b650f830-ca74-4a3b-8da1-6ee9fb41e8e5" />


**Segue abaixo a imagem da tabela criada a partir da consulta SQL:**

_Tabela - Média anual da variedade de produtos por Unidade da Federação_

<img width="317" height="332" alt="image" src="https://github.com/user-attachments/assets/b8253f63-7f0a-4c0e-9a36-4fc5c7a186ab" />


**Segue abaixo a imagem do gráfico de barras gerado a partir da tabela gerada:**

_Gráfico - Média anual da variedade de produtos por Unidade da Federação_

<img width="1337" height="407" alt="image" src="https://github.com/user-attachments/assets/57743a39-b7a9-486d-b79d-ae7544331479" />


  A média de produtos energéticos produzidos por Unidade da Federação permaneceu relativamente estável durante todo o período analisado, apresentando pequenas oscilações entre 2,65 e 2,83 produtos distintos.
  Embora os anos de 2022 e 2023 apresentem os maiores valores médios da série histórica, essa elevação é discreta e, isoladamente, não caracteriza uma tendência consistente de crescimento da diversificação ao longo do período analisado.


### 7.4. Pergunta 3

> Quais produtos apresentaram maior expansão territorial ao longo dos anos?

  A resposta a esta pergnta será baseada no Data Mart **gold.expansao_produtos**


**Para responder mais adequadamente à pergunta apresentada, serão criadas duas consultas complementares.**


 **_7.4.1. Consulta 1_**


**Segue abaixo a imagem da consulta SQL utilizada, contabilizando a quantidade de Unidades da Federação em que há a produção de cada um dos dos produtos analisados ano a ano:**

<img width="1335" height="197" alt="image" src="https://github.com/user-attachments/assets/981555b9-6653-429e-9426-c80355977cb5" />


**Segue abaixo a imagem da tabela criada a partir da consulta SQL:**

_Tabela - Expansão territorial dos produtos ano a ano_

<img width="407" height="450" alt="image" src="https://github.com/user-attachments/assets/f18b54c6-0708-4757-bcde-ef0c55b46de9" />


**Segue abaixo a imagem do gráfico de barras gerado a parte da tabela gerada:**

_Gráfico - Expansão territorial dos produtos ano a ano_

<img width="1337" height="400" alt="image" src="https://github.com/user-attachments/assets/f261efb5-a9d6-49cd-93fa-8cc4a97397f7" />


 **_7.4.2. Consulta 2_**


**Segue abaixo a imagem da consulta SQL utilizada, demonstrando a variação absoluta do número de Unidades da Federação produtoras de cada produto energético entre 2014 e 2023:**

<img width="1340" height="632" alt="image" src="https://github.com/user-attachments/assets/40e2dd17-9c37-4213-b620-96a9ff73fcd8" />


**Segue abaixo a imagem da tabela criada a partir da consulta SQL:**

_Tabela - Expansão territorial dos produtos de 2014 a 2023_

<img width="635" height="237" alt="image" src="https://github.com/user-attachments/assets/a507bbef-6c9c-413d-9a56-7b50d0970fd5" />


**Segue abaixo a imagem do gráfico de barras gerado a parte da tabela gerada:**

_Gráfico - Expansão territorial dos produtos de 2014 a 2023_

<img width="1340" height="402" alt="image" src="https://github.com/user-attachments/assets/fad7a103-d91c-4afc-91f8-f9b7edbb396b" />


### 7.5. Conclusão

   Observa-se nas tabelas e gráficos apresentados que o Biometano apresentou a maior expansão territorial da série histórica, passando de nenhuma Unidade da Federação produtora em 2014 para três em 2023. Biodiesel e Gás Natural também registraram crescimento, ainda que discreto, ampliando sua presença em uma Unidade da Federação cada.
  Em contrapartida, Etanol anidro e hidratado e LGN apresentaram redução no número de estados produtores durante o período analisado, enquanto o Petróleo manteve estabilidade. Esses resultados indicam que a evolução territorial da produção energética brasileira ocorreu de forma heterogênea, com comportamentos distintos entre os diferentes produtos energéticos.


## 8. Pipeline de Dados

  Este Pipeline foi construído na plataforma Databricks. 
  O pipeline inicia com a preparação do ambiente e carregamento dos arquivos CSV na Staging. Posteriormente, os dados são consolidados na Bronze, tratados e padronizados na Silver e modelados em Data Marts na Gold. Por fim, os Data Marts são utilizados nas consultas da etapa de análise.
  
  Foram criados 6 notebooks:

* 01MVP_preparacao
* 02MVP_download
* 03MVP_bronze
* 04MVP_silver
* 05MVP_gold
* 06MVP analise

Segue abaixo a imagem que apresenta a estrutura do Pipeline:

<img width="1983" height="615" alt="Pipeline_de_dados" src="https://github.com/user-attachments/assets/5f481f8e-e313-48b4-9803-d88365c4ff55" />

## 9. Autoavaliação

  Entende-se o objetivo do presente trabalho foi atingido de forma satisfatória, uma vez que foi possível implementar todas as etapas previstas da arquitetura de dados, desde a obtenção dos arquivos CSV até a construção dos Data Marts na camada Gold, utilizados para responder às perguntas de negócio definidas no planejamento do trabalho.
  Entre as principais dificuldades encontradas destaca-se a adaptação da estrutura original dos dados disponibilizados pela ANP. Durante o desenvolvimento também foram enfrentadas dificuldades relacionadas ao ambiente do Databricks, como problemas temporários de conexão com o recurso Serverless, posteriormente solucionados por meio da atualização do ambiente de execução.
  Para trabalhos futuros, o projeto pode ser enriquecido por meio da incorporação de séries históricas mais recentes, permitindo acompanhar a evolução da matriz energética em anos posteriores. Também seria interessante integrar outras bases públicas relacionadas ao setor energético, como consumo, capacidade instalada e indicadores econômicos, ampliando o potencial analítico da solução. Também pode ser considerada a criação dashboards interativos utilizando ferramentas de Business Intelligence ou os próprios recursos de visualização do Databricks, permitindo maior facilidade na exploração dos dados e na comunicação dos resultados.
  O desenvolvimento deste MVP permitiu consolidar os conceitos estudados na Sprint de Engenharia de Dados, proporcionando experiência prática na construção de pipelines em ambiente de nuvem, modelagem utilizando a arquitetura medalhão, documentação técnica e utilização de consultas SQL para geração de informações analíticas, atendendo aos objetivos propostos para a disciplina.
