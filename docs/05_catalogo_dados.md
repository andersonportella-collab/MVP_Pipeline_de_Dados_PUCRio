# 05 — Catálogo de Dados

## 1. Visão geral

A camada Gold do projeto foi estruturada como um modelo dimensional voltado à análise dos acidentes de trânsito registrados no município do Rio de Janeiro.

O modelo possui uma tabela fato central, `fato_acidentes`, e quatro dimensões:

- `dim_tempo`
- `dim_horario`
- `dim_bairro`
- `dim_regiao`

A granularidade da tabela fato é de um registro por acidente. Essa granularidade foi validada por meio do campo `num_acidente`, que apresentou 55.046 valores distintos para 55.046 registros, sem valores nulos ou duplicados.

O catálogo detalhado dos campos, tipos físicos, regras, relacionamentos e resultados de qualidade está disponível em `catalog/data_dictionary.xlsx`.

---

## 2. Tabela fato

### `workspace.gold.fato_acidentes`

Tabela central do modelo dimensional.

**Granularidade:** um registro representa um acidente.

**Quantidade de registros:** 55.046.

| Campo | Tipo | Papel | Descrição |
|---|---|---|---|
| `num_acidente` | string | Identificador | Identificador único do acidente |
| `id_tempo` | integer | FK | Referência à dimensão Tempo |
| `id_horario` | integer | FK | Referência à dimensão Horário |
| `id_bairro` | long | FK | Referência à dimensão Bairro |
| `id_regiao` | integer | FK | Referência à dimensão Região Administrativa |
| `qtde_acidente` | integer | Medida | Quantidade de acidentes representada pelo registro |
| `qtde_acid_com_obitos` | string | Medida/Atributo | Campo referente a acidente com óbito, preservado no tipo físico da Gold |
| `qtde_envolvidos` | integer | Medida | Quantidade de envolvidos |
| `qtde_feridosilesos` | integer | Medida | Quantidade de feridos/ilesos |
| `qtde_obitos` | integer | Medida | Quantidade de óbitos |
| `tipo_acidente` | string | Atributo | Tipo do acidente |
| `condicao_meteorologica` | string | Atributo | Condição meteorológica |
| `condicao_pista` | string | Atributo | Condição da pista |
| `fase_dia` | string | Atributo | Fase do dia |
| `tipo_pista` | string | Atributo | Tipo de pista |
| `tipo_pavimento` | string | Atributo | Tipo de pavimento |
| `tipo_rodovia` | string | Atributo | Tipo de rodovia |

O campo `qtde_acid_com_obitos` permaneceu fisicamente como `string` na Gold e não foi utilizado nas análises quantitativas principais.

---

## 3. Dimensão Tempo

### `workspace.gold.dim_tempo`

Representa as datas presentes nos registros de acidentes.

**Quantidade de registros:** 2.434.

| Campo | Tipo | Descrição |
|---|---|---|
| `data_acidente` | date | Data calendário |
| `id_tempo` | integer | Chave no formato `yyyyMMdd` |
| `ano` | integer | Ano |
| `mes` | integer | Mês |
| `dia` | integer | Dia |
| `dia_semana_num` | integer | Número do dia da semana |
| `dia_semana` | string | Nome do dia da semana |
| `trimestre` | integer | Trimestre |

---

## 4. Dimensão Horário

### `workspace.gold.dim_horario`

Representa os horários distintos encontrados nos acidentes.

**Quantidade de registros:** 1.319.

| Campo | Tipo | Descrição |
|---|---|---|
| `id_horario` | integer | Chave derivada de hora, minuto e segundo |
| `horario` | string | Horário no formato `HH:MM:SS` |
| `hora` | integer | Hora, entre 0 e 23 |
| `minuto` | integer | Minuto |
| `segundo` | integer | Segundo |
| `faixa_horaria` | string | `MADRUGADA`, `MANHA`, `TARDE` ou `NOITE` |

---

## 5. Dimensão Bairro

### `workspace.gold.dim_bairro`

Representa os valores distintos de bairro presentes nos dados tratados.

**Quantidade de registros:** 311.

| Campo | Tipo | Descrição |
|---|---|---|
| `id_bairro` | long | Chave determinística do bairro |
| `bairro` | string | Nome do bairro |

Foi criado o membro especial:

`id_bairro = -1` → `NAO INFORMADO`

Esse membro permite preservar acidentes sem bairro disponível sem produzir chaves órfãs na tabela fato.

---

## 6. Dimensão Região Administrativa

### `workspace.gold.dim_regiao`

Representa as Regiões Administrativas utilizadas na análise territorial.

**Quantidade de registros:** 34.

| Campo | Tipo | Descrição |
|---|---|---|
| `id_regiao` | integer | Código da Região Administrativa |
| `regiao_administrativa` | string | Nome da Região Administrativa |
| `area_planejamento` | integer | Área de Planejamento |

A associação entre bairro e Região Administrativa utilizou referência oficial do Instituto Pereira Passos / Prefeitura da Cidade do Rio de Janeiro.

Foi utilizada correspondência exata após normalização determinística dos nomes. Correspondências aproximadas não foram automaticamente aceitas.

Foi criado também o membro:

`id_regiao = -1` → `NAO INFORMADO`

---

## 7. Relacionamentos

A tabela `fato_acidentes` possui relacionamentos N:1 com as quatro dimensões:

- `fato_acidentes.id_tempo` → `dim_tempo.id_tempo`
- `fato_acidentes.id_horario` → `dim_horario.id_horario`
- `fato_acidentes.id_bairro` → `dim_bairro.id_bairro`
- `fato_acidentes.id_regiao` → `dim_regiao.id_regiao`

Os testes realizados identificaram:

- 0 chaves estrangeiras nulas;
- 0 chaves órfãs.

---

## 8. Linhagem dos dados

A linhagem principal do projeto é:

RENAEST/SENATRAN  
→ Landing Zone  
→ Bronze  
→ Silver  
→ Gold  
→ Análises

Na Bronze, os arquivos CSV oficiais são preservados próximos ao formato de origem e recebem metadados de ingestão.

Na Silver, são realizados recorte municipal, tipagem, padronização, tratamento de valores ausentes e validações.

Na Gold, os dados são reorganizados no modelo dimensional utilizado pelas análises.

Para a dimensão Região Administrativa foi adicionada uma referência oficial externa do IPP/PCRJ para estabelecer a relação entre bairros e Regiões Administrativas.

---

## 9. Considerações de qualidade relacionadas ao modelo

A completude do atributo bairro foi de 51,14% no conjunto total.

A correspondência territorial com a referência oficial apresentou:

- 27.075 acidentes associados a uma Região Administrativa (49,19%);
- 26.894 acidentes sem bairro informado (48,86%);
- 1.077 acidentes com bairro informado, mas sem correspondência exata com a referência oficial (1,96%).

Não foram realizadas correções automáticas baseadas apenas em similaridade textual, evitando atribuições territoriais não verificadas.

A reconciliação do recorte municipal apresentou:

`Bronze: 55.046 → Silver: 55.046 → Gold: 55.046`

Portanto, não houve perda nem multiplicação de acidentes durante as transformações consideradas.
