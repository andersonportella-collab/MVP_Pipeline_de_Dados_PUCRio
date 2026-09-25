# 03 — Arquitetura do Pipeline

## 1. Visão geral

O projeto foi estruturado segundo uma arquitetura **Lakehouse**, utilizando o padrão **Medallion Architecture** para separar as diferentes responsabilidades do processamento de dados.

A implementação foi realizada no **Databricks Free Edition**, utilizando Apache Spark, PySpark, Spark SQL, Delta Lake e Unity Catalog.

A arquitetura organiza o fluxo desde a preservação dos arquivos de origem até a disponibilização dos dados preparados para análise.

O fluxo principal implementado é:

```text
RENAEST / SENATRAN
        |
        v
Arquivos CSV oficiais
        |
        v
   LANDING ZONE
        |
        v
      BRONZE
        |
        v
      SILVER
        |
        v
       GOLD
      /    \
     v      v
QUALIDADE  ANALYTICS
```

Além da fonte principal RENAEST/SENATRAN, o pipeline utiliza uma referência territorial oficial do **Instituto Pereira Passos / Prefeitura da Cidade do Rio de Janeiro (IPP/PCRJ)** para enriquecer os dados com informações de Região Administrativa.

---

## 2. Princípios arquiteturais

A arquitetura foi construída segundo os seguintes princípios:

### Separação de responsabilidades

Cada camada possui uma finalidade específica dentro do pipeline.

### Preservação da origem

Os arquivos recebidos da fonte oficial são preservados na Landing Zone antes das transformações realizadas nas demais camadas.

### Transformação progressiva

Os dados evoluem progressivamente de estruturas próximas à origem para estruturas tratadas e, posteriormente, para um modelo dimensional voltado ao consumo analítico.

### Persistência

As camadas Bronze, Silver e Gold utilizam tabelas Delta persistidas no ambiente Databricks.

### Rastreabilidade

A separação entre as etapas permite acompanhar o fluxo desde os arquivos de origem até as informações utilizadas nas análises.

### Integridade

O modelo Gold é submetido a verificações de granularidade, unicidade e integridade referencial.

### Conservadorismo na integração

Informações territoriais externas somente são associadas quando existe uma regra de correspondência considerada segura.

### Consumo sobre dados preparados

As análises são realizadas sobre a camada Gold, evitando utilizar diretamente os dados brutos como fonte das respostas analíticas.

---

## 3. Landing Zone

A Landing Zone representa o primeiro ponto de armazenamento dos arquivos utilizados pelo pipeline.

Os arquivos CSV oficiais do RENAEST/SENATRAN são preservados antes da transformação para tabelas Delta.

No ambiente implementado, foi utilizado o volume:

```text
workspace.bronze.landing
```

A Landing Zone permite manter os arquivos de origem separados das estruturas posteriormente processadas.

De forma simplificada:

```text
RENAEST / SENATRAN
        |
        v
Arquivos CSV oficiais
        |
        v
workspace.bronze.landing
```

A preservação dessa etapa contribui para a rastreabilidade entre os dados recebidos e as tabelas construídas pelo pipeline.

---

## 4. Camada Bronze

A camada Bronze representa a primeira persistência estruturada dos dados ingeridos.

Seu objetivo é manter os dados próximos à estrutura da fonte, convertendo os arquivos utilizados na ingestão para tabelas Delta.

A organização implementada é:

```text
workspace.bronze
├── acidentes
├── localidade
├── tipo_veiculo
└── vitimas
```

Nessa etapa, a prioridade é preservar os registros recebidos e estabelecer uma base persistente para as transformações posteriores.

A camada Bronze não representa ainda o modelo preparado para consumo analítico.

---

## 5. Camada Silver

A camada Silver concentra as principais operações de tratamento e preparação dos dados.

Entre as operações realizadas estão:

- limpeza;
- tipagem;
- padronização;
- validação de atributos;
- tratamento de campos textuais;
- preparação de datas e horários;
- validação de medidas quantitativas;
- recorte geográfico;
- preparação dos dados para a modelagem dimensional.

