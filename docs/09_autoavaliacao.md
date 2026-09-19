# 09 — Autoavaliação

## 1. Avaliação geral do trabalho

O desenvolvimento deste MVP exigiu mais do que a implementação técnica de um pipeline de dados. Uma parte relevante do trabalho esteve relacionada à investigação das fontes, compreensão do significado dos atributos, avaliação da qualidade dos dados e definição de critérios que permitissem produzir análises rastreáveis sem introduzir informações não sustentadas pelas fontes.

O resultado foi um pipeline Lakehouse executado no Databricks, estruturado em Landing Zone e camadas Bronze, Silver e Gold, com validações de qualidade e uma camada analítica destinada a responder às perguntas de negócio propostas.

A implementação final processa os dados oficiais do RENAEST/SENATRAN e utiliza uma fonte oficial do Instituto Pereira Passos / Prefeitura da Cidade do Rio de Janeiro para enriquecimento territorial.

## 2. Busca e avaliação das fontes de dados

Uma das etapas que demandou maior investigação foi a identificação das fontes adequadas para o problema.

A intenção inicial era trabalhar com dados públicos oficiais relacionados aos acidentes de trânsito no município do Rio de Janeiro. Durante essa etapa, foram pesquisadas alternativas disponibilizadas por órgãos públicos, incluindo fontes e referências da Prefeitura da Cidade do Rio de Janeiro.

A escolha do RENAEST/SENATRAN como fonte principal ocorreu por se tratar de uma base pública oficial, estruturada e com registros de acidentes que permitiam o recorte do município do Rio de Janeiro por código IBGE.

Os arquivos oficiais disponíveis foram preservados como fonte de ingestão do pipeline, sem a criação artificial de uma API quando os dados necessários já estavam disponíveis em CSV.

A investigação das fontes continuou durante o desenvolvimento. Em particular, verificou-se que o atributo `regiao` existente no RENAEST não representava as Regiões Administrativas do município do Rio de Janeiro. Utilizá-lo com essa interpretação produziria uma classificação territorial conceitualmente incorreta.

Por esse motivo, foram pesquisadas fontes oficiais da Prefeitura para estabelecer a relação entre bairros e Regiões Administrativas. A referência estruturada utilizada foi obtida por meio do serviço cartográfico oficial do Instituto Pereira Passos / Prefeitura da Cidade do Rio de Janeiro.

Essa etapa evidenciou a importância de não avaliar uma fonte somente pela existência de um campo com determinado nome, mas também pelo significado semântico desse campo no contexto do problema.

## 3. Rigor metodológico no tratamento territorial

O enriquecimento territorial foi uma das decisões metodológicas mais relevantes do projeto.

Após a obtenção da referência oficial de bairros e Regiões Administrativas, foi realizada a comparação entre os bairros registrados no RENAEST e os nomes existentes na fonte oficial.

Os textos foram submetidos a normalizações determinísticas para permitir comparações consistentes, como tratamento de espaços, caixa e acentuação.

Também foram investigadas técnicas de similaridade textual para os bairros que não apresentavam correspondência exata.

Essa investigação mostrou que algumas sugestões aproximadas eram plausíveis, mas também produzia falsos positivos. Dessa forma, uma regra automática baseada exclusivamente em distância textual poderia atribuir um acidente à Região Administrativa incorreta.

A decisão final foi utilizar somente correspondências exatas após normalização determinística.

Como consequência, 1.077 acidentes que possuíam algum texto no campo bairro permaneceram sem associação a uma Região Administrativa quando não havia correspondência segura com a referência oficial.

Essa escolha reduz a cobertura territorial do modelo, mas evita aumentar artificialmente a cobertura por meio de classificações não verificadas.

## 4. Qualidade dos dados como parte da análise

Outro aprendizado relevante foi que qualidade de dados não deve ser tratada somente como uma etapa de limpeza.

Os testes mostraram que apenas 51,14% dos acidentes possuíam bairro identificado no conjunto completo.

Além disso, a completude territorial apresentou forte diferença entre os anos:

- 2018: 0,00%;
- 2019: 0,01%;
- 2020: 0,25%;
- 2021: 94,12%;
- 2022: 90,85%;
- 2023: 98,08%;
- 2024: 93,53%.

Essa diferença altera diretamente o significado de qualquer análise territorial realizada sobre todo o período.

Em vez de preencher os valores ausentes por suposição ou ignorar essa limitação, o problema foi quantificado e incorporado às conclusões do projeto.

O mesmo princípio foi aplicado à cobertura temporal. A análise identificou que 2023 e 2024 possuem apenas 10 meses disponíveis no conjunto utilizado. Por isso, seus totais anuais não foram tratados como diretamente comparáveis aos anos com 12 meses.

