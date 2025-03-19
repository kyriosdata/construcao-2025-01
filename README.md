# Construção de Software (2025/1)

Apresenta projetos a serem utilizados nas atividades práticas da disciplina [Construção de Software](docs/plano-construcao-2025-01.pdf) (2025/01).

Todos eles pertinentes
ao padrão [FHIR](https://hl7.org/fhir/R4). FHIR é um padrão para troca de dados em saúde. Alguns dos elementos relevantes:

- Recursos são "estruturas de dados". Por exemplo, para o registro de um paciente usa-se o recurso Patient. Uma instância serializada em JSON deste recurso pode ser obtida [aqui](https://www.hl7.org/fhir/R4/patient-example.json.html).
- Estas estruturas são representadas em JSON (além de XML e Turtle).
- Conjunto de instâncias de recursos são agrupados em arquivos .tgz (NPM Package).
- Informação em saúde é uma "rede" (grafo) de instâncias de recursos. Uma instância referencia outras. 

Opções:

- [FHIR Artifact Analyser](docs/faa.md)
- [FHIR Guard](docs/fg.md)
- [FHIR Unit Test](docs/fut.md)

Definição de contas no github e formação dos grupos: preencha a [planilha](https://docs.google.com/spreadsheets/d/1I3StF95ieTRd_je2aaprwDr5gfE4BAmdifdjKdcDRBA/edit?usp=sharing).

# Sprint 1 (19/03)
- Esclarecimento de dúvidas.
- Identificação do projeto a ser realizado pelo grupo.
- Atualização da planilha (indicação do repositório empregado pelo grupo)

# Sprint 2 (21/03)
- Proposta de plano de construção. Possivelmente conterá:
  - Uma análise dos requisitos (modelos, diagramas conceituais, ...)
  - Arquitetura (grandes componentes da aplicação a ser desenvolvida)
  - Tecnologias: Java, C# ou outra utilizada, bibliotecas, ferramentas como IDE, testes, ...
  - Repositório do projeto (github)
  - Critérios de aceitação (testes, medidas aplicáveis, ...)
  - Estratégia para comunicação entre membros da equipe (discord, ...)