A organização principal é:

```text
workspace.silver
├── acidentes
├── localidade
├── tipo_veiculo
└── vitimas
```

Para o escopo analítico do MVP, a tabela de acidentes constitui a principal entrada para a construção da camada Gold.

O recorte do município do Rio de Janeiro é realizado utilizando o código IBGE:

```text
3304557
```

Após o tratamento e o recorte municipal, a tabela Silver de acidentes contém:

**55.046 registros.**

---

## 6. Camada Gold

A camada Gold disponibiliza os dados em uma estrutura orientada ao consumo analítico.

Foi adotado um **modelo dimensional em esquema estrela**, composto por uma tabela fato central e quatro dimensões.

A organização implementada é:

```text
workspace.gold
├── fato_acidentes
├── dim_tempo
├── dim_horario
├── dim_bairro
└── dim_regiao
```

O modelo pode ser representado de forma simplificada como:

```text
                       dim_tempo
                           |
                           |
dim_bairro -------- fato_acidentes -------- dim_horario
                           |
                           |
                       dim_regiao
```

A granularidade da tabela `fato_acidentes` é de **um registro por acidente**.

As tabelas persistidas possuem as seguintes quantidades de registros:

| Tabela | Registros |
|---|---:|
| `fato_acidentes` | 55.046 |
| `dim_tempo` | 2.434 |
| `dim_horario` | 1.319 |
| `dim_bairro` | 311 |
| `dim_regiao` | 34 |

A modelagem detalhada é apresentada em:

`docs/04_modelagem.md`

---

## 7. Enriquecimento territorial

O RENAEST fornece informação de bairro para parte dos acidentes, mas o campo `regiao` existente na fonte não representa as Regiões Administrativas do município do Rio de Janeiro.

Por esse motivo, o pipeline utiliza uma referência territorial oficial do **Instituto Pereira Passos / Prefeitura da Cidade do Rio de Janeiro (IPP/PCRJ)**.

O fluxo de enriquecimento territorial é:

```text
IPP/PCRJ
   |
   v
Referência oficial
Bairro × Região Administrativa
   |
   v
Normalização determinística
dos nomes dos bairros
   |
   v
Correspondência exata
com o bairro do RENAEST
   |
   v
dim_regiao
   |
   v
id_regiao na fato_acidentes
```

Antes da associação, os nomes são submetidos a operações determinísticas de normalização, incluindo tratamento de espaços, caixa e acentuação para comparação.

A associação definitiva utiliza somente correspondências exatas após essa normalização.

Técnicas de similaridade textual foram utilizadas apenas como instrumento de diagnóstico durante o desenvolvimento e não como mecanismo automático de classificação.

Os registros sem associação territorial segura permanecem identificados como não informados, evitando a atribuição de uma Região Administrativa sem evidência suficiente.

---

## 8. Cobertura do enriquecimento territorial

A qualidade da informação territorial da fonte impõe uma limitação importante à arquitetura analítica.

Dos **55.046 acidentes**:

| Situação | Acidentes | Percentual |
|---|---:|---:|
| Associados à referência oficial | 27.075 | 49,19% |
| Sem bairro informado | 26.894 | 48,86% |
| Bairro informado sem associação exata | 1.077 | 1,96% |

A baixa cobertura territorial não representa uma falha de integridade referencial do modelo Gold.

Os registros sem correspondência segura são associados às chaves especiais previstas nas dimensões, preservando a integridade do modelo sem criar classificações territoriais inferidas.

A análise detalhada dessas limitações está documentada em:

`docs/07_qualidade_dados.md`

---

## 9. Persistência

As camadas Bronze, Silver e Gold são persistidas utilizando tabelas Delta.

A organização física e lógica principal pode ser resumida da seguinte forma:

```text
Landing
└── arquivos CSV originais

workspace.bronze
├── acidentes
├── localidade
├── tipo_veiculo
└── vitimas

workspace.silver
├── acidentes
├── localidade
├── tipo_veiculo
└── vitimas

workspace.gold
├── fato_acidentes
├── dim_tempo
├── dim_horario
├── dim_bairro
└── dim_regiao
```

