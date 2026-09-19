# 07 — Qualidade de Dados

## 1. Objetivo

A qualidade dos dados foi avaliada como uma etapa formal do pipeline, e não apenas como uma consequência das transformações realizadas na camada Silver.

As verificações foram implementadas no notebook `05_quality` e abrangeram as seguintes dimensões:

- completude;
- unicidade;
- validade temporal;
- validade de medidas quantitativas;
- consistência entre atributos;
- integridade referencial;
- consistência territorial;
- reconciliação entre camadas.

O objetivo foi identificar limitações que poderiam afetar as análises e verificar se as transformações realizadas no pipeline introduziram perdas, duplicidades ou inconsistências.

---

## 2. Completude dos dados

Foram avaliados atributos considerados relevantes para as análises do MVP.

Entre eles, o principal problema identificado foi a disponibilidade do bairro do acidente.

No conjunto Silver de acidentes do município do Rio de Janeiro foram encontrados:

| Indicador | Resultado |
|---|---:|
| Total de acidentes | 55.046 |
| Bairro informado | 28.152 |
| Bairro não informado | 26.894 |
| Completude do bairro | 51,14% |

Portanto, aproximadamente metade dos registros não possui informação de bairro utilizável.

Essa limitação afeta diretamente as análises territoriais por bairro e Região Administrativa.

---

## 3. Completude do bairro por ano

A completude do bairro não é uniforme ao longo da série histórica.

| Ano | Acidentes | Bairro informado | Bairro ausente | Completude |
|---:|---:|---:|---:|---:|
| 2018 | 9.683 | 0 | 9.683 | 0,00% |
| 2019 | 9.449 | 1 | 9.448 | 0,01% |
| 2020 | 6.036 | 15 | 6.021 | 0,25% |
| 2021 | 6.918 | 6.511 | 407 | 94,12% |
| 2022 | 7.619 | 6.922 | 697 | 90,85% |
| 2023 | 7.799 | 7.649 | 150 | 98,08% |
| 2024 | 7.542 | 7.054 | 488 | 93,53% |

Os dados demonstram uma mudança significativa na disponibilidade do atributo a partir de 2021.

Por esse motivo, rankings territoriais construídos com toda a série 2018–2024 não devem ser interpretados como se a cobertura de bairro fosse homogênea durante todo o período.

Em particular, 2018, 2019 e 2020 possuem cobertura territorial insuficiente para comparações equivalentes com os anos posteriores.

---

## 4. Tratamento de valores ausentes

Na camada Silver foram identificados e normalizados marcadores textuais utilizados pela fonte para representar ausência de informação.

Esses marcadores foram convertidos para valores nulos quando aplicável, evitando que categorias de ausência fossem interpretadas como categorias reais.

Não foram criados bairros artificialmente para substituir informações ausentes.

Na camada Gold, a ausência territorial foi representada por membros especiais das dimensões:

- `id_bairro = -1` → `NAO INFORMADO`;
- `id_regiao = -1` → `NAO INFORMADO`.

Essa estratégia preserva a integridade referencial sem fabricar informação territorial.

---

## 5. Unicidade do identificador do acidente

O campo `num_acidente` foi avaliado para verificar se a granularidade definida para a tabela fato era válida.

Resultado:

| Métrica | Resultado |
|---|---:|
| Registros | 55.046 |
| `num_acidente` distintos | 55.046 |
| Valores duplicados | 0 |
| Valores nulos | 0 |

Portanto, o campo sustenta a granularidade adotada no modelo:

**1 registro da tabela fato = 1 acidente.**

---

## 6. Validade temporal

Foram verificadas as datas e os horários dos acidentes após os tratamentos da camada Silver.

### Datas

O intervalo observado foi:

`2018-01-01` a `2024-11-30`

Não foram identificadas datas inválidas após o processo de tratamento.

### Horários

Os horários da fonte foram tratados e validados antes da construção da dimensão `dim_horario`.

Foram encontrados:

- 0 horários inválidos;
- 0 horários nulos no conjunto utilizado;
- horas entre 0 e 23.

Os valores válidos foram utilizados para construir a dimensão de horário e as respectivas faixas horárias.

---

## 7. Validade das medidas quantitativas

Foram avaliadas as principais medidas utilizadas no modelo.

| Campo | Nulos | Negativos | Mínimo | Máximo |
|---|---:|---:|---:|---:|
| `qtde_acidente` | 0 | 0 | 1 | 1 |
| `qtde_envolvidos` | 0 | 0 | 0 | 28 |
| `qtde_feridosilesos` | 0 | 0 | 0 | 28 |
| `qtde_obitos` | 0 | 0 | 0 | 4 |

Também foram verificadas relações entre essas medidas.

Não foram encontrados casos em que:

- `qtde_obitos > qtde_envolvidos`;
- `qtde_feridosilesos > qtde_envolvidos`;
- `qtde_obitos + qtde_feridosilesos > qtde_envolvidos`.

Os testes não identificaram violações dessas regras no conjunto analisado.

