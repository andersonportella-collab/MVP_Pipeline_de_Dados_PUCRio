# Pipeline Lakehouse para Análise de Acidentes de Trânsito no Município do Rio de Janeiro

MVP desenvolvido para a disciplina de **Engenharia de Dados da PUC-Rio**, com o objetivo de construir um pipeline de dados ponta a ponta em ambiente cloud, desde a ingestão de dados públicos oficiais até a disponibilização de um modelo dimensional para análises.

O projeto utiliza **Databricks, Apache Spark, PySpark, Delta Lake e Unity Catalog**, seguindo uma arquitetura Lakehouse em camadas.

---

## 1. Visão geral

O projeto processa dados públicos de acidentes de trânsito do município do Rio de Janeiro, tendo como fonte principal o **Registro Nacional de Sinistros e Estatísticas de Trânsito (RENAEST/SENATRAN)**.

Para o enriquecimento territorial, é utilizada uma segunda fonte oficial do **Instituto Pereira Passos / Prefeitura da Cidade do Rio de Janeiro (IPP/PCRJ)**, permitindo relacionar os bairros informados nos acidentes às Regiões Administrativas do município quando existe correspondência segura.

O fluxo implementado é:

```text
RENAEST / SENATRAN
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
      /    \
     v      v
Qualidade  Analytics
```

O resultado é um pipeline rastreável entre fonte, transformação, validação, modelagem e análise.

---

## 2. Objetivo

O objetivo geral do MVP é construir um pipeline de dados em ambiente cloud capaz de:

- utilizar dados públicos oficiais;
- preservar os arquivos de origem em uma Landing Zone;
- persistir os dados ingeridos em uma camada Bronze;
- realizar limpeza, tipagem, padronização e validação na camada Silver;
- construir uma camada Gold baseada em modelo dimensional;
- enriquecer os dados com informação oficial de Região Administrativa quando houver correspondência territorial segura;
- implementar verificações de qualidade e integridade;
- reconciliar os registros entre as diferentes camadas;
- disponibilizar dados estruturados para análises;
- documentar fontes, arquitetura, modelagem, catálogo, qualidade, limitações e resultados.

---

## 3. Fontes de dados

### RENAEST / SENATRAN

A fonte principal é o **Registro Nacional de Sinistros e Estatísticas de Trânsito — RENAEST**, disponibilizado pela Secretaria Nacional de Trânsito — SENATRAN.

O conjunto utilizado no projeto é:

`renaest_dabertos_20250512`

Foram considerados quatro arquivos da fonte:

- acidentes;
- localidades;
- tipos de veículos;
- vítimas.

O modelo analítico principal deste MVP utiliza os registros de acidentes.

O município do Rio de Janeiro foi selecionado por meio do código IBGE:

`3304557`

Após o recorte municipal e os tratamentos realizados, a base principal contém:

**55.046 acidentes.**

O período encontrado nos dados utilizados é:

**01/01/2018 a 30/11/2024.**

### IPP / Prefeitura da Cidade do Rio de Janeiro

Para o enriquecimento territorial foi utilizada uma referência oficial do **Instituto Pereira Passos / Prefeitura da Cidade do Rio de Janeiro**.

Essa fonte fornece a relação entre bairros e Regiões Administrativas e foi utilizada para complementar a informação territorial existente no RENAEST.

A associação definitiva utiliza somente correspondências exatas após normalização determinística dos nomes dos bairros.

Técnicas de similaridade textual foram investigadas como diagnóstico, mas não foram utilizadas para realizar classificações automáticas quando não havia correspondência segura.

---

## 4. Arquitetura

O pipeline segue uma arquitetura **Lakehouse / Medallion Architecture**.

### Landing Zone

Preserva os arquivos originais utilizados como fonte de ingestão.

### Bronze

Mantém os dados ingeridos próximos à estrutura de origem e persistidos em formato Delta.

### Silver

Realiza operações de tratamento, incluindo:

- limpeza;
- tipagem;
- padronização;
- validação;
- recorte geográfico;
- preparação dos dados para modelagem.

### Gold

Disponibiliza um modelo dimensional em esquema estrela voltado às análises do MVP.

### Qualidade

Executa verificações sobre completude, unicidade, validade, integridade referencial, consistência territorial e reconciliação entre camadas.

### Analytics

Utiliza a camada Gold para responder às perguntas de negócio definidas para o projeto.

---

## 5. Modelo dimensional

A camada Gold utiliza um **Star Schema** composto por uma tabela fato e quatro dimensões:

```text
                       dim_tempo
                           |
                           |
dim_bairro -------- fato_acidentes -------- dim_horario
                           |
                           |
                       dim_regiao
```

Tabelas implementadas:

