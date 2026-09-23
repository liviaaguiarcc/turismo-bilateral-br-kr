
# 🇧🇷 🇰🇷 Turismo Bilateral Brasil–Coreia do Sul

### Engenharia e Análise de Dados | Databricks · PySpark · SQL

## 1. Sobre o projeto

Este projeto tem como objetivo investigar a evolução e os padrões do turismo bilateral entre o Brasil e a Coreia do Sul, utilizando dados oficiais de chegadas de visitantes internacionais publicados pelos órgãos responsáveis de cada país.

A análise considera dois fluxos turísticos:

- **Coreia do Sul → Brasil:** chegadas de visitantes provenientes da Coreia do Sul ao território brasileiro.
- **Brasil → Coreia do Sul:** chegadas de visitantes brasileiros ao território sul-coreano.

O projeto utiliza a arquitetura Medallion para organizar a ingestão, a transformação e a preparação dos dados para análise.

Além da comparação histórica dos fluxos turísticos, pretende-se investigar a sazonalidade e as mudanças no volume de chegadas ao longo do tempo.

A análise de permanência, destinos visitados e padrões de gastos poderá ser incorporada posteriormente, conforme a disponibilidade de dados oficiais compatíveis.

---

## 2. Objetivos

### Objetivo geral

Construir um pipeline de dados para analisar e comparar a evolução dos fluxos turísticos entre Brasil e Coreia do Sul.

### Objetivos específicos

- Integrar dados oficiais de turismo publicados pelos dois países.
- Padronizar dados provenientes de fontes com estruturas distintas.
- Investigar a evolução histórica das chegadas de visitantes.
- Identificar padrões mensais e sazonais.
- Examinar variações nos fluxos turísticos ao longo do tempo.
- Desenvolver tabelas analíticas para consultas SQL e visualização de dados.
- Documentar a qualidade, as limitações e as transformações realizadas nos dados.

---

## 3. Fontes de dados

### Brasil

**Fonte:** Ministério do Turismo do Brasil

**Dataset:** Chegadas de Turistas Internacionais ao Brasil

**Período utilizado:** 1990–2025

**Formato original:** CSV, com arquivos anuais.

Os arquivos contêm informações sobre país e continente de origem, unidade federativa de entrada, via de acesso, ano, mês e quantidade de chegadas.

A estrutura dos arquivos apresenta diferenças entre os anos, incluindo alterações nos campos disponibilizados.

### Coreia do Sul

**Fonte:** Korea Tourism Organization (한국관광공사)

**Dataset:** Estatísticas de turismo da Coreia do Sul — 전체국가통계

**Arquivo utilizado:** 전체국가통계_202512.xlsx

**Formato original:** XLSX

**Cobertura temporal identificada:**

- Dados anuais: 1984–2025.
- Dados mensais: 1998–2025.

A planilha contém registros de entradas internacionais por país ou categoria estatística, com os períodos distribuídos horizontalmente em colunas.

Também apresenta totais gerais, agrupamentos regionais e categorias especiais.

### Período do comparativo bilateral

O projeto pretende utilizar o período comum de 1990 a 2025 para a análise histórica anual e de 1998 a 2025 para a análise mensal, respeitando a disponibilidade e a comparabilidade dos dados.

---

## 4. Tecnologias utilizadas

| Tecnologia | Aplicação |
|---|---|
| Databricks | Ambiente de desenvolvimento e processamento |
| Python | Desenvolvimento do pipeline de dados |
| PySpark | Ingestão e transformação dos dados |
| Spark SQL | Criação e consulta de tabelas |
| Delta Lake | Armazenamento das tabelas Bronze e Silver |
| Git e GitHub | Versionamento e documentação do projeto |

---

## 5. Arquitetura do projeto

O pipeline utiliza a arquitetura Medallion, organizada em três camadas:

### Bronze — Ingestão

Preservação dos dados provenientes das fontes oficiais, sem aplicação de filtros de negócio ou agregações.

### Silver — Transformação

Organização estrutural, tipagem, identificação de valores ausentes, padronização e documentação dos dados.

### Gold — Análise

Preparação das tabelas analíticas para o estudo comparativo dos fluxos turísticos.

**Status atual:** as camadas Bronze e Silver foram concluídas. A Gold ainda não foi iniciada.

---

## 6. Bronze — Ingestão dos dados

### 6.1. Brasil

Foram ingeridos os arquivos anuais de 1990 a 2025.