Essa separação mantém os arquivos recebidos distintos das tabelas processadas e das estruturas destinadas ao consumo analítico.

---

## 10. Qualidade como etapa do pipeline

A qualidade dos dados é tratada como uma etapa explícita da arquitetura.

Após a construção das camadas, o pipeline executa verificações relacionadas a:

- completude;
- unicidade;
- validade temporal;
- validade das medidas quantitativas;
- consistência entre atributos;
- integridade referencial;
- consistência territorial;
- reconciliação entre camadas.

A etapa é implementada principalmente no notebook:

`05_quality.ipynb`

A arquitetura pode, portanto, ser representada como:

```text
Fonte
  |
  v
Landing
  |
  v
Bronze
  |
  v
Silver
  |
  v
Gold
  |
  v
Qualidade
```

Os controles de qualidade não substituem as transformações das camadas anteriores. Sua função é verificar os resultados produzidos e identificar limitações relevantes para o consumo dos dados.

---

## 11. Reconciliação entre camadas

Um dos principais controles utilizados para verificar a preservação da granularidade dos acidentes é a reconciliação entre Bronze, Silver e Gold.

O resultado obtido para o conjunto do município do Rio de Janeiro foi:

```text
Bronze Rio: 55.046
      |
      v
Silver:     55.046
      |
      v
Gold:       55.046
```

As diferenças encontradas foram:

```text
Bronze → Silver = 0
Silver → Gold   = 0
```

Na tabela `fato_acidentes` também foram validados:

- **0 duplicidades de `num_acidente`**;
- **0 chaves estrangeiras nulas**;
- **0 chaves estrangeiras órfãs**.

Esses controles fornecem evidência de que as principais transformações preservaram a granularidade dos acidentes e a integridade referencial do modelo dimensional.

---

## 12. Consumo analítico

As análises são executadas sobre a camada Gold.

O fluxo de consumo é:

```text
Fonte
  |
  v
Landing
  |
  v
Bronze
  |
  v
Silver
  |
  v
Gold
  |
  v
Analytics
```

O notebook:

`06_analytics.ipynb`

utiliza o modelo dimensional para responder às perguntas de negócio relacionadas a:

- bairro;
- mês;
- dia da semana;
- horário;
- Região Administrativa;
- ano;
- qualidade e limitações dos dados.

Essa separação evita que as respostas analíticas dependam diretamente dos arquivos brutos de origem.

---

## 13. Fluxo arquitetural completo

Considerando as etapas de processamento, enriquecimento, qualidade e consumo, a arquitetura final pode ser representada como:

```text
                    RENAEST / SENATRAN
                            |
                            v
                   Arquivos CSV oficiais
                            |
                            v
                       LANDING ZONE
                            |
                            v
                          BRONZE
                            |
                            v
                          SILVER
                            |
                            v
                           GOLD
                            ^
                            |
              +-------------+-------------+
              |                           |
              |                        IPP/PCRJ
              |                           |
              |                    Referência oficial
              |                   Bairro × Região Adm.
              |                           |
              |                    Normalização +
              |                  correspondência exata
              |                           |
              +---------------------------+
                            |
                 +----------+----------+
                 |                     |
                 v                     v
             QUALIDADE              ANALYTICS
        Testes e controles      Perguntas de negócio
```

O RENAEST/SENATRAN constitui a fonte principal dos acidentes.

O IPP/PCRJ atua como fonte complementar para o enriquecimento territorial da camada Gold.

---

## 14. Organização dos notebooks

A implementação funcional do pipeline está distribuída nos seguintes notebooks:

| Notebook | Responsabilidade |
|---|---|
| `01_ingestao_api.ipynb` | Ingestão dos arquivos e persistência da camada Bronze |
| `03_silver.ipynb` | Tratamento e construção da camada Silver |
| `04_gold.ipynb` | Construção do modelo dimensional Gold e enriquecimento territorial |
| `05_quality.ipynb` | Testes e controles de qualidade |
| `06_analytics.ipynb` | Análises e respostas às perguntas de negócio |

