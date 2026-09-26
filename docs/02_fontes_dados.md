# 02 — Fontes de Dados

## 1. Estratégia de seleção das fontes

A seleção das fontes de dados priorizou três critérios:

1. origem oficial;
2. possibilidade de rastrear a procedência dos dados;
3. adequação ao problema de análise de acidentes de trânsito no município do Rio de Janeiro.

Durante o desenvolvimento foram investigadas fontes públicas relacionadas ao município e à administração territorial do Rio de Janeiro.

A fonte principal selecionada para os acidentes foi o Registro Nacional de Sinistros e Estatísticas de Trânsito — RENAEST, disponibilizado pela Secretaria Nacional de Trânsito — SENATRAN.

Posteriormente, foi necessária uma segunda fonte oficial para permitir o enriquecimento dos acidentes com as Regiões Administrativas do município. Para essa finalidade foi utilizada uma referência cartográfica oficial do Instituto Pereira Passos / Prefeitura da Cidade do Rio de Janeiro.

---

## 2. Fonte principal — RENAEST/SENATRAN

### Identificação

**Fonte:** Registro Nacional de Sinistros e Estatísticas de Trânsito — RENAEST  
**Órgão:** Secretaria Nacional de Trânsito — SENATRAN  
**Tipo:** dados públicos oficiais  
**Formato utilizado:** CSV  
**Conjunto utilizado:** `renaest_dabertos_20250512`

Página do conjunto de dados:

https://dados.transportes.gov.br/dataset/renaest

Página institucional do RENAEST:

https://www.gov.br/transportes/pt-br/assuntos/transito/conteudo-Senatran/registro-nacional-de-sinistros-e-estatisticas-de-transito

### Licenciamento e condições de uso

As fontes utilizadas neste projeto são disponibilizadas por órgãos públicos em iniciativas oficiais de dados abertos. O RENAEST/SENATRAN integra a política de disponibilização de estatísticas e dados abertos do Ministério dos Transportes, enquanto os dados territoriais do IPP/PCRJ são disponibilizados no contexto da política de dados abertos da Prefeitura da Cidade do Rio de Janeiro. A utilização no MVP preserva a identificação e a proveniência das fontes oficiais.  
A licença MIT presente no repositório refere-se **ao código-fonte e à documentação produzidos para este projeto**, não devendo ser interpretada como licença dos conjuntos de dados originais, cujas condições de disponibilização permanecem vinculadas aos respectivos órgãos públicos.

---

## 3. Arquivos utilizados

O conjunto disponibilizado pelo RENAEST contém diferentes arquivos relacionados aos acidentes.

Neste projeto foram ingeridos quatro arquivos:

### Acidentes

`Acidentes_DadosAbertos_20250512.csv`

Arquivo principal para o escopo analítico do MVP.

Na versão utilizada, possui aproximadamente 7,2 milhões de registros e 35 campos antes da aplicação do recorte municipal.

Entre os atributos disponíveis estão:

- identificador do acidente;
- data;
- horário;
- bairro;
- código IBGE;
- condição meteorológica;
- condição da pista;
- fase do dia;
- tipo de acidente;
- tipo de pista;
- tipo de pavimento;
- tipo de rodovia;
- quantidade de envolvidos;
- quantidade de feridos/ilesos;
- quantidade de óbitos.

### Localidade

`Localidade_DadosAbertos_20250512.csv`

Contém informações relacionadas às localidades presentes no RENAEST, incluindo atributos como:

- código IBGE;
- município;
- unidade federativa;
- região;
- população;
- frota.

O arquivo foi ingerido e tratado no pipeline, embora a tabela de acidentes seja a principal fonte do modelo analítico desenvolvido neste MVP.

### Tipo de Veículo

`TipoVeiculo_DadosAbertos_20250512.csv`

Contém informações relacionadas aos tipos e quantidades de veículos associados aos acidentes.

Entre os campos estão:

