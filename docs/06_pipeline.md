# 06 — Pipeline de Dados

## 1. Visão geral

O pipeline deste projeto foi implementado no Databricks Free Edition, utilizando Apache Spark, PySpark, Spark SQL, Delta Lake e Unity Catalog.

A arquitetura segue o padrão Medallion, com separação entre dados brutos, dados tratados e dados preparados para consumo analítico.

O fluxo implementado é:

RENAEST/SENATRAN  
→ Landing Zone  
→ Bronze  
→ Silver  
→ Gold  
→ Análises

Além da fonte principal RENAEST/SENATRAN, foi utilizada uma referência oficial do Instituto Pereira Passos / Prefeitura da Cidade do Rio de Janeiro (IPP/PCRJ) para enriquecimento territorial com Regiões Administrativas.

---

## 2. Ambiente de execução

O processamento foi realizado no Databricks Free Edition utilizando computação Serverless.

Os objetos foram organizados no catálogo `workspace`, nos seguintes schemas:

- `workspace.bronze`
- `workspace.silver`
- `workspace.gold`

Também foi criado o volume gerenciado:

`workspace.bronze.landing`

Caminho físico utilizado:

`/Volumes/workspace/bronze/landing`

Esse volume funciona como Landing Zone para os arquivos originais.

---

## 3. Fonte principal — RENAEST/SENATRAN

Foram utilizados dados públicos do Registro Nacional de Sinistros e Estatísticas de Trânsito — RENAEST, disponibilizados pela Secretaria Nacional de Trânsito — SENATRAN.

O conjunto utilizado foi:

`renaest_dabertos_20250512`

Foram carregados quatro arquivos CSV:

- `Acidentes_DadosAbertos_20250512.csv`
- `Localidade_DadosAbertos_20250512.csv`
- `TipoVeiculo_DadosAbertos_20250512.csv`
- `Vitimas_DadosAbertos_20250512.csv`

Os arquivos utilizam ponto e vírgula (`;`) como delimitador.

A análise deste MVP utiliza como recorte principal os acidentes ocorridos no município do Rio de Janeiro, identificado pelo código IBGE:

`3304557`

---

## 4. Landing Zone

Os quatro arquivos CSV foram carregados no volume:

`/Volumes/workspace/bronze/landing`

A Landing Zone preserva os arquivos originais antes das transformações realizadas pelo pipeline.

Essa separação permite manter uma referência próxima à fonte e desacoplar o armazenamento inicial das etapas posteriores de processamento.

---

## 5. Camada Bronze

A camada Bronze armazena os dados ingeridos em tabelas Delta, preservando-os próximos ao formato de origem.

Foram criadas as seguintes tabelas:

- `workspace.bronze.acidentes`
- `workspace.bronze.localidade`
- `workspace.bronze.tipo_veiculo`
- `workspace.bronze.vitimas`

Durante a ingestão foram adicionados metadados técnicos, incluindo:

- identificação do sistema de origem;
- data/hora de ingestão;
- caminho do arquivo de origem.

No Unity Catalog, o caminho do arquivo foi obtido por meio de:

`_metadata.file_path`

Essa abordagem foi utilizada porque `input_file_name()` não é suportado nesse contexto do Unity Catalog.

A Bronze não executa as regras analíticas do projeto. Seu objetivo é estabelecer uma camada persistida em Delta próxima aos dados recebidos.

---

## 6. Camada Silver

A camada Silver concentra as operações de limpeza, tipagem, padronização e validação.

Foram criadas:

- `workspace.silver.acidentes`
- `workspace.silver.localidade`
- `workspace.silver.tipo_veiculo`
- `workspace.silver.vitimas`

Para a tabela de acidentes, foi aplicado o recorte do município do Rio de Janeiro por meio do código IBGE `3304557`.

Após o recorte, foram obtidos:

**55.046 acidentes.**

Entre os principais tratamentos realizados estão:

- conversão de datas;
- tratamento e validação de horários;
- conversão de campos quantitativos;
- padronização de campos textuais;
- tratamento de marcadores conhecidos de ausência de informação;
- conversão desses marcadores para valores nulos quando aplicável;
- validação da unicidade de `num_acidente`;
- avaliação da completude dos atributos.

Os valores de horário foram validados antes da construção da dimensão temporal correspondente.