O nome `01_ingestao_api.ipynb` foi definido durante o planejamento inicial, quando uma API era considerada como possível mecanismo de aquisição.

Após a investigação da fonte, foram utilizados os arquivos CSV disponibilizados oficialmente. O nome do notebook foi preservado na estrutura implementada.

---

## 15. Separação entre processamento e análise

Uma decisão arquitetural importante do projeto é manter separadas as responsabilidades de transformação e consumo.

As etapas de ingestão e preparação dos dados são executadas antes das análises:

```text
01_ingestao_api
       |
       v
   03_silver
       |
       v
    04_gold
       |
       +----------------+
       |                |
       v                v
  05_quality       06_analytics
```

O notebook de Analytics não é responsável por reconstruir o modelo dimensional.

Da mesma forma, os arquivos CSV originais não são utilizados diretamente como fonte das respostas às perguntas de negócio.

Essa separação facilita a rastreabilidade entre processamento, validação e consumo.

---

## 16. Catálogo e documentação

A arquitetura é complementada por documentação específica para cada componente do projeto.

Os principais documentos são:

| Documento | Conteúdo |
|---|---|
| `01_contexto_negocio.md` | Contexto, objetivos, escopo e perguntas |
| `02_fontes_dados.md` | Fontes, aquisição e limitações dos dados |
| `03_arquitetura.md` | Arquitetura Lakehouse e fluxo entre camadas |
| `04_modelagem.md` | Modelo dimensional Gold |
| `05_catalogo_dados.md` | Estrutura e campos do modelo Gold |
| `06_pipeline.md` | Implementação do pipeline |
| `07_qualidade_dados.md` | Controles e resultados de qualidade |
| `08_analises.md` | Resultados das análises |
| `09_autoavaliacao.md` | Avaliação do projeto, limitações e evoluções |

O catálogo detalhado dos campos da camada Gold está disponível em:

`catalog/data_dictionary.xlsx`

---

## 17. Versionamento e rastreabilidade

O código e a documentação do projeto são mantidos em repositório Git.

Os notebooks implementados no Databricks são versionados juntamente com a documentação do MVP.

Essa organização permite manter no mesmo projeto:

- implementação do pipeline;
- documentação técnica e metodológica;
- catálogo de dados;
- controles de qualidade;
- análises;
- histórico de alterações.

O versionamento complementa a rastreabilidade interna do pipeline ao registrar a evolução dos artefatos utilizados na implementação.

---

## 18. Limitações arquiteturais e de dados

A arquitetura preserva e documenta as limitações encontradas nas fontes em vez de ocultá-las por meio de transformações não verificadas.

As principais limitações relevantes para o consumo analítico são:

- 48,86% dos acidentes não possuem bairro informado;
- 1,96% possuem algum valor de bairro, mas não apresentam associação exata com a referência territorial utilizada;
- a cobertura territorial varia significativamente entre os anos;
- 2023 e 2024 possuem somente 10 meses disponíveis no conjunto analisado.

Essas limitações afetam a representatividade de determinadas análises, principalmente as análises territoriais.

Elas não alteram, entretanto, os resultados dos controles de integridade estrutural do modelo Gold, que apresentou zero chaves estrangeiras nulas e zero chaves estrangeiras órfãs.

---

## 19. Resultado arquitetural

A arquitetura final implementada pode ser sintetizada como:

```text
Fonte oficial
     |
     v
Landing Zone
     |
     v
Bronze
     |
     v
Silver
     |
     v
Gold
  /     \
 v       v
Qualidade Analytics
```

Essa arquitetura mantém separados:

- os dados recebidos da fonte;
- os dados persistidos após a ingestão;
- os dados tratados;
- o modelo dimensional;
- os controles de qualidade;
- o consumo analítico.

O resultado é uma cadeia de dados rastreável entre **fonte, ingestão, transformação, modelagem, validação e análise**, preservando explicitamente as limitações identificadas nos dados utilizados.

**Status da arquitetura: implementada e validada no escopo do MVP.**