O campo `qtde_acid_com_obitos` permaneceu fisicamente como `string` na camada Gold e não foi incluído nessas validações quantitativas.

---

## 8. Integridade referencial da camada Gold

Foram verificadas as quatro chaves estrangeiras da tabela `fato_acidentes`:

- `id_tempo`;
- `id_horario`;
- `id_bairro`;
- `id_regiao`.

Resultado:

| Chave | Valores nulos | Chaves órfãs |
|---|---:|---:|
| `id_tempo` | 0 | 0 |
| `id_horario` | 0 | 0 |
| `id_bairro` | 0 | 0 |
| `id_regiao` | 0 | 0 |

Os membros especiais `-1` de bairro e Região Administrativa são registros válidos das respectivas dimensões e, portanto, não representam chaves órfãs.

---

## 9. Consistência territorial

Para permitir a análise por Região Administrativa, os bairros do RENAEST foram comparados com uma referência oficial do IPP/PCRJ.

A associação definitiva utilizou correspondência exata após normalização determinística dos textos.

O resultado sobre os 55.046 acidentes foi:

| Situação | Acidentes | Percentual |
|---|---:|---:|
| Bairro associado à referência oficial | 27.075 | 49,19% |
| Sem bairro informado | 26.894 | 48,86% |
| Bairro informado, mas não associado | 1.077 | 1,96% |

A soma percentual pode apresentar pequena diferença decorrente de arredondamento.

Entre os valores não associados foram observados casos como nomes alternativos, abreviações, valores não padronizados e categorias que não correspondem diretamente aos nomes oficiais da referência.

Durante a investigação foram avaliadas similaridades textuais. Entretanto, a correspondência aproximada produziu também associações potencialmente incorretas.

Por esse motivo, similaridade textual não foi utilizada como regra automática de enriquecimento.

Essa decisão privilegia a rastreabilidade e evita atribuir uma Região Administrativa sem evidência suficiente.

---

## 10. Reconciliação entre camadas

Foi realizada uma reconciliação da quantidade de acidentes do município do Rio de Janeiro ao longo das principais etapas do pipeline.

| Camada | Quantidade |
|---|---:|
| Bronze — recorte Rio de Janeiro | 55.046 |
| Silver | 55.046 |
| Gold — `fato_acidentes` | 55.046 |

Diferença Bronze → Silver:

`0`

Diferença Silver → Gold:

`0`

O teste demonstra que as transformações não provocaram perda nem multiplicação de acidentes.

---

## 11. Cobertura temporal dos dados

Além da validade individual das datas, foi avaliada a cobertura dos meses disponíveis em cada ano.

Os anos de 2018 a 2022 apresentam 12 meses no conjunto analisado.

Em contrapartida:

- 2023 possui 10 meses;
- 2024 possui 10 meses.

Na inspeção mensal foram identificadas ausências de:

- maio e agosto de 2023;
- abril e dezembro de 2024.

Essa característica afeta comparações baseadas em totais anuais.

Por esse motivo, os totais de 2023 e 2024 não são interpretados como diretamente equivalentes aos totais dos anos com 12 meses disponíveis.

---

## 12. Impactos sobre as análises

Os testes de qualidade produziram limitações que precisam acompanhar a interpretação dos resultados.

### Análises por bairro

A cobertura total é de 51,14%. Os rankings representam somente os acidentes para os quais existe bairro identificado.

### Análises por Região Administrativa

A cobertura efetivamente associada a uma Região Administrativa é de 49,19%.

Esses resultados representam frequência de acidentes associados e não devem ser interpretados como medida de risco territorial.

### Comparações históricas territoriais

A ausência quase total de bairro entre 2018 e 2020 impede considerar a série territorial como homogênea durante todo o período.

### Evolução anual

2023 e 2024 possuem apenas 10 meses disponíveis no conjunto analisado. Seus totais anuais não devem ser comparados diretamente aos anos completos sem considerar essa diferença de cobertura.

---

## 13. Síntese da avaliação

| Dimensão | Resultado | Situação |
|---|---|---|
| Completude geral dos atributos críticos avaliados | Adequada, exceto bairro | ATENÇÃO |
| Completude de bairro | 51,14% | ATENÇÃO |
| Unicidade de `num_acidente` | 0 duplicidades | OK |
| Validade temporal | 0 inconsistências identificadas | OK |
| Validade quantitativa | 0 violações identificadas nos testes realizados | OK |
| Integridade referencial Gold | 0 chaves órfãs | OK |
| Associação territorial | 49,19% dos acidentes associados a RA | ATENÇÃO |
| Reconciliação entre camadas | 55.046 → 55.046 → 55.046 | OK |
| Cobertura anual 2023/2024 | 10 meses em cada ano | ATENÇÃO |

A avaliação mostra que o pipeline apresenta consistência estrutural e integridade entre as camadas, mas existem limitações relevantes nos próprios dados de origem, principalmente relacionadas à cobertura territorial e à disponibilidade mensal.

Essas limitações foram mantidas explicitamente no projeto e consideradas na interpretação dos resultados, em vez de serem ocultadas por imputações ou correções sem evidência suficiente.
