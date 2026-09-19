# 08 — Análise de Dados

## 1. Objetivo

Após a construção e validação das camadas Bronze, Silver e Gold, a camada Gold foi utilizada para responder às perguntas de negócio definidas para o MVP.

As análises foram executadas no notebook `06_analytics`, utilizando principalmente a tabela `workspace.gold.fato_acidentes` e suas dimensões.

Foram investigadas as seguintes questões:

1. Quais bairros apresentam maior quantidade de acidentes?
2. Existe variação dos acidentes entre os meses do ano?
3. Quais dias da semana apresentam maior frequência?
4. Quais horários concentram mais ocorrências?
5. Como os acidentes se distribuem entre as Regiões Administrativas?
6. Como a quantidade de acidentes evoluiu ao longo dos anos?
7. Quais problemas de qualidade foram identificados e como eles afetam as análises?

As conclusões são descritivas e representam os registros disponíveis no conjunto analisado. Não devem ser interpretadas automaticamente como medidas de risco ou relações causais.

---

## 2. Pergunta 1 — Quais bairros apresentam maior quantidade de acidentes?

Dos 55.046 acidentes presentes na tabela fato:

- 28.152 possuem bairro identificado;
- 26.894 não possuem bairro identificado;
- a cobertura territorial por bairro é de 51,14%.

Considerando somente os registros com bairro identificado, os 15 bairros com maior quantidade de acidentes foram:

| Posição | Bairro | Acidentes | % dos acidentes com bairro identificado |
|---:|---|---:|---:|
| 1 | CAMPO GRANDE | 1.618 | 5,75% |
| 2 | BARRA DA TIJUCA | 1.360 | 4,83% |
| 3 | BANGU | 1.011 | 3,59% |
| 4 | SANTA CRUZ | 809 | 2,87% |
| 5 | CENTRO | 797 | 2,83% |
| 6 | BONSUCESSO | 776 | 2,76% |
| 7 | RECREIO DOS BANDEIRANTES | 772 | 2,74% |
| 8 | GUARATIBA | 750 | 2,66% |
| 9 | REALENGO | 725 | 2,58% |
| 10 | TAQUARA | 549 | 1,95% |
| 11 | JACAREPAGUA | 547 | 1,94% |
| 12 | BOTAFOGO | 521 | 1,85% |
| 13 | TIJUCA | 510 | 1,81% |
| 14 | COPACABANA | 504 | 1,79% |
| 15 | MADUREIRA | 436 | 1,55% |

### Interpretação

Campo Grande apresentou a maior quantidade observada entre os acidentes com bairro identificado, seguido por Barra da Tijuca e Bangu.

Entretanto, esse ranking deve ser interpretado considerando a limitação de completude do atributo bairro.

A cobertura é particularmente baixa entre 2018 e 2020, quando praticamente não há informação territorial utilizável.

Portanto, o resultado representa a distribuição dos registros com bairro conhecido e não uma distribuição territorial completa de todos os acidentes ocorridos durante o período.

Além disso, a quantidade absoluta de acidentes não representa, isoladamente, risco de acidente em cada bairro.

---

## 3. Pergunta 2 — Existe variação dos acidentes entre os meses do ano?

A inspeção da cobertura mensal identificou que:

- 2018 possui 12 meses;
- 2019 possui 12 meses;
- 2020 possui 12 meses;
- 2021 possui 12 meses;
- 2022 possui 12 meses;
- 2023 possui 10 meses;
- 2024 possui 10 meses.

Para evitar que meses ausentes em 2023 e 2024 distorcessem a comparação, a análise principal de variação mensal utilizou somente os anos completos de 2018 a 2022.

| Mês | Média mensal de acidentes | Total 2018–2022 | Mínimo | Máximo |
|---:|---:|---:|---:|---:|
| 1 | 649,4 | 3.247 | 554 | 740 |
| 2 | 615,8 | 3.079 | 514 | 701 |
| 3 | 695,2 | 3.476 | 448 | 900 |
| 4 | 604,2 | 3.021 | 259 | 870 |
| 5 | 610,0 | 3.050 | 272 | 805 |
| 6 | 654,2 | 3.271 | 360 | 827 |
| 7 | 644,6 | 3.223 | 492 | 797 |
| 8 | 734,6 | 3.673 | 565 | 885 |
| 9 | 651,6 | 3.258 | 426 | 814 |
| 10 | 705,4 | 3.527 | 577 | 861 |
| 11 | 675,0 | 3.375 | 532 | 772 |
| 12 | 701,0 | 3.505 | 517 | 900 |

### Interpretação

Existe variação observável entre os meses.

Agosto apresentou a maior média mensal no período completo analisado, com 734,6 acidentes, seguido por outubro, dezembro e março.

