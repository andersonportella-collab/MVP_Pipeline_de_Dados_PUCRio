# 04 — Modelagem de Dados

## 1. Objetivo da modelagem

A camada Gold foi estruturada para disponibilizar os dados tratados em um formato adequado às perguntas analíticas definidas para o MVP.

Foi adotado um modelo dimensional em esquema estrela, no qual uma tabela fato central representa os acidentes e dimensões fornecem os principais eixos de análise.

O modelo implementado é composto por:

- `fato_acidentes`;
- `dim_tempo`;
- `dim_horario`;
- `dim_bairro`;
- `dim_regiao`.

A modelagem foi construída a partir dos dados tratados na camada Silver e do enriquecimento territorial proveniente da referência oficial do Instituto Pereira Passos / Prefeitura da Cidade do Rio de Janeiro.

---

## 2. Modelo conceitual

O modelo Gold pode ser representado da seguinte forma:


                         dim_tempo
                             |
                             |
                             |
dim_bairro ----------- fato_acidentes ----------- dim_horario
                             |
                             |
                             |
                        dim_regiao

---

A tabela fato_acidentes ocupa o centro do modelo.

As dimensões permitem analisar os acidentes segundo diferentes perspectivas:
| Dimensão      | Perspectiva analítica                          |
| ------------- | ---------------------------------------------- |
| `dim_tempo`   | Data, ano, mês, dia, dia da semana e trimestre |
| `dim_horario` | Horário, hora e faixa horária                  |
| `dim_bairro`  | Bairro informado para o acidente               |
| `dim_regiao`  | Região Administrativa e Área de Planejamento   |

## 3. Granularidade da tabela fato

A definição do grão é:

1 registro em fato_acidentes = 1 acidente identificado por num_acidente.

Essa definição foi validada durante os testes de qualidade.

Resultado:

registros na fato: 55.046;
valores distintos de num_acidente: 55.046;
duplicidades de num_acidente: 0;
valores nulos de num_acidente: 0.

Dessa forma, a granularidade esperada foi preservada na construção da Gold.

## 4. Tabela fato_acidentes

A tabela:

workspace.gold.fato_acidentes

centraliza as ocorrências utilizadas nas análises.

Ela possui 55.046 registros.

Chaves dimensionais

A tabela contém as seguintes chaves:

id_tempo;
id_horario;
id_bairro;
id_regiao.

Essas chaves permitem relacionar cada acidente às dimensões correspondentes.

Medidas e atributos

Além das chaves, foram preservados atributos e medidas relevantes ao escopo analítico:

num_acidente;
qtde_acidente;
qtde_acid_com_obitos;
qtde_envolvidos;
qtde_feridosilesos;
qtde_obitos;
tipo_acidente;
condicao_meteorologica;
condicao_pista;
fase_dia;
tipo_pista;
tipo_pavimento;
tipo_rodovia.

O campo qtde_acid_com_obitos permanece fisicamente como string no modelo implementado. Essa característica é registrada no catálogo para que a documentação represente o schema efetivamente persistido.

## 5. Dimensão Tempo

Tabela:

workspace.gold.dim_tempo

Quantidade de registros:

2.434

A dimensão representa as datas distintas existentes nos acidentes.
| Campo            | Tipo    | Descrição                               |
| ---------------- | ------- | --------------------------------------- |
| `data_acidente`  | date    | Data do acidente                        |
| `id_tempo`       | integer | Chave da dimensão no formato AAAAMMDD   |
| `ano`            | integer | Ano                                     |
| `mes`            | integer | Mês                                     |
| `dia`            | integer | Dia do mês                              |
| `dia_semana_num` | integer | Representação numérica do dia da semana |
| `dia_semana`     | string  | Nome do dia da semana                   |
| `trimestre`      | integer | Trimestre do ano                        |

A chave id_tempo é derivada deterministicamente da data no formato:

yyyyMMdd

Exemplo conceitual:

2024-03-15 → 20240315

A dimensão sustenta as análises por ano, mês e dia da semana.

## 6. Dimensão Horário

Tabela:

workspace.gold.dim_horario

Quantidade de registros:

1.319

A dimensão representa os horários distintos presentes nos acidentes.

| Campo           | Tipo    | Descrição                                    |
| --------------- | ------- | -------------------------------------------- |
| `id_horario`    | integer | Chave da dimensão no formato HHMMSS          |
| `horario`       | string  | Horário padronizado                          |
| `hora`          | integer | Hora                                         |
| `minuto`        | integer | Minuto                                       |
| `segundo`       | integer | Segundo                                      |
| `faixa_horaria` | string  | Classificação do horário em uma faixa do dia |

A chave é derivada do horário:

HHMMSS

As faixas horárias foram definidas no projeto como:
| Intervalo   | Faixa     |
| ----------- | --------- |
| 00:00–05:59 | MADRUGADA |
| 06:00–11:59 | MANHA     |
| 12:00–17:59 | TARDE     |
| 18:00–23:59 | NOITE     |