Os arquivos históricos e o arquivo de 2025 possuem estruturas distintas. Por isso, a ingestão foi organizada em dois processos de leitura.

Após a padronização dos nomes das colunas, os DataFrames foram combinados utilizando `unionByName()` com `allowMissingColumns=True`.

Essa abordagem permite alinhar os dados pelos nomes das colunas, evitando deslocamentos de valores entre campos que ocupam posições diferentes nos arquivos originais.

Foram adicionados metadados de origem e ingestão para garantir a rastreabilidade dos registros.

**Tabela:** `chegadas.brasil.bronze`

**Total de registros:** 961.420

**Período:** 1990–2025

### 6.2. Coreia do Sul

A ingestão foi realizada a partir da planilha Excel oficial, preservando sua estrutura original.

O arquivo apresenta os países e as categorias estatísticas em linhas e os períodos históricos distribuídos em colunas.

Os valores foram inicialmente preservados como texto, permitindo investigar posteriormente seus diferentes formatos numéricos.

**Estrutura da Bronze:**

- 290 registros.
- 381 colunas, incluindo o metadado de ingestão.
- 378 colunas referentes a períodos anuais e mensais.

---

## 7. Silver — Tratamento e governança

As transformações foram realizadas de maneira independente para cada fonte, considerando suas características estruturais.

### 7.1. Silver Brasil

**Tabela:** `chegadas.silver.brasil`

**Total de registros:** 961.420

A tabela Silver brasileira segue a metodologia de espelho governado da Bronze.

Foram realizadas as seguintes operações:

- Preservação dos registros e da granularidade original.
- Conversão da coluna `ano` para INT.
- Conversão da coluna `chegadas` para BIGINT.
- Preservação dos códigos e campos categóricos como texto.
- Preservação dos metadados de origem e ingestão.
- Inclusão do metadado de transformação.
- Documentação das colunas e da tabela.

#### Diagnóstico de valores ausentes

Foram identificados 9.264 registros com valores nulos na coluna `chegadas`, distribuídos entre os anos de 1996, 1999, 2004, 2007, 2012 e 2014.

A análise exploratória identificou padrões de ausência relacionados a determinadas combinações de país, unidade federativa e via de acesso.

A inspeção de um registro do arquivo original de 1999 confirmou que o campo correspondente já estava vazio na fonte.

**Decisão de tratamento:** os valores ausentes foram preservados como NULL, sem substituição por zero ou aplicação de técnicas de imputação.

A ausência de informação não deve ser interpretada como ausência de chegadas.

#### Diferenças estruturais em 2025

O arquivo de 2025 não disponibiliza todos os campos presentes nos arquivos históricos.

As colunas ausentes nesse arquivo foram preservadas como NULL na estrutura consolidada, sem preenchimento artificial.

A coluna `uf` representa a unidade federativa de entrada no Brasil, não necessariamente o destino visitado pelo turista.

#### Validação

A tabela Silver manteve a mesma quantidade de registros da Bronze.

As conversões numéricas foram verificadas antes da materialização, sem identificação de valores não nulos incompatíveis com os tipos escolhidos.

---

### 7.2. Silver Coreia

**Tabela:** `chegadas.silver.coreia`

**Total de registros:** 109.242

A planilha coreana possui estrutura larga, com uma coluna para cada período histórico.

Foi realizado um unpivot para transformar os períodos em linhas, gerando uma estrutura adequada para consultas temporais.

As colunas de ano, mês e granularidade foram criadas a partir dos cabeçalhos originais da planilha.

Os registros anuais e mensais foram identificados separadamente para evitar a dupla contagem de visitantes.

#### Organização estrutural

Foram identificadas 378 colunas de períodos:

- 42 períodos anuais.
- 336 períodos mensais.

A operação de unpivot gerou inicialmente 109.620 registros.

Os 378 registros correspondentes à linha original de cabeçalhos foram retirados da tabela de observações, uma vez que suas informações já haviam sido preservadas nas colunas de período e rastreabilidade.

A tabela resultante contém 109.242 registros.

#### Classificação das categorias

A fonte contém países e territórios, mas também apresenta totais gerais, totais regionais e outras categorias estatísticas.

Foi criada a coluna `tipo_categoria` para identificar:

- Países, territórios ou outras categorias individuais.
- Totais gerais.
- Totais regionais.
- Outros agrupamentos regionais.
- Categorias especiais.
- Agrupamentos especiais.
- Notas explicativas.
- Linhas vazias.

As categorias foram preservadas na Silver, permitindo que sejam selecionadas adequadamente na construção das tabelas analíticas.

