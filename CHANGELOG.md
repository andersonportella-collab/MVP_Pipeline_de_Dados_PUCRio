# Changelog

Registro das principais etapas de desenvolvimento do MVP de Engenharia de Dados.

## v1.0 — MVP final

- Pipeline de dados concluído no Databricks.
- Ingestão dos dados públicos oficiais do RENAEST/SENATRAN.
- Implementação da Landing Zone para preservação dos arquivos de origem.
- Persistência da camada Bronze em Delta.
- Implementação da camada Silver com limpeza, tipagem, padronização, validação e recorte para o município do Rio de Janeiro.
- Implementação da camada Gold utilizando modelo dimensional em esquema estrela.
- Criação das dimensões `dim_tempo`, `dim_horario`, `dim_bairro` e `dim_regiao`.
- Criação da tabela fato `fato_acidentes`.
- Integração da referência territorial oficial do IPP/PCRJ para enriquecimento por Região Administrativa.
- Utilização de correspondência exata após normalização determinística dos bairros.
- Implementação dos controles de qualidade, integridade referencial e reconciliação entre camadas.
- Validação de 55.046 acidentes nas camadas Bronze, Silver e Gold.
- Validação de zero duplicidades de `num_acidente` na tabela fato.
- Validação de zero chaves estrangeiras nulas na tabela fato.
- Validação de zero chaves estrangeiras órfãs no modelo Gold.
- Implementação das análises para resposta às perguntas de negócio.
- Documentação das limitações de cobertura territorial e temporal dos dados.
- Criação e atualização do catálogo de dados da camada Gold.
- Consolidação da documentação técnica e metodológica do projeto.
- Versionamento dos notebooks finais no GitHub.
- Atualização do README para refletir o estado final do MVP.

## v0.3 — Pipeline Bronze

- Implementação da ingestão dos arquivos oficiais.
- Configuração da Landing Zone.
- Persistência inicial das tabelas da camada Bronze.

## v0.2 — Ambiente

- Configuração do ambiente Databricks.
- Estruturação inicial dos schemas e recursos necessários ao pipeline.
- Preparação da estrutura do repositório.

## v0.1 — Estrutura inicial

- Criação da estrutura inicial do projeto.
- Definição preliminar do escopo do MVP.
- Criação da estrutura inicial de documentação e versionamento.