Essa dimensão permite analisar tanto horas específicas quanto períodos mais amplos do dia.

## 7. Dimensão Bairro

Tabela:

workspace.gold.dim_bairro

Quantidade de registros:

311

Campos:
| Campo       | Tipo   | Descrição                                       |
| ----------- | ------ | ----------------------------------------------- |
| `id_bairro` | long   | Chave da dimensão                               |
| `bairro`    | string | Bairro registrado/padronizado a partir da fonte |

A chave id_bairro é produzida deterministicamente a partir do nome do bairro utilizado no modelo.

Foi incluído um membro especial:

id_bairro = -1

com o significado:

NAO INFORMADO

Esse membro é utilizado quando o acidente não possui bairro disponível.

A adoção desse registro permite manter a integridade referencial da tabela fato sem inventar um bairro para registros cuja informação não existe na fonte.

Valores não padronizados da origem

A dimensão pode preservar valores provenientes da fonte que não correspondem diretamente a bairros oficiais.

Durante a exploração foram encontrados exemplos como valores não cadastrados, grafias alternativas e outros textos sem correspondência oficial.

Esses valores não foram corrigidos automaticamente por suposição.

A existência de um registro na dim_bairro representa o valor territorial disponível para análise, mas não significa necessariamente que esse valor tenha sido validado como um bairro oficial da Prefeitura.

Essa distinção é importante para compreender a diferença entre dim_bairro e o enriquecimento realizado em dim_regiao.

## 8. Dimensão Região Administrativa

Tabela:

workspace.gold.dim_regiao

Quantidade de registros:

34

Campos:
| Campo                   | Tipo    | Descrição                           |
| ----------------------- | ------- | ----------------------------------- |
| `id_regiao`             | integer | Código da Região Administrativa     |
| `regiao_administrativa` | string  | Nome da Região Administrativa       |
| `area_planejamento`     | integer | Área de Planejamento correspondente |

A dimensão é construída a partir da referência oficial do Instituto Pereira Passos / Prefeitura da Cidade do Rio de Janeiro.

Após padronização da referência, foram obtidas:

33 Regiões Administrativas oficiais;
1 membro especial.

O membro especial utiliza:

id_regiao = -1

e representa acidentes para os quais não foi possível determinar uma Região Administrativa com segurança.

## 9. Associação Bairro → Região Administrativa

A associação territorial exigiu uma regra específica de modelagem.

O RENAEST possui informação de bairro em parte dos registros, enquanto a referência oficial do IPP/PCRJ fornece a relação entre bairros e Regiões Administrativas.

O processo conceitual foi:
Bairro do RENAEST
        |
        v
Normalização determinística
        |
        v
Comparação com bairro oficial IPP/PCRJ
        |
        +---- correspondência exata ----> Região Administrativa
        |
        +---- sem correspondência ------> id_regiao = -1

A normalização tratou diferenças de representação textual, incluindo espaços, caixa e acentuação.

A correspondência utilizada na construção definitiva da Gold foi exata após essa normalização.

## 10. Decisão sobre correspondência aproximada

Durante o desenvolvimento foram investigadas distâncias textuais entre bairros sem correspondência exata e a referência oficial.

O objetivo foi avaliar se técnicas de fuzzy matching poderiam aumentar a cobertura territorial.

Foram encontrados casos em que a aproximação sugeria uma correspondência plausível, mas também foram identificados falsos positivos.

Por esse motivo, fuzzy matching não foi utilizado como regra de produção do modelo.

A decisão foi preservar:

id_regiao = -1

quando não existisse correspondência considerada segura.

Essa escolha prioriza precisão da associação em vez de aumentar artificialmente a cobertura territorial.

## 11. Cobertura territorial do modelo

A associação final à Região Administrativa produziu:
| Situação                              | Acidentes | Percentual |
| ------------------------------------- | --------: | ---------: |
| Bairro associado à referência oficial |    27.075 |     49,19% |
| Sem bairro informado                  |    26.894 |     48,86% |
| Bairro informado, mas não associado   |     1.077 |      1,96% |

Assim, a cobertura efetiva de Região Administrativa é:

49,19%

Essa limitação é mantida explicitamente no modelo e considerada nas análises.
## 12. Relacionamentos

Os relacionamentos da tabela fato são:
| Origem           | Chave        | Destino                  |
| ---------------- | ------------ | ------------------------ |
| `fato_acidentes` | `id_tempo`   | `dim_tempo.id_tempo`     |
| `fato_acidentes` | `id_horario` | `dim_horario.id_horario` |
| `fato_acidentes` | `id_bairro`  | `dim_bairro.id_bairro`   |
| `fato_acidentes` | `id_regiao`  | `dim_regiao.id_regiao`   |