As menores médias foram observadas em abril, maio e fevereiro.

Entretanto, os resultados não são suficientes para afirmar a existência de uma sazonalidade forte ou explicar as causas dessas diferenças.

A análise demonstra um padrão descritivo de variação mensal, cuja explicação exigiria outras variáveis e análises adicionais.

---

## 4. Pergunta 3 — Quais dias da semana apresentam maior frequência?

A distribuição observada foi:

| Dia da semana | Acidentes | Percentual |
|---|---:|---:|
| SEGUNDA-FEIRA | 8.248 | 14,98% |
| QUINTA-FEIRA | 8.211 | 14,92% |
| SEXTA-FEIRA | 8.181 | 14,86% |
| QUARTA-FEIRA | 7.795 | 14,16% |
| TERCA-FEIRA | 7.591 | 13,79% |
| DOMINGO | 7.552 | 13,72% |
| SABADO | 7.468 | 13,57% |

### Interpretação

Segunda-feira apresentou a maior quantidade observada, com 8.248 acidentes.

Entretanto, a distribuição entre os dias da semana é relativamente homogênea.

A diferença entre o dia com maior quantidade, segunda-feira, e o dia com menor quantidade, sábado, foi de 780 acidentes ao longo de todo o período.

Assim, não foi identificada uma concentração extremamente acentuada em um único dia da semana.

---

## 5. Pergunta 4 — Quais horários concentram mais ocorrências?

### Distribuição por faixa horária

| Faixa horária | Acidentes | Percentual |
|---|---:|---:|
| TARDE | 17.213 | 31,27% |
| NOITE | 15.910 | 28,90% |
| MANHA | 15.847 | 28,79% |
| MADRUGADA | 6.076 | 11,04% |

A tarde apresentou a maior participação entre as quatro faixas utilizadas.

A madrugada apresentou uma quantidade substancialmente menor que as demais faixas.

### Horas com maior frequência

| Hora | Acidentes | Percentual |
|---:|---:|---:|
| 19h | 3.349 | 6,08% |
| 18h | 3.348 | 6,08% |
| 17h | 3.145 | 5,71% |
| 15h | 2.952 | 5,36% |
| 16h | 2.866 | 5,21% |
| 8h | 2.809 | 5,10% |
| 12h | 2.798 | 5,08% |
| 10h | 2.776 | 5,04% |
| 20h | 2.769 | 5,03% |
| 7h | 2.753 | 5,00% |

### Interpretação

Na divisão ampla por faixas, a tarde concentrou a maior quantidade de acidentes, com 31,27%.

Quando a análise é realizada por hora exata, 19h apresentou a maior frequência, praticamente empatada com 18h.

Os resultados também mostram valores elevados no intervalo entre 17h e 19h.

Esses dados indicam concentração temporal observada, mas não permitem determinar sua causa. Uma investigação causal exigiria informações adicionais, como volume de tráfego, condições viárias e exposição ao trânsito.

---

## 6. Pergunta 5 — Como os acidentes se distribuem entre as Regiões Administrativas?

Dos 55.046 acidentes:

- 27.075 foram associados a uma Região Administrativa;
- a cobertura efetiva dessa associação foi de 49,19%.

Entre os acidentes associados, as maiores frequências foram:

| Posição | Região Administrativa | Acidentes | % dos associados |
|---:|---|---:|---:|
| 1 | BARRA DA TIJUCA | 2.397 | 8,85% |
| 2 | CAMPO GRANDE | 2.259 | 8,34% |
| 3 | JACAREPAGUA | 2.055 | 7,59% |
| 4 | MEIER | 1.798 | 6,64% |
| 5 | MADUREIRA | 1.560 | 5,76% |
| 6 | RAMOS | 1.356 | 5,01% |
| 7 | REALENGO | 1.316 | 4,86% |
| 8 | BANGU | 1.314 | 4,85% |
| 9 | BOTAFOGO | 1.165 | 4,30% |
| 10 | SANTA CRUZ | 1.149 | 4,24% |
| 11 | PENHA | 891 | 3,29% |
| 12 | GUARATIBA | 863 | 3,19% |
| 13 | CENTRO | 801 | 2,96% |
| 14 | INHAUMA | 793 | 2,93% |
| 15 | ILHA DO GOVERNADOR | 724 | 2,67% |

### Interpretação

Barra da Tijuca apresentou a maior quantidade de acidentes entre os registros associados a uma Região Administrativa, seguida por Campo Grande e Jacarepaguá.

Esse resultado representa frequência absoluta nos registros associados.

Ele não deve ser interpretado como ranking de risco entre Regiões Administrativas.