Essas situações reforçaram uma conclusão importante do trabalho: uma transformação tecnicamente correta não garante, por si só, uma análise metodologicamente correta. É necessário compreender a cobertura e as limitações dos dados utilizados.

## 5. Aspectos técnicos alcançados

O MVP implementou um fluxo completo de engenharia de dados:

**Fonte → Landing → Bronze → Silver → Gold → Qualidade → Analytics**

Entre os resultados técnicos alcançados estão:

- utilização de dados públicos oficiais;
- armazenamento dos arquivos originais em Landing Zone;
- persistência das camadas utilizando Delta Lake;
- organização dos objetos por schemas no Unity Catalog;
- tratamento e padronização com PySpark;
- construção de modelo dimensional;
- criação de tabela fato e quatro dimensões;
- enriquecimento territorial com fonte oficial externa;
- implementação de testes de qualidade;
- validação da integridade referencial;
- reconciliação entre as camadas;
- execução de análises diretamente sobre a camada Gold;
- documentação das regras, limitações e linhagem dos dados.

A reconciliação resultou em:

`Bronze: 55.046 → Silver: 55.046 → Gold: 55.046`

Também foram obtidos:

- 0 duplicidades de `num_acidente`;
- 0 chaves estrangeiras nulas na tabela fato;
- 0 chaves estrangeiras órfãs.

Esses resultados fornecem evidências de que o pipeline preservou a granularidade dos acidentes durante as principais transformações.

## 6. Limitações do projeto

A principal limitação encontrada está na qualidade territorial da própria fonte.

Dos 55.046 acidentes analisados, 26.894 não possuem bairro informado. Isso restringe a cobertura das análises por bairro e Região Administrativa.

Outra limitação é a existência de 1.077 acidentes com algum valor de bairro, mas sem correspondência exata com a referência territorial oficial utilizada.

Também existe diferença de cobertura temporal, pois 2023 e 2024 possuem somente 10 meses disponíveis no conjunto analisado.

Por fim, as análises realizadas são baseadas principalmente em frequências absolutas.

Consequentemente, resultados como quantidade de acidentes por bairro ou Região Administrativa não devem ser interpretados como medidas de risco. Uma análise desse tipo exigiria denominadores de exposição adequados, como população, frota, extensão viária ou volume de tráfego.

## 7. Possíveis evoluções

Como continuidade do trabalho, algumas evoluções poderiam ser avaliadas sem alterar os resultados já obtidos neste MVP.

Uma possibilidade seria incorporar variáveis de exposição, permitindo desenvolver indicadores relativos em vez de trabalhar somente com frequências absolutas.

Também seria possível aprofundar o tratamento territorial utilizando regras adicionais previamente validadas ou outras referências oficiais, mantendo o princípio de não realizar associações automáticas sem evidência suficiente.

Outras evoluções possíveis incluem:

- automatização da atualização da fonte;
- criação de controles adicionais de observabilidade do pipeline;
- expansão das regras de qualidade;
- análise mais detalhada de vítimas e tipos de veículos;
- construção de indicadores normalizados de exposição;
- ampliação das visualizações analíticas.

Essas possibilidades são tratadas como evoluções futuras e não como componentes implementados no escopo atual.

## 8. Aprendizados

O principal aprendizado do projeto foi que engenharia de dados envolve decisões sobre significado, qualidade e rastreabilidade, além da implementação do processamento.

A investigação das fontes mostrou que a existência de dados públicos não elimina a necessidade de avaliar sua estrutura, cobertura e semântica.

A construção do enriquecimento territorial também mostrou que aumentar a quantidade de registros classificados não significa necessariamente aumentar a qualidade do dado. Em determinadas situações, manter um registro como não associado é metodologicamente mais adequado do que produzir uma classificação incerta.

Da mesma forma, os testes de qualidade demonstraram que limitações da fonte precisam acompanhar as análises produzidas a partir dela.

O MVP, portanto, não buscou apenas produzir tabelas e consultas, mas estabelecer uma cadeia rastreável entre fonte, transformação, validação e interpretação dos resultados.

## 9. Conclusão da autoavaliação

Os objetivos definidos para o MVP foram atendidos por meio da construção de um pipeline de dados em ambiente cloud, desde a ingestão de dados públicos oficiais até a geração de um modelo dimensional e sua utilização em análises.

O desenvolvimento também revelou limitações importantes nas fontes utilizadas, principalmente relacionadas à completude territorial e à cobertura temporal.

Em vez de ocultar essas limitações, o projeto procurou identificá-las, mensurá-las e incorporá-las às decisões de modelagem e às conclusões analíticas.

Considero que o aspecto mais relevante do trabalho foi justamente a combinação entre implementação técnica e cuidado metodológico: buscar fontes oficiais, compreender semanticamente os atributos, testar a qualidade dos dados, evitar correções não verificadas e documentar explicitamente as limitações das análises.