Conceitualmente, cada dimensão pode estar relacionada a diversos acidentes.
dim_tempo   1 -------- N fato_acidentes
dim_horario 1 -------- N fato_acidentes
dim_bairro  1 -------- N fato_acidentes
dim_regiao  1 -------- N fato_acidentes

## 13. Integridade referencial

A integridade das chaves foi validada após a construção do modelo.

Resultados:
| Chave        | Nulos | Órfãos |
| ------------ | ----: | -----: |
| `id_tempo`   |     0 |      0 |
| `id_horario` |     0 |      0 |
| `id_bairro`  |     0 |      0 |
| `id_regiao`  |     0 |      0 |

Os membros especiais -1 são registros válidos das respectivas dimensões.

Portanto, um acidente sem bairro ou Região Administrativa identificada continua possuindo uma chave estrangeira válida.

## 14. Reconciliação da tabela fato

A quantidade de acidentes foi reconciliada entre as principais etapas:

| Camada                               | Registros |
| ------------------------------------ | --------: |
| Bronze — acidentes do Rio de Janeiro |    55.046 |
| Silver — acidentes                   |    55.046 |
| Gold — `fato_acidentes`              |    55.046 |

Diferença Bronze → Silver:

0 registros

Diferença Silver → Gold:

0 registros

A reconciliação fornece uma verificação adicional de que a modelagem dimensional não eliminou nem multiplicou acidentes.

## 15. Linhagem conceitual

A linhagem principal da tabela fato é:
RENAEST/SENATRAN
        |
        v
Landing
        |
        v
workspace.bronze.acidentes
        |
        v
workspace.silver.acidentes
        |
        v
workspace.gold.fato_acidentes

Para as dimensões temporais:
workspace.silver.acidentes
        |
        +---- data_acidente ----> dim_tempo
        |
        +---- hora_acidente ----> dim_horario

Para bairro:

workspace.silver.acidentes
        |
        v
bairro
        |
        v
dim_bairro

Para Região Administrativa:
workspace.silver.acidentes
        |
        +---- bairro ------------------+
                                       |
IPP/PCRJ                               |
   |                                   |
   v                                   v
Bairro → Região Administrativa ---- associação
                                       |
                                       v
                                  dim_regiao
                                       |
                                       v
                              fato_acidentes.id_regiao

## 16. Relação entre modelagem e perguntas de negócio

O modelo foi construído para suportar diretamente as perguntas definidas no projeto.

| Pergunta                            | Estruturas utilizadas                  |
| ----------------------------------- | -------------------------------------- |
| Acidentes por bairro                | `fato_acidentes` + `dim_bairro`        |
| Variação mensal                     | `fato_acidentes` + `dim_tempo`         |
| Acidentes por dia da semana         | `fato_acidentes` + `dim_tempo`         |
| Acidentes por horário               | `fato_acidentes` + `dim_horario`       |
| Acidentes por Região Administrativa | `fato_acidentes` + `dim_regiao`        |
| Evolução anual                      | `fato_acidentes` + `dim_tempo`         |
| Impactos de qualidade               | fato + dimensões + testes de qualidade |

Isso permite que as análises sejam executadas sobre estruturas preparadas para consumo, sem necessidade de reconstruir as principais regras de tratamento a cada consulta.

## 17. Limitações da modelagem

O modelo possui limitações que devem ser consideradas.

Cobertura de bairro

A completude global do bairro é de 51,14%, com diferenças relevantes entre os anos.

Cobertura de Região Administrativa

A associação territorial alcança 49,19% dos acidentes.

Frequência não representa risco

O modelo permite calcular quantidade de acidentes por território, mas não contém um denominador de exposição suficiente para transformar automaticamente essa frequência em risco.

Escopo das demais entidades RENAEST

Os arquivos de vítimas e tipos de veículos foram ingeridos e tratados até a Silver, mas não foram incorporados ao modelo Gold atual.

Isso foi uma decisão de escopo do MVP, cujo modelo analítico foi concentrado nas perguntas definidas sobre os acidentes.

## 18. Catálogo detalhado

A descrição campo a campo do modelo implementado está disponível em:

catalog/data_dictionary.xlsx

e em:

docs/05_catalogo_dados.md

Esses artefatos complementam este documento com:

tipos físicos;
descrições;
regras;
domínios;
linhagem;
relacionamentos;
informações de qualidade.

## 19. Resultado da modelagem

O resultado é um esquema estrela composto por uma tabela fato com 55.046 acidentes e quatro dimensões analíticas.

A modelagem mantém:

granularidade explícita;
chaves dimensionais válidas;
ausência de registros órfãos;
membros especiais para informação indisponível;
separação entre bairro informado e Região Administrativa oficialmente associada;
rastreabilidade entre Silver e Gold;
compatibilidade direta com as perguntas de negócio do MVP.

A estrutura final da camada Gold é:
workspace.gold
├── fato_acidentes
├── dim_tempo
├── dim_horario
├── dim_bairro
└── dim_regiao