- `num_acidente`;
- `tipo_veiculo`;
- `qtde_veiculos`;
- indicador de veículo estrangeiro.

O arquivo foi incorporado às camadas Bronze e Silver, permanecendo disponível para possíveis extensões analíticas.

### Vítimas

`Vitimas_DadosAbertos_20250512.csv`

Contém informações relacionadas às pessoas envolvidas nos acidentes, incluindo atributos como:

- gênero;
- faixa etária;
- gravidade da lesão;
- tipo de envolvido;
- uso de equipamento de segurança;
- indicação de motorista;
- suspeita de álcool.

O arquivo também foi incorporado às camadas Bronze e Silver e pode sustentar análises futuras que não fazem parte do escopo principal da camada Gold deste MVP.

---

## 4. Recorte geográfico

A base nacional do RENAEST contém registros de diferentes localidades.

Para o escopo principal do projeto foi selecionado o município do Rio de Janeiro por meio do código IBGE:

`3304557`

Após esse recorte, a tabela Silver de acidentes contém:

**55.046 registros.**

O recorte por código IBGE foi preferido à comparação textual do nome do município, pois utiliza um identificador estruturado da localidade.

---

## 5. Período encontrado

Após o tratamento e a validação das datas, os acidentes utilizados no projeto apresentam o seguinte intervalo:

**Data mínima:** `2018-01-01`  
**Data máxima:** `2024-11-30`

A existência desse intervalo não significa que todos os meses estejam presentes em todos os anos.

A análise de cobertura identificou que:

- 2018 a 2022 possuem 12 meses;
- 2023 possui 10 meses;
- 2024 possui 10 meses.

Essa limitação é considerada nas análises temporais.

---

## 6. Limitações identificadas na fonte principal

A avaliação dos dados identificou limitações que afetam diretamente algumas análises.

### 6.1 Informação de bairro

O atributo bairro possui baixa completude quando todo o período é considerado.

Dos 55.046 acidentes:

- 28.152 possuem bairro identificado;
- 26.894 não possuem bairro;
- a completude global é de 51,14%.

A cobertura também varia significativamente entre os anos, sendo praticamente inexistente entre 2018 e 2020.

Essa característica limita análises territoriais realizadas sobre toda a série histórica.

### 6.2 Campo `regiao`

Durante a investigação dos dados foi verificado que o campo `regiao` existente na fonte RENAEST não representa as Regiões Administrativas do município do Rio de Janeiro.

Portanto, esse campo não foi utilizado para responder à pergunta relacionada às Regiões Administrativas.

Uma fonte territorial oficial complementar foi utilizada para essa finalidade.

### 6.3 Coordenadas geográficas

Durante a inspeção dos dados foram encontrados registros com valores de latitude e longitude sem utilidade para o georreferenciamento necessário ao projeto.

Por esse motivo, as coordenadas não foram utilizadas para determinar a Região Administrativa dos acidentes.

A estratégia territorial adotada foi baseada no bairro informado e em uma referência oficial de bairros e Regiões Administrativas.

### 6.4 Cobertura temporal

Foram identificados meses ausentes em 2023 e 2024.

Essa característica impede tratar os totais desses dois anos como diretamente equivalentes aos totais dos anos completos sem considerar a diferença de cobertura.

---

## 7. Fonte complementar — IPP/PCRJ

Para responder à pergunta sobre Regiões Administrativas foi necessário complementar o RENAEST com uma referência territorial oficial.

### Identificação

**Fonte:** Limites Administrativos  
**Órgão:** Instituto Pereira Passos / Prefeitura da Cidade do Rio de Janeiro  
**Tipo:** serviço cartográfico oficial  
**Tecnologia:** ArcGIS Feature Service

Camada utilizada:

https://pgeo3.rio.rj.gov.br/arcgis/rest/services/Cartografia/Limites_administrativos/FeatureServer/4

A camada consultada fornece atributos relacionados aos bairros e à organização administrativa do município.

Entre os campos utilizados no projeto estão:

- `nome`;
- `codbairro`;
- `regiao_adm`;
- `codra`;
- `area_plane`.

Foram obtidos 167 registros de bairros na referência consultada.

---

## 8. Motivo do enriquecimento territorial

A fonte principal permite identificar o bairro em parte dos acidentes, mas não fornece diretamente a Região Administrativa municipal necessária à pergunta de negócio.

A fonte do IPP/PCRJ foi utilizada para estabelecer a relação:

**Bairro do acidente → Região Administrativa**

Antes da associação, os textos foram submetidos a normalização determinística para reduzir diferenças puramente de representação.

Foram consideradas operações como:

- remoção de espaços excedentes;
- conversão para caixa alta;
- normalização de acentuação para comparação.

A associação definitiva utilizou somente correspondências exatas após essa normalização.

---

## 9. Tratamento de correspondências territoriais

A investigação mostrou que alguns bairros do RENAEST não apresentavam correspondência exata com a referência oficial.

Foram avaliadas técnicas de similaridade textual como diagnóstico.

Embora algumas sugestões fossem plausíveis, também foram observadas associações potencialmente incorretas.

Por esse motivo, similaridade textual não foi utilizada como mecanismo automático para determinar a Região Administrativa.

O resultado final foi:

| Situação | Acidentes | Percentual |
|---|---:|---:|
| Associados à referência oficial | 27.075 | 49,19% |
| Sem bairro informado | 26.894 | 48,86% |
| Bairro informado, sem associação exata | 1.077 | 1,96% |

Os registros sem associação segura foram mantidos como não informados na dimensão territorial, em vez de receberem uma classificação inferida.

---

## 10. Forma de aquisição dos dados

Os dados do RENAEST foram obtidos por meio dos arquivos públicos disponibilizados pela fonte oficial.

Embora uma API tenha sido considerada inicialmente no planejamento do projeto, não houve necessidade de criar ou forçar esse mecanismo de ingestão.

Como os dados necessários estavam oficialmente disponíveis em arquivos CSV, esses arquivos foram utilizados diretamente.

Os arquivos originais foram carregados na Landing Zone do Databricks e posteriormente persistidos na camada Bronze em formato Delta.

Essa decisão mantém o processo de ingestão simples e rastreável em relação aos arquivos publicados pela fonte.

---

## 11. Arquivos auxiliares

Também foi utilizado o dicionário de dados disponibilizado junto ao conjunto RENAEST como material de apoio para compreensão dos campos da fonte.

O catálogo produzido especificamente para este projeto está disponível em:

`catalog/data_dictionary.xlsx`

Esse catálogo documenta o modelo Gold implementado, seus campos, tipos físicos, relacionamentos, regras e resultados de qualidade.

---

## 12. Papel das fontes no pipeline

A utilização das fontes pode ser resumida da seguinte forma:

| Fonte | Papel |
|---|---|
| RENAEST/SENATRAN | Fonte principal dos acidentes e atributos relacionados |
| IPP/PCRJ | Enriquecimento oficial Bairro → Região Administrativa |

A linhagem principal do projeto é:

RENAEST/SENATRAN  
→ Landing  
→ Bronze  
→ Silver  
→ Gold

Para o enriquecimento territorial:

IPP/PCRJ  
→ normalização da referência territorial  
→ associação por bairro  
→ `dim_regiao` e `fato_acidentes`

---

## 13. Considerações metodológicas

A escolha das fontes e das regras de integração procurou privilegiar procedência, rastreabilidade e significado semântico.

Durante o desenvolvimento, nem todo dado disponível foi automaticamente utilizado.

Campos cuja semântica não correspondia ao conceito necessário, coordenadas que não forneciam informação geográfica utilizável e correspondências territoriais aproximadas consideradas inseguras foram excluídos dessas respectivas finalidades analíticas.

Essa abordagem evita aumentar artificialmente a cobertura do conjunto à custa da confiabilidade da informação produzida.