| Tabela | Registros | Papel |
|---|---:|---|
| `fato_acidentes` | 55.046 | Tabela fato — um registro por acidente |
| `dim_tempo` | 2.434 | Dimensão temporal |
| `dim_horario` | 1.319 | Dimensão de horário |
| `dim_bairro` | 311 | Dimensão de bairro |
| `dim_regiao` | 34 | Dimensão de Região Administrativa |

A granularidade da tabela `fato_acidentes` é de **um registro por acidente**.

---

## 6. Qualidade dos dados

A qualidade foi tratada como uma etapa explícita do pipeline.

Entre os controles implementados estão:

- verificação de completude;
- unicidade de identificadores;
- validade temporal;
- validade das medidas;
- consistência entre atributos;
- integridade referencial;
- consistência territorial;
- reconciliação entre as camadas.

A reconciliação principal resultou em:

```text
Bronze: 55.046
   ↓
Silver: 55.046
   ↓
Gold:   55.046
```

Na tabela fato foram obtidos:

- **55.046 registros**;
- **0 duplicidades de `num_acidente`**;
- **0 chaves estrangeiras nulas**;
- **0 chaves estrangeiras órfãs**.

Esses resultados confirmam a preservação da granularidade dos acidentes durante as principais transformações do pipeline.

---

## 7. Qualidade territorial e limitações

A principal limitação identificada está relacionada à informação territorial disponível na fonte.

Dos 55.046 acidentes:

| Situação | Registros | Percentual |
|---|---:|---:|
| Bairro informado | 28.152 | 51,14% |
| Bairro não informado | 26.894 | 48,86% |

Para a associação às Regiões Administrativas:

| Situação | Registros | Percentual |
|---|---:|---:|
| Associados à referência oficial | 27.075 | 49,19% |
| Sem bairro informado | 26.894 | 48,86% |
| Bairro informado sem associação exata | 1.077 | 1,96% |

Os registros sem correspondência territorial segura foram preservados como não associados, evitando atribuições baseadas exclusivamente em similaridade textual.

Também foi identificada diferença de completude territorial entre os anos, especialmente entre 2018 e 2020.

Outra limitação está na cobertura temporal: os anos de 2023 e 2024 possuem somente 10 meses disponíveis no conjunto analisado.

Por esse motivo, essas limitações são consideradas explicitamente na interpretação das análises.

---

## 8. Perguntas de negócio

O projeto foi estruturado para responder às seguintes perguntas:

1. Quais bairros apresentam maior quantidade de acidentes de trânsito?
2. Existe variação na quantidade de acidentes entre os meses do ano?
3. Quais dias da semana apresentam maior frequência de acidentes?
4. Quais horários concentram maior quantidade de ocorrências?
5. Como os acidentes se distribuem entre as Regiões Administrativas do município do Rio de Janeiro?
6. Como a quantidade de acidentes evoluiu ao longo dos anos disponíveis?
7. Quais problemas de qualidade existem nos dados e como eles podem afetar as análises produzidas?

As análises são predominantemente **descritivas**.

As frequências observadas por bairro ou Região Administrativa não devem ser interpretadas diretamente como medidas de risco. Uma análise de risco exigiria variáveis adicionais de exposição, como população, frota, extensão da malha viária ou volume de tráfego.

---

## 9. Principais resultados analíticos

Entre os resultados obtidos no conjunto analisado:

- Campo Grande apresentou a maior frequência entre os acidentes com bairro identificado;
- agosto apresentou a maior média mensal considerando os anos completos de 2018 a 2022;
- segunda-feira apresentou a maior frequência entre os dias da semana, embora a distribuição seja relativamente homogênea;
- a tarde apresentou a maior quantidade de registros entre as faixas horárias utilizadas;
- 19h e 18h apresentaram as maiores frequências por hora;
- Barra da Tijuca apresentou a maior frequência entre os acidentes associados às Regiões Administrativas;
- entre os anos completos, foi observada redução relevante em 2020 e aumento em 2021 e 2022;
- a qualidade da informação territorial constitui a principal limitação para as análises espaciais.

Esses resultados representam padrões observados nos registros disponíveis e não estabelecem relações causais.

---

## 10. Tecnologias utilizadas

- Databricks Free Edition
- Apache Spark
- PySpark
- Spark SQL
- Delta Lake
- Unity Catalog
- Python
- Pandas
- Matplotlib
- Git
- GitHub

---

## 11. Estrutura do repositório

