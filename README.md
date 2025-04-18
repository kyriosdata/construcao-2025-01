# Construção de Software (2025/1)

Apresenta projetos a serem utilizados nas atividades práticas da disciplina [Construção de Software](docs/plano-construcao-2025-01.pdf) (2025/01)
e permite acompanhar o [desempenho](https://docs.google.com/spreadsheets/d/1I3StF95ieTRd_je2aaprwDr5gfE4BAmdifdjKdcDRBA/edit?usp=sharing).

Os projetos são pertinentes
ao padrão [FHIR](https://hl7.org/fhir/R4). FHIR é um padrão para troca de dados em saúde. Alguns dos elementos relevantes:

- Recursos são "estruturas de dados". Por exemplo, para o registro de um paciente usa-se o recurso Patient. Uma instância serializada em JSON deste recurso pode ser obtida [aqui](https://www.hl7.org/fhir/R4/patient-example.json.html).
- Estas estruturas são representadas em JSON (além de XML e Turtle).
- Conjunto de instâncias de recursos são agrupados em arquivos .tgz (NPM Package).
- Informação em saúde é uma "rede" (grafo) de instâncias de recursos. Uma instância referencia outras. 

Opções:

- [FHIR Artifact Analyser](docs/faa.md)
- [FHIR Guard](docs/fg.md)
- [FHIR Unit Test](docs/fut.md)

# Sprint 4 (16/04, 18/04 e 23/04)

- Conclusão da arquitetura da aplicação. Os principais 
componentes já devem estar identificados, assim como a 
interação entre eles. Possivelmente, quase toda ela exercitada,
estendendo o objetivo da Sprint 3.

# Sprint 3 (04/04, 09/04 e 11/04)
- Código que exercita a arquitetura proposta para a aplicação.
- Nenhuma funcionalidade é esperada, mas a identificação de componentes e a conexão deve estar disponível e executável por meio de código.

# Sprint 2 (21/03)
- Proposta de plano de construção. Possivelmente conterá:
  - Uma análise dos requisitos (modelos, diagramas conceituais, ...)
  - Arquitetura (grandes componentes da aplicação a ser desenvolvida)
  - Tecnologias: Java, C# ou outra utilizada, bibliotecas, ferramentas como IDE, testes, ...
  - Repositório do projeto (github)
  - Critérios de aceitação (testes, medidas aplicáveis, ...)
  - Estratégia para comunicação entre membros da equipe (discord, ...)

# Sprint 1 (19/03)
- Esclarecimento de dúvidas.
- Identificação do projeto a ser realizado pelo grupo.
- Atualização da planilha (indicação do repositório empregado pelo grupo)

# 26/03
- Todos os elementos necessários para a implementação da aplicação são dominados? Se não,
trabalhar cada um deles neste período. Exemplo: operação para início de processo externo (gestão de processo), download de arquivos, interface gráfica, interface via linha de comandos, IDEs, ferramentas em geral e outros.
- Resultado esperado (1). Refinar o plano, em particular, com a identificação precisa das tecnologias a serem utilizadas.
- Resultado esperado (2). A organização da aplicação, componentes, está definida.
- Resultado esperado (3). Domínio das ferramentas necessárias (bibliotecas e outros).