O campo bairro apresentou uma limitação relevante de qualidade. Após a normalização dos marcadores de ausência, foram identificados:

- 28.152 registros com bairro informado;
- 26.894 registros sem bairro;
- completude global de 51,14%.

Essa limitação foi preservada e documentada, em vez de serem criados valores territoriais sem evidência suficiente.

---

## 7. Enriquecimento territorial

O campo `regiao` existente no RENAEST representa uma região geográfica mais ampla e não corresponde às Regiões Administrativas do município do Rio de Janeiro.

Por esse motivo, foi utilizada uma referência oficial do IPP/PCRJ contendo bairros e suas respectivas Regiões Administrativas.

O processo de associação territorial utilizou normalização determinística dos nomes, incluindo operações como:

- remoção de espaços excedentes;
- conversão para caixa alta;
- normalização de acentuação para comparação.

A associação final utilizou apenas correspondências exatas após essa normalização.

Técnicas de similaridade textual foram utilizadas apenas de forma diagnóstica durante a avaliação dos dados e não foram utilizadas para atribuir automaticamente uma Região Administrativa.

Essa decisão evita introduzir associações territoriais potencialmente incorretas.

---

## 8. Camada Gold

A camada Gold organiza os dados em um modelo dimensional para consumo analítico.

Foram criadas cinco tabelas:

- `workspace.gold.fato_acidentes`
- `workspace.gold.dim_tempo`
- `workspace.gold.dim_horario`
- `workspace.gold.dim_bairro`
- `workspace.gold.dim_regiao`

A tabela `fato_acidentes` possui granularidade de um registro por acidente.

A dimensão `dim_tempo` permite análises por:

- data;
- ano;
- mês;
- dia;
- dia da semana;
- trimestre.

A dimensão `dim_horario` permite análises por:

- horário;
- hora;
- minuto;
- segundo;
- faixa horária.

A dimensão `dim_bairro` representa os bairros presentes nos dados tratados.

A dimensão `dim_regiao` representa as Regiões Administrativas obtidas a partir da referência oficial do IPP/PCRJ.

Para preservar a integridade referencial mesmo quando a informação territorial não está disponível, foram utilizados membros especiais:

- `id_bairro = -1` → `NAO INFORMADO`
- `id_regiao = -1` → `NAO INFORMADO`

---

## 9. Validação entre as camadas

Foi realizada reconciliação da quantidade de acidentes ao longo do pipeline.

Resultado:

| Etapa | Quantidade de acidentes |
|---|---:|
| Bronze — recorte Rio de Janeiro | 55.046 |
| Silver | 55.046 |
| Gold | 55.046 |

Portanto, não houve perda nem multiplicação de acidentes durante as transformações entre as camadas analisadas.

Também foram verificados na Gold:

- 0 duplicidades de `num_acidente`;
- 0 chaves estrangeiras nulas;
- 0 chaves estrangeiras órfãs.

---

## 10. Notebooks do pipeline

A implementação foi distribuída funcionalmente nos notebooks do projeto.

### `01_ingestao_api`

Responsável pela leitura dos arquivos oficiais, inspeção inicial, ingestão e persistência das tabelas Bronze.

Apesar do nome originalmente planejado mencionar API, a fonte oficial utilizada disponibiliza os dados necessários em arquivos CSV. Portanto, a implementação utiliza os arquivos oficiais diretamente, sem introduzir uma API artificial no processo.

### `03_silver`

Responsável pelos tratamentos, padronizações, tipagens, recorte municipal e persistência das tabelas Silver.

### `04_gold`

Responsável pela construção do modelo dimensional e persistência das tabelas Gold.

### `05_quality`

Responsável pelas verificações formais de qualidade, incluindo completude, unicidade, validade, consistência, integridade referencial e reconciliação entre camadas.

### `06_analytics`

Responsável pelas consultas e análises utilizadas para responder às perguntas de negócio do MVP.

---

## 11. Resultado do pipeline

O pipeline resultante mantém separação clara entre:

**Landing:** arquivos originais;

**Bronze:** dados ingeridos e persistidos em Delta próximos à origem;

**Silver:** dados tratados, tipados, padronizados e validados;

**Gold:** modelo dimensional preparado para análise;

**Analytics:** consultas voltadas às perguntas de negócio.

Essa organização permite rastrear a transformação dos dados desde os arquivos oficiais até os resultados analíticos apresentados no MVP.