```text
MVP_Pipeline_de_Dados_PUCRio/
│
├── notebooks/
│   ├── 01_ingestao_api.ipynb
│   ├── 03_silver.ipynb
│   ├── 04_gold.ipynb
│   ├── 05_quality.ipynb
│   └── 06_analytics.ipynb
│
├── docs/
│   ├── 01_contexto_negocio.md
│   ├── 02_fontes_dados.md
│   ├── 03_arquitetura.md
│   ├── 04_modelagem.md
│   ├── 05_catalogo_dados.md
│   ├── 06_pipeline.md
│   ├── 07_qualidade_dados.md
│   ├── 08_analises.md
│   └── 09_autoavaliacao.md
│
├── catalog/
│   └── data_dictionary.xlsx
│
├── CHANGELOG.md
├── LICENSE
├── README.md
├── requirements.txt
└── .gitignore
```

Os notebooks representam a implementação executável do pipeline, enquanto a pasta `docs/` concentra a documentação metodológica e técnica do projeto.

O catálogo detalhado da camada Gold está disponível em:

`catalog/data_dictionary.xlsx`

---

## 12. Notebooks

### `01_ingestao_api.ipynb`

Responsável pela etapa de ingestão e preparação dos dados de origem para o pipeline.

### `03_silver.ipynb`

Executa os principais tratamentos, tipagens, padronizações, validações e o recorte dos dados para o município do Rio de Janeiro.

### `04_gold.ipynb`

Constrói o modelo dimensional da camada Gold, incluindo:

- `dim_tempo`;
- `dim_horario`;
- `dim_bairro`;
- `dim_regiao`;
- `fato_acidentes`.

Também realiza o enriquecimento territorial com a referência oficial do IPP/PCRJ.

### `05_quality.ipynb`

Executa os controles de qualidade, integridade e reconciliação do pipeline.

### `06_analytics.ipynb`

Utiliza a camada Gold para responder às perguntas de negócio e avaliar os impactos das limitações identificadas nos dados.

---

## 13. Documentação

A documentação detalhada está organizada em:

- [`docs/01_contexto_negocio.md`](docs/01_contexto_negocio.md) — contexto, objetivos, escopo e perguntas de negócio;
- [`docs/02_fontes_dados.md`](docs/02_fontes_dados.md) — fontes oficiais e estratégia de aquisição;
- [`docs/03_arquitetura.md`](docs/03_arquitetura.md) — arquitetura do pipeline;
- [`docs/04_modelagem.md`](docs/04_modelagem.md) — modelo dimensional;
- [`docs/05_catalogo_dados.md`](docs/05_catalogo_dados.md) — catálogo da camada Gold;
- [`docs/06_pipeline.md`](docs/06_pipeline.md) — implementação do pipeline;
- [`docs/07_qualidade_dados.md`](docs/07_qualidade_dados.md) — controles e resultados de qualidade;
- [`docs/08_analises.md`](docs/08_analises.md) — respostas às perguntas de negócio;
- [`docs/09_autoavaliacao.md`](docs/09_autoavaliacao.md) — avaliação do desenvolvimento, limitações e possíveis evoluções.

---

## 14. Reprodutibilidade

O projeto foi desenvolvido e executado no **Databricks Free Edition**, utilizando computação Serverless e tabelas Delta organizadas no catálogo `workspace`.

Os principais schemas utilizados são:

```text
workspace.bronze
workspace.silver
workspace.gold
```

As dependências Python registradas no projeto estão disponíveis em:

`requirements.txt`

A disponibilidade e a reprodução integral do pipeline dependem também do acesso às fontes públicas utilizadas e das funcionalidades disponibilizadas pelo ambiente Databricks.

---

## 15. Possíveis evoluções

Entre as extensões identificadas para trabalhos futuros estão:

- automatização da atualização das fontes;
- expansão dos controles de observabilidade;
- ampliação das regras de qualidade;
- análise mais detalhada de vítimas;
- análise dos tipos de veículos envolvidos;
- inclusão de variáveis de exposição;
- construção de indicadores relativos de risco;
- aprimoramento controlado do tratamento territorial;
- ampliação das visualizações analíticas.

Esses itens representam possíveis evoluções e não fazem parte do escopo implementado nesta versão do MVP.

---

## 16. Conclusão

O MVP implementa uma cadeia de dados completa:

```text
Fonte oficial
     ↓
Landing Zone
     ↓
Bronze
     ↓
Silver
     ↓
Gold
     ↓
Qualidade
     ↓
Analytics
```

O pipeline transforma dados públicos de acidentes em um modelo dimensional persistido e validado, preservando a rastreabilidade das fontes e explicitando as limitações encontradas.

Além da implementação técnica, o projeto demonstra a importância da avaliação semântica e da qualidade dos dados. Informações ausentes, diferenças de cobertura temporal e correspondências territoriais incertas foram quantificadas e documentadas em vez de serem ocultadas por imputações ou classificações não verificadas.

**Status do projeto: MVP concluído.**