A classificação de países e territórios permanece preliminar e não deve ser interpretada como uma lista exclusivamente de países independentes.

#### Padronização dos valores numéricos

A coluna original `chegadas_brutas` foi preservada como texto para fins de auditoria.

Foi criada a coluna numérica `chegadas`, utilizando BIGINT para os valores cuja representação pôde ser interpretada com segurança.

Foram identificados os seguintes formatos:

| Formato | Registros |
|---|---:|
| Inteiro simples | 59.775 |
| Número com separador de milhar | 17.736 |
| NULL | 22.373 |
| String vazia | 8.382 |
| Valor entre parênteses | 956 |
| Hífen | 16 |
| Sinal negativo e parênteses | 4 |

Os inteiros simples e os números com separadores de milhar foram convertidos para valores numéricos.

Os valores entre parênteses, os valores com sinal negativo e parênteses e os hífens foram mantidos na coluna original, sem conversão numérica automática, pois sua interpretação estatística ainda não foi confirmada.

#### Validação

Foram realizadas verificações da quantidade de registros após o unpivot, da classificação dos formatos numéricos e da organização dos dados anuais e mensais.

Também foi conferida a presença dos registros referentes ao Brasil em 2025, incluindo o total anual e os 12 meses correspondentes.

---


---

## 8. Gold — Construção das tabelas analíticas

A camada Gold foi desenvolvida para preparar os dados
dos dois fluxos turísticos para a análise comparativa
entre Brasil e Coreia do Sul.

Foram construídas duas tabelas analíticas: uma anual
e outra mensal.

Os dados foram selecionados e transformados a partir
das tabelas Silver, preservando as camadas anteriores.

### 8.1. Gold anual

**Tabela:** `chegadas.gold.turismo_bilateral_anual`

**Período:** 1990–2025

**Total de registros:** 72

**Granularidade:** um registro por ano e direção do fluxo.

A tabela reúne os dois fluxos turísticos:

- Coreia do Sul → Brasil
- Brasil → Coreia do Sul

Para o fluxo Coreia do Sul → Brasil, as chegadas foram
agregadas por ano a partir dos registros mensais,
considerando as diferentes UFs e vias de entrada.

Para o fluxo Brasil → Coreia do Sul, foram selecionados
exclusivamente os registros anuais da Silver coreana.

Essa separação evita a dupla contagem decorrente da
presença simultânea de totais anuais e dados mensais
na fonte coreana.

#### Tratamento dos valores ausentes

Foram identificados registros com valores nulos no
fluxo Coreia do Sul → Brasil nos anos de 1996, 1999,
2012 e 2014.

Para preservar a integridade dos indicadores, foram
criadas as seguintes colunas:

| Coluna | Descrição |
|---|---|
| `chegadas` | Total de chegadas, mantido como NULL quando existem registros de origem com valores ausentes. |
| `chegadas_observadas` | Soma das contagens disponíveis, mesmo quando existem registros nulos. |
| `total_registros` | Quantidade de registros utilizados no cálculo. |
| `registros_nulos` | Quantidade de registros de origem com valores ausentes. |
| `status_dados` | Indica se foram identificados registros nulos na construção do indicador. |

Os totais calculados a partir de dados parcialmente
preenchidos não são apresentados como totais completos.

### 8.2. Gold mensal

**Tabela:** `chegadas.gold.turismo_bilateral_mensal`

**Período:** 1998–2025

**Total de registros:** 672

**Granularidade:** um registro por ano, mês e direção
do fluxo turístico.

A tabela mensal foi construída para permitir a análise
da sazonalidade e da evolução dos fluxos turísticos
ao longo dos meses.

Foram realizadas as seguintes operações:

- Seleção dos registros correspondentes aos dois
  fluxos turísticos.
- Padronização dos nomes dos meses brasileiros para
  valores numéricos de 1 a 12.
- Agregação das chegadas de visitantes coreanos ao
  Brasil por ano e mês.
- Seleção exclusiva dos registros mensais dos
  brasileiros que chegaram à Coreia.
- Padronização dos indicadores de qualidade e
  das colunas das duas fontes.
- Integração dos dois fluxos em uma única tabela.

Durante a validação da cobertura temporal, foi
identificada uma diferença de grafia no arquivo
brasileiro de 2023: o mês de março também estava
representado como `marco`.

A regra de padronização foi ajustada para reconhecer
ambas as grafias.

Após a correção, a tabela passou a apresentar
12 meses para cada fluxo turístico em todos os anos
do período analisado.

