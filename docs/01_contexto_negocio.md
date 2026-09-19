# 01 — Contexto de Negócio e Perguntas

## 1. Contexto

Os acidentes de trânsito constituem eventos relevantes para a gestão urbana, pois envolvem impactos humanos, sociais e operacionais e apresentam distribuição espacial e temporal que pode ser investigada a partir de dados públicos.

O município do Rio de Janeiro possui diferentes características territoriais, viárias e de circulação entre seus bairros e Regiões Administrativas. Nesse contexto, dados de acidentes podem ser utilizados para identificar padrões de frequência ao longo do tempo e do território.

Entretanto, transformar dados públicos em informação analítica exige mais do que a disponibilização dos arquivos de origem. É necessário construir um processo que permita ingerir, organizar, tratar, validar e modelar esses dados antes de utilizá-los em análises.

Este MVP aborda esse problema sob a perspectiva de Engenharia de Dados, construindo um pipeline Lakehouse para dados públicos de acidentes de trânsito do município do Rio de Janeiro.

## 2. Problema

Os dados utilizados no projeto são disponibilizados em arquivos públicos que precisam passar por etapas de ingestão, tratamento, validação e modelagem antes de serem utilizados analiticamente.

Além das questões técnicas, existem limitações relacionadas à qualidade e à semântica dos dados.

Entre os problemas identificados durante o desenvolvimento estão:

- valores ausentes em atributos relevantes;
- diferenças de completude entre períodos;
- necessidade de padronização de campos textuais;
- necessidade de validação de datas, horários e medidas;
- necessidade de garantir a unicidade do identificador do acidente;
- diferenças entre a informação territorial da fonte e a divisão administrativa utilizada pelo município do Rio de Janeiro;
- necessidade de preservar a rastreabilidade entre os dados de origem e as informações utilizadas nas análises.

Um exemplo relevante é o atributo territorial. O campo `regiao` presente no RENAEST não representa as Regiões Administrativas do município do Rio de Janeiro.

Para permitir esse tipo de análise, foi necessário investigar uma fonte territorial oficial complementar e estabelecer uma regra controlada de associação entre os bairros informados no RENAEST e as Regiões Administrativas oficiais.

## 3. Objetivo geral

Construir um pipeline de dados em ambiente cloud, seguindo uma arquitetura Lakehouse em camadas, para ingerir, tratar, validar, modelar e analisar dados públicos de acidentes de trânsito no município do Rio de Janeiro.

O pipeline deve permitir rastrear os dados desde os arquivos oficiais até uma camada analítica estruturada para responder às perguntas definidas para o MVP.

## 4. Objetivos específicos

Os objetivos específicos são:

- utilizar dados públicos oficiais como fonte principal;
- preservar os arquivos de origem em uma Landing Zone;
- persistir os dados ingeridos em uma camada Bronze;
- realizar limpeza, tipagem, padronização e validação na camada Silver;
- construir uma camada Gold baseada em modelo dimensional;
- enriquecer os dados com informação oficial de Região Administrativa quando houver correspondência territorial segura;
- implementar verificações de qualidade;
- validar a integridade e a reconciliação entre as camadas;
- utilizar a camada Gold para responder às perguntas de negócio;
- documentar a arquitetura, modelagem, catálogo, linhagem, regras de qualidade e limitações dos dados.

## 5. Perguntas de negócio

O projeto busca responder às seguintes perguntas:

### Pergunta 1
Quais bairros apresentam maior quantidade de acidentes de trânsito?

### Pergunta 2
Existe variação na quantidade de acidentes entre os meses do ano?

### Pergunta 3
Quais dias da semana apresentam maior frequência de acidentes?

### Pergunta 4
Quais horários concentram maior quantidade de ocorrências?

### Pergunta 5
Como os acidentes se distribuem entre as Regiões Administrativas do município do Rio de Janeiro?

### Pergunta 6
Como a quantidade de acidentes evoluiu ao longo dos anos disponíveis?

### Pergunta 7
Quais problemas de qualidade existem nos dados e como eles podem afetar as análises produzidas?

## 6. Escopo

O escopo principal do MVP considera registros de acidentes associados ao município do Rio de Janeiro, identificado no RENAEST pelo código IBGE:

`3304557`

O período efetivamente encontrado no conjunto analisado vai de:

`01/01/2018` a `30/11/2024`

A fonte principal é o RENAEST/SENATRAN.

Para o enriquecimento das Regiões Administrativas é utilizada uma referência territorial oficial do Instituto Pereira Passos / Prefeitura da Cidade do Rio de Janeiro.

## 7. Limites da interpretação

As análises produzidas neste projeto são predominantemente descritivas.

A quantidade de acidentes observada em determinado bairro ou Região Administrativa representa frequência nos registros disponíveis e não, isoladamente, uma medida de risco.

Uma comparação de risco territorial exigiria variáveis adicionais de exposição, como população, frota, extensão da malha viária ou volume de tráfego.

Também existem diferenças importantes de completude territorial entre os anos analisados. Essas limitações são avaliadas explicitamente na etapa de qualidade de dados e consideradas na interpretação dos resultados.

O projeto não busca estabelecer relações causais entre os padrões encontrados e possíveis fatores externos.

## 8. Resultado esperado

Ao final do MVP, espera-se obter uma cadeia de dados rastreável:

Fonte oficial  
→ Landing Zone  
→ Bronze  
→ Silver  
→ Gold  
→ Qualidade  
→ Análises

A camada final deve permitir responder às perguntas propostas utilizando dados tratados e documentados, mantendo explícitas as limitações identificadas durante o desenvolvimento.