Para uma análise de risco seriam necessários denominadores de exposição adequados, como população, frota, extensão da malha viária ou volume de tráfego.

Além disso, a cobertura territorial de 49,19% limita a representatividade dessa comparação.

---

## 7. Pergunta 6 — Como a quantidade de acidentes evoluiu ao longo dos anos?

A quantidade observada por ano foi:

| Ano | Acidentes | Meses disponíveis | Média mensal |
|---:|---:|---:|---:|
| 2018 | 9.683 | 12 | 806,92 |
| 2019 | 9.449 | 12 | 787,42 |
| 2020 | 6.036 | 12 | 503,00 |
| 2021 | 6.918 | 12 | 576,50 |
| 2022 | 7.619 | 12 | 634,92 |
| 2023 | 7.799 | 10 | 779,90 |
| 2024 | 7.542 | 10 | 754,20 |

### Interpretação

Considerando os anos com 12 meses disponíveis, observa-se:

- pequena redução entre 2018 e 2019;
- queda acentuada em 2020;
- aumento em 2021;
- novo aumento em 2022.

Não é possível determinar, somente a partir deste conjunto de dados, a causa da redução observada em 2020.

Os totais de 2023 e 2024 exigem tratamento diferente, pois cada um possui somente 10 meses disponíveis no conjunto analisado.

Por esse motivo, seus totais anuais não devem ser comparados diretamente com os totais dos anos completos.

As médias mensais observadas para 2023 e 2024 são, respectivamente, 779,90 e 754,20 acidentes por mês, valores mais próximos dos níveis observados em 2018 e 2019 do que os totais anuais isolados poderiam sugerir.

---

## 8. Pergunta 7 — Quais problemas de qualidade foram identificados e como eles afetam as análises?

Os principais problemas identificados foram:

### 8.1 Ausência de bairro

Foram encontrados 26.894 acidentes sem bairro informado, correspondendo a 48,86% da tabela fato.

**Impacto:** redução da cobertura das análises por bairro e Região Administrativa.

### 8.2 Cobertura territorial desigual entre os anos

A completude de bairro foi:

- 2018: 0,00%;
- 2019: 0,01%;
- 2020: 0,25%;
- 2021: 94,12%;
- 2022: 90,85%;
- 2023: 98,08%;
- 2024: 93,53%.

**Impacto:** os resultados territoriais de toda a série não possuem representatividade temporal uniforme.

### 8.3 Bairros não associados à referência oficial

Foram encontrados 1.077 acidentes com bairro informado, mas sem associação exata com a referência territorial oficial.

Isso corresponde a 1,96% do total de acidentes.

Foram observados valores como nomes alternativos, abreviações, valores não padronizados e categorias sem correspondência direta.

**Impacto:** uma correção automática por similaridade poderia atribuir acidentes à Região Administrativa incorreta.

Por esse motivo, o pipeline preservou esses registros como não associados.

### 8.4 Cobertura mensal incompleta

Os anos de 2023 e 2024 possuem 10 meses disponíveis.

**Impacto:** os totais desses anos não são diretamente comparáveis aos totais de anos com 12 meses.

---

## 9. Síntese dos resultados

As análises permitiram identificar alguns padrões descritivos relevantes:

- Campo Grande apresentou a maior frequência entre os acidentes com bairro identificado;
- agosto apresentou a maior média mensal entre os anos completos de 2018 a 2022;
- segunda-feira apresentou a maior frequência entre os dias da semana, embora a distribuição seja relativamente homogênea;
- a tarde foi a faixa horária com maior quantidade de registros;
- 19h e 18h apresentaram as maiores frequências por hora;
- Barra da Tijuca apresentou a maior frequência entre os acidentes associados a Regiões Administrativas;
- entre os anos completos, houve queda relevante em 2020 e aumento em 2021 e 2022;
- a qualidade territorial é a principal limitação identificada para as análises espaciais.

Os resultados devem ser interpretados como frequências observadas no conjunto de dados disponível.

O MVP não estima causalidade nem risco individual ou territorial. Para esse tipo de análise seriam necessários dados adicionais de exposição e métodos analíticos específicos.

---

## 10. Conclusão analítica

O modelo Gold permitiu responder às sete perguntas definidas para o MVP e demonstrou a utilidade do pipeline para exploração temporal e territorial dos acidentes de trânsito.

Ao mesmo tempo, a análise evidenciou que a qualidade da fonte condiciona as conclusões possíveis.

A principal limitação está na informação territorial, especialmente entre 2018 e 2020. A cobertura mensal incompleta em 2023 e 2024 também exige cautela nas comparações anuais.

Essas limitações não foram ocultadas por imputações ou correções automáticas. Elas foram quantificadas, documentadas e incorporadas à interpretação dos resultados.