---

## 9. Validação cruzada da Gold

Foi realizada uma comparação entre os totais anuais
e a soma dos 12 meses correspondentes, utilizando
as duas tabelas Gold.

**Período validado:** 1998–2025

**Total de comparações:** 56

Cada comparação corresponde a um ano e a uma direção
do fluxo turístico.

### 9.1. Resultados

| Resultado | Comparações |
|---|---:|
| Totais coincidentes sem registros nulos | 53 |
| Totais coincidentes com registros nulos | 3 |
| Divergências numéricas | 0 |
| Total | 56 |

Não foram identificadas divergências numéricas entre
os totais anuais e as somas mensais no período
de 1998 a 2025.

As três comparações que apresentaram coincidência
numérica com registros nulos pertencem ao fluxo
Coreia do Sul → Brasil:

| Ano | Registros nulos |
|---|---:|
| 1999 | 12 |
| 2012 | 36 |
| 2014 | 12 |

Nesses anos, a coincidência foi verificada entre
as somas dos valores disponíveis, e não entre
totais comprovadamente completos.

O ano de 1996 também apresenta 36 registros nulos
no fluxo Coreia do Sul → Brasil, mas não integra
a validação cruzada, pois está fora do período
mensal comum adotado para o comparativo.

Os anos de 1990 a 1997 permanecem disponíveis na
tabela Gold anual.

### 9.2. Limitações da validação

A coincidência entre os totais anuais e mensais
demonstra consistência aritmética entre as tabelas
construídas, mas não comprova a completude estatística
das fontes originais.

Também não garante que as duas instituições utilizem
definições metodológicas equivalentes de visitantes
internacionais.

Essas limitações deverão ser consideradas na
interpretação dos resultados.

---

## 10. Limitações metodológicas

As principais limitações identificadas durante
a construção do pipeline são:

**Comparabilidade das fontes:** as estatísticas
brasileiras e coreanas são produzidas por instituições
diferentes. As definições de país de origem,
residência e nacionalidade precisam ser verificadas
antes de interpretar os fluxos como diretamente
comparáveis.

**Valores ausentes:** a base brasileira apresenta
registros sem contagem de chegadas em anos específicos.
Essas ausências foram preservadas e identificadas
nos indicadores analíticos.

**Convenções numéricas da fonte coreana:** alguns
valores apresentados entre parênteses, com sinal
negativo ou com hífen não tiveram sua interpretação
estatística confirmada. Os valores originais foram
preservados na Silver.

**Granularidade temporal:** a fonte coreana contém
registros anuais e mensais referentes aos mesmos
períodos. A seleção inadequada dessas informações
pode resultar em dupla contagem.

**Unidades federativas:** as UFs presentes nos dados
brasileiros representam locais de entrada dos
visitantes, não necessariamente os destinos turísticos
efetivamente visitados.

---

## 11. Próximas etapas

### Análise exploratória

- Investigar a evolução histórica dos dois fluxos
  turísticos por meio de consultas SQL e Python.
- Analisar a sazonalidade das chegadas internacionais.
- Examinar variações anuais e mensais.
- Investigar períodos de crescimento e retração.
- Considerar as limitações e os registros incompletos
  na interpretação dos indicadores.

### Visualização de dados

- Criar gráficos de evolução histórica.
- Desenvolver visualizações de sazonalidade.
- Construir um dashboard para apresentação
  dos resultados.

### Relatório final

- Apresentar a metodologia utilizada.
- Sintetizar os principais resultados da análise.
- Discutir as limitações das fontes.
- Documentar as conclusões do estudo.

---

## 12. Status do projeto

| Etapa | Status |
|---|---|
| Coleta dos dados oficiais | Concluída |
| Bronze — Ingestão | Concluída |
| Silver Brasil | Concluída |
| Silver Coreia | Concluída |
| Gold anual | Concluída |
| Gold mensal | Concluída |
| Validação cruzada da Gold | Concluída |
| Documentação das tabelas Gold | Concluída |
| Análise exploratória | Não iniciada |
| Visualização e dashboard | Não iniciados |
| Relatório final | Não iniciado |

**Status geral:** pipeline de engenharia de dados
concluído. Próxima fase: análise exploratória
dos fluxos turísticos bilaterais.

---

## 13. Autoria

Projeto independente de engenharia e análise de dados desenvolvido para fins de estudo e portfólio, com foco na integração de dados oficiais sobre o turismo bilateral entre Brasil e Coreia do Sul.