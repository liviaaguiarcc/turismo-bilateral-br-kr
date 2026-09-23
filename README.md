
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

## 8. Limitações e cuidados metodológicos

### Comparabilidade entre as fontes

As estatísticas brasileiras e coreanas são produzidas por instituições diferentes.

Antes da construção das tabelas comparativas, será necessário verificar as definições de visitante internacional, país de residência e nacionalidade utilizadas em cada fonte.

Os indicadores não devem ser tratados automaticamente como medidas metodologicamente equivalentes.

### Granularidade temporal

A base coreana contém totais anuais e registros mensais referentes aos mesmos períodos.

A soma simultânea desses dois tipos de registros produziria dupla contagem.

As análises deverão selecionar a granularidade adequada para cada indicador.

### Categorias estatísticas

A base coreana inclui totais gerais, agrupamentos regionais e outras categorias que não representam países individuais.

Esses registros devem ser diferenciados nas consultas analíticas para evitar a duplicação de contagens.

### Valores ausentes

Os valores nulos da base brasileira foram preservados e documentados.

Na base coreana, também foram identificados valores vazios e marcadores cujo significado ainda não foi confirmado.

Totais calculados a partir de dados incompletos devem ser interpretados considerando essas limitações.

### Dados de destinos turísticos

A unidade federativa de entrada no Brasil não representa necessariamente o destino final visitado.

Portanto, os dados de chegadas não devem ser utilizados isoladamente para inferir quais cidades ou regiões os turistas efetivamente visitaram.

---

## 9. Próximas etapas

### Gold — Preparação das tabelas analíticas

- Definir o período comum para as análises bilaterais.
- Selecionar os registros correspondentes a cada fluxo turístico.
- Organizar separadamente os indicadores anuais e mensais.
- Construir tabelas analíticas para estudar evolução histórica e sazonalidade.
- Verificar a comparabilidade dos indicadores entre as duas fontes.

### Análise e visualização

- Desenvolver consultas SQL para investigar os fluxos turísticos.
- Criar visualizações da evolução das chegadas.
- Comparar padrões temporais entre os dois países.
- Elaborar um relatório analítico com os resultados e as limitações.

---

## 10. Status do projeto

| Etapa | Status |
|---|---|
| Coleta dos dados oficiais | Concluída |
| Bronze — Ingestão | Concluída |
| Silver Brasil | Concluída |
| Silver Coreia | Concluída |
| Gold — Tabelas analíticas | Não iniciada |
| Análise e visualização | Não iniciada |
| Relatório final | Não iniciado |

---

## 11. Autoria

Projeto independente de engenharia e análise de dados desenvolvido para fins de estudo e portfólio, com foco na integração de dados oficiais sobre o turismo bilateral entre Brasil e Coreia do Sul.